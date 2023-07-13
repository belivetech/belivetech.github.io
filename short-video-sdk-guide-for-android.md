---
title: Short Video SDK Guide for Android
redirect_from:
  - /getting-started.html
---

## Getting Started
Short Video is an external Belive SDK module but it can be used independently. Please check the document for the Belive SDK setup first.

The Short Video requires a Belive core module, Belive common module and other 3rd libraries to work normally.

`build.gradle` dependencies:
```gradle
    // sdk aar files & required dependencies
    implementation (name:'belive-core-release', ext:'aar')
    implementation (name:'belive-common-release', ext:'aar')
    implementation (name:'belive-shortvideo-release', ext:'aar')
    implementation 'io.insert-koin:koin-android:3.4.0'
    implementation 'androidx.media3:media3-exoplayer:1.0.2'
    implementation 'androidx.media3:media3-ui:1.0.2'
    implementation 'androidx.media3:media3-exoplayer-hls:1.0.2'
    implementation "com.squareup.retrofit2:retrofit:2.9.0"
    implementation "com.squareup.retrofit2:converter-gson:2.9.0"
    implementation "com.squareup.okhttp3:logging-interceptor:4.9.0"
    implementation 'com.github.bumptech.glide:glide:4.15.1'
    kapt 'com.github.bumptech.glide:compiler:4.15.1'
```
## ShortVideoView
ShortVideoView is a UI entry point and designed to display a list of videos in many layout types: vertical grid, horizontal grid and carousel.
When a video item is clicked, it opens the main player and play the selected video.

Adding a ShortVideoView is as simple as a normal Android View.
```xml
<sg.belive.lib.shortvideo.view.ShortVideoView
    android:id="@+id/short_video_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```
ShortVideoView loads the video data from a Video Source which is defined by a View Config.
This View Config defines the layout type and provides some customization for the UI. It also controls some basic player behaviors.

### Grid layout
Grid layout supports vertical (default) and horizontal.
```kotlin
        val cfBuilder = ViewConfig.Builder(this) // `this` is a context
        // set the video source (DefaultVideoSource, SingleVideoSource, PlaylistVideoSource)
        cfBuilder.videoSource(source)
        // Create the detail view config which is used to render the main player
        val detailCfBuilder = DetailViewConfig.Builder(this)
        val detailLoCf = ShortVideoDetailLayout.createDefaultConfig(this)
        detailCfBuilder.layoutConfig(detailLoCf)
        // Set the detail view config
        cfBuilder.detailViewConfig(detailCfBuilder.build())
        shortVideoView.init(cfBuilder.build())
```
You can change the Grid orientation to horizontal or update the span count (number of grid columns/rows)
```kotlin
        val cfBuilder = ViewConfig.Builder(this)
        cfBuilder.videoSource(source)
        val loCf = ShortVideoGridLayout.createDefaultConfig(this)
        loCf.scrollDirection = ScrollDirection.HORIZONTAL
        loCf.numberOfRows = 2
        cfBuilder.layoutConfig(loCf)
        shortVideoView.init(cfBuilder.build())
```
The player behavior config is set as below
```kotlin
        val playerCf = PlayerItemConfiguration().apply {
            enableAutoPlay()
            actionOnCompleted = PlayerCompleteAction.STOP
        }
        cfBuilder.playerConfig(playerCf)
```
#### UI Customization
For Grid layout, you can customize the below UI components/attrs:
- Item width and height ratio: This ratio is in String format ("16:9"). It is recommended to use a portrait ratio
- Video title text view
- Play button
- Item's corner radius
- Video thumbnail scale mode

