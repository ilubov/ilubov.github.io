---
title: Redis存储用户浏览记录
date: 2020-03-01 22:04:02
tags:
    - java
    - redis
categories:
    - java
---

### Redis存储用户浏览记录

#### 代码
```
import cn.hutool.core.collection.CollUtil;
import com.ilubov.service.TestService;
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

    @Autowired
    private TestService testService;

    /**
     * 添加浏览历史
     *
     * @param id
     * @return
     */
    public static List<Long> set(Long id) {
        if (id == null) {
            return Lists.newArrayList();
        }
        // 当前登录人id
        long userId = 1L;
        String key = HISTORY_PREFIX + userId;
        List<Long> ids = get();
        CollUtil.reverse(ids);
        ids.add(id);
        ZSetOperations<String, String> zSet = instance.redisTemplate.opsForZSet();
        ids.forEach(index -> zSet.add(key, index.toString(), System.currentTimeMillis()));
        zSet.removeRange(key, 0, -1 - SAVE_SIZE);
        return ids;
    }

    /**
     * 获取浏览历史（id）
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
     * 获取浏览历史（详情）
     *
     * @param ids
     * @return
     */
    public static Object get(List<Long> ids) {
        if (CollUtil.isEmpty(ids)) {
            return Lists.newArrayList();
        }
        return instance.testService.listByIds(ids);
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

#### 测试
```
import com.alibaba.fastjson.JSON;
import com.ilubov.util.HistoryUtil;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class HistoryTest {

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

#### 结果
```
[1,9,8,7,6]
```
