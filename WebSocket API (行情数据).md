# WebSocket API (行情数据) 

## 开始使用    

WebSocket协议是基于TCP的一种新的网络协议。它实现了客户端与服务器之间在单个 tcp 连接上的全双工通信，由服务器主动发送信息给客户端，减少了频繁的身份验证等不必要的开销。其最大优点：

> - 两方请求的 header 数据很小，大概只有2 Bytes。
> - 服务器不再是被动的接到客户端的请求后才返回数据，而是有了新数据后主动推送给客户端。
> - 不需要多次创建TCP请求和销毁，节约宽带和服务器的资源。



## 请求与订阅说明

* 访问地址: `wss://www.lbkex.net/ws/V2/`

* 心跳（ping/pong）
    为了防止僵尸链接，减少服务器负荷，服务器端会定时向客户连接发送心跳PING信息。客户端收到PING消息后，应立即回应。如果超过一分钟没有回应任何PING消息，连接会被自动关闭。同时，客户也可以向服务器短发送PING消息，用于检查连接是否正常。服务器端收到PING消息后，也会回应对应端PONG消息。

**示例:**

```javascript
# ping
{
    "action":"ping",
    "ping":"0ca8f854-7ba7-4341-9d86-d3327e52804e"
}
# pong
{
    "action":"pong",
    "pong":"0ca8f854-7ba7-4341-9d86-d3327e52804e"
}
```

其中，pong字段必需和收到对ping消息字段完全一致。



* 订阅/退订数据（subscribe/unsubscribe）
    每个订阅数据应该至少包括一个subscribe字段，用于指定订阅的数据类型。现在可以订阅的数据包括：kbar，tick，depth，trade四种。每个订阅数据都需要一个pair字段，用来指定订阅的交易对，交易对以下划线（_)、连接。订阅成功后一旦所订阅的数据有更新，Websocket客户端将收到服务器推送的更新消息
    

1.订阅K线数据

**参数:**

|参数名|	参数类型|	必填|	描述|
| :-----    | :-----   | :-----    | :-----   |
|action|String|是|请求的动作类型:`subscribe`,`unsubscribe`|
|subscribe|String|是|`kbar`|
|kbar|String|是|可订阅的K线类型<br>`1min`:1分钟<br>`5min`:5分钟<br>`15min`:15分钟<br>`30min`:30分钟<br>`1hr`:1小时<br>`4hr`:4小时<br>`day`:1日<br>`week`:1周<br>`month`:1月<br>`year`:1年|
|pair|String|是|交易对:`eth_btc`|

**请求示例:**

```javascript
# Subscribe Request
{
    "action":"subscribe",
    "subscribe":"kbar",
    "kbar":"5min",
    "pair":"eth_btc"
}
# Subscribe Response
{
    "kbar":{
        "a":64.32991311,
        "c":0.02590293,
        "t":"2019-06-28T17:45:00.000",
        "v":2481.1912,
        "h":0.02601247,
        "slot":"5min",
        "l":0.02587925,
        "n":272,
        "o":0.02595196
    },
    "type":"kbar",
    "pair":"eth_btc",
    "SERVER":"V2",
    "TS":"2019-06-28T17:49:22.722"
}
```

**返回值说明:**

|参数名|	参数类型|	描述|
| :-----    | :-----  | :-----   |
|t|BigDecimal|K线更新时间戳|
|o|BigDecimal|开|
|h|BigDecimal|高|
|l|BigDecimal|低|
|c|BigDecimal|收|
|v|BigDecimal|成交量|
|a|BigDecimal|成交额, 即 sum(每一笔成交价 * 该笔的成交量)|
|n|BigDecimal|成交笔数|
|slot|String|K线类型|


2.订阅深度

**参数:**

|参数名|	参数类型|	必填|	描述|
| :-----    | :-----   | :-----    | :-----   |
|action|String|是|请求的动作类型:`subscribe`,`unsubscribe`|
|subscribe|String|是|`depth`|
|depth|String|是|支持选择:10/50/100|
|pair|String|是|交易对:`eth_btc`|


**请求示例:**

```javascript
# Subscribe Request
{
    "action":"subscribe",
    "subscribe":"depth",
    "depth":"100",
    "pair":"eth_btc"
}
# Subscribe Response
{
    "depth":{
        "asks":[
            [
                0.00003377,
                0.0176
            ],
            ...
        ],
        "bids":Array[100]
    },
    "type":"depth",
    "pair":"dax_eth",
    "count":100,
    "SERVER":"V2",
    "TS":"2019-06-28T19:48:23.938"
}
```

**返回值说明:**

