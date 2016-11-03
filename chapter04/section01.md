# 接收普通消息

**当普通微信用户向公众账号发消息时，微信服务器将POST消息的XML数据包到开发者填写的URL上。**

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
