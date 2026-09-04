# SLAM Note – NDT_TKU

> 原文網址：https://boochlin.com/?p=271
> 發布日期：2024-04-08
> 文章編號：271

---

** [NDT-TKU](//www.slideshare.net/boochlin/ndttku) ** from **[Booch Lin](//www.slideshare.net/boochlin)**

### NDT – TKU Introduction

睽違許久沒有寫 blog ，主要還是太忙。這周終於空出時間可以來補補之前的筆記，由於最近才對內部分享此主題。所以這個主題印象最深刻。
雖然很想再次就進入正題，但是由於這 blog 的風格就是廢話多，所以我繼續寫廢話，談到的這個主題，用到大量的數學，而小弟我在大學基本上『機率』『線性代數』『微積分』大概都有重修過，所以如果客倌對於數學的部份有問題，請自行上網查，因為我答的不會比google 大神好。

**這次主題的最終目標 SLAM （** **Simultaneous localization and mapping） 同步定位與地圖構建，是一個關於機器人或是自動駕駛必定研究的主題，而弄懂前也必須先具有大量的相關知識，前置部份我沒有辦法細細講解，主要也是所學太少，因此這次就只專注於 SLAM 的其中一個方法 NDT , 而 TKU 則是再 NDT 進一步的推展。**

在此之前，有興趣的人可以先看這篇論文

****[Magnusson, M. (2009). The Three-Dimensional Normal-Distributions Transform — an Efﬁcient Representation for Registration, Surface Analysis, and Loop Detection](http://aass.oru.se/Research/mro/publications/2009/Magnusson_2009-Doctoral_Thesis-3D_NDT.pdf)

這大概是目前對於 NDT 最入門的文件，弄懂他不會吃虧，甚至不用看我的文章啦。

SLAM 的所有最終目的就是要找出自身到底位於世界中的何處，這可能大家會直覺認為交給 GPS 就好啦，但是實際上他的不確定因素實在太多，各位自身用手機導航都會常常罵說定不準，那麼要求更高安全性的自動駕駛更不可能完全依託在這上面，當然目前兩派爭論也是沒有停過，不過爭論的點主要就是在於只要發展 GPS 技術到非常高精度，就可以不用研究其他相關定位技術。但是用膝蓋想也知道就還不夠精密，所以定位演算法至今能然是一個主要研究主題。

定位演算法一個重點就是 Sensing，機器人必需要能感知周圍環境，得到相關情報，最常見的就是相機，這確實是個主力，而且相較其他傳感器確實便宜，不過這次討論的目標並非相機，而是 LIDAR (光達)，通常掃描到的畫面長這樣

影片是 VELODYNE LIDAR 的示意圖，順代一題這東西超貴，數百萬有之。目前不少研究都是針對這種點雲資料作 SLAM 研究。

其實這類資料研究主要就圍繞在『利用這一次掃瞄到的點雲跟下一次掃瞄到點雲，判斷出機器人的位子與姿態』。說起來簡單是個匹配問題，但是做起來超級麻煩，光是分類就可以搞死自己了。(寫到這邊有點累了)。

看到這裡，有人可能直覺這會多難，把這次 Scan 每個點拿來跟下一個 Scan 比對不就好了，那我就會想問怎麼比，茫茫人海中，總要有個依據，啥-你說挑近的比阿，這個我聽過，專業點說法『ICP （ Iterative Closest Point ）』，這我就提到這裡，上面那篇文章有介紹。來張圖告訴你這水可深了

[http://www.cnblogs.com/gaoxiang12/p/3695962.html](http://www.cnblogs.com/gaoxiang12/p/3695962.html) 之呼上的大神

[http://ivory-cavern.blogspot.tw/2009/11/icp-iterative-closest-point-c.html](http://ivory-cavern.blogspot.tw/2009/11/icp-iterative-closest-point-c.html) ICP 實做

![ICP](../../assets/images/ICP.png)

[https://boochlin.com/wp-content/uploads/2017/04/ICP.png](https://boochlin.com/wp-content/uploads/2017/04/ICP.png)

好啦，ICP 扯遠了，只是要告訴大家並非只有 NDT 而已，回到正題 NDT – TKU，這部份我就照者投影片來談，也請各位同時看投影片會比較好懂

#### PAGE 2:

#### What is ndt_tku

NDT 在 SLAM 上的基本玩法算是行之多年，在 ROS 系統上也已經有 PCL 整成相當方便的 CLASS，[How to use Normal Distributions](http://Transformhttp://www.pointclouds.org/documentation/tutorials/normal_distributions_transform.php)這篇文章也已經詳細教學過，而 NDT-TKU，則是根據 竹內先生(TAKEUCHI) 教授提出的論文 [A 3-D Scan Matching using Improved 3-D Normal Distributions Transform for Mobile Robotic Mapping](http://ieeexplore.ieee.org/document/4058864/)(論文網路不公開) NDT 再進化版本，相關實做則是在他門的 Open source project『[Autoware](https://github.com/CPFL/Autoware)』中的 [ROS node](https://github.com/CPFL/Autoware/tree/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes)

#### PAGE3:

#### Why is ndt_tku

由於我是研究他門的整個 Autoware 專案，裡面其實已經有實做 PCL NDT 的 Node ，在非常好奇的情況下混進他門的 slack ，提問為何要自己作一套 NDT Tku 版本，主要原因就是

1. 是指導教授提出的

2. PCL 的 cuda 化極度麻煩，工程師表示不如自幹一套，然後 cuda 化

1 很好理解，2 則是為了利用顯示卡的 GPU加速化。對於自動駕駛來說，能快一點算出，就可以讓後續的判斷更容易執行，提高安全性

#### PAGE4:

#### Outline

理解這東西是要照順序來，先知道 Normal-Distributions ，才能知道 Normal-Distributions Transform 到底在玩啥。最後才能導到 TKU 做了啥改進。

#### PAGE 5:

#### Normal Distribution

正態分佈 – 是一個在[數學](https://zh.wikipedia.org/wiki/%E6%95%B8%E5%AD%B8)、[物理](https://zh.wikipedia.org/wiki/%E7%89%A9%E7%90%86)及[工程](https://zh.wikipedia.org/wiki/%E5%B7%A5%E7%A8%8B)等[領域](https://zh.wikipedia.org/wiki/%E9%A0%98%E5%9F%9F)都非常重要的機率分佈，由於這個[分布函數](https://zh.wikipedia.org/wiki/%E5%88%86%E4%BD%88%E5%87%BD%E6%95%B8)具有很多非常漂亮的性質.使得其在諸多涉及統計科學離散科學等領域的許多方面都有著重大的影響力

[https://en.wikipedia.org/wiki/Normal_distribution](https://en.wikipedia.org/wiki/Normal_distribution)

![nd](../../assets/images/nd.png)

[https://boochlin.com/wp-content/uploads/2017/04/nd.png](https://boochlin.com/wp-content/uploads/2017/04/nd.png)

連續的東西，數學只要能表示出來就能算，先理解這樣的分佈對於點雲的運算是有意義的

#### PAGE 6:

#### **Normal Distribution Transform**

這裡談談怎麼作一般版的 NDT，將所有的點雲先切成一格一格 ND Voxel

The normal-distributions transform can be  described as a method for compactly representing a surface.

正態分佈給出了點雲的分段平滑表示，具有連續的導數。 每個[PDF（機率密度函數）](https://zh.wikipedia.org/wiki/%E6%A9%9F%E7%8E%87%E5%AF%86%E5%BA%A6%E5%87%BD%E6%95%B8)可以看作是局部表面的近似值，描述了表面的位置以及其取向和平滑度。

A 2D laser scan from a mine tunnel (shown as points) and the PDFs describing the surface shape. Each cell is a square with 2 m side length in this case. Brighter areas represent a higher probability. PDFs have been computed only for cells with more than five points.

![CELL](../../assets/images/CELL.png)

[https://boochlin.com/wp-content/uploads/2017/04/CELL.png](https://boochlin.com/wp-content/uploads/2017/04/CELL.png)

#### PAGE7 :

#### NDT in tunnel – 3D

示意圖，自己看投影片啦

#### PAGE 8:

#### 如何表示點雲 Cell 的機率分布

D-dimensional normal random process, the likelihood of having measured ~x is  where ~yk=1,…, m are the positions of the reference scan points contained in the cell.

正態分佈給出了點雲的分段平滑表示，具有連續的導數。 每個PDF可以看作是局部表面的近似值，描述了表面的位置以及其取向和平滑度。

協方差矩陣的特徵向量和特徵值可以表達表面信息

Each PDF can be seen as an approximation of the local surface, describing the position of the surface as well as its orientation and smoothness.

#### PAGE 9, 10:

#### Scan registration

為了找出兩次 SCAN 的 spatial transformation function T(~p, ~x) that moves a point ~x in space by the pose ~p. 可以利用剛剛得到的 CELL PDF ，去找出最接近（max）的分佈函數

NDT score 可以用於牛頓法，表示是否達到最好的分數。Gaussian approximation 則可以減少運算

(看最上面推薦的論文或是投影片，沒辦法多加註解)

#### PAGE 11:

#### Newton’s algorithm for

* Newton’s algorithm can be used to find the parameters ~p that optimise s(~p)

* Newton’s method iteratively solves the equation H∆~p = −~g

* g and H are partial differential and second order partial differential of

老樣子超過15個數學符號我們就很不想理會

[https://www.youtube.com/watch?v=Quw4ZHLH2CY](https://www.youtube.com/watch?v=Quw4ZHLH2CY)

利用微分找出切線，疊代法趨近找出 function = 0 的根

#### PAGE 12:

#### NDT FLOW

![FLOW](../../assets/images/FLOW.png)

[https://boochlin.com/wp-content/uploads/2017/04/FLOW.png](https://boochlin.com/wp-content/uploads/2017/04/FLOW.png)

#### PAGE13:

#### **我知道****大家****看數字****很****痛苦**

其實當初分享的對象都並非數學相關出身，也並非作 SLAM 為主，所以我猜講得在略過，簡單，還是不好理解，因此不少東西都沒有在深入，但是為了能導到 TKU 的修改，還有實做上的參數設定，不得不提。

#### PAGE14:

#### About ND Voxel size

* 太小

  * 運算量大，memory 消耗大

  * 匹配精確

  * 但小於五個點，則很難形成正態分佈

* 太大

  * 運算量少

  * 匹配不精確

看到這裡，其實可以發現 NDT SLAM 主要的作法和重點都在於 處理 PDF 的最佳化問題，從結論來看，能下手的就是 Cell 的參數調整。這通常也很難決定，必須參照環境（是否附近有很多建築物，還是大平原），Sensor的參數（掃描密度，時間）等等，但總而言之都是上面的所提到的情況，太大或太小都不好。

#### PAGE 15 :

#### NDT – TKU version

使用格子重疊的方法，但這並非 TKU 所提出。

Peter Biber and Wolfgang Straßer: “The Normal Distributions Transform:A New Approach to Laser Scan Matching”, Proceedings of the 2003 IEEE/RSJ International Conference on Intelligent Robots and Systems, pp. 2743–2748, 2003

TKU 採用每個 Cell 都有一半的重疊。好處壞處都很明顯。這方發在分類上為『Trilinear interpolation』，利用重疊 Cell 的不連續性，提高 Cell 交界處的點雲分佈穩固姓

#### PAGE 16:

#### 圖片有感

自己看投影片啦

#### PAGE 17:

#### TKU – ND Voxel size

在CELL重疊的基礎上，追加距離和時間的參數，先分成兩個階段，

* Converging state

  * 按照距離切分 ND Voxel size，並運算

* Adjust state

  * 到一定次數後則通通用最小格子來運算

Converging 階段，先根據與 scanner 的距離來決定 Cell 的大小，這是屬於物理限制，離 Scanner 越近，點雲一定越密集，因此可以將 Cell 切小一點，越遠越大，這樣可以加速收斂運算。

Adjust 階段，當 Converging 收斂次數達到一定後，則可以開始通通使用最小 Cell 來進行收斂運算，這樣一來也可以保持 match 精度。

這是 TKU 所提出的主要概念，而詳細實驗數據，則請各為自己去看論文（著作權）。

#### Page 18

#### Reference:

1.A 3-D Scan Matching using Improved 3-D Normal Distributions Transform for Mobile Robotic Mapping(網路上不公開)

2.The Three-Dimensional Normal-Distributions Transform — an Efficient Representation for Registration, Surface Analysis, and Loop Detection

3.The Normal Distributions Transform:A New Approach to Laser Scan Matching

#### PAGE 19：

#### Other

這是一些實做上可以調整的參數，有空的人可以自行去看 PCL 實做

#### PAGE 20:

#### Score detail

這是補充 PAGE 10 的 Score 的運算。

Ending

如果各位看到這邊的話，也算是大功告成，但這篇真的並非入門文章，有問題的話可是發問看看，或是自行去看論文。

接下的部份，真的就是屬於 Autoware 的 trace code 紀錄。

——————分隔一下———————

### Parameter 可調控參數

#### **Iteration**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L321](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L321)

最大迭代次數，預設100

#### **G_MAP_CELLSIZE**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L11](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L11)

CELL 單位大小

等價於PCL setResolution 預設1.0

#### **leaf size**

Voxel grid size 調整參數

setLeafSize

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L233](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L233)

#### **scan_points_num**

掃描後的最大點雲數量 default 13000，這個要小心設定，點數太多會報掉

#### Epsilon

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L334](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L334)

收斂值得最小變動值，越小越精確，越大算越久，通常不改

#### **add_point_map(NDmap, &map_points[i]);**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L501](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L501)

這一行通常註解掉，主要用於邊 match 邊更新地圖，算是 MATCHING AND MAPPING 的結合，但是還沒有完成。

#### **_downsampler_num**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp#L418](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp#L418)

這是預設 voxel filter 的設定，0=distance，1= voxel_grid，通常是 1 ，正常來說這應該是外部可參數化的部分。但它們還沒完成。

#### **g_use_gnss**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L76](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L76)

預設沒有啟動 GPS , 照理說也應該室外部可調控參數，但是他門還在作。

#### **layer_select**

應該是為了之後的TKU切割格子而設計的，還不清楚怎麼用，問他們也沒回答

 

### **CODE 講解**

主要三大檔案

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp)

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/algebra.cpp](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/algebra.cpp)

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp)

#### **add_point_map**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L629](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L629)

一次份的點雲匯入

![FLOW](../../assets/images/FLOW.png)

[https://boochlin.com/wp-content/uploads/2017/04/FLOW.png](https://boochlin.com/wp-content/uploads/2017/04/FLOW.png)

#### **adjust3d**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp#L195](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp#L195)

包含一次份的 收斂計算，上面的一次while 運算

![hg](../../assets/images/hg.png)

[https://boochlin.com/wp-content/uploads/2017/04/hg.png](https://boochlin.com/wp-content/uploads/2017/04/hg.png)

gsum , hsum 分別表示底下的一次微分，和二次微分的 hessian matrix

![h](../../assets/images/h.png)

[https://boochlin.com/wp-content/uploads/2017/04/h.png](https://boochlin.com/wp-content/uploads/2017/04/h.png)

每次的運算都會呼叫 get_ND 去更新值

#### **get_ND**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L685](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L685)

這裡會進行 newton 法的更新值問題，上面的 for all points 一次運算

#### **update_covariance **

更新每個 ND 的MEAN AND CONVARIANCE

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L595](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L595)

論文裡有提到，求最小 eigen value of covariance 可能會遇到收斂質雖然很小，但是可以繼續收斂的情況，論文建議低於0.001時，則不再繼續 0.001，建議不要亂動，測試後並沒有特別效果，因為這是極限情況

A measure of this problem in [1] is to modify the minimum eigen value of the covariance matrix with 0.001 times as much as the maximum eigen value, if the minimum one is smaller than 0.001 times as much as the maximum one. The authors take same approaches. However, the number of overlapping ND voxel becomes eight in 3-D space.

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L882](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L882)

論文提到的減少運算量的方法，不進行全盤的 mean 值計算，而是有新點加入時判斷是否有需要嗎?，直接用粗略的方式加入新的值。

1.  Incremental update of ND voxel The number of reference scan points Mk in the ND voxel k is increased as the environment map is expanded. Therefore, the computational load to obtain mean vectors pk(k = 1, …, Mk) and covariance matrix Σk(k = 1, …, Mk) will increase according to equations (1), and (2). To decrease this load, the authors take the following incremental update equations to apply NDT to reference scan in ND voxel. When ND voxel k gets new reference points xki, then mean vector pk and covariance matrix Σk are updated by following equations;

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L595](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L595)

#### **set_sincos**

#### **set_sincos2**

#### **scan_transrate**

這三個都是用於表示 做 transform 的 矩陣旋轉式

![transform](../../assets/images/transform.png)

[https://boochlin.com/wp-content/uploads/2017/04/transform.png](https://boochlin.com/wp-content/uploads/2017/04/transform.png)

#### **calc_summand3d**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp#L38](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/newton.cpp#L38)

![score](../../assets/images/score.png)

[https://boochlin.com/wp-content/uploads/2017/04/score.png](https://boochlin.com/wp-content/uploads/2017/04/score.png)

計算 ndt 的機率分布總分數，會用到 probability_on_ND

#### **probability_on_ND**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L905](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/packages/ndt_localizer/nodes/ndt_matching_tku/ndt_matching_tku.cpp#L905)

投影片有提到過，用於計算分數，點雲落於每個nd裡的機率

#### **jacobi_matrix3d**

[https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/algebra.cpp#L354](https://github.com/CPFL/Autoware/blob/master/ros/src/computing/perception/localization/lib/ndt_tku/src/algebra.cpp#L354)

亞可比矩陣，用於牛頓法，求 eigen value.

 

### **code flow**

![ndt_flow](../../assets/images/ndt_flow.png)

[https://boochlin.com/wp-content/uploads/2017/04/ndt_flow.png](https://boochlin.com/wp-content/uploads/2017/04/ndt_flow.png)

 

如有任何侵權，請告知小弟，小弟會拿下，也請各位注意相關著作權。
