---
title: java多线程编程
date: 2019-10-07 20:47:12
category:
- javaSE
tags:
- java
- 多线程
- 并发
---

## 概念梳理

**并行**
多个线程在同一时间在CPU上运行。

**并发**
表示一会做这个事情，一会做另一个事情，存在着调度。单核 CPU 不可能存在并行（微观上）。

<!-- more -->

**临界区**
临界区用来表示一种公共资源或者说是共享数据，可以被多个线程使用。但是每一次，只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源，就必须等待。

**阻塞与非阻塞**
阻塞和非阻塞通常用来形容多线程间的相互影响。比如一个线程占用了临界区资源，那么其它所有需要这个资源的线程就必须在这个临界区中进行等待，等待会导致线程挂起。这种情况就是阻塞。

此时，如果占用资源的线程一直不愿意释放资源，那么其它所有阻塞在这个临界区上的线程都不能工作。阻塞是指线程在操作系统层面被挂起。阻塞一般性能不好，需大约8万个时钟周期来做调度。

非阻塞则允许多个线程同时进入临界区。

**死锁**
死锁是进程死锁的简称，是指多个进程循环等待他方占有的资源而无限的僵持下去的局面。

**活锁**
假设有两个线程1、2，它们都需要资源 A/B，假设1号线程占有了 A 资源，2号线程占有了 B 资源；由于两个线程都需要同时拥有这两个资源才可以工作，为了避免死锁，1号线程释放了 A 资源占有锁，2号线程释放了 B 资源占有锁；此时 AB 空闲，两个线程又同时抢锁，再次出现上述情况，此时发生了活锁。

**饥饿**
饥饿是指某一个或者多个线程因为种种原因无法获得所需要的资源，导致一直无法执行。

## 生命周期
![](java线程生命周期.jpg)

**创建状态**
当用 new 操作符创建一个新的线程对象时，该线程处于创建状态。
处于创建状态的线程只是一个空的线程对象，系统不为它分配资源。

**就绪状态**
执行线程的 start() 方法使线程处于就绪状态。

**运行状态**
调度为线程分配必须的系统资源，安排其运行，并调用线程体——run()方法，这样就使得该线程处于可运行状态（Runnable）。

**休眠状态**
- 调用了 sleep() 方法；

**阻塞状态**
当发生下列事件时，处于运行状态的线程会转入到阻塞状态：

- 当一个线程试图获取一个内部的对象锁（非java.util.concurrent库中的锁），而该锁被其他线程持有，则该线程进入阻塞状态。
- 当一个线程等待另一个线程通知调度器一个条件时，该线程进入等待状态。例如调用：Object.wait()、Thread.join()以及等待Lock或Condition。
- 线程输入/输出阻塞；
- 线程调用 wait() 方法等待特定条件的满足；

- 如果线程在等待某一条件，另一个对象必须通过 notify() 或 notifyAll() 方法通知等待线程条件的改变；
- 如果线程是因为输入输出阻塞，需要等待输入输出完成。


## 线程的创建
java在jdk5之前只提供继承Thread和实现Runnable方式来创建线程，在jdk5之后则新增了两个方法来创建线程。
### 继承Thread方式

```java
/**
 * 通过继承于Thread类
 * 重写run()方法，业务逻辑代码放在run()中。
 */

class Thread1 extends Thread{
    public long times;
    public Thread1(long times){
        this.times = times;
    }
    @Override
    public void run(){
        while (times >= 0) {
            System.out.println(Thread.currentThread().getName()+"***** times:"+times--);
        }
    }
}

// main中调用：
public static void main(String[] args) {
        Thread1 thread1 = new Thread1(5);
        thread1.start();

//        不能直接通过调用run()的方式直接启动线程,此时仍处于主线程(main)中。
//        thread1.run();

//         多次通过start()启动线程会报IllegalThreadStateException
//         thread1.start();

        int times = 5;
        while (times >= 0) {
            System.out.println(Thread.currentThread().getName()+"***** times:"+times--);
        }
    }
```
打印结果：
```java
main***** times:5
Thread-0***** times:5
Thread-0***** times:4
Thread-0***** times:3
main***** times:4
Thread-0***** times:2
main***** times:3
main***** times:2
main***** times:1
main***** times:0
Thread-0***** times:1
Thread-0***** times:0
```

### 实现Runnable接口方式

