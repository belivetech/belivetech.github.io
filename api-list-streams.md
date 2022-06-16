---
title: List Streams
---

To access the stream list, the request needs to be made to the endpoint: `/api/users/streams`  This requires a `POST` request 
which needs `beliveLicenseKey` and a username.  To get those data, we have to do following steps 
 

### beliveLicenseKey

Make sure that have `beliveLicenseKey` value. It should be provided for SDK's testing and production environment.


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


The userName field is the userName of the host that can be filtered by, and this field is `required`, if not provided, the data will return most recent streams. 

For the header,  provide the language data: `0`, and your BeLive license key.

`cURL` example:

```bash

curl --location --request POST 'https://SDK_BASE_URL/api/users/streams' \
--header 'Content-Type: application/json' \
--header 'language: 0' \
--header 'beliveLicenseKey: belive_license_key' \
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