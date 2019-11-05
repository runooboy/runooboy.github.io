---
title: java自定义注解以及通过反射获取注解信息
date: 2019-10-29 14:51:48
category:
- javaSE
tags:
- javaSE
- 注解
---

随着spring的流行，java注解的使用成为了java工程师必不可少的技能。

<!--more-->

### 注解基本知识

1. 元注解：@Retention、 @Target、 @Document、 @Inherited；

2. Annotation型定义为@interface, 所有的Annotation会自动继承java.lang.Annotation这一接口,并且不能再去继承别的类或是接口。

3. 参数成员只能用public或默认(default)这两个访问权修饰

4. 参数成员只能用基本类型byte，short，char，int，long，float，double，boolean八种基本数据类型和String、Enum、Class、annotations等数据类型，以及这一些类型的数组。

5. 要获取类、方法和字段的注解信息，必须通过Java的反射技术来获取 Annotation对象,除此之外没有别的获取注解对象的方法

6. 注解也可以没有定义成员, 不过这样注解就没啥用了，只起到标识作用

#### 元注解详解
自定义注解类时, 可以指定目标 (类、方法、字段, 构造函数等) , 注解的生命周期(运行时,class文件或者源码中有效), 是否将注解包含在javadoc中及是否允许子类继承父类中的注解, 这些都是通过元注解完成的。

1. @Target 表示该注解目标,可能的 ElemenetType 参数包括：
	ElemenetType.CONSTRUCTOR 构造器声明
	ElemenetType.FIELD 域声明(包括 enum 实例)
	ElemenetType.LOCAL_VARIABLE 局部变量声明
	ElemenetType.METHOD 方法声明
	ElemenetType.PACKAGE 包声明
	ElemenetType.PARAMETER 参数声明
	ElemenetType.TYPE 类，接口(包括注解类型)或enum声明

2. @Retention 表示该注解的生命周期,可选的 RetentionPolicy 参数包括
	RetentionPolicy.SOURCE 注解将被编译器丢弃
	RetentionPolicy.CLASS 注解在class文件中可用，但会被JVM丢弃
	RetentionPolicy.RUNTIME JVM将在运行期也保留注释，因此可以通过反射机制读取注解的信息

3. @Documented 指示将此注解包含在 javadoc 中

4. @Inherited 指示允许子类继承父类中的注解


### 自定义注解使用

自定义注解:
```java
@Target({ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME) // 如果不是RUNTIME，在运行期拿到的会是null
public @interface MyAnnotation {
    String desc();
    int value();
}
```

使用自定义注解：
```java
@MyAnnotation(desc = "class-Person", value = 0)
class Person{
    @MyAnnotation(desc = "field-name", value = 1)
    public String name;

    @MyAnnotation(desc = "function-show1()", value = 2)
    public String show1(@MyAnnotation(desc = "parameter-p1",value = 3) String p1){
        System.out.println("test1...");
        return p1;
    }
}
```
测试自定义注解：
```java
public class TestAnnotation {
    public static void printMyAnnotation(String fieldName, MyAnnotation annotation){
        System.out.println(fieldName+"上的注解：" + annotation + "; 它的desc："+annotation.desc()+"; 它的value:"+annotation.value());
    }
    public static void main(String[] args) throws Exception {
        Class<Person> clazz = Person.class;
        MyAnnotation annotation;
        // 获取类上的注解
        annotation = clazz.getAnnotation(MyAnnotation.class);
        printMyAnnotation("类", annotation);

        // 获取方法上的注解
        Method show1 = clazz.getDeclaredMethod("show1", String.class);
        annotation = show1.getAnnotation(MyAnnotation.class);
        printMyAnnotation("方法", annotation);

        // 获取参数上的注解
        Annotation[][] annotations = show1.getParameterAnnotations();
        for (Annotation[] annotation1 : annotations) {
            for (Annotation annotation2 : annotation1) {
                annotation = (MyAnnotation) annotation2;
                printMyAnnotation("方法参数", annotation);
            }
        }

        // 获取字段上注解
        Field name = clazz.getDeclaredField("name");
        annotation = name.getAnnotation(MyAnnotation.class);
        printMyAnnotation("字段", annotation);
    }
}
```
打印结果：
```java
类上的注解：@com.annotation.MyAnnotation(desc=class-Person, value=0); 它的desc：class-Person; 它的value:0
方法上的注解：@com.annotation.MyAnnotation(desc=function-show1(), value=2); 它的desc：function-show1(); 它的value:2
方法参数上的注解：@com.annotation.MyAnnotation(desc=parameter-p1, value=3); 它的desc：parameter-p1; 它的value:3
字段上的注解：@com.annotation.MyAnnotation(desc=field-name, value=1); 它的desc：field-name; 它的value:1
```


