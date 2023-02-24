---
title: Live Shopping Guide
---

Live shopping products can be configured in following two ways

* Content Management System (CMS)
* Upload and managed products using iOS and Android SDK API (Refer to respective guides)

Live shopping products for live and recorded streams can be obtained using SDK API.

* Fetch products during live and recorded stream viewing using Web, iOS and Android SDK API (Refer to respective guides)

Find below comprehensive guide for CMS.

## CMS Guide&#x20;

### Setting up products

1. Login to CMS using provided credentials
2. Click on Users in the left navigation.
3. Click on the shopping cart button to add / edit / deactivate products.
4. Verify all the products, before the host can go live. All active products will be exported to live stream. For product images, we recommend size of `400x400 pixels`

Click Shopping cart button on right

![Shopping cart button on right](assets/images/live_shop1.png)

Click on Add product to add new product or edit existing products 

![Click on Add product to add new product or edit existing products](assets/images/live_shop2.png)

Fill in all the required information and click save. In case of no promotion price, just enter 0

![Fill in all the required information and click save. In case of no promotion price, just enter 0](assets/images/live_shop3.png)

Verify activated products and Go Live!

![Verify activated products and Go Live!
](assets/images/live_shop4.png)

### Manage products during live broadcast

Go to Live Broadcast section in left menu. Click on shopping cart next to live stream. You can edit product information during live stream. After editing product, click on save. You can also mark product to be out of stock during live. It will be reflected on both host / broadcaster and Viewer side using SDK API.

Live Broadcast section

![Live Broadcast section](assets/images/live_shop5.png)

Edit stream products during live stream and then press save

![Edit stream products during live stream and then press save](assets/images/live_shop6.png)

### Manage products of archived stream

Go to Archive section on left menu of CMS. Click on shopping cart next to archived stream. You can edit product information for archived streams

Archived streams

![Archived streams](assets/images/live_shop7.png)

[Edit stream products in archived streams and click save

![Edit stream products in archived streams and click save](assets/images/live_shop8.png)

<div id="WATCH_STREAM"></div><script src="https://lora-sdk.belive.sg/player-widget/staging/player.min.js"></script><script> const player = BeLivePlayerWidget.initialize({    showId: "8a467150-dc16-4eab-ad7f-5048ff60dd2e",    container: document.getElementById("WATCH_STREAM"), });</script>