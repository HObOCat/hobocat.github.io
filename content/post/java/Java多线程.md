---
title: "Java多线程"
aliases: 
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



### JMM

### CAS(Compare And Swap)

### 工具类

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



