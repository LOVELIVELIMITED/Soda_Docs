# Partner Merchant 合作商家 REST API 使用详解

通过使用 Partner Merchant 合作商家 REST API 可以让你用任何支持发送 HTTP 请求的设备来与 Soda Express 进行交互。

## API 版本

| 版本   | 内容                                       |
| ---- | ---------------------------------------- |
| ```线上版本```1.0.2 Apr 14 2020 | First Release 合作商家接口上线 🎉|
| ```已弃用``` 0.6.0 March 7 2020 | Beta Release|

### Base URL

为保证用户、开发者的数据安全，请务必总是使用 HTTPS 来加密传输讯息。  
文档中 API Base URL 给予地理位置和国家进行区分，请注意使用。

| 区域   | Base URL                                |
| ---- | ---------------------------------------- |
| 🇳🇿新西兰  | nz-merc.lovelive.team |
| 🇬🇧英国  | uk-merc.lovelive.team |
| 🇦🇺澳大利亚  | au-merc.lovelive.team |
| 🇺🇸美国  | us-merc.lovelive.team |

### 订单操作
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
    <tr>
      <td>/opentp/v1/order/place</td>
      <td>POST</td>
      <td>合作商家下单</td>
    </tr>
  </tbody>
</table>

### 响应格式

对于所有的请求，响应格式都是一个 JSON 对象。

一个请求是否成功是由返回字段里的 `errno` 标明的。而服务端通常都会返回一个 2XX 的 HTTP 状态码表示成功，而一个 5XX 的 HTTP 状态码表示请求异常。当一个请求失败时响应的主体仍然是一个 JSON 对象，但是总是会包含 `errno` 和 `errmsg` 这两个字段，你可以用它们来进行调试。举个例子，如果尝试用非法的属性名来保存一个对象会得到如下信息：

```json
{
  "errno": 1000,
  "errmsg": "缺少必要字段：app_key"
}
```


## 订单操作

### 查询配送价格

* 注意本价格不包含 GST

查询配送价格服务接口的使用可以获得配送价格

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|startAddress|String|起点地址是一个 object 请求时将它转成 string 类型|
|endAddress|String|终点地址是一个 object 请求时将它转成 string 类型|
|service_type|String|服务类型 `帮我送` `帮我取`|
|weight|Number|单位为 `g` 如 `1` 公斤则填写 `1000`|

```json
{
  "app_key": "AppKey",
  "startAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "endAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "service_type": "帮我送",
  "weight": 10000
}
```

#### 起点地址终点地址 JSON 
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


### 合作商家下单

合作商家下单服务接口的使用可以通过接口直接在 Soda Express 下单

#### 请求示例

| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|app_key|String|AppKey 企业微信联系 `@liujiaqi` 获取|
|startAddress|String|起点地址是一个 object 请求时将它转成 string 类型|
|endAddress|String|终点地址是一个 object 请求时将它转成 string 类型|
|service_type|String|服务类型 `帮我送` `帮我取`|
|weight|Number|单位为 `g` 如 `1` 公斤则填写 `1000`|
|textarea|String|物品描述|

```json
{
  "app_key": "AppKey",
  "startAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "endAddress": "{ "formatted_addresse":"4 Yjsp St", "city":"Mount Eden\nAuckland 1024", "latitude":-36.89357, "longitude":174.759766, "phone":"0220826111", "name":"兔先生" }",
  "service_type": "帮我送",
  "textarea": "红茶叶",
  "weight": 10000
}
```

#### 起点地址终点地址 JSON 
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

#### 返回示例
| 字段  | 格式                               | 备注 |
| ---- | ---------------------------------- | |
|data.totalFee|Number|总价格 不包含 GST 进位保留两位小数点你|
|data.out_trade_no|String|平台唯一订单号|
|data.order_id|Number|平台订单号|

```json
{
  "errno": 0,
  "errmsg": "",
  "data": {
    "totalFee": 25.5875,
    "out_trade_no": "5404633a6434412eb5c64d556852b328",
    "order_id": 198
  }
}
```


