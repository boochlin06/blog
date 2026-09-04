# 麻煩 ViewModelProvider.Factory 你了，因為系統不讓我直接 new 一個 ViewModel

> 原文網址：https://boochlin.com/?p=636
> 發布日期：2025-07-14
> 文章編號：636

---

這個規範是為了解決 **Android 元件生命週期不一致**所帶來的大問題。

---

### ## 保險箱

* ** 開發者**

* **Activity/Fragment (UI 控制器)**：是您的「錢包」。您隨身帶著它，但它可能隨時會遺失、被偷或換新（例如螢幕旋轉時，舊的 Activity 會被銷毀，新的會被建立）。

* **ViewModel**：是您的「護照、房契」等**重要資產**。您絕對不會把這些東西放在錢包裡，因為錢包太不可靠了。

* **Android 作業系統**：是「政府保管機構」。它提供了一個超級安全的保管箱服務。

* **ViewModel 的資料**：就是保管箱裡的護照和房契。

現在，您想把您的護照和房契（ViewModel）存放到政府的保管箱裡。您不能直接跑進金庫、自己找個箱子把東西丟進去。

您必須遵循官方程序：

1. **填寫申請表**：您必須填寫一張詳細的申請表，告訴政府工作人員，當他們為您開設保管箱時，**要放入哪些東西**（例如，您的 Repository 或其他參數）。

2. **提交申請**：您將申請表交給工作人員（ViewModelProvider）。

3. **由工作人員操作**：工作人員根據您的申請表，為您建立保管箱，並將您指定的物品（參數）放入其中。

在這個比喻中，那張**「申請表」**，就是 ViewModelProvider.Factory。

---

### 為何要有這個規範？核心原因：生命週期不同步

Activity/Fragment 的生命週期 和 ViewModel 的生命週期 是完全不同的。

1. Activity/Fragment 的生命週期很脆弱：當使用者旋轉螢幕、系統記憶體不足、或切換深色模式時，當前的 Activity/Fragment 例項 (instance) 會被銷毀 (destroy)，然後一個全新的例項會被建立。

2. ViewModel 的生命週期很持久：ViewModel 的設計初衷，就是要活得比 Activity/Fragment 更久。它會在這些配置變更中存活下來，並被重新建立的 Activity/Fragment 繼續使用，從而保留 UI 相關的資料。

#### 如果沒有這個規範，會發生什麼？

假設 Android 允許您在 Activity 的 onCreate 中直接建立 ViewModel：

```
// ！！！這是一個錯誤的假設性程式碼！！！
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val myRepository = UserRepository() // 每次都會建立新的 Repository
        
        // 如果可以直接這樣做...
        val viewModel = MyViewModel(myRepository) 
    }
}
```

當螢幕一旋轉：

1. 舊的 MyActivity 被銷毀，您親手建立的 viewModel 物件也跟著**被銷毀**。

2. 新的 MyActivity 被建立，onCreate 再次執行。

3. 一個**全新的** viewModel 物件被建立。

**結果：ViewModel 裡面保存的所有資料全部遺失！** 這完全違背了 ViewModel 的設計目的。

---

### 系統如何解決這個問題？

為了讓 ViewModel 能夠存活，Android 系統採取了「控制權反轉」的設計。

1. **控制權交給系統**：ViewModel 的建立和儲存**不是由您直接控制**，而是交由 Android Framework 統一管理。系統會將 ViewModel 存放在一個獨立於 Activity/Fragment 的地方 (ViewModelStore)。

2. **開發者提供「說明書」**：當系統需要建立一個 ViewModel 時（通常是第一次請求時），它不知道該如何處理您自訂的建構函式參數（例如 UserRepository）。

3. **Factory 的角色**：ViewModelProvider.Factory 就是您提供給系統的「建立說明書」。您在 Factory 的 create 方法中，清楚地告訴系統：「當你需要建立 MyViewModel 時，請這樣做：MyViewModel(someRepository)」。

—
