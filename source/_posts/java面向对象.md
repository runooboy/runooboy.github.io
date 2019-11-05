---
title: java面向对象
date: 2019-09-16 13:46:48
category:
- javaSE
tags:
- 多态
- 面向对象
---
什么是面向对象？这大概是java中最重要的一部分了，写这篇文章主要是自己之前java基础不怎么样，然后今天看了一些关于面向对象和多态的文章，做下笔记。
<!-- more -->

## 面向对象
### java中什么是对象
java中万物皆对象，既所有的一切都可以是对象，动物可以是对象，人可以是对象，头也可以是对象... 总之，就是我们可以看到的事物基本上都可以是对象。

### 什么是面向对象
面向对象并不仅仅是这几个字而已，这是一种编程思想，它强调的是具备功能的对象，以对象为基本单位；在实际开发中，可以有效地帮助企业解决项目庞大不好管理的问题；讲到面向对象编程，就不得讲讲面向过程编程；面向过程强调的是功能行为，以函数为最小单位。

#### 面向过程编程和面向对象编程
在这里先讲个把大象放进冰箱的例子，在面向过程编程中，我们的做法是：
1. 打开冰箱
2. 把大象放进冰箱
3. 关闭冰箱

然而在面向对象编程时却不一样了，通常做法是这样的：
1. 创建冰箱类，并声明打开和关闭冰箱的方法
2. 创建大象类，并声明进入冰箱的方法
3. 创建操作者类，并声明将大象放入冰箱、打开和关闭冰箱的方法




## 面向对象三大特征

### 封装
#### 为什么需要封装？封装的作用？
1. 我要用洗衣机，只需要按一下开关，有必要了解洗衣机内部的结构吗？
2. 我要用电脑，我只需要知道怎么用就行了，有必要了解电脑的内部结构吗？
3. 。。。

#### 程序设计一直以来都在追求“高内聚、低耦合”
- 高内聚：类的内部数据细节自己完成，不允许外部干涉
- 低耦合：仅对外暴露少量的方法用于使用

隐藏对象内部的复杂性，只对外公开简单的接口。便于外界调用，从而提高系统的可扩展性、可维护性。通俗的说，把该隐藏的隐藏起来，该暴露的暴露出来，这就是封装性的设计思想。

#### java封装性的体现需要权限修饰符来配合。
四种权限修饰符（从小到大排列）：private、default(缺省)、protected、public

|作用域    | 当前类 | 同一包(package) | 子孙类 | 其他包 |
|-|-|-|-|-|
|public   | Y| Y| Y|Y|
|protected| Y| Y| Y|N|
|default  | Y| Y| N|N|
|private  | Y| N| N|N|

关于抽象类的一些变化：
JDK 1.8以前，抽象类的方法默认访问权限为protected
JDK 1.8时，抽象类的方法默认访问权限变为default
关于接口
JDK 1.8以前，接口中的方法必须是public的
JDK 1.8时，接口中的方法可以是public的，也可以是default的


### 继承
继承体现的是`is a `的关系，比如动物类继承生物类，那么就可以说动物类是一个生物类，反过来则不行。
在java中是没有多继承的，那么有的时候我们需要多继承怎么办呢？java提供了接口机制。
接口：比如鸟继承动物类，可是并不是所有的动物都能飞，而鸟是可以飞的，那么我们就可以自己写一个`Flyable`接口，然后鸟在继承动物类的时候实现这个接口，另外鸟是可以实现多个接口的：
```java
class Bird extends Animal implements Flyable, Eatable{
	// 重写接口中的方法。。。
}
```


### 多态
多态性：可以理解未一个事物的多种形态（或者多种行为）。
对象的多态性：父类的引用指向子类的对象。
多态的使用前提：1. 类的继承；2. 方法的重写

有了对象的多态性以后，我们在编译器，只能调用父类中声明的方法，但在运行期，我们实际执行的的是子类重写父类的方法，总结如下：
> 编译看左面，运行看右面

对象的多态性只适用于方法，不适用于属性（编译和运行都看左边）

#### 陷阱：“重写”私有方法

