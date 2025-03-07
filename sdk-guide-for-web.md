---
title: SDK Guide for Web
redirect_from:
  - /getting-started.html
---

**Good to know:** BeLive SDK for Web only supports video playback for live and recorded streams along with interactive elements.

## Table of contents

* [Getting started](#getting-started)
- [Step-by-step guide for viewer to watch stream](#step-by-step-guide-for-viewer-to-watch-stream)
  - [Step 1: Set up SDK](#step-1-set-up-sdk)
  - [Step 2: Insert video tag in your html](#step-2-insert-video-tag-in-your-html)
  - [Step 3: Create a viewer instance](#step-3-create-a-viewer-instance)
  - [Step 4: Add SDK event listeners](#step-4-add-sdk-event-listeners)
  - [Step 5: Login to identify a user](#step-5-login-to-identify-a-user)
  - [Step 6: Watch a livestream video or a recorded video](#step-6-watch-a-livestream-video-or-a-recorded-video)
  - [Step 7: Real-time messaging](#step-7-real-time-messaging)
  - [Step 8: Destroy live video and chat room](#step-8-destroy-live-video-and-chat-room)
- [Advanced guide](#advanced-guide)
  - [Anti-spamming messages](#anti-spamming-messages)
  - [Read history chat from a recorded stream](#read-history-chat-from-a-recorded-stream)
- [API](#api)
  - [General methods](#general-methods)
  -  [RTM Threshold Methods](#rtm-threshold-methods)
  - [Video Player Methods](#video-player-methods)
  - [Models](#models)
    - [RtmError](#rtmerror)
  - [SDK events](#sdk-events)
    - [General events emitted during sdk usage](#general-events-emitted-during-sdk-usage)
    - [Video Events](#video-events)
  - [Error codes](#error-codes)
    - [Status response](#status-response)
    - [General error codes](#general-error-codes)
    - [RTM error codes](#rtm-error-codes)

## Getting started

We provide support through a script and link tag as well as through an npm module.

Download SDK JS plugin from provided link

Or Install from **NPM**:

```
npm install --save @belive-tech/sdk
```

### Step-by-step guide for viewer to watch stream:

### Step 1: Setup SDK

**Import static files in html**

```html
<head>
   <meta charset="UTF-8">
   <title>Title</title>
   <link href="./belive-sdk.min.css" rel="stylesheet">
   <script src="./belive-sdk.umd.js" type="text/javascript"></script>
</head>
```

**Or import from `node_modules` after install from NPM**

```javascript
import * as BeliveSdk from "@belive-tech/sdk"
import "@belive-tech/sdk/dist/belive.min.css"
```

### Step 2: Insert video tag in your html

```html
<video id="playerViewWrapper"
	autoplay
	class="video-js video-center"
	oncontextmenu="return false;"
	preload="auto"
	muted
	playsinline
	disablePictureInPicture
	width="640"
	height="480">
	  <p class="vjs-no-js"> To view this video please enable JavaScript, 
      and consider upgrading to a web browser that 
      		<a href="<https://videojs.com/html5-video-support/>" 
      			target="_blank">supports HTML5 video</a>
	</p>
</video>
```

### Step 3: Create a viewer instance

```javascript
let sdkViewerInstance = new BeliveSdk.Viewer(
   { 
  // Create View instance with your config
	key: "key_place_holder",
	endpoint: "API_END_POINT",
	rtmService: "agora", // rtm service agora or belive
   }, 
   "playerViewWrapper"
);
```

### Step 4: Add SDK event listeners

```javascript
sdkViewerInstance.on("LOST_NETWORK", () => {
	//Callback whenever network lost
})

sdkViewerInstance.on("RECONNECT_NETWORK", () => {
	//Callback whenever network reconnected
})
```

For more events you can check on **General events emitted during sdk usage**

### Step 5: Login to identify a user

> _**NOTE:**_ This step is required for any further steps or action methods call from `sdkViewerInstance`

**Login as a guest user**

```javascript
sdkViewerInstance.guestLogin(null, function () {
	let currentUser =  Object.assign({}, sdkViewerInstance.getCurrentUser())
})
```

**Login as a user (SSO)**

```javascript
let userInfo = {
	beliveId: "user123",
	displayName: "Test User"
}
sdkViewerInstance.ssoLogin(
	userInfo,
	(data) => {
		// Success callback
		currentUser = sdkViewerInstance.getCurrentUser()
	},
	(error) => {
		// Error callback
		console.error(err)
	}
)
```

### Step 6: Watch a livestream video or a recorded video

```javascript
let streamData = null

let streamInfo = {
	streamName: '[STREAM_TITLE]',
	passwordStream: "[STREAM_PASSWORD]" // optional
}

let isPrivate = streamInfo.passwordStream ? true : false

sdkViewerInstance.getDetailStream(
	streamInfo,
	isPrivate,
	(response) => {
		// Success callback
		// Save this stream data to a 
		// global variable for further customizations
		streamData = response.data

		//Listen to video events
		const videoInstance = sdkViewerInstance.getVideoPlayer();
		if (streamData.status === 
				BeliveSdk.STATUS_VIDEO.RECORDED) {
			 //TIME_UPDATE will be useful 
			// for reading history chat in Advanced guide
		     videoInstance.on("TIME_UPDATE", (currentTime) => {
		         console.log(currentTime)
		     })
		     videoInstance.on("DURATION_CHANGED", (duration) => {
		         console.log(duration)
		     })
	    }
		videoInstance.on("PLAYING", () => {
		      console.log(`Video is playing.`)
		})
		videoInstance.on("PAUSED", () => {
		      console.log(`Video paused.`)
		})
	},
	(error) => {
		// Error callback
	}
)
```

### Step 7: Real-time messaging

**Connect to a chat room**

After get stream data success in previous step we are going to use that stream data to connect to a RTM server

> _**NOTE:**_ `streamData` is retrieved from **Step 6**

```javascript
// Initialize RTM server
sdkViewerInstance.createRtmServer(
	() => {
		// create RTM success callback
	},
	() => {
		// error callback
	}
)

// Connect to chat room
sdkViewerInstance.getRtmServer().createChatRoom(
	//streamData is saved from previous step
	streamData.slug,
	(msgJson) => {
		// Receive a message whenever a user 
		// in this chat room send a message
		console.log(msgJson)
	},
	(data) => {
		// Success callback
	},
	(data) => {
		// Close callback
	},
	(data) => {
		// Error callback
	}
)
```

**Send a message to channel**

> _**NOTE:**_ You should **ONLY** call `sendMessageRTM` after `createChatRoom` initialized successfully

```javascript
const messageJson = {
	message: "Test Message",
	messageType: BeliveSdk.MESSAGE_TYPE.MESSAGE
}
sdkViewerInstance.sendMessageRTM(
	streamData.slug,
	messageJson,
	(res) => {
		if (res.status  ===  "Success") {
			// success callback
		}
	},
	(err) => {
		console.log(err)
	}
)
```

### Step 8: Destroy live video and chat room

Call these functions whenever you no longer needed or a user navigate to a different page

> _**NOTE:**_ `streamData` is retrieved from **Step 6**

```javascript
sdkViewerInstance.leaveStream();
sdkViewerInstance.disconnectRtmChannel(streamData.slug, false, (res) => {
	console.log(`Disconnect RTM successfully!`)
})
```

### Advanced guide

#### Anti-spamming messages

This service is useful when you have a huge traffic of messages and want to render everything to the UI

First, we initialize `RTMThreshold` service and define `threshold` that only receive maximum of 100 messages per second and will **IGNORE** the rest

```javascript
const rtmThreshold = new BeliveSdk.RTMThreshold({
	//total messages that should be hold
	threshold: 100,
})

rtmThreshold.onReceiveMessages((listMessages) => {
	//Receive a list of messages
})
```

Add any incoming messages that you want to limit

```javascript
sdkViewerInstance.getRtmServer().createChatRoom(
	streamData.slug,
	(msgJson) => {
		// If the message is coming from 
		// current user, we should show it instantly
		if (msgJson.senderUserId === 
			sdkViewerInstance.getCurrentUser().userId)
			rtmThreshold.pushInstant(msgJson)
		else rtmThreshold.push(msgJson)
	},
	(data) => {
		// Success callback
	},
	(data) => {
		// Close callback
	},
	(data) => {
		// Error callback
	}
)
```

#### Read history chat from a recorded stream

> _**NOTE:**_\
> This function is **ONLY available** for recorded stream We can determine a video is live or recorded by check its status `streamData` is retrieved from **Step 6**

Get history messages from the recorded livestream

```javascript
let historyMessages = [];
const isRecordedStream = streamData.status === BeliveSdk.STATUS_VIDEO.RECORDED
if(isRecordedStream){
	sdkViewerInstance.getHistoryChat((res) => {
		if (res.data) historyMessages = res.data
	})
}
```

Notice that we have already had a listener when the video time update in **Step 6** and now we will get messages base on current video time

```javascript
videoInstance.on("TIME_UPDATE", (currentTime) => {
	const messagesToCurrentTime = 
	historyMessages.filter((message) => 
			message.recordedTime / 1000  <= Math.ceil(currentTime))	
})
```

## API

### General methods

| Method                     | Description                                                                   |
| -------------------------- | ----------------------------------------------------------------------------- |
| `getMobileOperatingSystem` | Get the mobile operating system.                                |
| `getNewAccessToken`        | Get new access token and which allows user to auto login |
| `guestLogin`               | Guest login to get access token                                 |
| `ssoLogin`                 | SSO login to get access token                                   |
| `registerUser`             | Register new user                                                             |
| `loginUser`                | Login existing user                                                           |
| `logoutUser`               | Logout user                                                                   |
| `on`                       | <code>on(event, callback): void</code> Listen for events        |
| `getCurrentUser`           | Get info of current user                                                      |
| `getShareUrl`              | Get on-link sharing url                                                       |
| `getVideoWrapper`          | Get Id of video wrapper                                                       |
| `getRtmServer`             | Get current Real time messaging (RTM) server instance    |
| `getHttpInstance`          | Get current HTTPS instance                                                    |
| `createRtmServer`          | Leave one/all RTM channels or destroy RTM instance       |
| `disconnectRtmChannel`     | Get info of current user                                                      |
| `sendMessageRTM`           | Send RTM message to channel                                                   |

**getMobileOperatingSystem**

```javascript
getMobileOperatingSystem(): String
```

Determine the mobile operating system.

`*Return*:String`

**guestLogin**

```javascript
guestLogin(deviceUdid = null, successCallback, errorCallback): void
```

Method for login as guest.

**Parameters**

* `deviceUdid (string):` default null, SDK auto generates deviceUdid.
* `successCallback (function)`
* `errorCallback (function)`

**ssoLogin**

```javascript
ssoLogin(user, successCallback, errorCallback): void
```

Method to login user.

**Parameters**

* `user (object)`

```javascript
{
	// Required fields:
	beliveId: "{beliveId}",,
	displayName: "{displayName}"
}
```

* `successCallback (function)`
* `errorCallback (function)`

**getNewAccessToken**

```javascript
getNewAccessToken(user = null, 
                autoLogin = true, 
                successCallback, errorCallback): void
```

Method to get new access token with user data input

**Parameters**

* `user (object):` default null, SDK auto generates user.

```javascript
{
	// Required fields:
	beliveId: `web-client-${indexUser}`,
	deviceType: "2", // 2 -> web
	deviceUdid: "web", // only accept one value: "web"
	userName: `sdk_user_${indexUser}`,
	displayName: `Sdk User ${indexUser}`
}
```

* autoLogin (boolean): default TRUE
* successCallback (function)
* errorCallback (function)

**registerUser**

```javascript
registerUser(user, autoLogin = true, successCallback, errorCallback): void
```

Method to register new user.

**Parameters**

* `user (object)`

```javascript
{
	// Required fields:
	beliveId: "{beliveId}",
	deviceType: "2", // 2 -> web
	deviceUdid: "web", // only accept one value: "web"
	userName: "{userName}",
	displayName: "{displayName}"
}
```

* `autoLogin (boolean):` default TRUE
* `successCallback (function)`
* `errorCallback (function)`

**loginUser**

```javascript
loginUser(user, successCallback, errorCallback): void
```

Method to login user.

**Parameters**

* `user (object)`

```javascript
{
	// Required fields:
     beliveId: "{beliveId}",
     displayName: "{displayName}"
     userName: "{userName}", // Optional
}
```

* `successCallback (function)`
* `errorCallback (function)`

**logoutUser**

```javascript
logoutUser(successCallback, errorCallback): void
```

Method to logout user.

**Parameters**

* `successCallback (function)`
* `errorCallback (function)`

**on**

```javascript
on(event, callback): void
```

Method to listen event.

**Parameters**

* `event (string)`

> Check all available events in the Events Section.

* `callback (function)`

**getCurrentUser**

```javascript
getCurrentUser(): User
```

Method to get current user.

**getShareUrl**

```javascript
getShareUrl(): string
```

Method to get on-link share url.

**getVideoWrapper**

```javascript
getVideoWrapper(): void
```

Method to get video wrapper element.

**getRtmServer**

```javascript
getRtmServer(): void
```

Method to get current Real Time Messaging (RTM) server instance.

**getHttpInstance**

```javascript
getHttpInstance(): void
```

Method to get current HTTP instance instance to fetch/request APIs.

**createRtmServer**

```javascript
createRtmServer(successCallback, errorCallback): void
```

Method to create RTM server.

**Emit event**

* `CONNECTED_RTM` if RTM server is created successfully.

**Parameters**

* `successCallback (function)`
* `errorCallback (function)`

Example

```javascript
// Create RTM server
sdkViewerInstance.createRtmServer(() => { // create RTM success callback
	console.log(`Create RTM Success!`);
}), () => { // error callback
	console.log(`Create RTM Error!`);
};
```

**disconnectRtmChannel**

```javascript
disconnectRtmChannel(channel = null, 
                isDestroyInstance = false, 
                successCallback, errorCallback): void
```

Method to leave one channel / all channels and destroy RTM server instance.

**Emit events**

* `DISCONNECTED_RTM` RTM server is destroyed
* `LEAVE_RTM_CHANNEL` User left RTM channel

**Parameters**

* `channel` (string | default: null ⇒ apply for all channels)
* `isDestroyInstance` (Boolean | default: False)
* `successCallback (function)`
* `errorCallback (function)`

Example

```javascript
 // Disconnect RTM server:
sdkViewerInstance.disconnectRtmChannel(channel,false,() => { 
// destroy RTM channel success callback
	console.log(`Create RTM Success!`);
}), () => { // error callback
	console.log(`Create RTM Error!`);
};
```

**reconnectChatRtm**

```javascript
reconnectChatRtm(channel = null, successCallback, errorCallback): void
```

Method to re-join one channel / all channels

**Emit events**

* `JOINED_RTM_CHANNEL` After RTM channel is re-joined

**Parameters**

* `channel` (string | default: null ⇒ apply for all channels)
* `successCallback (function)`
* `errorCallback (function)`

Example

```javascript
sdkViewerInstance.reconnectChatRtm(channelChat, (res) => {
		console.log(`Reconnect RTM successfully!`);
});
```

**sendMessageRTM**

```javascript
sendMessageRTM(channel, message, successCallback, errorCallback): void
```

Method to send RTM (chat) message

**Emit events**

* `JOINED_RTM_CHANNEL` After RTM channel is re-joined

**Parameters**

* `channel (string)`
* `message (object)`
* `successCallback (function)`
* `errorCallback (function)`

Example

```javascript
 // Send RTM message:
let message = {
	message: "This is text message",
	messageType: 1, // 1 for send text message
};
sdkViewerInstance.sendMessageRTM("channel_test", 
			message, (res) => { // success callback
	if (res.status === "Success") {
			console.log("Send message successfully!")
	}
});
```

#### SDK Viewer Methods

| Method                 | Description                                                                                                       |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `new BeliveSdk.Viewer` | Initializes the viewer object with provided configurations                                   |
| `getDetailStream`      | Get details of a stream and play if it's available                                                  |
| `leaveStream`          | Leave current stream (This will record leave-team in database)                               |
| `getHistoryChat`       | Get chat history for showing in recorded stream                                                     |
| `likeStream`           | Method to like current live or recorded stream                                                      |
| `shareStream`          | Method to track the number of share button clicked of current live or recorded stream |
| `getPinnedMessage`     | Get pinned message which is set by host in live stream                                       |
| `getProducts`          | Get live shopping products list with pagination                                                     |
| `setStreamName`        | Set stream title                                                                                                  |
| `setStreamPassword`    | Set stream password                                                                                               |
| `getVideoPlayer`       | Get Video Player instance                                                                                         |

**new BeliveSdk.Viewer**

```javascript
new BeliveSdk.Viewer(sdkConfig, 
                videoWrapper, 
                streamName = "", 
                passwordStream = "", 
                webRtcConfig = null)
```

Method to create Sdk Viewer object.

**Parameters**

* `sdkConfig (object)`, is a setting for SDK

```javascript
 {
	key: "license_key_place_holder", // SDK key
	endpoint: "API_ENDPOINT", // API Server Endpoint
	rtmService: "agora", // rtm service agora or belive
}
```

* `videoWrapper (string)`, id of video element.
* `onEvents (Object functions)`

```javascript
{
	onVideoReady: function,
	onVideoDestroyed: function,
	onVideoErrors: function(error)
}
```

* `streamName (string | default: "")`, title of stream
* `passwordStream (string | default: "")`, password of stream, default "", leave "" for stream without password (not in use)
* `webRtcConfig` (object), default NULL

Example

```javascript
 let sdkViewerInstance = new BeliveSdk.Viewer(
   { // create viewer instance with config
	key: "license_key_place_holder", // SDK key
	endpoint: "API_ENDPOINT", // API Server Endpoint
	rtmService: "agora", // rtm service agora or belive
},
   "playerViewWrapper"
);
```

**getDetailStream**

```javascript
getDetailStream(streamInfo = null, 
                isPrivateStream = false, 
                successCallback, 
                errorCallback)
```

Get the details of the stream.

**Emit events**

* `PLAYING_STREAM` called after getting details of stream and play video
* `URL_NOT_AVAILABLE` if stream URL not available
* `STOPPED_STREAM` if user was blocked or stream stopped or stream has error

**Parameters**

* `streamInfo (object)`

```javascript
 {
 // streamName is slug_stream when isPriaveteStream = FALSE, and 
 // streamName is title_stream when isPriaveteStream = TRUE
	streamName: "{title_stream|slug}", 
// required if isPrivateStream=TRUE (not in use)
	passwordStream: "{password_stream}" 
```

* `isPrivateStream (boolean)` default FALSE
* `successCallback (function)`
* `errorCallback (function)`

**leaveStream**

```javascript
leaveStream(successCallback, errorCallback)
```

This method will make user leave stream.

**Parameters**

* `successCallback (function)`
* `errorCallback (function)`

**getHistoryChat**

```javascript
getHistoryChat(successCallback, errorCallback)
```

Get recorded chat (Only available if stream is recorded)

**Parameters**

* `successCallback (function)`
* `errorCallback (function)`

Example

```javascript
sdkViewerInstance.getHistoryChat(
    (response)=>{
       // success callback
       console.log(response.data)
    },
    (error) => {
      // error callback
      console.log(error)
    }
)
```

**likeStream**

```javascript
likeStream(successCallback, errorCallback)
```

Call this function to like the live or recorded stream

**Parameters**

* `successCallback (function)`
* `errorCallback (function)`

**shareStream**

```javascript
shareStream(successCallback, errorCallback)
```

Call this function to track the number of share button clicked

**Parameters**

* `successCallback (function)`
* `errorCallback (function)`

**getPinnedMessage**

```javascript
getPinnedMessage(successCallback, errorCallback)
```

Get host/broadcasters's pinned message (Pinning of a live stream comment message is available on Go Live app)

**Parameters**

* `successCallback (function)`
* `errorCallback (function)`

**getProducts**

```javascript
getProducts(params: { nextId: number, limit: number }, 
                successCallback, errorCallback)
```

Get live shopping products of a stream with pagination. It returns total products count and a number of products in success callback

**Parameters**

* `params (Object):`
  * `nextId` the current page of product list
  * `limit` the number of products should return in current page
* `successCallback (function)`
* `errorCallback (function)`

**setStreamName**

```javascript
setStreamName(streamName): void
```

Set stream title

**Parameters**

* `streamName (string)` Title of stream.

**getVideoPlayer**

```javascript
getVideoPlayer()
```

Get video player instance

**enableProfanityFilter**

```typescript
enableProfanityFilter(): Promise<void>
```

Enable profanity filter feature, use `cleanWords` method after `enableProfanityFilter` initialized

Example:

```javascript
sdkViewerInstance.enableProfanityFilter().then(()=>{
	let wordsAfterCleaned = 
	sdkViewerInstance.cleanWords('words',{ placeholder: "*" })
})
```

**cleanWords**

```javascript
cleanWords(words: string, options: {placeholder: string}): string
```

**Parameters**

* `words (string)` Words need to be cleaned up
* `options (Object)` Filter instance options
  * `placeholder (string)` Character used to replace profane words

#### SDK RTM Methods

| Method           | Description                |
| ---------------- | -------------------------- |
| `createChatRoom` | Create an RTM chat channel |

**createChatRoom**

```javascript
createChatRoom(channel, 
                onReceiveMessage, 
                onOpen, 
                onClose, 
                onError, 
                additionalInfo = {})
```

Method to create an RTM channel

**Emit events**

* `NEW_CHANNEL_MESSAGE` This event is called when a new RTM message is received. It returns msgJson

**Parameters**

* `channel (string)`
* `onReceiveMessage (function)`
* `onOpen (function)`
* `onClose (function)`
* `onError (function)`
* `additionalInfo (object), default {}`

Example

```javascript
sdkViewerInstance.getRtmServer().createChatRoom(
	channelChat, 
	(msgJson) => {
		console.log("Received RTM message", JSON.stringify(msgJson));
	},
	(data) => { // On Open callback
		writeLog(`RTM logs: ${data}`);
	},
	(data) => { // On Close callback
		writeLog(`RTM logs: ${data}`);
	},
	(data) => { // On Error callback
		writeLog(`RTM Error: ${JSON.stringify(data)}`, "error");
	}
);
```

#### RTM Threshold Methods

Set RTM threshold and limit for `optimal performance`

| Method              | Description                                               |
| ------------------- | --------------------------------------------------------- |
| `push`              | Insert new message into queue               |
| `pushInstant`       | Insert new message direct into current list |
| `reset`             | Remove all messages and message queue       |
| `onReceiveMessages` | Callback messages after processed           |
| `destroy`           | Destroy instance                                          |

Example

```javascript
const rtmThreshold = new BeliveSdk.RTMThreshold()
rtmThreshold.onReceiveMessages((messages)=>{
   console.log(messages)
})
sdkViewerInstance.getRtmServer().createChatRoom(
	channelChat, 
	(msgJson) => {
	   rtmThreshold.push(msgJson)
	}
);
```

**push**

```javascript
push(message): void
```

Insert new message into queue to process

**Parameters**

* `message (any)` message model

**pushInstant**

```javascript
pushInstant(message): void
```

Insert new message direct into current list of messages without processing

**Parameters**

* `message (any)` message model

**reset**

```javascript
reset(): void
```

Remove all messages and message queue

**destroy**

```javascript
destroy(): void
```

Terminate the instance. After destroy, the instance no longer emits events or receives any data

### Video Player Methods

| Method      | Description                                      |
| ----------- | ------------------------------------------------ |
| `pause`     | Pause the video                                  |
| `play`      | Play/resume the video                            |
| `setMuted`  | Mute or Un-mute the video                        |
| `setVolume` | Method to set volume of the player |
| `seekTo`    | The time to seek to in seconds     |
| `isMuted`   | Check the video is muted or not    |
| `isPaused`  | Check the video is paused or not   |

**pause**

```javascript
pause()
```

Method to pause the video

Example

```javascript
sdkViewerInstance.getVideoPlayer().pause();
```

**play**

```javascript
play()
```

Method to play/resume the video

Example

```javascript
sdkViewerInstance.getVideoPlayer().play();
```

**setMuted**

```javascript
setMuted(mute: boolean)
```

Method to mute or unmute the player

**Parameters**

* `mute: boolean` Set true to mute the player, false to unmute

Example

```javascript
//Mute video 
sdkViewerInstance.getVideoPlayer().setMuted(true);
//Unmute video
sdkViewerInstance.getVideoPlayer().setMuted(false);
```

**setVolume**

```javascript
setVolume(volume: number)
```

Method to set volume of the player

**Parameters**

* `volume: number` The volume to be set. Valid values are in the range 0.0f to 1.0f

Example

```javascript
//set player volume to 50 percent
sdkViewerInstance.getVideoPlayer().setVolume(0.5);
```

**seekTo**

```javascript
seekTo(time: number)
```

Method to seeks to a specified time (in seconds) of the video

**Parameters**

* `time: number` The video time to seek to, in seconds

Example

```javascript
//set player volume to 50 percent
sdkViewerInstance.getVideoPlayer().setVolume(0.5);
```

**isMuted**

```javascript
isMuted
```

Method to check the video is muted or not

**isPaused**

```javascript
isPaused
```

Method to check the video is paused or not

Example

```javascript
//set player volume to 50 percent
 sdkViewerInstance.getVideoPlayer().isPaused;
```

### Models

### RtmError

| Property  | Data type | Description                                                                                                                                                                              |
| --------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `code`    | number    | A unique number that define an error, See more at RTM Error Codes                                                                                     |
| `message` | string    | Error description                                                                                                                                                                        |
| `state`   | string    | The connection state that refers to one of the defined:<br>ABORTED<br>CONNECTING<br>RECONNECTING<br>CONNECTED<br>DISCONNECTED                                       |
| `reason`  | string    | The connection reason that refers to one of the defined:<br>BANNED_BY_SERVER<br>INTERRUPTED<br>LOGIN<br>LOGIN_FAILURE<br>LOGIN_SUCCESS<br>LOGIN_TIMEOUT<br>LOGOUT<br>REMOTE_LOGIN |

### SDK Events

### General events emitted during sdk usage.

| Method                | Description                                  | Value Return                                                                 |
| --------------------- | -------------------------------------------- | ---------------------------------------------------------------------------- |
| `UPDATE_STATISTICS`   | Event when has new value of statistic stream | See below                                                                    |
| `UPDATE_TIME_STREAM`  | Event to update stream time                  | `Integer`                                                                    |
| `UPDATE_END_RESULT`   | Event to get end result of stream            | `{ DurationTime: 0, TotalLikes: 0, TotalReceivedStars: 0, TotalViewers: 0 }` |
| `PLAYING_STREAM`      | Event to update stream time                  | `Integer`                                                                    |
| `NEW_STREAM`          | Event on new stream                          | -                                                                            |
| `STREAMING`           | Event on streaming                           | -                                                                            |
| `PAUSED_STREAM`       | Event on stream is PAUSED                    | -                                                                            |
| `STOPPED_STREAM`      | Event on stream is Stopped                   | -                                                                            |
| `DISCONNECTED_RTM`    | Event on disconnected RTM                    | -                                                                            |
| `CONNECTED_RTM`       | Event on connected RTM                       | -                                                                            |
| `NEW_CHANNEL_MESSAGE` | Event on new channel message                 | See Below                                                                    |
| `LOST_NETWORK`        | Event when lost the internet                 | -                                                                            |
| `RECONNECT_NETWORK`   | Event when the internet is reconnected       | -                                                                            |

**UPDATE\_STATISTICS**

Value return

```javascript
{ 
	AdminDisplayName: "", 
	Combo: 0, 
	DurationTime: 0, 
	GiftColor:0, 
	GiftId: "", 
	GiftImageLink: "", 
	GiftName: "", 
	IsComboGift: false, 
	IsExpensive: false, 
	IsLiked: false, 
	Message: "",  
	ProfilePic: "", 
	RankingList: [], 
	TotalReceivedStars: 0, 
	VotingScores: 0, 
	isGroup: false, 
	luckyResult: -1, 
	mGiftCombo: 0, 
	streamTitleColor: "", 
	x: 0, 
	y: 0, 
	TotalLikes: 0, 
	TotalViewers: 0, 
	Slug: "" 
}
```

**NEW\_CHANNEL\_MESSAGE**

Value return

```javascript
{ 
	message: "Message SDK Viewer" 
	messageType: 1 
	senderDisplayName: "display_name" 
	senderUserId: 357502 
	senderUserName: "+guest_id" 
	channel: "channel_slug" 
	licenseKey: "license_key" 
}
```

#### Video Events 

| Method             | Description                                                                                                | Value Return                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `READY`            | Indicates that video is ready                                                                              | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `TIME_UPDATE`      | Indicates that current time of video changed                                                               | `Number (milliseconds)`                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `LOADED_METADATA`  | Indicates that video loaded metadata                                                                       | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `CAN_PLAY`         | Indicates that video ready to play                                                                         | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `WAITING`          | Indicates that player is stopped for buffering                                                             | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `PLAYING`          | Indicates that video is playing                                                                            | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `PAUSED`           | Indicates that video is paused                                                                             | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `MUTE_CHANGED`     | Indicates that video is muted or unmuted                                                                   | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `URL_CHANGED`      | Indicates that video url is changed to another backup url                                                  | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `VOLUME_CHANGED`   | Indicates that video volume changed                                                                        | `Number (between 0.0f and 1.0f)`                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `DURATION_CHANGED` | Indicates that the total duration of video changed                                                         | `Number (seconds)`                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `RETRY_LOAD_VIDEO` | Event trigger when playback has an issue. SDK will internally try to playback video (retry for 30 seconds) | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `ERROR`            | Indicates that an error occurred                                                                           | Error object:<br><code>{ code: number, message: string, MEDIA_ERR_ABORTED: 1, MEDIA_ERR_NETWORK: 2, MEDIA_ERR_DECODE: 3, MEDIA_ERR_SRC_NOT_SUPPORTED: 4 }</code><br>_MEDIA_ERR_ABORTED: W3C error code for media error aborted.<br>_MEDIA_ERR_NETWORK: W3C error code for any network error.<br>_MEDIA_ERR_DECODE: W3C error code for any decoding error.<br>_MEDIA_ERR_SRC_NOT_SUPPORTED: W3C error code for any time that a source is not supported. |
| `ENDED`            | Indicates that the video playback ended                                                                    | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `DESTROYED`        | Indicates that the video player destroyed                                                                  | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

### Error codes

### Status response

| Status code | Description |
| ----------- | ----------- |
| `ERROR`     | Error       |
| `SUCCESS`   | Success     |

#### General error codes

****

| Error code             | Description                                                                                                                                                                |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NO_MEDIAS`            | Connect Camera and microphone <strong>(for Live broadcast only)</strong>                                                                              |
| `NO_CAMERA`            | Connect Camera <strong>(for Live broadcast only)</strong>                                                                                                    |
| `NO_MIC`               | Connect microphone <strong>(for Live broadcast only)</strong>                                                                                                |
| `PERMISSION_DENIED`    | Permission denied <strong>(for Live broadcast only)</strong>                                                                                                 |
| `NOT_SUPPORT_MEDIA`    | Your browser does not support <code>getUserMedia</code> API which is used for media playback <strong>(for Live broadcast only)</strong> |
| `RTM_NOT_FOUND`        | RTM server not found                                                                                                                                                       |
| `RTM_CONFIG_INCORRECT` | RTM configs is incorrect                                                                                                                                                   |
| `RTM_ERROR`            | RTM error                                                                                                                                                                  |

#### RTM error codes

| Error code | Description                                                                                                                                                                                                                      |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `8`        | The user has created another chat room but has not destroyed an existed chat room                                                                                                                    |
| `4`        | The `licenseKey` is invalid                                                                                                                                                                                                      |
| `9`        | The <code>createChatRoom</code> times out. The current timeout is ten seconds. You need to <code>createChatRoom</code> again after timeout                                                    |
| `10`       | The call frequency of the <code>createChatRoom</code> method exceeds the limit of two calls per second                                                                                               |
| `1`        | Common <code>createChatRoom</code> failure                                                                                                                                                                         |
| `16`       | The user is joining or has joined the channel                                                                                                                                                                      |
| `212`      | The user has created multiple channel instances of the same channel name                                                                                                                                    |
| `15`       | The number of the RTM channels you are in exceeds the limit of 20                                                                                                                                           |
| `11`       | Common failure. The user fails to join the channel                                                                                                                                                                 |
| `13`       | The user fails to join the channel because the argument is invalid                                                                                                                                          |
| `18`       | The frequency of joining the same channel exceeds two times every five seconds                                                                                                                              |
| `17`       | The method call frequency exceeds 50 calls every three seconds                                                                                                                                              |
| `102`      | The user does not call the <code>createChatRoom</code> method, or the method call of <code>createChatRoom</code> does not succeed before calling the <code>disconnectServerChat</code> method |
| `14`       | A timeout occurs when joining the channel. The current timeout is set as five seconds.                                                                                                                      |
| `31`       | Common failure. The userfails to leave the channel.                                                                                                                                                                |
| `33`       | The user is not in the channel.                                                                                                                                                                                    |

## Live Shopping

SDK provides live shopping feature. You can get list of products during live or recorded using Web SDK API. Refer to `getProducts` api above.
