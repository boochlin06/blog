# Android ButterKnife introduction

> 原文網址：https://boochlin.com/?p=238
> 發布日期：2016-11-30
> 文章編號：238

---

![butterknifelogo](../../assets/images/butterKnifelogo.png)

[https://boochlin.com/wp-content/uploads/2016/11/butterKnifelogo.png](https://boochlin.com/wp-content/uploads/2016/11/butterKnifelogo.png)

Android 寫久了，大家總會有一些定番的起手式，像之前就介紹過不少方便的 lib ，但仔細想想好像從沒有提過常用的 android studio plug-in ，因此這次就來介紹個實用 plug-in，ButterKnife ，雖然這樣特別開一章節來說好像很厲害，但其實只是懶人工具，有沒有都沒關係，沒有就只是麻煩一點，但是用熟了，大概會….變習慣？

### [Butter-knife](http://jakewharton.github.io/butterknife/)

奶油刀，Github 上有上萬 star 的知名開源專案，主要用於 Android 的 View 注入，使用 Annotation 的方式讓程式碼更加簡潔。搭配自動 Annotation 工具 Android ButterKnife Zelezny 簡直方便到炸掉。以下為四大訴求

* Eliminate `findViewById` calls by using `@BindView` on fields.

* Group multiple views in a list or array. Operate on all of them at once with actions, setters, or properties.

* Eliminate anonymous inner-classes for listeners by annotating methods with `@OnClick` and others.

* Eliminate resource lookups by using resource annotations on fields.

雖然有點突兀，但是還請先載入 lib

```
compile 'com.jakewharton:butterknife:8.4.0'
annotationProcessor 'com.jakewharton:butterknife-compiler:8.4.0'
```

接者來看看這些訴求到底是啥 ? 以往身為一個 android 工程師一生中沒有寫幾百次底下的 CODE 實在是不可能，沒錯就是 findViewById and class cast

```
// 這個就是基本最常用的取出 view , 後面一定會搭配一個 layout xml
EditText edtTitle = (EditText) findViewById(R.id.edtTitle);
```

而 butter knife 就只要

```
// 底下兩行就可以收工
@BindView(R.id.edtTitle)
EditText edtTitle;
// 除了注入 view 以外，也可以用於其他資源
@BindString(R.string.title) String title;
@BindDrawable(R.drawable.graphic) Drawable graphic;
@BindColor(R.color.red) int red; // int or ColorStateList field
@BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
// ...
```

另外一個就是大家很常寫的 Onclick事件

```
// 通常呼叫 setOnClickListener
edtTitle.setOnClickListener(new View.OnClickListener {....});
// 而 butter knife 則只需要
OnClick(R.id.edtTitle)
public void OnEdtTitleClick(){
  //do something.
}
// 很多個view 也可以綁在同一個 click method
OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
  if (door.hasPrizeBehind()) {
    Toast.makeText(this, "You win!", LENGTH_SHORT).show();
  } else {
    Toast.makeText(this, "Try again", LENGTH_SHORT).show();
  }
}
```

想要一次處理一串 view 也可以這樣寫(通常是view 之間有直接的關係)

You can group multiple views into a List or array. 將多個view 整理在一個 list裡

```
@BindViews({ R.id.first_name, R.id.middle_name, R.id.last_name })
List nameViews;
```

The apply method allows you to act on all the views in a list at once. 針對屬性list 內的 view 一起設定 (其實這項功能我沒啥用過，使用情境不多)

```
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);
```

```
static final ButterKnife.Action DISABLE = new ButterKnife.Action() {
  @Override public void apply(View view, int index) {
    view.setEnabled(false);
  }
};
static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
  @Override public void set(View view, Boolean value, int index) {
    view.setEnabled(value);
  }
};
```

An Android Property can also be used with the apply method.修改列表內view 的所有 alpha 值

```
ButterKnife.apply(nameViews, View.ALPHA, 0.0f);
```

談到這裡 ButterKnife 已經幫工程師省了不少功夫，但應該還是有人會哭說『雖然不用寫 findViewById，但是annotation 也寫了一大堆，這也還是 dirty job 阿』

沒錯，就是有人想到這一點，乾脆連 annotation 也自動幫你生一生，就是 [ButterKnifeZelezny](https://github.com/avast/android-butterknife-zelezny) ，這個 android studio plug-in 完全為了 butter knife 而生，直接上張圖就知道多方便了（讓我糾正一下這張圖的 ButterKnife.inject(this) ，現在8.0.0 版本之後已經不用 inject ，而是 ButterKnife.bind(this)）

![zelezny_animated](../../assets/images/zelezny_animated.gif)

[https://boochlin.com/wp-content/uploads/2016/11/zelezny_animated.gif](https://boochlin.com/wp-content/uploads/2016/11/zelezny_animated.gif)

* 安裝方法

* in Android Studio: go to `Preferences → Plugins → Browse repositories` and search for `ButterKnife Zelezny`

* [download it](http://plugins.jetbrains.com/plugin/7369) and install via `Preferences → Plugins → Install plugin from disk`

寫到這裡算是告一段落，Butter Knife 還有一些有趣的功能，像是不少情況使用 靜態編譯 inject view 的方式並不合適，必須在 run-time 階段用 findViewById 動態取得 view ，這對於 Butter Knife 能做的事情就不多了，不過還是有提供簡略 class cast 的方法取代 findViewById

```
View view = LayoutInflater.from(context).inflate(R.layout.thing, null);
TextView firstName = ButterKnife.findById(view, R.id.first_name);
TextView lastName = ButterKnife.findById(view, R.id.last_name);
ImageView photo = ButterKnife.findById(view, R.id.photo);
```

reference:

[http://jakewharton.github.io/butterknife/](http://jakewharton.github.io/butterknife/)

[https://blog.ccjeng.com/2015/08/Android-ButterKnife.html](https://blog.ccjeng.com/2015/08/Android-ButterKnife.html)

[http://www.jianshu.com/p/9ad21e548b69](http://www.jianshu.com/p/9ad21e548b69)
