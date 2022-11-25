

这里算是思考一下李超tree的过程，以后写板子之前就这么干。

## 概述

李超树是一种变种线段树，可以动态维护一堆一次函数（可以有定义域的限制）。

然后可以询问这些一次函数在某个 $x$ 坐标下的最大 $y$ 值。

其大概思想是维护每个区间 $[l,r]$ 中点位置的可以产生最大 $y$ 的线段（的编号）。（这个东西本质上是一个 Lazytag）

注意到这个东西难以合并，Pushup 和 Pushdown 比较 hard，且我们没有初始状态，是一个一个插入的，所以我们用的是标记永久化的思想，维护且只维护 Lazytag。

标记永久化的原理简单来说就是修改时一路更改被影响到的点，询问时则一路累加路上的标记，从而省去下传标记的操作。

每次插入一个线段的时候，把维护被它定义域完整包含的区间的节点的信息全部更新（$O(\log n)$ 个），递归分割找到这 $O(\log n)$ 个区间的复杂度是 $O(\log n)$ 的，类似普通线段树区间查询。

然后，更新一个节点的信息的时候对于当前节点，分类讨论一下，处理新加入的线段对于当前区间的信息的影响，并且递归到子树处理一直到边界，这个复杂度是 $O(\log n)$ 的，因为当前线段和区间已经存了的线段的交点最多一个，只会递归一边。

具体分类就这么搞：

??? note "具体分类讨论方式（by dwt）"

	假设现在我们需要插入一条线段 $f$，在这条线段完整覆盖的线段树节点代表的区间中，某些区间的最优线段可能发生改变。

	考虑某个被新线段 $f$ 完整覆盖的区间，若该区间无最优线段，则该线段可以直接成为最优线段。

	否则，设该区间的中点为 $m$，我们拿新线段 $f$ 在中点处的值与原最优线段 $g$ 在中点处的值作比较。

	如果新线段 $f$ 更优，则将 $f$ 和 $g$ 交换。那么现在考虑在中点处 $f$ 不如 $g$ 优的情况：

	1. 若在左端点处 $f$ 更优，那么 $f$ 和 $g$ 必然在左半区间中产生了交点，递归到左儿子中进行插入；
	2. 若在右端点处 $f$ 更优，那么 $f$ 和 $g$ 必然在右半区间中产生了交点，递归到右儿子中进行插入。
	3. 若在左右端点处 $g$ 都更优，那么 $f$ 不可能成为答案，不需要继续下传。

注意有一个点是，如果插入了一条平行于 y 轴的线段 $(x, y_0) \to (x,y_1), (y_0 \le y_1)$，我们需要把它当成一个点插入进来，因为我们维护的信息是 Max，所以取 $y_1$ 这个值带进去就行了。

询问直接递归到对应叶子节点，累计路径上的信息就行了。

插入复杂度两个 log，查询一个 log。

## 应用

### [HEOI2013] Segment

> 要求在平面直角坐标系下维护两个操作（强制在线）：
>
> 1. 在平面上加入一条线段。记第 $i$ 条被插入的线段的标号为 $i$，该线段的两个端点分别为 $(x_0,y_0)$，$(x_1,y_1)$。
>
> 2. 给定一个数 $k$，询问与直线 $x = k$ 相交的线段中，交点纵坐标最大的线段的编号（若有多条线段与查询直线的交点纵坐标都是最大的，则输出编号最小的线段）。特别地，若不存在线段与给定直线相交，输出 $0$。
>
> 数据满足：操作总数 $1 \leq n \leq 10^5$，$1 \leq k, x_0, x_1 \leq 39989$，$1 \leq y_0, y_1 \leq 10^9$。

模板题，直接上代码：

这道题比较特殊，要求输出的不是值而是编号，且要求编号尽量小。

