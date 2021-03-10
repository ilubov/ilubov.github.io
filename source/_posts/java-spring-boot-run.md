---
title: SpringBoot启动流程
date: 2021-03-06 15:33:29
tags:
    - java
categories:
    - java
---

### SpringBoot启动流程

#### @SpringBootApplication

```
// 注解的适用范围，其中TYPE用于描述类、接口（包括包注解类型）或enum声明
@Target(ElementType.TYPE)
// 注解的生命周期，保留到class文件中（三个生命周期）
@Retention(RetentionPolicy.RUNTIME)
// 表明这个注解应该被javadoc记录
@Documented
// 子类可以继承该注解
@Inherited
// 继承了Configuration，表示当前是注解类
@SpringBootConfiguration
// 开启springboot的注解功能，springboot的四大神器之一，其借助@import的帮助
@EnableAutoConfiguration
// 扫描路径设置（具体使用待确认）
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {...}
```

#### SpringApplication.run
```
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 *
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 *
 * 运行spring应用，并刷新一个新的 ApplicationContext（Spring的上下文）
 * ConfigurableApplicationContext 是 ApplicationContext 接口的子接口。在 ApplicationContext
 * 基础上增加了配置上下文的工具。 ConfigurableApplicationContext是容器的高级接口
 */
public ConfigurableApplicationContext run(String... args) {
    // 记录程序运行时间
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // ConfigurableApplicationContext Spring 的上下文
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    // 从META-INF/spring.factories中获取监听器
    // 1、获取并启动监听器
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 2、构造应用上下文环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        // 处理需要忽略的Bean
        configureIgnoreBeanInfo(environment);
        // 打印banner
        Banner printedBanner = printBanner(environment);
        // 3、初始化应用上下文
        context = createApplicationContext();
        // 实例化SpringBootExceptionReporter.class，用来支持报告关于启动的错误
        exceptionReporters = getSpringFactoriesInstances(
                SpringBootExceptionReporter.class,
                new Class[]{ConfigurableApplicationContext.class}, context);
        // 4、刷新应用上下文前的准备阶段
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        // 5、刷新应用上下文
        refreshContext(context);
        // 6、刷新应用上下文后的扩展接口
        afterRefresh(context, applicationArguments);
        // 时间记录停止
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                    .logStarted(getApplicationLog(), stopWatch);
        }
        // 发布容器启动完成事件
        listeners.started(context);
        // 执行所有runner运行器
        callRunners(context, applicationArguments);
    } catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        // 发布应用上下文就绪事件
        listeners.running(context);
    } catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    // 返回上下文
    return context;
}
```
