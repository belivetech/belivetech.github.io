---
title: Short Video SDK Guide for Web
redirect_from:
  - /getting-started.html
---

## Table of contents

* [Getting started](#getting-started)
- [Step-by-step guide for Short Video SDK](#step-by-step-guide-for-short-video-sdk)
  - [Step 1: Set up Short Video SDK](#step-1-set-up-short-video-sdk)
  - [Step 2: Insert layout tag in your html](#step-2-insert-video-tag-in-your-html)
  - [Step 3: Create a BeLiveShortVideoSdk instance](#step-3-create-a-viewer-instance)
  - [Step 4: Create Component Single / Grid / Carousel](#step-4-create-component-single-grid-carousel)
  
- [Advanced guide](#advanced-guide)
  - [Handle Events](#handle-events)
  - [Call a function](#call-a-function)
- [API](#api)
  - [BeLive Short Video SDK methods](#belive-short-video-sdk-methods)
  - [Single Video SDK methods](#single-video-sdk-methods)
  - [Playlist (Grid/Carousel) Methods](#playlist-grid-carousel-methods)
- [Events](#events)
    - [BeLive Short Video SDK Events](#belive-short-video-sdk-events)
    - [Single Video Events](#single-video-events)
    - [Playlist Events](#playlist-events)

## Getting started

Download BELIVE SHORT VIDEO SDK JS plugin from provided link

### Step-by-step guide for Short Video SDK

### Step 1: Set up Short Video SDK

**Import static files in html**

```html
<head>
   <meta charset="UTF-8">
   <title>Title</title>
   <script src="./belive-short-video-sdk.js" type="text/javascript"></script>
</head>
```

### Step 2: Insert layout tag in your html

```html
<!-- Single Video (Set the height of layout) -->
<div class="YOUR_SINGLE_CLASS" style="height: 480px;"></div> 

<!-- Grid Playlist -->
<div class="YOUR_GRID_CLASS"></div>

<!-- Carousel Playlist -->
<div class="YOUR_CAROUSEL_CLASS"></div>
```

### Step 3: Create a BeLiveShortVideoSdk instance

```javascript
let shortVideoSdkInstance = new BeliveShortVideoSdk.ShortSdk(
	"{YOUR__API_END_POINT}",
    "{YOUR__BELIVE_LICENSE_KEY}",
    {}, // YOUR Icons for single video | OPTIONAL
    {} // YOUR Icons for playlist | OPTIONAL
)

sdkShortInstance.initShortSdk(() => {
    // Success callback
    // Your code here for create the single video, grid playlist or carousel playlist
    ....
}, (error) => {
    // Error callback
})
```

### Step 4: Create Component Single, Grid, Carousel

```javascript
sdkShortInstance.initShortSdk(() => {
    // Success callback
    // Create the single video
    const singleVideoLayout = document.querySelector("{YOUR_SINGLE_CLASS}")
    const singleVideoInstance = sdkShortInstance.createSingleVideo("YOUR_SHORT_VIDEO_ID", singleVideoLayout)

     // Create the Grid playlistplaylist
    const gridLayout = document.querySelector("{YOUR_GRID_CLASS}")
    const gridPlaylistInstance = sdkShortInstance.createGridPlaylist("YOUR_PLAYLIST_ID", gridLayout)

     // Create the carousel playlist
    const carouselLayout = document.querySelector("{YOUR_CAROUSEL_CLASS}")
    const carouselPlaylistInstance = sdkShortInstance.createCarouselPlaylist("YOUR_PLAYLIST_ID", carouselLayout)
}, (error) => {
    // Error callback
})
```

### Advanced guide

#### Handle Events
```javascript
singleVideoInstance.on("SHORT_SDK__PLAYING", () => {
	// Your logic here
})
```

#### Call a function
```javascript
singleVideoInstance.pauseVideo()
```

## API

### BeLive Short Video SDK methods

| Method                     | Description                                                                   |
| -------------------------- | ----------------------------------------------------------------------------- |
| `on` | <code>on(event, callback): void</code> Listen for events                                            |

**on**

```javascript
on(event, callback): void
```

Method to listen event.

**Parameters**

* `event (string)`

> Check all available events in the Events Section.

* `callback (function)`

### Single Video SDK methods

| Method                     | Description                                                                   |
| -------------------------- | ----------------------------------------------------------------------------- |
| `pauseVideo` | Pause the current video                                         |
| `playVideo` | Play the current video                                         |
| `muteVideo` | Mute/Un-mute the current video                                         |
| `pauseOtherVideos` | Pause the other videos                                       |
| `shareVideo` | Trigger to share the video                                       |
| `generateTheShareUrl` | Get the share URL                                    |
| `switchToFullscreen` | Switch to fullscreen mode                                       |
| `exitFullscreenMode` | Exit fullscreen mode                               |
| `switchToPip` | Switch to Picture in Picture (PiP) mode                                        |

**pauseVideo**

```javascript
pauseVideo(): void
```

Method to pause the current video

**playVideo**

```javascript
playVideo(): void
```

Method to play the current video

**muteVideo**

```javascript
muteVideo(muted = false): void
```

Method to play the current vide

**Parameters**

* `muted (boolean)`

> True: muted | False: un-muted


**pauseOtherVideos**

```javascript
pauseOtherVideos(): void
```

Method to pause the other videos

**shareVideo**

```javascript
shareVideo(): void
```

Method to share the current video

**generateTheShareUrl**

```javascript
generateTheShareUrl(): string
```

Method to get the share URL

**switchToFullscreen**

```javascript
switchToFullscreen(): void
```

Method to switch to Fullscreen mode

**exitFullscreenMode**

```javascript
exitFullscreenMode(): void
```

Method to exit Fullscreen mode

**switchToPip**

```javascript
switchToPip(): void
```

Method to switch to Picture in Picture (PiP) mode

### Playlist Grid, Carousel Methods

| Method                     | Description                                                                   |
| -------------------------- | ----------------------------------------------------------------------------- |
| `goToBackSlide` | Go to previous slide                                       |
| `goToNextSlide` | Go to next slide                                         |
| `moveSlide` | Move slide to index                                        |

**goToBackSlide**

```javascript
goToBackSlide(): void
```
Method to move the playlist to previous slide

**goToNextSlide**

```javascript
goToNextSlide(): void
```
Method to move the playlist to next slide

**moveSlide**

```javascript
moveSlide(index): void
```
Method to move the playlist to selected slide


**Parameters**

* `index (number)`

## Events

### BeLive Short Video SDK Events

| Method                | Description                                  | Value Return                                                                 |
| --------------------- | -------------------------------------------- | ---------------------------------------------------------------------------- |
| `SHORT_SDK__RESIZE`   | Event when the layout size was changed | -                                                         |

### Single Video Events

| Method                | Description                                  | Value Return                                                                 |
| --------------------- | -------------------------------------------- | ---------------------------------------------------------------------------- |
| `SHORT_SDK__PAUSED`   | Event when video was paused | -                                                         |
| `SHORT_SDK__PLAYING`   | Event when video is playing | -                                                         |
| `SHORT_SDK__LOADED_META_DATA`   | Event when video loaded the metadata | -                                                         |
| `SHORT_SDK__MUTED`   | Event when video was muted | -                                                         |
| `SHORT_SDK__UN_MUTED`   | Event when video was un-muted | -                                                         |
| `SHORT_SDK__VIDEO_ENDED`   | Event when video was ended | -                                                         |
| `SHORT_SDK__VIDEO_STATE_BTN_CLICKED`   | Event when user clicked on Play/Pause button | -                                                         |
| `SHORT_SDK__VIDEO_SOUND_BTN_CLICKED`   | Event when user clicked on Mute/UnMute button | -                                                         |
| `SHORT_SDK__HOVER_VIDEO`   | Event when user hover on video | -                                                         |
| `SHORT_SDK__SHARE_BTN_CLICKED`   | Event when user clicked on Share button | -                                                         |
| `SHORT_SDK__OPEN_FULLSCREEN`   | Event when switched to Fullscreen mode | -                                                         |
| `SHORT_SDK__EXIT_FULLSCREEN`   | Event when exit Fullscreen mode | -                                                         |
| `SHORT_SDK__OPEN_PIP`   | Event when switched to PiP mode | -                                                         |
| `SHORT_SDK__CANCEL_SHARE`   | Event when user clicked to Cancel Share button | -                                                         |
| `SHORT_SDK__CLOSE_SHARED_POPUP`   | Event when user closed the Shared Popup | -                                                         |
| `SHORT_SDK__FETCHED_DATA`   | Event when data was fetched | -                                                         |
| `SHORT_SDK__INITIAL_FAILED`   | Event when initial component failed | -                                                         |
| `SHORT_SDK__TRACKING_CTA`   | Event when tracking the CTA action | -                                                         |
| `SHORT_SDK__TRACKING_SHARE`   | Event when tracking the Share action | -                                                         |
| `SHORT_SDK__TRACKING_VIEW`   | Event when tracking the View action | -                                                         |
| `SHORT_SDK__REPEAT`   | Event when video was repeated | -                                                         |
| `SHORT_SDK__PAUSED_OTHER_VIDEOS`   | Event when paused other videos | -                                                         |
| `SHORT_SDK__CTA_CLICKED`   | Event when user clicked on item CTA | -                                                         |

### Playlist Events

| Method                | Description                                  | Value Return                                                                 |
| --------------------- | -------------------------------------------- | ---------------------------------------------------------------------------- |
| `SHORT_SDK__NEXT_SLIDE`   | Event when playlist go to next slide | -                                                         |
| `SHORT_SDK__BACK_SLIDE`   | Event when playlist go to back slide | -                                                         |
| `SHORT_SDK__SLIDE_CHANGED`   | Event when playlist changed the slide | -                                                         |
| `SHORT_SDK__LOAD_MORE_DATA`   | Event when playlist load more data | -                                                         |
