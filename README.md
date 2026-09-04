# Something Record - Boochlin 技術隨筆全集

> **「記錄真實工程現場的每一次摸索、踩坑、破局與架構演進。」**

歡迎來到 **Boochlin's Tech Blog** 的完整開源存檔站。

本站精選收錄了自 2015 年至今發布的 **82 篇深度技術文章**，涵蓋邊緣端嵌入式 AI、Android 系統底層調優、高通 QFastCV / ARM NEON SIMD 彙編、Jetpack Compose 渲染管線、Kotlin 協程併發，以及十年間的移動端架構演進與軟體工程哲學。

---

## 專欄十大技術板塊

1. **[邊緣 AI：低階硬體極限榨汁實錄](posts/section01_edge_ai/00_全系列架構對照導覽.md)**（18 篇）：高通驍龍 QM215 四核 A53 + 2GB RAM 記憶體極限榨汁、影像零拷貝與 3D 視覺光學實戰。
2. **[現代前端：Jetpack Compose 渲染機制與效能調校](posts/section02_compose/01_Compose三階段渲染管線.md)**（4 篇）：三階段渲染管線、derivedStateOf、@Immutable 重組防護與 Modifier 繪製。
3. **[語言深水區：Kotlin 非同步併發與底層機制](posts/section03_kotlin/01_Thread與Coroutine理解.md)**（7 篇）：Thread 與 Coroutine 切換成本、協程防抖防競態、內聯機制 inline/noinline/crossinline、建構順序與屬性地雷。
4. **[架構模式：現代 Android 架構與依賴注入](posts/section04_architecture/01_viewModelScope_vs_rememberCoroutineScope.md)**（7 篇）：viewModelScope 職責分離、SupervisorJob 取消機制、Hilt 組內指南與架構邊界。
5. **[系統底層：Android Framework 原理與原始碼探秘](posts/section05_framework/01_ThreadPoolManager的原罪.md)**（7 篇）：ThreadPoolManager 的原罪、RRO 運行時資源覆蓋、createPackageContext 沙盒隔離與反射黑魔法。
6. **[效能工程：系統診斷、除錯工具與排障實錄](posts/section06_performance/01_Analyzing_UI_Performance_Systrace.md)**（10 篇）：Systrace UI 效能分析、Stetho 除錯、Fragment 通訊與巢狀捲動 RecyclerView 排障。
7. **[多媒體驅動：相機 HAL 與底層驅動架構](posts/section07_camera_multimedia/01_Camera1_Architecture.md)**（8 篇）：Camera 1 到 Camera 2 架構演進三部曲、AOSP 原生 FMRadio 系統服務拆解五部曲。
8. **[實戰工程：獨立專案、機器人定位與跨平台](posts/section08_projects_iot/01_Funny_Vote_開發雜記.md)**（6 篇）：Funny Vote 獨立產品迭代全歷程、雷射雷達 SLAM NDT 點雲匹配定位、React Native 原生模組避坑。
9. **[拓荒記憶：經典網路庫、資料庫 ORM 與工具演進](posts/section09_orm_tools/01_Retrofit2_Upload_Files.md)**（10 篇）：GreenDAO 3、Retrofit 2 檔案上傳、Volley GZIP、ButterKnife 視圖綁定與 GAIQ 認證。
10. **[思考隨筆：軟體工程思維、代碼整潔與大師哲學](posts/section10_philosophy/01_Clean_code_書摘心得一.md)**（5 篇）：Clean Code 代碼整潔之道精華、The Clean Coder 職業素養、約爾趣談軟體筆記。

---

## 閱讀與搜尋指引

- 請點擊左側目錄導航開始閱讀各專欄章節。
- 按鍵盤 `S` 鍵可快速呼叫全站即時全文檢索（基於 Lunr 離線索引引擎）。
