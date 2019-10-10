---
title: java Stream
date: 2019-09-12 00:06:49
category:
- javaSE
tags:
- java
- stream
---
## java Stream是什么
首先，我们来看下java怎么描述Stream的：
>A sequence of elements supporting sequential and parallel aggregate operations.

其实学习stream之前我以为stream式很难的，但是学完之后，发现其实还好，并不难，主要在于是否熟练。
<!-- more -->
Stream是java8的新特性，它与 java.io 包里的 InputStream和 OutputStream是完全不同的概念.java8中的Stream主要是用于处理java中的数据的。而且Stream只是将原来的数据拷贝一份，然后通过中间操作处理拷贝的数据，最后通过终端操作将得到我们想要的数据。而且Stream是一种惰性加载的。（惰性加载：如果流的操作中没有执行终止操作也不会执行中间操作。）
接下来我们看下Stream的总纲：
![](Stream.jpg)

详细图：

![创建流](创建流.jpg)


![管道](管道.jpg)


![流](流.jpg)

![流和迭代器](流和迭代器.jpg)


### stream 创建
废话不多说，直接先上代码：
```java
// 创建Stream
@Test
public void test1(){
    // 1.通过Collection系列集合提供得stream()或parallelStream()(并行流)
	// 所有的Collection集合都可以通过stream默认方法获取流
    List<String> list = new ArrayList<>(16);
    Stream<String> stream1 = list.stream();

    // 2.通过Arrays中的静态方法stream() 获取数组流
    Employee[] employees = new Employee[10];
    Stream<Employee> stream2 = Arrays.stream(employees);

    // 3. 通过Stream中的静态方法of()，实际上还是数组流
    Stream<String> stream3 = Stream.of("aa","bb","cc");

    // 4. 无限流 (迭代,生成)
    Stream<Integer> stream4 = Stream.iterate(0,(x)->x+2);
    stream4.limit(10).forEach(System.out::println);

    // 5. 使用generate(Supplier<T> s)生成流，需要重写Supplier中的get()方法
    Stream.generate(()->Math.random()*10).limit(5).forEach(System.out::println);

    // 6.创建一个空的stream
    Stream<Integer> stream5  = Stream.empty();
}
```
上面大体上把大多数创建流的方式列出来了，但是我记得之前在csdn上看到一篇博客，上面记载了很多创建流的方式，现在找不到那篇博客了，所以就先把这些方式列出来。

### 中间操作
首先把测试过程中使用到的实体类以及对应的测试数据先整出来：
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
class Employee{
    String name;
    int age;
    int salary;
}
List<Employee> employees = Arrays.asList(
        new Employee("张三", 11, 9000),
        new Employee("李四", 33, 3000),
        new Employee("王五", 44, 4000),
        new Employee("王五", 44, 4000),
        new Employee("王五", 44, 4000),
        new Employee("p8", 44, 4000),
        new Employee("李四", 44, 4000),
        new Employee("p8", 55, 2000)
);
```
1. filter过滤 limit 截断
	```java
	@Test
	public void test2(){
	    // filter过滤
	    // limit 截断
	    employees.stream().filter(x->{
	        System.out.println("执行中间操作");
	        return x.getAge() > 22;
	    }).forEach(System.out::println);
	
	}
	```

2. skip
	```java
	@Test
	public void test3(){
	    // skip(n)，跳过n个元素
	    employees.stream().skip(1).forEach(System.out::println);
	}
	```

3. distinct去重
	```java
	@Test
	public void test4(){
	    // distinct去重:通过流所生成元素的hashCode()和equals()去除重复元素
	    employees.stream().distinct().forEach(System.out::println);
	}
	```

4. map
	```java
	@Test
	public void test5(){
	    // map 接收lambda，将元素转换成其他形式或提取信息。
	    // 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素
	    List<String> list = Arrays.asList("aaa","bbb","ccc","ddd","eee");
	    list.stream().map((str)->str.toUpperCase())
	            .forEach(System.out::println);
	    System.out.println("----------------------------");
	    employees.stream()
	            .map(Employee::getName)
	            .forEach(System.out::println);
	    System.out.println("----------------------------");
	    list.stream().map(StreamTest::filterCharacter).forEach((sm)->{
	        System.out.println("--------第一层----------");
	        sm.forEach(System.out::println);
	    });
	
	}
	```

5. flatMap,基本与map功能相同，只是可以把一个二位map整成一个一维的，多维的可不可以弄成一维的还没有测试，后面可以试下。

	```java
	@Test
	public void test6(){
	    List<String> list = Arrays.asList("aaa","bbb","ccc","ddd","eee");
	    // flatMap 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有的流连接成一个流。
	    list.stream().flatMap(StreamTest::filterCharacter).forEach(System.out::println);
	}
	public static Stream<Character> filterCharacter(String str){
	    List<Character> list = new ArrayList<>();
	    for(Character ch:str.toCharArray()){
	        list.add(ch);
	    }
	    return list.stream();
	}
	```

6. sorted,默认排序
	```java
	@Test
	public void test7(){
	    // sorted()-默认排序
	    List<String> list = Arrays.asList("aa","dd","cc","a");
	    list.stream().sorted().forEach(System.out::println);
	}
	```

7. sorted，重写Comparator中的compare方法
	```java
	@Test
	public void test8(){
	    // sorted-自定义排序
	    employees.stream().sorted((e1, e2)->{
	        if(e1.getAge() == e2.getAge()){
	            return e1.getName().compareTo(e2.getName());
	        }else{
	            return Integer.compare(e1.getAge(),e2.getAge());
	        }
	    }).forEach(System.out::println);
	}
	```

### 终止操作（终端操作）
- allMatch() 匹配查询集合中是否都符合条件，返回true/false
- anyMatch() 匹配查询集合中是否存在符合条件的，返回true/false
- count() 统计集合中数量
………………

在使用idea可以直接看到stream中的最终操作方法，也能一眼就看明白是要干什么的操作，所以就不一一写到笔记了。

### 惰性加载
stream中的惰性加载就是在没有执行最终操作的情况下，所有的中间操作也不会执行。
```java
@Test
public void test13(){
    List<String> list = new ArrayList<String>();

    list.add("1");

    list.add("2");

    list.add("3");

    list.stream().filter(x -> {
        System.out.println(x);
        return true;
    });

}
```
上面这段代码并不会产生任何的输出，原因是filter只刻画了stream，但是并没有产生新的集合，而像这种没有实际功能，只是描述stream的操作，就叫做惰性求值。而如果在filter之后加上count()（最终操作）则会执行print操作，这种操作叫做及早求值：
```java
@Test
public void test13(){
    List<String> list = new ArrayList<String>();

    list.add("1");

    list.add("2");

    list.add("3");

    list.stream().filter(x -> {
        System.out.print(x+" ");
        return true;
    }).count();
    // 打印结果： 1 2 3 
}
```
惰性加载和及早求值区别：
- 对流对象Stream进行惰性求值，返回值仍然是一个Stream对象。
- 对流对象Stream进行及早求值，返回值不再是一个Stream对象。

## Stream 并发流
测试Stream并发流主要是通过计算1到50亿的累加，如下：
```java
long s = 0L;
long e = 5000000000L;
```

### 普通for循环
使用最基础的for去计算累加，如下：
```java
@Test
public void test10(){
    Instant start = Instant.now();
    long sum = 0L;
    for(long i = s; i < e; i++) {
        sum += i;
    }
    System.out.println(sum);
    Instant end = Instant.now();
    System.out.println(Duration.between(start,end).toMillis());
}
```
具体时间就不写了，有兴趣的可以自己试下，下面两个也是一样。
### fork-join框架
使用fork-join框架先要重写好RecursiveTask中的compute方法，在compute方法中进行fork()和join()操作。
```java
public class ForkJoinSum extends RecursiveTask<Long> {
    private static final long serialVersionUID = 100000000L;
    private long start;
    private long end;
    private final long THRESHHOLD = 10000L;

