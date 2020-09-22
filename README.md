# 合作平台 REST API 使用详解

通过使用 Partner Merchant 合作平台  REST API 可以让你用任何支持发送 HTTP 请求的设备来与 Soda Express 进行交互。

## API 版本

| 版本   | 内容                                       |
| ---- | ---------------------------------------- |
| ```线上版本```1.7 Sep 15 2020 | 请使用本版本 API 🎉|
| ```已弃用``` 0.6.0 March 7 2020 | Beta Release|

### Base URL

为保证用户、开发者的数据安全，请务必总是使用 HTTPS 来加密传输讯息。  
文档中 API Base URL 给予地理位置和国家进行区分，请注意使用。

| 区域   | Base URL                                |
| ---- | ---------------------------------------- |
| 🇳🇿新西兰  | nz-merc.lovelive.team |

<!---| 🇬🇧英国  | uk-merc.lovelive.team || 🇦🇺澳大利亚  | au-merc.lovelive.team || 🇺🇸美国  | us-merc.lovelive.team |--->

### 标准响应格式

对于所有的请求，响应格式都是一个 JSON 对象。

一个请求是否成功是由返回字段里的 `errno` 标明的。而服务端通常都会返回一个 2XX 的 HTTP 状态码表示成功，而一个 5XX 的 HTTP 状态码表示请求异常。当一个请求失败时响应的主体仍然是一个 JSON 对象，但是总是会包含 `errno` 和 `errmsg` 这两个字段，你可以用它们来进行调试。举个例子，如果尝试用非法的属性名来保存一个对象会得到如下信息：

```json
{
  "errno": 4001,
  "errmsg": "app_key不存在"
}
```

### 请求签名

将全部的请求内容使用字母顺序 a-z 进行排序，排序后使用 md5 HEX 进行加密后放置于头部 sign 字段。  

如密钥错误会返回

```json
{
  "errno": 4003,
  "errmsg": "Wrong Sign"
}
```

### 回调接口

如需要完整接入 Soda 平台，体验更完整的服务，那么接入回调服务将会是一个非常好的选择。  

请通过企业微信 @liujiaqi 提供一个后端接口用于接受 Soda 的回调状态信息，这样将更好的获得订单和相关服务的状态。  
Soda 将以 Json 格式向提前预留的接口发送请求。请求示例如下。  

```json
{
  "order_no": "5404633a6434412eb5c64d556852b328",
  "status": "2",
  "msg": "订单已被跑男接到"
}
```

| Status | 定义  | 备注 |
| ---- | ----------------------------------| ------ |
|1|已经接单||
|2|已经拿到餐品||
|3|已经送达||
|-2|无法联系到收件人|重定向餐品至中转点|
|-6|争议订单进入流程|客服联系|
|1005|平台过载请停止接单|不会带订单号|

### 起点地址终点地址 JSON 
| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|formatted_addresse|String|街道名|
|city|String|城市区域|
|latitude|Number|纬度|
|longitude|Number|经度|
|phone|String|电话|
|name|String|名字|

```json
{
   "formatted_addresse":"4 Yjsp St",
   "city":"Mount Eden\nAuckland 1024",
   "latitude":-36.89357,
   "longitude":174.759766,
   "phone":"0220826111",
   "name":"兔先生"
}
```


## 价格相关接口

### 查询配送价格

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/opentp/v1/cost/calculation</td>
      <td>POST</td>
      <td>查询配送价格</td>
    </tr>
  </tbody>
</table>

* 注意本价格不包含 GST

查询配送价格服务接口的使用可以获得配送价格

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|startAddress|String|起点地址是一个 object 请求时将它转成 string 类型|
|endAddress|String|终点地址是一个 object 请求时将它转成 string 类型|
|service_type|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|weight|Number|单位为 `g` 如 `1` 公斤则填写 `1000`|

