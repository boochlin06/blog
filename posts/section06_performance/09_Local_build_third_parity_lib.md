# Android Local build- use third parity lib.

> 原文網址：https://boochlin.com/?p=55
> 發布日期：2015-07-10
> 文章編號：55

---

老舊筆記系列

在 FMRadio這種需要mm跟者系統一起build ap

並無法再eclipse 隨意加進third party lib，而是另外要寫一些東西

使用第三方lib

1.必須再Android.mk下加入

```
LOCAL_JNI_SHARED_LIBRARIES := libxxx
```

就是把so文件放到apk文件里的libs/armeabi里，而include $(LOCAL_PATH)/jni/Android.mk为了编译so文件。

```
LOCAL_JAVA_LIBRARIES := qcom.fmradio
```

**

**

LOCAL_JAVA_LIBRARIES
When linking Java apps and libraries, LOCAL_JAVA_LIBRARIES specifies which sets of java classes to include. Currently there are two of these: core andframework. In most cases, it will look like this:LOCAL_JAVA_LIBRARIES := core framework

Note that setting LOCAL_JAVA_LIBRARIES is not necessary (and is not allowed) when building an APK with “include $(BUILD_PACKAGE)”. The appropriate libraries will be included automatically.

2.再android manifest application欄位內加入引用lib的宣告

2.1並且增加全域的activity

android:name=”.FMAdapterApp”

ref:

这个name属性是来设置你所有activity所属于哪个application的，默认是android.app.Application，你也可以自己定义一个类例如

```
public class MyApplication extends Application {}
```

然后

就是这儿，将我们以前一直用的默认Application给他设置成我们自己做的MyApplication

MyApplication类的作用是为了放一些全局的和一些上下文都要用到变量和方法之类的。

3.再程式碼中加入讓系統正式讀入lib

```
static {
System.loadLibrary("qcomfm_jni");
}
```

最好放到static 裡執行，java會最先執行

4.更動import 的位址

reference:

http://yueguc.iteye.com/blog/820591

http://yueguc.iteye.com/blog/820591

http://www.kandroid.org/online-pdk/guide/build_cookbook.html

http://www.devdiv.com/forum.php?mod=viewthread&tid=45479
