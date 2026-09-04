# Jetpack Compose: 何時該用 derivedStateOf？

> 原文網址：https://boochlin.com/?p=713
> 發布日期：2025-07-17
> 文章編號：713

---

[官方導讀](https://medium.com/androiddevelopers/jetpack-compose-when-should-i-use-derivedstateof-63ce7954c11b)

derivedStateOf 是一個**效能優化工具**，專門用來防止因「來源狀態」變化過於頻繁而導致的**「不必要的 UI 重組 (Recomposition)」**。**。

---

#### 1. 衍生

一個常見的範例開場：一個可滾動的列表，以及一個「回到頂部」的按鈕。

* **需求：** 這個按鈕只有在使用者**向下滾動後**才需要顯示。

* **狀態關係：** 按鈕的顯示狀態 (showButton: Boolean)，是 derived **自**列表的滾動狀態 (listState.firstVisibleItemIndex > 0)。

Kotlin

```
// 錯誤的示範 (Naive Solution)
@Composable
fun MyList(listState: LazyListState) {
    // ...
    val showButton = listState.firstVisibleItemIndex > 0 // 直接計算

    // ...
    if (showButton) {
        ScrollToTopButton() // 任何用到 showButton 的地方都會受影響
    }
}
```

**問題在哪？**

listState.firstVisibleItemIndex (第一個可見項目) 這個狀態在使用者滾動列表時，會以**極高的頻率**改變（滾動觸發）。這導致 showButton 頻繁地重新計算。

雖然 showButton 的可能一直都是 true，但因為它的「來源」listState 一直在變，任何讀取 showButton 的 Composable 都會被**不斷地重組**，造成嚴重的效能浪費。

---

#### 2. derivedStateOf 的運作原理

Kotlin

```
// 推薦的正確解法 (Recommended Solution)
@Composable
fun MyList(listState: LazyListState) {
    // ...
    // 使用 derivedStateOf
    val showButton by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex > 0
        }
    }

    // ...
    if (showButton) {
        ScrollToTopButton()
    }
}
```

**它的運作方式如下：**

derivedStateOf 會觀察其 {…} 區塊內的計算。它只關心這個計算的**最終結果**是否與上一次相同。

* **初始狀態：** listState.firstVisibleItemIndex 是 0。derivedStateOf 計算出的結果是 false。

* **開始向下滾動：**

  * firstVisibleItemIndex 變成 1。計算結果從 false **變為** true。**→ derivedStateOf 通知 UI 重組，按鈕出現。**

  * firstVisibleItemIndex 繼續變成 2, 3, 4… 100。雖然來源狀態一直在變，但計算結果**始終是 true**。**→ derivedStateOf 發現結果沒變，因此「攔截」了更新，不會觸發 UI 重組。**

* **滾動回頂部：**

  * firstVisibleItemIndex 變回 0。計算結果從 true **變為** false。**→ derivedStateOf

**
