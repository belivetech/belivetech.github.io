---
title: SDK Guide for iOS
---

The BeLive iOS SDK provides the interfaces required to broadcast and playback on iPhone devices along with interactive features.&#x20;

The following operations are supported:&#x20;

• Set up (initialize) SDK&#x20;

• Manage live broadcasting.&#x20;

• Watch live stream&#x20;

• Watch Recorded stream

• Manage Live Shopping

Latest version of iOS SDK: 1.0.0 (Release Notes)

## Getting Started

### Requirements

* iOS 11.0+
* Xcode 13.0
* BeLiveCore, BeLiveBroadcaster and BeLiveAudience frameworks

### Install the SDK

BeLive SDK for iOS supports both Cocoapods and Carthage. Follow instructions below to install SDK

### Cocoapods (Recommended Integration)

You can use [CocoaPods](http://cocoapods.org) to install BeLive SDK by adding following to your `Podfile`

```ruby
platform :ios, '11.0'
use_frameworks!
pod 'BeLiveCore', :path => '../framework/BeLiveCore'
pod 'BeLiveAudience', :path => '../framework/BeLiveAudience'
pod 'BeLiveBroadcaster', :path => '../framework/BeLiveBroadcaster'
```

> Take note about framework folder which contains `podspec` files and `xcframework`

Each of the `podspec` file have their dependencies added in respective files.

**Alternative Approach**

1. Extract the contents of the framework. There will be three `xcframeworks` : `BeLiveCore.xcframework` , BeLiveAudience.`xcframework` and `BeLiveBroadcaster.xcframwork`
2. Embed  all three xcframework by dragging it into the Frameworks, Libraries, and Embedded Content section of the General tab for your application target.

![](<../.gitbook/assets/Screenshot 2022-01-23 at 11.22.06 PM.png>)

Belive iOS SDK uses following open source libraries which must be added in Podfile with specific versions

```
# BeLiveCore
'Alamofire', '5.5.0'
'AgoraRtm_iOS', '1.4.8'
# BeLiveAudience 
'AmazonIVSPlayer', '1.5.0'
# BeLiveBroadcaster 
'GPUImage', '0.1.7'
'libksygpulive',:git => 'https://github.com/ksvc/KSYLive_iOS.git', :tag => 'v3.0.5'

```

Run `pod install` . Open your `xcworkspace` and import BeLive SDK\


```swift
import UIKit
import BeLiveCore
import BeLiveAudience
import BeLiveBroadcaster
```

### Carthage

_in progress_

## **Start your first live stream**

Before starting or playing back live stream, SDK must be initialized with `license key.`Obtain license key from business team by providing your app's bundle Id.

### Step 1: Initialize SDK

First step is to **verify license key** and expiry time of SDK. Add following code in `didFinishLaunchingWithOptions` of AppDelegate or `viewDidLoad` of ViewController.&#x20;

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions 
    launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Init BeLive SDK with License Key and Domain
        BeLiveSDK.shared.initWith(
            licenseKey: "license_key",
            environment: .demo,
            delegate: self
        )
        // Custom environment.
        let custom = BLEnvironmentConfiguration.init(apiDomain: "", 
            wowzaHostDomain: "", 
            wowzaViewerDomain: "")
        return true
    }
    
extension AppDelegate: BeLiveSDKDelegate {
    func sdkExpired() {
        // This callback will be called only if Demo SDK is expired.
    }
}
```

For SDK testing, use demo environment. For production environment, `apiDomain` will be provided.

Nest step is to login which essentially is creating a session with Belive backend. There is slight difference between Host login and Viewer Login. Note that host login accounts are pre-generated while viewer accounts can be created using SDK API.\


**Access BeLive API Services from BeLiveSDK singleton functions**

**User Services**

```swift
BeLiveSDK.shared.userService()

/// Access to current user after logging in.
BeLiveSDK.shared.userService().currentUser()
/// Whenver you call login API, current user will be saved.
/// Whenver you call logout API, current user will be logged out.

/// Check login status
BeLiveSDK.shared.userService().isLogin()

/// Check is guest user
BeLiveSDK.shared.userService().isGuest()