```json
{
  "app_key": "AppKey",
  "startAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "endAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "service_type": "OPENID,
  "weight": 10000
}
```

#### 返回示例
| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|data.totalPrice|String|总价格|
|data.start.startDistance|Number|起步价距离|
|data.start.startPrice|String|起步价价格|
|data.weightPrice|Number|重量价格|
|data.excced.exccedDistance|Number|超出部分距离|
|data.excced.exccedPrice|String|超出部分价格|

```json
{
  "errno": 0,
  "errmsg": "",
  "data": {
    "totalPrice": "22.25",
    "start": {
      "startDistance": 2000,
      "startPrice": "6.25"
    },
    "weightPrice": 0,
    "excced": {
      "exccedDistance": 23918,
      "exccedPrice": "16.00"
    }
  }
}
```



## 订单相关接口

### 合作商家下单

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/opentp/v1/order/place</td>
      <td>POST</td>
      <td>合作商家下单</td>
    </tr>
  </tbody>
</table>

合作商家下单服务接口的使用可以通过接口直接在 Soda Express 下单

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|startAddress|String|起点地址是一个 object 请求时将它转成 string 类型|
|endAddress|String|终点地址是一个 object 请求时将它转成 string 类型|
|service_type|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|weight|Number|单位为 `g` 如 `1` 公斤则填写 `1000`|
|textarea|String|平台订单号|
|goodslist|String|商品描述是一个 object 请求时将它转成 string 类型|
｜est_time｜String｜预计时间｜

```json
{
  "app_key": "AppKey",
  "startAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "endAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "service_type": "OPENID",
  "textarea": "平台订单号",
  "weight": 10000,
  "goodslist":{
    "totalprice":1145.14,
    "noti":"多喝热水么么哒",
    "data":[
        "{
          "id":1,
          "name":"吃米酸奶",
          "sku":"去冰 少糖",
          "price":1.14,
          "qty":19
        }",
        "{
          "id":2,
          "name":"矢泽妮可",
          "sku":"24 岁女高生",
          "price":19.19,
          "qty":810
        }",
        "{
          "id":3,
          "name":"渡边曜",
          "sku":"四年级 很有精神",
          "price":8.10,
          "qty":19
        }"
    ]
  }
}
```

#### 返回示例
| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|data.totalFee|Number|总价格 不包含 GST 进位保留两位小数点你|
|data.out_trade_no|String|平台唯一订单号|

```json
{
  "errno": 0,
  "errmsg": "",
  "data": {
    "totalFee": 25.5875,
    "out_trade_no": "5404633a6434412eb5c64d556852b328",
  }
}
```


### 合作商家取消订单

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/opentp/v1/order/cancel</td>
      <td>POST</td>
      <td>合作商家取消订单</td>
    </tr>
  </tbody>
</table>

合作商家下单服务接口的使用可以通过接口直接在 Soda Express 关闭订单

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|order_no|String|Soda 订单号|

```json
{
  "app_key": "AppKey",
  "order_no": "order_no",
}
```

#### 返回示例

```json
{
  "errno": 0,
  "errmsg": "订单已取消",
}
```
```json
{
  "errno": 0,
  "errmsg": "订单将会送回取货地址",
}
```
```json
{
  "errno": 1404,
  "errmsg": "订单不存在",
}
```
```json
{
  "errno": 1403,
  "errmsg": "不是您的订单",
}
```
### 获得本周账期订单细节

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/opentp/v1/order/weeklist</td>
      <td>POST</td>
      <td>获得本周账期订单</td>
    </tr>
  </tbody>
</table>

通过请求本接口可以获得本账期的账单计费订单信息，仅供参考

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|

```json
{
  "app_key": "AppKey",
}
```

