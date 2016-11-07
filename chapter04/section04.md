# 接收其他普通消息

## 接收图片消息

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>1348831860</CreateTime>
<MsgType><![CDATA[image]]></MsgType>
<PicUrl><![CDATA[this is a url]]></PicUrl>
<MediaId><![CDATA[media_id]]></MediaId>
<MsgId>1234567890123456</MsgId>
</xml>
```
|参数|描述
|:----|:-----
|ToUserName|开发者微信号
|FromUserName|发送方帐号（一个OpenID）
|CreateTime|消息创建时间 （整型）
|MsgType|image
|PicUrl|图片链接
|MediaId|图片消息媒体id，可以调用多媒体文件下载接口拉取数据。
|MsgId|消息id，64位整型

## 接收视频消息

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>1357290913</CreateTime>
<MsgType><![CDATA[video]]></MsgType>
<MediaId><![CDATA[media_id]]></MediaId>
<ThumbMediaId><![CDATA[thumb_media_id]]></ThumbMediaId>
<MsgId>1234567890123456</MsgId>
</xml>
```
|参数|描述
|:----|:-----
|ToUserName|开发者微信号
|FromUserName|发送方帐号（一个OpenID）
|CreateTime|消息创建时间 （整型）
|MsgType|视频为video
|MediaId|视频消息媒体id，可以调用多媒体文件下载接口拉取数据。
|ThumbMediaId|视频消息缩略图的媒体id，可以调用多媒体文件下载接口拉取数据。
|MsgId|消息id，64位整型

## 接收小视频消息

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>1357290913</CreateTime>
<MsgType><![CDATA[shortvideo]]></MsgType>
<MediaId><![CDATA[media_id]]></MediaId>
<ThumbMediaId><![CDATA[thumb_media_id]]></ThumbMediaId>
<MsgId>1234567890123456</MsgId>
</xml>
```
|参数|描述
|:----|:-----
|ToUserName|开发者微信号
|FromUserName|发送方帐号（一个OpenID）
|CreateTime|消息创建时间 （整型）
|MsgType|小视频为shortvideo
|MediaId|视频消息媒体id，可以调用多媒体文件下载接口拉取数据。
|ThumbMediaId|视频消息缩略图的媒体id，可以调用多媒体文件下载接口拉取数据。
|MsgId|消息id，64位整型

## 接收语音消息

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>1357290913</CreateTime>
<MsgType><![CDATA[voice]]></MsgType>
<MediaId><![CDATA[media_id]]></MediaId>
<Format><![CDATA[Format]]></Format>
<MsgId>1234567890123456</MsgId>
</xml>
```
|参数|描述
|:----|:-----
|ToUserName|开发者微信号
|FromUserName|发送方帐号（一个OpenID）
|CreateTime|消息创建时间 （整型）
|MsgType|语音为voice
|MediaId|语音消息媒体id，可以调用多媒体文件下载接口拉取数据。
|Format|语音格式，如amr，speex等
|MsgID|消息id，64位整型

**请注意，开通语音识别后，用户每次发送语音给公众号时，微信会在推送的语音消息XML数据包中，增加一个Recongnition字段（注：由于客户端缓存，开发者开启或者关闭语音识别功能，对新关注者立刻生效，对已关注用户需要24小时生效。开发者可以重新关注此帐号进行测试）。开启语音识别后的语音XML数据包如下：**

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>1357290913</CreateTime>
<MsgType><![CDATA[voice]]></MsgType>
<MediaId><![CDATA[media_id]]></MediaId>
<Format><![CDATA[Format]]></Format>
<Recognition><![CDATA[腾讯微信团队]]></Recognition>
<MsgId>1234567890123456</MsgId>
</xml>
```
**多出的字段中，Format为语音格式，一般为amr，Recognition为语音识别结果，使用UTF8编码。**