/// Log-in as HOST/Broadcaster
/// - Parameters:
///   - beliveId: ID of User
///   - password: Password of User
///   - deviceUdid: Udid of device
///   - deviceType: Type of device (Android is 0, iOS is 1)
///   - latitude: Latitute of device (optional)
///   - longitude: Longitute of device (optional)
///   - completion: Result of User
func login(beliveId: String, 
                  password: String, 
                  deviceUdid: String, 
                  deviceType: Int, 
                  latitude: Double, 
                  longitude: Double,
                  completion: @escaping (BLAPIResult<BLUser>) -> Void

/// Log-in as Viewer
/// - Parameters:
///   - beliveId: ID of User
///   - displayName: Display name of User
///   - deviceUdid: Udid of device
///   - deviceType: Type of device (Android is 0, iOS is 1)
///   - latitude: Latitute of device (optional)
///   - longitude: Longitute of device (optional)
///   - completion: Result of User
func loginBeLive(beliveId: String, 
                  displayName: String, 
                  latitude: Double, 
                  longitude: Double,
                  deviceUdid: String, 
                  deviceType: Int, 
                  completion: @escaping (BLAPIResult<BLUser>) -> Void

/// Log-in as Guest user
/// - Parameters:
///   - deviceUdid: Udid of device
///   - completion: Result of User
func loginGuest(deviceUdid: String, 
                  completion: @escaping (BLAPIResult<BLUser>) -> Void)

/// Log-out user
/// - Parameters:
///   - completion: Result of Bool
func logoutBeLive(completion: @escaping (BLAPIResult<Bool>) -> Void)
```

\
**Host / Broadcaster Login**

```kotlin
BeLiveSDK.shared.userService().login(
            beliveId: "beliveId",
            password: "password",
            deviceUdid: "deviceUdid",
            latitude: 0,
            longitude: 0
        ) { [weak self] result in
            guard let self = self else { return }
            
            switch result {
            case .ok:
                DispatchQueue.main.async { [weak self] in
                    guard let self = self else { return }
                    // setUpUI()
                }
            case .error(error: let error):
                   print(error)
                }
            }
```

**Viewer Login (SSO Login with unique userId and displayName)**

```kotlin
BeLiveSDK.shared.userService().loginBeLive(
            beliveId: "beliveId",
            displayName: "displayName",
            deviceUdid: "deviceUdid",
            latitude: 0,
            longitude: 0
        ) { [weak self] result in
            guard let self = self else { return }
            
            switch result {
            case .ok:
                DispatchQueue.main.async { [weak self] in
                    guard let self = self else { return }
                    // setUpUI()
                }
            case .error(error: let error):
                   print(error)
                }
            }

```

**Guest Login**

```kotlin
BeLiveSDK.shared.userService().loginGuest(deviceUdid: "deviceUdid")
        {[weak self ] result in
            guard let self = self else { return }
            
            switch result {
            case .ok:
                DispatchQueue.main.async { [weak self] in
                    guard let self = self else { return }
                    // setUpUI()
                }
            case .error(error: let error):
                   print(error)
                }
        }
```

**Logout**&#x20;

```kotlin
 BeLiveSDK.shared.userService().logoutBeLive() {
            [weak self ] result in
                guard let self = self else { return }
        }
```

### Step 2 : Register broadcast delegates and set config

Summary

* Broadcast Manager : `BeLiveBroadcasterManager`
* Live Chat Delegate : `BeLiveBroadcasterManagerMessageDelegate`
* Live Broadcast Delegate : `BeLiveBroadcasterStreamKitDelegate`

```kotlin
// init BeLive Broadcaster Manager
let broadcasterManager = BeLiveBroadcasterManager(renderView: previewView)
broadcasterManager?.messageDelegate = self
broadcasterManager?.streamKitDelegate = self
broadcasterManager?.setMaxKeyInterval(value: 2.0)
broadcasterManager?.setQuality(BLStreamQuality.sd)
broadcasterManager?.startPreview()
```

**Broadcast configuration**&#x20;

| API               | Description                                                                            |
| ----------------- | -------------------------------------------------------------------------------------- |
| setStreamQuality  | Two available options for Stream Quality are BLStreamQuality.sd and BLStreamQuality.hd |
| setMaxKeyInterval | For lower end-to-end latency, set maxKeyInterval to 1F.                                |

**Summary**;

BroadcastView controller should implement below delegates to receive state updates about chat and broadcast session

```swift
/// BeLive Broadcaster Manager Message Delegate
public protocol BeLiveBroadcasterManagerMessageDelegate: AnyObject {
    func beLiveBroadcasterManagerDidJoinStreamChannel(manager: BeLiveBroadcasterManager)
    func beLiveBroadcasterManager(manager: BeLiveBroadcasterManager, didFailedToJoinStreamChannel error: String)
    func beLiveBroadcasterManager(manager: BeLiveBroadcasterManager,didReceivePeerMessage message: BLStreamMessage)
    func beLiveBroadcasterManager(manager: BeLiveBroadcasterManager, didReceiveStreamMessage message: BLStreamMessage)
    func beLiveBroadcasterManager(manager: BeLiveBroadcasterManager, userJoined userId: String)
    func beLiveBroadcasterManager(manager: BeLiveBroadcasterManager, userLeft userId: String)
}

```

```swift
/// BeLive Broadcaster Manager Stream Kit Delegate
public protocol BeLiveBroadcasterStreamKitDelegate: AnyObject {
    func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamDidCreate stream: BLStream)
    func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamDidBegin stream: BLStream)
    func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamDidEnd endStream: BLEndStream, withReason reason: String?)
    func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamError error: Error)
    func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamStatisticDidUpdate statistic: BLStreamStatistic)
    func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, stateDidChange state: BLStreamState)
    func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, netStateCodeDidChange state: BLNetStateCode)
    /// In the Trial SDK. The host can only stream for 15 minutes.
    func beLiveBroadcasterManagerLimitation(manager: BeLiveBroadcasterManager)
}
```

### Step 3 : Request Permissions;

Before start live stream, camera and microphone permissions are required. Your app must request permission to access the user’s camera and mic. (This is not specific to BeLive SDK; it is required for any application that needs access to cameras and microphones.)

```swift
switch AVCaptureDevice.authorizationStatus(for: .video) {
    case .authorized: // permission already granted.
    case .notDetermined:
    AVCaptureDevice.requestAccess(for: .video) { granted in
    // permission granted based on granted bool.
    }
    case .denied, .restricted: // permission denied.
    @unknown default: // permissions unknown.
}

