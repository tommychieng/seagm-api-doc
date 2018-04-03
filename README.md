# Rest API for SEAGM (2018-02-07)
# General API Information
* The base endpoint is: **https://api.seagm.io/**
* Sandbox service endpoint is: **https://api.sandbox.seagm.io/**
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. newest first, oldest last.
* All time and timestamp related fields are in seconds.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded`. You may mix parameters between both the
  `query string` and `request body` if you wish to do so.
* Parameters may be sent in any order.
* If a parameter sent in both the `query string` and `request body`, the
  `query string` parameter will be used.

# ERROR CODE
* HTTP `4XX` return codes are used for for malformed requests;
  the issue is on the sender's side.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `418` return code is used when an IP has been auto-banned for continuing to send requests after receiving `429` codes.
* HTTP `5XX` return codes are used for internal errors; the issue is on
  Binance's side.
* HTTP `504` return code is used when the API successfully sent the message
but not get a response within the timeout period.
It is important to **NOT** treat this as a failure; the execution status is
**UNKNOWN** and could have been a success.
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "code": 404,
  "message": "Invalid parameter."
}
```
* Specific error codes
## 1xxx - Network issues
#### 1000 UNKNOWN
 * An unknown error occured while processing the request.
#### 1001 DISCONNECTED
 * Internal error; unable to process your request. Please try again.
#### 1002 UNAUTHORIZED
 * You are not authorized to execute this request.
#### 1003 TOO_MANY_REQUESTS
 * Too many requests.
 * Too many requests queued.
 * Too many requests; current limit is %s requests per minute. Please use the websocket for live updates to avoid polling the API.
 * Way too many requests; IP banned until %s. Please use the websocket for live updates to avoid bans.
#### 1006 UNEXPECTED_RESP
 * An unexpected response was received from the message bus. Execution status unknown.
#### 1007 TIMEOUT
 * Timeout waiting for response from backend server. Send status unknown; execution status unknown.
#### 1013 INVALID_MESSAGE
 * INVALID_MESSAGE
#### 1014 UNKNOWN_ORDER_COMPOSITION
 * Unsupported order combination.
#### 1015 TOO_MANY_ORDERS
 * Too many new orders.
 * Too many new orders; current limit is %s orders per %s.
#### 1016 SERVICE_SHUTTING_DOWN
 * This service is no longer available.
#### 1020 UNSUPPORTED_OPERATION
 * This operation is not supported.
#### 1021 INVALID_TIMESTAMP
 * Timestamp for this request is outside of the recvWindow.
 * Timestamp for this request was 1000ms ahead of the server's time.
#### 1022 INVALID_SIGNATURE
 * Signature for this request is not valid.

## 2xxx - Success
 * 请求成功
#### 2000 Get Success
 * 获取数据成功
#### 2010 Post Success
 * 订单下单成功

## 4xxx - Request issues
 * 请求存在问题
#### 4000 Error
 * 出现错误
#### 4010 缺少UID或者UID无效
#### 4040 请求的内容不存在
#### 4050 请求的方式不正确
#### 4060 insufficient credit

## 5xxx - Server issues
#### 5000 服务器故障

# CURRENCY
* Can only use SEAGM credits to buy cards

# LIMITS
* A 429 will be returned when either rather limit is violated.
* When a 429 is recieved, it's your obligation as an API to back off and not spam the API.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (http status 418).**
* IP bans are tracked and **scale in duration** for repeat offenders, **from 2 minutes to 3 days**.

# Endpoint security type
* uid are passed into the Rest API via the query string uid.
* secret_key **are case sensitive**.

# SIGNED Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the second timestamp of when the request was created and sent.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 120) && timestamp > (serverTime - 120)) {
    // process request
  } else {
    // reject request
  }
  ```

**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server.

## Examples
### SIGNED Endpoint Examples for GET /v1/me
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
uid | 10000
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
timestamp | 1499827319559
uid | 10000

* **queryString:** timestamp=1499827319559&uid=10000
* **HMAC SHA256 signature:**
    ```
    [linux]$ echo -n "timestamp=1499827319559&uid=10000" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```
    
* **curl command:**
    ```
    (HMAC SHA256)
    [linux]$ curl -X POST 'https://api.seagm.com/v1/me?timestamp=1499827319559&uid=10000&sign=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

### SIGNED Endpoint Examples for POST /v1/orders
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
uid | 10000
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
timestamp | 1499827319559
uid | 10000

Body | Value
------------ | ------------
type_id | 1
buy_amount | 10

* **queryString:** uid=10000&timestamp=1499827319559&sign=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
* **requestBody:** type_id=1&buy_amount=10
* **HMAC SHA256 signature:**
    ```
    [linux]$ echo -n "buy_amount=10&timestamp=1499827319559&type_id=1&uid=10000" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77
    ```
    
* **curl command:**
    ```
    (HMAC SHA256)
    [linux]$ curl -X POST 'https://api.seagm.com/v1/orders?uid=10000&timestamp=1499827319559&sign=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71' -d 'type_id=1&buy_amount=10'
    ```

# Public API Endpoints
## ENUM definitions
**Order status:**

* NEW
* PARTIALLY_FILLED
* FILLED
* CANCELED
* REJECTED
* EXPIRED

