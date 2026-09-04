# [草稿] Android system design

> 建立日期：2025-08-27
> 文章編號：860
> 狀態：草稿 (Draft)

---

好的，這篇文章的完整翻譯如下：

---

Android 系統設計面試不僅僅是寫程式碼——更是**像軟體架構師一樣思考**。公司想知道你如何設計可擴展、高效且可維護的行動應用程式。

在這篇部落格中，我們將探討**常見的 Android 系統設計問題**，並用**真實世界的範例**來解釋。

### 什麼是 Android 系統設計？

這是規劃應用程式中不同元件（UI、API、資料庫、背景任務等）如何協同工作的過程。

它包括：

* **架構** (MVVM, Clean Architecture)

* **元件互動** (Activity, ViewModel, Repository)

* **資料流**

* **快取、效能、可擴展性**

* **離線優先策略**

* **安全性與可維護性**

---

### 你可能會遇到的問題類型

在 Android 系統設計面試中，問題可能來自兩大類：

1. 基於應用程式的設計

範例：

* 設計 WhatsApp

* 設計 Uber

* 設計 Instagram Stories

* 設計一個視訊通話 App

2. 基於函式庫的設計

範例：

* 設計一個圖片載入函式庫

* 設計一個網路層

* 設計一個分析 SDK

* 設計一個檔案下載器

---

### 如何應對面試？

這裡有一個聰明的逐步方法來處理任何系統設計問題：

#### 第 1 步：釐清需求

在開始之前，**提出問題**以獲得清晰的理解。例如：

* 聊天應該是 1 對 1 還是群組？

* 我們需要離線支援嗎？

* 是否需要推播通知？

#### 第 2 步：定義範疇

將範疇分為 3 個部分：

功能性需求

系統應該做什麼？

範例 (以 WhatsApp 為例)：

* 發送和接收訊息

* 顯示在線狀態

* 訊息確認機制

非功能性需求

系統應該表現得多好？

* 快速的回應時間

* 安全的通訊（端到端加密）

* 在低網速或離線時也能運作

超出範疇 (如果有的話)

清楚地定義你不需要建構什麼，以節省時間。

範例：

* 不做群組聊天

* 不做媒體分享

#### 第 3 步：識別客戶端元件

現在，拆解行動應用程式並列出關鍵元件。

範例：以一個像 WhatsApp 的 1 對 1 聊天 App 為例：

* **登入** (使用 OTP)

* **聯絡人畫面**

* **聊天畫面**

* **聊天服務** (發送和接收)

* **訊息資料庫** (本地儲存)

* **確認處理** (✓ 狀態)

* **在線/離線指示器**

* **推播通知支援**

* **個人資料/設定畫面**

#### 第 4 步：API 需求

列出使該系統運作所需的所有 API，以及將使用哪種協定：

HTTP APIs

用於一次性或偶爾的請求：

* 發送 OTP

* 驗證 OTP

* 獲取聯絡人

* 更新個人資料

WebSocket

用於即時通訊：

* 發送/接收訊息

* 確認勾號

* 在線狀態更新

#### 第 5 步：深入探討關鍵元件

你無法解釋所有事情，所以**根據面試官的興趣，專注於重要的部分**。

**範例 1：聊天架構**

* 使用 **MVVM 或 MVI** 來分離關注點。

* 使用**本地 Room 資料庫**來儲存訊息。

* 訊息透過 WebSocket 傳送到伺服器。

* 在 UI 中立即顯示（樂觀更新）。

**範例 2：確認處理**

* **單勾**：已發送

* **雙勾**：已送達接收方設備

* **藍勾**：接收方已讀

* 使用 WebSocket 的 **ack 事件**來更新資料庫/UI 中的狀態。

**範例 3：在線狀態**

* 使用 ping/pong WebSocket 機制。

* 用使用者的狀態更新後端。

* 向相關聯絡人廣播在線狀態。

---

### 面試官在尋找什麼

* 你是否理解**端到端的架構**？

