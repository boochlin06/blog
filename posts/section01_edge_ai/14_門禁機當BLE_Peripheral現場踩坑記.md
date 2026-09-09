# Android 爛裝置想跑人臉辨識14 – 門禁機當 BLE Peripheral 的現場踩坑記

做邊緣工控或門禁設備，最讓人頭痛的往往不是演算法，而是現場環境：機器被四顆螺絲死死鎖在兩公尺高的金屬立柱上。更致命的是，工控設備存在**完全沒有螢幕、沒有鍵盤、全機身零實體按鍵的純金屬防暴室外機（Headless）版本**。

一旦現場更換路由器、變更 WiFi 密碼或需要設定固定 IP，設備連個按鍵都沒有，不可能叫維護人員搬梯子把整台金屬外殼拆開接串口。

透過近場藍牙將門禁機設計為 **BLE Peripheral（GATT Server）**，由安裝工人的手機擔任 Central，是讓這類無螢幕純硬體設備具備現場配置與通訊能力的唯一工程解。

這篇以高通 QM215（4 顆弱雞 A53）門禁機為例，直擊一套在工控現場能穩定交付的 BLE Peripheral 架構設計與真實權衡。

---

## 先講結論：工控 BLE Peripheral 的五個核心架構決策

在資源受限且無螢幕的邊緣設備上搞 GATT Server，核心在於極限避繁就簡：

1. **門禁機當被動方（Peripheral / GATT Server）**：開機在 `MainActivity.onCreate()` 中無條件啟動，強制開啟藍牙硬體並建立 GATT Server，7x24 常駐。手機當主動發起方（Central）。廣播直接交由藍牙晶片排程，主 CPU 負載為 0%，絕對不搶佔相機雙鏡頭採集與人臉辨識的算力。
2. **31-Byte 雙層廣播拆分**：標準廣播封包只有 31 Bytes，扣除必要標頭根本塞不下專屬 128-bit Service UUID。將設備短序號留在第一層主廣播，長 UUID 丟到第二層（ScanResponse），手機點擊掃描時才回傳。
3. **拒絕 MTU 協商，死守 20-Byte 最小公分母**：配網總資料量（WiFi 帳密、靜態 IP、後台網址）全部加起來連 100 Bytes 都不到，搞雙端非同步 `requestMtu()` 純屬過度設計。直接鎖死 BLE 預設的 20-Byte 物理基線，手機端零協商代碼、連線即發，半天完成雙端對接交付。
4. **狀態反饋放棄 Notification，改走無狀態主動輪詢**：雖然設備端保留了 NOTIFY 特徵值與 0x2902 描述符，但手機端在配網這種只有幾十秒的流程中，為了避開非同步回調丟失，改用每秒主動 Read 一次 6-Byte 狀態，讀到 1 亮綠燈斷線收工。
5. **程式碼開機常駐永不關閉**：全專案唯一呼叫 `stopBleService()` 只有在 App 銷毀時。設備插市電無功耗焦慮，為了免去半年後改密碼要爬梯拆外殼拉總閘斷電的運維噩夢，常駐廣播是現場必然代價。

下圖為現場跑通的完整時序與角色定義：

![現場配網時序與角色定義架構圖](../../assets/images/ble_provisioning_sequence_diagram_1788359237209.jpg)

---

## 一、 通訊選型：角色倒轉與 31-Byte 雙層廣播

很多 Android 藍牙教學都預設以手機連接手環為場景，教你如何用 Android 當 Central 去掃描周邊。

但在工控閘機上，角色必須徹底倒轉：
* **若門禁機當 Central（主動掃描）**：閘機每天幾百人進出，藍牙晶片一旦開啟 Scan，空中數百隻耳機、手環的廣播封包會持續引發晶片中斷，瘋狂搶佔 CPU 時槽，相機畫面瞬間掉幀卡頓。
* **門禁機當 Peripheral（被動監聽）**：開機時將廣播封包註冊進底層藍牙控制器，主 CPU 完全放手，不產生任何排程負載；只有在手機真正發起 GATT 連線並寫入特徵值時，才會觸發 Binder 回調。

### 1.1 一開機就啟動的常駐機制

門禁終端採用專用 Kiosk 模式，開機進入桌面自動拉起 `MainActivity`，在 `onCreate()` 生命週期直接初始化：

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

標準廣播物理淨載只有 31 Bytes。若將 16 Bytes 的專屬 128-bit Service UUID 塞入主廣播，設備名稱只剩幾個字元，甚至會直接拋出廣播資料超限錯誤。

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

## 二、 20-Byte 限制下的協議設計：拒絕 MTU 協商的實戰真相

BLE 規範中，預設的單包 ATT Payload 只有 **20 Bytes**（預設 ATT MTU 23 扣除 3 Bytes 的 ATT Opcode 與 Attribute Handle）。

很多人會問：為什麼不調用 `requestMtu()` 擴展到 256 或 512 Bytes？