```kotlin
        val gridLoCf = ShortVideoGridLayout.createDefaultConfig(this)
        gridLoCf.ratio = "16:9"
        gridLoCf.itemConfiguration?.let { cf ->
            cf.title = TitleConfiguration().apply {
                this.color = ContextCompat.getColor(context, R.color.bls_sv_video_item_title_color)
                this.backgroundRes = R.drawable.bls_sv_bg_carousel_video_item_title
                this.textSize =
                    context.resources.getDimension(R.dimen.bls_sv_video_item_title_size)
                this.font = Typeface.DEFAULT
                this.gravity = Gravity.START or Gravity.CENTER_VERTICAL
            }
            cf.playButton = ButtonViewItemConfiguration().apply {
                this.onStateIconRes = R.drawable.ic_play
            }
            cf.cornerRadius = 10F
            cf.imageScaleMode = ImageView.ScaleType.CENTER
        }
```

### Carousel layout
Carousel layout brings a video player feature to the ShortVideoView.
When using this layout, it requires a parent container and this container should be a ConstraintLayout or FrameLayout to avoid unwanted layout behaviors when switching from Fullscreen and Compact mode.
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/loContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <sg.belive.lib.shortvideo.view.ShortVideoView
        android:id="@+id/short_video_view"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_margin="50dp"
        app:layout_constraintDimensionRatio="9:16"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```
The View config is almost same as the Grid type, except the ShortVideoCarouselLayout is used for the layoutConfig.
```kotlin
        val cfBuilder = ViewConfig.Builder(this) // `this` is a context
        cfBuilder.videoSource(source)
        val carouselLoCf = ShortVideoCarouselLayout.createDefaultConfig(this)
        carouselLoCf.containerView = loContainer
        // Create the detail view config which is used to render the main player
        val detailCfBuilder = DetailViewConfig.Builder(this)
        val detailLoCf = ShortVideoDetailLayout.createDefaultConfig(this)
        detailCfBuilder.layoutConfig(detailLoCf)
        // Set the detail view config
        cfBuilder.detailViewConfig(detailCfBuilder.build())
        shortVideoView.init(cfBuilder.build())
```
This layout requires a non-null containerView which is the parent view of the ShortViewView in Compact mode.
In Fullscreen mode, the parent view will be the content view of the Activity (if the ShortVideoView's context is an Activity), otherwise it will be a top most root view.
You can override this behavior by manually setting a fullScreenContainerView to the View Config.
```kotlin
        val cfBuilder = ViewConfig.Builder(this) // `this` is a context
        cfBuilder.videoSource(source)
        val carouselLoCf = ShortVideoCarouselLayout.createDefaultConfig(this)
        carouselLoCf.containerView = loContainer
        carouselLoCf.fullScreenContainerView = target_fullscreen_container_view
```
#### Play and pause the video
User can pause and resume the video by calling the following APIs
```kotlin
shortVideoView.pause()
//
shortVideoView.play()
```
#### Scrolling and navigation buttons
The Carousel layout can be configured to disable the scrolling behavior. This might be useful when you want to navigate the video items manually.
```kotlin
        val carouselLoCf = ShortVideoCarouselLayout.createDefaultConfig(this)
        // Disable the scrolling
        carouselLoCf.allowScroll = false
```

You can also hide the navigation buttons (next & previous button) by setting their view config to null.
```kotlin
        val carouselLoCf = ShortVideoCarouselLayout.createDefaultConfig(this)
        // Hide the navigation button
        carouselLoCf.nextButton = null
        carouselLoCf.prevButton = null
```
#### UI Customization
For Carousel layout, you can customize the below UI components/attrs:
- Video title text view (the layoutGravity attr is ignored)
- Item's corner radius
- Progress bar
- Play/pause, mute, full screen, share button
- Video thumbnail scale mode
- Video scale mode
```kotlin
        val carouselLoCf = ShortVideoCarouselLayout.createDefaultConfig(this)
        carouselLoCf.itemConfiguration?.let { cf ->
            cf.title = TitleConfiguration()
            cf.playPauseButton = ButtonViewItemConfiguration().apply {
                this.onStateIconRes = R.drawable.ic_play
                this.offStateIconRes = R.drawable.ic_pause
            }
            cf.imageScaleMode = ImageView.ScaleType.CENTER
            cf.videoScaleMode = BlsPlayerScaleMode.DEFAULT
        }
