# [草稿] produceState 橋接非 Flow 的「回呼」或「監聽器」模式

> 建立日期：2025-07-15
> 文章編號：707
> 狀態：草稿 (Draft)

---

當你需要與一些傳統的、基於監聽器 (Listener) 或回呼 (Callback) 的 API 互動時，`produceState` 也非常有用。你可以用它來將這些事件轉換成狀態流。

直接想到 `collectAsStateWithLifecycle()` API 實際上會使用 `produceState()` API。

[https://github.com/androidx/androidx/blob/androidx-main/lifecycle/lifecycle-runtime-compose/src/commonMain/kotlin/androidx/lifecycle/compose/FlowExt.kt](https://github.com/androidx/androidx/blob/androidx-main/lifecycle/lifecycle-runtime-compose/src/commonMain/kotlin/androidx/lifecycle/compose/FlowExt.kt)

```
// Copyright 2022 The Android Open Source Project
// ... (版權宣告)

/**
 * ... (官方註解)
 */
@Composable
public fun <T> Flow<T>.collectAsStateWithLifecycle(
    initialValue: T,
    lifecycle: Lifecycle,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED,
    context: CoroutineContext = EmptyCoroutineContext
): State<T> {
    // ---------------------- 證據就在這裡 ----------------------
    return produceState(initialValue, this, lifecycle, minActiveState, context) {
        lifecycle.repeatOnLifecycle(minActiveState) {
            if (context == EmptyCoroutineContext) {
                this@collectAsStateWithLifecycle.collect { this@produceState.value = it }
            } else {
                withContext(context) {
                    this@collectAsStateWithLifecycle.collect { this@produceState.value = it }
                }
            }
        }
    }
    // -----------------------------------------------------------
}
```

**範例：監聽位置更新**

假設你有一個 `LocationManager`，它透過 `registerLocationListener` 來接收位置更新。

Kotlin

```
@Composable
fun CurrentLocationDisplay(locationManager: LocationManager) {
    val location by produceState<Location?>(initialValue = null) {
        val listener = LocationListener { newLocation ->
            // 當監聽器被觸發時，更新 value
            value = newLocation
        }

        locationManager.registerLocationListener(listener)

        // awaitDispose 是關鍵，它會在 Composable 離開畫面時被調用
        // 用於執行清理工作，防止記憶體洩漏
        awaitDispose {
            locationManager.unregisterLocationListener(listener)
        }
    }

    if (location != null) {
        Text("Current location: ${location.latitude}, ${location.longitude}")
    } else {
        Text("Fetching location...")
    }
}
```

在這裡，`produceState` 將監聽器的回呼事件流，漂亮地轉換成了一個 UI 可以直接使用的 `State<Location?>`。

### `produceState` vs. 其他工具

理解它和相似工具的區別，能幫助你更好地做選擇：

* **vs. `LaunchedEffect`**:

  * `LaunchedEffect` 用於執行「射後不理」的副作用，它本身**不產生**狀態。例如：顯示一個 Snackbar、觸發一次分析事件。

  * `produceState` 的**核心目的就是產生一個狀態** (`State<T>`) 給 UI 使用。

* **vs. `collectAsState` / `collectAsStateWithLifecycle`**:

  * 如果你要觀察的非同步來源**已經是 `Flow`**，那麼 `collectAsState` 是更直接、更慣用的選擇。

  * `produceState` 更加通用。你可以認為 `flow.collectAsState()` 是 `produceState { flow.collect { value = it } }` 的一個語法糖。當你的資料來源不是 Flow (例如 RxJava 的 `Observable` 或是一個 suspend 函式) 時，`produceState` 就派上用場了。

### 總結

你應該在以下情況下優先考慮使用 `produceState`：

1. **你想將一個「一次性」的非同步操作 (如網路請求) 的結果，轉換成 Compose 的 `State`。**

2. **你想將一個「持續性」的、基於回呼/監聽器的外部事件源，橋接成 Compose 的 `State`。**

3. **你想觀察的資料來源不是 `Flow`，無法直接使用 `collectAsState`。**

它是一個強大的工具，能讓你以一種聲明式且生命週期安全的方式，將外部世界的動態資料引入到你的 Composable 函數中。
