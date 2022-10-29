
部分内容摘录自 [OI-Wiki](https://oi-wiki.org/math/combinatorics/inclusion-exclusion-principle/)

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

$a_i$ 是用来枚举集合的。

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
    
> 给出不定方程 $\sum_{i=1}^nx_i=m$ 和 $n$ 个限制条件 $x_i\leq b_i$，其中 $m,b_i \in \mathbb{N}$. 求方程的非负整数解的个数。

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

### CF997C Sky Full of Stars

> 有一个 $n\times n$ 的正方形网格，每个网格用 RGB 三种颜色染色
> 
> 求有多少种方案使得至少有一行或者一列的颜色完全一致，模 $998244353$。
>
> $n \le 1e6$。

经过一些思考发现这是容斥（经典的至少不好求，用容斥转化成钦定）。

注意不是转成恰好，这样的话除了被限制的部分，其他部分也会有额外的限制。

如果转成钦定就只需要考虑被限制的部分，其他部分直接随便算。

我们考虑抽象出容斥的模型：

1. 全集 U ：所有可能的染色方式，方案数是 $3^{n\times n}$。
2. 元素 ：**染色方式**。
3. 属性：（更好的理解是“一种限制/条件”）一行或者一列涂的颜色完全相同。
   
发现属性本质上就是 $2n$ 个限制，也就是第 $i$ 行全相同，第 $i$ 列全相同这样的 $2n$ 个条件。

要求的东西是至少有一行或者一列的限制/条件被满足的方案数。

也就是说我们如果设 $S_i$ 表示所有满足第 $i$ 个属性/限制的染色方案构成的集合，

显然不能直接把所有 $|S_i|$ 相加，肯定会算重，那么考虑容斥，要求的答案就是 $|\bigcup S_i|$。

写出来就是：

$$
ans = \sum\limits_{S \not= \emptyset}(-1)^{|S| - 1} f(S)
$$

其中 $S$ 表示若干个 $S_i$ 的交，$f(S)$ 表示这种情况的贡献。

考虑 $S$ 表示有某 $i$ 行，某 $j$ 列是相同的情况，那么可以发现，选出来的这 $i$ 行 $j$ 列的颜色都是一样的。

我们现在就相当于 **钦定** 这 $i$ 行 $j$ 列是同一种颜色，剩下的 $(n - i)(n - j)$ 个格子就随便选（就算又有新的相等的也不管，因为这里是钦定，也就是只考虑被钦定的部分，其他的部分都直接容斥掉了）。

因为 $i$ 行 $j$ 列是可以任意选的（这里只是考虑某个固定状态），那么 $f(S)$ 就是 $\dbinom{n}{i}\dbinom{n}{j}3^{1 + (n - i)(i - j)}$

可以得知：

$$
\begin{aligned}
ans &= \sum\limits_{i \not = 0 \lor j \not=0} \dbinom{n}{i}\dbinom{n}{j}(-1)^{i + j - 1} f(i,j) \\
&= 3\sum\limits_{i \not = 0 \lor j \not = 0} \dbinom{n}{i} \dbinom{n}{j} (-1)^{i + j - 1} 3^{i + j}
\end{aligned}
$$

考虑枚举 $i$，把式子拆开：

$$
\begin{aligned}
ans= 3\sum\limits_{i > 0} \ [\ \dbinom{n}{i}(-1)^i \sum\limits_{j > 0} \ [\dbinom{n}{j} (-1)^j 3^{(n - i)(n - j)}\ ]\ ]
\end{aligned}
$$

后面那坨看起来像二项式定理，把 $j=0$ 补上之后减去再用二项式定理：

$$
ans = 3\sum\limits_{i > 0}\ [\ \dbinom{n}{i}(-1)^i([3^{n - i} - 1]^n - 3^{(n - i)n})\ ]
$$

然后就可以 $O(n \log n)$ 做了。    

代码实现有几个细节，注意减法之后要加mod，然后尽量用 i64 做运算避免忘记乘 1ll。

```cpp

// author : black_trees

#include <cmath>
#include <cstdio>
#include <vector>
#include <cstring>
#include <iostream>
#include <algorithm>

using namespace std;
using i64 = long long;

const i64 si = 1e6 + 10;
const i64 mod = 998244353;

i64 n;
i64 inv[si], fact[si], invf[si];
void init(i64 n) {
    inv[1] = 1, fact[0] = invf[0] = 1;
    for(i64 i = 2; i <= n; ++i)
        inv[i] = 1ll * (mod - mod / i) * inv[mod % i] % mod;
    for(i64 i = 1; i <= n; ++i)
        fact[i] = 1ll * fact[i - 1] * i % mod,
        invf[i] = 1ll * invf[i - 1] * inv[i] % mod;
}
i64 C(i64 n, i64 m) {
    if(m < 0 || n < m) return 0;
    return 1ll * fact[n] * invf[n - m] % mod * invf[m] % mod;
}
i64 Qpow(i64 a, i64 b) {
    i64 ret = 1 % mod;
    for(; b; b >>= 1) {
        if(b & 1) ret = ret * a % mod;
        a = a * a % mod;
    } return ret % mod;
}

int main() {

    cin.tie(0) -> sync_with_stdio(false);
    cin.exceptions(cin.failbit | cin.badbit);

    i64 ans = 0, sum = 0;
    cin >> n, init(n);
	for(i64 i = 1; i <= n; i++) 
		ans = (ans + Qpow(3ll, (1ll * n * (n - i) + i)) * Qpow(-1ll, i + 1ll) * C(n, i) % mod + mod) % mod;
	ans = ans * 2ll % mod;
	i64 tmp = 0;
	for(i64 i = 0; i < n; i++) {
		i64 t = -Qpow(3ll, i);
		tmp = (tmp + C(n, i) * Qpow(-1, i + 1ll) * (Qpow(1 + t, n) - Qpow(t, n) + mod) % mod + mod) % mod;
	}
	ans = (ans + tmp * 3) % mod;
	cout << ans % mod << endl;
    return 0; 
}
```