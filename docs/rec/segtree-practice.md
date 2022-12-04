
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
