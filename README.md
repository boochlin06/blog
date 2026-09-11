# Something Record - Boochlin 技術隨筆全集

> **「記錄真實工程現場的每一次摸索、踩坑、破局與架構演進。」**

歡迎來到 **Boochlin's Tech Blog** 的完整開源存檔站。

本站精選收錄了自 2015 年至今發布的 **82 篇深度技術長文**。全站架構以「軟體工程哲學為基石、底層極限榨汁為突破、現代架構語言為骨幹、系統框架排障為底蘊、生態拓荒記憶為血肉」，依序分為卷首與四大分卷：

---

## 全書分卷體系導航

### [卷首：工程哲學與代碼整潔之道](posts/section10_philosophy/01_Clean_code_書摘心得一.md)（5 篇）
- 以經典大師著作（Clean Code、The Clean Coder、約爾趣談軟體）為起點，建立「工程師素養、代碼審美與系統邊界」的根本心法，作為全書技術探索的第一維度。

### [卷一：邊緣 AI 與低階硬體極限榨汁](posts/section01_edge_ai/00_全系列架構對照導覽.md)（旗艦專案 18 篇全集）
- 高通驍龍 QM215 四核 A53 + 2GB RAM 破局實錄：
  - 階段一：硬體現狀評估與系統裁剪（00 ~ 02 篇）
  - 階段二：Linux 核心調校與記憶體控場（03 ~ 05 篇）
  - 階段三：影像零拷貝、NEON 加速與保活（06 ~ 09.1 篇）
  - 階段四：3D 視覺光學與邊緣 IoT 通訊（10 ~ 14 篇）

### [卷二：現代 Android 架構、語言與渲染引擎](posts/section02_compose/01_Compose三階段渲染管線.md)（18 篇）
- Jetpack Compose 三階段渲染管線、derivedStateOf、重組防護與 Modifier 繪製。
- Kotlin 協程切換成本、搜尋防抖防競態、inline/noinline/crossinline 深度剖析。
- 現代架構模式（MVI / viewModelScope / Hilt 依賴注入 / Repository 邊界）。

### [卷三：系統核心機制、相機驅動與效能排障](posts/section05_framework/01_ThreadPoolManager的原罪.md)（25 篇）
- Android Framework 原始碼探秘、執行緒池歷史原罪、RRO 資源替換與 IPC 沙盒。
- 相機 HAL3 管道化模型（Camera 1 到 Camera 2 演進三部曲）、原生 FMRadio 系統服務拆解。
- 系統級效能診斷：Systrace 掉幀分析、Stetho 偵錯、軟鍵盤遮擋與滾動衝突解決。

### [卷四：工程專案實踐與生態拓荒史](posts/section08_projects_iot/01_Funny_Vote_開發雜記.md)（16 篇）
- Funny Vote 獨立產品全歷程（MVP 到 Kotlin 重構）、雷射雷達 SLAM NDT 點雲匹配、React Native 混合踩坑。
- 早期經典網路庫（Retrofit 2 / Volley）、ORM 資料庫（GreenDAO 3 原始碼追蹤）、視圖綁定 ButterKnife 與 GAIQ 數據認證。

---

## 全站分類與專題體系 (Taxonomy)

全站文章支援「縱向知識領域（7 大 Categories）」與「橫向專題連載（7 大 Series）」三維檢索：
* 完整架構規範與 82 篇文章精確分類對照表：請詳閱 **[全站分類與專題架構體系 (TAXONOMY.md)](TAXONOMY.md)**

---

## 閱讀與搜尋指引

- 請點擊左側目錄導航開始閱讀各分卷章節。
- 按鍵盤 `S` 鍵可快速呼叫全站即時全文檢索（基於 Lunr 離線索引引擎）。
