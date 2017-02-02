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
$ echo "_your_client_id_here:_your_client_secret_here"  | base64
Y2xpZW50X2lkOmNsaWVudF9zZWNyZXQK
```

> Use the received base64 encoded basic-auth token to get access_token:

```shell
$ curl -X POST 'https://accounts-staging.tokopedia.com/token' -H \
	"Authorization: Basic Y2xpZW50X2lkOmNsaWVudF9zZWNyZXQK" -d \
		'grant_type=client_credentials'
{"access_token":"LSPr7x7sRGaewzwZE6IcuA","expires_in":2592000,\
	"token_type":"Bearer"}
```

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl -XGET -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
	http://fs-staging.tokopedia.net/v1/sample/endpoint
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
shop_id | - | shop id
from_date | - | UNIX timestamp of date (hour,min,sec) from which the order details are requested
to_date | - | UNIX timestamp of date (hour,min,sec) to which the order details are requested

Timestamps follow regular UNIX timestamp format.

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
	http://fs-staging.tokopedia.net/v1/order/list?shop_id=\
		_your_shop_id&from_date=1481760000&to_date=1481814000
```

> The above command returns JSON structured like this:

```json
{
    "config": null,
    "data": [{
        "fs_id": "",
        "order_id": 12443547,
        "invoice_ref_num": "INV/20161215/XVI/XII/12443550",
        "products": [{
            "id": 14265919,
            "name": "Puding",
            "quantity": 3,
            "notes": "",
            "price": 100
        }],
        "device_type": "",
        "customer": {
            "id": 5480358,
            "name": "Emma Watson",
            "phone": "081939164777"
        },
        "shop_id": 479094,
        "payment_id": 10540008,
        "recipient": {
            "name": "Tifa",
            "address": "Jl. R. A. Cipocok Jaya, 42118, Banten",
            "phone": "081294647164"
        },
        "logistics": {
            "shipping_id": 1,
            "shipping_agency": "JNE",
            "service_type": "regular"
        },
        "amt": {
            "ttl_product_price": 300,
            "shipping_rate": 10,
            "insurance_price": 0,
            "ttl_amount": 310
        },
        "order_status": 400,
        "create_time": 1481774373,
        "custom_fields": null
    }],
    "server_process_time": 0,
    "status": "OK",
    "error_message": []
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

After registration is successful, you will receive notifications for new order, order cancellations and order status updates from tokopedia on respective endpoints you registered. You will also be able to accept/reject order or update order status for each order notification received.

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    http://fs-staging.tokopedia.net/v1/fs/9999999/register -d\'{
    	"fs_id":1001,
    	"order_actions_url":"http://yourstore.com/v1/order/actions",
    	"order_notification_url":"http://yourstore.com/v1/order/notify",
    	"order_cancellation_url":"http://yourstore.com/v1/order/cancel",
    	"order_status_url":"http://yourstore.com/v1/order/status"
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

### Form Data Parametes (JSON)

Parameter | Description
--------- | -----------
fs_id | fulfillment service id
shop_id | shop id
order_id | order id
products | products. each product contains `product_id`, `quantity_deliver` and `quantity_reject`
order_status | order status - either 400 for accepting complete order or 401 for accepting partial order.
reason | reason associated with e.g. accepting partial order
update_time | time of acceptance of order in `YYYY-MM-DD HH:mm:SS` fmt

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    http://fs-staging.tokopedia.net//v1/order/12345/fs/999999/ack -d\'{
		"fs_id": 999999, 
		"order_id": 12345, 
		"products": [
			{
				"product_id": 12,
				"quantity_deliver": 1, 
				"quantity_reject": 0
			}, 
			{
				"product_id": 14, 
				"quantity_deliver": 2, 
				"quantity_reject" 0
			}
		], 
		"order_status": 400, 
		"reason": "", 
		"update_time": "2017-01-01 18:00:10"
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
fs_id | fulfillment service id
shop_id | shop id
order_id | order id
products | products. optional. 
order_status | order status - status should be `10`
reason | reason associated with e.g. accepting partial order
update_time | time of acceptance of order in `YYYY-MM-DD HH:mm:SS` fmt

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    http://fs-staging.tokopedia.net//v1/order/12345/fs/999999/nack -d\'{
		"fs_id": 999999, 
		"order_id": 12345, 
		"reason": "not enough stock.", 
		"update_time": "2017-01-01 18:00:10"
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
fs_id | fulfillment service id
shop_id | shop id
order_id | order id
order_status | order status - either 400 for accepting complete order or 401 for accepting partial order.
reason | reason associated with e.g. accepting partial order
update_time | time of acceptance of order in `YYYY-MM-DD HH:mm:SS` fmt

<aside class="success">
Remember — this is authenticated request!
</aside>

```shell
curl -H "Authorization: Bearer LSPr7x7sRGaewzwZE6IcuA" \
    http://fs-staging.tokopedia.net//v1/order/12345/fs/999999/status -d\'{
		"fs_id": 999999, 
		"order_id": 12345, 
		"order_status": 500, 
		"reason": "", 
		"update_time": "2017-01-01 18:00:10"
}'
```

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
payment_id | int64 | payment id
logistics | Logistics | logistics data, ref: logistics structure below
amt | Amount | amount data, ref: amount structure below
device_type | string | device type of user 
order_status | int | order status 
create_time | time.Time | time in format 'YYYY-MM-DD HH:mm:SS" 
custom_fields | map[string]string | a map of string to string for custom fields for future

```json
{
		"fs_id": "",
		"order_id": 12443547,
		"invoice_ref_num": "INV/20161215/XVI/XII/12443550",
		"products": [{
			"id": 14265919,
			"name": "Puding",
			"quantity": 3,
			"notes": "",
			"price": 100
		}],
		"device_type": "",
		"customer": {
			"id": 5480358,
			"name": "Emma Watson",
			"phone": "081939164777"
		},
		"shop_id": 479094,
		"payment_id": 10540008,
		"recipient": {
			"name": "Tifa",
			"address": "Jl. R. A. ipocok Jaya, 42118, Banten",
			"phone": "081294647164"
		},
		"logistics": {
			"shipping_id": 1,
			"shipping_agency": "JNE",
			"service_type": "regular"
		},
		"amt": {
			"ttl_product_price": 300,
            "shipping_rate": 10,
            "insurance_price": 0,
            "ttl_amount": 310
        },
        "order_status": 400,
        "create_time": "2017-01-01 18:00:10",
        "custom_fields": {
		}
}

