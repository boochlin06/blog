# Android 爛裝置想跑人臉辨識4 – 記憶體控場師LMKD

> 原文網址：https://boochlin.com/?p=431
> 發布日期：2025-05-19
> 文章編號：431

---

![](https://lh7-rt.googleusercontent.com/slidesz/AGV_vUdzDPdHhBDxfveZWnWNdlOd01rnkLSEOUMWG5FCqNdT0xGepYZQHGsSdij1EP-tTi-6zjlgN4EgxsAioP-j5SVhVvSUdKm8i2Q3Cbba-S8mSj1StVmP5vSNwXn2RD4GeudUztTmzTXH6rLyW_mtIbx6B7o3_PY=s2048?key=Te3C0G-QmQixACYoDXXiOg)

你以為把沒用的系統服務砍掉、把 ZRAM 設好，這台破板子就高枕無憂了？

天真。

在只有 2GB RAM 的極限戰場上，雙鏡頭 30fps 影像流源源不絕灌進來，系統記憶體水位永遠在警戒線上游移。真正掌控全系統所有進程生殺大權的活閻王，叫做 **LMKD（Low Memory Killer Daemon）**。

很多工程師做門禁機，最常遇到的靈異現象就是：人臉辨識跑得好好的，某天突然畫面黑屏，程式莫名其妙重啟。去翻 Logcat，連半行崩潰堆疊（Crash Stacktrace）都沒有。

不用懷疑，你的主程式就是被 LMKD 提著大刀在背景給直接斬首了。

手機的邏輯是「盡量不殺、留著快取讓切換 App 絲滑」；但我們做的是專用人臉辨識門禁機，整台設備只有一個核心使命：**人臉辨識 App 必須誓死活著，其他人只要擋路全部給我去死。**

這篇就來拆解我們如何徹底馴服 LMKD，讓它從隨機殺人的死神，變成專屬我們的護航保鏢。

---

## 演進的荒謬：從 OOM Killer 到 Userspace LMKD

搞懂 LMKD 之前，得先看清 Linux 與 Android 記憶體殺手的演進歷史：

### 1. 原生 Linux OOM Killer（遲鈍的莽漢）

Linux 內核自古就有 OOM Killer，但它在低階 Android 設備上根本是個災難。

它的觸發時機是「實體記憶體徹底歸零」。在只有 2GB 的設備上，等到記憶體見底才反應，系統早已陷入嚴重的記憶體顛簸（Thrashing），CPU 被換頁卡死，整台機器當場凍結。而且它的評分公式只看進程吃多少記憶體：

> oom_score = (RSS + Page tables + Swap) / Total RAM

誰吃得多就殺誰。在我們機器上，吃最多記憶體的偏偏就是相機管線與 AI 推理進程，OOM Killer 一出手，第一個槍斃的就是我們自己。

### 2. In-kernel LMK（僵硬的內核驅動）

Android 早期為了解決這個問題，在 Linux 內核寫了一個 `lowmemorykiller` 驅動，引入了 `oom_adj_score`（-1000 到 1000），並劃分出前景、可見、服務、快取等層級，搭配 `minfree` 閾值進行階梯式斬殺。

但 In-kernel LMK 有個致命硬傷：它綁定在內核的 Slab Shrinker 機制上，重度的進程掃描與殺戮邏輯卡死在 `vmscan` 流程中，導致內核效能劇烈抖動。

### 3. Userspace LMKD（現代體系：策略與機制分離）

從 Android 9 / 10 開始，Google 徹底將殺手移出內核，交給用戶空間的守護行程 **lmkd**。

Kernel 專心監控壓力並發送信號，Userspace 的 LMKD 負責讀取配置、挑選祭品並下達 `SIGKILL`。

---

## 壓力探測器的代差：Vmpressure vs PSI

要讓 Userspace LMKD 及時殺人，關鍵在於「它怎麼知道現在快爆了？」

在 Android 10 上，有兩種完全不同時代的感測機制：

### 1. 舊時代的 Vmpressure（事後諸葛）

Linux 3.14 引入的機制。它不是看 Free Memory，而是觀察內核在回收 Page 時發出的事件。

缺點非常明顯：反應極度遲鈍。當它發出「記憶體吃緊」的警報時，系統通常已經在缺頁異常的泥淖中掙扎了幾百毫秒，對 30fps 即時人臉辨識來說，掉幀早已發生。

### 2. 新世代的 PSI（Pressure Stall Information，精確脈搏）

Linux 4.20 引入的神級特性。PSI 透過在內核追蹤所有任務因等待記憶體而被阻塞的時間比例：

* **some**：至少有一個執行緒因為缺記憶體被卡住的時間佔比。
* **full**：所有非閒置執行緒全部被卡死、系統徹底失去響應的時間佔比。

PSI 就像直接在心臟裝上壓力感測器，不需要等記憶體崩潰，只要系統開始出現微小的排隊延遲，LMKD 就能在微秒級瞬間察覺！

---

## 實戰調教：打造專屬門禁機的冷血殺手

在我們的門禁終端上，所有系統配置必須貫徹一個最高原則：**保證人臉辨識進程永遠為 oom_adj = 0，並在記憶體稍有波動時，毫不留情處決所有無關的背景進程。**

在 `build.prop` 與系統屬性中下達以下配置：

```properties
# 啟用 Android 低記憶體精簡模式
ro.config.low_ram=true

# 強制切換為 PSI 壓力探測（拋棄遲鈍的 vmpressure）
ro.lmk.use_psi=true
ro.lmk.use_minfree_levels=false

# 調整 PSI 觸發靈敏度（單位：ms 停滯時間）
# 輕度停滯超過 200ms 即開始戒備，完全卡死超過 700ms 展開大屠殺
ro.lmk.psi_partial_stall_ms=200
ro.lmk.psi_complete_stall_ms=700

# 階梯式處決閾值（數值越低越兇狠）
# 低壓力時立刻清理快取進程（oom_adj >= 900）
ro.lmk.low=900
# 中壓力時清理無用背景服務（oom_adj >= 500）
ro.lmk.medium=500
# 極端壓力下容許處決的最低下限（絕對不能低於 0，0 是我們的前景主程式！）
ro.lmk.critical=0

# 一旦動刀，直接處決該層級吃最多 RAM 的大胖子，以最快速度騰出空間
ro.lmk.kill_heaviest_task=true
```

---

## Framework 深層動刀：修改 ProcessList.java

除了 LMKD 的屬性設定，AOSP 的 Framework 層還有個核心控場閥門：`ProcessList.java`。

原生 Android 為了讓使用者多工切換時不重新載入 App，預設保留了多達 32 個背景快取進程（`MAX_CACHED_PROCESSES = 32`）。這對門禁機來說簡直荒謬絕倫。

直接在 AOSP 源碼 `frameworks/base/services/core/java/com/android/server/am/ProcessList.java` 動刀：

```java
// 將背景快取進程上限從 32 個腰斬到 3 個！
// 門禁機不需要切換 App，多留一個快取都是在浪費寶貴的實體 RAM
public static final int MAX_CACHED_PROCESSES = 3;

// 將快取進程的記憶體保留閾值降為原本的 1/10
long getCachedRestoreThresholdKb() {
    return mCachedRestoreLevel / 10;
}
```

這兩行修改的效果極其立竿見影：任何在背景偷跑的殘餘進程，只要敢吃超過一點點記憶體，AMS 就會主動判定其為無效進程並迅速回收，根本輪不到 LMKD 出馬。

---

## 防坑血淚：小心系統中「兩個殺手」同時火拼

在我們調試 Android 10 的初期，曾遇到過一個匪夷所思的靈異 Bug：

明明在屬性裡設定了 `ro.lmk.use_psi=true`，但某些特定背景進程依然在不合理的時機被莫名斬殺。翻閱 Logcat，我們差點沒氣到吐血——**系統裡竟然同時跑著兩套記憶體殺手！**

BSP 廠商在移植內核時偷懶，既編譯了舊的 In-kernel LMK 驅動，又啟用了 Android 10 的 Userspace LMKD。兩邊判定規則衝突，在背景搶著殺人。

請務必敲下這幾條指令驗收你的系統：

```bash
# 檢查 lmkd 是否正常運行
adb shell ps -A | grep lmkd

# 確認 PSI 是否生效（必須回傳 true）
adb shell getprop ro.lmk.use_psi

# 檢查 Logcat 關鍵字
adb shell "logcat -d | grep -E 'Using psi monitors|lowmemorykiller'"
```

如果看到 `lowmemorykiller` 的痕跡，立刻回內核 `defconfig` 把它徹底閹割：

```ini
# 徹底禁用內核舊版殺手驅動
# CONFIG_ANDROID_LOW_MEMORY_KILLER is not set

# 確保啟用現代 Memory Cgroup 控制
CONFIG_MEMCG=y
CONFIG_MEMCG_SWAP=y
```

---

## 戰果驗收：小學算術對比

做完 LMKD + PSI + ProcessList 的三合一手術後，設備開機穩定運作 24 小時的 `dumpsys meminfo` 最終對比：

* **調教前**：
  * Total RAM: 1,909,104K
  * Free RAM: 456,339K（實際可用 free 只有可憐的 **64,232K**）
  * Used RAM: 1,503,496K（系統滿到喉嚨）
* **調教後**：
  * Total RAM: 1,909,104K
  * Free RAM: 1,135,724K（實際可用 free 暴增至 **469,728K**！）
  * Used RAM: 804,931K（整整奪回將近 700MB 的絕對安全區）

算式一目了然：

> 469MB (乾淨自由空間) - 300MB (人臉辨識引擎峰值) = +169MB 充裕安全餘量

從此以後，無論門口人潮多洶湧、連續刷臉多久，LMKD 就像忠誠的獵犬，只咬死外來的雜草，我們的主程式穩如泰山。

---

## 結語與防坑心法

1. **認清設備本質**：門禁機不是手機，不要為了虛榮的「多工流暢度」在背景保留任何快取進程。
2. **現代感測器才是王道**：果斷拋棄 vmpressure，全面擁抱 PSI，在系統產生停滯的最初幾毫秒就將危機撲滅。
3. **消除機制衝突**：確認關閉 In-kernel LMK，讓 Userspace LMKD 獨攬大權。

**在資源過剩的世界裡，大家講體面、講公平排程；但在 2GB 的破板子上，不心狠手辣處決別人，死的就是你自己的主程式。**

下一篇，我們聊聊那個在 LMKD 拔刀之前，唯一會試圖救你一命的難兄難弟：**ART GC**。

---

## 參考資料 (References)

1. **Android Low Memory Killer Daemon (LMKD) AOSP Documentation**.
https://android.googlesource.com/platform/system/memory/lmkd/

2. **Linux Kernel Pressure Stall Information (PSI)**.
https://www.kernel.org/doc/Documentation/accounting/psi.txt

3. **AOSP ProcessList and ActivityManager Architecture**.
https://source.android.com/docs/core/perf/low-ram
