# 获取流程

## 1. 设置网页授权回调域名

在微信公众号请求用户网页授权之前，开发者需要先到公众平台官网中的开发者中心页配置授权回调域名。请注意，这里填写的是域名（是一个字符串），而不是URL，因此请勿加 http:// 等协议头；

授权回调域名配置规范为全域名，比如需要网页授权的域名为：www.qq.com，配置以后此域名下面的页面http://www.qq.com/music.html 、 http://www.qq.com/login.html 都可以进行OAuth2.0鉴权。但http://pay.qq.com 、 http://music.qq.com 、 http://qq.com无法进行OAuth2.0鉴权。

![配置页面](/images/oauth2_host_config.png)
![配置页面](/images/oauth2_host_config2.png)

## 2. 用户同意授权，获取code

让用户访问一下链接地址：
> https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect

![参数说明](/images/oauth2_get_code.png)

下图为scope等于snsapi_userinfo时的授权页面：

![授权页面](/images/snsapi_userinfo.png)

**用户同意授权后**

如果用户同意授权，页面将跳转至 redirect_uri/?code=CODE&state=STATE。若用户禁止授权，则重定向后不会带上code参数，仅会带上state参数redirect_uri?state=STATE

## 3. 通过code换取网页授权access_token

**请求方法**

> https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code

**参数说明**

![参数说明](/images/oauth2_access_token.png)

**返回值**

正确时返回的JSON数据包如下：

```JSON
{
   "access_token":"ACCESS_TOKEN",
   "expires_in":7200,
   "refresh_token":"REFRESH_TOKEN",
   "openid":"OPENID",
   "scope":"SCOPE"
}
```
![返回值](/images/oauth2_access_token_response.png)

错误时微信会返回JSON数据包如下（示例为Code无效错误）:

```JSON
{
    "errcode":40029,
    "errmsg":"invalid code"
}
```

## 4. 拉取用户信息(需scope为 snsapi_userinfo)

**请求方法**

> https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN

**参数说明**

![参数说明](/images/oauth2_get_userinfo_param.png)

**返回值**

正确时返回的JSON数据包如下：

```JSON
{
   "openid":" OPENID",
   " nickname": NICKNAME,
   "sex":"1",
   "province":"PROVINCE"
   "city":"CITY",
   "country":"COUNTRY",
    "headimgurl":    "http://wx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/46", 
    "privilege":[
    "PRIVILEGE1"
    "PRIVILEGE2"
    ],
    "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
}
```
![返回值](/images/oauth2_get_userinfo_response.png)

错误时微信会返回JSON数据包如下:

```JSON
{
    "errcode":40003,
    "errmsg":" invalid openid "
}
```