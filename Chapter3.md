# Chapter 3 DFA-AP

参考PPT  DFA-AP(I,II) 数据流应用分析

[TOC]

## 数据流分析概述

may analysis: 输出的信息可能正确（over-approximation, 可能存在误报）

must analysis: 输出的信息一定正确（under-approximation, 可能存在漏报）

**无论是may analysis还是must analysis, 都是为了分析安全的角度来考虑（在下面的三种具体的数据流分析中可以具体了解）**

如下图所示：

1. Nodes(BBs/statements) -> Transfer function(转换函数)
2. Edges(control flows) -> Control-flow handling(例如：并和交)
3. CFG(control flow graph) -> a program

**不同的数据流分析应用 有 不同的数据抽象和不同的流安全策略（may or must analysis）,如不同的转化规则和控制流处理**

<img src="./assets/Chapter3/overview.png" style="zoom:70%">

## 数据流分析初探（准备工作）

**输入/输出状态**：IR语句执行前/后的状态（本质就是抽象表达的数据的状态，如变量的状态）。

<img src="./assets/Chapter3/inputAndOutputState.png" style="zoom: 60%;">

对于上图中的程序点，在每个数据流分析应用中，我们会将每一个数据流值与每个程序点相关联，该值表示对该点可以观察到的所有可能程序状态的抽象。

**数据流的最终目的：** 数据流分析是**基于语句语义（tranfer functions）**以及**基于控制流**为所有语句的IN[s]和OUT[s]找到一组安全近似的定向的约束

**数据流实现的本质：**求解最优解

> 前向分析（Forward Analysis）:按程序执行顺序。OUT[s] = *f<sub>s</sub>*(IN[s]) 	s- statement
>
> 后向分析（Backward Analysis）:逆向分析。 IN[s] = *f<sub>s</sub>*(OUT[s])

**Basic Blocks中控制流符号**

<img src="./assets/Chapter3/BBNotations.png" style="zoom:67%;" >

## Reaching Definition Analysis	

### **问题描述**

给变量v一个定义d(赋值)，存在一条路径使得程序点p能够到达程序点q,且在此过程中变量不会被重新赋值。（也就是意味着从程序点p到程序点q，变量不会被重新赋值）

<img src="./assets/Chapter3/reachingDefinitionProblem.png" style="zoom:70%">

<center style="color:blue">Reaching Definition 问题描述</center>

### **实际应用**

检测未定义变量：在CFG开始处将变量进行假定义，如果假的定义到达程序点p时仍在使用，则代表改变量未被定义

### 数据流分析设计过程

**抽象表示：**设程序有n条赋值语句，则可以将其抽象为n-bit的数组来表示reaching definition状态

**安全策略的选择（Safe approximation）**

1. **Transfer function:** $OUT[B] = gen_B \cup (IN[B] - kill_B)$

   解释：基本块B的输出 = 基本块B 的所有变量v的定义(赋值/修改) 语句 $\cup$ (基本块B 的输入 -  程序中其他所有定义了变量v的语句)

2. **Control Flow:**$IN[B] = \bigcup_{P\ is\ predecessor\ of\ B}\ OUT[P]$

   解释：主要是并集的解释，因为根据问题描述是只要存在一条路径有定义到达程序点即可，所以采用并集

<img src="./assets/Chapter3/TF1.png" style="zoom:70%">

<center>Transfer Function</center>

### 算法实现

经典迭代算法实现

<img src="./assets/Chapter3/Algorithm1.png" style="zoom:70%">

迭代算法的输入：程序的CFG ($kill_B$ and $gen_B$ 可以通过对于BBs中语句分析获得)

迭代算法的输出：每个基本块的输入和输出状态（所需要的最优解，最小/最大不动点）

算法设计：

1. 初始化边界值（前向分析：Entry Node的输出状态 ；后向分析：Exit Node的输入状态）
2. 初始化除边界外的所有基本块的输出状态/输入状态（分别对应于前向分析/后向分析）
3. 循环直到所有的基本块的输出/输入状态不再发生变化（分别对应于前向分析/后向分析）

**补充说明：**

