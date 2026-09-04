# Android Intents with Chrome

> 原文網址：https://boochlin.com/?p=74
> 發布日期：2015-07-31
> 文章編號：74

---

讓網頁去連結 android ap

這是還蠻正常的需求衍生，主要可以來看 google play store ，當你在逛一些app 介紹時

看到link直接點下去，通常有兩種可能

1. 直接開啟新分頁到 play store 下載頁面

2. 啟動play store app ，然後進入下載頁面

這次要談的就是第二點

### **如何讓網頁的連結具有開啟app的效力**

[英文官網](https://developer.chrome.com/multidevice/android/intents)

[中文官網](https://developers.google.com/app-indexing/webmasters/app?hl=zh-tw)

中文居然寫的比較詳細

懶的看官網的話

以下節錄重點，以及後面測試心得

intent:

HOST/URI-path // Optional host

#Intent;

package=[string];

action=[string];

category=[string];

component=[string];

scheme=[string];

end;

對應到

intent:

//scan/

#Intent;

package=com.google.zxing.client.android;

scheme=zxing;

end;

寫出來應該會像是

<a href=”intent://scan/#Intent;scheme=zxing;package=com.google.zxing.client.android;end”> Take a QR code </a>

上述是google 的教學資料 根據自己測試結果

<a href=”intent://com.android.launcher3/#Intent;scheme=http;action=com.android.ACTION.LAUNCH_LEFTPAGE;end;”>

start launcher</a>

如果對應到自己相關的ap ，必須先在 android manifest 裡加入 intent filter

<intent-filter>

<action android:name=”android.intent.action.VIEW” />

<action android:name=”com.android.ACTION.LAUNCH_LEFTPAGE” />

<data android:scheme=”http”

android:host=”com.android.launcher3″/>

<category android:name=”android.intent.category.DEFAULT” />

<category android:name=”android.intent.category.BROWSABLE” />

</intent-filter>

以上是最基本的配置資料 如果你成功加入到你的app 之後，可以先用 adb 來進行測試

```
adb shell am start -a com.android.ACTION.LAUNCH_APP -d "http://com.android.launcher3"
```

這樣應該會成功開啟你的app

接者進入正題

[start your app](intent://com.android.launcher3/#Intent;action=com.android.ACTION.LAUNCH_APP;scheme=http;end)

點擊上述連結，應該就會開啟你的activity.

其他補充事項：

1.<action android:name=“android.intent.action.VIEW” />

可加可不加，這只是多對應一種 view action.(通常browser會發)

2.<category android:name=“android.intent.category.BROWSABLE” />

這個一定要加，基本上這就是規則

Only activities that have the category filter, android.intent.category.BROWSABLE are able to be invoked using this method as it indicates that the application is safe to open from the Browser.

3.<category android:name=“android.intent.category.DEFAULT” />

這個基本上已經是對android 了不了解的問題，不加的話 app 無法開啟這個activity.

4. host 是不一定要加，但是沒有host 就一定要加 path (可以參考[官網](https://developer.chrome.com/multidevice/android/intents)的兩種寫法)

也可以只有host

其他神奇情況：

1.你說你點了上面的連結 app 卻沒有反應

廢話，你在電腦上面用滑鼠點擊當然沒屁用阿

2.你說你在手機上面開官方browser點擊了阿，app 還是沒反應

你的browser可能沒porting 這段[code](https://code.google.com/p/android-source-browsing/source/browse/core/java/android/content/Intent.java?repo=platform--frameworks--base#6514)

3.你說你開 chrome 才去點擊的

唉，這功能也是google 後來才想到的，去更新一下你的chrome

4.你說其他browser（firefox,百度）點擊了沒用

糟糕，其他browser當然沒有parsing這段網頁的功能，這功能基本上只有 chrome , 原生browser 會 support.

5.以上我都有搞定，但還是不能用

那應該是我的問題，抱歉，你可以留言反應，謝謝
