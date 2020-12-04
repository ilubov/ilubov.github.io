---
title: Transactional
date: 2020-02-20 23:33:10
tags:
    - java
    - spring
categories:
    - java
---
### @Transactional
#### 实现
Transactional是spring中定义的事务注解，在方法或类上加该注解开启事务。主要是通过反射获取bean的注解信息，利用AOP对编程式事务进行封装实现。
```java
@Component
@Aspect
// 基于AOP的环绕通知和异常通知实现Spring事务
public class AopTransaction {

    @Autowired
    private MyTransaction myTransaction;
    
    // 环绕通知  在方法之前和之后处理事情
    @Around("execution(* com.wulei.service.UserService.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        // 开启事务    调用方法之前执行 
        TransactionStatus transactionStatus = myTransaction.begin();
        proceedingJoinPoint.proceed();// 代理调用方法 注意点： 如果调用方法抛出溢出不会执行后面代码
        // 提交事务  调用方法之后执行
        myTransaction.commit(transactionStatus);
    }
    // 异常通知
    @AfterThrowing("execution(* com.wulei.service.UserService.add(..))")
    public void afterThrowing() {
        System.out.println("回滚当前事务");
        // 获取当前事务 直接回滚
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```
#### 特性
1、service类标签(一般不建议在接口上)上添加@Transactional，可以将整个类纳入spring事务管理，在每个业务方法执行时都会开启一个事务，不过这些事务采用相同的管理方式。
2、@Transactional 注解只能应用到 public 可见度的方法上。 如果应用在protected、private或者 package可见度的方法上，也不会报错，不过事务设置不会起作用。
3、默认情况下，Spring会对unchecked异常进行事务回滚；如果是checked异常则不回滚。 
辣么什么是checked异常，什么是unchecked异常
java里面将派生于Error或者RuntimeException（比如空指针，1/0）的异常称为unchecked异常，其他继承自java.lang.Exception得异常统称为Checked Exception，如IOException、TimeoutException等
辣么再通俗一点：你写代码出现的空指针等异常，会被回滚，文件读写，网络出问题，spring就没法回滚了。然后我教大家怎么记这个，因为很多同学容易弄混，你写代码的时候有些IOException我们的编译器是能够检测到的，说以叫checked异常，你写代码的时候空指针等死检测不到的，所以叫unchecked异常。这样是不是好记一些啦
4、只读事务： 
`@Transactional(propagation=Propagation.NOT_SUPPORTED,readOnly=true) `
只读标志只在事务启动时应用，否则即使配置也会被忽略。 
启动事务会增加线程开销，数据库因共享读取而锁定(具体跟数据库类型和事务隔离级别有关)。通常情况下，仅是读取数据时，不必设置只读事务而增加额外的系统开销。

#### @Transactional注解参数详解
##### 参数一：事物传播行为 propagation
* REQUIRED：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 
* SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。 
* MANDATORY：支持当前事务，如果当前没有事务，就抛出异常。 
* REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。 
* NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 
* NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。 
* NESTED：支持当前事务，如果当前事务存在，则执行一个嵌套事务，如果当前没有事务，就新建一个事务。 

##### 参数二：事物超时设置 timeout
* timeout=30 默认是30秒

##### 参数三：事务隔离级别 isolation
* Isolation.READ_UNCOMMITTED：读取未提交数据(会出现脏读, 不可重复读) 基本不使用
* Isolation.READ_COMMITTED：读取已提交数据(会出现不可重复读和幻读)
* Isolation.REPEATABLE_READ：可重复读(会出现幻读)
* Isolation.SERIALIZABLE：串行化

MYSQL：默认为REPEATABLE_READ级别
SQLSERVER：默认为READ_COMMITTED

##### 参数四：readOnly 
* 属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。

##### 参数五：rollbackFor
* 该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：
    * 指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)
    * 指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, Exception.class})

##### 参数六：rollbackForClassName
* 该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如：
    * 指定单一异常类名称：@Transactional(rollbackForClassName="RuntimeException")
    * 指定多个异常类名称：@Transactional(rollbackForClassName={"RuntimeException","Exception"})

##### 参数七：noRollbackForClassName
* 该属性用于设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。例如：
    * 指定单一异常类名称：@Transactional(noRollbackForClassName="RuntimeException")
    * 指定多个异常类名称：@Transactional(noRollbackForClassName={"RuntimeException","Exception"})

##### 参数八：noRollbackFor
* 该属性用于设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。例如：
    * 指定单一异常类：@Transactional(noRollbackFor=RuntimeException.class)
    * 指定多个异常类：@Transactional(noRollbackFor={RuntimeException.class, Exception.class})

#### 注意点    
* 1、@Transactional 只能被应用到public方法上, 对于其它非public的方法,如果标记了@Transactional也不会报错,但方法没有事务功能.
* 2、用 spring 事务管理器,由spring来负责数据库的打开,提交,回滚.默认遇到运行期例外(throw new RuntimeException("注释");)会回滚，即遇到不受检查（unchecked）的例外时回滚；而遇到需要捕获的例外(throw new Exception("注释");)不会回滚,即遇到受检查的例外（就是非运行时抛出的异常，编译器会检查到的异常叫受检查例外或说受检查异常）时，需我们指定方式来让事务回滚要想所有异常都回滚,要加上 @Transactional( rollbackFor={Exception.class,其它异常}) .如果让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)
* 3、@Transactional 注解应该只被应用到 public 可见度的方法上。 如果你在 protected、private 或者 package-visible 的方法上使用 @Transactional 注解，它也不会报错， 但是这个被注解的方法将不会展示已配置的事务设置。
* 4、@Transactional 注解可以被应用于接口定义和接口方法、类定义和类的 public 方法上。然而，请注意仅仅 @Transactional 注解的出现不足于开启事务行为，它仅仅 是一种元数据，能够被可以识别 @Transactional 注解和上述的配置适当的具有事务行为的beans所使用。上面的例子中，其实正是 元素的出现 开启 了事务行为。
* 5、Spring团队的建议是你在具体的类（或类的方法）上使用 @Transactional 注解，而不要使用在类所要实现的任何接口上。你当然可以在接口上使用 @Transactional 注解，但是这将只能当你设置了基于接口的代理时它才生效。因为注解是不能继承的，这就意味着如果你正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装（将被确认为严重的）。因此，请接受Spring团队的建议并且在具体的类上使用 @Transactional 注解。

#### 解决Transactional注解不回滚
1、检查你方法是不是public的
2、你的异常类型是不是unchecked异常 
如果我想check异常也想回滚怎么办，注解上面写明异常类型即可
`@Transactional(rollbackFor=Exception.class) `
类似的还有norollbackFor，自定义不回滚的异常
3、数据库引擎要支持事务，如果是MySQL，注意表要使用支持事务的引擎，比如innodb，如果是myisam，事务是不起作用的
