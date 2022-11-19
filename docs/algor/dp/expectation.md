
## 数学期望

数学期望的定义是，对于一个随机变量 $X$，假设 $X$ 的取值有 $x_1, x_2, x_3, \dots, x_n$ 这么多种。

并且 $P(X = x_i) = p_i$，那么随机变量 $X$ 的数学期望 $E(X)$ 定义为随机变量 $X$ 的所有取值与它对应概率的乘积的总和。

即是: $E(X) = \sum\limits_{i = 1}^n x_i\cdot p_i$

性质（数学期望的线性性）：$E(aX + bY) = aE(X) + bE(Y)$。

这个性质非常有用，比如这样一个问题：

> 求扔两个骰子的点数之和的期望。

设扔一个骰子的点数取值为随机变量 $X$，那么 $E(X) = \dfrac{1}{6}(1 + 2 + 3 + 4 + 5 + 6) = 3.5$。

那么两枚骰子的点数之和可以表示为 $X + X$ 或者 $2X$，根据期望的线性性有 $E(2X) = 3.5 \times 2 = 7$。

## 期望 dp

> 概述

### Acwing217. 绿豆蛙的归宿

> 给出一个有向无环的连通图，起点为 $1$，终点为 $N$，每条边都有一个长度。
>
> 数据保证从起点出发能够到达图中所有的点，图中所有的点也都能够到达终点。
>
> 绿豆蛙从起点出发，走向终点。
>
> 到达每一个顶点时，如果有 $K$ 条离开该点的道路，绿豆蛙可以选择任意一条道路离开该点，并且走向每条路的概率为 $\dfrac{1}{K}$。
>
> 现在绿豆蛙想知道，从起点走到终点所经过的路径总长度的期望是多少？
> 
> $1\le n \le 1e5, 1\le m \le 2e5$。

首先我们注意到要求的是从 $1 \to n$ 的路径长度的期望，也就是所有可能的路径长度乘上对应的概率。

显然我们不可能直接一个式子就把这个东西写出来，所以我们考虑期望递推/dp。

设 $dp(u)$ 表示从 $1 \to u$ 的路径长度的期望。

我们根据定义尝试推一下式子，假设 $u$ 是从 $v_1 \dots v_r$ 这些节点可以到达的。

假设 $x(v_i)_j$ 表示从 $1 \to v_i$ 的路径的某一种可能长度，$p(v_i)_j$ 表示出现 $x(v_i)_j$ 这种情况的概率。

那么就有：

$$
\begin{aligned}
	dp(u) &= \sum\limits_i\sum\limits_j(p(v_i)_j \times \dfrac{1}{deg(v_i)} \times (x(v_i)_j + w(u, v_i))) \\
	&= \sum\limits_i(\dfrac{1}{deg(v_i)} \times \sum\limits_j (p(v_i)_j \times x(v_i)_j + p(v_i)_j \times w(u, v_i))) \\
	&= \sum\limits_i(\dfrac{1}{deg(v_i)} \times (dp(v_i) + w(u, v_i) \times \sum\limits_j p(v_i)_j))
\end{aligned}
$$

注意到 $\sum\limits_j (p(v_i)_j)$ 就是从 $1$ 出发，走到 $v_i$ 的概率。

所以我们转移 dp 的时候顺带着处理一个数组 $P(v_i)$ 表示从 $1 \to v_i$ 的概率就行了。

初始化 $dp(1) = 0$，终态 $dp(n)$。

其实这种“正推”的方法比较麻烦，其原因在在于需要处理 $P$ 这个数组，因为从 $1 \to v_i$ 的概率是不能直接确定的。

注意到我们最后一定会走到 $n$，也就是说从 $\forall u \not= n, P(u \to n) = 1$，那么如果我们考虑倒推会怎样呢？

设 $dp(u)$ 表示从 $u \to n$ 的路径长度期望，并且 $u$ 可以到达 $v_1 \dots v_r$。

根据定义，类似正推，我们可以写出：

$$
\begin{aligned}
	dp(u) = \dfrac{1}{deg(u)}\sum\limits_i(dp(v_i) + w(u, v_i) \times \sum\limits_j p(v_i)_j)
\end{aligned}
$$

是不是基本一样？并不，注意到这里的 $\sum\limits_jp(v_i)_j$ 表示的是从 $v_i$ 出发走到 $n$ 的概率，这个概率**一定是** $1$，所以状态转移变为：

$$
\begin{aligned}
	dp(u) = \dfrac{1}{deg(v_i)} \sum\limits_i(dp(v_i) + w(u, v_i))
\end{aligned}
$$

然后转移就不用额外维护信息了。

??? note "Code"

	```cpp
	// author : black_trees

	#include <cmath>
	#include <queue>
	#include <cstdio>
	#include <cstring>
	#include <iomanip>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	using i64 = long long;
	using ldb = long double;

	const int si = 1e5 + 10;

	int n, m;
	int tot = 0, head[si];
	struct Edge { int ver, Next, w; } e[si << 1];
	inline void add(int u, int v, int w) { e[tot] = (Edge) {v, head[u], w}, head[u] = tot++; }

	std::queue<int> q;
	int deg[si], k[si];
	ldb dp[si];

	int main() {

		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);

		memset(head, -1, sizeof head);
		cin >> n >> m, dp[n] = 0;
		for(int i = 1; i <= m; ++i) {
			int u, v, w;
			cin >> u >> v >> w;
			add(v, u, w), deg[u]++, k[u]++;
		}
		q.push(n);
		while(!q.empty()) {
			int u = q.front();
			q.pop();
			for(int i = head[u]; ~i; i = e[i].Next) {
				int v = e[i].ver, w = e[i].w;
				deg[v] --;
				dp[v] += (dp[u] + 1.0 * w) / (1.0 * k[v]);
				if(!deg[v]) q.push(v);	
			}
		}
		cout << fixed << setprecision(2) << dp[1] << endl;

		return 0;
	}
	```

### Acwing218. 扑克牌

> Admin 生日那天，Rainbow 来找 Admin 玩扑克牌。
>
> 玩着玩着 Rainbow 觉得太没意思了，于是决定给 Admin 一个考验。
>
> Rainbow 把一副扑克牌(54 张)随机洗开，倒扣着放成一摞。
> 
> 然后 Admin 从上往下依次翻开每张牌，每翻开一张黑桃、红桃、梅花或者方块，就把它放到对应花色的堆里去。
> 
> Rainbow 想问问 Admin，得到 A 张黑桃、B 张红桃、C 张梅花、D 张方块需要翻开的牌的张数的期望值 E 是多少？
> 
> 特殊地，如果翻开的牌是大王或者小王，Admin 将会把它作为某种花色的牌放入对应堆中，使得放入之后 E 的值尽可能小。
>
> 由于 Admin 和 Rainbow 还在玩扑克，所以这个程序就交给你来写了。
>
> $0\le A,B,C,D\le 15$

基本的期望 dp

### Atcoder Dp Edu J - Sushi

基本的期望 dp

### [HNOI2013] 游走

期望 dp 结合高斯消元
