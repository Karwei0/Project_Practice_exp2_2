# 1 实验内容
构建CameraX应用
# 2 实验记录
## 2.1 创建项目
### 2.1.1 新建空项目
在Android Studio中创建一个新项目，**选择“Empty views Activity”（AS2023版）**,为项目命名，选择Kotlin语言开发，设定最低支持API Level 21（**CameraX所需的最低级别**）<br />![](./screenshot/emptyactivity.jpg#id=e2Vwc&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![0YA0CP`8W6~B[4]YF2FPK~V.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715413324614-8a69b4ea-a698-49f6-aac8-d01dcf0a7a6d.jpeg#averageHue=%23e8f0f2&clientId=udcc7eb26-6b4e-4&from=paste&height=608&id=ud811a78f&originHeight=836&originWidth=1139&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=54055&status=done&style=none&taskId=uad35df16-25ad-438b-bf88-5de64386937&title=&width=828.3636363636364)
### 2.1.2 添加Gradle依赖
打开项目的模块的build.Gradle(module)文件，添加CameraX依赖项：
```xml
val camerax_version = "1.1.0-beta01"
implementation("androidx.camera:camera-core:${camerax_version}")
implementation("androidx.camera:camera-camera2:${camerax_version}")
implementation("androidx.camera:camera-lifecycle:${camerax_version}")
implementation("androidx.camera:camera-video:${camerax_version}")

implementation("androidx.camera:camera-view:${camerax_version}")
implementation("androidx.camera:camera-extensions:${camerax_version}")
```
由于`kotlin("android.extensions")`已经弃用，在此项目中采用ViewBinding，为每个xml布局生成一个绑定类，替代findViewById。故需要在android{}代码块末尾添加如下代码：
```java
android {
...
    buildFeatures {
        viewBinding true
    }
}
```
并且需要指定对应的JDK。
```
android{
....
  compileOptions {
      sourceCompatibility = JavaVersion.VERSION_1_8
      targetCompatibility = JavaVersion.VERSION_1_8
  }
}
```
点击Sync Now，重新构建Gradle<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/38674938/1715413611523-b2bdd530-e426-4f97-8f42-2b745fe7f3ac.png#averageHue=%23f3f5ea&clientId=udcc7eb26-6b4e-4&from=paste&height=71&id=ue7ab740b&originHeight=97&originWidth=593&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=13008&status=done&style=none&taskId=u8f95df5b-afcc-4638-bc91-c274ee5e777&title=&width=431.27272727272725)
### 2.1.3 创建项目布局
在activity_main布局文件中添加如下布局代码：
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.camera.view.PreviewView
        android:id="@+id/viewFinder"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <Button
        android:id="@+id/image_capture_button"
        android:layout_width="110dp"
        android:layout_height="110dp"
        android:layout_marginBottom="50dp"
        android:layout_marginEnd="50dp"
        android:elevation="2dp"
        android:text="@string/take_photo"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintEnd_toStartOf="@id/vertical_centerline" />

    <Button
        android:id="@+id/video_capture_button"
        android:layout_width="110dp"
        android:layout_height="110dp"
        android:layout_marginBottom="50dp"
        android:layout_marginStart="50dp"
        android:elevation="2dp"
        android:text="@string/start_capture"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toEndOf="@id/vertical_centerline" />

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/vertical_centerline"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintGuide_percent=".50" />

</androidx.constraintlayout.widget.ConstraintLayout>
```
previewView和其他组件的**添加顺序不能颠倒**，否则按钮会被盖住。<br />在string.xml中添加如下资源代码：
```xml
<string name="app_name">CameraXApp</string>
<string name="take_photo">Take Photo</string>
<string name="start_capture">Start Capture</string>
<string name="stop_capture">Stop Capture</string>
```
布局效果如下图所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/38674938/1715413685195-cbcd33a6-2895-4573-a04d-b32cb235be43.png#averageHue=%234f6771&clientId=udcc7eb26-6b4e-4&from=paste&height=433&id=u76579077&originHeight=596&originWidth=721&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=20407&status=done&style=none&taskId=ub0d3037b-3477-4474-ae12-c62459b20e1&title=&width=524.3636363636364)
### 2.1.4 请求必要权限
打开相机和录制音频都需要对应的权限，所以需要打开AndroidManifest.xml文件，将下列权限请求代码添加到application标记之前
```xml
<uses-feature android:name="android.hardware.camera.any" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
    android:maxSdkVersion="28" />
```
其中android.hardware.camera.any表示设备可以使用前置摄像头或者后置摄像头，避免了设备没有后置摄像头的情况下，相机无法正常运行的情况。
### 2.1.5 编写MainActivity.kt代码
在MainActivity.kt中添加以下代码，其中包含将要实现的函数，并在onCreate()中实现了检查相机权限、启动相机和按钮的onClickListener()以及实现cameraExecutor。
```java
class MainActivity : AppCompatActivity() {
   private lateinit var viewBinding: ActivityMainBinding

   private var imageCapture: ImageCapture? = null

   private var videoCapture: VideoCapture<Recorder>? = null
   private var recording: Recording? = null

   private lateinit var cameraExecutor: ExecutorService

   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       viewBinding = ActivityMainBinding.inflate(layoutInflater)
       setContentView(viewBinding.root)

       // Request camera permissions
       if (allPermissionsGranted()) {
           startCamera()
       } else {
           ActivityCompat.requestPermissions(
               this, REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS)
       }

       // Set up the listeners for take photo and video capture buttons
       viewBinding.imageCaptureButton.setOnClickListener { takePhoto() }
       viewBinding.videoCaptureButton.setOnClickListener { captureVideo() }

       cameraExecutor = Executors.newSingleThreadExecutor()
   }

   private fun takePhoto() {}

   private fun captureVideo() {}

   private fun startCamera() {}

   private fun allPermissionsGranted() = REQUIRED_PERMISSIONS.all {
       ContextCompat.checkSelfPermission(
           baseContext, it) == PackageManager.PERMISSION_GRANTED
   }

   override fun onDestroy() {
       super.onDestroy()
       cameraExecutor.shutdown()
   }

   companion object {
       private const val TAG = "CameraXApp"
       private const val FILENAME_FORMAT = "yyyy-MM-dd-HH-mm-ss-SSS"
       private const val REQUEST_CODE_PERMISSIONS = 10
       private val REQUIRED_PERMISSIONS =
           mutableListOf (
               Manifest.permission.CAMERA,
               Manifest.permission.RECORD_AUDIO
           ).apply {
               if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.P) {
                   add(Manifest.permission.WRITE_EXTERNAL_STORAGE)
               }
           }.toTypedArray()
   }
}
```
运行应用，可以发现程序请求使用摄像头和麦克风：<br />![](./screenshot/effect1.jpg#id=J3jIA&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![9ed6b338b9443efad94f48eef553d910.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414822681-9b8da74d-1776-4ffe-bfb8-ec193af399f9.jpeg#averageHue=%238d7f72&clientId=udcc7eb26-6b4e-4&from=paste&height=767&id=ub5d0762f&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=118944&status=done&style=none&taskId=u5f6a0a96-3d06-4be8-902c-9c72a1820a3&title=&width=354)![c80e7010afdddb910db1013da8864279.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414764791-4737d1a6-22af-4cde-a820-505b3003945a.jpeg#averageHue=%238d7f72&clientId=udcc7eb26-6b4e-4&from=paste&height=765&id=u8fb13fe4&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=119770&status=done&style=none&taskId=u202ad41a-c260-4534-a17a-195bf85b985&title=&width=353)
## 2.2 实现preview用例
在上一步中已经实现了相机的调用，但此时屏幕中还看不到任何画面，为了能够将相机当前所捕捉的画面显示在应用中，需要使用CameraX Preview类实现取景器功能。<br />在startCamera()函数中填充以下代码：
```java
private fun startCamera() {
   val cameraProviderFuture = ProcessCameraProvider.getInstance(this)

   cameraProviderFuture.addListener({
       // Used to bind the lifecycle of cameras to the lifecycle owner
       val cameraProvider: ProcessCameraProvider = cameraProviderFuture.get()

       // Preview
       val preview = Preview.Builder()
          .build()
          .also {
              it.setSurfaceProvider(viewBinding.viewFinder.surfaceProvider)
          }

       // Select back camera as a default
       val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA

       try {
           // Unbind use cases before rebinding
           cameraProvider.unbindAll()

           // Bind use cases to camera
           cameraProvider.bindToLifecycle(
               this, cameraSelector, preview)

       } catch(exc: Exception) {
           Log.e(TAG, "Use case binding failed", exc)
       }

   }, ContextCompat.getMainExecutor(this))
}
```
再次运行应用，可以看到相机预览:<br />![98920d0f53bfcac956e6270354c8f193.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414670513-b5cd4da2-9314-4100-a0cc-995c37570b99.jpeg#averageHue=%233c4d3d&clientId=udcc7eb26-6b4e-4&from=paste&height=600&id=ue217adbe&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=167502&status=done&style=none&taskId=u7210b9bd-21b6-4b7b-80fb-0ead6a09a27&title=&width=277)
## 2.3 实现ImageCapture用例（拍照功能）
通过实例化ImageCapture类来实现拍照功能，按下photo按钮时将会调用takephoto()，在takephoto()方法中填充如下代码：
```java
private fun takePhoto() {
   // Get a stable reference of the modifiable image capture use case
   val imageCapture = imageCapture ?: return

   // Create time stamped name and MediaStore entry.
   val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US)
              .format(System.currentTimeMillis())
   val contentValues = ContentValues().apply {
       put(MediaStore.MediaColumns.DISPLAY_NAME, name)
       put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg")
       if(Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
           put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/CameraX-Image")
       }
   }

   // Create output options object which contains file + metadata
   val outputOptions = ImageCapture.OutputFileOptions
           .Builder(contentResolver,
                    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                    contentValues)
           .build()

   // Set up image capture listener, which is triggered after photo has
   // been taken
   imageCapture.takePicture(
       outputOptions,
       ContextCompat.getMainExecutor(this),
       object : ImageCapture.OnImageSavedCallback {
           override fun onError(exc: ImageCaptureException) {
               Log.e(TAG, "Photo capture failed: ${exc.message}", exc)
           }

           override fun
               onImageSaved(output: ImageCapture.OutputFileResults){
               val msg = "Photo capture succeeded: ${output.savedUri}"
               Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
               Log.d(TAG, msg)
           }
       }
   )
}
```
在startCamera()中添加ImageCapture的实例，并在try代码块中更新bindToLifecycle()以包含新的用例
```java
    imageCapture = ImageCapture.Builder()
           .build()

    try {
        // Unbind use cases before rebinding
        cameraProvider.unbindAll()

        // Bind use cases to camera
        cameraProvider.bindToLifecycle(
            this, cameraSelector, preview, imageCapture)

    } catch(exc: Exception) {
        Log.e(TAG, "Use case binding failed", exc)
    }
