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
getItemStream 會永遠是只讀取到０．主要是因為 Int 的預設值就是0
```
