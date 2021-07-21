# Chapter1 Introduction

[TOC]

## 程序设计语言和静态分析

程序设计语言（PL）: programming Languages可以分为以下三个部分：

<img src="./assets/Chapter1/PL.png" style="zoom:70%;" />

**p.s  三类编程语言: 命令式、函数式、逻辑式编程语言**

## 为什么需要学习静态分析

### 静态分析的作用

- Program Reliability (程序可靠性)
  - Null Pointer dereference, memory leak,etc.
- Program Security (程序安全性)
  - Private information leak, injection attack, etc.
- Compiler Optimization (编译优化)
  - Dead code elimination, code motion(代码调整), etc.
- Program Understanding
  - IDE call hierarchy, type information, etc.

### Rice Theorem (大米定理)

**Rice Theorem:** Any non-trivial property of the behavior of programs in a r.e. language is undecidable.

p.s: r.e. (recursively enumerable 递归可枚举) = recognizable by a Turing-machine

完美的静态分析是不存在的。

Sound/Truth/Complete:

Truth:代表程序中的事实，例如：程序中有10个内存泄漏

Sound(Overapproximate):会产生误报的情况（可能误报为12个）

Complete(精度，Underapproximate):会产生漏报的情况（可能只报了5个bug,还有5个没有分析出来）

<img src="./assets/Chapter1/sound_complete.png" style="zoom:67%;" />

所以不存在完美静态分析，只需要静态分析有意义即可（Useful static analysis）,所以一般会有两种方式来解决：

1. compromise soundness:就是会产生漏报(false negative)
2. compromise complete:就是会产生误报（false positive）

绝大多数都是compromise completeness: Sound and not fully-precise static analysis。（宁可误报，但一定要全面）

## 什么是静态分析

**静态分析可以由两个词来概括**

抽象（Abstraction）、近似（Over-approximation）

抽象：阈值转化（domain transfer）

近似：转换函数（transfer function，定义domain基础上的运算方式）、Control flow(控制流)

## 静态分析的特点和例子

### Abstraction

<img src="./assets/Chapter1/Abstraction.png" style="zoom:40%;" />

### Over-approximation: Transfer Function

 <img src="./assets/Chapter1/transferFunction.png" style="zoom:38%;" />

### Over-approximation: Control Flows

<img src="./assets/Chapter1/controlFlows.png" style="zoom:40">

个人对于控制流的理解，就是如上图PPT所说静态分析中对于路径的控制操作（常见的有join,meet）