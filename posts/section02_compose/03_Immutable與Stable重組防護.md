# @Immutable @Stable 如果 immutableList 內包含 list ．這樣會造成 recompstion 嗎？

> 原文網址：https://boochlin.com/?p=820
> 發布日期：2025-08-05
> 文章編號：820

---

會．結束

#### **1. List 會造成嚴重的重組問題嗎？在哪種情況？**

**答案是：會，而且絕對比你想像的更嚴重。**

kotlin.collections.List 在 Compose 的世界中被視為「不穩定 (Unstable)」，因為它是一個介面，Compose 編譯器無法保證它的實作在執行期間不會被修改。

**最嚴重的情況有兩種：**

1. LazyColumn 或 LazyRow 顯示成百上千個項目時，一次不必要的重組意味著上百個 Composable 函式被重新執行，CPU 瞬間飆高。

2. 當列表所在的畫面上方有一個計時器或任何頻繁變動的狀態時，即使這個狀態與列表資料無關，也可能導致列表被一遍又一遍地無謂重繪。

Kotlin

```
@Composable
fun MainScreen() {
    // 這個計數器與列表資料無關，但它的改變會觸發 MainScreen 重組
    var unrelatedCounter by remember { mutableStateOf(0) }
    val userList = remember { listOf("Alice", "Bob", "Charlie") } // 一個標準 List

    Column {
        Text("計數器: $unrelatedCounter")
        Button(onClick = { unrelatedCounter++ }) {
            Text("觸發父層重組")
        }

        // 因為 userList 是不穩定的 List，每次按鈕被點擊，
        // UserList 這個 Composable 都會被強制重組。
        UserList(users = userList)
    }
}

@Composable
fun UserList(users: List<String>) {
    Log.d("Recomposition", "UserList 正在重組...") // 你會在 Logcat 看到這行不斷出現
    LazyColumn {
        items(users) { user ->
            Text(user)
        }
    }
}
```

在上述範例中，每次點擊按鈕，UserList 都會重組，這就是效能浪費的根源。

#### **2. 如果用 ImmutableList 可以改善嗎？**

**可以，而且是立竿見影的改善。**

kotlinx.collections.immutable.ImmutableList 從類型層面就向編譯器立下了一個「保證書」：**我一旦被創建，就永遠不會改變**。

有了這個保證，Compose 就可以放心地進行優化。當父層重組時，Compose 檢查傳給 UserList 的 ImmutableList，只需做一次結構相等性比較（equals()）。若物件的內容沒變（equals() 返回 true），Compose 就知道不需要觸發重組。

你只需將 listOf(…) 改為 persistentListOf(…) 或 .toImmutableList()，上面範例中的效能問題就迎刃而解。

#### **3. ImmutableList 如果裡面有 List 會有問題嗎？**或者反過來想想看

**會。** 但是到底是啥問題

如果你這麼寫：會發生啥事？

Kotlin

```
State<ImmutableList<MutableList<T>>>
// 對 MutableList add new item,  不會發生重組

State<ImmutableList<List<T>>>
// List<T> 是不穩定的，不管你怎麼包，我就是不信任，不會跳過重組，每次都重組

State<ImmutableList<ImmutableList<T>>>
// 超級保證，內到外都保證．不會重組

@Immutable
data class StableListWrapper<T>(val items: List<T>)
// 都說了，下保證@Immutable 絕對不會重組
```

```
// 這是一個父層元件，它有兩個自己的狀態
@Composable
fun ParentScreen() {
    // 狀態 1：控制一個開關是否開啟
    var isToggleOn by remember { mutableStateOf(false) }

    val userList: ImmutableList<List<String>> = remember {
        persistentListOf(
            listOf("Alice", "Engineer"),
            listOf("Bob", "Designer")
        )
    }

    Column {
        Row {
            Text("Enable Feature:")
            Switch(checked = isToggleOn, onCheckedChange = { isToggleOn = it })
        }

        UserListComponent(users = userList)
    }
}

// 這是一個子層元件，專門用來顯示列表
@Composable
fun UserListComponent(users: ImmutableList<List<String>>) {
    //... 這裡會用 LazyColumn 等等來顯示用戶列表...
    Log.d("Recomposition", "UserListComponent is being recomposed.")
}
```

#### **4. @Immutable 怎麼做到的？**

當 Compose 無法自動推斷一個類別的穩定性時（例如它來自別的模組，或包含了非標準的類型），你可以手動為它蓋上一個「品質保證」的章

@Immutable 是你能給出的**最強**保證書。它告訴編譯器：

> 

「我保證，這個類別的所有公開屬性都是 val，且它們的類型也都是不可變的。你可以完全信任我，只要我的物件參考沒變，我從裡到外都絕無變化。」

Kotlin

```
@Immutable
data class UiConfig(val title: String, val features: ImmutableSet<String>)
```

Compose 看到這個註解，就會將 UiConfig 視為穩定類型，賦予它跳過重組的能力。

#### **5. @Stable 聽說也是差不多功能，但是…**

是的，@Stable 和 @Immutable 的目標相同：**都是為了告訴 Compose「我是穩定的，你可以優化我」**。

**「但是」**，它們的承諾方式不同：

* **@Immutable 一個更強的保證。表示所有屬性都是 val 且類型不可變。該物件是「深度」不可變的。**

* **@Stable 一個更通用的保證。表示該類型可能是可變的，但它承諾在任何變更發生時都會通知 Compose。**

@Stable 允許一個類別內部持有**可變的狀態**，但前提是這個狀態的變化必須是 Compose **可以觀察到的**（例如使用 MutableState）。

Kotlin

```
@Stable
class SearchUiState(
    val query: MutableState<String>,
    val searchHistory: String
)
```

@Stable 告訴 Compose：「我的 query 可能會變，但它是 MutableState，所以它一變你就會知道。你可以根據 query 的變化來決定是否重組。」

這讓 @Stable 在管理需要與 UI 互動的、動態變化的狀態時非常有用。但要小心，如果你在 @Stable 類別中混入了像 List 這樣的不穩定類型，它的穩定性承諾就會被打折扣，甚至失效。
