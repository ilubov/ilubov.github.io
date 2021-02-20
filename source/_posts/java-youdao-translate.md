---
title: 有道翻译api
date: 2021-02-20 13:15:25
tags:
    - java
categories:
    - java
---

### 有道翻译api

#### 代码
```java
package com.ilubov.util.youdao;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Maps;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.util.Map;

public class YoudaoUtil {

    public static final String TRANSLATE_URL = "http://fanyi.youdao.com/translate?&doctype=json&type=AUTO&i=%s";

    public static void main(String[] args) {
        translate("一");
    }

    public static void translate(String word) {
        Map<String, Object> result = get(String.format(TRANSLATE_URL, word));
        System.out.println(JSON.toJSONString(result, true));
    }

    public static Map<String, Object> get(String url) {
        HttpGet req = new HttpGet(url);
        try (CloseableHttpClient client = HttpClients.createDefault();
             CloseableHttpResponse resp = client.execute(req)) {
            String text = EntityUtils.toString(resp.getEntity(), "UTF-8");
            JSONObject json = JSON.parseObject(text);
            return json.getInnerMap();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return Maps.newHashMap();
    }
}

```

#### 结果

```
{
	"errorCode":0,
	"translateResult":[
		[
			{
				"tgt":"one",
				"src":"一"
			}
		]
	],
	"type":"ZH_CN2EN",
	"elapsedTime":0
}
```
