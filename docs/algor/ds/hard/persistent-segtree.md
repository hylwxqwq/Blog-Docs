
# 主席树

## 泛化

主席树全称可持久化权值线段树，因为权值线段树和线段树没有本质区别，就只说主席树了。

主席树的思想和可持久化 Trie 的思想是一致的，都是以函数式编程的方式保存历史版本，以把信息从全局转化为区间。

一个比较经典的问题就是静态区间第 k 大，权值线段树上二分显然可以很轻松搞定全局的版本。

于是把权值线段树可持久化之后利用前缀和的思想就可以做区间了。

不过主席树一般难以支持区间修改，最多支持一下单点修改。

区间修改在信息比较特殊的情况下可以用标记永久化做，不过也非常有局限性。

## 基本思想

仍然考虑静态区间第 k 大问题。

主席树的主要思想就是，对于一个序列，从左往右依次插入序列的每一个元素。

每次插入一个元素的时候维护历史版本，也就是维护每一个元素刚插入的时候，线段树的样子。

显然一个比较暴力的做法是，每次插入一个数（新建一个版本）的时候，都把原来的线段树复制一次，然后把新的版本加上去。

不过这样空间是 $O(nm)$ 的，非常不优秀。

发现每次插入一个数，在值域上只会修改一个点，对应到线段树上就是只修改一条从根节点到对应叶子节点的链。

所以我们可以考虑把这个被修改的链单独复制一次拉出来，记录一下插入之后，当前版本树根的编号。

于是空间开销就大大减小了，从 $O(nm)$ 变成了 $O(n + m\log n)$。

具体来说是这样的：

