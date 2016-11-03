# 上代码

** 后端Python代码**

```Python
class JsapiTicket(object):
    """微信js接口凭证"""
    _jsapi_ticket = {
        "ticket":"kgt8ON7yVITDhtdwci0qeY2aVYbJVdZOHB5ceViWWImj1PGpJNlcWjwRyfijQR0YFa2CtVpK7AYE3aK4kJ7ZdQ",
        "updatetime":datetime.datetime.now()
    }

    @classmethod
    @tornado.gen.coroutine
    def update_jsapi_ticket(cls):
        """更新jsapi_ticket"""
        client = AsyncHTTPClient()
        url = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=%s&type=jsapi" % AccessToken.get_access_token()
        resp = yield client.fetch(url)
        print resp.body
        ret = json.loads(resp.body)
        if 0 == ret.get("errcode"):
            cls._jsapi_ticket["ticket"] = ret.get("ticket")
            cls._jsapi_ticket['updatetime'] = datetime.datetime.now()

    @classmethod
    def get_jsapi_ticket(cls):
        """获取jsapi_ticket"""
        if not cls._jsapi_ticket["ticket"] or (datetime.datetime.now()-cls._jsapi_ticket["updatetime"]).seconds>=6600:
            cls.update_jsapi_ticket()
        return cls._jsapi_ticket["ticket"]


class UserHandler(tornado.web.RequestHandler):
    """获取粉丝信息页面"""
    @tornado.gen.coroutine
    def get(self):
        code = self.get_argument("code")
        if not code:
            self.write("您未授权，无法获取您的信息！")
            return
        client = AsyncHTTPClient()
        url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=%s&secret=%s&code=%s&grant_type=authorization_code" % (WECHAT_APPID, WECHAT_APPSECRET, code) 
        resp = yield client.fetch(url)
        ret = json.loads(resp.body)
        access_token = ret.get("access_token")
        if access_token:
            openid = ret.get("openid")
            url = "https://api.weixin.qq.com/sns/userinfo?access_token=%s&openid=%s&lang=zh_CN" % (access_token, openid)
            resp = yield client.fetch(url)
            ret = json.loads(resp.body)
            if ret.get("errcode"):
                self.write(ret.get("errmsg", "获取个人信息错误"))
            else:
                jsapi_ticket = JsapiTicket.get_jsapi_ticket()
                timestamp = int(time.time())
                sign_str = "jsapi_ticket=%s&noncestr=itcast&timestamp=%s&url=%s" % (jsapi_ticket, timestamp, "http://"+self.request.host+self.request.uri)
                print sign_str
                signature = hashlib.sha1(sign_str).hexdigest()
                sign_data = {
                    "appId":WECHAT_APPID,
                    "timestamp":timestamp,
                    "noncestr":"itcast",
                    "signature":signature 
                }
                self.render("user.html", user=ret, sign_data=sign_data)
        else:
            self.write(ret.get("errmsg", "获取access_token错误"))
```

**前段user.html代码**

```HTML
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>{{user["nickname"]}}的个人主页</title>
</head>
<body>
    <img alt="头像" src="{{user['headimgurl']}}">
    <table>
        <tr>
            <th>openid</th>
            <td>{{user["openid"]}}</td>
        </tr>
        <tr>
            <th>昵称</th>
            <td>{{user["nickname"]}}</td>
        </tr>
        <tr>
            <th>性别</th>
            <td>
                {% if 1 == user["sex"] %}
                    男
                {% elif 2 == user["sex"] %}
                    女
                {% else %}
                    未知
                {% end if %}
            </td>
        </tr>
        <tr>
            <th>省份</th>
            <td>{{user["province"]}}</td>
        </tr>
        <tr>
            <th>城市</th>
            <td>{{user["city"]}}</td>
        </tr>
        <tr>
            <th>国家</th>
            <td>{{user["country"]}}</td>
        </tr>
    </table> 
    <script src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js" type="text/javascript"></script>
    <script type="text/javascript">
        wx.config({
            debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
            appId: '{{sign_data["appId"]}}', // 必填，公众号的唯一标识
            timestamp: {{sign_data["timestamp"]}}, // 必填，生成签名的时间戳
            nonceStr: '{{sign_data["noncestr"]}}', // 必填，生成签名的随机串
            signature: '{{sign_data["signature"]}}',// 必填，签名，见附录1
            jsApiList: [
                'onMenuShareTimeline',
                'onMenuShareAppMessage'
            ] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
        }); 
        wx.ready(function(){
            wx.onMenuShareTimeline({
                title: '{{user["nickname"]}}说：你丑你先睡，我帅我直播！', // 分享标题
                link: 'http://www.itcast.cn/subject/pythonopen/index.shtml?open', // 分享链接
                imgUrl: '{{user["headimgurl"]}}', // 分享图标
                success: function () { 
                    alert("分享成功！");
                },
                cancel: function () { 
                    alert("分享失败！");
                }
            }); 
            wx.onMenuShareAppMessage({
                title: '{{user["nickname"]}}在上传智公开课', // 分享标题
                desc: '{{user["nickname"]}}说：好喜欢传智Python课程', // 分享描述
                link: 'http://www.itcast.cn/subject/pythonzly/index.shtml', // 分享链接
                imgUrl: '{{user["headimgurl"]}}', // 分享图标
                type: 'link', // 分享类型,music、video或link，不填默认为link
                dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
                success: function () { 
                    alert("分享成功！");
                },
                cancel: function () { 
                    alert("分享失败！");
                }
            }); 
        });
    </script>
</body>
</html>
```

<img alt="分享给朋友" src="/images/share1.png" width="375" height="667">
<img alt="分享到朋友圈" src="/images/share2.png" width="375" height="667">