# 关注/取消关注事件

用户在关注与取消关注公众号时，微信会把这个事件推送到开发者填写的URL。

微信服务器在五秒内收不到响应会断掉连接，并且重新发起请求，总共重试三次。

假如服务器无法保证在五秒内处理并回复，可以直接回复空串，微信服务器不会对此作任何处理，并且不会发起重试。

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[FromUser]]></FromUserName>
<CreateTime>123456789</CreateTime>
<MsgType><![CDATA[event]]></MsgType>
<Event><![CDATA[subscribe]]></Event>
</xml>
```

|参数  |描述
|:----|:----
|ToUserName  |开发者微信号
|FromUserName    |发送方帐号（一个OpenID）
|CreateTime  |消息创建时间 （整型）
|MsgType |消息类型，event
|Event   |事件类型，subscribe(订阅)、unsubscribe(取消订阅)


```python
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
        msg_type = req.get("MsgType")
        if "text" == msg_type:
            resp = {
                "ToUserName":req.get("FromUserName", ""),
                "FromUserName":req.get("ToUserName", ""),
                "CreateTime":int(time.time()),
                "MsgType":"text",
                "Content":req.get("Content", "")
            }
        elif "voice" == msg_type:
            resp = {
                "ToUserName":req.get("FromUserName", ""),
                "FromUserName":req.get("ToUserName", ""),
                "CreateTime":int(time.time()),
                "MsgType":"text",
                "Content":req.get("Recognition", u"未识别")
            }
        elif "event" == msg_type:
            if "subscribe" == req.get("Event"):
                resp = {
                     "ToUserName":req.get("FromUserName", ""),
                    "FromUserName":req.get("ToUserName", ""),
                    "CreateTime":int(time.time()),
                    "MsgType":"text",
                    "Content":u"感谢您的关注！"
                }
            else:
                resp = None
        else:
            resp = {
                "ToUserName":req.get("FromUserName", ""),
                "FromUserName":req.get("ToUserName", ""),
                "CreateTime":int(time.time()),
                "MsgType":"text",
                "Content":"I love you, itcast!"
            }
        if resp:
            resp_xml = xmltodict.unparse({"xml":resp})
        else:
            resp_xml = ""
        self.write(resp_xml)
```


