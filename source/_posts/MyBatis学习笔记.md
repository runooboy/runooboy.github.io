---
title: MyBatis学习笔记
date: 2019-09-07 13:17:43
category:
- javaEE
tags:
- MyBatis
---
### MyBatis学习笔记

#### 在MyBatis中一些值得注意的坑
1. 使用xml配置使用MyBatis时，xxxmapper.xml中必须配置namespace，否则会报出statement xxxx的错误。
<!-- more -->
#### MyBatis中的一对一，一对多，多对多关联映射查询

- ##### 一对多（嵌套结果）
将结果嵌套，在collection标签中加入对应的行表列映射：
```xml
<resultMap id="BaseResultMap" type="com.luolin.springssm.model.Student">
	<!-- id 表示主键, column 表示数据库中表的column名称，jdbcType表示，property表示对应model中的实体类的属性 -->
	<id column="s_id" jdbcType="BIGINT" property="sId"/>
	<result column="s_name" jdbcType="VARCHAR" property="sName"/>
	<result column="s_birth" jdbcType="VARCHAR" property="sBirth"/>
	<result column="s_sex" jdbcType="VARCHAR" property="sSex"/>
	<!--使用collention来做到头行的效果-->
	<collection property="sScore" ofType="com.luolin.springssm.model.Score">
		<id column="c_id" jdbcType="VARCHAR" property="cId"/>
		<result column="s_id" jdbcType="VARCHAR" property="sId"/>
		<result column="s_score" jdbcType="BIGINT" property="sScore"/>
	</collection>
</resultMap>
<select id="getAll" resultMap="BaseResultMap" >
	select student.s_id, s_name, s_birth, s_sex
	<!-- 连接查询只要是在sql语句中加上join on的写法 -->
	FROM student left join score c on student.s_id = c.s_id
</select>
```

- ##### 一对多（关联的嵌套查询，在collection中添加select属性）：
这种方式只是写法与上述不同，只要是利用了collection的select属性，select对应的是一个select标签的id，并将结果映射到一个resultMap上去，使用resultType可能会出现数据查找不到的结果，不知道为什么，在网上是看到有人写的是resultType，然后自己本地测试写着没有用。
```xml
<resultMap id="BaseResultMap" type="com.luolin.springssm.model.Student">
	<id column="s_id" jdbcType="VARCHAR" property="sId"/>
	<result column="s_name" jdbcType="VARCHAR" property="sName"/>
	<result column="s_birth" jdbcType="VARCHAR" property="sBirth"/>
	<result column="s_sex" jdbcType="VARCHAR" property="sSex"/>
	<collection property="sScore" javaType="list" column="s_id" ofType="com.luolin.springssm.model.Score" select="queryScore"/>
</resultMap>
<resultMap id="ScoreResultMap" type="com.luolin.springssm.model.Score">
	<result column="s_id" jdbcType="VARCHAR" property="sId"/>
	<result column="c_id" jdbcType="VARCHAR" property="cId"/>
	<result column="s_score" jdbcType="BIGINT" property="sScore"/>
</resultMap>
<select id="getAll" resultMap="BaseResultMap" >
	select s_id, s_name, s_birth, s_sex
	FROM student
</select>
<select id="queryScore" parameterType="int" resultMap="ScoreResultMap" >
	<!--下面这两行效果是一样的-->
	select s_id, c_id, s_score from score where s_id = #{s_id}
	select s_id, c_id, s_score from score where s_id = #{sId}
</select>
```

- ##### 多对多
多对多暂时先不写吧，后面有空，然后碰到了类似的问题的时候再来写好了。
#### MyBatis插入时参数问题和MyBatis中#{}和${}区别
- MyBatis中做insert操作时，values后面的参数可以直接使用插入的数据类型中的属性，接下来写一个案例：
在这个案例中Controller中的方法为：
```java
@PostMapping("/addStudent")
public int addStudent(@RequestBody Student student){
	return studentMapper.addStudent(student);
}
```
传递为json格式数据。
在xxxMapper.xml中写的配置为：
```xml
<insert id="addStudent" parameterType="com.luolin.springssm.model.Student">
	insert into student(s_id, s_name, s_birth, s_sex) values(#{sId}, #{sName}, #{sBirth}, #{sSex})
</insert>
```
此时在接口xxxMapper就应该是写：
```java
public int addStudent(Student student);
```
倘若此时写的是：
```java
public int addStudent(@Param("student") Student student);
```
那么对不起，直接会报错：
```java
Parameter 'sId' not found. Available parameters are [student, param1]
```
此外还有一种写法，那就是在接口xxxMapper中给参数加上注解：
```java
public int addStudent(@Param("student") Student student);
```
但是这个时候xxxMapper.xml中写的配置为：
```xml
<insert id="addStudent" parameterType="com.luolin.springssm.model.Student">
	insert into student(s_id, s_name, s_birth, s_sex) values(#{student.sId}, #{student.sName}, #{student.sBirth}, #{student.sSex})
</insert>
```
这个时候用student去标识一下是哪个类（本例中为Student）的属性
总结，第一种写法为什么会成功，目前我也不清楚，暂时没有太多时间去了解，只能后面在去研究一下MyBatis的源码才能了解，我现在的猜测是MhyBatis自动映射过来了（有待验证）。个人推荐使用第二种方式写吧，感觉更清晰。

- MyBatis中#{}和${}区别还是很大的：
\#{}是预编译好的占位符，它在编译前就已经给值加上了''，可以有效防止sql注入，这是它最大的优点。比如：
```sql
select * from user where name = #{name}
如果此时我传一个markdown，那么传过来就是:
select * from user where name = 'markdown'
```
${}是不会将传入的值做预编译的，比如
```sql
select * from user where name = ${name}
如果此时我传一个markdown，那么传过来就是:
select * from user where name = markdown
```
关于网上说的** MyBatis排序时使用order by 动态参数时需要注意，用$而不是#，**其实现在不是很懂，希望以后遇到然后弄明白。