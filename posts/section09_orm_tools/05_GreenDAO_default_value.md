# Android ORM Library – Green DAO（二）Question: Why can’t set default value for entities

> 原文網址：https://boochlin.com/?p=92
> 發布日期：2015-11-11
> 文章編號：92

---

### Question: Why no set default value for entities

其實一開始標題不是這樣，而是 some tips，但是光是聊第一個問題就兩千多字，所以乾脆切出一個新的。另外建議先看[第一篇](https://boochlin.com/?p=85)，或是已經有相關知識

**topic：Why can’t set default value for entities**

### Question:

**為何 GreenDAO 在建立Entity階段沒有辦法對每個 property 設定初值？**

結果就是以下的 code 無法用GreenDAO generator 的相關 method 自動產生出來。原因出在 DEFAULT -1 這敘述句沒辦法自動產生。

```
db.execSQL("CREATE TABLE " + constraint + "'LeftPage' (" + //
        "'DATA_ID' INTEGER PRIMARY KEY ," + // 0: DataID
        "'CATEGORY' TEXT NOT NULL ," + // 1: Category
        "'PROVIDER' TEXT," + // 2: Provider
        "'DISPLAY_ORDER' INTEGER DEFAULT -1））;"); // 3: DisplayOrder
```

這問題當然不只我一個人想到，畢竟 Default value 幾乎是 sql 的起手式。

隨便收集一下

[解决android greenDAO没有默认值default value和没有原始数据类型的问题](http://blog.csdn.net/enyusmile/article/details/45498307)

[Android – add default value in GreenDao database](http://stackoverflow.com/questions/18823795/android-add-default-value-in-greendao-database)

[Adding default values for entities](https://groups.google.com/forum/#!topic/greendao/cbbUkP6DdHs)

根據上面，甚至出現要求作者加入此功能的連署聲音

[Default Values for Entity Properties](https://github.com/greenrobot/greenDAO/issues/17)

通常有關 sqlite script 的這段創建 table 的 code 是由GreenDAO generator 在設定 Entity 後自動建出來的：如下

```
Entity commonData = schema.addEntity("CommonData");
commonData.setTableName("LeftPage");
//commonData.addContentProvider();

commonData.addLongProperty("DataID").primaryKey();
commonData.addStringProperty("Category");
commonData.addStringProperty("Provider");
commonData.addIntProperty("DisplayOrder");
```

前面所有的欄位name , primary key , data type 都可以在建制 Entity 時指定好，唯一的遺憾就是為何不能在這階段加入 DEFAULT value 進而去影響GreenDAO generator 產生的 sql create table script code ，做個類似的api 不就搞定了 像是：setDefaultValue(String xxx) or DefaultValue(int xxx) 。

相信這絕對不難[解决android greenDAO没有默认值default value和没有原始数据类型的问题](http://blog.csdn.net/enyusmile/article/details/45498307)這位大大就自己搞了一套，如果看官確實想要這樣的功能的話，請自取。

如果懶得拿就直接修改 xxxDao.java 裡面的 sql script（像最上面這段code） , 這樣最快

**那某我就想知道為何GreenDAO 開發者為何不導入這樣的功能，而且看起來2012 就有人在喊這件事，但是作者還是不搞**

以下開始就是個人見解，說不定有點強解作者意圖 『為何GreenDAO 不讓使用者可以自由設定 default value ？』

拆成幾個步驟來看這問題

**1. 基本上只修改 create table sqlite 對orm 操作並不完整**

簡單來說 通常倒入orm 後，insert 通常都會是以 object 為對象進行操作，而非是Content value，所以當你進行以下操作時

```
CommonDataDao.insert(new CommonData());
```

結果預定要設  default value  的欄位，還是塞入 null 進去，因為當你new object 時，裡面的欄位變數初值還是null，不管type 是 int/double/string 都一樣，因為 object 裡面的每一個欄位並不是 基本資料型類別(PRIMITIVE TYPE)，而都是 外包類別(wrapper classes)。

 這造成你不特別 assign 的話就是null , 連基本資料型類別的初值都沒有：當然如果是用 content value 搭陪content resolver還是有用的。不過那這樣的話你就不應該用 orm 阿

**2. 那就在 object 裡面設定初值阿，或者特定setArg(int value);**

直接去修改CommonData.java，例如：定義欄位時直接給值

```
private String category;
    private String mProvider;
    private Integer DisplayOrder = -1;
```

看起來到這邊，至少處理 insert 是對的

```
CommonDataDao.insert(new CommonData());
```

**3. 但是如果接下來是處理update的情況呢？**

```
CommonData data = new CommonData();
    data.setDataID(target_id);
    CommonDataDao.update();
```

這會發生啥情況呢？ 由於update資料時事根據 primary key 作為取代目標的 index

 所以一定要先指定 DataID 作為取代目標的id，但是這次update的對象裡並沒有要修改 display order

 那某data 的 display order 就會維持初值 -1 ，進而去更新 db 的那欄資料，很明顯這樣會很糟糕，因為你弄壞資料庫啦。

其實問題沒那某大，只要記得 update 之前，assign null to display order 進去就可以：

或者有人會反駁，通常都是從 db 拿出來後，然後修改其中幾項就在 update 回去

沒錯，這樣就沒有問題啦，甚至說只要開發者小心一點，就一切安康

對於GreenDAO framework 開發者來說絕對可以很快的把 setDefaultValue(int xxxx); 給做出來

 但是他沒有辦法在 framework 層級幫你處理掉後續我上面提到 side effect

最多可以在insert 時 new CommonData() 自動幫你加入設定的初值，但是後續的 update 大概就沒辦法那某簡單處理掉（說不定有辦法，但是我目前想不到）

**簡單來說就是後續問題多多。**
