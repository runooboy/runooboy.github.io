---
title: sql中的联表查询
date: 2019-10-23 10:48:07
category:
- sql
tags:
- sql
- join
---
sql中的join（本文仅介绍left join、right join、join on，图片是网上找的，原作者如果不允许使用请联系我）如下：

<!--more-->

![](join.png)
<!-- more -->

测试数据：

![](table.png)

### 左连接，left join
左连接使用示例：
```sql
select * from teacher t left join course c on t.tid = c.tid;
```

结果：

![](leftjoin.jpg)

通过上面的结果可以看出，左连接会将其左边的表作为主表（记为A），右边的表作为从表（记为B），A中的记录必定会存在，B中没有对应记录时显示为空，B中有多条记录对应时则A中的记录会重复显示。

经过测试有(下面这个仅作记录)：
```
-- 下面两种写法结果一样
select * from teacher t left join course c on t.tid = c.tid;
select * from teacher t left join course c on c.tid = t.tid;
```

### 右连接，right join

右连接与左连接则正好相反，使用如下sql语句：
```sql
select * from course c right join teacher t on t.tid = c.tid;
```
结果如下：
![](rightjoin.jpg)

此时也得到了如下结论：
```sql
-- 下面两种写法结果一样、只是列的顺序正好反过来的
select * from course c right join teacher t on t.tid = c.tid;
select * from teacher t left join course c on c.tid = t.tid;
```
显然这个结果也印证了我上面说的右连接与左连接正好相反。

### 内连接，join on

内连接与上面两种不同，它会将两张表中不符合on后面条件的记录全部筛掉，可以理解为在左连接的基础上将从表中记录为null的记录去除掉（左连接中所说的B表），使用如下sql语句：

```sql
select * from course c join teacher t on t.tid = c.tid;
```

结果为：

![](innerjoin.jpg)

### 多个联表查询

多个联表查询原理其实还是和上面的类似，只要懂了上面所说的内容，那么多个联表查询也就十分简单了，看下面的sql：

```sql
select * from teacher t join teachercard tc on t.tid = tc.tid 
left join course c on tc.tid = c.tid;
```

查询出来的结果如下：

![](join.jpg)

显然它的结果是根据teacher表和teachercard表联表查出来的结果再去进行左连接查询。
