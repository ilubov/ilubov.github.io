---
title: mybatis-plus二级缓存
date: 2021-01-15 18:22:18
tags:
    - java
    - mybatis
categories:
    - java
---

### mybatis-plus二级缓存

* #### 将cache-enabled修改为true
```
mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true    
    cache-enabled: true
  global-config:
    db-config:
      logic-delete-value: 1
      logic-not-delete-value: 0
```

* #### 添加cache标签
```
<mapper namespace="com.ilubov.mapper.UserMapper">
    <cache eviction="FIFO" flushInterval="10800000" size="512" readOnly="true"/>
</mapper>
```
* eviction（可用的收回策略 默认的是LRU）:
    * LRU – 最近最少使用的:移除最长时间不被使用的对象。
    * FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
    * SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
    * WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。
* flushInterval（刷新间隔）可以被设置为任意的正整数,而且它们代表一个合理的毫秒 形式的时间段。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。
* size（引用数目）可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的 可用内存资源数目。默认值是 1024。
* readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓 存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存 会返回缓存对象的拷贝（通过序列化） 。这会慢一些,但是安全,因此默认是 false。

* #### 部分select不开启缓存，可将useCache设置为false
```
<select id="test" resultType="com.ilubov.entity.Test" useCache="false">
    select id from test where id = #{id}
</select>
```

* #### 自定义缓存
```
<cache type="com.ilubov.cache.MybatisRedisCache"/>
```

```
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.cache.Cache;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.RedisServerCommands;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.Set;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

@Slf4j
@Component
public class MybatisRedisCache implements Cache {

    // 读写锁
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock(true);

    @Autowired
    private RedisTemplate redisTemplate;

    private static MybatisRedisCache instance = null;

    private String id;

    public MybatisRedisCache() {
        instance = this;
    }

    public MybatisRedisCache(final String id) {
        if (id == null) {
            throw new IllegalArgumentException("Cache instances require an ID");
        }
        this.id = id;
    }

    @Override
    public String getId() {
        return this.id;
    }

    @Override
    public void putObject(Object key, Object value) {
        if (value != null) {
            instance.redisTemplate.opsForValue().set(key.toString(), value);
        }
    }

    @Override
    public Object getObject(Object key) {
        try {
            if (key != null) {
                return instance.redisTemplate.opsForValue().get(key.toString());
            }
        } catch (Exception e) {
            e.printStackTrace();
            log.error("缓存出错");
        }
        return null;
    }

    @Override
    public Object removeObject(Object key) {
        if (key != null) {
            instance.redisTemplate.delete(key.toString());
        }
        return null;
    }

    @Override
    public void clear() {
        log.debug("清空缓存");
        Set<String> keys = instance.redisTemplate.keys("*:" + this.id + "*");
        if (!CollectionUtils.isEmpty(keys)) {
            instance.redisTemplate.delete(keys);
        }
    }

    @Override
    public int getSize() {
        Long size = (Long) instance.redisTemplate.execute(RedisServerCommands::dbSize);
        return size.intValue();
    }

    @Override
    public ReadWriteLock getReadWriteLock() {
        return this.readWriteLock;
    }
}
```

* #### 在mapper上加上注解@CacheNamespace
```
@CacheNamespace(implementation = MybatisRedisCache.class, eviction = MybatisRedisCache.class)
public interface TestMapper extends BaseMapper<Test> {
}
```
