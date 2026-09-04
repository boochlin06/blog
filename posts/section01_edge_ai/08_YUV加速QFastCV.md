# Android 爛裝置想跑人臉辨識8 – YUV_420_888 還能怎樣加速：QFastCV

> 原文網址：https://boochlin.com/?p=952
> 發布日期：2026-07-11
> 文章編號：952

---

前幾篇我們在跟 LMKD 搏鬥、跟 ART GC 算帳、在 Linux 內核裡斤斤計較每一個 Page。

今天換個全新戰場：在影像資料送進 AI 模型之前，將相機硬體吐出來的 `YUV_420_888` 轉換為模型所需的格式。這是整個影像流水線中最容易被菜鳥忽視、卻能拼出十倍加速空間的兵家必爭之地。

在我們這台 4 核心 1.3GHz 弱雞 CPU 的破板子上，光是做一次 YUV 格式轉換與去 Padding，純 Java 寫法就硬生生吃掉了將近 15ms。

30fps 的影像串流，留給每一幀的總時間預算只有 33.3ms。一個小小的格式轉換吃掉 15ms，相當於直接揮霍掉半幀的算力預算。

直到我深入翻開相機硬體為什麼要吐出這種怪異格式，才發現這不是 Android 的鍋，而是幾十年前矽晶片設計師留下的歷史業障。

---

## 人眼對顏色是瞎子：YUV420 的生理學甜蜜點

`YUV_420_888` 這個格式的誕生，根源在於人類眼睛演化上的先天缺陷。

人眼中負責感知亮度的視桿細胞高達 1.2 億個，而負責感知顏色的視錐細胞只有可憐的 600 萬個。這意味著：人類大腦對「明暗邊界」極度敏感，但對「色彩細節」本質上是個半瞎子。

工程師抓住這個生理漏洞發明了 YUV：

* **Y（Luma 亮度）**：每個像素都完整保留，一個都不能少。
* **U、V（Chroma 色度）**：人眼看不出細節，那就讓 2x2 的四個像素共用一組 U 和 V。

小學算術對比：

> RGB 格式：4 個像素 = 4 x 3 = 12 bytes
> YUV420 格式：4 個像素 = 4 (Y) + 1 (U) + 1 (V) = 6 bytes
> 頻寬節省：(12 - 6) / 12 = 50%

記憶體頻寬與資料傳輸量直接腰斬，而肉眼幾乎看不出任何畫質差異。這就是為什麼全世界所有相機感測器與視訊晶片，預設全部使用 YUV420。

---

## UV 為什麼是交錯存的？（Semi-Planar NV21 的硬體歷史）

當你從 Android Camera2 取得 YUV 資料時，UV 記憶體佈局長成這樣：

```text
Y 平面:  [Y0 Y1 Y2 Y3 Y4 Y5 ...]
UV 平面: [U0 V0 U1 V1 U2 V2 ...]  <-- U 和 V 交錯擠在一起
```

U 和 V 交錯存放在同一個 Buffer 中，這叫 **Semi-Planar（NV21 / NV12）**。

為什麼不乾脆把 U 和 V 完全分開存放成獨立的兩個平面（Planar 格式，如 I420）？**因為相機 ISP 硬體工程師要省晶片成本。**

相機內部的 ISP（影像訊號處理器）透過 DMA 引擎將像素搬移到記憶體，其運作方式像掃描機一樣從第 0 行逐行掃到最後一行。當 ISP 掃完每兩行 Y 時，對應的一組 U 和 V 剛好計算完成：

* **選 Semi-Planar**：U 和 V 算出來就直接交錯寫入記憶體，電路設計最簡單、晶片面積最小、成本最低。
* **選 Planar**：必須在晶片內增加額外的 SRAM 快取，把 U 全部暫存起來，等整張圖的 Y 寫完後再分批寫入 U 與 V。這需要更複雜的硬體排程器，晶片成本直線上升。

幾十年前工程師在成本壓力下的妥協，就這樣變成了我們今天必須面對的標準。

---

## 為什麼每行末尾有多餘的垃圾？（Row Stride 的 64-byte 對齊）

在我們的門禁機中，當 RGB 相機切換到 800x600 解析度時，你會發現一個詭異現象：每行有效像素明明只有 800 bytes，但底層 Buffer 的每行跨度（Row Stride）卻是 **832 bytes**。

多出來的 32 bytes 全是無意義的 Padding 垃圾：

```text
行 0: [800 bytes 有效 Y 數據] [32 bytes 垃圾 Padding]
行 1: [800 bytes 有效 Y 數據] [32 bytes 垃圾 Padding]
... 共 600 行
```

