# FMRadio 開發筆記 （二）- FMRadioMain

> 原文網址：https://boochlin.com/?p=5
> 發布日期：2015-02-08
> 文章編號：5

---

## FMRadioMain

#### **LAYOUT**

 

![fmradioMain](../../assets/images/fmradioMain.png)

[https://boochlin.com/wp-content/uploads/2015/02/fmradioMain.png](https://boochlin.com/wp-content/uploads/2015/02/fmradioMain.png)

## 

## 1.常用class

### **NumberPicker**

[http://code.google.com/p/taketoma-android-number-picker/](http://code.google.com/p/taketoma-android-number-picker/)

類似可輸入的DIALOGUE,

[http://blog.csdn.net/barryhappy/article/details/7363230](http://blog.csdn.net/barryhappy/article/details/7363230)

**** ****

### **AudioManager**

用於音量調整

```
setVolumeControlStream(AudioManager.STREAM_FM);
```

設定收音機的音量控制策略

STREAM_FM IS A STREAM STRATGE  HAS 10 LEVEL VOLUME

### **DisplayMetric **

取得螢幕的相關資訊類別

藉著幾行程式碼來取得手機螢幕的解析度，當中的關鍵為DisplayMetrics類別應用。

```
DisplayMetrics metrics = new DisplayMetrics();

getWindowManager().getDefaultDisplay().getMetrics(metrics);

TextView TextView1 = (TextView)findViewById(R.id.TextView01);

TextView1.setText("手機銀幕大小為 "+metrics.widthPixels+" X "+metrics.heightPixels);
```

[http://style77125tech.pixnet.net/blog/post/16792883-%5Bandroid%5D-displaymetrics-%E5%8F%96%E5%BE%97%E6%89%8B%E6%A9%9F%E8%9E%A2%E5%B9%95%E5%A4%A7%E5%B0%8F](http://style77125tech.pixnet.net/blog/post/16792883-%5Bandroid%5D-displaymetrics-%E5%8F%96%E5%BE%97%E6%89%8B%E6%A9%9F%E8%9E%A2%E5%B9%95%E5%A4%A7%E5%B0%8F)

**** ****

### **LoadedDataAndState**

是FMRadio2中獨有物件

於FMRadio2中為保存service中的fm硬體的開關機狀態

物件會在onRetainNonConfigurationInstance()階段時，進行組態內容保存

****

```
@override

public object onretainnonconfigurationinstance()

{

    //这里需要保存的内容，在切换时不是bundle了，我们可以直接通过object来代替

    return LoadedDataAndState;

}
```

[http://chengbs.iteye.com/blog/1156167](http://chengbs.iteye.com/blog/1156167)Q

比较onsaveinstancestate() 与 onretainnonconfigurationinstance()在不同需求中的用法

很多网友可能知道android横竖屏切换时会触发onsaveinstancestate，而还原时会产生onrestoreinstancestate，但是android的activity类还有一个方法名为onretainnonconfigurationinstance和getlastnonconfigurationinstance这两个方法。

onRetainNonCongigurationInstance

当device configuration发生改变时，将伴随destroying被系统调用。通过这个方法可以像onsaveinstancestate()的方法一样保留变化前的activity state，最大的不同在于这个方法可以返回一个包含有状态信息的object，其中甚至可以包含activity instance本身。新创建的activity可以继承大量来至于parent activity state信息。

用这个方法保存activity state后，通过getlastnonconfigurationinstance()在新的activity instance中恢复原有状态。

於fmradio2中則是複寫成保留底層mservice是否啟動.故調用isFmOn();

 

 

## 2.some method

```
mLinearLayout = (LinearLayout) findViewById(R.id.layout_panel);
if (mLinearLayout != null){
mLinearLayout.setOnTouchListener(new OnTouchListener() {
//do something}
```

此段針對再PANEL上的滑動提供頻道切換的功能.根據移動距離切換

**** ****

### registerForContextMenu（mMenuButton）

將view的點擊後效果連結到contextmenu

**** ****

 

### **filter()**

於此過濾三種情況…

1.當電話接通後以及電話正在想…必須使用**TelephonyManager**進行**getCallState（）確認狀態**

2.沒有接上耳機…必須使用**(AudioManager) getSystemService(Context.AUDIO_SERVICE);**確認**isWiredHeadsetOn()**確認耳機是否戴上

3.確認是否為AirPlaneMode()…必須搭配

```
isAirplaneModeEnabled（）…裡面實作
   Settings.System.getInt(context.getContentResolver(),
   Settings.System.AIRPLANE_MODE_ON) == 1;
```

確認手機系統狀態

**** ****

**←————————————divider————————————>**

### **isCallActive()**

向mService確認電話狀態

個人覺得的有點多此一舉…可以直接再main裡面進行 **TelephonyManager**.getCallState()

**** ****

**←—————————-divider——————————————–>**

### **CreateFrequencyPickerDialog()**

創造選擇頻道的對話框

```
AlertDialog.Builder builder = new AlertDialog.Builder(this);

builder.setTitle(this.getString(R.string.menu_set_frequency));

 

LayoutInflater inflater = (LayoutInflater) this

    .getSystemService(LAYOUT_INFLATER_SERVICE);

View layout = inflater.inflate(R.layout.value_picker,

    (ViewGroup) findViewById(R.id.layout_root));
```

這兩個比較常用到

**** 此外****針對頻道選擇上作些限制…畢竟不是每個頻道都能選…可能是國籍或開放頻道的關係

因此****

```
FmConfig fmConfig = FmSharedPreferences.getFMConfiguration()
```

;

取的lower higher limitation,

還必須根據國籍調整

```
int countryCode = FmSharedPreferences.getCountry();
```

給予對應的string提示

**** ****

**←——————————-divider——————————————–>**

### **createRangeSettingsDialog（）**

由於國家的不同…頻寬開放的也不同…所以提供切換的功能

利用

```
int countryCode = FmSharedPreferences.REGIONAL_BAND_AUTO;
```

以及**FmSharedPreferences.setCountry(countryCode);**

從新進行Fmradio.service的從組**fmConfigure();**

 

**←——————————-divider——————————————–>**

### **createProgressDialog()**

```
boolean bSearchActive = isSeekActive() || isScanActive() ||isSearchActive();
```

確認mService的運作狀態 當還沒有任何search相關的動作會創見新的dialog

目前是專為 isFirstRun而設計的…只有此時會用到progress dialog

**** ****

**←——————————divider——————————————–>**

### **createCmdXXXDlg()**

這類皆為原生method()

為提供各種警告提示

**** ****

### **searchFavoriteChanel（）**

為提供給favoriteChange使用…由於再切換我的最愛電台時…可能會出現此最愛電台並非當前mService所服務的區段…因此我們必須先去檢查下一個或上一個最愛電台是否合乎所選擇的國籍與區段

**** ****

**←—————————–divider——————————————–>**

### **enableRadioOnOffUI(false);**

於程式判斷是否有待上天線

enableRadioOnOffUI(false);

`// HDMI and FM concurrecny is not supported.`

並不支援同時有fm and hdmi

```
if (isHdmiOn()) {

    showDialog(DIALOG_CMD_FAILED_HDMI_ON);
} else {
    if (false == bindToService(this, osc)) {

        Log.d(LOGTAG, "onCreate: Failed to Start Service");

    } else {
        Log.d(LOGTAG, "onCreate: Start Service completed successfully");
    }
}
```

**** ****

```
private boolean isHdmiOn() {
// HDMI and FM concurrecny is not supported.

    try {
        String hdmiUserOption = android.provider.Settings.System.getString(
        getContentResolver(), "HDMI_USEROPTION");
        //這裡我試出來…hdmiUserOption == null;
        這段code沒有意義阿,永為false

    } catch (Exception ex) {
    }
return false;
}
```

**** ****

**←——————————–divider——————————————–>**

### bar頻道響應

```
private View.OnTouchListener mShiftFineTuneFigureListener = new View.OnTouchListener() {

@Override

public boolean onTouch(View v, MotionEvent event) {

這裡主要再處理當滑動底下shiftbar所對應的動作…原本想做祥解的

淡大多數多為對耶面物件的位址計算…因此不多提了

每次重讀時…請自行看程式碼
```

**** ****

**←——————————-divider——————————————–>**

### **** Handler Control****

```
private Handler mEnableRadioHandler = new Handler();

private Handler mDisableRadioHandler = new Handler();

private Runnable mEnableRadioTask = new Runnable() {
```

這三個行為非常簡單…主要執行一些mService已經定義好的行為…用handler包起來則是另外必須對layout的控制

**** ****

**←——————————-divider——————————————–>**

### Search Progress control

```
private void resetSearchProgress() {

private void updateSearchProgress() {
```

再我們進行seek時…會去呼叫mService進行相關的chanel search…於此同時也必須對layout進行畫面更新.也就是searchProgress的顯示…這由於是修改前人的程式…所以也保留method的命名※但主要已經沒有progress bar的存在…而是progress txt的更新

會發出mSearchProgressHandler去跟新畫面(msg.what = UPDATE_PROGRESS_TXT_1;)

Scanning.   -> Scanning .. -> Scanning…

(UPDATE_PROGRESS_TXT_1)(UPDATE_PROGRESS_TXT_2)(UPDATE_PROGRESS_TXT_3)

search 結束後 發出END_PROGRESS_TXT

**** ****

**←——————————-divider——————————————–>**

### **** First Run Something****

```
private void FirstRunSeekNextStation() {
private void updateFirstRunSearchProgress() {
```

應用餘地一次開啟FMRADIO

跟前面的功能很像…但是這次不是UPDATE_PROGRESS_TXT_2…而是對mProgressDialog逕行控制

**** ****

這裡我認為FirstRunSeek與非firstrunSEEK很像

或許可以修改乘更加精簡…畢竟行為很像

可是這樣繪影想到可讀性

**** ****

所以先保持這樣好了

**←——————————-divider——————————————–>**

### UI Update

```
private void updateStationInfoToUI() {
private void updateFineTuneUI() {
```

基本上任何可能的行為※都會碰到修改LAYOUT的情況

**** ****

**←——————————divider——————————————–>**

### SeekSomething

```
private void SeekPreviousStation() {

private void SeekNextStation() {
```

Seek基本上就是移動到下一個具有效力的電台

 

 

**←——————————-divider——————————————–>**

### Animation for seek

************************************

```
private static final int UPDATE_PROGRESS_TXT_1 = 1;   //three type for scanning anim

    private static final int UPDATE_PROGRESS_TXT_2 = 2;

    private static final int UPDATE_PROGRESS_TXT_3 = 3;

    private static final int END_PROGRESS_TXT = 4;

    private static final int TIMEOUT_PROGRESS_DLG = 5;

    private static final int FIRSTRUN_UPDATE_PROGRESS_DLG = 6;

    private static final int FIRSTRUN_END_PROGRESS_DLG = 7;

    private static final int SHOWBUSY_TIMEOUT = 300000;

    private Handler mSearchProgressHandler = new Handler() {
```

**** ****

所有對於search所進行的畫面變動都在此除裡

要注意的是  message之間的交互關係…所以必須注意要

mSearchProgressHandler.**removeMessages**

**** ****

**←—————————–divider——————————————–>**

### **History Station**

```
private void addHistoryStation(PresetStation station){

follow tuneRadio();mSearchComplete;mUpdateStationInfo
```

有幾項規則

1.list有重複的則移動到最上面一個

2.沒有重復則放在地一個

3.list內總數不得超過10個

**** ****

為何要mSearchComplete.mUpdateStationInfo都加一個**addHistoryStation**

**如果只在searchcomplete後加入**

主要在於mSearchComplete和mUpdateStationInfo是不同的THREAD

在searchComplete之後去跟心畫面resetFMStationInfoUI（）

而同時和mUpdateStationInfo by onTuneStatusChanged()也在運行

再search時間極短下…有可能onTuneStatusChanged會先完成

mTunedStation並沒有被跟新成searchcomplete後的結果

此時searchcomplete就不會將最新的history station 加入

**** ****

所以在兩邊都加入addhistorystation會比較好

避免完成時間差resetFMStationInfoUI();

**** ****

**←——————————–divider——————————————–>**

**** ****

```
private void tuneRadio(int frequency) {
```

當改變後都會呼叫…接者後面跟者要加入歷史電台

讓mService去播放電台…..大概是最常用的功能

**** ****

**←——————————–divider——————————————–>**

**final Runnable mSearchListComplete = new Runnable() {**

等價於所謂的ScanComplete

**** ****

**←——————————-divider——————————————–>**

**private IFMRadioServiceCallbacks.Stub mServiceCallbacks = new IFMRadioServiceCallbacks.Stub() {**

**        public void onEnabled() {**

****[主要關於aidl的bindservice的機制](https://docs.google.com/document/d/1eZwGVfXnpBE5K8m-QjJdsHEvRdinKjbvE7XTffSGiEA/edit)

 

```
public static boolean bindToService(Context context, ServiceConnection callback)
```
