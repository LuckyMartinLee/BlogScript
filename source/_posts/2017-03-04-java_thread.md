---
typora-root-url: ..\post_imgs
---

### java 多线程

#### 创建线程

1.extends Thread 方式

创建一个类去继承Thread类然后重写run方法，在main方法中调用该类实例对象的start方法即可实现多线程并发

``` java
public class MyThread extends Thread {
    // 重写 run 方法，由jvm调用
    @Override
    public void run() {
        System.out.println("子线程在执行 ...");
    }
    
    public static void main(String[] args) {
        Thread thread = new MyThread();
        // 启动线程,线程就绪,系统马上调用 thread.run方法
        thread.start();
        System.out.println("主线程在执行 ...");
    }
}
```

2.implements Runnable 方式

``` java
public class MyRunnable implements Runnable{
    // 实现 run 方法，由jvm调用
    @Override
    public void run() {
        System.out.println("子线程在执行 ...");
    }
    
    public static void main(String[] args) {
        // 这里main中可以看到真正创建新线程还是通过Thread创建
        // 这一步Thread类的作用就是把run()方法包装成线程执行体，然后依然通过start去告诉系统这个线程已经准备好了可以安排执行
        Runnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        // 启动线程,线程就绪，系统马上调用 runnable.run方法
        thread.start();
        System.out.println("主线程在执行 ...");
    }
}
```

注意：

main方法中应该调用的是Thread的start方法，而不是run()方法。

调用start()方法是告诉CPU此线程已经准备就绪可以执行，进而系统有时间就会来执行其run()方法。

如直接调用run()方法，则不是异步执行，而是等同于调用函数般按顺序同步执行，这就失去了多线程的意义了。

3.implements Callable 和 创建 FutureTask 对象方式

上面1，2 两个方式无法解决下面两个问题

- 无法获取子线程的返回值
- run方法不可以抛出异常

Callable并不是Runnable的子接口，是个全新的接口，它的实例不能直接传入给Thread构造函数，需要另一个接口来转换一下，Java提供了 Future 接口来代表 Callable 接口里 call() 方法的返回值，而 FutureTask 类实现了RunnableFuture接口， RunnableFuture接口 extends 了 Future 接口和 Runnable 接口

![](/post_imgs/java_thread_1.jpg)

![](/post_imgs/java_thread_2.jpg)

```java
public class MyCallable implements Callable {
    @Override
    public Object call() throws Exception {
        System.out.println("Callable 子线程在执行 ...");

        return "MyCallable";
    }
    
    public static void main(String[] args) {
        Callable callable = new MyCallable();
        FutureTask task = new FutureTask(callable);

        Thread threadc = new Thread(task);
        // 启动线程
        threadc.start();
        try {
            //获取子线程的返回值 或 捕获子线程异常
            System.out.println("子线程返回值：" + task.get());
        }  catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

注意：

线程的执行顺序与start()的执行顺序无关，而是CPU有空隙了就过来执行该线程，所以执行顺序具有随机性。

小结：

实际开发中可能有更复杂的代码实现，需要继承其他的类，所以平时更推荐通过实现接口来实现多线程，也就是通过第二或第三种方式来实现，这样能保持代码灵活和解耦。
而选择第二还是第三种方式，则要根据run()方法是不是需要返回值或者捕获异常来决定，如果不需要，可以选择用第二种方式实现，代码更简洁。

#### 中止线程

通常情况下，run() 方法执行完毕后，该线程就终止了。但是在某些特殊的情况下，run() 方法会被一直执行，如 run方法中使用了 while 等循环语句，此时就要通过其他方式来 停止线程线程运行

1、设置中止标记位

``` java
@Setter
public class VariableStopRunnable implements Runnable {
    private boolean exit = false;

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " is running ...");

        while (! this.exit) {
            System.out.println("子线程正在执行 ...");
        }
        System.out.println("子线程停止.");
    }

    public static void main(String[] args) throws InterruptedException {
        VariableStopRunnable runnable = new VariableStopRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(100);
        runnable.setExit(true);
        System.out.println("主线程停止.");

    }
}
```

2、利用中断标识

#### 线程状态及转换

![](/post_imgs/java_thread_3.jpg)

1、线程休眠

``` java
// 当前线程进行特定时间的休眠，这段时间线程处于阻塞状态
// 时间单位：毫秒
Thread.sleep(1000);
```



2、线程合并

```java
// 一定是当前线程调用此方法，将其他线程合并到当前线程，等待其他线程执行结束，当前线程再执行，这段时间，当前线程处于阻塞状态
Thread.join();
// 也允许加上超时等待时间，如到了等待时间，其他线程仍然没有结束，那么当前线程不再阻塞，回到可运行状态，参与CPU资源抢占
Thread.join(1000);

```



3、线程礼让

```java
// 一定是当前线程调用此方法，当前线程放弃获取的CPU时间片，下一时刻，立即参与CPU资源抢占，期间线程不会进入阻塞状态
Thread.yeild();
```



### 线程同步

#### synchronized

``` java
// synchronized 同步方法

// synchronized 同步代码块
```

synchronized 同步静态方法与同步非静态方法的区别

#### lock

lock是接口，常用的是 ReentrantLock 重入锁

```java
// ReentrantLock 重入锁
```

synchronized 与 ReentrantLock 比较

1. synchronized  是关键字，jvm 负责处理。ReentrantLock 是类，由 jdk 提供

### JUC

java.util.concurrent : java 1.5 提供的 java 并发开发工具包

### 锁

### Volatile 关键字



### 递归





### 线程池

```java
 public ThreadPoolExecutor(
     int corePoolSize, // 核心线程数，常驻线程数
     int maximumPoolSize, // 线程池最大线程数
     long keepAliveTime, // 动态线程空闲存活最大时间
     TimeUnit unit, // 动态线程空闲存活最大时间单位，超过时间任空闲，线程被销毁
     BlockingQueue<Runnable> workQueue, // 线程任务等待队列，被添加到线程池中，但尚未被执行的任务
     ThreadFactory threadFactory, // 线程工厂类，用于创建线程
     RejectedExecutionHandler handler // 拒绝策略；当任务太多来不及处理时，如何拒绝任务。有下面四种策略
     							   // AbortPolicy ：超过队列长度，丢弃任务，抛异常
     							   // DiscardPolicy ：超过队列长度，丢弃任务，但不抛异常
     							   // DiscardOldestPolicy ：超过队列长度，新任务与队列中最前面的任务竞争线程，竞争失败者，丢弃
     							   // CallerRunsPolicy ：超过队列长度，主线程参与执行任务，
 )
```