這是因為高通 SoC 的 CPU 與記憶體匯流排（System Bus）以 64 bytes 為一個傳輸區塊（Burst Transfer），且要求每次 DMA 傳輸的起始位址必須是 64 的整數倍。

800 不是 64 的倍數（800 / 64 = 12.5）。為了讓 DMA 傳輸效率達到硬體最高峰值，ISP 會自動在每行末尾補齊 32 bytes 垃圾，湊成 832（832 / 64 = 13，完美整除）。

硬體爽了，代價是軟體工程師必須在資料餵進 AI 模型前，手動把每行末尾的 32 bytes 垃圾挑出來扔掉。

---

## 原本怎麼做的：純 Java 苦工（24 萬次邊界檢查）

在導入硬體加速前，我們在 `CameraRgbHelper.java` 中的備用路徑寫了標準的 Java 轉換迴圈：

```java
// CameraRgbHelper.java
private void yuv420888toNV21(Image image, byte[] nv21) {
    // ... 處理 Y 平面 Padding ...

    // 處理 UV 交錯提取
    for (int row = 0; row < height / 2; row++) {
        for (int col = 0; col < width / 2; col++) {
            int vuPos = col * pixelStride + row * rowStride;
            nv21[pos++] = vBuffer.get(vuPos); // 撿一個 V
            nv21[pos++] = uBuffer.get(vuPos); // 撿一個 U
        }
    }
}
```

在 800x600 解析度下，UV 平面的尺寸是 400x300：

* 雙重迴圈總共要執行 400 x 300 = 120,000 次。
* 總計執行 **24 萬次陣列存取與 `ByteBuffer.get()`**。

Java 虛擬機在每次呼叫 `get()` 時，底層都會強制做一次記憶體邊界檢查（Bounds Checking）。在 1.3GHz 的弱 CPU 上，這 24 萬次安全確認直接吃掉了 **5 ~ 15ms**！在 30fps（每幀僅 33.3ms）的流水線中，光格式轉換就吃掉將近一半的算力預算。

---

## 換成 QFastCV：一行搞定（< 1ms）

為了解決這個瓶頸，我們在相機流水線中掛載高通原生 **QFastCV** 庫：

```java
// CameraRgbHelper.java
if (true == useFastCv) {
    // 調用高通 Native 函式庫：直接傳入 DirectByteBuffer，實現真正物理零拷貝！
    result = QFastCV.yuv420SPToYuv420PZeroCopy(
        image.getPlanes()[0].getBuffer(), // DirectByteBuffer
        image.getPlanes()[1].getBuffer(),
        CAM_RGB_WIDTH, CAM_RGB_HEIGHT,
        image.getPlanes()[0].getRowStride(), // 自動處理 Row Stride Padding
        image.getPlanes()[1].getRowStride(),
        dstDirectBufY, dstDirectBufU, dstDirectBufV
    );
}
```

### 終極榨汁：為什麼不要用 byte[] 中轉？

很多工程師寫 JNI，直覺會在 Java 端先用 `byteBuffer.get(bufferY)` 把數據拷貝進 Java 的 `byte[]` 鐵碗，再把陣列傳進 C++。

但在每秒 30 幀高頻下，這在 Java Heap 與 Native 間多做了一次整包 `memcpy`（每秒無謂消耗 20MB/s 記憶體頻寬）。

Camera2 的 `Image.Plane.getBuffer()` 本身就是 **`DirectByteBuffer`**，其底層是相機 ISP 硬體 DMA 直接寫入的 ION 堆外記憶體。最頂級的寫法是**連 Java byte 陣列都不借**，直接在 JNI C++ 層直取硬體物理指標：

```cpp
// QFastCV_JNI.cpp：真正的物理零拷貝
JNIEXPORT jboolean JNICALL
Java_com_edge_ai_vision_QFastCV_yuv420SPToYuv420PZeroCopy(
    JNIEnv *env, jclass clazz,
    jobject directBufY, jobject directBufUV,
    jint width, jint height,
    jint srcStrideY, jint srcStrideUV,
    jobject dstBufY, jobject dstBufU, jobject dstBufV) {

    // 直取相機硬體 DMA 映射的虛擬位址，0 拷貝開銷！
    uint8_t* pSrcY  = (uint8_t*) env->GetDirectBufferAddress(directBufY);
    uint8_t* pSrcUV = (uint8_t*) env->GetDirectBufferAddress(directBufUV);
    uint8_t* pDstY  = (uint8_t*) env->GetDirectBufferAddress(dstBufY);
    uint8_t* pDstU  = (uint8_t*) env->GetDirectBufferAddress(dstBufU);
    uint8_t* pDstV  = (uint8_t*) env->GetDirectBufferAddress(dstBufV);

    if (!pSrcY || !pSrcUV || !pDstY || !pDstU || !pDstV) return JNI_FALSE;

    // 直接調用高通 FastCV 原生 NEON 向量指令拆分 UV 與去 Padding
    fcvColorYUV420u8ToYUV420Planaru8(
        pSrcY, pSrcUV, width, height,
        srcStrideY, srcStrideUV,
        pDstY, pDstU, pDstV, 0, 0, 0
    );
    return JNI_TRUE;
}
```

