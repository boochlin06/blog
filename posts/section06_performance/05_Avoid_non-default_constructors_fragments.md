# React native Native android module – my god , many issue.

> 原文網址：https://boochlin.com/?p=150
> 發布日期：2016-07-27
> 文章編號：150

---

### 

![螢幕快照 2016-07-27 下午5.40.34](../../assets/images/螢幕快照-2016-07-27-下午5.40.34.png)

[https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午5.40.34.png](https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午5.40.34.png)

### React native 第二彈 之 媽的這坑好多

目前遇到的坑真心不少，每次一個改版都是一場浩劫。但是官方又大概維持兩週一次的改版，第三方又不一定那麼勤勞。那爆掉的情況就不少啦。相信不少同仁都花不少時間一起在幫忙第三方開源 lib debug ,但我認為這就是開源軟體最大的優點。下面就隨手筆記一下。

### 1. [Native module Android 範例](https://facebook.github.io/react-native/docs/native-modules-android.html)

先說明情境，想寫個根據 package name 去抓 app label name。

react native – android

```
@ReactMethod
    public void getLabel(
            String packageName,
            Callback errorCallback,
            Callback successCallback) {
        SharedPreferences spref = this.reactContext.
                getSharedPreferences(this.reactContext.getPackageName() + "_preferences", Context.MODE_PRIVATE);
        Log.i(TAG, "getLabelInvoke package :"+ packageName + " isChecked=" + spref.getBoolean(packageName, false));
        try {
            String appName = (String) reactContext.getPackageManager().getApplicationLabel(
                    reactContext.getPackageManager().getApplicationInfo(packageName, PackageManager.GET_META_DATA));
            successCallback.invoke(appName,spref.getBoolean(packageName, spref.getBoolean(packageName, false)));
        } catch (PackageManager.NameNotFoundException e) {
            errorCallback.invoke(e.getMessage());
        }
    }
    @ReactMethod
    public void getLabel1(
            String packageName,
            Promise promise) {
        SharedPreferences spref = this.reactContext.
                getSharedPreferences(this.reactContext.getPackageName() + "_preferences", Context.MODE_PRIVATE);
        Log.i(TAG, "getLabelInvoke package :"+ packageName + " isChecked=" + spref.getBoolean(packageName, false));
        try {
            String appName = (String) reactContext.getPackageManager().getApplicationLabel(
                    reactContext.getPackageManager().getApplicationInfo(packageName, PackageManager.GET_META_DATA));

            WritableMap map = Arguments.createMap();
            map.putString("label",appName);
            map.putBoolean("enable",spref.getBoolean(packageName, false));
            promise.resolve(map);
        } catch (PackageManager.NameNotFoundException e) {
            promise.reject(e);
        }
    }
```

react native – js

```
};
  fetchAppLabel(packageName: string) {
    self = this;
    MessageListenerModule.getLabel(packageName,
      (msg: string) => {
        console.log('Fail to get labe , package:', msg);
      },
      (appLabel: string, enable: boolean) => {
        console.log('getAppLabel app :', packageName, ':', appLabel, ':', enable);
       }
    );
  }
  async fetchAppLabel1(packageName: string) {
    try {
      let {
        label,
        enable,
      } = await MessageListenerModule.getLabel1(packageName);
      console.log('package:', packageName, ' label:', label);
    } catch (e) {
      console.log(e);
    }
  }
```

### 2. [react-native-router-flux](https://github.com/aksonov/react-native-router-flux)  sub scene

