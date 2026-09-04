# Clean code: 無瑕的程式碼 – 書摘心得（一）

> 原文網址：https://boochlin.com/?p=98
> 發布日期：2025-01-15
> 文章編號：98

---

![](http://im1.book.com.tw/image/getImage?i=http://www.books.com.tw/img/001/057/98/0010579897.jpg&v=513f2000&w=348&h=348)

![wtf](http://www.osnews.com/images/comics/wtfm.jpg)

『唯一有效的的『程式品質』度量單位： 每分鐘罵髒話的次數(wtf/minutes)』[版權網址附上](http://www.osnews.com/story/19266/WTFs_m)

這句話讓我想到同事改其他部門程式碼時的狀況，我想以此作為引言，已經相當足夠的表示出 寫出clean code 的重要性

（序）

認真一點的話就是：

『你有兩個原因來讀這本書：首先你是個程式設計師，接者，你想成為一個更好的設計師』

我自己加上去的：『剛好，好的設計師薪水好像還能看』

（p1）

這裡偷偷工商時間一下，小弟寫了一個簡單的 app [趣投票](https://play.google.com/store/apps/details?id=com.heaton.funnyvote)希望大家有空可以幫忙玩一下。開發小故事[Funny Vote 開發雜記](https://boochlin.com/?p=289)

書摘心得：

## 第一章：無瑕的程式碼

承接<序>的部份，為何我們需要寫出 clean code:

『在大部份的情況下，如果把自己的 coding 過程給錄下來的話，會發現以下行為

Heaton 打開程式碼

找到要修改的函式，開始想想要改啥

向上捲，檢查函數的初始狀況

向下卷回剛剛的函式，開始改一改

阿，又刪掉剛剛的程式碼，再輸入一次，再一次刪掉

發現跟其他函式有關係，捲到剛剛的函數，看了一下要怎某呼叫

回到剛剛的函式，重新修改，發現打錯，再刪掉，又增加』

（p.15）

前面這一段話完全可以用個迴圈給包起來，因為只會不斷地重複

以這樣的記錄來看，其實花費在閱讀程式跟撰寫程式的時間大概是１０：１

所以說如果把技能點數點在閱讀程式的技能上，會比點在寫程式這技能

 在開發上，效益好很多，所以一開始就把code 寫得能看，保持整潔，對於之後的開發絕對是１０倍數的影響

## 第二章：有意義的命名

要有意義的區分

請問以下三個函數要做的事情你分的出來嗎？

1. getActitiveAccount();

2. getActitiveAccounts();

3. getActitiveAccountInfo();

請盡量避免這樣曖昧的命名

（p.23）

### 成員的字首：

#### 2.1  全域變數:

很多情況下，會在全域變數的前面加上 m 或者 m_ 這一類的開頭，來幫助開發者更加容易的便是此變數，但實際上這並沒有太有意義，大部分的人還是只會閱讀 m 之後有意義的命名，更何況現在的ide 都已經進化到，可以主動把 全域變數自動換色 所以 m 這樣的字首提示就變得只會干擾

（p27）

本人 m 的習慣用法到現在還是不太會改，這實在有點見仁見智

#### 2.2  介面和實作:

介面的的宣告時，不少 open source project 都會特地加上 I 這樣的開頭，例如 IImageBackView，藉此告知開發者這是一個介面，但這個 I 同樣也只有讓人分心的功能，開發者根本不在意是不是介面，而是在意這 ImageBackView 到底是怎樣的 view，如果真的想要區分 interface 還是 implement ，那會比較建議在實作端改成 ImageBackViewImpl。

(p28)

#### 2.3 每種概念使用一種字詞：

在同一個專案時，盡可能的在某些關鍵字保持一致，例如在取得資料時函式命名 fetchXXX , getXXX , retrieveXXX 這幾個開頭，要區分開來，或是統一只用一種，維持一致性，讓開發者在 search method 時可以更快地找出關鍵字。

同樣的用在 class 命名，一個專案裡同時有 XXXController , XXXManager , XXXDriver 在，對於開發者也只會感到困惑，這幾種到底差在哪裡，如果沒有差到天翻地遠，統一起來只會有好處，這裡可以配合正在用的 design pattern 習慣用名，幫助開發者進入狀況。

（p.30）

個人在home 開發時，我就常常同時使用 controller 和 manager 混用，後來覺得好像變得不好跟老大說明哪個東西放在哪個 class ，那乾脆通通用 manager ，這樣溝通時也方便多了。

## 第三章：函式

簡短，只做一件事

一段程式碼，假設做了三件事情，最好把這三件事情分開獨立成三個函式，然後給予有意義的命名，幫助開發者更快理解程式碼

以 home 的 BitmapVolleyManager 的部分程式碼來看（如下），以下 load bitmap 做了三件事情

1. set view loading url and check url is valid.

2. get bitmap from image cache.

3. assign bitmap worker task for this view and url

```
public synchronized void loadBitmap(String imageUri, ImageBackView imageView
            , BitmapConfiguration configuration, boolean stackKeep, boolean clearFirst) {

        imageView.setKey(imageUri);
        if (clearFirst)
            imageView.setBitmap(null);
        if (!Utils.isValid(imageUri)) {
            Log.d("BitmapVolleyManager", "loadBitmap, image uri should not be null, cancel loading");
            return;
        }

        Bitmap bitmap = VolleyManager.getPageImageCache(mContext)
                .getBitmap(ImageLoader.getCacheKey(imageUri, configuration.requestWidth
                        , configuration.requestHeight, configuration.mScaleType));
        if (bitmap != null) {
            imageView.setBitmap(bitmap);
            return;
        }
        final BitmapWorkerTask task = new BitmapWorkerTask(imageView, imageUri, configuration);
        if (stackKeep) {
            synchronized (this) {
                mStackLoadTask.push(task);
            }
        } else {
            Log.d("BitmapVolleyManager", "Add a Bitmap task ,image uri:" + imageUri);
            task.executeOnExecutor(getExecutorPool());
        }
    }
```

直接來看也並不難懂，但是更進一步來看或許可以更加獨立成

```
public synchronized void loadBitmap(String imageUri, ImageBackView imageView
            , BitmapConfiguration configuration, boolean stackKeep, boolean clearFirst) {

        checkViewForURL(imageuri, imageview);
        
        if (clearFirst)
            imageView.setBitmap(null);

        Bitmap bitmap = getBitmapFromCache(imageUri, configuration);
        if (bitmap != null) {
            imageView.setBitmap(bitmap);
            return;
        }
        startNetworkBitmapLoadingTask(imageUri, imageView , configuration , stackkeep);     
    }
public boolean checkViewURLIsValid(String targetURL, ImageBackView imageView) {
    imageView.setKey(imageUri);
    if (!Utils.isValid(imageUri)) {
       Log.d("BitmapVolleyManager", "loadBitmap, image uri should not be null, cancel loading");
       return false;
       }
    } else {
       return true;
    }
}

public Bitmap getBitmapFromCache(String targetURL , BitmapConfiguration configuration) {
    Bitmap bitmap = VolleyManager.getPageImageCache(mContext)
                .getBitmap(ImageLoader.getCacheKey(imageUri, configuration.requestWidth
                        , configuration.requestHeight, configuration.mScaleType));
    return bitmap;
}

public void startNetworkBitmapLoadingTask(String targetURL , ImageBackView imageView , BitmapConfiguration configuration
    , boolean stackKeep) {
    final BitmapWorkerTask task = new BitmapWorkerTask(imageView, targetURL, configuration);
        if (stackKeep) {
            synchronized (this) {
                mStackLoadTask.push(task);
            }
        } else {
            Log.d("BitmapVolleyManager", "Add a Bitmap task ,image uri:" + targetURL);
            task.executeOnExecutor(getExecutorPool());
        }
}
```

好吧，至少第一階段的原則已經修正完成了（雖然還是違反很多之後會談到的原則，提到時再改）

(p40.41)

### 3.1 switch 敘述：

要 switch 簡短，並不容易，因為switch 總是會違反『單一原則（single responsibility principle）』

 和 『開放閉和原則（open closed principle）』，詳細程式碼的範例還是看書比較好。

簡單來說就是 switch 的用法往往綁定特殊值，例如 enum  , 這樣在要擴增的時候，勢必要到處修改用到的地方。可是書中也提到這是必要之惡，如果可以的話，最好一大段程式碼，就只有一個switch 就好，剩下的用多型給處理掉。

（p.44）

### 3.2 函式的參數：

函式的最佳參數數量，最理想的是零個，其次是一個，接下來是兩個，好吧三個是極限，別再上去了，因為參數本身也是一種概念的傳達。

下面兩個函式哪個比較好理解

includeSetupPageInfo(newPageContent);

includeSetupPage();

基本上參數和函式名稱是屬於不同分區，理解上是會比較困難一些，無法一眼就快速釐清，參數和函式的關係，而且參數會強迫開發者去理解不必要的資訊，因為開發者會去擔心這參數到底會影響到啥，又該怎某處理，此外，參數越多，測試的複雜度更會指數型上升。

(p.47)

輸出型參數的設計更是一大難以理解點，大部分開發者對於函式的理解通常都是從參數輸入資料，然後再return 裡得到輸出資料。一般情況下根本不會想到會用放在參數的資料做為傳遞的工具。

（本人這點深感痛苦，老是有人把 list 放進參數裡，然後直接對 list 操作，一般我在 trace 都會感到困惑，想說啥時這list 被改過了）

```
List output = sortByTime(input);
底下的寫法真的很容易困惑
sortByTime(input);
```

(p.52)

#### 3.2.1 單一的參數

這一小段，作者十分反對使用 flag 作為參數

主因為 boolean is true 會做第一種事，那 false 就會做第二種事

很明顯地違反單一原則，以上面的例子就是

```
public synchronized void loadBitmap(String imageUri, ImageBackView imageView
            , BitmapConfiguration configuration, boolean stackKeep, boolean clearFirst)
```

這裡面的 stackKeep , clearFirst

這兩個參數分別是，讀取圖片任務，先放在stack 等待其他時間，在開始啟動任務和 先清除 ImageView 上面的圖片，在載入圖片

老實說只看這參數名稱幾乎不懂啥意義

(p.48)

（不過我個人並不是那某反對單一 boolean 參數的設計，因為任何彈性的設計再多形的基礎上

總會有個 method 是最大參數集合，如下，因此只要特地把參數還有函式功能的註解寫好就可以了，不過之後的章節也會提到作者並不喜歡註解）

```
/**
    
    **/
    public void loadBitmap(String imageUri, ImageBackView imageView) {
        loadBitmap(imageUri, imageView, new BitmapConfiguration(DEFAULT_LEFT_ITEM_WIDTH, DEFAULT_LEFT_ITEM_HEIGHT), false, true);
    }

    public void loadBitmapStackKeep(String imageUri, ImageBackView imageView, BitmapConfiguration configuration) {
        loadBitmap(imageUri, imageView, configuration, true, true);
    }
    public synchronized void loadBitmap(String imageUri, ImageBackView imageView
            , BitmapConfiguration configuration, boolean stackKeep, boolean clearFirst) {
```

#### 3.2.2 物件型態的參數

超過兩個三個的參數時，很可能需要將其中幾個組合成一個類別，看起來很像只是規避參數數量盡量少的原則

 其實並不，這完全可以幫助開法者去理解參數的理念，以上面例子來看 就是 BitmapConfiguration 裡面包含寬高，縮放類型

或許可以再把 stack keep 和 clearfirst 加進去，不過個人覺得加進去的話就不能用BitmapConfiguration 而是使用

LoadConfiguration 會比較好，然後包含BitmapConfiguration and stackkeep and clearfirst.

(可是老樣子，我覺得這有點見仁見智)

book example:

```
Circle makeCircle(double x , double y , double radius);
Circle makeCircle(Point center , double radius);
```

(p.50)

### 3.3 使用例外處理代替回傳錯誤碼

使用try/catch，並在 catch 把後續處理完成，而不要用回傳錯誤碼的方式，這代表者事情往後面丟，形成更深層的巢狀結構，尤其是自定義的 enum error code.

### 3.4 把 try/catch 區塊提取出來

在 try/catch 區塊內最好就只有一行函式，然後把code寫在函式裡

（p.54）

少用 break , continue，保持效率可以斟酌使用

 還有絕對不要用 goto，這只會讓看的人痛苦

```
while (condition1) {
  xxx
    if (condition2) {
      break;
    }
}
```

換成下面，這樣明顯好多了

```
while (condition1 && !condition2) {
  xxx
}
```

至於continue，怎某不考慮一下把條件式反轉一下

```
List goodNames = new ArrayList<>();
for (String name: names) {
    if (name.contains("bad")) {
       continue;
    }
    goodNames.add(name);
    ...
}
```

改寫一下

```
List goodNames = new ArrayList<>();
for (String name: names) {
    if (!name.contains("bad")) {
       goodNames.add(name);
       ...
    }
}
```

上面兩段code , reference :

[http://www.jianshu.com/p/7645a5ea7f46?utm_content=buffer50622](http://www.jianshu.com/p/7645a5ea7f46?utm_content=buffer50622)

（p.57）

## 第四章 註解

沒有任何東西可以比一段放對位置的註解還有用，但是放錯地方的註解，更可以弄亂模組化，更別說寫錯的註解，這根本是謠言產生器

（p.61）

### 4.1 註解沒辦法彌補糟糕的程式碼

如果程式碼已經髒亂到必須寫一大堆註解來搞清楚意圖，那請重寫你的程式碼，讓他更整潔，進而不需要那一堆難搞的註解

 整潔的程式碼絕對好過一大堆的註解，最高境界就是不用註解

（p.63）

#### **4.1.1 當然也是有必要或好的註解**

１．法律形版權宣告

２．函式名稱多到不行，弄個有分字的註解比較好懂

３．解釋自己設計的意圖想法，例如函式的順序，或是特殊的spec request.

４．函式庫裡的函式真的太詭異，但是我們不能去修改函式庫的函式名稱，只好自己在使用時，說清楚

５．side effect 的提示

６．TODO 的註解，這最高境界就是全部清掉，但是我們也知道這不太可能，使用TODO可以幫助開發者快速找到修改點

#### **4.1.2 糟糕的註解**

１．神秘註解，只有你看得懂的文字

２．寫太長註解，看你的註解可能能比看你程式碼還要浪費時間的註解, html 式的註解，解釋 rfc 文件編碼模式啊，理解起來又累又不需要

３．誤導形註解，這可能是歷史共業，拜託快刪掉

４．規定行註解，例如 java doc 的規定格式，每個都要寫真的蠢爆了

５．日誌形註解，老實說這在沒有 git 的時代還有理由，但現在用 commit message 去取代不是很好嗎？作者跟出處沒規定的話也是同樣

６．干擾形註解，就是廢話，寫錯地方啊，全域和區域的註解要分清楚

７．被註解起來的程式碼

這個我覺得要大力提倡，通常你註解起來的程式碼，其他人都會以為很重要，然後不敢去刪掉，其他都會以為留者必有原因，最後就變成類似黴菌的東西，實際上這在沒 git 的時代或許可以幫助把 code 保留著，但是現在有版本控制了，請直接刪掉。

 （這個可以大力推廣，老實說註解起來到最後的下場都只是忘了刪，而且隨者開發，那些被註解的程式碼一點都不重要）

接下來我來表演一段糟糕的代碼，至於如何 refactor 請自己練習吧：

```
/**
 * changes (from 11-OCT-2009 ) author : heaton
 *
 * Add a CD with title , author , number , like celine.
 * @param title The title of the CD
 * @param author The author of the CD
 * @param tracks The number of tracks on the CD
*/
public void addCD(String title , String author , int tracks) {
   // new CD for add to total list.
   CD cd  = new CD();
   cd.title = title;
   cd.author = author;
   // size mean the cd duration from start to end.
   //cd.size = size;
   cd.tracks = tracks;
   cdList.add(cd);
}
```

上面的吐槽點真的太多了，請好好檢視現在手上的專案

（p.63-77）

## 第五章 編排

程式的編排很重要，維持一定的團隊規則進行編排絕對可以加速閱讀

 更重要的是 只要確立好團隊規則後，自動化工具可以幾乎無痛導入，cp值極高

(P.86)

### 個人覺得這段還好，主要就是兩個重點

5.1水平距離：

5.1.1 :程式碼的任何一行不應該太長，最低最低的要求就是不得超出螢幕頁面寬度

，保持在視線範圍內，如果需要左右滑動，那肯定造成困擾

5.1.2：每行的程式碼，是情況保持空白行，喘口氣

(P.86)

### 5.2. 垂直距離：

5.2.1：變數宣告位置統一放置，java 習慣全域變數都放在文件最上端，而區域變數則盡量靠近使用位子

5.2.2：相依函式盡量附近，最好的情況就是，可以從開頭的函式順順地往下看，而不需要大範圍的捲動滑鼠

5.2.3：保持一定的空白行，人們喜歡有些留白的人生

(P.90)

這一段個人覺的，上面的水平和垂直距離的規則都可以靠IDE做到

但是共用的變數名，或是數值定義的位址要確不好執行

ex: 像是常見的 xxxSetting.java , xxxConstant.java , xxxDefine.java , 這一類的百家爭鳴

 不值得鼓勵，勢必要有人統一規範並整理，放在差不多的路徑下，並用正確的命名幫助閱讀

 這點手上的 Home 專案，也有同樣的問題

## 第六章 物件及資料結構

java 開發上我們常常讓物件的變數保持 private ，避免其他開發者突發奇想的想要改動其數值導致狀況大亂

 但為何還是有那某多工程師會加上 setter and getter，搞得像是 public 一樣，大家想改就改

（p.105）

其實就是資料抽象化，我提供你的讀取資料的目的要求，但是手段由我控制，重setter and getter 來看

這樣的效果並不明顯，但同樣是不告訴你裡面詳細怎某運作的

### 6.1 物件與資料的反對稱性

首先先定義好物件與資料

 物件將他們的資料再抽象層之後隱藏起來，然後將操作這些資料的函式暴露在外

 資料則將資料暴露在外，而且也沒提供有意義的函式

 注意這兩個定義，這是對立且也互補

（p.107）

看個例子：

```
public class Square {
    public Point topLeft;
    public double side;
}
public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}
public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public final double PI = 3.14159;
    
    public double area(Object Shape) throws NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square) shape;
            return s.side * s.side;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.height * r.width
        } else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```

老實說這樣的程式，忠實的物件導向程式設計師會覺得很詭異（我），這根本就是 procedural function

所以換個方式試試看（物件版）

```
interface Shape {
   public double area();
}

public class Square implement Shape {
   private Point topLeft;
   private double side;
   public double area() {
       return side * side;
   }
}
//....other class implement Shape
```

從上面兩個例子可以看出

 如果要加入一個新的圖形類別時，會發生啥事？

 第一段程式碼，我必須要修改Geometry 內的所有的相關函式才能處理這件事情

 但是採用第二段程式碼的話，則根本不需要Geometry 這個類別

 新增任何類別都不會影響到現有的函式。

那如果要加入一個新的函式像是 perimeter() 的話，會發生啥事？

 第二段程式碼的所有 Shape 物件都必須跟者修改，

 但第一段程式碼卻只要修改 Geometry 類別，增加一個函式即可

（p.108）

由此可以看出這兩種設計的對立以及互補面

再次推導出來：

『結構化程式碼容易添加新的函式，而不需要變動已有的資料結構

 而物件導向的程式碼，容易添加新的類別，而不用變動已有的函式』

反過來說：

『結構化程式碼難以添加新的資料結構，因為必須改變所有的函式

 物件導向程式碼難以添加新的函式，因為必須改變所有的類別』

（p.109）

本人儘管身為物件導向派的一份子，當然也知道全部都是物件，基本上是個神話

所以在適合的時候，要懂得用結構化的程式（需要常常增加減少函式的情況），幫助開發

所有的語法都是工具，挑最適合的就對了

### 6.2 Data Transfer Object

這種物件非常常見，基本上就是資料庫的原始資料 轉成 應用的程式內的物件

 例如： greendao , json to object , gson 這類的關鍵字

 俗稱bean

 這其實很多資料內容都可以用公開變數，但是還是習慣寫成setter and getter

 其實就是一種像物件導向開發者的妥協

（p.113）

讀到這段時，突然想到之前跟老大在討論（感覺跟這章節沒啥關係）

android 提供外部 api access content provider 的方法，用哪種比較好？

本人覺得 就走 content provider 的玩法加文件，提供 URI 然後用android 原生的 sqlite crud 操作

主要就是保持資料的彈性操作，原生的玩法到哪都不會有錯，這樣可以保持目前的程式碼，以最小的修改幅度

就可以完成這項任務

老大覺得 出專門的 lib 把所有的crud操作都規範起來，提供 example , 要求存取者造規則走

有點忘了為何，但主要是利用函式的操作，讓第三方必須認真的看懂才能去操作資料庫

確保安全，並保持程式的抽象化，以後隨便改都可以

不知道個位看官覺得優缺點是啥，或是可以補充沒注意到的點，歡迎討論

## 第七章 錯誤處理

定義最小的 handle scope 和正常的程式流程

### 7.1 try/catch 最好能定義在最精準的範圍：

一般情況下，不少人會貪圖方便，或只是想避開編譯器跳出的錯誤，進而把一大段程式碼用 try/catch 包起來，更甚者只用 Exception 這個巨大的錯誤目標，但這樣以後必定付出代價，最主要的就是無法在錯誤時，提供足夠精確的 error message．

### 7.2 正常的程式流程

try/catch 的 catch 只去處理錯誤的回復，確保程式不會卡在這裡，但是不要把商業邏輯坐在 catch 處理，也就是把錯誤發生情況給常態化，這會造成後續其他開發者，無法區分商業邏輯和錯誤處理的差異，也就是不好讀，光是這樣就違反了 clean code.

### 7.3 不要回傳 null

『许多语言（C，C++，Java，C#，……）的类型系统对于null的处理，其实是完全错误的。这个错误源自于Tony Hoare最早的设计，Hoare把这个错误称为自己的“billion dollar mistake”，因为由于它所产生的财产和人力损失，远远超过十亿美元。』

有人回傳 null 那就代表有人要處理 null ， 很好這時候個人確認一下手上專案的 LPageLayout.java ，大概總共寫了約４０個 null 檢查，這完全就是給自己找麻煩的行為，這時候有人可能認為可以用 NullPointerException 來省下大量的 null check , 但這時候和第一項的『try/catch 最好能定義在最精準的範圍：』完全衝突，如果範圍大，確實可以省下大量的 null check ， 但是第一項的麻煩也會跟者產生。

因此最好的方法就是： 不要回傳 null

最常出現這種情況就是，List 的 null check ，

```
List personalList = loader.getPersionalData();
if (personalList != null) {
    for (int i = 0; i < personalList.size() ; i ++ ) {
        xxxxx;
        // 如果這時候牽扯到多執行緒那就更刺激了
    }
}
```

因此這種情況下，我們完全建議 loader.getPersionalData() 回傳一個 list size is 0 的 empty list，可以用

```
Collections.emptyList();
```

來產生一個空 list

### 7.4 不要傳遞 null

比回傳 null 還要糟糕的就是傳遞 null 參數到函式裡，大部分函式的參數處理都不會預想到輸入參數是 null 的情況（基本型別想到０的情況可能還有），大部分就直接爆個 NullPointerException 給你，這造成要碼是你做 null check ，不然就是函式特別對參數做 null check （建議利用 @NonNull），然後爆 InvalidArgumentException，我想這劇情沒有好太多，一樣搞死大家.

因此同樣最好的方法就是 不要傳遞 null

—–第一節先到這邊－－－－

Clean code: 無暇的程式碼的第一段心得先寫到這裡

總共有１７章，繼續整理

ps.原本下一章以為會寫很長的

如果有任何違反著作權的情事發生，請主動告知，小弟會快速下架
