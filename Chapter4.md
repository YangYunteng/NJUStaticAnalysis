# Chapter 4 DFA-FD

参考PPT DFA-FD(I,II) 数据流分析理论部分

[TOC]

## 迭代算法：另一个角度

### 理论

**定义1：**给定有k个节点（基本块）的CFG，迭代算法就是在每次迭代时，更新每个节点n的OUT[n]。

**定义2：**假设数据流分析的值域为V，则我们可以定义一个k-元组：$(OUT[n_1],OUT[n_2],...,OUT[n_k])$。则k-元组是集合$(V_1 \times V_2 ... \times V_n)$，记为$V^k$的其中一个元素，表示每次迭代后k个节点整体的值。

**定义3**：通过定义1和定义2，我们可以将每次迭代视为将一个$V^k$映射到一个新的$V^k$(映射规则是由transfer function 和control flow来决定)，记作函数$F:V^k \rightarrow V^k$

**迭代算法另一个角度：**通过不断迭代，直到相邻两次迭代的k-元组值相同，算法结束

### 图示

<img src="./assets/Chapter4/iteration.png" style="zoom:50%">

<center style="color:gray">迭代算法数学表示</center>

**不动点：**当$X_i=F(X_i)$时，Xi被称为不动点

**问题：**

1. 迭代算法是否会终止，或者说是否会到达不动点，更或者说存在解
2. 如果算法可以终止，那么不动点是否只有一个？如果不止一个解，那么那个解是最优的？
3. 算法经过几次迭代可以或者不动点或者是最终解？



## 偏序（Partial Order）

**定义：**给定偏序集$(P,\sqsubseteq)$，其中$\sqsubseteq$是集合P上的二元关系，并且$\sqsubseteq$满足一下关系

1. 自反性（Reflexivity）:$\forall x \in P, x \sqsubseteq x$
2. 对称性（Antisymmetry）:$\forall x,y\in P,x\sqsubseteq y \ \wedge y\sqsubseteq \ x \Rightarrow x=y$
3. 传递性（Transitivity）:$\forall x,y,z \in P,x\sqsubseteq y \ \wedge \  y\sqsubseteq z \Rightarrow \ x\sqsubseteq z$

**例子：**

1. P为整数集，二元关系为$\leq$，是偏序集；若二元关系$<$，则不是偏序集（不满足自反性）
2. P为英文单词组合，二元关系为$\sqsubseteq$子串关系（可以存在两个元素不具有可比性），是偏序集



## 上下界（Upper and Lower Bounds）

**定义：**给定偏序集$(P,\sqsubseteq)$以及它的子集$S\sqsubseteq P$，我们称

1. $如果\ \forall x \in S,\ x \sqsubseteq u, 那么u\in P是子集S的上界$
2. $如果\ \forall x\in S,\ l\ \sqsubseteq x,\ 那么l\in P是子集S的下界 $

**最小上界：**定义S的最小上界（least upper bound, lub or join）,记作$\sqcup S$。 上确界，对于子集S 的任意上界u, 均有$\sqcup S \sqsubseteq u$

**最大下界：**定义S的最大下界（greatest lower bound, glb or meet）,记作$\sqcap S$。下确界，对于子集S 的任意下界l, 均有$l \sqsubseteq \sqcap S $

**p.s** (后续格中使用较多): 此外如果子集S只拥有两个元素a和b(S={a,b})，	那么

$\sqcup S 可以写为 a\sqcup b$(the join of a and b)

$\sqcap S可以写为a\sqcap b$(the meet of a and b)

<img src="./assets/Chapter4/glb&lub.png" style="zoom:70%">

**属性：**

1. 并非每个偏序集都存在上下确界
2. 如果存在上下确界，那么则唯一（通过反证法以及对称性可以进行证明）



## 格（Lattice）,半格（Semilattice）,全格和格点积（Complete and Product Lattice）

### 格（Lattice）

**定义(格，Lattice)：**给定一个偏序集$(P,\sqsubseteq)$，对$\forall a,b \in P$，如果$a\sqcup b$ 和$a\sqcap b$存在，则偏序集$(P,\sqsubseteq)$被称为格（Lattice）

**例子：**

