# REST APIs

## 仕様
* APIユーザーからアクセスされるAPI群です。

### リクエストの仕様について
* 適切なHTTPメソッドを設定して下さい。
* リクエストパラメーターについて
  * GET, DELETEはクエリパラメーターにて指定して下さい。
  * POST, PUTはリクエストボディにて指定して下さい。

#### 認証の方法
* API-KEY, NONCE, SIGNATUREをリクエストヘッダーに付けて通信する。
  * API-KEY: 公開鍵的なもの。取引所ページでユーザー毎に生成する。
    * SECRET: API-KEYと一緒に生成される、非公開鍵的なもの。
  * NONCE: ミリ秒のtimestamp
  * SIGNATURE: ユニークな値

#### SIGNATUREの作り方
* 下記文字列を、SECRETを使ってHMAC SHA-256でハッシュ化する
* GET, DELETE: NONCE + URI + queryString
  * 1586345939000/v1/spot/order/65?id=1
* POST, PUT: NONCE + Json encoded request body
  * 1586345939000{"label":"apiKeyLabel"}

### レスポンスの仕様について
* 通信の成否は全てレスポンスのHTTPステータスコードで判別します。

---
## Public APIs
### 銘柄一覧取得
```
GET /api/v1/symbol
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
#### response
Name | Type    | Description
--- |---------| --- 
id| long    | シンボルID
tradeType| string  |取引タイプ
currencyPair| string  |通貨ペア
baseCurrency| string  |ベース通貨
quoteCurrency| string  |クォート通貨
basePrecision| int     |数量の小数点以下桁数(ベース通貨桁数)　例) 1の場合0.1など
quotePrecision| int     | 価格の小数点以下桁数(クォート通貨桁数)　例) 0の場合、50000など
makerTradeFeePercent| decimal |メイカー約定手数料率
takerTradeFeePercent| decimal |テイカー約定手数料率
tradable| boolean |取引可否
enabled| boolean |有効

```json
[
  {
    "id": 123,
    "tradeType": "SPOT",
    "currencyPair": "BTC_JPY",
    "baseCurrency": "BTC",
    "quoteCurrency": "JPY",
    "basePrecision": 8,
    "quotePrecision": 0,
    "makerTradeFeePercent": -0.1,
    "takerTradeFeePercent": 0.1,
    "tradable": true,
    "enabled": true
  },
  ...
]
```

### ローソク足取得
```
GET /api/v1/candlestick
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
symbolId | long | YES | シンボルID
candlestickType | string | YES | ローソク足の種類（ISO8601に準拠）<br> ex) 1分足: PT1M, 5分足: PT5M, 15分足: PT15M, 30分足: PT30M, 1時間足: PT1H, 4時間足: PT4H, 8時間足: PT8H, 日足: P1D, 週足: P1W, 月足: P1M
dateFrom | long | NO | 取得開始日時（指定日時以上検索　例 : 1611029776000）
dateTo | long | NO | 取得終了日時（指定日時未満検索　例 : 1611029776000）
#### response
Name | Type    | Description
--- | --------- | --- 
symbolId | long | シンボルID  
candlesticks | CandlestickElement[] | ローソク足データ配列
timestamp | long | 作成日時  

### CandlestickElement
Name | Type | Description
--- | --------- | --- 
open | decimal | 始値
high | decimal | 高値
low | decimal | 安値
close | decimal | 終値
volume | decimal | 出来高
time | long | 対象日時

```json
{
  "symbolId": 1,
  "candlesticks": [
    {
      "open": 3633849,
      "high": 3878414,
      "low": 3615752,
      "close": 3802653,
      "volume": 94.71371641,
      "time": 1611029776000
    },
    ...
  ],
  "timestamp": 1611029776000
}
```

### 板取得
```
GET /api/v1/orderbook
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
symbolId | long | YES | シンボルID
#### response

Name | Type | Description                             | Not Null
--- | --- |-----------------------------------------| ---
symbolId | long    | シンボルID
asks | Orderbook[] | 売り板
bids | Orderbook[] | 買い板
bestAsk | decimal | ベストアスク（買い板が空の場合は0）                      | No
bestBid | decimal | ベストビッド（買い板が空の場合は0）                      | No
midPrice | decimal | ミッドプライス（ベストアスク または ベストビッドがnullの場合はnull） | No
spread | decimal スプレッド（ベストアスク または ベストビッドがnullの場合はnull） |                                         | No
timestamp | long | 作成日時

#### Orderbook
Name | Type | Description
--- | --- | ---
price | decimal | 価格
amount | decimal | 数量

```json
{
  "symbolId": 1,
  "asks": [
    {
      "price": 3633849,
      "amount": 0.1
    },
    ...
  ],
  "bids": [
    {
      "price": 3633749,
      "amount": 0.1
    },
    ...
  ],
  "timestamp": 1611029776000,
  "midPrice": 3050000.50000000000000000000,
  "spread": 45000.00000000000000000000,
  "bestBid": 3475000.00000000000000000000,
  "bestAsk": 3520000.00000000000000000000
}
```
### ティッカー取得
```
GET /api/v1/ticker
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
symbolId | long | YES | シンボルID
#### response

