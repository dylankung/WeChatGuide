# 上代码

**后端Python程序（以Tornado框架为例）**

```Python
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
                self.render("user.html", user=ret)
        else:
            self.write(ret.get("errmsg", "获取access_token错误"))
```

**前端user.html文件代码**

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
</body>
</html>
```