[![xEPuB6.png](https://s1.ax1x.com/2022/09/25/xEPuB6.png)](https://imgse.com/i/xEPuB6)

蓝色节点是原来版本上的链，红色节点是现在版本上的链。

简单来说就是，对于上一个版本的节点 $p$，复制一个新的节点 $q$ 出来。

如果下一步是递归 $p.ls$，那么 $q.ls$ 就需要新建，然后 $q.rs$ 就是原来的 $p.ls$，反过来同理。

查询 $[l, r]$ 的第 k 大也比较好做，就直接线段树二分，然后用两个指针 $p, q$，**同步**遍历 $l - 1$，$r$ 两个版本。

比较一下当前的 k 和 `lcnt = dat[ls[q]] - dat[ls[p]]` 的大小，如果 $k \le lcnt$，就让 $p,q$ 都往左子树走，否则 $k - lcnt$，然后走右子树就行了。

**可以理解成在两颗权值线段树合并之后的权值线段树上进行操作**，合并后的信息就是 $r$ 版本的信息减去 $l - 1$ 的信息。

（所以主席树维护的信息不仅要满足幺半群性质，还需要满足区间可减性！）

建树的时候先对于值域建一个每个节点权值都为空的线段树，根节点标记为第零个版本的根节点，方便之后更新。

空间开个 `<<5` 就行了，反正一般能用主席树的题空间都很宽松。

??? note "代码：[POJ2104 - K-th Number](http://poj.org/problem?id=2104)"
	```cpp
	// author : black_trees
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define endl '\n'
	
	#ifndef ONLINE_JUDGE
	  #include<cstdarg>
	  #define meow(format, ...) \
	    fprintf(stderr, format, ## __VA_ARGS__)
	  // remember to open stream sync!
	#else
	  #define meow(format, ...) 1231
	#endif
	
	using namespace std;
	// using i64 = long long;
	
	template <typename __Tp> void read(__Tp &x) {
	    int f = x = 0; char ch = getchar();
	    for (; !isdigit(ch); ch = getchar()) if (ch == '-') f = 1;
	    for (; isdigit(ch); ch = getchar()) x = (x << 1) + (x << 3) + (ch ^ 48);
	    if (f) x = -x;
	}
	// void read(modint &x) { int __value; read(__value); x = __value; return; } 
	void read(char &ch) { for (ch = getchar(); isspace(ch); ch = getchar()); }
	// template <typename __Tp1, typename ...__Tp2> void read(__Tp1 &x, __Tp2 &... y) { read(x), read(y...); }
	template <typename __Tp> void write(__Tp x) {
	    if (x < 0) putchar('-'), x = -x;
	    if (x > 9) write(x / 10);
	    putchar(x % 10 + 48);
	}
	void write(char ch) { putchar(ch); }
	// void write(modint x) { write(x.val()); }
	void write(const char *s) { for (; *s; ++s) putchar(*s); }
	// template <typename __Tp1, typename ...__Tp2> void write(__Tp1 x, __Tp2 ... y) { write(x), write(y...); }
	
	const int si = 1e5 + 10;
	
	int n, m, len;
	int a[si], id[si];
	
	int tot = 0;
	int ls[si << 5], rs[si << 5];
	int root[si << 5], dat[si << 5];
	
	int build(int l, int r) {
		int p = ++tot;
		if(l == r) return p;
		int mid = (l + r) >> 1;
		ls[p] = build(l, mid), rs[p] = build(mid + 1, r);
		return p;
	}
	int insert(int last, int l, int r, int val) { // last 是上一个版本的 [l, r] 节点。
		int p = ++tot;
		dat[p] = dat[last] + 1;
		if(l == r) return p;
		int mid = (l + r) >> 1;
		if(val <= mid) 
			ls[p] = insert(ls[last], l, mid, val), rs[p] = rs[last];
		else 
			rs[p] = insert(rs[last], mid + 1, r, val), ls[p] = ls[last];
		return p;
	}
	int ask(int p, int q, int l, int r, int kth) {
		if(l == r) return l;
		int mid = (l + r) >> 1;
		int lcnt = dat[ls[q]] - dat[ls[p]];
		if(kth <= lcnt) 
			return ask(ls[p], ls[q], l, mid, kth);
		else 
			return ask(rs[p], rs[q], mid + 1, r, kth - lcnt);
	}
	
	int index(int val) {
		return lower_bound(id + 1, id + 1 + len, val) - id;
	}
	
	int main() {
	
		read(n), read(m);
		for(int i = 1; i <= n; ++i)
			read(a[i]), id[i] = a[i];
		sort(id + 1, id + 1 + n);
		len = unique(id + 1, id + 1 + n) - id - 1;
		root[0] = build(1, len);
	
		for(int i = 1; i <= n; ++i)
			root[i] = insert(root[i - 1], 1, len, index(a[i]));
	
		while(m --) {
			int l, r, k; read(l), read(r), read(k);
			write(id[ask(root[l - 1], root[r], 1, len, k)]);
			write(endl);
		}
	
		return 0;
	}
	```

复杂度显然 1log。

## 例题

### Luogu3567 [POI2014]KUR-Couriers

> 给一个数列，每次询问一个区间内有没有一个数出现次数超过一半

妥妥的板子题，就是一个简单的线段树二分就行了。

??? note "Code" 

	```cpp
	// author : black_trees

	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	using i64 = long long;

	const int si = 5e5 + 10;

	int n, m, len;
	int a[si], id[si];

	int tot = 0;
	int ls[si << 5], rs[si << 5];
	int root[si << 5], dat[si << 5];

	int get_id(int val) {
		return lower_bound(id + 1, id + 1 + len, val) - id;
	}
	int build(int l, int r) {
		int p = ++tot;
		if(l == r) return p;
		int mid = (l + r) >> 1;
		ls[p] = build(l, mid), rs[p] = build(mid + 1, r);
		return p;
	}
	int insert(int last, int l, int r, int v) {
		int p = ++tot;
		dat[p] = dat[last] + 1;
		if(l == r) return p;
		int mid = (l + r) >> 1;
		if(v <= mid) 
			ls[p] = insert(ls[last], l, mid, v), rs[p] = rs[last];
		else 
			rs[p] = insert(rs[last], mid + 1, r, v), ls[p] = ls[last];
		return p;
	}
	int ask(int p, int q, int l, int r, int k) {
		if(l == r) return l;
		int mid = (l + r) >> 1;
		int lcnt = dat[ls[q]] - dat[ls[p]];
		int rcnt = dat[rs[q]] - dat[rs[p]];
		if(k < lcnt) return ask(ls[p], ls[q], l, mid, k);
		if(k < rcnt) return ask(rs[p], rs[q], mid + 1, r, k);
		return 0;
	}

	int main() {	

		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);

		cin >> n >> m;
		for(int i = 1; i <= n; ++i)
			cin >> a[i], id[i] = a[i];
		sort(id + 1, id + 1 + n);
		len = unique(id + 1, id + 1 + n) - id - 1;

		root[0] = build(1, len);
		for(int i = 1; i <= n; ++i)
			root[i] = insert(root[i - 1], 1, len, get_id(a[i]));

		while(m --) {
			int l, r;
			cin >> l >> r;
			cout << id[ask(root[l - 1], root[r], 1, len, (r - l + 1) >> 1)] << endl;
		}

		return 0;
	}
	```