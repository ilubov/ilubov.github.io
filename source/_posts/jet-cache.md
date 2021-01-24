---
title: JetCache
date: 2021-01-22 22:13:25
tags:
    - java
categories:
    - java
---

### JetCache
JetCache是一个基于Java的缓存系统封装，提供统一的API和注解来简化缓存的使用。 
JetCache提供了比SpringCache更加强大的注解，可以原生的支持TTL、两级缓存、分布式自动刷新，还提供了Cache接口用于手工缓存操作。 
当前有四个实现，RedisCache、TairCache（此部分未在github开源）、CaffeineCache(in memory)和一个简易的LinkedHashMapCache(in memory)，要添加新的实现也是非常简单的。

#### 全部特性

* 通过统一的API访问Cache系统
* 通过注解实现声明式的方法缓存，支持TTL和两级缓存
* 通过注解创建并配置Cache实例
* 针对所有Cache实例和方法缓存的自动统计
* Key的生成策略和Value的序列化策略是可以配置的
* 分布式缓存自动刷新，分布式锁 (2.2+)
* 异步Cache API (2.2+，使用Redis的lettuce客户端时)
* Spring Boot支持

#### 使用

* Maven
```
<dependency>
    <groupId>com.alicp.jetcache</groupId>
    <artifactId>jetcache-starter-redis</artifactId>
    <version>2.5.11</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

* yml
```
jetcache:
  statIntervalMinutes: 15
  areaInCacheName: false
  # 本地缓存
  local:
    default:
      type: linkedhashmap
      keyConvertor: fastjson
      limit: 100
      expireAfterWriteInMillis: 3600000
      expireAfterAccessInMillis: 3600000
  # 远程缓存
  remote:
    default:
      type: redis
      keyConvertor: fastjson
      valueEncoder: java
      valueDecoder: java
      poolConfig:
        minIdle: 5
        maxIdle: 20
        maxTotal: 50
      host: ${spring.redis.host}
      port: ${spring.redis.port}
      password: ${spring.redis.password}
```

* 启动类开启缓存
```
// 用于激活@Cached注解的使用
@EnableMethodCache(basePackages = "com.ilubov.cache")
// 用于激活@CreateCache注解的使用
@EnableCreateCacheAnnotation
@SpringBootApplication
public class IlubovApplication {
    public static void main(String[] args) {
        SpringApplication.run(IlubovApplication.class);
    }
}
```

* 自定义缓存
```
import cn.hutool.core.collection.CollUtil;
import com.alicp.jetcache.Cache;
import com.alicp.jetcache.anno.CreateCache;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.ilubov.entity.SysCode;
import com.ilubov.service.SysCodeService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.util.List;

/**
 * 编码缓存
 *
 * @author ilubov
 * @date 2021/01/18
 */
@Slf4j
@Component
public class CodeCacheUtil {

    private static final String CODE_NAME_PREFIX = "CODE_NAME_";

    private static CodeCacheUtils instance = null;

    public CodeCacheUtils() {
        instance = this;
    }

    @Autowired
    private SysCodeService sysCodeService;

    @CreateCache(name = CODE_NAME_PREFIX)
    private Cache<String, String> codeNameCache;

    /**
     * 初始化
     */
    @PostConstruct
    public void init() {
        log.info("===> code cache init");
        List<SysCode> sysCodes = sysCodeService.list();
        if (CollUtil.isNotEmpty(sysCodes)) {
            sysCodes.forEach(CodeCacheUtils::set);
        }
    }

    /**
     * 获取编码名称
     *
     * @param code
     * @return
     */
    public static String get(String code) {
        return instance.codeNameCache.computeIfAbsent(code, (o) -> {
            SysCode sysCode = instance.sysCodeService.getOne(new QueryWrapper<SysCode>()
                    .eq("code", code).last("limit 1"));
            if (sysCode != null) {
                return sysCode.getCodeName();
            }
            return code;
        });
    }

    /**
     * 设置编码名称
     *
     * @param sysCode
     */
    public static void set(SysCode sysCode) {
        instance.codeNameCache.put(sysCode.getCode(), sysCode.getCodeName());
    }
}
```

* 方法缓存
```java
public interface TestService {
    @Cached(name = "test_cache_", expire = 3600)
    TestVO getById(Long id);
}
```