## General endpoints
### Test connectivity
```
GET /ping
```
Test connectivity to the Rest API.

**Parameters:**
NONE

**Response:**
```javascript
{}
```

### Check server time
```
GET /time
```
Test connectivity to the Rest API and get the current server time.

**Parameters:**
NONE

**Response:**
```javascript
{
  "serverTime": 1499827319
}
```

### Get account details
```
GET /v1/me?uid=10001&timestamp=xx&sign=xx
```
获取余额等账户信息。

**Parameters:**
NONE

**Rresponse:**
```javascript
{
    "code": 200, 
    "data": {
        "credits": 1000, 
        "email": "fairynomore@seagm.com", 
        "id": 10001, 
        "username": "fairynomore"
    }
}
```

### Get card category list
```
GET /v1/card-categories 
```

**Response:**
```javascript
{
    "code": 200, 
    "data": [
        {
            "id": 1, 
            "name": "Cubizone Cubicards (MY)"
        }, 
        {
            "id": 3, 
            "name": "A-Cash (SEA)"
        }, 
        {
            "id": 4, 
            "name": "GoPlayPlay "
        }, 
        {
            "id": 6, 
            "name": "MyCard (TW)"
        }, 
        .....
 ```

### Get card category details
```
GET /v1/card-categories/:id 
```
**Response:**
```javascript
{
    "code": 200, 
    "data": {
        "id": 1, 
        "name": "Cubizone Cubicards (MY)"
    }
}
```

### Get card type list of specified card category
```
GET /v1/card-categories/:id/card-types
```

**Response:**
```javascript
{
    "code": 200, 
    "data": [
        {
            "id": 1, 
            "categoryId": 1, 
            "name": "Cubizone 100 Cubits", 
            "unitPrice": 0, 
            "parValue": 10, 
            "parValueCurrency": "MYR", 
            "maxAmount": 0
        }, 
        {
            "id": 2, 
            "categoryId": 1, 
            "name": "Cubizone 200 Cubits", 
            "unitPrice": 0, 
            "parValue": 20, 
            "parValueCurrency": "MYR", 
            "maxAmount": 0
        },
        ...
 ```

### Get card type details
```
GET /v1/card-types/:id 
```
**Response:**
```javascript
{
    "code": 200, 
    "data": {
        "id": 1, 
        "categoryId": 1, 
        "name": "Cubizone 100 Cubits", 
        "unitPrice": 0, 
        "parValue": 10, 
        "parValueCurrency": "MYR", 
        "maxAmount": 0
    }
}
```

### Create order
```
POST /v1/card-orders 
```
**Response:**
```javascript
{
    "code": 200, 
    "data": {
        "id": 10026, 
        "status": "new",
        "title": "Cubicards (MY) 100 cubits",
        "buy_amount": 2,
        "cards": [
            {
                "sn": "xxxxx",
                "pin": "xxxxx",
                "expired": "2018-12-30"
            },
            {
                "sn": "xxxxx",
                "pin": "xxxxx",
                "expired": "2018-12-30"
            }
        ]
    }
}
```

### Get order list
```
GET /v1/card-orders
```
**Parameters:**

Parameter | Value
------------ | ------------
offset | 0
limit | 12

**Response:**
```javascript
{
    "code": 200, 
    "data": {
        "total": 244,
        "begin": 0, 
        "end": 12,
        "orders": [
            {
                "id": 10021, 
                "title": ""
            }, 
            {
                "id": 10026, 
                "title": "Cubicards (MY) 100 cubits"
            }, 
            {
                "id": 10512, 
                "title": "MyCard (TW/HK) Lineage II 18 Hours (MY) = 5,000 iCoins"
            }, 
            {
                "id": 10513, 
                "title": "MyCard (TW/HK) Lineage II 18 Hours (MY) = 5,000 iCoins"
            }, 
            {
                "id": 69507, 
                "title": "A-Cash (SEA) A-Cash 5,000"
            }, 
            {
                "id": 134549, 
                "title": "NCsoft  Cash (Global)  Aion Online Cd Key - Aion Standard Edition [US] "
            }, 
            {
                "id": 134550, 
                "title": "NCsoft  Cash (Global)  Aion Online Cd Key - Aion Standard Edition [US] "
            }, 
            {
                "id": 324083, 
                "title": "Rixty Code RixtyCode USD$5"
            }, 
            {
                "id": 465704, 
                "title": "A-Cash (SEA) A-Cash 5,000"
            }, 
            {
                "id": 606426, 
                "title": "9 Card 9卡150點"
            }, 
            {
                "id": 609809, 
                "title": "9 Card 9卡150點"
            }, 
            {
                "id": 609810, 
                "title": "9 Card 9卡400點送金庸究极2.0版400點禮包"
            }
        ]
    }
}
 ```

### Get order details
```
GET /v1/card-orders/:id 
```
**Response:**
```javascript
{
    "code": 200, 
    "data": {
        "id": 10026, 
        "status": "completed",
        "title": "Cubicards (MY) 100 cubits",
        "buy_amount": 2,
        "cards": [
            {
                "sn": "xxxxx",
                "pin": "xxxxx",
                "expired": "2018-12-30"
            },
            {
                "sn": "xxxxx",
                "pin": "xxxxx",
                "expired": "2018-12-30"
            }
        ]
    }
}
```
