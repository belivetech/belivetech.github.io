---
title: SDK Guide for Android
---

The BeLive Android SDK provides the interfaces required to broadcast and playback on Android along with interactive features.&#x20;

The following operations are supported:&#x20;

• Set up (initialize) SDK&#x20;

• Manage live broadcasting.&#x20;

• Watch live stream&#x20;

• Watch Recorded stream

• Manage Live Shopping

Latest version of Android SDK: 1.0.5 (Release Notes)

## Getting Started

### Requirements

* Android Studio 4 and above
* Java 8 and above

### Install the SDK

To add the BeLive Android SDK library to your Android development environment, add the `aar` files to `lib` folder of your module. After that add them in your module's `build.gradle` file, as shown here:

Project level `build.gradle`

```
repositories {
        ....
        flatDir {
            dirs 'libs'
        }
        
    }
```

app or module level `build.gradle` file

**Release aar files**

```groovy
    releaseImplementation (name:'belive-host-1.0.5.107-release', ext:'aar')
    releaseImplementation (name:'belive-viewer-1.0.5.107-release', ext:'aar')
    releaseImplementation (name:'belive-core-1.0.5.107-release', ext:'aar')
    releaseImplementation (name:'belive-common-1.0.5.106-release', ext:'aar')
    releaseImplementation (name:'belive-shopping-1.0.0.102-release', ext:'aar')
    releaseImplementation (name:'belive-ui-1.0.0.101-release', ext:'aar')
```

