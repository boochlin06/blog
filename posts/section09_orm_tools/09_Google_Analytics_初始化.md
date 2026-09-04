# Android google analyze 使用教學(一) 初始化

> 原文網址：https://boochlin.com/?p=129
> 發布日期：2016-04-25
> 文章編號：129

---

![google analyze](https://www.google.com/intl/zh-TW_tw/analytics/images/feat-hed-mobile.png)

『Google Analytics (分析) 可讓您評估行動網站、應用程式，以及可上網的行動裝置 (高階和基本手機/平板電腦都算在內) 所帶來的訪客流量。我們為行銷人員提供了與客戶相關的分析數據，協助他們掌握關鍵時刻，在各種螢幕和裝置上成功創造業績。您可以開始製作目標明確且成效良好的行銷廣告活動，隨時隨地接觸訪客。』

老實說還有啥工具可以幫忙統計行動裝置相關數據的，我不知道，但至少 GA 用起來手感還不錯，重點一般版是不用錢的，這也是推給老闆的最後且最強一根稻草。至於 Google Analytics Premium，我想那價格會讓你嚇到尿出來，當然功能也超強。

借用別人做好的圖，一般版跟進階版到底差在哪裡

reference : [http://blog.wis.com.tw/2013/steven/gaversion/](http://blog.wis.com.tw/2013/steven/gaversion/)

![版本差異-680x369](../../assets/images/版本差異-680x369.png)

[https://boochlin.com/wp-content/uploads/2016/04/版本差異-680x369.png](https://boochlin.com/wp-content/uploads/2016/04/版本差異-680x369.png)

還有一個差異沒有畫出來，就是一般版只要０元，但是進階版要每年十五萬美元，這可能要超大型app才有機會啦，至少是要能每年賺上十五萬美元的高利潤服務，結案。

前面掰一堆只是要告訴你最後我是使用一般版的ga，接下來才是使用心得及教學。

由於開始使用在新的案子時就已經是 google analyze v4.0 了，所以當然我只會介紹最新版 v4 的使用方法，但是老實說我覺得自己看 GA [官方資料](https://developers.google.com/analytics/devguides/collection/android/v4/)也很足夠，google 已經詳細到很佛心了。

１．設定 GA 到 project

GA 是會使用到網路的(好像廢話)，所以第一步就是在 manifest 裡要求網路存取權限

![ga_manifest](../../assets/images/ga_manifest.png)

[https://boochlin.com/wp-content/uploads/2016/04/ga_manifest.png](https://boochlin.com/wp-content/uploads/2016/04/ga_manifest.png)

然後在 Gradle 加入底下的 code 就可以拿到 GA lib

```
compile 'com.google.android.gms:play-services-analytics:8.4.0'
```

一開始的程式碼就到這，剩下要去跟 [google申請]( https://www.google.com/analytics/) GA 相關資料，才能繼續使用，首先你的 Google 帳號裡要有一隻 app 的 project (可以還沒有發佈)，才能[一步一步](https://developers.google.com/mobile/add?platform=android&cntapi=analytics&cnturl=https:%2F%2Fdevelopers.google.com%2Fanalytics%2Fdevguides%2Fcollection%2Fandroid%2Fv4%2Fapp%3Fconfigured%3Dtrue&cntlbl=Continue%20Adding%20Analytics)([多圖操作教學](http://rx1226.pixnet.net/blog/post/290552098-%5Bandroid%5D-3-12-google-analytics))的取得 google-services.json ，這裡面會記載者 project 和 ap 的相關組態值（ project_id , project_number之類的東西，提供給 google play service 讀取參數，用以連接 GA web service），最後把這個檔案放進你的ap底下就可以開始使用啦(注意要放對位子，才能讓 google play service 讀取到，通常是 project/app 下)。

接者根據官方範例。開始做一個全域 google tracker 掛在 Application 底下。

```
/*
 * Copyright Google Inc. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.google.samples.quickstart.analytics;

import android.app.Application;

import com.google.android.gms.analytics.GoogleAnalytics;
import com.google.android.gms.analytics.Tracker;

/**
 * This is a subclass of {@link Application} used to provide shared objects for this app, such as
 * the {@link Tracker}.
 */
public class AnalyticsApplication extends Application {
  private Tracker mTracker;

  /**
   * Gets the default {@link Tracker} for this {@link Application}.
   * @return tracker
   */
  synchronized public Tracker getDefaultTracker() {
    if (mTracker == null) {
      GoogleAnalytics analytics = GoogleAnalytics.getInstance(this);
      // To enable debug logging use: adb shell setprop log.tag.GAv4 DEBUG
      mTracker = analytics.newTracker(R.xml.global_tracker);
    }
    return mTracker;
  }
}
```

但是如果造上面做的話，會發現缺少 R.xml.global_tracker 這個檔案，而且還是在創造新 tracker 一定會用到的檔案。這到底是拿來做啥呢？

global_tracker.xml

![ga_config](../../assets/images/ga_config.png)

[https://boochlin.com/wp-content/uploads/2016/04/ga_config.png](https://boochlin.com/wp-content/uploads/2016/04/ga_config.png)

其實只是很單純的組態設定檔，如果你沒有啥特別想設定的話，那就只要留下最基本的 ga_trackingId （[如何取得](https://analytics.google.com/analytics/web/)，先到後台管理頁面，點擊上方”管理“ -> ”追蹤資訊“ -> ”追蹤程式碼“ -> ”追蹤編號“）就可以啟用，甚至可以不用寫這 xml 檔(記得要放 project/xml 下)，只要把上述程式碼改一下就可以。如果想瞭解上面的參數設定意義，可以看[這裡](https://developers.google.com/analytics/devguides/collection/android/v4/parameters#parameters) 或 [中文](http://blog.csdn.net/figo0423/article/details/47666315)

```
mTracker = analytics.newTracker(R.xml.global_tracker);
// modify to
mTracker = analytics.newTracker("UA-0000-1");
```

其實看到上面程式碼，是採用工廠模式去產生 tracker ，就可以發現 GA 是容許多個 tacker 共存在同一隻 app 底下，不同的 tracker id 對應到不同 GA 帳戶，也就是說針對不同的行為，發送到不同的 GA 帳戶，這功能我一開始還想不透有啥好處，因為分開存紀錄的話，就無法利用 GA 強大的報表功能，一口氣拿全部的資料來處理，變成要到處拿帳號來匯出匯入，但是看到[這篇](http://ascii-iicsa.blogspot.tw/2014/04/google-analytics-for-android.html)，有點恍然大悟，可以對重要性比較高的商業行為 event 做獨立的紀錄，提高或降低sample rate , 或是 發送間隔時間，指定輸出特定的 log。

這邊提供一個簡單的範例，多個 tracker 的意義就就看得出來，當你做產品到一個階段，想先讓部分公司內部受測者先用用看，但是不想因此打亂原本 GA 帳戶的內容，就可以先導向到 test GA 帳戶，這樣的概念同時也可以應用到只是想分開 phone 跟 tablet 的操作導向，這有可能是因為 phone and tablet 的設計差太多，不想混在一起看。

```
public final class GoogleAnalyticsTrackers {

    public enum Target {
        APP,
        TEST
        // Add more trackers here if you need, and update the code in #get(Target) below
    }

    private static GoogleAnalyticsTrackers sInstance;

    public static synchronized void initialize(Context context) {
        if (sInstance != null) {
            throw new IllegalStateException("Extra call to initialize analytics trackers");
        }

        sInstance = new GoogleAnalyticsTrackers(context);
    }

    public static synchronized GoogleAnalyticsTrackers getInstance() {
        if (sInstance == null) {
            throw new IllegalStateException("Call initialize() before getInstance()");
        }

        return sInstance;
    }

    private final Map<Target, Tracker> mTrackers = new HashMap<Target, Tracker>();
    private final Context mContext;

    /**
     * Don't instantiate directly - use {@link #getInstance()} instead.
     */
    private GoogleAnalyticsTrackers(Context context) {
        mContext = context.getApplicationContext();
    }

    public synchronized Tracker get(Target target) {
        if (!mTrackers.containsKey(target)) {
            Tracker tracker;
            switch (target) {
                case APP:
                    tracker = GoogleAnalytics.getInstance(mContext).newTracker(R.xml.app_tracker);
                    break;
                case TEST:
                    tracker = GoogleAnalytics.getInstance(mContext).newTracker(R.xml.test_tracker);
                default:
                    throw new IllegalArgumentException("Unhandled analytics target " + target);
            }
            mTrackers.put(target, tracker);
        }

        return mTrackers.get(target);
    }
}
```

以上就是初始化的部分，到這裡喘口氣，接下來是個項行為的紀錄操作

——喘口氣，寫到真的有累——

通常我們 app 第一個想記錄的就是各頁面的瀏覽行為，以及使用的操作習慣，如果 app 是以 activity 作為畫面切換的主體的話，可以直接在上述的 xml 設定檔加入 ga_autoActivityTracking 為 true

或是

```
tracker.enableAutoActivityTracking(true);
```

就會自動記錄頁面的切換，頁面名稱會是 activity 的 component name, 但是目前會全部用 activity 當作頁面的app 根本沒幾隻，通常會有很多自定義 view (tab view , page switcher) ， fragment 作為頁面操作的載體，而這類情況則需要開發者自己去掌握切換點，以及名稱

```
Log.i(TAG, "Setting screen name: " + name);
mTracker.setScreenName("Image~" + name);
mTracker.send(new HitBuilders.ScreenViewBuilder().build());
```

通常發送的時間點就在切換的時候，舉例來說就是 onTabChanged() , onPageSwitch() ，onFragmentTransaction() 這類函數裡面。所以通常我都把 auto activity track 關掉，然後自己到處埋 code ，主要會在頁面切換還有 activity 的 onResume() 記錄。

記錄這個到底有何好處，主要可以看出個頁面的瀏覽量，時間以及使用者的操作習慣，有可能九成的使用者看完首頁就跳出 app 了，並沒有造流程引導到我們想要讓使用者看到的頁面（這可能是廣告，有可能是付費的網頁），那對開發者的收益就很低，因此這是一個修正 app 行為的良好指標。

ga 上面的行為流程圖

![flow](../../assets/images/flow.png)

[https://boochlin.com/wp-content/uploads/2016/04/flow.png](https://boochlin.com/wp-content/uploads/2016/04/flow.png)

除了頁面記錄以外，剩下的就是開發者自己在意的東西，這一類我們通通用 event 一概稱之，這個的定義就可以很廣啦，像是 button 點擊，back 事件，只要你想得到的通通可以用者個，以官方提供的用法來說，主要有四個參數- 分類 – 行為 – 標籤 – 價值。雖然這四個參數有對應的意義，但是基本上是隨便你要塞啥進去都可以，當然要讓他變得有意義，而且會對應在 ga 後台的報告中。

```
mTracker.send(new HitBuilders.EventBuilder()
    .setCategory("Action")
    .setAction("Share")
    .setLabel("facebook")
    .setValue(10)
    .build());
```

以上面的程式碼來看，分類是屬於“動作”類型，行為是“分享”，標籤是 “facebook” ，價值是 10 , 這樣就是一個Event，記錄者 使用者觸發了“分享一筆資料到facebook”的行為。

對於開發者比較特別的是 value 這個參數，只有他是整數類型，其實這就是設計者自己對這行為進行價值評分，至於標準怎麼訂，這看人所好，對於 ui , ux , pm 來說，價值是一個評斷使用者有沒有達到設計者期望的行為參考數值，作為修正的依據。（這有點多談了，這段偏向 ga 的數據參考分析，有用沒用，多有用的數據設計，反而比較偏向問卷設計）

ga event 報表

![ga_event](../../assets/images/ga_event.png)

[https://boochlin.com/wp-content/uploads/2016/04/ga_event.png](https://boochlin.com/wp-content/uploads/2016/04/ga_event.png)

第一階段先以基本使用及介紹，到此告一段落，下一篇開始談比較進階的部分
