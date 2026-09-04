# lateinit vs lazy

> 原文網址：https://boochlin.com/?p=778
> 發布日期：2024-07-21
> 文章編號：778

---

**lateinit：var** **給別人初始化的承諾** 最適合用在生命週期或依賴注入，當你確定某個外部力量會幫你搞定初始化時：

Kotlin

```
lateinit var adapter: RecyclerView.Adapter

fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    adapter = MyAdapter(data) 
}
```

**lazy：val把昂貴的計算往後推** 用在成本高昂、又不一定會用到的唯讀資源，把運算拖到最後一刻：

Kotlin

```
// 這張圖超大，沒人要看就別浪費記憶體載入
val profileBitmap: Bitmap by lazy {
    Log.d("Perf", "Loading large bitmap NOW!")
    BitmapFactory.decodeResource(resources, R.drawable.large_image)
}
```

一個是「**之後會有人搞定**」的 var，一個是「**我只在必要時算一次**」的 val。別再用錯地方，浪費那寶貴的資源了。