??? note "Code"

	有一个要注意的点，bool 转换的时候只要不是 0 就是 true ！！！！

	因为这个被坑了 5 hours！！！！！！！

	感谢 Uoj 群友，感谢可爱 do_while_true。

	```cpp

	// author : black_trees

	#include <cmath>
	#include <cstdio>
	#include <cstring>
	#include <utility>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'
	#define int long long

	using namespace std;
	// using i64 = long long;
	using ldb = long double;
	using pdi = std::pair<ldb, int>;

	const ldb eps = 1e-9;
	const int mod1 = 39989;
	const int mod2 = 1e9;
	const int si = 1e5 + 10;

	int n, tot = 0;
	struct Line { double k, b; } a[si];
	ldb calc(int idx, int x) { return (a[idx].k * x + a[idx].b); }
	void add(int x, int y, int xx, int yy) {
		++tot;
		if(x == xx) a[tot].k = 0, a[tot].b = max(y, yy);
		else a[tot].k = (ldb)((1.0 * (yy - y)) / (1.0 * (xx - x))), a[tot].b = y - a[tot].k * x;
	}
	int cmp(ldb x, ldb y) {
		if((x - y) > eps) return 1; // Greater.
		else if((y - x) > eps) return -1; // Less
		return 0;
	}
	pdi Max(pdi x, pdi y) { 
		if(cmp(x.first, y.first) == 1) return x;
		else if(cmp(y.first, x.first) == 1) return y;
		return (x.second < y.second) ? x : y;
	}

	struct LichaoTree {
		int id[si << 2];
		void modify(int p, int l, int r, int u) {
			int &v = id[p], mid = (l + r) >> 1;
			if(cmp(calc(u, mid), calc(v, mid)) == 1) 
				swap(u, v);
			int boundl = cmp(calc(u, l), calc(v, l));
			int boundr = cmp(calc(u, r), calc(v, r));
			if(boundl == 1 || (!boundl && u < v)) 
				modify(p << 1, l, mid, u);
			if(boundr == 1 || (!boundr && u < v))
				modify(p << 1 | 1, mid + 1, r, u);
		}
		void update(int p, int nl, int nr, int l, int r, int u) {
			if(l <= nl && nr <= r) 
				return modify(p, nl, nr, u);
			int mid = (nl + nr) >> 1;
			if(l <= mid) 
				update(p << 1, nl, mid, l, r, u);
			if(r > mid) 
				update(p << 1 | 1, mid + 1, nr, l, r, u);
		} 
		pdi query(int p, int l, int r, int x) {
			if(x < l || r < x) 
				return {0.0, 0};
			ldb ret = calc(id[p], x), mid = (l + r) >> 1;
			if(l == r) 
				return {ret, id[p]};
			return Max({ret, id[p]}, Max(query(p << 1, l, mid, x), query(p << 1 | 1, mid + 1, r, x)));
		}
	} tr;

	signed main() {
	 
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);

		int lasans = 0;
		auto decode = [&](int &v, const int mod) {
			v = (v + lasans - 1 + mod) % mod + 1;
		};

		cin >> n;
		for(int i = 1; i <= n; ++i) {
			int opt; cin >> opt;
			if(opt == 0) {
				int k; cin >> k, decode(k, mod1);
				cout << (lasans = tr.query(1, 1, mod1, k).second) << endl;
			}
			if(opt == 1) {
				int x, y, xx, yy;
				cin >> x >> y >> xx >> yy;
				decode(x, mod1), decode(xx, mod1), decode(y, mod2), decode(yy, mod2);
				if(x > xx) swap(x, xx), swap(y, yy);
				add(x, y, xx, yy), tr.update(1, 1, mod1, x, xx, tot);
			}
		}

		return 0;
	}
	```

### 维护凸壳

这玩意儿，谔谔，就直接把所有线段扔上去然后问题转化为询问 Min/Max，就很好做了。

理所当然的，这东西还可以用于斜率优化。

但是这个有时间再来写吧，放在这里。
