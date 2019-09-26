---
title: java基础知识
date: 2019-09-17 09:13:11
category:
- javaSE
tags:
- javaSE
---

### Java中默认的访问权限作用域

|作用域    | 当前类 | 同一包(package) | 子孙类 | 其他包 |
|-|-|-|-|-|
|public   | Y| Y| Y|Y|
|protected| Y| Y| Y|N|
|default  | Y| Y| N|N|
|private  | Y| N| N|N|

关于抽象类
JDK 1.8以前，抽象类的方法默认访问权限为protected
JDK 1.8时，抽象类的方法默认访问权限变为default
关于接口
JDK 1.8以前，接口中的方法必须是public的
JDK 1.8时，接口中的方法可以是public的，也可以是default的



