# 验证服务器地址的有效性

开发者提交信息后，微信服务器将发送GET请求到填写的服务器地址URL上，GET请求携带四个参数：

![请求参数](/images/valid_server.png)

开发者通过检验signature对请求进行校验。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败。

校验流程：
1. 将token、timestamp、nonce三个参数进行字典序排序
2. 将三个参数字符串拼接成一个字符串进行sha1加密
3. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信

Python代码实现（以Tornado框架为例）：
```Python
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
```