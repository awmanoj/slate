---
title: Fulfillment Service API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Tokopedia Fulfillment API! You can use these API to connect your backend systems with Tokopedia systems.


<aside class="notice">
All requests to fulfillment controller are authenticated and authorized using a mechanism discussed in <a href="#authentication">Authentication</a> section.
</aside>

# Authentication

Authentication/Authorization is handled by OAuth from Tokopedia Accounts module.

1. Apply for a `fs_id`, `client_id` and `client_secret` from Tokopedia. Contact us. 

2. Generate a basic authentication code.

3. Use the received base64 encoded basic-auth token to get access_token.

4. To authorize further requests, Use this access_token.

> As an example, generate a basic authentication code by using following command line (on Linux):

```shell
1. Go to https://www.base64encode.org/
2. Generate base64 token for: _your_client_id_here:_your_client_secret_here" 
3. For example, it generates: Y2xpZW50X2lkOmNsaWVudF9zZWNyZXQK
```

> Use the received base64 encoded basic-auth token to get access_token:

Staging Url: https://accounts-staging.tokopedia.com/token
Production Url: https://accounts.tokopedia.com/token 

```shell
$ curl -X POST 'https://accounts.tokopedia.com/token' -H \
	"Authorization: Basic Y2xpZW50X2lkOmNsaWVudF9zZWNyZXQK" -d \
		'grant_type=client_credentials'
{"access_token":"LSPr7x7sRGaewzwZE6IcuA","expires_in":2592000,\
	"token_type":"Bearer"}
```

> To authorize, use this code:

Staging Url: https://fs-staging.tokopedia.net
Product Url: https://fs.tokopedia.net

```shell
# With shell, you can just pass the correct header with each request
curl -XGET -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
	https://fs.tokopedia.net/v1/sample/endpoint
```

<aside class="notice">
You must replace <code>_your_client_id_here</code> and <code>_your_client_secret_here</code> with your API key. Also, follow step 2 uptil 4 to generate your access token. 
</aside>

<aside class="notice">
Please note that access token has an expiry and you will need to renew the access token following step 3, on expiry.
</aside>

<aside class="warning">
Don't share your <code>client_secret</code> and <code>access_token</code> with anyone. Protect it. 
</aside>

# Order 

You can use Order API for new order event notifications, accepting/rejecting an order, getting or updating an order status etc. 

<img src="images/workflow.png"/>

Notes: 

1. Use <a href="#register">Register</a> to register your merchant fulfillment service to Tokopedia fulfillment controller. 

2. Once registered, you start receiving new order notifications as a `POST` http request on the `order_notification_url` endpoint specified in the <a href="#register">Register</a> call. The data received in this notification is in <a href="#notification-data-schema">Order Notification</a> format (ref section: `Order Notification`).

3. On receiving an order notification it is the responsibility of Merchant fulfillment service to ACK (accept) or NACK (reject) the order by calling <a href="#ack">ACK</a> or <a href="#nack">NACK</a> endpoint passing the data in <a href="#notification-data-schema">order status</a> format (ref section: `Order Ack` and `Order Nack`).

4. On any status update on merchant end (e.g. product shipped to customer) they should call <a href="#status">Status</a> endpoint with data in <a href="#notification-data-schema">order status</a> format (ref section: `Order Status`).

5. If there is a cancellation (e.g. from resolution center or requested by buyer) from Tokopedia end, then a notification will be received as a `POST` http request on the `order_cancellation_url` endpoint specified in the <a href="#register">Register</a> call. The data received in this notification is in <a href="#notification-data-schema">Order Notification</a> format (ref section: `Order Status`).

6. If there is a status change from Tokopedia end, then a notification will be received as a `POST` http request on the `order_status_url` endpoint specified in the <a href="#register">Register</a> call. The data received in this notification is in <a href="#notification-data-schema">Order Notification</a> format (ref section: `Order Status`).

## Get All Orders 

This endpoint retrieves all orders for your shop between given timestamps. 

### HTTP Request

`GET /v1/order/list`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
fs_id | - | FS id
from_date | - | UNIX timestamp of date (hour,min,sec) from which the order details are requested
to_date | - | UNIX timestamp of date (hour,min,sec) to which the order details are requested

Timestamps follow regular UNIX timestamp format.

### Pagination 

