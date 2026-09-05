# Android runtime resource overlay

> 原文網址：https://boochlin.com/?p=44
> 發布日期：2015-07-10
> 文章編號：44

---

# Android runtime resource overlay

 

或許可以先看看

[http://developer.sonymobile.com/2014/04/22/sony-contributes-runtime-resource-overlay-framework-to-android-code-example/](http://developer.sonymobile.com/2014/04/22/sony-contributes-runtime-resource-overlay-framework-to-android-code-example/)

這位老兄Mårten Kongstad算是目前有比較完整再談 runtime resources overlay 的第一人

如果英文還行的話的可以直接看原文

硬要說的話最好有寫assetmanager 的概念，會比較理解這中間的原理

可以搭配老羅大師的幾篇文章（老羅真的很猛，可惜他的書有點舊了，不然還真想買一本）

[http://blog.csdn.net/luoshengyang/article/details/8738877](http://blog.csdn.net/luoshengyang/article/details/8738877)

正文開始

續前一篇 Android 實作 data/control 分離 – createPackageContext()

這種方法達到客製化的功能，再 android l 之後可以不用搞的那某麻煩

由sony 的團隊 提出一套新的 resources match framework.

Runtime Resource Overlay (RRO) ，sony 本身也同時利用於 Xperia Themes.

所以很明顯的 這就是要用在換圖片，背景之類的神兵利器.

### Original Mechanism

Resources in Android consist of three parts:

1. A symbolic name – package.type/name, for example com.android.launcher.string/hello_world.

2. A set of configuration constraints that must not be violated if this resource’s value is to be used.

3. An actual value.

根據以上這些資訊 可以自動批配到最適合的機器狀況

![how to match resource](http://developer.android.com/images/resources/res-selection-flowchart.png)

上述匹配規則這就不詳談了(看官網或老羅比較快)，直接解釋底下這張圖片

![getresoures](http://developer-static.se-mc.com/wp-content/blogs.dir/1/files/2014/04/vanilla-resource-lookup.png)

在android裡開發中常常用到的語法

```
context.getResources().getDrawable(R.id.hello_world);
```

這就是指定 resources 去取資料

接者此 resources id 資料會傳給 system 的 assetmanager， 利用 id+package name +機器的 configure(cfg)

進行最適合的資料批配（第一張圖），取得唯一的id值（0x7f），最後利用這值到apk拿出資源，至於左邊的 framework-res.apk 則是 android 提供的原生素材，資料等，可以發現resource id都是 0x01 開頭

就是利用這樣的機制，可以讓application 根據不同的機器狀態提供合適的資源，這也是為何大部分的ap開發者都會在res/底下設計不同的圖片資源或是語系對應，而在此之上，我想sony想要更加進一步，更彈性的提供這些資源去對應theme的對應，而不是在每次更新才提供新的樣貌，因此提出了新的runtime resources overlay的機制給google

以便在framework曾就把這些麻煩的外部資源access給作掉

### How to use Runtime resource overlay

前面歷史講了那某多，其實用起來非常般單純，除了你本來就用被覆蓋的目標，以launcher為例，

(這是一個希望常常變更風格的app，或是想要客製化的部分)

[(可以跳到這篇文章，看看為何需要這種要求)](https://boochlin.com/?p=41)

一個專門overlay別人的ap，以下為必要的幾個要件

1. Have an AndroidManifest.xml.

2. May be installed.

3. Are built using aapt.

在AndroidManifest加入你想要覆蓋的對象

```
<overlay android:priority="2" android:targetPackage="com.example.android.launcher"/>
```

因為同時可能會有好幾個想要去overlay的ap(resources1.apk , resources2.apk)，這時候系統就會去比對priority 值，越高的為越優先，如果沒有指定priority則以後安裝的為主，除了這樣，你想overlay的資源，必須跟對象擺在相同的對應路徑，且有相同的name.

### Runtime resource  overlay Mechanism

![image_overlay](http://developer-static.se-mc.com/wp-content/blogs.dir/1/files/2014/04/multiple-overlays-resource-lookup.png)

上述這張圖的路徑跟第二張圖蠻像的，就只差在找資源的時候會一個一個overlay的apk都去檢索一次，target apk是最後一個找的，其他多個overlay apk 則根據priority決定換上那一張圖

### 安全性呢?

這套功能感覺非常威猛，畢竟可以隨意亂覆蓋別人的ap資源，那不就代表可以惡搞別人的ap，像是 welcome , 改成shock的字串，其實這也沒那某容易，畢竟你要知道其他人的ap package name(雖然不難), 還有 id 對應name

(其實這也不難，畢竟decompile 都一堆人都在教了)，上面我都說了一對不難了，難道google會覺得難嗎，所以他目前為了安全性，限制所有overlay apk，都會放在device的/vendor/overlay，而這個目錄一般開發者是沒辦法access的(至少要root)。

vendor們想要利用google play 的更新機制也必須確保是同個sign key，所以目前看起來，一般開發者還是不太能利用這樣的機制

### 總是有個極限

想要overlay的東西百百種，但是這機制也沒有那某厲害

1. 至少要有個id吧，也就是說只要有id就有機會(integers, strings, drawable, arrays, and so on)

2. AndroidManifest.xml 這個不行(同上)

3. overlay reference 到其他的 reference(這很難解釋，等開下一篇)

### try it.

最後你身為一個開發者，自己也去拉code，或者是同業去拉高通，發哥的code，然後也遵守上述的規則。發現還是沒有這機制，那請你一定先去確認overlay的id name 相關規則(我真的遇到很多人名子打錯)

如果還是沒問題，那應該就是code base 根本沒上這幾條patch

—-

runtime resources overlay patch

[https://android-review.googlesource.com/#/c/113651/](https://android-review.googlesource.com/#/c/113651/)

[https://android-review.googlesource.com/#/c/113652/](https://android-review.googlesource.com/#/c/113652/)

[https://android-review.googlesource.com/#/c/113653/](https://android-review.googlesource.com/#/c/113653/)

[https://android-review.googlesource.com/#/c/121589/](https://android-review.googlesource.com/#/c/121589/)

結束，下一篇考慮寫遇到的各種疑難雜症

ref:

[http://developer.sonymobile.com/2014/04/22/sony-contributes-runtime-resource-overlay-framework-to-android-code-example/](http://developer.sonymobile.com/2014/04/22/sony-contributes-runtime-resource-overlay-framework-to-android-code-example/)
