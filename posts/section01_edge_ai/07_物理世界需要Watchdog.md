# Android 爛裝置想跑人臉辨識7 – 物理世界需要 watchdog

> 原文網址：https://boochlin.com/?p=895
> 發布日期：2026-07-01
> 文章編號：895

---

我們做的不是給手機用戶躺在沙發上滑的 App。

手機 App 閃退了，用戶手指點一下重新打開。我們做的是一台掛在牆上、7x24 小時風吹日曬的人臉辨識門禁機。當它在半夜三點因為相機斷流、網路假死或 OOM 崩潰時，難道你要半夜騎車去現場拔電源重插？

我們需要一隻真正具有「不死之身」的看門狗（Watchdog）。

---

## 工業級主流標準 vs 我們的 BOM 表業障

在車載與工業級主流架構中，系統穩定性由三層嚴密的防禦捍衛：

* **頂層（AOSP）**：`Watchdog.java` 監控 SystemServer 核心鎖。
* **中層（Linux）**：啟用 `/dev/watchdog`（SoC PMIC 晶片級看門狗），Kernel Panic 自動 Reset。
* **底層（外置 MCU）**：主板標配獨立 STM32 晶片。Android 透過 GPIO 定時送方波脈衝（硬體餵狗）；一旦連續 60 秒沒收到脈衝，MCU 透過 MOS 管直接切斷整機 12V 供電強制冷重啟（Cold Power Cycle）。

這是教科書裡最優雅的防禦。

但我們面對的是高通 QM215 配 2GB RAM 的廉價板子。老闆為了省下幾毛美金的成本，硬是把獨立的硬體 MCU 給拔了！

身為軟體工程師，沒有硬體神仙保護，我們只能用 Android 與底層 GPIO 把軟體看門狗逼到極限。

---

## 顯式靜態廣播的「起死回生術」

很多人做看門狗，直覺就是用背景 `RestartService` 跑無窮迴圈。

但在 2GB 爛板子上，當 AI 模型吃滿記憶體時，LMKD 一旦發起 OOM 秒殺，依附在進程裡的 `Service` 會當場跟著陪葬，連個遺言都留不下來。

我們的解法是：**`AlarmManager`（`setExactAndAllowWhileIdle` 鏈式調度）+ 顯式靜態註冊 `SystemEventReceiver`**。

```java
// SystemWatchdog.java
public static void scheduleNextHeartbeat(Context context) {
    AlarmManager am = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
    Intent intent = new Intent(context, SystemEventReceiver.class);
    PendingIntent pi = PendingIntent.getBroadcast(
        context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE
    );

    long triggerAtMillis = SystemClock.elapsedRealtime() + 15 * 1000;

    // 穿透 Android 9/10 Doze Mode 的核心防禦：
    // 普通 setInexactRepeating 會在夜間待機時被系統對齊延遲數十分鐘！
    // 必須使用 setExactAndAllowWhileIdle 確保低功耗期準時觸發：
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        am.setExactAndAllowWhileIdle(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtMillis, pi);
    } else {
        am.setExact(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtMillis, pi);
    }
}
```

配套工控系統配置：
1. **遞迴鏈式預約**：在 `SystemEventReceiver.onReceive()` 處理完餵狗邏輯後，末尾立刻再次調用 `scheduleNextHeartbeat(context)`，形成永不中斷的 15 秒精確脈衝。
2. **ROM 級電池白名單**：工控機韌體在出廠時，於 `/etc/sysconfig/whitelist.xml` 加入 `<allow-in-power-save package="com.edge.ai.gate" />`，徹底豁免 Doze Mode 休眠限制。

### 為什麼進程死光了，系統真的能把它拉活？

1. **定時器活在系統層**：`PendingIntent` 是註冊並保存在 `system_server` 的 `AlarmManagerService` 中。App 進程被 LMKD 砍掉時，系統定時器依然完好無損。
2. **被殺 ≠ 強制停止**：門禁機沒人會去點系統設定裡的「強制停止」（不會被打上 `FLAG_STOPPED`）。被 OOM 殺死的進程，在 Android 眼中依然是活躍應用。
3. **顯式指定 Class 穿透限制**：Android 8.0+ 封殺了隱式廣播，但我們明確指名 `SystemEventReceiver.class`。時間一到，AMS 發現進程已死，**會立刻命令 `Zygote` 當場 fork 一個全新 Linux 進程冷啟動**！

---

## 網路死鎖偵測：為什麼 NUD 是天然的底層看門狗？

傳統的網路檢查有兩大致命陷阱：

1. **`ConnectivityManager.isConnected()` 的假連線陷阱**：它只看網線有沒有插上。網卡晶片底層死鎖時依然回報 `true`，門禁機直接變磚。
2. **Ping / HTTP 輪詢**：封閉局域網（無外網）直接失效，而且雲端伺服器稍微抖動極易引發誤殺重啟。

