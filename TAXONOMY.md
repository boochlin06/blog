# Something Record - 技術部落格分類與專題體系（Taxonomy Architecture）

> 本文件定義 Boochlin 技術隨筆全集（82 篇硬核技術文章）的「三維檢索架構」：
> 1. **Category（縱向知識領域）**：全站 7 大骨架領域，每篇文章僅指定一個主分類。
> 2. **Series（橫向專題連載）**：精選 7 大專題路線，提供循序漸進的深讀體驗。
> 3. **Tag（精確檢索座標）**：API、框架、函式庫與名詞，禁止作為 Category。

---

## 一、 7 大縱向領域分類（Categories）

| # | 分類名稱 | 英文 Slug | 篇數 | 領域定位與涵蓋範圍 |
| :-: | :--- | :--- | :-: | :--- |
| 1 | **現代 Android 開發** | `modern-android` | 18 | Jetpack Compose 渲染機制、Kotlin 協程深水區、Hilt 依賴注入、現代應用架構與生命週期 |
| 2 | **Android 效能與穩定性** | `android-performance` | 15 | Systrace 卡頓診斷、LMKD 記憶體控場、ZRAM/KSM、GC 治理、零拷貝防護、CTS 排障 |
| 3 | **工程工具與交付** | `engineering-tools` | 12 | 經典網路庫（Retrofit/Volley）、資料庫 ORM（GreenDAO）、Gradle 建構打包、GA 數據分析、跨平台模組 |
| 4 | **Android 平台與系統** | `android-platform` | 11 | AOSP Framework 原理、系統服務裁剪、Runtime Resource Overlay (RRO)、進程生命週期、長連線保活 |
| 5 | **邊緣 AI 與智慧裝置** | `edge-ai-iot` | 10 | 高通低階硬體極限榨汁、ARM NEON 向量加速、ToF 3D 點雲解算、BLE 近場配網、SLAM 機器人定位 |
| 6 | **影像、相機與多媒體** | `camera-multimedia` | 8 | Camera 1 經典驅動、Camera 2 HAL3 管道化模型、CaptureSession、原生 FMRadio 系統服務與中斷處理 |
| 7 | **架構實戰與技術觀點** | `engineering-notes` | 8 | Clean Code 無瑕代碼心得、軟體工匠素養、Joel on Software 研讀、獨立產品 Funny Vote 演進史 |

---

## 二、 7 大橫向專題連載（Series）

專題為「橫向貫穿」的沉浸式閱讀體驗，文章同時享有其縱向 Category：

1. **[旗艦專案] 爛裝置想跑 AI（18 篇全集）** (`running-ai-on-constrained-devices`)
   - 完整記錄高通四核 A53 + 2GB RAM 上，從系統裁剪、ZRAM、LMKD、零拷貝、NEON、ToF 到 BLE 配網的破局歷程。
2. **Jetpack Compose 深入剖析（4 篇）** (`jetpack-compose-in-depth`)
   - Compose 三階段渲染管線、derivedStateOf、Immutable/Stable 重組防護、Modifier 繪製優化。
3. **Kotlin 執行原理與併發（7 篇）** (`kotlin-under-the-hood`)
   - 協程與 Thread 底層、搜尋防抖、inline 位元組碼、lateinit vs lazy、init 陷阱。
4. **Android 現代架構與生命週期（7 篇）** (`android-architecture-lifecycle`)
   - viewModelScope、SupervisorJob、Hilt 多模組、ViewModelProvider.Factory、CreationExtras。
5. **Android 效能診斷與 UI 排障（10 篇）** (`android-performance-diagnostics`)
   - Systrace 掉幀分析、Stetho 除錯、軟鍵盤遮擋、NestedScrollView 衝突修復。
6. **Android 原生相機與多媒體子系統（8 篇）** (`android-camera-multimedia-stack`)
   - Camera 1 到 Camera 2 HAL3 演進三部曲、FMRadio 硬體服務中斷與音訊焦點。
7. **獨立產品 Funny Vote 架構演進史（3 篇）** (`funny-vote-evolution`)
   - 獨立 App 從零到一、MVP 架構重構到全專案 Kotlin 遷移之全歷程。

---

## 三、 歷史技術文章處置哲學（Legacy Tech Strategy）