```
重新运行应用，点击TakePhoto按钮，将会弹出一个Toast消息框：<br />![57fe0088634d9fa8e1ed35d9badf786a.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414162708-01ab8d62-f69a-4c9d-9446-7dcb5054156c.jpeg#averageHue=%23a0b398&clientId=udcc7eb26-6b4e-4&from=paste&height=1129&id=u350ee309&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=203155&status=done&style=none&taskId=u33a2d887-fbcb-40bc-824d-662ad582469&title=&width=521)<br />![](./screenshot/takephoto1.jpg#id=HIVPw&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />打开相册可以看到刚刚拍摄的照片：<br />![](./screenshot/takephoto2.jpg#id=kz65T&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![2c6d6bb85e7252bc6dd15c7329d65eb3.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715413993963-f30215b3-0de2-4a3b-8004-f0be19bfaa49.jpeg#averageHue=%239b8a7a&clientId=udcc7eb26-6b4e-4&from=paste&height=1168&id=u80340d73&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=204164&status=done&style=none&taskId=u5edbf7ad-688a-47d7-b785-69282866624&title=&width=539)
## 2.4 实现ImageAnalysis用例
在MainActivity中定义一个类实现ImageAnalysis.Analtyzer接口，使用传入的相机帧调用该类，重写接口中的analyze函数：
```java
private class LuminosityAnalyzer(private val listener: LumaListener) : ImageAnalysis.Analyzer {