```java
class Aniaml{
    private void f() {
        System.out.println("private f()");
    }
}
class Cat extends Aniaml{
    public void f() {
        System.out.println("public f()");
    }
}
public class PrivateOverride {
    public static void main(String[] args) {
        Aniaml aniaml = new Cat();
        aniaml.f();
        /**
         * 报错：
         * Error:(22, 15) java: f()可以在com.polymorphism.Aniaml中访问private
         */
    }
}
```
其实这也体现了上面说的方法的多态性为`编译看左面，运行看右面`。
另外，也许你一不小心会尝试这样做：
```java
public class PrivateOverride {
    private void f() {
        System.out.println("private f()");
    }

    public static void main(String[] args) {
        PrivateOverride po = new Derived();
        po.f();
		/**
	     * 打印结果：
	     * private f()
	     */
    }
}
class Derived extends PrivateOverride {
    public void f() {
        System.out.println("public f()");
    }
}
```
你期待的 打印结果是`public f()`，然而 private 方法也是 final 的，对于派生类来说是隐蔽的。因此，这里 Derived 的 f() 是一个全新的方法；因为基类版本的 f() 屏蔽了 Derived ，因此它都不算是重载方法。
结论是只有非 private 方法才能被重写，但是得小心重写 private 方法的现象，编译器不报错，但不会按我们所预期的执行。为了清晰起见，派生类中的方法名采用与基类中 private 方法名不同的命名。

## 面向对象剩余知识点

### 基本数据类型、包装类、String


|基本数据类型    | 大小/字节 | 包装类|
|-|-|-|
|byte   | 1| Byte|
|short| 2| Short|
|char  | 2| Character|
|int  | 4| Integer|
|float  | 4| Float|
|long  | 8| Long|
|double  | 8| Double|
|boolean  | JVM决定| Boolean|

#### 基本数据类型转换为包装类
基本数据类型转换为包装类都可以通过调用包装类的构造器来完成，比如：
```java
Integer in1 = new Integer(10);
Float f1 = new Float(1.23f);
Boolean b1 = new Boolean(true);
```
关于包装类，可以看包装类里面的源码实现。比如：
```java
Boolean b2 = new Boolean("true123");
// b2为false
```
Boolean类中对使用String类型的参数构造器处理方式为：
```java
return ((s != null) && s.equalsIgnoreCase("true"));
```
所有上面的b2为false。


#### 包装类转换为基本数据类型
包装类转换为基本数据类型基本都是通过调用包装类Xxx的xxxValue()完成的，比如：
```java
Integer in1 = new Integer(12);
int i1 = in1.intValue();
System.out.println(i1+11);
/**
 * 打印结果：
 * 23
 */
Float fl1 = new Float(12.3f);
float f1 = fl1.floatValue();
/**
 * 打印结果：
 * 12.3
 */
```
#### 自动装箱和自动拆箱
在jdk5之后，jdk提供类自动装箱和自动拆箱，不再要求程序员像上面一样强行处理基本数据类型，为处理基本数据类型提供了极大地便利：
```java
// 自动装箱
Integer in1 = 10;

// 自动拆箱
int in2 = in1;
System.out.println(in1 + " "+in2);
/**
 * 打印结果：
 * 10 10
 */
```
当然，在这里也有一个小陷阱需要注意一下，就是Boolean和boolean的初始化是不同的，Boolean初始化出来时null，而boolean初始完成后时false：
```java
class D{
    Boolean flag1; //null
    boolean flag2; //false
}
```
#### 基本数据类型、包装类和String之间的相互转换
基本数据类型、包装类转换为String类型有两种方式
1. 使用连接运算：
	```java
	int num1 = 10;
    String str1 = num1+"";
	```
2. 调用String重载的valueOf(Xxx xxx)方法即可。
	```java
	float f1 = 12.3f;
	String str2 = String.valueOf(f1);
	```

String类型转换为基本数据类型、包装类调用包装类的parseXxx(String s)方法
```java
@Test
public void test2(){
    String str1 = "123a";
//  错误的情况 ：
//  int in1 = (int)str1;
//  Integer in2 = (Integer) str1;
    
    // 可能会报NumberFormatException
    int in1 = Integer.parseInt(str1);

    boolean b1 = Boolean.parseBoolean("true1");
}
```

### 类的内部结构——代码块
1. 代码块的作用：初始化类、对象
2. 代码块如果有修饰的话，只能使用static
3. 代码块只要分为静态代码块和非静态代码块

#### 静态代码块
1. 内部可以有输出语句
2. 随着类的加载而执行，而且只执行一次
3. 作用：初始化类的信息
4. 如果一个类中定义了多个静态代码块，则按照声明的先后顺序执行


#### 非静态代码块
1. 内部可以有输出语句
2. 随着对象的创建而执行
3. 每创建一个对象，就执行一次非静态代码块
4. 作用：可以才创建对象时，对对象的属性等进行初始化
5. 如果一个类中定义了多个非静态代码块，则按照声明的先后顺序执行

