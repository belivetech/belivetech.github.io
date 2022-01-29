---
layout: article
---


# BeLive Interactive Live Streaming SDK

BeLive SDK is an end-to-end live streaming solution with customizable UI and content management system which can be integrated in your existing iOS, Android and Web apps with few lines of code. SDK provides fully functional live shopping capability.

> Throughout this document we will refer to host / broadcaster as source of live stream while viewer or audience as consumer/watcher of live or recorded stream


BeLive SDK provides following core features with flexibility to extend and add-on more features:

* Start a live stream using iOS, Android or professional broadcasting software such as OBS
* Watch live stream on any iOS and Android device. In addition, we also support playback on web browser
* Support RTMP Streaming (Industry standard for transmitting video over network)\_
* Record and encode live stream (HLS or mp4 format)
* Enable Beauty filter for enhancing the live stream experience
* Live chat for interactivity during live stream
* Per minute stream statistics (live view count and likes)
* Live shopping feature with unlimited shop items / products
* Multiple resolutions supports such as SD and HD.
* Content Management System to manage streams and live shopping experience

BeLive SDK lets you focus on building your own custom application and audience experience with our cutting-edge SDK. With BeLive SDK, you don't need to manage infrastructure or develop and configure components of your live stream workflows, to be secure, reliable, and cost effective.

### SDK Package

SDK Package contains following core components:

#### For Native Mobile Apps

* Android SDK with sample code
* iOS SDK with Sample Code

#### For Web Apps

* Javascript SDK for custom UI integration in your existing Web App

#### For Content Managers

* Fully functional Content Management system

#### Customizable UI

* Fully customizable UI with SDK core functions.

## Streaming Configuration

### Stream ingest

Codec : BeLive SDK supports H.264 for video and AAC (LC) for audio

#### Resolution/Bitrate/FPS <a href="#streaming-config-settings-res-bitrate-fps" id="streaming-config-settings-res-bitrate-fps"></a>

BeLive SDK provides following recommended settings for SD and HD quality. API for both SD and HD qualities are readily available.

| Parameter             | Quality (SD) (540x960)   | Quality (HD) (720x1280)  |
| --------------------- | ------------------------ | ------------------------ |
| **Bitrate**           | 1000 Kbps                | 1500 Kbps                |
| **FPS**               | 20                       | 24                       |
| **Keyframe Interval** | Adjustable (2.0 default) | Adjustable (2.0 default) |

### Network Requirements <a href="#streaming-config-network" id="streaming-config-network"></a>

You must have a stable internet connection that can maintain an adequate, constant upload stream. We strongly suggest to allocate 50% more bandwidth than the minimum required. For SD Stream of 1000 Kbps, at-least 1500 kbps (1.5 Mbps) upload speed is required because overhead is added to compensate for the bitrate fluctuations.

### Guides: Jump right in

Follow our handy guides to get started on the basics as quickly as possible:

[sdk-guide-for-android.md](guides/sdk-guide-for-android.md)

[sdk-guide-for-ios.md](guides/sdk-guide-for-ios.md)

[sdk-guide-for-web.md](guides/sdk-guide-for-web.md)

[live-shopping-guide.md](guides/live-shopping-guide.md)
