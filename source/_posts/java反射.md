---
title: java反射
date: 2019-09-29 19:55:27
category:
- javaSE
tags:
- 反射
---
在说反射之前，必须说一句，所谓java框架大多数都是`动态代理+注解`完成的，而动态代理必须使用反射完成。而反射可以获取本类、父类以及实现的接口中的所有方法以及所有属性。
### 获取运行时类的实例
获取运行时类得实例主要有四种方式：
```java
@Test
public void test4() throws ClassNotFoundException {
    /**
     * 获取运行时类的实例
     */
    // 调用运行时类的属性
    Class<Person> clazz1 = Person.class;
    System.out.println(clazz1);

    // 通过运行时类的对象调用getClass()
    Person person = new Person();
    Class<? extends Person> clazz2 = person.getClass();
    System.out.println(clazz2);

    // 调用Class的静态方法：forName(String classPath)
    Class clazz3 = Class.forName("com.reflect.Person");
    System.out.println(clazz3);

    System.out.println(clazz1 == clazz2);
    System.out.println(clazz1 == clazz3);

    // 使用类的加载器
    ClassLoader classLoader = ReflectTest.class.getClassLoader();
    Class clazz4 = classLoader.loadClass("com.reflect.Person");
    System.out.println(clazz4);
    System.out.println(clazz1 == clazz4);
}
```
另外，关于java.lang.Class类：
1. 类的加载过程：
程序经过javac.exe命令后，会生成一个或者多个字节码文件(以.class结尾)
接着使用java.exe命令后对某个字节码文件进行解释运行，相当于将某个字节码文件加载到内存中，此过程称为类的加载。加载到内存中的类，称之为运行时类，此运行时类，作为Class的一个实例
2. 换句话说，Class的实例就对应这一个运行时类
3. 加载到内存中的运行时类，会缓存一定的时间，在此时间之内，我们可以通过不同的方式来获取此运行时类,在其生命周期类，这个运行时类也可以说是一个单例的

### 反射调用私有属性和私有方法
在使用反射之前，由于java的封装性原因，我们是访问不到私有属性和私有方法的，但是如果使用反射的话是可以访问到私有属性和私有方法的,通过。
```java
@Test
public void test3() throws Exception{
    Class clazz = Person.class;
    // 通过反射可以调用Person类的私有接口，比如私有构造器、私有方法和私有属性
    // 调用私有构造器
    Constructor constructor2 = clazz.getDeclaredConstructor(String.class);
    // 私有构造器、私有方法和私有属性必须使用setAccessible打开权限
    constructor2.setAccessible(true);
    System.out.println("begin....");
    //此时通过对应的构造器创建实例
    Object jim = constructor2.newInstance("jim");
    System.out.println(jim);
    Person person =(Person)jim;
    person.t2("iop");
    System.out.println("********************");

    // 调用私有属性
    Field name = clazz.getDeclaredField("name");
    name.setAccessible(true);
    name.set(jim, "som");
    System.out.println(jim);
    System.out.println("*************************8");

    // 调用私有方法
    Method t1 = clazz.getDeclaredMethod("t1", String.class);
    t1.setAccessible(true);

    Object dd = (String)t1.invoke(jim, "dd");
    System.out.println(dd);
}
```
在使用私有属性和私有方法的时候必须开放权限：`constructor2.setAccessible(true);`
问1：通过直接new的方式或反射的方式都可以调用公共结构，开发中该使用哪个？
建议：直接new的方式
什么时候会使用反射的方式：编译的时候不能确定应该new哪个对象（可以参考servlet、spring）

问2：反射时可以访问内部资源和java的封装性是否有冲突
不冲突，因为java的封装性只是说建议这样子做，而反射式提供了一种访问内部资源的方式

### 代理模式

java中代理模式分为静态代理和动态代理，其中动态代理就是通过反射完成的。代理模式的主要思想就是有些类不好直接做某些操作，于是就通过代理类来完成。静态代理是每一个类如果想要使用代理模式，必须有一个代理类；动态代理则是有了明显的进步：所有的类使用一个代理类。
#### 静态代理
![](proxy.gif)

实现代码如下：
```java
/**
 * 公共实现的接口，（主题）
 */
interface ClothFactory{
    void produceCloth();
}

/**
 * 代理类
 */
class ProxyClothFactory implements ClothFactory{

    private ClothFactory clothFactory;

    public ProxyClothFactory(ClothFactory clothFactory){
        this.clothFactory = clothFactory;
    }

    @Override
    public void produceCloth() {
        System.out.println("代理工厂开始准备工作。。。");
        clothFactory.produceCloth();
        System.out.println("代理工厂做后续工作。。。");
    }
}

/**
 * 被代理类
 */
class NikeClothFatory implements ClothFactory{

    @Override
    public void produceCloth() {
        System.out.println("NikeClothFatory生产工作。。。");
    }
}
public class StaticProxyTest{
    public static void main(String[] args) {
        // 创建被代理对象
        NikeClothFatory nikeClothFatory = new NikeClothFatory();

        // 创建代理对象
        ProxyClothFactory proxyClothFactory = new ProxyClothFactory(nikeClothFatory);
        proxyClothFactory.produceCloth();
    }
}
```

#### 动态代理
![](proxy.gif)
动态代理思想：所有的被代理类使用一个代理类去完成工作。要想实现动态代理，必须要解决的问题：
1. 如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象
2. 当通过代理类的对象调用方法的时候，如何动态的去调用被代理类中的同名方法。

实现代码如下：
```java
interface Human{
    String getBelief();
    void eat(String food);
}
class SuperMan implements Human{

    @Override
    public String getBelief() {
        return "i believe i can!";
    }

    @Override
    public void eat(String food) {
        System.out.println("你吃"+food);
    }
}

/**
 * 动态代理思想：所有的被代理类使用一个代理类去完成工作。
 * 要想实现动态代理，必须要解决的问题：
 * 1. 如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象
 * 2. 当通过代理类的对象调用方法的时候，如何动态的去调用被代理类中的同名方法。
 */
class ProxyFactory{
    // 动态的创建一个代理类及其对象，解决第一个问题
    // obj 为代理的对象
    public static Object getProxyInstance(Object obj){
        MyInvocationHandler myInvocationHandler = new MyInvocationHandler();
        myInvocationHandler.bind(obj);
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), myInvocationHandler);
    }
}
class MyInvocationHandler implements InvocationHandler{

    // 需要使用被代理类的对象进行赋值
    private Object obj;

    public void bind(Object obj){
        this.obj = obj;
    }

    /**
     * 当通过代理类的对象调用方法a时，就会自动调用invoke()方法
     * 将被代理类要执行的方法a的功能声明在invoke()中
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // method 为代理类对象调用的方法， 此方法也就作为了被代理类对象要调用的方法
        // obj 被代理的对象
        Object invoke = method.invoke(obj, args);
        // method.invoke方法的返回值就作为当前类中的invoke()
        return invoke;
    }
}


public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        // proxyInstance代理类的对象
        Human proxyInstance = (Human)ProxyFactory.getProxyInstance(superMan);
        System.out.println(proxyInstance.getBelief());
        proxyInstance.eat("西瓜");
    }
}
```
关于动态代理原理可以参考https://www.jianshu.com/p/95970b089360
