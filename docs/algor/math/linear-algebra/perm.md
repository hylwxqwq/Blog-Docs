> 大部分摘抄自 : [OI-Wiki](https://oi-wiki.org/math/permutation-group/)

## 定义

有限集合到自身的双射（即一一对应）称为置换。集合 $S=\{a_1,a_2,\dots,a_n\}$ 上的置换可以表示为

$$
f=\begin{pmatrix}a_1,a_2,\dots,a_n\\
a_{p_1},a_{p_2},\dots,a_{p_n}
\end{pmatrix}
$$

意为将 $a_i$ 映射为 $a_{p_i}$，其中 $p_1,p_2,\dots,p_n$ 是 $1,2,\dots,n$ 的一个排列。显然 $S$ 上所有置换的数量为 $n!$。

## 置换乘法

对于两个置换 $f=\begin{pmatrix}a_1,a_2,\dots,a_n\\a_{p_1},a_{p_2},\dots,a_{p_n}\end{pmatrix}$ 和 $g=\begin{pmatrix}a_{p_1},a_{p_2},\dots,a_{p_n}\\a_{q_1},a_{q_2},\dots,a_{q_n}\end{pmatrix}$，$f$ 和 $g$ 的乘积记为 $f\circ g$，其值为

$$
f\circ g=\begin{pmatrix}a_1,a_2,\dots,a_n\\
a_{q_1},a_{q_2},\dots,a_{q_n}\end{pmatrix}
$$

简单来说就是先后经过 $f$ 的映射，再经过 $g$ 的映射。

## 循环置换

循环置换是一类特殊的置换，可表示为

$$
(a_1,a_2,\dots,a_m)=\begin{pmatrix}a_1,a_2,\dots,a_{m-1},a_m\\
a_2,a_3,\dots,a_m,a_1\end{pmatrix}
$$

若两个循环置换不含有相同的元素，则称它们是 **不相交** 的。有如下定理：

任意一个置换都可以分解为若干不相交的循环置换的乘积，例如

$$
\begin{pmatrix}a_1,a_2,a_3,a_4,a_5\\
a_3,a_1,a_2,a_5,a_4\end{pmatrix}=(a_1,a_3,a_2)\circ(a_4,a_5)
$$

该定理的证明也非常简单。如果把元素视为图的节点，映射关系视为有向边，则每个节点的入度和出度都为 1，因此形成的图形必定是若干个环的集合，而一个环即可用一个循环置换表示。

这个东西也可以叫做“置换环”，在很多序列的变换问题里出现比较频繁。

不过用到的时候一般 $S$ 都是一个可重集（可以看作一个序列）。

假设一次 “操作” 是，交换 $a$ 中的任意两个元素，

$b$ 是 $a$ 经过一次置换之后得到的序列 $\{a_{p_1},a_{p_2},\dots,a_{p_n}\}$。

并且我们将置换

$$
f=\begin{pmatrix}a_1,a_2,\dots,a_n\\
a_{p_1},a_{p_2},\dots,a_{p_n}
\end{pmatrix}
$$

拆成了若干个循环置换 $g_1,g_2,\dots$。

那么可以有如下结论：

1. 循环置换上不能有相同的元素（就是说，假设这个循环置换是 $a$ 的一个子序列 $c$ 的置换，那么 $c$ 不能有重复的元素出现）
   
   这个用置换的定义（集合意义上的双射）可得。

2. 如果把两个循环置换 $g_1,g_2$ 上分别拿两个元素出来，交换一次，那么 $g_1,g_2$ 会合并成一个循环置换。

3. 如果把一个循环置换 $g$ 的任意两个元素交换一次，它会分裂成两个循环置换 $g_1,g_2$。

4. 对于一个循环置换 $g$，单独做一次这个置换最少需要花费 $siz(g)-1$ 次操作。
   
   其中 $siz()$ 是这个循环置换包含的元素个数。
   
   这个由 2 可以得知

5. 由 $a$ 到 $b$，至少需要使用 $n - cnt(g)$ 次操作。
   
   其中 $cnt(g)$ 是总共拆成的循环置换的个数。
   
   这个由 2, 3, 4 可以得知。

最近一些用到它的 CF 题：

[1672F1 - Codeforces](https://codeforces.com/contest/1672/problem/F1)

[1672F2 - Codeforces](https://codeforces.com/contest/1672/problem/F2)

[1670C - Codeforces](https://codeforces.com/contest/1670/problem/C)

[1678E - Codeforces](https://codeforces.com/contest/1678/problem/E)

[1682E - Codeforces](https://codeforces.com/contest/1682/problem/E)

一般都是直接连边，然后 Tarjan 或者 dfs 找环，然后处理。

个人喜欢使用 Tarjan，因为在找完环之后能做额外的处理，比较方便。

不过 Tarjan 就需要判一下自环，因为它容易处理不了，或者说其实根本没必要。

但如果要打起来快，并且不需要什么额外的信息，还是直接写个 dfs 比较好。