```java
/**
 * 实现Runnable接口方式，
 * 重写run()方法，业务逻辑代码放在run()中。
 * 调用的时候需要new Thread(传入实现了Runnable接口的实体类对象)，比如：new Thread(p).start();
 */
class Thread3 implements Runnable{
    public long times;
    public Thread3(long times){
        this.times = times;
    }
    @Override
    public void run() {
        while (times >= 0) {
            System.out.println(Thread.currentThread().getName()+"***** times:"+times--);
        }
    }
}

// main中调用：
public static void main(String[] args) {
    Thread3 thread3 = new Thread3(5);
    new Thread(thread3).start();

    int times = 5;
    while (times >= 0) {
        System.out.println(Thread.currentThread().getName()+"***** times:"+times--);
    }
}
```

打印结果：
```java
main***** times:5
Thread-0***** times:5
main***** times:4
Thread-0***** times:4
main***** times:3
main***** times:2
main***** times:1
main***** times:0
Thread-0***** times:3
Thread-0***** times:2
Thread-0***** times:1
Thread-0***** times:0
```


### 实现Callable接口方式
Callable接口源码：
```java
/**
 * A task that returns a result and may throw an exception.
 * Implementors define a single method with no arguments called
 * {@code call}.
 *
 * <p>The {@code Callable} interface is similar to {@link
 * java.lang.Runnable}, in that both are designed for classes whose
 * instances are potentially executed by another thread.  A
 * {@code Runnable}, however, does not return a result and cannot
 * throw a checked exception.
 *
 * <p>The {@link Executors} class contains utility methods to
 * convert from other common forms to {@code Callable} classes.
 *
 * @see Executor
 * @since 1.5
 * @author Doug Lea
 * @param <V> the result type of method {@code call}
 */
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

```java
/**
 * 实现Callable接口方式，
 * 重写call()方法，将线程中需要执行的业务逻辑代码放在run()中。
 */
// 1.创建一个实现Callable的类
class Thread4 implements Callable<Integer>{
    // 2.实现call()，将线程中需要执行的业务逻辑代码放在run()中。
    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for(int i = 0; i <= 5; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName()+"***** i:"+i);
                sum+=i;
            }
        }
        return sum;
    }
}

// main中调用：
public static void main(String[] args) {
    // 3.创建Callable接口实现类的对象
    Thread4 thread4 = new Thread4();
    // 4.将此Callable接口实现类的对象作为参数传递到FutureTask构造器中，创建FutureTask的对象
    FutureTask futureTask = new FutureTask(thread4);
    // 5.将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象并调用start()
    new Thread(futureTask).start();
    int times = 5;
    while (times >= 0) {
        System.out.println(Thread.currentThread().getName()+"***** times:"+times--);
    }
    try {
        // 6.获取Callable中call方法的返回值
        // get()返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值。
        Object o = futureTask.get();
        System.out.println(o);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

打印结果：

```java
main***** times:5
Thread-0***** i:0
main***** times:4
main***** times:3
main***** times:2
main***** times:1
main***** times:0
Thread-0***** i:2
Thread-0***** i:4
6
```

### 使用线程池
main中调用，Thread3和Thread4同上

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    // 只适合Runnable方式,
    executorService.execute(new Thread3(5));

    // 适合Callable方式和Runnable方式,返回Future对象
    Future<Integer> submit = executorService.submit(new Thread4());
    try {
        System.out.println(submit.get());
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }finally {
        executorService.shutdown();
    }
}
```

打印结果：

```java
pool-1-thread-1***** times:5
pool-1-thread-1***** times:4
pool-1-thread-1***** times:3
pool-1-thread-1***** times:2
pool-1-thread-1***** times:1
pool-1-thread-1***** times:0
pool-1-thread-2***** i:0
pool-1-thread-2***** i:2
pool-1-thread-2***** i:4
6
```

### 四种创建方式比较

通过继承Thread和实现Runnable接口创建线程的方式比较简单，只要注意将需要进行逻辑处理的代码方法重写的`run()`就好。通过实现Callable接口和使用线程池来创建线程是jdk5之后的新方式。

**实现Callable接口**

与使用Runnable相比，Callable功能更强大些
- 可以有返回值，FutureTask的对象的get()可以获取重写的`call()`的返回值
- 方法可以抛出异常
- 支持泛型的返回值
- 需要借助FutureTask类，比如获取返回结果

**使用线程池**
背景： 经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大
思路：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁，实现重复利用。类似生活中的公共交通工具。
好处：
- 提高响应速度（减少了创建新线程的时间）
- 降低资源消耗（重复利用线程池中线程，不需要每次都创建）
- 便于线程管理

Java中创建线程池很简单，只需要调用Executors中相应的便捷方法即可，比如Executors.newFixedThreadPool(int nThreads)，但是便捷不仅隐藏了复杂性，也为我们埋下了潜在的隐患（OOM，线程耗尽）。

```java
// Java线程池的完整构造函数
public ThreadPoolExecutor(
  int corePoolSize, // 线程池长期维持的线程数，即使线程处于Idle状态，也不会回收。
  int maximumPoolSize, // 线程数的上限
  long keepAliveTime, TimeUnit unit, // 超过corePoolSize的线程的idle时长，
                                     // 超过这个时间，多余的线程会被回收。
  BlockingQueue<Runnable> workQueue, // 任务的排队队列
  ThreadFactory threadFactory, // 新线程的产生方式
  RejectedExecutionHandler handler) // 拒绝策略
```
详细可以查看ThreadPoolExecutor类源码。也可以参考https://www.cnblogs.com/CarpenterLee/p/9558026.html

## 线程安全问题
假设有3个窗口在同时卖50张火车票，通过如下代码来实现：

```java
class Window extends Thread{
    private static int tickets = 50;
    // 用来计算run()总共跑了多少次
    public static int sum = 0;
    @Override
    public void run() {
        while (tickets > 0){
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":卖票，票号为："+tickets--);
            sum++;
        }
    }
}

