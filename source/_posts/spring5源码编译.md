---
title: spring5源码编译
date: 2019-10-13 17:55:30
category:
- javaEE
tags:
- spring
- java
---

这一阵子在读spring源码，但是网上的文章，构建spring源码大都有问题，导致我构建源码的时候出现了很多问题，所以我自己重新构建了一下。纠结了一会儿要不要写这篇文章，还是决定写下来，当作笔记用，同时给广大网友提供一个编译spring源码的方法。

<!-- more -->

### 本机环境

win10 + jdk1.8.0_151 + IntelliJ IDEA 2019.1.3
spring使用的是5.0版本（下面介绍的方法在5.0和5.1下都可以成功）

### 源码构建
1. 在cmd命令框下进入源码目录，使用命令： `gradlew :spring-oxm:compileTestJava`

	> 网上很多文章都说这里要下载gradle，其实是可以不用的，输入这个指令之后会自己下载的（亲测），截图如下（中间可能会出错，十有八九是网络问题，建议挂vpn，或者多试几次）

	![](预编译1.jpg)
	
	最好再预编译一下spring-core：

	![](预编译2.jpg)

2. 打开Intellij IDEA，依次选择`File -> New -> Project from Existing Sources -> spring项目根目录 -> 选择 build.gradle`，然后如下图，一路ok即可。
	
	![](new.jpg)
	
	![](config.jpg)
	
	点击完finish之后会要等待一段时间，看个人网速，大概20分钟吧，构建成功后的结果如下图所示，如果中间有什么问题就直接点击idea左边的刷新试试（这里我没有出过问题..）。
	![](success.jpg)

	其实到了这一步已经差不多完成了，只不过后面如果要使用到spring源码还得接着往下看

3. 新建module：
	![](new-module.jpg)
	
	![](name.jpg)
	
	添加spring-context依赖：

	![](importSpringContext.jpg)

	新建MyConfig类和Main类用来测试
	```java
	@Configuration
	public class MyConfig {
	
	}

	public class Main {
		public static void main(String[] args) {
			AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(MyConfig.class);
			System.out.println(ac);
		}
	}
	```
	点击运行，此时会出现错误如下，原因是spring-context中没有添加spring-instrument依赖

	![](error.jpg)

	只需要在spring-context模块中的gradle配置文件中添加依赖即可：

	![](spring-context-config.jpg)

	此时再次运行即可成功：
	![](finalSuccess.jpg)

4. **注意**：后面如果需要使用aspects模块的时候，需要先将这个模块unload（亲测需要这样做，别问我原因，我看文档说要这样做的...）。
