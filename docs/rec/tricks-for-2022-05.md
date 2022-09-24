## 五月 Tricks 整理

### CF833B The Bakery

> 给你一个序列 $a$，长度为 $n$。
> 
> 定义一个区间的贡献 $F(l, r)$ 为，这个区间中不同的数的个数。
> 
> 要求你将序列划分为 $m$ 个非空区间，使得 $\sum F(l, r)$ 最大。
> 
> $1\le n \le 3.5e4, 1\le a_i \le n, 1\le m \le 50$

本题是一个很经典的划分型区间 DP 问题，最近遇到了很多次。

不过和最近遇到的不一样，本题要求你必须分成 $m$ 个非空区间。

所以我们可以设出状态 ： $dp_{i,j}$ 表示考虑前 $i$ 个，分成 $j$ 段的最大贡献和。

可以很容易的写出方程：

$$dp_{i, j} = \max\limits_{j - 1 \le k < i}\{dp_{k, j - 1} + f(k + 1, i)\}$$

也就是枚举上一段的右端点在哪里进行转移。

因为这里不是前缀 $\max$ 而是一段的 $\max$，所以朴素算法的复杂度只能是 $\text{O}(n^3)$ 的。

状态已经是最精简的了，无法优化（滚动数组是对空间的优化而不是对时间）。

所以考虑对决策集合进行优化，发现这里如果固定 $i$，当 $j + 1$ 的时候，决策集合会整个变化，不好搞。

发现这里 $j$ 的状态只会由 $j - 1$ 转移而来，所以 $j$ 也可以作为阶段保证无后效性。

那么我们把 $j$ 放到外层，固定 $j$，观察决策集合。

当 $i + 1$ 的时候，决策集合只增加一个 $dp_{i, j - 1}$。

这里又是在询问区间 $\max$，所以我们可以考虑使用线段树优化。

但是最棘手的是 $F$，这个东西既没有单调性又没有什么快速的计算方式（最好的也是主席树这种，本题明显空间不够），

所以要观察 $F$ 是否具有什么性质。

因为 $F$ 是不同的数的个数，所以一个数 $a_i$，他能做 $1$ 的贡献的位置就是 $(pre_{a_i}, i]$。

也就是上一个 $a_i$ 这个元素出现的位置 $+1$，到它本身的位置。

并且因为 $1\le a_i \le n$，所以我们可以对于每一个位置 $i$，都算一下这个位置在哪里做了贡献，然后给这一段区间加 $1$。

那么我们发现，其实可以直接利用线段树维护 $dp$ 的第一维。

然后算每个位置的贡献，区间加直接在线段树上给 $dp_{j - 1}$ 系的状态都算一次即可。

转移直接询问 $\max$，对于每层循环 $j$，里面都重新建树（或者直接修改），然后把 $j - 1$ 系的 DP 值放到线段树上。

复杂度 $\text{O}(m\times n \log n)$，可以通过本题。