* 你是否能**分離關注點**（UI vs 邏輯 vs 後端）？

* 你是否了解**安全性**（驗證、加密）？

* 你是否知道哪些部分在**客戶端 vs 伺服器**？

* 你是否能在時間限制下做出**權衡取捨**？

---

### 不該做的事

* 不要立即開始寫程式碼。

* 不要只專注於 UI。

* 不要對工具或框架過於僵化。

* 不要解釋每個小元件——專注於最重要的部分。

---

### 常見問題與回答

**1. 如何設計一個像 Inshorts 或 Google News 這樣可擴展的新聞 App？**

**目標**：使用者可以閱讀短新聞、支援離線、有推播通知。

**系統元件**：

* **UI 層**：Jetpack Compose 或 XML 搭配 RecyclerView 來列表新聞。

* **ViewModel**：使用 LiveData/StateFlow 儲存新聞資料。

* **Repository**：作為 ViewModel 和資料來源（API, DB）之間的橋樑。

* **API 層**：使用 Retrofit 獲取新聞文章。

* **本地資料庫**：使用 Room DB 快取新聞以供離線使用。

* **推播通知**：使用 Firebase Cloud Messaging (FCM)。

**資料流**：

1. App 啟動 → ViewModel 呼叫 Repository。

2. Repository 檢查本地快取 → 顯示快取資料。

3. 在背景 → 從 API 獲取 → 儲存到 DB → 更新 UI。

**其他設計考量**：

* 使用 **Paging 3** 來實現高效滾動。

* 使用 **WorkManager** 定期同步新聞。

* 使用 **Hilt/Dagger** 進行依賴注入。

---

**2. 如何在 Android App 中處理離線支援？**

使用**本地資料庫 + 網路備援策略**。

**範例**：在一個筆記 App 中：

* 將筆記儲存在 **Room DB**。

* 當網路可用時，使用 **WorkManager** 將變更同步到伺服器。

* 使用 **NetworkCallback** 或 **ConnectivityManager** 顯示離線橫幅。

**技術棧**：

* Room DB

* WorkManager

* Coroutines + Flow

* **Network Bound Resource 模式** (先檢查快取，然後再請求網路)

---

**3. 如何設計一個 Android 聊天 App (像 WhatsApp)？**

**核心功能**：

* 即時訊息

* 媒體上傳/下載

* 通知

* 聊天歷史

**系統設計**：

* **UI 層**：使用 Composables 或 RecyclerView 顯示聊天泡泡。

* **ViewModel**：持有聊天列表和訊息狀態。

* **Socket 層**：使用 **WebSockets** 或 **Firebase Realtime Database**。

* **媒體處理**：上傳到**雲端儲存**，發送前進行壓縮。

* **訊息資料庫**：使用 **Room** 在本地儲存訊息。

* **同步邏輯**：當使用者上線時，使用 WorkManager 同步錯過的訊息。

---

**4. 如何確保 Android App 的可擴展性？**

使用**模組化架構**和**乾淨的關注點分離**。

**最佳實踐**：

* 使用**模組化程式碼庫**（功能、核心、資料模組）。

* 遵循 **SOLID 原則**。

* 使用 **Repository 模式**。

* 使用**依賴注入** (Hilt)。

* 採用 **Coroutines + Flow** 處理異步工作。

---

**5. 如何以省電的方式處理背景任務？**

對於可延遲的任務使用 **WorkManager**，對於長時間運行的任務使用**前景服務 (Foreground Services)**。

**範例**：

* 上傳日誌 = **WorkManager**

* App 在背景時錄音 = **前景服務**

* 定期同步 = **PeriodicWorkRequest**

---

**6. 如何保護 Android App 中的使用者資料？**

* 對敏感的鍵值資料使用 **EncryptedSharedPreferences**。

* 對資料庫使用 **SQLCipher** 或 **Room Encryption**。

* 使用 **ProGuard/R8** 來混淆程式碼。

* 不要用明文的 SharedPreferences 儲存 token。