```

You need to do this for both .video and .audio. You also need to add entries for **NSCameraUsageDescription** and **NSMicrophoneUsageDescription** to your Info.plist. Otherwise, your app will not work when trying to request permissions.

This is optional but recommended. It prevents your device from going to sleep while using the broadcast SDK, which would interrupt the broadcast.

```swift
override func viewDidAppear(_ animated: Bool) {
  super.viewDidAppear(animated)
  UIApplication.shared.isIdleTimerDisabled = true
}
override func viewDidDisappear(_ animated: Bool) {
   super.viewDidDisappear(animated)
   UIApplication.shared.isIdleTimerDisabled = false
}
```

### Step 4 : Start Live Broadcast Session

SDK requires stream title as mandatory field to start stream.

API 

```swift
public func createStream(title: String, coverImage: UIImage?, completion: @escaping ((BLStream?, Error?) -> Void))
public func beginStream() 
public func beginObsStream()

```

```swift
fun startLiveBroadcast() {
        let obs = false
        self.broadcasterManager?.createStream(title: title, coverImage: self.viewModel.input?.coverImage.value, completion: 
        { [weak self] stream, error in
            guard let self = self else { return }
            if let error = error {
               // Error occured while creating stream.
                return
            }

            // Create stream success
            if isOBS {

                // copy stream slug (which is stream key for Broadcasting software)
                 print("\(stream.slug)")
                // After starting stream with OBS, call the below method to begin obs stream 
                self.broadcasterManager?.beginObsStream()
            } else {
                // Begin normal stream 
                self.broadcasterManager?.beginStream()
            }
        })
    }
```


At this moment, SDK supports only `portrait` stream. Following delegate method will be called for stream status.

**Stream created succesfully** 

```swift 
func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamDidCreate stream: BLStream) {
       // stream created successfully
}