   private fun ByteBuffer.toByteArray(): ByteArray {
       rewind()    // Rewind the buffer to zero
       val data = ByteArray(remaining())
       get(data)   // Copy the buffer into a byte array
       return data // Return the byte array
   }

   override fun analyze(image: ImageProxy) {

       val buffer = image.planes[0].buffer
       val data = buffer.toByteArray()
       val pixels = data.map { it.toInt() and 0xFF }
       val luma = pixels.average()

       listener(luma)

       image.close()
   }
}
```
在startCamera()中添加如下代码，并在try代码块中更新bindToLifecycle()以包含新的用例：
```java
val imageAnalyzer = ImageAnalysis.Builder()
   .build()
   .also {
       it.setAnalyzer(cameraExecutor, LuminosityAnalyzer { luma ->
           Log.d(TAG, "Average luminosity: $luma")
       })
   }

try {
    // Unbind use cases before rebinding
    cameraProvider.unbindAll()

    // Bind use cases to camera
    cameraProvider.bindToLifecycle(
        this, cameraSelector, preview, imageCapture, imageAnalyzer)

} catch(exc: Exception) {
    Log.e(TAG, "Use case binding failed", exc)
}
```
重新运行应用，可以在logcat中看到不断生成的分析信息：<br />![](./screenshot/analysis.jpg#id=DRf0k&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)![image.png](https://cdn.nlark.com/yuque/0/2024/png/38674938/1715416610116-78a2f350-8659-4184-a212-40df5198298d.png#averageHue=%23faf9f8&clientId=u58004441-cb55-4&from=paste&height=299&id=u4e6768c7&originHeight=411&originWidth=1715&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=158489&status=done&style=none&taskId=u134aa7db-239e-4b7b-828e-908c9e124ef&title=&width=1247.2727272727273)
## 2.5 实现VideoCapture用例（拍摄视频功能）
通过实例化VideoCapture类，实现视频拍摄功能，在captureVideo()中填充如下代码：
```java
// Implements VideoCapture use case, including start and stop capturing.
private fun captureVideo() {
   val videoCapture = this.videoCapture ?: return

   viewBinding.videoCaptureButton.isEnabled = false

   val curRecording = recording
   if (curRecording != null) {
       // Stop the current recording session.
       curRecording.stop()
       recording = null
       return
   }

   // create and start a new recording session
   val name = SimpleDateFormat(FILENAME_FORMAT, Locale.US)
              .format(System.currentTimeMillis())
   val contentValues = ContentValues().apply {
       put(MediaStore.MediaColumns.DISPLAY_NAME, name)
       put(MediaStore.MediaColumns.MIME_TYPE, "video/mp4")
       if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
           put(MediaStore.Video.Media.RELATIVE_PATH, "Movies/CameraX-Video")
       }
   }

   val mediaStoreOutputOptions = MediaStoreOutputOptions
       .Builder(contentResolver, MediaStore.Video.Media.EXTERNAL_CONTENT_URI)
       .setContentValues(contentValues)
       .build()
   recording = videoCapture.output
       .prepareRecording(this, mediaStoreOutputOptions)
       .apply {
           if (PermissionChecker.checkSelfPermission(this@MainActivity,
                   Manifest.permission.RECORD_AUDIO) ==
               PermissionChecker.PERMISSION_GRANTED)
           {
               withAudioEnabled()
           }
       }
       .start(ContextCompat.getMainExecutor(this)) { recordEvent ->
           when(recordEvent) {
               is VideoRecordEvent.Start -> {
                   viewBinding.videoCaptureButton.apply {
                       text = getString(R.string.stop_capture)
                       isEnabled = true
                   }
               }
               is VideoRecordEvent.Finalize -> {
                   if (!recordEvent.hasError()) {
                       val msg = "Video capture succeeded: " +
                           "${recordEvent.outputResults.outputUri}"
                       Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT)
                            .show()
                       Log.d(TAG, msg)
                   } else {
                       recording?.close()
                       recording = null
                       Log.e(TAG, "Video capture ends with error: " +
                           "${recordEvent.error}")
                   }
                   viewBinding.videoCaptureButton.apply {
                       text = getString(R.string.start_capture)
                       isEnabled = true
                   }
               }
           }
       }
}
```
在startCamera()中添加如下代码：
```java
val recorder = Recorder.Builder()
   .setQualitySelector(QualitySelector.from(Quality.HIGHEST))
   .build()
