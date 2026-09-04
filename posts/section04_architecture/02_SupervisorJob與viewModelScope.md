# SupervisorJob 不就是 viewModelScope嗎？

> 原文網址：https://boochlin.com/?p=672
> 發布日期：2025-07-14
> 文章編號：672

---

**Job 不僅僅是 launch 的回傳值，它是我們管理 Coroutine 的「遙控器」。**

---

### Job ：

Job 是最基本的協定，它遵循嚴格的「結構化併發」原則。

* **特性：「一損俱損」。** 在一個由標準 Job 管理的範疇內，任何一個子 Coroutine 的失敗（拋出未處理的例外），都會像推倒第一張骨牌一樣，立刻導致整個父 Job 和其下所有其他的子 Coroutine 被連鎖取消。

#### 聊天 APP 場景：使用者登入初始化

一個使用者登入流程，可能包含「驗證帳密」、「讀取個人資料」、「獲取好友列表」等多個非同步步驟。這些步驟是一個不可分割的整體，任何一步失敗，整個登入流程就該立即宣告失敗，

---

### SupervisorJob

SupervisorJob 是 Job 的一個特殊變體，它打破了「一損俱損」的規則，提供了更強的韌性。你可以把它想像成一位經驗豐富、處變不驚的工廠監工。

* **特性：「失敗隔離」。** 在 SupervisorJob 的監督下，某個子 Coroutine 的失敗不會向上傳遞，因此不會影響到父 Job 或其他的兄弟 Coroutine。

* **適用場景：長期共存、需要獨立運作的元件集合。**

#### 聊天 APP 場景：客戶端核心服務

一個客戶端的連線 Session，需要同時處理「接收訊息」、「發送訊息」、「心跳維持」這三個常駐任務。我們絕不希望因為「發送」時的網路抖動失敗，就導致更重要的「接收」和「心跳」也跟著被取消。SupervisorJob 就像那位監工，他確保一條生產線的短暫故障不會導致整個工廠停擺。

#### viewModelScope 的秘密

「這個模式聽起來很棒，但這是我需要手動處理的特殊情況嗎？」不用，已經內建了

你可能會思考：「如果我要隔離，在 Scope 中分別啟動三個獨立 Job 不就好了嗎？」但接著你會發現，這樣做會導致**災難**——當 Session 結束時，你必須手動取消每一個 Job，非常容易出錯並造成資源洩漏。

迎來頓悟：**viewModelScope 就是這樣設計的！**

Kotlin

```
// ViewModel.kt in androidx.lifecycle
val ViewModel.viewModelScope: CoroutineScope
    get() {
        // ...
        val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
        // ...
        return scope
    }
```

沒錯，viewModelScope 天生就是一個 SupervisorJob。Google 的工程師早就料到，在一個 ViewModel 中，你會啟動多個互相獨立的 UI 相關任務，而一個任務的失敗不應該讓整個畫面邏輯崩潰。**所以我們該關注的，正是如何利用 SupervisorJob 的特性去區隔場景，**

---

### Deferred — 附帶取餐功能的遙控器

Deferred 是一種更特殊的 Job，它承諾會在未來回傳一個結果。正如其源碼所定義，Deferred **繼承自 Job**。

```
// public interface Deferred<out T> : Job { ... }
```

這意味著 Deferred 本身就是一個 Job，但它額外提供了一個 await() 方法來獲取結果。

* **何時使用？** 當你的 Coroutine 不只是「做一件事」，而是要「**算一個結果**」時，就該用 async 啟動器，它會回傳一個 Deferred 物件。

*  Deferred 就像你去餐廳點餐拿到的**號碼牌**。它本身是「店家正在做菜」的憑證（一個 Job），但更重要的是，你可以稍後憑著它，用 .await() 去取回你的餐點（結果）。

—
