# Android 爛裝置想跑人臉辨識14 – 門禁機當 BLE Peripheral 的現場踩坑記

做邊緣工控或門禁設備，最讓人頭痛的往往不是演算法，而是現場環境：機器被四顆螺絲死死鎖在兩公尺高的金屬立柱上。

很多人以為配網是為了解決「螢幕反光不好戳鍵盤」的體驗問題，但真實情況比這殘酷得多：門禁產品線存在大量**完全沒有螢幕、沒有鍵盤、全機身零實體按鍵的純金屬防暴室外機（Headless 盲盒版本）**。

這類純硬體設備一旦在現場更換路由器、變更 WiFi 密碼或需要設定固定 IP，連個可供操作的實體介面都沒有，總不可能叫維護人員搬梯子把防水密封的金屬外殼拆開接串口。

透過近場藍牙將門禁機設計為 **BLE Peripheral（GATT Server）**，由安裝工人的手機擔任 Central，是讓這類無螢幕純硬體在現場能夠配置與診斷的唯一生命線。

這篇拿高通 QM215（4 顆弱雞 A53）門禁機為例，直接撕開那些看似高大上的理論包裝，還原一套現場能跑通、但也充滿妥協與黑歷史的 BLE Peripheral 實戰代碼。

---

## 先講結論：五個實戰取捨（去偽存真）

在資源受限且無螢幕的工控設備上搞 GATT Server，核心在於極限避繁就簡：

1. **門禁機當被動方（Peripheral / GATT Server）**：開機在 `MainActivity.onCreate()` 中無條件啟動，強制開啟藍牙硬體並建立 GATT Server，7x24 常駐。手機當主動發起方（Central）。廣播直接交由藍牙控制器排程，主 CPU 負載為 0%，絕對不搶佔相機雙鏡頭採集與人臉辨識的算力。
2. **31-Byte 雙層廣播拆分**：標準廣播封包物理極限只有 31 Bytes，扣除必要標頭根本塞不下專屬 128-bit Service UUID。將設備短序號留在第一層主廣播，長 UUID 丟到第二層（ScanResponse），手機點擊掃描時才回傳。
3. **拒絕 MTU 協商，直接鎖死 20-Byte 預設基線**：配網總資料量（WiFi 帳密、固定 IP、後台網址）加起來連 100 Bytes 都不到，空中傳輸幾十毫秒的事，搞雙端非同步 `requestMtu()` 純屬沒事找事的過度設計。直接鎖死 BLE 出廠預設的 20-Byte 物理基線，手機端（通常是外包或前端寫的）零協商代碼、連線即發，半天對接上線。
4. **放棄 Notification，改走無狀態主動輪詢**：雖然設備端老老實實宣告了 NOTIFY 特徵值並掛上 0x2902 描述符，但手機端為了省去處理非同步回調監聽的麻煩，索性直接用定時器每秒發一次 Read 讀取 6-Byte 狀態，讀到 1 亮綠燈斷線收工。
5. **程式碼開機常駐永不關閉**：全專案唯一呼叫 `stopBleService()` 只有在 App 銷毀時。設備插市電無功耗焦慮，為了免去半年後改密碼要爬梯拆外殼拉總閘斷電的運維噩夢，常駐廣播是必然代價。

下圖為現場跑通的完整時序與角色定義：

![現場配網時序與角色定義架構圖](../../assets/images/ble_provisioning_sequence_diagram_1788359237209.jpg)

---

## 一、 角色倒轉與 31-Byte 雙層廣播

很多 Android 藍牙教學都預設以手機去連手環為範例，教你如何用 Android 當 Central 去掃描周邊。

但在工控閘機上，角色必須徹底倒轉：
* **若門禁機當 Central（主動掃描）**：閘機每天幾百人進出，藍牙晶片一旦開啟 Scan，空中數百隻耳機、手環的廣播封包會持續引發晶片中斷，瘋狂搶佔 CPU 時槽，相機畫面瞬間掉幀卡頓。
* **門禁機當 Peripheral（被動監聽）**：開機時將廣播封包註冊進底層藍牙控制器，主 CPU 完全放手，不產生任何排程負載；只有在手機真正發起 GATT 連線並寫入特徵值時，才會觸發 Binder 回調。

### 1.1 一開機就啟動的常駐機制

門禁終端採用專用 Kiosk 模式，開機進入桌面自動拉起 `MainActivity`，在 `onCreate()` 生命週期直接無條件啟動：

```java
// File: MainActivity.java
@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    BleService.createInstance(this); // 開機無條件啟動，7x24 常駐
    ...
}
```

在 `BleService` 建構子內，若藍牙未開直接強制開啟，並透過 `bluetoothManager.openGattServer()` 完成服務註冊：

