# 带参数的二维码

为了满足用户渠道推广分析和用户帐号绑定等场景的需要，公众平台提供了生成带参数二维码的接口。使用该接口可以获得多个带不同场景值的二维码，用户扫描后，公众号可以接收到事件推送。

目前有2种类型的二维码：

1. 临时二维码，是有过期时间的，最长可以设置为在二维码生成后的30天（即2592000秒）后过期，但能够生成较多数量。临时二维码主要用于帐号绑定等不要求二维码永久保存的业务场景

2. 永久二维码，是无过期时间的，但数量较少（目前为最多10万个）。永久二维码主要用于适用于帐号绑定、用户来源统计等场景。

**获取带参数的二维码的过程包括两步，首先创建二维码ticket，然后凭借ticket到指定URL换取二维码。**

## 创建二维码ticket

每次创建二维码ticket需要提供一个开发者自行设定的参数（scene_id），分别介绍临时二维码和永久二维码的创建二维码ticket过程。

#### 临时二维码请求说明

```
http请求方式: POST
URL: https://api.weixin.qq.com/cgi-bin/qrcode/create?access_token=TOKEN
POST数据格式：json
POST数据例子：{"expire_seconds": 604800, "action_name": "QR_SCENE", "action_info": {"scene": {"scene_id": 123}}}
```

#### 永久二维码请求说明

```
http请求方式: POST
URL: https://api.weixin.qq.com/cgi-bin/qrcode/create?access_token=TOKEN
POST数据格式：json
POST数据例子：{"action_name": "QR_LIMIT_SCENE", "action_info": {"scene": {"scene_id": 123}}}
或者也可以使用以下POST数据创建字符串形式的二维码参数：
{"action_name": "QR_LIMIT_STR_SCENE", "action_info": {"scene": {"scene_str": "123"}}}
```

|参数  |说明
|:---|:----
|expire_seconds  |该二维码有效时间，以秒为单位。 最大不超过2592000（即30天），此字段如果不填，则默认有效期为30秒。
|action\_name |二维码类型，QR\_SCENE为临时,QR\_LIMIT\_SCENE为永久,QR\_LIMIT\_STR\_SCENE为永久的字符串参数值
|action_info |二维码详细信息
|scene_id    |场景值ID，临时二维码时为32位非0整型，永久二维码时最大值为100000（目前参数只支持1--100000）
|scene_str   |场景值ID（字符串形式的ID），字符串类型，长度限制为1到64，仅永久二维码支持此字段

#### 返回说明

正确的Json返回结果:

```json
{"ticket":"gQH47joAAAAAAAAAASxodHRwOi8vd2VpeGluLnFxLmNvbS9xL2taZ2Z3TVRtNzJXV1Brb3ZhYmJJAAIEZ23sUwMEmm3sUw==","expire_seconds":60,"url":"http:\/\/weixin.qq.com\/q\/kZgfwMTm72WWPkovabbI"}
```

|参数  |说明
|:----|:---
|ticket  |获取的二维码ticket，凭借此ticket可以在有效时间内换取二维码。
|expire_seconds  |该二维码有效时间，以秒为单位。 最大不超过2592000（即30天）。
|url |二维码图片解析后的地址，开发者可根据该地址自行生成需要的二维码图片

错误的Json返回示例:
```json
{"errcode":40013,"errmsg":"invalid appid"}
```

## 通过ticket换取二维码

获取二维码ticket后，开发者可用ticket换取二维码图片。请注意，本接口无须登录态即可调用。

请求说明

```
HTTP GET请求（请使用https协议）
https://mp.weixin.qq.com/cgi-bin/showqrcode?ticket=TICKET
```

## 代码实例

```python
class QRCodeHandler(RequestHandler):
    @tornado.gen.coroutine
    def get(self):
        access_token = yield AccessToken.get_access_token()
        print "access_token", access_token
        url = "https://api.weixin.qq.com/cgi-bin/qrcode/create?access_token=%s" % access_token
        scene_id = self.get_argument("scene_id")
        req_body = '{"expire_seconds": 7200, "action_name": "QR_SCENE", "action_info": {"scene": {"scene_id": %s}}}' % scene_id
        client = AsyncHTTPClient()
        req = HTTPRequest(url, method="POST", body=req_body)
        resp = yield client.fetch(req)
        if "errcode" in resp.body:
            self.write("error")
        else:
            resp_data = json.loads(resp.body)
            ticket = resp_data['ticket']
            self.write('<img src="https://mp.weixin.qq.com/cgi-bin/showqrcode?ticket=%s">' % ticket)
```