??? note "Code"

	```cpp
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const int si = 3.5e4 + 10;
	const int inf = 0x3f3f3f3f;
	
	int n, m;
	int last[si];
	int pre[si], a[si];
	int dp[si][50 + 10];
	
	class Segment_Tree {
		private : 
			struct Node {
				int l, r;
				int dat, tag;
			}t[si << 2];
			inline void pushup(int p) {
				t[p].dat = max(t[p << 1].dat, t[p << 1 | 1].dat);
			}
			inline void pushdown(int p) {
				if(t[p].tag) {
					t[p << 1].tag += t[p].tag, t[p << 1 | 1].tag += t[p].tag;
					t[p << 1].dat += t[p].tag, t[p << 1 | 1].dat += t[p].tag;
					t[p].tag = 0;
				}
			}
		public : 
			void build(int p, int l, int r) {
				t[p].l = l, t[p].r = r, t[p].tag = 0;
				if(l == r) {
					t[p].dat = 0;
					return;
				}
				int mid = (l + r) >> 1;
				build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
				pushup(p);
			}
			void modify(int p, int x, int v) {
				t[p].tag = 0;
				if(t[p].l == t[p].r) {
					t[p].dat = v;
					return;
				}
				int mid = (t[p].l + t[p].r) >> 1;
				if(x <= mid) modify(p << 1, x, v);
				else modify(p << 1 | 1, x, v);
				pushup(p); 
			}
			void update(int p, int l, int r, int v) {
				if(l <= t[p].l && t[p].r <= r) {
					t[p].dat += v, t[p].tag += v;
					return;
				}
				pushdown(p);
				int mid = (t[p].l + t[p].r) >> 1;
				if(l <= mid) 
					update(p << 1, l, r, v);
				if(r > mid)
					update(p << 1 | 1, l, r, v);
				pushup(p);
			}
			int query(int p, int l, int r) {
				int res = -inf;
				if(l <= t[p].l && t[p].r <= r) 
					return t[p].dat;
				pushdown(p);
				int mid = (t[p].l + t[p].r) >> 1;
				if(l <= mid) 
					res = max(res, query(p << 1, l, r));
				if(r > mid)
					res = max(res, query(p << 1 | 1, l, r));
				return res;
			}
	};
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		Segment_Tree tr;
	
		cin >> n >> m;
		memset(dp, 0, sizeof dp);
		for(int i = 1; i <= n; ++i) {
			cin >> a[i];
			if(!last[a[i]]) last[a[i]] = i;
			else {
				pre[i] = last[a[i]];
				last[a[i]] = i;
			}
		}
	
		// for(int i = 1; i <= n; ++i) 
		// 	meow(pre[i]) << endl;
	
		for(int j = 1; j <= m; ++j) {
			tr.build(1, 0, n);
			for(int i = 1; i <= n; ++i) {
				tr.modify(1, i, dp[i][j - 1]);
			}
			for(int i = 1; i <= n; ++i) {
				tr.update(1, pre[i], i - 1, 1);
				dp[i][j] = tr.query(1, j - 1, i - 1);
			}
		}
	
		cout << dp[n][m] << endl;
	
		return 0;
	}
	```

```cpp
Tag : 线段树优化DP/数据结构维护下标/一个区间DP模型的优化
```

### CF1591F Non-Equal Neighbours

> 给你一个序列 $a$，长度为 $n$。
> 
> 要求你构造一个序列 $b$，使得 $1\le b_i \le a_i$，且 $b_i \not = b_{i + 1}$。
> 
> 问方案数对 $998244353$ 取模。
> 
> $1\le n \le 2\times 10^5, 1\le a_i \le 10^9$。

考虑一个暴力的 DP。

设 $dp_{i, j}$ 表示前 $i$ 个位置，且第 $i$ 个 位置选 $j$ 的方案数。

然后可以得到方程：

$$dp_{i, j} = \sum\limits_{k = 1}^{a_{i - 1}} dp_{i - 1, k} - dp_{i - 1, j}$$

发现这个东西就是算上一个阶段的和，然后扣掉上一个阶段的 $j$ 位置。

考虑使用动态开点线段树维护第二维。

所有转移直接在线段树上修改。

所以可以先算出 $sum = \sum\limits_{k = 1}^{a_{i - 1}} dp_{i - 1, k}$，

让线段树上 $[1, a_{i - 1}]$ 全部取反，然后给 $[1, a_{i}]$ 全部加上 $sum$。

$(a_i, 10^9]$ 全部赋值为 $0$ 即可。

只需要一个区间加，区间查，区间乘 $0, -1$ 的 lazytag 线段树即可。

类似线段树 2。

但是注意，因为动态开点需要很大的空间，所以需要卡一卡，具体可以参见日常随笔的 “卡常小技巧”。

