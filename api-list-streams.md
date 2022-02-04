---
title: List Streams
---

To access the stream list, the request needs to be made to the endpoint: `/api/users/streams`  This requires a `POST` request 
which needs an authorization token and a username.  To get those data, we have to do following steps 


1. first do a login(either guest or BeLive login), that will give us the authorization token.  
2. Use `authorization token` for list streams api.  

### BeLive Login 

BeLive login is accessible as a `POST` request at this endpoint: `/api/users/belive-login`.  Provide the following data in body as 
JSON:

```json
{
  "beliveId": "string",
  "displayName": "string",
  "deviceUdid": "string",
  "deviceType": 2,
  "latitude": 0,
  "longitude": 0
}
```

`beliveId` is required. `deviceType=2` indicates that it is web client.

In the headers, provide the language data: `0`, and your BeLive license key.  

`cURL` example.

```bash
curl --location --request POST 'https://SDK_BASE_URL/api/users/belive-login' \
--header 'Content-Type: application/json' \
--header 'language: 0' \
--header 'beliveLicenseKey: belive_license_key' \
--data-raw '{
  "beliveId": "beliveid",
  "displayName": "optional",
  "deviceUdid": "optional",
  "deviceType":2,
  "latitude": 0,
  "longitude": 0
}'

```

In the return data, you will receive the information regarding the user session under the `data` key.  
The most important for the next step are the following:

```json
{
 "data":
  {
   "accessToken": "string",
    "expires": "string"
  }
}
```

You can also use guest login endpoint: `/api/users/guest-login` to generate `accessToken` and use in next step.

### List Streams

Listing stream is accessible as a `POST` request at this endpoint: `/api/users/streams`.  Provide the following data in body as 
JSON:

```json
{
 "userName": "string"
  "limit": 0,
  "nextId": 0
}
```


The userName field is the userName of the host that can be filtered by, and this field is `required`, if not provided, the data will be empty. 

For the header,  provide the language data: `0`, and your BeLive license key as well as the Authorization field in this format:
```
Authorization: Bearer {accessToken}
```
Where accessToken is the accessToken field received in **BeLive Login**

`cURL` example:

```bash

curl --location --request POST 'https://SDK_BASE_URL/api/users/streams' \
--header 'Content-Type: application/json' \
--header 'language: 0' \
--header 'beliveLicenseKey: belive_license_key' \
--header 'Authorization: Bearer access_token' \
--data-raw '{
  "userName": "hostusername",
  "nextId": 0,
  "limit": 20
}'


```

Sample response 

```json

{
  "code": 0,
  "data": {
    "result": [
      {
        "streamCount": 0,
        "stream": {
          "likeCount": 0,
          "isLike": 0,
          "isShowPromo": true,
          "isMute": true,
          "historyChat": "string",
          "viewSessionId": 0,
          "publisher": {
            "isFollow": 0,
            "isSeller": true,
            "userId": 0,
            "userName": "string",
            "displayName": "string",
            "userImage": "string",
            "gender": "string",
            "handle": "string",
            "deviceType": 0,
            "followerCount": 0,
            "userThumbnailImage": "string",
            "isFirstLive": true,
          },
          "webStreamUrl": "string",
          "webStreamPlay": "string",
          "rtmpUrl": "string",
          "streamId": 0,
          "userId": 0,
          "slug": "string",
          "coverImage": "string",
          "title": "string",
          "description": "string",
          "duration": 0,
          "viewCount": 0,
          "streamType": 0,
          "screenType": 0,
          "countryCode": "string",
          "streamUrl": "string",
          "status": 0,
          "tags": "string",
          "isRecorded": true,
          "isDeleted": true,
          "created": "string",
          "beginStream": "string",
          "endStream": "string",
          "resolution": 0,
          "orderButtonText": "string",
          "orderInProgressText": "string",
          "streamMode": 0,
          "promoCode": "string",
          "promoCodeDescription": "string",
          "cartUrl": "string",
          "isShowLikeCount": true,
          "isShowViewCount": true,
          "titlePlainText": "string"
        }
      }
    ],
    "nextId": 0,
    "isEnd": true
  },
  "message": "string",
  "codeDetails": [
    {
      "code": 0,
      "message": "string"
    }
  ]
}

````
Note : Some of response parameters that are reserved for future use are omitted from above sample. 

List of streams will be returned in `result` array. If there is on-going live stream, it will be first one in the list. You can filter it out by `status=1`,
Recorded streams are identified by `status=2` and `isRecorded=true`.

### Recommendations

We recommend to implement this api flow for fetching stream list in your backend. Then your front-end can call your backend which will call belive stream list api.