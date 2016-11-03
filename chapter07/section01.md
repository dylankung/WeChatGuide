# 开发流程

## 1. 绑定域名

![绑定域名](/images/js_host_config.png)

## 2. 引入JS文件

在需要调用JS接口的页面引入如下JS文件，（支持https）：

> http://res.wx.qq.com/open/js/jweixin-1.0.0.js

## 3. 通过config接口注入权限验证配置

```Javascript
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，公众号的唯一标识
    timestamp: , // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，签名
    jsApiList: [] // 必填，需要使用的JS接口列表
});
```

### signature的生成

生成签名之前必须先了解一下jsapi\_ticket，jsapi\_ticket是公众号用于调用微信JS接口的临时票据。正常情况下，jsapi\_ticket的有效期为7200秒，通过access\_token来获取。由于获取jsapi\_ticket的api调用次数非常有限，频繁刷新jsapi\_ticket会导致api调用受限，影响自身业务，开发者必须在自己的服务全局缓存jsapi\_ticket 。

1. 获取jsapi_ticket

> https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi

```JSON
{
    "errcode":0,
    "errmsg":"ok",
    "ticket":"bxLdikRXVbTPdHSM05e5u5sUoXNKd8-41ZO3MhKoyN5OfkWITDGgnr2fwJ0m9E8NYzWKVZvdVtaUgWvsdshFKA",
    "expires_in":7200
}
```

2. 计算signature

签名生成规则如下：参与签名的字段包括noncestr（随机字符串）, 有效的jsapi\_ticket, timestamp（时间戳）, url（当前网页的URL，不包含#及其后面部分）。对所有待签名参数按照字段名的ASCII码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。这里需要注意的是所有参数名均为小写字符。对string1作sha1加密，字段名和字段值都采用原始值，不进行URL 转义。

**拼接字符串**

```
jsapi_ticket=sM4AOVdWfPE4DxkXGEs8VMCPGGVi4C3VM0P37wVUCFvkVAy_90u5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcHKP7qg&noncestr=Wm3WZYTPz0wzccnW&timestamp=1414587457&url=http://mp.weixin.qq.com?params=value
```

**进行sha1签名, 得到signature**

```
0f9de62fce790f9a083d5c99e95740ceb90c27ed
```

## 4. 通过ready接口处理成功验证

```Javascript
wx.ready(function(){

    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
});
```