??? note "Code"

	```cpp
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define ls lson[p]
	#define rs rson[p]
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	
	const int si = 2e5 + 10;
	const int sii = 1e7 + 1;
	constexpr int mod = 998244353ll;
	
	int n;
	int a[si];
	
	int lson[sii], rson[sii], dat[sii], add[sii], mul[sii];
	// 空间一定要开够。
	// 开到 class 里面的时候要提前算好空间。
	// 如果空间比较卡，就要提出来写。！！！
	int cnt_node = 0;
	inline int Newnode() {
		return ++cnt_node;
	}
	inline void pushup(int p) {
		dat[p] = (dat[ls] + dat[rs]) % mod;
	}
	inline void pushdown(int p, int l, int r) {
		if(!add[p] && mul[p] == 1) 
			return;
		
		if(!ls) ls = Newnode();
		if(!rs) rs = Newnode();
	
		int mid = (l + r) >> 1;
	
		dat[ls] = (1ll * dat[ls] * mul[p] + 1ll * add[p] * (mid - l + 1)) % mod;
		dat[rs] = (1ll * dat[rs] * mul[p] + 1ll * add[p] * (r - mid - 1 + 1)) % mod;
		add[ls] = (1ll * add[ls] * mul[p] + 1ll * add[p]) % mod;
		add[rs] = (1ll * add[rs] * mul[p] + 1ll * add[p]) % mod;
		mul[ls] = (1ll * mul[ls] * mul[p]) % mod;
		mul[rs] = (1ll * mul[rs] * mul[p]) % mod;
	
		add[p] = 0, mul[p] = 1;
	}
	void update_add(int &p, int l, int r, int ql, int qr, int v) {
		if(l > r) return;
		if(!p) p = Newnode();
		if(ql <= l && r <= qr) {
			add[p] = (add[p] + v) % mod;
			dat[p] = (dat[p] + 1ll * v * (r - l + 1)) % mod;
			return;
		}
		pushdown(p, l, r);
		int mid = (l + r) >> 1;
		if(ql <= mid) 
			update_add(ls, l, mid, ql, qr, v);
		if(qr > mid)
			update_add(rs, mid + 1, r, ql, qr, v);
		pushup(p); return;
	}	
	void update_mul(int &p, int l, int r, int ql, int qr, int v) {
		if(l > r) return;
		if(!p) p = Newnode();
		if(ql <= l && r <= qr) {
			dat[p] = (dat[p] * 1ll * v) % mod;
			add[p] = (add[p] * 1ll * v) % mod;
			mul[p] = (mul[p] * 1ll * v) % mod;
			return; 
		}
		pushdown(p, l, r);
		int mid = (l + r) >> 1;
		if(ql <= mid)
			update_mul(ls, l, mid, ql, qr, v);
		if(qr > mid)
			update_mul(rs, mid + 1, r, ql, qr, v);
		pushup(p); return;
	}
	int query(int p, int l, int r, int ql, int qr) {
		if(l > r) return 0ll;
		if(!p) return 0ll;
		if(ql <= l && r <= qr) 
			return dat[p] % mod;
		pushdown(p, l, r);
		int mid = (l + r) >> 1;
		int res = 0ll;
		if(ql <= mid) 
			res = (res + query(ls, l, mid, ql, qr)) % mod;
		if(qr > mid)
			res = (res + query(rs, mid + 1, r, ql, qr)) % mod;
		return res % mod;
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		int root = 0;
		cin >> n;
		for(int i = 1; i <= n; ++i)
			cin >> a[i];
		update_add(root, 1, 1e9, 1, a[1], 1);
		for(int i = 2; i <= n; ++i) {
			int sum = query(root, 1, 1e9, 1, 1e9) % mod;
			update_mul(root, 1, 1e9, 1, a[i], -1);
			update_add(root, 1, 1e9, 1, a[i], sum);
			update_mul(root, 1, 1e9, a[i] + 1, 1e9, 0);
		}
	
		cout << (query(root, 1, 1e9, 1, 1e9) + mod) % mod << endl;
	
		return 0;
	}
	
	/*
	// 被卡空间的代码。
	
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define ls t[p].lson
	#define rs t[p].rson
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const int si = 2e5 + 10;
	constexpr i64 mod = 998244353ll;
	
	int n;
	i64 a[si];
	
	class Segment_Tree {
		private : 
			struct Node {
				int lson = 0, rson = 0;
				i64 dat = 0ll, add = 0ll, mul = 1ll;
			}t[si * 60];
			// 空间一定要开够。
			int cnt_node = 0;
			inline int Newnode() {
				return ++cnt_node;
			}
			inline void pushup(int p) {
				t[p].dat = t[ls].dat + t[rs].dat;
			}
			inline void pushdown(int p, int l, int r) {
				if(!t[p].add && t[p].mul == 1) 
					return;
				
				if(!ls) ls = Newnode();
				if(!rs) rs = Newnode();
	
				int mid = (l + r) >> 1;
	
				t[ls].dat = (t[ls].dat * t[p].mul + t[p].add * (mid - l + 1)) % mod;
				t[rs].dat = (t[rs].dat * t[p].mul + t[p].add * (r - mid - 1 + 1)) % mod;
				t[ls].add = (t[ls].add * t[p].mul + t[p].add) % mod;
				t[rs].add = (t[rs].add * t[p].mul + t[p].add) % mod;
				t[ls].mul = (t[ls].mul * t[p].mul) % mod;
				t[rs].mul = (t[rs].mul * t[p].mul) % mod;
	
				t[p].add = 0ll, t[p].mul = 1ll;
			}
		public : 
			void update_add(int &p, int l, int r, int ql, int qr, i64 v) {
				if(l > r) return;
				if(!p) p = Newnode();
				if(ql <= l && r <= qr) {
					t[p].add = (t[p].add + v) % mod;
					t[p].dat = (t[p].dat + v * (r - l + 1)) % mod;
					return;
				}
				pushdown(p, l, r);
				int mid = (l + r) >> 1;
				if(ql <= mid) 
					update_add(ls, l, mid, ql, qr, v);
				if(qr > mid)
					update_add(rs, mid + 1, r, ql, qr, v);
				pushup(p); return;
			}	
			void update_mul(int &p, int l, int r, int ql, int qr, i64 v) {
				if(l > r) return;
				if(!p) p = Newnode();
				if(ql <= l && r <= qr) {
					t[p].dat = (t[p].dat * v) % mod;
					t[p].add = (t[p].add * v) % mod;
					t[p].mul = (t[p].mul * v) % mod;
					return; 
				}
				pushdown(p, l, r);
				int mid = (l + r) >> 1;
				if(ql <= mid)
					update_mul(ls, l, mid, ql, qr, v);
				if(qr > mid)
					update_mul(rs, mid + 1, r, ql, qr, v);
				pushup(p); return;
			}
			i64 query(int p, int l, int r, int ql, int qr) {
				if(l > r) return 0ll;
				if(!p) return 0ll;
				if(ql <= l && r <= qr) 
					return t[p].dat % mod;
				pushdown(p, l, r);
				int mid = (l + r) >> 1;
				i64 res = 0ll;
				if(ql <= mid) 
					res = (res + query(ls, l, mid, ql, qr)) % mod;
				if(qr > mid)
					res = (res + query(rs, mid + 1, r, ql, qr)) % mod;
				return res % mod;
			}
	}tr;
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		int root = 0;
		cin >> n;
		for(int i = 1; i <= n; ++i)
			cin >> a[i];
		tr.update_add(root, 1, 1e9, 1, a[1], 1ll);
		for(int i = 2; i <= n; ++i) {
			i64 sum = tr.query(root, 1, 1e9, 1, 1e9) % mod;
			tr.update_mul(root, 1, 1e9, 1, a[i], -1ll);
			tr.update_add(root, 1, 1e9, 1, a[i], sum);
			tr.update_mul(root, 1, 1e9, a[i] + 1, 1e9, 0ll);
		}
	
		cout << (tr.query(root, 1, 1e9, 1, 1e9) + mod) % mod << endl;
	
		return 0;
	}
	
	*/
	```


