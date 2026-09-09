# Android 爛裝置想跑人臉辨識14 – 門禁機當 BLE Peripheral 的現場踩坑記

做邊緣工控，最煩的永遠是擦現場的爛攤子。

別跟我扯什麼「陽光下不好按螢幕」的貼心鬼話。真實情況是：硬體廠為省一塊面板的錢，丟過來的是一台**完全沒螢幕、沒鍵盤、零實體按鍵的純金屬盲盒機**。

機器被死死釘在兩公尺高立柱上，一旦現場改了 WiFi 密碼或要設固定 IP，連操作介面都沒有，總不可能叫工人扛鋁梯拆防水外殼接串口。把門禁機做成 **BLE Peripheral（GATT Server）** 讓手機連線配網，純粹是**這台盲盒在現場唯一的命根子**。

這篇掀開高通 QM215 上的底牌，聊聊怎麼用一套充滿偷懶、黑歷史但能秒交差的土砲代碼搞定這破事。

---

## 先講結論：老架構師的四個偷懶取捨

搞邊緣設備，核心就八個字：**能跑就行，早點下班**。

1. **門禁機當被動方（Peripheral）**：開機直接啟動，丟給藍牙 Controller 晶片自己發廣播，Host CPU 負載直接歸零，絕不搶相機人臉辨識算力。
2. **31-Byte 雙層廣播**：廣播封包只有 31 Bytes，塞不下 128-bit UUID。短機號放第一層，長 UUID 踢去第二層 ScanResponse，手機要了再給。
3. **拒絕 MTU 協商，一刀切 20 Byte**：配網資料不到 100 Bytes，搞非同步 `requestMtu()` 純屬吃飽太閒。鎖死 20 Byte 預設基線，外包寫個 for 迴圈切一切發過來，半天收工。
4. **7x24 常駐，壓根沒寫關閉代碼**：設備插市電不用省電，代碼裡連關閉計時器都懶得寫。免得哪天改密碼藍牙關了，還得扛梯子去拉總閘重開機。

手機與設備的交互時序，整套流程就這麼點破事：

```mermaid
sequenceDiagram
    autonumber
    participant M as 手機端 App (Central)
    participant D as 門禁機 (Peripheral / GATT Server)
    participant S as 雲端管理後台

    Note over D: 開機常駐發射主廣播 (31B 內)
    D-->>M: 主廣播 ADV_IND (設備名稱-短序號, TxPower)
    M->>D: 主動掃描 SCAN_REQ
    D-->>M: 掃描回應 SCAN_RSP (128-bit 專屬 Service UUID)
    
    Note over M,D: 建立連線 (不搞 MTU 協商，直接切 20-Byte)
    M->>D: CONNECT_REQ 建立連線並 Discover Services

    rect rgb(240, 245, 255)
    Note over M,D: 階段一：讀設備 ID (土砲 10 片特徵值)
    M->>D: Read CHAR_READ_DEVICE_ID_CHUNK_NUM
    D-->>M: 回傳總片數 (例: 0x04)
    M->>D: 依序 Read CHUNK_0 ~ CHUNK_3
    D-->>M: 回傳各片 20-Byte 序號數據
    end

    rect rgb(245, 255, 245)
    Note over M,D: 階段二：寫入網路配置 (Wi-Fi 或 乙太網靜態 IP)
    alt Wi-Fi 配網情境
        M->>D: Write SSID 分包 [Index, TotalLen, Payload]
        D-->>M: GATT ACK
        M->>D: Write 密碼分包 [Index, TotalLen, Payload]
        D-->>M: GATT ACK
        Note over D: 密碼與 SSID 湊齊，立刻觸發 connectToWifi()！
        M->>D: Write 後台網址分包 [Index, TotalLen, Payload]
        D-->>M: GATT ACK (置位 flag，等 Wi-Fi 成功後寫入)
    else 乙太網路固定 IP 情境
        M->>D: 依序寫入 7-Byte 定長幀 (0x01=IP ~ 0x04=DNS1)
        D-->>M: GATT ACK
        M->>D: 寫入 0x05 (DNS2)
        D-->>M: GATT ACK
        Note over D: 收到 0x05 觸發 setStaticIp() 重啟網卡
        M->>D: Write 後台網址分包
        D-->>M: GATT ACK
    end
    end

    rect rgb(255, 250, 240)
    Note over M,D: 階段三：主動輪詢與狀態閉環
    loop 手機端每秒發一次 Read 特徵值
        M->>D: Read CHAR_READ_NOTIFY_SERVER_CONNECTED
        D-->>M: 回傳 6-Byte [Status: 0x02(連線中), IPVer: 0x04, IP: 0.0.0.0]
    end
    D->>S: 網路連通，WebSocket 握手成功 (Status 變 0x01)
    M->>D: Read CHAR_READ_NOTIFY_SERVER_CONNECTED
    D-->>M: 回傳 6-Byte [Status: 0x01(已連線), IPVer: 0x04, IP: 192.168.1.100]
    Note over M: 手機 UI 亮綠燈，配網成功
    M->>D: 主動發起 DISCONNECT 斷開藍牙
    end
    Note over D: 門禁機回到被動廣播，繼續常駐待命
```