1. 集合P为整数集，二元关系$\sqsubseteq$为$\le$，则偏序集$(P,\le)$为格，$\sqcup$代表max，$\sqcap$代表min
2. 集合P为{a,b,c}的幂集，$\sqsubseteq$为$\subseteq$，则偏序集$(P,\subseteq)$为格，$\sqcup$代表$\cup$，$\sqcap$代表$\cap$

### 半格（SemiLattice）

**定义（半格，Semilattice）：**给定一个偏序集$(P,\sqsubseteq)$，$\forall a,b \in P$,

​	当且仅当$a\cup b$存在（上确界），该偏序集称为join smeilattice

​	当且仅当$a\cap b$存在（下确界），该偏序集称为meet smeilattice

### 全格（Complete Lattice）

**定义（全格，complete lattice）：**给定一个格$(P,\sqsubseteq)$，对$\forall S \subseteq P$，如果$\sqcup S$并且$\sqcap S$存在，则$(P,\sqsubseteq)$被称为全格。简单的来说，格的所有子集都有上下确界。

**例子：**

1. P是整数集，$\sqsubseteq$是$\subseteq$，不是全格，因为P的子集正整数集没有上确界。
2. $(S,\sqsubseteq)$中S 是{a,b,c}的幂集，$\sqsubseteq$表示$\subseteq$子集，是全格

**符号:**

1. 每个全格都有一个最大的元素，称作top，记作$\top=\sqcup P$
2. 每个全格都有一个最小的元素，称作bottom，记作$\bot=\sqcap P$

**性质：**每个有穷的格点都是全格。但是无穷的格也有可能是全格，如实数集[0,1]

### 格点积（Complete and Product Lattice）

**定义：**给定一组格$L_1=(P_1,\sqsubseteq_1),L_2=(P_2,\sqsubseteq_2),....,L_n=(P_n,\sqsubseteq_n)$，都有上确界$\sqcup_i$和下确界$\sqcap_i$,则通过下面属性来定义格点积$L^n=(P,\sqsubseteq)$：

- $P=P_1\times P_2...\times P_n$
- $(x_1,x_2,....,x_n)\sqsubseteq(y_1,y_2,...,y_n)\Leftrightarrow(x_1 \sqsubseteq_1 y_1)\wedge(x_2 \sqsubseteq_2 y_2)...\wedge(x_n\sqsubseteq y_n)$
- $(x_1,x_2,...,x_n)\sqcup(y_1,y_2,...,y_n)=(x_1 \sqcup_1 y_1,x_2 \sqcup_2 y_2,...,x_n\sqcup_n y_n)$
- $(x_1,x_2,...,x_n)\sqcap(y_1,y_2,...,y_n)=(x_1 \sqcap_1 y_1,x_2 \sqcap_2 y_2,...,x_n\sqcap_n y_n)$

**性质：**格点积是格；如果构成格点积的所有格为全格，那么格点积也是全格



## 数据流分析框架（via lattice）

数据流分析框架（D, L, F）:

- D(direction): 数据流分析的方向
- L(Lattice): 格（值域V（数据抽象），meet $\sqcap$ 和join$\sqcup$(Control Flow)）
- F(transfer function): 转换规则

数据流分析可以视为通过迭代算法对格的值域进行转换规则和meet/join操作

**目标问题：**迭代算法一定会停止（或者到达不动点）吗？

## 单调性（Monotonicity）和不动点理论(Fixed Point Theorem)

### 单调性

**定义：**存在函数$f:L \rightarrow L（L是格）$,如果对$\forall x,y \in L,满足 x\sqsubseteq y \Rightarrow f(x)\sqsubseteq f(y)$,那么称函数$f$单调。

### 不动点理论

**定义：**给定一个全格$(L,\sqsubseteq)$，如果

1. $f:L \rightarrow L$是单调的
2. $L$是有限格

则最小不动点可以通过迭代计算$f(\bot),f(f(\bot)),...,f^k(\bot)$直到达到不动点

最大不动点同理，通过迭代计算$f(\top),f(f(\top)),...,f^k(\top)$直到达到不动点

### 证明

#### 不动点的存在（Existence of Fixed Point）

<img src="./assets/Chapter4/existenceOfFixedPoint.png" style="zoom:50%">

关于证明，可能需要解释的有以下两处：

