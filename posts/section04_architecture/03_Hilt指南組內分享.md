# Hilt 指南 組內分享

> 原文網址：https://boochlin.com/?p=802
> 發布日期：2025-07-26
> 文章編號：802

---

市場上已經很多依賴注入套件(dagger , koin)的選擇 ，還有很多文章介紹了，我會選擇 hilt 很單純，就是對於各類 mvvm 的物件，有魔法般的支援，真的少非常多代碼，選擇hilt 並非絕對正確，我的文章一直提到一件事 就是 根據情境，才是正確的

---

依賴注入 (Dependency Injection) 的手動實作

我們每天都在 new 物件，然後像快遞員一樣，把它們從 Activity 傳到 ViewModel，再傳到 Repository，最後送到 DataSource 手中。這是在寫程式，還是在做物流？

### 為何需要 Hilt？

依賴注入是個好東西，它能讓我們的程式碼解耦、易於測試。但手動實現它的過程，卻充滿了大量容易出錯的建構子傳遞和工廠模式，這完全違背了我們追求「簡潔」的初衷。

Hilt 的誕生，就是為了解決這個核心矛盾。它大聲地說：「**專心在你的業務邏輯上，這些無聊的體力活，我包了！**」

### Hilt 的三大核心武器：入門的基礎

要讓 Hilt 為你工作，你首先需要掌握三樣最基本的武器。

#### 1. @Inject 與 @AndroidEntryPoint

這是 Hilt 的基礎。@Inject 用來告訴 Hilt「這個東西你可以幫我準備」，而 @AndroidEntryPoint 則是告訴 Hilt「這個地方需要你來服務」。

Kotlin

```
// UserRepository.kt - 一個普通的類別
// 在建構子上標記，Hilt 就知道如何建立它
class UserRepository @Inject constructor(
    private val apiService: ApiService
) { /*...*/ }

// MainActivity.kt - 一個 Android 元件
@AndroidEntryPoint // 告訴 Hilt，這個 Activity 需要注入
class MainActivity : AppCompatActivity() {

    @Inject // 提出需求：「我需要一個 UserRepository」
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        // 在 super.onCreate() 之後，userRepository 就已經被 Hilt 準備好了
    }
}

// 別忘了在 Application 中啟用 Hilt
@HiltAndroidApp
class MyApplication : Application() {}
```

#### 2. @Provides：客製化生產說明書

當 Hilt 遇到它無法直接 new 的物件（例如來自 Retrofit、Room 等第三方庫的物件），你就需要提供一份詳細的「DIY 組裝指南」。

Kotlin

```
// NetworkModule.kt - 提供網路相關物件的說明書
@Module
@InstallIn(SingletonComponent::class) // 這本說明書的內容適用於整個 App
object NetworkModule {

    @Provides // 這是一份指南
    @Singleton // 產出的物件在整個 App 中只會有一個
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .client(okHttpClient) // Hilt 會自動把 OkHttpClient 遞過來
            .build()
    }

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder().build()
    }
}
```

#### 3. @Binds：高效率職位指派令

當你只是想告訴 Hilt「某個介面（職位）由哪個具體類別（員工）來實現」時，使用 @Binds。它不做實際的建立工作，只做「綁定」，因此效能更好。

Kotlin

```
// UserRepository.kt - 介面 (職位)
interface UserRepository { /*...*/ }

// UserRepositoryImpl.kt - 實作 (員工)
class UserRepositoryImpl @Inject constructor(...) : UserRepository { /*...*/ }

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule { // 使用 @Binds 的模組必須是 abstract

    @Binds
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}
```

### **InstallIn and @Qualifier**

物件的生命週期管理是避免記憶體洩漏的關鍵。Hilt 透過作用域 (Scope) 和限定符 (Qualifier) 提供了精準的控制。

* **@InstallIn 與作用域 (@…Scoped)**: @InstallIn 決定了依賴被安裝在哪個 Hilt 元件中（SingletonComponent、ActivityComponent 等），決定了其生命週期的上限。而作用域註解（@Singleton、@ActivityScoped、@ViewModelScoped）

```
// 此物件的生命週期與單一 ViewModel 實例綁定
@Module
@InstallIn(ViewModelComponent::class)
object SessionModule {
    @Provides
    @ViewModelScoped
    fun provideSessionCache(): SessionCache { /*...*/ }
}
```

* **@Qualifier (如 @Named)**: 當你需要提供多個相同型別的物件時（例如 API URL 和 API Key 都是 String），使用限定符為它們貼上標籤，以消除歧義。