Name | Type    | Description
--- |---------| ---
symbolId | long    | シンボルID
bestAsk | decimal | 直近の売り板の最良価格
bestBid | decimal | 直近の買い板の最良価格
open | decimal | 24時間前の約定価格
high | decimal | 直近24時間の高値（約定価格）
low | decimal | 直近24時間の安値（約定価格）
last | decimal | 直近の約定価格
volume | decimal | 直近24時間の出来高
timestamp | long    | 作成日時

```json
{
  "symbolId": 1,
  "bestAsk": 3802663,
  "bestBid": 3802643,
  "open": 3633849,
  "high": 3878414,
  "low": 3615752,
  "last": 3802653,
  "volume": 94.71371641,
  "timestamp": 1611029776000
}
```

### 歩み値取得
```
GET /api/v1/trades
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
symbolId | long | YES | シンボルID
#### response
Name | Type | Description
--- | --- | --- 
symbolId | long | シンボルID
tradesData | Trade[] | 歩み値データ
timestamp | long | 作成日時

#### Trade
Name | Type    | Description
--- |---------| --- 
id | long    | 約定履歴ID
orderSide | string  | 売買
price | decimal | 価格
amount | decimal | 数量
tradedAt | long    | 約定日時

```json
{
  "symbolId": 1,
  "tradesData": [
    {
      "id": 789,
      "orderSide": "BUY",
      "price": 3802653,
      "amount": 0.04,
      "tradedAt": 1611029775000
    },
    ...
  ]
  "timestamp": 1611029776000
}
```

---
## Private APIs

### 残高一覧取得
```
GET /api/v1/asset
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---

#### response
Name | Type    | Description
--- |---------| ---
userId | long    | ユーザーID
currency | string  | 通貨
onhandAmount | decimal | 保有数量
lockedAmount | decimal | ロック数量
user | object  | ユーザー情報 null
jpyOnhandAmount | decimal | 保有数量(円換算)
jpyLockedAmount | decimal | ロック数量(円換算)

```json
[
  {
    "userId": 123,
    "currency": "BTC",
    "onhandAmount": 1.00000001,
    "lockedAmount": 0.30000001,
    "unlockedAmount": 0.7,
    "user": null,
    "jpyOnhandAmount": 1000000,
    "jpyLockedAmount": 300000
  },
  ...
]
```

### 注文一覧取得
```
GET /api/v1/spot/order
```
#### parameters

Name | Type     | Mandatory | Description
--- |----------| --- | ---
symbolId | long     | YES | シンボルID
id | long     | NO | 注文ID
idFrom | long     | NO | 検索開始注文ID
idTo | long     | NO | 検索終了注文ID
dateFrom | long     | NO | 取得開始日時（指定日時以上検索　例 : 1611029776000）
dateTo | long     | NO | 取得終了日時（指定日時未満検索　例 : 1611029776000）
orderStatus | string[] | NO | WAITING, UNFILLED, PARTIALLY_FILLED, CANCELED_UNFILLED, CANCELED_PARTIALLY_FILLED, FULLY_FILLED
orderType | string   | NO | MARKET, LIMIT
orderSide | string   | NO | SELL, BUY
number | int      | NO | ページ番号。default: 0
size | int      | NO | 1ページに表示する件数。default: 30

#### response

Name | Type    | Description | Not Null
--- |---------| --- | ---
id | long    | 注文ID
symbolId | long    | シンボルID
userId | long    | ユーザーID
orderSide | string  | 売買
orderType | string  | 執行条件
price    | decimal | 価格。成行の場合はpriceがnullになる | No
averagePrice | decimal | 平均約定価格
amount | decimal | 数量
remainingAmount | decimal | 未約定数量
orderStatus | string  | 注文ステータス
orderOperator | string  | 注文したオペレーター
orderChannel | string  | 注文経路
createdAt | long    | 注文日時
updatedAt | long    | 更新日時 | No

```json
[
  {
    "id": 123,
    "symbolId": 1,
    "userId": 123,
    "orderSide": "BUY",
    "orderType": "LIMIT",
    "price": 3650000,
    "averagePrice": 0,
    "amount": 0.1,
    "remainingAmount": 0.1,
    "orderStatus": "UNFILLED",
    "orderOperator": "USER",
    "orderChannel": "PC_WEB",
    "createdAt": 1611029776000,
    "updatedAt": 1611029776000
  },
  ...
]
```

