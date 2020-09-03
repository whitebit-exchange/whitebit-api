# Private HTTP API V4

## Private endpoints V4 for Main balance changes

* [Main balance](#main-balance)
* [Get cryptocurrency deposit address](#get-cryptocurrency-deposit-address)
* [Get fiat deposit address](#get-fiat-deposit-address)
* [Create withdraw request](#create-withdraw-request)
* [Transfer between main and trade balances](#transfer-between-main-and-trade-balances)
* [Get deposit/withdraw history](#get-depositwithdraw-history)
* [Create new address for deposit](#create-new-address-for-deposit)
* Codes
    * [Create code](#create-code)
    * [Apply code](#apply-code)
    * [Get my codes](#get-my-codes)
    * [Get codes history](#get-codes-history)
    
Base URL is https://whitebit.com

Endpoint example: https://whitebit.com/api/v4/{endpoint}

All endpoints return time in Unix-time format.

All endpoints return either a __JSON__ object or array.

For receiving responses from API calls please use http method __POST__

#### Error messages V4 format:
___
```json5
{
    "code": 0,
    "errors": {
        "PARAM1": [
            "MESSAGE"
        ],
        "PARAM2": [
            "MESSAGE"
        ]
    },
    "message": "MESSAGE"
}
```
___
### Terminology

#### Pair:

`Stock` - currency that you want to buy or sell

`Money` - currency that you are using to buy or sell something

`Maker` - person who puts an order and waiting till this order will be finished

`Taker` - person who finishes the existing order

`Precision` - is the number of digits to the right of the decimal point

`Bid` - buy order

`Ask` - sell order

`Limit order` - to place this order, you need to fill in the 'Price' and 'Amount' fields. If this order finds a corresponding order on the opposite side, it will be executed. Otherwise it will be placed into the orderbook.

`Fiat` - is a currency (a medium of exchange) established as money, often by government regulation, but that does not have intrinsic value (value independent of the nominal value, such as a precious metal might have).

`Provider` - fiat currencies has different providers that helps people making deposits and withdraws by bank transfers.

`Multinetwork` - cryptocurrency like USDT obtained in different networks, like: OMNI, Tron, Ethereum etc. Network should be selected in order to make a deposit or withdraw.

`Main balance` - balance on exchange that accepts deposits and/or withdraws.

`Memo` - some currencies like XLM can create only one address for exchange. So for detecting which transaction is yours exchanges uses additional data - memo.


___
### Main balance

```
[POST] /api/v4/main-account/balance
```
This endpoint retrieves the main balance by currency ticker or all balances.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ticker | String | **No** | Currency's ticker. Example: BTC

**Request BODY raw:**
```json5
{
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**


Available statuses:
* `Status 200`
* `Status 400 if request validation failed`

```json5
{
    "BSV": {
        "main_balance": "0"           // main balance volume of BSV
    },
    "BTC": {
        "main_balance": "0"           // main balance volume of BTC
    },
    "BTG": {
        "main_balance": "0"           // main balance volume of BTG
    },
    "BTT": {
        "main_balance": "0"           // main balance volume of BTT
    },
    "XLM": {
        "main_balance": "36.48"       // main balance volume of XLM
    },
    "currecty_ticker": {...}
}
```

<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

</details>
___

### Get cryptocurrency deposit address

```
[POST] /api/v4/main-account/address
```
This endpoint retrieves a deposit address of the cryptocurrency.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ticker | String | **Yes** | Currencies ticker. Example: BTC ⚠ Currency ticker should be: not fiat and has "can_deposit" status must be "true". If currency has multinetwork like USDT - you need use multinetwork ticker you can find it in https://whitebit.com/api/v4/public/assets request. Default network for USDT is Ethereum.

**Request BODY raw:**
```json5
{
    "ticker": "BTC",
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 200`
* `Status 400 if request validation failed`
* `Status 503 if service temporary unavailable`

```json5
{
    "account": {
        "address": "GDTSOI56XNVAKJNJBLJGRNZIVOCIZJRBIDKTWSCYEYNFAZEMBLN75RMN",        // deposit address
        "memo": "48565488244493"                                                      // memo if currency requires memo
    },
    "required": {
        "fixedFee": "0",                                                              // fixed deposit fee
        "flexFee": {                                                                  // flexible fee - is fee that use percent rate
            "maxFee": "0",                                                            // maximum fixed fee that you will pay
            "minFee": "0",                                                            // minimum fixed fee that you will pay
            "percent": "0"                                                            // percent of deposit that you will pay
        },
        "maxAmount": "0",                                                             // max amount of deposit that can be accepted by exchange - if you deposit more than that number, it won't be accepted by exchange 
        "minAmount": "1"                                                              // min amount of deposit that can be accepted by exchange - if you will deposit less than that number, it won't be accepted by exchange 
    }
}
```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 1,
    "errors": {
        "ticker": [
            "Currency is not depositable"
        ]
    },
    "message": "Inner validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "Fiat are available on the front-end only"
        ]
    },
    "message": "Validation failed"
}
```

</details>

___


### Get fiat deposit address

```
[POST] /api/v4/main-account/fiat-deposit-url
```
This endpoint retrieves a deposit address of the cryptocurrency.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ticker | String | **Yes** | Currencies ticker. Example: UAH ⚠ Currencies ticker should be: not cypto and has "can_deposit" status must be "true". Use this [url](https://whitebit.com/api/v4/public/assets) to know more about currency.
provider | String | **Yes** | Fiat currency provider. Example: VISAMASTER ⚠ Currency provider should be taken from https://whitebit.com/api/v4/public/assets response.
amount | Numeric String | **Yes** | Deposit amount.
uniqueId | String | **Yes** | Unique transaction identifier on client's side.

**Request BODY raw:**
```json5
{
    "ticker": "UAH",
    "provider": "VISAMASTER",
    "amount": "10",
    "uniqueId": "someID",
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 200`
* `Status 400 if request validation failed`
* `Status 503 if service temporary unavailable`

```json5
{
    "url": "https://someaddress.com"                  // address for deposit
}
```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "Amount is too little for deposit"
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "provider": [
            "Cannot find currency for specified provider"
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "uniqueId": [
            "The unique id has already been taken."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount must be a number."
        ],
        "provider": [
            "The selected provider is invalid."
        ],
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "Amount is too little for deposit"
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 10,
    "message": "Failed to generate deposit url"
}
```

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount field is required."
        ],
        "provider": [
            "The provider field is required."
        ],
        "ticker": [
            "The ticker field is required."
        ],
        "uniqueId": [
            "The unique id field is required."
        ]
    },
    "message": "Validation failed"
}
```

</details>

___

### Create withdraw request

```
[POST] /api/v4/main-account/withdraw
```
This endpoint creates withdraw for the specified ticker.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ticker | String | **Yes** | Currencies ticker. Example: BTC ⚠ Currencies ticker should be: not cypto and has "can_deposit" status must be "true". Use this [url](https://whitebit.com/api/v4/public/assets) to know more about currency.
amount | Numeric string | **Yes** | Deposit amount.
address | String | **Yes** | Target address (wallet address for cryptocurrencies, identifier/card number for fiat currencies)
memo | String | **Yes, if currency is memoable** | Target address (wallet address for cryptocurrencies, identifier/card number for fiat currencies)
uniqueId | String | **Yes** | Unique transaction identifier on client's side.
provider | String | **Yes, if currency is fiat** | Fiat currency provider. Example: VISAMASTER ⚠ Currency provider should be taken from https://whitebit.com/api/v4/public/assets response.
network | String | **No** | Cryptocurrency network. Available for multi network currencies. Example: USDT_OMNI ⚠ Currency network should be taken from https://whitebit.com/api/v4/public/assets response. Default for USDT is USDT_ETH

**Request BODY raw:**
```json5
{
    "ticker": "ETH",
    "amount": "0.9",
    "address": "0x0964A6B8F794A4B8d61b62652dB27ddC9844FB4c",
    "uniqueId" : "24529041",
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if validation succeeded and withdraw creation process is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`

Response error codes:
   * 1 - currency is not withdrawable
   * 2 - specified address is invalid
   * 3 - amount is too small
   * 4 - amount is too small for the payment system
   * 5 - not enough balance 
   * 6 - amount is less than or equals fee
   * 7 - amount should be integer (can happen for currencies with zero precision like Neo)
   * 8 - target withdraw amount without fee equals zero
   * 9 - address is unavailable (occurs for withdraws to own address)

```json5
[
                                // empty array - has success status - go to deposit/withdraw history and check you request status by uniqueId
]
```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "address": [
            "The address field is required."
        ],
        "amount": [
            "The amount field is required."
        ],
        "ticker": [
            "The ticker field is required."
        ],
        "uniqueId": [
            "The unique id field is required."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "uniqueId": [
            "The unique id has already been taken."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 2,
    "errors": {
        "address": [
            "The address is invalid"
        ]
    },
    "message": "Inner validation failed"
}
```

```json5
{
    "code": 5,
    "errors": {
        "amount": [
            "Not enough money, Ethereum balance = 1"
        ]
    },
    "message": "Inner validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "provider": [
            "Provider is required for fiat currency"
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "memo": [
            "The memo field is required."
        ]
    },
    "message": "Validation failed"
}
```

</details>

___

### Transfer between main and trade balances

```
[POST] /api/v4/main-account/transfer
```
This endpoint transfers the specified amount between main and trade balances

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
method | String | **Yes** | Method. Example: **deposit** if you need to transfer from main to trade / **withdraw** if you need to transfer from trade balance to main
ticker | String | **Yes** | Currency's ticker. Example: BTC
amount | Numeric string | **Yes** | Amount to transfer. Max precision = 8, value should be greater than zero and less or equal your available balance.

**Request BODY raw:**
```json5
{
    "ticker": "XLM",
    "amount": "0.9",
    "method": "deposit",
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if all validations succeeded and creating transaction is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`

Response error codes:
   * 1 - transfers from trade to main are disabled
   * 2 - transfers from main to trade are disabled
   * 3 - not enough balance

```json5
[
                                // empty array - has success status
]
```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount field is required."
        ],
        "method": [
            "The method field is required."
        ],
        "ticker": [
            "The ticker field is required."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 3,
    "errors": {
        "amount": [
            "You don't have such amount for transfer (available 34.68, in amount: 1000000)"
        ]
    },
    "message": "Inner validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "method": [
            "The selected method is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount must be at least 0.00000001."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount must be a number.",
            "Invalid number"
        ]
    },
    "message": "Validation failed"
}
```

</details>

___

### Get deposit/withdraw history

```
[POST] /api/v4/main-account/history
```
This endpoint retrieves the history of deposits and withdraws

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
transactionMethod | Number | **Yes** | Method. Example: **1** if need to display deposits / **2** if need to display withdraws
ticker | String | **No** | Currency's ticker. Example: BTC
address | String | **No** | Can be used for filtering transactions by specific address.
uniqudId | String | **No** | Can be used for filtering transactions by specific unique id
limit | Int | **Yes** | LIMIT is a special clause used to limit records a particular query can return. Default: 50, Min: 1, Max: 100
offset | Int | **Yes** | If you want the request to return entries starting from a particular line, you can use OFFSET clause to tell it where it should start. Default: 0, Min: 0, Max: 10000

**Request BODY raw:**
```json5
{
    "transactionMethod": "1",
    "ticker": "BTC",
    "offset": 0,
    "limit": 100,
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if all validations succeeded and creating transaction is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`

Response error codes:
* 1 - transfers from trade to main are disabled
* 2 - transfers from main to trade are disabled
* 3 - not enough balance
   
Deposit status codes:
* `Pending` - 15
* `Unconfirmed by user` - 5
* `Canceled` - 9 an 4
* `Successful` - 3 and 7

Withdraw status codes:
* `Pending` - 1, 2, 6, 10, 11, 12, 13, 14
* `Unconfirmed by user` - 5
* `Canceled` - 4
* `Successful` - 3 and 7

```json5
{
    "limit": 100,
    "offset": 0,
    "records": [
        {
            "address": "3ApEASLcrQtZpg1TsssFgYF5V5YQJAKvuE",                                              // deposit address
            "amount": "0.000600000000000000",                                                             // amount of deposit
            "createdAt": 1593437922,                                                                      // timestamp of deposit
            "currency": "Bitcoin",                                                                        // deposit currency
            "description": "",                                                                            // deposit description
            "fee": "0.000000000000000000",                                                                // deposit fee
            "memo": "",                                                                                   // deposit memo
            "method": 1,                                                                                  // called method 1 - deposit, 2 - withdraw
            "network": null,                                                                              // if currency is multinetwork
            "status": 15,                                                                                 // transactions status
            "ticker": "BTC",                                                                              // deposit currency ticker
            "transactionHash": "a275a514013e4e0f927fd0d1bed215e7f6f2c4c6ce762836fe135ec22529d886",        // deposit transaction hash
            "uniqueId": null,                                                                             // unique Id of deposit
            "confirmations": {                                                                            // if transaction status == 15 you can see this object
                "actual": 1,                                                                              // current block confirmations
                "required": 2                                                                             // required block confirmation for successful deposit
            }                                                                                             
        },
        {...},
        {...},
        {...}
    ],
    "total": 4                                                                                             // total number of  transactions that returns in response
}

```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit field is required."
        ],
        "offset": [
            "The offset field is required."
        ],
        "transactionMethod": [
            "The transaction method field is required."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "transactionMethod": [
            "The selected transaction method is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit may not be greater than 100."
        ],
        "offset": [
            "The offset may not be greater than 10000."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit must be at least 1."
        ],
        "offset": [
            "The offset must be at least 0."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit must be an integer."
        ],
        "offset": [
            "The offset must be an integer."
        ]
    },
    "message": "Validation failed"
}
```

</details>

___

### Create new address for deposit

```
[POST] /api/v4/main-account/create-new-address
```
This endpoint creates a new address even when the last created address is not used. This endpoint is not available by default, you need to contact support@whitebit.com for getting such permissions.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ticker | String | **Yes** | Currency's ticker. Example: BTC
network | String | **No** | Currency's network if it multinetwork currency. Example: USDT_OMNI. For USDT default network is ETH.


**Request BODY raw:**
```json5
{
    "ticker": "XLM",
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if all validations succeeded and creating transaction is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`

```json5
{
    "account": {
        "address": "GDTSOI56XNVAKJNJBLJGRNZIVOCIZJRBIDKTWSCYEYNFAZEMBLN75RMN",        // deposit address
        "memo": "48565488244493"                                                      // memo if currency requires memo
    },
    "required": {
        "fixedFee": "0",                                                              // fixed deposit fee
        "flexFee": {                                                                  // flexible fee - is fee that use percent rate
            "maxFee": "0",                                                            // maximum fixed fee that you will pay
            "minFee": "0",                                                            // minimum fixed fee that you will pay
            "percent": "0"                                                            // percent of deposit that you will pay
        },
        "maxAmount": "0",                                                             // max amount of deposit that can be accepted by exchange - if you deposit more than that number, it won't be accepted by exchange 
        "minAmount": "1"                                                              // min amount of deposit that accepted by exchange - if you deposit less than that number, it won't be accepted by exchange 
    }
}

```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "The ticker field is required."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "network": [
            "Unsupported network"
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "network": [
            "The network must be a string."
        ],
        "ticker": [
            "The ticker must be a string."
        ]
    },
    "message": "Validation failed"
}
```

</details>

___

## Codes

### Create code

```
[POST] /api/v4/main-account/codes
```
This endpoint creates WhiteBIT code.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
ticker | String | **Yes** | Currency's ticker. Example: BTC
amount | Numeric string | **Yes** | Amount to transfer. Max precision = 8, value should be greater than zero and your main balance.
passphrase | String | **No** | Passphrase that will used for applying codes. Max: 25 symbols.
description | String | **No** | Additional text description for code. Visible only for creator Max: 75 symbols.

**Request BODY raw:**
```json5
{
    "ticker" : "ETH",
    "amount" : "0.002",
    "passphrase": "someStrongPassphrase",
    "description": "Some description", 
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if all validations succeeded and creating transaction is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`

Response error codes:
* 0 - Fiat are available on the front-end only
* 1 - currency is not withdrawable
* 2 - specified address is invalid
* 3 - amount is too small
* 4 - amount is too small for the payment system
* 5 - not enough balance
* 6 - amount is less than or equals fee

```json5
{
    "code": "WBe11f4fce-2a53-4edc-b195-66b693bd77e3ETH",         // generated WhiteBIT code
    "message": "Code was successfully created"
}

```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount field is required."
        ],
        "ticker": [
            "The ticker field is required."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount must be a number.",
            "Invalid number"
        ],
        "description": [
            "The description must be a string."
        ],
        "passphrase": [
            "The passphrase must be a string."
        ],
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "amount": [
            "The amount must be at least 0."
        ],
        "description": [
            "The description may not be greater than 75 characters."
        ],
        "passphrase": [
            "The passphrase may not be greater than 25 characters."
        ],
        "ticker": [
            "The selected ticker is invalid."
        ]
    },
    "message": "Validation failed"
}
```

</details>

___

### Apply code

```
[POST] /api/v4/main-account/codes/apply
```
This endpoint applies WhiteBIT code.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
code | String | **Yes** | Code that will be applied.
passphrase | String | **No** | Should be provided if the code was created with passphrase Max: 25 symbols.

**Request BODY raw:**
```json5
{
    "code" : "WBe11f4fce-2a53-4edc-b195-66b693bd77e3ETH",
    "passphrase": "someStrongPassphrase",
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if all validations succeeded and creating transaction is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`


```json5
{
    "amount": "0.002",
    "message": "Code was successfully applied",
    "ticker": "ETH"
}

```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "code": [
            "The code field is required."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "_token": null,
    "errors": {
        "code": [
            "Incorrect code or passphrase"
        ]
    },
    "notification": null,
    "response": null,
    "status": 422,
    "warning": "Incorrect code or passphrase"
}
```

</details>

___

### Get my codes

```
[POST] /api/v4/main-account/codes/my
```
This endpoint retrieves the list of WhiteBIT codes created by my account.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | Int | **No** | LIMIT is a special clause used to limit records a particular query can return. Default: 30, Min: 1, Max: 100
offset | Int | **No** | If you want the request to return entries starting from a particular line, you can use OFFSET clause to tell it where it should start. Default: 0, Min: 0, Max: 10000


**Request BODY raw:**
```json5
{
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if all validations succeeded and creating transaction is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`


```json5
{
    "data": [
        {
            "amount": "0.002",                                          // code amount
            "code": "WBe11f4fce-2a53-4edc-b195-66b693bd77e3ETH",        // code
            "date": 1598296332,                                         // code creation timestamp
            "status": "Activated",                                      // code status
            "ticker": "ETH"                                             // code ticker
        },
        {...}
    ],
    "limit": 30,
    "offset": 0,
    "total": 15
}


```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit may not be greater than 100."
        ],
        "offset": [
            "The offset may not be greater than 10000."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit must be at least 1."
        ],
        "offset": [
            "The offset must be at least 0."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit must be an integer."
        ],
        "offset": [
            "The offset must be an integer."
        ]
    },
    "message": "Validation failed"
}
```

</details>

___

### Get codes history

```
[POST] /api/v4/main-account/codes/history
```
This endpoint retrieves the whole codes history on your account.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | Int | **No** | LIMIT is a special clause used to limit records a particular query can return. Default: 30, Min: 1, Max: 100
offset | Int | **No** | If you want the request to return entries starting from a particular line, you can use OFFSET clause to tell it where it should start. Default: 0, Min: 0, Max: 10000


**Request BODY raw:**
```json5
{
    "request": "{{request}}",
    "nonce": "{{nonce}}"
}
```

**Response:**
Available statuses:
* `Status 201 if all validations succeeded and creating transaction is started`
* `Status 400 if request validation failed`
* `Status 422 if inner validation failed`


```json5
{
    "data": [
        {
            "amount": "+0.002",                                           // code amount change; - is created; + is applied
            "code": "WBe11f4fce-2a53-4edc-b195-66b693bd77e3ETH",          // code
            "date": 1598296734,                                           // code activation timestamp
            "status": "Activated",                                        // current code status
            "ticker": "ETH"                                               // code ticker
        },
        {
            "amount": "-0.002",                                           // code amount change; - is created; + is applied
            "code": "WBe11f4fce-2a53-4edc-b195-66b693bd77e3ETH",          // code
            "date": 1598296332,                                           // code creation timestamp
            "status": "Activated",                                        // current code status
            "ticker": "ETH"                                               // code ticker
        },
        {...}
    ],
    "limit": 100,
    "offset": 0,
    "total": 29
}



```
<details>
<summary><b>Errors:</b></summary>

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit may not be greater than 100."
        ],
        "offset": [
            "The offset may not be greater than 10000."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit must be at least 1."
        ],
        "offset": [
            "The offset must be at least 0."
        ]
    },
    "message": "Validation failed"
}
```

```json5
{
    "code": 0,
    "errors": {
        "limit": [
            "The limit must be an integer."
        ],
        "offset": [
            "The offset must be an integer."
        ]
    },
    "message": "Validation failed"
}
```

</details>

___
