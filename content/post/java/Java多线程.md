---
title: "Java多线程"
aliases: Java
categories: Java
tags: [Java]
date: 2024-07-02
time: 15:50
--- 
## 线程

### 几个概念
| 并行 | 并发
- **并行(parallel)**
    多个任务同时操作多个资源，每个任务独立执行，互不影响
- **并发(concurrent)**
    多个任务同时操作同一资源，多个线程交替执行

| 进程 | 线程 | 管程 
- **进程(process)**
    操作系统上任务执行的最小单元，一个服务就是一个进程。放在Java里，启动一个程序就是一个进程。

- **线程(thread)**
    线程是比进程更小的执行单元，一个进程内包含n多个线程

- **管程monitor**
    可以理解为锁

### 线程状态
> `线程分为 用户线程和守护线程（Damon=true） 生命周期随进程周期 如 GC 线程`

- **`NEW`**：新建状态，未运行
- **`RUNNABLE`**：可执行状态{运行状态，待运行}
- **`BLOCKED`**：阻塞状态 等待锁，然后执行
- **`WAITING`**：无限期等待
- **`TIMED_WAITING`**：限时等待
- **`TERMINATED`**：终止状态

![线程状态变换](/img/java/thread-state.png)


### 线程的创建

**1. 继承 Thread    实际上 Thread 也是实现了 Runnable 接口**

```java
public class MyThread extends Thread {
    
    void run () {
        // 执行体
    }
}
// 创建并运行
new MyThread().start();
// 简易写法
Thread a = new Thread(r -> {
    // 执行体
})；

// 运行
a.start();   

```

**2. 实现 Runnable  无返回值**

```java
public class MyRunnable implements Runnable {
    
    void run（）{
        // 执行体
    }
}

// 调用
new Thread(new MyRunnable()).start();

```

**3. 实现 Callable   有返回值 ，获取返回值会阻塞|抛出异常**

```java
public class MyCallable implements Callable{
    
    V call（）{
        // 执行体
        return null;
    }
}

// 调用
MyCallable call = new MyCallable ();
new Thread(call).start();

```

### run() 与 start() 区别

- **run**:  只是方法的执行体
- **start**: 创建一个新的线程，会执行 run（）

### 常用操作

**线程停止**

    1. 正常停止，即执行完run()方法
    2. 设置一个标志位，暴露public方法停止
        a. `volatile boolean flag`
        b. `AtomicBoolean flag`
        c. 中断操作  `thread.interrupt()`
    3. 不建议使用jdk提供的`stop()` 或 `destory()` 方法


**中断机制**

- 概念：停止线程的协商机制
- 中断标志位： interrupt=true。发起一个协商，而不是立即停止线程
- 常用方法：
    ```java
    // 将标志位设置为true，线程处于阻塞状态时，会抛出异常，且标志位会置为false
    void interrupt()

    // 1. 判断当前线程是否已经中断，并返回中断状态
    // 2. 若线程已是中断状态，则清空状态位，并设为false
    Thread.interrupt()

    // 返回中断状态位，线程正常停止的话返回false
    boolean isInterrupted()

    ```

**线程的等待与唤醒**

```java
// synchronized 
wait()
nofity()
notifyAll()

// lock  unlock 块中
lock.newCondition().await()
lock.newCondition().singal()
lock.newCondition().singalAll()

// LockSupport
LockSupport.park(Thread thread)
LockSupport.unpark()

```

### 线程安全

**原子性**
    同一个操作不能被中途打断，类似事务，要么全部完成，要么全不完成

**可见性**
    有一个线程变更了共享变量，主线程或其他线程需要知道变量已经变更

**有序性**
    指令重排问题， 编译器编译代码的过程中，会对代码执行顺序重排
    
### Future

传统的创建线程方式都无法获取到异步执行结果，通过实现Callback接口，并用Future可以来接收多线程的执行结果。
Future表示一个可能还没有完成的异步任务的结果，针对这个结果可以添加Callback以便在任务执行成功或失败后作出相应的操作。

![Future类图](/img/java/future-class.png)


**Future主要方法**

![Future主要方法](/img/java/future-interface-method.png)

> **`FutureTask<T>`**

能用来包装一个Callable或Runnable对象，因为它实现了Runnable接口，而且它能被传递到Executor进行执行。为了提供单例类，这个类在创建自定义的工作类时提供了protected构造函数。

```java
FutureTask<String> task = new FutureTask<>(Callable<V> callable);

FutureTask<String> task = new FutureTask<>(Runnable runnable, V result);
// 会阻塞线程
task.get()
```
> **`SchedualFuture`**

这个接口表示一个延时的行为可以被取消。通常一个安排好的future是定时任务SchedualedExecutorService的结果