For pagination, use `page=` and `per_page=` query parameters.

Staging Url: https://fs-staging.tokopedia.net
Product Url: https://fs.tokopedia.net

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
	https://fs.tokopedia.net/v1/order/list?fs_id=\
		_your_shop_id&from_date=1481760000&to_date=1481814000&page=1&per_page=1
```

> The above command returns JSON structured like this:

```json
{
    "jsonapi": {
        "version": "1.0"
    },
    "meta": null,
    "data": [{
    "fs_id": 12002,
    "order_id": 12463948,
    "accept_partial": false,
    "invoice_ref_num": "INV/20170505/XVII/V/12463898",
    "products": [{
        "id": 14286522,
        "name": "Product A",
        "quantity": 1,
        "notes": "",
        "weight": 0.01,
        "total_weight": 0.01,
        "price": 10000,
        "total_price": 10000,
        "currency": "Rp"
    }],
    "buyer": {
        "id": 5484718,
        "name": "Franky Buyer",
        "phone": "08128533762",
        "email": "franky.wibisono+buyer@tokopedia.com"
    },
    "recipient": {
        "name": "Franky",
        "address": {
            "address_full": "Wisma 7 Tower 2 Lantai 6",
            "district": "Palmerah",
            "city": "Jakarta Barat",
            "province": "DKI Jakarta",
            "country": "Indonesia",
            "postal_code": "11410"
        },
        "phone": "08128533762"
    },
    "shop_id": 479569,
    "shop_name": "Orami Official Store",
    "payment_id": 11521618,
    "logistics": {
        "shipping_id": 1,
        "shipping_agency": "JNE",
        "service_type": "Reguler"
    },
    "amt": {
        "ttl_product_price": 10000,
        "shipping_cost": 9000,
        "insurance_cost": 0,
        "ttl_amount": 19000,
        "voucher_amount": 0,
        "toppoints_amount": 0
    },
    "dropshipper_info": {
        "name": "",
        "phone": ""
    },
    "voucher_info": {
        "voucher_type": 0,
        "voucher_code": ""
    },
    "device_type": "default_v3",
    "order_status": 220,
    "create_time": "2017-05-05 18:06:51",
    "custom_fields": {}
    }],
    "links": {
        "self": "",
        "related": "",
        "first": "",
        "last": "",
        "prev": "",
        "next": ""
    }
}
```

## Register 

For receiving notifications for new order, order cancellations and order status changes from tokopedia, you will need to first register as a fulfillment service by following endpoint:

### HTTP Request

`POST /v1/fs/:fs_id/register`

### Path Parameters

Parameter | Description
--------- | -----------
fs_id | fulfillment service id 

### Form Data Parametes (JSON)

Parameter | Description
--------- | -----------
fs_id | fulfillment service id
order_notification_url | callback endpoint on which to receive new payment verified order notifications 
order_cancellation_url | callback endpoint on which to receive order cancellation notifications
order_status_url | callback endpoint on which to receive order status notification
order_actions_url | callback endpoint that provides all actions' updated URLs
webhook_secret | secret for generating HMAC while making callbacks

After registration is successful, you will receive notifications for new order, order cancellations and order status updates from tokopedia on respective endpoints you registered. You will also be able to accept/reject order or update order status for each order notification received.

Staging Url: https://fs-staging.tokopedia.net
Product Url: https://fs.tokopedia.net

<aside class="notice">
Security:

Registered callbacks are protected from MOTM attacks using HMAC authorization. All HTTP requests to callbacks will contain the authorization header "Authorization: TKPD Tokopedia:" followed by hmac code. This should be verified on the fulfillment service (merchant) end.
</aside>

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    https://fs.tokopedia.net/v1/fs/9999999/register -d\'{
    	"fs_id":1001,
    	"order_actions_url":"http://yourstore.com/v1/order/actions",
    	"order_notification_url":"http://yourstore.com/v1/order/notify",
    	"order_cancellation_url":"http://yourstore.com/v1/order/cancel",
    	"order_status_url":"http://yourstore.com/v1/order/status",
		"webhook_secret": "abracadabr@"
}'
```

## Ack 

Acknowledge the order (Partially/Fully accept the order)

### HTTP Request

`POST /v1/order/:order_id/fs/:fs_id/ack`

### Path Parameters

Parameter | Description
--------- | -----------
order_id | order id
fs_id | fulfillment service id

