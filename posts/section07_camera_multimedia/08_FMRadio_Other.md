# FMRadio 開發筆記 （五）- Other

> 原文網址：https://boochlin.com/?p=17
> 發布日期：2015-02-14
> 文章編號：17

---

**這篇功能單純，紀錄其他比較小的元件用法，也是FMRadio最後一篇。**

1.挑出Manifest沒用過的參數，筆記下來

2.FMMediaButtonIntentReceiver.java 的 ACTION 相關

## 1.Android Manifest note:

```
[supports-screens
    android:largeScreens="true"

    android:normalScreens="true"

    android:smallScreens="true"

    android:anyDensity="false"/]
```

「normalScreens」代表的是一般解析度（如 G1, Hero），「smallScreens」代表的是較低解析度（如 Tatoo）。

現在將程式發佈到 Android Market 時若沒做以上設定，你寫的程式將不會顯示在 QVGA/WVGA 機器的 Market 中。

PS: 將 Target 設為 1.6 跟在 AndroidManifest 清單中設定 minSdkVersion 最低相容版本並不衝突，只要 minSdkVersion 維持不變，低於 1.6 版的機器還是可以使用你的程式。

update: 事實上「smallScreens」代表的是小螢幕，「normalScreens」是一般螢幕，「largeScreens」當然是大螢幕。一般三者的分界點大概在3吋跟4吋。所以 Tattoo (2.8″) 被歸在「smallScreens」範疇。

**** ****

```
android:lunchmode = singleTask
```

**** ****

该模式的Activity是单例的，即在同一时刻Android系统中只能存在该Activity

的一个实例，所以当重复启动该Activity时并不会创建该Activity的新实例，而

是重新使该Activity的实例进入交互状态，如果该Activity的实例不位于当前任

务的栈顶，则将该实例之上的所有Activity释放，以使该Activity重新成为栈顶。

ps.為了讓fmradiomain永遠只有一個

**** ****

**receiver中的**

```
ACTION_MEDIA_BUTTON  ="android.intent.action.MEDIA_BUTTON"
```

Intent 附加值为(Extra)点击MEDIA_BUTTON的按键码 ：

//获得KeyEvent对象

**

```
KeyEvent keyEvent = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
```

//获得Action

```
String intentAction = intent.getAction() ;
```

ps.監聽耳機上的按鈕事件

ref:http://blog.gasolin.idv.tw/2009/10/android-market.html

**有些ref忘記，sorry**

## 2.FMMediaButtonIntentReceiver.java

extend broadcastReceiver

監聽

android.intent.action.MEDIA_BUTTON

(耳機上的下一首，暫停)

利用

```
KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
```

取出事件類型

根據KEYCODE & key_action

辨識

KEYCODE_HEADSETHOOK 為必傳到要條件

ACTION_DOWN 為確實按下

MEDIA_PAUSE OR MEDIA PLAY 等符合需求的ACTION確認後

最後用

FM_MEDIA_BUTTON = “xxxxx   公司機密   xxxxx”為action為目標

將KeyEvent event 包在Intent.EXTRA_KEY_EVENT用 context.sendBroadcast傳給service使用
