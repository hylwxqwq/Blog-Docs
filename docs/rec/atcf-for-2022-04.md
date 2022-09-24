## 四月好题改错

!!! 一个单独的Trick
	MJC 告诉我：约束关系很难处理的时候，不是并查集就是连边。

### CF1668D/CF1667B Optimal Partition

`Apr/20/2022`

> 给你一个序列 $a$ ，要求将其分成若干个子段。
>
> 定义 $ss(l,r)$ 为 $a_l + a_{l + 1} + a_{l + 2} + \dots + a_r$。
>
> 每个子段 $a[l,r]$ 的贡献 $f(l,r)$ 为:
>
> 1. $r - l + 1$，如果 $ss(l,r) > 0$
> 2. $-(r- l + 1)$，如果 $ss(l,r) < 0$。
> 3. $0$ ，如果 $ss(l,r) = 0$。
>
> 求一种分割方式，使得被选出的子段的 $\sum f(l,r)$ 最大。
>
> $n \le 5e5$。

首先我们可以想到一个 $\text{O}(n^2)$ 的 DP：

设 $dp_{i,j}$ 表示前 $i$ 个位置，分成 $j$ 段的所有方案，属性为贡献和的最大值。 

然后发现数组就已经开不下了。

但是题目**没有要求你具体要分多少段，任意分割成多少段都是可以的**。

所以可以利用”任务安排1“ 当中用到的思想。

任务安排1 是设 $dp_{i}$ 表示将前 $i$ 个任务分成若干批次处理的所有方案，属性为总时间花费的最小值。

那么本题，我们就设 $dp_i$ 表示将前 $i$ 个位置分成若干个子段的所有方案，属性为总贡献的最大值。

那么我们就可以**只考虑最后一段是什么**，这也恰好是集合划分的依据：”最后一个不同的“。

那么枚举最后一段的起始位置进行转移即可。

$$dp_{i} = \max\limits_{0\le j < i}\{dp_j + f(j+1,i)\}$$

初始化 $dp_0 = 0$，预处理前缀和 $s$ 即可，时间复杂度仍然是 $\text{O}(n^2)$，所以考虑使用带 $\log$ 的数据结构维护转移。

将 $f$ 对应的三种情况对应的 DP 式子**拆开**（省略了对应情况的条件）：

$$\begin{cases}dp_i = dp_j \\ dp_i = dp_j + (i - j) & s_i > s_j\\ dp_i = dp_j - (i - j) & s_i < s_j\end{cases}$$

然后移项，把关于同一个变量的扔到一起：

$$\begin{cases}dp_i = dp_j \\ dp_i - i = dp_j - j  & s_i > s_j\\ dp_i + i = dp_j + j & s_i < s_j \end{cases}$$

发现在最后一段长度大于 $1$ 时，可能成为最优决策的只有第二个式子。

所以先处理最后一段长度等于 $1$ 的情况，然后对于第二个式子考虑优化，也就是如何快速找到能让第二个式子最优的一个决策。

在 $i$ 固定时，$i$ 是常量，$dp_j,j$ 是变量，所以决策集合维护的应当是 $dp_j - j$ 的 $\max$，并且 $s_j < s_i$。

那么只需要找到 $s_j < s_i$ 且 $i > j$ 的所有 $j$ 当中能使 $dp_j-j$ 取到最大值的一个进行转移即可。

而每次 $i + 1$ 时决策集合都只会增加 $dp_i - i$ 这个元素。

那么这个题就变得和 The Battle Of Chibi 那一题非常像了，需要的信息是：

**某一个位置** $i$ **之前**，**所有关键码小于** $i$ **的关键码的位置** $j$ 的 $dp_j - j$ 的最大值（信息）。

所以用同样的思想，我们将 $s$ 作为关键码，$dp_i - i$ 作为权值插入一个树状数组。

并且在决策完之后再插入，以保证 $i > j$ 的条件在求 $\min$ 时仍然被满足。

这里只是求前缀最值，所以树状数组是挺方便的。

时间复杂度 $\text{O}(n \log n)$。