---

**7. 如何設計一個模組化的 Android 應用程式？**

模組化有助於將**程式碼庫分割**成**獨立的模組**，從而提高**可擴展性、可重用性、建置速度**和**團隊協作**。

範例：你正在建構一個購物 App。

你可以這樣進行模組化：

```
app/
|-- core/
|   |-- network/
|   |-- utils/
|-- feature_home/
|-- feature_cart/
|-- feature_checkout/
|-- feature_login/
```

**好處**：

* 每個團隊可以獨立工作。

* 更快的建置速度（Gradle 並行化）。

* 更好的測試（模組級別的單元測試）。

* 跨專案的程式碼重用。使用 Gradle 的 implementation project(":core:network") 來處理模組依賴。

---

**8. 如何在 Android 中實現 Clean Architecture？**

Clean Architecture 將**商業邏輯、UI** 和**資料**分開，以提高可維護性。

**分層**：

1. **表現層 (UI)**：

  * Activities, Fragments, Jetpack Compose

  * 與 ViewModel 對話

2. **領域層 (商業邏輯)**：

  * UseCases

  * Repositories 的介面

  * 平台無關

3. **資料層**：

  * 實現 Repositories

  * 呼叫 API 或 DB

**範例**：對於一個**天氣 App**，定義：

* `GetWeatherUseCase(city): Weather`

* Repository: `WeatherRepositoryImpl(api, dao)`

* ViewModel 呼叫 use case。

這樣一來，**UI 不知道 API/DB 的邏輯**——非常適合測試和擴展。

---

**9. 如何實現即時功能（如即時比賽更新或聊天）？**

使用 **WebSockets、Firebase Realtime Database** 或 **SignalR** 進行即時更新。

**架構**：

* **UI 層**：顯示即時更新。

* **ViewModel**：使用 StateFlow 或 LiveData 持有即時狀態。

* **WebSocket 客戶端**：監聽事件並將資料推送到 ViewModel。

* 可選地將訊息/事件快取在 Room 中以供離線存取。

---

**10. 如何在 Android App 中處理大型媒體檔案？**

處理大型檔案（影片、圖片）需要：

* 高效的壓縮

* 串流或分塊上傳

* 適當的記憶體處理

**策略**：

* 使用像 **Glide 或 Coil** 這樣的函式庫來載入圖片。

* 使用 **WorkManager** + **Retrofit with Multipart** 進行上傳。

* 使用 `BitmapFactory.Options.inSampleSize` 進行壓縮。

* 對於影片：使用 **ExoPlayer** 進行串流播放。

---

**11. 設計一個影片串流 App (像 YouTube)**

**核心功能**：

* 影片摘要

* 播放（從上次位置繼續）

* 離線下載

* 評論

**關鍵元件**：

* **UI**：在 Compose/Fragment 中使用 **ExoPlayer**。

* **ViewModel**：管理播放器狀態和影片元數據。

* **Repository**：獲取影片 URL 和資訊。

* **影片資料庫**：Room 儲存觀看歷史、離線影片。

* **播放器配置**：

  * 使用 **ExoPlayer**。

  * 在本地資料庫中儲存播放進度。

  * 在 RecyclerView 中預載縮圖。

---

**12. 在系統設計中使用 ViewModel 的最佳實踐是什麼？**

* 保持 ViewModel 專注於 UI（不要直接放入 repository 邏輯）。

* 使用 **SavedStateHandle** 來應對程序終止 (process death)。

* 透過 **StateFlow/Livedata** 公開資料，而不是可變狀態。

* 使用 **viewModelScope** 取消協程任務。

---

**13. 如何優化 RecyclerView 的效能？**

* 使用 **DiffUtil** 和 **ListAdapter**。

* 使用 ViewHolder 模式以避免重複的 inflation。

* 在可能的情況下使用 `setHasStableIds(true)`。

* 透過 Glide 的快取重用圖片資源。

* 除非必要，否則避免巢狀 RecyclerView。

---

