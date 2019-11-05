---
title: sql优化-Mysql
date: 2019-10-17 20:30:21
category:
- sql
tags:
- sql优化
- mysql
- 性能优化
---

## Mysql理论

### Mysql分层、存储引擎

![](MySql分层.png)

<!-- more -->
常用引擎对比：
InnoDB:事务优先(适合高并发操作，使用行锁)
MyISAM:性能优先（使用表锁）

可以使用`show engines;`查看当前使用数据库支持哪些引擎；使用`show VARIABLES LIKE '%storage_engine%';`正在使用的引擎。

### sql解析过程、索引、B树

#### sql解析过程

一条sql示例如下：
```sql
SELECT DISTINCT
    < select_list >
FROM
    < left_table > < join_type >
JOIN < right_table > ON < join_condition >
WHERE
    < where_condition >
GROUP BY
    < group_by_list >
HAVING
    < having_condition >
ORDER BY
    < order_by_condition >
LIMIT < limit_number >
```

然而它的执行过程却是：
```sql
FROM <left_table>
ON <join_condition>
<join_type> JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT 
DISTINCT <select_list>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

#### 索引
索引相当于书的目录，可以帮助Mysql高效获取数据的数据结构，一般使用树、B树、Hash树..完成

索引的弊端：
1. 索引本身很大，可以存放在内存/硬盘（通常为硬盘）
2. 索引不是所有情况均适用：
	少量数据
	频繁更新的字段
	很少使用的字段
3. 索引会降低增删改的效率

索引的优势：
1. 提高查询效率（降低IO使用率）
2. 降低CPU使用率(... order by age desc，因为B树索引本身就是一个排好序的结构，不再需要排序)

**索引分类**
- 主键索引：不能重复。id 不能是null
- 唯一索引：不能重复。id 可以是null
- 单值索引：单列，age；一个表可以有多个单值索引,name。
- 复合索引：多个列构成的索引（相当于二级目录：z:zhao）

**创建索引**

1. 方式一：create <索引类型> <索引名> on <表名(字段)>
	单值： `create index dept_index on tb(dept)`
	唯一： `create unique index name_index on tb(name)`
	复合： `create index dept1_dept2_index on tb(dept1,dept2)`
2. 方式二：alter table <表名> add <索引类型> <索引名(字段)>
	单值： `alter table tb add index dept_index(dept)`
	唯一： `alter table tb add unique index name_index(name)`
	复合： `alter table tb add index dept1_dept2_index(dept1,dept2)`
	
**删除索引**
drop index 索引名 on 表名

**查询索引**
show index from 表名


## sql优化
sql优化主要通过优化索引完成。
通过`explain`分析sql的执行计划，可以模拟sql优化器执行sql语句，从而知道 MySQL 是如何处理你的 SQL 语句的。分析查询语句或是表结构的性能瓶颈。

### EXPLAIN 使用方式
```sql
EXPLAIN + SQL 语句
```

执行计划包含的信息
- 表的读取顺序 ( id )

- 数据读取操作的操作类型 ( select_type )

- 每个子查询使用了哪种类型 ( type )

- 哪些索引可以使用 ( possible_keys )

- 哪些索引被实际使用 ( key )

- 索引的长度 ( key_len )

- 表的连接匹配条件，即哪些列或常量被用于查找索引列上的值 ( ref )

- 每张表有多少行被优化器查询 ( rows )

#### 执行计划各字段含义： id

- id 相同，执行顺序由上至下
- id 不同，如果是子查询，id 的序号会递增，id 值越大优先级越高，越先被执行
- id 同时存在相同的和不同的,还是按照上面两个规则执行
- id 为 null 时表示一个结果集，不需要使用它查询，常出现在包含 union 等查询语句中

#### 执行计划各字段含义： select_type

分别用来表示查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询。

- SIMPLE 简单的 select 查询，查询中不包含子查询或者 union
- PRIMARY 查询中若包含任何复杂的子部分，最外层查询则被标记为 primary
- SUBQUERY 在 select 或 where 列表中包含了子查询
- DERIVED 在 from 列表中包含的子查询被标记为 derived（衍生），MySQL 会递归执行这些子查询，把结果放在临时表中
- UNION 若第二个 select 出现在 union 之后，则被标记为 union，若 union 包含在 from - 子句的子查询中，外层 select 将被标记为：derived
- UNION RESULT 从 union 表获取结果的 select

#### 执行计划各字段含义： table
标识查询的数据表
1. 若查询时表使用了别名，则 table 显示别名；
2. 当从衍生表（即临时表）中查数据时会显示 <derived x>, x 表示对应的执行计划 id；
3. 若显示 <union x,y>，也表示一个临时表，表示这个结果来自于执行计划 id 为 x，y 的结果集；


#### 执行计划各字段含义： type
type 所显示的是查询使用了哪种类型，type 包含的类型包括如所示的几种：
```
system > const > eq_ref > ref > range > index > all
```
> 一般来说，得保证查询至少达到 range 级别，最好能达到 ref

- NULL MySQL 在优化过程中分解语句，执行时甚至不用访问表或索引
- system 表只有一行记录（等于系统表），这是 const 类型的特列，平时不会出现，这个也可以忽略不计
- const 表示通过索引一次就找到了，const 用于比较 primary key 或者 unique 索引。因为只匹配一行数据，所以很快。如将主键置于 where 列表中，MySQL 就能将该查询转换为一个常量
- eq_ref 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
- ref 非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体
- range 只检索给定范围的行，使用一个索引来选择行，key 列显示使用了哪个索引，一般就是在你的 where 语句中出现 between、< 、>、in 等的查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引
- index Full Index Scan，Index 与 All 区别为 index 类型只遍历索引树。这通常比 All 快，因为索引文件通常比数据文件小。（也就是说虽然 All 和Index都是读全表，但 Index 是从索引中读取的，而 All 是从硬盘读取的）
- all Full Table Scan 将遍历全表以找到匹配的行

#### 执行计划各字段含义： possible_keys 和 key

**possible_keys**

显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用。

**key**
- 实际使用的索引，如果为NULL，则没有使用索引。（可能原因包括没有建立索引或索引失效）
- 查询中若使用了覆盖索引（select 后要查询的字段刚好和创建的索引字段完全相同），则该索引仅出现在 key 列表中

#### 执行计划各字段含义： key_len
查询中若使用了覆盖索引（select 后要查询的字段刚好和创建的索引字段完全相同），则该索引仅出现在 key 列表中

#### 执行计划各字段含义：ref
显示索引的那一列被使用了，如果可能的话，最好是一个常数。哪些列或常量被用于查找索引列上的值

#### 执行计划各字段含义： rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数，也就是说，用的越少越好

#### 执行计划各字段含义： Extra
- Using filesort：说明 MySQL 会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL 中无法利用索引完成的排序操作称为“文件排序”。
- Using temporary：表示 MySQL 需要使用临时表来存储结果集，常见于 order by 和 group by。
- Using index：表示相应的 select 操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错。如果同时出现 using where，表明索引被用来执行索引键值的查找；如果没有同时出现 using where，表明索引用来读取数据而非执行查找动作。
- Using where：表明使用了where过滤
- Using join buffer：表明使用了连接缓存，比如说在查询的时候，多表 join 的次数非常多，那么将配置文件中的缓冲区的 join buffer 调大一些。
- impossible where：where 子句的值总是 false，不能用来获取任何元组
- select tables optimized away：在没有 group by 子句的情况下，基于索引优化 MIN/MAX 操作或者对于 MyISAM 存储引擎优化 COUNT(*) 操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
- distinct：优化 distinct 操作，在找到第一匹配的元组后即停止找同样值的动作
- firstmatch：5.6.x 开始引入的优化子查询的新特性之一，常见于 where 字句含有 in() 类型的子查询。如果内表的数据量比较大，就可能出现这个
- loosescan：5.6.x 之后引入的优化子查询的新特性之一，在 in() 类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个


