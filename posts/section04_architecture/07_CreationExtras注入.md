# CreationExtras.inventoryApplication() 這有點跳

> 原文網址：https://boochlin.com/?p=641
> 發布日期：2025-09-14
> 文章編號：641

---

## CreationExtras 到底是什麼？

CreationExtras 是一個在 **ViewModel 創建時自動附帶的「額外資訊包裹」**。

以前，想給 ViewModel 傳參數（如 SavedStateHandle），你得為每個 ViewModel 寫一個專屬的 Factory，這既麻煩又冗餘。

CreationExtras 的出現，就是為了解決這個問題，它提供了一個**標準化的統一管道**來傳遞數據。

---

### CreationExtras 裡面有什麼？

它本質上是一個 **Map (鍵值對)**。裡面會包含：

* **系統提供的標準資訊：**

  * **Application **

  * **SavedStateRegistryOwner 相關資訊：** 讓 ViewModel 可以**自動創建 SavedStateHandle**，保存和恢復 UI 狀態。

* **你自定義的額外資訊：** 你也可以往裡面放任何你需要的數據。

---

### CreationExtras 如何運作？

你通常不用直接操作它，框架會在幕後處理：

1. 當你使用 viewModel() 函數時，Android 系統會自動收集當前的上下文資訊（如 Application、SavedStateRegistryOwner），將它們打包成 CreationExtras。

2. **Factory 接收並使用：** 你用 viewModelFactory DSL 創建的 Factory 會接收這個 CreationExtras。在 initializer 裡，你可以直接用 this.createSavedStateHandle() 或 inventoryApplication() 這樣的方法，它們底層就是從 CreationExtras 裡提取數據來建構 ViewModel。

---

`*/**

** * Extension function to queries for **[Application]** object and returns an instance of

** * **[InventoryApplication]**.

** */

*fun CreationExtras.inventoryApplication(): InventoryApplication =

(this[AndroidViewModelFactory.APPLICATION_KEY] as InventoryApplication)`

這段程式碼是一種 **Kotlin 的擴充函式 (Extension Function)**就是為了解決底下的狀況

```
class AppViewModelFactory : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>, extras: CreationExtras): T {
        
        if (modelClass.isAssignableFrom(GameViewModel::class.java)) {
            // 在這裡手動寫一次取值和轉型
            val application = extras[AndroidViewModelFactory.APPLICATION_KEY] as InventoryApplication
            val itemRepository = application.container.itemsRepository
            
            @Suppress("UNCHECKED_CAST")
            return GameViewModel(itemRepository) as T
        }

        if (modelClass.isAssignableFrom(SettingsViewModel::class.java)) {
            // 在這裡又得把一模一樣的程式碼再寫一次！
            val application = extras[AndroidViewModelFactory.APPLICATION_KEY] as InventoryApplication
            val settingsRepository = application.container.settingsRepository
            
            @Suppress("UNCHECKED_CAST")
            return SettingsViewModel(settingsRepository) as T
        }

        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

1. **核心問題**：ViewModel 是由 Android 框架管理的，我們不能直接 new 它。但它在建立時，常常需要 Application Context 或 Repository 這類依賴。如何安全又優雅地把這些依賴傳遞進去？

2. **現代解法：CreationExtras** Android Jetpack 的現代解法是 CreationExtras。你可以把它想像成一個由框架傳遞過來的**標準化「工具箱」**。當框架要建立 ViewModel 時，會把 Application 這類有用的工具放進箱子裡，交給你的工廠。

3. **捷徑** 直接從「工具箱」裡拿工具的程式碼比較囉嗦，像這樣：

```
val application = extras[AndroidViewModelFactory.APPLICATION_KEY] as InventoryApplication
```

1. 擴充函式，就是把這段重複、醜陋的程式碼，封裝成一個優雅的捷徑： fun CreationExtras.inventoryApplication(): …

2. **目的**：為 CreationExtras（工具箱）增加一個名為 inventoryApplication() 的新功能。

3. **作用**：幫你完成「用官方鑰匙打開工具箱，拿出 Application 工具，並轉換成你自訂的 InventoryApplication 類型」這一整套操作

[Android — AndoridViewModel透過Factory建立帶有參數的ViewModel如何設定](https://jefflin1982.medium.com/android-andoridviewmodel%E9%80%8F%E9%81%8Efactory%E5%BB%BA%E7%AB%8B%E5%A6%82%E4%BD%95%E8%A8%AD%E5%AE%9A-769e9544db24)

> 

其實viewModel 的constructor本身應該是要被設定為private constructor才對，

[https://blog.csdn.net/vitaviva/article/details/123321254](https://blog.csdn.net/vitaviva/article/details/123321254)
