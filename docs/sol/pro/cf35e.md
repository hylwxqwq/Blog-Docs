
## 题意

给出 $n$ 个底边在 $x$ 轴上的矩形，求外面的轮廓线顶点。

$1\le n \le 10^5$，输入以 $h_i, l_i, r_i$ 的方式给出，表示有一个覆盖 $x \in [l_i, r_i]$ 的高为 $h_i$ 的矩形。

## 题解

首先考虑离散化然后直接一个扫描线扫过去（从左到右）。

然后注意到拐点只会在整体最大高度变化的时候出现。

所以我们考虑直接维护一下扫描线上的 $y$ 的最大值，记 $maxy$ 表示线段树上一个节点内被覆盖到的最高 $y$ 值，记 $cnt$ 表示线段树上的节点自身被覆盖的次数，记 $len$ 表示这个节点维护的段内总共被覆盖的长度（记录这个是为了判断是否能更新 $maxy$）

令第 $i$ 段为 $[raw(i), raw(i + 1)]$，令线段树上的 $[l, r]$ 维护第 $l \sim r$ 段。 

修改的时候注意，如果对于扫描线上的 $[l, r]$，操作时应当修改线段树上的 $[val(l), val(r) - 1]$。

因为我们维护的是一个段，维护点是没有意义的。

这里的 Pushup 方式和主流不太一样，在 Change 到一个节点的时候仍然需要做类似 Pushup 的处理，这样可以省去 Pushdown。

具体来说，对于 Pushup，如果当前节点的 $cnt > 0$，那么令 $len = raw(r + 1) - raw(l)$ 并令 $maxy = raw(r + 1)$，否则考虑从儿子节点更新 $len$，如果更新完之后 $len > 0$，再从子树更新 $maxy$。

然后在修改的时候也要直接处理更改，方式类似，只不过在处理 $cnt = 0$ 的情况的时候，需要先判是否为叶子节点（不然要额外再开空间），然后再从子树上传信息，这里就算修改了 $cnt$，还是需要考虑用儿子节点信息更新 $maxy$，不然没法处理有多个矩形的更改堆在一起的情况。

那么每次更新完我们只需要询线段树树根的 $maxy$ 即可，并和修改前的 $maxy^{\prime}$ 比较，如果发生改变，就直接把 $(x, maxy), (x, maxy^{\prime})$ 扔进 Vector 就行。

还有一个要注意的地方是，因为这个做法是在每次 Query 之后直接比对并更新答案，所以如果在一个 $x$ 上有多个更改要处理，需要一口气全部处理完再更新，否则会多出来一些实际上不存在的顶点。

最后空间记得开够，不然会 RE on #22.

