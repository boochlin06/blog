# FMRadio 開發筆記 （四）- AlterActivity

> 原文網址：https://boochlin.com/?p=14
> 發布日期：2015-02-14
> 文章編號：14

---

## AlterActivity.java

此程式主要當使用者拔掉耳機後，會跳出一個視窗告訴使用者，請插回耳機，不然就要關掉FMRadio。

雖然這段程式挺短的.但是非常精華阿

開頭第一句話就解釋了一切

/** AlertActivity: use as a dialog, since Service cannot pop up a dialog */

這是個問題

原因

當我們利用Service 去接受系統傳來的訊息…但由於service並沒有對其他activity進行畫面變更的功能…這也是android 對於service所作的限制

也可以轉化成

/*在界面上添加一个VIEW，但是这个窗口不想依附任何一个activity*/

有以下幾個解決方案

A1:

主要來說service並沒有context去取得view

那或許可以說…那當要顯示dialog時…我就傳入一個context

但是service接收到系統訊息時（hdmi,usb,phone call）並無法確認自己目前再mainthread的activity是哪一個

那我們乾脆就不管是哪一個activity再跑

### 直接去取得**WindowManager****,ViewManager不就好了**

的然後添加一個dialog view

[http://developer.android.com/reference/android/view/WindowManager.html](http://developer.android.com/reference/android/view/WindowManager.html)

[http://developer.android.com/reference/android/view/ViewManager.html#addView(android.view.View, android.view.ViewGroup.LayoutParams)](http://developer.android.com/reference/android/view/ViewManager.html#addView(android.view.View,%20android.view.ViewGroup.LayoutParams))

window view manger

[http://blog.csdn.net/xieqibao/article/details/6567814](http://blog.csdn.net/xieqibao/article/details/6567814)

[http://disanji.net/2011/08/31/android-%E4%B9%8B-window%E3%80%81windowmanager-%E4%B8%8E%E7%AA%97%E5%8F%A3%E7%AE%A1%E7%90%86/](http://disanji.net/2011/08/31/android-%E4%B9%8B-window%E3%80%81windowmanager-%E4%B8%8E%E7%AA%97%E5%8F%A3%E7%AE%A1%E7%90%86/)

**** ****

code implement:

[http://blog.csdn.net/fengqiaoyebo2008/article/details/6309676](http://blog.csdn.net/fengqiaoyebo2008/article/details/6309676)

**** ****

A2:

如果不傳一個context給我…那麼啟動一個新的activity長的跟dialog很像不就好了

FMRadio2的實作方法就是這樣

需要注意的部份在於啟動方法

```
Intent startIntent = new Intent(this, AlertActivity.class);

startIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
```

**** ****

```
<activity

android:name=".AlertActivity"

android:theme="@style/Theme.AlertDialog" >

<intent-filter>

    <action android:name="toActivity" />

    <category android:name="android.intent.category.DEFAULT" />

</intent-filter>

</activity>
```

A3:

修改建立的Dialog屬性LayoutParams

```
WindowManager.LayoutParams.TYPE_SYSTEM_ALERT
```

[http://hi.baidu.com/lphack/item/1879eb8a614c9659840fabc4](http://hi.baidu.com/lphack/item/1879eb8a614c9659840fabc4)

[http://www.dotblogs.com.tw/merlin/archive/2012/04/13/71480.aspx](http://www.dotblogs.com.tw/merlin/archive/2012/04/13/71480.aspx)
