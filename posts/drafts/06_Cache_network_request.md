# [草稿] Cache the result of the network request

> 建立日期：2025-07-20
> 文章編號：781
> 狀態：草稿 (Draft)

---

class NewsRepository(private val newsRemoteDataSource: NewsRemoteDataSource) {
private val latestNewsMutex = Mutex()
private var latestNews: List = emptyList()

```
suspend fun getLatestNews(refresh: Boolean = false): List<ArticleHeadline> {
    // 將整個「檢查-然後-執行」的邏輯塊鎖起來
    latestNewsMutex.withLock {
        // 只有在需要刷新，或快取為空時，才執行網路請求
        if (refresh || latestNews.isEmpty()) {
            // 注意：這個修正會將網路請求放在鎖內，
            // 這意味著如果網路很慢，會長時間阻塞其他試圖讀取快取的執行緒。
            // 對於需要高併發讀取的場景，這不是最佳方案，但它能 100% 解決競爭條件。
            this.latestNews = newsRemoteDataSource.fetchLatestNews()
        }
    }
    // 再次讀取並回傳，確保拿到的是最新的值
    return latestNewsMutex.withLock { this.latestNews }
}
```

}
