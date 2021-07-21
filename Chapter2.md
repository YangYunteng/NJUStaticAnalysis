# Chapter 2 IR

[TOC]

## 编译器和静态分析

<img src="./assets/Chapter2/Compiler.png" style="zoom:70%">

Source Code 进过以下步骤变为机器码

1. Scanner (Lexical Analysis  词法分析（检查字符以及单词是否合理）需要一定的词典（Regular Expression）)

   生成Tokens

2. Parser (Syntax Analysis  语法分析  Context-Free Grammer 上下文无关，具体解释参见https://www.zhihu.com/question/21833944)

   生成AST

3. Type Checker(Semantic Analysis 语义分析 Attribute Grammer(当然编译器只能做简单的语义分析，比如浮点数转整数))

   生成Decorated AST

4. Translator

   生成IR(静态分析使用的便是IR)

5. Code Generator

   生成Machine Code

p.s 从编译的过程可以看出，静态分析可以作为编译Translator的一部分，来实现静态分析。（因此，对于不同语言，我们只需要将代码转换为相同的IR就可以对其进行同样的静态分析）

## AST vs. IR

<img src="./assets/Chapter2/IRvsAST.png" style="zoom:50%">

AST 会更加贴合程序语言（可以参考编译过程）

1. high-level and closed to grammer structure
2. usually language dependent
3. suitable for fast type checking;
4. lack of control flow information

IR 会更加贴近汇编语言

2. usually language independent
3. compact and uniform
4. contains control flow information
5. usually considered as the basis for static analysis

## IR：Three-Address Code(3AC)

三地址码（3AC）在指令的右边最多只有一个操作符 (并且每个3AC指令最多三个地址)

```
Normal:
t2 = a + b + 3 
3AC:
t1 = a + b
t2 = t1 + 3
```

Address可以包含以下三种

1. Name(变量名) : a, b
2. Constant(常量名): 3
3. Compiler-generated temporary(编译器生成的临时变量): t1, t2

## 实际静态分析器的3AC—Soot（Java）

Soot中存在多种IR：Baf, Jimple, Grimp, Shimple

其中Jimple 是3AC，同时也是Soot中用于静态分析的IR

JVM 四种类型函数调用

1. invokespecial: call constructor, call superclass methods, call private methods
2. invokevirtual: instance methods call (virtual dispatch 派生)
3. invokeinterface(类似invokevirtual): cannot optimization, checking interface implementation
4. invokestatic: call static methods

Java 7之后: invokedynamic -> Java static typing, dynamic language runs on JVM

method signature: class name; return type(返回值); method name(parameter1 type,parameter2 type,...)

clinit 在Java中进行静态属性的初始化

## SSA- 静态单赋值

### Static Single Assignment(SSA)

每个变量只会进行一次定义

对于分支中SSA的使用，会通过merge function函数进行变量合并

SSA优点

1. 流信息可以间接融入到每个单独的变量名
2. Define-and-Use pairs are explicit（定义和使用都是显示的，对于特定分析（条件常量传播，全局值编号）效果会很好）

SSA缺点

1. SSA会引入大量变量名，phi-function（SSA中的变量合并函数）
2. 在转换为机器码之后会导致出现低效问题

### Control Flow Analysis

IR(3AC)最终还是需要转换为CFG(Control Flow Graph)进行分析

CFG serves as the basic structure of static analysis

CFG中的节点可以是一个单独的三地址码或者可以是Basic Blocks(BB)

## 基本块 Basic Blocks (BBs)

### Node: Basic Blocks

Basic Blocks 可以认为是CFG的节点	

Basic Blocks(BB) are maximal sequences of consecutive three-address instructions（最大连续三地址码指令） with the properties that:

1. 此段程序的入口只能在最开始
2. 此段程序只能在最后退出

<img src="./assets/Chapter2/BB.png" style="zoom:80%">

### 基本块创建基本算法

<img src="./assets/Chapter2/buildBB.png" style="zoom:60%">

重点（leaders的确定）

1. The first instruction in P is a leader
2. Any target instruction of a conditional or unconditional jump is a leader（满足性质1）
3. Any instruction that immediately follows a conditonal or unconditional jump is a leader（满足性质2）

### 基本块划分实例

<img src="./assets/Chapter2/buildBBExample.png" style="zoom:75%">

## 控制流图 Control Flow Graphs(CFG)

在了解完基本块的划分之后，那么控制流图现在只需要添加Edge即可

### Edges of CFG

<img src="./assets/Chapter2/edgeofCFG.PNG" style="zoom:75%">

重点（Edge添加的条件）

1. 在基本块A和基本块B之间存在有条件或无条件跳转
2. 基本块B直接位于基本块A之后，并且基本块A最后并不是以无条件为结尾

### CFG构建的例子

<img src="./assets/Chapter2/CFG.png" style="zoom:75%">

CFG构建的顺序

1. 首先先将无条件以及有条件跳转进行连接
2. 在将基本块B直接跟在基本块A（并且A最后不是无条件结束）进行连接

p.s : 称基本块B为基本块A 的后继，基本块A为基本块B 的前驱

同样为了CFG 的完整性，添加Entry和Exit两个节点