```

### Main player
The ShortVideo module provides a main player as an internal screen (activity) that can be opened when clicking on a ShortVideoView's video item.
This screen includes a video player with many functions and a product list.
The UI and player behavior can be configured when setting up the Grid/Carousel layout.
```kotlin
        val gridViewCfBuilder = ViewConfig.Builder(this)
        // Create the detail view config
        val detailCfBuilder = DetailViewConfig.Builder(this)
        detailCfBuilder.layoutConfig(ShortVideoDetailLayout.createDefaultConfig(this))
        detailCfBuilder.playerConfig(PlayerItemConfiguration())
        // Set the detail view config
        gridViewCfBuilder.detailViewConfig(detailCfBuilder.build())
        shortVideoView.init(gridViewCfBuilder.build())
```
#### Picture in Picture (PiP)
The main player supports play a video in PiP mode on devices that support Pip (Android 8+).
You can disable this feature by hiding the PiP button.
```kotlin
        val detailCfBuilder = DetailViewConfig.Builder(this)
        val detailLoCf = ShortVideoDetailLayout.createDefaultConfig(this)
        detailLoCf.itemConfiguration?.let { cf ->
            cf.pipButton = null
        }
```
#### Product list
A list of products can be attached to a video item and it will be displayed in the main player.
Each product item might contain a Shop CTA button which opens the product URL in an external browser (you can override this behavior).
The product item UI can also be customized.

#### Scrolling and navigation buttons
Same as the Carousel layout, the main player can be configured to disable the scrolling behavior.
```kotlin
        val detailLoCf = ShortVideoDetailLayout.createDefaultConfig(this)
        // Disable the scrolling
        detailLoCf.allowScroll = false
```

You can also hide the navigation buttons (next & previous button) by setting their view config to null.
```kotlin
        val detailLoCf = ShortVideoDetailLayout.createDefaultConfig(this)
        // Hide the navigation button
        detailLoCf.nextButton = null
        detailLoCf.prevButton = null
```

#### UI Customization
For main player layout, you can customize the below UI components/attrs:
- Video title, duration text view (the layoutGravity attr is ignored)
- Item's corner radius
- Seek bar
- Play/pause, mute, pip, share, close button
- Video thumbnail scale mode
- Video scale mode
- Product item view.

For product item layout, you can customize the below UI components/attrs:
- Product title text view (the layoutGravity attr is ignored)
- Item's corner radius/background
- Shop CTA text color and icon resource.
- Shop CTA text content. If set, it will override the remote value.
- Product thumbnail scale mode

```kotlin
        val detailLoCf = ShortVideoDetailLayout.createDefaultConfig(this)
        detailLoCf.itemConfiguration?.let { cf ->
            cf.closeButton = ButtonViewItemConfiguration().apply {
                onStateIconRes = R.drawable.ic_close_main_player
            }
            cf.muteButton = ButtonViewItemConfiguration().apply {
                onStateIconRes = R.drawable.ic_unmute
                offStateIconRes = R.drawable.ic_muted
            }
            cf.seekBar = SeekViewItemConfiguration().apply {
                thumbIconRes = R.drawable.ic_seek_thumb
                progressRes = R.drawable.progress_seek_bar
            }
            cf.productItem = ShortVideoDetailLayout.ProductItemConfiguration().apply {
                this.title = TitleConfiguration().apply {
                    this.color =
                        ContextCompat.getColor(
                            context,
                            R.color.product_item_title_color
                        )
                    this.textSize =
                        resources.getDimension(R.dimen.product_item_title_size)
                    this.font = Typeface.DEFAULT
                    this.gravity = Gravity.START
                }
                this.cornerRadius = 10F
                this.backgroundColor = ContextCompat.getColor(context, R.color.white)
                this.ctaTextColor =
                    ContextCompat.getColor(
                        context,
                        R.color.color_shop_cta_button
                    )
                this.ctaIconRes = R.drawable.ic_product_cta_nav
            }
        }
