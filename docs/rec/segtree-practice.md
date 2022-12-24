
## Vjudge 线段树小专题

### A - Circular RMQ

> 在环上做区间加区间最值。
>
> $1\le n \le 10^6$。

就拆一下就行了，没啥好说的

??? note "Code"
	```cpp
	// author : black_trees

	#include <cmath>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	using i64 = long long;

	const int si = 2e5 + 10;
	const i64 inf = 0x3f3f3f3f3f3f3f3fll;

	int n, m, a[si];
	class SegTree {
	private:
		struct Node {
			int l, r;
			i64 val, tag;
		} t[si << 2];
		void pushup(int p) {
			t[p].val = min(t[p << 1].val, t[p << 1 | 1].val);
			return;
		}
		void pushdown(int p) {
			if(!t[p].tag) return;
			t[p << 1].val += t[p].tag, t[p << 1 | 1].val += t[p].tag;
			t[p << 1].tag += t[p].tag, t[p << 1 | 1].tag += t[p].tag;
			t[p].tag = 0;
		}
	public: 
		void Build(int p, int l, int r) {
			t[p].l = l, t[p].r = r, t[p].tag = 0;
			if(l == r) { t[p].val = 1ll * a[l]; return; }
			int mid = (l + r) >> 1;
			Build(p << 1, l, mid), Build(p << 1 | 1, mid + 1, r), pushup(p);
		}
		void Inc(int p, int l, int r, i64 v) {
			int nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) { t[p].tag += v, t[p].val += v; return; }
			pushdown(p); int mid = (nl + nr) >> 1;
			if(l <= mid) Inc(p << 1, l, r, v);
			if(r > mid) Inc(p << 1 | 1, l, r, v);
			pushup(p);
		}
		i64 Rmq(int p, int l, int r) {
			i64 ret = inf;
			int nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) return t[p].val;
			pushdown(p); int mid = (nl + nr) >> 1;
			if(l <= mid) ret = min(ret, Rmq(p << 1, l, r));
			if(r > mid) ret = min(ret, Rmq(p << 1 | 1, l, r));
			return ret;
		}
	} tr;

	char s[400005];

	int main() {

		// cin.tie(0) -> sync_with_stdio(false);
		// cin.exceptions(cin.failbit | cin.badbit);

		cin >> n;	
		for(int i = 1; i <= n; ++i) 
			cin >> a[i];
		tr.Build(1, 1, n), cin >> m, getchar();
		for(int i = 1; i <= m; ++i) {
			gets(s);
			int l, r; i64 k;
			if(sscanf(s, "%d%d%I64d", &l, &r, &k) == 2) {
				l++, r++;
				if(l <= r) cout << tr.Rmq(1, l, r) << endl;
				else cout << min(tr.Rmq(1, 1, r), tr.Rmq(1, l, n)) << endl;
			}
			else {
				l++, r++;
				if(l <= r) tr.Inc(1, l, r, k);
				else tr.Inc(1, 1, r, k), tr.Inc(1, l, n, k);
			}
		}

		return 0;
	}
	```

### B - Buy Tickets

> 有 $n$ 个人和一个初始为空的队列，每个人都被分配了一个数值，并且第 $i$ 个人会插队到第 $pos_i$ 个人后面（$pos_i = 0$ 意味着站到队头）。
>
> 求经过 $n$ 次插队后整个队列的情况，$1\le n \le 2\times 10^5$。

好久以前就见过这题了，只是当时没有做出来。

正着做很难搞，怎么做都是 $O(n^2)$ 的，考虑类似 ARC080E Young Maids 那题的思路，倒过来观察合法解的形状。

发现如果一个人是最后插队的，那么他的位置一定是固定的，类似地往前走就能确定每一个人的位置。

具体来说，我们维护 $[1, n]$ 的区间“空位”个数和，然后倒序插入，过程类似这个（图源<https://www.cnblogs.com/zhengguiping--9876/p/4717024.html>）：

