# Android 爛裝置想跑人臉辨識5 – GC 是唯一會關心你 OOM 的好人

> 原文網址：https://boochlin.com/?p=453
> 發布日期：2025-06-10
> 文章編號：453

---

![](../../assets/images/Android-記憶體調教.png)

在 Android 上搞高負載應用的工程師，每個人都跟 OOM（Out Of Memory）打過交道。不如說自從我出社會的第一年開始，這玩意基本上就是我的三餐便當。

尤其當你手上的板子是弱雞到不能再弱雞的高通 QM215，配上區區 2GB RAM。

上一篇我們把 LMKD 調成了冷血殺手，只要有人搶記憶體就一刀剁掉。但在 LMKD 提刀破門而入之前，整個 Android 系統裡唯一還抱著菩薩心腸、在最後一刻試圖拉你一把的，就只有 **ART 的垃圾回收器（Garbage Collector, GC）**。

很多人把 GC 當成系統卡頓的原罪，恨不得把 GC 徹底關掉。

聽我一句勸：在 2GB 的爛板子上，GC 不是你的敵人，它是唯一願意替你的爛代碼擦屁股的好人。問題從來不在 GC 本身，而在於你無節制地在每秒 30 幀的相機循環裡瘋狂拉屎。

這篇我們就來把 ART GC 的底牌掀開，教你如何把「致命急診」調教成「無感健檢」。

---

## 診斷現場：看懂這兩行天書的生死密碼

設備接上 USB，打開 Logcat，在我們這台跑著雙鏡頭的人臉辨識機上，最常看到的日誌長這樣：

```text
com.edge.ai I/com.edge.ai: NativeAlloc concurrent copying GC freed 111541(2850KB) AllocSpace objects, 15(19MB) LOS objects, 10% free, 67MB/75MB, paused 337us total 396.170ms
com.edge.ai I/com.edge.ai: Background concurrent copying GC freed 138625(3293KB) AllocSpace objects, 3(14MB) LOS objects, 10% free, 67MB/75MB, paused 1.042ms total 410.415ms
```

這串 Log 不是亂碼，請用解剖刀仔細拆解它的格式：

> `<GC_Reason> <GC_Name> <Freed_Objects>(<Freed_Size>) AllocSpace, <LOS_Freed>(<LOS_Size>) LOS, <Heap_Stats>, paused <Pause_Time> total <Total_Time>`

翻開 AOSP 源碼，決定這場 GC 是福是禍的靈魂，在於最開頭的 **GC_Reason**：

```cpp
// 摘自 AOSP art/runtime/gc/collector/gc_type.h
enum GcCause {
    kGcCauseForAlloc,       // 急診室等級：記憶體不夠分配了，當場阻塞當前執行緒！
    kGcCauseBackground,     // 健檢等級：系統在背景預防性清理，非同步不卡頓
    kGcCauseForNativeAlloc, // 暗黑大魔王：C++ Native 層哭著要記憶體觸發的 GC
    kGcCauseExplicit,       // 菜鳥手賤呼叫了 System.gc()
};
```

---

## 兩大 GC 形態：急診 vs 健檢

### 1. kGcCauseForAlloc（急診室等級）

* **觸發時機**：你的 Java 代碼執行了 `new byte[]` 或 `Bitmap.createBitmap()`，Java Heap 空間不足以容納，當場觸發緊急搶救。
* **致命代價**：**阻塞（Blocking）！** 負責分配記憶體的執行緒會被原地凍結，必須乖乖等待 GC 回收完畢才能繼續。如果發生在 UI 執行緒或相機回調執行緒，畫面瞬間定格卡死。
* **體感診斷**：肉眼可見的掉幀與嚴重頓挫，這代表你的記憶體已經躺在加護病房插管急救了。

### 2. kGcCauseBackground（健檢等級）

* **觸發時機**：ART 發現 Heap 水位達到預先設定的閾值，很貼心地在背景排程非同步清理。
* **運行代價**：非同步執行，`paused` 暫停時間極短（通常在 1ms 上下），主執行緒幾乎無感。
* **戰略價值**：健康的系統應該充滿這種預防性 GC，把垃圾提前收走，換取永遠不觸發阻塞式的 `for alloc`。

我們的核心戰術非常純粹：**徹底消滅 `kGcCauseForAlloc`，並大幅降低 `kGcCauseBackground` 的觸發頻率。**

---

## 戰術突破：第一性原理——不要在 Java Heap 裡拉屎

在 30fps 的影像串流下，相機每秒吐出 30 幀畫面。如果每一幀都在 Java 層 `new` 物件，小學算術會立刻賞你一記耳光：

> 640x480x1.5 (YUV 幀) = 460.8 KB
> 460.8 KB x 30 fps = 13.8 MB/s
> 如果加上轉碼中間容器與暫存物件 = 每秒製造超過 40 MB 的 Java Heap 垃圾！

在 `dalvik.vm.heapgrowthlimit=160m` 的枷鎖下，Java Heap 只要 3 秒就會被撐爆，系統不得不每 3 秒發動一次阻塞式 GC 來清垃圾。

怎麼破局？回歸第一性原理：**既然 Java Heap 吃緊，那就根本不要在 Java 裡面分配記憶體！**

把所有高負載的大型記憶體操作，全部下沉到 **C++ Native 層**：

