# viewModelScope vs rememberCoroutineScope 其實沒啥好 versus ，就是 SOC 職責分離

> 原文網址：https://boochlin.com/?p=634
> 發布日期：2025-07-15
> 文章編號：634

---

* viewModelScope：與 **ViewModel** 的生命週期綁定，只能用在**資料/邏輯層**。

* rememberCoroutineScope：與 **Composable 畫面元件**的生命週期綁定，只能用在 **UI 層**。

永遠不要在 rememberCoroutineScope 中處理業務邏輯，也永遠不要在 viewModelScope 中直接控制需要 Composable 上下文的 UI 元件

---

### 何時該用哪個？

你可以問自己一個問題：

「當手機螢幕旋轉時，這個非同步任務應該要繼續執行，還是應該要取消重來？」

* **如果答案是「應該繼續執行」**: 那就用 viewModelScope。

  * **情境**: 你正在從網路下載一個使用者頭像。使用者只是旋轉一下手機，你不希望下載中斷，而是希望它繼續在背景完成，然後顯示在新方向的畫面上。這是資料層的任務，與 ViewModel 的生命週期一致。

```
// 在 ViewModel 中
class ProfileViewModel : ViewModel() {
    fun fetchAvatar() {
        viewModelScope.launch {
            // 即使螢幕旋轉，這個下載任務也不會中斷
            val avatar = repository.downloadAvatar()
            _uiState.update { it.copy(avatar = avatar) }
        }
    }
}
```

* **如果答案是「應該取消重來」或「應該停止」**: 那就用 rememberCoroutineScope。

  * **情境**: 當某個狀態變為 true 時，你需要顯示一個 Snackbar 提示「訊息已傳送」。如果使用者在這時旋轉了螢幕，舊的 Snackbar 就沒有意義了，你可能會想在新畫面上根據新的狀態決定是否重新顯示。這個任務完全依附於當前的 UI。

```
// 在 Composable 中
@Composable
fun MessageSender(snackbarHostState: SnackbarHostState) {
    // 取得一個與 Composable 生命週期綁定的 scope
    val scope = rememberCoroutineScope()

    Button(
        onClick = {
            // 在點擊事件這種 callback 中，手動啟動協程
            scope.launch {
                // 這個任務與 UI 相關，如果畫面消失，任務也該取消
                snackbarHostState.showSnackbar("訊息已傳送！")
            }
        }
    ) {
        Text("傳送訊息")
    }
}
```

* viewModelScope 關心的是**資料的存活**，它比 UI 更持久，能跨越短暫的 UI 重建（如螢幕旋轉）。

* rememberCoroutineScope 關心的是**畫面的呈現**，它的生命非常短暫，與你在螢幕上看到的元件同生共死。

---

類似 viewModelScope 的有

### LifecycleScope

這是 viewModelScope 最直接的兄弟。如果說 viewModelScope 是為 ViewModel 量身打造的，那 LifecycleScope 就是為 **Activity 和 Fragment** 這類具有明確生命週期的 UI 元件所設計的。

```
class MyFragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // 使用 lifecycleScope 啟動協程
        viewLifecycleOwner.lifecycleScope.launch {
            // 在這裡執行長時間的背景任務，例如網路請求或資料庫讀取
            val data = fetchData()
            updateUi(data)
        }
    }
}
```

### GlobalScope (全域作用域)

GlobalScope 是一個**重量級**的工具，需要謹慎使用。它正如其名，是一個**全域**的作用域

```
// 警告：除非你非常確定，否則不要這樣做！
// 這個任務會一直執行，直到 App 被殺掉。
GlobalScope.launch {
    while (true) {
        delay(30_000) // 每 30 秒執行一次
        Log.d("GlobalScope", "Doing some background work...")
    }
}
```

### 自訂 CoroutineScope. 其實就是SupervisorJob

這在處理非標準生命週期的物件時特別有用。

```
class BluetoothConnectionManager {

    // 1. 建立一個包含 Job 和 Dispatcher 的自訂 Scope
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.IO)

    fun startListening() {
        scope.launch {
            // 開始監聽藍牙訊號...
        }
    }

    // 2. 必須手動在適當的時機取消！
    fun disconnect() {
        // 這會取消 scope 內所有由它啟動的協程
        scope.cancel()
    }
}
```

---

—