轉換耗時直接從原本 Java 迴圈的 15ms 暴跌到 **< 0.8ms**！

---

## QFastCV 憑什麼這麼快？（ARM NEON 向量指令解密）

為什麼 C++ 呼叫 QFastCV 能產生超過 15 倍的效能差距？核心秘密在於 ARM NEON SIMD（單指令多資料流）的專屬硬體指令：

1. **零 JVM 邊界檢查**：
C++ 直接操作記憶體原生指標，那 24 萬次 Java 虛擬機的安全邊界檢查直接歸零。

2. **ARM NEON `VLD2` 專屬向量指令**：
ARM 架構專門為拆解交錯資料設計了 **`VLD2`（Vector Load 2-element structure）** 指令。
傳統 CPU 讀取 `U0 V0 U1 V1 ...` 需要進行 16 次讀取與位移操作；而 NEON 指令 `vld2.8 {d0, d1}, [r0]!` 可以在 **1 個時脈週期內**，直接將記憶體中連續的 16 bytes 拆解成獨立的 8 個 U 暫存器（d0）與 8 個 V 暫存器（d1）。Java 要跑 16 次迴圈的事，NEON 靠硬體電路一個指令搞定。

3. **記憶體連續寫入（Cache Line Friendly）**：
拆解後的 U 與 V 透過 `VST1` 向量指令連續寫入獨立的 Planar 緩衝區，完美契合 CPU L1/L2 快取行，最大化記憶體寫入吞吐量。

---

## 三路相機資料流，誰真正受益？

這套 QFastCV 加速只對 RGB 相機生效，因為只有它牽涉到複雜的色彩空間轉換：

* **RGB 鏡頭（800x600 / 640x480 YUV_420_888）**：
最大受益者。徹底解決 Semi-Planar 拆分與 Row Stride 去 Padding 的計算瓶頸，單幀耗時節省 14ms。
* **Sony IMX516 ToF 深度鏡頭（640x1920 RAW12）**：
無關。ToF 輸出的是純 3D 深度原始 Raw 訊號，沒有顏色與 UV 分量，直接以 `byteBuffer.get(buffer)` 整包搬入記憶體。
* **IR 紅外鏡頭（640x480 灰階）**：
無關。IR 影像是由 ToF Raw 訊號經 SDK 解算出的單通道 8-bit 灰階圖，每個像素僅佔 1 byte，完全沒有 UV 交錯問題。

---

## 結語與防坑心法

`YUV_420_888`、`Semi-Planar`、`Row Stride`——這三個概念追根究底，都是幾十年前晶片設計師在有限的矽晶圓面積與記憶體頻寬壓力下，做出的最務實硬體妥協。

你不理解底層硬體原理，就看不懂為什麼一個看似簡單的「格式轉換」能吃掉 15ms。搞懂整條因果鏈之後，才能精準下刀——用高通 QFastCV 與 ARM NEON 向量指令，在 1 個時脈週期內做掉 Java 要跑 16 次迴圈的事。

在低配邊緣設備上，每一個你以為「應該很快」的細節，都可能藏著拖垮整個系統的暗箭。

下一篇，我們聊聊人臉辨識最核心的數學戰場：**5000 人 1:N 特徵比對，如何用 ARM Compute Library (ACL) 把比對時間從 320ms 暴降到 3ms！**

---

## 參考資料 (References)

1. **ARM NEON Technology: Vector Load and Store Instructions (VLD2/VST2)**.
https://developer.arm.com/architectures/instruction-sets/simd-isas/neon

2. **Qualcomm FastCV Computer Vision SDK Developer Guide**.
https://developer.qualcomm.com/software/fastcv-sdk

3. **Android Camera2 YUV_420_888 Image Format Specification**.
https://developer.android.com/reference/android/graphics/ImageFormat#YUV_420_888