**14. 解釋你將如何快取 API 回應以供離線使用。**

使用 **Network Bound Resource** 模式：

Kotlin

```
fun getData(): Flow<Resource<Data>> = flow {
    emit(Resource.Loading())
    val local = dao.getData()
    emit(Resource.Success(local))

    val remote = api.getData()
    if (remote.isSuccessful) {
        dao.save(remote.data)
        emit(Resource.Success(remote.data))
    }
}
```

像 **Room + Flow** 這樣的函式庫可以幫助你**立即發送快取資料**，並在網路資料到達時更新。

---

**15. 如何管理 App 的配置（如 base URL、功能開關、密鑰）？**

使用：

* **BuildConfig** 用於建置時的常數。

* **Remote Config (Firebase)** 用於功能標記。

* **.properties 檔案** 用於本地開發配置。

* **EncryptedSharedPreferences** 用於密鑰。

使用 flavors：

Groovy

```
productFlavors {
    dev {
        buildConfigField "String", "BASE_URL", '"https://dev.example.com"'
    }
    prod {
        buildConfigField "String", "BASE_URL", '"https://api.example.com"'
    }
}
```

---

**16. 如何設計一個支援多語言 (i18n) 的 App？**

使用 Android 的**字串資源**系統：

* 在 `res/values/strings.xml` 中定義字串。

* 在 `res/values-hi/strings.xml`、`values-fr` 等檔案中添加翻譯。

* 使用 `LocaleHelper` 類來強制切換語言。

要動態更改語言：

Kotlin

```
fun updateLanguage(context: Context, langCode: String): Context {
    val locale = Locale(langCode)
    Locale.setDefault(locale)
    val config = Configuration()
    config.setLocale(locale)
    return context.createConfigurationContext(config)
}
```

---

**17. 管理 App 更新和版本控制的最佳方式是什麼？**

* 使用 **Play Core Library** 進行應用內更新。

* 保持**語意化版本控制** (MAJOR.MINOR.PATCH)。

* 使用 **BuildConfig.VERSION_CODE** 和 **VERSION_NAME**。

* 維護一個更新日誌 (changelog)。

* 對於強制更新，從 API 檢查最低支援版本，如果過時則將使用者重導向到 Play Store。

---

**18. 設計一個美食外送 App (像 Zomato/Swiggy)**

**功能**：

* 餐廳列表

* 購物車和結帳

* 即時訂單追蹤

* 評分和評論

**系統設計**：

* **模組**：

  * `feature_home`

  * `feature_cart`

  * `feature_order_tracking`

  * `core/network`, `core/db`, `core/di`

* **流程**：

  * 使用者瀏覽餐廳 → 透過 Retrofit 進行 API 呼叫。

  * 將食物加入購物車 → ViewModel 儲存狀態。

  * 結帳 → API + 付款。

  * 訂單追蹤 → WebSocket 或每幾秒輪詢一次 API。

* **設計亮點**：

  * 使用 **Room DB** 實現離線購物車。

  * **Coroutines + StateFlow** 用於響應式狀態更新。

  * **Firebase Remote Config** 動態切換餐廳可用性。

  * **Foreground Service + LiveData** 用於追蹤外送員位置。

---

**19. 設計一個 UPI 支付 App (像 Paytm 或 Google Pay)**

**功能**：

* UPI ID 管理

* 支付和餘額查詢

* 交易歷史

* 安全與詐欺偵測

**關鍵設計目標**：

* **安全儲存** token、金鑰。

* 穩健的離線處理。

* 失敗時重試（網路問題）。

**系統設計**：

* 將敏感資料儲存在 **EncryptedSharedPreferences** 中。

* 使用 **WorkManager 搭配指數退避**進行 API 呼叫重試。

* 使用 **ViewModel + StateFlow** 實現 UI 響應性。

* 使用 **BiometricPrompt API** 保護 UI。

---

**20. 設計一個檔案下載器函式庫**

面試官意圖：

