# 回复其他普通消息

## 回复图片消息

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>12345678</CreateTime>
<MsgType><![CDATA[image]]></MsgType>
<Image>
<MediaId><![CDATA[media_id]]></MediaId>
</Image>
</xml>
```
|参数|是否必须|说明
|:---|:------|:---
|ToUserName|是|接收方帐号（收到的OpenID）
|FromUserName|是|开发者微信号
|CreateTime|是|消息创建时间 （整型）
|MsgType|是|image
|MediaId|是|通过素材管理接口上传多媒体文件，得到的id。

## 回复语音消息

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>12345678</CreateTime>
<MsgType><![CDATA[voice]]></MsgType>
<Voice>
<MediaId><![CDATA[media_id]]></MediaId>
</Voice>
</xml>
```

|参数  |是否必须    |说明
|:---|:------|:---
|ToUserName  |是   |接收方帐号（收到的OpenID）
|FromUserName    |是   |开发者微信号
|CreateTime  |是   |消息创建时间戳 （整型）
|MsgType |是   |语音，voice
|MediaId |是   |通过素材管理接口上传多媒体文件，得到的id

## 回复视频消息

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>12345678</CreateTime>
<MsgType><![CDATA[video]]></MsgType>
<Video>
<MediaId><![CDATA[media_id]]></MediaId>
<Title><![CDATA[title]]></Title>
<Description><![CDATA[description]]></Description>
</Video> 
</xml>
```

|参数  |是否必须    |说明
|:----|:----|:----
|ToUserName  |是   |接收方帐号（收到的OpenID）
|FromUserName    |是  |开发者微信号
|CreateTime  |是   |消息创建时间 （整型）
|MsgType |是   |video
|MediaId |是   |通过素材管理接口上传多媒体文件，得到的id
|Title   |否   |视频消息的标题
|Description |否   |视频消息的描述