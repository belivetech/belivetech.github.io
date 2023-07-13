---
title: Short Video SDK Guide for iOS
redirect_from:
  - /getting-started.html
---

**`ShortVideoSDK`** is a module of `BeLiveSDK`, can be used with other modules of `BeLiveSDK` or independently. Short Video requires `BeLiveCore` module to work normally.

# Features
- Video Grid View
- Video Carousel View
- Playlist
- PIP

# Requirements
- iOS 11.0+
- Xcode 13.0 and above

# Integration
Below is how to add `ShortVideoSDK` independently into project. If you want to add other module of `BeLiveSDK`, please check `BeLiveSDK` document for integration instruction.

## CocoaPods
This is simplest way to integrate `ShortVideoSDK` into your app.
You can use [CocoaPods](http://cocoapods.org/) to install `ShortVideoSDK` by adding it to your `Podfile`

```ruby
platform :ios, '11.0'
use_frameworks!
pod 'BeLiveCore', :path => '../framework/BeLiveCore' # path to podspec file
pod 'BeLiveShortVideo', :path => '../framework/BeLiveShortVideo' # path to podspec file
```

## Carthage

`ShortVideoSDK` is distributed via `xcframework` which is recommended by Apple since WWDC 2019

### Step 1: Add frameworks
Drag `BeLiveCore.xcframework` and `BeLiveShortVideo.xcframework` into **Frameworks, Libraries, and Embeded Contents** and Select **Embeded & Sign**  

### Step 2: Update `Cartfile` 
You may encounter the build error or get crash: `dyld: Symbol not found` on Xcode 12 and above when using `xcframework` . Here is [workaround](https://github.com/Carthage/Carthage/blob/master/Documentation/Xcode12Workaround.md) to fix this issue

- Create file `carthage.sh` in your root project folder with this content

```ruby
# carthage.sh
# Usage example: ./carthage.sh build --platform iOS

set -euo pipefail
 
xcconfig=$(mktemp /tmp/static.xcconfig.XXXXXX)
trap 'rm -f "$xcconfig"' INT TERM HUP EXIT
 
# For Xcode 12 make sure EXCLUDED_ARCHS is set to arm architectures otherwise
# the build will fail on lipo due to duplicate architectures.
 
CURRENT_XCODE_VERSION="$(xcodebuild -version | grep "Xcode" | cut -d' ' -f2 | cut -d'.' -f1)00"
CURRENT_XCODE_BUILD=$(xcodebuild -version | grep "Build version" | cut -d' ' -f3)

echo "EXCLUDED_ARCHS__EFFECTIVE_PLATFORM_SUFFIX_simulator__NATIVE_ARCH_64_BIT_x86_64__XCODE_${CURRENT_XCODE_VERSION}__BUILD_${CURRENT_XCODE_BUILD} = arm64 arm64e armv7 armv7s armv6 armv8" >> $xcconfig
 
echo 'EXCLUDED_ARCHS__EFFECTIVE_PLATFORM_SUFFIX_simulator__NATIVE_ARCH_64_BIT_x86_64__XCODE_'${CURRENT_XCODE_VERSION}' = $(EXCLUDED_ARCHS__EFFECTIVE_PLATFORM_SUFFIX_simulator__NATIVE_ARCH_64_BIT_x86_64__XCODE_$(XCODE_VERSION_MAJOR)__BUILD_$(XCODE_PRODUCT_BUILD_VERSION))' >> $xcconfig
echo 'EXCLUDED_ARCHS = $(inherited) $(EXCLUDED_ARCHS__EFFECTIVE_PLATFORM_SUFFIX_$(EFFECTIVE_PLATFORM_SUFFIX)__NATIVE_ARCH_64_BIT_$(NATIVE_ARCH_64_BIT)__XCODE_$(XCODE_VERSION_MAJOR))' >> $xcconfig

# build all libraries for distribution
# https://github.com/Carthage/Carthage/issues/2845
echo 'BUILD_LIBRARY_FOR_DISTRIBUTION=YES' >> $xcconfig

export XCODE_XCCONFIG_FILE="$xcconfig"
carthage "$@"
```
- Add run permission to this file by `chmod +x carthage.sh`
- Instead of running `carthage` command, we'll change to run `carthage.sh`. Eg: `sh carthage.sh update --platform iOS --no-use-binaries --use-xcframeworks`

# ShortVideoView
`ShortVideoView` is a Ul entry point and designed to display a list of videos in many view types: vertical grid, horizotal grid and carousel. When a video item is clicked, it opens the main player and play the selected video.
Adding a `ShortVideoView` is as simple as a normal other view, can be done via Storyboard or code.

```swift
let shortVideoView = ShortVideoView(frame: self.view.bounds)
```
When running, `ShortVideoView` loads the video data from a `ShortVideoSource` which is defined by a `ShortVideoConfiguration`. This config defines the layout
type and provides some customization for UI. It also controis some basic player behaviors.

## Grid Layout
Grid layout supports vertical (default) and horizontal scroll direction
```swift
let layout = ShortVideoGridLayout.default
let configuration = ShortVideoConfiguration()
configuration.layout = layout
.
.
.
shortVideoView.configuration = configuration
```

You can change the scroll direction, disable scroll, number of columns/row via the layout config
```swift
layout.scrollDirection = .vertical
layout.numberOfColumns = numberOfColumsRows
.
.
.
layout.scrollDirection = .horizontal
layout.numberOfRows = numberOfColumsRows
```
Other attributes can be customized with grid layout:
- Background color
- Spacing between items

### Grid Item
To change UI of grid item, ShortVideoSDK supports below attributes:
- Item ratio (width/height)
- Corner radius
- Background color
- Play button: if you want to hide it, simply set it to `nil`
- Thumbnail scaling mode

## Carousel Layout
```swift
let layout = ShortVideoCarouselLayout.default
let configuration = ShortVideoConfiguration()
configuration.layout = layout
.
.
.
shortVideoView.configuration = configuration
```
You can disable scroll, update item configuration, next/prev button. Below is attributes you can customize:
- Background color
- Previous button: if you want to hide it, simply set it to `nil`
- Next button: if you want to hide it, simply set it to `nil`

### Carousel Item 
To change UI of carousel item, `ShortVideoSDK` supports below attributes:
- Background color
- Corner radius
- Title
- Progress
- Play/pause button
- Mute/unmute button
- Fullscreen button
- Share button
- Thumbnail scale mode
- Video scale mode

## Detail Player
When video item is clicked, a detail video screen will open to play this video. This screen supports products view, video seeking, PIP. Detail Player is a internal controller, we don't allow to change it but you can config UI and player behavior of this screen via `configuration.detailConfiguration`. If you don't want this screen show up, set `configuration.detailConfiguration = nil`
Below is list of attributes can be edit:
- Allow scroll
- Background color
- Previous button: if you want to hide it, simply set it to `nil`
- Next button: if you want to hide it, simply set it to `nil`
- Close button: if you want to hide it, simply set it to `nil`

### Detail Player Item
To change UI of detail player item, `ShortVideoSDK` supports below attributes:
- Background color
- Corner radius
- Title
- Progress seekbar
- Play/pause button
- Mute/unmute button
- PIP button
- Share button
- Thumbnail scale mode
- Video scale mode

### Product Item
A list of products can be attached to a video item and it will be displayed in the detail. Each product item might contain a Shop CTA (Call to action) button which opens the product URL in an external browser (you can override this behavior). The product item UI can also be customized:
- Background color
- Corner radius
- Title
- CTA button: Text, Color, Image
- Thumbnail scale mode

### PIP
The detail player supports play a video in PIP mode. You can disable this feature by hidding the PIP button by set it to `nil`.

## Player behaviors
Player behavior defines a set of behaviors when video item changed. `ShortVideoSDK` supports 2 behaviors is `autoplay` and `Player complete action` . Default config is following the setting on CMS. If set, it'll override the setting on CMS. For more detail, please read the swift documents.

### Playlist autoLoop
For Playlist video source, there will be a flag called autoLoop which is set by the CMS.
For other sources (Default/Single), this flag is always off.
If this flag is on, the whole playlist will be repeated (depends on the SDK's autoPlay flag and action on player complete).
If this flag is off, the playlist will behave same as other video sources.
Note: The autoLoop is applied to the Playlist but the autoPlay and action on player complete are applied to every single video.
It means, even the autoLoop is on, the playlist might not be repeated if a video in the list is not playing or the action is not configured to play the next item.

### Auto play
By default, the video autoPlay (Carousel and Main player) flag is controlled by the CMS.
You can override this behavior by explicitly setting it in the SDK.
```swift
configuration.player.autoPlay = .none
```
When the auto play flag is enabled/disabled by the SDK, the one from CMS will be ignored.

### Player complete action
You can control the behavior when the player finish a playback.
```swift
configuration.player.playerCompleteAction = .stop
```
There are 4 options that you can set:
- `None`: This is the default value.
  The behavior now will be controlled by the playlist autoLoop.
  If the autoLoop value is on, then it will behave like `PlayNextItem`,
  except the player will repeat the playlist after playing the last video.
  If the autoLoop value is off, then it will behave like `Stop`
- `Stop`: The player will always stop the video after playing it. The playlist's autoLoop is ignored.
- `PlayNextItem`: The player will always play the next video in the list. The playlist's autoLoop is ignored.
  If the current video is the last item, the player will stop.
- `Loop`: The player will always loop the current video. The playlist's autoLoop is ignored.
## ShortVideoSource
`ShortVideoSDK` supports 3 types of video sources
- All: Show all videos
- Playlist: All videos of a playlist
- VideoId: A single video identifier

## Play & Pause
User can pause and resume the video by calling the following APIs:
```swift
shortVideoView.pause()
//
shortVideoView.play()
```

## Delegates
`ShortVideoSDK` provides several delegates to user to listen events when tapping on item, product CTA or error occur.
```swift
public protocol BLShortVideoErrorDelegate: AnyObject {
    /// Callback when any error occur
    /// - Parameters:
    ///   - view: ShortVideoView
    ///   - error: ShortVideoError error
    func shortVideoView(view: ShortVideoView, onError error: ShortVideoError)
}

public protocol BLShortVideoItemDelegate: AnyObject {
    /// Callback when short video item is clicked
    /// - Parameters:
    ///   - view: ShortVideoView
    ///   - videoId: videoId
    func shortVideoView(view: ShortVideoView, itemDidClick videoId: String)
}

public protocol BLShortVideoShareItemDelegate: NSObject {
    /// Callback when share button is clicked
    /// - Parameters:
    ///   - videoId: videoId
    ///   - handle: if handle completion is `true`, nothing happen otherwise it'll show default system share
    func shortVideoItemDidShare(videoId: String, handle: @escaping (Bool) -> Void)
}

public protocol BLShortVideoProductItemCTADelegate: AnyObject {
    /// Callback when product item call to action button is clicked
    /// - Parameter product: product object
    func shortVideoViewProductItemCTADidClick(product: BLShortVideoProduct)
}
```