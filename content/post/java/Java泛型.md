---
title: "Java泛型"
aliases: 
tags: [Java]
date: 2024-07-02
time: 15:15
---
## 种类

`List、List<Object> 、List<?> 、List<T>、List<? extend T>、List<? Super T>` 


**1. `List<Object>`**
与无泛型 List 差不多，都可往里 add 任何对象，但意义不大，只能往里 add，取出遍历往往出现类型错误

**2. `List<?>`**
泛型不确定的集合，一般用来接收不能确定泛型的集合

**3. 重点区分 List<？ extend T>、List<? Super T> **
| 功能/类型     |  List<？ extend T>  |   List<? Super T> |
| -- | -- | -- |
| 特性         |  GET FIRST   |  PUT  FIRST |
| **实际存储内容** | `T 及 T 子类对象， null` <br>1.  初始化时接受 T 及 T 子类型的 list。<br>2.  不能 add| `T 及 T 子类对象，T 的父类对象，null` <br >1.  初始化时可接受父类类型的 list。 <br>2.  add 操作时只能添加 T 及 T 子类对象|
| get 功能 | 获取到的值强转为 T，子类对象丢失泛型<br> `T t = list.get(0);` | 获取到对象类型为 Object，所有对象丢失类型<br> `Object o = list.get(0);` |
| put 功能 | 除 null 外，其余对象均不能被 add | 可 put 对象：null，T 及 T 子类对象 |
| 集合值接收（初始化）| T 及 T 子类类型的 List 可赋值给该泛型 list | T 及 T 父类类型的 list 可赋值给该泛型 list |
| PECS | 常用来做结果接收  producer Extend | 常用来做参数 Consumer  Super |