```java
// File: BleService.java (行 147-150, 231-233)
BluetoothAdapter bluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
if (bluetoothAdapter != null && !bluetoothAdapter.isEnabled()) {
    bluetoothAdapter.enable(); // 硬體保底強制啟動
}
...
if (bluetoothManager != null)
    mBluetoothGattServer = bluetoothManager.openGattServer(context, mBluetoothGattServerCallback);
mBluetoothGattServer.addService(service);
```

### 1.2 31-Byte 雙層廣播拆分實作

標準廣播物理淨載只有 31 Bytes。若將 16 Bytes 的專屬 128-bit Service UUID 塞入主廣播，設備名稱只剩幾個字元，甚至直接拋錯。

解法是將廣播拆為兩層：第一層只放發射功率與機號，長 UUID 移至第二層 ScanResponse：

```java
// File: BleService.java
uniqueSerial = DeviceCodeUtils.getSerialNum().substring(4);
bluetoothAdapter.setName(deviceName + "-" + uniqueSerial);

// 第一層主廣播：僅設備名稱與發射功率（嚴格控制在 31B 內）
AdvertiseData advertiseData = new AdvertiseData.Builder()
        .setIncludeDeviceName(true).setIncludeTxPowerLevel(true).build();

// 第二層掃描回應：128-bit 專屬 Service UUID 丟在 ScanResponse
AdvertiseData scanResponse = new AdvertiseData.Builder()
        .addServiceUuid(new ParcelUuid(UUID_SERVICE)).build();
```

---

## 二、 20-Byte 限制下的協議：不用裝逼，就是為了好交差

BLE 規範預設單包 ATT Payload 只有 **20 Bytes**（預設 ATT MTU 23 扣除 3 Bytes 的 ATT Opcode 與 Attribute Handle）。

為什麼不搞動態 MTU 協商？
實戰中的理由非常直白：
1. **資料量少到根本不配搞協商**：WiFi 帳密 20 Bytes、固定 IP 4 Bytes、雲端管理後台網址 30 Bytes，全部加起來連 100 Bytes 都不到。為這點資料量在雙端寫非同步 `requestMtu()` 握手回調，純屬過度設計。
2. **GATT 角色限制**：在 BLE 規範中，`requestMtu()` 是 Central（手機端）的專屬權限，門禁機作為 GATT Server 根本無權主動發起協商。
3. **極限降低跨團隊對接成本**：現場配網的手機 App 往往是外包或前端工程師用跨平台框架開發。直接定死 20-Byte 物理基線，手機端寫個最無腦的 `for` 迴圈分包發送，雙端完全不需要寫任何 MTU 協商代碼，連線即發，半天就能測通交付。

![BLE 三大二進制通訊封包佈局圖](../../assets/images/ble_binary_packet_layouts_1788423952069.jpg)

### 2.1 讀取 Device ID：不懂 Read Blob 留下的土砲遺產

在 BLE GATT 規範中，讀取大於 20 Bytes 的長特徵值，官方標準機制叫 **Read Blob Request（利用回調中的 `offset` 參數自動分段撈取）**，只要宣告一個特徵值就能搞定。

但翻開舊代碼，當初寫這段的人顯然不知道 `offset` 是幹嘛用的，居然在迴圈裡動態 `addCharacteristic` 註冊了 10 個不同的 UUID，然後用字串末碼減法算位移：

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

這種靠 `uuid[uuid.length - 1] - '0'` 的寫法最多只能切 0 到 9 共 10 個分片（上限 200 Bytes）。這不是什麼高深的分片演算法，而是不懂標準協議硬幹出來的歷史技術債。

### 2.2 寫入 WiFi 帳密與後台網址：2-Byte 標頭分包

長字串封包格式定義為：`[Index(1B), TotalLength(1B), Payload(10B)]`。

每次僅傳輸 10 Bytes 淨載，加上 2 Bytes 標頭僅 12 Bytes，遠小於 20-Byte 限制：

```java
// File: BleUtils.java
public static boolean multipleBytesToString(byte[] bytes, StringBuffer sb) {
    int index = bytes[0] & 0xFF;
    int length = bytes[1] & 0xFF; // 無符號轉換，避免長度大於 127 溢位為負數

    int expectedLen = index * 10;
    if (expectedLen < sb.length()) sb.delete(0, sb.length()); // 序號回退代表重傳，清空
    else if (expectedLen != sb.length()) return false;        // 序號不符直接丟棄

    String payload = new String(bytes, 2, bytes.length - 2, StandardCharsets.UTF_8);
    sb.append(payload);
    return length == sb.length(); // 收滿預期總長度才返回 true
}
```

### 2.3 乙太網路靜態 IP：7-Byte 定長二進制控制幀

很多文章喜歡把二進制封包包裝成高大上的「TLV（Type-Length-Value）」，但檢視實際的 7-Byte 封包：
`[Type(1B), Ver(1B), Key(1B), IPv4(4B)]`