```cpp
Tag : 线段树优化DP/数据结构维护下标
```

### CF1485F Copy or Prefix Sum

> 给你一个数列 $b$，要求构造一个数列 $a$，满足以下条件：
> 
> + $a_i = b_i$ 或者 $b_i  = \sum\limits_{j = 1}^{i} a_j$。
> 
> 求可能的 $a$ 的数量模 $10^9 + 7$。
> 
> $1\le n \le 2\times 10^5, 1\le b_i \le 10^9$。

考虑一个朴素的 DP，直接把所有要素处理进去。

前缀和这个要素就不用在状态里体现了，直接在转移决策的时候体现。

所以设 $dp(i, j)$ 为，考虑前 $i$ 个数，当前选的 $\sum a_i = j$，总共的方案数。

然后针对两种情况写转移方程：

$$\begin{cases}dp(i, j) = dp(i - 1, j - b_i) & a_i = b_i \\ dp(i, b_i) = \sum\limits_{j = -\infty}^{\infty} dp(i - 1, j) - dp(i - 1, 0) & \texttt{otherwise.}\end{cases}$$

第一种比较显然，第二种从 $-\infty$ 求和到 $\infty$ 是因为，假设你 $a_i$ 随便选了一个数 $x$，你只需要让 $\sum\limits_{j = 1}^{i - 1} a_j= b_i - x$ 即可，所以不管 $a_i$ 取啥，直接前面对应的方案数就行。

