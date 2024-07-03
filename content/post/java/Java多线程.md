---
title: "Java多线程基础篇"
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

`主要解决问题：串行 并行  数据安全问题`

### 什么是锁

多线程环境下，存在资源抢占问题，可能出现多个线程同时访问同一资源而导致的数据不一致或异常情况，为了保证共享资源的安全性，就出现了锁。

`锁是用于控制多个线程对共享资源访问的机制。`

### 锁-锁的是什么 8大锁

**锁类模版**

```java
public class A {
    // 静态方法 使用了 synchronized修饰，  则锁住的的是类模板 
    public synchronized static void method() {
        System.out.println("");
    } 
}  
```

```java
public class A {
    // 静态方法 
    public static void method() {
        // 此处锁的也是类模板
        synchronized(this){
            System.out.println("此处锁的也是类模板");
        }
    } 
}  
```

**锁方法的调用对象**

```java
public class A {
    // 普通方法 使用了 synchronized修饰，  则锁住的的是方法的调用对象  即类的实例对象 
    public synchronized void method() {
        System.out.println("");
    } 
} 

A a = new A();
// a调用锁a
a.method();

A b = new A();
// b调用锁b
b.method();
```

**锁变量**

```java
public class A {

    private int index = 0;
    
    public void method() {
        // 此处锁住的是index变量
        synchronized(index) {
            // 代码块中执行对index的操作
            System.out.println("");
        }
    } 
} 
```

**锁对象**

```java
public class A {
    
    public void method() {
        // 此处锁住的是this对象，即 A的实例
        synchronized(this) {
            // 代码块中执行对index的操作
            System.out.println("");
        }
        ....
    } 
} 
```


### 锁分类及概念

> **按线程是否阻塞可分为 `有锁（悲观锁）` 和 `无锁（乐观锁）`**

> **按线程是否共享分为 `共享锁（读锁）` 和 `排它锁（写锁）`**

> **按竞争性可分为 `公平锁` 和 `非公平锁`**

> **按锁状态划分为 `偏向锁` | `轻量级锁` | `重量级锁`**

> **按递归性 分为 `递归锁（可重入锁）`**

### 问题

加锁会影响性能，使并行改为串行。因此在实际的使用中，根据场景的不同，要合理的控制锁的粒度，尽量减少性能的开销，线程的阻塞等。这个过程，也就是锁的优化，JDK7 以上自带锁的膨胀与消除，即在加锁的情况下，编译器会自动根据实际情况进行锁的消除或膨胀，如不会产生竞争的情况加锁了，会自动消除锁，避免性能损耗；


### 分布式锁

> **`以上锁均为本地锁，即在同一个 JVM 中有效，而在不同的 JVM 中无效`**

在集群环境中，就需要使用分布式锁来保证资源的统一

分布式锁详见 [分布式锁](/post/tool/2022-02-15-distributed-lock) 

此处做简单总结

目前常用的分布式锁 
1. 基于 redis 的 set NX 的简易分布式锁。
原理：利用 redis 的 set NX（当不存在的时才设置成功，存在则不成功） 特性，实现简易的分布式锁。
缺点： 
  1. 不可重入：
  2. 不可重试：无法重试，加锁失败即刻返回
  3. 超时释放问题：业务卡死或者服务器宕机，锁一直无法释放，卡死了
  4. 主从一致问题：主节点宕机，从节点转变为主，锁数据还未同步到该从节点，锁丢失
可在获取锁之后在设置超时时间，但不是原子操作了。
2. Redisson 基于 redis 的分布式锁

实现原理也是基于 redis 的原子操作，使用 lua 脚本实现。
它的功能更加完善，主要有以下优点：
1. 对锁设置了自动过期时间，避免了因为服务器宕机而导致的锁无法释放问题。
2. 采用哨兵机制看门狗超时续约，1 中对锁设置了过期时间，而过期时间的设置又与业务代码的执行时间有关，假若业务代码执行时长大于锁的过期时间，会造成锁的提前释放；使用看门狗模式，就避免了这种情况。看门狗原理：看门狗在每隔一段时间会检测业务代码是否执行完，若没完则续期过期时间，知道业务代码执行完毕。此时也有问题，若业务代码出现异常卡死，则会造成死锁，锁一直无法释放，这时会有一个最大等待时间，过了这个时间，锁同样也会自动释放。
3. 可以设置最大等待时间，过期锁自动释放。即 2 中最后提到的问题。
4. 实现了可重入功能：使用 hash 结构，存储了重入次数及当前线程标识，同一线程每获取一次锁，重入次数加一。释放一次减一。到 0 就是彻底释放了。
5. 实现了可重试功能：利用消息订阅与信号量机制，在约定时间内多次获取锁。
6. multiLock:解决了主从一致问题。同时向多个节点获取锁，所有节点获取成功，才算锁成功。


## 线程安全的集合

**juc下的类**

- `BlockingQueue`
- `BlockingDeque`
- `LinkedBlockingQueue`
- `ConcurrentLinkedQueue`
- `ConcurrentHashMap`
- `CopyOnWriteArrayList`
- `CopyOnWriteArraySet`


## 常用线程类

```java
FutureTask<T>
CompletabledFuture<T>
CyclicBarrier
CountDownLatch
LockSupport
Semaphore

BlockingQueue

Exchanger
```

## 线程池

### 概念

### 线程池状态

### 线程池创建方法

**7大参数**

```java
ThreadPoolExcutor pool = ThreadPoolExecutor(
                          int corePoolSize,  // 线程池初始化大小 核心线程数
                          int maximumPoolSize,  // 最大线程数
                          long keepAliveTime, // 当线程数大于核心数时，这时多余的空闲线程在终止之前等待新任务的最长时间。
                          TimeUnit unit,  //  keepAliveTime 单位
                          BlockingQueue<Runnable> workQueue, // 等待队列
                          ThreadFactory threadFactory,  // 执行程序创建新线程时使用的工厂
                          RejectedExecutionHandler handler) // 拒绝策略
                          
                          
                          
最大线程数 = 最大线程数 + 等待队列大小
// corePoolSize 线程池初始化大小 
核心池大小是保持活动状态（并且不允许超时等）的最小工作线程数
// maximumPoolSize最大线程数
// keepAliveTime 
等待工作的空闲线程的超时（以纳秒为单位）。当存在超过 corePoolSize 或 allowCoreThreadTimeOut 时，
线程将使用此超时。否则，他们将永远等待新的工作。
// workQueue
```

### 常见线程池

**4大方法**

```java
Executors.newFixedThreadPool(5);  // 固定线程数
Executors.newSingleThreadExecutor();  // 单线程
Executors.newCachedThreadPool(); // 缓存线程池
Executors.newWorkStealingPool(); // 
Executors.newScheduledThreadPool(3);  // 定时任务线程池
```