裡面的 [sub scene](https://github.com/aksonov/react-native-router-flux/blob/master/docs/OTHER_INFO.md) 使用方法超難懂。 不知道有沒有人在用 drawer 搭配多層 scene ，達到 drawer 上切換主類別，然後進入個子項目裡，按照範例的寫法，每次切換drawer 選項，都會失敗，如果有子分頁的話。以下面例子來說，每次只要在 drawer 上切換到 device config 頁面，你會發現外表是 device config 的 list page，但是內心他自己會以為自己是 settings about 子分頁，因此，你不管怎麼切換都不會成功。這很明顯不是我們要的。原因我也不知道。
[https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午3.36.30.png](https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午3.36.30.png)

![Screenshot_20160727-155517](../../assets/images/Screenshot_20160727-155517.png)

[https://boochlin.com/wp-content/uploads/2016/07/Screenshot_20160727-155517.png](https://boochlin.com/wp-content/uploads/2016/07/Screenshot_20160727-155517.png)

![螢幕快照 2016-07-27 下午3.36.30](../../assets/images/螢幕快照-2016-07-27-下午3.36.30.png)

後來發現解法，就是順他的毛做，加一個 main 最為子選項的第一項。

![螢幕快照 2016-07-27 下午3.43.15](../../assets/images/螢幕快照-2016-07-27-下午3.43.15.png)

[https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午3.43.15.png](https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午3.43.15.png)

### 3. render right button

呈上，很多情況下，你會想要自己客製化 router 上的 right button，像是上面的程式碼中的 renderRightButton ，內容如下，要特別記住 renderRightButton 的範圍是 router 的左上方為起始點，儘管它的名稱叫做 renderRightButton。

![Screenshot_20160727-155634](../../assets/images/Screenshot_20160727-155634.png)

[https://boochlin.com/wp-content/uploads/2016/07/Screenshot_20160727-155634.png](https://boochlin.com/wp-content/uploads/2016/07/Screenshot_20160727-155634.png)
[https://boochlin.com/wp-content/uploads/2016/07/Screenshot_20160727-155634.png](https://boochlin.com/wp-content/uploads/2016/07/Screenshot_20160727-155634.png)

![螢幕快照 2016-07-27 下午3.49.53](../../assets/images/螢幕快照-2016-07-27-下午3.49.53.png)

為了讓他變成是 right button ，因此需要先設定 justifyContent : flex-end ，才會從右邊開始算，但是新的問題就發生了，就是你的 back button 點不下去了，這是因為 right button layout 的範圍蓋在 back button 上面，導致按不到，因此必須空出一個範圍給 back key button. 我的做法就是設定 marginLeft。

![螢幕快照 2016-07-27 下午3.51.28](../../assets/images/螢幕快照-2016-07-27-下午3.51.28.png)

[https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午3.51.28.png](https://boochlin.com/wp-content/uploads/2016/07/螢幕快照-2016-07-27-下午3.51.28.png)

### 4. React native 的 phone call listener error

想要監聽 phone call 進來，然後傳到 js level， code 如下

EventReceiver.java

```
public class EventReceiver extends BroadcastReceiver {
    private static final String TAG = "EventReceiver";

    private static boolean incomingFlag = false;
    private static String incoming_number = null;

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals(TelephonyManager.ACTION_PHONE_STATE_CHANGED)) {
            TelephonyManager tm = (TelephonyManager) context.getSystemService(Service.TELEPHONY_SERVICE);

            switch (tm.getCallState()) {
                case TelephonyManager.CALL_STATE_RINGING:
                    incomingFlag = true;
                    incoming_number = intent.getStringExtra("incoming_number");
                    Log.i(TAG, "RINGING :" + incoming_number);
                    sendIncallNotification(context, true, incoming_number);
                    break;
                case TelephonyManager.CALL_STATE_OFFHOOK:
                    if (incomingFlag) {
                        Log.i(TAG, "incoming ACCEPT :" + incoming_number);
                        sendIncallNotification(context, false, incoming_number);
                    }
                    break;

                case TelephonyManager.CALL_STATE_IDLE:
                    if (incomingFlag) {
                        Log.i(TAG, "incoming IDLE");
                        sendIncallNotification(context, false, incoming_number);
                    }
                    break;
            }
        } else if (intent.getAction().equals(Telephony.Sms.Intents.SMS_RECEIVED_ACTION)) {
            Log.i(TAG, "Receive a new message");
            Intent serviceIntent = new Intent();
            serviceIntent.setClass(context, NListenerService.class);
            serviceIntent.setAction(NListenerService.SEND_NOTIFICATION_SMS);
            context.startService(serviceIntent);
        } else if (intent.getAction().equals(CalendarContract.ACTION_EVENT_REMINDER)) {
            Log.i(TAG, "Receive a calendar event");
            Intent serviceIntent = new Intent();
            serviceIntent.setClass(context, NListenerService.class);
            serviceIntent.setAction(NListenerService.SEND_NOTIFICATION_EVENT);
            context.startService(serviceIntent);
        }
    }

    private void sendIncallNotification(Context context, boolean stauts, String phoneNumber) {
        Intent serviceIntent = new Intent();
        serviceIntent.setClass(context, NListenerService.class);
        serviceIntent.setAction(NListenerService.SEND_NOTIFICATION_CALL);
        serviceIntent.putExtra("status", stauts);
        serviceIntent.putExtra("phoneNumber", phoneNumber);
        context.startService(serviceIntent);
    }
}

NListenerService.java
```

```
public class NListenerService extends NotificationListenerService {
    private String TAG = "NListenerService";
    public final static String SEND_NOTIFICATION_SMS = "com.android.notification.sendsms";
    public final static String SEND_NOTIFICATION_EVENT = "com.android.notification.event";
    public final static String SEND_NOTIFICATION_CALL = "com.android.notification.incomingcall";
    private String mPhoneNumber = "";


    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        if (intent == null || intent.getAction() == null) {
            return START_STICKY;
        }

        if (intent.getAction().equals(SEND_NOTIFICATION_SMS)) {
            sendSmsNotification();
        }

        if (intent.getAction().equals(SEND_NOTIFICATION_EVENT)) {
            sendEventNotification();
        }

        if (intent.getAction().equals(SEND_NOTIFICATION_CALL)) {
            sendCallNotification(intent.getBooleanExtra("status", false), intent.getStringExtra("phoneNumber"));
            mPhoneNumber = intent.getStringExtra("phoneNumber");
        }
        // We want this service to continue running until it is explicitly
        // stopped, so return sticky.
        return START_STICKY;
    }

    private void sendCallNotification(boolean b, String phoneNumber) {
        SharedPreferences spref =
                getSharedPreferences(getPackageName() + "_preferences", Context.MODE_PRIVATE);
        Log.i(TAG, "sendCallNotification:" + b);
        Log.i(TAG, "spref:" + spref.getBoolean(MessageListenerModule.PHONE_CALL_KEY, true));
        if (spref.getBoolean(MessageListenerModule.PHONE_CALL_KEY, true)) {

            if (phoneNumber != null) {
                // TODO : Issue , if app is dismiss from system.
                // Facebook arch cant init quickly when phone call, will cause error.
                WritableMap params = Arguments.createMap();
                params.putString("phoneNumber", phoneNumber);
                sendEventToJS(MessageListenerModule.JS_EVENT_PHONE_CALL, params);
            }
        }
    }

    private void sendEventNotification() {
        SharedPreferences spref =
                getSharedPreferences(getPackageName() + "_preferences", Context.MODE_PRIVATE);
        if (spref.getBoolean(MessageListenerModule.CALENDAR_KEY, true)) {

            WritableMap params = Arguments.createMap();
            sendEventToJS(MessageListenerModule.JS_EVENT_CALENDAR, params);
        }
    }

    @TargetApi(Build.VERSION_CODES.KITKAT)
    private void sendSmsNotification() {
        SharedPreferences spref =
                getSharedPreferences(this.getPackageName() + "_preferences", Context.MODE_PRIVATE);
        if (spref.getBoolean(Telephony.Sms.getDefaultSmsPackage(this), false)) {
            WritableMap params = Arguments.createMap();
            params.putString("package", Telephony.Sms.getDefaultSmsPackage(this));
            sendEventToJS(MessageListenerModule.JS_EVENT_SMS, params);
        }
    }

    @TargetApi(Build.VERSION_CODES.KITKAT)
    @Override
    public void onNotificationPosted(StatusBarNotification sbn) {
        Log.i(TAG, "onNotificationPosted:" + sbn.getPackageName());

        String packageName = sbn.getPackageName();
        SharedPreferences spref =
                getSharedPreferences(this.getPackageName() + "_preferences", Context.MODE_PRIVATE);
        Log.i(TAG, "onNotificationPosted:" + "isChecked=" + spref.getBoolean(packageName, false));
        if (spref.getBoolean(packageName, false) || packageName.equals(Telephony.Sms.getDefaultSmsPackage(this))) {
            WritableMap params = Arguments.createMap();
            params.putString("package", packageName);
            sendEventToJS(MessageListenerModule.JS_EVENT_NOTIFICATION, params);
        }
    }

    @Override
    public void onNotificationRemoved(StatusBarNotification sbn) {
        Log.i(TAG, "********** onNOtificationRemoved:" + sbn.getPackageName());
    }

    private void sendEventToJS(String eventName,
                               @Nullable WritableMap params) {
        ReactContext reactContext = ((CybertoolApplication) this.getApplicationContext()).getReactContext();
        reactContext
                .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
                .emit(eventName, params);
    }
}
```

這程式碼沒有問題，只要 app 沒有被 kill 過，如果 app 被移出 memory ， 那麼在 service 開始處理 phone call event 就會拋出 error ，主要就是phone call event 是屬於必須及時處理的 event ， 但是這時候 app 被 kill ，那麼系統必須馬上 init android service ，到這階段沒有問題，但是 react native 的 module 卻無法來得及。

```
java.lang.ExceptionInInitializerError
	at com.facebook.react.bridge.ReactBridge.staticInit(ReactBridge.java:39)
	at com.facebook.react.bridge.NativeMap.(NativeMap.java:22)
	at com.facebook.react.bridge.Arguments.createMap(Arguments.java:29)
	at com.acer.android.message.NListenerService.sendCallNotification(NListenerService.java:72)
	at com.acer.android.message.NListenerService.onStartCommand(NListenerService.java:54)
	at android.app.ActivityThread.handleServiceArgs(ActivityThread.java:3067)
	at android.app.ActivityThread.access$2200(ActivityThread.java:154)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1489)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:224)
	at android.app.ActivityThread.main(ActivityThread.java:5526)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:726)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:616)
Caused by: java.lang.RuntimeException: SoLoader.init() not yet called
	at com.facebook.soloader.SoLoader.assertInitialized(SoLoader.java:234)
	at com.facebook.soloader.SoLoader.loadLibrary(SoLoader.java:169)
	at com.facebook.react.bridge.ReactBridge.staticInit(ReactBridge.java:39)
	at com.facebook.react.bridge.ReactBridge.(ReactBridge.java:31)
```

解法來看，目前沒有？等官方？不然就是都在 android level 解決。

感謝收看，歡迎分享
