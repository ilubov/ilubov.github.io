---
title: java用户线程和守护线程区别
date: 2021-03-18 21:52:29
tags:
    - java
categories:
    - java
---

### java用户线程和守护线程区别

#### 用户线程
* 平时用到的普通线程均是用户线程
* 用户线程是独立存在的，不会因为其他用户线程退出而退出

#### 守护线程
* 指在程序运行的时候在后台提供一种通用服务的线程，守护线程是为用户线程服务的，当有用户线程在运行，那么守护线程同样需要工作，当所有的用户线程都结束时，守护线程也就会停止
* 守护线程是依赖于用户线程，用户线程退出了，守护线程也就会退出，典型的守护线程如垃圾回收线程。
* 守护线程不应该去访问固有资源，如进行读写操作(文件，数据库)，因为守护线程是跟随用户线程的，当没有用户线程工作时，守护线程会立即结束
* 如果JVM中所有的线程都是守护线程，那么JVM就会退出，进而守护线程也会退出
* 如果JVM中还存在用户线程，那么JVM就会一直存活，不会退出

#### 测试主线程中启动一个守护线程
```
public class Test {
    
    public static void main(String[] args) {
        Thread daemon = new Thread(() -> {
            while (true) {
                try {
                    System.out.println("守护线程心跳");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        // 将该线程设置为守护线程
        daemon.setDaemon(true);
        daemon.start();
        try {
            Thread.sleep(3000);
            System.out.println("主线程" + Thread.currentThread().getName() + "退出");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

out：
守护线程心跳一次
守护线程心跳一次
守护线程心跳一次
主线程main退出！

conclusion：
发现主线程一旦退出，守护线程也就不再运行，直接退出了
```

#### 测试主线程中启动一个守护线程和一个用户线程
```
public class Test {
    
    public static void main(String[] args) {
        Thread daemon = new Thread(() -> {
            while (true) {
                try {
                    System.out.println("守护线程" + Thread.currentThread().getName() + "心跳");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        // 将该线程设置为守护线程
        daemon.setDaemon(true);
        daemon.start();
        Thread thread = new Thread(() -> {
            while (true) {
                try {
                    System.out.println("用户线程" + Thread.currentThread().getName() + "心跳");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread.start();
        try {
            Thread.sleep(3000);
            System.out.println("主线程" + Thread.currentThread().getName() + "退出！");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

out：
守护线程Thread-0心跳
用户线程Thread-1心跳
守护线程Thread-0心跳
用户线程Thread-1心跳
守护线程Thread-0心跳
用户线程Thread-1心跳
主线程main退出！
守护线程Thread-0心跳
用户线程Thread-1心跳
守护线程Thread-0心跳
用户线程Thread-1心跳

conclusion：
主线程退出后，守护线程依然在运行！由此得到只要任何非守护线程还在运行，守护线程就不会终止
```