因为 $a_i$ 可以随便选，所以自然是从负无穷一直求和到正无穷。

要减去 $dp(i - 1, 0)$ 是因为，当 $\sum\limits_{j = 1}^{i - 1} a_j = 0$ 时， 这两种方案都是等价的。

然后第一个就是给数组平移了一下。

后一个看起来很吓人，其实发现就是一个全局求和，然后发现每次决策完之后，会改变的只有 $dp(i, b_i)$。

所以我们记录三个变量，一个是偏移量 $\Delta$，一个是全局和 $\sum$，一个是上一个被改变的位置 $dp(i - 1, b_{i - 1})$ 平移之前的值。

然后转移就很方便了。

??? note "Code"
	```cpp
	
	#include <map>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	#include <ext/pb_ds/hash_policy.hpp>
	#include <ext/pb_ds/assoc_container.hpp>
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const int si = 3e5 + 10;
	const int mod = 1e9 + 7;
	
	int n;
	i64 a[si];
	i64 sum = 0, delta = 0;
	// __gnu_pbds::gp_hash_table<i64, i64> dp;
	map<i64, i64> dp;
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		int T; cin >> T;
		while(T--) {
			cin >> n;
			dp.clear();
			for(int i = 1; i <= n; ++i) 
				cin >> a[i];
			dp[0] = 1ll, sum = 1ll, delta = 0ll;
			for(int i = 1; i <= n; ++i) {
				int kot = (sum - dp[0ll - delta] + mod) % mod;
				sum = (sum + kot) % mod, delta += a[i];
				dp[a[i] - delta] = (dp[a[i] - delta] + kot) % mod;
			}
			cout << sum << "\n";
		}
	
		return 0;
	}
	```

```cpp
Tag : DP优化/水位线法
```

### CF1156F Card Bag

> 给你 $n$ 张牌，每张牌上有一个数字 $a_i, 1\le a_i \le n$。
> 
> 你每次会从牌堆里抽一张牌，抽完之后不放回。
> 
> 假设当前抽到 $x$，上一张抽出的是 $y$。
> 
> + 如果 $x = y$，win。
> + 如果 $x < y$，lose。
> + 如果 $x > y$，continue。
> 
> 问你最后获胜的概率是多少，对 $998244353$ 取模，$1\le n \le 5000$。

发现要处理的要素有点多：

+ 抽完不放回去
+ 每次可能有三种游戏状态：win,lose,continue。
+ 每张牌的数量

第一个直接记录当前抽了第几次就行了。

第二个也比较好做，lose就直接退出了，对于最后的答案不会有贡献，continue 需要设计状态表示，但是发现 win 只需要考虑最后两个牌相同就行了，所以也不用设计状态，最后统一计算贡献即可。

所以我们只需要处理 continue 的情况。

最麻烦的是最后一个，感觉上来说需要记录每张牌剩余的数量，但这样状态承受不了，也不好转移。

但其实不用，因为有一个性质（见下面的 Warning）。

所以可以设计出 DP ： 设 $dp(i, j)$ 表示考虑第 $i$ 次，抽到 $j$ 且继续下去的概率。

因为要保证一直继续，所以转移的时候不能从上一个取大于 $j$ 的状态转移过来，又要保证需要继续下去，所以也不能从上一个取 $j$ 的转移。

写出方程：

$$dp(i, j) = \sum\limits_{k = 1}^{j - 1} dp(i - 1, k) + \dfrac{cnt(j)}{n - i + 1}$$

可以前缀和优化到 $\text{O}(n^2)$。

考虑计算答案。

发现只要 $cnt(j) > 1$，那么状态 $dp(i, j)$ 对于答案的贡献就是 $\dfrac{cnt(j) - 1}{n - i}$。

因为只要 $cnt(j) > 1$，那你前后两个就可以取一样的，然后在 $dp(i, j)$ 的基础上乘一个再次取到 $j$ 的概率即可。

