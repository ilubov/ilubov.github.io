---
title: String/StringBuilder/StringBuffer区别
date: 2020-01-20 20:10:18
tags:
    - java
categories:
    - java
---


### String/StringBuilder/StringBuffer区别

#### String
```
/*
 * java.lang.String部分代码
 */
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```
首先String是一个final修饰不可继承的类，内部使用final修饰的char[]来保存数据，String是不可变的
```
String str = "123";
System.out.println((str + "456") == str);
// output : false
```
每次对String拼接都会创建一个新String对象

#### AbstractStringBuilder
AbstractStringBuilder是一个抽象类，是StringBuilder和StringBuffer的父类，二者大部分操作都是基于此类实现的
```
/*
 * java.lang.AbstractStringBuilder部分代码
 */
abstract class AbstractStringBuilder 
	implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;
}
```
AbstractStringBuilder是一个可变的字符串类，内部使用char[]来保存数据，count用来记录value已用长度。
每次append操作都会向value中添加数据，若数组容量不够则进行扩容，扩容后容量是原容量的2倍
append操作并不会创建新对象，而是在原有对象上进行修改，所以AbstractStringBuilder是可变的
```
StringBuilder bd = new StringBuilder("123");
System.out.println(bd.append("345") == bd);
// output : true
StringBuffer bf = new StringBuffer("123");
System.out.println(bf.append("345") == bf);
// output : true
```

##### StringBuilder
StringBuilder继承AbstractStringBuilder类所有方法均调用父类实现

##### StringBuffer
StringBuffer继承AbstractStringBuilder类所有方法均调用父类实现
所有方法均使用synchronized关键字修饰，所以StringBuffer是线程安全的

#### 结语
实际上在编译期优化了的string+操作的运行效率并非低于使用其他两个类
因为许多编译器会将此操作优化为builder.append的形式
理论上执行效率 StringBuilder > StringBuffer > String