??? note "Code"
	```cpp
	#include <bits/stdc++.h>
	 
	using namespace std;
	using i64 = long long;
	 
	const i64 si = 5e5 + 10;
	const i64 inf = 0x3f3f3f3f;
	 
	i64 T;
	i64 n;
	i64 a[si], dp[si];
	i64 sum[si], pos[si];
	 
	i64 t[si];
	inline i64 lowbit(i64 x) {
		return x & -x;
	}
	void add(i64 p, i64 x) {
		while(p <= n) {
			t[p] = max(t[p], x);
			p += lowbit(p);
		}
	}
	i64 ask(i64 p) {
		i64 res = -inf;
		while(p) {
			res = max(res, t[p]);
			p -= lowbit(p);
		}
		return res;
	}
	 
	int main() {
		cin >> T;
		while(T--) {
			std::vector<pair<i64, i64> > v;
			cin >> n;
			sum[0] = 0;
			for(i64 i = 1; i <= n; ++i)
				cin >> a[i],
				sum[i] = sum[i - 1] + a[i],
				v.push_back({sum[i], -i});
	 
			sort(v.begin(), v.end());
	 
			for(i64 i = 0; i < n; ++i) {
				pos[-v[i].second] = i + 1;
			}
	 
			for(int i = 0; i <= n; ++i) 
				t[i] = -inf;
	 
			dp[0] = 0;
			for(i64 i = 1; i <= n; ++i) {
				i64 preval;
				if(a[i] == 0) preval = 0;
				if(a[i] > 0) preval = 1;
				if(a[i] < 0) preval = -1;
	 
				dp[i] = dp[i - 1] + preval;
	            // 处理最后一段长度为 1 的情况
				dp[i] = max(dp[i], ask(pos[i]) + i);
	            // 询问最优值，转移过来
				if(sum[i] > 0) dp[i] = i;
	            // s[1 ~ i] 大于零，所以贡献是 i，必然是当前最优的。
				add(pos[i], dp[i] - i);
	            // 插入决策集合。
			}
	 
			cout << dp[n] << endl;
	 
			for(int i = 0; i <= n; ++i) 
				dp[i] = pos[i] = sum[i] = a[i] = 0;
		}
	 
		return 0;
	}
	```

```cpp
Tag : DP/数据结构优化DP/一个区间DP的经典模型
```

### CF1672F1 Array Shuffling

`Apr/23/2022`

> 给你一个序列 $a$，定义一个 $a$ 的排列 $b$ 的贡献为，通过交换 $a$ 中任意两个数若干次，得到 $b$ 的最小交换次数。
>
> 求 $a$ 的所有排列中贡献最大的那一个排列，并输出。
>
> $n \le 2e5, a_i, b_i \in [1, n]$。