又因为所有胜利的情况是互斥（互不影响）的，所以最后的答案直接求个 $\sum$ 即可。

!!! Warning 

	解释一下为什么转移的时候取到 $j$ 的概率是 $\dfrac{cnt(j)}{n - i + 1}$，

	发现你需要继续下去，当且仅当你取出来的数按照时间戳排序之后是严格单调递增的。

	所以其实如果要一直继续，每种牌是只能取一张的，也就是说，如果你当前考虑的是一直继续，那么抽到 $j$ 的时候一定是第一次抽到。

	所以概率可以这么算。

	最后计算答案的时候同理。

??? note "Code"
	```cpp
	// author : black_trees
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const int si = 5e3 + 10;
	constexpr int mod = 998244353;
	
	int n;
	int a[si], inv[si];
	int cnt[si], sum[si];
	int dp[si][si], ans = 0;
	
	i64 qpow(i64 __a, i64 __b){
	    i64 __ans = 1 % mod;
	    for(; __b; __b >>= 1){
	        if(__b & 1) __ans = __ans * __a % mod;
	        __a = __a * __a % mod;
	    }
	    return __ans % mod;
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n;
		for(int i = 1; i <= n; ++i)
			cin >> a[i], cnt[a[i]] ++;
	
		inv[0] = 0 % mod, inv[1] = 1 % mod;
		for(int i = 2; i <= n; ++i)
			inv[i] = qpow(i, mod - 2) % mod;
		
		sum[0] = 0;
		for(int i = 1; i <= n; ++i) 
			dp[1][i] = 1ll * cnt[i] % mod * 1ll * inv[n] % mod;	
	
		for(int i = 2; i <= n; ++i) {
			for(int j = 1; j <= n; ++j) 
				sum[j] = (sum[j - 1] + dp[i - 1][j]) % mod;
			for(int j = 1; j <= n; ++j) {
				dp[i][j] = 1ll * sum[j - 1] * 1ll * cnt[j] % mod * 1ll * inv[n - i + 1] % mod;
			}
		}
		for(int i = 1; i <= n; ++i) {
			for(int j = 1; j <= n; ++j) {
				if(cnt[j] >= 2) 
					ans = (ans + (1ll * dp[i][j] * 1ll * (cnt[j] - 1) % mod * 1ll * inv[n - i]) % mod) % mod;
			}
		}
	
		cout << ans % mod << endl;
	
		return 0;
	}
	```

```cpp
Tag : 概率DP
```

### CF1077F2 Pictures with Kittens(Hard version)

> 给你一个数列 $a$，你需要选择 $m$ 个元素，
> 
> 使得连续的 $k$ 个元素都至少有一个被选中。
>
> 需要你最大化选出来的所有数的和。
> 
> $1\le m, k \le n \le 5000, 1\le a_i \le 10^9$。

考虑一个朴素的 DP，因为此处有要求选多少个的限制，所以可以有：

$dp(i, j)$ 表示，考虑前 $i$ 个元素，当前选择了 $j$ 个元素的最大价值。

然后又有一个连续 $k$ 个必须要选的限制，考虑选 $i$ 进行决策即可。

所以有方程：

$$dp(i, j) = \max\limits_{\max(0, i - k) \le l < i}\{dp(l, j - 1) + a_i\}$$

考虑怎么优化，感觉这个式子很像单调队列优化的 1D1D，所以先把 $a_i$ 提出来。

所以试着固定一下 $i$，发现每当 $j + 1$，决策集合变化是很奇怪的。

因为后面的状态是 $dp(l, j - 1)$，$j$ 一变他也要变。

所以换种思考方式，我们考虑直接让 $i$ 单调变化，看决策集合怎么变化（因为决策集合上下界都是关于 $i$ 的式子）。

那么发现这里实际上就是对一个特定区间取最值，且这个区间会单调滑动。

所以可以考虑对于每一个 $j$ 维护一个单调队列，维护 $kot \in [i - k, i)$ 的 $\max\{dp(kot, j)\}$

然后每当 $i$ 变化的时候维护一下即可。

每次询问只需要在 $j - 1$ 的单调队列里面找最值转移就好了。

