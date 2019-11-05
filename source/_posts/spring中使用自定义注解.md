---
title: spring中使用自定义注解
date: 2019-10-29 14:54:07
category:
- javaEE
tags:
- javaEE
- spring
- 注解
---
使用spring时，aop可以大幅度减少我们的工作量，这里主要介绍一些spring中aop如何切注解。

<!--more-->

1. 自定义注解：

	```java
	@Target({ElementType.METHOD})
	@Retention(RetentionPolicy.RUNTIME)
	public @interface MyAnnotation {
	    String value();
	}
	```

2. 编写切面类，切点在自定义注解上，切面类添加到spring容器中。

	```java
	@Aspect
	@Component
	public class AnnotationAspect {
	    @Pointcut("@annotation(com.annotation.learn.aop.MyAnnotation)")
	    public void pointCut(){}
	
	    @Before(value = "pointCut() && @annotation(myAnnotation)")
	    public void start(JoinPoint joinPoint, MyAnnotation myAnnotation){
	        System.out.println("Before...");
	        Object[] args = joinPoint.getArgs();
	        System.out.print(joinPoint.getSignature().getName()+"运行，参数列表："+ Arrays.asList(args));
	        System.out.println("；注解为："+myAnnotation);
	    }
	
	    @After(value = "pointCut()")
	    public void end(){
	        System.out.println("After...");
	    }
	
	    @AfterReturning(value = "pointCut()", returning = "result")
	    public void logAfterReturning(Object result){
	        System.out.print("AfterReturning ....");
	        System.out.println("结果为："+result);
	    }
	
	    @AfterThrowing(value = "pointCut()", throwing = "exception")
	    public void logException(Exception exception){
	        System.out.print("exception...");
	        System.out.println("异常信息为："+exception.getMessage());
	    }
	}
	```
3. 使用自定义注解：

	```java
	@Service
	public class PersonServiceImpl implements PersonService {
	    @MyAnnotation(value = "这是PersonServiceImpl,show1()的注解")
	    public void show1(String p){
	        System.out.println("PersonServiceImpl... show1()"+"；参数是："+p);
	    }
	}
	```
4. 测试注解：

	```java
	@Test
	public void test2(){
	    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(ConfigOfAOP.class);
	    PersonServiceImpl bean = applicationContext.getBean(PersonServiceImpl.class);
	    bean.show1("aop");
	}
	```

5. 打印结果：
	```java
	Before...
	show1运行，参数列表：[aop]；注解为：@com.annotation.learn.aop.MyAnnotation(value=这是PersonServiceImpl.show1()的注解)
	PersonServiceImpl.show1()；参数是：aop
	After...
	AfterReturning ....结果为：null
	```