??? note "Code - 保留注释版本"

	```cpp
	// author : black_trees

	#include <cmath>
	#include <cstdio>
	#include <vector>
	#include <cstring>
	#include <utility>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'
	// #define int long long

	using namespace std;
	using i64 = long long;

	const int si = 1e5 + 10;

	int n, m; // m 是离散化之后的值域上界(1~m).
	std::vector<int> v;
	int raw[si * 3];
	int val(int x) { return lower_bound(v.begin(), v.end(), x) - v.begin(); }

	struct Tuple {
		int l, r, k;
		void trans() {
			int vl = val(l), vr = val(r), vk = val(k);
			raw[vl] = l, raw[vr] = r, raw[vk] = k, l = vl, r = vr, k = vk;
		} 
		bool operator < (const Tuple &b) const {
			if(l == b.l) return r > b.r;  // 这里是一开始的想法，后面用同时计算解决了。
			return l < b.l;
		}
	} a[si], b[si << 1]; // 这里是同时存输入的信息和三元组信息的

	// how to maintain maxy??
	class SegTree {
		private:
			struct Node {
				int l, r;
				int cnt, len, mxy;
				int Len() { return raw[r + 1] - raw[l]; } 
			} t[si * 11]; // 可能要开大一点，因为值域？
			inline void pushup(int p) {
				if(t[p].cnt > 0) t[p].len = t[p].Len(), t[p].mxy = raw[t[p].r + 1];
				else {	
					t[p].len = t[p << 1].len + t[p << 1 | 1].len;
					if(t[p].len > 0) t[p].mxy = max(t[p << 1].mxy, t[p << 1 | 1].mxy);
					else t[p].mxy = 0;
				}
			}
		public:
			void build(int p, int l, int r) {
				t[p].l = l, t[p].r = r, t[p].mxy = t[p].len = t[p].cnt = 0;
	// if(abs(l) > 1e9 || abs(r) > 1e9) exit(2);
	// if(p > 8e5 + 80) exit(p);
				if(l == r) return;
				int mid = (l + r) >> 1;
				build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
			}
			void change(int p, int l, int r, int v) {
				int nl = t[p].l, nr = t[p].r;
				if(l <= nl && nr <= r) {
					t[p].cnt += v;
					if(t[p].cnt > 0) t[p].len = t[p].Len(), t[p].mxy = raw[t[p].r + 1];
					else {
						if(nl == nr) t[p].len = 0, t[p].mxy = 0;
						else {
							t[p].len = t[p << 1].len + t[p << 1 | 1].len;
							if(t[p].len > 0) t[p].mxy = max(t[p << 1].mxy, t[p << 1 | 1].mxy);
							else t[p].mxy = 0;
						}
					}
					return;
				}
				int mid = (nl + nr) >> 1;
				if(l <= mid) change(p << 1, l, r, v);
				if(r > mid) change(p << 1 | 1, l, r, v);
				pushup(p);
			}
			int Query_global() { return t[1].mxy; } 
	} tr; 

	std::vector<std::pair<int, int> > ans;

	signed main() {

		freopen("input.txt", "r", stdin);
		freopen("output.txt", "w", stdout);

		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);

		cin >> n, v.push_back(0);
		for(int i = 1; i <= n; ++i) {
			cin >> a[i].k >> a[i].l >> a[i].r;
			v.push_back(a[i].k), v.push_back(a[i].l), v.push_back(a[i].r);
		}
		sort(v.begin(), v.end()); v.erase(unique(v.begin(), v.end()), v.end());
		for(int i = 1; i <= n; ++i) a[i].trans();
		tr.build(1, 1, (int)v.size() + 1);
		
		for(int i = 1; i <= n; ++i) {
			b[i] = (Tuple){a[i].l, a[i].k, 1};
			b[i + n] = (Tuple){a[i].r, a[i].k, -1};
		}
		sort(b + 1, b + 1 + n + n);
	 
	/*
	// for(int i = 1; i <= n + n; ++i)
	// cout << i << " -> " << raw[b[i].l] << " " << raw[b[i].r] << " " << b[i].k << endl;
	// cout << "====" << endl;
		int nowy = 0;
		for(int i = 1; i <= n + n; ++i) {
			int x = raw[b[i].l], l = val(0), r = b[i].r, v = b[i].k;
	// cout << i << " -> " << l << " " << r << " " << v << endl;
			tr.change(1, l, r - 1, v);
			int nmxy = tr.Query_global();
	// cout << i << " -> " << nowy << " " << nmxy << endl;
			if(nmxy != nowy) {
				ans.push_back(make_pair(x, nowy));
				ans.push_back(make_pair(x, nmxy));
				nowy = nmxy;
			}
		} // sweeping line
	*/
		int nowy = 0;	
		for(int i = 1; i <= n + n; ) {
			int x = raw[b[i].l];
			int l, r, v;
			while(true) {
				l = val(0), r = b[i].r, v = b[i].k;
				tr.change(1, l, r - 1, v);
				if(raw[b[i + 1].l] != x) break;
				i++;
			}
			int nmxy = tr.Query_global();	
			if(nmxy != nowy) {
				ans.push_back(make_pair(x, nowy));
				ans.push_back(make_pair(x, nmxy));
				nowy = nmxy;
			}
			i++;
		}

		// ans.erase(unique(ans.begin(), ans.end()), ans.end());
		cout << ans.size() << endl;
		for(auto [x, y] : ans) cout << x << " " << y << endl;

		// 第 i 段是 [raw(i), raw(i + 1)]
		// 所以如果加入了一个元组 (x, l, r, +-1);（存的 val）
		// 修改是 change(l, r - 1)。
		
		// 现在的问题出在处理信息的地方，而且肯定不是赋值的错误，应该是某个顺序错了或者覆盖了不应该覆盖的值。
		// 好，现在处理信息没有问题了，现在要做的应该是考虑怎么处理那个重合的地方。
		// 能不能考虑把同一个 $x$ 位置的元组全部同时做更改？应该可以，明天来改, 11点了，先睡觉。
		// 确实可以，但是现在问题是一直 Re22，cf 说 line 54 RE，我没看出来啊，数组也开够了？
		// 突然想起可能是 p RE了，用 exit 试一试。
		// 我草，真的是，改了下就过了。
		// 感觉这个是非常 Educational的，特别是思考和调试过程。
		return 0;
	}

	// done:
		// 好像这个 maxy 维护之后单调不降了？
		// 搓一个数据试试，果然单调不降了，可能要改改 maxy 的维护？
		// 是不是因为 cnt = 0 或者 len = 0 之后 maxy 没有强制传递上去？
		// 确实，改了一下 maxy 的 update 方式就行了。

	// 先要离散化，对于每个输入先转化成两个三元组，然后把三元组直接存起来
	// 这个时候不着急排序，先输入完，然后把每一个数字都扔进vector里面离散化。
	// 记 val(i) 表示 i 离散后的结果，raw(i) 表示 i 离散前的结果。
	// 哎哎，反正 2e5 个三元组，多几个 log 也没有关系，raw 直接记录，val 直接每次 log 二分一下就行。
	// 实现方便可以在三元组的结构体里面直接用一个成员函数来把 raw -> val.
	// 这是离散化的部分。
	//
	```

