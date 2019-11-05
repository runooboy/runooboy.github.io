---
title: sql中count(1)、count(*)和count(字段名)的区别
date: 2019-10-23 10:48:25
category:
- sql
tags:
- sql
- count
---

### 测试实例

![](table_info.jpg)

<!--more-->


1. count(1)测试，sql语句如下：
	```sql
	select count(1) from ums_member;
	```

	结果如下：
	![](coung1.jpg)

2. count(*)测试，sql语句如下：
	```sql
	select count(*) from ums_member;
	```

	结果如下：
	![](countxing.jpg)

3. count(主键)测试，sql语句如下：
	```sql
	select count(id) from ums_member; --id为主键字段
	```

	结果如下：
	![](countprimary.jpg)

4. count(普通列)测试，sql语句如下：
	```sql
	select count(phone) from ums_member;
	```

	结果如下：
	![](countlie.jpg)

### count(1)、count(*)和count(字段名)执行效果上的区别  
  
- count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL  
- count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL  
- count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

### count(1)、count(*)和count(字段名)执行效率上的区别 

如上可以看出其实四种方式的效率是一样的，但是网上有如下说法：

- 列名为主键，count(列名)会比count(1)快  
- 列名不为主键，count(1)会比count(列名)快  
- 如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）  
- 如果有主键，则 select count（主键）的执行效率是最优的  
- 如果表只有一个字段，则 select count（*）最优。

具体的情况还是得看业务场景中使用时的效果，在业务场景中可以多试试。