```
@Provides
@Named("BaseUrl")
fun provideBaseUrl(): String = "https://api.example.com/"

@Provides
@Named("ApiKey")
fun provideApiKey(): String = "your_secret_key"

// 注入時也使用同樣的標籤
class ApiClient @Inject constructor(
    @Named("BaseUrl") private val baseUrl: String,
    @Named("ApiKey") private val apiKey: String
)
```

### Jetpack 的高度整合

Hilt 的真正威力在於它與 Jetpack 的深度整合，專治各種「Android 特有種」的疑難雜症。

* **@HiltViewModel**: 讓你徹底告別手寫 ViewModelProvider.Factory 的痛苦。只需一個註解，Hilt 就會自動處理 ViewModel 的建立與依賴注入，甚至免費贈送 SavedStateHandle。

```
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: UserRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel()

// 在 Fragment/Activity 中正常獲取即可
private val viewModel: MyViewModel by viewModels()
```

* **@HiltWorker**: 透過 @Assisted。 Inject，讓由系統管理的 WorkManager 背景任務也能輕鬆獲取 Hilt 提供的依賴。

### Hilt 多模組依賴注入

在大型 Android 專案中，多模組架構是提升建構速度和程式碼分離度的關鍵。然而，如何有效管理跨模組的依賴注入，成為了核心挑戰。本文將以程式碼為核心，講解 Hilt 在多模組環境下的標準實踐。

#### **核心原則：共享頂層元件**

Hilt 的多模組策略，是利用一個所有模組共享的頂層元件 SingletonComponent 作為依賴的橋樑。其運作模式如下：

1. **底層模組 (library, core)**：負責「提供」共享的依賴物件 (如 OkHttpClient, Room Database)，並將其註冊到 SingletonComponent 中。

2. **上層模組 (feature, app)**：從 SingletonComponent 中「獲取」並注入這些共享的依賴，無需關心其具體實作來源。

---

#### **實作步驟**

假設專案結構為 :app -> :feature_login -> :core_network。

##### **1. 在 :core_network 模組中提供依賴**

首先，在最底層的 :core_network 模組中建立 Hilt Module，來提供一個全域共享的 OkHttpClient 實例。

**core_network/src/main/java/com/example/core/network/NetworkModule.kt**

Kotlin

```
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import okhttp3.OkHttpClient
import javax.inject.Singleton

@Module
// 關鍵：將此模組安裝到 SingletonComponent，使其成為全域可用的元件。
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    // 關鍵：確保在 App 生命週期中，此物件只有一個實例。
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        // ... 進行 OkHttpClient 的相關設定
        return OkHttpClient.Builder().build()
    }
}
```

##### **2. 在 :feature_login 模組中使用依賴**

接著，在 :feature_login 模組中，我們需要設定 Gradle 依賴，並直接注入 OkHttpClient。

**feature_login/build.gradle.kts**

Kotlin

```
dependencies {
    // 依賴底層的 :core_network 模組
    implementation(project(":core_network"))

    // Hilt 相關依賴
    implementation("com.google.dagger:hilt-android:2.51.1")
    kapt("com.google.dagger:hilt-compiler:2.51.1")
}
```

**feature_login/src/main/java/com/example/feature/login/LoginActivity.kt**

Kotlin

```
import androidx.appcompat.app.AppCompatActivity
import com.example.core.network.OkHttpClient // 注意：這裡是示意，通常會注入 Repository
import dagger.hilt.android.AndroidEntryPoint
import javax.inject.Inject

@AndroidEntryPoint
class LoginActivity : AppCompatActivity() {

    // 關鍵：直接使用 @Inject 請求依賴。
    // Hilt 會自動查找 SingletonComponent 中已註冊的提供者。
    @Inject
    lateinit var okHttpClient: OkHttpClient

    // ...
}
```

#### **api vs implementation**

在 Gradle 中，依賴的傳遞性至關重要。

* **implementation(project(“:…”))**: 依賴不會向上傳遞。如果 :app 依賴 :feature，而 :feature implementation :core，則 :app 看不見 :core 的內容。

* **api(project(“:…”))**: 依賴會向上傳遞。在上述情況下，若 :feature api :core，則 :app 也能存取 :core 的內容。

**Hilt 規則**：如果一個模組 A 提供了希望被模組 C (C -> B -> A) 注入的 Hilt 綁定，那麼在 B 的 build.gradle 中，對 A 的依賴必須使用 api。Hilt 利用 AGP plugin 與 Dagger 的 Aggregating Processor，能穿透專案依賴圖自動收集所有標記 @Module + @InstallIn 的類別（即使是透過 implementation 傳遞的間接依賴）。因此，在多模組專案中，不需要為了讓頂層模組抓到 Hilt 綁定而刻意將依賴改為 api。使用 api 的決定應基於是否需要暴露該模組的公開 API 型別，而非 Hilt 注入需求。