```

**Stream began succesfully** 
```swift 
func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamDidBegin stream: BLStream) {
        // Stream began successfully - Update UI 
}
```

**Streaming server connection status** 

`Connected` state indicates that streaming server is getting stream data from mobile client

> Note : Below delegate will not be called in case of OBS streaming

```swift
func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, 
    stateDidChange state: BLStreamState) {
        switch state {
        case .idle:
            BLLog.s("BLStream Kit Idle")
        case .connecting:
            BLLog.s("BLStream Kit Connecting")
        case .connected:
            BLLog.s("BLStream Kit Connected")
            // live stream started
        case .disconnecting:
            BLLog.s("BLStream Kit Disconnecting")
        case .error(let code):
            BLLog.e("BLStream Kit Error Code: \(code)")
        }
    }
```

**Stream error** 

```swift
func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamError error: Error) {
        // Error while starting stream
    }

```

At the same time, host is connected to live chat channel. Refer to below delegate methods for success and failure case&#x20;

```swift
func beLiveBroadcasterManagerDidJoinStreamChannel(
                        manager: BeLiveBroadcasterManager) {
    // TODO
    print("beLiveBroadcasterManagerDidJoinStreamChannel")
 }

func beLiveBroadcasterManager(
                    manager: BeLiveBroadcasterManager, 
                    didFailedToJoinStreamChannel error: String) {
    // TODO
    print("didFailedToJoinStreamChannel :\(error)")
}
```

### Step 5 : Stop Live Broadcast Session

API

```swift
public func endStream(slug: String, 
    completion: @escaping (BLAPIResult<BLEndStream>) -> Void)
```

Usage

```swift
func endStream() {
        self.broadcasterManager?.endStream()
    }
```

Callback

Follow delegate method will be called for end stream status

```swift
func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamDidEnd endStream: BLEndStream, withReason reason: String?) {
    print("streamDidEnd")

}
```

### Step 6 : Save Stream

API

```swift
public func recordStream(completion: @escaping (Error?) -> Void)
```

Usage

```swift
self.broadcasterManager?.recordStream(completion: { [weak self] error in
            guard let self = self else { return }
            if let error = error {
                // error while recording stream
            }
})
```

You can release broadcast session by calling following method&#x20;

```swift
 self.broadcasterManager?.releaseKit()
```

### Step 7 : Send and Receive Chat message

BeLive SDK provides ability to live chat. Refer to following API for sending live chat message.

```swift
func sendChatMessage() {
    self.broadcasterManager?.sendChatMessage(content: "string".censored(),
            completion: { 
                [weak self] message in
                guard let self = self else { return }
                DispatchQueue.main.async { [weak self] in
                    guard let self = self, let message = message else {
                        print("Failed to send message! Please try again")
                    return
                }
                // update UI
            }
        })   
}
```

Following delegate methods are called when host receives new chat message from other viewers or admin.

```swift
func beLiveBroadcasterManager(manager: BeLiveBroadcasterManager, didReceivePeerMessage message: BLStreamMessage) {
        self.receiveNewMessage(message: message)
    }

func beLiveBroadcasterManager(manager: BeLiveBroadcasterManager, didReceiveStreamMessage message: BLStreamMessage) {
        self.receiveNewMessage(message: message)
}
func receiveNewMessage(message: BLStreamMessage)  {
    switch message.messageType {
        case BLMessageType.message.rawValue, BLMessageType.share.rawValue:
            // normal chat message or share message
        case BLMessageType.viewerJoined.rawValue:
        case BLMessageType.like.rawValue:
            // like message type
        case BLMessageType.adminStop.rawValue:
            print("Your live has been removed due to poor content")
            // force close live stream
        default:
            break
        }
}
```

### Step 8 : Receive statistics updates

BeLive SDK updates view counts and like counts every `30` seconds.

Refer to following `BeLiveBroadcasterStreamKitDelegate` callback for receiving updates.

```kotlin
func beLiveBroadcasterManager(kit: BeLiveBroadcasterManager, streamStatisticDidUpdate statistic: BLStreamStatistic) {
        // Promo code
        print(statistic.promoCode)
        // Promo code description
        print(statistic.promoCodeDescription)
}

