# Android 好文整理

> 原文網址：https://boochlin.com/?p=79
> 發布日期：2015-09-24
> 文章編號：79

---

之前一週一章節把累積的存貨給消耗不少掉

現在啥鬼都還拉不出來，先把目前正在消化的網站整理出來

1. [https://www.gitbook.com/book/bng86/android-third-party-/details](https://www.gitbook.com/book/bng86/android-third-party-/details)

Facebook 上的 android 開發社團，所分享的third-party 使用紀錄

一個步驟一個步驟都有慢慢寫，超級簡單好上手，目前也在這邊看一些 orm 資料

2. [GreenDAO](http://greendao-orm.com/)

android orm 的好工具，至於為何是greendao這就可以看官方自吹自擂一下啦

目前正在將Greendao導入到目前的專案中

未來會完整把使用心得給發出來

3. [Google I/O 2015 – 100 Days of Google Dev](https://www.youtube.com/playlist?list=PLOU2XLYxmsIJDPXCTt5TLDu67271PruEk)

身為一個android 工程師，永遠跑不掉的就是google 開發資料

這次的2015，100篇都是乾貨，雖然之前用過得就不少，但是在聽大牛講一次也是獲益良多

在身為工程師之前，更是個台灣人，所以提供一個[中文版](http://www.ithome.com.tw/news/98805)

4. [JAVA複習 skywang](http://wangkuiwu.github.io/2100/01/01/index/)

這是由中國人寫的一系列JAVA,DATA STRUCTURE 讀書心得，通常有受過正規學院訓練應該不會太陌生

但是能寫的那某詳細真的不容易（外觀也簡單到不容易），當年上課要是那某認真就厲害了

此外還有不少ubuntu and android 資料可以惡補自己的學識，感謝skywang ，引用他『關於我』的部份

『他是个不错的家伙』—-大概吧

5.[awesome-android-performance](https://github.com/Juude/awesome-android-performance)

performance 永遠是 android 開發工程中最頭痛的部份，畢竟你總是會遇到一堆爛到不行的測試機

想當然，這種事不可能只有你遇得到，網路上不少好傢伙門把他整理起來，有空可以試試看

目前正在看 custom viewgroup 的部份

這作者還有其他大分類 [awesome android](https://github.com/JStumpp/awesome-android)

6.[造車輪這件事情](https://android-libraries.zeef.com/jurgen.stumpp)

上面那項老大把很多 awesome library 整理在一起，弄的漂漂亮亮的

就這樣，有空多看看別人的車輪總不是一件壞事，最多就是你沒時間玩遊戲

7.[leakcanary](https://github.com/square/leakcanary)

A memory leak detection library for Android and Java.

“A small leak will sink a great ship.” – Benjamin Franklin

利用 weak reference 相關機制確認 garbage collection 是否有處理掉，藉此處理可能memory leak 的情況

個人覺得開發完成後，可以拿來在daily used 更加確認是否有memory leak , 但是目標還是要自己先鎖定，有點還是要自己找嫌疑犯的感覺

提供[中文參考資料](http://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)

8.[Android’s multidex slows down app startup](https://medium.com/groupon-eng/android-s-multidex-slows-down-app-startup-d9f10b46770f#.uauzgq4bj)

這篇文章真的厲害，是在碼天狗上看到的，驚為天人，檢視一下手上專案，居然method已經到達可怕的51k數量，不過還好，強制綁定5.0以上，使用art，對於效能沒有影響。[中文版](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1223/3796.html)

9. [android book](https://github.com/yongjhih/android-gitbook)

 

2016/01/05

note: 消化中，未來有更新的話，會另開一帖並整理成系列文