**這裡面根本沒有 Length（長度）欄位**，因為 IPv4 長度永遠固定是 4 Bytes。它就是個純粹的定長二進制控制幀：
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
    else if (key == 0x05) { // 收到 DNS2 當作最後一筆，觸發生效
        staticIpConfig.dns2 = BleUtils.ipv4ToString(ipBytes);
        ethernetManager.setStaticIp(staticIpConfig);
    }
}
```

---

## 三、 狀態反饋：從 Notification 到主動輪詢的工程妥協

在 `BleService.java` 中，特徵值確實宣告了 `PROPERTY_NOTIFY`，也掛上了 `0x2902` 描述符（`DESC_NOTIFY_KMS_CONNECTED`），甚至在 `onDescriptorWriteRequest` 中寫了線程推播的原型代碼。

但實務上手機端最終選擇了**「每秒主動 Read 狀態」**的輪詢機制。

這在消費級 BLE 開發中屬於反模式，但在配網這種僅維持幾十秒的封閉流程中，純粹是手機端為了避開非同步監聽的偷懶妥協：
- 手機端用跨平台框架處理 0x2902 描述符寫入與非同步推播監聽時，容易遇到回調丟失與狀態不同步；
- 門禁機插著市電無功耗顧慮，手機連上藍牙後，直接用定時器每秒發一次 `readCharacteristic` 讀取 6 個 Byte 狀態：
  `[Status(1B), Reserved(1B), DeviceIPv4(4B)]`
  Status: `0x01 (已連上雲端管理後台)`, `0x02 (正在連線)`, `0x03 (閒置/失敗)`
- 手機只要讀到 `0x01`，當場亮綠燈並主動中斷藍牙。雖然土砲，但現場狀態閉環收斂率 100%。

---

## 四、 真實翻車點與原地自癒

拋開那些過度修飾的理論，這套代碼真正留在線上的技術地雷主要有兩處：

### 4.1 誤回 ACK 的 API 誤用

在 `onCharacteristicWriteRequest` 結尾，原代碼無條件執行：
`mBluetoothGattServer.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, offset, requestBytes);`

這是一個經典的 API 誤用。BLE 規範中 Write Command（`WRITE_TYPE_NO_RESPONSE`）明令禁止回覆，此時系統傳入的 `responseNeeded` 為 `false`。強行回覆會導致手機藍牙協議棧狀態機錯亂。

必須嚴格加入判斷：
```java
// 只有在 responseNeeded 為 true 時才回覆 ACK
if (responseNeeded) {
    mBluetoothGattServer.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, offset, requestBytes);
}
```

### 4.2 20 秒盲等 while 迴圈的荒謬時序

原代碼中有一段令人啼笑皆非的寫法：在 WiFi 連線成功的回調裡，啟動背景線程開 while 迴圈盲等 20 秒去檢查後台網址變數 `serverHostDone`。

這種「依賴特定發包先後順序」的代碼在非同步通訊中極其脆弱。自癒解法根本不需要盲等，收到後台網址當下直接寫入本機 Settings 即可，連線服務自會定時重試：

```java
// 收到網址當下即刻持久化，徹底拔除 20 秒盲等線程
case CHAR_WRITE_SERVER_HOST_URL:
    if (BleUtils.multipleBytesToString(requestBytes, sbServerHost)) {
        String fullUrl = sbServerHost.toString().replaceFirst("^(http[s]?://)", "");
        SettingServ.setSyncServerIp(fullUrl);
        sbServerHost.setLength(0);
    }
    break;
```

---

## 五、 現場排障速查清單 (Troubleshooting Cheat Sheet)

在現場排查 BLE 通訊異常時，看這兩點：

* **現象 1：手機搜得到廣播，但一發起寫入就卡死或逾時斷線**
  * 排查點：確認 `onCharacteristicWriteRequest` 是否嚴格根據 `if (responseNeeded)` 回覆 ACK，避免對 Write Command 強推 `sendResponse`。
* **現象 2：手機端掃描清單中完全看不到設備，或開機時廣播拋錯**
  * 排查點：檢查第一層主廣播數據長度。確認設備名稱加上發射功率長度是否超過 31 Bytes；確認 128-bit Service UUID 是否有正確放進第二層 ScanResponse。

---

## 結語

對於**無螢幕、無鍵盤、全機身零實體按鍵的純盲盒邊緣工控機**，BLE Peripheral 是它唯一的現場物理生命線。

撕下那些「白牌手機相容性地獄」的遮羞布，承認當初為了半天交付而採用的 20-Byte 最小公分母與無狀態主動輪詢，並把漏掉的 API 判斷和盲等迴圈修乾淨——這才是工控現場最真實的工程實戰。