```


### Step 9 : Start OBS Stream (Optional)

Refer to step 4 (Start Live Broadcast Session) for more details.

### Utility Methods 

#### Switch Camera 

Use following method to switch between front and back camera.

```swift
self.broadcasterManager?.switchCamera()
```
You can check which camera is active by using below boolean 

```swift
let isFrontCamera = self.broadcasterManager?.isFrontCamera

```

### Enable beautyfilter 

Enable or disable beauty filter using following methods

```swift
self.broadcasterManager?.enableBeautyFilter()
self.broadcasterManager?.disableBeautyfilter()
```

You can check whether beauty filter is enabled by using below boolean 

```swift
let isBeautyEnable =  self.broadcasterManager?.isBeautyEnable 
```

## Watch your first live stream;

Summary of the classes for Viewer:

* View/Watch Stream Manager class : `BeLiveAudienceManager`
* Player Callbacks : `BeLiveAudienceManagerPlayerDelegate`
* Live Chat callbacks : `BeLiveAudienceManagerMessageDelegate`

Refer to Initialize SDK (Viewer) section for initializing and create a session with BeLive API before proceeding to next step&#x20;

### Step 1 : Register Stream playback delegates

```swift
 var viewerManager: BeLiveAudienceManager?
func setupView() {
    // Init BeLive Audience Viewer Manager
    self.viewerManager = BeLiveAudienceManager(renderView: self.playerView, stream: stream)
    self.viewerManager?.playerDelegate = self
    self.viewerManager?.messageDelegate = self
}
```

Note that `playerView` is the UIView where you want to show video.&#x20;

### Step 2 : Join Stream with slug

To join stream, you need unique stream slug which is generated for each stream. Refer to below API for joining stream

```swift
func joinWithSlug(slug: String) {

        let streamService = BeLiveSDK.shared.streamService()
        streamService.getStreamDetail(slug: slug) { [weak self] result in
            guard let self = self else { return }
            switch result {
                case .ok(data: let stream):
                    print(stream.title)
                case .error(error: let error):
                    print(error)
            }
        }
}
```

Following delegate methods are called for success and failure case

```swift
func beLiveAudienceManager(manager: BeLiveAudienceManager, 
                    player: BLPlayerKit, 
                    playerStateDidChange state: BLPlayerKitState) {

    switch state {
        case .ready, .playing:
            // Playback has started
        case .idle, .stopped, .ended:
            // Playback state changed
        default:
            break
        }
s}

func beLiveAudienceManager(manager: BeLiveAudienceManager, 
                    player: BLPlayerKit, 
                    playerDidFailWithError error: Error?) {
    if let error = error {
        print(error)
    }
}
```

At the same time, viewer is connected to live chat channel. Refer to below `BeLiveAudienceManagerMessageDelegate` callback.

```swift
func beLiveAudienceManagerDidJoinStreamChannel(manager: BeLiveAudienceManager) {
    manager.sendJoinMessage(completion: { [weak self] message in
        guard let self = self, let message = message else { return }
            self.receiveNewMessage(message: message)
    })
}

