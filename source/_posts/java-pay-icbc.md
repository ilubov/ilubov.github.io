---
title: 工行支付
date: 2020-03-28 20:31:56
tags:
    - java
    - 支付
categories:
    - java
---
### 工行支付
#### 相关依赖
```xml
<dependency>
    <groupId>icbc</groupId>
    <artifactId>icbc-api-sdk-cop</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>icbc</groupId>
    <artifactId>icbc-api-sdk-cop-io</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>icbc</groupId>
    <artifactId>icbc-ca</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>icbc</groupId>
    <artifactId>InfosecCrypto_Java1_02_JDK14</artifactId>
    <version>2.0.0</version>
</dependency>

<dependency>
    <groupId>icbc</groupId>
    <artifactId>proguard</artifactId>
    <version>2.0.0</version>
</dependency>
```
#### 公钥私钥
```java
@Bean("key")
public byte[] getKey() throws IOException {
    ClassPathResource classPathResource = new ClassPathResource("/META-INF/siff.key");
    InputStream inputStream = classPathResource.getInputStream();
    return FilesUtils.toByteArray(inputStream);
}

@Bean("crt")
public byte[] getCrt() throws IOException {
    ClassPathResource classPathResource = new ClassPathResource("/META-INF/siff.crt");
    InputStream inputStream = classPathResource.getInputStream();
    return FilesUtils.toByteArray(inputStream);
}
```
#### 公共参数
```java
// 私钥
@Resource(name = "key")
private byte[] key;
// 证书公钥
@Resource(name = "crt")
private byte[] cert;
// 商户ID
@Value("${icbc.appId}")
private String appId;
// 支付 https://gw.open.icbc.com.cn/ui/b2c/pc/pay/transfer/V1
@Value("${icbc.payServiceUrl}")
private String payServiceUrl;
// 查询 https://gw.open.icbc.com.cn/api/b2c/orderqry/search/V1
@Value("${icbc.searchServiceUrl}")
private String searchServiceUrl;
// 回调接口
@Value("${icbc.notifyUrl}")
private String notifyUrl;
// 通知页面地址
@Value("${icbc.returnUrl}")
private String returnUrl;
// 服务域名 ilubov.com
@Value("${icbc.merReference}")
private String merReference;
// 网关公钥
@Value("${icbc.publicKey}")
private String publicKey;
// 密码
@Value("${icbc.password}")
private String password;
// 商户代码
@Value("${icbc.merId}")
private String merId;
// 商户账户
@Value("${icbc.merAcct}")
private String merAcct;
// 商户账户
@Value("${icbc.o2oMerId}")
private String o2oMerId;
```
#### 支付
```java
public void icbcPay(HttpServletResponse response) {
    // 证书公钥
    byte[] encCert = ReturnValue.base64enc(cert);
    String CertBase64 = new String(encCert);
    String reCert = FilesUtils.replace(CertBase64);
    // 私钥
    byte[] encKey = ReturnValue.base64enc(key);
    String keyBase64 = new String(encKey);
    String reKey = FilesUtils.replace(keyBase64);
    UiIcbcClient client = new UiIcbcClient(appId, reKey, IcbcConstants.CHARSET_UTF8, reCert, password);
    B2cPcPayTransferRequestV1 requestApi = new B2cPcPayTransferRequestV1();
    B2cPcPayTransferRequestV1.B2cPayTransferRequestV1Biz bizContent = new B2cPcPayTransferRequestV1.B2cPayTransferRequestV1Biz();
    // 订单对象
    B2cPcPayTransferRequestV1.B2cPcPayTransferRequestV1OrderInfo orderInfo = new B2cPcPayTransferRequestV1.B2cPcPayTransferRequestV1OrderInfo();
    // 扩展信息
    B2cPcPayTransferRequestV1.B2cPcPayTransferRequestV1Custom custom = new B2cPcPayTransferRequestV1.B2cPcPayTransferRequestV1Custom();
    // 商品通讯区d
    B2cPcPayTransferRequestV1.B2cPcPayTransferRequestV1Message message = new B2cPcPayTransferRequestV1.B2cPcPayTransferRequestV1Message();
    bizContent.setCustom(custom);
    bizContent.setOrderInfo(orderInfo);
    bizContent.setMessage(message);
    orderInfo.setMerID(merId);
    orderInfo.setMerAcct(merAcct);
    // 订单日期
    LocalDateTime time = LocalDateTime.now();
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyyMMddHHmmss");
    orderInfo.setOrderDate(dtf.format(time));
    // 付款单号
    orderInfo.setOrderId("No1234");
    // 付款金额 单位分
    orderInfo.setAmount("100");
    // 分期付款期数
    orderInfo.setInstallmentTimes("1");
    // 币种
    orderInfo.setCurType("001");
    // 联名校验标志，手机银行订单必输0，不校验
    custom.setVerifyJoinFlag("0");
    // 语言版本，默认为中文版，目前只支持中 文版 取值："zh_CN"
    custom.setLanguage("zh_CN");
    // 通知类型，在交易转账处 理完成后把交易结果通知 商户的处理模式。取 值"HS"：
    // 在交易完成后实时将通知信息以HTTP协 议POST方式，主动发送给商户，发送地址为商户端 随订单数据提交的接收工行支付结果的URL；
    // 取 值"AG"：在交易完成后 不通知商户。商户需要使用浏览器登录工行的B2C商户服务网站，或者使用 工行提供的客户端程序API 主动获取通知信息。
    // 取 值"LS"：在交易完成后 实时将通知信息以HTTP协 议POST方式，主动发送给商户，发送地址为商户端
    // 随订单数据提交的接受工行支付结果的URL，即表单中的notify_url字段，商户响应银行通知时返回取 货链接给工行，
    // 如工行未 收到商户响应则重复发送 通知消息，发送次数由工 行参数配置
    message.setNotifyType("HS");
    message.setOrderFlagZtb("0");
    // 结果发送类型，取 值"0"：无论支付成功或 者失败，银行都向商户发送交易通知信息；
    // 取 值"1"，银行只向商户发 送交易成功的通知信息。
    // 只有通知方式为HS时此值 有效，如果使用AG方式，
    // 可不上送此项，但签名数 据中必须包含此项，取值 可为空。
    // 注：如商户同时 支持线上支付和线下扫码 支付则该字段取值方式只 支持：1-银行只向商户发 送交易成功的通知信息
    message.setResultType("0");
    // 移动端送空，pc端支付必 输。默认"2"。取值范围 为0、1、2，
    // 其中0表示仅 允许使用借记卡支付，1表 示仅允许使用信用卡支 付，2表示借记卡和信用卡 都能对订单进行支付
    message.setCreditType("2");
    // 如需支持线下支付必输。 商户需要开通银联支付或 线下支付时生成的特约商 户编号（特约商户12位， 特约部门15位）
    message.setO2oMerId("");
    // 如需支持线下支付必输。 商户在工行开户时，开通 线下支付档案时选择同步 开通e生活档案时生成的编 号，由工行告知商户
    message.setElifeMerId("");
    // 如需支持线下支付必输。 二维码有效期，单位： 秒，必须小于24小时
    message.setPayExpire("300");
    // 移动端送空，pc端支付选输，工行在支付页面显示该信息
    // 中文需要64编码
    message.setMerReference(merReference);
    // 通知页面地址
    message.setReturnUrl(returnUrl);
    // 商品名称
    message.setGoodsName(new BASE64Encoder().encode("payment".getBytes()));
    // 实物 1 虚拟 0
    message.setGoodsType("0");
    message.setMerHint("RFJQ");
    message.setRemark1("");
    message.setRemark2("");
    message.setMerCustomIp("");
    message.setMerCustomID("1");
    message.setMerCustomPhone("18888888888");
    message.setGoodsAddress(new BASE64Encoder().encode("none".getBytes()));
    message.setMerOrderRemark(new BASE64Encoder().encode("none".getBytes()));
    message.setAuto_refer_sec("5");
    message.setBackup1("");
    message.setBackup2("");
    message.setBackup3("");
    message.setBackup4("");
    // 支付接口
    requestApi.setServiceUrl(payServiceUrl);
    // 通知地址，工行服务器主动通知商户 服务器里指定的页 面http/https路径
    requestApi.setNotifyUrl(notifyUrl);
    // 接口名
    requestApi.setInterfaceName("ICBC_PERBANK_B2C");
    // 接口版本号
    requestApi.setInterfaceVersion("1.0.0.11");
    requestApi.setBizContent(bizContent);
    System.out.println("工行支付请求参数为：" + JSONObject.toJSONString(requestApi));
    PrintWriter out = null;
    try {
        response.setContentType("text/html;charset=utf-8");
        out = response.getWriter();
        // 工行返回
        String form = client.buildPostForm(requestApi);
        out.write("<html>");
        out.write("<head>");
        out.write("<meta http-equiv=\"Content-Type\" content=\"text/html; charset=" + "utf-8" + "\">");
        out.write("</head>");
        out.write("<body>");
        out.write(form);
        out.write("</body>");
        out.write("</html>");
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        assert out != null;
        out.flush();
        out.close();
    }
}
```
#### 支付结果查询
```java
public String getPayResult(String orderNo) throws IcbcApiException {
    byte[] EncCert = ReturnValue.base64enc(cert);
    String CertBase64 = new String(EncCert);
    String reCert = FilesUtils.replace(CertBase64);
    byte[] EncKey = ReturnValue.base64enc(key);
    String keyBase64 = new String(EncKey);
    String reKey = FilesUtils.replace(keyBase64);
    DefaultIcbcClient client = new DefaultIcbcClient(appId, reKey, publicKey, reCert, password);
    // 新建服务请求类实例
    B2cOrderqrySearchRequestV1 requestApi = new B2cOrderqrySearchRequestV1();
    // 新建服务请求类的业务参数类，该类为内部类
    B2cOrderqrySearchRequestV1.B2cOrderqrySearchRequestV1Biz bizContent
            = new B2cOrderqrySearchRequestV1.B2cOrderqrySearchRequestV1Biz();
    bizContent.setConsumerId("行外商户_查询支付订单");
    // 查询线上支付订单则必输
    bizContent.setMerid(merId);
    // 查询线下支付订单则必输
    bizContent.setO2o_merid(o2oMerId);
    // 查询的付款单号
    bizContent.setOrderid(orderNo);
    requestApi.setServiceUrl(searchServiceUrl);
    requestApi.setBizContent(bizContent);
    B2cOrderqrySearchResponseV1 responses = client.execute(requestApi, String.valueOf(System.currentTimeMillis()));
    JSONObject jsonObject = JSONObject.parseObject(JSONObject.toJSONString(responses));
    String msg;
    // 支付金额
    int sum = Integer.parseInt(jsonObject.get("order_sum") == null ? "0" : jsonObject.get("order_sum").toString());
    // status 2-支付成功 3-支付失败 4-未支付
    String status = jsonObject.getString("status");
    // 支付成功
    if ("2".equals(status) && sum > 0) {
        BigDecimal money = new BigDecimal(sum).divide(new BigDecimal(100));
        msg = "支付成功，支付金额：" + money + "元";
    } else {
        msg = jsonObject.get("returnMsg").toString();
    }
    return msg;
}
```