# Summary

* [全站導讀：十年隨筆與工程破局心法](README.md)
* [分類體系：全站 7 大領域與專題架構指南](TAXONOMY.md)

## 卷首：工程哲學與代碼整潔之道（5 篇）

* [代碼整潔：Clean Code 無瑕的程式碼（一）：命名、函式與職責精準](posts/section10_philosophy/01_Clean_code_書摘心得一.md)
* [代碼整潔：Clean Code 無瑕的程式碼（二）：錯誤處理、邊界與單元測試](posts/section10_philosophy/02_Clean_code_書摘心得二.md)
* [工程師素養：The Clean Coder 專業程式設計師的生存與承諾法則](posts/section10_philosophy/03_The_Clean_coder_番外篇.md)
* [產品哲學：約爾趣談軟體（Joel on Software）經典研讀筆記](posts/section10_philosophy/04_約爾趣談軟體筆記.md)
* [早期拓荒：Android 核心架構技術精選與歷史記憶（2015）](posts/section10_philosophy/05_Android好文整理.md)

## 卷一：邊緣 AI 與低階硬體極限榨汁（旗艦專案 18 篇）

* [架構導讀：全篇演進與防坑總對照](posts/section01_edge_ai/00_全系列架構對照導覽.md)
* 階段一：硬體現狀評估與系統裁剪
  * [00 篇：到底有多爛？算力深淵與硬體現狀評估](posts/section01_edge_ai/00_到底有多爛.md)
  * [01 篇：評估第一：硬體瓶頸與選型考量](posts/section01_edge_ai/01_評估第一.md)
  * [02 篇：系統裁剪：無用系統服務徹底剔除](posts/section01_edge_ai/02_無用系統服務out.md)
* 階段二：Linux 核心調校與記憶體控場
  * [03 篇：核心擴張：ZRAM、SWAP 與 KSM 調校](posts/section01_edge_ai/03_ZRAM_SWAP_KSM.md)
  * [04 篇：記憶體控場：LMKD 策略與防 OOM 實戰](posts/section01_edge_ai/04_記憶體控場師LMKD.md)
  * [05 篇：記憶體哲學：GC 是唯一關心你 OOM 的好人](posts/section01_edge_ai/05_GC是唯一關心你OOM的好人.md)
* 階段三：影像零拷貝、NEON 加速與保活
  * [06 篇：影像零拷貝：三路高頻影像流記憶體防護](posts/section01_edge_ai/06_三路高頻影像流零拷貝.md)
  * [07 篇：硬體保活：物理世界需要 Watchdog](posts/section01_edge_ai/07_物理世界需要Watchdog.md)
  * [07.1 篇：進程調度：GC Thrashing 調校思路補充](posts/section01_edge_ai/07_1_GC_Thrashing思路補充.md)
  * [08 篇：色彩加速：YUV420 888 與 QFastCV 實戰](posts/section01_edge_ai/08_YUV加速QFastCV.md)
  * [09 篇：矩陣加速：ARM Compute Library SGEMM 實戰](posts/section01_edge_ai/09_兄弟齊心ACL_SGEMM.md)
  * [番外篇：效能榨汁：ARM NEON 與 DirectBuffer 終極調優](posts/section01_edge_ai/09_1_NEON與記憶體極限榨汁番外篇.md)
* 階段四：3D 視覺光學與邊緣 IoT 通訊
  * [10 篇：時空對齊：RGB 與 ToF 雙鏡頭同步](posts/section01_edge_ai/10_雙鏡頭時空同步.md)
  * [11 篇：點雲解算：ToF RAW12 轉 3D 空間點雲](posts/section01_edge_ai/11_ToF_RAW12轉3D點雲.md)
  * [12 篇：長連線架構：WebSocket 離線雙寫同步機制](posts/section01_edge_ai/12_說到底還是一台IoT_WebSocket.md)
  * [13 篇：即時通訊：WebRTC 視訊對講底層魔改](posts/section01_edge_ai/13_為了極少用功能大改的WebRTC.md)
  * [14 篇：近場配網：門禁機當 BLE Peripheral 現場踩坑記](posts/section01_edge_ai/14_門禁機當BLE_Peripheral現場踩坑記.md)

## 卷二：現代 Android 架構、語言與渲染引擎（18 篇）

* Jetpack Compose 渲染機制與效能調校
  * [渲染管線：Compose 三階段渲染（Composition / Layout / Draw）深度剖析](posts/section02_compose/01_Compose三階段渲染管線.md)
  * [狀態衍生：derivedStateOf 的精確使用時機與底層機制](posts/section02_compose/02_何時該用derivedStateOf.md)
  * [重組防護：@Immutable 與 @Stable 包含集合時的 Recomposition 機制](posts/section02_compose/03_Immutable與Stable重組防護.md)
  * [繪製優化：Modifier 繪製效能抉擇：drawBehind vs drawWithCache](posts/section02_compose/04_Modifier繪製效能比較.md)
