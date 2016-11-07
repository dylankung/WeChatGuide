# 扫描带参数二维码

用户扫描带场景值二维码时，可能推送以下两种事件：

+ 如果用户还未关注公众号，则用户可以关注公众号，关注后微信会将带场景值关注事件推送给开发者。

+ 如果用户已经关注公众号，则微信会将带场景值扫描事件推送给开发者。

## 1. 用户未关注时，进行关注后的事件推送

推送XML数据包示例：

```xml
<xml><ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[FromUser]]></FromUserName>
<CreateTime>123456789</CreateTime>
<MsgType><![CDATA[event]]></MsgType>
<Event><![CDATA[subscribe]]></Event>
<EventKey><![CDATA[qrscene_123123]]></EventKey>
<Ticket><![CDATA[TICKET]]></Ticket>
</xml>
```

|参数  |描述
|:---|:---
|ToUserName  |开发者微信号
|FromUserName    |发送方帐号（一个OpenID）
|CreateTime  |消息创建时间 （整型）
|MsgType |消息类型，event
|Event   |事件类型，subscribe
|EventKey    |事件KEY值，qrscene_为前缀，后面为二维码的参数值
|Ticket  |二维码的ticket，可用来换取二维码图片

## 2. 用户已关注时的事件推送

推送XML数据包示例：

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[FromUser]]></FromUserName>
<CreateTime>123456789</CreateTime>
<MsgType><![CDATA[event]]></MsgType>
<Event><![CDATA[SCAN]]></Event>
<EventKey><![CDATA[SCENE_VALUE]]></EventKey>
<Ticket><![CDATA[TICKET]]></Ticket>
</xml>
```
|参数  |描述
|:----|:---
|ToUserName  |开发者微信号
|FromUserName    |发送方帐号（一个OpenID）
|CreateTime  |消息创建时间 （整型）
|MsgType |消息类型，event
|Event   |事件类型，SCAN
|EventKey    |事件KEY值，是一个32位无符号整数，即创建二维码时的二维码scene_id
|Ticket  |二维码的ticket，可用来换取二维码图片

## 代码

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
                if None != req.get("EventKey"):
                    resp["Content"] += u"场景值:"
                    resp["Content"] += req.get("EventKey")[8:]
            elif "SCAN" == req.get("Event"):
                resp = {
                    "ToUserName":req.get("FromUserName", ""),
                    "FromUserName":req.get("ToUserName", ""),
                    "CreateTime":int(time.time()),
                    "MsgType":"text",
                    "Content":u"您扫描的场景值为：%s" % req.get("EventKey")
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