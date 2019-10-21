---
title: java常用类
date: 2019-10-15 11:34:42
category:
- javaSE
tags:
- javaSE
- String
- StringBuffer
- StringBuilder
- Date
---

## String、StringBuffer、StringBuilder



### String

![](memory.png)

<!-- more -->
#### String的概述

String类代表字符串，Java程序中的所有字符串字面值（如“abc”）都作为此类的实例实现。
String类是一个final类，代表不可变的字符序列。字符串是常量，用双引号引起来表示，他们的值在创建之后不能更改。
String对象的字符内容是存储在一个字符数组value[]中的。
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
```

**String的不可变性**

字面量值保存在方法区的String常量池中，每次通过字面量方式实例化一个String对象后都会在常量池中添加一个值，后面如果修改的话，修改的是地址（指向另一个String常量（新建）），
```java
@Test
public void test1(){
    String s1 = "abc";
    String s2 = s1;
    s1 = "def";
    System.out.println(s1.equals(s2)); // false
    System.out.println(s1 == s2); // false

    s1 = "abc";
    System.out.println(s1.equals(s2)); //true
    System.out.println(s1 == s2); // true
}
```

#### String类的不同实例化方式对比


#### 不同拼接操作的对比


#### String中的常用方法

### StringBuffer

### StringBuilder

## Date

