# Android home screen life cycle.

> 原文網址：https://boochlin.com/?p=75
> 發布日期：2015-08-05
> 文章編號：75

---

一般的activity life cycle  大概網路上已經提到爛掉了

想當然不會跟者淌這個混水，不過還是要先有基本的life cycle 觀念

這次要提的是 **Home Screen Life cycle.**

[activity life cycle](http://developer.android.com/training/basics/activity-lifecycle/starting.html)

各位先感受感受，

這是一般的情況

![activity flow](http://developer.android.com/images/training/basics/basic-lifecycle.png)

使用ap1 中，切換到  ap2 正常來說會是

![](http://developer.android.com/images/training/basics/basic-lifecycle-paused.png)

#### **ap1  進入  onPause —->  ap2  進入 onResume**

 

繼續延伸來思考，按了home button 會發生怎樣的狀態切換

#### **ap1  進入  onPause —-> home ap  進入 onResume.**

home ap  也是一種application 所以當然也符合這個生命週期

home button 只是提供一種進入home screen的手法

需要補充的是 home button 切到 home screen 的情況

會多進入一個 onNewIntent 的狀態，所以要特別處理 home button event

 則是在這作手腳

#### 把home button 補完

**ap1 進入 onPause —–> home ap 進入 onNewIntent —–> home ap 進入 onResume**

好了，第一階段 home button left cycle 搞定

接下來請思考一下,onResume 和 onPause 狀態差在哪裡

主要在於 ap view 是否顯示在foreground，

onResume 是view 顯示在foreground

onPause 是 view 不顯示在foreground

#### 第二個問題是

home screen 已經在onResume 階段，顯示在 foreground

這時候 click home button 會發生何事？

#### home ap 進入 onPause –> home ap 進入 onNewIntent –> home ap 進入 onreusme

結論是 home screen 自己玩自己的那一套

 一人分飾兩角 自己瞬間 消失 再出現

會提到這個，主要是提醒發發home screen ap 處理home button event

 也要同時考慮 onPause 和 onresume 的觸發關係，而不能單純考慮 onNewIntent 的 handle.

 算是home screen的特殊案例，與一般 ap 不同的地方