### 網卡硬體假死的 4 大物理成因

* **繼電器電磁鎖反向電動勢（Back-EMF）**：開門瞬間上百伏特反衝電壓透過地線雜訊打進晶片，觸發 CMOS 晶閘管鎖死（Latch-up）。
* **幾十米長網線靜電（ESD）**：破壞網卡時脈恢復電路（PLL 失鎖），數位解調器癱瘓但類比前端依然回報 Link Up。
* **USB 轉網卡端點掛死（USB Endpoint Halt）**。
* **廣播風暴與 FIFO 溢位**：CPU 忙於算臉時，內網廣播灌爆網卡微小的 16KB SRAM 緩衝區。

### NUD 是如何完美扮演看門狗的？

Linux 核心的 **NUD（Neighbor Unreachability Detection）** 在 Layer 2 內建了一套完整的看門狗狀態機：

```java
// SystemEventReceiver.java
if (nudState == SystemEventReceiver.NUD_FAILED) {
    if (curTime - prevArpFailTimeMS > ARP_RESET_USB_TIME_INTERVAL /* 60秒 */) {
        gpIO.hwRest(); // 判定網卡硬體死鎖，觸發硬體復位
    }
} else if (nudState == SystemEventReceiver.NUD_REACHABLE) {
    prevArpFailTimeMS = 0;
    arpResetCount = 0; // 成功收到網關 ARP，看門狗計數器歸零（餵狗！）
}
```

* **心跳探針（0 CPU 負擔）**：Linux 核心向外發包時自動發起 ARP 探測網關，不需要 Java 開任何 Ping 執行緒。
* **定時器復位（餵狗）**：網關正常回覆 ARP，狀態為 `NUD_REACHABLE`，看門狗計數器歸零。
* **超時咬死（復位）**：網卡晶片死鎖導致 ARP 全滅，核心廣播 `NUD_FAILED`。累積 60 秒沒被餵狗，寫入 `/mnt/vendor/persist/vendor/nudReport.txt` 留證，並觸發 GPIO 斷電重啟。

外網斷了但網關還在（`NUD_REACHABLE`），門禁機絕不重開，優雅降級為離線刷臉；只有網卡硬體徹底死鎖時才觸發重啟。

---

## 相機斷流與兩段式硬體電擊

USB 相機控制器死鎖時，API 回報正常但畫面凍結在最後一幀。看門狗每 15 秒檢查一次：**超過 30 秒沒收到新畫面（`curTime - rgbAliveTimestampMS > 30s`）**，判定相機死亡。

我們設計了**「兩段式降維打擊」**：

```java
// RestartService.java & DeviceGpio.java
if (curTime - rgbAliveTimestampMS > LIMIT /* 30秒 */) {
    // 第一階段：局部硬體電擊（500ms GPIO 切斷相機供電）
    gpIO.usbSwitchOn();
    gpIO.usbRgbHwReset();
    Thread.sleep(500); // 斷電 500ms
    gpIO.usbSwitchOff();
    rgbResetCount++;

    // 第二階段：連續 3 次急救無效，全機重開
    if (rgbResetCount > RGB_RESET_LIMIT) {
        gpIO.hwRest();           // 拉高主板 HW_RESET GPIO 引腳
        RuntimeUtils.reboot();   // 執行 reboot 指令
    }
}
```

* **第一階段（局部電擊）**：GPIO 切斷 USB 相機供電 500ms，迫使 USB PHY 重新枚舉，90% 的相機死鎖在 1 秒內滿血復活，無需重開機。
* **第二階段（全機死刑）**：連續電擊 3 次無效，判定 USB 總線死鎖，拉高 `HW_RESET` GPIO 全機冷重啟。

---

## 結語：軟體看門狗的物理天花板

在被閹割獨立 MCU 的硬體上，我們靠「顯式廣播冷啟動 + NUD 天然看門狗 + 500ms GPIO 電擊」搭起極具韌性的自救體系。

但架構師必須清醒：**純軟體看門狗救不了 Linux Kernel Panic。** 在那種極限下，底層唯一的保險只剩 Linux 核心的 `/dev/watchdog`。

在有限預算下把爛牌打到極致，用軟硬協同彌補硬體缺陷，這才是邊緣端工程的精髓。

---

## 參考資料 (References)

1. **Android Broadcasts Overview: Implicit vs Explicit Broadcast Restrictions**.
https://developer.android.com/guide/components/broadcasts

2. **Linux Neighbor Unreachability Detection (NUD) RFC 4861**.
https://datatracker.ietf.org/doc/html/rfc4861

3. **AOSP IpReachabilityMonitor in Android NetworkStack**.
https://android.googlesource.com/platform/frameworks/libs/net/+/master/common/framework/com/android/net/module/util/IpReachabilityMonitor.java
