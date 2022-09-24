
摘录自 [OI-Wiki](https://oi-wiki.org/math/combinatorics/inclusion-exclusion-principle/)

我这里 Mathjax 不会显示 `\overline`，所以用了 `\overrightarrow` 代替。

## 容斥原理

设 U 中元素有 n 种不同的属性，而第 i 种属性称为 $P_i$，拥有属性 $P_i$ 的元素构成集合 $S_i$，那么

$$
\begin{split}
\left|\bigcup_{i=1}^{n}S_i\right|=&\sum_{i}|S_i|-\sum_{i<j}|S_i\cap S_j|+\sum_{i<j<k}|S_i\cap S_j\cap S_k|-\cdots\\
&+(-1)^{m-1}\sum_{a_i<a_{i+1} }\left|\bigcap_{i=1}^{m}S_{a_i}\right|+\cdots+(-1)^{n-1}|S_1\cap\cdots\cap S_n|
\end{split}
$$

即

$$
\left|\bigcup_{i=1}^{n}S_i\right|=\sum_{m=1}^n(-1)^{m-1}\sum_{a_i<a_{i+1} }\left|\bigcap_{i=1}^mS_{a_i}\right|
$$

### 证明

对于每个元素使用二项式定理计算其出现的次数。对于元素 x，假设它出现在 $T_1,T_2,\cdots,T_m$ 的集合中，那么它的出现次数为

$$
\begin{split}
Cnt=&|\{T_i\}|-|\{T_i\cap T_j|i<j\}|+\cdots+(-1)^{k-1}\left|\left\{\bigcap_{i=1}^{k}T_{a_i}|a_i<a_{i+1}\right\}\right|\\
&+\cdots+(-1)^{m-1}|\{T_1\cap\cdots\cap T_m\}|\\
=&C_m^1-C_m^2+\cdots+(-1)^{m-1}C_m^m\\
=&C_m^0-\sum_{i=0}^m(-1)^iC_m^i\\
=&1-(1-1)^m=1
\end{split}
$$

于是每个元素出现的次数为 1，那么合并起来就是并集。证毕。

### 补集

对于全集 U 下的 **集合的并** 可以使用容斥原理计算，而集合的交则用全集减去 **补集的并集** 求得：

$$
\left|\bigcap_{i=1}^{n}S_i\right|=|U|-\left|\bigcup_{i=1}^n\overrightarrow{S_i}\right|
$$

右边使用容斥即可。

## 应用

### 不定方程非负整数解计数

???+note "不定方程非负整数解计数"
    给出不定方程 $\sum_{i=1}^nx_i=m$ 和 $n$ 个限制条件 $x_i\leq b_i$，其中 $m,b_i \in \mathbb{N}$. 求方程的非负整数解的个数。

#### 没有限制时

如果没有 $x_i<b_i$ 的限制，那么不定方程 $\sum_{i=1}^nx_i=m$ 的非负整数解的数目为 $C_{m+n-1}^{n-1}$.

略证：插板法。

相当于你有 $m$ 个球要分给 $n$ 个盒子，允许某个盒子是空的。这个问题不能直接用组合数解决。

于是我们再加入 $n-1$ 个球，于是问题就变成了在一个长度为 $m+n-1$ 的球序列中选择 $n-1$ 个球，然后这个 $n-1$ 个球把这个序列隔成了 $n$ 份，恰好可以一一对应放到 $n$ 个盒子中。那么在 $m+n-1$ 个球中选择 $n-1$ 个球的方案数就是 $C_{m+n-1}^{n-1}$。

#### 容斥模型

接着我们尝试抽象出容斥原理的模型：

1. 全集 U：不定方程 $\sum_{i=1}^nx_i=m$ 的非负整数解
2. 元素：变量 $x_i$.
3. 属性：$x_i$ 的属性即 $x_i$ 满足的条件，即 $x_i\leq b_i$ 的条件

目标：所有变量满足对应属性时集合的大小，即 $|\bigcap_{i=1}^nS_i|$.

这个东西可以用 $\left|\bigcap_{i=1}^{n}S_i\right|=|U|-\left|\bigcup_{i=1}^n\overrightarrow{S_i}\right|$ 求解。$|U|$ 可以用组合数计算，后半部分自然使用容斥原理展开。

那么问题变成，对于一些 $\overrightarrow{S_{a_i}}$ 的交集求大小。考虑 $\overrightarrow{S_{a_i} }$ 的含义，表示 $x_{a_i}\geq b_{a_i}+1$ 的解的数目。而交集表示同时满足这些条件。因此这个交集对应的不定方程中，有些变量有 **下界限制**，而有些则没有限制。

能否消除这些下界限制呢？既然要求的是非负整数解，而有些变量的下界又大于 $0$，那么我们直接 **把这个下界减掉**，就可以使得这些变量的下界变成 $0$，即没有下界啦。因此对于

$$
\left|\bigcap_{a_i<a_{i+1} }^{1\leq i\leq k}S_{a_i}\right|
$$

的不定方程形式为

$$
\sum_{i=1}^nx_i=m-\sum_{i=1}^k(b_{a_i}+1)
$$

于是这个也可以组合数计算啦。这个长度为 $k$ 的 $a$ 数组相当于在枚举子集。