# Android 實作 data/control  分離 – createPackageContext()

> 原文網址：https://boochlin.com/?p=41
> 發布日期：2015-06-17
> 文章編號：41

---

一隻寫好的android app 最佳情況下，你會希望不管porting 到哪一台機器都能吃遍天下無敵手

這聽起來很像廢話，但是其實搞起來不是隨隨便便的

光是一般third app 就要注意各種dpi的對應，以及語系的對應（阿拉伯語是從右邊開始的喔）

但是這不是此文的重點

本文這次重點是如何  customize app by access other app 

——以下為碎碎唸專區，想看code 直接拉到最底————

對oem/odm廠來說，常常聽到的需求都是

pm:xxx機器希望上面的launcher,桌面 wallpaper是粉紅色，因為主打的是女性市場喔

rd:喔，了解

pm: 中國地區不能出現google app喔，記得關掉喔

rd:喔，了解

pm: 中華電信的機子一定要放hami在default workspace喔

rd:喔，了解

硬要說起來，這種東西改起來一點都不難，但是每次都要花時間（重點是context switch）

而且如果要放上 play store 的話，上述的對話根本…

這時候，你老闆看到很多launcher 都可以放上play store上，想說放上去可以順便賺一下

下載錢

但是pm的要求可能又會根據要出貨的機子，千變萬化，放在play store上又很難達到

加上這群人很煩

這時乾脆把這種要求給切出來，然後讓其他的team去handle.可是總要提出一個方案

讓它門可以自動化去搞定

**前言狗屁那某多，結論就是先在各自的機種中將要客製化的部份專門放到專門的app裡**

** 例如 customize.apk之類的，然後在裡面定義好格式放在resource裡**

而這些值可以再build system 就定義好，再機子出貨的時候直接包含再裡面，然後讓

launcher 去準備存取 ，這樣就是一個簡單的d/c分離了，雖然你可能會覺的只是把

修改資料轉移到另外一支 ap 上面，工還是一樣，但其實還是有差異的

**1.至少把修該的資料分離出來，以便於其他人進行修改，不會不小心改壞你原本的code**

** (job fw to others is good)**

** 2.可以把control端的ap分離出來，然後可以丟上 play store.**

上code啦：

跨 Package 存取資源時，絕對不能使用當前 App 的 R class 常數，因為不同 APK 在編譯期生成的資源 ID 數值不一致。必須透過 getIdentifier() 動態查詢目標 Package 的資源 ID。

```java
static boolean FEATURE_UI_CUSTOMIZE;
Context context;
try {
     Context targetContext = createPackageContext("com.android.customize", Context.CONTEXT_INCLUDE_CODE);
     int resId = targetContext.getResources().getIdentifier("feature_ui_customize", "bool", "com.android.customize");
     boolean featureEnabled = targetContext.getResources().getBoolean(resId);
     FEATURE_UI_CUSTOMIZE = featureEnabled;
} catch (NameNotFoundException e) {
     // If not found , you can assign default value here.
     FEATURE_UI_CUSTOMIZE  = false;
     e.printStackTrace();
 }
```

其實也可以利用這個避開一些 sku 問題

像是再大陸區 不想出現六四天安門，那也可以這樣去取代。（以上於2014年的心得）

雖然這一套拿資料方法不錯，可是再5.0之後推出的runtime overlay 完全簡化一上步驟

因此可能在5.0之前比較實用，之後再出一章節談 runtime overlay.

ref:

http://blog.csdn.net/darlk/article/details/7742918
