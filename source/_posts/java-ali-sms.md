---
title: 阿里云短信服务
date: 2021-04-15 21:41:50
tags:
    - java
categories:
    - java
---

### 阿里云短信服务

#### pom.xml
```
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.5.3</version>
</dependency>
```

#### application.yml
```
# 阿里云短信服务
ali-sms:
  # AccessKey
  access-key-id: AccessKey ID
  access-key-secret: AccessKey Secret
  # 国内消息-签名管理-签名名称
  sign-name: 签名名称
  # 国内消息-模板管理-模板CODE
  template-code: 模板CODE
```

#### Property类
```
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Data
@Configuration
@ConfigurationProperties("ali-sms")
public class AliSmsProperty {

    @ApiModelProperty("AccessKey ID")
    private String accessKeyId;

    @ApiModelProperty("AccessKey Secret")
    private String accessKeySecret;

    @ApiModelProperty("国内消息-签名管理-签名名称")
    private String signName;

    @ApiModelProperty("国内消息-模板管理-模板CODE")
    private String templateCode;
}
```

#### 发送短信工具类
```
import cn.hutool.core.util.RandomUtil;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.aliyuncs.CommonRequest;
import com.aliyuncs.CommonResponse;
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.http.MethodType;
import com.aliyuncs.profile.DefaultProfile;
import com.ilubov.exception.BusinessException;
import com.ilubov.property.AliSmsProperty;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class AliSmsUtil {

    @Autowired
    private AliSmsProperty property;

    private static AliSmsUtil instance = null;

    public static final long EXPIRE_TIME = 5 * 60L;

    public AliSmsUtil() {
        instance = this;
    }

    public static void sendSms(String telephone) {
        // 六位验证码
        String authCode = String.valueOf(RandomUtil.randomInt(100000, 999999));
        DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou",
                instance.property.getAccessKeyId(), instance.property.getAccessKeySecret());
        IAcsClient client = new DefaultAcsClient(profile);
        CommonRequest request = new CommonRequest();
        request.setSysMethod(MethodType.POST);
        request.setSysDomain("dysmsapi.aliyuncs.com");
        request.setSysVersion("2017-05-25");
        request.setSysAction("SendSms");
        request.putQueryParameter("RegionId", "cn-hangzhou");
        request.putQueryParameter("PhoneNumbers", telephone);
        request.putQueryParameter("SignName", instance.property.getSignName());
        request.putQueryParameter("TemplateCode", instance.property.getTemplateCode());
        // 模板参数 ${code}
        request.putQueryParameter("TemplateParam", "{\"code\":" + authCode + "}");
        JSONObject result = null;
        try {
            CommonResponse response = client.getCommonResponse(request);
            String data = response.getData();
            log.info("手机号: {}, 短信服务返回信息: {}", telephone, data);
            result = JSON.parseObject(data);
        } catch (Exception e) {
            log.error("短信服务系统错误", e);
            throw new BusinessException("短信服务系统错误");
        }
        if (!"OK".equals(result.getString("Code"))) {
            throw new BusinessException(result.getString("Message"));
        }
        log.info("手机号: {}, 验证码: {}", telephone, authCode);
        // 存缓存
        RedisUtil.set(telephone, authCode, EXPIRE_TIME);
    }
}
```