---

## 一、 為什麼當被動方？別讓 A53 猝死

安卓教學常教你用手機當 Central 去掃周邊。但你要在門禁機上開掃描，現場幾百個人帶著耳機走過去，A53 四顆弱雞核心光處理藍牙中斷就升天了，相機預覽直接卡死。

乖乖當個被動的 **Peripheral**。開機把廣播註冊進藍牙晶片後完全放生，只有手機寫入資料時才觸發回調，省心省力。

### 開機就常駐，完全不設防

門禁機是專用 Kiosk 系統，`MainActivity.onCreate()` 裡直接無條件啟動：

```java
// File: MainActivity.java
@Override
protected void onCreate(Bundle savedInstanceState) {
    BleService.createInstance(this); // 開機無條件啟動，永遠不關
}
```

在 `BleService` 建構子裡粗暴開幹：沒開藍牙就強制 `enable()`，呼叫 `openGattServer()` 註冊服務：

```java
// File: BleService.java (行 147-150, 231-233)
BluetoothAdapter bluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
if (bluetoothAdapter != null && !bluetoothAdapter.isEnabled()) {
    bluetoothAdapter.enable(); // 沒開就霸道強開
}
if (bluetoothManager != null)
    mBluetoothGattServer = bluetoothManager.openGattServer(context, mBluetoothGattServerCallback);
mBluetoothGattServer.addService(service);
```

### 31-Byte 廣播太小？丟到第二層

標準廣播淨載只有 31 Bytes。若硬塞 16 Bytes 的 128-bit UUID，設備名稱只剩幾顆字母，直接拋錯。

解法很單純：主廣播只放裁剪後的短機號；長 UUID 扔到第二層 ScanResponse，手機掃描要了再給：

```java
// File: BleService.java
uniqueSerial = DeviceCodeUtils.getSerialNum().substring(4);
bluetoothAdapter.setName(deviceName + "-" + uniqueSerial);

// 第一層：僅名稱與功率（壓在 31B 內）
AdvertiseData advertiseData = new AdvertiseData.Builder()
        .setIncludeDeviceName(true).setIncludeTxPowerLevel(true).build();

// 第二層：128-bit UUID 丟在 ScanResponse
AdvertiseData scanResponse = new AdvertiseData.Builder()
        .addServiceUuid(new ParcelUuid(UUID_SERVICE)).build();
```

---

## 二、 20-Byte 協議：別裝逼，就是為了好交差

BLE 預設單包 ATT Payload 就是 **20 Bytes**。

不搞動態 MTU 的原因就三個：
1. **資料量少到不值一提**：帳密、IP、網址加起來不到 100 Bytes，幾十毫秒搞定的東西，寫什麼非同步握手？
2. **Server 無權發起**：規範卡死 `requestMtu()` 是手機端的專屬權限，門禁機只能乾等。
3. **外包不想改**：手機 App 是外包趕工出來的。定死 20 Byte，手機端無腦寫個 for 迴圈分包，半天對接上線。

![BLE 三大二進制通訊封包佈局圖](../../assets/images/ble_binary_packet_layouts_1788423952069.jpg)

### 2.1 讀 Device ID：不懂 Read Blob 留下的奇葩遺產