#### 静态代码块和非静态代码块使用实例
```java
public class BlockTest {
    @Test
    public void test1(){
        System.out.println(Person.desc);

        Person p1 = new Person();
        Person p2 = new Person();
        System.out.println("\n"+p1.getName()+" "+p1.getAge());
        System.out.println(p2.getName()+" "+p2.getAge());
    }
}
class Person{
    private String name;
    private int age;
    static String desc = "这是一个人";
    // 静态代码块
    static {
        System.out.println("static block");
        desc = "在静态进行了修改。。。"+desc;
    }

    // 非静态代码块
    {
        System.out.println("nonstatic block");
        this.name = "dd";
        this.age = 13;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```
打印结果：
```java
static block
在静态进行了修改。。。这是一个人
nonstatic block
nonstatic block

dd 13
dd 13
```

### 类的内部结构——内部类
- 当一个事物的内部，还有一个部分需要一个完整的结果进行描述，而这个内部的完整结构又只为外部事物提供服务，那么整个内部的完整结构最好使用内部类。比如人可以抽象成一个类，而人的脑袋也可以抽象成一个类，这个时候就可以将脑袋声明为人的内部类。
- 在java中，允许一个类的定义位于另一个类的内部，前者称为称为内部类，后者称为外部类。
- Inner class一般用于定义它的类或语句之内，在外部引用它时必须给出完整的名称，Inner class的名字不能与包含它的外部类类名相同；

#### 成员内部类（static成员内部类与非static成员内部类）
一方面，作为外部类的成员具有如下特点：
1. 调用外部类的结构
2. 可以被static修饰
3. 可以被4中不同的权限修饰

另一方面，作为一个类具有如下特点：
1. 类里面可以定义属性、方法、构造器等
2. 可以被final修饰，表示此类不能被继承，言外之意，不使用final，就可以被继承
3. 可以被abstract修饰

通常，我们使用内部类主要关注如下3个问题：
1. 如何实例化成员内部类的对象
2. 如何在成员内部类中区分调用外部类的结构
3. 开发中局部内部类的使用（详细请看下面的[局部内部类](#part)）

关于前面两个问题，可以参考下面的使用：

```java
public class InnerClass {

    @Test
    public void test1(){
        // 实例化非静态内部类
        Animal a1 = new Animal();
        Animal.Head head1 = a1.new Head();
        head1.show();

        // 实例化静态内部类
        Animal.Hand hand = new Animal.Hand();
        hand.f();

        // 外部类有继承关系时，若内部类没有进行重写，则默认还是使用外部类父类的内部类。
        Person p1 = new Person("tom",22);
        Person.Head head2 = p1.new Head();
        head2.thinking();
        Animal.Footer footer = p1.new Footer();
        footer.show();

        p1.change();
        p1.show();

        /**
         * 打印结果：
         * Animal head....
         * animal hand f()...
         * thinking...
         * Animal footer show...
         * null
         * Person{name='eye change tom', age=22}
         */
    }
}

class Animal{
    static class Hand{
        public void f(){
            System.out.println("animal hand f()...");
        }
    }
    class Head{
        public void show(){
            System.out.println("Animal head....");
        }
    }
    class Footer{
        public void show(){
            System.out.println("Animal footer show...");
        }
    }
}
class Person extends Animal{

    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    class Head{
        public void thinking(){
            System.out.println("thinking...");
        }
    }
    class Eye{
        String name;
        public void changeName(){
            System.out.println(this.name);
            // 使用Person.this.xx来调用外部类的属性
            Person.this.name = "eye change "+ Person.this.name;
        }
    }

    public void change(){
        Eye eye = new Eye();
        eye.changeName();
    }
    public void show(){
        System.out.println(this.toString());
    }
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#### <span id="part">局部内部类</span>（方法内，代码块内，构造器内）
局部内部类包含着匿名内部类，其实匿名内部类就是局部内部类的一种简写。
在java中类似下面这种局部内部类的并不常见：
```java
public void method(){
    class AA{
        
    }
}
```

主要的还是下面这种：
```java
// 返回一个实现了Comparable接口的类的对象
public Comparable getComparable(){
    class MyComparable implements Comparable{

        @Override
        public int compareTo(Object o) {
            return 0;
        }
    }
    return new MyComparable();
}
```
上面这种写法可以更加简便，既使用匿名内部类：
```java
// 返回一个实现了Comparable接口的类的对象
public Comparable getComparable(){
    return new Comparable() {
        @Override
        public int compareTo(Object o) {
            return 0;
        }
    };
}
```
而且匿名内部类里面只能调用局部变量，不能修改局部变量的值，但是可以修改外部成员变量的值，使用`OutClass.this.value`访问。