* Kotlin 語言深水區與非同步併發
  * [併發模型：Thread 與 Coroutine 的底層機制與上下文切換成本](posts/section03_kotlin/01_Thread與Coroutine理解.md)
  * [併發防抖：協程防連發、搜尋任務防抖與競態消除](posts/section03_kotlin/02_Kotlin預防Launch連發防抖.md)
  * [內聯機制：inline、noinline、crossinline 的本質與存在意義](posts/section03_kotlin/03_inline_noinline_crossinline.md)
  * [延遲載入：lateinit 與 lazy 的位元組碼實現與執行期差異](posts/section03_kotlin/04_lateinit_vs_lazy.md)
  * [物件建構：次級建構子（Constructor）與 init 程式碼區塊的執行順序](posts/section03_kotlin/05_constructors_and_init.md)
  * [屬性初始化：init 區塊與屬性初始化的順序地雷](posts/section03_kotlin/06_kotlin_init屬性陷阱.md)
  * [構建腳本：Gradle Plugin Management 中 apply false 的深層原因](posts/section03_kotlin/07_gradle_kotlin_plugins.md)
* 現代架構模式與依賴注入實戰
  * [職責劃分：viewModelScope 與 rememberCoroutineScope 的職責分離](posts/section04_architecture/01_viewModelScope_vs_rememberCoroutineScope.md)
  * [異常處理：SupervisorJob 與 viewModelScope 的生命週期綁定機制](posts/section04_architecture/02_SupervisorJob與viewModelScope.md)
  * [依賴注入：Hilt 實戰架構指南與工程實踐](posts/section04_architecture/03_Hilt指南組內分享.md)
  * [實例工廠：ViewModelProvider.Factory 的設計必要性與自訂實例化](posts/section04_architecture/04_ViewModelProviderFactory.md)
  * [注入陷阱：Composable 預設參數直接實例化 ViewModel 的架構反模式](posts/section04_architecture/05_預設參數注入陷阱.md)
  * [架構邊界：超出 ComposableViewModel 生命週期的 Repository 任務邊界](posts/section04_architecture/06_RepositoryScope架構邊界.md)
  * [動態參數：CreationExtras 與自訂 Application 依賴提取](posts/section04_architecture/07_CreationExtras注入.md)

## 卷三：系統核心機制、相機驅動與效能排障（25 篇）

* Android Framework 原理與執行緒池演進
  * [執行緒池演進：全域單例 ThreadPoolManager 的歷史原罪與現代化重構](posts/section05_framework/01_ThreadPoolManager的原罪.md)
  * [併發診斷：ExecutorService 執行緒與執行緒池的規範命名實踐](posts/section05_framework/02_Naming_threads_and_thread_pools.md)
  * [系統自訂：Runtime Resource Overlay (RRO) 運行期資源替換原理](posts/section05_framework/03_runtime_resource_overlay.md)
  * [沙盒隔離：跨應用存取與資料隔離：createPackageContext() 實戰](posts/section05_framework/04_datacontrol分離_createPackageContext.md)
  * [系統生命週期：Android Home Screen 與 Launcher 生命週期深度解析](posts/section05_framework/05_home_screen_life_cycle.md)
  * [動態載入：Java 反射黑魔法：FastScrollBar 動態載入外部 APK](posts/section05_framework/06_Java_reflection_dynamic_load.md)
  * [系統演進：Android Nougat (Android 7.0) 核心新特性與架構變革](posts/section05_framework/07_Android_Nougat_introduction.md)
* 相機 HAL 與原生多媒體驅動框架
  * [Camera 架構（一）：Camera 1 經典架構與驅動層資料流](posts/section07_camera_multimedia/01_Camera1_Architecture.md)
  * [Camera 架構（二）：Camera 2 HAL3 管道化模型與核心概念](posts/section07_camera_multimedia/02_Camera2_Introduction.md)
  * [Camera 架構（三）：Camera 2 Session 建立與 CaptureRequest 實戰](posts/section07_camera_multimedia/03_How_to_create_Camera2.md)
  * [FMRadio 開發（一）：系統架構概論與硬體中介層通訊](posts/section07_camera_multimedia/04_FMRadio_概論.md)
  * [FMRadio 開發（二）：FMRadioMain 前端介面與硬體狀態同步](posts/section07_camera_multimedia/05_FMRadio_FMRadioMain.md)
  * [FMRadio 開發（三）：FMRadioService 背景常駐服務與音訊焦點](posts/section07_camera_multimedia/06_FMRadio_FMRadioService.md)
  * [FMRadio 開發（四）：AlertActivity 系統中斷與彈出視窗處理](posts/section07_camera_multimedia/07_FMRadio_AlterActivity.md)
  * [FMRadio 開發（五）：特殊情境例外排障與全系列收官](posts/section07_camera_multimedia/08_FMRadio_Other.md)