他們想檢查你對網路處理、大檔案支援、暫停/恢復和儲存的理解。

**必備功能**：

* 支援**背景下載**（使用 WorkManager/Service）。

* 使用 **OkHttp 或 Retrofit** 進行網路請求。

* 使用 HTTP **`Range`** 標頭處理**下載恢復**。

* **儲存**：使用串流將檔案儲存到內部/外部儲存空間。

* 透過 **LiveData 或 Flow** 更新進度。

* 優雅地處理**網路中斷**。

---

**21. 設計一個分析 SDK**

面試官意圖：

他們想看看你如何設計可重用、模組化的函式庫，並在多個 App 中使用。

**關鍵點**：

* 追蹤使用者事件（例如，登入、購買）。

* 支援**批次處理**和**離線同步**。

* 將事件排入佇列並在背景中發送。

* 使用**工作執行緒 (worker/thread)** 將資料發送到伺服器。

* 確保**執行緒安全**。

* 允許輕鬆初始化和配置（例如，API 金鑰）。

---

**22. Repository 在 Clean Architecture 中的角色是什麼？**

Repository 是**單一資料來源 (single source of truth)**，它決定：

* 是從**本地資料庫**還是**網路**獲取資料。

* 將 **DTOs 轉換為領域模型**。

* 保持程式碼**乾淨且可測試**。

---

**23. 如何在 Android App 中設計一個防崩潰和可重啟的安全架構？**

你的 App 應該能夠優雅地處理**程序終止**或**意外關閉**。

**策略**：

* 在 ViewModel 中使用 **SavedStateHandle** 來儲存 UI 狀態。

* 即時將資料儲存在 **SharedPreferences/Room DB** 中。

* 在 Compose/Activity 中使用 `onSaveInstanceState()`。

**範例**：

Kotlin

```
val state = savedStateHandle.getLiveData("cart_items")
```

將此與 ViewModel 結合，在 App 重啟後恢復購物車。

---

**24. 如何在一個舊的單體 (monolith) Android App 中實施應用模組化？**

採取**漸進式**方法：

1. 識別可重用的功能（認證、產品、購物車）。

2. 建立 Gradle 模組 (`feature_auth`, `core_ui` 等）。

3. 明智地使用 `api` vs `implementation`。

4. 逐塊遷移程式碼。

5. 確保所有模組都有自己的測試。

---

**25. 如何建構一個能擴展到 1000 萬以上使用者的 App？**

你必須分層思考：

* 高效的**網路**（分頁、壓縮、重試）。

* 使用全域例外處理器實現**無崩潰的程式碼**。

* **本地快取**以減少 API 呼叫。

* **模組化結構**以提高可維護性。

* **A/B 測試**功能以避免大規模更新失敗。

* **Proguard/R8 優化**。

**工具**：

* Firebase Crashlytics

* App Performance SDK

* 分析 + A/B 測試

---

**26. 設計一個位置分享 App (像 Life360)**

面試官意圖：

測試你對位置 API、背景任務和功耗優化的理解。

**考量因素**：

* 使用 **FusedLocationProviderClient** 進行高效追蹤。

* 處理**權限和電池優化**。

* 使用**前景服務**進行即時追蹤。

* 使用 HTTP 或 WebSocket 將位置同步到後端。

* 使用本地資料庫在離線時快取資料。

* 通知和位置分享 UI。

💬 **追問**：如果位置更新在背景中受到限制怎麼辦？

> 

使用帶有約束的 WorkManager 或帶有通知的前景服務。

---

**27. 設計一個快取函式庫**

面試官意圖：

他們想檢查你如何優化資料存取並減少不必要的 API 呼叫。

**需要討論的內容**：

* **多級快取**：

  * **記憶體中** (最快)

  * **磁碟上** (持久化)

  * **網路** (備援)

* 定義**快取過期策略**。

* 處理**過期資料 vs 新鮮資料**的邏輯。

* **執行緒安全**的讀/寫操作。

* 可配置的快取大小和淘汰規則。

---
