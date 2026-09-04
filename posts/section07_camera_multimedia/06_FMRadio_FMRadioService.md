# FMRadio 開發筆記 （三）- FMRadioService

> 原文網址：https://boochlin.com/?p=13
> 發布日期：2015-02-14
> 文章編號：13

---

# FMRadioService.java

Service 為android 固定運作在後台的一個componemt，聽收音機一般來說是放在後台運作，然後可以同時做其他事情，或是單純希望在sleep mode 能繼續運作，為此，此service主要功能為保持與電台的連線，運用 audio stream 作持續輸出，以及對應其他相關硬體的回應，如下:

1. 與底層溝通，取得 fm chip的相關參數，作為電台資訊的讀取(根據地點變更頻段，或是訊號強弱，做出對應行為)。

2. 取得audio stream，對耳機或是喇叭作相關輸出。

3.監聽硬體的狀態變化，做出對應行為(拔掉耳機之類的)。

4.監聽電話狀態的改變，一般來說不會希望因為聽收音機 ，導致忽略來電，這也是一般音樂類ap需要注意的地方。

## *class*

## 

### **WakeLock  ＆****PowerManager**

**** ****

**是系統的遠端服務**

**wl.acquire(); //喚醒點亮螢幕**

**** ****

**//這個期間螢幕將點亮**

**mWakeLock.acquire(10*1000);**

**wl.release(); //恢復螢幕到黑暗**

**wakeLock.setReferenceCounted(false);為設置常亮必備前提**