過時的是方案，不是經驗。對於 2015~2017 年的技術文章（如 GreenDAO, Volley, ButterKnife），採取以下三原則：
- **按技術本質歸入主分類**：不設「老文」或「過時技術」分類，避免貶低技術沉澱。
- **標記技術生命週期**：
  - `Current`：仍適用於現代新專案。
  - `Contextual`：核心觀念與架構仍具參考價值，但實作版本已演進。
  - `Legacy`：不建議新專案採用，具備除錯、系統維護與技術演進價值。
- **附加 `legacy-tech` 標籤**。

---

## 四、 全站 82 篇文章精確分類對照總表

| # | 文章標題 | 領域主分類 (Category) | 專題連載 (Series) | 狀態 | 本地檔案路徑 |
| :-: | :--- | :--- | :--- | :-: | :--- |
| 1 | [Android 爛裝置想跑人臉辨識系列 – 風格重構審閱導覽](posts/section01_edge_ai/00_全系列架構對照導覽.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/00_全系列架構對照導覽.md` |
| 2 | [Android 爛裝置想跑人臉辨識0 – 到底有多爛？](posts/section01_edge_ai/00_到底有多爛.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/00_到底有多爛.md` |
| 3 | [Android 爛裝置想跑人臉辨識1 – 評估第一](posts/section01_edge_ai/01_評估第一.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/01_評估第一.md` |
| 4 | [Android 爛裝置想跑人臉辨識2 – 記憶體優化：無用系統服務徹底剔除](posts/section01_edge_ai/02_無用系統服務out.md) | `Android 平台與系統` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/02_無用系統服務out.md` |
| 5 | [Android 爛裝置想跑人臉辨識3 – 系統優化：ZRAM, SWAP, KSM 擴張最大值](posts/section01_edge_ai/03_ZRAM_SWAP_KSM.md) | `Android 效能與穩定性` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/03_ZRAM_SWAP_KSM.md` |
| 6 | [Android 爛裝置想跑人臉辨識4 – 記憶體控場師LMKD](posts/section01_edge_ai/04_記憶體控場師LMKD.md) | `Android 效能與穩定性` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/04_記憶體控場師LMKD.md` |
| 7 | [Android 爛裝置想跑人臉辨識5 – GC 是唯一會關心你 OOM 的好人](posts/section01_edge_ai/05_GC是唯一關心你OOM的好人.md) | `Android 效能與穩定性` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/05_GC是唯一關心你OOM的好人.md` |
| 8 | [Android 爛裝置想跑人臉辨識6 – 三路高頻影像流：三倍蓋亞，趕快 GC](posts/section01_edge_ai/06_三路高頻影像流零拷貝.md) | `Android 效能與穩定性` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/06_三路高頻影像流零拷貝.md` |
| 9 | [Android 爛裝置想跑人臉辨識 – GC Thrashing 思路補充](posts/section01_edge_ai/07_1_GC_Thrashing思路補充.md) | `Android 效能與穩定性` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/07_1_GC_Thrashing思路補充.md` |
| 10 | [Android 爛裝置想跑人臉辨識7 – 物理世界需要 watchdog](posts/section01_edge_ai/07_物理世界需要Watchdog.md) | `Android 平台與系統` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/07_物理世界需要Watchdog.md` |
| 11 | [Android 爛裝置想跑人臉辨識8 – YUV_420_888 還能怎樣加速：QFastCV](posts/section01_edge_ai/08_YUV加速QFastCV.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/08_YUV加速QFastCV.md` |
| 12 | [Android 爛裝置想跑人臉辨識 番外篇 – 從 50ms 到 3ms：ARM NEON、FastCV 與記憶體極限榨汁指南](posts/section01_edge_ai/09_1_NEON與記憶體極限榨汁番外篇.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/09_1_NEON與記憶體極限榨汁番外篇.md` |
| 13 | [Android 爛裝置想跑人臉辨識9 – 兄弟齊心，軟硬兼施：ARM Compute Library (ACL)](posts/section01_edge_ai/09_兄弟齊心ACL_SGEMM.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/09_兄弟齊心ACL_SGEMM.md` |
| 14 | [Android 爛裝置想跑人臉辨識10 – 雙鏡頭的時空修羅場：RGB 與 ToF](posts/section01_edge_ai/10_雙鏡頭時空同步.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/10_雙鏡頭時空同步.md` |
| 15 | [Android 爛裝置想跑人臉辨識11 – ToF RAW12 轉 3D 點雲](posts/section01_edge_ai/11_ToF_RAW12轉3D點雲.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/11_ToF_RAW12轉3D點雲.md` |
| 16 | [Android 爛裝置想跑人臉辨識12 – 說到底還是一台 IOT：WebSocket 篇](posts/section01_edge_ai/12_說到底還是一台IoT_WebSocket.md) | `Android 平台與系統` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/12_說到底還是一台IoT_WebSocket.md` |
| 17 | [Android 爛裝置想跑人臉辨識13 – 為了幾乎不會用到的功能需要大改特改的 WebRTC](posts/section01_edge_ai/13_為了極少用功能大改的WebRTC.md) | `Android 平台與系統` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/13_為了極少用功能大改的WebRTC.md` |
| 18 | [Android 爛裝置想跑人臉辨識14 – 門禁機當 BLE Peripheral 的現場踩坑記](posts/section01_edge_ai/14_門禁機當BLE_Peripheral現場踩坑記.md) | `邊緣 AI 與智慧裝置` | 爛裝置想跑 AI | `Current` | `posts/section01_edge_ai/14_門禁機當BLE_Peripheral現場踩坑記.md` |
| 19 | [Compose 三階段渲染管線](posts/section02_compose/01_Compose三階段渲染管線.md) | `現代 Android 開發` | Jetpack Compose 深入剖析 | `Current` | `posts/section02_compose/01_Compose三階段渲染管線.md` |
| 20 | [Jetpack Compose: 何時該用 derivedStateOf？](posts/section02_compose/02_何時該用derivedStateOf.md) | `現代 Android 開發` | Jetpack Compose 深入剖析 | `Current` | `posts/section02_compose/02_何時該用derivedStateOf.md` |
| 21 | [@Immutable @Stable 如果 immutableList 內包含 list ．這樣會造成 recompstion 嗎？](posts/section02_compose/03_Immutable與Stable重組防護.md) | `現代 Android 開發` | Jetpack Compose 深入剖析 | `Current` | `posts/section02_compose/03_Immutable與Stable重組防護.md` |
| 22 | [Modifier 效能 Draw 啥比較好：drawBehind , drawWithContent,drawWithCache](posts/section02_compose/04_Modifier繪製效能比較.md) | `現代 Android 開發` | Jetpack Compose 深入剖析 | `Current` | `posts/section02_compose/04_Modifier繪製效能比較.md` |
| 23 | [Thread 和 Coroutine 的理解](posts/section03_kotlin/01_Thread與Coroutine理解.md) | `現代 Android 開發` | Kotlin 執行原理與併發 | `Current` | `posts/section03_kotlin/01_Thread與Coroutine理解.md` |
| 24 | [Kotlin 如何預防 Launch 連發，搜尋任務](posts/section03_kotlin/02_Kotlin預防Launch連發防抖.md) | `現代 Android 開發` | Kotlin 執行原理與併發 | `Current` | `posts/section03_kotlin/02_Kotlin預防Launch連發防抖.md` |
| 25 | [inline, noinline, crossinline：存在即合理](posts/section03_kotlin/03_inline_noinline_crossinline.md) | `現代 Android 開發` | Kotlin 執行原理與併發 | `Current` | `posts/section03_kotlin/03_inline_noinline_crossinline.md` |
| 26 | [lateinit vs lazy](posts/section03_kotlin/04_lateinit_vs_lazy.md) | `現代 Android 開發` | Kotlin 執行原理與併發 | `Current` | `posts/section03_kotlin/04_lateinit_vs_lazy.md` |
| 27 | [Difference between constructors and init in kotlin](posts/section03_kotlin/05_constructors_and_init.md) | `現代 Android 開發` | Kotlin 執行原理與併發 | `Current` | `posts/section03_kotlin/05_constructors_and_init.md` |
| 28 | [kotlin init 前初始化屬性會發生啥事](posts/section03_kotlin/06_kotlin_init屬性陷阱.md) | `現代 Android 開發` | Kotlin 執行原理與併發 | `Current` | `posts/section03_kotlin/06_kotlin_init屬性陷阱.md` |
| 29 | [id(“org.kotlin.xxxx”) version “2.1.0” apply false  這不要你還特地寫上去啊？.](posts/section03_kotlin/07_gradle_kotlin_plugins.md) | `現代 Android 開發` | Kotlin 執行原理與併發 | `Current` | `posts/section03_kotlin/07_gradle_kotlin_plugins.md` |
| 30 | [viewModelScope vs rememberCoroutineScope 其實沒啥好 versus ，就是 SOC 職責分離](posts/section04_architecture/01_viewModelScope_vs_rememberCoroutineScope.md) | `現代 Android 開發` | Android 現代架構與生命週期 | `Current` | `posts/section04_architecture/01_viewModelScope_vs_rememberCoroutineScope.md` |
| 31 | [SupervisorJob 不就是 viewModelScope嗎？](posts/section04_architecture/02_SupervisorJob與viewModelScope.md) | `現代 Android 開發` | Android 現代架構與生命週期 | `Current` | `posts/section04_architecture/02_SupervisorJob與viewModelScope.md` |
| 32 | [Hilt 指南 組內分享](posts/section04_architecture/03_Hilt指南組內分享.md) | `現代 Android 開發` | Android 現代架構與生命週期 | `Current` | `posts/section04_architecture/03_Hilt指南組內分享.md` |
| 33 | [麻煩 ViewModelProvider.Factory 你了，因為系統不讓我直接 new 一個 ViewModel](posts/section04_architecture/04_ViewModelProviderFactory.md) | `現代 Android 開發` | Android 現代架構與生命週期 | `Current` | `posts/section04_architecture/04_ViewModelProviderFactory.md` |
| 34 | [fun GameScreen(gameViewModel: GameViewModel = GameViewModel()) 這樣寫有啥問題？](posts/section04_architecture/05_預設參數注入陷阱.md) | `現代 Android 開發` | Android 現代架構與生命週期 | `Current` | `posts/section04_architecture/05_預設參數注入陷阱.md` |
| 35 | [handle all repostiory operation  that out of Composable/ViewModel scope](posts/section04_architecture/06_RepositoryScope架構邊界.md) | `現代 Android 開發` | Android 現代架構與生命週期 | `Current` | `posts/section04_architecture/06_RepositoryScope架構邊界.md` |
| 36 | [CreationExtras.inventoryApplication() 這有點跳](posts/section04_architecture/07_CreationExtras注入.md) | `現代 Android 開發` | Android 現代架構與生命週期 | `Current` | `posts/section04_architecture/07_CreationExtras注入.md` |
| 37 | [說到底沒有吐槽以前自己寫的 code 代表你沒進步-ThreadPoolManager 的原罪](posts/section05_framework/01_ThreadPoolManager的原罪.md) | `Android 平台與系統` | - | `Current` | `posts/section05_framework/01_ThreadPoolManager的原罪.md` |
| 38 | [Naming threads and thread pools of ExecutorService](posts/section05_framework/02_Naming_threads_and_thread_pools.md) | `Android 平台與系統` | - | `Current` | `posts/section05_framework/02_Naming_threads_and_thread_pools.md` |
| 39 | [Android runtime resource overlay](posts/section05_framework/03_runtime_resource_overlay.md) | `Android 平台與系統` | - | `Current` | `posts/section05_framework/03_runtime_resource_overlay.md` |
| 40 | [Android 實作 data/control  分離 – createPackageContext()](posts/section05_framework/04_datacontrol分離_createPackageContext.md) | `Android 平台與系統` | - | `Current` | `posts/section05_framework/04_datacontrol分離_createPackageContext.md` |
| 41 | [Android home screen life cycle.](posts/section05_framework/05_home_screen_life_cycle.md) | `Android 平台與系統` | - | `Current` | `posts/section05_framework/05_home_screen_life_cycle.md` |
| 42 | [Java reflection in Android －fast scroll bar in list view / dynamic load apk.](posts/section05_framework/06_Java_reflection_dynamic_load.md) | `Android 平台與系統` | - | `Current` | `posts/section05_framework/06_Java_reflection_dynamic_load.md` |
| 43 | [Android Nougat introduction (一)](posts/section05_framework/07_Android_Nougat_introduction.md) | `Android 平台與系統` | - | `Contextual` | `posts/section05_framework/07_Android_Nougat_introduction.md` |
| 44 | [Analyzing UI Performance with Systrace](posts/section06_performance/01_Analyzing_UI_Performance_Systrace.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/01_Analyzing_UI_Performance_Systrace.md` |
| 45 | [Chrome dev tool for android –  steho](posts/section06_performance/02_Chrome_dev_tool_stetho.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/02_Chrome_dev_tool_stetho.md` |
| 46 | [Android Intents with Chrome](posts/section06_performance/03_Android_Intents_with_Chrome.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/03_Android_Intents_with_Chrome.md` |
| 47 | [Communicate with fragments](posts/section06_performance/04_Communicate_with_fragments.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/04_Communicate_with_fragments.md` |
| 48 | [React native Native android module – my god , many issue.](posts/section06_performance/05_Avoid_non-default_constructors_fragments.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/05_Avoid_non-default_constructors_fragments.md` |
| 49 | [Android issue – UI佈局被鍵盤擋住](posts/section06_performance/06_UI佈局被鍵盤擋住.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/06_UI佈局被鍵盤擋住.md` |
| 50 | [How to use RecyclerView inside NestedScrollView?](posts/section06_performance/07_RecyclerView_inside_NestedScrollView.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/07_RecyclerView_inside_NestedScrollView.md` |
| 51 | [Android AppIntro Proguard issue.](posts/section06_performance/08_AppIntro_Proguard_issue.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/08_AppIntro_Proguard_issue.md` |
| 52 | [Android Local build- use third parity lib.](posts/section06_performance/09_Local_build_third_parity_lib.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/09_Local_build_third_parity_lib.md` |
| 53 | [Android CTS Fail – cant remove external folder](posts/section06_performance/10_Android_CTS_Fail.md) | `Android 效能與穩定性` | Android 效能診斷與 UI 排障 | `Current` | `posts/section06_performance/10_Android_CTS_Fail.md` |
| 54 | [Android Camera analyze (一) – Camera 1 Architecture](posts/section07_camera_multimedia/01_Camera1_Architecture.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Contextual` | `posts/section07_camera_multimedia/01_Camera1_Architecture.md` |
| 55 | [Android Camera analyze (二) – Camera 2 Introduction.](posts/section07_camera_multimedia/02_Camera2_Introduction.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Current` | `posts/section07_camera_multimedia/02_Camera2_Introduction.md` |
| 56 | [Android Camera analyze (三) – How to create Camera 2.](posts/section07_camera_multimedia/03_How_to_create_Camera2.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Current` | `posts/section07_camera_multimedia/03_How_to_create_Camera2.md` |
| 57 | [FMRadio 開發筆記 （一）-概論](posts/section07_camera_multimedia/04_FMRadio_概論.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Contextual` | `posts/section07_camera_multimedia/04_FMRadio_概論.md` |
| 58 | [FMRadio 開發筆記 （二）- FMRadioMain](posts/section07_camera_multimedia/05_FMRadio_FMRadioMain.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Contextual` | `posts/section07_camera_multimedia/05_FMRadio_FMRadioMain.md` |
| 59 | [FMRadio 開發筆記 （三）- FMRadioService](posts/section07_camera_multimedia/06_FMRadio_FMRadioService.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Contextual` | `posts/section07_camera_multimedia/06_FMRadio_FMRadioService.md` |
| 60 | [FMRadio 開發筆記 （四）- AlterActivity](posts/section07_camera_multimedia/07_FMRadio_AlterActivity.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Contextual` | `posts/section07_camera_multimedia/07_FMRadio_AlterActivity.md` |
| 61 | [FMRadio 開發筆記 （五）- Other](posts/section07_camera_multimedia/08_FMRadio_Other.md) | `影像、相機與多媒體` | Android 原生相機與多媒體子系統 | `Contextual` | `posts/section07_camera_multimedia/08_FMRadio_Other.md` |
| 62 | [Funny Vote 開發雜記](posts/section08_projects_iot/01_Funny_Vote_開發雜記.md) | `架構實戰與技術觀點` | 獨立產品 Funny Vote 架構演進史 | `Contextual` | `posts/section08_projects_iot/01_Funny_Vote_開發雜記.md` |
| 63 | [Funny vote APP -MVP 更新雜記](posts/section08_projects_iot/02_Funny_vote_MVP更新雜記.md) | `架構實戰與技術觀點` | 獨立產品 Funny Vote 架構演進史 | `Contextual` | `posts/section08_projects_iot/02_Funny_vote_MVP更新雜記.md` |
| 64 | [Funny vote APP -MVP Kotlin 改版雜記](posts/section08_projects_iot/03_Funny_vote_Kotlin改版雜記.md) | `架構實戰與技術觀點` | 獨立產品 Funny Vote 架構演進史 | `Contextual` | `posts/section08_projects_iot/03_Funny_vote_Kotlin改版雜記.md` |
| 65 | [SLAM Note – NDT_TKU](posts/section08_projects_iot/04_SLAM_Note_NDT_TKU.md) | `邊緣 AI 與智慧裝置` | - | `Current` | `posts/section08_projects_iot/04_SLAM_Note_NDT_TKU.md` |
| 66 | [React Native 學習筆記](posts/section08_projects_iot/05_React_Native_學習筆記.md) | `工程工具與交付` | - | `Contextual` | `posts/section08_projects_iot/05_React_Native_學習筆記.md` |
| 67 | [React Native 學習筆記](posts/section08_projects_iot/06_React_native_Native_android_module.md) | `工程工具與交付` | - | `Contextual` | `posts/section08_projects_iot/06_React_native_Native_android_module.md` |
| 68 | [Retrofit 2 — How to Upload Files and Parameter list to Server](posts/section09_orm_tools/01_Retrofit2_Upload_Files.md) | `工程工具與交付` | - | `Contextual` | `posts/section09_orm_tools/01_Retrofit2_Upload_Files.md` |
| 69 | [Android volley customize request : gzip and gson.](posts/section09_orm_tools/02_Volley_gzip_gson.md) | `工程工具與交付` | - | `Legacy` | `posts/section09_orm_tools/02_Volley_gzip_gson.md` |
| 70 | [GreenDao 3 Introduction](posts/section09_orm_tools/03_GreenDao3_Introduction.md) | `工程工具與交付` | - | `Legacy` | `posts/section09_orm_tools/03_GreenDao3_Introduction.md` |
| 71 | [Android ORM Library- Green DAO（一） introduction](posts/section09_orm_tools/04_GreenDAO_introduction.md) | `工程工具與交付` | - | `Legacy` | `posts/section09_orm_tools/04_GreenDAO_introduction.md` |
| 72 | [Android ORM Library – Green DAO（二）Question: Why can’t set default value for entities](posts/section09_orm_tools/05_GreenDAO_default_value.md) | `工程工具與交付` | - | `Legacy` | `posts/section09_orm_tools/05_GreenDAO_default_value.md` |
| 73 | [Android ButterKnife introduction](posts/section09_orm_tools/06_ButterKnife_introduction.md) | `工程工具與交付` | - | `Legacy` | `posts/section09_orm_tools/06_ButterKnife_introduction.md` |
| 74 | [Facebook api on Android : Login FB AP on device , but account manager not show.](posts/section09_orm_tools/07_Facebook_api_Login.md) | `工程工具與交付` | - | `Contextual` | `posts/section09_orm_tools/07_Facebook_api_Login.md` |
| 75 | [Google Analytics Individual Qualification(GAIQ) GET !](posts/section09_orm_tools/08_GAIQ_GET.md) | `工程工具與交付` | - | `Contextual` | `posts/section09_orm_tools/08_GAIQ_GET.md` |
| 76 | [Android google analyze 使用教學(一) 初始化](posts/section09_orm_tools/09_Google_Analytics_初始化.md) | `工程工具與交付` | - | `Contextual` | `posts/section09_orm_tools/09_Google_Analytics_初始化.md` |
| 77 | [Android studio release jar by gradle](posts/section09_orm_tools/10_Release_jar_by_gradle.md) | `工程工具與交付` | - | `Contextual` | `posts/section09_orm_tools/10_Release_jar_by_gradle.md` |
| 78 | [Clean code: 無瑕的程式碼 – 書摘心得（一）](posts/section10_philosophy/01_Clean_code_書摘心得一.md) | `架構實戰與技術觀點` | - | `Current` | `posts/section10_philosophy/01_Clean_code_書摘心得一.md` |
| 79 | [Clean code: 無瑕的程式碼 – 書摘心得（二）](posts/section10_philosophy/02_Clean_code_書摘心得二.md) | `架構實戰與技術觀點` | - | `Current` | `posts/section10_philosophy/02_Clean_code_書摘心得二.md` |
| 80 | [The Clean coder: 無瑕的程式碼 番外篇 – 書摘心得](posts/section10_philosophy/03_The_Clean_coder_番外篇.md) | `架構實戰與技術觀點` | - | `Current` | `posts/section10_philosophy/03_The_Clean_coder_番外篇.md` |
| 81 | [約爾趣談軟體-筆記](posts/section10_philosophy/04_約爾趣談軟體筆記.md) | `架構實戰與技術觀點` | - | `Current` | `posts/section10_philosophy/04_約爾趣談軟體筆記.md` |
| 82 | [Android 好文整理](posts/section10_philosophy/05_Android好文整理.md) | `架構實戰與技術觀點` | - | `Current` | `posts/section10_philosophy/05_Android好文整理.md` |
