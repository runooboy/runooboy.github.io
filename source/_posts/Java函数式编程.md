---
title: Java函数式编程
date: 2019-09-07 20:57:34
category:
- javaSE
tags:
- java
- lambda
---
## lambda
### 初识lambda表达式
一个简单的方法，比较两个Integer：
```java
public void Test1() {
	Comparator<Integer> com = new Comparator<Integer>() {
		@Override
		public int compare(Integer o1, Integer o2) {
			return Integer.compare(o1, o2);
		}
	};
	TreeSet<Integer> treeSet = new TreeSet<>(com);
}

```
下面的这个方法与上面的方法效果相同：
```java
public void Test2() {
	Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
	TreeSet<Integer> treeSet = new TreeSet<>(com);
}
```
从上面就可以看出lambda表达式可以极大地简化代码量
现在有一个需求，找出所有年龄大于35的员工,Employee的实体类属性如下
<!-- more -->
```java
String name;
int age;
int salary;
```
创建一个List保存数据：
```java
List<Employee> employees = Arrays.asList(
		new Employee("张三", 11, 9000),
		new Employee("李四", 33, 3000),
		new Employee("王五", 44, 4000),
		new Employee("p8", 55, 2000)
);
```
一开始，我们也许会选择一种最简单的方法去实现：
```java
@Test
public List<Employee> Test3() {
	List<Employee> list = new ArrayList<>();
	for (Employee employee : employees) {
		if (employee.age > 35) {
			list.add(employee)
		}
	}
    return list;
}
```
#### 优化一、使用策略设计模式
##### 自己新建实现类来过滤员工功能
然后有了一个新的需求，找出所有员工中工资高于3500的员工，此时就需要加方法了，然后也许还会有别的需求（姓李的员工）等，我们不可能每次加需求都要加方法，于是这时候可以采用策略设计模式，设计一个接口MyInterface,里面声明一个用于处理过滤员工的方法：
```java
public interface MyInterface<T> {    
    boolean filter(T t);
}
```
编写一个专门用于过滤员工的方法：
```java
/**
 * 用于过滤 Employee
 *
 * @param list
 * @param myInterface
 * @return
 */
List<Employee> filterEmployees(List<Employee> list, MyInterface<Employee> myInterface) {
    List<Employee> emp = new ArrayList<>();
    for (Employee employee : list) {
        if (myInterface.filter(employee)) {
            emp.add(employee);
        }
    }
    return emp;
}
```
之后如果需要使用员工过滤的方法时，调用filterEmployees方法时传入MyInterface的具体实现类，这个实现类中重写好所需要的filter方法。
##### 使用内部匿名类来实现过滤员工功能
我们此时在调用的方法中就是传入一个匿名内部类，重写好我们所需要的方法即可：
```java
@Test
public void test4() {
    List<Employee> list = filterEmployees(employees, new MyInterface<Employee>() {

        @Override
        public boolean filter(Employee employee) {
            if (employee.age > 22) {
                return true;
            }
            return false;
        }
    });
    for (Employee employee : list) {
        System.out.println(employee.name + " " + employee.age + " " + employee.salary);
    }
}
```

#### 优化二、使用lambda表达式
lambda表达式其实就是简化了使用匿名内部类的写法繁琐
```java
@Test
public void test5() {
    List<Employee> list = filterEmployees(employees, (employee -> employee.age > 22));
    for (Employee employee : list) {
        System.out.println(employee.name + " " + employee.age + " " + employee.salary);
    }
}
```
### lambda原理
在讲almbda原理之前先说下函数式接口FunctionalInterface，FunctionalInterface就是接口中只有一个抽象方法的接口，此时可以使用注解@FunctionalInterface，如果注解的接口内部有多个抽象方法，编译器会报错。
那么这和lambda有什么关系呢，思考一下，如果使用lambda来替代一个匿名内部类的写法，我们使用lambda的时候是没有指明使用实现接口的哪个方法的，也就是说如果接口内有多个抽象方法，编译器就不知道lambda想要重写哪个方法，这个时候自然会报错。
注意事项：
1. 保持Lambda表达式简短和一目了然，过长的Lambda表达式通常是危险的，因为代码越长越难以读懂，意图看起来也不明，并且代码也难以复用，测试难度也大。
2. 使用@FunctionalInterface注解，否则他人修改了函数式接口后使用lambda的地方就用不了了。
3. 不要在Lambda表达中执行有"副作用"的操作，"副作用"是严重违背函数式编程的设计原则，比如在forEach操作里面操作外面的某个List或者设置某个Map这其实是不对的。
4. 不要把Lambda表达式和匿名内部类同等对待。lambda和匿名内部类是有区别的，主要区别在于匿名内部类中的this和lambda中的this指代的是不同的实例。匿名内部类中的this指向当前匿名内部类的实例（且匿名内部类中的this只能指向内部类的实例，不能指向所在类的实例），而lambda中的this指向所在类的实例，测试如下：
	```java
	private String value = "Enclosing scope value";
	@Test
	public void Test6() {
	    int num = 333;
	    MyInterface<Employee> myInterface = new MyInterface<Employee>() {
	
	        String value = "Inner class value";
	        @Override
	        public boolean filter(Employee employee) {
	            System.out.println("resultIC(this):"+this.value+num);
	            System.out.println("resultIC(normal):"+value+num);
	            return false;
	        }
	    };
	
	    Employee employee = employees.get(0);
	    myInterface.filter(employee);
	
	    MyInterface<Employee> myInterface1 = employee1 -> {
	        String value = "Lambda value";
	        System.out.println("resultLambda(normal):"+value+num);
	        System.out.println("resultLambda(this):"+this.value+num);
	        this.value = "show";
	        return true;
	    };
	    System.out.println("this.value(before filter):"+this.value);
	    myInterface1.filter(employee);
	    System.out.println("this.value(after filter):"+this.value);
	}
	打印结果：
	resultIC(this):Inner class value333
	resultIC(normal):Inner class value333
	this.value(before filter):Enclosing scope value
	resultLambda(normal):Lambda value333
	resultLambda(this):Enclosing scope value333
	this.value(after filter):show
	
	```
