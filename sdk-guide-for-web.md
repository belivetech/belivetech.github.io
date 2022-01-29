---
title: SDK Guide for Web
redirect_from:
  - /getting-started.html
---

**Good to know:** BeLive SDK for Web only supports video playback for live and recorded streams along with interactive elements.

## Table of contents

* Getting started
* Step-by-step guide for viewer to watch stream
  * Step 1: Set up SDK
  * Step 2: Insert video tag in your html
  * Step 3: Create a viewer instance
  * Step 4: Add SDK event listeners
  * Step 5: Login to identify a user
  * Step 6: Watch a livestream video or a recorded video
  * Step 7: Real-time messaging
  * Step 8: Destroy live video and chat room
* Advanced guide
  * Anti-spamming messages
  * Read history chat from a recorded stream
* API
  * General methods
  * RTM Threshold Methods
  * Video Player Methods
  * Models
    * RtmError
  * SDK events
    * General events emitted during sdk usage
    * Video Events
  * Error codes
    * Status response
    * General error codes
    * RTM error codes

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

### General

| Method                     | Description                                                                   |
| -------------------------- | ----------------------------------------------------------------------------- |
| `getMobileOperatingSystem` | <p>Get the mobile operating </p><p>system.</p>                                |
| `getNewAccessToken`        | <p>Get new access token </p><p>and which allows </p><p>user to auto login</p> |
| `guestLogin`               | <p>Guest login to get </p><p>access token</p>                                 |
| `ssoLogin`                 | <p>SSO login to get </p><p>access token</p>                                   |
| `registerUser`             | Register new user                                                             |
| `loginUser`                | Login existing user                                                           |
| `logoutUser`               | Logout user                                                                   |
| `on`                       | <p><code>on(event, callback): void</code> </p><p>Listen for events</p>        |
| `getCurrentUser`           | Get info of current user                                                      |
| `getShareUrl`              | Get on-link sharing url                                                       |
| `getVideoWrapper`          | Get Id of video wrapper                                                       |
| `getRtmServer`             | <p>Get current Real time </p><p>messaging (RTM) </p><p>server instance</p>    |
| `getHttpInstance`          | Get current HTTPS instance                                                    |
| `createRtmServer`          | <p>Leave one/all RTM </p><p>channels or destroy </p><p>RTM instance</p>       |
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
| `new BeliveSdk.Viewer` | <p>Initializes the viewer </p><p>object with provided </p><p>configurations</p>                                   |
| `getDetailStream`      | <p>Get details of a stream </p><p>and play if it's available</p>                                                  |
| `leaveStream`          | <p>Leave current stream </p><p>(This will record </p><p>leave-team in database)</p>                               |
| `getHistoryChat`       | <p>Get chat history for </p><p>showing in recorded stream</p>                                                     |
| `likeStream`           | <p>Method to like current </p><p>live or recorded stream</p>                                                      |
| `shareStream`          | <p>Method to track the </p><p>number of share button </p><p>clicked of current live </p><p>or recorded stream</p> |
| `getPinnedMessage`     | <p>Get pinned message </p><p>which is set by host </p><p>in live stream</p>                                       |
| `getProducts`          | <p>Get live shopping </p><p>products list with pagination</p>                                                     |
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
| `push`              | <p>Insert new message </p><p>into queue</p>               |
| `pushInstant`       | <p>Insert new message </p><p>direct into current list</p> |
| `reset`             | <p>Remove all messages </p><p>and message queue</p>       |
| `onReceiveMessages` | <p>Callback messages </p><p>after processed</p>           |
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
| `setVolume` | <p>Method to set volume </p><p>of the player</p> |
| `seekTo`    | <p>The time to seek to </p><p>in seconds</p>     |
| `isMuted`   | <p>Check the video is </p><p>muted or not</p>    |
| `isPaused`  | <p>Check the video is </p><p>paused or not</p>   |

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

**RtmError**

| Property  | Data type | Description                                                                                                                                                                              |
| --------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `code`    | number    | <p>A unique number </p><p>that define an </p><p>error, See </p><p>more at </p><p>RTM Error Codes</p>                                                                                     |
| `message` | string    | Error description                                                                                                                                                                        |
| `state`   | string    | <p>The connection </p><p>state that refers </p><p>to one of the defined:<br>ABORTED<br>CONNECTING<br>RECONNECTING<br>CONNECTED<br>DISCONNECTED</p>                                       |
| `reason`  | string    | <p>The connection reason that refers to one of the defined:<br>BANNED_BY_SERVER<br>INTERRUPTED<br>LOGIN<br>LOGIN_FAILURE<br>LOGIN_SUCCESS<br>LOGIN_TIMEOUT<br>LOGOUT<br>REMOTE_LOGIN</p> |

