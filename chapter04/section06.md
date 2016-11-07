# 回复用户语音消息识别

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