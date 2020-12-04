---
title: 支付宝支付
date: 2020-03-25 22:11:10
tags:
    - java
    - 支付
categories:
    - java
---
### 支付宝支付
#### 相关依赖
```xml
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
    <version>3.3.4.ALL</version>
</dependency>
```
#### 公共参数
```java
// 商户appid
public static String APP_ID = "";
// 私钥 pkcs8格式的
public static String RSA_PRIVATE_KEY = "";
// 服务器异步通知页面路径 需http://或者https://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问
public static String NOTIFY_URL = "http://ilubov.cn/xxx";
// 页面跳转同步通知页面路径 需http://或者https://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问 商户可以自定义同步跳转地址
public static String RETURN_URL = "http://ilubov.cn/xxx.html";
// 请求网关地址
public static String URL = "https://openapi.alipay.com/gateway.do";
// 编码
public static String CHARSET = "UTF-8";
// 返回格式
public static String FORMAT = "json";
// 支付宝公钥
public static String ALI_PUBLIC_KEY = "";
// 应用公钥
public static String APPLICATION_PUBLIC_KEY = "";
// RSA2
public static String SIGN_TYPE = "RSA2";
```
#### 支付宝支付（H5）
```java
public static String aliPayH5() {
    // 封装请求参数
    String outTradeNo = "商户订单号";
    String totalAmount = "付款金额，必填，最多两位小数";
    String subject = "商品名称，必填";
    String body = "商品描述，可空";
    String hbNum = "花呗分期期数";
    // 同步地址参数
    String userId = "", token = "", orderId = "";
    // 返回form表单
    String form = "";
    try {
        // SDK 公共请求类，包含公共请求参数，以及封装了签名与验签，开发者无需关注签名与验签
        // 调用RSA签名方式
        AlipayClient client = new DefaultAlipayClient(AliPayDemo.URL, AliPayDemo.APP_ID,
                AliPayDemo.RSA_PRIVATE_KEY, AliPayDemo.FORMAT, AliPayDemo.CHARSET,
                AliPayDemo.ALI_PUBLIC_KEY, AliPayDemo.SIGN_TYPE);
        AlipayTradeWapPayRequest request = new AlipayTradeWapPayRequest();
        // 封装请求支付信息
        AlipayTradeWapPayModel model = new AlipayTradeWapPayModel();
        model.setOutTradeNo(outTradeNo);
        model.setTotalAmount(totalAmount);
        model.setSubject(subject);
        model.setBody(body);
        if (StringUtils.isNotBlank(hbNum)) {
            ExtendParams extendParams = new ExtendParams();
            extendParams.setHbFqNum(hbNum);
            extendParams.setHbFqSellerPercent("0");
            model.setExtendParams(extendParams);
        }
        // 超时时间 可空 2分钟
        model.setTimeoutExpress("2m");
        // 销售产品码 必填
        model.setProductCode("QUICK_WAP_WAY");
        model.setGoodsType("1");
        request.setBizModel(model);
        // 设置异步通知地址
        request.setNotifyUrl(AliPayDemo.NOTIFY_URL);
        // 设置同步地址
        request.setReturnUrl(AliPayDemo.RETURN_URL + "?user_id=" + userId + "&token=" + token + "&order_id=" + orderId);
        // 调用SDK生成表单
        AlipayTradeWapPayResponse response = client.pageExecute(request);
        form = response.getBody();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return form;
}
```
#### 支付宝支付（APP）
```java
public static String aliPayApp() {
    // 封装请求参数
    String outTradeNo = "商户订单号";
    String totalAmount = "付款金额，必填，最多两位小数";
    String subject = "商品名称，必填";
    String body = "商品描述，可空";
    // 返回app端请求参数
    String form = "";
    try {
        AlipayClient alipayClient = new DefaultAlipayClient(AliPayDemo.URL,
                AliPayDemo.APP_ID, AliPayDemo.RSA_PRIVATE_KEY, AliPayDemo.FORMAT, AliPayDemo.CHARSET,
                AliPayDemo.APPLICATION_PUBLIC_KEY, AliPayDemo.SIGN_TYPE);
        AlipayTradeAppPayRequest request = new AlipayTradeAppPayRequest();
        AlipayTradeAppPayModel model = new AlipayTradeAppPayModel();
        model.setOutTradeNo(outTradeNo);
        model.setTotalAmount(totalAmount);
        model.setSubject(subject);
        model.setBody(body);
        // 超时时间 可空 90分钟
        model.setTimeoutExpress("90m");
        model.setProductCode("QUICK_MSECURITY_PAY");
        request.setBizModel(model);
        request.setNotifyUrl(AliPayDemo.NOTIFY_URL);
        // 调用SDK
        AlipayTradeAppPayResponse response = alipayClient.sdkExecute(request);
        form = response.getBody();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return form;
}
```
#### 支付宝退款
```java
public static AlipayTradeRefundResponse aliPayRefund() {
    // 封装请求参数
    String outTradeNo = "商户订单号";
    String tradeNo = "支付宝交易号";
    String refundAmount = "退款金额，必填，最多两位小数";
    String refundReason = "退款原因";
    // 返回值：code: 10000 成功 fundChange: Y 成功, N并且msg = "Success" 重复申请
    AlipayTradeRefundResponse response = null;
    try {
        AlipayClient client = new DefaultAlipayClient(AliPayDemo.URL, AliPayDemo.APP_ID,
                AliPayDemo.RSA_PRIVATE_KEY, AliPayDemo.FORMAT, AliPayDemo.CHARSET,
                AliPayDemo.ALI_PUBLIC_KEY, AliPayDemo.SIGN_TYPE);
        AlipayTradeRefundRequest request = new AlipayTradeRefundRequest();
        AlipayTradeRefundModel model = new AlipayTradeRefundModel();
        // 商户订单号和支付宝交易号不能同时为空 trade_no、out_trade_no如果同时存在优先取trade_no
        model.setOutTradeNo(outTradeNo);
        model.setTradeNo(tradeNo);
        model.setRefundAmount(refundAmount);
        model.setRefundReason(refundReason);
        request.setBizModel(model);
        response = client.execute(request);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return response;
}
```