# 鹦鹉学舌代码实现

我们现在来实现一个针对文本消息的收发程序。实现的业务逻辑类似与“鹦鹉学舌”，粉丝发什么内容，我们就传回给粉丝什么内容。

```Python
from lxml import etree
from xml.etree.ElementTree import Element
from xml.etree.ElementTree import tostring

def Xml2Dict(raw_xml):
    """xml转为字典"""
    xmlstr = etree.fromstring(raw_xml)
    dict_xml = {}
    for child in xmlstr:
        dict_xml[child.tag] = child.text
    return dict_xml


def Dict2Xml(tag, d):
    """字典转为xml"""
    elem = Element(tag)
    for key, val in d.items():
        child = Element(key)
        if isinstance(val, unicode):
            child.text = val
        else:
            child.text=str(val)
        elem.append(child)
    my_str = tostring(elem, encoding='UTF-8')
    return my_str
    

class WechatHandler(tornado.web.RequestHandler):
    """微信接入接口"""
    def get(self):
        """开发者验证接口"""
        signature = self.get_argument("signature", "")
        timestamp = self.get_argument("timestamp", "")
        nonce = self.get_argument("nonce", "")
        echostr = self.get_argument("echostr", "")
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
        req = Xml2Dict(req_xml)
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
        resp_xml = Dict2Xml("xml", resp)
        self.write(resp_xml)
```

<img alt="效果图" src="/images/receive_reply_msg.png" width="375" height="667">