BeLive Android SDK uses open source libraries. Add following dependencies to app level `build.gradle` For detailed explanation about Third party libraries used in SDK, visit [Third Party Libraries](https://github.com/belivetech/Documentation/blob/master/android/Third-Party-Libs.md)

```
    // coroutines : Use for Async tasks
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.6"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.6"

    //dependency injection koin
    api 'org.koin:koin-androidx-scope:2.1.5'
    api 'org.koin:koin-androidx-viewmodel:2.1.5'
    
    // lifecycle : For lifecycle aware components
    implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.0"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.4.0"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.4.0"

    // retrofit : Type safe HTTP client for Android
    implementation "com.squareup.retrofit2:retrofit:2.9.0"
    implementation "com.squareup.retrofit2:converter-gson:2.9.0"
    implementation "com.squareup.okhttp3:logging-interceptor:4.9.0"
    implementation 'com.facebook.stetho:stetho-okhttp3:1.5.1'
    
    // Real-time chat
    implementation 'com.github.agorabuilder:rtm-sdk:1.4.7'

    // (Exclude below libraries if you are not using belive-host)
    // Streamer : Libraries for pushing RTMP to streaming server for transcoding
    implementation 'com.ksyun.media:libksylive-armv7a:3.0.4'
    implementation "com.ksyun.media:libksylive-arm64:3.0.2"
    implementation "com.ksyun.media:libksylive-x86:3.0.2"

    // exoplayer : Required for playback of streams
    implementation 'com.google.android.exoplayer:exoplayer:2.9.6'
    implementation 'com.google.android.exoplayer:exoplayer-dash:2.9.6'
    implementation 'com.google.android.exoplayer:extension-rtmp:2.9.6'
    implementation 'com.google.android.exoplayer:exoplayer-ui:2.9.6'
    implementation 'com.github.rubensousa:previewseekbar:2.0.0'
    implementation 'com.github.rubensousa:previewseekbar-exoplayer:2.8.1.0'

    // Image Loading
    implementation 'com.github.bumptech.glide:glide:4.11.0'
    kapt 'com.github.bumptech.glide:compiler:4.11.0'
    
    // misc : Utility libraries for UI
    implementation 'com.google.code.gson:gson:2.8.6'
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
    implementation 'com.hannesdorfmann:adapterdelegates4-kotlin-dsl:4.2.0'
  
```

> You can skip libraries which are already added in your `build.gradle`. In case of any conflict, let us know and our team will check for compatibility with SDK.

**Additional settings**\
\
SDK requires additional configuration in module's `build.gradle`

Obtain license key from business team by providing your package Id. Place license key in `app/src/main/assets` folder of your app module and name it `belive.license` Add following variables to app level `build.gradle`

> For sample app we have included `belive.license`file but for integration in your app you must request from us&#x20;

```groovy
android {
        def beliveSdkApiDomain = "api_domain"
        def beliveSdkId = "beliveId"
         manifestPlaceholders = [
            beliveSdkApiDomain: beliveSdkApiDomain,
            beliveSdkId       : beliveSdkId
         ]
}
```

> Check sample app for `beliveSdkApiDomain`

Add following lines in `AndrodiManifes.xml` before closing `<application>` tag.

```xml
 <!-- belive sdk configuration here-->
        <meta-data
            android:name="sg.belive.sdk.beliveSdkApiDomain"
            android:value="${beliveSdkApiDomain}" />

        <meta-data
            android:name="sg.belive.sdk.beliveSdkId"
            android:value="${beliveSdkId}" />

<!-- end belive sdk configuration here-->
```

BeLive SDK for Android has plenty of features. By default all features are enabled. You can disable them by using a `json` file. Create a new file `belive-configuration.json` in folder `app/src/main/assets`.

```json
{
  "features": {
    "watcherList": true,
    "beautyFilter": true,
    "shareButton": true,
    "streamQuality": 1,
    "allowPictureInPicture": true,
    "shoppingCart": true,
    "obs": false
  }
}
```

#### Grant system permissions

BeLive SDK requires system permissions. To grant system permissions such as camera, microphone and others, We have added the following permissions in our `aar` library. No action is required on your side.

> Note : Host requires additional permissions for live stream.

```
    <!-- Host and Viewer -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

   <!-- Host only -->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.FLASHLIGHT" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
```

You are all set to start your first live stream!&#x20;

> For more information about permissions, see [official guide](https://developer.android.com/training/permissions/requesting.html)

## **Start your first live stream**

### Step 1 : Initialize SDK

Firstly we need to initialize provider and ImageLoader. The main reason for using external ImageLoader is to use consistent settings of Glide (third party image loading library).

```kotlin
val provider = BeliveSdkInitProvider(this.applicationContext)
provider.initBeLiveSdk(object : BeliveSdkInitProvider.ISDKInitStatus {
            override fun onStatus(status: String, error: BeliveException?) {}
        })
// Uncomment below line with your ImageLoader Implementation
// LbsImageLoaderUtil.registerImageLoader(ImageLoaderImpl())
```

Next step is to initialize BeLive SDK for Android. There is slight difference between Host login and Viewer Login.  Note that host login accounts are pre-generated while viewer accounts can be created using SDK API\
\
**Host / Broadcaster**&#x20;

```kotlin
BeliveSdkInitialization.Builder(this)
    .setUsername(username)
    .setUserPassword(pass)
    .setInitCallback(object : BeliveSdkInitialization.BeliveSdkInitCallback {
        override fun onFinished(
            status: BeliveInitStatus.Status, 
            message: String?) {
            if (isFinishing || isDestroyed) return
            // Make UI Changes
            when (status) {
                BeliveInitStatus.Status.INITIALIZATION_LOGIN_FAILED_USER_NOT_FOUND,
                BeliveInitStatus.Status.INITIALIZATION_LOGIN_FAILED_INVALID_PASSWORD 
                -> {
                    // Invalid username or password
                    }
                BeliveInitStatus.Status.INITIALIZATION_LOGIN_FAILED -> {
                            // Show error message
                }
                BeliveInitStatus.Status.INITIALIZATION_SUCCESS -> {
                            // openStreamActivity()
                }
            }
        }
    }
).build().init()
```

Username and password for host / broadcaster will be provided (Contact business team). Only pre-registered hosts can start live. \
\
**Viewer (SSO Login with unique userId and displayName)**

```kotlin
BeliveSdkInitialization.Builder(this)
    .setUserId(uniqueUserId)
    .setUserDisplayName(displayName)
    .setInitCallback(object : BeliveSdkInitialization.BeliveSdkInitCallback {
        override fun onFinished(status: BeliveInitStatus.Status, message: String?) {
            // Make UI changes
            when (status) {
                BeliveInitStatus.Status.INITIALIZATION_SUCCESS -> {
                    // Belive SDK is ready!

                }
                BeliveInitStatus.Status.INITIALIZATION_LOGIN_FAILED -> {
                    // failed to login
                }
                BeliveInitStatus.Status.INITIALIZATION_REGISTER_FAILED -> {
                    // failed to register
                }
                BeliveInitStatus.Status.INITIALIZATION_LOAD_CONFIG_FAILED -> {
                    // load config failed
                }
            else -> { }
            }
        }
}).build().init()
```



### Step 2 : Register Stream callbacks and set broadcast config

HostActivity should implement `BlsLiveStreamCallback` and `BlsChatCallback` callbacks. BeLive SDK provides helper class `BlsLiveStreamManager` for managing callbacks and broadcast configuration. Refer to code below:

```kotlin
class HostActivity : FullScreenActivity(R.layout.activity_host),
    BlsLiveStreamCallback,
    BlsChatCallback {

    private lateinit var blsLiveStreamManager: BlsLiveStreamManager
  
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        findViews()
        blsLiveStreamManager = 
        BlsLiveStreamManager.getInstance(this, cameraPreview).apply {
            setLiveStreamCallback(this@HostActivity)
            setChatCallback(this@HostActivity)
            setStreamFPS(STREAM_FPS)
            setStreamQuality(STREAM_QUALITY)
            setKeyFrameInterval(STREAM_KEYFRAME_INTERVAL)
        }

    }
    
}
```

**Broadcast configuration**&#x20;

| API                     | Description                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| <p>setStreamFPS<br></p> | Set Frame Per seconds (FPS). We suggest 20 for streamQuality `RESOLUTION_SD` and 24 for `RESOLUTION_HD` |
| setStreamQuality        | Two available options for Stream Quality are `RESOLUTION_SD` and `RESOLUTION_HD`                        |
| setKeyFrameInterval     | For lower end-to-end latency, set keyFrameInterval to 1F.                                               |

Summary&#x20;

* Live Stream Manager class : `BlsLiveStreamManager`
* Live Stream Callbacks : `BlsLiveStreamCallback`
* Live Chat callbacks : `BlsChatCallback`

### Step 3 : Request Permissions&#x20;

Your app must request permission to access the user’s camera and mic. (This is not specific to BeLive SDK; it is required for any application that needs access to cameras and microphones.)

Below, we check whether the user has already granted permissions and, if not, ask for them:

```kotlin
    private fun startCameraPreviewWithPermCheck() {
        if (isFinishing || isDestroyed) {
            return
        }
        val cameraPerm: Int =
            ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
        val audioPerm: Int =
            ActivityCompat.checkSelfPermission(this, 
                                    Manifest.permission.RECORD_AUDIO)
        if (cameraPerm != PackageManager.PERMISSION_GRANTED ||
            audioPerm != PackageManager.PERMISSION_GRANTED
        ) {
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
                // No CAMERA or AudioRecord or Storage permission, please check
                )
                // No CAMERA or AudioRecord or Storage permission, please check
            } else {
                val permissions = arrayOf(
                    Manifest.permission.CAMERA,
                    Manifest.permission.RECORD_AUDIO
                )
                ActivityCompat.requestPermissions(
                    this,
                    permissions,
                    PERMISSION_REQUEST_CAMERA_AUDIO_REC
                )
            }
        } else {
            blsLiveStreamManager.startCameraPreview()
        }
    }
```

### Step 4 : Start Live Broadcast Session

SDK requires stream title as mandatory field to start stream. \
\
API&#x20;

```kotlin
fun startStream(title: String, 
    password: String? = null, 
    isLandscape: Boolean = false, 
    isObs: Boolean) 
```

How to use

```
private fun goLive(title: String) {

        blsLiveStreamManager.startStream(title, pass, landscapeMode, isObs)
    }
```

At this moment, SDK supports only `portrait` stream so landscapeMode will be always 0. BeLive SDK for Android supports streaming with professional broadcasting Software such as [OBS](https://obsproject.com). Set isObs parameter to `true` if you wish to get stream key and stream from broadcasting software.&#x20;

Following callbacks will be triggered if start stream is successful

```kotlin
  override fun onStreamCreated(slug: String) {
        // Stream created
    }
   
   override fun onStartStreamSuccess(slug: String) {
       // We suggest to keep screen ON for continuous streaming
        window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
    }
```

In case of error, SDK will throw error callback&#x20;

```kotlin
override fun onStartStreamError(code: Int, message: String) {
        // Error message and code
    }
```



At the same time, host is connected to live chat channel. Refer to below `BlsChatCallback` callback.

if success, the \[code] will be \[sg.belive.lib.core.chat.ChatManager.ERR\_CODE\_OK], otherwise it will be \[sg.belive.lib.core.chat.ChatManager].ERR\_CODE\_JOIN\_CHANNEL\_XYZ with XYZ is the error type

```kotlin
    override fun onJoinChannelResult(code: Int) {
        Log.i(TAG, "onJoinChannelResult $code")
    }
```

> **Important**: All calls to the BeLive SDK for Android must be made on the thread on which the SDK is instantiated. A call from a different thread will cause the SDK to not work as expected

### Step 5 : Stop Live Broadcast Session

API&#x20;

```kotlin
fun closeLiveStream()
```

Usage&#x20;

```kotlin
 private fun onCloseStreamButtonClicked() {
        blsLiveStreamManager.closeLiveStream()
    }
```



You will get one of the following `BlsLiveStreamCallback` callbacks

```kotlin
 override fun onEndStreamSuccess(endReason: Int, 
                                 slug: String, 
                                 statistics: BlsLiveStreamStatistics) {
 // show end stream UI 
 }
override fun onEndStreamError(endReason: Int, 
                                code: Int, 
                                message: String)
```

### Step 6 : Save Stream

Here you can show save stream screen for giving option to users to save live stream. Use following method for saving stream.

API

```
fun saveStream()
```

Usage

```kotlin
btnSaveStream.setOnClickListener { blsLiveStreamManager.saveStream() }
```

You will get one of the following `BlsLiveStreamCallback` callbacks

```
override fun onSaveStreamSuccess() {}
override fun onSaveStreamError(code: Int, message: String) {}
```

You can release broadcast session by using below code:

```kotlin
blsLiveStreamManager.release()
```

BeLive SDK provides many interactive feature. In next few steps, we are going to list down additional interactions that are possible with SDK APIs&#x20;

### Step 7 : Send and Receive Chat message

BeLive SDK provides ability to live chat. Refer to following API for sending live chat message.

```
fun sendMessage(message: String)
```

Following `BlsChatCallback` callback method is called when host receives new chat message from other viewers.

Return true to consume this message. False to let the library continue handling.

```
// Called when receiving a message.
fun onReceivedMessage(chatMessage: ChatMessage): Boolean
```

Following `BlsChatCallback` callback method is called when new chat message is sent from other viewers.

if success, the \[errorCode] will be \[sg.belive.lib.core.chat.ChatManager.ERR\_CODE\_OK], otherwise it will be \[sg.belive.lib.core.chat.ChatManager].ERR\_CODE\_SEND\_MSG\_XYZ with XYZ is the error type

```
// Notify the send message result.
fun onSendMessageResult(chatMessage: ChatMessage, errorCode: Int)
```

### Step 8 : **Receive statistics update**

BeLive SDK updates view counts and like counts every `20~30` seconds.

Refer to following `BlsLiveStreamCallback` callbacks for receiving updates.

```
override fun onStatistics(statistics: BlsLiveStreamStatistics) {
       // update view count
       // update like count
    }
override fun onDuration(duration: Long)  {
 // Duration of live stream
}
```

### Step 9 : Start OBS Stream (Optional)

BeLive SDK provides an option to start stream using professional broadcasting softwares such as OBS, Wirecast, Vmix etc.

```
fun startObs()
```

You can set `isOBS = true` in `startStream()` method. A unique slug key will be generated and propagated back to SDK in following `BlsLiveStreamCallback` callback method.

```
fun onObsCreated(slug: String) {
     // Get slug here.
     // After starting stream using OBS, make sure that you call following API
    blsLiveStreamManager.startObs()
  }
```

For more details, check `HostActivity` in sample app. You can find default dialog implementation for OBS stream flow.

## Watch your first live stream&#x20;

Summary of the classes for Viewer:

* View/Watch Stream Manager class : `BlsLiveStreamViewerManager`
* View/Watch Stream Callbacks : `BlsLiveStreamViewerCallback`
* Live Chat callbacks : `BlsChatCallback`
* Player Listener : `BlsPlaybackControls.BlsPlaybackListeners`

Refer to Initialize SDK (Viewer) section for initializing and create a session with BeLive API before proceeding to next step&#x20;

### Step 1 : Register Stream callbacks

Initialize `BlsLiveStreamViewerManager` class. Refer to following code in `onCreate()` method of `ViewerActivity` in sample app.

```kotlin
class ViewerActivity : FullScreenActivity(R.layout.activity_viewer), 
    BlsLiveStreamViewerCallback,
    BlsChatCallback, 
    BlsPlaybackControls.BlsPlaybackListeners {

    private lateinit var blsLiveStreamViewerManager: BlsLiveStreamViewerManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        blsLiveStreamViewerManager =
            BlsLiveStreamViewerManager.getInstance(this, belivePlayer).apply {
                setLiveStreamCallback(this@ViewerActivity)
                setChatCallback(this@ViewerActivity)
            }
    }
}
```

Note  that `belivePlayer` is the player view which should be defined in layout:

```xml
<sg.belive.beliveplayer.view.BelivePlayer
        android:id="@+id/belivePlayer"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:isScrollContainer="false"/>
```

### STEP 2 : Join Stream with slug&#x20;

To join stream, you need unique stream slug which is generated for each stream. Refer to below API for joining stream&#x20;

Refer to following method for joining stream with slug:

```kotlin
fun joinStreamWithSlug(slug: String)
```

Refer to following `BlsLiveStreamViewerCallback` callbacks if error occurs during join process:

```kotlin
fun onJoinLiveError(code: Int, message: String?)
```

Refer to following `BlsLiveStreamViewerCallback` callbacks for joining live or recorded stream successfully.

```kotlin
override fun onFirstFrameLoaded() {
    // First video frame is loaded
    // Update UI
}
```

```kotlin
override fun onStartWatching(hostInfo: HostInfo, 
                                    isLiveMode: Boolean) {
    // isLiveMode parameter: whether playing a live (true) or playback (false)
    // Start watching live or recorded video
}
```

At the same time, viewer is connected to live chat channel. Refer to below `BlsChatCallback` callback.

* if success, the \[code] will be \[sg.belive.lib.core.chat.ChatManager.ERR\_CODE\_OK], otherwise it will be \[sg.belive.lib.core.chat.ChatManager].ERR\_CODE\_JOIN\_CHANNEL\_XYZ with XYZ  is the error type

```kotlin
// Notify the joining channel result.
fun onJoinChannelResult(code: Int) { // show chat UI if success }
```

### Step 3 : **Leave Stream**

API

```kotlin
fun leaveStream()
```

Refer to following code for leaving stream. You can find sample code in `ViewerActivity`

```kotlin
private fun leaveStream() {
    blsLiveStreamViewerManager.leaveStream()
}
```

You can release viewer session by using below code:

```kotlin
blsLiveStreamViewerManager.release()
```

### Step 4 : Send Chat message

BeLive SDK provides ability to live chat. Refer to following API for sending live chat message.

```
fun sendMessage(message: String)
```

Following `BlsChatCallback` callback method is called when viewer receives new chat message from other viewers.

Return true to consume this message. False to let the library continue handling.

```kotlin
// Called when receiving a message.
fun onReceivedMessage(chatMessage: ChatMessage): Boolean
```

Following `BlsChatCallback` callback method is called when new chat message is sent from other viewers.

if success, the \[errorCode] will be \[sg.belive.lib.core.chat.ChatManager.ERR\_CODE\_OK], otherwise it will be \[sg.belive.lib.core.chat.ChatManager].ERR\_CODE\_SEND\_MSG\_XYZ with XYZ is the error type

```kotlin
// Notify the send message result.
fun onSendMessageResult(chatMessage: ChatMessage, errorCode: Int)
```

### Step 5 : **Receive statistics update**

BeLive SDK updates view counts and like counts every 20\~30 seconds.

Refer to folowing `BlsLiveStreamViewerCallback` callback for receiving updates.

```
fun onStatistics(statistics: BlsLiveStreamStatistics) {
       // update view count
       // update like count
    }
```

**For recorded video playback, there will be additional callbacks from player to update video progress**

```kotlin
override fun onVideoProgress(currentMls: Long, 
                                s: Int, 
                                duration: Long, 
                                percent: Int) {
        //update the progress here
        blsPlaybackControl.updatePlaybackProgress(currentMls, duration, percent)
    }
```

### Step **6 :  Handle Stream ended by Host**

In case live stream is ended by host, viewer will receive following `BlsLiveStreamViewerCallback` callbacks.

```kotlin
fun onStreamEnded(reason: Int) { 
// handle visibilty of blsEndStreamViewerScreen 
}
```

### Step **7 :  Recorded video playback**

Refer to following `BlsPlaybackControls.BlsPlaybackListeners` callbacks for recorded video playback. `stream.status` for recorded streams will be 2 and `isRecorded` field will be true .

> Note that play, pause, and seekbar buttons are not available in Live stream mode

```kotlin
interface BlsPlaybackListeners{
     fun onPlayButtonClicked()
     fun onPauseButtonClicked()
     fun onSeekBarStartTracking()
     fun onSeekBarStopTracking(percent: Int, isBackWard: Boolean)
}
```

Refer to following `BlsLiveStreamViewerCallback` callbacks for recorded video playback.

```kotlin
fun onVideoProgress(currentMls: Long, s: Int, duration: Long, percent: Int)
fun updatePlaybackBuffered(bufferedPercentage: Int)
```

Example code for how to handle above callbacks is provided in `ViewerActivity` sample.

### Picture-in-picture (PIP) mode

BeLive SDK supports picture-in-picture mode.

> Note : Picture-in-picture mode is supported on Android 8.0 (API level 26) and above. [Reference](https://developer.android.com/guide/topics/ui/picture-in-picture)

To enable picture-in-picture (pip) model set `"allowPictureInPicture": true` in `belive-configuration.json` file.

**1. Declare in AndroidManifest**

Add following under `ViewerActivity` tag

```
android:supportsPictureInPicture="true"
```

**2. Request PIP permissions**

Refer to following code for entering PIP Code. Sample code is provided in  `ViewerActivity`

```
private fun requestPip() {
    val hasPipPermission =  blsLiveStreamViewerManager.hasPipPermission()
    Log.d(TAG, "has pip permission = $hasPipPermission, ")
    if (!hasPipPermission){
        openPopupToRemindUserEnablePipPermission()
        return
    }
    blsLiveStreamViewerManager.enterPipMode()
}
```

**3. Handle Activity's `onPictureInPictureModeChanged` method**

```
override fun onPictureInPictureModeChanged(
        isInPictureInPictureMode: Boolean,
        newConfig: Configuration?
    ) {
        if (isInPictureInPictureMode) {
            // Hide the full-screen UI (controls, etc.) 
            // while in picture-in-picture mode.
            setViewsVisibilityForPipMode()
        } else {
            // Restore the full-screen UI.
            restoreViewsVisibilityFromPipMode()
        }
        isPipMode = isInPictureInPictureMode
        checkToShowPipEndScreenIfNeed()
    }
```

## Stream Health Monitoring (Beta)

SDK provides videoBitrate monitoring during stream. It determines the quality of stream and an indication of host's network conditions. `onStatistics` callback returns average bitrate with parameter `actualBitrate`

```kotlin
data class BlsLiveStreamStatistics(
    val viewCount: Int = 0,
    val likeCount: Int = 0,
    val duration: Long = 0,
    /**
     * The max bitrate which is set to the stream.
     * This value is used as a hint to the streamer 
     * and it is not warranty that the actual bitrate 
     * always be limited by this value.
     * It means the actual bitrate and uploadBitrate might 
     * be greater than this max bitrate
     */
    val maxBitRate: Int = 0,
    /**
     * The actual bitrate that is calculated by the server
     */
    val actualBitRate: Int = 0,
    /**
     * The current bitrate that is calculated by the streamer
     */
    val uploadBitRate: Int = 0,
)
```

## Live Shopping

SDK provides live shopping feature. You can use either add products for a host using CMS or SDK API before starting stream. (We will refer to host as `seller`).  Refer to Live Shopping guide section for more details.\
\
Live Shopping module of BeLive SDK provides following classes, methods and callbacks.

`BlsLiveShoppingManager`. This class works for both host/broadcast and viewer. But on viewer side, calling edit products, add products, delete products will return error (`INVALID_DATA`)

This liveshopping module provides single callback interface to listen for any changes related to the products.

\
`BlsLiveShoppingCallback`

```kotlin
interface BlsLiveShoppingCallback {
  fun onProductListResult(productListResultData: ProductListResultData)
  fun onUpdateProductResult(updateProductResultData: UpdateProductResultData)
  fun onDeleteProductResult(deleteProductResultData: DeleteProductResultData)
  fun onAddProductResult(addProductResultData: AddProductResultData)
  fun onPromoCodeUpdated(promoCodeData: PromoCodeData)
  fun onUpdatePromoCodeResult(updatePromoCodeResultData: UpdatePromoCodeResultData)
  fun onLiveShoppingError(errorInfo: ErrorInfo)
}
```

The `BlsLiveShoppingManager` class provides these api to manage products.

**Get product list in pre-live (before stream starts for host) and during live/recorded stream**

```kotlin

fun getProducts(getProductRequest: GetProductRequest)
/*
class GetProductRequest {
    var slug: String? = null 
    var pageIndex: Int = 0
    var limit: Int = BasePagingRequestModel.DEF_LIMIT_PER_PAGE
}
   slug: the stream slug. If null it will returns products that are set to the 
   current logged in user (host/broadcaster). otherwise, the returned products 
   are stream products.
   pageIndex: The page index to load the corresponding products
   limit: Limit the products per page (default is 20)
   
   Callback: BlsLiveShoppingCallback.onProductListResult(..). 
    The productListResultData contains error if the action is failed. 
    otherwise it contains product list
 */
```

**Add new product (For Only Host / Seller)**

```
fun addProduct(addProductRequest: AddProductRequest)
/* 
data class AddProductRequest(
    val name: String,
    val price: Double,
    val description: String,
    val promotionPrice: Double,
    val sku: String,
    val url: String,
    val imageFile: Uri? = null,
)
Callback: BlsLiveShoppingCallback.onAddProductResult(..)
*/
```

**Edit product (For Only Host / Seller)**

```
fun editProducteditProduct(editProductRequest: EditProductRequest)
/* 
data class EditProductRequest(
    val product: ProductModel,
    val newName: String,
    val newPrice: Double,
    val newDescription: String,
    val newPromotionPrice: Double,
    val newSku: String,
    val newUrl: String,
    val newImageFile: Uri? = null,
    val newOrderIndex: Int,
)
Callback: BlsLiveShoppingCallback.onUpdateProductResult with 
          UpdateProductResultData.TYPE_UPDATE
*/
```

**Update product Order (For Only Host / Seller)**

```
fun updateProductOrder(updateProductOrderRequest: UpdateProductOrderRequest)
/* 
data class UpdateProductOrderRequest(
    val product: ProductModel,
    val slug: String? = null,
    val newIndex: Int = 0,
)

Callback: BlsLiveShoppingCallback.onUpdateProductResult with 
          UpdateProductResultData.TYPE_ORDER_CHANGED
*/
```

**Delete (deactivate) product (For Only Host / Seller)**

```
fun deleteProduct(deleteProductRequest: DeleteProductRequest)
/* 
data class DeleteProductRequest(
    val product: ProductModel,
)

Callback: BlsLiveShoppingCallback.onDeleteProductResult(..)
*/
```

**Delete (deactivate) all products (For Only Host / Seller)**

```
fun deleteAllProduct(deleteAllProductRequest: DeleteAllProductRequest)
/* 
class DeleteAllProductRequest()

Callback: BlsLiveShoppingCallback.onDeleteProductResult with 
                DeleteProductResultData.isDeleteAll = true
*/
```

**Edit Promo code before starting live stream (For Only Host / Seller)**

Note : During live stream, we cannot edit promo code from Android SDK. It should be done using CMS

```
fun editPromoCode(editPromoRequest: EditPromoCodeRequest)
/* 
data class EditPromoCodeRequest(
    val promoCode: String,
    val promoDesc: String,
) 

Callback: BlsLiveShoppingCallback.onUpdatePromoCodeResult
*/
```

Other callbacks:

```
fun onLiveShoppingError(errorInfo: ErrorInfo)

/* This callback is triggered when the promo code is updated 
* (it might trigger many times). Because the promo 
* code might depend on a stream, you must attach it with 
* a BlsBaseStreamManager for this callback to be triggered correctly,
*/
fun onPromoCodeUpdated(promoCodeData: PromoCodeData)

```
