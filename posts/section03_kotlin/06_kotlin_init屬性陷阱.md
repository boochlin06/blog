# kotlin init 前初始化屬性會發生啥事

> 原文網址：https://boochlin.com/?p=639
> 發布日期：2025-08-17
> 文章編號：639

---

**Kotlin 的 class 內部，在初始化階段，是無情地由上到下執行的**。

Kotlin

****

```
// 下面這樣寫會發生啥事，不會發生啥事，因為編譯不過，所以不能跑
class ItemEditViewModel(...) : ViewModel() {
    init {

            itemUiState = itemsRepository.getItemStream(itemId)
                .filterNotNull()
                .first()
                .toItemUiState(true)
        
        // ... use itemId ...
    }
// ❌ 錯誤示範. 屬性後初始化
    private val itemId: Int = checkNotNull(savedStateHandle[ItemEditDestination.itemIdArg])
}
```

---

### 那我加上一個 非同步處理，如下，這樣會發生啥事

```
init {
        viewModelScope.launch {
            itemUiState = itemsRepository.getItemStream(itemId)
                .filterNotNull()
                .first()
                .toItemUiState(true)
        }
    }
private val itemId: Int = checkNotNull(savedStateHandle[ItemEditDestination.itemIdArg])

可以編譯過，但是會有問題
由於 viewModelScope.launch 預設派發至 Dispatchers.Main，協程會被排入 Main Looper 的事件佇列。在大多數情況下，當協程真正被調度執行時，類別建構過程（包含後續的 itemId 初始化）已經完成，因此 itemId 通常能讀到正確值。然而，這種依賴執行時序的寫法本質上是脆弱的——若 Dispatcher 改為 Dispatchers.Unconfined 或在測試環境中使用 UnconfinedTestDispatcher，協程會立即內聯執行，此時 itemId 尚未初始化，確實會讀到預設值 0。正確做法是將 itemId 宣告在 init 區塊之前，或透過建構子參數直接傳入。
```