****[http://www.cnblogs.com/nbtsy/archive/2012/03/07/2383093.html](http://www.cnblogs.com/nbtsy/archive/2012/03/07/2383093.html)

**再fmradio中…於fmoff & fmon中能保持休眠時保留fmradio的cpu**

**從powmanager 中取得新的newWakeLock並設置為PowerManager.PARTIAL_WAKE_LOCK**

**** ****

下面定义的flag是在newWakeLock方法中要接收的参数，通过该flag，你可以定义系统的电源的展示效果。比如：

*                                                             cpu    screen    keyboard

* PARTIAL_WAKE_LOCK                    on      off          off

* SCREEN_DIM_WAKE_LOCK           on      dim        off

* SCREEN_BRIGHT_WAKE_LOCK    on      bright     off

* FULL_WAKE_LOCK                          on      bright     bright

这些flag是相互排斥的，一次只能定义一个。

如果你持有PARTIAL_WAKE_LOCK锁，不论任何定时器甚至是按下电源按钮，cpu都将继续运行，无法进入休眠状态。除非你释放掉它。

其他锁的话，虽然cpu也在运行，但是当用户按下电源按钮时，设备将立刻进入休眠状态。

如果申请了partial wakelock,那么即使按Power键,系统也不会进Sleep,如Music播放时

如果申请了其它的wakelocks,按Power键,系统还是会进Sleep

正常情况下wakelocks实际上是没有被打开的，当需要时，它将通过特定的flag启动屏幕和键盘。 比如在应用中，涉及到向用户发送消息时，需要让用户立刻看到。此时会点亮屏幕。当WakeLock锁被释放的时候，activity的定时器将被重设，这将导致屏幕亮更长的时间。

[http://blog.csdn.net/xieqibao/article/details/6562256](http://blog.csdn.net/xieqibao/article/details/6562256)

[http://blog.csdn.net/hzdysymbol/article/details/4004791](http://blog.csdn.net/hzdysymbol/article/details/4004791)

 

### **使用wakelock**

当设计在后台播放媒体的应用时，当你的service正在运行时，设备可能进入休眠．因为Android系统在休眠时会试着节省电能，那么系统会试着关闭电话的任何不必要的特性，包括CPU和WiFi．然而，如果你的service正在播放或接收音乐，你就想阻止系统干涉你的播放工作．

 

 

为了在上述情况下保证你的service继续运行，你必须使用”wakelocks”．一个wakelock是一种通知系统在手机空闲时也应为你的应用保留所用特性的途径．

注意：你总是应该保守的使用wakelocks并且仅在真证需要时才持有它．因为它们会显著的减少设备电池的寿命．

 

 

### **MediaRecorder**

非常完整的錄音功能state translater

[http://android.toolib.net/reference/android/media/MediaRecorder.html](http://android.toolib.net/reference/android/media/MediaRecorder.html)

[http://blog.xuite.net/dain198/study/46393059-%E7%AD%86%E8%A8%98+Android+MediaRecorder+%E5%AA%92%E9%AB%94%E9%8C%84%E8%A3%BD%E9%A1%9E%E5%88%A5+-+%E7%B0%A1%E6%98%93%E9%8C%84%E9%9F%B3%E6%A9%9F](http://blog.xuite.net/dain198/study/46393059-%E7%AD%86%E8%A8%98+Android+MediaRecorder+%E5%AA%92%E9%AB%94%E9%8C%84%E8%A3%BD%E9%A1%9E%E5%88%A5+-+%E7%B0%A1%E6%98%93%E9%8C%84%E9%9F%B3%E6%A9%9F)

**** ****

於FMRADIO中維護錄音功能…將收音機的聲音錄起來

不過由於後來spec拿掉…所以……

**** ****

### **AudioManager**

[http://android.toolib.net/reference/android/media/AudioManager.html](http://android.toolib.net/reference/android/media/AudioManager.html)

於fmradiomain.java中有談過…主要以再開啟時檢查手機狀態

於fmradioservice.java功能可多了

1.取得a2dp藍牙

2.於startfm stopfm中取得audiofocus

3.focus change 對應動作

4.detect routeAudio

5.mute  之類的…

[http://developer.android.com/reference/android/media/AudioManager.html#abandonAudioFocus(android.media.AudioManager.OnAudioFocusChangeListener)](http://developer.android.com/reference/android/media/AudioManager.html#abandonAudioFocus(android.media.AudioManager.OnAudioFocusChangeListener))

[http://blog.csdn.net/nkmnkm/article/details/7607934](http://blog.csdn.net/nkmnkm/article/details/7607934)

[http://www.cnblogs.com/over140/archive/2011/08/07/2130393.html](http://www.cnblogs.com/over140/archive/2011/08/07/2130393.html)

 

 

### **TelephonyManager**

**** ****

```
tmgr = (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);

tmgr.listen(mPhoneStateListener, PhoneStateListener.LISTEN_CALL_STATE |

PhoneStateListener.LISTEN_DATA_ACTIVITY);

PhoneStateListener.LISTEN_DATA_ACTIVITY

PhoneStateListener.LISTEN_CALL_STATE
```

代表意義請參考

[http://hi.baidu.com/wentaokou/item/717a61d23bb49cbd32db904b](http://hi.baidu.com/wentaokou/item/717a61d23bb49cbd32db904b)

於fmradio中主要為fmActionOnCallState（）對應手機打來的行為.

```
//if Call Status is non IDLE we need to Mute FM as well stop recording if

//any. Similarly once call is ended FM should be unmuted.

CALL_STATE_RINGING

CALL_STATE_OFFHOOK

CALL_STATE_IDLE
```

****

————————–divider————————-****

## **Method**

#### 1.對外部事件的控制.hdmi.headset,sd card airplane mode control.audiomanger..

名子可以看出各大對應動作

**registerScreenOnOffListener（）**

```
/**

* Registers an intent to listen for

* ACTION_SCREEN_ON/ACTION_SCREEN_OFF notifications. This intent

* is called to know iwhen the screen is turned on/off.

*/
```

Intent.ACTION_SCREEN_OFF和Intent.ACTION_SCREEN_ON是不能在AndroidManifest.xml裡面宣告的

[http://hi.baidu.com/hzd2712/item/0877d33760e0c6f7e6bb7a59](http://hi.baidu.com/hzd2712/item/0877d33760e0c6f7e6bb7a59)

screen on： 当按Power键，屏幕亮的时候即为screen on;

screen off:　当按Power键，屏幕黑的时候即为screen off;

unlock (解锁)：当屏幕亮起，移除keyguard的动作；

針對狀態去setLowPowerMode(xxx);

**** ****

### **registerHeadsetListener()**

/**

* Registers an intent to listen for ACTION_HEADSET_PLUG

* notifications. This intent is called to know if the headset

* was plugged in/out

*/

主要用來檢測耳機（含藍牙）插入狀態

#### public static final [String](http://developer.android.com/reference/java/lang/String.html) **ACTION_HEADSET_PLUG**

Added in [API level 1](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#ApiLevels)

Broadcast Action: Wired Headset plugged in or unplugged.

The intent will have the following extra values:

* *state* – 0 for unplugged, 1 for plugged.

* *name* – Headset type, human readable string

* *microphone* – 1 if headset has a microphone, 0 otherwise

沒差耳機就直接冒出提示…接者關閉

**還有檢查 HDMI是否插入**

這裡比較特別的是原生android 並沒有提供hdmi插入的action

WindowManagerPolicy 為隱藏的第三方framework （不是很確定…又好像是在windowmanagerservice下的內千類別）

```
if (action.equals(WindowManagerPolicy.ACTION_HDMI_PLUGGED)) {

//FM should be off when HDMI is connected.
boolean connected = intent.getBooleanExtra(WindowManagerPolicy.EXTRA_HDMI_PLUGGED_STATE, false);
   if (connected) {
      fmClose();
}
```

對應行為為部提示直接關機

**最後ACTION_SHUTDOWN**

**** ****
