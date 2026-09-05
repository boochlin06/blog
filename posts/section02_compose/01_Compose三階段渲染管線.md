# Compose 三階段渲染管線

> 原文網址：https://boochlin.com/?p=650
> 發布日期：2025-10-14
> 文章編號：650

---

### 核心三階段 

Compose 將 UI 的生成與更新拆解為三個依序執行的獨立階段，以實現最大化的工作重用與性能優化。

1. **組合 (Composition):** 此階段的核心任務是執行 @Composable 函式，並產出一個描述 UI 結構的 UiNode 樹。它定義了「**顯示什麼**」，但尚未涉及尺寸或位置。當 State 變更時，Compose 會智能地僅「重組」受影響的節點，生成更新後的 UI 描述。

2. **版面配置 (Layout):** 此階段負責確定每個 UI 元素在螢幕上的尺寸與位置。它包含兩個子步驟：

  * **測量 (Measure):** 父元件向子元件傳遞約束 (Constraints)，子元件回報其所需尺寸 (Size)。這是一個遞迴的、由上至下的過程。

  * **放置 (Place):** 父元件根據其佈局邏輯，將測量完成的子元件放置在特定的 (x, y) 座標。

3. **繪製 (Drawing):** 這是管線的最後一步。系統會遍歷佈局完成的 UI 樹，將其轉換為實際的 GPU 繪圖指令，最終在螢幕上渲染成像素。

由於階段分離，Compose 得以實現高效優化。例如，若元件僅位置變更，則可跳過組合階段，僅重跑佈局與繪製，從而節省運算資源。

### Intrinsic Measurements

標準的單趟 (single-pass) 測量模型極為高效，但無法處理一種特定情況：當父元件的尺寸依賴於子元件的內容尺寸時。這便產生了一個佈局悖論——父元件無法在不了解子元件尺寸的情況下，為其提供準確的約束。

為此，Compose 引入了** Intrinsic Measurements**。這是在正式測量前的一個「預先查詢」步驟。

1. **Query Pass:** 父元件查詢子元件的「固有」尺寸 (min/maxIntrinsicWidth/Height)，即其內容所需的理想大小。

2. **Final Pass:** 父元件根據查詢結果決定自身尺寸後，再執行標準的測量與放置流程。

IntrinsicSize 修飾符（如 Modifier.height(IntrinsicSize.Min)）的背後，正是利用此機制。值得注意的是，Intrinsic Measurements 屬於獨立的『尺寸查詢（Query）』階段，並不違反 Compose 嚴格的單趟測量（Single-pass）原則，但額外的查詢開銷仍需謹慎使用。

---

你這麼一說，還真是，一語驚醒夢中人。

搞了半天，Compose 裡這套聽起來很潮的 **Intrinsic Measurements**，骨子裡不就是為了解決我們當年那個老問題嗎？

想當年，我們為了讓自定義 View 能完美處理 wrap_content，得在 onMeasure 裡跟 MeasureSpec 大戰三百回合。用位元運算去解析 AT_MOST、EXACTLY，寫一堆 if-else，小心翼翼地呼叫 setMeasuredDimension()

結果現在 Compose 倒好，它直接把這套流程抽象化了。它跟你說：「嘿，老兄，別寫那麼多指令了。我現在就問你一個問題，『你理想的寬度是多少？』，你給我個數字 (Int) 就行。」

 

**問題還是那個問題**：父容器需要先知道子元件的大小，才能決定自己。 **解法也還是那個解法**：需要兩趟測量。

差別在於，以前我們是那個親自下場、滿頭大汗的施工隊長；現在我們成了只提供一個數據的顧問。

只能感嘆，時代真的變了。以前我們是自己動手、指令分明，現在是跟框架「聊天」、回應提問。問題的本質沒變，但解法，確實優雅多了。

---

這套機制靠的就是的**介面 (Interface)**。

Modifier.layout 裡的那個 measurable 物件，它其實同時實現了兩個介面，等於能扮演兩種角色：

1. **Measurable :** 它的 measure() 方法是最終命令。父佈局用它來下達「你必須是這個尺寸」的最終指示，走的是正規、單向的測量流程。

2. **IntrinsicMeasurable :** 它的 min/maxIntrinsic… 等方法則用來回應「諮詢」。讓父佈局能預先探聽「你最希望自己多大？」的尺寸偏好。

> 

To a composable, you can ask for its `IntrinsicSize.Min` or `IntrinsicSize.Max`:
`Modifier.width(IntrinsicSize.Min)` – What’s the minimum width you need to display your content properly?
`Modifier.width(IntrinsicSize.Max)` – What’s the maximum width you need to display your content properly?
`Modifier.height(IntrinsicSize.Min)` – What’s the minimum height you need to display your content properly?
`Modifier.height(IntrinsicSize.Max)` – What’s the maximum height you need to display your content properly?