實戰中放棄 MTU 協商的理由非常純粹：
1. **資料量少到不配搞協商**：WiFi SSID、WiFi 密碼、靜態 IP、雲端管理後台網址，全部加起來連 100 Bytes 都不到，空中傳輸頂多幾十毫秒。為這點資料量在雙端寫非同步 `requestMtu()` 握手回調，純屬過度設計。
2. **GATT 角色限制**：在 BLE 規範中，`requestMtu()` 是 Central（手機端）的專屬權限，門禁機作為 GATT Server 根本無權主動發起協商。
3. **極限降低跨團隊對接成本**：現場配網的手機 App 往往是前端或外包工程師用 Flutter 或跨平台框架開發。直接定死 20-Byte 物理基線，手機端無腦寫個 `for` 迴圈分包發送，雙端完全不需要寫任何 MTU 協商代碼，連線即發，半天就能對接上線。

![BLE 三大二進制通訊封包佈局圖](../../assets/images/ble_binary_packet_layouts_1788423952069.jpg)

### 2.1 WiFi 帳密與後台網址：2-Byte 標頭分包

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

### 2.2 乙太網路靜態 IP：7-Byte 定長二進制控制幀

工廠常走固定 IP。這套封包長度固定為 7 Bytes：
`[Type(1B), Ver(1B), Key(1B), IPv4(4B)]`

注意：這不是 TLV，因為它沒有 Length 欄位，而是純粹的定長二進制幀：
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
    else if (key == 0x05) { // 收到最後一個參數觸發生效
        staticIpConfig.dns2 = BleUtils.ipv4ToString(ipBytes);
        ethernetManager.setStaticIp(staticIpConfig);
    }
}
```

---

## 三、 狀態反饋：從 Notification 到主動輪詢的工程妥協

照藍牙官方規範，連網狀態變更應透過 `notifyCharacteristicChanged` 推播給手機。

在 `BleService.java` 中，特徵值確實宣告了 `PROPERTY_NOTIFY`，也老實掛上了 `0x2902` 描述符（`DESC_NOTIFY_KMS_CONNECTED`），甚至在 `onDescriptorWriteRequest` 中寫了線程推播的原型代碼。

但實務上手機端最終選擇了**「每秒主動 Read 狀態」**的輪詢機制。

這在消費級 BLE 開發中屬於反模式，但在配網這種僅維持幾十秒的流程中，卻是現場最穩的土砲解：
- 手機端在跨平台框架下處理 0x2902 描述符寫入與非同步監聽時，回調丟失率偏高；
- 門禁機插著市電無功耗顧慮，手機連上藍牙後，每秒發一次 `readCharacteristic` 讀取 6 個 Byte 狀態：
  `[Status(1B), Reserved(1B), DeviceIPv4(4B)]`
  Status: `0x01 (已連上雲端管理後台)`, `0x02 (正在連線)`, `0x03 (閒置/失敗)`
- 手機只要讀到 `0x01`，當場亮綠燈並主動中斷藍牙。不依賴空中推播，現場狀態閉環收斂率 100%。

---

## 四、 7x24 常駐與維運的現場權衡

檢視全專案代碼，`stopBleService()` 唯一的調用點只有在 `MainActivity.onDestroy()`。專案中**完全沒有**寫任何「開機 10 分鐘後自動關閉」或「連網成功後關閉藍牙」的邏輯。

這不是疏忽，而是面對無螢幕工控機的現實考量：
- 機器被螺絲鎖在兩米高處，機身無螢幕、無鍵盤、無實體重置鍵；
- 若設定連線成功後自動關閉藍牙，半年後廠區更換路由器密碼或更換機房網段時，現場維護人員唯一能重新配網的手段，就是去拉總配電箱總閘斷電重啟；
- 維持 7x24 常駐廣播，讓後續維護人員拿著手機隨時走過去就能重新配網，是用極低射頻開銷換取最低維護成本的現場最優解。

---

## 五、 現場排障速查清單 (Troubleshooting Cheat Sheet)

在現場排查 BLE 通訊異常時，看這兩點：

* **現象 1：手機搜得到廣播，但一發起寫入就卡死或逾時斷線**
  * 排查點：確認 `onCharacteristicWriteRequest` 是否嚴格根據 `if (responseNeeded)` 回覆 ACK：
    ```java
    // 只有當協議要求回覆（Write Request）時才送 ACK
    // 若對 Write Command (WRITE_TYPE_NO_RESPONSE) 強行 sendResponse，手機藍牙 Stack 會當場錯亂
    if (responseNeeded) {
        mBluetoothGattServer.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, offset, requestBytes);
    }
    ```
* **現象 2：手機端掃描清單中完全看不到設備，或開機時廣播拋錯**
  * 排查點：檢查第一層主廣播數據長度。確認設備名稱加上發射功率長度是否超過 31 Bytes；確認 128-bit Service UUID 是否有正確放進第二層 ScanResponse。

---

## 結語

在有螢幕的裝置上，藍牙只是輔助；但對於**無螢幕、無鍵盤的純盲盒邊緣工控機**，BLE Peripheral 是它唯一的現場物理生命線。

拋開教科書上繁瑣的動態 MTU 協商，死守 20-Byte 最小公分母與無狀態主動輪詢，才是邊緣工控設備在真實環境中最務實的生存之道。