```
### Video source
#### Default
The default video source will fetch all the available videos data from server.
To create a default source, use the below API
```kotlin
        ShortVideoSource.default()
```
The default source will load the data in pagination. When scrolling the list to the end, it will automatically load the next page until reaching the end of the source.

#### Playlist
The playlist video source will fetch all the available videos from a specific playlist by an alias.
To create a playlist source, use the below API
```kotlin
        ShortVideoSource.playlist(playlistAlias)
```

#### Single
The single video source will fetch a single video by a video id.
To create a single source, use the below API
```kotlin
        ShortVideoSource.singleVideo(videoId)
```

### Listeners
#### Error listener
When an error happens, the ShortVideo module triggers an error callback with a code and a message. To listen to the error, set a `BlsShortVideoErrorListener` to the ShortVideoView

```kotlin
        shortVideoView.setErrorListener(object : BlsShortVideoErrorListener {
            override fun onShortVideoError(code: Int, msg: String?) {
            }
        })
```
The error code can be found in the `BlsShortVideoErrorCode` and the message will describe the error

#### Video Item Click listener
When user clicks on a video item, it will open the main player and trigger a callback.
```kotlin
        shortVideoView.setVideoItemClickedListener(object : BlsShortVideoItemClickListener {
            override fun onShortVideoItemClicked(videoId: String) {
            }

        })
```
The implementation of the listener will not affect the flow, it is used mainly for analytics purpose.

#### Main Player Action listener
This listener is different to other listeners that it is bound to an singleton handler.
You have to take care of the usage of this listener or a memory leak can occur.
You should not implement this interface as an anonymous class in the activity context.
A recommended approach is to create a class and implement this interface to avoid any strong reference to the activity or fragment.

```kotlin
class MainPlayerAction : BlsShortVideoMainPlayerActionListener {
    override fun onProductItemClicked(
        context: Context, item: ShortVideoProductModel
    ): Boolean {
        context.openBrowser(item.productUrl.orEmpty())
        return true
    }

    override fun onShareVideo(context: Context, videoId: String): Boolean {
        context.shareText(videoId)
        return true
    }
}
```
The `onProductItemClicked` is triggered when a shop CTA button is clicked. You can return true in this API to consume the event and stop the flow.
If you return false, the SDK will continue handling the event (if any).

The `onShareVideo` is triggered when a share button is clicked (in both Carousel and Main player). You can return true in this API to consume the event and stop the flow.
If you return false, the SDK will continue handling the event. The default handler is to open the system share dialog.

You have to reset the listener when unused
```kotlin
    binding.shortVideoView.releaseMainPlayerActionListener(this)