### 注文
```
POST /api/v1/spot/order
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
symbolId | long | YES | シンボルID
orderType | string | YES | MARKET, LIMIT
orderSide | string | YES | SELL, BUY
price | decimal | NO | 注文価格。orderType=LIMITの時に指定する。
amount | decimal | YES | 注文数量

#### response

Name | Type    | Description | Not Null
--- |---------| --- | ---
id | long    | 注文ID
symbolId | long    | シンボルID
userId | long    | ユーザーID
orderSide | string  | 売買
orderType | string  | 執行条件
price    | decimal | 価格。成行の場合はpriceがnullになる | No
averagePrice | decimal | 平均約定価格
amount | decimal | 数量
remainingAmount | decimal | 未約定数量
orderStatus | string  | 注文ステータス
orderOperator | string  | 注文したオペレーター
orderChannel | string  | 注文経路
createdAt | long    | 注文日時
updatedAt | long    | 更新日時 | No

```json
{
  "id": 123,
  "symbolId": 1,
  "userId": 123,
  "orderSide": "BUY",
  "orderType": "LIMIT",
  "price": 3650000,
  "averagePrice": 0,
  "amount": 0.1,
  "remainingAmount": 0.1,
  "orderStatus": "UNFILLED",
  "orderOperator": "USER",
  "orderChannel": "PC_WEB",
  "createdAt": 1611029776000,
  "updatedAt": 1611029776000
}
```

### 注文取消
```
DELETE /api/v1/spot/order
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
symbolId | long | YES | シンボルID
id | long | YES | 注文ID
#### response

Name | Type    | Description | Not Null
--- |---------| --- | ---
symbolId | long    | シンボルID
id | long    | 注文ID
userId | long    | ユーザーID
orderSide | string  | 売買
orderType | string  | 執行条件
price    | decimal | 価格。成行の場合はpriceがnullになる | no
averagePrice | decimal | 平均約定価格
amount | decimal | 数量
remainingAmount | decimal | 未約定数量
orderStatus | string  | 注文ステータス
orderOperator | string  | 注文したオペレーター
orderChannel | string  | 注文経路
createdAt | long    | 注文日時
updatedAt | long    | 更新日時 | No

```json
{
  "id": 123,
  "symbolId": 1,
  "userId": 123,
  "orderSide": "BUY",
  "orderType": "LIMIT",
  "price": 3650000,
  "averagePrice": 0,
  "amount": 0.1,
  "remainingAmount": 0.1,
  "orderStatus": "CANCELED_UNFILLED",
  "orderOperator": "ADMIN",
  "orderChannel": "PC_WEB",
  "createdAt": 1611029776000,
  "updatedAt": 1611029776000
}
```

### 約定一覧取得
```
GET /api/v1/spot/trade
```
#### parameters
Name | Type | Mandatory | Description
--- | --- | --- | ---
symbolId | long | YES | シンボルID
id | long | NO | 注文ID
idFrom | long | NO | 検索開始注文ID
idTo | long | NO | 検索終了注文ID
dateFrom | long | NO | 取得開始日時（指定日時以上検索　例 : 1611029776000）
dateTo | long | NO | 取得終了日時（指定日時未満検索　例 : 1611029776000）
orderType | string | NO | MARKET, LIMIT
orderSide | string | NO | SELL, BUY
number | int | NO | ページ番号。default: 0
size | int | NO | 1ページに表示する件数。default: 30

#### response
Name | Type    | Description | Not Null
--- |---------| --- | --- 
id | long    | 約定ID
symbolId | long    | シンボルID
userId | long    | ユーザーID
orderSide | string  | 売買
orderType | string  | 執行条件
price    | decimal | 価格。成行の場合はpriceがnullになる | No
amount | decimal | 数量
tradeAction | string  | Taker, Maker
orderId | long    | 注文ID
fee    | decimal | 約定手数料
orderChannel | string  | 注文経路
jpyConversion | decimal | 円換算価格（約定時点）
targetOrderId | long    | 対当注文ID null値固定
targetUserId | long    | 対当注文顧客ID null値固定
assetAmount    | decimal | 総額。価格 * 数量 * 円換算価格(買いの場合は買い板ベストBid, 売りの場合は売り板ベストAsk)　円換算価格取得不可時はnull
createdAt | long    | 注文日時
updatedAt | long    | 更新日時 | No

```json
[
  {
    "id": 123,
    "symbolId": 1,
    "userId": 123,
    "orderSide": "BUY",
    "orderType": "LIMIT",
    "price": 3650000,
    "amount": 0.1,
    "tradeAction": "MAKER",
    "orderId": 456,
    "fee": 365,
    "orderChannel": "PC_WEB",
    "jpyConversion": 1,
    "targetOrderId": null,
    "targetUserId": null,
    "assetAmount": 17250.00000000000000000000,
    "createdAt": 1611029776000,
    "updatedAt": 1611029776000
  },
  ...
]
```