標準 BLE 讀長資料官方機制叫 **Read Blob Request（利用回調裡的 `offset` 分段讀取）**，一個特徵值搞定。

但當初寫這段的人顯然不知道 `offset` 是幹嘛的，居然在迴圈裡動態註冊 10 個 UUID，用字串末碼減法算位移：

```java
// File: BleService.java (行 80-89)
private final int DEVICE_ID_CHUNK_SIZE = 20;

if (characteristic.getUuid().toString().startsWith(STR_CHAR_READ_DEVICE_ID_CHUNK_PREFIX)) {
    char[] uuid = characteristic.getUuid().toString().toCharArray();
    int shift = DEVICE_ID_CHUNK_SIZE * (uuid[uuid.length - 1] - '0'); // 土砲手動算分片
    mBluetoothGattServer.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, 0,
        Arrays.copyOfRange(DeviceCodeUtils.getCode().getBytes(), shift, shift + DEVICE_ID_CHUNK_SIZE));
    return;
}
```

手機要讀設備 ID，得先讀 A 拿總片數，再依序讀 B_0 到 B_9。用末碼減法最多只能切 10 片（上限 200 Bytes）。這不是精妙設計，就是不懂協議硬幹出來的歷史技術債。但線上既然能跑，沒人會手癢去動它。

### 2.2 寫入帳密與網址：老代碼的 2-Byte 標頭分包

長字串格式：`[Index(1B), TotalLength(1B), Payload(10B)]`。

翻開當年的 `BleUtils.java`，這才是原汁原味的現場土砲代碼：

```java
// File: BleUtils.java (老代碼的真面目)
private static byte[] stringBytes; // 靜態共享緩衝區，多特徵值並發直接踩爛

public static boolean multipleBytesToString(byte[] bytes, StringBuffer sb) {
    byte index = bytes[0];
    byte length = bytes[1]; // 有符號 byte，長度大於 127 直接變負數死鎖

    int sbLengthShouldBe = index * 10;
    if (sbLengthShouldBe < sb.length()) sb.delete(0, sb.length());
    else if (sbLengthShouldBe != sb.length()) return false;

    String str = bytesToString(bytes); // 內部用 static 陣列 memcpy 截掉前 2 bytes
    sb.append(str);
    return length == sb.length();
}
```

每次只傳 10 Bytes 淨載，加上 2 Bytes 標頭才 12 Bytes。代碼裡雖然留著靜態陣列共享和有符號 byte 的隱患，但在配網這種只有手機單線程循序寫入的場景下，它居然就這麼奇蹟般地穩穩跑了好幾年。

### 2.3 靜態 IP：別亂叫 TLV，這叫定長幀

別被「TLV」忽悠了，看這 7 個 Byte：
`[Type(1B), Ver(1B), Key(1B), IPv4(4B)]`

**這裡面壓根沒 Length 欄位**，IPv4 長度永遠固定 4 Bytes。它就是個普通的 7-Byte 定長控制幀：
- Type: `0x01 (ETH)`
- Ver: `0x04 (IPv4)`
- Key: `0x01=IP`, `0x02=Mask`, `0x03=Gateway`, `0x04=DNS1`, `0x05=DNS2`
- IPv4: 4 個 Raw Bytes

```java
// File: BleService.java (行 377-437 節選)
byte type = requestBytes[0];
if (type == ETH) {
    byte key = requestBytes[2];
    byte[] ipBytes = Arrays.copyOfRange(requestBytes, 3, requestBytes.length);
    if (key == 0x01) staticIpConfig.ip = BleUtils.ipv4ToString(ipBytes);
    else if (key == 0x05) { // 收到 DNS2 當作最後一筆，直接生效
        staticIpConfig.dns2 = BleUtils.ipv4ToString(ipBytes);
        ethernetManager.setStaticIp(staticIpConfig);
    }
}
```

---

## 結語

別把工控現場想得太優雅。

對於一台**沒螢幕、沒鍵盤的純金屬盲盒機**，BLE 是它唯一的現場物理生命線。

撕下那些把偷懶包裝成「深思熟慮」的遮羞布，承認當初為了早點下班採用的 20-Byte 最小公分母——在極限資源與惡劣現場面前，能穩定跑通、沒人半夜打電話叫你回公司修 Bug 的土砲，就是最好的架構。
