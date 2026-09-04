# Android AppIntro Proguard issue.

> 原文網址：https://boochlin.com/?p=262
> 發布日期：2017-04-17
> 文章編號：262

---

![Screenshot_20170308-012917](../../assets/images/Screenshot_20170308-012917.png)

[https://boochlin.com/wp-content/uploads/2017/02/Screenshot_20170308-012917.png](https://boochlin.com/wp-content/uploads/2017/02/Screenshot_20170308-012917.png)

[https://github.com/apl-devs/AppIntro](https://github.com/apl-devs/AppIntro)

算是使用網路現成的 introduction lib 所發生的問題。在此簡單筆記一下。此 lib 主要功能就是可以快速做出往右翻頁的 activity ，內容配置通常就是一圖一文字，如上圖表示，但是這次問題並非功能上，而是 『Proguard-project.txt included in lib is hinders obfuscation of final apk』

通常我們 build release app 時，會啟用 Proguard 來混淆程式碼，以及降低 apk size，但是這次 import app intro lib 4.2 時，卻發現整個 app 沒有被混淆。

```
compile 'com.github.paolorotolo:appintro:4.1.0'
```

![螢幕快照 2017-04-18 上午1.31.19](../../assets/images/螢幕快照-2017-04-18-上午1.31.19.png)

[https://boochlin.com/wp-content/uploads/2017/04/螢幕快照-2017-04-18-上午1.31.19.png](https://boochlin.com/wp-content/uploads/2017/04/螢幕快照-2017-04-18-上午1.31.19.png)

用 [dex-method-counts](https://github.com/mihaip/dex-method-counts) 跑出來是這樣子，第一個是 function name 沒有被混淆，可以看出原本的 function method name，第二是 function 數量也太多了，完全要突破 65k method limit，這樣完全沒有 Proguard 的效果。而正常情況應該是

![螢幕快照 2017-04-18 上午1.38.03](../../assets/images/螢幕快照-2017-04-18-上午1.38.03.png)

[https://boochlin.com/wp-content/uploads/2017/04/螢幕快照-2017-04-18-上午1.38.03.png](https://boochlin.com/wp-content/uploads/2017/04/螢幕快照-2017-04-18-上午1.38.03.png)

function name 會用 a,b,c,d 這種不知所云的 method name 取代，function 數量也會降下來。

老實說在找這個問題時非常困擾，因為完全不知道那裡出錯，瘋狂的查找 build.gradle 的 build flag ，最後是靠者 git 的不段版本回朔才推估出這問題，接者回去找作者的 github 才發現也有人提出這問題，也已經提出[PR](https://github.com/apl-devs/AppIntro/commit/bf1fb7d4e83d6743e4e74e9c9b30a78de248aa60)解決此問題。但是作者並還沒有根據此PR釋出新版本。

[https://github.com/apl-devs/AppIntro/issues/336](https://github.com/apl-devs/AppIntro/issues/336)

[https://github.com/apl-devs/AppIntro/issues/337](https://github.com/apl-devs/AppIntro/issues/337)

[https://github.com/apl-devs/AppIntro/commit/bf1fb7d4e83d6743e4e74e9c9b30a78de248aa60](https://github.com/apl-devs/AppIntro/commit/bf1fb7d4e83d6743e4e74e9c9b30a78de248aa60)

看起來是 proguard-project.txt 的設定值問題，他不小心把使用者project 的所有 method 都包進 proguard 的保護規則中，造成混淆失敗。這裡基本上建議所有 lib 的開發者最好使用 『consumerProguardFiles』來設定 lib 的混淆規則，將影響範圍都限制在 lib。

好啦，最後我是靠者退回前一版的 AppIntro 4.0.0 先避開這問題，然後檢查使用上跟最新版的差異是否有影響。

```
compile 'com.github.paolorotolo:appintro:4.0.0'
```

如果有中文圈的開發者遇到這問題，希望有幫助到你。至於正式解只能期待作者發布 4.2.0 版本。
