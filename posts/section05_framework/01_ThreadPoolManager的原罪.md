# 說到底沒有吐槽以前自己寫的 code 代表你沒進步-ThreadPoolManager 的原罪

> 原文網址：https://boochlin.com/?p=1022
> 發布日期：2026-08-29
> 文章編號：1022

---

每次接手舊專案，十個有九個會看到一個全家桶 `ThreadPoolManager`。因為這是我的起手式，本質上是為了避免系統資源被抽乾的「節流閥」。

## 每 new 一條 Thread，你都在燒 RAM

在 Linux 底層，每 `new` 一條 Java 執行緒，作業系統就得在 Kernel 態配置資料結構，並切出約 1MB 的記憶體當作 Thread Stack。

十幾條沒事幹的執行緒掛在那邊發呆，光站著就白白噴掉幾十 MB。而當十幾條執行緒在背景爭搶 CPU 時，真正致命的是頻繁的 **Context Switch**：CPU 必須不斷保存與恢復暫存器，頻繁觸發 TLB Miss 並把 L1/L2 快取強制洗掉重載。導致 CPU 算力全耗在「排程切換」的無效 overhead 上，UI 執行緒直接卡死掉幀。

## Google 當年的 AsyncTask 到底有多搞笑

* **全 App 排同一個隊列**：Android 3.0 後預設共用全域 `SERIAL_EXECUTOR`。模組 A 卡住 30 秒，模組 B 在另一個畫面只想讀 2KB 本地快取，也得在後面乖乖排隊罰站 30 秒；

* **旋轉螢幕與生命週期脫節**：寫成內部類別會強引用 Activity 導致 Memory Leak。更慘的是 Fragment 已經被 Detach，背景任務跑完在主線程呼叫 `requireActivity()`，直接噴出 `IllegalStateException: Fragment not attached` 當場閃退；

* **跨版本精神分裂與 Reject 崩潰**：Android 1.5 是單執行緒；1.6~2.3 預設池配置為 `Core=5, Queue=10, Max=128`，只要瞬間發出超過 138 個任務，第 139 個直接噴出 `RejectedExecutionException` 閃退；到了 3.0 滿地哀鴻遍野，Google 才認慫縮回單執行緒串列；

* **`cancel(true)` 只是個心理安慰劑**：底層 Thread 根本不會停，耗電死循環繼續在背景跑到天荒地老，只是跑完不回呼而已。

## 真地雷

後端面試愛考的死鎖，在 Android 上很少見，因為 5 秒卡死系統就直接噴 ANR 閃退了。一般 App 真正天天在 Crashlytics 上榜的，全是下面這三種通用地雷：

### 1. 搜尋與連點時的「時序錯亂舊蓋新」

使用者在搜尋框輸入「A」發出請求（耗時 2 秒），緊接著輸入「AB」發出請求（耗時 0.2 秒）。結果「AB」的結果先跑出來，隨後「A」的舊結果才回來，**直接把正確的新畫面無情覆蓋掉**。連按兩次按鈕、切換 Tab 也同理，加鎖根本沒用，鎖只管併發不管誰先誰後。

### 2. 離開畫面後回呼 View 的

背景執行緒做完工作，興高采烈地用 `mainHandler.post()` 回主線程更新畫面。結果使用者早就按返回退出了，Fragment 已經被 Detach 或 View 已經銷毀，直接噴出 `IllegalStateException` 閃退。

### 

## 現代 Android 怎麼寫

現在早就不准你自己寫 ThreadPool 了，全面轉向 **Kotlin Coroutines**，精準破解上述三大地雷：

* **算力與 I/O 嚴格分流**：高密度運算交給 `Dispatchers.Default`（精準綁定 CPU 實體核心數，保底 2 條），網路與資料庫等阻塞型任務切給 `Dispatchers.IO`；

* **生命週期自動止損**：丟給 `viewModelScope`，使用者退出頁面時非同步任務全體自動 Cancel，滅絕 Memory Leak 與 Detach 閃退；

* **時序精準覆蓋**：用 `Flow.debounce(300)` 擋住高頻連點，再搭配 `collectLatest()`——只要新請求一進來，底層自動 Cancel 掉前一次還在跑的舊請求，連 `isCancelled` 旗標都不用寫；

* **非阻塞掛起滅絕 OOM**：Coroutines 的 `suspend` 特性讓網路請求「掛起而不阻塞執行緒」，一條 Thread 就能抗上萬個併發，徹底告別 Thread 暴增引發的 `pthread_create failed`。
