---
title: java泛型
date: 2019-09-17 09:49:49
category:
- javaSE
tags:
- 泛型
---
### 概述
泛型在java中有很重要的地位，在面向对象编程及各种设计模式中有非常广泛的应用。

什么是泛型？为什么要使用泛型？
<!-- more -->
Java泛型（generics）是JDK 5中引入的一个新特性，允许在定义类和接口的时候使用类型参数（type parameter）。
>泛型就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。
泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

在泛型出现之前，使用容器时我们都是使用Object来作为参数，然后通过向下转型获取我们想要的数据。比如：
```java
@Test
public void test1(){
    List list = new ArrayList();
    list.add("aaaa");
    list.add(100);
    for(int i = 0; i< list.size();i++){
        String item = (String)list.get(i);
        System.out.println(item);
    }
    /**
     * 报错：
     * aaaa
     *
     * java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
     */
}
```
显然此时向下转型失败，因为List中有Integer数据。在没有泛型的情况下，这就要求编程人员对代码的掌控能力十分高，而且还得时刻准备好检查是否可以向下转型。有没有办法处理这种情况呢，我们可以在编写代码的时候创建一个约定，约定好只能向这个容器放入什么类型的数据，在编译的时候编译器如果发现容器中出现了其他类型的数据就报错。

### 编译期有效
上面也说了，泛型只在编译期有效，我们可以测试一下：
```java
@Test
public void test2(){
    List<String> list1 = new ArrayList<String>();
    List<Integer> list2 = new ArrayList<Integer>();

    Class classStringArrayList = list1.getClass();
    Class classIntegerArrayList = list2.getClass();

    if(classStringArrayList.equals(classIntegerArrayList)){
        System.out.println("类型相同");
    }
    /**
     * 打印结果：
     * 类型相同
     */
}
```
通过上面的例子可以证明，在编译之后程序会采取去泛型化的措施。也就是说Java中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段。

对此总结成一句话：泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。


### 泛型的使用

泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法

#### 泛型类
泛型类的最基本的写法如下：
```java
public class Order<T> {
	private T orderNum;
    private int orderLine;
    private String money;
	...
}
```
在上面这个例子中，我们`new`对象的时候传入什么类型，那么orderNum就是什么类型的，比如：
```java
Order<String> order = new Order<>();
```
则此时orderNum就是一个String类型的变量。而如果我们不传类型时，则默认是为Object，那么这个时候就可能出现这种情况：
```java
@Test
public void test3(){

    Order order13 = new Order(13, 11, "99");
    System.out.println(order13.getOrderNum().getClass().getName());

    order13 = new Order("123", 11, "99");
    System.out.println(order13.getOrderNum().getClass().getName());
    /**
     * 打印结果：
     * java.lang.Integer
     * java.lang.String
     */
}
```
另外，泛型的类型参数只能是类类型，不能是简单类型。

接下来看下泛型类在继承时会出现的情况：
```java
class Father<T1, T2>{}

// 子类不保留父类的泛型
// 1） 没有类型 擦除
class Son<A, B> extends Father{} //等价于class Son extends Father<Object, Object>{}

// 2)具体类型
class Son2<A, B> extends Father<Integer, String>{}

//子类保留父类的泛型
// 1)全部保留
class Son3<T1, T2, A, B> extends Father<T1, T2>{}

// 2)部分保留
class Son4<T2, A, B> extends Father<Integer, T2>{}
```


泛型在继承方面的体现：
```java
List<Object> list = null;
List<String> list1 = null;
list = list1; // 编译不通过，IDE直接报错

List<String> stringList = null;
ArrayList<String> stringArrayList = null;
stringList = stringArrayList; // 编译通过
```
结论：
1. 类A是类B的父类，G< A >和G< B >是不相关的类型，既并列的类型
2. 类A是类B的父类，A< G >是B< G >的父类


#### 泛型接口

泛型接口与泛型类的定义及使用基本相同。感觉差不多，这里就不做过多的赘述。

#### 泛型方法

泛型方法跟它所在的类是不是泛型没关系，调用泛型方法时不需要特意指定类型参数的实际类型，编译器可以自动推断。
一个简单的泛型方法例子（主要使用< E > 这个标记来标识泛型方法）：
```java
private final static  <E> boolean check(E e){
    if(e instanceof String){
        return true;
    }else{
        return false;
    }
}
@Test
public void test6(){
    Employee employee = new Employee();
    System.out.println(check(employee));
}
@Test
public void test7(){
    String str = null;
    System.out.println(check(str));
}
```
显然，它是根据传入的参数类型来进行判定的，与方法所在的类是没有关系的。而且泛型方法是可以加上static和final关键字的，而泛型类中的静态方法是不能访问类上定义的泛型的，否则会报错，因为静态方法加载时在类实例化之前的，也就是说泛型T是一个不确定的类型。

总结一下，泛型方法能使方法独立于类而产生变化。下面时在网上看到的一个指导规则：
```java
无论何时，如果你能做到，你就该尽量使用泛型方法。也就是说，如果使用泛型方法将整个类泛型化，

那么就应该使用泛型方法。另外对于一个static的方法而已，无法访问泛型类型的参数。

所以如果static方法要使用泛型能力，就必须使其成为泛型方法。
```

### 泛型通配符以及泛型上下限
经常发现有List<? super T>、Set<? extends T>的声明，是什么意思呢？
- <? super T> 有下限，表示包括T在内的任何T的父类;
- <? extends T> 有上限，表示包括T在内的任何T的子类;

### 关于泛型数组(待更)
```java
List<String>[] ls = new ArrayList<String>[10]; // 不允许
List<?>[] ls1 = new ArrayList<?>[10]; // 允许
List<String>[] ls2 = new ArrayList[10]; // 运行
```

下面使用Sun的一篇文档的一个例子来说明这个问题：
```java
List<String>[] lsa = new List<String>[10]; // Not really allowed.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
String s = lsa[1].get(0); // Run-time error: ClassCastException.
```
这种情况下，由于JVM泛型的擦除机制，在运行时JVM是不知道泛型信息的，所以可以给oa[1]赋上一个ArrayList而不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现ClassCastException，如果可以进行泛型数组的声明，上面说的这种情况在编译期将不会出现任何的警告和错误，只有在运行时才会出错。而对泛型数组的声明进行限制，对于这样的情况，可以在编译期提示代码有类型安全问题，比没有任何提示要强很多。