??? note "Code"
	```cpp
	#include <cmath>
	#include <deque>
	#include <cstdio>
	#include <cstring>
	#include <utility>
	#include <iostream>
	#include <algorithm>
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const int si = 5e3 + 10;
	const i64 inf = 0x3f3f3f3f3f3f3f3fll;
	
	int n, m, k;
	i64 a[si], dp[si][si];
	deque<pair<i64, int>> Q[si];
	
	int main() {	
		auto query = [&](int id, int pos) -> i64 {
			auto &q = Q[id];
			while(!q.empty() && q.front().second <= pos - k)
				q.pop_front();
			return q.empty() ? -inf : q.front().first;
		};
	
		auto update = [&](int id, int pos, i64 num) -> void{
			auto &q = Q[id];
			while(!q.empty() && q.back().first <= num)
				q.pop_back();
			q.push_back(make_pair(num, pos));
		};
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n >> k >> m;
	
		if(ceil((1.0 * (n - k + 1)) / (1.0 * k)) > m)
			cout << "-1\n", exit(0);
	
		for(int i = 1; i <= n; ++i)
			cin >> a[i];
		for(int i = 1; i <= k; ++i) 
			dp[i][1] = a[i];
	
		for(int i = 1; i <= n; ++i) {
			for(int j = min(i, m); j >= 2; --j) {
				dp[i][j] = query(j - 1, i - 1) + a[i];
				update(j, i, dp[i][j]);			
			}
			if(i <= k){
				dp[i][1] = a[i];
				update(1, i, dp[i][1]);
			}
		}
	
		i64 res = 0;
	
		for(int i = n - k + 1; i <= n; ++i)
			res = max(res, dp[i][m]);
	
		cout << res << "\n";
	
		return 0;
	}
	```

### CF1407D Discrete Centrifugal Jumps

> 有 $n$ 栋楼，你每次可以从 $i$ 跳到 $j$，当且仅当一下三个条件有至少一个被满足。
> 
> （设每栋楼的高度为 $h_i$）
> 
> + $i + 1 = j$
> + $min(h_i,h_j) > max(h_{i + 1}, \dots, h_{j - 1})$
> + $max(h_i, h_j) < min(h_{i + 1}, \dots, h_{j - 1})$
> 
> 请问从 $1$ 跳到 $n$ 的最小步数是多少？
> 
> $2 \le n \le 3\times 10^5$。
> 
> $1\le h_i \le 10^9$。

考虑一个非常朴素的 DP，

设 $dp(i)$ 表示从 $1$ 到 $i$ 的最小步数。

$\text{O}(n^3)$ 的 DP 比较容易，第一种情况直接从 $dp(i - 1)$ 转移。

后两种情况枚举即可。

然后考虑优化后两种情况，

比如一个局部长成这样：

$$4\ 3\ 2\ 1\ 5$$

```
        +
+       +
+ +     +
+ + +   +
+ + + + +
---------
1 2 3 4 5 (position)
```

然后你要考虑 $dp(5)$ 的转移。

发现 **位置** $1, 2, 3$ 都可以转移到 $5$，因为满足了第二种情况。

然后 $4$ 也可以转移到 $5$，因为他们相邻。

注意到这实际上是维护了一个栈顶为最小值的单调栈（从顶向下非严格递增），

然后不断把弹出栈的位置作为上一个位置进行转移的过程。

第三种情况类似，所以我们维护两个单调栈即可。

不过需要注意这种情况：

$$3\ 2\ 3\ 2\ 3$$

此时位置 $1$ 无法转移到位置 $5$，因为不满足严格小于的限制。

但是位置 $3$ 则可以，因为他是和位置 $5$ 的 $h$ 值相等且可以转移到 $5$ 的所有位置中距离 $5$ 最近的一个。

那么就需要在正常弹栈之后，再把和当前位置的值 $h_i$ 相同的元素全部弹掉，然后再把 $h_i$ 压进去。

但是弹掉相同元素之前不要忘记和第一个相同的（距离最近的一个）元素做一次转移。