* 效能診斷、除錯工具與 UI 排障實錄
  * [卡頓診斷：使用 Systrace 深度分析 UI 幀率與掉幀瓶頸](posts/section06_performance/01_Analyzing_UI_Performance_Systrace.md)
  * [除錯工具：Facebook Stetho 結合 Chrome DevTools 實戰](posts/section06_performance/02_Chrome_dev_tool_stetho.md)
  * [跨進程通訊：Chrome 深度連結與 Android Intent 協議交互實作](posts/section06_performance/03_Android_Intents_with_Chrome.md)
  * [元件通訊：Fragment 之間高內聚、低耦合的通訊模式](posts/section06_performance/04_Communicate_with_fragments.md)
  * [生命週期地雷：Fragment 為何絕對嚴禁非預設無參建構子？](posts/section06_performance/05_Avoid_non-default_constructors_fragments.md)
  * [軟鍵盤遮擋：adjustResize、adjustPan 與沉浸式輸入框遮擋排障](posts/section06_performance/06_UI佈局被鍵盤擋住.md)
  * [滾動衝突解決：NestedScrollView 內部巢狀 RecyclerView 的滑動與尺寸修復](posts/section06_performance/07_RecyclerView_inside_NestedScrollView.md)
  * [程式碼混淆：AppIntro 開源庫引發的 ProGuard 規則踩坑記錄](posts/section06_performance/08_AppIntro_Proguard_issue.md)
  * [模組化依賴：本機構建引入第三方二進制依賴庫的配置策略](posts/section06_performance/09_Local_build_third_parity_lib.md)
  * [CTS 認證排障：Android CTS 測試失敗 — 外部儲存目錄無法刪除診斷](posts/section06_performance/10_Android_CTS_Fail.md)

## 卷四：工程專案實踐與生態拓荒史（16 篇）

* 獨立產品實戰、機器人定位與跨平台
  * [獨立產品：Funny Vote 投票 App 從零架構與開發雜記](posts/section08_projects_iot/01_Funny_Vote_開發雜記.md)
  * [架構演進：Funny Vote 引入 MVP 架構模式與模組化重構](posts/section08_projects_iot/02_Funny_vote_MVP更新雜記.md)
  * [語言遷移：Funny Vote 全專案 Kotlin 改版遷移紀錄](posts/section08_projects_iot/03_Funny_vote_Kotlin改版雜記.md)
  * [機器人定位：雷達 SLAM 演算法與 NDT 點雲匹配筆記](posts/section08_projects_iot/04_SLAM_Note_NDT_TKU.md)
  * [跨平台技術：React Native 早期架構與混合開發實踐](posts/section08_projects_iot/05_React_Native_學習筆記.md)
  * [原生模組互通：React Native 與 Android 原生 Native Module 互通踩坑](posts/section08_projects_iot/06_React_native_Native_android_module.md)
* 經典網路庫、資料庫 ORM 與工具演進
  * [網路庫演進：Retrofit 2 檔案上傳與 Multipart 參數列表實戰](posts/section09_orm_tools/01_Retrofit2_Upload_Files.md)
  * [網路庫演進：Google Volley 自訂 Request：整合 Gzip 壓縮與 Gson 解析](posts/section09_orm_tools/02_Volley_gzip_gson.md)
  * [ORM 資料庫：GreenDAO 3 註解架構與效能機制入門](posts/section09_orm_tools/03_GreenDao3_Introduction.md)
  * [ORM 資料庫：GreenDAO 核心架構、資料表關聯與 Session 管理](posts/section09_orm_tools/04_GreenDAO_introduction.md)
  * [ORM 資料庫：GreenDAO 欄位預設值無效之底層原始碼追蹤](posts/section09_orm_tools/05_GreenDAO_default_value.md)
  * [視圖綁定：ButterKnife 註解綁定原理與 View 編譯期生成](posts/section09_orm_tools/06_ButterKnife_introduction.md)
  * [社群整合：Facebook SDK 登入授權與原生 App 交互接入](posts/section09_orm_tools/07_Facebook_api_Login.md)
  * [數據分析認證：Google Analytics 個人認證（GAIQ）備考與技術心得](posts/section09_orm_tools/08_GAIQ_GET.md)
  * [數據埋點：Google Analytics for Android SDK 整合與初始化指南](posts/section09_orm_tools/09_Google_Analytics_初始化.md)
  * [構建發布：使用 Gradle 腳本打包與發布獨立 JAR 庫](posts/section09_orm_tools/10_Release_jar_by_gradle.md)
