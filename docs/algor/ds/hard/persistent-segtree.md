
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

总结一下，主席树题只需要考虑，怎么维护历史版本以达到区间查询，

怎么凑出一个新的信息，放到一个新的线段树上，在这个新的线段树上进行操作。

（新的线段树并不需要实际合并出来，只需要多个指针同步遍历需要的版本即可）

## 几个简单的例题

### Luogu3567 [POI2014]KUR-Couriers

> 给一个数列，每次询问一个区间内有没有一个数出现次数超过一半
> 如果有输出这个数，否则输出 0.
> 5e5

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

### Luogu3919 【模板】可持久化线段树 1（可持久化数组）

> 首先给定一个长度为 $n \le 10^6$ 的数组，然后， 
> 实现一个可持久化的数组，支持以下操作：
> 1. 在某个历史版本上修改某一个位置上的值。
> 2. 询问某一个历史版本上某一个位置的值，并复制一个新版本。

很简单的板子题，不过这里不是主席树了，而是一般的可持久化线段树。

本质一样，不过可持久化线段树不需要一个一个插入原序列的元素，因为此处不是维护值域而是直接维护整个序列。

而且这里可以支持单点修改，主席树不套一个别的数据结构很难支持单点修改，因为主席树是维护的值域，在维持权值信息的区间可加减性的同时，对于序列直接操作很不方便。

具体实现就看代码吧，这里还不需要维护区间和，只需要维护叶子节点的值就行了。