func beLiveAudienceManager(manager: BeLiveAudienceManager, 
                    didFailedToJoinStreamChannel error: String) {
    print("\(error)")
}
```

### Step 3 : **Leave Stream**

```swift
func leaveStream() {

        let streamService = BeLiveSDK.shared.streamService()
        let streamUserId = stream.userId
        let slug = stream.slug
        streamService.leaveStream(streamUserId: streamUserId, slug: slug) { 
        [weak self] result in
            guard let self = self else { return }
            switch result {
                case .ok(data: let status):
                    print(status)
                case .error(error: let error):
                    print(error)
            }
        }
}
```

You can release resources by using below code:

```swift
self.viewerManager?.release()
```

### Step 4 : Send and Receive Chat message

BeLive SDK provides ability to live chat. Refer to following API for sending live chat message.

```swift
func sendChatMessage() {
    self.viewerManager?.sendChatMessage(content: content.censored()) 
    { [weak self] message in
        guard let self = self else { return }
            DispatchQueue.main.async { [weak self] in
                guard let self = self, let message = message else { return }
                 // update UI
            }
}
```

Following delegate methods are called when host receives new chat message from other viewers or admin.

```swift
func beLiveAudienceManager(manager: BeLiveAudienceManager, 
    didReceivePeerMessage message: BLStreamMessage) {
        self.receiveNewMessage(message: message)
    }

func beLiveAudienceManager(manager: BeLiveAudienceManager, 
        didReceiveStreamMessage message: BLStreamMessage) {
        self.receiveNewMessage(message: message)
    }
func receiveNewMessage(message: BLStreamMessage)  {
    switch message.messageType {
        case BLMessageType.message.rawValue, BLMessageType.share.rawValue:
            // normal chat message or share message
        case BLMessageType.viewerJoined.rawValue:
        case BLMessageType.like.rawValue:
            // like message type
        case BLMessageType.adminStop.rawValue:
            print("Live has been removed due to poor content")
            // leave live stream
        case BLMessageType.hostStreamEnd.rawValue:
            print("Host ended this stream!!!")
            // Update UI
            self.viewerManager?.release()
        default:
            break
        }
}
```

### Step 5 : **Receive statistics update**

In didReceiveStreamMessage, filter out message type `statistics` to get stream statistics update&#x20;

```swift
case BLMessageType.statistic.rawValue:
    guard let statisticMessage = message as? BLStatisticMessage else { return }
    DispatchQueue.main.async { [weak self] in
        guard let self = self else { return }
        let viewCount = String(statisticMessage.totalViewers ?? 0)
        let likeCount = String(statisticMessage.totalLikes ?? 0)
        
        // Promo code update (related to live shopping)
        print(statisticMessage?.promoCode)
        print(statisticMessage?.promoCodeDescription)
}
```

### Step **6 :  Handle Stream ended by Host**

In case live stream is ended by host, viewer will receive following message type

```swift
 case BLMessageType.hostStreamEnd.rawValue:
            print("Host ended this stream!!!")
            // Update UI
            self.viewerManager?.release()
```

### Step **7 :  Recorded video playback**

SDK supports utility methods for playback of recorded stream. `stream.status` for recorded streams will be 2 and `isRecorded` field will be true . 
Refer to below API

```swift
/// Player
self.viewerManager?.play() /// Play stream
self.viewerManager?.pause() /// Pause stream
self.viewerManager?.stop() /// Stop stream
// Seek to a time of stream
self.viewerManager?.seak(to: 10.0) { success in
}
```

Delegate methods for player status&#x20;

```swift
func blViewerManager(manager: BLViewerManager, 
                    player: BLPlayerKit, 
                    playerDidChangeDuration duration: TimeInterval) {
        print(duration)
        // Refer top to sample app for example
    }
    func blViewerManager(manager: BLViewerManager, 
                        player: BLPlayerKit, 
                        playerDidChangePosition position: TimeInterval) {
        print(position)
        // Refer top to sample app for example
    }
```

### Picture-in-picture (PIP) mode

BeLive SDK supports picture-in-picture mode. Refer to our sample app.

**Picture in Picture (PIP) In-App**

```swift
/// Present a Viewer Controller for PIP mode
BLPIP.shared.present(with: yourViewController)
/// Show PIP
BLPIP.shared.makeSmaller()
/// Resume
BLPIP.shared.makeLarger()
/// Dismiss PIP
BLPIP.shared.dismiss(animation: true)
```

**Picture in Picture (PIP) Out-of-App**

```swift
/// This feature works only in iOS version 14.2 and above.

/// Show PIP. Call this when app did enter background
self.viewerManager.startPIP()

/// Hide PIP. Call this when app enters foreground 
self.viewerManager.stopPIP()
```

## Stream Health Monitoring (Beta)

SDK provides videoBitrate monitoring during stream. It determines the quality of stream and an indication of host's network conditions. `didReceiveStreamMessage` callback returns average `bitrate` in BLMessageType.statistic type. See below for all parameters of BLStaticMessage

``

**Statistic Message**

```swift
/// Every 30s, the app will receive statistic 
/// message with detail information as below, 
/// Viewer can use these infomation to show on UI

 class BLStatisticMessage: BLStreamMessage {
    
    public var slug: String?        // slug of stream
    public var streamDuration: Int? // duration of stream
    public var totalViewers: Int?   // total viewer of stream
    public var totalLikes: Int?     // total like of stream
    public var totalReceivedStars: Int?  // total star received of stream
    public var gender: Int?                 
    public var isShowLike: Bool?   // enable show like count stream
    public var isShowView: Bool?   // enable show view count of stream
    public var promoCode: String?  // promote code of stream
    public var placingOrderCount: Int?  
    public var cartUrl: String?
    public var bitRate: Int?    // current bitrate of stream
}
```

## Live Shopping

SDK provides live shopping feature. You can use either add products for a host using CMS or SDK API before starting stream. (We will refer to host as `seller`).  Refer to Live Shopping guide section for more details.

Live Shopping module of BeLive SDK provides following classes and methods and callbacks.

`BLProductService :` This class works for host/broadcaster only. It provides below utility methods. On viewer side, calling edit products, add products, delete products will return error.

**Get product list in pre-live (before stream starts for host)**

```swift
func getProductsInPreLive() {
        
        let productService = BeLiveSDK.shared.productService()
        productService.getProducts(nextId: 0, completion: {
            [weak self] result in
            guard let self = self else { return }
            switch result {
                case .ok(data: let products):
                    // promo code
                    print(products.promoCode)
                    // promo code description
                    print(products.promoCodeDescription)
                    // total shopping items
                    print(products.totalRecords)
                    // shopping items array (default limit : 20)
                    print(products.result)
                case .error(error: let error):
                    print(error)
                @unknown default:
                    print("default")
            }
        })
            
}
```

**Add new product (For Only Host / Seller). Host can add products before starting broadcast.**

```swift
func getProductsInPreLive() {
        
        let productService = BeLiveSDK.shared.productService()
        productService.addProduct(name: "name", 
                            price: 100, 
                            promotionPrice: 90, 
                            description: "description", 
                            image: Data(), 
                            url: "url", 
                            sku: "sku", 
                            completion: {
            [weak self] result in
            guard let self = self else { return }
            switch result {
                case .ok(data: let product):
                    // Uploaded product
                    print(product.name)
                case .error(error: let error):
                    print(error)
                @unknown default:
                    print("default")
            }
        })
            
    }
```

Similarly other available APIs are listed below&#x20;

```swift
/// Edit product
    /// - Parameters:
    ///   - id: Current product id.
    ///   - name: New product name.
    ///   - price: New product price.
    ///   - promotionPrice: New product promotion price.
    ///   - description: New product description. Optional value
    ///   - image: New product image.
    ///   - url: New product url.
    ///   - sku: New product SKU. Optional value
    ///   - completion: Result of BLStreamProduct.
    public func editProduct(
        id: Int,
        name: String,
        price: Double,
        promotionPrice: Double,
        description: String?,
        image: Data,
        url: String,
        sku: String?,
        completion: @escaping (BLAPIResult<BLStreamProduct>) -> Void
    )
    
/// Delete one product
    /// - Parameters:
    ///   - productId: with Product ID
    ///   - completion: Result of Bool
    public func deleteProduct(productId: Int, 
                            completion: @escaping (BLAPIResult<Bool>) -> Void)

/// Edit displaying order of a product
    /// - Parameters:
    ///   - productId: Id of Product
    ///   - slug: Stream Identifier
    ///   - orderIndex: Order Index
    ///   - completion: Result of Bool
    public func editProductOrder(
        productId: Int,
        slug: String,
        orderIndex: Int,
        completion: @escaping (BLAPIResult<Bool>) -> Void
    )

/// Delete all product
    /// - Parameter completion: Result of Bool
    public func deleteAllProduct(completion: @escaping (BLAPIResult<Bool>) -> Void )
    
```

BLStreamService.getStreamProductList : This method is used by both host/broadcaster and viewer to fetch product list during live and recorded stream.

```swift
func getProductsInStream() {
        
        let streamService = BeLiveSDK.shared.streamService()
        
        streamService.getStreamProductList(slug: "stream.slug",
                                           nextId: 20, completion: {
            [weak self] result in
            guard let self = self else { return }
            switch result {
                case .ok(data: let products):
                    // Shopping items array (default limit : 20)
                    print(products.result)
                    // promo code
                    print(products.promoCode)
                    // promo code description
                    print(products.promoCodeDescription)
                    // total shopping items
                    print(products.totalRecords)
                case .error(error: let error):
                    print(error)
                @unknown default:
                    print("default")
                }
        })
            
    }
```

