---
title: sql中的select 1
date: 2019-10-23 10:52:19
category:
- sql
tags:
- sql
---

前两天在公司项目中看到有人在一个子查询中写了select 1，然后就很好奇这个写法的作用，通过网上各位大佬的介绍，得知select 1在作为子查询中的判断子查询结果是否存在条件时效率较高（大量数据情况下）。

<!--more-->

#### 用法示例

teacher表：
![](teacher.jpg)

teachercard表：
![](teacherCard.jpg)


使用的sql语句如下：

```sql
select * from teachercard tc where exists (select 1 from teacher t where t.dd is not null);
```
sql执行后的结果如下：
![](result1.jpg)

删除teacher表中dd字段存在的记录后，再次执行相同的sql，结果如下：

![](result2.jpg)

根据上面的测试可以得出如下结论：当不需要知道结果是什么，只需要知道有没有结果的时候，可以使用select 1 作为子查询结果是否存在判断，这样可以提高性能。
