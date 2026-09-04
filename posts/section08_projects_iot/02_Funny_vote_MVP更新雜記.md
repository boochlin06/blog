# Funny vote APP -MVP 更新雜記

> 原文網址：https://boochlin.com/?p=320
> 發布日期：2021-09-11
> 文章編號：320

---

![g294](../../assets/images/g294.png)

[https://boochlin.com/wp-content/uploads/2017/04/g294.png](https://boochlin.com/wp-content/uploads/2017/04/g294.png)

![mvp](../../assets/images/mvp.png)

[https://boochlin.com/wp-content/uploads/2018/09/mvp.png](https://boochlin.com/wp-content/uploads/2018/09/mvp.png)

好一陣子忘記要更新一下 blog，沒想到好不容易建立的開發習慣也會忘記，可見怠惰真的開發的源頭啊(?)，秉持者 something record 的精神，繼續保持好習慣。其實這一篇是 Funny vote 開發札記的後續，主要是記錄釋出後的幾次技術改版練習。順道一題不會教你怎麼改，怎麼用，也沒放上啥程式碼，幾乎都是我的 murmur。

根據 google 官方的 Android arch 的技術藍圖，將 Funny vote 分成幾個 branch ，各自放上想玩玩的技術架構，主要可分為

1. [MVP 架構改版，補上 Unit test by mockito , instrument test by espresso , JaCoCo 整入 travis。](https://github.com/boochlin06/FunnyVote/tree/mvp)

2. [MVP_dagger 在 MVP 架構上使用 Dagger 進行依賴注入。](https://github.com/boochlin06/FunnyVote/tree/mvp_dagger)

3. [MVP_Kotlin 在 MVP 架構上改用 Kotlin 重寫。](https://github.com/boochlin06/FunnyVote/tree/mvp_kotlin)

### MVP 架構改版

先說說最初的程式架構並沒有採用 MVP 架構，而是 MVC ，原因沒甚麼特別的，就是我沒想到要用啊，人只要一急者想出產品時，都只會考慮最快做出來的方法(也不會想要寫 test case)，這並非是說 MVC 是個糟糕但快速的架構(是說到現在 MVC 到底是啥，好像也不是很確定)，而是因為公司的所有project 幾乎都是 MVC的關係，所以最熟悉，可以開發的很快，但是也深深知道他的缺點及優點，且跟 MVP 比較之後，多多少少能理解 google 想力推 MVP 架構的原因(其他還有一堆啦，像是 MVVM , Live Cycle , Clean arch)。

MVC 最主要的問題是讓 Activity 去負責所有的 Controller 邏輯任務，但是本身又負擔 View 展示，用戶介面操作交互，這樣變成 Controller 和 View 的任務混雜再一起，所以常常會看得到超過 2000 行的 Activity ，這非常臃腫，根據 clean code 的概念，這肯定會造成 debug 上的困難，因此 google 提出了切分更乾淨 MVP 架構。

#### VIEW 只負責 UI Init , User 操作互動，不能直接與 Model 溝通，只能接受 Presenter 的指令。

#### Model 只負責處理資料庫互動，提供資料給 Presenter 做決策。

#### Presenter 則負責 View 與 Model 的中介互動，負責所有業務邏輯。

 

跟 MVC 實作上的最大差異則是讓 Activity 只單純地去處理 View 任務，把原本在 Activity 中的 Controller 的邏輯任務全部另外抽出放到 Presenter 裡，所有的交互任務則透過 View interface 和 Presenter interface 來進行，這非常方便於 Unit test。實際程式碼如下

[Funny Vote MVP version Github](https://github.com/boochlin06/FunnyVote/tree/mvp)

在改版成 MVP 架構的同時也補上 UT ，正因為有加入測試的概念，更能理解 MVP 架構的解偶優點。為了方便測試考量，在 Model , Presenter ,View 交互的 interface 設計時，自然而然的會使得 code 變得更乾淨，我想這應該就是 TDD 的理念。雖然我並沒有讓 converge rate 衝高，不過主要是因為自己懶得啥都加上測試，只主要測試核心邏輯業務。

實作開發心得

* 通常一個 VIEW，對應一個 Presenter，但是如果一個 activity 有兩個 fragment view 且自己部分 activity view，像是 Funny vote 創建投票頁面，先不說我當初 UI 設計是怎麼想的，最初的思路認為應該讓一個 CreateVotePresenter 去 Handle setting fragment view and option fragment view 和 activity 的 title and image view。

![Screenshot_20170411-000636](../../assets/images/Screenshot_20170411-000636.png)

[https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170411-000636.png](https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170411-000636.png)

會這樣想，只是單純的認為 activity  跟 fragment  View 之間的互動，可以很簡單的在一個 presenter 中控制，可以少掉很多業務上的 callback ，可以讓 code 變得易讀。但是事情果然不是小弟想的那麼簡單，在經過實作後，發現就會發生一個 presenter 的 code 太多，且太多 view 要一起 ready ， 這在一個人開發時還好，但如果是多人開發時，對於協調 interface設計工作 就會有極大的負擔。

當然這並非是絕對的原則，主要看 widget 的內容量來定義 MVP 會比較好，至於 Callback 太多這個問題，當然不是只有我想到，各方大神早已有解法，也就是 RxJava ，該有的 callback 是少不了，但是可以讓 callback 好創造，好管理，這就是好框架。

* Presenter 不應該有任何相關 android 的 object and method，這沒啥好質疑的，只是我想來自我提醒而已，不然以後寫 UT 只會想哭。Presenter 也不應該傳入 Adapter 裡。

* Model 設計上，如果常常會遇到後端還未 ready 時，常常需要自己先在 local fake data，當後端完成後在重新接上，這種情況下非常適合使用樣板模式，搭配上 Injection 和 build variant 可以簡單區分出何時用何種 content ，超級舒爽。在這個概念之上，同時方便測試的目的，Dagger 的依賴注入框架也因此被提出來。

* 測試框架在沒有特殊需求下，則依據 [google 建議](https://developer.android.com/training/testing/)的 mockito 做 UT, Espresso 做 instrument test，目前感覺就是寫而已，也沒用過別的測試框架，硬要說的話就是 test method 的命名怎麼樣比較好 ([大師說法](http://teddy-chen-tw.blogspot.com/2016/05/blog-post_12.html))，一只以來都是Presenter,View UT 用以想測啥功能作為命名規則，Model UT則多採用預期行為加上測試狀態作為命名規則， Instrument test 則以 Given-When-Then 為命名基準。老實說這樣好不好我也不適很清楚，個人覺得有規則就好。

* CI 這個以前完全沒有摸過，但是聽說 Travis 超簡單，所以我就給他蓋下去了，老實說爽感十足，看他在那邊自動化，我在看 youtube 真的方便

基本功能只驗證能不能 build 過，如果有寫測試的話，則會順手跑完，可是卻發現用 Android studio 都內建顯示 coverage rate 範圍，結果 Travis 沒有順手提供這功能不是太可惜了嗎，因此也整入 jacoco ，測試完後自動 gen report ([大神整合好的](https://github.com/codecov/example-android))。

節錄查資料時看到的名言

> 

You need to provide a way of accessing the member variables so you can pass in a mock (the most common ways would be a setter method or a constructor which takes a parameter).

If your code doesn’t provide a way of doing this, it’s incorrectly factored for TDD (Test Driven Development).

 

### MVP with Dagger

在上一章節談到兩個可以在 MVP 架構上使用的框架，一個是解決大量 Callback 以及流程化的  Rxjava ，另一個是解決大量 new 依賴且方便 Injection 管理的 Dagger2，而這個章節則是談引入 Dagger2 的心得，你說為何沒有一起引入 Rxjava ，難道那些問題不用一起解決嗎，我絕對不會說因為看起來太簡單了，所以我懶得用，而是因為 Kotlin 就直接把這塊整合得更好，雖然我知道這有點雞腿比懶覺的問題，但是因為這之後我打算直接寫 kotlin 版本，所以就忽略了。

根據上面一堆屁話，直接開始 Dagger 整合，通常一開始查找資料都會聽到 MVP + Retrofit + RxJava +Dagger 整個超舒爽，但是接下來就會看到說 Dagger 的學習難度比較高，大部分用在大型協作專案比較適合，恩~老實說我覺得還行阿，以前有玩過，原本預估個八小時應該可以搞定，最後花了15小時，是比較麻煩啦，主要是 dagger 的錯誤題示訊息有點難懂。

這種東西就是初始化麻煩，但是後來用起來整個舒爽，最明顯的就是 MVP 版本時有一個 Injection.java ，這東西可以根據放的資料夾以及 build variant 去提供 debug version , prod version 不一樣的實作注入，但是 Dagger 就不用了阿，等等，好像只是移到 Module 了，不要小看這個 Module , Component ，一堆奇妙的 new 就通通不見了(解偶的性質就出現了)，也不用管理物件的生命週期(專注業務邏輯)，在管理上也因為有好好的分類，開發效率也會提高(清晰的 code)，撰寫 UT 也會方便得多。實際程式碼如下

[Funny Vote MVP with Dagger2 version Github](https://github.com/boochlin06/FunnyVote/tree/mvp_dagger)

實作開發心得:

* 使用任何新框架開發都請先理解為何要導入，以 Dagger 來說，主要是解決依賴嚴重的問題，接者就會引導到方便測試的好處。但是導入框架都會有它應該付出的代價，個人覺得最煩的是初始化流程，接者為了讓 Module , Component整個流程能達到注入的效果，大概會多出兩倍的接口設計，真的一般小型開發的 solo 玩家們，可以好好思考需要嗎?

* 由於多出大量  Module , Component 的物件和接口設計，這就是一個麻煩的問題，寫的程式碼越多就越有機會出 BUG，那麼問題來了，就要思考有沒有辦法簡化，因為大部分的 Module, Component  雖然一堆接口要設計，但大都是模板化的代碼，果然 google 大神提出了[dagger.android](https://google.github.io/dagger/android) 的工具，用 annotation 去省略模板代碼，讓 compiler 去自動生成程式碼。

* 模組解偶後，務必寫測試，舒爽

節錄查資料時看到的名言，如果寫完你有這樣的感覺，代表..GOOGLE 很厲害啊

> 

Dependency injection frameworks can simplify the code you write and provide an adaptive environment that’s useful for testing and other configuration changes.

If you intend to use a dependency injection framework in your app, consider using [Dagger 2](http://google.github.io/dagger/). Dagger does not use reflection to scan your app’s code. Dagger’s static, compile-time implementation means that it can be used in Android apps without needless runtime cost or memory usage.

以上是本次小弟的感想，我真的懶得貼 CODE ，各位真的想看去 Github 吧，通通都放在那，至於 kotlin 版本我則放到下一篇，對小弟來說 kotlin 算是真的完全沒用過，所以測重點會多放在怎麼用，根本篇性質不太相同。所以新開文章
