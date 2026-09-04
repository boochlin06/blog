# Android Nougat introduction (一)

> 原文網址：https://boochlin.com/?p=180
> 發布日期：2016-09-14
> 文章編號：180

---

![android_n](../../assets/images/image09.png)

[https://boochlin.com/wp-content/uploads/2016/09/image09.png](https://boochlin.com/wp-content/uploads/2016/09/image09.png)

雖然 Android N Preview 從 Google I/O 五月大會就釋放出來，但是一直找不到機子來玩(嗚呼我的 z3 不支援升級)，就一路拖阿托阿托，果然到了 8/22 正式版出來還是沒有機子玩，但是預先 Survey Android 新版的變動，幾乎是系統廠的功課，因為你永遠不知道接下來會不會有要求，要讓以前或是未來的手機通通升上去。因此團隊果然還是派人(Not me)出來研究研究啦，由於 google 的官方更新項目是不斷增長的，且不少東西也沒有詳細的實務經驗，我就只能看到哪寫到哪。以下是簡單介紹。

看完的，有補充資料

1. [Multi-Window Support 多視窗支援](https://developer.android.com/guide/topics/ui/multi-window.html)

2. [Notification Enhancements 通知欄強化](https://android-developers.blogspot.tw/2016/06/notifications-in-android-n.html)

3. [Multi-locale Support, More Languages 多地區設定支援，更多語言](https://developer.android.com/guide/topics/resources/multilingual-support.html)

4. [Doze on the Go 移動時休眠](https://developer.android.com/training/monitoring-device-state/doze-standby.htm)

5. [Project Svelte: Background Optimizations 專案 Svelte：背景最佳化](https://developer.android.com/topic/performance/background-optimization.html)

6. [Data Saver](https://www.youtube.com/watch?v=H-9xKmuwawg)

7. [Quick Settings Tile API 快速設定磚 API](https://developer.android.com/reference/android/service/quicksettings/Tile.html)

8. [Android TV 錄製](https://developer.android.com/training/tv/tif/content-recording.html)

9. [Network Security Configuration 網路安全性強化](https://developer.android.com/training/articles/security-config.html)

10. [Scoped Directory Access 限定範圍目錄存取](https://developer.android.com/training/articles/scoped-directory-access.html)

11. [Virtual Files 虛擬檔案](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=en)

12. [Supporting Direct Boot 直接開機](https://developer.android.com/training/articles/direct-boot.html)

13. [Android for Work Updates](https://www.android.com/work/)

14. [Java 8 Language features JAVA 8 支援](https://developer.android.com/guide/platform/j8-jack.html?hl=en)

15. Profile-guided JIT/AOT Compilation 設定檔指引 JIT/AOT 編譯

底下是還沒細看的（待補充，我決定放到第二篇），一句話結論

1. [Vulkan API](https://developer.android.com/ndk/guides/graphics/index.html) －新的 3D 渲染 api

2. [SurfaceView](https://developer.android.com/reference/android/view/SurfaceView.html) －請大家用 SurfaceView 取代 TextureView ，比較省電

3. [Number Blocking](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=en#print_svc)－系統級的 who’s call

4. [Call Screening](https://developer.android.com/reference/android/telecom/CallScreeningService.html)－系統級的 who’ call

5. [New Emojis](http://unicode.org/emoji/charts/full-emoji-list.html)－更多 Emojis

6. [ICU4J APIs in Android](http://ICU4J APIs in Android) －更小的 size

7. [WebView](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=en#webview)－Chrome + WebView, Together and Multiprocess

8. [OpenGL™ ES 3.2 API](https://developer.android.com/guide/topics/graphics/opengl.html)－更新

9. [Accessibility Enhancements](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService.html) －nope

10. [Key Attestation](https://developer.android.com/training/articles/security-key-attestation.html) －nope

11. [APK Signature Scheme v2](https://developer.android.com/studio/build/build-variants.html#signing)－改善驗證速度並增強完整性保證

12. [Keyboard Shortcuts Helper](http://www.androidpolice.com/2016/05/19/android-n-preview-3-introduces-a-new-shortcuts-helper-screen-for-physical-keyboards/)－**Meta（alt/command） + /** to trigger a *Keyboard Shortcuts* screen

13. [Custom Pointer API](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=en#custom_pointer_api)－自訂游標外觀

14. [Sustained Performance API](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=en#sustained_performance_api)－用來測試 long-running apps

15. [Print Service Enhancements](https://developers.google.com/vr/android/)－nope

16. [FrameMetricsListener API](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=en#framemetrics_api) －monitor its UI rendering performance，not limited to the past 120 frames.

#### [Multi-Window Support 多視窗支援](https://developer.android.com/guide/topics/ui/multi-window.html)

![mw-splitscreen](../../assets/images/mw-splitscreen.png)

[https://boochlin.com/wp-content/uploads/2016/09/mw-splitscreen.png](https://boochlin.com/wp-content/uploads/2016/09/mw-splitscreen.png)

Android N 新增一次顯示多個應用程式的支援。 在手持式裝置上，兩個應用程式可以在「分割畫面」模式中並排或上下排列。 在電視裝置上，應用程式能使用「子母畫面」模式，在使用者與另一個應用程式互動時持續播放影片。

這功能其實之前就有其他大廠做過，我想主因其實很簡單，就是在大螢幕機器上，這樣的操作可以有效利用所有的畫面，是個非常直覺的功能。但是相對的對裝置效能更加要求，尤其是對 RAM，這樣的功能實在很難想像在 256 MB 的手機上可以使用。

至於如何使用，官網是這樣說

使用者可以透過下列方式來切換多視窗模式：

* 如果使用者開啟[總覽畫面](https://developer.android.com/guide/components/recents.html?hl=zh-tw)並長按活動標題，就可以將該標題拖曳到畫面醒目提示的部分，將活動放入多視窗模式。

* 如果使用者長按「總覽」按鈕，裝置會將目前的活動放入多視窗模式，並開啟總覽畫面，讓使用者選擇要分享螢幕的另一個活動。

使用者可以在活動分享螢幕時，將一個活動中的資料[拖放](https://developer.android.com/guide/topics/ui/drag-drop.html?hl=zh-tw)到另一個活動。 (之前，使用者只能在單一活動內拖放資料)。

ps. 總覽畫面就是 recent app 頁面，也就是大家手機按下右下或是左下的方塊按鈕會出現的畫面，老實說我覺得千言萬語比不上一段[影片演示](https://www.youtube.com/watch?v=laHfmVN01FQ)。

看到這邊注意到官網提到的

『Manufacturers of larger devices can choose to enable freeform mode, in which the user can freely resize each activity. If the manufacturer enables this feature, the device offers freeform mode in addition to split-screen mode.』

這個也是文字敘述並不太好理解，簡單來說就是可以像平常操作電腦，可以自由縮放視窗，尤其是在向電視這種超大型畫面上，個人覺得所有做 Android 電視棒的廠商都要好好注意這塊，不過也是需要 app vendor 去支援。

![ap_resize](../../assets/images/ap_resize.png)

[https://boochlin.com/wp-content/uploads/2016/09/ap_resize.png](https://boochlin.com/wp-content/uploads/2016/09/ap_resize.png)

[開發方面就不多說，要注意的東西不少。](https://developer.android.com/guide/topics/ui/multi-window.html)需要注意的是 multi-window activity 的生命週期基本上跟原本一樣，但是仔細想想就知道多視窗操作時就會有其中一個才是 focus 的 window，而沒有 focus 的 window activity 都是處在 onPause 階段，並非 onStop 階段，相對地拿到 focus 的 windows activity 會進入 onResume。

另外就是我很好奇 home app 會是怎麼處理 multi-windows ，之後拿到機子一定要試試看。此外 task queue 的處理也是需要仔細研究。ps. multi-windows mode 下 Android N是可以支援兩個view (兩個Activities之間)交換資訊([multi-window](https://developer.android.com/guide/topics/ui/multi-window.html));實作 [View.OnDragListener](https://developer.android.com/reference/android/view/View.OnDragListener.html)，傳遞 [ClipData](https://developer.android.com/reference/android/content/ClipData.html)。([Drag and Drap](https://developer.android.com/guide/topics/ui/drag-drop.html))

****[而在此項目之下所衍生的就是『Picture in picture(子母畫面)』，](https://boochlin.com/wp-content/uploads/2016/09/pip-active.png)[影片演示](https://www.youtube.com/watch?v=TxAbht2DkyU) ，(TV only)

> 

當使用者從影片返回瀏覽其他內容，您的應用程式可以將影片移入 PIP 模式。

當使用者觀看到影片內容的結尾時，您的應用程式可將影片切換到 PIP 模式。

主畫面顯示系列中下一集的預告或摘要資訊時。您的應用程式可以為使用者提供一個觀賞影片時佇列其他內容的方式。 當主畫面顯示內容選擇活動時，影片繼續在 PIP 模式中播放。

開發上需要注意的基本跟 multi-windows 是一樣的，生命週期，通常進入 onPause 後，影片就會停止播放，因此必須在 onPause 階段作一些處理，讓影片繼續播放，同樣的也必須在 onResume 階段作相對處理。另外需要注意的就是 pip mode 是 240×135 dp。（這很明顯是為了 tv 設計的，可是目前感覺市面上好像還沒有出旗艦款的電視，同時處理子母畫面感覺很吃資源）。

![pip-active](../../assets/images/pip-active.png)

[https://boochlin.com/wp-content/uploads/2016/09/pip-active.png](https://boochlin.com/wp-content/uploads/2016/09/pip-active.png)

#### 

#### [Notification Enhancements 通知欄強化](https://android-developers.blogspot.tw/2016/06/notifications-in-android-n.html)

這部份有以下五大改版

**1. Template updates**: We’re updating notification templates to put a new emphasis on hero image and avatar. Developers will be able to take advantage of the new templates with minimal adjustments in their code.

![new-style](../../assets/images/new-style.png)

[https://boochlin.com/wp-content/uploads/2016/09/new-style.png](https://boochlin.com/wp-content/uploads/2016/09/new-style.png)

**樣板更新**，老實說我覺得沒啥好說，看上面圖最快。使用者可以更加簡單的換成新版的ui

**2. Messaging style customization**: You can customize more of the user interface labels associated with your notifications using the `MessagingStyle `class. You can configure the message, conversation title, and content view.

![messagingstyle](../../assets/images/MessagingStyle.png)

[https://boochlin.com/wp-content/uploads/2016/09/MessagingStyle.png](https://boochlin.com/wp-content/uploads/2016/09/MessagingStyle.png)

**訊息風格客製化**，其實這個很方便，可以根據情況套入不同的 message style ，其實以前就可以用 客製化 notification ui 的方式達到，但這個很明顯 google 多個規範，讓大家更好做事。

**3. [Bundled notifications:](https://www.youtube.com/watch?v=2ZQ1D003g14)** The system can group messages together, for example by message topic, and display the group. A user can take actions, such as Dismiss or Archive, on them in place. If you’ve implemented notifications for Android Wear, you’ll already be familiar with this model.

![ian_n](../../assets/images/Ian_N.png)

[https://boochlin.com/wp-content/uploads/2016/09/Ian_N.png](https://boochlin.com/wp-content/uploads/2016/09/Ian_N.png)

**通知歸群**，這個的中文實在不好翻，看了一下 i/o 影片介紹，個人覺得這個比較符合意義，過去在通知欄裡其實沒啥可以玩得，因為所有的通知都是照時間排序好，彼此之間都是獨立的。但也不是說一定不行，如果有開發 email 的人就會知道有個 style 叫做 inbox ，可以作到類似的事情但是處理實在麻煩。 現在多一個 setGroup() and setGroupSummary() 幫助各位更方便處理這件事情。此外在 api 24 以上 ，系統會自動把4條通知以上歸類成一組，如果怕被系統弄的不好看，請自己先處理吧。

**4. [Direct reply:](https://www.youtube.com/watch?v=Zg4v9G-lku8)** For real-time communication apps, the Android system supports inline replies so that users can quickly respond to an SMS or text message directly within the notification interface.

![messagingstyle](../../assets/images/MessagingStyle.png)

[https://boochlin.com/wp-content/uploads/2016/09/MessagingStyle.png](https://boochlin.com/wp-content/uploads/2016/09/MessagingStyle.png)直接回應，就我所知以往要回覆信件或是 sns， notification 最多能作到的就是利用 pendingIntent 去開啟 activity 進行回應。而這次多了一個新的 api 就是 RemoteInput，配合 message style 可以作到直接在 notificaion 上輸入文字，直接回覆，而不需要離開通知欄。感覺就是為了 Android wear 所設計的，實際上 Android wear 上早已經有了。

**5. Custom views**: Two new APIs enable you to leverage system decorations, such as notification headers and actions, when using custom views in notifications.

**畫面客製化**，主要就是多了兩個 api style，這一樣會需要搭配 message style 會更好。

[Notification.DecoratedCustomViewStyle:](https://developer.android.com/reference/android/app/Notification.DecoratedCustomViewStyle.html) Notification style for custom views that are decorated by the system.

[Notification.DecoratedMediaCustomViewStyle](https://developer.android.com/reference/android/app/Notification.DecoratedMediaCustomViewStyle.html):Notification style for media custom views that are decorated by the system.

總結:這次改動頗大，上次有比較大的改版感覺就是 notification service listener。而這次改版則是讓使用者可以有更多操作在 notification上面（受益最明顯的就是 email 這類 app (gmail)），其實這兩種改版都是為了讓 notification 不再像是潑出去的水一樣，就只是個通知，而是提高使用者，activity和 notification 之間的關聯性。ps.甚至在官方 blog 上提到 Android N 的 N 就是 notification。

 

#### [Multi-locale Support, More Languages 多地區設定支援，更多語言](https://developer.android.com/guide/topics/resources/multilingual-support.html)

Android N 現在可讓使用者在 [設定] 中選取**多個地區設定**，以便以更好的方式支援雙語言使用案例。應用程式可以使用新的 API 取得使用者選取的地區設定，然後為多地區設定使用者提供更精細的使用者體驗 — 例如以多語言顯示搜尋結果，以及不為使用者已經熟知語言的網頁提供翻譯。

簡單來說，過去你只能選擇一種語言作為使用環境，假設是繁體中文，如果 App 並不支援繁體中文那就只能是預設英文，但現在你可以在『設定』裡選擇多個你最習慣的語言與優先權，假設是第一優先是法文（fr_cj），第二是義大利文(it_ch)，如果 App 沒有支援法文，那就會自動找義大利文，如果還是找不到，那才會跳到英文。

對於國籍以及區碼的對應方式也改變。底下用幾張圖表示更加清楚

![android_n_locale_1](../../assets/images/android_n_locale_1.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_1.png](https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_1.png)

**第一張圖：**這是 Android N 之前的作法，當使用者只設定 fr_CH ，但 App Resources 只有 en , de_DE , es_ES , fr_FR , it_IT，那他的比對過程就會是

1. 先確認 fr_CH => 沒有   2. 接者對應國籍 fr => 沒有 3. 最後只好使用預設 en
[https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_2.png](https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_2.png)

![android_n_locale_2](../../assets/images/android_n_locale_2.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_2.png](https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_2.png)

**第二張圖：**這是 Android N 的作法，當使用者只設定 fr_CH ，但 App Resources 只有 en , de_DE , es_ES , fr_FR , it_IT（同上），那他的比對過程就會是

1. 先確認 fr_CH => 沒有    2. 接者對應國籍 fr => 也沒有   3.找找看有沒有 Child 區碼對應到，就是 fr_FR  4.  就決定是 fr_FR

![android_n_locale_3](../../assets/images/android_n_locale_3.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_3.png](https://boochlin.com/wp-content/uploads/2016/09/android_n_locale_3.png)

第三張圖：這是如果設定兩種以上預設語言 P1: fr_CH , P2: it_CH，但 App Resources 只有 en , de_DE , es_ES , it_IT，那他的比對過程就會是

1. 先確認第一優先 fr_CH => 沒有    2. 接者看國籍 fr => 也沒有    3. 找找看有沒有 Child 區碼對應到 => 還是沒有  4. 那開始找 P2:it_CH  => 依然沒有   5. 找國籍 it => 就是沒有   6. 那找找有沒有 Child 區碼對應到，就是 it_IT      7. 就決定是 it_IT

總結：為了加快批配速度，開發者最好能更加精準的給予資源對應。對使用者也是一大方便，有人就是只看的懂 中文和日文，Android 每次沒中文就跳到英文，煩不煩阿

 

#### [Doze on the Go 移動時休眠](https://developer.android.com/training/monitoring-device-state/doze-standby.htm)

![doze-diagram-1](../../assets/images/doze-diagram-1.png)

[https://boochlin.com/wp-content/uploads/2016/09/doze-diagram-1.png](https://boochlin.com/wp-content/uploads/2016/09/doze-diagram-1.png)Android 6.0 引進休眠功能，這是可節省電池電力的系統模式，它會在裝置閒置時 (例如放在桌子上或抽屜中) 延後應用程式的 CPU 與網路活動。

現在 Android N 中的休眠功能更進一步進展，『可在移動時節省電池電力』。只要螢幕關閉一段時間且裝置拔除電源的情況下，休眠功能就會將熟悉的 CPU 與網路限制的子集套用到應用程式（Doze applies a subset of the familiar CPU and network restrictions to apps）。這表示即使使用者將裝置放在口袋內時也可以節省電池電力。

原本Android M 的休眠情況只有以下

* 裝置閒置（不包含移動）

* 螢幕關閉

* 裝置拔除電源

那麼『休眠功能就會將熟悉的 CPU 與網路限制的子集』這句是啥？看中英文都不好理解，再進一步就是底下這些情況

* Network access is suspended.

* The system ignores [wake locks](https://developer.android.com/reference/android/os/PowerManager.WakeLock.html).

* Standard `[AlarmManager](https://developer.android.com/reference/android/app/AlarmManager.html)` alarms (including `[setExact()](https://developer.android.com/reference/android/app/AlarmManager.html#setExact(int, long, android.app.PendingIntent))` and `[setWindow()](https://developer.android.com/reference/android/app/AlarmManager.html#setWindow(int, long, long, android.app.PendingIntent))`) are deferred to the next maintenance window.

* 

  * If you need to set alarms that fire while in Doze, use `[setAndAllowWhileIdle()](https://developer.android.com/reference/android/app/AlarmManager.html#setAndAllowWhileIdle(int, long, android.app.PendingIntent))` or `[setExactAndAllowWhileIdle()](https://developer.android.com/reference/android/app/AlarmManager.html#setExactAndAllowWhileIdle(int, long, android.app.PendingIntent))`.

  * Alarms set with `[setAlarmClock()](https://developer.android.com/reference/android/app/AlarmManager.html#setAlarmClock(android.app.AlarmManager.AlarmClockInfo, android.app.PendingIntent))` continue to fire normally — the system exits Doze shortly before those alarms fire.

* The system does not perform Wi-Fi scans.

* The system does not allow [sync adapters](https://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html) to run.

* The system does not allow `[JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html)` to run

總結：一句話『就算是移動中，沒在用手機我一樣進入休眠模式』，這裡 google 有提供建議 『[Using GCM to Interact with Your App While the Device is Idle](https://developer.android.com/training/monitoring-device-state/doze-standby.html#using_gcm)（we strongly recommend that you **use GCM if possible**, rather than maintaining your own persistent network connection.）』，對於 GCM 的通知，Android 會有系統級的對應，相對好很多。

 

#### [Project Svelte: Background Optimizations 專案 Svelte：背景最佳化](https://developer.android.com/topic/performance/background-optimization.html)

專案 Svelte 一直努力在生態系統中讓各種 Android 裝置上系統與應用程式使用最少的 RAM。在 Android N 中，「專案 Svelte」專注於最佳化應用程式在背景執行的方式。

上面就是一段廢話，重點就是 Android 7.0 we[‘](https://developer.android.com/reference/android/hardware/Camera.html)re removing three commonly-used implicit broadcasts — [CONNECTIVITY_ACTION](https://developer.android.com/reference/android/net/ConnectivityManager.html), [ACTION_NEW_PICTURE](https://developer.android.com/reference/android/hardware/Camera.html), and [ACTION_NEW_VIDEO](https://developer.android.com/reference/android/hardware/Camera.html)— since those can wake the background processes of multiple apps at once and strain memory and battery. If your app is receiving these, take advantage of the Android 7.0 to migrate to **JobScheduler** and related APIs instead.

總結:『從 Android 5.0，我就一直拜託大家快使用 **JobScheduler，可是你們好像又不太愛用，是不是做的不好阿？那我這次下大招了』google 如是說。**

其實就是這三種 action 都有『thundering herd（驚群）』的問題，會造成 action 發出去的那一瞬間，一大堆 service 都醒來，讓 device 變得耗電，且浪費資源，所以這次配合上一段 Doze 來改善這問題。[CONNECTIVITY_ACTION](https://developer.android.com/reference/android/net/ConnectivityManager.html)，在執行中的AP還是可以用[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)向 [Context.registerReceiver()](https://developer.android.com/reference/android/content/Context.html)註冊去聽 CONNECTIVITY_CHANGE。

#### 

#### [Data Saver](https://www.youtube.com/watch?v=H-9xKmuwawg)

![datasaver](../../assets/images/datasaver.png)

[https://boochlin.com/wp-content/uploads/2016/09/datasaver.png](https://boochlin.com/wp-content/uploads/2016/09/datasaver.png)

* Android 7.0 introduces Data Saver mode, a new system service that helps reduce cellular data use by apps. Data Saver gives users control over how apps use cellular data and lets developers provide more efficient service when Data Saver is on.

* When Data Saver is on, the system blocks background data usage and signals apps to use less data in the foreground wherever possible — such as by limiting bit rate for streaming, reducing image quality, deferring optimistic precaching, and so on.

* Android N 擴充 `[ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager.html)` 為應用程式提供[擷取使用者的 Data Saver 喜好設定](https://developer.android.com/preview/features/data-saver.html#status)與[監視喜好設定變更](https://developer.android.com/preview/features/data-saver.html#monitor-changes)的方法。所有應用程式應該檢視使用者是否已啟用 Data Saver 並努力限制前景與背景的數據使用量（期許阿，希望大家能加入以下 code）。

```
ConnectivityManager connMgr = (ConnectivityManager)
        getSystemService(Context.CONNECTIVITY_SERVICE);
// Checks if the device is on a metered network
if (connMgr.isActiveNetworkMetered()) {
  // Checks user’s Data Saver settings.
  switch (connMgr.getRestrictBackgroundStatus()) {
    case RESTRICT_BACKGROUND_STATUS_ENABLED:
    // Background data usage is blocked for this app. Wherever possible,
    // the app should also use less data in the foreground.

    case RESTRICT_BACKGROUND_STATUS_WHITELISTED:
    // The app is whitelisted. Wherever possible,
    // the app should use less data in the foreground and background.

    case RESTRICT_BACKGROUND_STATUS_DISABLED:
    // Data Saver is disabled. Since the device is connected to a
    // metered network, the app should use less data wherever possible.
  }
} else {
  // The device is not on a metered network.
  // Use data as required to perform syncs, downloads, and updates.
}
```

Data saver 對於 app 的處理流程

![datasaver_flow_chart](../../assets/images/datasaver_flow_chart.png)

[https://boochlin.com/wp-content/uploads/2016/09/datasaver_flow_chart.png](https://boochlin.com/wp-content/uploads/2016/09/datasaver_flow_chart.png)

總結：看完[i/o 介紹影片](https://www.youtube.com/watch?v=H-9xKmuwawg)之後，又是一個拜託各位用 JobSchedule 的系統級限制（那女講者表情超有趣）。過去對於 Data usage 設定最多只有針對個別 App 的 cellular networks 開啟或關閉。如果有好心的 App (Google 自家 Youtube如下)，了不起在底下加個 APP SETTINGS 讓你開啟 activity 進行流量限制設定。

![datasaver_m_collection](../../assets/images/datasaver_m_collection.jpg)

[https://boochlin.com/wp-content/uploads/2016/09/datasaver_m_collection.jpg](https://boochlin.com/wp-content/uploads/2016/09/datasaver_m_collection.jpg)

如果是懶得做的 APP(Facebook 夠大廠了吧)，啥鬼設定都沒有。

#### 

#### [Quick Settings Tile API 快速設定磚 API](https://developer.android.com/reference/android/service/quicksettings/Tile.html)

![quicksettings](../../assets/images/quicksettings.png)

[https://boochlin.com/wp-content/uploads/2016/09/quicksettings.png](https://boochlin.com/wp-content/uploads/2016/09/quicksettings.png)

快速設定是直接從通知欄顯示關鍵設定與動作的常用簡單方式。在 Android N 中，我們擴充了快速設定的範圍，讓它變得更實用、更便利。

我們也為額外的快速設定磚增加了更多空間，使用者可以透過向左或向右撥動存取分頁顯示區域。我們也讓使用者能夠控制要顯示的快速設定磚與顯示位置 — 使用者只需拖放磚，即可新增或移除它們。

對於開發人員，Android N 也加入了新的 API，讓您定義自己的快速設定磚，以便使用者輕鬆存取您應用程式中的關鍵控制項與動作。

快速設定磚是專為急需或常用的控制項或動作而保留的，它不應該做為啟動應用程式的捷徑。

定義磚之後，您即可將它們顯示給使用者，使用者只需拖放這些磚，即可將它們新增到快速設定中。

如需建立應用程式磚的相關資訊，請參閱可下載之 [API 參考資料](https://developer.android.com/reference/android/service/quicksettings/package-summary.html)中的`android.service.quicksettings.Tile`。

總結：[直接看影片最清楚](https://www.youtube.com/watch?v=vTwJNlKXuMI)

老實說這東西我開發上也沒有特別玩過，之前都要是裝置開發商才能用，現在終於釋放出來給一般APP 開發者玩 （這樣真的部會出現一大堆在 setting bar 上面嗎？之後可以實驗，一隻 app 到底可以放幾個 tile上去。主要做法須要實作 TileService 去聽 change 且有 permission “android.permission.BIND_QUICK_SETTINGS_TILE”）。（ps. 有機會的話，請我的好同事 J大來談一下這章節，來談談 api 24 之前的 quick settings 開發辛酸史，聽他轉述感覺 google 就是在做他之前的事情啊）

![%e8%9e%a2%e5%b9%95%e5%bf%ab%e7%85%a7-2016-09-14-%e4%b8%8a%e5%8d%8811-12-46](../../assets/images/螢幕快照-2016-09-14-上午11.12.46.png)

[https://boochlin.com/wp-content/uploads/2016/09/螢幕快照-2016-09-14-上午11.12.46.png](https://boochlin.com/wp-content/uploads/2016/09/螢幕快照-2016-09-14-上午11.12.46.png)

 

#### [Android TV 錄製](https://developer.android.com/training/tv/tif/content-recording.html)

![gp-tv-quality](../../assets/images/gp-tv-quality.png)

[https://boochlin.com/wp-content/uploads/2016/09/gp-tv-quality.png](https://boochlin.com/wp-content/uploads/2016/09/gp-tv-quality.png)

Android N 透過新的錄製 API，新增了錄製和播放 Android TV 輸入服務內容的功能。以現有的時間位移 API 為建置基礎，TV 輸入服務可以控制要錄製哪個頻道的資料、如何儲存已錄製的時段，以及管理使用者與錄製內容的互動。

如需詳細資訊，請參閱 [Android TV 錄製 API](https://developer.android.com/preview/features/tv-recording-api.html)。

總結：沒用過也沒玩過，但感覺很方便。歡迎大家提供看法。

 

#### [Network Security Configuration 網路安全性強化](https://developer.android.com/training/articles/security-config.html)

路安全性設定功能，讓應用程式在安全的宣告式設定檔中即可自訂網路安全性設定，而不必修改應用程式的程式碼。 這些設定可以針對特定網域以及針對特定應用程式來設定。主要有以下四點。

這一段真的比較是屬於開發層面，目前還是比較少碰一塊，希望我的理解是對得，也歡迎各位指教

**1. Custom trust anchors:**Customize which Certificate Authorities (CA) are trusted for an app’s secure connections. For example, trusting particular self-signed certificates or restricting the set of public CAs that the app trusts.

設定自憑證 ＆ 限止信任的 CA 組

![android_security_1](../../assets/images/android_security_1.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_security_1.png](https://boochlin.com/wp-content/uploads/2016/09/android_security_1.png)

信任其他 CA

![android_security_2](../../assets/images/android_security_2.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_security_2.png](https://boochlin.com/wp-content/uploads/2016/09/android_security_2.png)

**自訂信任錨點: **其實只看上面的 xml code ，個人很直覺得就是把以前對 ssl(Secure Sockets Layer) 的處理由程式碼轉換到 google xml 的規範化，這確實可以有效降低開發者的苦工。以前的玩法就是在 Android 裡使用 HttpUrlConnection 使用 TrustManager 去處理。事實上大部分開發下不少開發者是不太處理這塊，主要是因為 Android 已經預先把合法發行單位（VeriSign）的憑證都拉進來。而比較特殊的情況則是開發者自己需要連線到自己的 server 作存取，然後又不想使用發行單位發行的憑證，原因其實蠻簡單，就是送去發行單位一年大概要繳費 100 – 500 美金，且大部分 app 會存取的 server 都蠻固定（股票:通常會要求使用個人 CA，信件:通常會自己 Import 公司的 CA….）。

以下為過去的寫法，為了處理自發行憑證，不少 google c/p（就是我） 工程師就會犯一些錯誤，第一段程式碼會信任所有的 CA ，這樣搞的話有 SSL 跟沒 SSL 是一樣的。

```
import org.apache.http.conn.ssl.SSLSocketFactory;
public class MySSLSocketFactory extends SSLSocketFactory {
    SSLContext sslContext = SSLContext.getInstance("TLS");
 
    public MySSLSocketFactory(KeyStore truststore) throws NoSuchAlgorithmException, KeyManagementException, KeyStoreException, UnrecoverableKeyException {
        super(truststore);
 
        TrustManager tm = new X509TrustManager() {
            public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            }
 
            public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            }
 
            public X509Certificate[] getAcceptedIssuers() {
                return null;
            }
        };
 
        sslContext.init(null, new TrustManager[] { tm }, null);
   }
 
    @Override
    public Socket createSocket(Socket socket, String host, int port, boolean autoClose) throws IOException, UnknownHostException {
        return sslContext.getSocketFactory().createSocket(socket, host, port, autoClose);
    }
 
    @Override
    public Socket createSocket() throws IOException {
        return sslContext.getSocketFactory().createSocket();
    }
}
```

而以下這段則是 買一套送全部的典型範例，當憑證接受其中一個網站的 server ，那麼其他的也統統都接受了

```
HostnameVerifier hostnameVerifier = org.apache.http.conn.ssl.SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER;
 
DefaultHttpClient client = new DefaultHttpClient();
 
SchemeRegistry registry = new SchemeRegistry();
SSLSocketFactory socketFactory = SSLSocketFactory.getSocketFactory();
socketFactory.setHostnameVerifier((X509HostnameVerifier) hostnameVerifier);
registry.register(new Scheme("https", socketFactory, 443));
SingleClientConnManager mgr = new SingleClientConnManager(client.getParams(), registry);
DefaultHttpClient httpClient = new DefaultHttpClient(mgr, client.getParams());
 
// Set verifier     
HttpsURLConnection.setDefaultHostnameVerifier(hostnameVerifier);
 
// Example send http request
final String url = "https://www.pchome.com”
HttpPost httpPost = new HttpPost(url);
HttpResponse response = httpClient.execute(httpPost);
 
HttpsURLConnection.setDefaultHostnameVerifier(hostnameVerifier);

HostnameVerifier hostnameVerifier = org.apache.http.conn.ssl.SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER;
```

扯的這邊有點遠了，主要是想把前因後果還有耍笨的故事寫一下。另外不知道有沒有人覺得 raw/my_ca 這樣寫會有問題，如果被反組譯，那這個 CA 憑證不是很危險嗎？根據小組討論，感覺這個 ca 應該單純只是個公鑰，或是只是單純為了範例簡單寫而已  ，歡迎有高手幫忙解答。

**2. Debug-only overrides:**Safely debug secure connections in an app without added risk to the installed base.

![android_security_debug](../../assets/images/android_security_debug.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_security_debug.png](https://boochlin.com/wp-content/uploads/2016/09/android_security_debug.png)

設定 CA 進行偵錯：「只」有在 [android:debuggable](https://developer.android.com/guide/topics/manifest/application-element.html#debug) 為 `true` 時才予以信任。就是實用測試功能，可以把在 code 裡面的 DEBUG 搬出來。因為 play store 採取的安全措施是，不接受標示為可偵錯的應用程式，所以這種方式會比條件式程式碼安全。

**3. Cleartext**** traffic opt-out:**Protect apps from accidental usage of cleartext traffic.

![android_security_cleartext](../../assets/images/android_security_cleartext.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_security_cleartext.png](https://boochlin.com/wp-content/uploads/2016/09/android_security_cleartext.png)

**退出明碼流量**：打算只使用安全連線連線至目的地的應用程式，可以針對那些目的地退出支援明碼 (使用未加密的 HTTP 通訊協定，而非 HTTPS)。 此選項有助於避免應用程式由於外部來源 (例如，後端伺服器) 提供的 URL 中發生變更，而造成意外回復。 如需更多詳細資料，請參閱 `[NetworkSecurityPolicy.isCleartextTrafficPermitted()](https://developer.android.com/reference/android/security/NetworkSecurityPolicy.html?hl=km#isCleartextTrafficPermitted())`。『

This flag is honored on a best effort basis because it’s impossible to prevent all cleartext traffic from Android applications given the level of access provided to them. For example, there’s no expectation that the `[Socket](https://developer.android.com/reference/java/net/Socket.html?hl=km)` API will honor this flag because it cannot determine whether its traffic is in cleartext. However, most network traffic from applications is handled by higher-level network stacks/components which can honor this aspect of the policy.

NOTE: `[WebView](https://developer.android.com/reference/android/webkit/WebView.html?hl=km)` does not honor this flag.』

例如上面，應用程式想要確保 secure.example.com 的所有連線一律要透過 HTTPS 完成，以保護敏感流量不受惡意網路危害。

補充：在 API level 23. 有加入這個 android:usesCleartextTraffic，但這個是所有網域都被限制住，預設值是 true (for example, HTTP and FTP stacks, `[DownloadManager](https://developer.android.com/reference/android/app/DownloadManager.html)`, `[MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer.html)`)

**4. Certificate pinning:**Restrict an app’s secure connection to particular certificates.

**

![android_security_pin](../../assets/images/android_security_pin-1.png)

[https://boochlin.com/wp-content/uploads/2016/09/android_security_pin-1.png](https://boochlin.com/wp-content/uploads/2016/09/android_security_pin-1.png)關聯憑證：**

應用程式一般會信任所有預先安裝的 CA。若這類的任何 CA 意在發行詐騙憑證，應用程式會有遭受 MiTM 攻擊的風險。 有些應用程式選擇透過限制所信任的 CA 組或關聯憑證，來限制可接受的憑證組。

憑證關聯的方法是，透過公用金鑰的雜湊 (X.509 憑證的 SubjectPublicKeyInfo) 來提供一組憑證。 只有當憑證鏈至少包含一個關聯的公用金鑰時，才是有效的憑證鏈。

請注意，使用憑證關聯時，您務必要包括備份金鑰，這樣萬一強制您切換到新的金鑰或變更 CA (關聯到 CA 憑證或該 CA 的中繼者) 時，您的應用程式連線才不會受到影響。 否則，您必須推出應用程式更新，才能還原連線。

總結：這段我只有在開發 mail 時有稍微涉略。沒辦法給出更多實務上的感想。但看得出來 google 從 M 開始就非常注意，但是 M 所在意的則是 device 操作以及使用者資料的權限安全，而 N 則是更加強化網路安全這塊。看起來應該是發生過太多問題啦。

 

#### [Scoped Directory Access 限定範圍目錄存取](https://developer.android.com/training/articles/scoped-directory-access.html)

應用程式 (例如，相片應用程式) 通常只需要存取外部儲存空間中的特定目錄，例如 `Pictures` 目錄。 目前用來存取外部儲存空間的方式並非設計來輕鬆地為這些類型的應用程式提供已設定目標的目錄存取。 例如：

* 在您的宣示說明中要求 `[READ_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)` 或 `[WRITE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)` 可允許存取外部儲存空間ac上的所有公用目錄，但這可能超過您應用程式所需的存取權。

* 使用[儲存空間存取架構](https://developer.android.com/guide/topics/providers/document-provider.html)通常會使得您的使用者透過系統 UI 挑選目錄，這在您的應用程式一律存取相同外部目錄的情況下是不必要的。

Android N 提供新的簡化 API，可用來存取常用外部儲存空間目錄。

```
// 開啟主要共用儲存空間中的 Pictures 目錄
StorageManager sm = (StorageManager)getSystemService(Context.STORAGE_SERVICE);
StorageVolume volume = sm.getPrimaryVolume();
Intent intent = volume.createAccessIntent(Environment.DIRECTORY_PICTURES);
startActivityForResult(intent, request_code);
```

並向使用者確認權限

![%e8%9e%a2%e5%b9%95%e5%bf%ab%e7%85%a7-2016-09-12-%e4%b8%8b%e5%8d%884-57-59](../../assets/images/螢幕快照-2016-09-12-下午4.57.59.png)

[https://boochlin.com/wp-content/uploads/2016/09/螢幕快照-2016-09-12-下午4.57.59.png](https://boochlin.com/wp-content/uploads/2016/09/螢幕快照-2016-09-12-下午4.57.59.png)

存取外部空間則需要，另外宣告

![%e8%9e%a2%e5%b9%95%e5%bf%ab%e7%85%a7-2016-09-12-%e4%b8%8b%e5%8d%885-17-16](../../assets/images/螢幕快照-2016-09-12-下午5.17.16.png)

[https://boochlin.com/wp-content/uploads/2016/09/螢幕快照-2016-09-12-下午5.17.16.png](https://boochlin.com/wp-content/uploads/2016/09/螢幕快照-2016-09-12-下午5.17.16.png)

```
// BroadcastReceiver has already cached the MEDIA_MOUNTED
// notification Intent in mediaMountedIntent
StorageVolume volume = (StorageVolume)
    mediaMountedIntent.getParcelableExtra(StorageVolume.EXTRA_STORAGE_VOLUME);
volume.createAccessIntent(Environment.DIRECTORY_PICTURES);
startActivityForResult(intent, request_code);
```

總結：一直到目前，幾乎所有的 app 開發對於 store access 權限都是全有或是全無，這對使用者會在給予權限感到恐慌。google 也不是沒有想過這問題，就是在 kitkat 時加入的 [Storage Access Framework](https://developer.android.com/guide/topics/providers/document-provider.html) 讓使用者自己挑選資料夾給予存取，但老實說我還真的很少看到會使用 SAF 的 app ，主要還是因為使用者會覺得太麻煩，總和『好麻煩』和『通天的權限』兩個問題， 就希望能取消兩大資料存取權限，然後針對固定通用型的系統資料夾做出簡化操作，改成發 intent 的方式去請系統幫忙保證資料的存取範圍。ps.取得對特定外部目錄的存取權也會取得對該目錄之子目錄的存取權。

#### [Virtual Files 虛擬檔案](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=en)

In previous versions of Android, your app could use the Storage Access Framework to allow users to select files from their cloud storage accounts, such as Google Drive. However, there was no way to represent files that did not have a direct bytecode representation; every file was required to provide an input stream.

Android 7.0 adds the concept of *virtual files* to the Storage Access Framework. The virtual files feature allows your `[DocumentsProvider](https://developer.android.com/reference/android/provider/DocumentsProvider.html)` to return document URIs that can be used with an `[ACTION_VIEW](https://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)` intent even if they don’t have a direct bytecode representation. Android 7.0 also allows you to provide alternate formats for user files, virtual or otherwise.

To get a URI for a virtual document in your app, first you create an `[Intent](https://developer.android.com/reference/android/content/Intent.html)` to open the file picker UI. Since an app cannot directly open a virtual file by using the `[openInputStream()](https://developer.android.com/reference/android/content/ContentResolver.html#openInputStream(android.net.Uri))` method, your app does not receive any virtual files if you include the `[CATEGORY_OPENABLE](https://developer.android.com/reference/android/content/Intent.html#CATEGORY_OPENABLE)` category.

After the user has made a selection, the system calls the `[onActivityResult()](https://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent))` method. Your app can retrieve the URI of the virtual file and get an input stream, as demonstrated in the code snippet below.

師爺來說說這是啥，就是以前雖然可以利用 Storage Access Framework 拿到雲端空間的 URI，但是因為雲端文件沒有在本地端的直接 bytecode 那就無法被顯示出來讓使用者選擇（這段話超難理解，簡單來說就是 `[ACTION_VIEW](https://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)` intent 沒辦法找到這個雲端文間 URI 的檔案），那現在加入 virtual file 的概念，這東西可以被 [DocumentsProvider](https://developer.android.com/reference/android/provider/DocumentsProvider.html) 找到，但是畢竟 virtual file 並不存在於 local ，所以就算現在可以用 [ACTION_VIEW](https://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW) 找到， [openInputStream](https://developer.android.com/reference/android/content/ContentResolver.html#openInputStream(android.net.Uri)) 依然無法存取 byte code.那要怎麼辦呢，我給你 code 你參考參考。

千言萬語不如來段 code

```
// Other Activity code ...

  final static private int REQUEST_CODE = 64;

  // We listen to the OnActivityResult event to respond to the user's selection.
  @Override
  public void onActivityResult(int requestCode, int resultCode,
    Intent resultData) {
      try {
        if (requestCode == REQUEST_CODE &&
            resultCode == Activity.RESULT_OK) {

            Uri uri = null;

            if (resultData != null) {
                uri = resultData.getData();

                ContentResolver resolver = getContentResolver();

                // Before attempting to coerce a file into a MIME type,
                // check to see what alternative MIME types are available to
                // coerce this file into.
                String[] streamTypes =
                  resolver.getStreamTypes(uri, "*/*");

                AssetFileDescriptor descriptor =
                    resolver.openTypedAssetFileDescriptor(
                        uri,
                        streamTypes[0],
                        null);

                // Retrieve a stream to the virtual file.
                InputStream inputStream = descriptor.createInputStream();
            }
        }
      } catch (Exception ex) {
        Log.e("EXCEPTION", "ERROR: ", ex);
      }
  }
```

底下是一般情況，非 virtual file

```
@Override
public void onActivityResult(int requestCode, int resultCode,
        Intent resultData) {

    // The ACTION_OPEN_DOCUMENT intent was sent with the request code
    // READ_REQUEST_CODE. If the request code seen here doesn't match, it's the
    // response to some other intent, and the code below shouldn't run at all.

    if (requestCode == READ_REQUEST_CODE && resultCode == Activity.RESULT_OK) {
        // The document selected by the user won't be returned in the intent.
        // Instead, a URI to that document will be contained in the return intent
        // provided to this method as a parameter.
        // Pull that URI using resultData.getData().
        Uri uri = null;
        if (resultData != null) {
            uri = resultData.getData();
            Log.i(TAG, "Uri: " + uri.toString());
            showImage(uri);
        }
    }
}
```

總結：無感，因為我還是不太理解直接的應用點在哪？而且他的英文描述不太好懂，希望我沒有理解錯。歡迎大家指教。

#### [Supporting Direct Boot 直接開機](https://developer.android.com/training/articles/direct-boot.html)

直接開機可加速裝置啟動時間，並讓已註冊的應用程式只能使用有限的功能 (即使在未預期的重新開機之後)。例如，如果一個加密裝置在使用者睡覺時重新開機，已註冊的鬧鐘、訊息與來電現在可以持續如常通知使用者。這也表示協助工具服務在重新啟動之後可立即使用。

為了達到上述的目的，Android N 會在一個安全的 *直接開機* 模式下執行，這是裝置已經開啟電源但使用者尚未解鎖裝置的期間。 為了支援這種方式，系統為資料提供兩個儲存位置：

* *認證加密的儲存空間*：這是預設的儲存位置，只有在使用者解鎖裝置之後才能使用。

* *裝置加密的儲存空間*：這是「直接開機」模式期間與使用者解鎖裝置之後都可以使用的儲存位置。

這個部分我認為應該和 [file-based encryption (FBE)](http://file-based encryption (FBE)) 一起看，在 Android N 之前全部都是 full-disk encryption ，而這一版改成 FBE ，這直接受益部分就是直接開機。

『File-based encryption enables a new feature introduced in Android 7.0 called [Direct Boot](https://developer.android.com/preview/features/direct-boot.html). Direct Boot allows encrypted devices to boot straight to the lock screen. Previously, on encrypted devices using [full disk encryption](https://source.android.com/security/encryption/full-disk.html) (FDE), users needed to provided credentials before any data could be accessed, preventing the phone from performing all but the most basic of operations。』

由於可以在兩種空間上進行切換，那對於開發者來說需要注意的是 切換點以及兩邊資料整合。先註冊 broadcastReceiver

```
android.intent.action.ACTION_LOCKED_BOOT_COMPLETED
```

Getting Notified of User Unlock

  * If your app has foreground processes that need immediate notification, listen for the [ACTION_USER_UNLOCKED](https://developer.android.com/reference/android/content/Intent.html#ACTION_USER_UNLOCKED) message.

  * If your app only uses background processes that can act on a delayed notification, listen for the [ACTION_BOOT_COMPLETED](https://developer.android.com/reference/android/content/Intent.html#ACTION_BOOT_COMPLETED) message.

Migrating Existing Data

Use Context.moveSharedPreferencesFrom() and Context.moveDatabaseFrom() to migrate preference and database data between credential encrypted storage and device encrypted storage.

```
Context directBootContext = appContext.createDeviceProtectedStorageContext();
// Access appDataFilename that lives in device encrypted storage
FileInputStream inStream = directBootContext.openFileInput(appDataFilename);
// Use inStream to read content...
```

總結：我怎麼覺得 mtk 有個關機鬧鐘，那到底怎麼弄的。這次 Android N 在系統層面的安全性真的強化不少。

#### [Android for Work Updates](https://www.android.com/work/)

來段廣告詞『Android for Work 是協助企業使用 Android 的方案，包括 Android 的產品功能、Google Play for Work 及其他生產力工具。Android for Work 主要透過 API 發佈，可讓企業行動管理服務 (EMM) 供應商和企業應用程式開發商為客戶員工提供安全、有效率而多元化的行動環境。

* **Work profile security challenge**

* 設定檔擁有者可以為以工作設定檔執行之應用程式指定個別的安全性查問。當使用者嘗試開啟任何工作應用程式時會顯示工作查問。成功完成安全性查問可將工作設定檔解鎖並在必要時將它解密。

* **Toggle Work Mode**

* 在具有工作設定檔的裝置上，使用者可以切換工作模式。當工作模式關閉時，受管理的使用者會暫時關機，因此而停用工作設定檔應用程式、背景同步與通知。這也包括設定檔擁有者應用程式。當工作模式關閉時，系統會顯示持續的狀態圖示，提醒使用者他們無法啟動工作應用程式。啟動器會指出工作應用程式和小工具無法存取。

* **Always-On VPN**

* 裝置擁有者和設定檔擁有者可確保工作應用程式一律透過指定的 VPN 連線。系統會自動在裝置開機時啟動該 VPN。新的 `DevicePolicyManager` 方法是 `setAlwaysOnVpnPackage()` 與 `getAlwaysOnVpnPackage()`。因為系統無需透過應用程式互動即可直接連結 VPN 服務，所以 VPN 用戶端需要為「一律開啟的 VPN」處理新的進入點。正如以往，透過符合動作`android.net.VpnService` 的意圖篩選器向系統指明服務。使用者也可以使用 [設定] > [更多] > [VPN] 手動設定「一律開啟的 VPN」用戶端，這些用戶端在主要使用者中實作 `VPNService` 方法。

* **Customized provisioning**

* An application can customize the profile owner and device owner provisioning flows with corporate colors and logos. `DevicePolicyManager.EXTRA_PROVISIONING_MAIN_COLOR` customizes flow color. `DevicePolicyManager.EXTRA_PROVISIONING_LOGO_URI`customizes the flow with a corporate logo.

* 使用者可以自己定義驗證流程的主要顏色，或是放置公司 logo 。。

總結：這個我真的連玩都沒玩過，更別說開發。感覺 always-on vpn 還蠻實用的。google 真的越來越重視 BYOD，除了區分工作與個人，安全性更是一大重點。我在看 api 時發現可以直接讀取公司的 calendar, email user contact。感覺很好用啊。

#### [Java 8 Language features JAVA 8 支援](https://developer.android.com/guide/platform/j8-jack.html?hl=en)

Android does not currently support all Java 8 language features. However, the following features are now available when developing apps targeting the Android N Preview:

須使用 Android Studio 2.1 (預覽版) 與 Android N Preview SDK，其中包括必要的 Jack tool chain 與適用於 Gradle 的已更新 Android 外掛程式。

  * [Default and static interface methods](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)

  * [Lambda expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) (also available on API level 23 and lower)

  * [Repeatable annotations](https://docs.oracle.com/javase/tutorial/java/annotations/repeating.html)

  * [Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html) (also available on API level 23 and lower)

  * [Type Annotations](https://docs.oracle.com/javase/tutorial/java/annotations/type_annotations.html) (also available on API level 23 and lower)

總結：千呼萬喚始出來，第二項的 Lambda 這東西從好久之前就不少人很想拉進 Android，還弄個 Retrolambda 來嚐鮮，不過我個人用的不太習慣啊。此外，都已經提到 Java 8 ，那怎麼可以不提 [Jack (Java Android Compiler Kit )](https://source.android.com/source/jack.html)and Jill (Jack Intermediate Library Linker)，但目前看起來效能差異不大，最直接受益就是沒有 『65k method limit』 問題。

* Legacy javac toolchain:

**javac** (`.java` → `.class`) → **dx** (`.class` → `.dex`)

* New Jack toolchain:

**Jack** (`.java` → `.jack` → `.dex`)

然後這裡有個大神寫得非常好，[Android 新一代编译 toolchain Jack & Jill 简介](http://taobaofed.org/blog/2016/05/05/new-compiler-for-android/)

#### 

#### Profile-guided JIT/AOT Compilation 設定檔指引 JIT/AOT 編譯

在 Android N 中，我們新增了 Just in Time (JIT) 編譯器搭配程式碼分析工具到 ART，讓 Android 應用程式在執行時能夠持續改善其效能。JIT 編譯器補充了 ART 目前的 Ahead of Time (AOT) 編譯器，協助改善執行階段效能、節省儲存空間以及加速應用程式更新和系統更新。

設定檔指引編譯讓 ART 根據每個應用程式的實際用情形與裝置上的情況來管理其 AOT/JIT 編譯。例如，ART 會維護每個應用程式常用方法的設定檔，而且可以預先編譯和快取那些方法以獲得最佳效能。它不會編譯應用程式的其他部分，直到實際要使用這些部分時才會編譯。

除了改善應用程式關鍵部分的效能以外，設定檔指引編譯還有助於降低應用程式的整體 RAM 使用量，包括關聯的二進位檔案。此功能對於低記憶體裝置特別重要。

ART 透過對裝置電池產生最小影響的方式來管理設定檔指引編譯。它只會在裝置閒置和充電時預先編譯，這種預先工作的方式可以節省時間和電池電力。

總結：Android M 的時候，只有在 apk 安裝時才會編譯。這個說來慚愧，敝司曾經出過一隻超低階手機（512mb 還是 256 mb），系統更新慢到以為當機，但是等個２０分鐘，還是成功了，但是好戲還在後頭，居然不能安裝 facebook app , 這還能賣嗎？就是因為安裝時編譯消耗大量 RAM ，但是手機太爛了，就當機了，就當機了，就當機了。我想可能就是上面這些問題，所以還是退回部分先編譯這樣折衷的方式。

 

第一篇就到此為止，剩下的會在第二篇補完(希望)，如果有任何問題或是我寫錯的部分，歡迎底下回應。有興趣且有機子的人可以利用 [Google Sample](https://developer.android.com/about/versions/nougat/android-7.0-samples.html)進行測試。

reference:

[官方](https://developer.android.com/about/versions/nougat/android-7.0.html)

[官方影片](https://www.youtube.com/channel/UCVHFbqXqoYvEWM1Ddxl0QDg)

FB 上的 [Scott Tsai](https://www.facebook.com/scottt.tw) 的[感想](https://www.facebook.com/groups/system.software2016/permalink/1097245500354954/)

[为你的安卓应用实现自签名的 SSL 证书](http://www.oschina.net/translate/android-security-implementation-of-self-signed-ssl)

[从开发者角度解析 Android N 新特性！](https://gank.io/post/56e0b83c67765963436fcb94)

Android M新特性Doze and App Standby模式详解 – 不能轉載

[Android 6.0电源管理方式](https://my.oschina.net/lorcan/blog/539208)

[Android N的五項全新功能（加上分屏）](https://read01.com/RM3RQ0.html)
