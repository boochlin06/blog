# Modifier 效能 Draw 啥比較好：drawBehind , drawWithContent,drawWithCache

> 原文網址：https://boochlin.com/?p=653
> 發布日期：2025-03-17
> 文章編號：653

---

Compose 的自訂繪圖是效能殺手。每次畫面重組，你的繪圖邏輯就得重跑一次，FPS 隨時可能崩盤。

#### 1. Modifier.drawBehind：背景板

它會先讓你畫圖，**然後**才把元件內容（文字、圖片）畫上去。

* **比喻：** 先在牆上塗鴉，再把畫框掛上去。

* **用途：** 畫簡單的背景或裝飾。

#### 2. Modifier.drawWithContent

它把繪製主導權交給你，你必須手動呼叫 drawContent()。你能在內容**前後**都畫圖。

* **比喻：** 攝影師，能決定先加濾鏡再拍照，或拍完照再加浮水印。

* **用途：** 做內容遮罩、漸層或添加邊框。

#### 3. Modifier.drawWithCache

這是前兩者的效能進化版。只要元件尺寸或你指定的狀態沒變，它就會直接用**快取**好的繪圖結果，而不是傻傻地重算一次。

* **比喻：** 餐廳預先做好一道功夫菜，客人點了直接上，不用等廚師現做。

* **用途：** 任何涉及複雜計算、建立 Shader 或 Path 的繪圖。

* **簡單背景？** 用 drawBehind。

* **要疊加內容？** 用 drawWithContent。

* **繪圖邏輯稍微複雜？** **一律用 drawWithCache 包起來！** 這是避免 App 卡頓的保命符。

* 

---

第二版，個人認為需要加入程式碼範例才符合我寫blog 的初衷

---

我們用 `Modifier.draw` 函數來實作一個圓餅圖，快速了解三者的區別。

首先，這是圓餅圖共用的資料：

Kotlin

```
import androidx.compose.ui.graphics.Color as ComposeColor

// 圓餅圖的數據模型
data class PieSlice(val value: Float, val color: ComposeColor, val label: String)

// 圓餅圖的數據
val pieChartData = listOf(
    PieSlice(30f, ComposeColor.Red, "Apple"),
    PieSlice(20f, ComposeColor.Blue, "Banana"),
    PieSlice(15f, ComposeColor.Green, "Orange"),
    PieSlice(35f, ComposeColor.Magenta, "Grape")
)
```

---

### 1. `Modifier.drawBehind`：背景板

在元件內容之前繪製，適合畫簡單的背景或裝飾。

比喻： 先在牆上塗鴉，再把畫框掛上去。

Kotlin

```
import androidx.compose.ui.draw.drawBehind
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier

@Composable
fun SimplePieChartWithDrawBehind(data: List<PieSlice>, modifier: Modifier = Modifier) {
    val total = data.sumOf { it.value.toDouble() }.toFloat()

    Box(modifier = modifier.fillMaxSize().drawBehind { // 在內容前繪製
        var startAngle = 0f
        data.forEach { slice ->
            val sweepAngle = (slice.value / total) * 360f
            drawArc(
                color = slice.color,
                startAngle = startAngle,
                sweepAngle = sweepAngle,
                useCenter = true
            )
            startAngle += sweepAngle
        }
    }) {
        // 內容 (Text) 會自動疊在圓餅圖之上
        Column {
             data.forEach { slice -> Text(text = slice.label) }
        }
    }
}
```

`drawBehind` 在 `Box` 的背景上畫圓餅圖，`Box` 裡面的 `Column` 和 `Text` 會自然地呈現在圖的上方。語法最簡單。

---

### 2. `Modifier.drawWithContent`：繪製主導權

讓你完全控制繪圖順序，你必須手動呼叫 drawContent() 來決定何時繪製元件內容。

比喻： 像攝影師，能決定先加濾鏡（繪製），再拍照 (drawContent)，或拍完照再加浮水印（再繪製）。

Kotlin

```
import androidx.compose.ui.draw.drawWithContent

@Composable
fun PieChartWithDrawWithContent(data: List<PieSlice>, modifier: Modifier = Modifier) {
    val total = data.sumOf { it.value.toDouble() }.toFloat()

    Box(modifier = modifier.fillMaxSize().drawWithContent { // 獲得繪製主導權
        // 1. 在內容前繪圖
        var startAngle = 0f
        data.forEach { slice ->
            val sweepAngle = (slice.value / total) * 360f
            drawArc(
                color = slice.color,
                startAngle = startAngle,
                sweepAngle = sweepAngle,
                useCenter = true
            )
            startAngle += sweepAngle
        }

        // 2. 繪製 Box 原本的內容 (如果有的話)
        // drawContent() 

        // 3. 在內容後繪圖 (例如加上標籤)
        // ... 繪製文字的邏輯 ...
    })
}
```

你可以自由決定在 `drawContent()` 前後都加上繪圖邏輯，適合做遮罩、漸層等需要疊加的效果。

---

### 3. `Modifier.drawWithCache`：效能

這是效能的保命符。繪圖邏輯只在元件尺寸或你指定的狀態改變時才重跑，否則直接使用快取好的結果。

比喻： 餐廳預先做好一道功夫菜，客人點了直接上，不用等廚師現做。

Kotlin

```
import androidx.compose.ui.draw.drawWithCache

@Composable
fun PerformantPieChartWithDrawWithCache(data: List<PieSlice>, modifier: Modifier = Modifier) {
    val total = data.sumOf { it.value.toDouble() }.toFloat()

    Box(modifier = modifier.fillMaxSize().drawWithCache {
        // 複雜的計算或物件建立 (如 Path, Shader) 放這裡
        // 這裡的程式碼只在尺寸改變時重跑

        onDrawWithContent { // 實際的繪圖指令放這裡
            var startAngle = 0f
            data.forEach { slice ->
                val sweepAngle = (slice.value / total) * 360f
                drawArc(
                    color = slice.color,
                    startAngle = startAngle,
                    sweepAngle = sweepAngle,
                    useCenter = true
                )
                startAngle += sweepAngle
            }
            // drawContent() // 如果需要，也可以呼叫
        }
    })
}
```

將繪圖邏輯包在 `onDrawWithContent` 中。只要尺寸不變，Compose 就會直接使用上次畫好的結果，避免了不必要的重算，效能最佳。

###
