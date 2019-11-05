---
title: sql的null、''
date: 2019-09-27 15:18:40
category:
- sql
---
### sql中null,''的区别

#### sql中Varchar为null,''的区别

首先先看一下java中的String类型数据在数据库中为null或者空以及''时的区别，数据库中的数据如下：
<!-- more -->
![](string.png)

本次测试主要是使用`springboot+mybatis`来实现的，java代码如下：
```java
@ApiOperation(value = "测试String 数据库中的null和''")
@PostMapping("/test9")
public ResponseEntity<List<EntityTable>> test9() {
    List<EntityTable> entityTables = entityTableMapper.selectAll();
    for (EntityTable entityTable : entityTables) {
        System.out.println(entityTable.getEntityName());
        System.out.println(entityTable.getEntityName() == null);
        System.out.println("".equals(entityTable.getEntityName()));
        System.out.println(entityTable);
        System.out.println("\n");
    }
    return Results.success(entityTables);
}
```

打印结果：
```java
2
false
false
EntityTable{id=1, tableName='2', entityName='2', multiLanguage=1, primaryKeyName='2', primaryKeyStrategy='2', primaryKeyGenerator='2', supportWho=2, supportVersion=2, status='2', meaning='null'}



false
true
EntityTable{id=2, tableName='string', entityName='', multiLanguage=null, primaryKeyName='string', primaryKeyStrategy='string', primaryKeyGenerator='string', supportWho=0, supportVersion=0, status='string', meaning='null'}


null
true
false
EntityTable{id=6, tableName='', entityName='null', multiLanguage=null, primaryKeyName='null', primaryKeyStrategy='null', primaryKeyGenerator='null', supportWho=null, supportVersion=null, status='null', meaning='null'}
```
结果可以看到，在数据库中`''!=null`，也就是说如果数据库中某个记录的某一个字段(Varchar类型)原本是有数据的，但是经过sql操作后删除了里面的数据，这个时候这条记录其实和一开始插入数据时这个字段为空是有区别的。


#### sql中数字型为null,''的区别

数据库中的字段：
![](number.png)

java代码：
```java
@ApiOperation(value = "测试int、long 数据库中的null和空，''")
@PostMapping("/test8")
public ResponseEntity<List<EntityTable>> test8(){
    List<EntityTable> entityTables = entityTableMapper.selectAll();
    for(EntityTable entityTable : entityTables){
        System.out.println(entityTable.getMultiLanguage());
        System.out.println(entityTable.getMultiLanguage() == null);
        System.out.println("".equals(entityTable.getMultiLanguage()));
        System.out.println(entityTable);
        System.out.println("\n");
    }
    return Results.success(entityTables);
}
```

打印结果：
```java
1
false
false
EntityTable{id=1, tableName='2', entityName='2', multiLanguage=1, primaryKeyName='2', primaryKeyStrategy='2', primaryKeyGenerator='2', supportWho=2, supportVersion=2, status='2', meaning='null'}


null
true
false
EntityTable{id=2, tableName='string', entityName='string', multiLanguage=null, primaryKeyName='string', primaryKeyStrategy='string', primaryKeyGenerator='string', supportWho=0, supportVersion=0, status='string', meaning='null'}


3
false
false
EntityTable{id=3, tableName='2', entityName='2', multiLanguage=3, primaryKeyName='3', primaryKeyStrategy='3', primaryKeyGenerator='3', supportWho=3, supportVersion=2, status='3', meaning='null'}
```
而数据库中的数据类型字段则不一样，它是没有`''`的，如果这个字段没有值就直接是null,而不是其他

