# Summary

* [前言：十載技術筆記與破局心法](README.md)

## 一、 邊緣 AI 與人臉辨識硬核實戰

* [00 篇：到底有多爛？算力深淵與硬體現狀評估](posts/section01_edge_ai/00_到底有多爛.md)
* [01 篇：評估第一：選型考量與硬體瓶頸分析](posts/section01_edge_ai/01_評估第一.md)
* [02 篇：記憶體優化：無用系統服務徹底剔除](posts/section01_edge_ai/02_無用系統服務out.md)
* [03 篇：系統優化：ZRAM, SWAP, KSM 擴張記憶體最大值](posts/section01_edge_ai/03_ZRAM_SWAP_KSM.md)
* [04 篇：記憶體控場師：LMKD 策略與防 OOM 實戰](posts/section01_edge_ai/04_記憶體控場師LMKD.md)
* [05 篇：GC 是唯一會關心你 OOM 的好人](posts/section01_edge_ai/05_GC是唯一關心你OOM的好人.md)
* [06 篇：三路高頻影像流，三倍蓋亞，趕快 GC](posts/section01_edge_ai/06_三路高頻影像流零拷貝.md)
* [07 篇：守護進程：物理世界需要 watchdog](posts/section01_edge_ai/07_物理世界需要Watchdog.md)
* [補充篇：GC Thrashing 思路補充](posts/section01_edge_ai/07_1_GC_Thrashing思路補充.md)
* [08 篇：YUV 420 888 還能怎樣加速：QFastCV 實戰](posts/section01_edge_ai/08_YUV加速QFastCV.md)
* [09 篇：兄弟齊心，軟硬兼施：ARM Compute Lib (ACL)](posts/section01_edge_ai/09_兄弟齊心ACL_SGEMM.md)
* [番外篇：從 50ms 到 3ms：ARM NEON、FastCV 與記憶體極限榨汁指南](posts/section01_edge_ai/09_1_NEON與記憶體極限榨汁番外篇.md)
* [10 篇：雙鏡頭的時空修羅場：RGB 與 ToF 同步](posts/section01_edge_ai/10_雙鏡頭時空同步.md)
* [11 篇：ToF RAW12 轉 3D 點雲](posts/section01_edge_ai/11_ToF_RAW12轉3D點雲.md)
* [12 篇：說到底還是一台 IOT：WebSocket 篇](posts/section01_edge_ai/12_說到底還是一台IoT_WebSocket.md)
* [13 篇：為了幾乎不會用到的功能需要大改特改的 WebRTC 視訊對講](posts/section01_edge_ai/13_為了極少用功能大改的WebRTC.md)
* [14 篇：門禁機當 BLE Peripheral 的現場踩坑記](posts/section01_edge_ai/14_門禁機當BLE_Peripheral現場踩坑記.md)
* [系列導讀：全系列架構演進與防坑總對照](posts/section01_edge_ai/00_全系列架構對照導覽.md)

## 二、 Jetpack Compose 深度原理與效能調校

* [渲染管線：Compose 三階段渲染管線剖析](posts/section02_compose/01_Compose三階段渲染管線.md)
* [狀態衍生：Jetpack Compose 何時該用 derivedStateOf？](posts/section02_compose/02_何時該用derivedStateOf.md)
* [重組防護：@Immutable @Stable 如果包含 List 會造成 recomposition 嗎？](posts/section02_compose/03_Immutable與Stable重組防護.md)
* [繪製優化：Modifier 效能 Draw 啥比較好：drawBehind vs drawWithCache](posts/section02_compose/04_Modifier繪製效能比較.md)

## 三、 Kotlin 語言深水區與非同步併發控制

* [併發概念：Thread 和 Coroutine 的理解與切換成本](posts/section03_kotlin/01_Thread與Coroutine理解.md)
* [時序防抖：Kotlin 如何預防 Launch 連發，搜尋任務防抖防競態](posts/section03_kotlin/02_Kotlin預防Launch連發防抖.md)
* [內聯機制：inline, noinline, crossinline：存在即合理](posts/section03_kotlin/03_inline_noinline_crossinline.md)
* [延遲初始化：lateinit vs lazy 底層實現差異](posts/section03_kotlin/04_lateinit_vs_lazy.md)
* [建構順序：Difference between constructors and init in kotlin](posts/section03_kotlin/05_constructors_and_init.md)
* [屬性陷阱：kotlin init 前初始化屬性會發生啥事](posts/section03_kotlin/06_kotlin_init屬性陷阱.md)
* [Gradle 依賴：id('org.kotlin.xxxx') version '2.1.0' apply false 這不要你還特地寫？](posts/section03_kotlin/07_gradle_kotlin_plugins.md)

## 四、 現代 Android 架構設計與依賴注入