#### 返回示例
```json
{
  "errno": 0,
  "errmsg": "success",
  "data": [
    {
      "id": 3366,
      "order_no": "705ecf800d864bf180c9052704c5f60a",
      "pay_amount": 3.49,
      "pay_type": 1,
      "service_type": "OPENID",
      "interal_value": 0,
      "interal_amount": 0,
      "night_price": 0,
      "start_distance_amount": 3.49,
      "exceed_distance_amount": 0,
      "exceed_distance": 0,
      "weight_id": 0,
      "weight_price": 0,
      "status": 3,
      "refund_amount": 0,
      "refund_status": 0,
      "start_address": "",
      "start_hash": null,
      "end_address": "",
      "goods_des": "测试物品",
      "send_time": "2020-06-22 16:49:39",
      "create_time": "2020-06-22 16:49:40",
      "expt_desc": null,
      "wx_id": 0,
      "form_ids": "",
      "openid": "0",
      "tip": 0,
      "distance": 1540,
      "ws_id": 0,
      "refund_no": null,
      "argue_desc": "糖吃洒了",
      "order_type": 1,
      "opentp_key": "APPKEY"
    },
  ]
}
```

### 获得订单细节

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/opentp/v1/order/track</td>
      <td>POST</td>
      <td>订单状态跟踪</td>
    </tr>
  </tbody>
</table>

通过请求本接口可以获得本账期的账单计费订单信息，仅供参考

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|order_no|String|订单编号|

```json
{
  "app_key": "AppKey",
  "order_no": "705ecf800d864bf180c9052704c5f60a"
}
```

#### 返回示例

```json
{
  "errno": 0,
  "errmsg": "success",
  "data": [
    {
      "id": 3366,
      "order_no": "705ecf800d864bf180c9052704c5f60a",
      "pay_amount": 3.49,
      "pay_type": 1,
      "service_type": "OPENID",
      "interal_value": 0,
      "interal_amount": 0,
      "night_price": 0,
      "start_distance_amount": 3.49,
      "exceed_distance_amount": 0,
      "exceed_distance": 0,
      "weight_id": 0,
      "weight_price": 0,
      "status": 3,
      "refund_amount": 0,
      "refund_status": 0,
      "start_address": "",
      "start_hash": null,
      "end_address": "",
      "goods_des": "测试物品",
      "send_time": "2020-06-22 16:49:39",
      "create_time": "2020-06-22 16:49:40",
      "expt_desc": null,
      "wx_id": 0,
      "form_ids": "",
      "openid": "0",
      "tip": 0,
      "distance": 1540,
      "ws_id": 0,
      "refund_no": null,
      "argue_desc": "",
      "order_type": 1,
      "opentp_key": "APPKEY",
      "lat": -36.801526,
      "lnt": 174.740425,
    },
  ]
}
```


### 发起订单争议

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/opentp/v1/order/argue</td>
      <td>POST</td>
      <td>发起订单争议</td>
    </tr>
  </tbody>
</table>

通过请求本接口可以获得本账期的账单计费订单信息，仅供参考

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|argue_desc|String|争议内容，请描述|
|order_no|String|订单编号|

```json
{
  "argue_desc": "糖吃洒了",
  "order_no": "705ecf800d864bf180c9052704c5f60a1",
  "app_key": "APPKEY"
}
```

#### 返回示例

```json
{
  "errno": 0,
  "errmsg": "",
  "data": "收到，订单进入争议流程"
}
```

## 评价相关接口

### 评价订单

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/opentp/v1/evaluate/add</td>
      <td>POST</td>
      <td>评价订单接口</td>
    </tr>
  </tbody>
</table>

通过请求本接口可以进行订单的相关评价

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|score|String|分数 区间 1-5|
|order_id|String|订单号|
|msg|String|评价内容|
|ws_id|String|配送司机|

```json
{
  "ws_id": "12",
  "order_id": "sadfsgdhdgfsdfas",
  "score": "5",
  "msg": "verygood",
  "app_key": "AppKey"
}
```

#### 返回示例

```json
{
  "errno": 0,
  "errmsg": "评价成功",
  "data": {}
}
```