public static void main(String[] args) {
        Window window1 = new Window();
        Window window2 = new Window();
        Window window3 = new Window();
        window1.setName("窗口1");
        window2.setName("窗口2");
        window3.setName("窗口3");


        window1.start();
        window2.start();
        window3.start();
        try {
            sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Window.sum);
    }
```
最后几条打印结果：

```java
窗口1:卖票，票号为：3
窗口3:卖票，票号为：2
窗口1:卖票，票号为：1
窗口3:卖票，票号为：1
窗口2:卖票，票号为：0
72
```
结果发现3个窗口总共卖了72次，而且其中有多张票号有问题（重票和负数、0号票）的票，原因是多个线程可能同时执行下面代码：
```java
System.out.println(Thread.currentThread().getName()+":卖票，票号为："+tickets--);
```
而且这些线程持有的tickets相同，所有就会出现重票，这就是线程安全问题。

### 同步代码块解决线程安全问题

#### 解决通过继承Thread类创建的多线程程序引起的安全问题

```java
class Window extends Thread {
    private static int tickets = 50;
    // 用来计算run()总共跑了多少次
    public static int sum = 0;

    @Override
    public void run() {
        while (true) {
            synchronized (Window.class) {
                if(tickets > 0){
                    try {
                        sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets--);
                    sum++;
                }else {
                    break;
                }
            }
        }
    }
}

public static void main(String[] args) {
        Window window1 = new Window();
        Window window2 = new Window();
        Window window3 = new Window();
        window1.setName("窗口1");
        window2.setName("窗口2");
        window3.setName("窗口3");


        window1.start();
        window2.start();
        window3.start();
        try {
            sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Window.sum);
    }
```

打印结果：

```java
窗口1:卖票，票号为：50
窗口1:卖票，票号为：49
...
窗口2:卖票，票号为：4
窗口2:卖票，票号为：3
窗口2:卖票，票号为：2
窗口2:卖票，票号为：1
50
```

#### 解决通过实现Runnable接口创建的多线程程序引起的安全问题

```java
class Window2 implements Runnable {
    private int tickets = 50;
    public static int sum = 0;

    @Override
    public void run() {

        while (true) {
            synchronized (this) {
                if (tickets > 0) {
                    try {
                        sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets--);
                    sum++;
                } else {
                    break;
                }
            }
        }

    }
}

public static void main(String[] args) {
         Window2 window = new Window2();
         Thread window1 = new Thread(window);
         Thread window2 = new Thread(window);
         Thread window3 = new Thread(window);

         window1.setName("窗口1");
         window2.setName("窗口2");
         window3.setName("窗口3");

         window1.start();
         window2.start();
         window3.start();

         try {
         sleep(6000);
         } catch (InterruptedException e) {
         e.printStackTrace();
         }
         System.out.println(Window2.sum);
    }
```

打印结果：

```java
窗口2:卖票，票号为：50
窗口2:卖票，票号为：49
...
窗口3:卖票，票号为：3
窗口3:卖票，票号为：2
窗口3:卖票，票号为：1
50
```

### 同步方法解决线程安全问题

#### 解决通过继承Thread类创建的多线程程序引起的安全问题

```java
class Window extends Thread {
    private static int tickets = 50;
    // 用来计算run()总共跑了多少次
    public static int sum = 0;

