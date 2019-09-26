---
title: Java异常
date: 2019-09-07 21:00:18
category:
- javaSE
tags:
- Exception
- java
---
### Java Exception

>抛异常既代表 java 运行进入了一个新的分支，不再执行 new Exception() 后面的语句了，之后再来执行finally中的内容。
<!-- more -->
![](ExceptionStruct.png)
* java将所有的错误封装为一个对象，其根本父类为 Throwable, Throwable 有两个子类：Error 和 Exception。
* Error 是 Throwable 的子类，用于指示合理的应用程序不应该试图捕获的严重问题。大多数这样的错误都是异常条件。虽然 ThreadDeath 错误是一个“正规”的条件，但它也是 Error 的子类，因为大多数应用程序都不应该试图捕获它。在执行该方法期间，无需在其 throws 子句中声明可能抛出但是未能捕获的 Error 的任何子类，因为这些错误可能是再也不会发生的异常条件。
* Exception 类及其子类是 Throwable 的一种形式，它指出了合理的应用程序想要捕获的条件。
* RuntimeException 是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类。可能在执行方法期间抛出但未被捕获的, <u>RuntimeException 的任何子类都无需在 throws 子句中进行声明。</u>它是Exception 的子类。
* 异常的分类
Error：一般为底层的不可恢复的类；
Exception：分为未检查异常(RuntimeException)和已检查异常(非RuntimeException)。


未检查异常是因为程序员没有进行必需要的检查，因为疏忽和错误而引起的错误。几个经典的RunTimeException如下：

1. java.lang.NullPointerException;
2. java.lang.ArithmaticException;
3. java.lang.ArrayIndexoutofBoundsException

Runtime Exception 和 Exception：
Runtime Exception： 
在定义方法时不需要声明会抛出runtime exception； 在调用这个方法时不需要捕获这个runtime exception； runtime exception是从java.lang.RuntimeException或java.lang.Error类衍生出来的。 例如：nullpointexception，IndexOutOfBoundsException就属于runtime exception 


Exception:
定义方法时必须声明所有可能会抛出的exception； 在调用这个方法时，必须捕获它的checked exception，不然就得把它的exception传递下去；exception是从java.lang.Exception类衍生出来的。例如：IOException，SQLException就属于Exception
 
Exception 属于应用程序级别的异常，这类异常必须捕捉,Exception体系包括RuntimeException体系和其他非RuntimeException的体系RuntimeException 表示系统异常，比较严重，如果出现RuntimeException，那么一定是程序员的错误

什么是unchecked异常?
即RuntimeException（运行时异常）
不需要try...catch...或throws 机制去处理的异常

一些测试例子：

1. 已经捕获过父级异常，再去捕获子级异常
	```java
	class Annoyance extends Exception {    
	    public void f() throws Exception {        
	        throw new Exception();    
	    }
	}
	public class Human {   
	    public static void main(String[] args) {
	    Annoyance annoyance = new Annoyance();
	    // Catch the exact type:        
	    try {            
	        annoyance.f();        
	        } catch (Exception e) {
	        e.printStackTrace();
	        // 如下写法，此时会报错：Annoyance已经被捕获，表示捕获到了父类异常，既不会再去捕获子类异常
	        } catch (Annoyance e){
	         }   
	    }
	}
	```

2. 自定义异常继承自Throwable和继承自Exception的区别

	```java
	class SimpleException extends Throwable {
	    public SimpleException(){}
	}
	public class InheritingExceptions {    
	    public void f() throws SimpleException {        
	        System.out.println("f() Throw SimpleException");        
	        throw new SimpleException();    
	    }
	}
	public static void main(String[] args) {       
	    InheritingExceptions inheritingExceptions = new InheritingExceptions();        
	    try {           
	        inheritingExceptions.f();
	    // 此时
	    }catch (Throwable e){            
	        e.printStackTrace(System.out);       
	    }
	}
	```