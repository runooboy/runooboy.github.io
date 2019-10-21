---
title: RESTFul
date: 2019-10-17 20:30:21
category:
- javaEE
tags:
- javaWeb
---
## RESTful是什么？

REST是**REpresentational State Transfer**的缩写（表述性状态转移），REST是一种体系结构。

<!-- more -->

简单的说，REST就是将资源的状态以合适客户端或服务端的形式从服务端转移到客户端（或者反过来）。在REST中，资源通过 **URL** 进行识别和定位，然后通过不同的 **action（既HTTP方法，GET等）** 来定义REST应该完成的功能。

在实际的web开发中，CRUD动作与HTTP方法的匹配一般如下：

| CRUD动作 | HTTP方法    |
| ------ | --------- |
| Create | POST      |
| Read   | GET       |
| Update | PUT或PATCH |
| Delete | DELETE    |

上述匹配不是绝对的，可以根据需要进行改动，但一般情况这样写比较好。

在使用RESTful风格之前，如果想要增加一条商品数据，通常写法是这样的：

```java
/addCategory?name=xxx
```

但是使用RESTful之后，就会变成这样：

```java
/category   (请求方式为POST即可)
```

这就是使用同一个URL，但是通过约定不同的HTTP方法对应不同的CRUD动作来实现不同的业务。

下面再写一个具体的RESTful的API实现：

| 请求类型   | URL      | 功能说明          |
| ------ | -------- | ------------- |
| GET    | /users   | 查询用户列表        |
| POST   | /users   | 创建一个用户        |
| GET    | /user/id | 根据id查询一个用户的数据 |
| PUT    | /user/id | 根据id更新一个用户的数据 |
| DELETE | /user/id | 根据id删除一个用户数据  |