```

### View configuration
The basic configuration for a view item are `size` and `backgroundRes`. If the `size` is not set, a LayoutParams with `WRAP_CONTENT` for both width and height will be generated.

#### Text View
A TextView can be customized using a `TitleConfiguration`. This configuration provides some basic attributes:
- color: map to the TextView's textColor
- font: map to the TextView's typeface
- maxLines: map to the TextView's maxLines. The default value is 1
- layoutGravity: map to the LayoutParams's gravity. The default value is Gravity.BOTTOM
- gravity: map to the TextView's gravity. The default value is Gravity.START
- textSize: map to the TextView's textSize

#### Image Button
A ImageButton can be customized using a `ButtonViewItemConfiguration`. This configuration provides some basic attributes:
- onStateIconRes: The initialize state icon of the ImageButton
- offStateIconRes: The next state icon of the button. If a button has 1 state only (like the share, close button), this attribute is ignored.

#### Progress Bar
A ProgressBar can be customized using a `ProgressViewItemConfiguration`. This configuration provides some basic attributes:
- normalColor: The color of the progress background (not the background of the whole view) (map to the `progressBackgroundTintList`)
- progressColor: The color of the progress (map to the `progressTintList`)
- normalTintMode: The tintMode of the progress background (map to the `progressBackgroundTintMode`)
- progressTintMode: The tintMode of the progress background (map to the `progressTintList`)
- progressRes: Map to the `progressDrawable`. If set, the above attributes will be ignored.

#### Seek Bar
A SeekBar can be customized using a `SeekViewItemConfiguration`. This configuration provides some basic attributes:
- normalColor: The color of the progress background (not the background of the whole view) (map to the `progressBackgroundTintList`)
- progressColor: The color of the progress (map to the `progressTintList`)
- normalTintMode: The tintMode of the progress background (map to the `progressBackgroundTintMode`)
- progressTintMode: The tintMode of the progress background (map to the `progressTintList`)
- progressRes: Map to the `progressDrawable`. If set, the above attributes will be ignored.
- thumbColor: The color of the thumb (map to the `thumbTintList`)
- thumbTintMode: The tintMode of the thumb (map to the `thumbTintMode`)
- thumbIconRes: Map to the `thumb` attr. If set, the above thumb attributes will be ignored.

#### Image/Video scale mode
Some of the layout items support image and video scale mode.
The Image scale mode can be set from `ImageView.Scale`.
The default value is `ImageView.ScaleType.CENTER_CROP`

The Video Scale mode can be set from `BlsPlayerScaleMode`. Here are the support modes:
- DEFAULT: Let the SDK choose the best scale mode bases on video/device orientation. This is default and recommended mode.
- FIT: Make the video fit the view and keep the ratio in at least 1 dimension (width or height)
- FILL: Zoom the video and do not keep the ratio to fit the view in both width and height
- FIXED_WIDTH: Make the video fit the view width and keep the ratio
- FIXED_HEIGHT: Make the video fit the view height and keep the ratio
- ZOOM: Zoom the video and keep the ratio to fit the view in both width and height

### Player configuration
#### Playlist autoLoop
For Playlist video source, there will be a flag called autoLoop which is set by the CMS.
For other sources (Default/Single), this flag is always off.

If this flag is on, the whole playlist will be repeated (depends on the SDK's autoPlay flag and action on player complete).

If this flag is off, the playlist will behave same as other video sources.

Note: The autoLoop is applied to the Playlist but the autoPlay and action on player complete are applied to every single video.
It means, even the autoLoop is on, the playlist might not be repeated if a video in the list is not playing or the action is not configured to play the next item.

#### Auto play
By default, the video autoPlay (Carousel and Main player) flag is controlled by the CMS.
You can override this behavior by explicitly setting it in the SDK.

```kotlin
        val playerCf = PlayerItemConfiguration().apply {
            enableAutoPlay()
            // 
            disableAutoPlay()
        }
```
When the auto play flag is enabled/disabled by the SDK, the one from CMS will be ignored.

#### Action on player complete
You can control the behavior when the player finish a playback.
```kotlin
        val playerCf = PlayerItemConfiguration().apply {
            actionOnCompleted = PlayerCompleteAction.STOP
        }
```
There are 4 options that you can set:
- NONE: This is the default value.
  The behavior now will be controlled by the playlist autoLoop.
  If the autoLoop value is on, then it will behave like `PLAY_NEXT_ITEM`,
  except the player will repeat the playlist after playing the last video.
  If the autoLoop value is off, then it will behave like `STOP`
- STOP: The player will always stop the video after playing it. The playlist's autoLoop is ignored.
- PLAY_NEXT_ITEM: The player will always play the next video in the list. The playlist's autoLoop is ignored.
  If the current video is the last item, the player will stop.
- REPEAT: The player will always loop the current video. The playlist's autoLoop is ignored.
