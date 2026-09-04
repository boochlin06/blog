# Android Camera analyze (一) – Camera 1 Architecture

> 原文網址：https://boochlin.com/?p=20
> 發布日期：2015-03-29
> 文章編號：20

---

** [Introduction of Android Camera1](//www.slideshare.net/boochlin/camera1-46414495)**

這篇算是比較久的資料

主要是在談andorid 4.4之前的Camera架構

在5.0之前都是以 camera 1 的api為主

在研究camera架構時，要特別注意不少都要跟底層溝通，（不過本篇也沒談到kernel那一端）

想當然必須要看c code，這點可能需要一些時間

 

#### page 4 :

1.Camera framework to native code 放在xxx

會build成xxxxxxxx.so

2.Camera 是走 servce client的方式在運作，所以有這些實作的class

Before 4.2

frameworks\base\include\camera

 frameworks\base\libs\camera

 frameworks\base\services\camera\libcameraservice\

#### page 5 :

三個so之間的關係

Camera java call to native by jni 藉由libandroid_runtime.so

調用libcamera_client去對libcameraservice.os對hard ware 操作

#### page 6 :

整理起來就是這樣

1. Java framewrok 呼叫

2. 透過jni去call native 的 framewrk

3. 利用ipc的方式去跟camera service 溝通

4. Camera 則是呼叫 hal interface 去操作haradware

5. 而hal 的實作則是由各個vendor去完成

#### page 7 :

**由java jni去呼叫camera的實體**

** Camera 主要有兩大功能:**

1. 接收來自service 的callback 所以他繼承cmera client 並實作 ex:jpeg call back

2. 對camera service 直接操作(像是 start previcew 所以同時也有個 camera:service:client的成員，實作於icamera

由於都是採用ipc 的架構，所以會有bp,bn的class

Bp就是代理人窗口，bn就是本人

Icameraservice則是提供initialize open時會使用到的

#### page 8 到 18:

這裡非常建議參考

[http://blog.chinaunix.net/uid-28416178-id-3446034.html](http://blog.chinaunix.net/uid-28416178-id-3446034.html)

老實說應該寫的比我好 ，不過可以參照投影片上的code片段，有更新成後期的code。

 

#### page 19:

如果有認真看完前幾頁，應該很好理解

這是一份從上到下的串連過程，以startPreview()的過程為例

1. 透過bnCameraClient 呼叫 Icamera

2. Icamera 利用 ipc 的機制與BnCamera溝通

3. Hal層完成之後一樣透過ipc回應給Icamera 發送完成資料

4. 同時notifyCallback給 IcameraClient 給上層通知完成

要注意的是 發送完成資料跟notifycallback 是走不同的ipc過程

#### page 20:

Callback的過程介紹

**1. CameraService:call back from hardward , flow start from camera client ex:jpeg call back.**

Code:

```
client->handleCompressedPicture(dataPtr);
```

從上面取得的client 端 呼叫handleCompressedPicture，完成的資料則是 dataPtr

**2. ClientCamera:use: ipc binder “bpcameraclient , bncameraclient in IcameraClient ****to send data , and implement in Camera.cpp.**

 Code:

```
sp c = mRemoteCallback;
c->dataCallback(CAMERA_MSG_COMPRESSED_IMAGE, mem, NULL);
```

還是要先取得 IcameraClient 然後發出 dataCallback

比較特別的是這裡要開始帶入 指令(CAMERA_MSG_COMPRESSED_IMAGE)+data(mem)

#### page 21:

**3. JNI : use ipc binder “bpcameraclient , bncameraclient in IcameraClient to send data ****, and implement in Camera.cpp.**

 Code: 

```
listener->postData(msgType, dataPtr, metadata);
終於接到了jni層，這裡listener是上面所設定的接受者
```

#### page 22:

**承續上頁 CameraListener 的設計，這裡的寫法是為了和 JNIEnv 做銜接.**

** Note:**

msgType enum values and data structures, ex: “camera_frame_metadata_t”

are defined in “system\core\include\system\camera.h.

```
class CameraListener: virtual public RefBase
{
public:
virtual void notify(int32_t msgType, int32_t ext1, int32_t ext2) = 0;
virtual void postData(int32_t msgType, const sp& dataPtr,
camera_frame_metadata_t *metadata) = 0;
virtual void postDataTimestamp(nsecs_t timestamp, int32_t msgType,
const sp& dataPtr) = 0;
};
```

#### page 23:

**4. Implement cameraListener post data in android_haradware_camera.cpp.**

Code: 

```
copyAndPost(env, dataPtr, dataMsgType);
```

**5.post image data to Java by JNI.**

Code:

```
env->CallStaticVoidMethod(mCameraJClass, fields.post_event,
mCameraJObjectWeak, msgType, 0, 0, obj);
```

個人認為這裡可以詳細去研究jni的機制，這裡不多談，很酷的手法

#### page 24:

**6. Call back to ap level :Framework/base/core/java/android/hardware/camera.java.**

Code: 

```
public void handleMessage(Message msg) {
switch(msg.what) {
case CAMERA_MSG_COMPRESSED_IMAGE:
mJpegCallback.onPictureTaken((byte[])msg.obj, mCamera);
```

終於接到java層啦，看c code真的有點累，從ap層註冊的callback發送回去

以上完成，下一回則是新的api 運用  “camera2”

ref:

http://blog.chinaunix.net/uid-28416178-id-3446034.html

http://blog.csdn.net/llping2011/article/details/9372799

http://blog.163.com/shawpin@126/blog/static/116663752201092394147937/

http://www.eoeandroid.com/blog-10407-5106.html
