# Kotlin 如何預防 Launch 連發，搜尋任務

> 原文網址：https://boochlin.com/?p=649
> 發布日期：2025-11-14
> 文章編號：649

---

不管處理何種問題，我們都必須考慮是出於何種場景，就像是

> 

使用者在 1 秒內點了 10 次。由於 launch 的天性是「**射後不理 (fire-and-forget)**」，你的 App 也會忠實地啟動 10 個獨立的 Coroutine，發起 10 次網路請求。

這樣看起來問題是 不想要同時跑十次任務，那為何不想要？如果不想要是為啥？是為了解決啥情境遇到的問題

---

要解決這個問題，我們有幾個層次的策略：

1. **UI 層** 在 launch 開始時立刻設定 button.isEnabled = false，並在 finally 區塊中將它設回 true。這是最直觀的防禦，直接阻止了使用者發起新任務的可能。

2. **Job 層** launch 會返回一個 Job 物件，你可以把它想像成這個任務的「遙控器」。遙控器可以控制任務。

3. Flow層 ，如果是壹連串改變的流程，那肯定有可以修改的部分

底下我們從場景來分析怎麼改？

---

## **「搜尋」功能 — 我只關心最新的結果**

在搜尋這種場景，使用者在短時間內快速輸入 “A”、”AP”、”APP”、”APPL”、”APPLE”，我們其實只關心最後 “APPLE” 的搜尋結果。前面幾次的請求，不僅浪費資源，如果舊的結果比新的結果晚回來，還會覆蓋掉正確的搜尋結果。

**如果舊任務還在跑，就取消它，立刻開始新任務。**

### **Kotlin Job**

```
// 在你的 ViewModel 或 Presenter 中
private var searchJob: Job? = null

fun onSearchQueryChanged(query: String) {
    // 1. 如果上一個搜尋任務 (searchJob) 還在執行中，就取消它！
    searchJob?.cancel()

    // 2. 啟動一個全新的 Coroutine 來處理這次的搜尋請求
    searchJob = lifecycleScope.launch {
        try {
            // 模擬網路延遲
            delay(1000)
            val result = apiService.search(query)
            _uiState.value = result // 更新 UI
        } catch (e: CancellationException) {
            // 當 Job 被 cancel 時會拋出這個異常，這是正常的流程，可以不用處理
            Log.i("SearchJob", "Job for query '$query' was cancelled.")
        } catch (e: Exception) {
            // 處理其他真實的錯誤
            _uiState.value = "Error: ${e.message}"
        }
    }
}
```

### **Kotlin **Flow

### 等他停止輸入一小段時間後。debounce

```
// 在你的 ViewModel 中
viewModelScope.launch {
    // 假設 clicksFlow 是從你的 UI 按鈕點擊事件轉換而來的 Flow
    clicksFlow
        .debounce(300L) // 只有當 300 毫秒內沒有新的點擊時，才讓最新的事件通過
        .onEach { query ->
            // 在這裡，query 就是通過了 debounce 考驗的最終搜尋關鍵字
            _uiState.value = "正在搜尋: $query"
            // 觸發網路請求
            val result = apiService.search(query)
            _uiState.value = result
        }
        .catch { throwable ->
            // 優雅地處理上游或 onEach 中發生的任何錯誤
            _uiState.value = "Error: ${throwable.message}"
        }
        .launchIn(viewModelScope) // 在 viewModelScope 中啟動這個 Flow 的收集
}
```

### 只關心最新輸入的結果 (transformLatest)

```
viewModelScope.launch {
    clicksFlow
        // mapLatest 是 transformLatest 的簡化版
        .mapLatest { query ->
            // 當新的 query 進來時，前一個尚未完成的 apiService.search 會被自動取消
            apiService.search(query)
        }
        .catch { ... }
        .collect { result ->
            // 這裡收到的永遠是最新 query 對應的結果
            _uiState.value = result
        }
}
```

## **「付款」按鈕 — 關鍵操作不容打斷**

對於像「確認付款」、「送出訂單」這種一次性的關鍵操作，我們絕不希望它被中途取消。而且，在第一次點擊的任務還沒完成前（例如，還在跟銀行伺服器溝通），我們應該忽略所有後續的點擊，避免重複下單。

**如果任務正在執行中，就忽略所有新的點擊請求。**

### **Kotlin Job**

```
// 在你的 ViewModel 或 Presenter 中
private var paymentJob: Job? = null

fun onPayButtonClicked() {
    // 1. 檢查 paymentJob 是否存在，並且是否還在活躍狀態 (isActive)
    if (paymentJob?.isActive == true) {
        // 2. 如果是，表示上次的付款流程還沒跑完，直接 return，不做任何事。
        Log.d("PaymentJob", "Payment is already in progress. Ignoring new click.")
        return
    }

    // 3. 如果沒有正在進行的任務，就啟動一個新的 Coroutine
    paymentJob = lifecycleScope.launch {
        try {
            _uiState.value = "處理中..." // 顯示處理中狀態
            button.isEnabled = false // 同時搭配 UI 防守

            // 模擬一個無法被取消的關鍵網路請求
            val result = withContext(NonCancellable) {
                apiService.processPayment()
            }

            _uiState.value = "付款成功！" // 更新 UI
        } catch (e: Exception) {
            _uiState.value = "付款失敗: ${e.message}"
        } finally {
            button.isEnabled = true // 無論如何，最後都要恢復按鈕狀態
        }
    }
}
```

### **Kotlin **Flow

```
// 在 ViewModel 中

// 1. 一個「忙碌信號燈」，用來標記是否正在處理付款
private val isProcessing = MutableStateFlow(false)

// 2. 一個「點擊觸發器」，UI 層只需要呼叫它來發送點擊事件
//     使用 SharedFlow，因為我們可能有多個地方需要觸發或監聽
private val paymentTrigger = MutableSharedFlow<Unit>()

// 3. 在 ViewModel 初始化時，設定好監聽和處理邏輯
init {
    viewModelScope.launch {
        paymentTrigger.collect {
            // 每次點擊事件觸發時，先檢查「信號燈」
            if (isProcessing.value) {
                // 如果是紅燈（正忙），直接 return，忽略這次觸發
                return@collect 
            }

            // 如果是綠燈（空閒），立刻切換為紅燈，並開始工作
            isProcessing.value = true
            try {
                _uiState.value = "處理中..."
                val result = apiService.processPayment()
                _uiState.value = "付款成功！"
            } catch (e: Exception) {
                _uiState.value = "付款失敗: ${e.message}"
            } finally {
                // 無論成功或失敗，工作結束後，都要把信號燈切回綠燈
                isProcessing.value = false 
            }
        }
    }
}

// 4. UI 層只需要呼叫這個簡單的函式
fun onPayButtonClicked() {
    viewModelScope.launch {
        paymentTrigger.emit(Unit)
    }
}
```

—
