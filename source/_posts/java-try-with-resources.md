---
title: try-with-resources语句
date: 2021-01-02 10:10:18
tags:
    - java
categories:
    - java
---

### try-with-resources

#### 以前
```
try {
    // ...
} catch (Exception e) {
    // ...
} finally {
    // xxx.close();
}
```

#### Java7新的try-with-resources语句，自动资源释放
```
try (OutputStream os = new FileOutputStream("...");) {
    // ...
} catch (Exception e) {
    // ...
}
```

* 所有实现Closeable的类声明都可以写在里面，
最常见的是用于流操作、socket操作、新版的httpclient也可以；
需要注意的是，try()的括号中可以写多行声明，
每个声明的变量类型都必须是Closeable的子类，用分号（;）隔开。
从而可以简化许多的代码，不用再在finally中手动的关闭资源了。
