# Android CTS Fail – cant remove external folder

> 原文網址：https://boochlin.com/?p=46
> 發布日期：2015-06-18
> 文章編號：46

---

Compatibility Test Suite

這東西有17000條測試，但是我不是專門負責cts 測試的仁兄

詳細就不多談了，反正就是出事了，要來解

突然

某一天

居然被傳換了，表示external folder 一直被佔用，導致cts測試失敗

當然突然被傳喚的人，第一句話通常就是，『這不關我的事情』

但是案子還是要繼續破，犯人還是要找

但 cts team 同仁，也無法定案，到底是誰呢？

此時，ap team 一名大師走了過來

『adb shell lsof |grep Android』

trace the which data current access the file.

大師說lsof 可是debug 之神武，居然不知道

所以趕快寫起來

好了，案子破了，犯人真的launcher的disk cache

但問題真的白痴，disk cache 當然一定一直開者

而且通常情況下，cache放在外部資料夾根本是常理

然後cts要檔這個，嗚呼哀哉

最後只好隨便找個地方放了，只要不是在 external folder

尤其是preload app , 這鬼東西也是5.1才有的測項

認真要研究lsof 的話 還是看別人資料比較好喔

[http://jashliao.pixnet.net/blog/post/163589216-%E6%AF%8F%E5%A4%A9%E4%B8%80%E5%80%8Blinux%E6%8C%87%E4%BB%A4–lsof%E6%8C%87%E4%BB%A4(%E5%88%97%E5%87%BA%E7%95%B6%E5%89%8D%E7%B3%BB%E7%B5%B1%E6%89%93%E9%96%8B](http://jashliao.pixnet.net/blog/post/163589216-%E6%AF%8F%E5%A4%A9%E4%B8%80%E5%80%8Blinux%E6%8C%87%E4%BB%A4--lsof%E6%8C%87%E4%BB%A4(%E5%88%97%E5%87%BA%E7%95%B6%E5%89%8D%E7%B3%BB%E7%B5%B1%E6%89%93%E9%96%8B)