### SDK Events

**General events emitted during sdk usage.**

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

**Video Events**

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
| `ERROR`            | Indicates that an error occurred                                                                           | <p>Error object:<br><code>{ code: number, message: string, MEDIA_ERR_ABORTED: 1, MEDIA_ERR_NETWORK: 2, MEDIA_ERR_DECODE: 3, MEDIA_ERR_SRC_NOT_SUPPORTED: 4 }</code><br>_MEDIA_ERR_ABORTED: W3C error code for media error aborted.<br>_MEDIA_ERR_NETWORK: W3C error code for any network error.<br>_MEDIA_ERR_DECODE: W3C error code for any decoding error.<br>_MEDIA_ERR_SRC_NOT_SUPPORTED: W3C error code for any time that a source is not supported.</p> |
| `ENDED`            | Indicates that the video playback ended                                                                    | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `DESTROYED`        | Indicates that the video player destroyed                                                                  | -                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

### Error codes

**Status response**

| Status code | Description |
| ----------- | ----------- |
| `ERROR`     | Error       |
| `SUCCESS`   | Success     |

**General error codes**

****

| Error code             | Description                                                                                                                                                                |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NO_MEDIAS`            | <p>Connect Camera and </p><p>microphone </p><p><strong>(for Live broadcast only)</strong></p>                                                                              |
| `NO_CAMERA`            | <p>Connect Camera </p><p><strong>(for Live broadcast only)</strong></p>                                                                                                    |
| `NO_MIC`               | <p>Connect microphone </p><p><strong>(for Live broadcast only)</strong></p>                                                                                                |
| `PERMISSION_DENIED`    | <p>Permission denied </p><p><strong>(for Live broadcast only)</strong></p>                                                                                                 |
| `NOT_SUPPORT_MEDIA`    | <p>Your browser does </p><p>not support <code>getUserMedia</code> </p><p>API which is used for </p><p>media playback </p><p><strong>(for Live broadcast only)</strong></p> |
| `RTM_NOT_FOUND`        | RTM server not found                                                                                                                                                       |
| `RTM_CONFIG_INCORRECT` | RTM configs is incorrect                                                                                                                                                   |
| `RTM_ERROR`            | RTM error                                                                                                                                                                  |

**RTM error codes**

| Error code | Description                                                                                                                                                                                                                      |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `8`        | <p>The user has created </p><p>another chat room but </p><p>has not destroyed an </p><p>existed chat room</p>                                                                                                                    |
| `4`        | The `licenseKey` is invalid                                                                                                                                                                                                      |
| `9`        | <p>The <code>createChatRoom</code> </p><p>times out. The current </p><p>timeout is ten seconds. </p><p>You need to <code>createChatRoom</code> </p><p>again after timeout</p>                                                    |
| `10`       | <p>The call frequency of the </p><p><code>createChatRoom</code> method </p><p>exceeds the limit of two </p><p>calls per second</p>                                                                                               |
| `1`        | <p>Common <code>createChatRoom</code> </p><p>failure</p>                                                                                                                                                                         |
| `16`       | <p>The user is joining or has</p><p> joined the channel</p>                                                                                                                                                                      |
| `212`      | <p>The user has created </p><p>multiple channel instances </p><p>of the same channel name</p>                                                                                                                                    |
| `15`       | <p>The number of the RTM </p><p>channels you are in </p><p>exceeds the limit of 20</p>                                                                                                                                           |
| `11`       | <p>Common failure. The user </p><p>fails to join the channel</p>                                                                                                                                                                 |
| `13`       | <p>The user fails to join the </p><p>channel because the </p><p>argument is invalid</p>                                                                                                                                          |
| `18`       | <p>The frequency of joining </p><p>the same channel exceeds </p><p>two times every five seconds</p>                                                                                                                              |
| `17`       | <p>The method call frequency </p><p>exceeds 50 calls every </p><p>three seconds</p>                                                                                                                                              |
| `102`      | <p>The user does not call the <code>createChatRoom</code> method, </p><p>or the method call of </p><p><code>createChatRoom</code> does </p><p>not succeed before calling the <code>disconnectServerChat</code> </p><p>method</p> |
| `14`       | <p>A timeout occurs when joining </p><p>the channel. The current </p><p>timeout is set as five seconds.</p>                                                                                                                      |
| `31`       | <p>Common failure. The user</p><p>fails to leave the channel.</p>                                                                                                                                                                |
| `33`       | <p>The user is not in the </p><p>channel.</p>                                                                                                                                                                                    |

## Live Shopping

SDK provides live shopping feature. You can get list of products during live or recorded using Web SDK API. Refer to `getProducts` api above.