1. $\bot \sqsubseteq f(\bot)$，因为$\bot$是格$L$中最小的元素，所以经过函数变化之后可以很好的推理出$\bot \sqsubseteq f(\bot)$
2. 此外$L$是有限的，并且f是单调的，所以必定会出现阶梯，否则与现有的条件矛盾

#### 最小不动点（Least Fixed Point）

<img src="./assets/Chapter4/leastFixedPoint.png" style="zoom:50%">

## 迭代算法转化为不动点理论

**问题：**我们如何在理论上证明**迭代算法有解**、**有最优解**、**何时到达不动点**？那就是将迭代算法转化为**不动点理论**。因为不动点理论已经证明了，单调、有限的完全lattice，存在不动点，且从⊤开始能找到最大不动点，从⊥开始能找到最小不动点。

**目标：**证明迭代算法是基于一个完全格$(L,\sqsubseteq)$，且值域有限，转换函数具有单调性。

### 完全格的证明

迭代算法的每个节点的值域可以视为一个lattice，并且是有限的，所以每个节点的值可以视为完全格。每次迭代的k格基本块的值域是一个k元组，k元组可以视为格点积，根据格点积的性质；若$L^k$中的每个格都是完全的，则$L^k$也是完全格。

### 转换函数具有单调性

函数F：基本块中转换函数$f_i:L\rightarrow L+ 基本块分支之间的控制流影响（汇聚：join \sqcup / meet\sqcap;分叉：copy操作）$

1. 转换函数：基本块中的gen、kill是固定的，值域一旦变成1，就不会变回0，显然单调。
2. join/meet操作：L × L → L 。证明：∀x,y,z∈L，且有x⊑y需要证明x⊔z⊑y⊔z。

### 算法何时达到不动点

**定义（格的高度）：**格的高度是从格的top到达bottom之间最长的路径

<img src="./assets/Chapter4/whenGetFixedPoint.png" style="zoom: 33%;">

## May/Must Analysis, a Lattice View

<img src="./assets/Chapter4/maymustAnalysis.png" style="zoom:40%">

**p.s:**图中不管是may analysis 还是must analysis 都是从安全到不安全。

### May analysis(以reaching definition为例)

1. 从$\bot$开始，$\bot$表示没有任何定义到达（这就以为着，如果我们去做变量未定义检测应用的时候，就会导致认为所有的变量在中间过程已经全部被赋值，代表不需要对任何变量进行初始化，这是不安全的）

2. 相对于$\bot$而言的，$\top$代表之前的假定义全部到达，也就意味着我们需要对所有的变量全部进行初始化，这样虽然是安全的但是没有意义。

3. Truth:表明最准确的验证结果，假设{a,c}是truth，那么包括其以上的都是safe的，以下的都是unsafe，就是上图的阴影和非阴影。

   <img src="./assets/Chapter4/mayAnalysis.png" style="zoom:50%">

4. 从 $\bot$到$\top$，得到的**最小不动点**最准确，离Truth最近。上面还有多个不动点，越往上越不准。

### Must Analysis(以Avaliable Expressions 为例)

1. 从$\top$开始，表示所有表达式可用。如果用在表达式计算优化中，那么有很多已经被重定义的表达式也被优化了（实际上不能被优化），那么该优化就是错误的，**不安全**！
2. $\bot$表示没有表达式可用，都不需要优化，很**安全**！但没有用。
3. 从$\top$ 到$\bot$,就是从不安全到安全，存在一个Truth，代表准确的结果。
4. 从$\top$ 到$\bot$,达到一个**最大不动点**，离truth最近的最优解。

## 分配性(Distributivity)和MOP 

### MOP

**目的**：MOP（meet-over-all-paths）衡量迭代算法的精度。

**定义：**最终将所有的路径在计算之后再进行join/meet 操作

<img src="./assets/Chapter4/MOP.png" style="zoom:60%">

**MOP准确性**：有些路径不会被执行，所以不准确；若路径包含循环，或者路径爆炸，所以实操性不高，只能作为理论的一种衡量方式。

迭代算法 vs. MOP

<img src="./assets/Chapter4/iterativeVsMOP.png" style="zoom:60%">

## 常量传播

作为代码进行具体实现，具体解释参考PPT。

## Worklist 算法

**本质**：对迭代算法进行优化，采用队列来存储需要处理的基本块，减少大量的冗余的计算。

<img src="./assets/Chapter4/workList.png" style="zoom:60%">