    @Override
    public void run() {
        while (true) {
            if(!compute()){
                break;
            }
        }
    }
    private static synchronized boolean compute(){
        if(tickets > 0){
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets--);
            sum++;
            return true;
        }
        return false;
    }
}
```

打印结果：

```java
窗口1:卖票，票号为：50
窗口3:卖票，票号为：49
...
窗口2:卖票，票号为：3
窗口2:卖票，票号为：2
窗口2:卖票，票号为：1
50
```
#### 解决通过实现Runnable接口创建的多线程程序引起的安全问题

```java
class Window2 implements Runnable {
    private int tickets = 50;
    public static int sum = 0;

    @Override
    public void run() {
        while (true) {
            if(!compute()){
                break;
            }
        }
    }
    private synchronized boolean compute(){
        if(tickets > 0){
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets--);
            sum++;
            return true;
        }
        return false;
    }
}
```

打印结果：

```java
窗口1:卖票，票号为：50
窗口1:卖票，票号为：49
...
窗口3:卖票，票号为：3
窗口3:卖票，票号为：2
窗口3:卖票，票号为：1
50
```

### Lock解决线程安全问题
jdk5之后，

#### 解决通过继承Thread类创建的多线程程序引起的安全问题
```java
class Window2 implements Runnable {
    private int tickets = 50;
    public static int sum = 0;
    private ReentrantLock lock = new ReentrantLock(true);
    @Override
    public void run() {
        while (true) {
            try {
                lock.lock();
                if(tickets > 0){
                    try {
                        sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets--);
                    sum++;
                }else {
                    break;
                }
            }finally {
                lock.unlock();
            }

        }
    }
}
```

打印结果：

```java
窗口2:卖票，票号为：50
窗口1:卖票，票号为：49
窗口3:卖票，票号为：48
...
窗口2:卖票，票号为：2
窗口1:卖票，票号为：1
50
```


#### 解决通过实现Runnable接口创建的多线程程序引起的安全问题

```java
class Window extends Thread {
    private static int tickets = 50;
    // 用来计算run()总共跑了多少次
    public static int sum = 0;
    private static ReentrantLock lock = new ReentrantLock(true);

