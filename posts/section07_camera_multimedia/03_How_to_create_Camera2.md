# Android Camera analyze (三) – How to create Camera 2.

> 原文網址：https://boochlin.com/?p=30
> 發布日期：2015-03-31
> 文章編號：30

---

### How to create a camera 2

 

** [How to create a camera2](//www.slideshare.net/boochlin/camera2-how-tocreate)**

 

#### page2:

**How to create a camera 2.**

Step1:To enumerate, query, and open available camera devices, obtain a CameraManager instance.

從system取得cameraManager。

```
// To get a list of available sizes of camera preview, we retrieve an instance of
// StreamConfigurationMap from CameraCharacteristics
CameraCharacteristics characteristics = manager.getCameraCharacteristics(cameraId);
StreamConfigurationMap map = characteristics
                    .get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
 (Camera.Size) mPreviewSize = map.getOutputSizes(SurfaceTexture.class)[0];
 (SurfaceTexture) mTextureView.setAspectRatio(mPreviewSize.getWidth(), 
// We are opening the camera with a listener. When it is ready, onOpened of
// (CameraDevice.StateListener)mStateListener is called.
manager.openCamera(cameraId, mStateListener, null);
```

#### page3:

**Initialize CameraDevice:**

Step2:Individual CameraDevices provide a set of static property information that describes the hardware device and the available settings and output parameters for the device.

根據system 取得 CameraCharacteristics 去準備開啟CameraDevice

```
CameraCharacteristics characteristics = manager.getCameraCharacteristics(cameraId);
manager.openCamera(cameraId, mStateListener, null);
```

#### page4:

**CameraDevice.StateListener:**

A listener for notifications about the state of a camera device

總是要去監聽device是否開啟，關閉，硬體錯誤等等

#### page5:

**CameraDevice:**

The CameraDevice class is a representation of a single camera connected to an Android device, allowing for fine-grain control of image capture and post-processing at high frame rates.

Device is modeled as a pipeline

重要的主控端，負責接收所有callback 和對device的控制

是一個pipline的模組。簡單來說的流程

Input : CaptureRequest for a single Frame

Output: CaptureResult Metadata packet + output image buffers for the request. These buffers can go to one or multiple output targets.

#### page6:

**Start preview :**

Step3:To capture or stream images from a camera device, the application must first create a camera capture session with a set of output Surfaces for use with the camera device.

開啟預覽畫面，基本上可以同時發到好幾個surface

這裡以單一surface為主，這裡要記得開啟capturesession是必須包含未來最大子集需要用到的surface

```
// We configure the size of default buffer to be the size of camera preview we want.
texture.setDefaultBufferSize(mPreviewSize.getWidth(), mPreviewSize.getHeight());
// This is the output Surface we need to start preview.
Surface surface = new Surface(texture);
// We set up a CaptureRequest.Builder with the output Surface.
mPreviewBuilder =  mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
mPreviewBuilder.addTarget(surface);
// Here, we create a CameraCaptureSession for camera preview.
mCameraDevice.createCaptureSession(Arrays.asList(surface),
                    new CameraCaptureSession.StateListener(){.....}
```

#### page7:

**CaptureRequest.Builder:**

– A builder for capture requests.

– To obtain a builder instance, use the createCaptureRequest(int) method, which initializes the request fields to one of the templates defined in CameraDevice.

所有的request都必須經過指定提供的builder的提供，這在不少android的設計都是這樣搞

像是notification。

通常要寫入captureRequest的field 設定和輸出的target surface.

#### page8:

**CaptureRequest:**

– An immutable package of settings and outputs needed to capture a single image from the camera device.

– Contains the configuration for the capture hardware (sensor, lens, flash), the processing pipeline, the control algorithms, and the output buffers. Also contains the list of target Surfaces to send image data to for this capture.

– CaptureRequests can be created by using a CaptureRequest.Builder instance, obtained by calling createCaptureRequest(int)

懶得解釋了，根據上頁，做出來的request通常會有大量的資訊，一般來說1 request  對應1 image data

這也是camera2 的重要特色，在request 中可以放入各種設定和輸出surface，所以可以同時有很多種變化在很短的時間(fps)

#### page9:

**CameraCaptureSession.StateListener:**

A listener for tracking the state of a camera capture session.

除了之前的cameradevice監控，之外還要監控capturesession狀況

前述有提過，當開啟cameraDevice還是要開啟專門的session去執行

captureSession的詳細內容可以參考下一頁

```
public void onConfigured(CameraCaptureSession cameraCaptureSession) {
             // When the session is ready, we start displaying the preview.
             mPreviewSession = cameraCaptureSession;
             // In this sample, we just let the camera device pick the automatic settings.
		     mPreviewBuilder.set(CaptureRequest.CONTROL_MODE
				, CameraMetadata.CONTROL_MODE_AUTO);
             HandlerThread thread = new HandlerThread("CameraPreview");
            thread.start();
            Handler backgroundHandler = new Handler(thread.getLooper());
            // Finally, we start displaying the camera preview.
            mPreviewSession.setRepeatingRequest(mPreviewBuilder.build(), null, backgroundHandler);
       }
```

#### page10:

**CameraCaptureSession:**

– A configured capture session for a CameraDevice, used for capturing images from the camera.

– A CameraCaptureSession is created by providing a set of target output surfaces to createCaptureSession. Once created, the session is active until a new session is created by the camera device, or the camera device is closed.

CaptureSession的幾個api很明顯就是來解決 zero shutter lag，自己感受感受

```
capture(CaptureRequest request, CameraCaptureSession.CaptureListener listener, Handler handler)
Submit a request for an image to be captured by the camera device.

captureBurst(List requests, CameraCaptureSession.CaptureListener listener, Handler handler)
Submit a list of requests to be captured in sequence as a burst.

setRepeatingBurst(List requests, CameraCaptureSession.CaptureListener listener, Handler handler)
Request endlessly repeating capture of a sequence of images by this capture session.

setRepeatingRequest(CaptureRequest request, CameraCaptureSession.CaptureListener listener, Handler handler)
Request endlessly repeating capture of images by this capture session.
```

不得不說那個burst根本就是burst，自己寫過幾次就知道啦，不過要真的快的話目前只有nexus 9 和 6。

#### page11:

**Take picture1:**

Step4: Capture of JPEG images or RAW buffers for Dng Creator can be done with ImageReader with the {android.graphics.ImageFormat#JPEG} and {android.graphics.ImageFormat#RAW_SENSOR} formats.

raw buffer  是camera2 才支援～～

```
CameraManager manager =
            (CameraManager) activity.getSystemService(Context.CAMERA_SERVICE);
// Pick the best JPEG size that can be captured with this CameraDevice.
CameraCharacteristics characteristics =
manager.getCameraCharacteristics(mCameraDevice.getId());
Size[] jpegSizes = null;
if (characteristics != null) {
           jpegSizes = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP) .getOutputSizes(ImageFormat.JPEG);
   }
 int width = 640; int height = 480;
 if (jpegSizes != null && 0 < jpegSizes.length) {
            width = jpegSizes[0].getWidth();
             height = jpegSizes[0].getHeight();
    }

// This is the CaptureRequest.Builder that we use to take a picture.
final CaptureRequest.Builder captureBuilder =
mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_STILL_CAPTURE);
captureBuilder.addTarget(reader.getSurface());
// In this sample, we just let the camera device pick the automatic settings. captureBuilder.set(CaptureRequest.CONTROL_MODE,CameraMetadata.CONTROL_MODE_AUTO);
```

#### page12:

**承上 takePicture2**

Step5:Use ImageReader to get output data

```
// We use an ImageReader to get a JPEG from CameraDevice.
// Here, we create a new ImageReader and prepare its Surface as an output from camera.
ImageReader reader = ImageReader.newInstance(width, height, ImageFormat.JPEG, 1);
List outputSurfaces = new ArrayList(2);
outputSurfaces.add(reader.getSurface());
outputSurfaces.add(new Surface(mTextureView.getSurfaceTexture()));

// This is the CaptureRequest.Builder that we use to take a picture.
final CaptureRequest.Builder captureBuilder =
mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_STILL_CAPTURE);
captureBuilder.addTarget(reader.getSurface());
// In this sample, we just let the camera device pick the automatic settings. captureBuilder.set(CaptureRequest.CONTROL_MODE,CameraMetadata.CONTROL_MODE_AUTO);
```

#### page13:

**ImageReader:**

The ImageReader class allows direct application access to image data rendered into a Surface

為一種暫存的 image data buffer consumer. 比較特別的是有size可以設定，api19前幾乎沒有。

#### page14:

**Take picture3**

Step6:Set output data parameters.

此階段同時設定 image reader listener，用來接收device發出的資料。

```
// In this sample, we just let the camera device pick the automatic settings.
capturebuilder.set(CaptureRequest.CONTROL_MODE, CameraMetadata.CONTROL_MODE_AUTO);
// Orientation
int rotation = activity.getWindowManager().getDefaultDisplay().getRotation();
captureBuilder.set(CaptureRequest.JPEG_ORIENTATION, ORIENTATIONS.get(rotation));
// Output file
final File file = new File(activity.getExternalFilesDir(null), "pic.jpg");
// This listener is called when a image is ready in ImageReader
ImageReader.OnImageAvailableListener readerListener =
new ImageReader.OnImageAvailableListener() {….}
```

#### page15:

**ImageReader.OnImageAvailableListener**

Callback interface for being notified that a new image is available.

```
onImageAvailable(ImageReader reader) { 
image = reader.acquireLatestImage();
ByteBuffer buffer = image.getPlanes()[0].getBuffer();
byte[] bytes = new byte[buffer.capacity()];
buffer.get(bytes);
save(bytes);
…..//Use output stream to write out to file.
……
Image.close();
```

#### page16:

**Take picture4**

Step7:Set output data parameters. And start camera capture picture thread. 

```
// In this sample, we just let the camera device pick the automatic settings.
capturebuilder.set(CaptureRequest.CONTROL_MODE, CameraMetadata.CONTROL_MODE_AUTO);
// Orientation
int rotation = activity.getWindowManager().getDefaultDisplay().getRotation();
captureBuilder.set(CaptureRequest.JPEG_ORIENTATION, ORIENTATIONS.get(rotation));
// Output file
final File file = new File(activity.getExternalFilesDir(null), "pic.jpg");
// This listener is called when a image is ready in ImageReader
 ImageReader.OnImageAvailableListener readerListener =
               new ImageReader.OnImageAvailableListener() {….}
// We create a Handler since we want to handle the result JPEG in a background thread
HandlerThread thread = new HandlerThread("CameraPicture");
thread.start();
final Handler backgroundHandler = new Handler(thread.getLooper());
reader.setOnImageAvailableListener(readerListener, backgroundHandler);
```

#### page17:

**takePicture5**

Step8:Restart preview thread after capture complete.

這裡是有順序的，再capturesession listener 的 capture complete 後重起startpreview()

先說這個startpreview()不是camera1的api ，而是之前我們自己寫的 startpreview()

以發生時間來排序是右邊先 左邊後

```
// Finally, we can start a new CameraCaptureSession to take a picture. mCameraDevice.createCaptureSession(outputSurfaces,
new CameraCaptureSession.StateListener() {
public void onConfigured(CameraCaptureSession session) {
	try {
		session.capture(captureBuilder.build(), captureListener,
                                        backgroundHandler);
	} catch (CameraAccessException e) {
		e.printStackTrace();
	}
     }
}, backgroundHandler );
```

then

```
// This listener is called when the capture is completed.
// Note that the JPEG data is not available in this listener, but in the
 // ImageReader.OnImageAvailableListener we created above.
 final CameraCaptureSession.CaptureListener captureListener =
	new CameraCaptureSession.CaptureListener() {
	public void onCaptureCompleted(CameraCaptureSession session,
	CaptureRequest request,   TotalCaptureResult result) {
          // We restart the preview when the capture is completed
                            startPreview();
			}
		};
```

以上是Camera2的takePicture簡易實作過程，最重要的還是找到一台可以跑hal3的機器

 不然很多功能都是關掉的，結果就會像是只是換個寫法寫camera1

完整程式碼

[https://github.com/googlesamples/android-Camera2Basic](https://github.com/googlesamples/android-Camera2Basic)

google i/o 介紹

[https://www.google.com/events/io](https://www.google.com/events/io)

以下是一些可能看投影片或是開發時想不透的部份做成QA

#### **Q1:synchronize?**

A1:新的capture request 同時包含parameter的setting和capture 的行為，並執行於camera device端，過去old camera 則是將這兩個行為分開並由ap端去control。

![syn](../../assets/images/syn.jpg)

[https://boochlin.com/wp-content/uploads/2015/03/syn.jpg](https://boochlin.com/wp-content/uploads/2015/03/syn.jpg)

#### 

![system](../../assets/images/system.jpg)

[https://boochlin.com/wp-content/uploads/2015/03/system.jpg](https://boochlin.com/wp-content/uploads/2015/03/system.jpg)

#### **Q2:為何為何 captureSession 時，會放入outputSurface(Surface[]) 多個surface 而並非指定單一surface?**

A2:再創建CameraCaptureSession 通常會放入未來可能會用到的target surfaces ，並不會只放入單一的target surface ，而單一指定surface 的需求寫入capture request裡

#### **Q3:ImageReader 使用方法？**

A3:Use ImageReader.getSurface() to get a Surface that can be used to produce Images for this ImageReader.

#### **Q3-1:what is surface?**

A3-1:

1.Handle onto a raw buffer that is being managed by the screen compositor.

2. surface 的canvas成員可以用來提供繪圖的場所，raw buffer 用來保存data ，取得surface 等同取得canvas 和 raw buffer的內容。

summary: 於camera ap 裡則是等同提供capture輸出data的的目標surface，接者imagereader會從surface 去封裝data變成image class。

下一章節會來探討 Camera2 framework.
