# Funny Vote 開發雜記

> 原文網址：https://boochlin.com/?p=289
> 發布日期：2021-05-16
> 文章編號：289

---

![g294](../../assets/images/g294.png)

[https://boochlin.com/wp-content/uploads/2017/04/g294.png](https://boochlin.com/wp-content/uploads/2017/04/g294.png)

基於 blog 的主旨『Something Record』，這次來寫一下自己的 Android side project 『趣投票』開發紀錄，老實說提到技術的部份不多，更多的是紀錄專案怎麼推進的。感覺犯了不少蠢事，就當作故事寫下來給大家笑笑。也希望大家可以澎場，玩一下這支 APP

下載地址 [https://play.google.com/store/apps/details?id=com.heaton.funnyvote](https://play.google.com/store/apps/details?id=com.heaton.funnyvote)

很久很久以前，大概是 2016 年5月左右吧（看了 github 紀錄，應該是吧，好像也沒多久），當時公司的專案開發已經漸漸轉往其他語言，印象是 React Native（所以寫了[React Native 學習筆記](https://boochlin.com/?p=136)），後期更轉向成自動駕駛，專職寫 c 語言 （也寫了[SLAM Note – NDT_TKU](https://boochlin.com/?p=271)）。但是總覺得自己不能放掉 Android 技術，必須保持一定的 Android 技術敏感度，也順便玩玩一些新的 framework，所以開了一個練手的 Android Side project。

至於會決定這個題目『投票』，主要是發現朋友之前有用 PHP 寫一個線上網頁投票平台 [Easy Vote](http://easyvote.chunchung.com/)，當時雖然覺得這個點子很常見，但也不伐有人使用(而且沒啥對手，在繁中界)，因此就決定來寫個 Android 版投票軟體。

雖然是這麼說，但是畢竟沒有單槍匹馬導過一個完整的專案，而且基於懶人沒想那麼多的風格，根本沒去想詳細的 SPEC 規劃，就直接隨便參考 [Easy Vote](http://easyvote.chunchung.com/) 的功能亂刻 UI，想當然更沒有去想後端串接那塊，想說反正又不是不會寫後端，總覺得先動手再說，不然嘴巴說在多也沒屁用。

底下是當時最早寫的創建投票的頁面

![螢幕快照 2017-04-27 上午2.41.50](../../assets/images/螢幕快照-2017-04-27-上午2.41.50.png)

[https://boochlin.com/wp-content/uploads/2017/04/螢幕快照-2017-04-27-上午2.41.50.png](https://boochlin.com/wp-content/uploads/2017/04/螢幕快照-2017-04-27-上午2.41.50.png)

不是我在說，當我刻完這個畫面，自己都覺得『馬的，那麼鳥的畫面最好是會有人用』，『果然我的美感根本不行』，『這拿去給面試官都覺得拍謝』，因此痛定思痛，最後決定先廢個一陣子好了（哈哈哈，當時工作也變多了）。

廢了一陣子之後，勉強重起幹勁，腦中充滿 UI/UX 才是是重點的想法，但是以往 UI/UX 都有專人負責，我只要負責把功能生出來就好了，更何況功能不都是 PM 在想的嗎？

『對喔，我現在是一個人單幹，應該先從 PM 開始當才對，先想 SPEC』

有了這想法之後卻發現，我在公司跟 PM 交集不多，沒啥人可問，而且我常常覺得 PM 不太靠鐠（抱歉，這是錯誤的想法），因此那時候有機會就問主管或是其他同僚 SPEC 制定的概念。

2016/7月，在看看網路上的資料，東問西問之後，發現 SPEC 應該要從現有的投票 APP 參考才行，先把自己覺得不錯的頁面設計記下來，在此階段參考了**『甭糾結』『爆炸投』『PollPic』『匿名投票』『OurVote』**等等。無聊也去逛逛 [Dribbble](https://dribbble.com/) 提昇自己的設計概念（雖然覺得自己弄不出來），最後終於弄出了[第一份 SPEC](https://docs.google.com/document/d/1QEij-CPvxShB9tBn9p0l4iuBYI87ZpNauc0RG7fvRVs/edit?usp=sharing)

![FIRST_VOTE_DESIGN](../../assets/images/FIRST_VOTE_DESIGN.png)

[https://boochlin.com/wp-content/uploads/2017/04/FIRST_VOTE_DESIGN.png](https://boochlin.com/wp-content/uploads/2017/04/FIRST_VOTE_DESIGN.png)

大概是長的上面這樣，雖然還是很鳥，但是 SPEC 開始確定了(也開始佩服 PM 是多麼厲害的崗位)。同時也在思考，後端應該切出去給別人做，自己並非擅長這一塊，應該交給專家才對，所以就產生把 Easy Vote 的開發者 JJJ 也拉進來一起玩的想法。（另外，現在來看當時的 SPEC 真心覺得有趣，因為大部分功能都沒有去作，像是一開始很想要的可以多重訂單的功能。樣貌也改了不少，我想這應該是想法發散跟收斂的過程吧（哈哈哈））

雖然想拉 JJJ 進來玩，但是總覺得沒有先作個 『Prototype』出來，他大概不會搭理我吧。另一方面目前雖然 SPEC 定完，但是 UI/UX 並沒有提昇多少，一點都不好看，這樣作為 Prototype實在沒啥說服力，所以又開始參考網路大神的設計，也買了書（Android 介面設計…）來看。最後 UI 風格定案為 Google 強推的 Material design。

[https://material.io/guidelines/material-design/introduction.html](https://material.io/guidelines/material-design/introduction.html)

老實說 Material Design 官網真的寫的很好，搭配常見的 MD Component（RecycleView , DrawerLayout , TabLayout …），真的可以簡單的弄出還能看的 UI，從建議場景到 Pixel 怎麼定義都有很詳細的說明，而且連 icon 跟色系都提供。

經過約兩個多月的懶人開發之後，終於做出第一個 Prototype，也定案 App 名稱為**『Funny vote 趣投』，如下**

![LINE_MOVIE_1493214410282](../../assets/images/LINE_MOVIE_1493214410282.gif)

[https://boochlin.com/wp-content/uploads/2017/04/LINE_MOVIE_1493214410282.gif](https://boochlin.com/wp-content/uploads/2017/04/LINE_MOVIE_1493214410282.gif)

2016/10月，正式開始拉攏後端開發 JJJ，並順手抓一個很閒的人 NICK 進來當文案。為了能讓他們清楚理解這個專案到底在做啥，正式開了一個線上會議，底下是當時的投影片（PM事情好多阿）

當開始團體合作時，PM 的職能就變多了，之前只需要訂定 SPEC，現在必須要分配任務，管理進度，不過幸好我是 PM 兼 主力開發 RD ，所以很多溝通可以避免掉了。但是該有的還是必須有，因此開始使用 Trello 作為進度管理工具，Line 作為主要溝通工具。

![trello](../../assets/images/trello.png)

[https://boochlin.com/wp-content/uploads/2017/04/trello.png](https://boochlin.com/wp-content/uploads/2017/04/trello.png)

實際開始運作起來後，發現自身溝通能力實在不足，且由於這並非有獲利的業外專案，也很難使全部人有幹勁，所以進度上一直是很慢很慢。但雖然這樣說，每周釘還是有成果出來，在這階段主要是 搞定Android 端和後端之間的 API 協定，這是在考驗雙方對於 API 設計的概念，非常費工，詳細就不討論，有機會的話另外寫一篇。

———–兩千多字了，好累，分段休息一下———–

可能會有人覺得怎麼沒有提到開發上的問題，我只能表示『給我回去看之前的文章』。實際上遇到的 Android 技術問題都不太困難，主要還是要下功夫去做（突然想到之前看過得『[因為自動飲料機而延畢的那一年](http://opass.logdown.com/posts/1300438-the-story-of-auto-beverage-machine-23)』系列文章，感到佩服），相關技術文章有

[How to use RecyclerView inside NestedScrollView?](https://boochlin.com/?p=169)

[GreenDao 3 Introduction](https://boochlin.com/?p=226)

[Android ButterKnife introduction](https://boochlin.com/?p=238)

[Retrofit 2 — How to Upload Files and Parameter list to Server](https://boochlin.com/?p=247)

[Android issue – Avoid non-default constructors in fragments](https://boochlin.com/?p=254)

[Android AppIntro Proguard issue.](https://boochlin.com/?p=262)

[Android issue – UI佈局被鍵盤擋住](https://boochlin.com/?p=263)

當然不只這些，這些是剛好有空就紀錄下來的文章，也歡迎各位去看我的 GitHub

[https://github.com/boochlin06/FunnyVote](https://github.com/boochlin06/FunnyVote)

另外可以一提的開發問題是，開發會員第三方登入（FB, Twitter , Google）時，雖然以前有玩過，但是實在過太久，很多 API 都改過，必須重看，此時正感到時間不夠用也很麻煩，居然剛好前同事『E神N』 Hangout 發來訊息表示最近好閒，有沒有啥專案好練手。

『馬的，剛好這就交給他了』，因此第四位夥伴加入啦….（撒花，感謝E神N）

技術以外，真正麻煩的是 UI 部份，沒錯，我也不會畫圖，也找不到人來畫 UI ，公司裡 UI Team 的人也不熟，最後想來想去就只有外包出去。

這聽起來很像很順利，但實際上開始在網路上發包後，發現按照我的頁數應該會讓我荷包大傷，而且出於對專業的尊重，我也不太想殺價，畢竟大家都是出來混口飯吃，最後折衷只發包 LOGO 設計，經過來來回回好幾次討論後，最後就是

![g294](../../assets/images/g294.png)

[https://boochlin.com/wp-content/uploads/2017/04/g294.png](https://boochlin.com/wp-content/uploads/2017/04/g294.png)

至於剩下其他需要圖的部份，就自己硬者頭皮開始學 『**[Inkscape](https://inkscape.org/en/)』（會選這個原因很簡單，就是免費）。至於其他自己畫不出來的，就上網找免費圖檔（感謝 freepik。歡迎美術專業的人加入）。此外非常感謝[林阿衛](https://www.youtube.com/channel/UChntdOQjE1yO8psm4atHbTw)的 youtube 教學頻道，真的學習非常多。

2016/12月，開發到這階段，要花錢的事情也慢慢跑出來了，JJJ 表示想要開 Server 需要錢，也不想繼續用 IP 作為測試，要用 Domain name，不過這方面沒啥異議，簡單來說就是用最便宜的 Server 『DIGITAL OCEAN』，唯一堅持的大概就是一定要 『.com』 結尾，當然這部份基於錢的考量，最後就決定是 699 元的

#### 『[funny-vote.com](http://funny-vote.com)』

 

2017/2月，在邊砍 SPEC 邊開發的情況下，也不知道是不是有花錢，還是新成員的關係，算是進度最快的一段日子，API 大部分也串接好了，SPEC 居然完成將近 80 啪。雖然是這樣說，但 Android 端測試測試者，總覺得完成度好像沒有很高阿，因此又開始一段參考啥某是『完成度高的 APP』的故事，順便等等後端的開發進度。

『文字沒有梗』『沒有 Introduction Page』『沒有 Tutorial Page』『FLOW 怪怪的』在這些想法浮現之後，又開始改 SPEC ，繼續開發並修改一些有趣的小功能，最後就是
[https://boochlin.com/wp-content/uploads/2017/02/Screenshot_20170308-012917.png](https://boochlin.com/wp-content/uploads/2017/02/Screenshot_20170308-012917.png)
[https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170411-000643.png](https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170411-000643.png)

![Screenshot_20170411-000636](../../assets/images/Screenshot_20170411-000636.png)

[https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170411-000636.png](https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170411-000636.png)

![Screenshot_20170411-000643](../../assets/images/Screenshot_20170411-000643.png)

![](../../assets/images/Screenshot_20170308-014043.png)

[https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170308-014043.png](https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170308-014043.png)

![Screenshot_20170427-013144](../../assets/images/Screenshot_20170427-013144.png)

[https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170427-013144.png](https://boochlin.com/wp-content/uploads/2017/04/Screenshot_20170427-013144.png)

2017/3月，在開發上述東西的同時，JJJ 也完成了初版官方網站。在3月底大部分 SPEC 都完成的情況下，終於發布阿法版（當初覺得2016/12 就可以完成了，哈哈哈），並於內部測試。

[官方網站](https://funny-vote.com/)（有分大小網）

發佈在 PLAY STORE 的介紹文

> 

『人生自古誰無惱』

—賈明延

我是說在場的各位，每天都會面臨各式各樣需要作抉擇的情況，然而，最常遇到下面這種情況

『欸，等下中午要吃啥？』

你突然發現你沒有朋友。『無人可問』

『剛剛是不是有地震？』

你只是想跟風。『就是想問問』

『下禮拜我要約她出去吃飯，挑了兩間餐廳，猶豫中』

發在群組裡問人卻得到

『別放閃』

『你到底想對你的右手作啥』

『你聽過安利嗎』

你覺的原來我的朋友只會嘴砲。『答非所問』

『下禮拜要約哪裡吃飯？』

『統計一下，這次誰人氣最高？』

『是否支持XXX法案？』

你真的需要眾人之力，可是作問卷好麻煩。『想簡單的問更多人』

或者你是另外一類情況，

『平常在知識家之類的地方貢獻智慧，但是最近對於長篇大論決得很麻煩』

『看別人為一些雞毛蒜皮的事情在煩惱，覺得很爽』

『我最喜歡推人入坑了，選我選我』

全台最 x 投票軟體上線啦，以上『趣投票』通通一次解決

快速發起投票：會想發起投票就是因為心中有苦，有猶豫，或是只是好奇，這種情況下，想當然是趕快問問眾人的意見，趣投票讓發起投票這件事變得超級方便。

快速投票：替人解惑乃是大善事大慈悲，但是如果搞得很麻煩，還要註冊，驗證碼，那就提不起勁，趣投票完全不需要繁複的過程，只需要您的手指按個三秒就可以作功德。

沒有廢話：求助時最怕有疑惑時，想尋求眾人智慧，卻一堆人指指點，趣投票考慮到這一點，所以我們『沒有留言板』。

大數據：關於這個，感覺 APP 沒有提到這個就弱掉了，但是我們真的完全沒有，絕對不是技術上的問題。

共享經濟：聽說共享很流行，其實投票不也是一種智慧共享嗎？只需要將投票連結分享出去，全世界都會來幫你。至於經濟的部分就只能看廣告收入了。

重點是順手打個五星評價

開始測試之後，先不說啥登入會 Force Close，重點是 Nick 發現『趣投』實在不太好念，當初是想取其台語發音『chi to』，表達玩樂的概念，但中文真的很嗷口，所以也在此時從新定案 APP 名稱為『趣投票』

2017/4月，其實寫到這裡也接近尾聲了，BUG 修完就跟者出貝塔版，這階段其實也沒啥事可做，SPEC 也不應該有改動，只能繼續修修圖，改改小功能，測試測試，想文案。（其實到這階段也有點累了，並沒有即將完成的成就感，只有想要趕快早早發布，收工的心情。）

2017/4月底，正當貝塔版全面發布之後，原本以為在商店應該就搜索的到，卻遲遲沒有看到，想說應該只是系統處理慢了一些，便直接睡個覺，卻在小瞇兩小時之後，渾渾噩噩的情況下收到了一封『來自 Google Play 關於 Funny Vote 的通知』的信，一打開被指出『違反了**[假冒他人](https://play.google.com/intl/zh-TW_ALL/about/ip-deception-spam/impersonation-ip/impersonation/)**政策』，整個嚇醒。這時候好像也只能快速確認到底哪裡假冒他人，摸了兩小時候，真心找不到問題在哪裡，就只好提出上訴，希望 Google 能指出更明確的地方，或是直接恢復。

經過來來回回兩次上訴之後，發現原來是 APP 裡，太常提到『www.funny-vote.com』，卻沒有證明這個網站是我的，因此被懷疑可能假冒此網站，被要求提出證明。而這邊我的對應方法是提出購買網域的證明，以及利用 search console 的網站證明機制，保證我是此網站的雍有者。老實說我不知道是哪個方法有效，但是最後終於還我一個清白啦，只能說以後要多多注意，不知道有沒有人跟我遇到一樣的情況，歡迎分享。

2017/5，經過四月底的恐怖事件之後，終於正式進入最後測試階段，其實越接近尾聲，心中就對這支 App 操作意見越多，但用膝蓋想也知道，這階段絕對不可能大改。有關 SPEC 之類的通通必須熄火，想動的通通交給第二版，不然永遠都不用上線了。

實際上這階段最重要的是，怎麼樣才會讓人來玩這隻 APP，也就是『宣傳』，但是這一塊可能比美術還要麻煩，因為幾乎沒有概念。而我自己唯一想到的方法就是寫一篇『Funny vote 開發故事』，讓大家來看看一個普通人怎麼做出一個小 App 的故事。我並沒有用特別的架構( clean architecture , MVP , TDD  之類的，甚至連 UT 都沒寫，也沒 CI, CD)或技術(圖學，串流，演算法之類的)，很多情況都只是串起來而已。如果有人也想要開始寫寫小 APP，或許可以參考參考，並順手順手玩我的 App。(雖然是這樣說，最後宣傳還是要外包給專業團隊…)

 

———-收尾分割線————-

Funny vote 1.0 版開發的故事到這邊算是結束了，接下來我也不知道會不會做第二版，也不知道會不會一正式發布就直接當機，營運後實在太多太多變數，不過現在階段我終於可以說 『我做完啦，你要不要一起來玩啊』。

### 寫在最後，謝謝你們

『後端主力』JJJ

 如果連不到 Server ，百分之百是他的問題。

 『Android Support』Eason

 他本人堅持要說 “宿霧超爽”，但他就是個台灣人

 『QA and 文字』Nick

 大概是最認真做 QA 的男人了。

最後拜託大家下載一下來玩啊 [https://play.google.com/store/apps/details?id=com.heaton.funnyvote](https://play.google.com/store/apps/details?id=com.heaton.funnyvote)

 

Framework Reference

網路框架

Retrofit2 , Glide

資料庫

GreenDao

網路監控

Firebase , GA

跨元件溝通

eventbus

小工具

Butterknife

stetho

UI 元件

這就太多了

感謝觀看，收工