5. 多使用方法引用， 在Lambda表达式中 a -> a.toLowerCase()和String::toLowerCase都能起到相同的作用，但两者相比，后者通常可读性更高并且代码会简短。
6. 尽量避免在Lambda的方法体中使用{}代码块：
	```java
	优先使用
	Foo foo = parameter -> buildString(parameter);
	private String buildString(String parameter) {
	    String result = "Something " + parameter;
	    //many lines of code
	    return result;
	}
	而不是
	Foo foo = parameter -> { String result = "Something " + parameter;
	    //many lines of code
	    return result;
	};
	```

### lambda语法
lambda表达式的语法主要可以参考[github on java8](https://github.com/LingCoder/OnJava8/blob/master/docs/book/13-Functional-Programming.md)

### java8四大内置核心函数式接口

#### 消费型接口(Consumer<T>)
消费型接口源码：
```java
/**
 * Represents an operation that accepts a single input argument and returns no
 * result. Unlike most other functional interfaces, {@code Consumer} is expected
 * to operate via side-effects.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #accept(Object)}.
 *
 * @param <T> the type of the input to the operation
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
}
```
从上述注释中可以看出，消费型接口就是接收一个参数，然后利用这个参数完成一系列操作。
消费型接口使用：
```java
public void happy(double money, Consumer<Double> consumer){
    consumer.accept(money);
}
@Test
public void test1(){
    happy(10, (m)-> System.out.println(m));
    // 打印：10.0
}
@Test
public void test5(){
	// 与使用lambda表达式的效果相同
    happy(10, new Consumer<Double>() {
        @Override
        public void accept(Double aDouble) {
            System.out.println(aDouble);
        }
    });
}
```
我们必须有一个方法A调用了消费型接口的`accept`方法，然后运行的时候调用方法A并传入一个消费者对象（可以通过匿名内部类完成），使用lambda则是简化了这段代码。

#### 供给型接口(Supplier<T>)
供给型接口源码：
```java
/**
 * Represents a supplier of results.
 *
 * <p>There is no requirement that a new or distinct result be returned each
 * time the supplier is invoked.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #get()}.
 *
 * @param <T> the type of results supplied by this supplier
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```
供给型接口在`get`方法完成一系列操作后返回一个结果。
供给型接口使用：
```java
public List<Integer> getNumList(int length, Supplier<Integer> integerSupplier){
        List<Integer> list = new ArrayList<>(length);
        for(int i = 0; i < length; i++){
            list.add(integerSupplier.get());
        }
        return list;
    }
    @Test
    public void test2(){
        List<Integer> list = getNumList(10, ()->(int)(Math.random()*100));
        for(Integer in:list){
            System.out.print(in + " ");
        }
        // 打印：41 13 90 13 23 54 69 98 22 25
    }
```
和消费者接口一样，必有有一个方法A调用供给型接口中的`get`方法，然后调用这个方法A并提供一个供给型接口参数。
#### 函数型接口(Function<T>)
函数型接口源码：
```java
/**
 * Represents a function that accepts one argument and produces a result.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #apply(Object)}.
 *
 * @param <T> the type of the input to the function
 * @param <R> the type of the result of the function
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}

```
函数型接口使用：
```java
@Test
public void test3(){
    System.out.println(strHandler("test", (s)->s.toUpperCase()));
    // 打印：TEST
}

public String strHandler(String str, Function<String, String> function){
    return function.apply(str);
}
```


#### 断言型接口(Predicate<T>)
断言型接口源码：
```java
/**
 * Represents a predicate (boolean-valued function) of one argument.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #test(Object)}.
 *
 * @param <T> the type of the input to the predicate
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
```
断言型接口使用：
```java
@Test
public void test4(){
    List<String> list = Arrays.asList("Hello", "Lambda", "Go", "java");
    list = filterStr(list, (s)->s.contains("o"));
    for(String str:list){
        System.out.print(str+" ");
    }
    // 打印：Hello Go
}
public List<String> filterStr(List<String> list, Predicate<String> predicate){
    List<String> stringList = new ArrayList<>(16);
    for(String str: list){
        if(predicate.test(str)){
            stringList.add(str);
        }
    }
    return stringList;
}
```

## 方法引用
### 什么是方法引用
> 若lambda体中的内容已经有方法已经实现了，那么就可以使用方法引用

其实方法引用只是在lambda的基础上进一步简化编程的繁重工作，上面这句话就很好地说明了这一点，在使用lambda的过程中，也许我们会自己实现所需要的方法，但是如果已经有实现好的方法时，我们直接调用就好了，此时java就使用一种叫做方法引用的东西来简化调用的代码。
使用方法引用主要有如下形式：
- 引用静态方法	`Class::staticMethodName`
- 引用某个对象的实例方法	`object::instanceMethodName`
- 引用某个类型的任意对象的实例方法	`type::methodName`
- 引用构造方法	`ClassName::new`

### 引用某个对象的实例方法
废话不多说，直接看代码：
```java
@Test
public void test1(){
    // 使用lambda
    PrintStream ps = System.out;
    Consumer<String> con = (x)->ps.println(x);
    con.accept("haha");

    // 使用方法引用（对象::实例方法名）
    PrintStream ps1 = System.out;
    Consumer<String> con1 = ps::println;
    con1.accept("dd");

    Consumer<String> con2 = System.out::println;
    con2.accept("iop");
    /**
     * 打印结果：
     * haha
     * dd
     * iop
     */
}
```
上面的代码就展示了从lambda到方法引用的改变过程。当然，这里只是引用某个对象的实例方法。接下来进行一点改变：

```java
@Test
public void test2(){
    Employee employee = new Employee("李四",33,24);
    Supplier<String> supplier = ()-> employee.getName();
    System.out.println(supplier.get());
    // 对象::实例方法名
    Supplier<Integer> supplier1 = employee::getAge;
    System.out.println(supplier1.get());
    /**
     * 打印结果：
     * 李四
     * 33
     */
}
```
上面的test2()使用供应者接口，通过供应者接口拿到一个Integer，再输出到cmd，使用的还是 **对象::实例方法名 **。

### 引用静态方法
```java
@Test
public void test3(){
    Comparator<Integer> comparator = (x,y)->Integer.compare(x,y);

    // 类名::静态方法名
    Comparator<Integer> comparator1 = Integer::compare;

    System.out.println(comparator.compare(10,11));
    System.out.println(comparator1.compare(10,11));
    /**
     * 打印结果：
     * -1
     * -1
     */
}
```

上面的test3()是通过 `Integer::compare` 来生成一个Comparator的实例，再调用这个实例的compare方法来获取比较的结果，并将其输出到打印台。
### 引用某个类型的任意对象的实例方法
这个引用讲道理我还不会用，主要是看到网上有人说有这个，自己对这些的研究还不够，暂时先写到这里，后面有机会再来更新一下。

### 构造器引用
构造器引用，顾名思义，其实就是通过`类名::new`来获取一个引用，过程中使用了构造器。
```java
@FunctionalInterface
// 取名这么随意主要是测试了一些今天看到的泛型使用时泛型的名称随意使用
interface MyFunction<T,U,R,J>{
    J apply(T t,U u,R r);
}
@Test
public void test5(){
    Supplier<Employee> supplier = ()->new Employee("dd",22,33);
    System.out.println(supplier.get());
    // 其中new使用的Employee构造器与函数式接口中传入的相同，如下，
    // Supplier<Employee>只是说明要获取一个Employee对象，但是没有传入参数，则调用的为无参构造器
    Supplier<Employee> supplier1 = Employee::new;
    System.out.println(supplier1.get());
    System.out.println("-------------");

    // Function<Integer, Employee> function = Employee::new;
    // 由于使用了lombok，这里就不能这样写了，得传入三个参数，然后返回一个Employee
    // 于是自定义了一个函数式接口，传入三个参数，返回一个值（这里返回Employee）
    MyFunction<String, Integer, Integer, Employee> function = Employee::new;
    System.out.println(function.apply("dd",33,900));
    /**
     * 打印结果：
     * Employee(name=dd, age=22, salary=33)
     * Employee(name=null, age=0, salary=0)
     * -------------
     * Employee(name=dd, age=33, salary=900)
     */
}
```
其实很多在上面代码中的注释中已经写清楚了，这里就不再赘述。
这里还有学到的一个例子，如下
```java
@Test
public void test6(){
    Function<Integer, String[]> function = (x)->new String[x];
    String[] strings = function.apply(10);
    System.out.println(strings.length);

    Function<Integer, Integer[]> function1 = Integer[]::new;
    System.out.println(function1.apply(20).length);
    /**
     * 打印结果：
     * 10
     * 20
     */
}
```
这个test中使用的是`Integer[]::new`，而不是普通的`类名::new`，所以我觉得还是有必要拿出来看看的，以后使用的时候可以参考下。
## 总结
函数式编程就和vim一样，是一个熟能生巧的东西，结合Stream可以极大地方面编程，所以以后还是要多练多使用。



