---
title: EventBus、AsyncEventBus
date: 2020-08-10 01:13:44
tags:
    - java
    - guava
categories:
    - java
---
### EventBus、AsyncEventBus

#### 简介
EventBus是google的Guava库中的一个处理组件间通信的事件总线，它基于发布/订阅模式，实现了多组件之间通信的解耦合，事件产生方和事件消费方实现解耦分离，提升了通信的简洁性。
#### 使用场景
在工作中，经常会遇见使用异步的方式来发送事件，或者触发另外一个动作：经常用到的框架是MQ（分布式方式通知）。如果是同一个jvm里面通知的话，就可以使用EventBus。由于EventBus使用起来简单、便捷，因此，工作中会经常用到。
#### EventBus的三个关键点
##### 事件（Event)
事件是EventBus之间相互通信的基本单位，一个Event可以是任何类型。就是Object，只要你想将任意一个Bean作为事件，这个类不需要做任何改变，就可以作为事件Event。不过在项目中不会这么随便，一般会定义特定的事件类，类名以Event作为后缀，里面定义一些变量或者函数等。
```java
public class OperationLogEvent implements Serializable {
    private String username;
    private String description;
    private String method;
    private String params;
    private String logType;
    private String requestIp;
    private String browser;
    private Long executeTime;
    private String exceptionDetail;
}
```
##### 事件发布者（Publisher）
事件发布者，就是发送事件到EventBus事件总线的一方，事件发布者调用Post()方法，将事件发给EventBus。你可以在程序的任何地方，调用EventBus的post()方法，发送事件给EventBus，由EventBus发送给订阅者们。
```java
@Component
public final class EventUtils {

    @Autowired
    private AsyncEventBus asyncEventBus;

    private static EventUtils instance = null;

    public EventUtils() {
        instance = this;
    }

    public static void postEvent(Object event) {
        instance.asyncEventBus.post(event);
    }
}
```
##### 事件订阅者（Subscriber）
事件订阅者，就是接收事件的一方，这些订阅者需要在自己的方法上，添加@Subscribe注解声明自己为事件订阅者。不过只声明是不够的，还需要将自己所在的类，注册到EventBus中（可搭配自定义注解@AsyncEventSubscriber），EventBus才能扫描到这个订阅者。
```java
@AsyncEventSubscriber
@Component
public class OperationLogSubscriber {

    @Autowired
    private OperationLogService operationLogService;

    @Subscribe
    @AllowConcurrentEvents
    public void receiveOperationLogEvent(OperationLogEvent operationLogEvent) {
        OperationLog operationLog = BeanUtil.toBean(operationLogEvent, OperationLog.class);
        operationLogService.save(operationLog);
    }

}
```
##### 自定义注解将事件订阅者注册到EventBus中
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface AsyncEventSubscriber {
}
```
```java
@Component
public class EventBusBeanPostProcessor implements BeanPostProcessor, DisposableBean {

    @Autowired
    private AsyncEventBus asyncEventBus;

    List<Object> subscribers = Lists.newArrayList();

    public EventBusBeanPostProcessor() {}

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        AsyncEventSubscriber annotation = bean.getClass().getAnnotation(AsyncEventSubscriber.class);
        if (annotation != null) {
            this.asyncEventBus.register(bean);
            this.subscribers.add(bean);
        }
        return bean;
    }

    public void destroy() {
        this.subscribers.forEach((bean) -> this.asyncEventBus.unregister(bean));
    }
}
```
#### EventBus和AsyncEventBus使用区别

##### EventBus:同步事件总线
* 同步执行，事件发送方在发出事件之后，会等待所有的事件消费方执行完毕后，才会回来继续执行自己后面的代码。
* 事件发送方和事件消费方会在同一个线程中执行，消费方的执行线程取决于发送方。
* 同一个事件的多个订阅者，在接收到事件的顺序上面有不同。谁先注册到EventBus的，谁先执行，如果是在同一个类中的两个订阅者一起被注册到EventBus的情况，收到事件的顺序跟方法名有关。

##### AsyncEventBus:异步事件总线
* 异步执行，事件发送方异步发出事件，不会等待事件消费方是否收到，直接执行自己后面的代码。
* 在定义AsyncEventBus时，构造函数中会传入一个线程池。事件消费方收到异步事件时，消费方会从线程池中获取一个新的线程来执行自己的任务。
* 同一个事件的多个订阅者，它们的注册顺序跟接收到事件的顺序上没有任何联系，都会同时收到事件，并且都是在新的线程中，异步并发的执行自己的任务。

#### [可搭配aop记录操作日志](https://github.com/ilubov/operation-log)
