# Communicate with fragments

> 原文網址：https://boochlin.com/?p=772
> 發布日期：2025-08-21
> 文章編號：772

---

[參考原文](https://developer.android.com/guide/fragments/communicate)

請忘掉 getActivity() 或 findFragmentByTag()。

現代化的 Fragment 通訊，核心是「中介者模式」。你有兩種武器：ViewModel 和 Fragment Result API。

#### **一：ViewModel – **持續性的

當多個 Fragment 需要共享「持續性的狀態」時，就用 ViewModel。但關鍵是，這個「佈告欄」要放在哪？這取決於你選擇的作用域（Scope）。

**1. by viewModels()：私人筆記本** 作用域綁定 Fragment 自己，狀態不外流。

Kotlin

```
// ViewModel 只屬於這個 Fragment
private val viewModel: MyPrivateViewModel by viewModels()
```

**2. by activityViewModels()：大廳公用佈告欄** 作用域綁定宿主 Activity，適合跨 Fragment 共享。

Kotlin

```
// 所有依附於同一個 Activity 的 Fragment 都能拿到它
private val sharedViewModel: SharedViewModel by activityViewModels()
```

**3. by navGraphViewModels(graphId)：專案團隊的白板** 作用域綁定一個 Navigation Graph，適合特定業務流程（如註冊、結帳）中的數據共享。

Kotlin

```
// 只有在 R.id.signup_flow 這個導航圖中的 Fragment 能拿到它
private val signupViewModel: SignupViewModel by navGraphViewModels(R.id.signup_flow)
```

#### **二：Fragment Result API – 一次性**

當你需要「一次性的結果回傳」時，例如 A 叫 B 去選個日期再告訴 A，就用這個。

**1. 接收方 (RequestFragment)：登記收信** 用 setFragmentResultListener 監聽一個特定鑰匙 requestKey。

Kotlin

```
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setFragmentResultListener("requestKey") { key, bundle ->
        val result = bundle.getString("bundleKey")
        // 處理結果...
    }
}
```

**2. 發送方 (ResultFragment)：寄出回條** 用 setFragmentResult 發送帶有同樣 requestKey 的結果。

Kotlin

```
// 事情辦完後...
val result = bundleOf("bundleKey" to "some_result")
setFragmentResult("requestKey", result)
parentFragmentManager.popBackStack()
```
