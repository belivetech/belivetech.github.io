---
title: Moderation API
---

Live Shopping SDK comes with out-of-box moderation features such as 

1. Pin/Unpin comments during Live streams
2. Mute/Un Mute users from commenting during live streams


### Pin/Un-Pin comments during Live Stream 


#### Android 

**Pin Comment**

A pinned comment/message is just a chat message (normal message, like message, share message, etc ) with type `MESSAGE_TYPE_HOST_PINNED_MESSAGE`.
During a live stream, host can choose a message and pin it.
To pin a message, use `BlsLiveStreamManager.pinChatMessage(ChatMessageModel)`.

The result will be reported to this callback:

```kotlin
onMessageUpdateResult(chatMessage: ChatMessage?, updateType: Int, resultCode: Int)
```

`chatMessage`: The message that is getting updated.
`updateType`: For pinning a message, it is always ChatMessageUpdateType.PIN
`resultCode`: If the message is pinned successfully, this code will be `sg.belive.lib.core.chat.ChatManager.ERR_CODE_OK`, otherwise it will be `sg.belive.lib.core.chat.ChatManager.ERR_CODE_UPDATE_MSG_XXX`

**Un-Pin Comment**

To unpin a message, use `BlsLiveStreamManager.unpinChatMessage()`. This will unpin the previous pinned message.

The result will be reported to this callback:

```kotlin
onMessageUpdateResult(chatMessage: ChatMessage?, updateType: Int, resultCode: Int)
```

`chatMessage`: It will be null
`updateType`: For unpinning a message, it is always ChatMessageUpdateType.UNPIN
`resultCode`: If the message is unpinned successfully, this code will be `sg.belive.lib.core.chat.ChatManager.ERR_CODE_OK`, otherwise it will be `sg.belive.lib.core.chat.ChatManager.ERR_CODE_UPDATE_MSG_XXX`

**Limitation**

The Belive SDK allows 1 pinned message at a time per stream. Calling `pinChatMessage` API will override the previous pinned message (if any).

**Pin/Unpin Comment at viewer side**

Viewer side will receive Pin/Unpin message updates in real time through this callback. This is also applicable for viewers who joined stream after a messages has been pinned by Host.

```kotlin
onMessageUpdateResult(chatMessage: ChatMessage?, updateType: Int, resultCode: Int)
```

`chatMessage`: The pinned message data or null if it is unpin type.
`updateType`: ChatMessageUpdateType.PIN, ChatMessageUpdateType.UNPIN or ChatMessageUpdateType.PIN_MESSAGE_RESET
`resultCode`: It is always ERR_CODE_OK

**Recorded Streams**

For recorded stream, pin/unpin messages is also recorded and the `onMessageUpdateResult` is called correctly with the stream timeline.
Because the recorded stream can be seeked, the Belive SDK will trigger `onMessageUpdateResult` with `ChatMessageUpdateType.PIN_MESSAGE_RESET` updateType if there is no pin/unpin message at the current seeked position.


#### iOS

**Pin Comment**

For host to pin a comment/message. call following function.

```swift
BeLiveBroadcasterManager.sendPinnedMessage(message: BLStreamMessage, completion: BLMessageCompletion)
```

**Un-Pin Comment**

For host to un-pin a comment/message, call following functions.

```swift
BeLiveBroadcasterManager.sendUnpinnedMessage(completion: BLMessageCompletion)
```

**Pin/Unpin Comment at viewer side**

For Viewers to receive pin/unpin message, SDK provides following callbacks

```swift
BeLiveAudienceManagerMessageDelegate.beLiveAudienceManager(manager: BeLiveAudienceManager, didPinMessage message: BLStreamMessage)
BeLiveAudienceManagerMessageDelegate.beLiveAudienceManager(manager: BeLiveAudienceManager, didUnpinMessage message: BLStreamMessage)

```

For Viewer to get current pin message when entering stream, use following SDK function. Same method can be used for recorded streams

```swift
BLStreamService.getPinnedMessage(slug: String)
```

#### Web 

Viewer side will receive Pin/Unpin message updates in real time through RTM message callback function `onReceiveMessages`. You can filter our `msgJson` based on 
messageType : `BeliveSdk.MESSAGE_TYPE.HOST_PINNED_MESSAGE`.

For new viewers who join the live stream after host has pinned message, they can use `SDK Viewer Method: getPinnedMessage` to retrieve it. For viewers, who join recorded stream, they can use same method `SDK Viewer Method: getPinnedMessage` to get pinned message during live stream.


### Mute/Un Mute users from commenting during live stream

Mute and unmute feature is avilable via Live Shopping CMS. To complete flow, our SDKs provide real-time callbacks for taking action immediately during live streams. Refer to below APIs. 

#### Android 

**Mute and Unmute user**

When a user is muted via CMS, Android SDK will trigger following callback method.

```kotlin
 override fun onMuteStatusChanged(isMuted: Boolean) {
        if (isMuted) {
            // hide comment box
        } else {
           // un-hide comment
        }
    }
```

 For viewers who re-enter stream after being muted, you can check status in stream model parameter `isMute`. 

#### iOS

**Mute user**

When a user is muted via CMS, SDK will trigger following callback method of `BeLiveAudienceManagerDelegate` for viewers.

```swift
 func beLiveAudienceManager(manager: BeLiveAudienceManager, didMuteMessage message: BLStreamMessage)
```
We suggest to hide comment box so that user is unable to send comment. 

**Un-mute user**

CMS also provides option to un-mute users, SDK will trigger following callback method of `BeLiveAudienceManagerDelegate` for viewers.

```swift
func beLiveAudienceManager(manager: BeLiveAudienceManager, didUnMuteMessage message: BLStreamMessage)

```
 We suggest to un-hide comment box so that user is able to send comment. 

 For viewers who re-enter stream after being muted, you can check status in stream model parameter `isMute`. 