如果要维护区间和就要加上 Pushup 了。

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
	
	const int si = 1e6 + 10;
	
	int n, m;
	int a[si];
	
	int tot = 0;
	int ls[si << 5], rs[si << 5];
	int root[si << 5], dat[si << 5];
	
	int build(int l, int r) {
		int p = ++tot;
		if(l == r) {
			dat[p] = a[l];
			return p;
		}
		int mid = (l + r) >> 1;
		ls[p] = build(l, mid), rs[p] = build(mid + 1, r);
		return p;
	}
	int modify(int last, int l, int r, int pos, int v) {
		int p = ++tot;
		if(l == r) { dat[p] = v; return p; }
		int mid = (l + r) >> 1;
		if(pos <= mid) 
			ls[p] = modify(ls[last], l, mid, pos, v), rs[p] = rs[last];
		else 
			rs[p] = modify(rs[last], mid + 1, r, pos, v), ls[p] = ls[last];
		return p;
	}
	int query(int p, int l, int r, int pos) {
		if(l == r) return dat[p];
		int mid = (l + r) >> 1;
		if(pos <= mid) return query(ls[p], l, mid, pos);
		else return query(rs[p], mid + 1, r, pos);
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n >> m;
		for(int i = 1; i <= n; ++i)
			cin >> a[i];
		root[0] = build(1, n);
	
		int cnt = 0;
		while(m --) {
			int ver, opt;
			cin >> ver >> opt;
			if(opt == 1) {
				int x, v;
				cin >> x >> v;
				root[++cnt] = modify(root[ver], 1, n, x, v);
			}
			else {
				int x; cin >> x;
				int val = query(root[ver], 1, n, x);
				root[++cnt] = modify(root[ver], 1, n, x, val);
				cout << val << endl;
			}
		}
	
		return 0;
	}
	```

### Luogu2633 Count on a tree

> 给定一棵 $n$ 个节点的树，每个点有一个权值。有 $m$ 个询问，
> 每次给你 $u,v,k$，你需要回答 $u \text{ xor last}$ 和 $v$ 这两个节点间第 $k$ 小的点权。
> 其中 $\text{last}$ 是上一个询问的答案，定义其初始为 $0$，即第一个询问的 $u$ 是明文。
> 1e5

发现第 $i$ 个版本的主席树可以感性的理解为一种前缀和。

于是在询问 $[l, r]$ 的时候可以把 `dat[r] - dat[l - 1]` 当作信息拍到一个新的线段树上操作（实际操作就是拿两个指针同步走两个版本的线段树）。

这里是在树上询问，可以考虑树剖，但是似乎很麻烦。

主席树维护的信息具有区间可加可减性，换句话说可以做前缀和也可以差分。

所以我们考虑怎么凑一个新的线段树，使他能包含路径 $(u,v)$ 的信息，然后在上面线段树二分就可以了。

我们设版本 $i$ 表示根节点到节点 $i$ 的路径上的所有节点构成的主席树，

然后发现此时可以树上差分来凑出路径 $(u, v)$，于是新的线段树的信息就是 `dat[u] + dat[v] - dat[lca(u, v)] - dat[fa(lca(u, v))]`。

实现直接拿四个指针同步遍历这四个版本即可。

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
	
	const int si = 1e5 + 10;
	
	int n, m, len, cnt;
	int a[si], id[si];
	
	int head[si << 1];
	struct Edge {
		int ver, Next;
	}e[si << 1];
	void add(int u, int v) {
		e[cnt].ver = v, e[cnt].Next = head[u], head[u] = cnt++;
	}
	
	int get_id(int val) {
		return lower_bound(id + 1, id + 1 + len, val) - id;
	}
	
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
	int ask(int p, int q, int u, int v, int l, int r, int kth) {
		if(l == r) return l;
		int mid = (l + r) >> 1;
		int lcnt = dat[ls[p]] + dat[ls[q]] - dat[ls[u]] - dat[ls[v]];
		if(kth <= lcnt) 
			return ask(ls[p], ls[q], ls[u], ls[v], l, mid, kth);
		else 
			return ask(rs[p], rs[q], rs[u], rs[v], mid + 1, r, kth - lcnt); 
	}
	
	int dep[si];
	int f[si][21];
	void dfs(int u, int fa) {
		f[u][0] = fa, dep[u] = dep[fa] + 1;
		for(int i = 1; i <= 20; ++i) 
			f[u][i] = f[f[u][i - 1]][i - 1];
		for(int i = head[u]; ~i; i = e[i].Next) {
			int v = e[i].ver;
			if(v == fa) continue;
			root[v] = insert(root[u], 1, len, get_id(a[v]));
			dfs(v, u);
		}
	}
	int lca(int u, int v) {
		if(dep[u] < dep[v]) swap(u, v);
		for(int i = 20; i >= 0; --i) 
			if(dep[f[u][i]] >= dep[v]) 
				u = f[u][i];
		if(u == v) return u;
		for(int i = 20; i >= 0; --i) 
			if(f[u][i] != f[v][i])
				u = f[u][i], v = f[v][i];
		return f[u][0];
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		memset(head, -1, sizeof head);
	
		cin >> n >> m;
		for(int i = 1; i <= n; ++i)
			cin >> a[i], id[i] = a[i];
		sort(id + 1, id + 1 + n);
		len = unique(id + 1, id + 1 + n) - id - 1;
	
		for(int i = 1; i < n; ++i) {
			int u, v; cin >> u >> v;
			add(u, v), add(v, u);
		}
	
		root[0] = build(1, len);
		root[1] = insert(root[0], 1, len, get_id(a[1]));
	
		dfs(1, 0);
	
		int lastans = 0;
		while(m --) {
			int u, v, k;
			cin >> u >> v >> k;
			u ^= lastans;
			int Lca = lca(u, v), Fa = f[Lca][0];
			cout << (lastans = id[ask(root[u], root[v], root[Lca], root[Fa], 1, len, k)]) << endl;
		}
	
		return 0;
	}
	```

## 支持单点修改的第 k 大

> Luogu2617 Dynamic Rankings
> 
> 单点修改区间询问第 $k$ 大，1e5。

直接暴力做的话，每次单点修改 $i$ 需要修改 $[i,n]$ 的所有版本，单次复杂度是 $O(n \log n)$ 的，不能接受。

可以想到主席树本质上是一个“广义”的前缀和，所以我们可以考虑使用树状数组维护主席树，来让需要修改的版本数减少到 $O(\log n)$ 个。

具体来说，我们每次修改 $x$ 这个版本的某个值，就修改 $x + \text{lowbit}(x), x + \text{lowbit}(x) + \text{lowbit}(x + \text{lowbit}(x)), \dots$ 这几个版本的这个值就行了。

