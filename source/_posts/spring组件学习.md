---
title: spring组件学习
date: 2019-10-26 19:17:22
tags:
- spring
- bean
category:
- javaSE
---

本文主要记录学习到的spring组件以及spring组件设置。

### 给spring容器中注册组件

通过使用注解方式向spring容器中添加组件是十分简单的，大体有如下方式：

<!--more-->

1. <a href="#component1">在config类中通过添加@Bean来添加注解</a>
2. <a href="#component2">通过扫描添加组件</a>
3. <a href="#component3">使用@Import添加一个外部组件</a>
4. <a href="#component4">使用@ImportSelector添加一个组件</a>
5. <a href="#component5">使用ImportBeanDefinitionRegistrar添加一个组件</a>
6. <a href="#component6">使用FactoryBean注册组件</a>

#### <p name="component1">在config类中通过添加@Bean来添加注解</p>
使用方式如下：
```java
@Configuration
public class MyConfig {
	@Bean
	public Target target(){
		return new Target();
	}
}
```

#### <p name="component2">通过扫描添加组件</p>
使用@ComponentScan+组件注解@Controller/@Service/@Repository/@Component扫描添加组件
使用方式如下：
```java
@ComponentScan("org.springframework.demo")
@Configuration
public class MyConfig {
}
```
另外，通过扫描方式添加组件的方式可以通过一些注解来配置组件：
1. @TypeFilter指定过滤规则
2. @Scope设置组件作用域
3. @Lazy设置是否懒加载
4. @Conditional按照条件注册bean

#### <p name="component3">使用@Import添加一个外部组件</p>
使用方式如下：
```java
@Configuration
@Import(value = {Example1.class})
public class MyConfig {
}
```


#### <p name="component4">使用@ImportSelector添加一个组件</p>
使用方式如下：
```java
@Configuration
@Import(value = {MyImportSelector.class})
public class MyConfig {
}
```

#### <p name="component5">使用FactoryBean注册组件</p>
使用方式如下：
```java
@Configuration
@Import(value = {MyBeanDefinitionRegister.class})
public class MyConfig {
}

public class MyBeanDefinitionRegister implements ImportBeanDefinitionRegistrar {
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Example1.class);
		registry.registerBeanDefinition("Dog",rootBeanDefinition);
	}
}
```

#### <p name="component6">使用FactoryBean注册组件</p>
使用方式如下：
```java
// 创建一个FactoryBean的实现类
public class ExampleFactoryBean implements FactoryBean<Example3> {
	// 返回一个Example3对象，这个对象会被添加到spring容器中
	@Override
	public Example3 getObject() throws Exception {
		return new Example3();
	}

	@Override
	public Class<?> getObjectType() {
		return Example3.class;
	}

	// 如果是单例，则返回true，否则返回false
	@Override
	public boolean isSingleton() {
		return true;
	}
}

@Configuration
public class MyConfig {
	// 将这个FactoryBean的实现类添加到spring容器中
	@Bean
	public ExampleFactoryBean exampleFactoryBean(){
		return new ExampleFactoryBean();
	}
}

AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyConfig.class);
ExampleFactoryBean bean = applicationContext.getBean(ExampleFactoryBean.class);
try {
	// getObject()返回Example3 bean
	System.out.println(bean.getObject());
} catch (Exception e) {
	e.printStackTrace();
}
```





