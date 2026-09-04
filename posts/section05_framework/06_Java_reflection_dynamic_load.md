# Java reflection in Android －fast scroll bar in list view / dynamic load apk.

> 原文網址：https://boochlin.com/?p=122
> 發布日期：2016-03-27
> 文章編號：122

---

雖然最近並沒有找工作，但是看了一本書『Android 工程師面試寶典』，主要是因為這是一本充滿小技巧的技術書，看一看就會有一種『原來如此』的奇妙讚嘆，但是如過硬要說一句結論的話，就是面試問這種問題真的超無聊，根本是 stackoverflow 問題大集合（而且有點舊）。而不少問題是用 java reflection 的技術來解答。老實說面試中，或是出在筆試考卷上，這不上網查根本答不出來啦。但是這也引起對於 java relection 技術框架的興趣。

這一章節先提一下此書 android 在 java reflection 的應用情況。

１．如何在 list view 上加上 fast scroll bar ?

２．如何動態載入 apk file 的 class？

————-分割線———————

### 1. 如何 list view 加上 fast scroll bar ?

首先要先理解 fast scroll bar 是啥鬼？可以吃嗎？

其實就是我們常在使用網頁瀏覽器的右邊，滑鼠可以拖曳的 bar ，可以快速移動頁面內容（上下左右都可以），還有不可以吃

直接上圖

![fscrbar](../../assets/images/fscrbar.jpg)

[https://boochlin.com/wp-content/uploads/2016/03/fscrbar.jpg](https://boochlin.com/wp-content/uploads/2016/03/fscrbar.jpg)

完了，看了最新的資料發現這有點無聊，因為後來 android 直接釋出 api 來了，而且超簡單。那某這本書到底在問題個屁啊？原來這問題是 2011 的問題，那抹我對應一下 api level，原來是 3

算了，都提到了就繼續吧，就當作是簡單提一下java reflection api 介紹。

```
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
    try {
        // FastScroller.mThumbDrawable 為 fast scroll bar 的畫面，首先必須要透過 AbstractList.mFastScroller
        // 取得 FastScroller obj.
        Field field = AbstractList.class.getDeclaredField("mFastScroller");
        // 設定可以存取
        field.setAccessible(true);
        Object obj = field.get(mContentListView);
        // 取得 FastScroller.mThumbDrawable 變數的 field obj.
        field = field.getType().getDeclaredField("mThumbDrawable");
        field.setAccessible(true);
        // 取得要載入的 fast scroller bar 的 drawable
        Drawable drawable = (Drawable)field.get(obj);
        drawable = getResources().getDrawable(R.drawable.widget_button);
        // 重新設定fast scroller bar 的 drawable
        field.set(obj, drawable);
    } catch (NoSuchFieldException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
}
```

首先可以看得出來，我們要先知道我要處理的對象到底在哪裡且是啥名子，然後用 Field 取得實體，設定可以 accessible (true)，然後直接對obj設定內容，其實可以發現最重要的工作是找出為子，同時這也有可能要考慮 framework 的內容，各家廠商也有可能匯改，所以這招基本上有時靈有時不靈。

如果看官只是想看快速改出 list view scroll bar.

```
listView = (ListView) getView().findViewById(R.id.listView);
listView.setFastScrollEnabled(true);
```

突然覺得上面的內容有點太過簡單或是太舊，換個題目，到第二題

### ２．如何動態載入 apk file 的 class？

首先在這之前，先擴展這問題，如何動態載入**未安裝**的 apk 檔案?

由於android 是 run on dalivk machine, 並非 傳統的 jvm 上，因此無法直接載入傳統的jar檔，但apk file 已經被編譯成 dalivk 的 dex file. 因此才可以正常載入。但同時我們也必須用特殊的方法去開啟 dex file.

那某這方法啥時會使用到，主要是要避免頻煩更新 native app 對使用者造成的煩躁感，利用這技術可以讓新內容的 apk 可以再不用play store 更新的情況，偷偷重網路下載下來，並更新資料。

```
try {
    // myapk.apk 是我們想要載入的對象，myapk_temp.apk 為系統最佳化臨時產生的檔案
    DexFile dexFile = DexFile.loadDex("sdcard/myapk.apk","/sdcard/myapk_temp.apk",0);
    //or 使用dexclassloader , 並利用官方指令直接拿到路徑，達到上述的效果
    DexClassLoader cl = new DexClassLoader(optimizedDexOutputPath.getAbsolutePath(),
                    Environment.getExternalStorageDirectory().toString(), null, getClassLoader());
    //下述兩具效果是一樣的， 利用 java reflection 技術取得 getHigh 方法的物件，
    Object obj = dexFile.loadClass("com.android.acer.home",null).newInstance();
    cl.loadClass("com.android.acer.home")
    Method method = obj.getClass().getDeclaredMethod("getUserName", null);
    // invoke 方法，並取得返回值
    String result = String.valueOf(method.invoke(obj,null));
} catch (IOException e) {
    e.printStackTrace();
} catch (InstantiationException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (NoSuchMethodException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

好吧，一些好像有用又沒有用的 java reflection 使用情境介紹到此為止。下一章節有可能會是介紹java reflection本身是啥鬼

reference:

[http://blog.coliam.net/?p=993](http://blog.coliam.net/?p=993)

android 工程師面試寶典
