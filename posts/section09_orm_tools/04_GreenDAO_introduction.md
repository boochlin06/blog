# Android ORM Library- Green DAO（一） introduction

> 原文網址：https://boochlin.com/?p=85
> 發布日期：2015-10-23
> 文章編號：85

---

** [GreenDao Introduction](//www.slideshare.net/secret/rT2IItArugcUc9) **

『greenDAO is an open source project to help Android developers working with data stored in SQLite. 』

前言很簡單，就是我可以幫助你存取SQLite

但是你可能會想，連SQLite都不會 還來跟人家學寫啥Android，退下

but要記住會寫跟寫的好，這是兩碼子事情

而且身為OOP的支持者，應該都不是會喜歡SQLite script and Cusor 的硬漢

廢話結束，先強調在前，這是一篇科普ORM and GreenDao 的介紹文，不會有太高深的東西

有興趣的話接下來就慢慢一頁一頁講解

### page 1-2

#### Outline

這次講解分為五個段落

1. Why greendao

最根本的原因是啥？不直接用Android 的sqlite

2. Feature —- 有啥花招（加速開發）

3. Performance — 速度（執行速度）

4. How to getting start

5. How to use

開始玩吧

### page 3

#### 究竟為何要使用green dao?

這就要回到開發Android database 的根本上，有些東西是單純Android 開發者不喜歡的事情

就像是寫 Content resolver and SQLite script 這種令人覺的渾身不對勁的東西，一流開發感整個不見

再仔細想想原因，就是這幾點不爽

1. DB 和 Curosr 一點都不Object-Oriented Programming, OOP，物件導向程式設計。

2. 寫SQLite script 一點都不高階，感覺就是組句子而已

3. 不高階就算了，老是在select * 這種差不多的句子

4. 其實我不知道原來組SQLite script 裡面不能用 『||』這種上cs101就該知道的東西

5. 好啦，我承認我超弱的

為何我要花那某多時間去介紹一套新的lib，簡單來說就是team裡面的小夥伴(我)需要教育

 

### page 4

#### It is popular?

好不好用不能只聽我說之其實他很流行

至少pinterest 總聽過了吧

### page 5

#### Features

**1. 快**

**2. ORM （Object Relational Mapping，簡稱ORM，或O/RM，或O/R mapping）**

物件導向是從軟體工程基本原則（如耦合、聚合、封裝）的基礎上發展起來的，而關聯式資料庫則是從數學理論發展而來的，兩套理論存在顯著的區別。為了解決這個不匹配的現象，物件關聯對映技術應運而生。（from wiki）

**2.1 Code generate**

幫你搞定dirty code的部份，就是以下幾個topic

**2.2 Read and write object.**

ORM最大目的就是降低開發上物件與資料庫的不批配問題，對於前端工程師來說則是希望輸入和輸出都只是個物件，最好能只利用物件層級就完成所有對db的處理

而在開發上最常見的第一步就是把每一列資料視為一個物件，將column的資料一個一個對應的塞進物件，也就是最常見的 **setter and getter**

example:

```
Car car = new Car();
car.setHeight(100);  car.getHeight()
car.setHeight(100);  car.getHeight()
```

這通常就是一段dirty code，畢竟有幾項參數就要寫幾個 setter and getter

**2.3 Create table done for you**

在Android SQLite 的語法中創造 table 是非常固定的寫法

因此順便幫忙generate 出來

example:

```
db.execSQL("CREATE TABLE " + constraint + "'LeftPage' (" + //
                "'DataID' INTEGER PRIMARY KEY ," + // 0: DataID
                "'Category' TEXT," + // 1: Category
                "'Provider' TEXT," + // 2: Provider);
```

**2.4 Experessing queries**

在2.2的介紹中提到，希望所有對處db的處理都能以物件方式來對應

這最大的好處就是可以更容易的使用 query 功能，詳細的介紹可以看之後page

不過至少可以確保query 後回傳的東西是你最習慣的 list<T> 。

**3. Session cache**

一大重點，但是這次主要是科普教育，並不多談

### page6-7

#### Performance

效能永遠是最重要的一環，自己手動測試

insert and delete 10000 row

 load 20000 row

基本上最可怕的差異在於 insert ，不過這是在沒有 transcation的情況，有點不公平

不過看load的時間，就可以發現差異啦

另外也可以看大師跟其他orm lib 的比較[benchmark](https://github.com/daj/android-orm-benchmark)

delete到是沒有差多少

### page 8

#### ORM in greendao

![ORM-GREENDAO](http://greendao-orm.com/wordpress/wp-content/uploads/greenDAO-orm-320.png)

**GreenDao**

所有的 SQLite database 的操作全部都透過greendao

而輸出輸入的對象都是以java object 為標的。這是最大目的。

其實類似的東西就是 hibernate，只不過hibernate 更大尾

### page 9

#### A example for present

先為了確保之後我在介紹時，介紹的例子是連貫的，先來個基本定義

打算作一個 social data collector，裡面會收集各種 social ap 的資料

全部都存成 CommonData 這個物件，裡面包含這些欄位

DataID as primary key

 Category 類別

 Provider 提供者 ：facebook, twitter…

 DisplayOrder 根據big data 後的排序權重

### page 10-15

#### Expressing Queries: What difference

**這幾頁都在談，何謂的greendao Expressing Queries 他與Android 自帶的 content provider 差在哪裡**

content resolver 真的超煩

page 10 只是拿loadAll()來暖暖場，畢竟拿這個method來比一點都不彈性化

### page 11-12

根據之前的定義，假設要去 query 符合以下條件的式子

Provider equal facebook

 Order by display order

 Limit data count100

以Android sql 來看就是

code:

```
String selection = StorageField.Provider. + "=?" ;
String[] selectionargs = new String {“facebook”} ;
String sortorder = “ ASC ” + StorageField.Displayorder + “ LIMIT ” + "100";
Cursor c = getContentResolver() . query (uriLPageProvider , projection  ,  selection, selectionargs, sortorder ) ;
```

也許你是個大師，可以寫的更簡潔一點

code:

```
Select * from leftpage  where Provider = ‘facebook’ order asc ‘displayorder’ limit ‘100’
```

恭喜你完成SQLite script

但還有一關：就是把cursor 變成 list，其實這超花時間

**但是這在 GreenDao裡面就只要

**code:

```
mCommonDataDao.queryBuilder()
   .where(Properties.Provider.eq(Provider.Facebook))
   .orderAsc(Properties.DisplayOrder)
   .limit(100)
   .list();
```

好吧，這整個變得超乾淨又好懂，多某美好的世界寫到有點累了

### page 16-17

#### How to getting start?

第一次總是比較麻煩一些

![](http://greendao-orm.com/wordpress/wp-content/uploads/greenDAO-Projects-640.png)

由此圖看以看得出來，使用dao 必須先創造dao (好像廢話)

建立 greendao generator project （normal java class）去產生（code generateion）

daos 相關檔案到Android project 以供使用

### page 18-23

#### Step by step

**Step 1:** 創建 greendao 相關code 必須先在一般class寫 greendao-generator 的code

以Android studio 來看就是先創建 module

然後 gradle script 加入 greendao-generator的引用

code:

```
compile 'de.greenrobot:greendao-generator:1.3.1
```

**Step 2:** 接下來在 module 開normal class 名子隨便，在裡面開始設定有關 db的屬性（欄位，位址…）

code:

```
public class GreenDao {
    public static void main(String[] args){
        // 創建基本 schema 指定 version and package
        Schema schema = new Schema(1, "com.acer.android.leftpage.provider");

        // 利用 schema 開始設定 entity, 
        // 設定 dao name
        Entity commonData = schema.addEntity("CommonData");
        // 設定 table name
        commonData.setTableName("LeftPage");
        // 至少指定一個 primary key
        commonData.addLongProperty("DataID").primaryKey();
        // 其他 table column name
        commonData.addStringProperty("Category");
        commonData.addStringProperty("Provider");
        commonData.addLongProperty(“DisplayOrder”);
        // 設定 add content provider 會自動連 content provider 固定的code 給建起來
        // 建好也不能馬上用，必須跟者修改 manifest 和 部份code
        commonData.addContentProvider();
        // 利用 generator 指定 code generate 位址 ，跟package 不同，請指定合法實體路徑
        // 通常就是對應到你要使用greendao 的 android project path.
        DaoGenerator generator = new DaoGenerator();
        generator.generateAll(schema, "../AcerHome/LeftPage/src/main/java/");}
```

**Step 3:**完成後，請直接 compile and run to generate code.

通常就會在你指定的路徑下產生底下四個檔案

![core_class](http://greendao-orm.com/wordpress/wp-content/uploads/Core-Classes-150.png)

對應到上面的code就是就是底下這四個core class

**1. DaoMaster**

『The entry point for using greenDAO. DaoMaster holds the database object (SQLiteDatabase) and manages DAO classes (not objects) for a specific schema. It has static methods to create the tables or drop them.』

主要管理對 SQLiteDatabase 的基本操作（drop , create, find），以及對 dao 的操作

也是所有對 greendao 的操作的起始點。

code:

```
package com.acer.android.greendao;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteDatabase.CursorFactory;
import android.database.sqlite.SQLiteOpenHelper;
import android.util.Log;

import de.greenrobot.dao.AbstractDaoMaster;
import de.greenrobot.dao.identityscope.IdentityScopeType;
// 很貼心的提醒，別亂動阿
// THIS CODE IS GENERATED BY greenDAO, DO NOT EDIT.

/**
 * Master of DAO (schema version 1): knows all DAOs.
 */
public class DaoMaster extends AbstractDaoMaster {
    public static final int SCHEMA_VERSION = 1;
    /**
     * Creates underlying database table using DAOs.
     */
    public static void createAllTables(SQLiteDatabase db, boolean ifNotExists) {
        CommonDataDao.createTable(db, ifNotExists);
    }
    /**
     * Drops underlying database table using DAOs.
     */
    public static void dropAllTables(SQLiteDatabase db, boolean ifExists) {
        CommonDataDao.dropTable(db, ifExists);
    }
    public static abstract class OpenHelper extends SQLiteOpenHelper {
        public OpenHelper(Context context, String name, CursorFactory factory) {
            super(context, name, factory, SCHEMA_VERSION);
        }
        @Override
        public void onCreate(SQLiteDatabase db) {
            Log.i("greenDAO", "Creating tables for schema version " + SCHEMA_VERSION);
            createAllTables(db, false);
        }
    }
    /**
     * WARNING: Drops all table on Upgrade! Use only during development.
     */
    public static class DevOpenHelper extends OpenHelper {
        public DevOpenHelper(Context context, String name, CursorFactory factory) {
            super(context, name, factory);
        }
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            Log.i("greenDAO", "Upgrading schema from version " + oldVersion + " to " + newVersion + " by dropping all tables");
            dropAllTables(db, true);
            onCreate(db);
        }
    }
    public DaoMaster(SQLiteDatabase db) {
        super(db, SCHEMA_VERSION);
        registerDaoClass(CommonDataDao.class);
    }
    public DaoSession newSession() {
        return new DaoSession(db, IdentityScopeType.Session, daoConfigMap);
    }
    public DaoSession newSession(IdentityScopeType type) {
        return new DaoSession(db, type, daoConfigMap);
    }

}
```

**2. DaoSession**

『Manages all available DAO objects for a specific schema, which you can acquire using one of the getter methods. DaoSession provides also some generic persistence methods like insert, load, update, refresh and delete for entities. Lastly, a DaoSession objects also keeps track of an identity scope.』

利用 DaoMaster 可以拿到 DaoSession

DaoSession 主要提供最常見的 CRUD 方法，並對identity scope作cache 跟蹤

code:

```
package com.acer.android.greendao;
import android.database.sqlite.SQLiteDatabase;
import java.util.Map;
import de.greenrobot.dao.AbstractDao;
import de.greenrobot.dao.AbstractDaoSession;
import de.greenrobot.dao.identityscope.IdentityScopeType;
import de.greenrobot.dao.internal.DaoConfig;

import com.acer.android.leftpage.greendao.CommonData;

// THIS CODE IS GENERATED BY greenDAO, DO NOT EDIT.

/**
 * {@inheritDoc}
 *
 * @see de.greenrobot.dao.AbstractDaoSession
 */
public class DaoSession extends AbstractDaoSession {
    private final DaoConfig commonDataDaoConfig;
    private final CommonDataDao commonDataDao;
    public DaoSession(SQLiteDatabase db, IdentityScopeType type, Map<Class<? extends AbstractDao>, DaoConfig>
            daoConfigMap) {
        super(db);
        commonDataDaoConfig = daoConfigMap.get(CommonDataDao.class).clone();
        commonDataDaoConfig.initIdentityScope(type);
        commonDataDao = new CommonDataDao(commonDataDaoConfig, this);
        registerDao(CommonData.class, commonDataDao);
    }
    public void clear() {
        commonDataDaoConfig.getIdentityScope().clear();
    }
    public CommonDataDao getCommonDataDao() {
        return commonDataDao;
    }
}
```

**3. CommonDataDao**

『Data access objects (DAOs) persists and queries for entities. For each entity, greenDAO generates a DAO. It has more persistence methods than the DaoSession, for example: count, loadAll, and insertInTx.』

daos 提供更多更精細的操作（count, loadAll, and insertInTx）在 daosession 的base 上

，而我們操作的起點通常就是拿個commondao 來進行 crud， 相較之下啥 session , master反而還沒啥再用。

code 部份主要分成3塊

**3.1. 提供 static class: Properties 用來幫助對column 的 reference

**code:

```
/**
     * Properties of entity CommonData.
     * Can be used for QueryBuilder and for referencing column names.
     */
    public static class Properties {
        public final static Property DataID = new Property(0, Long.class, "DataID", true, "DataID");
        public final static Property Category = new Property(1, String.class, "Category", false, "Category");
        public final static Property Provider = new Property(2, String.class, "Provider", false, "Provider");
        public final static Property DisplayOrder = new Property(3, Integer.class, "DisplayOrder", false, "DisplayOrder");
        // ...... 還有很多很多
    }
```

**3.2. create table : 之前有提過了，這種固定戲碼的東西當然是自動generate

**code:

```
public static void createTable(SQLiteDatabase db, boolean ifNotExists) {
        String constraint = ifNotExists? "IF NOT EXISTS ": "";
        db.execSQL("CREATE TABLE " + constraint + "'LeftPage' (" + //
                "'DataID' INTEGER PRIMARY KEY ," + // 0: DataID
                "'Category' TEXT," + // 1: Category
                "'Provider' TEXT," + // 2: Provider
                "'DisplayOrder' INTEGER ）"); + // 3: DisplayOrder
        }
```

**3.3. bindvalue : 就是 object to cursor or cursor to object.**

不上code

**4. CommonData**

『Persistable objects. Usually, entities are generated (you do not have to), and are objects representing a database row using standard Java properties (like a POJO or a JavaBean).』

這裡就是大量的 setter and getter，這種dirty code 當然是 auto generate阿

### page 23

**Step 4: **目前generate 出來的檔案在你的android project 裡面

但是應該還是不能用，因為很多奇妙的 class 卻引用不到

因此請在 project gradle 加入

code:

```
compile 'de.greenrobot:greendao:1.3.7'
```

### page 24-25

#### HOW TO USE

終於可以開始使用啦，只要記住有啥事就直接找 CommonDao 來處理，一切沒問題

initial 方式如下

code:

```
private static final String DB_NAME =  “LeftpageProvider.db";
private DaoSession daoSession;
CommonDataDao mCommonDataDao
private DBHelper(Context context) {
     // 直接指定db name ，這比較少見
    DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(context, DB_NAME, null);
    DaoMaster daoMaster = new DaoMaster(helper.getWritableDatabase());
    daoSession = daoMaster.newSession();
    mCommonDataDao= daoSession.getCommonDataDao();
}
```

### page 26-27

#### Query

老實說我不知道還有啥需要講解，這是最基本用法

也多了不少在 content resolver 不太會用到的寫法,但在sql 很常用到的敘述句

**Query with syntax**

相同：eq , notEq

大於小於：ge , le , gt , lt

other:in , between,like…..

code:

```
// 先做出queryBuilder 後，開始填條件式
List dataList = CommonDataDao.queryBuilder()
   .where(Properties.Provider.eq(Provider.Social))
   .orderAsc(mCommonDataDao.Properties.DisplayOrder) .list();
```

如果還是很眷戀 cursor and SQLitedatabase

幫你保留 raw query

code:

```
SQLiteDatabase db = daoSession.getDatabase();
Cursor cursor = db.rawQuery("SELECT *FROM leftpage WHERE .....", null);
```

### page 28

#### Insert

基本用法如下，另外想要多多宣導，塞大量資料時請善用 bulk insert 以改善效率～～謝謝

SQLitedatabase 就是請block 後在塞大量資料

另外小小提醒，primary key 請保持為null ，由系統幫你填值（我知道這是常識，但是總有小夥伴..）

code:

```
//Insert:
    CommonDataDao .insert(new CommonData(DATA_ID, arg1, arg2,x……));
//Bulk Insert 
    CommonDataDao .insertln(CommonDataList);
    CommonDataDao.insertln(CommonData[]);
//InsetOrReplace
    CommonDataDao.insertOrReplaceln(CommonDataList)
```

### page 29

#### Delete

基本用法如下

code:

```
// Delete sentence
// 跟query一樣，先利用 QueryBuilder 把條件式給列出來，但是最後運行的方法事根據條件式
// 運行 delete command
CommonDataDao.QueryBuilder()
  .where(Properties.Provider.eq(facebook))
  .buildDelete()  .executeDeleteWithoutDetachingEntities();
// Delete all
CommonDataDao.deleteAll();
```

### page 30

#### Update

該注意的點基本上跟insert 一樣

就是 拜託善用 bulk update ，還有搞清楚 primary key 的用途及意義

code:

```
// Update sentence
CommonDataDao.update(new CommonData(Data_ID , arg1 , arg2 , …….));
// Bulk Update 
CommonDataDao.updateln(CommonDataList);
CommonDataDao.updateln(CommonData[]);
```

### page 31

#### Thanks

Reference:

[http://greendao-orm.com/features/](http://greendao-orm.com/features/)

[http://bng86.gitbooks.io/android-third-party-/content/greendao.html](http://bng86.gitbooks.io/android-third-party-/content/greendao.html)

[http://www.slideshare.net/SmileeYang/greendao-43892958?qid=b95e0f35-a323-4930-8e8a-d0e298a1eb10&v=qf1&b=&from_search=1](http://www.slideshare.net/SmileeYang/greendao-43892958?qid=b95e0f35-a323-4930-8e8a-d0e298a1eb10&v=qf1&b=&from_search=1)

[http://www.slideshare.net/ssuser8674c1/green-dao-50914708?qid=b95e0f35-a323-4930-8e8a-d0e298a1eb10&v=qf1&b=&from_search=5](http://www.slideshare.net/ssuser8674c1/green-dao-50914708?qid=b95e0f35-a323-4930-8e8a-d0e298a1eb10&v=qf1&b=&from_search=5)

### page 32

#### 其實這是個契子

greendao 不只有上述這樣基本的功能，還有很多很強但是我看不懂的東西

還有很多更加進階的功能，如果還有下一章節的話，會開始介紹其他我在開發上導入的問題

謝謝觀賞 長文科普一篇
