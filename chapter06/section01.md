# 创建自定义菜单

## 关于菜单

1. 自定义菜单最多包括3个一级菜单，每个一级菜单最多包含5个二级菜单。
2. 一级菜单最多4个汉字，二级菜单最多7个汉字，多出来的部分将会以“...”代替。
3. 创建自定义菜单后，菜单的刷新策略是，在用户进入公众号会话页或公众号profile页时，如果发现上一次拉取菜单的请求在5分钟以前，就会拉取一下菜单，如果菜单有更新，就会刷新客户端的菜单。测试时可以尝试取消关注公众账号后再次关注，则可以看到创建后的效果。

**菜单类型**

1. click：点击推事件
用户点击click类型按钮后，微信服务器会通过消息接口推送消息类型为event 的结构给开发者（参考消息接口指南），并且带上按钮中开发者填写的key值，开发者可以通过自定义的key值与用户进行交互；
2. view：跳转URL
用户点击view类型按钮后，微信客户端将会打开开发者在按钮中填写的网页URL，可与网页授权获取用户基本信息接口结合，获得用户基本信息。
3. scancode_push：扫码推事件
用户点击按钮后，微信客户端将调起扫一扫工具，完成扫码操作后显示扫描结果（如果是URL，将进入URL），且会将扫码的结果传给开发者，开发者可以下发消息。
4. scancode_waitmsg：扫码推事件且弹出“消息接收中”提示框
用户点击按钮后，微信客户端将调起扫一扫工具，完成扫码操作后，将扫码的结果传给开发者，同时收起扫一扫工具，然后弹出“消息接收中”提示框，随后可能会收到开发者下发的消息。
5. pic_sysphoto：弹出系统拍照发图
用户点击按钮后，微信客户端将调起系统相机，完成拍照操作后，会将拍摄的相片发送给开发者，并推送事件给开发者，同时收起系统相机，随后可能会收到开发者下发的消息。
6. pic_photo_or_album：弹出拍照或者相册发图
用户点击按钮后，微信客户端将弹出选择器供用户选择“拍照”或者“从手机相册选择”。用户选择后即走其他两种流程。
7. pic_weixin：弹出微信相册发图器
用户点击按钮后，微信客户端将调起微信相册，完成选择操作后，将选择的相片发送给开发者的服务器，并推送事件给开发者，同时收起相册，随后可能会收到开发者下发的消息。
8. location_select：弹出地理位置选择器
用户点击按钮后，微信客户端将调起地理位置选择工具，完成选择操作后，将选择的地理位置发送给开发者的服务器，同时收起位置选择工具，随后可能会收到开发者下发的消息。
9. media_id：下发消息（除文本消息）
用户点击media_id类型按钮后，微信服务器会将开发者填写的永久素材id对应的素材下发给用户，永久素材类型可以是图片、音频、视频、图文消息。请注意：永久素材id必须是在“素材管理/新增永久素材”接口上传后获得的合法id。
10. view_limited：跳转图文消息URL
用户点击view_limited类型按钮后，微信客户端将打开开发者在按钮中填写的永久素材id对应的图文消息URL，永久素材类型只支持图文消息。请注意：永久素材id必须是在“素材管理/新增永久素材”接口上传后获得的合法id。

## 接口说明

**请求方法**

POST方式

> https://api.weixin.qq.com/cgi-bin/menu/create?access_token=ACCESS_TOKEN

**参数说明**

```JSON
{
    "button":[
    {
        "type":"click",
        "name":"今日歌曲",
        "key":"V1001_TODAY_MUSIC"
    },
    {
        "name":"菜单",
        "sub_button":[
        {
            "type":"view",
            "name":"搜索",
            "url":"http://www.soso.com/"
        },
        {
            "type":"view",
            "name":"视频",
            "url":"http://v.qq.com/"
        },
        {
            "type":"click",
            "name":"赞一下我们",
            "key":"V1001_GOOD"
        }]
    }]
}
```

![参数说明](/images/create_menu_param.png)

**返回值**

正确时返回的JSON数据包如下：

```JSON
{
    "errcode":0,
    "errmsg":"ok"
}
```

错误时微信会返回JSON数据包如下:

```JSON
{
    "errcode":40018,
    "errmsg":"invalid button name size"
}
```

## 代码实现

```Python
from urllib import quote

class CreateMenuHandler(tornado.web.RequestHandler):
    """创建微信菜单"""
    @tornado.gen.coroutine
    def get(self):
        access_token = yield AccessToken.get_access_token() 
        if not access_token:
            self.write("token error")
        else:
            menu_data = {
                "button":[{
                    "type":"view",
                    "name":"测试网页链接",
                    "url":"https://open.weixin.qq.com/connect/oauth2/authorize?appid=%s&redirect_uri=%s&response_type=code&scope=snsapi_userinfo&state=1#wechat_redirect" % (WECHAT_APPID, quote("http://wechat.idehai.com/wechat/user"))
                    },
                ]
            }
            client = AsyncHTTPClient()
            url = "https://api.weixin.qq.com/cgi-bin/menu/create?access_token=%s" % access_token
            req = HTTPRequest(url, method="POST", body=json.dumps(menu_data, ensure_ascii=False))
            resp = yield client.fetch(req)
            ret = json.loads(resp.body)
            if 0 == ret.get("errcode"):
                self.write("创建菜单成功")
            else:
                self.write(ret.get("errmsg", "创建菜单失败"))

```

<img alt="自定义菜单" src="/images/menu.png" width="375" height="667">