videoCapture = VideoCapture.withOutput(recorder)
```
在try代码块中更新bindToLifecycle()，删除analysis用例，包含videoCapture用例：
```java
try {
    // Unbind use cases before rebinding
    cameraProvider.unbindAll()

    // Bind use cases to camera
    cameraProvider.bindToLifecycle(
        this, cameraSelector, preview, imageCapture, videoCapture)

} catch(exc: Exception) {
    Log.e(TAG, "Use case binding failed", exc)
}
```
重新运行应用，点击StartCapture按钮，视频开始录制，可以看到按钮的文字变成StopCapture，再次点击按钮，停止视频录制，将会弹出Toast消息框提示信息：<br />![0fe0f907e1c9690c576124f7783bc68e.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414245442-7f58b706-7c3c-46e6-a354-4223a5bbfe32.jpeg#averageHue=%23a8bba1&clientId=udcc7eb26-6b4e-4&from=paste&height=834&id=uf78bcd6a&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=168183&status=done&style=none&taskId=udc714dfe-9d2e-49e3-8370-74cb86130aa&title=&width=385)<br />![](./screenshot/videocapture.jpg#id=ELWRs&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />由于拍照和视频拍摄功能运行在两个不同的线程上，所以在视频拍摄过程中也可以点击TakePhoto按钮来拍摄照片，运行截图中可以看到当按钮文字为StopCapture时，成功拍摄了一张照片：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/38674938/1715414299201-388381b1-fdfc-4792-831a-939f1284899f.png#averageHue=%23a6b79c&clientId=udcc7eb26-6b4e-4&from=paste&height=962&id=u61b99e0f&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=2402214&status=done&style=none&taskId=ua469b039-99f2-4d9d-abd4-803abfb4114&title=&width=444)<br />![](./screenshot/videophoto.jpg#id=diyt8&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />打开相册，可以看到刚刚拍摄的视频和照片：<br />![2c6d6bb85e7252bc6dd15c7329d65eb3.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715413993963-f30215b3-0de2-4a3b-8004-f0be19bfaa49.jpeg#averageHue=%239b8a7a&clientId=udcc7eb26-6b4e-4&from=paste&height=1168&id=KELoc&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=204164&status=done&style=none&taskId=u5edbf7ad-688a-47d7-b785-69282866624&title=&width=539)
# 3 实验结果
最后，对整个的实验结果进行总结。<br />首先，当程序启动时，应用会申请手机的相机和麦克风的权限。<br />![9ed6b338b9443efad94f48eef553d910.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414822681-9b8da74d-1776-4ffe-bfb8-ec193af399f9.jpeg#averageHue=%238d7f72&clientId=udcc7eb26-6b4e-4&from=paste&height=767&id=sNtWN&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=118944&status=done&style=none&taskId=u5f6a0a96-3d06-4be8-902c-9c72a1820a3&title=&width=354)![c80e7010afdddb910db1013da8864279.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414764791-4737d1a6-22af-4cde-a820-505b3003945a.jpeg#averageHue=%238d7f72&clientId=udcc7eb26-6b4e-4&from=paste&height=765&id=deQqz&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=119770&status=done&style=none&taskId=u202ad41a-c260-4534-a17a-195bf85b985&title=&width=353)<br />然后，进入到程序，此时是对预览（preview）状态进行测试。<br />![98920d0f53bfcac956e6270354c8f193.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414670513-b5cd4da2-9314-4100-a0cc-995c37570b99.jpeg#averageHue=%233c4d3d&clientId=udcc7eb26-6b4e-4&from=paste&height=600&id=gYCsn&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=167502&status=done&style=none&taskId=u7210b9bd-21b6-4b7b-80fb-0ead6a09a27&title=&width=277)<br />接着，可以点击 take photo（imageCapture），对画面进行拍照，并且有 toast 提示。<br />![57fe0088634d9fa8e1ed35d9badf786a.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414162708-01ab8d62-f69a-4c9d-9446-7dcb5054156c.jpeg#averageHue=%23a0b398&clientId=udcc7eb26-6b4e-4&from=paste&height=1129&id=d4gl4&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=203155&status=done&style=none&taskId=u33a2d887-fbcb-40bc-824d-662ad582469&title=&width=521)<br />与此同时，可以在Android Studio的 logcat 上，检查 imageAnalysis。<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/38674938/1715416598162-fc246105-ce4d-4dae-89cd-507eeb39ae59.png#averageHue=%23faf9f8&clientId=u58004441-cb55-4&from=paste&height=299&id=u327518ee&originHeight=411&originWidth=1715&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=158489&status=done&style=none&taskId=u180644dc-2b4a-4e05-89c4-cc5cc0acb7a&title=&width=1247.2727272727273)<br />最后，是拍摄视频。<br />在启动视频的时候，video capture的文本会转变为 stop capture。<br />![9ccc5afc6038f05ba7fe159459af1e34.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715415843130-d1ad4baf-0755-464c-9d6c-a94980ff7d2c.jpeg#averageHue=%23c2d2bb&clientId=udcc7eb26-6b4e-4&from=paste&height=940&id=u320e2860&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=202960&status=done&style=none&taskId=u5447f3b7-0638-4ac8-bb96-f0972a63253&title=&width=434)<br />在此期间，仍然可以进行拍照。<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/38674938/1715414299201-388381b1-fdfc-4792-831a-939f1284899f.png#averageHue=%23a6b79c&clientId=udcc7eb26-6b4e-4&from=paste&height=962&id=T2W84&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=2402214&status=done&style=none&taskId=ua469b039-99f2-4d9d-abd4-803abfb4114&title=&width=444)<br />最后点击 stop capture，有 toast 提示。<br />![0fe0f907e1c9690c576124f7783bc68e.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715414245442-7f58b706-7c3c-46e6-a354-4223a5bbfe32.jpeg#averageHue=%23a8bba1&clientId=udcc7eb26-6b4e-4&from=paste&height=834&id=PPxpY&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=168183&status=done&style=none&taskId=udc714dfe-9d2e-49e3-8370-74cb86130aa&title=&width=385)<br />对于过程中的视频以及照片，可以在相册中看到。<br />![2c6d6bb85e7252bc6dd15c7329d65eb3.jpg](https://cdn.nlark.com/yuque/0/2024/jpeg/38674938/1715413993963-f30215b3-0de2-4a3b-8004-f0be19bfaa49.jpeg#averageHue=%239b8a7a&clientId=udcc7eb26-6b4e-4&from=paste&height=1168&id=wHLsE&originHeight=1920&originWidth=886&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=204164&status=done&style=none&taskId=u5edbf7ad-688a-47d7-b785-69282866624&title=&width=539)