为什么算法Step1中的初始化需要和Step2中初始化分开？

​	答：因为Step1中的初始化未必是空集（这主要涉及到must analysis 和 may analysis的区别，在理论部分会给予解释，因为must analysis 中边界值是初始化为全集）

为什么迭代算法会最终停止？

​	答：算法停止的根本在于以下两点原因

1. $OUT[B] = gen_B\ \cup \ ({IN[B]-kill_B}) $ 这个transfer function中可以看出$OUT[B]$只取决于$IN[B]$ (因为$gen_B \&kill_B$对于每个基本块B是不会发生变化的，所以当输入状态稳定之后，输出状态也会随之稳定)

2. 还是从Transfer Function:$OUT[B] = gen_B\ \cup \ ({IN[B]-kill_B}) $来看,$OUT[B]$输出状态由$gen_B和（IN[B]-kill_B）(此部分记为suvivor_S)$两部分决定,我们可以完全想象从，也就是意味着$gen_B$是不变的，也就是这一部分的n-bits是不会从1->0;而对于suvivor部分是由$IN[B]$决定，而$IN[B]$也就是上一个基本块的输出状态，那么在CFG中，按照程序执行顺序一定存在一个基本块会先出现$OUT[B]$的事实数大于$IN[B]$,这也就意味着输出状态的值是不会出现缩减的情况，因此n-bit是会趋向于全集的稳定趋向变化。

3. 最后，由1和2可以知道，因为事实数是有限的（也就是所需要记录的n-bit的n是稳定的），所以迭代算法会终止

   具体的算法终止证明在后续理论部分会进行介绍

### 算法实例

<img src="./assets/Chapter3/reachingDefinitionExample.png" style="zoom:70%">

图中第三次迭代的结果便是我们所求解的稳定的安全的近似约束集（此处为最小不动点，在理论部分会进行具体分析）

## Live Variables Analysis

### 问题描述

在某个程序点p处，从p开始到Exit Node的CFG中存在某条路径用到了v,如果用到了v,则变量v在p点时live 的，否则为dead。（这也就意味着从程序点p到Exit Node 变量v不能重新定义）

### 实际应用

可用于寄存器分配（在某一点所有寄存器都被占用了，我们就可以通过此种方式去查看是否存在dead 的变量，便可以将其替换掉）

### 数据流分析设计过程

**数据抽象：**程序中的n个变量用长度为n bit的向量来表示，对应bit为1，则该变量为live，反之为0则为dead。

**安全策略的选择（Safe approximation）**

1. **Transfer function:** $IN[B] = use_B \cup (OUT[B] - def_B)$

   解释：活跃变量分析采用的是后向分析，所有需要由输出状态来计算输入状态。基本块B 的输入状态 = 在基本块B中在被重定义之前使用的变量$\cup$在基本块B的输出状态中在B中未被使用的变量。

2. **Control Flow:**$OUT[B] = \bigcup_{S\ is\ successor\ of\ B}\ IN[S]$

   解释：采用的是后向分析并且是may analysis,所以是基本块所有后继的并集

### 算法实现

经典迭代算法的实现

<img src="./assets/Chapter3/Algorithm2.png" style="zoom:70%">

迭代算法的输入：程序的CFG ($def_B$ and $use_B$ 可以通过对于BBs中语句分析获得)

迭代算法的输出：每个基本块的输入和输出状态（所需要的最优解，最小/最大不动点）

算法设计：

1. 初始化边界值（前向分析：Entry Node的输出状态 ；后向分析：Exit Node的输入状态）
2. 初始化除边界外的所有基本块的输出状态/输入状态（分别对应于前向分析/后向分析）
3. 循环直到所有的基本块的输出/输入状态不再发生变化（分别对应于前向分析/后向分析）

### 算法实例

<img src="./assets/Chapter3/Example2.png" style="zoom:70%">

图中第三次迭代的结果便是我们所求解的稳定的安全的近似约束集（此处为最小不动点，在理论部分会进行具体分析）

## Available Expressions Analysis

### 问题描述

