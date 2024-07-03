---
title: "Java集合"
aliases: 
tags: [Java]
date: 2024-07-02
time: 14:28
---
## 一、集合框架图
![Java集合大体类图](/img/java/collection-inteface.png)
## 二、List
### 1、 概述
    List 是一种有序列表，有明确的第一个元素、最后一个元素、上一个元素和下一个元素。
  list 的行为和数组几乎一致：list 内部按照放啊如元素的先后顺序存放，每个元素都可以通过索引确定元素的位置，List 的索引和数组一样，从 0 开始。
List 可以添加重复元素、null 等
遍历尽量使用遍历器 iterator 进行遍历，效率高。for each 默认实现了 Iterator 迭代器遍历。

    ![list-interface](/img/java/list-interface.png)
### 2、 ArrayList
    查询快、增删慢；自动扩容（1.5 倍）；线程不安全；   
 ArrayList  底层使用数组实现，在新添或删除一个元素时，都需要通过数组的移动来完成，如下图
 ![arraylist-add-del](/img/java/arraylist-add-del.png)  
 初始化时最好设置初始化值，默认大小 **10** 。因为结构底层使用数组存储，数组是不可变长度的，当空闲元素填充满了时，需要扩充空间原有容量大小的 **1.5** 倍，每次扩展容都需创建一个新数组，然后进行数组的拷贝，会消耗性能（时间、空间） 
### 3、 LinkedList
    （有序） 线程不安全 增删效率高，检索效率低；空间占用率高；可作为队列、栈来使用
- 单向链表存储结构如下
 ![单向链表存储结构](/img/java/linedlist-struct.png)  
- LinkedList 是通过双向链表实现，结构如下
 ![双向链表存储结构](/img/java/linedlist-double-struct.png)  
## 三、 Queue

1. **队列**

    队列 FIFO（First Input First OutPut）  先进先出

    队列元素只能添加到末尾；只能从头部去除元素；能添加 null，但是避免添加 null

队列示意图
![队列](/img/java/queue.png)     
 **队列操作方法对比**

| 操作 | 抛异常 | 返回false或null |
| -- | -- | -- |
| 添加元素到队尾 | add(E e) | boolean offer(E e) |
| 取队首元素并删除 | E remove() | E poll() |
| 取队首元素但不删除 | E element() | E peek() |   

2. **栈**

    栈 stack  FILO （First Input Last OutPut）先进后出 or （Last in First Out）后进先出

    栈只有入栈 push 和出栈 pop2 种操作 peek（）取栈顶而不弹出

 栈示意图
 ![栈](/img/java/stack-struct.png)
 java 种没有 Stack 接口，一般将 Deque 来模拟 Stack 使用。只调用  push（） pop（） peek（） 方法来模拟

使用实例
中缀表达式计算  1 + 3  * （9 - 2）、进制转换   

## 四、Set

## 五、Map
    Map<K， V> 是一种键值映射表
![map类图](/img/java/map-interface.png)
### 1. HashMap
数组+链表实现 不指定初始化大小时 默认 **16**，加载因子 **0.75**（即空间再次扩容的判断依据。`threshold=table.length * loadFactor）`，第一次 put 时初始化 table ；
size >= threshold（size 为 map 种 entry 的实际个数）时 ，需要扩容，扩容到 table.length 的 **2** 倍

![hashmap结构及扩容](/img/java/hashmap-struct.png)

JDK1.8 之后，若某一节点链表元素超过 **8** 个，同时 `table.length > 64`，则将链表转为**红黑树**。红黑树相关内容详见算法与数据结构树内容。

### 2. linkedHashMap
### 3. TreeMap
### 4. ConcurrentHashMap