## 调试复盘

Debug 这个题的过程感觉很有必要记下来，我之前几乎没有怎么这样用心的思考调试过，可能也是水平涨不上去的原因吧。

首先我认为如果一个地方有重复的，计算时可能会出问题，所以写了一个 cmp。

然后理了一遍思路写出了代码，过不了 sample1，看了一下 output。

发现是少了一些顶点没有输出，观察输出了的点发现一个规律，似乎当 $h = 4$ 出现之后，就算它被删掉了，$h = 2$ 还是不会被计算，直接被 ignore 了。

我猜是 $maxy$ 没能在 $-v$ 之后更新成功，于是定位到 maxy 的更新那个地方，发现 cnt, len 这两个东西就是和扫描线模板一样的，不会有问题，所以就更确定 maxy 出事情了。

然后思考了一下原因，发现是没有按 len 更新，于是重构写出代码，在 $cnt = 0$ 的时候多加了特判，先判 $len$ 再考虑是否需要从儿子节点的 maxy 上传信息。

此时过了 sample1，但是输出顺序有问题，思考了一下应该是要加一个特判？

但是输出此时并没有变化，放下这个休息了一会，回来发现我之前的代码有个 `sort(ans.begin(), ans.end())`，赶紧删掉了，然后发现其实不需要特判，然后就过了 sample1。

但是 Wa on 2，发现好像还有点问题，具体是啥我忘记了，反正改了一下过了 2，Wa on 3.

上 cf 翻出 #3 的数据，手元画图，对比了一下 output 和 forward，发现是在有多个处理重合的时候没有弄好，之前有的 data 这里并没有删，被 ignore 掉换成了在这里新更新的一个 data。

于是猜测又是 maxy 出问题了，思考了一会发现确实，change 完了也是需要考虑从儿子更新的，不然之前的信息可能就传不上来。

然后过了 #21, RE on #22，发现可能是数组没开够，开了之后顺手开了 long long，但是没用。

看 CF 一直给我提示 line 54 out of bounds，不懂，于是用类似 assert 的方式判了一下 l, r。

但是没用啊。

突然思考到可能 $p$ 会越界，`exit(p)` 发现 p > si * 8 了，于是开大，但是 MLE on 1，关掉 long long 就过了。 

debug 过程全部录了屏的，在 bilibili 上面：

- [第一部分，帧率很低，比特率很低。](https://www.bilibili.com/video/BV1JA411X7Hk/)
- [第二部分，帧率很高，比特率很低。](https://www.bilibili.com/video/BV1qg411t7nf/)
- [第三部分，帧率很高，比特率还行。](https://www.bilibili.com/video/BV1uR4y1r79S/)
