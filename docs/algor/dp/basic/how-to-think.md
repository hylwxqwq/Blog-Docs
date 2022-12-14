## 分析

最重要的是提取出题目中的所有要素，并尽可能的将其转化为状态的信息维度或者状态所表示的信息。

就是本着“问啥设啥”的理念。

举个例子：

> 一个公司有三个移动服务员，最初分别在位置 $1，2，3$ 处。
>
> 如果某个位置（用一个整数表示）有一个请求，那么公司必须指派某名员工赶到那个地方去。
>
> 某一时刻只有一个员工能移动，且不允许在同样的位置出现两个员工。
>
> 从 $p$ 到 $q$ 移动一个员工，需要花费 $c(p,q)$。
>
> 这个函数不一定对称，但保证 $c(p,p)=0$。
>
> 给出 $N$ 个请求，请求发生的位置分别为 $p_1 \sim p_N$。
>
> 公司必须按顺序依次满足所有请求，且过程中不能去其他额外的位置，目标是最小化公司花费，请你帮忙计算这个最小花费。
>
> $1\le L \le 200, 1\le N \le 3000$。

首先找 DP 的”阶段“，也就是最外层循环应该是由什么决定的。

可以发现，这是一个线性 DP 问题，那么要找的就是“按顺序成一条线“的要素。

（树形 DP 就是找以 xxx 为根的子树，区间 DP 是找划分和区间长度）

仔细阅读能发现：

> **顺序依次满足所有请求**

所以”请求“就构成了 DP 状态信息维度的”阶段“这一维。

然后所以可以先设计出一维状态：$dp_i$ 表示处理**前** $i$ 个请求，什么什么东西。

阶段类的信息一般都带有 ”前（$i$ 个），当前（第 $i$ 个）“ 的字样。

本题是要**处理完请求**，所以选择 ”前“ 作为要素。

然后设计 DP 状态的决策信息维度。 

可以发现，最终 DP 状态所表示的东西应当和 ”花费“ 有关，且属性应为最小值。

决策维度是用于”决定/确定“ DP 状态最终表示的东西应当是多少的。

所以本题的决策维度需要能够**决定**”花费“的值。

而发现本题的”花费“是由**函数** $c()$ 决定的，而函数 $c$ 的**自变量**则是”**员工的位置**“。

所以可以知道，DP 状态的决策信息维度应当含有 ”员工的位置“ 这一要素。

我们将三个员工所处的位置都设计进状态，因为无后效性已经由阶段保证，所以”位置“这一状态不需要从小到大顺序转移。

那么设计出 DP 状态：

设 $dp_{i,x,y,z}$ 表示**处理完**前 $i$ 个请求，员工 $1,2,3$ 分别在位置 $x,y,z$ 的所有方案，属性为花费的最小值。

然后转移方程就需要根据决策信息维度和代价函数来设计。

并且，因为题目中提到：

> **不允许在同样的位置出现两个员工**

所以转移时还要保证状态的合法性，即 $x\not=y\not=z$。

但是这个方程必然无法通过所有数据，因为空间开销太大了（$3000 \times 200 \times 200 \times 200$）。

所以我们要在题目要素中寻找能够减少空间开销的要素。

但是发现我们要想转移，就必须同时知道三个人的位置。

所以此处考虑的是**通过更少的信息可以推出完整的信息**的优化。

也就是要想办法只用一个人或者两个人的位置确定三个人的位置。

发现题目中说：

> **且过程中不能去其他额外的位置**

那么优化方式就明了了，利用另外两个人的位置 $x,y$，和最后处理的请求位置（$p_{i}$），推出处理第 $i$ 个请求的员工的位置 $z=p_i$ 即可。

所以可以写出 Code 

```cpp
memset(dp, 0x3f, sizeof dp);
dp[0][1][2] = 0, p[0] = 3;
for(int i = 0; i <= n; ++i) {
	for(int x = 1; x <= l; ++x) {
		for(int y = 1; y <= l; ++y) {
			if(x == y || x == p[i] || y == p[i]) continue;
			dp[i + 1][x][y] = min(dp[i + 1][x][y], dp[i][x][y] + c[p[i]][p[i + 1]]);
			dp[i + 1][p[i]][y] = min(dp[i + 1][p[i]][y], dp[i][x][y] + c[x][p[i + 1]]);
			dp[i + 1][x][p[i]] = min(dp[i + 1][x][p[i]], dp[i][x][y] + c[y][p[i + 1]]);
		}
	}
}
```

状态的初始化也是一个需要注意的问题。

一般来说都需要**根据题目要求**初始化所有的状态，然后单独给类似 "0" 这种边界状态赋值。

比如本题，因为求的是最小值，所以自然可以想到给所有 DP 值赋值为 $+\infty$。

但是如果直接赋值之后什么都不干，那所有的 DP 值都不会被更新。

所以我们转移的时候要让 $dp_{0}$ 这种状态派上用场，一般来说这种状态都会设计为 $0$，便于转移。

本题说，初始三个员工分别在 $1,2,3$ ，那么无优化的状态下，就需要令 $dp_{0,1,2,3} = 0$。

但是当前已经优化了，第三个员工的位置是要推出来的，那就假设最开始有一个 $0$ 号请求，

这个请求由 $3$ 号员工完成，且它的位置是 $p_0 =3$。

同时令 $dp_{0,1,2}=0$ 即可完成初始化。

当然，循环时不要忘记令 $i = 0$ ，要不然初始化都没用了。

## 总结

+ 设计状态：抓题目要素，按照”阶段->决策->意义“ 的顺序去决定 DP 状态。

  其中，”阶段“由 DP 种类决定，”决策“由计算花费的函数的自变量决定，”意义“由题目要求决定。

+ 设计方程：根据状态转移，注意无后效性，合法性，可行性。

+ 状态初始化：根据要求，单独处理边界。

