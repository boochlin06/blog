# Android 爛裝置想跑人臉辨識 – GC Thrashing 思路補充

> 原文網址：https://boochlin.com/?p=933
> 發布日期：2026-07-07
> 文章編號：933

---

在我們那台只有 2GB RAM、CPU 效能孱弱的嵌入式門禁機上跑人臉辨識，開發者的日常基本上就是跟系統在懸崖邊搶記憶體。

一切的開端，源自於一段讓我們毛骨悚然的 Logcat。

某天，我們發現門禁機連續運行幾小時後，畫面開始劇烈掉幀、辨識卡成幻燈片。系統沒有 OOM 閃退，但接上 adb 打開 logcat 一看，畫面正在以每秒數十行的速度瘋狂洗頻：

```text
I/art: Background concurrent copying GC freed 50(2KB) AllocSpace objects, 0(0B) LOS objects, 11% free, 21MB/23MB, paused 2.1ms total 25.1ms
I/art: Background concurrent copying GC freed 12(1KB) AllocSpace objects, 0(0B) LOS objects, 10% free, 21MB/23MB, paused 1.8ms total 22.3ms
```

這在系統層面叫做 **GC Thrashing（垃圾回收顛簸）**。

注意看那個 `freed` 的數字：ART 的垃圾回收器正處於極度恐慌的狀態，每 20 毫秒就拚老命跑一次 GC，每次卻只能清出微不足道的 1~2 KB 垃圾。CPU 算力全部被 GC 搶光，AI 模型根本分不到時間運算。

我們用 Memory Profiler 抓下 Heap Dump，程式碼明明沒有 Memory Leak，但卻看到幾千個 OpenCV 的 `Mat` 和 ToF 點雲包裝物件，全部卡在一個叫 `java.lang.ref.FinalizerReference` 的隊列裡大排長龍。

這讓我們當場傻眼：**「等等，我們不是早就升到 Android 10 了嗎？官方不是早就全面改用 Cleaner 了嗎？為什麼 Finalizer 的幽靈還在？」**

---

## 歷史幽靈：Android 10 為什麼還有 Finalizer？

很多開發者都有一個美麗的誤解：以為 Android 8.0 把 `Bitmap` 改用 `NativeAllocationRegistry` 之後，`finalize()` 這個歷史包袱就已經徹底從 Android 世界上消失了。

**現實給了我們一記重拳。**

在 Android 10 中，真相是這樣的：

* **AOSP 系統類別確實換了**：`Bitmap`、`Paint`、`Surface` 確實全面遷移到了 `Cleaner` / `NativeAllocationRegistry`。
* **ART 虛擬機依然保留相容機制**：`FinalizerDaemon`（單執行緒執行隊列）與 `FinalizerWatchdogDaemon`（10 秒看門狗）依然常駐在背景。
* **最致命的刺客：第三方函式庫與舊 JNI 封裝**：許多第三方 C++ 封裝庫（例如某些版本的 OpenCV `Mat`、舊版相機 HAL 包裝層），為了防止開發者忘記手動釋放 Native 記憶體，底層依然寫著 `protected void finalize()` 來當作安全網。

在一般手機 App 上，偶爾 new 幾個 Mat 根本感覺不出差別。

但在我們這台每秒吞吐 30 幀高頻影像的門禁機上，每秒產生幾百個帶有 `finalize()` 的影像與點雲物件。**只要 Class 宣告了 `finalize()`，Android 10 的 ART 在 new 物件時，就會無條件為它建立一個 `FinalizerReference` 塞進隊列！**

---

## 不是洩漏，是嚴重塞車

看一眼我們以前自以為聰明的舊代碼，你就知道問題出在哪：

```java
// 以前自以為省事的寫法：把釋放責任推給 GC
public class TofBufferWrapper {
    private long nativePtr; // 指向 C++ 幾十 MB 的 16-bit 點雲

    public TofBufferWrapper(long ptr) {
        this.nativePtr = ptr;
    }

    // 毒瘤所在：試圖靠 finalize() 幫我自動呼叫 C++ free
    @Override
    protected void finalize() throws Throwable {
        try {
            if (nativePtr != 0) {
                nativeRelease(nativePtr); // JNI 呼叫 C++ free
            }
        } finally {
            super.finalize();
        }
    }
}
```

這段代碼在低階硬體上引爆了連環車禍：

1. **獨木橋效應**：全 App 只有唯一一條執行緒（`FinalizerDaemon`）負責執行所有物件的 `finalize()`。在 1.3GHz 的破 CPU 上，單一執行緒根本消化不完每一幀幾十萬像素的圖像殘骸。
2. **記憶體殭屍化**：當一個物件被 GC 判定不可達時，它**不會當場被釋放**，而是被送進 Finalizer 隊列排隊等待。這意味著它至少要多苟延殘喘一輪 GC。空間清不出來，Java Heap 被佔滿，GC 就更頻繁觸發，直接引爆 GC Thrashing。
3. **隱藏的 10 秒死刑**：`FinalizerWatchdogDaemon` 只要發現某個物件的 `finalize()` 執行超過 10 秒（在 CPU 被 AI 與 GC 搶光時極容易發生），看門狗就會直接噴出 `TimeoutException`，無情地把 App 整個進程槍斃。
4. **GC 盲區**：Java 的 `TofBufferWrapper` 只有幾十 bytes，Java Heap 覺得天下太平；但底層 C++ 早就堆了幾百 MB 的點雲，直接觸發 LMKD 殺進程。

---

## 徹底拆彈：從 Finalize 走向顯式生命週期

徹底根治 GC Thrashing 的方法只有一個：**拔掉所有 `finalize()`，回歸顯式生命週期。**

1. **實作 `AutoCloseable`**：所有持有 Native 記憶體的類別，全部強迫實作 `close()`，配合 `try-with-resources` 或顯式回收。
2. **對標 `NativeAllocationRegistry`**：如果真的需要安全網，改用 Android 官方的 `NativeAllocationRegistry` 註冊 Cleaner，讓釋放動作走非同步輕量 Cleaner 機制，避開 `FinalizerDaemon` 的獨木橋。
3. **終極鐵律：物件池重用**：回到前面說的鐵碗架構，連 `new` 都不要 `new`，從源頭讓 `FinalizerReference` 的生產線徹底停工。

---

## 結語

**在每秒 30 幀的影像管線裡，把生命週期交給 GC finalize() 的那一刻，你的系統就已經寫好了死亡預告。**
