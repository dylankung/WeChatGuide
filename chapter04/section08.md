# 获取接口调用凭据

access\_token是公众号的全局唯一票据，公众号调用各接口时都需使用access\_token。开发者需要进行妥善保存。access\_token的存储至少要保留512个字符空间。access\_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access\_token失效。

## 接口说明

**请求方法**

> https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET

**参数说明**

![参数说明](/images/get_access_token_param.png)

**返回值**

正确时返回的JSON数据包如下：

```JSON
{
    "access_token":"ACCESS_TOKEN",
    "expires_in":7200
}
```
![返回值](/images/get_access_token_response.png)

错误时微信会返回JSON数据包如下:

```JSON
{
    "errcode":40013,
    "errmsg":"invalid appid"
}
```

## 代码实现

```Python
class AccessToken(object):
    """微信接口调用Token"""
    _access_token = {
        "token":"",
        "updatetime":datetime.datetime.now()
    }

    @classmethod
    @tornado.gen.coroutine
    def update_access_token(cls):
        """更新access_token"""
        client = AsyncHTTPClient()
        url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=%s&secret=%s" % (WECHAT_APPID, WECHAT_APPSECRET)
        resp = yield client.fetch(url)
        print resp.body
        ret = json.loads(resp.body)
        token = ret.get("access_token")
        if token:
            cls._access_token["token"] = token
            cls._access_token['updatetime'] = datetime.datetime.now()

    @classmethod
    @tornado.gen.coroutine
    def get_access_token(cls):
        """获取access_token"""
        if not cls._access_token["token"] or (datetime.datetime.now()-cls._access_token["updatetime"]).seconds>=6600:
            yield cls.update_access_token()
        raise tornado.gen.Return(cls._access_token["token"])
```
