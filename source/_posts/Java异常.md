---
title: Java异常
date: 2019-09-07 21:00:18
category:
- javaSE
tags:
- Exception
- java
---
### Exception概述

>`throw new Exception()`既代表 java 运行进入了一个新的分支，不再执行`new Exception()`后面的语句了，之后再来执行finally中的内容。

<!-- more -->
![](ExceptionStruct.png)

java将所有的错误封装为一个对象，其根本父类为 Throwable, Throwable 有两个子类：Error 和 Exception。
1. Error 是 Throwable 的子类，用于指示合理的应用程序不应该试图捕获的严重问题。大多数这样的错误都是异常条件。虽然 ThreadDeath 错误是一个“正规”的条件，但它也是 Error 的子类，因为大多数应用程序都不应该试图捕获它。在执行该方法期间，无需在其 throws 子句中声明可能抛出但是未能捕获的 Error 的任何子类，因为这些错误可能是再也不会发生的异常条件。
2. Exception 类及其子类是 Throwable 的一种形式，它指出了合理的应用程序想要捕获的条件。
3. RuntimeException（运行时异常、既unchecked，未检查异常）是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类。可能在执行方法期间抛出但未被捕获的, <u>RuntimeException 的任何子类都无需在 throws 子句中进行声明。</u>它是Exception 的子类。

异常的分类
1. Error：一般为底层的不可恢复的类；
2. Exception：分为未检查异常(RuntimeException)和已检查异常(非RuntimeException)。


未检查异常是因为程序员没有进行必需要的检查，因为疏忽和错误而引起的错误。几个经典的RunTimeException如下：

1. java.lang.NullPointerException;
2. java.lang.ArithmaticException;
3. java.lang.ArrayIndexoutofBoundsException

Runtime Exception 和 Exception：
1. Runtime Exception： 
在定义方法时不需要声明会抛出runtime exception； 在调用这个方法时不需要捕获这个runtime exception； runtime exception是从java.lang.RuntimeException或java.lang.Error类衍生出来的。 例如：nullpointexception，IndexOutOfBoundsException就属于runtime exception 

2. Exception:
定义方法时必须声明所有可能会抛出的exception； 在调用这个方法时，必须捕获它的checked exception，不然就得把它的exception传递下去；exception是从java.lang.Exception类衍生出来的。例如：IOException，SQLException就属于Exception。 
Exception 属于应用程序级别的异常，这类异常必须捕捉,Exception体系包括RuntimeException体系和其他非RuntimeException的体系RuntimeException 表示系统异常，比较严重，如果出现RuntimeException，那么一定是程序员的错误

### 异常处理机制
在java中，异常有两种处理机制：
1. try-catch-finally
2. throw

上面说的异常处理的两种方式是有区别的，第一种遇到异常后在catch中直接处理掉，处理完成后继续执行try-catch代码块后面的代码，而throw则是抛给上一层处理（可以一直向上抛，直到在）。

一个简单的例子：

```java
@Test
public void test1(){
    int i = 10 / 0;
    System.out.println("程序除0了...");
    /**
     * 打印结果：
     * java.lang.ArithmeticException: / by zero
     *
     * 	at com.exception.ExceptionTest.test2(ExceptionTest.java:32)
     */
}
```
在这个例子中我是直接尝试让程序除0，然后此时就会直接抛出`ArithmeticException`，后面的打印代码也是直接不再执行的，这样就影响了程序的正常运行。


#### 异常处理机制一：try-catch-finally
其实不管是哪一种Exception，在我们知道会发生异常的时候，最终的解决方案都是`try-catch-finally`,throw到最后还是要通过`try-catch-finally`来解决的。我们对上面的例子进行改良：
```java
@Test
public void test2(){
    try {
        int i = 10 / 0;
        System.out.println("程序除0了...");
    }catch (Exception e){
        System.out.println("catch到了异常...");
    }finally {
        System.out.println("finally中的代码...");
    }
    System.out.println("try-catch-finally之后的代码...");
    /**
     * 打印结果：
     * catch到了异常...
     * finally中的代码...
     * try-catch-finally之后的代码...
     */
}
```
此时我们就看到，在发生异常的时候，我们可以在catch中解决好碰到的异常，然后代码块后面的代码跟着正常执行。

#### 异常处理机制二：throw
在上面的`try-catch`机制中我们可以看成自身感冒了，然后自己去药店买了点药，吃完后就好了，不会有什么影响；而throw就类似生病了，自身吃药治不好(既当前方法不处理)，这个时候我们就需要到医院跟医生交代我们有什么问题，然后让医生来解决，可能当前医生治不好，那么这个医生是可以接着`throw`到上一级的。
```java
@Test
public void test4() throws FileNotFoundException {
    new FileInputStream(new File("dd"));
    System.out.println("---------");
    /**
     * 打印结果：
     * java.io.FileNotFoundException: dd (系统找不到指定的文件。)
     *
     * 	at java.io.FileInputStream.open0(Native Method)
     */
}
```
直接爆红出异常，而且后面的打印语句是没有执行的。说明throw这种方式也会影响程序的正常执行，所有我们需要在调用test4()的地方`try-catch`处理。

### 手动抛异常
在这里介绍手动抛RuntimeException和Exception：
```java
public class MyClass{
    public MyClass() throws RuntimeException{
        throw  new RuntimeException();
    }
    public MyClass(String msg) throws Exception {
        throw new Exception(msg);
    }
}
```
然后我们调用这两个构造器的时候的处理方式如下：
```java
@Test
public void test5(){
    new MyClass();
    /**
     * 打印结果：
     * java.lang.RuntimeException
     * 	at com.exception.MyClass.<init>(MyClass.java:15)
     */
}
@Test
public void test6(){
    try {
        new MyClass("测试untracked Exception");
    } catch (Exception e) {
        System.out.println("捕获到了异常");
    }
    System.out.println("执行完了try-catch");
    /**
     * 打印结果：
     * 捕获到了异常
     * 执行完了try-catch
     */
}
```

### 自定义异常
自定义异常必须继承自Exception或者Exception的子类，一个简单的自定义异常如下：
```java
public class MyException extends Exception {
    public MyException(){
        super();
    }
    public MyException(String msg){
        super(msg);
    }
}
```
抛的方式如下：
```java
public MyClass(String msg) throws MyException {
    throw new MyException(msg);
}
```
### 异常中的子父类
1. 在java中，若碰到异常的时候采用try-catch方式捕获异常，如果第一个catch捕获的异常必须和后面捕获的异常同级或者是后面捕获的异常的子类。
2. 在代码中throw出去的异常一定得是方法上throws出去的异常的子类。