### (Optional) Form Data Parametes (JSON)

Staging Url: https://fs-staging.tokopedia.net
Product Url: https://fs.tokopedia.net

<aside class="notice">
POST data is needed only when merchant accepts partial order.
</aside>

Parameter | Description
--------- | -----------
products | products fulfilled. each product contains `product_id`, `quantity_deliver` and `quantity_reject`
reason | reason (especially for order rejection e.g. Out of stock, Variant unavailable, Wrong price or weight, Shop closed etc.

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    https://fs.tokopedia.net//v1/order/12345/fs/999999/ack -d\'{
		"products": [
			{
				"product_id": 12,
				"quantity_deliver": 1, 
				"quantity_reject": 1
			}, 
			{
				"product_id": 14, 
				"quantity_deliver": 2, 
				"quantity_reject" 0
			}
		]
}'
```

## Nack 

Negative acknowledge the order (reject the order)

### HTTP Request

`POST /v1/order/:order_id/fs/:fs_id/nack`

### Path Parameters

Parameter | Description
--------- | -----------
order_id | order id
fs_id | fulfillment service id

### Form Data Parametes (JSON)

Parameter | Description
--------- | -----------
reason_code | must be a integer as mentioned in table below.
reason | reason (especially for order rejection e.g. Out of stock, Variant unavailable, Wrong price or weight, Shop closed etc.
shop_close_end_date | if reason_code is 4 i.e. shop closed (Details below) then it is mandatory to provide shop close end date. date string with layout DD/MM/YYYY e.g. 17/05/2017.
shop_close_note | if reason_code is 4 i.e. shop closed (Details below) then it is mandatory to provide shop close note. max length 100 characters.

### Reason Codes 

Code  | Description
------| -------------
0 | Only for reject shipping case - reason required with max length 512 characters
1 | Product(s) out of stock - list_prd required
2 | Product variant unavailable
3 |	Wrong price or weight
4 | Shop closed - close_end and closed_note required
7 |	Courier problem - reason required with max length 490 characters
8 |	Buyer's request - reason required with max length 490 characters

Staging Url: https://fs-staging.tokopedia.net
Product Url: https://fs.tokopedia.net

<aside class="success">
Remember — this is authenticated request!
</aside>


```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    https://fs.tokopedia.net//v1/order/12345/fs/999999/nack -d'{
		"reason_code": 1,
		"reason": "out of stock" 
}'
```

## Status

Update the status of an order 

### HTTP Request

`POST /v1/order/:order_id/fs/:fs_id/status`

### Path Parameters

Parameter | Description
--------- | -----------
order_id | order id
fs_id | fulfillment service id

### Form Data Parametes (JSON)

Parameter | Description
--------- | -----------
order_status | Order Status. Only valid values: 500 (confirm shipping)
shipping_ref_num | AWB number if the status is to indicate that products have shipped

Staging Url: https://fs-staging.tokopedia.net
Product Url: https://fs.tokopedia.net

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    https://fs.tokopedia.net//v1/order/12345/fs/999999/status -d\'{
		"order_status": 500,
		"shipping_ref_num": "BCD42324C",
}'
```

## Order Status Codes

Code | Description
-----| -----------
400 | seller accepted the order            
401 | seller accepted the order, partially           
 10 | seller rejected the order
500 | seller confirm for shipment            

## Order Reject Reason (possible reasons)

S. No. | Description 
-------|------------
01 | Out of stock - Once selected, Seller need to update the selected item to not available.
02 | Variant unavailable - Once selected, Seller can edit the Product Description or update the selected item to not available.
03 | Wrong price or weight - Once selected, Seller can edit the Product Price and Weight, or update the selected item to not available.
04 | Shop closed - Once selected, Seller can choose between Close Now or Managed Out Of Stock Product. If Close Now, Seller need to provide the From Date, Until Date, and Notes. Other, Seller need to update the selected item to not available.
05 | Courier problem - Once selected, Seller need to provide some Notes.
06 | Buyer request - Once selected, Seller need to provide some Notes. This option must be trigger by Buyer's Cancellation Request after Payment Verified.

## Notification Data Schema 

There are two types of notifications primarily - new order events (post payment-verification) and order status updates. Latter is used for cancellation update, status update. Former is used only for new order event notification. Here, we discuss these schema. On the right, you can see reference JSON.

