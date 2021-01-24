---
title: JetCache
date: 2021-01-24 10:13:65
tags:
    - java
categories:
    - java
---

### 自定义支持break的forEach

#### 代码
```
import java.util.Spliterator;
import java.util.function.BiConsumer;
import java.util.stream.Stream;

public class ForEachUtils {

    public static class Breaker {

        private volatile boolean stop = false;

        public void stop() {
            stop = true;
        }

        boolean isStop() {
            return !stop;
        }
    }

    public static <T> void forEach(Stream<T> stream, BiConsumer<T, Breaker> consumer) {
        Spliterator<T> spliterator = stream.spliterator();
        boolean hadNext = true;
        Breaker breaker = new Breaker();
        while (hadNext && breaker.isStop()) {
            hadNext = spliterator.tryAdvance(element -> consumer.accept(element, breaker));
        }
    }
}
```

#### Test
```
import com.google.common.collect.Lists;

import java.util.List;

public class Test {

    public static void main(String[] args) {
        List<Integer> list = Lists.newArrayList(1, 2, 3, 4, 5);
        ForEachUtils.forEach(list.stream(), ((i, breaker) -> {
            if (i == 2) {
                return;
            }
            if (i == 4) {
                breaker.stop();
                return;
            }
            System.err.println(i);
        }));
    }
}
```

#### 结果
```
1
3
```
