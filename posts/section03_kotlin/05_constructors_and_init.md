# Difference between constructors and init in kotlin

> 原文網址：https://boochlin.com/?p=765
> 發布日期：2025-07-17
> 文章編號：765

---

[原文](https://medium.com/@appdevinsights/difference-between-constructors-and-init-in-kotlin-3ba940d1abdc)

## 誰負責「定義」？誰負責「執行」？

在 **Kotlin** 中，Object 創建是個有順序的過程。constructor 和 init block 是關鍵成員，各自有明確職責。搞不清這點，可能導致初始化順序錯誤或 Runtime Issues。

### **執行順序：**

1. **Primary Constructor**

2. **Property Initializers**

3. **init blocks** (按宣告順序)

4. **Secondary Constructor** (次要建構函式) 的主體 (如果有，且在呼叫 Primary Constructor 之後)

簡單來說：constructor 決定參數，Property 接收並初始化，然後 init block 才根據這些去執行邏輯。

---

## 職責區分

1. constructor：Object 的「初始藍圖」

  * **職責：** 定義並接收創建 Class Instance 所需的**初始 Parameters**。

  * **類型：**

    * **Primary Constructor：** 最常見，直接在 Class Header 中定義，可將 Parameters 宣告為 **Properties**。

    * **Secondary Constructor：** 提供多種創建方式，但**必須呼叫 Primary Constructor**。

  * **執行時機：** 在 init blocks 和 Property Initializers **之前**執行。

2. init block：Object 的「立即設定與驗證」

  * **職責：** 在 Primary Constructor 完成後，**立即執行**與 Object 狀態相關的**初始化 Logic**。它不接收 Parameters，只利用已收到的 Parameters 或 Properties 進行額外設定、Validation 或複雜操作。

  * **執行時機：** 緊隨 Primary Constructor 和所有 Property Initializers 之後自動執行。可有多個 init blocks，依序執行。

---

## 案例：用戶設定檔的初始化

Kotlin

```
class UserProfile(val userId: String, email: String) { // Primary Constructor
    val emailAddress: String = email // Property Initializer

    init { // Init Block 1: Validation
        println("Init Block 1: Validating user data.")
        require(userId.isNotBlank()) { "User ID cannot be blank." }
    }

    init { // Init Block 2: Further setup
        println("Init Block 2: User profile setup complete for $userId.")
    }

    constructor(userId: String) : this(userId, "default@example.com") { // Secondary Constructor
        println("Secondary Constructor: User created with default email.")
    }
}

fun main() {
    val userA = UserProfile("john.doe", "john.doe@example.com")
    // Output sequence:
    // Init Block 1: Validating user data.
    // Init Block 2: User profile setup complete for john.doe.

    println("---")

    val userB = UserProfile("jane.doe") // Calls Secondary Constructor
    // Output sequence:
    // Init Block 1: Validating user data.
    // Init Block 2: User profile setup complete for jane.doe.
    // Secondary Constructor: User created with default email.
}
```