然后修改的时候需要自己在自己版本上新建，因为如果直接修改可能会影响到后面的版本（后面的版本对前面的版本有依赖）。

查询就预处理出对应的两批 $O(\log n)$ 个主席树的信息，合并之后在上面线段树二分即可。

这里的代码使用了另外一种动态开点的写法，更节省空间。

注意一定不要忘记在初始的时候把主席树扔到树状数组上！

??? note "Code"
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
	using i64 = long long;

	const int si = 2e5 + 10;

	int n, m, len;
	int a[si], id[si << 1];

	int tot = 0;
	int ls[si << 8], rs[si << 8];
	int root[si << 8], dat[si << 8];

	int cnt1, cnt2;
	int tr1[si], tr2[si];

	struct Query { char opt; int l, r, x; } q[si];

	inline int lowbit(int x) { return x & -x; }
	inline int getid(int val) { return lower_bound(id + 1, id + 1 + len, val) - id; }

	int build(int l, int r) {
	    int p = ++tot;
	    if(l == r) return l;
	    int mid = (l + r) >> 1;
	    ls[p] = build(l, mid), rs[p] = build(mid + 1, r);
	    return p;
	}
	void insert(int &p, int last, int l, int r, int val, int delta) {
		p = ++tot;
		dat[p] = dat[last] + delta, ls[p] = ls[last], rs[p] = rs[last];
		if(l == r) return;
		int mid = (l + r) >> 1;
		if(val <= mid) insert(ls[p], ls[last], l, mid, val, delta);
		else insert(rs[p], rs[last], mid + 1, r, val, delta);
	}
	int ask(int l, int r, int kth) {
	    if(l == r) return l;
	    int mid = (l + r) >> 1;
	    int lcnt = 0;
	    for(int i = 1; i <= cnt2; ++i) lcnt += dat[ls[tr2[i]]];
	    for(int i = 1; i <= cnt1; ++i) lcnt -= dat[ls[tr1[i]]];

	    if(kth <= lcnt) {
	        for(int i = 1; i <= cnt1; ++i) tr1[i] = ls[tr1[i]];
	        for(int i = 1; i <= cnt2; ++i) tr2[i] = ls[tr2[i]];
	        return ask(l, mid, kth);
	    }   
	    else {
	        for(int i = 1; i <= cnt1; ++i) tr1[i] = rs[tr1[i]];
	        for(int i = 1; i <= cnt2; ++i) tr2[i] = rs[tr2[i]];
	        return ask(mid + 1, r, kth - lcnt);
	    }
	}

	void change(int x, int v) {
	    int y = getid(a[x]);
	    while(x <= n) {
	    	insert(root[x], root[x], 1, len, y, v);
	    	x += lowbit(x);
	    }
	}
	int query(int l, int r, int kth) {
	    l --, cnt1 = cnt2 = 0;
	    while(l) tr1[++cnt1] = root[l], l -= lowbit(l);
	    while(r) tr2[++cnt2] = root[r], r -= lowbit(r);
	    return ask(1, len, kth);
	}

	int main() {    

	    // cin.tie(0) -> sync_with_stdio(false);
	    // cin.exceptions(cin.failbit | cin.badbit);

	    cin >> n >> m;
	    int cnt = 0;
	    for(int i = 1; i <= n; ++i)
	        cin >> a[i], id[++cnt] = a[i];
	    for(int i = 1; i <= m; ++i) {
	        Query &p = q[i];
	        cin >> p.opt;
	        if(p.opt == 'C')
	            cin >> p.l >> p.x, id[++cnt] = p.x;
	        if(p.opt == 'Q')
	            cin >> p.l >> p.r >> p.x;
	    }

	    sort(id + 1, id + 1 + cnt);
	    len = unique(id + 1, id + 1 + cnt) - id - 1;

	    for(int i = 1; i <= n; ++i) change(i, 1);

	    for(int i = 1; i <= m; ++i) {
	        Query &p = q[i];
	        if(p.opt == 'C') change(p.l, -1), a[p.l] = p.x, change(p.l, 1);
	        if(p.opt == 'Q') cout << id[query(p.l, p.r, p.x)] << endl;
	    }

	    return 0;
	}

	// 下面这个虽然可以，不过空间可能就寄掉了，所以用上面的写法会好得多。
	/*
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
	using i64 = long long;

	const int si = 2e5 + 10;

	int n, m, len;
	int a[si], id[si << 1];

	int tot = 0;
	int ls[si << 9], rs[si << 9];
	int root[si << 9], dat[si << 9];

	int cnt1, cnt2;
	int tr1[si], tr2[si];

	struct Query { char opt; int l, r, x; } q[si];

	inline int lowbit(int x) { return x & -x; }
	inline int getid(int val) { return lower_bound(id + 1, id + 1 + len, val) - id; }

	int build(int l, int r) {
	    int p = ++tot;
	    if(l == r) return l;
	    int mid = (l + r) >> 1;
	    ls[p] = build(l, mid), rs[p] = build(mid + 1, r);
	    return p;
	}
	void insert(int &p, int last, int l, int r, int val, int delta) {
	    p = ++tot;
	    dat[p] = dat[last] + delta, ls[p] = ls[last], rs[p] = rs[last];
	    if(l == r) return;
	    int mid = (l + r) >> 1;
	    if(val <= mid) insert(ls[p], ls[last], l, mid, val, delta);
	    else insert(rs[p], rs[last], mid + 1, r, val, delta);
	}
	int ask(int l, int r, int kth) {
	    if(l == r) return l;
	    int mid = (l + r) >> 1;
	    int lcnt = 0;
	    for(int i = 1; i <= cnt2; ++i) lcnt += dat[ls[tr2[i]]];
	    for(int i = 1; i <= cnt1; ++i) lcnt -= dat[ls[tr1[i]]];

	    if(kth <= lcnt) {
	        for(int i = 1; i <= cnt1; ++i) tr1[i] = ls[tr1[i]];
	        for(int i = 1; i <= cnt2; ++i) tr2[i] = ls[tr2[i]];
	        return ask(l, mid, kth);
	    }   
	    else {
	        for(int i = 1; i <= cnt1; ++i) tr1[i] = rs[tr1[i]];
	        for(int i = 1; i <= cnt2; ++i) tr2[i] = rs[tr2[i]];
	        return ask(mid + 1, r, kth - lcnt);
	    }
	}

	void change(int x, int v) {
	    int y = getid(a[x]);
	    while(x <= n) {
	        insert(root[x], root[x], 1, len, y, v);
	        x += lowbit(x);
	    }
	}
	int query(int l, int r, int kth) {
	    l --, cnt1 = cnt2 = 0;
	    while(l) tr1[++cnt1] = root[l], l -= lowbit(l);
	    while(r) tr2[++cnt2] = root[r], r -= lowbit(r);
	    return ask(1, len, kth);
	}

	int main() {    

	    // cin.tie(0) -> sync_with_stdio(false);
	    // cin.exceptions(cin.failbit | cin.badbit);

	    cin >> n >> m;
	    int cnt = 0;
	    for(int i = 1; i <= n; ++i)
	        cin >> a[i], id[++cnt] = a[i];
	    for(int i = 1; i <= m; ++i) {
	        Query &p = q[i];
	        cin >> p.opt;
	        if(p.opt == 'C')
	            cin >> p.l >> p.x, id[++cnt] = p.x;
	        if(p.opt == 'Q')
	            cin >> p.l >> p.r >> p.x;
	    }

	    sort(id + 1, id + 1 + cnt);
	    len = unique(id + 1, id + 1 + cnt) - id - 1;

	    for(int i = 1; i <= n; ++i) 
	        root[i] = build(1, len);
	    for(int i = 1; i <= n; ++i) change(i, 1);

	    for(int i = 1; i <= m; ++i) {
	        Query &p = q[i];
	        if(p.opt == 'C') change(p.l, -1), a[p.l] = p.x, change(p.l, 1);
	        if(p.opt == 'Q') cout << id[query(p.l, p.r, p.x)] << endl;
	    }

	    return 0;
	}
	*/
	```


## 标记永久化

HDU4348 To the moon

暂时咕咕咕