??? note "Code"
	```cpp
	// author : black_trees
	
	#include <stack>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const int si = 3e5 + 10;
	
	int n;
	int a[si];
	int dp[si];
	
	std::stack<int> upper, lower;
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		memset(dp, 0x3f, sizeof dp);
	
		cin >> n;
		for(int i = 1; i <= n; ++i) 
			cin >> a[i];
	
		upper.push(1), lower.push(1), dp[1] = 0;
		for(int i = 2; i <= n; ++i) {
			dp[i] = min(dp[i], dp[i - 1] + 1);
			
			while(!upper.empty() && a[i] < a[upper.top()]) 
				dp[i] = min(dp[i], dp[upper.top()] + 1), upper.pop();
			if(!upper.empty()) // 这里不管当前栈顶是不是一样的都要取，因为不去会漏掉一些转移。
							   // 有无例子？
				dp[i] = min(dp[i], dp[upper.top()] + 1);
			while(!upper.empty() && a[i] == a[upper.top()])
				upper.pop();
			// 相等的弹掉，因为只能取第一个相等的转移。
			upper.push(i);
	
			while(!lower.empty() && a[i] > a[lower.top()]) 
				dp[i] = min(dp[i], dp[lower.top()] + 1), lower.pop();
			if(!lower.empty())
				dp[i] = min(dp[i], dp[lower.top()] + 1);
			while(!lower.empty() && a[i] == a[lower.top()])
				lower.pop();
			lower.push(i);
		}
	
		cout << dp[n] << endl;
	
		return 0;
	}
	```

### CF980D Perfect Groups

> 你有一个序列 $a$，你需要分别计算贡献为 $1\sim n$ 的 $a$ 的子串的数量。
> 
> 对于任意一个子串，将这个子串里的所有元素分成 $k$ 组，保证每组里的数两两相乘之后是完全平方数。
> 
> 这个子串的贡献就是最小的 $k$。
> 
> $1\le n\le 5000, |a_i| \le 10^9$。

第一种比较暴力的方式就是，直接用并查集把所有乘起来是完全平方数的合并到一个集合里。

因为如果 $a\times b$ 是完全平方，且 $b\times c$ 是完全平方，那么 $c\times a$ 也是完全平方。

然后 $\text{O}(n^2)$ 枚举所有子串计算答案即可。

或者用一个巧妙一点的办法（可以过天元公学的某场提高邀请赛的 B 的做法）。

直接把所有平方因子筛掉，之后相等的就应该扔到一个集合里面。

注意这两种做法都要特判 $0$ 的影响。

??? note "$\text{O}(n^2)$ Code"
	```cpp
	// author : black_trees
	
	#include <map>
	#include <cmath>
	#include <bitset>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#pragma GCC target("avx,sse2,sse3,sse4,mmx")
	#pragma GCC optimize("Ofast", "inline", "-ffast-math")	
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	using ldb = long double;
	
	const int si = 5e3 + 10;
	
	int n;
	i64 a[si];
	int ans[si];
	
	int pa[si];
	int root(int x) {
		if(pa[x] != x)
			return pa[x] = root(pa[x]);
		return pa[x];
	}
	void merge(int x, int y) {	
		int rx = root(x), ry = root(y);
		if(rx == ry) return;
		pa[rx] = ry;
	}
	bool issqr(i64 m) {
	    i64 t = sqrt((ldb)m); 
	    return ((t * t) == m);
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n;
		for(int i = 1; i <= n; ++i) 
			cin >> a[i], pa[i] = i;
	
		for(int i = 1; i <= n; ++i) {
			for(int j = i + 1; j <= n; ++j) {
				if(!a[i] || !a[j])
					continue;
				if(issqr(1ll * a[i] * a[j]))
					merge(i, j);
			}
		}
	
		// for(int i = 1; i <= n; ++i) 
			// meow(pa[i]) << endl;
	
		// memset(ans, 0, sizeof ans);
		// std::bitset<si> Set;
		// std::map<int, bool> Set;
		bool Set[si];
		for(int i = 1; i <= n; ++i) {
			for(int j = 1; j <= n; ++j) 
				Set[j] = false;
			
			for(int j = i, cnt = 0; j <= n; ++j) {
				if(!a[j]) ans[max(1, cnt)] ++;
				else ans[cnt = Set[root(j)] ? cnt : ++cnt] ++, Set[root(j)] = true;
			} 
		}
	
		for(int i = 1; i <= n; ++i) 
			cout << ans[i] << " ";
		cout << "\n";
	
		return 0;
	}
	```

```cpp
Tag : 并查集/暴力/数论/筛法/平方因子/唯一分解定理
```