[![zruIM9.png](https://s1.ax1x.com/2022/12/03/zruIM9.png)](https://imgse.com/i/zruIM9)

于是线段树二分一下就能解决了。

??? note "Code"
	```cpp
	// author : black_trees

	#include <cmath>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl "\n"

	using namespace std;
	// using i64 = long long;

	const int si = 2e5 + 10;

	int n, m, ord[si];

	class SegTree {
	private:
		struct Node {
			int l, r;
			int sum;
		} t[si << 2];
		inline void pushup(int p) { t[p].sum = t[p << 1].sum + t[p << 1 | 1].sum; }
	public:
		void build(int p, int l, int r) {
			t[p].l = l, t[p].r = r;
			if(l == r) return (void)(t[p].sum = 1);
			int mid = (l + r) >> 1;
			build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r), pushup(p);
		}
		void insert(int p, int x, int val) {
			if(t[p].l == t[p].r) { t[p].sum = 0, ord[t[p].l] = val; return; }
			if(t[p << 1].sum >= x) insert(p << 1, x, val);
			else insert(p << 1 | 1, x - t[p << 1].sum, val);
			pushup(p);
		}
	} tr;
	struct Op { int pos, val; } a[si];

	int main() {

		// cin.tie(0) -> sync_with_stdio(false);
		// cin.exceptions(cin.failbit | cin.badbit);

		while(~scanf("%d", &n)) {
			tr.build(1, 1, n); 
			for(int i = 1; i <= n; ++i) scanf("%d%d", &a[i].pos, &a[i].val);  
			for(int i = n; i >= 1; --i) tr.insert(1, a[i].pos + 1, a[i].val);
			for(int i = 1; i <= n; ++i) printf("%d ", ord[i]); printf(endl);
		}
		return 0;
	}

	```

POJ 神笔玩意儿卡 cin，记得用 scanf。

### C - Who Gets the Most Candies?

??? note "Statement"
	N children are sitting in a circle to play a game.

	The children are numbered from 1 to N in clockwise order. Each of them has a card with a non-zero integer on it in his/her hand. The game starts from the K-th child, who tells all the others the integer on his card and jumps out of the circle. The integer on his card tells the next child to jump out. Let A denote the integer. If A is positive, the next child will be the A-th child to the left. If A is negative, the next child will be the (−A)-th child to the right.

	The game lasts until all children have jumped out of the circle. During the game, the p-th child jumping out will get F(p) candies where F(p) is the number of positive integers that perfectly divide p. Who gets the most candies?

	Input
	There are several test cases in the input. Each test case starts with two integers N (0 < N ≤ 500,000) and K (1 ≤ K ≤ N) on the first line. The next N lines contains the names of the children (consisting of at most 10 letters) and the integers (non-zero with magnitudes within 108) on their cards in increasing order of the children’s numbers, a name and an integer separated by a single space in a line with no leading or trailing spaces.
	Output
	Output one line for each test case containing the name of the luckiest child and the number of candies he/she gets. If ties occur, always choose the child who jumps out of the circle first.

就是一个约瑟夫环的问题，想法类似 B 题，就是维护空位之类的东西然后线段树二分。

??? note "Code"

	```cpp
	// author : black_trees

	#include <cmath>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	// using i64 = long long;

	const int si = 5e5 + 10;

	int n, k, id;
	int F[si];
	void init() {
		memset(F, 0, sizeof F);
		for(int i = 1; i <= n; ++i) {
			F[i]++;
			for(int j = i * 2; j <= n; j += i) {
				F[j]++;
			}
		}
		id = 1; int m = F[1];
		for(int i = 1; i <= n; ++i) {
			if(m < F[i]) id = i, m = F[i];
		}
	}

	int a[si];
	char c[si][15];

	class SegTree {
	private:
		struct Node {
			int l, r;
			int sum;
		} t[si << 4];
		inline void pushup(int p) { t[p].sum = (t[p << 1].sum + t[p << 1 | 1].sum); }
	public:
		void build(int p, int l, int r) {
			t[p].l = l, t[p].r = r;
			if(l == r) return (void)(t[p].sum = 1);
			int mid = (l + r) >> 1;
			build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
			pushup(p);
		}
		int query(int p, int l, int r) {
			int ret = 0, nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) return t[p].sum;
			int mid = (nl + nr) >> 1;
			if(l <= mid) ret += query(p << 1, l, r);
			if(r > mid) ret += query(p << 1 | 1, l, r);
			return ret;
		}
		int modify(int p, int x) {
			int l = t[p].l, r = t[p].r;
			if(l == r) return t[p].sum = 0, l;
			int mid = (l + r) >> 1, ret;  
			if(t[p << 1].sum >= x) ret = modify(p << 1, x);
			else ret = modify(p << 1 | 1, x - t[p << 1].sum);
			pushup(p); return ret;
		}
	} tr;

	int main() {

		// cin.tie(0) -> sync_with_stdio(false);
		// cin.exceptions(cin.failbit | cin.badbit);

		while(~scanf("%d%d", &n, &k)) {
			for(int i = 1; i <= n; ++i) scanf("%s%d", &c[i], &a[i]);	
			init(), tr.build(1, 1, n);
			a[0] = 0; int pos = 0, mod = n, tmp = id;
			while(tmp --) {
				if(a[pos] > 0) k = ((k - 1 + a[pos] - 1) % mod + mod) % mod + 1;
				else k = ((k - 1 + a[pos]) % mod + mod) % mod + 1;
				pos = tr.modify(1, k), mod = tr.query(1, 1, n);
			}
			printf("%s %d\n", c[pos], F[id]);
		}	

		return 0;
	}

	```

### D - Fast Matrix Operations

> 有一个 $r$ 行 $c$ 列的全 $0$ 矩阵，有以下三种操作。
> 
> + `1 X1 Y1 X2 Y2 v` 将子矩阵 $(X1,Y1,X2,Y2)$ 的元素加 $v,(v > 0)$。
> 
> + `2 X1 Y1 X2 Y2 v` 将子矩阵 $(X1,Y1,X2,Y2)$ 的所有元素变为 $v$。
>
> + `3 X1 Y1 X2 Y2` 查询子矩阵 $(X1,Y1,X2,Y2)$ 的和，最大值，最小值。
> 
> 输入保证和不超过 $10^9$，矩阵不超过 $20$ 行，矩阵元素个数不超过 $10^6$。
> 
> 翻译来自 _Luogu-@Himself65_。

发现矩阵不超过 $20$ 行，所以可以直接暴力对每一行维护。

然后我们就只需要区间加区间修改区间求和区间最值。

注意到区间加和区间修改都是需要 lazytag 的，我们思考一下 lazytag 的顺序。

这里为了方便就懒得用群论的语言解释了。

可以发现，如果一个节点被先打上一个 add 标记，然后打上一个 set 标记，add 标记会被直接**覆盖**。

如果先打上一个 set 标记，后打上一个 add 标记，是没有任何影响的。

所以在 pushdown 的时候我们需要先处理 set，在区间修改和下放 set 标记的时候要注意清空 add 标记。

就这样，只是比较难写，然后修改的 $v$ 好像没有限制，所以保险起见我们用一个值域外的数表示空标记就行了。

??? note "Code"
	```cpp
	// author : black_trees

	#include <cmath>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	using i64 = long long;

	const int inf = 1e9 + 7;
	const int si = 1e6 + 10;

	int n, m, q;
	class SegTree {
	private: 
		struct Node {
			int l, r;
			int mi, mx, sm;
			int set, add;
			int len() { return r - l + 1; }
		} t[si << 2];
		inline void pushup(int p) {
			t[p].sm = t[p << 1].sm + t[p << 1 | 1].sm;
			t[p].mx = max(t[p << 1].mx, t[p << 1 | 1].mx);
			t[p].mi = min(t[p << 1].mi, t[p << 1 | 1].mi);
		}
		inline void pushdown(int p) {
			if(t[p].set != inf) { 	
				t[p << 1].sm = t[p << 1].len() * t[p].set;
				t[p << 1 | 1].sm = t[p << 1 | 1].len() * t[p].set;	
				t[p << 1].mi = t[p << 1].mx = t[p].set;
				t[p << 1 | 1].mi = t[p << 1 | 1].mx = t[p].set;
				t[p << 1].set = t[p].set, t[p << 1 | 1].set = t[p].set, t[p].set = inf;
				t[p << 1].add = 0, t[p << 1 | 1].add = 0; // Attention.
			}
			if(t[p].add != 0) {
				t[p << 1].sm += t[p << 1].len() * t[p].add;
				t[p << 1].mi += t[p].add, t[p << 1].mx += t[p].add;
				t[p << 1 | 1].sm += t[p << 1 | 1].len() * t[p].add;
				t[p << 1 | 1].mi += t[p].add, t[p << 1 | 1].mx += t[p].add;
				t[p << 1].add += t[p].add, t[p << 1 | 1].add += t[p].add, t[p].add = 0;
			}
		}
	public: 
		void build(int p, int l, int r) {
			t[p].l = l, t[p].r = r;
			if(l == r) {
				t[p].sm = 0, t[p].mx = 0, t[p].mi = 0;
				t[p].add = 0, t[p].set = inf; return; 
			}
			int mid = (l + r) >> 1;
			build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r), pushup(p);
		}
		void Set(int p, int l, int r, int v) {
			int nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) {
				t[p].sm = t[p].len() * v;
				t[p].mx = t[p].mi = v;
				t[p].add = 0, t[p].set = v; // Attention
				return;
			}
			pushdown(p);
			int mid = (nl + nr) >> 1;
			if(l <= mid) Set(p << 1, l, r, v);
			if(r > mid) Set(p << 1 | 1, l, r, v);
			pushup(p);
		}
		void Add(int p, int l, int r, int v) {
			int nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) {
				t[p].sm += t[p].len() * v;
				t[p].mx += v, t[p].mi += v;
				t[p].add += v; return;
			}
			pushdown(p);
			int mid = (nl + nr) >> 1;
			if(l <= mid) Add(p << 1, l, r, v);
			if(r > mid) Add(p << 1 | 1, l, r, v);
			pushup(p);
		}
		int Quesm(int p, int l, int r) {
			int ret = 0, nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) return t[p].sm;
			pushdown(p);
			int mid = (nl + nr) >> 1;
			if(l <= mid) ret += Quesm(p << 1, l, r);
			if(r > mid) ret += Quesm(p << 1 | 1, l, r);
			return ret;
		}
		int Quemi(int p, int l, int r) {
			int ret = inf, nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) return t[p].mi;
			pushdown(p);
			int mid = (nl + nr) >> 1;
			if(l <= mid) ret = min(ret, Quemi(p << 1, l, r));
			if(r > mid) ret = min(ret, Quemi(p << 1 | 1, l, r));
			return ret;
		}
		int Quemx(int p, int l, int r) {
			int ret = -inf, nl = t[p].l, nr = t[p].r;
			if(l <= nl && nr <= r) return t[p].mx;
			pushdown(p);
			int mid = (nl + nr) >> 1;
			if(l <= mid) ret = max(ret, Quemx(p << 1, l, r));
			if(r > mid) ret = max(ret, Quemx(p << 1 | 1, l, r));
			return ret;
		}
	};

	int main() {

		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);

		// while(cin >> n >> m >> q) {
		cin >> n >> m >> q;
			SegTree* tr = new SegTree[n+1];
			for(int i = 1; i <= n; ++i)
				tr[i].build(1, 1, m);
			for(int i = 1; i <= q; ++i) {
				int opt; cin >> opt;
				if(opt == 1) {
					int x, xx, y, yy, v;
					cin >> x >> y >> xx >> yy >> v;
					for(int j = x; j <= xx; ++j) 
						tr[j].Add(1, y, yy, v);
				}	
				if(opt == 2) {
					int x, xx, y, yy, v;
					cin >> x >> y >> xx >> yy >> v;
					for(int j = x; j <= xx; ++j) 
						tr[j].Set(1, y, yy, v);
				}
				if(opt == 3) {
					int x, xx, y, yy;
					cin >> x >> y >> xx >> yy;
					int sum = 0, mxv = -inf, miv = inf;
					for(int j = x; j <= xx; ++j) {
						sum += tr[j].Quesm(1, y, yy);
						mxv = max(mxv, tr[j].Quemx(1, y, yy));
						miv = min(miv, tr[j].Quemi(1, y, yy));
					}
					cout << sum << " " << miv << " " << mxv << endl;
				}
			}
		// }
		return 0;
	}
	```

还有一个点是，直接开 $21$ 个线段树会炸，所以我们需要用 `new/delete` 动态创建数组。

这个题好像说是有多测，但是数据可能是用脚造的，所以我忘记取消注释了也都没有事情，也有可能是我读错了。

### E - Parade

题意和题解看这里的[复盘报告](../sol/pro/cf35e.md)。

### F - Help with Intervals

题意有点复杂，我直接粘贴原题面了：

??? note "Statement"
	```
	Description

	LogLoader, Inc. is a company specialized in providing products for analyzing logs. While Ikki is working on graduation design, he is also engaged in an internship at LogLoader. Among his tasks, one is to write a module for manipulating time intervals, which have confused him a lot. Now he badly needs your help.

	In discrete mathematics, you have studied several basic set operations, namely union, intersection, relative complementation and symmetric difference, which naturally apply to the specialization of sets as intervals.. For your quick reference they are summarized in the table below:

	Operation	Notation	
	Definition

	Union	A ∪ B	{x : x ∈ A or x ∈ B}
	Intersection	A ∩ B	{x : x ∈ A and x ∈ B}
	Relative complementation	A − B	{x : x ∈ A but x ∉ B}
	Symmetric difference	A ⊕ B	(A − B) ∪ (B − A)
	Ikki has abstracted the interval operations emerging from his job as a tiny programming language. He wants you to implement an interpreter for him. The language maintains a set S, which starts out empty and is modified as specified by the following commands:

	Command	Semantics
	U T	S ← S ∪ T
	I T	S ← S ∩ T
	D T	S ← S − T
	C T	S ← T − S
	S T	S ← S ⊕ T
	Input

	The input contains exactly one test case, which consists of between 0 and 65,535 (inclusive) commands of the language. Each command occupies a single line and appears like

	X T

	where X is one of ‘U’, ‘I’, ‘D’, ‘C’ and ‘S’ and T is an interval in one of the forms (a,b), (a,b], [a,b) and [a,b] (a, b ∈ Z, 0 ≤ a ≤ b ≤ 65,535), which take their usual meanings. The commands are executed in the order they appear in the input.

	End of file (EOF) indicates the end of input.

	Output

	Output the set S as it is after the last command is executed as the union of a minimal collection of disjoint intervals. The intervals should be printed on one line separated by single spaces and appear in increasing order of their endpoints. If S is empty, just print “empty set” and nothing else.
	```

首先考虑一个问题是，怎么处理开区间和闭区间。

这个比较简单，我们把每个点看作两个点，分别表示这个点作为开/闭区间端点的情况。

然后这样就很方便修改。

比如 $(1, 3]$ 就拆成 $(1), [2], (2), [3]$ 就行了。

转化回来的话，你发现开区间一定是奇数，闭区间一定是偶数，然后就行了。

于是现在要做的就是看这个操作怎么转化成比较方便操作的语言。

其实就是维护一堆 0/1，然后要支持一定的区间翻转和区间异或操作，只不过维护的时候可以直接只维护两个 tag，最后再暴力处理，毕竟只有 0/1 嘛。

- `U` : 把区间 $[l,r]$ 覆盖成 $1$
- `I` : 把 $[-\infty,l), (r,\infty]$ 覆盖成 $0$
- `D` : 把区间 $[l,r]$ 覆盖成 $0$
- `C` : 把 $[-\infty,l), (r,∞]$ 覆盖成 $0$, 且 $[l,r]$ 区间整体异或 $1$
- `S` : $[l,r]$ 区间整体异或 $1$

Cover 操作是 Trivial 的，Xor 只需要分割区间然后在区间上面打一个标记。

注意 Xor 不能覆盖 Cover 只能把 Xor 打到 Cover 身上，Cover 可以覆盖 Xor，就是一个优先级的问题。

嗯嗯，然后做完了。

??? note "Code"
	```cpp
	// 这个没调完
	// author : black_trees

	#include <cmath>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	// using i64 = long long;

	const int si = 2e5 + 10;

	bool vis[si << 2];
	class SegTree {
		private:
			struct Node {
				int l, r;
				int set, Xor;
			} t[si << 2];
			inline void pushdown(int p) {
				if(t[p].set != -1) {
					t[p << 1].set = t[p << 1 | 1].set = t[p].set;
					t[p << 1].Xor = t[p << 1 | 1].Xor = 0;
					t[p].set = -1;
				}
				if(t[p].Xor) {
					if(t[p << 1].set != -1) t[p << 1].set ^= 1;
					else t[p << 1].Xor ^= 1;
					if(t[p << 1 | 1].set != -1) t[p << 1 | 1].set ^= 1;
					else t[p << 1 | 1].Xor ^= 1;
					t[p].Xor = 0;
				}
			}
		public:
			void init() { t[1].set = t[1].Xor = 0; } 
			void build(int p, int l, int r) {
				t[p].l = l, t[p].r = r, t[p].set = -1, t[p].Xor = 0;
				if(l == r) return;
				int mid = (l + r) >> 1;
				build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
			}
			void change(int p, int l, int r, char op) {
				int nl = t[p].l, nr = t[p].r;
				if(l <= nl && nr <= r) {
					if(op == 'U') t[p].set = 1, t[p].Xor = 0;
					if(op == 'D') t[p].set = 0, t[p].Xor = 0;
					if(op == 'C' || op == 'S') {
						if(t[p].set != -1) t[p].set ^= 1;
						else t[p].Xor ^= 1;
					}
					return;
				}
				pushdown(p);
				int mid = (nl + nr) >> 1;
				if(l <= mid) change(p << 1, l, r, op);
				else t[p << 1].Xor = t[p << 1].set = 0;
				if(r > mid) change(p << 1 | 1, l, r, op);
				else t[p << 1 | 1].Xor = t[p << 1 | 1].set = 0;
			}
			void query(int p, int l, int r) {
				if(t[p].set == 1) {
					for(int i = l; i <= r; ++i) 
						vis[i] = true;
					return;
				}
				if(t[p].set == 0 || l == r) return;
				pushdown(p), query(p << 1, l, r), query(p << 1 | 1, l, r);
			}
	} tr;

	int main() {

		// cin.tie(0) -> sync_with_stdio(false);
		// cin.exceptions(cin.failbit | cin.badbit);
	#ifndef ONLINE_JUDGE
		freopen("input_f.txt", "r", stdin);
		freopen("output_f.txt", "w", stdout);
	#endif
		char suzui;
		tr.build(1, 1, si), tr.init();	
		while(scanf("%c", &suzui) != EOF) {
			char cl, cr; int l, r; 
			scanf(" %c%d,%d%c\n", &cl, &l, &r, &cr);
			l *= 2, r *= 2;
			if(cl == '(') l++;
			if(cr == ')') r--;
			tr.change(1, l, r, suzui);
		}
		tr.query(1, 1, si);
		int f = 0, s = -1, e;
		for(int i = 1; i <= si; ++i) {
			if(vis[i]) {
				if(s == -1) s = i;
				e = i;
			}
			else if(s != -1) {
				if(f) putchar(' ');
				f = 1; 
				char cl = (s & 1) ? '(' : '[', cr = (e & 1) ? ')' : ']';
				printf("%c%d,%c%d\n", cl, s / 2, (e + 1) / 2, cr), s = -1;
			}
		}
		if(!f) puts("empty set");

		return 0;
	}

	```
