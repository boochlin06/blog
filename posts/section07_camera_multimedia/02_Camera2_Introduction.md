# Android Camera analyze (二) – Camera 2 Introduction.

> 原文網址：https://boochlin.com/?p=27
> 發布日期：2015-03-29
> 文章編號：27

---

#### **Camera ****2 Introduction**

自從2014 google i/o android 5.0介紹新的camera api 2 相信有不少人躍躍欲試這新功能

畢竟demo上看起來非常的完整，不少新的功能都有

最主要有

1. 原生支援 raw 照片輸出

2. 即時拍攝模式

3. 提高連拍速度，過去的架構常常壓制在軟體的設計，這次算是大修正，連拍可以達到非常高。

4. 增加可以控制的要件。

不過老實說到目前2015年初，有支援到那某多的要素控制根本沒幾隻手機阿

#### page2:

Camera API History

從 Camera 1 定案之後，基本上都沒有改過

API 14: 臉部偵測

API 15: Video stabilization (這很難解釋阿)

API 16: 移動中自動對焦

API 17: 關閉 Shutter sound。

API 20: Camera 2 啟用

看得出來大部分都是小改動

而這次的重點就是 大改中的大改 Camera2 api.

#### page3:

很明顯的google 想要捨棄原本的camera api

畢竟 “不然我做這投影片幹嘛啦”，原本的camera1 就像是傻瓜相機一樣

幾乎全部設定都寫好，沒啥可以改動

而新的 camera 2 就像是單眼相機，幾乎所有的選項設定都可以自己控制

#### page4:

究竟camera1 到底有啥令人不快的地方，不少開發者應該直接想到

1. 幾乎只有幾個黑箱功能能用，可以讓開發者玩得實在太少

– preview , still capture , video record.

2. burst mode 根本做不出來阿，如果沒有vendor去配合，第三方太困難了

只靠camera1 api 有些實在無理…ex:

– zero shutter lag , multi shot HDR , Panoramic stitch.

3. 能提供的資訊實在太少，這會使得後製不好玩

4. 硬體反應太慢的話，很多effect 的控制會很難保證

#### page5:

Camera1 的架構(請看前一頁的圖示)只能讓連拍8mp照片時，維持在1-3fps

光看這樣就知道根本知道 HDR 不可能啦

#### page6:

Camera 2 三句以蔽之

More manual control , pipeline model. More feedback

就是變得超快，可以玩的選項也變多，以及更多照片訊息

#### page7:

讓連拍8mp照片時，維持在30fps————

快能搞定一切

#### page8:

讓連拍8mp照片時，維持在30fps，加上各種變化—圖為曝光變化

這很明顯HDR有搞頭了

#### page9:

Camera2 的基礎架構，可以看得出來跟camera1 的用法有變

主要有:

1. Camera的service改為從 system 取得

2. 根據system提供的CameraCharacteristics 去開啟 CameraDevice

3. 不在鎖定幾種camera 使用方法(capture,startPreview)，而是可以分離出

不同的需求(CaptureRequest)作變化

4. 對於camera的使用生命周期也是鎖定在 CameraCaptureSession

5. 在輸出方面也不向camera1指定在某個單一surface，而是多個

這樣可以讓後製變得更加趣味。

#### page10:

這是上述pipeline的簡易圖示

1. 可以在captureRequest指定output surface.並排序進入 request queue.

2. 處理完成的request在captureSession中完成後

可以直接對應到個別surface，push result and view.

#### page11:

這完全可以看出 camera2 可以玩的項目變得超多

1. 各種manual control , auto-mode control。

2. 完成後的處理class也變多，印象中camera1 只有兩種。

3. output format最大改變就是 RAW_SENSOR的支援。

#### page12:

呈上

4. CameraCharacteristics 的資訊變超多。

5. Camera1 的太少result資訊，也至少變成三倍。

#### page13:

其實就是重複前幾頁的資料

記住:

1 Request -> 1 image captured -> 1 result metadata + N image buffers.

以上是camera2 可以做的事情，引起開發的可能性

下一章節為 教大家 利用新的camera2去開發新的camera
