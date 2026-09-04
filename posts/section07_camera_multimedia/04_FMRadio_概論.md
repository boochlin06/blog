# FMRadio 開發筆記 （一）-概論

> 原文網址：https://boochlin.com/?p=8
> 發布日期：2015-02-08
> 文章編號：8

---

——-廢話開始——

身為FMRadio第一篇紀錄用的文章

先來申明，這是開工後第一份的開發任務，當年在記錄開發心得，並沒有想說要貼到blog上面，所以就只是用google doc 去記錄相關資料，這樣做的後果就是完全不考慮錯字以及排版問題，而且很多情況大概只有我看得懂，如blog title 一樣  只是 something record for me。

未來的幾個系列都有可能會這樣，實在糟糕，未來再慢慢改好了

此外我個人是很想弄出其他網站上的那種帥氣code hight light，但是發現這種事情還是不要往前追朔，因為實在太麻煩了。至少前幾個系列都不太想用。

——-正文開始——

開發FMRadio其實相當麻煩，主要在於各層級都會碰到，與底層不少溝通，與一般單純寫third party app有不少差異

廢話不多說

FMRadio 開發上主要來說可以切成這幾大元件。

### ****[1. FMRadioMain](https://boochlin.com/?p=5%20)

* 1.1主要於一開始進入頁面的配置，針對當searching , scanning,頻道轉換時進行頁面配置

* 1.2對FmRadioService進行bindservice的互換溝通由於FMRADIO必須對service進行遠端非同步的溝通,因此採用aidl的bindservice mode bindservice的callback函數都放至於此,因此其他activity諾想使用fmservice則通常經過broadcast的方式

### ****[2. FMRadioService](https://boochlin.com/?p=13)

* 2.1 對底層硬體的控制.searching ,scanning ,disable ,fmoff 之類的….需要細細控制

* 2.2 對外部事件的控制.hdmi.headset,sd card airplane mode control.audiomanger..他媽的這裡超麻煩

### **3. FmShareprefrence**

* 3.1 對於所有可能使用的資料，參數，國籍，電台儲存，tuneFrequency的儲存

* 3.2 大部分電台的儲存採用PresetStation和PresetList的類別

### **4. TabstationActivity**

* 採用tabview的方式安放資料，根據spec可分成

* 4.1 TabFavorite 此頁功能最多，針對電台提供我的最愛參數設計，並提供大量功能…favoriteArrange的功能大概是最麻煩的

* 4.2 TabHistroy 針對最近聽過的電台，整理於此，困難處在FMRadioMain何時要將電台插入到histroy list

* 4.3 TabScan 相較先前兩頁，這頁算是功能比較單純的一頁，scan 相對訊號強的電台列表。不過由於跟新是採用service的遠端資料更新，所以在dialog的處理上也要注意。

### ****[5. AlertActivity](https://boochlin.com/?p=14%20)

* 如同名子所示，提供警告的提示dialog，至於為何要獨立出來，可以參考service如何在偵測到broadcast之後，在各個activity中顯示警告。

### **6. Define**

* 較為單純的class.用於模組化可用或必用的數據

### **7. FavoritesScrollView & FavoritesStationView & PolyToPolyAnimation**

* 大概是最麻煩的部份…用於對favoriteItem進行動畫化…進行顯示 基本上是三個一體的

### **8. PresetStation & PresetList**

* 為原生class.紀錄station播放所需的資訊※以及列表呈現的專用class.可搭載排序.搜尋等功能

### 之後幾篇文章會挑幾個比較好談得獨立出來

**ps.FmShareprefrence，****Define，****PresetStation & PresetList 屬於設定值相關class ，因此不予多談**

實用

[android-PhoneStateListener](http://www.google.com/url?q=http%3A%2F%2Fhi.baidu.com%2Fwentaokou%2Fitem%2F717a61d23bb49cbd32db904b&sa=D&sntz=1&usg=AFQjCNFm3ouQrreQM4lm0ntnMn62wjNF8Q)

[android 不常用代码【转】](http://hi.baidu.com/qmiao128/item/d8336d7446f1d92b5c17892a)