这是一道关于置换环的结论题，在 zhihu 上看到了一个链接，给出了这个结论的[证明](https://blog.csdn.net/yunxiaoqinghe/article/details/113153795)

假设原序列 $a$ 是这样的：

$$1\ \ 1\ \ 4\ \ 5\ \ 1\ \ 4$$

排序后的 $b$ 是这样的

$$1\ \ 4\ \ 5\ \ 1\ \ 1\ \ 4$$

将他们放到一起：

$$a= 1\ \ 1\ \ 4\ \ 5\ \ 1\ \ 4 $$

$$b = 1\ \ 4\ \ 5\ \ 1\ \ 1\ \ 4$$

可以发现，$a_2$ 被换到了位置 $4$，$a_3$ 被换到了位置 $2$，$a_4$ 被换到了位置 $3$。

用箭头表示就是：$[3]\to [2], [2] \to [4],[4] \to [3]$，这是一个环状结构，称它为“**置换环**”。

当然，如果是自己换到自己，也可以算作一个置换环，所以这里的置换环还有 $[1] \to [1], [5] \to [5], [6] \to [6]$。

但是如果是 $a_1 = a_5 = b_1 = b_5$ 这种情况，$[1] \to [5], [5] \to [1]$ 是不能算作一个置换环的，应当单独考虑成 $[1] \to [1], [5] \to [5]$。

也就是说，如果 $a_i = b_i$ ，那么 $i$ 一定处于自己指向自己的一个置换环上。

现在又有一个结论：

> $a \to b$ 的最小交换次数 $F$，等于序列长度 $n$ 减去置换环个数 $cnt$。

另一个结论：

> 对于一个有 $k$ 个节点的置换环，使得对于环上的所有位置 $i$，由 $a_i \to b_i$ ，需要的最小交换次数是 $F=k - 1$。

这两个很容易证明。

上面说的 $a_i = b_i$，则 $i$ 自己形成一个置换环的条件也可以扩展成：

> 置换环上不能有两个 $a_i = a_j$ 的位置 $i,j$。

假设有一个置换环，换上的元素用它的权值表示：

$$5 \to 1 ,1 \to 4, 4 \to 1, 1 \to 5$$

也就是 $1 4 1 5$ 变成 $4151$。

用结论 $2$ 可以得到它的最小交换次数是 $3$。

但是，实际上只需要交换 $[2],[1]$，然后交换 $[3],[4]$，只需要 $2$ 次就可以了。

所以**置换环上不能有权值相同的元素**。

好，那么现在来看这道题，实际上就是要你求出使得 $F$ 最大的那一个排列。

写出式子：$F = n - cnt$，$n$ 是常数，所以让 $cnt$ 尽量小即可。

而置换环上不能有权值相同的元素，所以每次选取尽可能多的不同元素构造一个置换环，然后把被选中的这个子序列 **循环左移** 一位。

可以通过置换环本身的形状，证明这样子做就会让置换环上的交换次数达到 $k-1$。

然后一直这么做，直到只剩一种元素即可。

??? note "Code"
	```cpp
	#include <bits/stdc++.h>
	
	using namespace std;
	
	const int si = 2e5 + 10;
	int n;
	int a[si];
	std::vector<int> cnt[si], pos[si];
	
	#define pb emplace_back
	#define sz(v) ((int)v.size())
	
	int main() {
		int T;
		cin >> T;
		while(T--) {
			vector<int> tmp, ans;
			cin >> n;
			tmp.clear(), ans.clear();
			ans.resize(n + 1);
			for(int i = 1; i <= n; ++i)
				cin >> a[i];
	
			for(int i = 1; i <= n; ++i) 
				cnt[i].clear(), pos[i].clear();
			for(int i = 1; i <= n; ++i)
				cnt[a[i]].pb(i);
			for(int i = 1; i <= n; ++i)
				pos[sz(cnt[i])].pb(i);
	
			for(int i = 1; i <= n; ++i)
				if(sz(pos[i]) > 0) tmp.pb(i);
			vector<int> qwq;
			for(int i = 1; i <= n; ++i) {
				qwq.clear();
				for(auto j : tmp) {
					if(j >= i) {
						for(int k = 0; k < sz(pos[j]); ++k) {
							qwq.pb(cnt[pos[j][k]][sz(cnt[pos[j][k]])- 1]);
							cnt[pos[j][k]].pop_back();
						}
					}
				} 
				if(sz(qwq) > 0) {
					for(int j = 0; j < sz(qwq) - 1; ++j) 
						ans[qwq[j + 1]] = a[qwq[j]];
					ans[qwq[0]] = a[qwq.back()];
				}
			}
	
			for(int i = 1; i <= n; ++i) 
				cout << ans[i] << " ";
			cout << endl;
		}
		return 0;
	}
	// tjx
	```

```cpp
Tag : 置换环
```

### CF1672F2 Checker for Array Shuffling

`Apr/27/2022`

> 你需要写一个 F1 的Checker，即判定给定的序列 $b$ 是不是 $a$ 的最优解之一。

依旧考虑 F1 的结论，最优解就是让置换环数量更小，构造尽可能大的置换环。

换句话说，考虑 F1 的构造过程，你发现出现次数最多的那个元素在任意一个置换环上都要出现，否则不是最优的。

因为假设这个元素选完之后，还出现了另外的置换环，证明这个元素必然不是出现次数最多的那个。

所以先从编号为 $b_i$ 的点连向 $a_i$，（以值域大小为节点个数），删去出现次数最多的那个。

!!! Warning "tips"
	
	一般找置换环的方式是对于序列 $a$ 记录一个 $lst(a_i)$ 表示 $a_i$ 出现的位置。

	然后扫一遍 $b$，只要 $lst(b_i)$ 不等于 $i$，那么就给 $i$ 和 $lst(b_i)$ 连一条有向边即可。

	只是这题比较特殊, 值域是 $[1, n]$，所以可以直接连 $a_i, b_i$ （只要不是自环）。

如果图中还存在环，那么必然不是最优的，输出 `WA`，否则 `AC`。

使用 Tarjan 即可线性判是否有判环（判断 SCC 的大小是否 $>1$），但是 Tarjan 处理不了自环，自环需要特判。

??? note "Code"
	```cpp
	
	#include <stack>
	#include <cassert>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define yuyuko "AC"
	#define kawaii "WA"
	
	using namespace std;
	using i64 = long long;
	
	const int si = 2e5 + 10;
	
	int n, a[si], b[si];
	int exist[si];		// 出现次数
	int most;
	bool visited[si];   // 是否访问过这个节点
	
	int tot = 0;
	int head[si];
	struct Edge {
		int ver, Next;
	}e[si << 1];
	inline void add(int u, int v) {
		e[tot] = (Edge){v, head[u]}, head[u] = tot++;
	}
	
	int dfn[si], low[si];
	int tim = 0, cnt = 0;
	bool ins[si];
	stack<int>s;
	bool sol;
	
	void init() {
		tot = 0, tim = 0, cnt = 0, sol = true;
		for(int i = 0; i <= n; ++i) {
			head[i] = -1;
			exist[i] = dfn[i] = low[i] = 0;
			visited[i] = ins[i] = false;
		}
	}
	
	void tarjan(int u) {
		dfn[u] = low[u] = ++tim;
		s.push(u), ins[u] = true;
	
		for(int i = head[u]; ~i; i = e[i].Next) {
			int v = e[i].ver;
			if(!dfn[v]) {
				tarjan(v);
				low[u] = min(low[u], low[v]);
			}
			else if(ins[v]) 
				low[u] = min(low[u], dfn[v]);
		}
	
		if(dfn[u] == low[u]) {
			int x; ++cnt;
			int num = 0;
			do {
				x = s.top(), s.pop();
				ins[x] = false, num++;
			} while(u != x);
			if(num > 1)
				sol = false; 
		}
		return ;
	}
	
	void solve() {
		cin >> n, init();
		for(int i = 1; i <= n; ++i)
			cin >> a[i], 
			exist[a[i]] += 1;
	
		most = max_element(exist + 1, exist + 1 + n) - exist;
		// 出现最频繁的是哪个
	
		// cerr << most << endl;
	
		for(int i = 1; i <= n; ++i) {
			cin >> b[i];
			if(a[i] == most || b[i] == most) 
				continue;
			if(a[i] == b[i])
				sol = false;
			// Tarjan 判不了自环，所以特判。
			add(b[i], a[i]);
		}
	
		for(int i = 1; i <= n; ++i) 
			if(!dfn[i] && i != most) 
				tarjan(i);
	
		if(sol) 
			cout << yuyuko << endl;
		else 
			cout << kawaii << endl;
		return;
	}
	
	int main() {
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		int T; cin >> T;
		while(T--)
			solve();
	
		return 0;
	}
	```

```cpp
Tag : 置换环/Tarjan
```

### CF1672E notepad.exe

`Apr/24/2022`

> 给你一个文本编辑器，有 $n$ 个字符串，长度分别为 $w_1,w_2,w_3\dots$。
>
> $w$ 只有 Grader 才知道，你每次可以询问一个宽度 $W$，Grader 会返回以这个宽度显示所有字符串所需要的最小高度。
>
> 字符串的显示必须按顺序，同一行内的相邻两个字符串之间至少要有一个空格，行内可以有任意多个空格。
>
> 如果 $W < \max\{w_i\}$，文本编辑器会 Crash，Grader 会返回 $0$。
>
> 问可能的 $H\times W$ 最小是多少，你最多可以问 $n + 30$ 次。
>
> $n,w_i \le 2000$。

首先看到 $30$，可以考虑这样的情况，让所有字符串都在一行，二分使得这种情况最小的 $W$，设这个 $W = S$

可以发现 $S = \sum w_i + n - 1$，也就是从长度加上行内空格。

又可以发现，要想最优，$H$ 一定在 $[1,n]$ 这个范围之内。

假设我们二分用完了 $30$ 次（当然是不可能用完的）。

然后还剩 $n$ 次，刚好够每一个可能的 $H$ 都问一遍。

所以二分出 $S$ 之后直接暴力问 $W = S/H$ 的情况（此处的除法是 C++ 的除法）

只要编辑器没有 Crash，统计答案即可。

??? note "Code"
	```cpp
	int l = 1, r = 2011 * 2011;
	while(l < r) {
		int mid = l + r >> 1;
		if(ask(mid) == 1)
			r = mid;
		else 
			l = mid + 1;
	}
	
	int ans = l;
	
	for(int i = 1; i <= n; ++i) {
		int x = ask(l / i);
		if(x && (l / i) != 0) 
			ans = min(ans, (l / i) * x);
	}
	cout << "! " << ans << endl;
	```

```cpp
Tag : 二分/构造/贪心
```

### CF1661D Progressions Covering

`Apr/25/2022`

> 给你一个序列 $a$，初始全部为 $0$，给你一个序列 $b$。
>
> 你可以对 $a$ 做任意次这样的操作：
>
> + 对于一个长度为 $k$ 的区间 $[l,r]$，分别让 $a_l,a_{l + 1},\dots,a_{r}$ 加上 $1,2,3,\dots,k$。
>
> 问使得 $\forall i,a_i \ge b_i$ 的最小操作次数。

这是一个关于**等差数列**的经典 Trick。

区间 $[l,r]$ 加等差数列，等同于在**差分数组**上的 $[l + 1,r]$ 做一次**区间加** $d$，然后令 $c[l] + \text{BEGIN}$，$c[r+1] - \text{END}$。

$\text{BEGIN,END}$ 分别是首项和末项。

单点询问只需要询问线段树上的 $sum(1,pos)$ 即可。

发现序列 $a$ 开头的元素只能一个一个减去，结尾的元素只能 $k$ 个 $k$ 个减去，

最终结果要求最小，所以尽量让每一个元素都多减去大一点的。

考虑一个贪心，从结尾开始扫，如果当前位置的值 $a_i$ 不满足条件，不断加 $k$ 直到 $b_i \le a_i$。

然后这些操作对于前面的贡献也需要算上。

扫的时候累加答案即可。

代码直接写了差分。

??? note "Code"
	```cpp
	
	#include <bits/stdc++.h>
	
	using namespace std;
	
	int main() {
	
	    int n, k;
	    scanf("%d %d", &n, &k);
	    vector<long long> b(n);
	    for (auto &it : b) {
	        scanf("%lld", &it);
	    }
	    
	    vector<long long> closed(n);
	    long long sum = 0, cnt = 0, ans = 0;
	    for (int i = n - 1; i >= 0; --i) {
	        sum -= cnt;
	        cnt -= closed[i];
	        b[i] -= sum;
	        if (b[i] <= 0) {
	            continue;
	        }
	        int el = min(i + 1, k);
	        long long need = (b[i] + el - 1) / el;
	        sum += need * el;
	        cnt += need;
	        ans += need;
	        if (i - el >= 0) {
	            closed[i - el] += need;
	        }
	    }
	    
	    printf("%lld\n", ans);
	    
	    return 0;
	}
	```

```cpp
Tag : 差分/线段树/贪心/等差数列
```

### CF1661E Narrow Components

`Apr/27/2022`

> 给你一个 $3\times m$ 的 $01$ 矩阵，每次询问一个区间 $[l,r]$，求第 $l,r$ 这两列之间有多少个 $1$ 连通块。
>
> $m,q \le 3e5$。

一个一眼的思路是，用数位DP中类似前缀和的思想，把一个 $[l,r]$ 的询问转化成 $[1,l],[1,r]$ 的两个询问。

所以设 $s_i$ 表示 $1\sim i$ 的连通块个数。

但是这里发现不能直接减，因为当两列断开的时候，有可能会产生新的连通块，或者令连通块数量减少。

既然有这种问题，就尝试解决，因为前缀和的思路还是蛮对的，放弃了估计一时半会儿想不到别的办法。

所以可以预处理出一个 $m_i$，表示 $i,i+1$ 这两列断开的时候，会产生的新连通块个数。

然后询问的时候带上 $m$ 即可。

```cpp
for (int i = 0; i < n; i++) {
    s1[i + 1] = s1[i] + a[0][i] + a[1][i] + a[2][i] - (a[0][i] & a[1][i]) - (a[1][i] & a[2][i]);
}
for (int i = 0; i < n - 1; i++) {
    s2[i + 1] = s2[i] - (a[0][i] & a[0][i + 1]) - (a[1][i] & a[1][i + 1]) - (a[2][i] & a[2][i + 1])
    + (a[0][i] & a[1][i] & a[0][i + 1] & a[1][i + 1])
    + (a[1][i] & a[2][i] & a[1][i + 1] & a[2][i + 1]);
}

// when asking 
ans = s1[r] - s1[l] + s2[r - 1] - s2[l];
```

但是如果遇到：

```
1 1 1 1 1 1 1 1
1 0 0 0 0 0 0 0
1 1 1 1 1 1 1 0
```

就会 G，所以需要分类讨论有 $101$ 的情况。

```cpp
for (int i = 0; i < n - 1; i++) {
    if (a[0][i] & a[1][i] & a[2][i]) {
        int j = i + 1;
        while (j < n && (a[0][j] & !a[1][j] & a[2][j]))
            j++;
        if (j < n && j > i + 1 && a[0][j] && a[1][j] && a[2][j]) {
            R[j]++;
            if (i + 1 < n)
                L[i + 1]++;
        }
        i = j - 1;
    }
}
for (int i = 1; i < n; i++)
    L[i] += L[i - 1],
    R[i] += R[i - 1];
```

本题还有线段树+并查集维护连通性的做法，并且有

[SDOI2013]城市规划这一道类似的题目（本题只能使用线段树+并查集维护）。

```cpp
Tag : 前缀和/思维
```

### CF1671E Preorder

`Apr/28/2022`

> 给你一棵有 $2^n-1$ 个节点的满二叉树，每个节点上只可能有 `A/B` 两种值。
>
> 你可以对做任意次这样的操作：
>
> 对于一个非叶子节点 $u$，交换以它的左右儿子 $2u,2u+1$ 为根的子树。
>
> 问可能得到的前序遍历有多少种。
>
> $n \le 18$。

比较 Tricky 的 Problem。

考虑设 $dp_u$ 表示以 $u$ 为根的子树中有多少种可能方案。

注意到题目中对前序遍历的定义是，$u$ 上**的字符 + 左儿子的前序遍历 + 右儿子的前序遍历**。

所以我们可以把 $2u,2u+1$ 的 **DP 值看作常量**来考虑。

考虑划分集合 $dp_u$。

如果说两颗子树本质不同（即是不同构），那么不交换的时候，由乘法原理可以得到，方案数为 $dp_{2u}\times dp_{2u+1}$。

交换之后又有一个 $dp_{2u}\times dp_{2u+1}$。

所以两颗子树不同构时，$dp_{u} = 2\times dp_{2u} \times dp_{2u + 1}$。

如果两颗子树不同构，方案数就只有 $dp_{2u} \times dp_{2u + 1}$。

+ 但是有个问题：

那如果不同构的时候，$2u$ 的所有方案中有一种和 $2u + 1$ 里的一种方案完全一致。

不会算重吗？

其实不会，你可以发现，出现这种情况的充要条件就是两颗子树同构。

具体证明只需要**先从儿子为叶子节点的节点的情况**开始（或者说就在这里先猜一个结论），然后**不断往上走，递归证明结论**。

+ 怎么判断同构？

其实只需要让递归时，记录一个新的 $preorder$，并强制这个 $preorder$ 是字典序最大的那一个，然后因为这是满二叉树，判断同构只需要判断字符串是否相等即可。

所以就不用写树哈希了。

??? note "Code"
	```cpp
	
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	using i64 = long long;
	
	const int si = (1 << 18) + 1;
	constexpr int mod = 998244353;
	
	int n, ans = 1;
	string preorder;
	string cons[si];
	
	void dfs(int p, int len) {
		if(len == 1) {
			cons[p] = preorder[p];
			return;
		}
	
		dfs(p << 1, len / 2), dfs(p << 1 | 1, len / 2);
		if(cons[p << 1] > cons[p << 1 | 1])
			swap(cons[p << 1], cons[p << 1 | 1]);
		if(cons[p << 1] != cons[p << 1 | 1]) 
			ans = (ans + ans) % mod;
		cons[p] = preorder[p] + cons[p << 1] + cons[p << 1 | 1];
	}
	
	int main() {
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n >> preorder;
		preorder = ' ' + preorder;
	
		dfs(1, (1 << n) - 1);
	
		cout << ans << endl;
	
		return 0;
	}
	```

```cpp
Tag : 树形DP/树的同构/树的前序遍历
```

### CF1668E & CF1667C Half Queen Cover

`Apr/28/2022`

> 给你一个 $n\times n$ 的棋盘，和无限个皇后，但是这里的皇后只能攻击同行列和从左上到右下的对角线。
>
> 另外一条对角线攻击不到，问最少需要多少个皇后才能覆盖整个棋盘，皇后之间是否攻击不管。
>
> $1\le n \le 1e5$，构造解。

一个很妙的构造题，作为题本身是很妙的，但是这种东西放在 div2 E 我觉得很烦。

考虑最优解放了 $k$ 个 Queen，且不考虑对角线，那么可以把 Queen 移动成这样：

```
x o o o o o o
o x o o o o o
o o x o o o o 
o o o . . . .        x 是 Queen
o o o . . . .        o 是 Under
o o o . . . .        Attack 的格子
o o o . . . .  
```

可以发现，现在剩下了一个 $(n-k) \times (n -k)$ 的矩阵。

必然是用 Queen 的对角线来覆盖，所以需要 $2\times (n - k) - 1$ 个 Queen。

所以可以列出不等式：$2\times(n - k) - 1 \le k$。

可以得到 $k = \lceil \frac{2n - 1}{3} \rceil$。

那么最少需要的 Queen 的个数就是 $\lceil \frac{2n - 1}{3} \rceil$ 个。

至于构造解，只需要分别让他们覆盖一个对角线即可。

但是还需要保证那个 $(n-k) \times (n - k)$ 矩阵之外的地方都要被 Attack。

所以还需要保证这些 Queen 互相不在同行列上（在左上的 $k\times k$ 的地方放）。

这里的一个 Trick 是，把 Queen 当成国际象棋里面的 Knight 来移动。

比如 $5 \times 5$ 的时候：

```
x . . . . 
. . . x . 
. x . . . 
. . . . x 
. . x . . 
```

然后就做完了。

??? note "Code"
	```cpp
	
	#include <cmath>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	
	int n, k;
	
	int main() {
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n;
	
		if(n == 1) {
			cout << "1\n" << "1 1 \n";
			return 0; 
		}
	
		k = ceil(((2 * n - 1) * 1.0) / 3.0);
	
		cout << k << endl;
	
		for(int i = 1, j = 1; i <= k; ++i) {
			cout << i << " " << j << endl;
			j = (j + 2 > k) ? 2 : j + 2;
		}
		return 0;
	}
	```

```cpp
Tag : 构造/思维
```

### CF1665D GCD Guess

> Grader 有一个整数 $x, 1\le x \le 10^9$。
> 
> 你有 $30$ 次机会，每次可以询问 $\gcd(x + a, x + b)$ 是多少，$|a,b| \le 2\times 10^9$。
> 
> 请你问出 $x$。

发现 $\lceil \log_2(10^9) \rceil = 30$，所以我们可以考虑二分或者对 $x$ 二进制拆分。

二分显然不可行，因为 $\gcd$ 在本题的要求下是没有单调性质的，我赛时就是被这个卡住了，一直没有想到他没有单调性然后取考虑二进制拆分。

所以对 $x$ 二进制拆分，我们要做的就是靠一次询问问出 $x$ 的某一位是 $0$ 还是 $1$。

做法很简单，把 $i$ 位以前的位全部置为 $0$，设现在的数是 $x\prime$。

每次询问 $\gcd(x\prime + 1, 2^{i + 1}) = 2^{i + 1}$ 是否成立即可。

如果成立，则第 $i$ 位是 $1$，反之为 $0$。

然后题目要求问的是 $\gcd(x + a, x + b)$，且 $a,b$ 可以是负数，

所以记录一个变量 $r$，表示当前一共减去了多少。

然后询问 $a = - r + 2^{i - 1}, b = 2^{i} + a$ 即可。

这个是更相减损术的结论：$\forall a \ge b \in \mathbb{N}, \gcd(a, b) = \gcd(b, a - b) = \gcd(a, a - b)$。

??? note "Code"
	```cpp
	int n, m;
	int a[si], b[si];
	
	int query(int a, int b) {
		cout << "? " << a << " " << a + b << endl;
		int get; cin >> get; return get;
	}
	
	void solve() {
		int r = 0;
		for(int i = 1; i <= 30; ++i) {
			int ans = query((1 << (i - 1)) - r, (1 << i));
			if(ans == (1 << i)) r += (1 << (i - 1));
		} 
		cout << "! " << r << endl;
	}
	```

```cpp
Tag : gcd/数论
```

### ABC247F Cards

> 有 $n$ 张卡片,每张卡片正面有一个数字 $a_i$,背面也有一个数字 $b_i$,
>
> 保证所有牌中正面和反面出现的数字都是一个排列,现在想要取一些牌,这些牌正反面必须包含 $1 \sim n$ 的所有数字,求方案数.

可以把牌看作连接 $a_i, b_i$ 两个节点的边，

每个点的出入度就必然为 $1$。

然后原图转化成多个不连通的环，方案数就是他们的各自的方案乘起来（乘法原理）.

考虑计算每一个环的答案。

设 $dp_{i,0/1}$ 表示选 $i$ 或者不选 $i$ 的方案。

如果 $i$ 选了，那么和 $i$ 相同（在环上也应该是相邻的），的节点就可选可不选。

这个是一个状态机模型，环的处理就强制选 $1$，强制不选 $1$ 分别跑一遍。

第一种情况 $n$ 随意取，第二种必须取。

然后特判下自环就行了。

??? note "Code"
	```cpp
	#include<iostream>
	#include<cstring>
	#include<algorithm>
	using namespace std;
	typedef long long LL;
	const int maxn = 2e5 + 5, mod = 998244353;
	int a[maxn], b[maxn];
	bool v[maxn];
	LL f1[maxn][2];
	LL f2[maxn][2];
	#define x first
	#define y second
	
	int main(){
	    int n;
	    cin >> n;
	    f1[1][0] = 1;
	    for(int i = 2; i <= n; i++){
	        f1[i][0] = f1[i - 1][1];
	        f1[i][1] = (f1[i - 1][0] + f1[i - 1][1]) % mod;
	    }
	    f2[1][1] = 1;
	    for(int i = 2; i <= n; i++){
	        f2[i][0] = f2[i - 1][1];
	        f2[i][1] = (f2[i - 1][0] + f2[i - 1][1]) % mod;
	    }
	    for(int i = 1; i <= n; i++) scanf("%d", &a[i]);
	    for(int i = 1; i <= n; i++){
	        int x;
	        scanf("%d", &x);
	        b[a[i]] = x;
	    }
	    LL res = 1;
	    for(int i = 1; i <= n; i++){
	        if (!v[i]){
	            int cnt = 0;
	            for(int j = i; !v[j]; j = b[j]){
	                v[j] = 1;
	                cnt++;
	            }
	            if (cnt != 1) res = res * (f1[cnt][1] + f2[cnt][0] + f2[cnt][1]) % mod;
	        }
	    }
	    cout << res << '\n';
	}
	// 贺的
	// https://zhuanlan.zhihu.com/p/496253093
	```	

```cpp
Tag : DP/状态机/图论关系
```

### ABC246F typewriter

> 给你 $N(1\le N \le 18)$ 个集合 $S_i \in \{\texttt{a} \sim \texttt{z}\}$。
> 
> 你可以选择任意一个字符集 $S_i$，然后用它的字符打一个长度为 $L \le 10^9$ 的字符串。
> 
> 问可能的方案数，对 $998244353$ 取模。

发现计算一个集合能打出多少个字符串是很简单的，对于每个位置乘法原理即可。

但是麻烦的地方就在于计算重复，怎么搞呢？

我们考虑设 $A_i$ 表示 $S_i$ 能打出来的字符串的集合。

那么最终我们要求的答案就是 ：

$$|A_1 \cup A_2 \cup A_3 \cup \dots \cup A_n|$$

发现这个式子很容易使用容斥原理计算。

$$|A_1 \cup A_2 \cup A_3 \cup \dots \cup A_n| = $$

$$\sum\limits_{i = 1}^{n} |A_i| - \sum\limits_{1\le i < j \le n} |A_i \cap A_j| + \sum\limits_{1\le i < j < k \le n} |A_i \cap A_j \cap A_k| \dots + (-1)^{n + 1} |A_1 \cap A_2 \cap \dots \cap A_n|$$

所以我们要做的就是计算任意两个集合 $A_i,A_j$ 的交集。

发现直接计算很不好搞，我们发现 $S_i, S_j$ 大小很小，分析一波性质可以发现：

$A_i \cap A_j = A_{|S_i \cap S_j|}$。

也就是先对 $S_i, S_j$ 求个交集，再看这个交集能生成的字符串数量是多少。

然后这个题就很简单了，求交集是 $\text{O}(2^{\text{|S|}})$ 的。

然后这玩意儿就能很快求了。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	#define mod 998244353

	using namespace std;

	long long power(long long a,long long b){
	  long long x=1,y=a;
	  while(b>0){
	    if(b&1ll){
	      x=(x*y)%mod;
	    }
	    y=(y*y)%mod;
	    b>>=1;
	  }
	  return x%mod;
	}

	int main(){
	  int n,k;
	  cin >> n >> k;
	  vector<int> v(n,0);
	  for(int i=0;i<n;i++){
	    string s;
	    cin >> s;
	    for(auto &nx : s){v[i]|=(1<<(nx-'a'));}
	  }

	  long long res=0;
	  for(int i=1;i<(1<<n);i++){
	    int ch=(1<<26)-1;
	    for(int j=0;j<n;j++){
	      if(i&(1<<j)){ch&=v[j];}
	    }
	    int pc=__builtin_popcount((unsigned int)ch);
	    if(__builtin_popcount((unsigned int)i)%2){res+=power(pc,k);res%=mod;}
	    else{res+=(mod-power(pc,k));res%=mod;}
	  }

	  cout << res << '\n';
	  return 0;
	}
	```

```cpp
Tag : 容斥原理
```