程序点p处的表达式`x op y`可用需满足2个条件，一是从entry到p点必须经过`x op y`，二是最后一次使用`x op y`之后，没有重定义操作数x、y。（如果重定义了x 或 y，如x = `a op2 b`，则原来的表达式`x op y`中的x或y就会被替代）。

### 实际应用

用于优化，检测全局公共子表达式。

### 数据流分析过程

**数据抽象：**程序中的n个表达式，用长度为n bit的向量来表示，1表示可用，0表示不可用。

**安全策略的选择（Safe approximation）**

1. **Transfer function:** $OUT[B] = gen_B \cup (IN[B] - kill_B)$

   解释：genB—基本块B中所有新的表达式（并且在这个表达式之后，不能对表达式中出现的变量进行重定义）-->加入到OUT；killB—从IN中删除变量被重新定义的表达式。

2. **Control Flow:**$IN[B] = \bigcap_{P\ is\ predecessor\ of\ B}\ OUT[P]$

   解释：前向分析，从Entry到p点的所有路径都必须经过该表达式

**可能存在的误区**

<img src="./assets/Chapter3/correct3.png" style="zoom:75%">

虽然在上述这样一个CFG中给人的直观的感觉x值发生了变化，但是实际上是可行的，因为在程序实际使用中我们可以将程序中a,b,c这三个变量全部改变为同一个变量t,那么在后续使用这个所谓的Available expression的时候直接调用变量t(也就是说不会影响到具体优化，因为我们只会取到最后一次的表达式的值)

此外，为什么这个数据流分析采用的是must analysis,是因为从问题表述中可以看出从entry 到p必须经过表达式，这也就是要求所有path中都至少含有该表达式。而这样的话可能会带来一定的误报，如上图所示，可能对于x的赋值实际上等同于表达式中的x的值，这就可能会带来漏报。但是如果是采用may analysis 产生误报就会对全局公共子表达式优化造成错误。

### 算法实现

经典数据流分析迭代算法

<img src="./assets/Chapter3/Algorithm3.png" style="zoom:70%">

迭代算法的输入：程序的CFG ($kill_B$ and $gen_B$ 可以通过对于BBs中语句分析获得)

迭代算法的输出：每个基本块的输入和输出状态（所需要的最优解，最小/最大不动点）

算法设计：

1. 初始化边界值（前向分析：Entry Node的输出状态 ；后向分析：Exit Node的输入状态）
2. 初始化除边界外的所有基本块的输出状态/输入状态（分别对应于前向分析/后向分析）
3. 循环直到所有的基本块的输出/输入状态不再发生变化（分别对应于前向分析/后向分析）

### 算法实例

<img src="./assets/Chapter3/Example3.png" style="zoom:70%">

图中第三次迭代的结果便是我们所求解的稳定的安全的近似约束集（此处为最大不动点，在理论部分会进行具体分析）

## 总结

### 三种算法的比较图

<img src="./assets/Chapter3/comparison.png" style="zoom:70%">

此处关于Transfer function是一种模板的形式，因为这三个活跃变量分析的问题都是gen-kill problem, 所以可以写为图表中的通用形式。

另外一种通用形式，就是将IN和OUT位置进行调换。

### 任意数据流分析迭代算法设计

此处为我个人的总结和理解

首先不妨会议一下在introduction中对于静态分析的描述：Abstraction & over-approximation(抽象和近似)

所以在设计一个数据流分析算法的时候，首先就是抽象Domain(数据抽象形式)

其次，我们需要根据问题来确定Direction 和 May/Must analysis ,确定了这两个方面之后，如果是对应于gen-kill problem来说，我们的tranfer fucntion($OUT=gen\bigcup(IN-kill)$)以及Control Flow($\bigcup 或 \bigcap$这两个操作符)都可以得到确定。

最后采用经典迭代算法进行套用

1. 初始化边界条件（may analysis:空集 must analysis: 全集）
2. 初始化输入/输出状态为空集（前向分析：输出为空集；后向分析：输入为空集）
3. 通过循环，等待输入/输出状态进行稳定状态，不再发生变化结束（前行分析：输入状态稳定；后向分析：输出状态稳定；也就是你初始化的是什么状态就是什么状态啦）