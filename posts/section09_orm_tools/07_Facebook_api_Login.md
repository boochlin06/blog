# Facebook api on Android : Login FB AP on device , but account manager not show.

> 原文網址：https://boochlin.com/?p=111
> 發布日期：2016-01-31
> 文章編號：111

---

好啦，看這標題可能以為我是要來解這個問題

就是 facebook 官方ap登入了，卻在settings -> account 裡卻找不到，但是這問題是用我的sony z3 才有問題，用了nexus 5 卻沒此問題，所以看起來也是sony 的 framework有問題，這我當然不解

如果有人真的想解，這裡有個方法很便宜

[http://www.knowyourmobile.com/sony/sony-xperia-z/19718/how-use-facebook-your-sony-xperia-z](http://www.knowyourmobile.com/sony/sony-xperia-z/19718/how-use-facebook-your-sony-xperia-z)

Step1. Grab your Sony Xperia Z 開啟手機

Step2. Tap ‘Menu’ 點開app 選單

Step3. Hit ‘Settings’ 進入 『Settings』裡的『帳戶』

Step4. Choose ‘Add Account’ 選擇 『新增帳戶』

Step5. Select ‘Xperia with Facebook’ 選擇 『Xperia Facebook整合』

Step6 Enter your Facebook details, or if you’ve been living in a cave for the last half a decade and don’t have an account, you can create one. 輸入帳密，一路下一步下一步

回到正題

狀況描述：

其實這是一段古老的 code，我也不知道我的前輩從哪裡挖出來

主要功能就是可以讓 ap 可以去抓 facebook 的資料，然後顯示在 ap 上面，

使用的facebook sdk 為 ‘com.facebook.android:facebook-android-sdk:3.23.1’這２０１５不在使用的東西

裡面再取得 facebook 的 access token 必須先確認 facebook ap是否有登入，因此前人使用的是

```
private boolean isLogin(String type) {
        LOG.d(TAG, "isLogin(" + type + ")");
        mAccountManager = (AccountManager) getSystemService(Context.ACCOUNT_SERVICE);
        Account[] accounts = mAccountManager.getAccountsByType(type);

        if (accounts.length > 0) {
            LOG.d(TAG, "isLogin: true");
            return true;
        } else {
            LOG.d(TAG, "isLogin: false");
            return false;
        }
    }
```

原本這樣的方法，大概都通用，但是卻在我的z3不能使用，跟最上述的狀況一樣，那想當然地每次這段都會回傳false 導致根本不能抓資料。

但是其實已經拿過access token，所以只要把這存成 sharepreference ，然後三不五時去確認一下就好啦

所以 islogin 這方法根本不是重點，廢掉。

結論就是用這方法判斷根本不對啊啊啊啊啊啊啊啊，一切都是 sony 的問題

老實說我不知道這有沒有問題，像是更新 token 時，沒有跟者去寫值，因為也不知道啥時改的，但是我試了老半天好像也沒有啥問題，

在這之前我也拜訪了一下google大神，得到了

```
public boolean isLoggedIn() {
    Session session = Session.getActiveSession();
    return (session != null && session.isOpened());
}
```

and

```
private boolean isLoggedIn = false; // by default assume not logged in

private Session.StatusCallback callback = new Session.StatusCallback() {
    @Override
    public void call(Session session, SessionState state, Exception exception) {
        if (state.isOpened()) { //note: I think session.isOpened() is the same
            isLoggedIn = true;
        } else if (state.isClosed()) {
            isLoggedIn = false;
        }
    }
};

public boolean isLoggedIn() {
    return isLoggedIn;
}
```

可惜結果還是一直回傳 false ，這樣當然也無法判斷 facebook ap 到底有沒有登入。結論我還是不知道這在搞啥？sony what happened.

好啦，其實改天就通通改成4.0版，畢竟這樣放下去只會更臭

對不起，我來不及寫4.0的文章

最近都在看google analytic的資料，但是目前還沒看到有個頭緒，無法整理成好文

以上這段話，等到我發相關資料就會刪掉啦

我先承認這一篇有點水，畢竟到過年，總是沒啥力，但是水歸水，說不定有人跟遇到一樣的問題
