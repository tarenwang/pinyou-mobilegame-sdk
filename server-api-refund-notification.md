# 内购商品退款通知(Refund Notification)

**修改记录**

| 修订号   | 修改描述       | 修改日期       |
| ----- | -------- |  ---------- |
| 1.0.0 | 初稿完成       | 2021-03-16 |

### 接口说明

玩家在google play和apple store后台申请内购商品订单退款后，需要告知游戏服务器。

### 请求地址

开发商自定并告知发行商（举例：[http://yourdomain.com/ipn.php](http://yourdomain.com/ipn.php)）

### 请求方式(发行商服务器请求游戏商服务器)

POST

### 请求参数

|字段名|是否必填|类型|说明|
|---|---|---|---|
|appId|是|String|SDK平台分配的游戏id|
|accountId|是|String|SDK平台帐号|
|orderId|是|String|SDK平台订单号|
|extra|否|string|游戏请求生成订单时的透传字段，回调时保持原样返回；这里建议传递游戏内部生成的订单号|
|signature|是|string|签名 [参考签名规则](server-api-overview.md#签名规则)|

**本接口中参与签名计算的参数包含appId、accountId、orderId**

### 返回结果TEXT

```
// 成功
success
// 失败示例
failure
```
**注意：重复退单的通知(orderId重复)，游戏方不需要重复处理，但是接口必须返回成功**
