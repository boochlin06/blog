# id(“org.kotlin.xxxx”) version “2.1.0” apply false  這不要你還特地寫上去啊？.

> 原文網址：https://boochlin.com/?p=632
> 發布日期：2025-02-14
> 文章編號：632

---

假設我們的專案 MyPlayerApp 是一個音樂播放器，它有三個模組：

1. app: 主應用程式模組，包含 UI 畫面。**（需要 Compose）**

2. feature_playlist: 播放清單功能的模組，也包含 UI 畫面。**（需要 Compose）**

3. core_network: 負責處理網路請求的模組，沒有任何 UI。**（不需要 Compose）**

### 專案檔案結構

```
MyPlayerApp/
├── app/
│   └── build.gradle.kts  (app 模組的設定檔)
├── feature_playlist/
│   └── build.gradle.kts  (播放清單模組的設定檔)
├── core_network/
│   └── build.gradle.kts  (網路模組的設定檔)
└── build.gradle.kts      (專案根目錄的設定檔，也就是「總管理處」)
```

---

### 第 1 步：在「總管理處」公告規則

我們在專案**根目錄**的 build.gradle.kts 中，使用 apply false 來**統一定義**所有模組可能會用到的外掛版本。

**MyPlayerApp/build.gradle.kts (根目錄)**

```
// 這裡是「總管理處」，只發布公告，不執行具體安裝
plugins {
    // 公告1：本公司是「Android 應用程式」專案，若有模組是 App，請用 8.4.1 版
    id("com.android.application") version "8.4.1" apply false

    // 公告2：若有模組是「Android 函式庫」，請用 8.4.1 版
    id("com.android.library") version "8.4.1" apply false

    // 公告3：本公司統一使用 Kotlin 語言，標準版號是 1.9.23
    id("org.jetbrains.kotlin.android") version "1.9.23" apply false

    // 公告4：若需要 Jetpack Compose UI 功能，統一使用 1.6.10 版
    id("org.jetbrains.kotlin.plugin.compose") version "1.6.10" apply false
}
```

**重點**：這個檔案現在就像是公司的公告欄，清楚列出了所有工具的標準版本。

---

### 第 2 步：在「各部門」按需申請

現在，每個模組根據自己的需求，去「申請」使用這些已經被定義好的外掛。

#### app 模組 (需要 Compose)

app 模組是主程式，有 UI 介面，所以它需要 com.android.application 和 org.jetbrains.kotlin.plugin.compose。

**MyPlayerApp/app/build.gradle.kts**

```
plugins {
    // 申請1：向總部申請成為一個「Android 應用程式」
    id("com.android.application")

    // 申請2：申請使用「Kotlin 語言」
    id("org.jetbrains.kotlin.android")

    // 申請3：申請使用「Jetpack Compose UI 功能」
    id("org.jetbrains.kotlin.plugin.compose") // 這裡完全不用寫 version
}

android {
    // ... 其他 android 設定
    buildFeatures {
        compose = true // 啟用 Compose 功能
    }
    // ...
}
```

#### feature_playlist 模組 (需要 Compose)

這個模組是個函式庫，但它也有 UI，所以它需要 com.android.library 和 org.jetbrains.kotlin.plugin.compose。

**MyPlayerApp/feature_playlist/build.gradle.kts**

```
plugins {
    // 申請1：向總部申請成為一個「Android 函式庫」
    id("com.android.library")

    // 申請2：申請使用「Kotlin 語言」
    id("org.jetbrains.kotlin.android")

    // 申請3：申請使用「Jetpack Compose UI 功能」
    id("org.jetbrains.kotlin.plugin.compose") // 同樣不用寫 version
}

android {
    // ...
}
```

#### core_network 模組 (不需要 Compose)

這個模組只負責網路，沒有 UI，所以它**完全不需要申請** Compose 外掛。

**MyPlayerApp/core_network/build.gradle.kts**

```
plugins {
    // 申請1：向總部申請成為一個「Android 函式庫」
    id("com.android.library")

    // 申請2：申請使用「Kotlin 語言」
    id("org.jetbrains.kotlin.android")

    // 完全沒有申請 Compose，因為它用不到
}

android {
    // ...
}
```

###  

這樣做最大的好處是**維護性**。

假設幾個月後，Google 發布了新的 Compose Compiler 1.7.0 版。你**只需要做一件事**：

**回到專案根目錄的 build.gradle.kts，修改那一條「公告」即可。**

Kotlin

```
// MyPlayerApp/build.gradle.kts
plugins {
    // ...
    // 只需修改這一行，整個專案就升級了！
    id("org.jetbrains.kotlin.plugin.compose") version "1.7.0" apply false
}
```

所 Compose 功能的模組（app 和 feature_playlist）在下次建構時，都會自動拿到 1.7.0 版本，你完全不需要去一個個修改它們的檔案。這就是 apply false 在實際開發中的強大之處。

—
