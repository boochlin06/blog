# React Native 學習筆記

> 原文網址：https://boochlin.com/?p=136
> 發布日期：2016-06-30
> 文章編號：136

---

![ReactNativelogo](../../assets/images/ReactNativelogo.png)

[https://boochlin.com/wp-content/uploads/2016/06/ReactNativelogo.png](https://boochlin.com/wp-content/uploads/2016/06/ReactNativelogo.png)

## 官方網站 ： [React Native](http://facebook.github.io/react-native/)

中文版：[React Native cn](http://reactnative.cn/docs/0.27/)

### 

### 簡單問答：

**Q：React Native 點評？**

A：其主旨『What we really want is the user experience of the native mobile platforms, combined with the developer experience we have when building with React on the web.』以及『The focus of React Native is on developer efficiency across all the platforms you care about — learn once, write anywhere.』

個人覺得第一句話雖然說的是『有native的使用者經驗，卻同時有react的開發效率』，但我看到的是，這是為了讓寫 web 的前端工程師更加方便開發 mobile app 而設計的，因為開發者只要用 react.js 的語法概念就可以構築跨平台 mobile app , 這無疑使得 前端工程師的開發領域更加寬闊，如果只是寫 ui 和 network 為主的 app，甚至可以不用懂 native ap 的一些概念。

但對於本來就是寫 android（or ios） app 的工程師來說，想跨平台到也同時能寫 ios(or android)，對於這件事，學習曲線不會直接去學 ios(or android ) 還要低，因為你還是要懂 react , flexbox 這一堆前端該懂的概念。

但目前學習到一個階段，第二句話的效應才真的感受到，這並非『寫一次，就可以個平台到處運作』，而是『學一次，到處寫』』。這東西對本來是前端的工程師也沒有那麼友善，畢盡 react native 不少元件還是要往下串到 native component 上，因此你其實兩個平台還是要很熟悉。要了解各平台的限制。希望未來 facebook 會改善串接 native component 的相關開發流程，這還蠻值得期待的，畢竟目前連正式版也還沒有出。

[我对 React Native 的理解和看法](http://div.io/topic/851?page=1#3453) 大師的看法

知乎上也有許多不錯的回答，懶得轉貼了，像是『**只熟练Java和Android开发，想直接上手React Native进行iOS开发坑在哪里**？』

**Q：React Native 跟 React.js 有啥關係？**

A：React Native 和 React.js 共用一些抽象層，但用途與實作方面都不同：React Native 目前只能開發 iOS 和 Android，而 React.js主要用於開發web頁面。

### 開發環境設定：

**[官方環境](http://facebook.github.io/react-native/docs/getting-started.html#content)**

**[React Native 的 50 道陰影](http://elvis.csie.io/2016/05/react-native-50.html)**

我覺得前輩寫得不錯啦，人又有趣

結論就是 用 mac 真好啊，windows 就是負責裝steam

### Third party lib:

**[js.coach](https://js.coach/)**

目前最齊全的 lib 整理，目前react native 雖然算是活躍，但是相較已經成熟的社群，個人覺得還是有很多未知地需要探索

### Reference app:

**[UIExploer](http://www.reactnative.com/uiexplorer/)**

五花八門的基本用法

**[F8](https://github.com/fbsamples/f8app):**

** [中文](http://f8-app.liaohuqiu.net/)**

一些新的framework也加上去，算是 ui explorer 的強化版

**[APP coda](http://www.appcoda.com.tw/react-native-introduction/)**

這是我第一個看的資料，造者打一次，還蠻淺顯易懂。

書本

**『React Native』入門與實戰**

老實說這本是比較早出的，只有ios版本的對應，所以有點可惜，但作為上手用是可以的。

### npm 管理工具：

**[npm 官方網站](https://docs.npmjs.com/getting-started/what-is-npm)**

npm makes it easy for JavaScript developers to share and reuse code, and it makes it easy to update the code that you’re sharing.

**[rnpm](https://github.com/rnpm/rnpm)**

React Native Package Manager built to ease your daily React Native development. Inspired by CocoaPods, fastlane and react-native link it acts as your best friend and guides you through the native unknowns. It aims to work with almost all packages available with no extra configuration required.

沒有這個，一個一個加第三方module 會很累，這種 dirty job 一定要交給電腦做啊

```
npm install -g rnpm

rnpm link xxxxx=npm package
```

### Database：

**[Realm:](https://realm.io/docs/react-native/latest/)**

Realm React Native enables you to efficiently write your app’s model layer in a safe, persisted and fast way.

當然最主要還是 realm 是同時兩個平台都能通用的資料庫

### Crash report:

**[crashlytics](https://try.crashlytics.com/)**

** [fabric](https://get.fabric.io/)**

** [npm integrate lib](https://github.com/corymsmith/react-native-fabric) **整合上述兩種的 js react native lib.

重點是免錢的商業版

### localization language:

**
[https://github.com/stefalda/ReactNativeLocalization](https://github.com/stefalda/ReactNativeLocalization)**

** [https://www.npmjs.com/package/react-native-i18n-complete](https://www.npmjs.com/package/react-native-i18n-complete)**

語系切換問題是遲早的

### Redux:

**[官方](http://redux.js.org/)**

** [Redux 介绍](https://segmentfault.com/a/1190000003503338?_ea=323420)**

** [Redux 入門](https://rhadow.github.io/2015/07/30/beginner-redux/)**

對於 action , reduicer , store 已經有些概念了，但很多還是不太懂，大概是因為沒碰過其他的資料流串法(Flux)。需要惡補

### React Native component communication:

**[React Native 自定义事件机制](http://www.ghugo.com/react-native-event-emitter/5/)**

裡面提到三種溝通方法，分別是

[EventEmitter](https://github.com/medikoo/event-emitter)

RCTDeviceEventEmitter 這個最無腦

Subscribable

```
const RCTDeviceEventEmitter = require('RCTDeviceEventEmitter');
    RCTDeviceEventEmitter.emit('AddNewAlarm','test');

    RCTDeviceEventEmitter.addListener('AddNewAlarm',
      (test) => {
        self.showTimePicker(this.state.currentTouchItem,true);
      }
    );
```

### Drawer:

[react-native-drawer](https://github.com/root-two/react-native-drawer)

基本上快要是 android 開發的起手式，而這類的頁面切換，就會用到 router 這類的頁面切換功能，其實原生就有 navigator 的元件。但有第三方再度改良，更直覺使用，切換更加方便。

[react-native-router-flux](https://github.com/aksonov/react-native-router-flux)

這裡有不少坑，之後再開一頁來聊詳細

### listener series:

[sms](https://github.com/CentaurWarchief/react-native-android-sms-listener) listener

notification listener

目前都只找到發送 notification 的發送 Lib ，但是想找找聽 notification的 lib，但是這個 ios 和 android 的處理方法有很大的差異，可能要自己實做

### 

### Device info:

[https://github.com/rebeccahughes/react-native-device-info](https://github.com/rebeccahughes/react-native-device-info)ˇ

這個沒啥好解釋，非常實用。

### note:

有很多時候加入新的圖片，然後 reload app ，卻發現一直跳出 transform error 找不到圖片，這時候就把模擬器關掉，還有 npm server 關掉，然後重啟。真的在不行那就

```
rm -rf node_module/
npm install
react-native run-ios
```

 

還有一些用到的工具，之後隨者開發進度，會慢慢地更新（應該吧），把一些奇怪的坑還有基本用法 code 補上。發現每個 lib 基本都大到可以開一篇。也有很多 react native ㄧ更新就掛的 lib。 2016/06/30

### 小故事

睽違許久，距離上次更新有段時間，簡單來說就是各類事情混雜在一起，忙到不行，不過這樣倒是挺充實的。回到主題上，這次是『React Native』，可能會有人想說平常都是談『android』，這次居然變了，那就趁此機會順便把自己稱號改一下好了，原本在『about me』的分頁都自稱 『android 工程師』，這次就順便改成『mobile 工程師』。

決定來個前情提要，其實開始弄這東西主要還是公司要求，希望之後開發的 app 都不要只有 android 版本，同時要有 ios 版本，因此我們偉大的長官就發布『那就用跨平台啊』雖然我覺得他可能沒有 survey 過，只是單純的被『跨平台』給吸引住。順道一題，長官是提倡使用 MS 的『Xamarin』，原因不知道。但最後一番唏哩花啦驚天動地的溝通之後決定就是你了『React Native』。

接者繼續講故事，隨者確定平台之後，團隊們要開始學習新的語言『java script』，不是在開玩笑，整個團隊大家在 android 都有各自擅長的領域，但幾乎沒有人有專門深入研究過 『java script』，千萬不要跟我說 『java script 也有 java，android 不是跟 java 也很像嗎？這有啥難』，還好這種話連我長官都不會說，不然真的要開啟履歷啦。這種感覺就是有人會把『巴基斯坦』跟『卡巴斯基』搞混，哭笑不得。

結論，感覺很有趣，以前也寫過一陣子，忘記那那就重新開始學阿，管他是 react , java script , node.js 通通放馬過來。反正『麻，寫程式這件事不就是這樣，只能不斷學習』，感覺自己說了名言，所以我決定也順便把 blog 副標題改掉好啦。

終於要進入正題啦，接下就只是單純筆記啦。如果有人想知道故事的後續或是那一番稀哩嘩啦驚天動地的溝通是啥？歡迎底下留言。
