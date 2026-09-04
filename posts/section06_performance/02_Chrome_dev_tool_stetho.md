# Chrome dev tool for android –  steho

> 原文網址：https://boochlin.com/?p=40
> 發布日期：2015-05-21
> 文章編號：40

---

### facebook  開發的開源工具  “steheo”

官網簡介
「Stetho is a sophisticated debug bridge for Android applications. When enabled, developers have access to the Chrome Developer Tools feature natively part of the Chrome desktop browser. Developers can also choose to enable the optional dumpapp tool which offers a powerful command-line interface to application internals.」
這個單字是聽診器，一言以蔽之「可以用chrome devtools就可以debug android」
1. network inspection
2. database inspection
3. view hierarchy
其實市面上也有一些想關工具，甚至原生的android device monitor都可以很猛了
但是大概最大誘因就在於他可以免root就可以觀測了
因此我想到的scenrio  就是拿到user版的機器，想要看裡面的資料，這個就可能有點用
另外一個就是network inspection  這個可以動態觀測 網路下載資料的大小，耗時
玩玩看吧

getting started
官網教學
[http://facebook.github.io/stetho/](http://facebook.github.io/stetho/)
中文版

[http://stormzhang.com/android/2015/03/05/android-debug-use-chrome/](http://stormzhang.com/android/2015/03/05/android-debug-use-chrome/)

自己動手改
1.

```
dependencies { compile 'com.facebook.stetho:stetho:1.1.1' }
```

2. 再application裡加入

```
public class MyApplication extends Application { 
    public void onCreate() { 
         super.onCreate();
         Stetho.initialize( Stetho.newInitializerBuilder(this).enableDumpapp(
              Stetho.defaultDumperPluginsProvider(this)) .enableWebKitInspector(
              Stetho.defaultInspectorModulesProvider(this)) .build());
    }
 }
```

3. 網路觀測比較麻煩  要在http 相關class  裡做攔截
以okhttp來說

```
OkHttpClient client = new OkHttpClient();
client.networkInterceptors().add(new StethoInterceptor());
```

 使用前要記得

```
dependencies {
  // 各自取用
  compile 'com.facebook.stetho:stetho-okhttp:1.0.1'
  compile 'com.facebook.stetho:stetho-urlconnection:1.0.1'
}
```

 如果是HttpURLConnection 這就比較麻煩
需要在header 加入可解構gzip 並以StethoURLConnectionManager去做統合處理

 如果要跟google 出的volley 做整合，可以參考
[http://ligol.github.io/blog/2015/05/05/discovering-and-using-stetho-with-some-network-library/](http://ligol.github.io/blog/2015/05/05/discovering-and-using-stetho-with-some-network-library/)
其實就是先將okhttp 作為volley 網路底層工作者並加入觀測者於其中
然後放入 volley 的work queue,
okhttpstack是另外寫的，主要是將有關測者的okhttp 加入httpstack
的實做流程裡…

4. 最後開啟你的chrome browserr然後再網址列輸入

```
chrome://inspect
```

大功告成