|参数名|	参数类型|	描述|
| :-----    | :-----  | :-----   |
|asks|List<BigDecimal>|卖方深度,list.get(0):委托价, list.get(1):委托数量|
|bids|List<BigDecimal>|买方深度|


3.成交记录

**参数:**

|参数名|	参数类型|	必填|	描述|
| :-----    | :-----   | :-----    | :-----   |
|action|String|是|请求的动作类型:`subscribe`,`unsubscribe`|
|subscribe|String|是|`trade`|
|pair|String|是|交易对:`eth_btc`|


**请求示例:**

```javascript
# Subscribe Request
{
    "action":"subscribe",
    "subscribe":"trade",
    "pair":"eth_btc"
}
# Subscribe Response
{
    "trade":{
        "volume":6.3607,
        "amount":77148.9303,
        "price":12129,
        "direction":"sell",
        "TS":"2019-06-28T19:55:49.460"
    },
    "type":"trade",
    "pair":"btc_usdt",
    "SERVER":"V2",
    "TS":"2019-06-28T19:55:49.466"
}
```

**返回值说明:**

|参数名|	参数类型|	描述|
| :-----    | :-----  | :-----   |
|amount|String|最近成交数量|
|price|Integer|成交价|
|volumePrice|String|最近成交数额|
|direction|String|`sell`,`buy`|
|TS|String|成交时间|
      
 
4.快照记录

**参数:**

|参数名|	参数类型|	必填|	描述|
| :-----    | :-----   | :-----    | :-----   |
|action|String|是|请求的动作类型:`subscribe`,`unsubscribe`|
|subscribe|String|是|`tick`|
|pair|String|是|交易对:`eth_btc`|


**请求示例:**

```javascript
# Subscribe Request
{
    "action":"subscribe",
    "subscribe":"tick",
    "pair":"eth_btc"
}
# Subscribe Response
{
    "tick":{
        "to_cny":76643.5,
        "high":0.02719761,
        "vol":497529.7686,
        "low":0.02603071,
        "change":2.54,
        "usd":299.12,
        "to_usd":11083.66,
        "dir":"sell",
        "turnover":13224.0186,
        "latest":0.02698749,
        "cny":2068.41
    },
    "type":"tick",
    "pair":"eth_btc",
    "SERVER":"V2",
    "TS":"2019-07-01T11:33:55.188"
}
```

**返回值说明:**

|参数名|	参数类型|	描述|
| :-----    | :-----  | :-----   |
|high|BigDecimal|24小时内最高价|
|low|BigDecimal|24小时内最低价|
|latest|BigDecimal|最新成交价|
|vol|BigDecimal|成交量|
|turnover|BigDecimal|成交额, 即 sum(每一笔成交价 * 该笔的成交量)|
|to_cny|BigDecimal|成交时间|
|to_usd|BigDecimal|成交时间|
|cny|BigDecimal|成交时间|
|usd|BigDecimal|成交时间|
|dir|String|`sell`,`buy`|
|change|BigDecimal|24小时内涨跌幅|

      
    
**取消订阅示例:**

```javascript
{
    "action":"unsubscribe",
    "subscribe":"kbar",
    "kbar":"5min",
    "pair":"eth_btc"
}
```

* 请求数据（request）
    Websocket服务器同时支持一次性请求数据


    每个订阅数据应该至少包括一个request字段，用于指定订阅的数据类型。现在可以订阅
    的数据包括：kbar，tick，depth，trade四种。除kbar外，其它三种数据暂时不支持
    历史数据的查询。
    * kbar:  K线数据（蜡烛图）。除了上面的两个字段，还需要一个额外字段：kbar，
      用于指定K线的时间间隔，当前该参数接受的选项包括1min, 5min, 15min, 30min, 
      1hr, 4hr, day, week, month, year。如果
      需要查询历史数据，需要增加字段start, end, size。start是开始时间, end是截止时间,
      接受两种格式，一种是ISO格式，精确到秒，如"2018-08-03T17:32:00"（北京时间）,
      另一种是UNIX时间戳，暨1970年1月1日0时（UTC/GMT）开始的秒数，如1514736000.0代
      表北京时间2018年1月1日0点。size是需要获取的kbar的条数。最大不超过1440条。
      若没有start, end, size参数，默认获取最新的一条kbar数据。
    * depth：当前交易的深度数据。除了上面的两个字段，还需要一个额外字段：depth，
      用于指定深度，现在有 10/50/100 三个深度可以选择    
    * tick： 当前交易的实时快照
    * trade：获取最近的交易记录。除了上面的两个字段，还需要一个额外字段：size，用
      于指定获取的交易数目。最多不超过100条。
      
      

