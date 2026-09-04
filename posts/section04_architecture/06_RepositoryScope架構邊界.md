# handle all repostiory operation  that out of Composable/ViewModel scope

> 原文網址：https://boochlin.com/?p=783
> 發布日期：2025-07-21
> 文章編號：783

---

#### 作法一：async + await（我要結果，但任務不能停）

這招的核心是「分離執行與等待的生命週期」。Repository 接受一個長壽的 externalScope，用 async 把任務丟進去獨立執行，UI 層則用 await 等待結果。UI 離開，await 取消，但 async 區塊的任務會繼續完成。

Kotlin

```
// In NewsRepository.kt
class NewsRepository(
    private val newsApi: NewsApiService,
    // 由外部 DI 框架提供一個 Application-Scope
    private val externalScope: CoroutineScope
) {
    private var cachedNews: List<News> = emptyList()
    private val mutex = Mutex()

    suspend fun getLatestNews(): List<News> {
        // 使用 async 將任務發射到外部 Scope，並用 await 等待結果
        return externalScope.async {
            val result = newsApi.fetchLatest()
            mutex.withLock { cachedNews = result }
            result
        }.await()
    }
}
```

#### 作法二：launch + Flow（射後不理，我只觀察）

說實話，async/await 有點像在門口乾等外送。在多數情況下，一個更優雅、更解耦的作法是「射後不理」。UI 只負責觸發刷新，然後持續觀察一個資料流 (Flow) 的變化。

Kotlin

```
// In NewsRepository.kt
class NewsRepository(...) {
    private val _newsFlow = MutableStateFlow<List<News>>(emptyList())
    val newsFlow: StateFlow<List<News>> = _newsFlow

    // 刷新方法變成射後不理
    fun refreshNews() {
        externalScope.launch {
            val result = newsApi.fetchLatest()
            _newsFlow.value = result // 更新資料流，UI 會自動收到
        }
    }
}

// In NewsViewModel.kt
class NewsViewModel(private val repo: NewsRepository) : ViewModel() {
    val news = repo.newsFlow // 直接觀察 Flow
    fun onRefresh() {
        repo.refreshNews() // 只需觸發，不用等待
    }
}
```
