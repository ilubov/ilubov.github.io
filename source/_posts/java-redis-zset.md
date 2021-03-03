---
title: Redis zSet使用
date: 2021-03-01 22:04:02
tags:
    - java
    - redis
categories:
    - java
---

### Redis zSet使用

zset会按score进行排序，如果score代表想要执行时间的时间戳。在某个时间将它插入zset集合中，它变会按照时间戳大小进行排序，也就是对执行时间前后进行排序。

#### 存储用户浏览记录

##### util代码
```
import cn.hutool.core.collection.CollUtil;
import com.google.common.collect.Lists;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ZSetOperations;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

@Component
public class HistoryUtil {

    private static final long SAVE_SIZE = 5;

    private static final String HISTORY_PREFIX = "HISTORY_USER_ID_";

    private static HistoryUtil instance = null;

    public HistoryUtil() {
        instance = this;
    }

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 添加浏览历史
     *
     * @param id
     * @return
     */
    public static void set(Long id) {
        if (id == null) {
            return;
        }
        // 当前登录人id
        long userId = 1L;
        String key = HISTORY_PREFIX + userId;
        ZSetOperations<String, String> zSet = instance.redisTemplate.opsForZSet();
        zSet.add(key, id.toString(), System.currentTimeMillis());
        zSet.removeRange(key, 0, -1 - SAVE_SIZE);
    }

    /**
     * 获取浏览历史
     *
     * @return
     */
    public static List<Long> get() {
        // 当前登录人id
        long userId = 1L;
        String key = HISTORY_PREFIX + userId;
        ZSetOperations<String, String> zSet = instance.redisTemplate.opsForZSet();
        Set<String> carIdSet = zSet.reverseRange(key, 0, SAVE_SIZE - 1);
        if (carIdSet != null) {
            return carIdSet.stream().map(Long::parseLong).collect(Collectors.toList());
        }
        return Lists.newArrayList();
    }

    /**
     * 清空浏览历史
     */
    public static void remove() {
        // 当前登录人id
        long userId = 1L;
        String key = HISTORY_PREFIX + userId;
        ZSetOperations<String, String> zSet = instance.redisTemplate.opsForZSet();
        Set<String> idSet = zSet.reverseRange(key, 0, SAVE_SIZE - 1);
        if (CollUtil.isEmpty(idSet)) {
            return;
        }
        zSet.remove(key, idSet.toArray(new String[0]));
    }
}
```

##### 测试
```
import com.alibaba.fastjson.JSON;
import com.ilubov.util.HistoryUtil;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.Serializable;
import java.util.List;

@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class HistoryTest implements Serializable {

    @Test
    public void test() {
        for (long i = 0; i < 10; i++) {
            HistoryUtil.set(i);
        }
        HistoryUtil.set(1L);
        List<Long> ids = HistoryUtil.get();
        log.info(JSON.toJSONString(ids));
    }
}
```

##### 结果
```
[1,9,8,7,6]
```

#### 多维度排序（根据时间最大和size最多）

##### vo代码
```
import lombok.Data;

import java.util.Date;

@Data
public class TestVo {

    private Long id;

    private Long size;

    private Date date;

    public TestVo(Long id, Long size, Date date) {
        this.id = id;
        this.size = size;
        this.date = date;
    }
}
```
##### util代码
```
import cn.hutool.core.collection.CollUtil;
import com.alibaba.fastjson.JSON;
import com.cloudyoung.competing.entity.TestVo;
import com.google.common.collect.Lists;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ZSetOperations;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Set;

@Component
public class TopUtil {

    private static final long SAVE_SIZE = 5;

    private static final String TOP_PREFIX = "TOP_TEST";

    private static TopUtil instance = null;

    public TopUtil() {
        instance = this;
    }

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 添加top
     *
     * @param vo
     * @return
     */
    public static void set(TestVo vo) {
        if (vo == null) {
            return;
        }
        ZSetOperations<String, String> zSet = instance.redisTemplate.opsForZSet();
        // 加权分数计算（倒叙可以用一个大数去减）
        double score = vo.getDate().getTime() + vo.getSize();
        zSet.add(TOP_PREFIX, JSON.toJSONString(vo), score);
        zSet.removeRange(TOP_PREFIX, 0, -1 - SAVE_SIZE);
    }

    /**
     * 获取top
     *
     * @return
     */
    public static List<String> get() {
        ZSetOperations<String, String> zSet = instance.redisTemplate.opsForZSet();
        Set<String> sets = zSet.reverseRange(TOP_PREFIX, 0, SAVE_SIZE - 1);
        if (sets != null) {
            return Lists.newArrayList(sets);
        }
        return Lists.newArrayList();
    }

    /**
     * 清空top
     */
    public static void remove() {
        ZSetOperations<String, String> zSet = instance.redisTemplate.opsForZSet();
        Set<String> sets = zSet.reverseRange(TOP_PREFIX, 0, SAVE_SIZE - 1);
        if (CollUtil.isEmpty(sets)) {
            return;
        }
        zSet.remove(TOP_PREFIX, sets.toArray(new String[0]));
    }
}
```

##### 测试
```
import cn.hutool.core.date.DateUtil;
import com.alibaba.fastjson.JSON;
import com.ilubov.vo.TestVo;
import com.ilubov.util.TopUtil;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class TopTest {

    @Test
    public void test() {
        // 6 5 2 3 1 7 4
        TopUtil.set(new TestVo(1L, 20L, DateUtil.parse("2020-01-02", "yyyy-MM-dd")));
        TopUtil.set(new TestVo(2L, 20L, DateUtil.parse("2020-01-03", "yyyy-MM-dd")));
        TopUtil.set(new TestVo(3L, 30L, DateUtil.parse("2020-01-02", "yyyy-MM-dd")));
        TopUtil.set(new TestVo(4L, 10L, DateUtil.parse("2020-01-02", "yyyy-MM-dd")));
        TopUtil.set(new TestVo(5L, 10L, DateUtil.parse("2020-01-04", "yyyy-MM-dd")));
        TopUtil.set(new TestVo(6L, 20L, DateUtil.parse("2020-01-04", "yyyy-MM-dd")));
        TopUtil.set(new TestVo(7L, 10L, DateUtil.parse("2020-01-02", "yyyy-MM-dd")));
        System.out.println(JSON.toJSONString(TopUtil.get(), true));
    }
}
```

##### 结果
```
[
	"{\"date\":1578067200000,\"id\":6,\"size\":20}",
	"{\"date\":1578067200000,\"id\":5,\"size\":10}",
	"{\"date\":1577980800000,\"id\":2,\"size\":20}",
	"{\"date\":1577894400000,\"id\":3,\"size\":30}",
	"{\"date\":1577894400000,\"id\":1,\"size\":20}"
]
```