### Order Notification 

Field | Type | Description 
----- | ---- | -----------
fs_id | int64 | fulfillment service id 
order_id | int64 | order id
invoice_ref_num | string | invoice reference number 
products | []Product | products data, ref: product structure below
customer | Customer | customer data, ref: customer structure below
shop_id | int64 | shop id
shop_name | string | shop name
payment_id | int64 | payment id
logistics | Logistics | logistics data, ref: logistics structure below
amt | Amount | amount data, ref: amount structure below
device_type | string | device type of user 
order_status | int | order status 
create_time | time.Time | time in format 'YYYY-MM-DD HH:mm:SS" 
custom_fields | map[string]string | a map of string to string for custom fields for future

```json
{
    "fs_id": 12002,
    "order_id": 12463948,
    "accept_partial": false,
    "invoice_ref_num": "INV/20170505/XVII/V/12463898",
    "products": [{
        "id": 14286522,
        "name": "Product A",
        "quantity": 1,
        "notes": "",
        "weight": 0.01,
        "total_weight": 0.01,
        "price": 10000,
        "total_price": 10000,
        "currency": "Rp"
    }],
    "buyer": {
        "id": 5484718,
        "name": "Franky Buyer",
        "phone": "08128533762",
        "email": "franky.wibisono+buyer@tokopedia.com"
    },
    "recipient": {
        "name": "Franky",
        "address": {
            "address_full": "Wisma 7 Tower 2 Lantai 6",
            "district": "Palmerah",
            "city": "Jakarta Barat",
            "province": "DKI Jakarta",
            "country": "Indonesia",
            "postal_code": "11410"
        },
        "phone": "08128533762"
    },
    "shop_id": 479569,
    "shop_name": "Orami Official Store",
    "payment_id": 11521618,
    "logistics": {
        "shipping_id": 1,
        "shipping_agency": "JNE",
        "service_type": "Reguler"
    },
    "amt": {
        "ttl_product_price": 10000,
        "shipping_cost": 9000,
        "insurance_cost": 0,
        "ttl_amount": 19000,
        "voucher_amount": 0,
        "toppoints_amount": 0
    },
    "dropshipper_info": {
        "name": "",
        "phone": ""
    },
    "voucher_info": {
        "voucher_type": 0,
        "voucher_code": ""
    },
    "device_type": "default_v3",
    "order_status": 220,
    "create_time": "2017-05-05 18:06:51",
    "custom_fields": {}
}

```

### Order Ack  

Field | Type | Description 
----- | ---- | -----------
products | []ProductFulfilled | this is only valid for ack. absent for cancellation, nack and status. 

```json
{
    "products": [{
            "id": 14265919,
            "quantity_deliver": 3,
            "quantity_reject": 1
	}]
}
```

### Order Nack  

```json
{
	"reason_code": 4, 
	"reason": "shop is closed on the occasion of liburan",
	"shop_close_end_date": "13/07/2017",
	"shop_close_note": "Maf Pak, shop saya tutup untuk liburan"
}
```

### Order Status  

Field | Type | Description 
----- | ---- | -----------
shipping_ref_num | string | AWB number

```json
{
	"shipping_ref_num": "BCD324243D",
}
```


### Product 

Field | Type | Description 
----- | ---- | -----------
id | int64 | product id
name | string | product name 
quantity | int64 | product quantity 
notes | string | product notes 
price | float64 | product price 
total_price | float64 | total price 
currency | string | currency code 

### Customer 

Field | Type | Description
----- | ---- | -----------
id | int64 | customer id 
name | string | customer name 
phone | string | customer phone

### Logistics 

Field | Type | Description
----- | ---- | -----------
shipping_id | int64 | shipping id
shipping_agency | string | shipping agency 
service_type | string | service type 

### Amount

Field | Type | Description
----- | ---- | -----------
ttl_product_price | float64 | total product price 
shipping_rate | float64 | shipping rate
insurance_price | float64 | insurance price 
ttl_amount | float64 | total amount 

### ProductFulfilled 
 
Field | Type | Description
----- | ---- | -----------
product_id | int64 | product id 
quantity_deliver | int64 | quantity that is accepted by seller (to be delivered)
quantity_reject | int64 | quantity that is rejected by seller (not to be delivered)