> **`CompleteFuture`**

一个Future类是显示的完成，而且能被用作一个完成等级，通过它的完成触发支持的依赖函数和行为。当两个或多个线程要执行完成或取消操作时，只有一个能够成功。

```java
// 1.无返回值

//   1.1 不指定线程池，使用默认线程池 ForkJoinPool.commonPool
CompletableFuture.runAsync(Runnable runnable)
//   1.2 自定义线程池
CompletableFuture.runAsync(Runnable runnable, Excutor excutor)

// 2. 有返回值
//  2.1 不指定线程池，使用默认线程池 ForkJoinPool.commonPool
CompletableFuture.supplyAsync(Supplier<U> supplier)
//  2.2 自定义线程池
CompletableFuture.supplyAsync(Supplier<u> supplier, Excutor excutor)


// 3结果处理 有返回值
//    3.1 串行。入参分别是上一步的处理结果，一旦发生异常，直接断路 进入异常处理方法
.thenApply(Function<? super T,? extends U> fn)
//    3.2 handle 是依次执行，串行。入参分别是上一步的处理结果。发生异常，并未发生断路，而是继续执行其余的handle方法和whenComplete方法 
.handle(BiFunction<? super T, Throwable, ? extends U> fn)
//    3.3 
.whenComplete(BiConsumer<? super T, ? super Throwable> action)
//    3.4 handle发生异常时并未进入该方法执行. whenComplete发生异常会调用执行异常处理
.exceptionally(Function<Throwable, ? extends T> fn)


// 4结果处理  无返回值
/**
*   1. 可以看到串行方法上一步若无返回值，则下一个方法入参为null，即结果丢失* 
*   2. consumer类型方法都无返回值，或许再接其他方法，上面的执行结果会丢失* 
*   3. consumer类型方法应在supplier类型方法之后执行 个人总结
*/
.thenAccept(Function<? super T,? extends U> fn)

// 5. 执行速度选择 
//   5.1有2任务future1和future2  ApplyToEither对比哪个任务先完成，则执行后续 fn
future1.applyToEither(future2, Function<? super T, U> fn)
future1.applyToEitherAsync(future2, Function<? super T, U> fn)
// 6. 结果合并
//    6.1 有2任务future1和future2  合并方法时，哪个任务先完成，则等待其余任务完成后执行合并任务 fn
future1.thenCombine(future2, BiFunction<? super T,? super U,? extends V> fn)
future1.thenCombineAsync(future2, BiFunction<? super T,? super U,? extends V> fn)


```
> **`ForkJoinTask`**

基于任务的抽象类，可以通过ForkJoinPool来执行。一个ForkJoinTask是类似于线程实体，但是相对于线程实体是轻量级的。大量的任务和子任务会被ForkJoinPool池中的真实线程挂起来，以某些使用限制为代价。

### JMM

    Java 内存模型 CPU 和内存的桥梁


> **`概念`**

JMM 是一种抽象的概念，并不真实存在，描述的是一种规范或约束。通过这个规范定义了程序中多线程下各线程之间变量的读写访问，并决定一个线程对共享变量的写入什么时候对另外的线程可见。

> **`共享变量`**

共享变量是存在主内存中的，多线程下，访问共享变量，新的线程将会创建一个变量的副本，各自独立。若要更新主内存中共享变量的值，主要是将自己副本中的值写会主内存中。

    不同线程之间变量是独立的，不能直接访问，都需要通过主内存。

> **`Happen-before 约定，本质上可见性`**

一个线程内，程序执行得满足约定的顺序，预期的结果。

> **`关键点`**

    1. 原子性
    2. 可见性
    3. 有序性

> **`volatile关键字`**

1. **作用**： 保证了有序性和可见性，不保证原子性
2. **可见性**： 某一线程对 volatile 修饰的变量更改，会立马同步到主内存中
3. **有序性**： volatile 修饰的变量的操作，能锁定某些代码的重排

    ![有序性](/img/java/thread-orderliness.png)
4. **原子性**： 多线程下可能发生写丢失

> **`常用场景`**

  - 线程标志位
  - 多线程下的单例 DCL 单例（double check ）
  - 低消耗的读 同步写

> **`内存屏障`**

即线程对资源变更的一种保护机制。java 内存模型的重排规则要求 java 编译器在生成 jvm 指令时插入特定的内存屏障指令，通过这些【屏障指令】，volatile 实现了可见性和有序性。

  1. volatile 写之前的操作，都禁止重排序到 volatile 之后
  2. volatile 读之后得操作，都禁止重排序到 volatile 之前
  3. volatile 写之后 volatile 读，禁止重排序

    内存屏障之前的所有写操作都要回写到主内存
    内存屏障之后的读操作都能获得内存屏障之前的所有写操作的最新结果

 > **`分类`**

