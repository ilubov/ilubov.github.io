---
title: 企业微信电脑客户端打开系统默认浏览器
date: 2021-01-17 10:33:25
tags:
    - java
categories:
    - java
---

### 企业微信电脑客户端打开系统默认浏览器

#### [JS-SDK](https://work.weixin.qq.com/api/doc/90000/90136/90514)

#### vue
```
isWeChatOpen() {
    const local = location.href.split('#')[0];
    const that = this;
    makeTicket({url: local}).then((res) => {
        if (res.success) {
            wx.config({
                beta: true,// 必须这么写，否则wx.invoke调用形式的jsapi会有问题
                debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
                appId: res.data.appId, // 必填，企业微信的corpID
                timestamp: res.data.timestamp, // 必填，生成签名的时间戳
                nonceStr: res.data.nonceStr, // 必填，生成签名的随机串
                signature: res.data.signature,// 必填，签名，见 附录-JS-SDK使用权限签名算法
                jsApiList: ['openDefaultBrowser', 'closeWindow'] // 必填，需要使用的JS接口列表，凡是要调用的接口都需要传进来
            });
            wx.ready(function () {
                const url = that.workwx.author_url + '?appid=' + that.workwx.appid +
                    '&redirect_uri=' + encodeURIComponent(that.workwx.redirect_uri) +
                    '&response_type=code&scope=snsapi_privateinfo&agentid=' +
                    that.workwx.agentid + '&state=STATE#wechat_redirect';
                wx.invoke('openDefaultBrowser', {'url': url,}, function (res) {
                    if (res.err_msg == 'openDefaultBrowser:ok') {
                        // wx.closeWindow()
                        // window.close()
                    }
                })
            })
        } else {
            this.$message.warning(res.msg)
        }
    })
}
```

#### java
```java
public static Map<String, Object> getSignature(String url) {
    // 企业id
    String corpId = "ww41b472dada54eaa4";
    // 应用Secret
    String secret = "ns8uPdMr5U0FYy9CinJusT0xJ3-8tiNV9WPUPUP1B04";
    // 获取access_token
    String getAccessTokenUrl = "https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=%s&corpsecret=%s";
    getAccessTokenUrl = String.format(getAccessTokenUrl, corpId, secret);
    Map<String, Object> accessTokenMap = HttpClientUtils.get(getAccessTokenUrl);
    String token = accessTokenMap.get("access_token").toString();
    // 获取企业的jsapi_ticket
    String getJsApiTicketUrl = "https://qyapi.weixin.qq.com/cgi-bin/get_jsapi_ticket?access_token=%s";
    getJsApiTicketUrl = String.format(getJsApiTicketUrl, token);
    Map<String, Object> ticketMap = HttpClientUtils.get(getJsApiTicketUrl);
    String ticket = ticketMap.get("ticket").toString();
    // 获取签名
    long timestamp = System.currentTimeMillis() / 1000;
    String nonceStr = UUID.randomUUID().toString();
    Map<String, Object> params = Maps.newTreeMap();
    params.put("jsapi_ticket", ticket);
    params.put("noncestr", nonceStr);
    params.put("timestamp", timestamp);
    params.put("url", url);
    String text = params.entrySet().stream()
            .map(e -> e.getKey() + "=" + e.getValue())
            .collect(Collectors.joining("&"));
    String signature = DigestUtils.sha1Hex(text);
    return ImmutableMap.of("appId", corpId,
            "timestamp", timestamp, "nonceStr", nonceStr, "signature", signature);
}
```