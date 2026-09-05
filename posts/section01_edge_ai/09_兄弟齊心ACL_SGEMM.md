# Android 爛裝置想跑人臉辨識9 – 兄弟齊心，軟硬兼施：ARM Compute Library (ACL)

> 原文網址：https://boochlin.com/?p=943
> 發布日期：2026-07-24
> 文章編號：943

---

前面幾篇都在跟記憶體搏鬥、跟 GC 算帳，今天來談純粹的「硬體算力」。

在 1.3GHz 弱雞 CPU 的破板子上跑即時人臉辨識：相機抓到人臉、抽出了一串 512 維的浮點數特徵向量，接下來要在本地資料庫裡的 5,000 名員工特徵庫中大海撈針，找出這個人是誰。

在老闆最初提出「辨識至少要 10 FPS」的無理要求下，算盤一敲，小學算術直接把所有人嚇傻：

> 512 (維度) x 5000 (員工) x 10 (FPS) = 25,600,000 次 / 秒

每秒鐘必須吞下 **2,560 萬次** 浮點數乘加運算！

為了活下來，我第一時間先說服老闆把目標砍到 3 FPS——畢竟我們要的是人臉辨識成功開門，不是在門禁機上看 60fps 絲滑動畫。

但就算把幀率砍到 3 FPS，單次比對 5,000 人依然要算：

> 512 x 5000 = 2,560,000 次浮點數運算（256 萬次）

在我們這台只有 4 核心 Cortex-A53 弱 CPU、沒有 NPU 的破板子上，特徵比對模組歷經了三次極其慘烈的演化。

---

## 第一次演化：天真的 Java 苦工（150ms）

最直覺的做法，就是在 Java 裡寫個雙重迴圈算內積（餘弦相似度）：

```java
float bestScore = -1.0f;
int bestIndex = -1;

for (int i = 0; i < 5000; i++) {
    float[] dbFeature = database.get(i);
    float score = 0;
    for (int j = 0; j < 512; j++) {
        score += inputFeature[j] * dbFeature[j];
    }
    if (score > bestScore) {
        bestScore = score;
        bestIndex = i;
    }
}
```

* **實測耗時**：**約 150 毫秒**。
* **慘況分析**：低階 ARM 晶片上的 Android JVM 根本沒有強大的自動向量化優化能力。CPU 只能像個老實巴交的苦力，老老實實跑完這 256 萬次乘法。單次比對吃掉 150ms，聽起來好像還能擠出 6 FPS？別天真了，這只是整個 Pipeline 的最後一步，前面的人臉偵測、品質過濾、3D 活體防偽全都要吃時間啊！

---

## 第二次演化：把 1:1 邏輯套進 1:N 迴圈的 JNI 災難（320ms）

很多工程師直覺認為：「Java 慢？那就交給影像神器 OpenCV 的 C++ 底層來算！」

在我們的專案裡，本來就有一個專門給「1:1 刷卡認證」使用的 `AuthFeatureMatcher.java`。當員工先刷工號卡、帶入身分後，呼叫 OpenCV 的 `Core.gemm` 比對單一目標：

```java
// AuthFeatureMatcher.java（原本專為 1:1 刷卡比對設計）
featureDet.put(0, 0, inputFeature);
featureCmp.put(0, 0, authFace.getRgbFeature());

// 呼叫 OpenCV 底層算單次內積分數
// 第二矩陣需轉置為 512x1 才能與 1x512 相乘，結果為 1x1 餘弦相似度純量
Core.gemm(featureDet, featureCmp, 1.0, new Mat(), 0.0, cvResult, Core.GEMM_2_T);
```

在 1:1 模式下，這段代碼只執行一次，耗時不到 0.5ms，爽快得很。

但如果把它拿來做 1:N 無感刷臉（不刷卡、從 5,000 人大海撈針），在 Java 外層套上 `for (int i = 0; i < 5000; i++)` 迴圈——**下場就是一場災難**。

* **實測耗時**：**約 320 毫秒（居然比純 Java 慢了兩倍以上！）**。
* **致命的 JNI 過路費**：
這犯了跨語言呼叫的大忌：在 Java 迴圈裡頻繁呼叫 JNI。
為了算一筆小小的乘法，程式過海關（JNI Context Switch）來回跑了 5,000 趟！
這就像為了榨 5,000 杯果汁，開卡車一次只載「一顆蘋果」跑 5,000 趟去工廠，路上的過路費與換向開銷早就遠遠超過榨果汁的時間了。

---

## 最終進化：降維打擊（ACL SDK + ARM NEON，3ms）

被 JNI 痛擊之後，我們徹底推翻架構，為 1:N 打造了專屬的 **`NesFeatureMatcher.java`**：