- 粗分
    - 读屏障 load memory barrier ：告诉处理器在写屏障之前，将所有存储在缓存 store buffer 中的数据同步到主内存
    - 写屏障 store memory barrier:
- 细分
    - Load-load
    - Load-store
    - Store-store
    - Store-load

![内存屏障](/img/java/memory-barrier.png)


### CAS(Compare And Swap)

> **`概念`**

比较并替换，当且仅当预期值与内存中值相同时，更新为新值；非阻塞的原子操作 硬件保证

    CAS（V， A， B）  参数：V 内存地址； A 旧的预期值； B 新值

![CAS](/img/java/cas.png)

> **`非阻塞的原子操作(硬件保证)`**

底层使用 Unsafe 类，如 compareAndSwapInt 方法，底层使用汇编 Atomic::cmpchg 命令,保证了其是原子操作

> **`自旋`**

多线程下，跟获取锁类似，需要先获取到资源，才可执行 +1 操作。若没获取到，则自旋一次，再次尝试，直到成功。

例 `new AtomicInteger.getAndIncrement()`

![自旋示例](/img/java/getAndIncrement.png)


> **`自旋锁`**

详见锁篇章

> **`缺点`**

  - 自旋带来的资源浪费
  - ABA 问题（偷梁换柱）：解决方案---带版本号判断， AtomicStampRefrence

> **`ABA 问题`**

- 产生原因：Compare 比较值和替换结果，CAS 只检查最终的结果，而不关心中间的过程，中间过程中发生了什么不清楚

- 举例

        目的 CAS（1， 3） ，中间出现 CAS（1， 0）  → CAS（0， 2） → CAS（2， 1）
        解决方案， 加版本号对比

> **`原子类 java.util.consurrent.atomic 包下类`**

![常见原子类](/img/java/juc-atomic.png)

### 工具类

> **`LockSupport`**

    线程阻塞工具类，本身就持有锁，最多一个许可证，不会累计 | 单一开关式
    
```java
// 发放许可证
LockSupport.unpark(thread);

// 获取通行证
LockSupport.park();
LockSupport.park(thread);

```
> **`Semaphore`**

    计数信号量  维护一组许可证|  坑位抢占式

```java
// 每个人都 acquire 会阻止，直到获得许可证，然后拿走它。
// 每个都 release 增加了一个许可证，可能会释放一个阻止的收购方。
// 但是，没有使用实际的许可对象;只是 Semaphore 保留可用数量的计数并采取相应的行动。
 private final Semaphore available = new Semaphore(MAX_AVAILABLE, true);
// 通常用于限制可以访问某些（物理或逻辑）资源的线程数

// 从信号量获取许可，阻塞，直到一个信号量可用或线程 中断。
void acquire()
// 释放许可证，将其返回到信号量。
void release()
```

> **`CyclicBarrier`**

    循环屏障  | 分片处理型

```java
// 等到 各方 都援用这个 await 屏障
int await()

// 所有线程都到达await()方法
CyclicBarrier barrier = new CyclicBarrier(7, ()-> {
    System.out.println("所有parties都完成了，该结束了");
});
```

> **`CountDownLatch`**

    一种同步辅助工具，它允许一个或多个线程等待，直到在其他线程中执行的一组操作完成。

```java
// 阻塞  等待计数器减到0
// 计数器减到0 或 超时
boolean await(long timeout, TimeUnit unit)
// 一直阻塞，直到计数器减为0
// 使当前线程等待，直到闩锁倒计时为零，除非线程中断。
void await()

// 递减闩锁的计数，如果计数达到零，则释放所有等待的线程。
// 如果当前计数大于零，则递减。如果新计数为零，则重新启用所有等待线程以进行线程调度。
// 如果当前计数等于零，则不会发生任何反应。
// 计数器减1
void countDown()

```

> **`BlockingQueue`**

    阻塞队列

```java
// 不阻塞，返回异常
add()  添加元素到队列，若队列已满，则抛出异常
remove() 获取并删除元素，若独立已空，则抛出异常

// 不阻塞
offer() 若添加失败，则返回false
poll()  获取头元素，获取失败则返回null


// 阻塞
put()  添加时若队列已满，则一直等待
take() 获取时，若队列已空，则一直等待

```

## 锁

### 什么是锁

### 锁-锁的是什么 8大锁

### 分类及概念

### 分布式锁


## 线程安全的集合

## 常用线程类

## 线程池

### 概念

### 线程池状态

### 线程池创建方法

### 常见线程池



