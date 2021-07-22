# Chapter 6 Pointer Analysis

参考PPT  Chapter6_PTA.ppt

[TOC]

## Motivation

指针分析的必要性：此处以CHA为例，由于其只是考虑类之间的继承关系，所以会导致大量的假阳性。

<img src="./assets/Chapter6/1-1.png" style="zoom:70%">

<center style="color:gray">CHA 和 指针分析对比</center>



## 指针分析介绍

**介绍**

1. 指针分析是重要的静态分析技术，用于计算指针可能指向的内存空间
2. 对于面向对象语言（focus on java），指针分析主要是计算每个指针所指向的对象
3. 指针分析被认为是may analysis，计算一个指针所有可能指向对象的集合（i.e: 指针可能指向哪些对象）

**示例**

指针分析输入：程序本身

指针分析输出：指向关系表

<img src="./assets/Chapter6/1-2.png" style="zoom:70%">

<center style="color:gray">指针分析实例</center>

**指针分析 vs. 别名分析**

这两个是非常相关但不同的概念：

1. 指针分析：指针所指向的对象有哪些
2. 别名分析：两个指针所指向的是否是同一个对象

当然两者为什么又是紧密相关的，因为别名信息可以源于指向关系

**指针分析的应用**

1. 基本信息（别名分析/调用图）
2. 编译优化（虚调用内联）
3. bug检测（空指针检测）
4. 安全分析（信息流）



## 影响指针分析的关键要素

**指标：**精度（precision） & 效率（efficiency）

**影响因素：**堆抽象（Heap abstraction）,上下文敏感（Context sensitivity）,流敏感（Flow sensitivity）,分析域（Analysis scope）

<img src="./assets/Chapter6/3-1.png" style="zoom:70%">

<center style="color:gray">静态分析 影响因素</center>

本课程考虑的**分配点**（Allocation-site）, **上下文敏感/不敏感**（Context-senstive/Context-insenstive）, **流不敏感**（Flow-insensitive）, **全程序分析**（Whole-program）。

### Heap Abstraction(堆抽象)

#### **目的**

程序动态执行时，由于循环和递归等原因，堆对象理论上是可能无穷无尽的。为了可以确保指针分析可以有效终止，采用堆抽象技术，将无穷的具体对象抽象成**有限的**抽象对象（比如说将具有共性的对象抽象为一个对象）。

#### **技术概览**

<img src="./assets/Chapter6/3-2.png" style="zoom:70%">

<center style='color:gray'>堆抽象技术</center>

本次课程中只学习分配点技术（Allocation site）,最普遍使用的堆抽象技术。

#### **Allocation site**

**原理**：通过具体对象的创建点来抽象每个具体对象，通过创建点来表是在该点创建的所有对象。（因为Allocation site数量是有限的，所以抽象的对象也是有限的，是一种有效的堆抽象技术）。

<img src="./assets/Chapter6/3-3.png" style="zoom:70%"></img>

<center style="color:gray">Allocation site 示例</center>

### Context Senstivity

#### context-senstive vs. context-insensitive

<img src="./assets/Chapter6/3-4.png" style="zoom:70%"></img>

<center style="color:gray">context-senstive vs. context-insensitive</center>

**上下文敏感（context-senstive）:**对于不同的上下文时，对于同一个函数需要进行多次分析

**上下文不敏感（context-insenstive）:**合并一个方法的所有调用上下文，每个函数只需要分析一次

### Flow Sensitivity

#### Flow-sensitive vs. Flow-insensitive

<img src="./assets/Chapter6/3-5.png" style="zoom:70%">

<center sytle="color:gray"><b>Flow-senstive vs. Flow-insensitive</b></center>

**Flow-senstive：**尊重程序执行的顺序，在每个程序点都维持一张指向关系表

**Flow-insenstive：**忽略程序执行的顺序，整个程序只维持一张指向关系表

### Analysis Scope

<img src="./assets/Chapter6/3-6.png" style="zoom:70%"></img>

<center><b>Whole-program vs. Demand-driven</b></center>

**Whole-program：**全程分析计算程序中所有的指向关系，为可能的候选变量提供信息

**Demand-driven：**只分析影响特定域的指针的指向关系，为特定的候选提供信息

## 相关的语句

**目的：**在指针分析过程中，我们只需要关系指针相关的语句。	

**Java指针类型**

<img src="./assets/chapter6/4-1.png" style="zoom:70%">

<center><b>图4-1 Java 指针</b></center>

在图4-1中，我们在进行指针分析的时候只需要研究本地变量（local variable）和实例字段(instance field);因为Static field尝尝会设置为去全局变量；而数组在静态分析中会忽视索引，我们会将数组建模为一个单一的变量（等价于Instance field），记为arr，可能指向数组中存在的任意一个值。

**影响指针分析的语句**

<img src="./assets/Chapter6/4-2.png" style="zoom:70%">

<center><b>图4-2 影响指针分析的语句</b></center>

其中对于复杂的实例字段(复杂的内存访问)问题，我们会将其转换为三地址码来进行分析。

<img src="./assets/Chapter6/4-3.png" style="zoom:70%">

<center><b>图4-3 复杂内存访问</b></center>

关于Call 语句，主要还是java的4种调用方式：invokestatic, invokespecial, invokevirtual/invokeinterface（**重点关注**）。



