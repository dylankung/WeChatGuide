# 鹦鹉学舌代码实现

我们现在来实现一个针对文本消息的收发程序。实现的业务逻辑类似与“鹦鹉学舌”，粉丝发什么内容，我们就传回给粉丝什么内容。

```Python
# coding:utf-8

import tornado.web
import tornado.httpserver
import tornado.ioloop
import tornado.options
import hashlib
import xmltodict
import time

from tornado.web import RequestHandler
from tornado.options import define, options

WECHAT_TOKEN = "itcast"

define("port", default=8080, type=int)

class WeChatHandler(RequestHandler):
    """微信接入接口"""
    def get(self):
        """开发者验证接口"""
        signature = self.get_argument("signature")
        timestamp = self.get_argument("timestamp")
        nonce = self.get_argument("nonce")
        echostr = self.get_argument("echostr")
        tmp = [WECHAT_TOKEN, timestamp, nonce]
        tmp.sort()
        tmp = "".join(tmp)
        tmp = hashlib.sha1(tmp).hexdigest()
        if tmp == signature:
            self.write(echostr)
        else:
            self.write("error")

    def post(self):
        """收发消息接口"""
        req_xml = self.request.body
        req = xmltodict.parse(req_xml)['xml']
        if "text" == req.get("MsgType"):
            resp = {
                "ToUserName":req.get("FromUserName", ""),
                "FromUserName":req.get("ToUserName", ""),
                "CreateTime":int(time.time()),
                "MsgType":"text",
                "Content":req.get("Content", "")
            }
        else:
            resp = {
                "ToUserName":req.get("FromUserName", ""),
                "FromUserName":req.get("ToUserName", ""),
                "CreateTime":int(time.time()),
                "MsgType":"text",
                "Content":"I love you, itcast!"
            }
        resp_xml = xmltodict.unparse({"xml":resp})
        self.write(resp_xml)


def main():
    tornado.options.parse_command_line()
    app = tornado.web.Application([
            (r"/wechat", WeChatHandler),
        ])
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.current().start()


if __name__ == '__main__':
    main()
```

<img alt="效果图" src="/images/receive_reply_msg.png" width="375" height="667">

### 有趣的表情

#### QQ表情

实际是字符串转义，如 `/::D`、`/::P` 等，仍属于文本信息。

![qq_face_1.jpg](/images/qq_face_1.jpg)
![qq_face_2.jpg](/images/qq_face_2.jpg)
![qq_face_3.jpg](/images/qq_face_3.jpg)
![qq_face_4.jpg](/images/qq_face_4.jpg)

#### emoji

绘文字（日语：絵文字/えもじ emoji）是日本在无线通信中所使用的视觉情感符号，绘意指图形，文字则是图形的隐喻，可用来代表多种表情，如笑脸表示笑、蛋糕表示食物等。

在NTTDoCoMo的i-mode系统电话系统中，绘文字的尺寸是12x12 像素，在传送时，一个图形有2个字节。**Unicode编码为E63E到E757**，而在Shift-JIS编码则是从F89F到F9FC。基本的绘文字共有176个符号，在C-HTML4.0的编程语言中，则另增添了76个情感符号。

最早由栗田穰崇（Shigetaka Kurita）创作，并在日本网络及手机用户中流行。

自苹果公司发布的iOS 5输入法中加入了emoji后，这种表情符号开始席卷全球，目前emoji已被大多数现代计算机系统所兼容的Unicode编码采纳，普遍应用于各种手机短信和社交网络中。

**本质是Unicode字符**，也属于文本消息。

#### 自定表情

微信的自定义表情不是文本，也不是图片，而是一种不支持的格式，微信未提供处理此消息的接口。

## 改写代码

微信发送的请求中会携带签名验证信息（正如验证服务器有效性一章节所示），我们需要对收到的请求进行验证是否来自微信服务器，所以在处理请求前都要按照验证算法进行检验。

```python
class WeChatBaseHandler(RequestHandler):
    def prepare(self):
        """验证请求是否来自微信服务器"""
        signature = self.get_argument("signature")
        timestamp = self.get_argument("timestamp")
        nonce = self.get_argument("nonce")
        tmp = [WECHAT_TOKEN, timestamp, nonce]
        tmp.sort()
        tmp = "".join(tmp)
        tmp = hashlib.sha1(tmp).hexdigest()
        if tmp != signature:
            self.send_error(403) # 若是非法请求，返回403错误

class WeChatHandler(WeChatBaseHandler):
    """微信接入接口"""
    def get(self):
        """开发者验证接口"""
        echostr = self.get_argument("echostr")
        self.write(echostr)

    def post(self):
        """收发消息接口"""
        req_xml = self.request.body
        req = xmltodict.parse(req_xml)['xml']
        if "text" == req.get("MsgType"):
            resp = {
                "ToUserName":req.get("FromUserName", ""),
                "FromUserName":req.get("ToUserName", ""),
                "CreateTime":int(time.time()),
                "MsgType":"text",
                "Content":req.get("Content", "")
            }
        else:
            resp = {
                "ToUserName":req.get("FromUserName", ""),
                "FromUserName":req.get("ToUserName", ""),
                "CreateTime":int(time.time()),
                "MsgType":"text",
                "Content":"I love you, itcast!"
            }
        resp_xml = xmltodict.unparse({"xml":resp})
        self.write(resp_xml)
```