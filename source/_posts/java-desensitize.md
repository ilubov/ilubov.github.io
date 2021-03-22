---
title: redis的String数据结构底层实现
date: 2021-03-22 20:41:50
tags:
    - java
categories:
    - java
---

### 脱敏处理

使用@ControllerAdvice & ResponseBodyAdvice & 自定义注解

#### 定义注解
* DesensitizeSupport: 用于方法或类
```
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

@Retention(value = java.lang.annotation.RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE, ElementType.METHOD})
@Documented
public @interface DesensitizeSupport {
}
```
* Desensitize: 用于脱敏字段
```
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

@Retention(value = java.lang.annotation.RetentionPolicy.RUNTIME)
@Target(value = {ElementType.FIELD})
@Documented
public @interface Desensitize {

    Type value();

    enum Type {
        MOBILE,
        PASSWORD
    }
}
```

#### ResponseBodyAdvice
```
import cn.hutool.core.util.ReflectUtil;
import com.ilubov.util.ReturnData;
import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;

import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;

@ControllerAdvice
public class DesensitizeResponseBodyAdvice implements ResponseBodyAdvice<Object> {

    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
        Method method = methodParameter.getMethod();
        assert method != null;
        // 判断方法或者类上是否存在DesensitizeSupport注解
        return method.getAnnotation(DesensitizeSupport.class) != null
                || method.getDeclaringClass().getAnnotation(DesensitizeSupport.class) != null;
    }

    @Override
    public Object beforeBodyWrite(Object o, MethodParameter methodParameter, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        if (o == null) {
            return null;
        }
        ReturnData<?> returnData = (ReturnData<?>) o;
        Object data = returnData.getData();
        if (data == null) {
            return o;
        }
        Collection<?> collection;
        if (data instanceof Collection) {
            collection = (Collection<?>) data;
        } else {
            collection = Collections.singletonList(data);
        }
        // 进行脱敏
        collection.forEach(obj -> {
            Class<?> clazz = obj.getClass();
            Arrays.stream(clazz.getDeclaredFields())
                    .filter(field -> field.getAnnotation(Desensitize.class) != null && String.class == field.getType())
                    .forEach(field -> {
                        Object oldValue = ReflectUtil.getFieldValue(obj, field.getName());
                        if (oldValue == null) {
                            return;
                        }
                        // 脱敏类型
                        Desensitize.Type type = field.getAnnotation(Desensitize.class).value();
                        String newValue = this.desensitize(oldValue.toString(), type);
                        ReflectUtil.setFieldValue(obj, field, newValue);
                    });
        });
        return o;
    }

    private String desensitize(String value, Desensitize.Type type) {
        if (type == null) {
            return value;
        }
        // 脱敏处理
        switch (type) {
            case MOBILE:
                return value.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
            case PASSWORD:
                return "********";
            default:
                return value;
        }
    }
}
```

#### 实体类注解使用
```
import com.ilubov.annotations.Desensitize;
import lombok.Data;

@Data
public class TestVo {

    private Long id;

    private String name;

    @Desensitize(Desensitize.Type.MOBILE)
    private String mobile;

    @Desensitize(Desensitize.Type.PASSWORD)
    private String password;

    public TestVo(Long id, String name, String mobile, String password) {
        this.id = id;
        this.name = name;
        this.mobile = mobile;
        this.password = password;
    }
}
```

#### Controller注解使用
```
import com.ilubov.vo.TestVo;
import com.ilubov.util.ReturnData;
import com.ilubov.annotations.DesensitizeSupport;
import com.google.common.collect.ImmutableList;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/test")
//@DesensitizeSupport //也可以直接用在类上
public class TestController {

    @GetMapping("/t1")
    @DesensitizeSupport
    public ReturnData t1(Long id) {
        List<TestVo> list = ImmutableList.of(
                new TestVo(1L, "张三", "18812345678", "123456"),
                new TestVo(2L, "李四", "13377881200", "123456"),
                new TestVo(3L, "王二麻", "18644110099", "123456"));
        return ReturnData.ok("success", list);
    }

    @GetMapping("/t2")
    @DesensitizeSupport
    public ReturnData t2(Long id) {
        return ReturnData.ok("success", new TestVo(1L, "张三", "18812345678", "123456"));
    }
}
```