* [範疇劃分：viewModelScope vs rememberCoroutineScope 其實是 SOC 職責分離](posts/section04_architecture/01_viewModelScope_vs_rememberCoroutineScope.md)
* [取消機制：SupervisorJob 不就是 viewModelScope 嗎？](posts/section04_architecture/02_SupervisorJob與viewModelScope.md)
* [依賴注入：Hilt 指南 組內分享](posts/section04_architecture/03_Hilt指南組內分享.md)
* [實例工廠：麻煩 ViewModelProvider.Factory 你了，因為系統不讓我直接 new ViewModel](posts/section04_architecture/04_ViewModelProviderFactory.md)
* [預設參數陷阱：fun GameScreen(viewModel = ViewModel()) 這樣寫有啥問題？](posts/section04_architecture/05_預設參數注入陷阱.md)
* [架構邊界：handle all repository operation that out of ComposableViewModel scope](posts/section04_architecture/06_RepositoryScope架構邊界.md)
* [實例注入：CreationExtras.inventoryApplication() 這有點跳](posts/section04_architecture/07_CreationExtras注入.md)

## 五、 Android 系統核心原理與 Framework 原始碼探秘

* [執行緒池原罪：說到底沒有吐槽以前自己寫的 code 代表你沒進步 - ThreadPoolManager 的原罪](posts/section05_framework/01_ThreadPoolManager的原罪.md)
* [執行緒命名：Naming threads and thread pools of ExecutorService](posts/section05_framework/02_Naming_threads_and_thread_pools.md)
* [運行時替換：Android runtime resource overlay (RRO)](posts/section05_framework/03_runtime_resource_overlay.md)
* [沙盒隔離：Android 實作 datacontrol 分離 – createPackageContext()](posts/section05_framework/04_datacontrol分離_createPackageContext.md)
* [系統啟動：Android home screen life cycle](posts/section05_framework/05_home_screen_life_cycle.md)
* [反射黑魔法：Java reflection in Android － fast scroll bar dynamic load apk](posts/section05_framework/06_Java_reflection_dynamic_load.md)
* [系統演進：Android Nougat introduction (一)](posts/section05_framework/07_Android_Nougat_introduction.md)

## 六、 效能診斷、除錯工具與 UI 渲染排障

* [效能檢測：Analyzing UI Performance with Systrace](posts/section06_performance/01_Analyzing_UI_Performance_Systrace.md)
* [診斷工具：Chrome dev tool for android – stetho](posts/section06_performance/02_Chrome_dev_tool_stetho.md)
* [跨進程意圖：Android Intents with Chrome](posts/section06_performance/03_Android_Intents_with_Chrome.md)
* [元件通訊：Communicate with fragments](posts/section06_performance/04_Communicate_with_fragments.md)
* [構造地雷：Android issue – Avoid non-default constructors in fragments](posts/section06_performance/05_Avoid_non-default_constructors_fragments.md)
* [佈局衝突：Android issue – UI 佈局被鍵盤擋住](posts/section06_performance/06_UI佈局被鍵盤擋住.md)
* [滾動巢狀：How to use RecyclerView inside NestedScrollView](posts/section06_performance/07_RecyclerView_inside_NestedScrollView.md)
* [代碼混淆：Android AppIntro Proguard issue](posts/section06_performance/08_AppIntro_Proguard_issue.md)
* [本地相依：Android Local build - use third parity lib](posts/section06_performance/09_Local_build_third_parity_lib.md)
* [相容性測試：Android CTS Fail – cant remove external folder](posts/section06_performance/10_Android_CTS_Fail.md)

## 七、 相機 HAL 與多媒體驅動框架

* [Android Camera analyze (一) – Camera 1 Architecture](posts/section07_camera_multimedia/01_Camera1_Architecture.md)
* [Android Camera analyze (二) – Camera 2 Introduction](posts/section07_camera_multimedia/02_Camera2_Introduction.md)
* [Android Camera analyze (三) – How to create Camera 2](posts/section07_camera_multimedia/03_How_to_create_Camera2.md)
* [FMRadio 開發筆記（一）- 概論](posts/section07_camera_multimedia/04_FMRadio_概論.md)
* [FMRadio 開發筆記（二）- FMRadioMain](posts/section07_camera_multimedia/05_FMRadio_FMRadioMain.md)
* [FMRadio 開發筆記（三）- FMRadioService](posts/section07_camera_multimedia/06_FMRadio_FMRadioService.md)
* [FMRadio 開發筆記（四）- AlterActivity](posts/section07_camera_multimedia/07_FMRadio_AlterActivity.md)
* [FMRadio 開發筆記（五）- Other](posts/section07_camera_multimedia/08_FMRadio_Other.md)

## 八、 專案實戰、機器人定位與跨平台探索