    @Override
    public void run() {
        while (true) {
            try{
                lock.lock();
                if(tickets > 0){
                    try {
                        sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets--);
                    sum++;
                } else {
                    break;
                }
            }finally {
                lock.unlock();
            }

        }
    }
}
```

打印结果：

```java
窗口1:卖票，票号为：50
窗口2:卖票，票号为：49
...
窗口3:卖票，票号为：3
窗口1:卖票，票号为：2
窗口2:卖票，票号为：1
50
```
## 线程通信

### 生产者消费者问题

生产者-消费者模式是一个十分经典的多线程并发协作的模式，所谓生产者-消费者问题，实际上主要是包含了两类线程，一种是生产者线程用于生产数据，另一种是消费者线程用于消费数据，为了解耦生产者和消费者的关系，通常会采用共享的数据区域，就像是一个仓库，生产者生产数据之后直接放置在共享数据区中，并不需要关心消费者的行为；而消费者只需要从共享数据区中去获取数据，就不再需要关心生产者的行为。但是，这个共享数据区域中应该具备这样的线程间并发协作的功能：

- 如果共享数据区已满的话，阻塞生产者继续生产数据放置入内；
- 如果共享数据区为空的话，阻塞消费者继续消费数据；

实现案例：
```java
class Clerk{
    private int sum = 0;
    public synchronized void pruduce(){
        if(sum < 20){
            sum++;
            System.out.println(Thread.currentThread().getName()+"******生产第"+sum+"个数据");
            notify();
        }else {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public synchronized void comsumer(){
        if(sum > 0){
            System.out.println(Thread.currentThread().getName()+"******消费第"+sum+"个数据");
            sum--;
            notify();
        }else {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
class Productor implements Runnable{

    Clerk clerk;
    public Productor(Clerk clerk){
        this.clerk = clerk;
    }
    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            clerk.pruduce();
        }
    }
}

class Comsumer implements Runnable{

    Clerk clerk;
    public Comsumer(Clerk clerk){
        this.clerk = clerk;
    }
    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            clerk.comsumer();
        }
    }
}
public class ComsumerTest {
    public static void main(String[] args) {
        Clerk clerk = new Clerk();
        Productor productor = new Productor(clerk);
        Comsumer comsumer = new Comsumer(clerk);

        Thread threadProductor = new Thread(productor);
        Thread threadComsumer = new Thread(comsumer);

        threadProductor.setName("Productor");
        threadComsumer.setName("Comsumer");
        threadProductor.start();
        threadComsumer.start();
    }

}
```

## 死锁以及死锁避免

### 死锁的定义
所谓死锁是指多个线程因竞争资源而造成的一种僵局（互相等待），若无外力作用，这些进程都将无法向前推进。
例如，某计算机系统中只有一台打印机和一台输入 设备，进程P1正占用输入设备，同时又提出使用打印机的请求，但此时打印机正被进程P2 所占用，而P2在未释放打印机之前，又提出请求使用正被P1占用着的输入设备。这样两个进程相互无休止地等待下去，均无法继续执行，此时两个进程陷入死锁状态。

### 死锁产生原因

**系统资源的竞争**

通常系统中拥有的不可剥夺资源，其数量不足以满足多个进程运行的需要，使得进程在运行过程中，会因争夺资源而陷入僵局，如磁带机、打印机等。只有对不可剥夺资源的竞争才可能产生死锁，对可剥夺资源的竞争是不会引起死锁的。

**进程推进顺序非法**

 进程在运行过程中，请求和释放资源的顺序不当，也同样会导致死锁。例如，并发进程 P1、P2分别保持了资源R1、R2，而进程P1申请资源R2，进程P2申请资源R1时，两者都会因为所需资源被占用而阻塞。

 Java中死锁最简单的情况是，一个线程T1持有锁L1并且申请获得锁L2，而另一个线程T2持有锁L2并且申请获得锁L1，因为默认的锁申请操作都是阻塞的，所以线程T1和T2永远被阻塞了。导致了死锁。这是最容易理解也是最简单的死锁的形式。但是实际环境中的死锁往往比这个复杂的多。可能会有多个线程形成了一个死锁的环路，比如：线程T1持有锁L1并且申请获得锁L2，而线程T2持有锁L2并且申请获得锁L3，而线程T3持有锁L3并且申请获得锁L1，这样导致了一个锁依赖的环路：T1依赖T2的锁L2，T2依赖T3的锁L3，而T3依赖T1的锁L1。从而导致了死锁。

### 死锁产生条件

**互斥条件**
进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
通俗点讲就是一个资源每次只能被一个进程使用。独木桥每次只能通过一个人。

**不剥夺条件**
进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能由获得该资源的进程自己来释放（只能是主动释放)。
通俗点讲就是一个进程因请求资源而阻塞时，对已获得的资源保持不放。乙不退出桥面，甲也不退出桥面。

**请求和保持条件**
进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
通俗点讲就是进程已获得的资源，在未使用完之前，不能强行剥夺。甲不能强制乙退出桥面，乙也不能强制甲退出桥面。

**循环等待条件**
循环等待条件：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有。
通俗点讲就是若干进程之间形成一种头尾相接的循环等待资源关系。如果乙不退出桥面，甲不能通过，甲不退出桥面，乙不能通过。

### 死锁产生实例
```java
class Dead implements Runnable{

    StringBuffer s1 = new StringBuffer();
    StringBuffer s2 = new StringBuffer();

    @Override
    public void run() {
        synchronized(s1){
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            s1.append("ab");
            synchronized(s2){
                s2.append("cd");
            }
        }
        synchronized (s2){
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            s2.append("ab");
            synchronized(s1){
                s1.append("cd");
            }
        }
        System.out.println(s1+"\n"+s2);
    }
}
public static void main(String[] args) {
    Dead dead = new Dead();
    new Thread(dead).start();
    new Thread(dead).start();
}
```


### 如何避免死锁
在有些情况下死锁是可以避免的。下面介绍三种用于避免死锁的技术：
- 加锁顺序（线程按照一定的顺序加锁）
- 加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）
- 死锁检测

**1、加锁顺序**
当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。上面的那个例子就是加锁顺序不当导致的。
避免嵌套封锁：这是死锁最主要的原因的，如果你已经有一个资源了就要避免封锁另一个资源。如果你运行时只有一个对象封锁，那是几乎不可能出现一个死锁局面的。

**2、加锁时限**
 另外一个可以避免死锁的方法是在尝试获取锁的时候加一个超时时间，这也就意味着在尝试获取锁的过程中若超过了这个时限该线程则放弃对该锁请求。若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。这段随机的等待时间让其它线程有机会尝试获取相同的这些锁，并且让该应用在没有获得锁的时候可以继续运行(加锁超时后可以先继续运行干点其它事情，再回头来重复之前加锁的逻辑)。

**3、死锁检测**
死锁检测是一个更好的死锁预防机制，它主要是针对那些不可能实现按序加锁并且锁超时也不可行的场景。主要可以参考[银行家算法](https://baike.baidu.com/item/%E9%93%B6%E8%A1%8C%E5%AE%B6%E7%AE%97%E6%B3%95/1679781)的思想。



