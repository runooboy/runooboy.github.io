---
title: Java参数问题
date: 2019-09-09 15:01:26
category:
- javaSE
tags:
- java
- java参数
---

### Java传参机制
首先看下java中的参数传递机制:
![](clipboard.png)

<!-- more -->
* 基本数据类型传值（未传递地址）方式：
	```java
	class Main{ 
		public static void main(String[] args) { 
			Main main = new Main(); 
			int a = 20; 
			System.out.println("test之前的a:" + a); 
			main.test(a); 
			System.out.println("test之后的a:" + a); 
		} 
		void test(int a) { 
			a = 10; 
			System.out.println("test中的a:" + a); 
		} 
		/* print: 
		test之前的a:20 
		test中的a:10 
		test之后的a:20 */ 
	}
	```

* 引用数据类型传值（传递的是引用，既地址）方式：
	```java
	@Test
	public void test1(){
	    int[] arr = {2,4,5,6};
	    int[] arr2 = getArr(arr);
	    for(int i = 0; i < arr.length; i++) {
	        System.out.print(arr[i]+" ");
	    }
	
	    System.out.println();
	    for(int i = 0; i < arr2.length; i++) {
	        System.out.print(arr[i]+" ");
	    }
	}
	public int[] getArr(int[] arr){
	    for(int i = 0; i < arr.length; i++){
	        arr[i] *= 2;
	    }
	    return arr;
	}
	/**
	 * 打印结果：
	 * 4 8 10 12 
	 * 4 8 10 12 
	 */
	```

### java可变参数列表
从java5开始，java支持一种参数写法：Java类型后面三个点(如String…)，叫可变长度参数列表。向其中传入参数时，它支持传入数组，个数不一定的同类型（...前面的类型）参数。第一次看到别人的代码里写了它的时候是有点懵的，但是用过两次之后就发现这个东西其实很简单。
```java
public void printNames(String...names){
    for(String name:names){
        System.out.println(name);
    }
}
@Test
public void test2(){
    // 传入数组
    String[] names = {"baidu","ali","tentce"};
    printNames(names);
}
@Test
public void test3(){
    // 传入个数不一定的参数(可以为0个)
    printNames("baidu","ali","tentce");
}
```
其实查看idea反编译后的文件可以发现：
```java
public void printNames(String... names) {
    String[] var2 = names;
    int var3 = names.length;

    for(int var4 = 0; var4 < var3; ++var4) {
        String name = var2[var4];
        System.out.println(name);
    }

}
@Test
public void test2() {
    String[] names = new String[]{"baidu", "ali", "tentce"};
    this.printNames(names);
}
@Test
public void test3() {
    this.printNames("baidu", "ali", "tentce");
}
```
也就是说其实java编译器是将这个可变长度参数当成了一个数组，只是说这个数组在写法上是方便了程序员的。接着再测试一波：
```java
public void printNames(String...names){
    System.out.println(names);
}
public void printNames(String[] names){
    for(String name:names){
        System.out.println(name);
    }
}
```
编译器报错，printNames is already defined in ....。也就是说编译器的确将这个可变长度参数当成了一个数组。另外，我试着将一个可变数组作为实参传入一个形参为数组的方法中：
```java
@Test
public void test4(){
    String[] str = new String[]{"abc","def"};
    test5(str);
}
public void test5(String...str){
    test6(str);
}
public void test6(String[] str){
    for (String s : str) {
        System.out.print(s+",");
    }
}
```
编译器不报错，说明有戏，运行后打印结果：
```java
abc,def,
```
这也就证明了可变参数其实就是数组，只是写法上有些不同而已。