    public ForkJoinSum(long start, long end){
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if(end - start <= THRESHHOLD){
            long sum = 0L;
            for(long i = start; i < end; i++) {
                sum += i;
            }
            return sum;
        } else {
            long mid = (end+start)/2;
            ForkJoinSum left = new ForkJoinSum(start, mid);
            left.fork();

            ForkJoinSum right = new ForkJoinSum(mid+1, end);
            right.fork();

            return left.join()+right.join();

        }
    }
}
```
然后再来测试：
```java
@Test
public void test9(){
    // 使用fork，join框架
    Instant start = Instant.now();
    ForkJoinSum forkJoinSum = new ForkJoinSum(s,e);
    System.out.println(forkJoinSum.compute());
    Instant end = Instant.now();
    System.out.println(Duration.between(start, end).toMillis());
}
```

### stream并发
使用stream并发流计算累加：
```java
@Test
public void test11(){
    // 流式并行
    Instant start = Instant.now();
    OptionalLong optionalLong = LongStream.rangeClosed(s,e)
            .parallel()
            .reduce(Long::sum);
    System.out.println(optionalLong.getAsLong());
    Instant end = Instant.now();
    System.out.println(Duration.between(start,end).toMillis());
}
```
### 小结
虽然上面的几个测试我没有写结果，但是我自己之前测的时候结果是：数据量较小的时候直接使用for累加花费的时间较短，另外两个所用时间都较多，而数据量较大的时候使用并发计算则所用时间较短，但是使用for累加所用时间是成倍数上升的。而stream流计算并发和fork-join框架计算所用时间是差不多的。这里只是使用了.parallel()来生成并发流，使用集合的parallelStream()也可以生成并发流的。

## Optional
什么是Optional，我们先来看下java是怎么讲的：
>A container object which may or may not contain a non-null value.

显然，Optional是一个容器类，主要解决的问题是臭名昭著的空指针异常（NullPointerException）。当然也可以用于装载经过stream操作后的数据（可能stream后的数据为null），比如：
```java
Optional<A> firstA= AList.stream() .filter(a -> "张三".equals(a.getUserName())) .findFirst();
```
那Optional是如何解决NPE的呢？我们先来看一个小例子：
```java
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
```
如上，在不想要触发NPE的情况下，我们需要在访问每一个值之前都进行明确的检查。而Optional提供了一个`ifPresent(Consumer<? super T> consumer)`方法，ifPresent方法允许我们传入一个消费者进去，让消费者替我们完成拿到数据之后的操作。
其实有些用法我现在也还没有搞明白，后续待更。。。