```

### Order Ack  

Field | Type | Description 
----- | ---- | -----------
fs_id | int64 | fulfillment service id
shop_id | int64 | shop id 
order_id | int64 | order id
products | []ProductFulfilled | this is only valid for ack. absent for cancellation, nack and status. 
order_status | int | order status
reason | string | reason 
update_time | time.Time | time in format 'YYYY-MM-DD HH:mm:SS"

```json
{
	"fs_id": 99999,
	"shop_id": 99999,
	"order_id": 12443547,
    "products": [{
            "id": 14265919,
            "quantity_deliver": 3,
            "quantity_reject": 1
	}],
	"order_status": 410,
	"reason": "",
    "update_time": "2017-01-01 18:00:10"
}
```

### Order Nack  

Field | Type | Description 
----- | ---- | -----------
fs_id | int64 | fulfillment service id
shop_id | int64 | shop id 
order_id | int64 | order id
reason | string | reason 
update_time | time.Time | time in format 'YYYY-MM-DD HH:mm:SS"

```json
{
	"fs_id": 99999,
	"shop_id": 99999,
	"order_id": 12443547,
	"reason": "no stock available",
    "update_time": "2017-01-01 18:00:10"
}
```

### Order Status  

Field | Type | Description 
----- | ---- | -----------
fs_id | int64 | fulfillment service id
shop_id | int64 | shop id 
order_id | int64 | order id
order_status | int | order status
reason | string | reason 
update_time | time.Time | time in format 'YYYY-MM-DD HH:mm:SS"

```json
{
	"fs_id": 99999,
	"shop_id": 99999,
	"order_id": 12443547,
	"order_status": 500,
	"reason": "",
    "update_time": "2017-01-01 18:00:10"
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



