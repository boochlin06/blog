# GreenDao 3 Introduction

> 原文網址：https://boochlin.com/?p=226
> 發布日期：2016-10-31
> 文章編號：226

---

![greendao-vs-ormlite-vs-activeandroid](../../assets/images/greenDAO-vs-OrmLite-vs-ActiveAndroid.png)

[https://boochlin.com/wp-content/uploads/2016/10/greenDAO-vs-OrmLite-vs-ActiveAndroid.png](https://boochlin.com/wp-content/uploads/2016/10/greenDAO-vs-OrmLite-vs-ActiveAndroid.png)

首先先來一張 GreenDao 3 官方的自吹自擂，雖然看起來跟 GreenDao 2 差不多，就是尬爆其他目前所有 ORM Lib 的效能，會這樣說是因為之前就是用 GreenDao 2 作為主要開發 DB 框架，當時就看到同一張圖表。詳情可以看這一篇 『[Android ORM Library- Green DAO（一） introduction](https://boochlin.com/?p=85)』。基於以上理由，這次更新，自然就開開心心得來試試看。至於好處壞處使用原因情境，全部跟前一篇一模一樣，不再贅述。

雖然 Greendao 2 時用的就不是很深入，但是現在一比較發現，這次改進最大的就是建制基本 DAO 的方法，活用了 android studio 中的 gradle plugin。會這樣說就是以前的方法實在不好上手，必需要先在 project 外建置一個 java file，對，就是有一個 public static main 的那種程式，然後把欄位寫進去，在搭配 Gradle 的插件『greendao generator』才能自動生成 DaoMaster , DaoSession , xxxDao , xxxEnity 等 Class，這件事大大地讓我不想要隨便去改動 db ，因為實在麻煩，而且那個 java file 根本不會在用到，可說是廢 code.

而現在則是大大改進那一堆我都不知道上面再說啥的部份，因此接下來慢慢一步一步得來解說到底變成怎麼樣了

#### step 1. 一樣先在 build.gradle 裡加入

```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.0'
    }
}

apply plugin: 'org.greenrobot.greendao'

dependencies {
    compile 'org.greenrobot:greendao:3.2.0'
}
```

#### step 2 . 然後厲害的部份就來啦，可以開始創 dao

這邊以官方範例 note 物件來寫，以 id , text , date 作為欄位名稱。只要在要使用的專案中多加個 class ， 像是 Note.java ，並在 class 上加入 annotation Entity 就可以搞定， 接下來根據需求設定其他 annotation 像是 Id , NotNull  , Unique 之類的。

```
@Entity
public class Note {
 
    @Id
    private Long id;
 
    @NotNull
    private String text;
    private Date date;
}
```

補充：

* The **@Id** annotation selects a long/ Long property as the entity ID. In database terms, it’s the primary key. The parameter **autoincrement** is a flag to make the ID value ever increasing (not reusing old values).

* **@Property** lets you define a non-default column name, which the property is mapped to. If absent, greenDAO will use the field name in a SQL-ish fashion (upper case, underscores instead of camel case, for example customName will become CUSTOM_NAME). *Note: you currently can only use inline constants to specify a column name.*

* **@NotNull** makes the property a “NOT NULL” column on the database side. Usually it makes sense to mark primitive types (long, int, short, byte) with @NotNull, while having nullable values with wrapper classes (Long, Integer, Short, Byte).

* **@Transient** marks properties to be excluded from persistence. Use those for temporary states, etc. Alternatively, you can also use the **transient** keyword from Java.

#### step3 . Note.java 寫好後直接按 project make ，然後喝個咖啡， Note.java 就會搖身一遍成…..

『一個加強版的 Note.java』

多了一堆有的沒有的 setter , getter ，沒錯這樣就完成啦，一個全新的 entity，另外啥小 DaoMaster , DaoSession , xxxDao 也都跟者一起跑出來。預設 package 會是 note.java 的 package 。gen dir 是 src/java/main

```
@Entity(indexes = {
    @Index(value = "text, date DESC", unique = true)
})
public class Note {

    @Id
    private Long id;

    @NotNull
    private String text;
    private String comment;
    private java.util.Date date;

    @Convert(converter = NoteTypeConverter.class, columnType = String.class)
    private NoteType type;

    @Generated(hash = 1272611929)
    public Note() {
    }

    public Note(Long id) {
        this.id = id;
    }

    @Generated(hash = 1686394253)
    public Note(Long id, @NotNull String text, String comment, java.util.Date date, NoteType type) {
        this.id = id;
        this.text = text;
        this.comment = comment;
        this.date = date;
        this.type = type;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    @NotNull
    public String getText() {
        return text;
    }

    /** Not-null value; ensure this value is available before it is saved to the database. */
    public void setText(@NotNull String text) {
        this.text = text;
    }

    public String getComment() {
        return comment;
    }

    public void setComment(String comment) {
        this.comment = comment;
    }

    public java.util.Date getDate() {
        return date;
    }

    public void setDate(java.util.Date date) {
        this.date = date;
    }

    public NoteType getType() {
        return type;
    }

    public void setType(NoteType type) {
        this.type = type;
    }

}
```

如果不喜歡這個生成的位址。就在 gradle 加入設定

```
greendao {
    schemaVersion 2
    daoPackage  'com.android.heaton.database'
    targetGenDir 'src/main/java'
}
```

補充：

* **schemaVersion:** The current version of the database schema. This is used by the *OpenHelpers classes to migrate between schema versions. If you change your entity/database schema, this value has to be increased. *Defaults to 1.版本號*

* **daoPackage**: The package name for generated DAOs, DaoMaster, and DaoSession. *Defaults to the package name of your source entities. 生成 Package PATH*

* **targetGenDir:** The location where generated sources should be stored at. *Defaults to the generated source folder inside the build directory* ( build/generated/source/greendao).

* **generateTests:** Set to true to automatically generate unit tests.

* **targetGenDirTests:** The base directory where generated unit tests should be stored at. *Defaults tosrc/androidTest/java*.

#### **step 4.  到這階段就是一般 CRUD**

Query:通常都會使用 QueryBuilder 作為query的主要輔助工具，where , or 之類的 sql statement 用物件的方式表現出來

```
QueryBuilder qb = userDao.queryBuilder();
qb.where(Properties.FirstName.eq("Joe"),
qb.or(Properties.YearOfBirth.gt(1970),
qb.and(Properties.YearOfBirth.eq(1970), Properties.MonthOfBirth.ge(10))));
List youngJoes = qb.list();
```

* **list()** All entities are loaded into memory. The result is typically an ArrayList with no magic involved. Easiest to use. 最常見的一班用法，返回 ArrayList

* **listLazy()** Entities are loaded into memory on-demand. Once an element in the list is accessed for the first time, it is loaded and cached for future use. Must be closed. 分階段讀取，當真的需要時，才會讀取。

* **listLazyUncached()** A “virtual” list of entities: any access to a list element results in loading its data from the database. Must be closed. 不經過 memory cache，直接去 db access 

* **listIterator()** Let’s you iterate through results by loading the data on-demand (lazily). Data is not cached. Must be closed. 基本上同第一個

**Insert:**

```
// Insert one object
NoteDataDao.insert(new NoteData());
// Insert and update object
NoteDataDao.insertOrReplace(new NoteData());
// block insert objects
NoteDataDao.insertInTx(List list);
```

**Update:**

```
NoteDataDao.update(new NoteData());
NoteDataDao.updateKeyAfterInsert(new NoteData() , 2);
NoteDataDao.updateInTx(list);
```

 

**Delete**

```
NoteDataDao.deleteAll();
NoteDataDao.deleteByKey(list);
NoteDataDao.deleteInTx(list);
//  最常見的還是給予條件式然後刪除符合條件的物件，這就一樣需要 query builder 
//  去 build 出 delete sql statement  
NoteDataDao.queryBuilder(XXX condition ).buildDelete().executeDeleteWithoutDetachingEntities();
```

ps. Greendao 3 依然沒有可以設定 entity 欄位初始值功能，詳情依樣可以看[Android ORM Library – Green DAO（二）Question: Why can’t set default value for entities](https://boochlin.com/?p=92)

因為之前已經寫過一篇很詳細的，這次就不詳述，只談差異部分，感謝收看

 

reference:

[http://greenrobot.org/greendao/documentation/](http://greenrobot.org/greendao/documentation/)

[Android数据存储之GreenDao 3.0 详解](http://www.cnblogs.com/whoislcj/p/5651396.html)