* 相機 YUV 幀數據傳遞：Direct ByteBuffer（0 拷貝指針傳遞）。
* 影像前處理（旋轉、縮放、格式轉換）：C++ OpenCV / QFastCV。
* AI 模型推理（人臉偵測、特徵萃取）：TFLite / NCNN C++ 核心。
* 人臉特徵庫比對：C++ 矩陣向量加速。

整個系統架構變成一條純粹的 Native 管線：

> Camera HAL -> DirectBuffer (Native 指針) -> JNI -> C++ 運算 -> JNI 返回結果坐標 -> OpenGL 渲染

Java 層只充當遙控木偶的細線，底層幾十 MB 的記憶體巨獸在 C++ 裡面自生自滅。Java Heap 全天候維持在可憐的 15MB 左右，GC 根本連出動的理由都沒有。

---

## Framework 參數調教：壓制對手，放寬呼吸

除了 App 代碼下沉 Native，我們還在系統層級對 ART 虛擬機進行了精準調教（寫入 `build.prop`）：

```properties
# 壓制其他 App 的 Java Heap 上限（防止背景服務偷偷膨脹）
dalvik.vm.heapgrowthlimit=160m
dalvik.vm.heapsize=256m

# 關鍵調優：適度放寬空閒緩衝空間
dalvik.vm.heapstartsize=8m
dalvik.vm.heapminfree=8m
dalvik.vm.heapmaxfree=16m

# 堆積目標利用率（設為 0.7，平衡記憶體佔用與 GC 頻率）
dalvik.vm.heaptargetutilization=0.7
```

* **為什麼要調低 `heapgrowthlimit`？**
  我們的主程式跑在 Native，不需要肥大的 Java Heap。把上限鎖在 160MB，可以強力壓制系統其他啟用 `largeHeap=true` 的應用，逼迫它們提早克制，把實體 RAM 讓出來。
* **為什麼要拉大 `heapminfree` 與 `heapmaxfree`？**
  如果緩衝區設得太小，Heap 一稍微波動就會頻繁觸發鋸齒狀的背景 GC。適度給予 8MB ~ 16MB 的呼吸餘裕，可以大幅減少 GC 啟動次數，避免 4 顆孱弱的 A53 核心頻繁被 GC 執行緒搶佔算力。

---

## 隱形刺客：kGcCauseForNativeAlloc

當你以為把代碼搬到 C++ 就萬事大吉時，Logcat 突然無情地噴出這行：

```text
com.edge.ai I/com.edge.ai: NativeAlloc concurrent copying GC freed ...
```

你心中一定暗罵：「C++ 明明沒有 GC，為什麼 Java GC 偏偏要多管閒事？」

### 為什麼 Java GC 要管 Native 的事？

因為在 Android 體系中，很多 Native 記憶體是透過 **`NativeAllocationRegistry`** 跟 Java 物件強行綁定的（例如 `Bitmap`、`CameraMetadataNative`）。

當你在 C++ 用 `malloc` 分配了 50MB 記憶體，ART 虛擬機透過註冊表感知到了這筆龐大的 Native 開銷。當系統記憶體吃緊時，ART 會自作聰明地想：

> 「這個 Native malloc 快失敗了，是不是因為 Java Heap 裡還活著一些沒被回收的 Java Wrapper？讓我跑一次 GC，看能不能順便把對應的 Native 記憶體釋放掉！」

這就是 `kGcCauseForNativeAlloc` 的真相。

### 拆彈守則：

1. **嚴禁在影像回調裡頻繁 `Bitmap.createBitmap()`**：每造一張 Bitmap，就在 Native 埋下一顆定時炸彈。
2. **記憶體池（Buffer Pool）重用**：在 C++ 與 Java 之間只傳遞固定預先分配好的 Buffer 鐵碗。
3. **主動呼叫 `recycle()` 與 `close()`**：不要依賴 GC 的被動回收，用完當場顯式釋放，斬斷 `NativeAllocationRegistry` 的牽掛。

---

## 結語與防坑心法

1. **GC 是鏡子，映出的是代碼的平庸**：不要怪 GC 卡頓，先看自己有沒有在每秒 30 幀的相機管線裡瘋狂 new 物件。
2. **Native 化是唯一救贖**：在 2GB 設備上跑 CV 與 AI，把圖像數據全部留在 C++ Native 層，讓 Java 只負責 UI 事件調度。
3. **理智平衡 VM 參數**：用適度的 Heap 緩衝空間換取平穩的 CPU 曲線，避免鋸齒狀的頻繁 GC 顛簸。

**把 Java Heap 壓制到極限，把舞台讓給 Native，GC 才能從卡頓殺手變成守護系統的好人。**

下一篇，我們把鏡頭對準最恐怖的記憶體土石流：**雙鏡頭每秒近 70MB 的高頻數據**，看看如何用一套「鐵碗架構」，做到真正的 0-new 與 30fps 極速運轉！

---

## 參考資料 (References)

1. **ART GC Architecture and Log Analysis in AOSP**.
https://source.android.com/docs/core/runtime/gc-debug

2. **Android NativeAllocationRegistry Deep Dive**.
https://android.googlesource.com/platform/libcore/+/master/luni/src/main/java/libcore/util/NativeAllocationRegistry.java

3. **Managing Bitmap Memory and In-place Reuse**.
https://developer.android.com/topic/performance/graphics/manage-memory
