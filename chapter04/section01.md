# 接收普通消息

**当普通微信用户向公众账号发消息时，微信服务器将POST消息的XML数据包到开发者填写的URL上。**

微信服务器在五秒内收不到响应会断掉连接，并且重新发起请求，总共重试三次。假如服务器无法保证在五秒内处理并回复，可以直接回复空串，微信服务器不会对此作任何处理，并且不会发起重试。

各消息类型的推送使用XML数据包结构，如：

```xml
<xml>
<ToUserName><![CDATA[gh_866835093fea]]></ToUserName>
<FromUserName><![CDATA[ogdotwSc_MmEEsJs9-ABZ1QL_4r4]]></FromUserName>
<CreateTime>1478317060</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[你好]]></Content>
<MsgId>6349323426230210995</MsgId>
</xml>
```
**注意：`<![CDATA` 与 `]]>` 括起来的数据不会被xml解析器解析。**

## xmltodict 模块基本用法

xmltodict 是一个用来处理xml数据的很方便的模块。包含两个常用方法parse和unparse

### 1. parse

xmltodict.parse()方法可以将xml数据转为python中的dict字典数据：

```python
>>> import xmltodict
>>> xml_str = """
... <xml>
... <ToUserName><![CDATA[gh_866835093fea]]></ToUserName>
... <FromUserName><![CDATA[ogdotwSc_MmEEsJs9-ABZ1QL_4r4]]></FromUserName>
... <CreateTime>1478317060</CreateTime>
... <MsgType><![CDATA[text]]></MsgType>
... <Content><![CDATA[你好]]></Content>
... <MsgId>6349323426230210995</MsgId>
... </xml>
... """
>>>
>>> xml_dict = xmltodict.parse(xml_str)
>>> type(xml_dict)
<class 'collections.OrderedDict'>  # 类字典型，可以按照字典方法操作
>>>
>>> xml_dict
OrderedDict([(u'xml', OrderedDict([(u'ToUserName', u'gh_866835093fea'), (u'FromUserName', u'ogdotwSc_MmEEsJs9-ABZ1QL_4r4'), (u'CreateTime', u'1478317060'), (u'MsgType', u'text'), (u'Content', u'\u4f60\u597d'), (u'MsgId', u'6349323426230210995')]))])
>>>
>>> xml_dict['xml']
OrderedDict([(u'ToUserName', u'gh_866835093fea'), (u'FromUserName', u'ogdotwSc_MmEEsJs9-ABZ1QL_4r4'), (u'CreateTime', u'1478317060'), (u'MsgType', u'text'), (u'Content', u'\u4f60\u597d'), (u'MsgId', u'6349323426230210995')])
>>>
>>> for key, val in xml_dict['xml'].items():
...     print key, "=", val
... 
ToUserName = gh_866835093fea
FromUserName = ogdotwSc_MmEEsJs9-ABZ1QL_4r4
CreateTime = 1478317060
MsgType = text
Content = 你好
MsgId = 6349323426230210995
>>> 
```

### 2. unparse

xmltodict.unparse()方法可以将字典转换为xml字符串：

```python
xml_dict = {
    "xml": {
        "ToUserName" : "gh_866835093fea",
        "FromUserName" : "ogdotwSc_MmEEsJs9-ABZ1QL_4r4",
        "CreateTime" : "1478317060",
        "MsgType" : "text",
        "Content" : u"你好",
        "MsgId" : "6349323426230210995",
    }
}

>>> xml_str = xmltodict.unparse(xml_dict)
>>> print xml_str
<?xml version="1.0" encoding="utf-8"?>
<xml><FromUserName>ogdotwSc_MmEEsJs9-ABZ1QL_4r4</FromUserName><MsgId>6349323426230210995</MsgId><ToUserName>gh_866835093fea</ToUserName><Content>你好</Content><MsgType>text</MsgType><CreateTime>1478317060</CreateTime></xml>
>>>
>>> xml_str = xmltodict.unparse(xml_dict, pretty=True) # pretty表示友好输出
>>> print xml_str
<?xml version="1.0" encoding="utf-8"?>
<xml>
    <FromUserName>ogdotwSc_MmEEsJs9-ABZ1QL_4r4</FromUserName>
    <MsgId>6349323426230210995</MsgId>
    <ToUserName>gh_866835093fea</ToUserName>
    <Content>你好</Content>
    <MsgType>text</MsgType>
    <CreateTime>1478317060</CreateTime>
</xml>
>>> 
```

## 普通消息类别

1. 文本消息
2. 图片消息
3. 语音消息
4. 视频消息
5. 小视频消息
6. 地理位置消息
7. 链接消息

## 文本消息

```XML
 <xml>
 <ToUserName><![CDATA[toUser]]></ToUserName>
 <FromUserName><![CDATA[fromUser]]></FromUserName> 
 <CreateTime>1348831860</CreateTime>
 <MsgType><![CDATA[text]]></MsgType>
 <Content><![CDATA[this is a test]]></Content>
 <MsgId>1234567890123456</MsgId>
 </xml>
```

![文本消息报文说明](/images/receive_text_msg.png)