* [獨立 App 開發：Funny Vote 開發雜記（2021）](posts/section08_projects_iot/01_Funny_Vote_開發雜記.md)
* [架構迭代：Funny vote APP - MVP 更新雜記（2021）](posts/section08_projects_iot/02_Funny_vote_MVP更新雜記.md)
* [語言遷移：Funny vote APP - MVP Kotlin 改版雜記（2022）](posts/section08_projects_iot/03_Funny_vote_Kotlin改版雜記.md)
* [機器人定位：SLAM Note – NDT TKU（2024）](posts/section08_projects_iot/04_SLAM_Note_NDT_TKU.md)
* [跨平台初探：React Native 學習筆記（2016）](posts/section08_projects_iot/05_React_Native_學習筆記.md)
* [原生模組互通：React native Native android module – many issue（2016）](posts/section08_projects_iot/06_React_native_Native_android_module.md)

## 九、 早期經典套件、資料庫 ORM 與生態拓荒史

* [網路庫演進：Retrofit 2 — How to Upload Files and Parameter list](posts/section09_orm_tools/01_Retrofit2_Upload_Files.md)
* [網路庫演進：Android volley customize request gzip and gson](posts/section09_orm_tools/02_Volley_gzip_gson.md)
* [ORM 資料庫：GreenDao 3 Introduction](posts/section09_orm_tools/03_GreenDao3_Introduction.md)
* [ORM 資料庫：Android ORM Library - Green DAO（一） introduction](posts/section09_orm_tools/04_GreenDAO_introduction.md)
* [ORM 資料庫：Green DAO（二） Question Why can't set default value](posts/section09_orm_tools/05_GreenDAO_default_value.md)
* [視圖綁定：Android ButterKnife introduction](posts/section09_orm_tools/06_ButterKnife_introduction.md)
* [社群整合：Facebook api on Android Login FB AP on device](posts/section09_orm_tools/07_Facebook_api_Login.md)
* [數據認證：Google Analytics Individual Qualification (GAIQ) GET !](posts/section09_orm_tools/08_GAIQ_GET.md)
* [數據追蹤：Android google analyze 使用教學(一) 初始化](posts/section09_orm_tools/09_Google_Analytics_初始化.md)
* [編譯打包：Android studio release jar by gradle](posts/section09_orm_tools/10_Release_jar_by_gradle.md)

## 十、 軟體工程思維、大師書摘與漫談

* [代碼整潔之道：Clean Code 無瑕的程式碼 – 書摘心得（一）](posts/section10_philosophy/01_Clean_code_書摘心得一.md)
* [代碼整潔之道：Clean Code 無瑕的程式碼 – 書摘心得（二）](posts/section10_philosophy/02_Clean_code_書摘心得二.md)
* [職業素養思維：The Clean Coder 無瑕的程式碼 番外篇 – 書摘心得](posts/section10_philosophy/03_The_Clean_coder_番外篇.md)
* [工程哲學漫談：約爾趣談軟體 - 筆記](posts/section10_philosophy/04_約爾趣談軟體筆記.md)
* [知識萃取：Android 好文整理（2015）](posts/section10_philosophy/05_Android好文整理.md)

## 附錄：待整理深入技術草稿（12 篇）

* [[草稿] Jetpack Compose 的性能優化建議](posts/drafts/01_Compose性能優化建議.md)
* [[草稿] Flow vs Channel & SharedFlow vs StateFlow](posts/drafts/02_Flow_vs_Channel.md)
* [[草稿] LaunchedEffect 和 rememberUpdatedState and snapshotFlow](posts/drafts/03_LaunchedEffect_rememberUpdatedState.md)
* [[草稿] produceState 橋接非 Flow 的「回呼」或「監聽器」模式](posts/drafts/04_produceState_橋接非Flow.md)
* [[草稿] 狀態容器、連帶效果 API 深度解析](posts/drafts/05_狀態容器連帶效果API.md)
* [[草稿] Cache the result of the network request](posts/drafts/06_Cache_network_request.md)
* [[草稿] ActivityRetainedComponent vs ActivityComponent](posts/drafts/07_ActivityRetainedComponent.md)
* [[草稿] @EntryPoint 機制手動提取 Hilt 依賴](posts/drafts/08_EntryPoint_手動提取Hilt.md)
* [[草稿] Compose strong skipping mode](posts/drafts/09_strong_skipping_mode.md)
* [[草稿] Android System Design 實戰設計](posts/drafts/10_Android_system_design.md)
* [[草稿] Mobile System Design 實踐指南](posts/drafts/11_Mobile_system_design.md)
* [[草稿] 解決 Jetpack Compose 中效能問題的實用做法](posts/drafts/12_解決Compose效能問題做法.md)