1. **不能過海關 5,000 次**：整個 1:N 比對迴圈必須整包搬到 C++ 底層。
2. **OpenCV 太通用**：我們需要針對這顆 ARM Cortex-A53 量身打造的專屬武器。
3. **從「單挑」升級為「打群架」**：
我們不再一對一單挑了。程式啟動時，直接將 5,000 名員工的特徵在 C++ 記憶體中拼接成一個 **5000 x 512 的二維超級矩陣（Tensor）**。

新畫面進來時，**全流程只過「一次」海關**：

```java
// NesFeatureMatcher.java
// 跨一次 JNI，底層直接發動 1 x 5000 大矩陣乘法
int faceIndex = aclLib.cachedNesMatch(inputFeatures, threshold);
```

在 C++ 底層，改用 ARM 原廠的 **Compute Library (ACL)** 發動 `NESGEMM`：

```cpp
// C++ 底層：ACL SGEMM 運算
NESFeatureMatcher::copy_to_tensor(*_tensor_a, inputFeature);

// 呼叫 ARM 底層硬體加速 (NEON SIMD) 一擊必殺
sgemm.run();

NESFeatureMatcher::copy_from_tensor(*_tensor_o, results.get());
```

* **實測耗時**：**約 3 毫秒（整整百倍秒殺！）**。

---

## 為何如此狂暴？（蓋一萬份公文的頂級辦公室 SOP）

如果要用最直白的方式總結 SGEMM 與硬體加速，我們可以想像 **「一個辦公室員工（CPU）要蓋 1 萬份公文（資料）」** 的現場：

### 1. 傳統寫法（一般迴圈）
員工走到遙遠的檔案室（主記憶體 RAM），找出一份公文，走回位子，拿起印章蓋下去。然後再走去檔案室拿下一份……
**結果**：來回跑死，一天只能蓋 100 份，大部分時間都在走路上浪費掉了。

### 2. 快取預讀（Prefetch）—— 請個小助手
為了解決走路的時間，員工請了一個小助手。當員工在位子上蓋章時，小助手拼命去檔案室把接下來要蓋的公文一疊一疊搬到員工桌上（L1 快取）。
**結果**：員工再也不用起身，伸手就能拿到下一份公文。

### 3. SIMD 向量化 —— 特製連體印章
就算不用走路，一次蓋一份還是太慢。於是公司配發了一個「特製的四連體印章」（ARM NEON 128-bit 向量暫存器），一次壓下去同時蓋 4 份公文（FMA 融合乘加）。
**結果**：蓋章速度直接翻了 4 倍。

### 4. 矩陣分塊（Tiling）—— 桌面空間管理
雖然小助手一直搬公文，但員工的「桌面空間有限」（Cortex-A53 L1 數據快取只有 32KB）。如果一次搬 1 萬份過來，會塞爆桌面，公文掉到地上反而更亂（Cache Thrashing）。
所以員工規定：「每次只搬剛好鋪滿桌面的數量」（4x4 或 8x8 Tile）。這疊蓋完收走，再換下一疊。
**結果**：桌面永遠保持最高效率的運作狀態，快取命中率飆破 99%。

### 5. 重新打包（Packing）—— 事前整理
檔案室裡的公文是照「年份」排的，但老闆要求照「部門」蓋章。如果你一邊蓋章一邊自己翻找，會浪費超多時間（跨步讀取懲罰）。
所以小助手在把公文放到桌上之前，會先在旁邊排好順序（線性連續記憶體佈局）。
**結果**：員工閉著眼睛一路狂蓋就好，完全不用停下來找。

---

## 結語與防坑心法

SGEMM 就是這整套 **「頂級辦公室 SOP」** 的終極結合體。

你自己寫的 C++，就像是叫一個很猛的員工去蓋章，但他不懂辦公室架構與 SOP，只會傻傻來回跑。

而 ARM 原廠工程師寫的 SGEMM，是連桌子大小、小助手走路速度、印章尺寸都精算過的完美生產線。把 1:N 比對壓到 3ms，省下來的算力才能讓 1.3GHz 弱晶片在 300ms 內從容完成開門。

下一篇，我們把鏡頭切到雙目 3D 視覺最兇險的戰場：**RGB 與 Sony ToF 深度相機的時空同步！**

---

## 參考資料 (References)

1. **ARM Compute Library (ACL) Documentation**.
https://arm-software.github.io/ComputeLibrary/latest/

2. **OpenCV Core Operations and Matrix Multiplication (GEMM)**.
https://docs.opencv.org/4.x/d2/de8/group__core__array.html

3. **Cortex-A53 Software Optimization Guide**.
https://developer.arm.com/documentation/den0042/a/
