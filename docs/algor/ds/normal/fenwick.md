
## 概述

数论里面有一个很经典的结论，任意正整数都能被唯一拆分成若干个不同的 $2$ 的次幂的和

转化一下，对于前缀 $[1,x]$，假设 $x$ 可以被拆分成 $2^{c_1} + 2^{c_2} + 2^{c_3} + \dots 2^{c_m}$，那么 $[1,x]$ 就可以被拆分成 $m$ 个小区间：$[1, 2^{c_1}], [2^{c_1} + 1, 2^{c_1} + 2^{c_2}], \dots [2^{c_m - 1} + 1, 2^{c_m}]$。

其中 $m = O(\log x), c_i \in [0, m], c_1 > c_2 > c_3 \dots > c_m$。

如果 $x$ 的拆分包含 $2^{c_i}$，那么 $x$ 二进制表示下的第 $c_i$ 位就是 $1$（最低位为第 $0$ 位）。

有一个经典的运算叫 $\text{lowbit}$，可以取出 $x$ 二进制表示下的最低的 $1$ 和比它更低的所有 $0$ 组成的数。

于是我们就可以用 $\text{lowbit}$ 来遍历前缀 $[1,x]$ 分成的这些区间：

```cpp
int lowbit(int x) { return x & -x;} 

while(x) {
	cout << x - lowbit(x) + 1 << " " << x << endl;
	x -= lowbit(x);
}
```

树状数组差不多就是利用这个思想，定义 $t(x)$ 为 $\sum\limits_{i = x - \text{lowbit}(x) + 1}^x a(i)$，然后通过一些性质 $O(\log n)$ 地快速维护序列前缀和，并且支持单点修改。

结构如图：

[![vWI5SH.png](https://s1.ax1x.com/2022/08/28/vWI5SH.png)](https://imgse.com/i/vWI5SH)

实际上树状数组是一个森林，因为当 $n$ 不是 $2$ 的整数次幂时，树根是不唯一的。

树状数组具有的性质（知道 $1 \sim 4$ 条就行）：

- $t(x),x \not\in \text{root}$ 的父节点是 $t(x + \text{lowbit}(x))$
- $t(x)$ 的子节点个数是 $\text{lowbit}(x)$ 的位数（最低的一位 $1$ 是第几位，第 $0$ 位是最低位）。
- $t(x)$ 保存的是子树叶节点（第 0 层）和。
- 深度为 $O(\log n)$。
- 第 $i$ 层的节点间距为 $2^{i - 1}$。
- 节点编号的 $\text{lowbit}$ 为 $2^{i-1}$ （实际上就是把所有 $t(x)$ 按 $\text{lowbit}$ 分类）
- 第 $i$ 层的节点个数为 $\lfloor\dfrac{n + 1}{2^i}\rfloor$
- 第 $i$ 层的节点本质上是在确定 $\text{lowbit}$ 之后，不断在 $\text{lowbit}$ 后面的位放 $1$（前面不能放，不然 $\text{lowbit}$ 就变了）。

## 操作

### 单点修改

比较简单，修改位置 $a(i)$，本质上就是先修改 $t(i)$，然后向上不断更新父节点权值就行了。

```cpp
void add(int x, int v) {
	while(x <= n) {
		t[x] += v;
		x += lowbit(x);
	}
}
```

复杂度是 $O(\log n)$。

### 建树

直接对于每个 $a(i)$，$\text{add}(i, a(i))$ 即可。

不过这样复杂度是 $O(n\log n)$ 的，还有两种 $O(n)$ 建树的方法：

法一：直接处理前缀和，然后赋值给 $t$ （要消耗空间换时间）。

法二：每一个节点的值是由所有与自己直接相连的儿子的值求和得到的。因此可以倒着考虑贡献，即每次确定完儿子的值后，用自己的值更新自己的父亲，这样可以减少运算次数。

```cpp
for (int i = 1; i <= n; ++i) {
    t[i] += a[i];
    int j = i + lowbit(i);
    if (j <= n) t[j] += t[i];
}
```

### 查询前缀和

直接利用 $\text{lowbit}$ 扫一遍 $[1,x]$ 分成的 $O(\log x)$ 个小区间就行，

因为定义了 $t(x) = \sum\limits_{i = x - \text{lowbit}(x) + 1}^x a(i)$，所以直接累加就行了。

```cpp
int query(int x) {
	int ret = 0;
	while(x) {
		ret += t[x];
		x -= lowbit(x);
	}
	return ret;
}
```

复杂度 $O(\log n)$。

区间查询减一下就行。

### 时间戳优化

对付多组数据很常见的技巧。

如果每次输入新数据时，都暴力清空树状数组，就可能会造成超时。

因此使用 $tag$ 标记，存储当前节点上次使用时间（即最近一次是被第几组数据使用）。每次操作时判断这个位置 $tag$ 中的时间和当前时间是否相同，就可以判断这个位置应该是 $0$ 还是当前数据的 $a$ 数组内的值。

如果修改的时候遇到了之前用的节点，就直接先置零再修改打 $tag$。

如果询问的时候遇到了之前的节点，证明这个位置在这组数据都没被动过，直接不加就行了。

```cpp
void reset() { ++dfn; }
void add(int k, int v) {
    while(k <= n) {
    	if(tag[k] != dfn) t[k] = 0;
    	t[k] += v, tag[k] = dfn;
	    k += lowbit(k);
	}
}
int query(int k) {
	int ret = 0;
	while(k) {
		if(tag[k] == dfn) ret += t[k];
		k -= lowbit(k);
	}
	return ret;
}
```

## 扩展

### 区间修改单点查询

用树状数组维护差分数组 $c$ 即可。

单点查直接用树状数组求前缀和。

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
	
	int n, q;
	int a[si], c[si];
	i64 t[si];
	
	inline int lowbit(int x) {
		return x & -x;
	}
	void add(int p, int x) {
		while(p <= n) {
			t[p] += x;
			p += lowbit(p);
		}
	}
	i64 query(int p) {
		i64 ret = 0;
		while(p) {
			ret += t[p];
			p -= lowbit(p);
		}
		return ret;
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n >> q;
		a[0] = 0;
		for(int i = 1; i <= n; ++i)
			cin >> a[i], c[i] = a[i] - a[i - 1], add(i, c[i]);
	
		while(q--) {
			int opt; cin >> opt;
			if(opt == 1) {
				int l, r, x;
				cin >> l >> r >> x;
				add(l, x), add(r + 1, -x);
			}
			else {
				int i;
				cin >> i;
				cout << query(i) << endl;
			}
		} 
	
		return 0;
	}
	```

### 区间修改区间查询

还是维护差分数组，写出已知条件可以得到：$a(i) = \sum\limits_{j = 1}^{i} c(j)$。

然后区间查询可以转化成前缀求和，本质上是求这个东西： $\sum\limits_{j = 1}^i a(j) = \sum\limits_{j = 1}^{i}\sum\limits_{k = 1}^j c(k)$。

但是直接求复杂度过高没法接受，考虑怎么把 Sigma 拆掉（减少），转化成几个可以维护前缀和的式子用树状数组维护。

我们发现，每个 $c(k)$ 被计算的次数是固定的，在求前缀 $[1,i]$ 的和的时候， $c(k), k\in[1,i]$ 被加的次数一共就是 $i - k + 1$。

也就是我们可以用算每个节点对于答案的贡献的方式来替代直接 Sigma，可以少一层 Sigma，而且能推出直接能维护前缀和的式子。

所以写成：$\sum\limits_{j = 1}^ic(j)\times (i - j + 1)$。

然后我们把**常数项和所有的变量都分开求和**：

$$
\sum\limits_{j = 1}^ic(j)\times (i - j + 1) = (i + 1)\cdot\sum\limits_{j = 1}^{i}c(j) - \sum\limits_{j = 1}^i [c(j)\times j]
$$

所以维护两个树状数组，一个维护 $c(i)$，一个维护 $c(i) \times i$。

修改类似一维差分，询问类似一维前缀和（树状数组本质上就是前缀和的升级）。

??? note "Code"
	```cpp
	// author : black_trees
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#pragma GCC target("avx,sse2,sse3,sse4,mmx")
	#pragma GCC optimize("Ofast", "inline", "-ffast-math")
	
	#define endl '\n'
	
	using namespace std;
	using i64 = long long;
	
	const int si = 1e6 + 10;
	
	int n, q;
	int a[si], c[si];
	i64 t[si], tt[si];
	int lowbit(int x) {
		return x & -x;
	}
	void add(int p, int x) {
		int i = p;
		while(p <= n) {
			t[p] += 1ll * x, tt[p] += 1ll * x * 1ll * i;
			p += lowbit(p);
		} 	
	}
	i64 query(int p) {
		int i = p;
		i64 ret = 0, rett = 0;
		while(p) {
			ret += t[p], rett += tt[p];
			p -= lowbit(p);
		}
		return (1ll * i + 1ll) * ret - rett;
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n >> q;
		a[0] = 0;
		for(int i = 1; i <= n; ++i)
			cin >> a[i], c[i] = a[i] - a[i - 1], add(i, c[i]); 
	
		while(q--) {
			int opt; cin >> opt;
			if(opt == 1) {
				int l, r, x;
				cin >> l >> r >> x;
				add(l, x), add(r + 1, -x);
			}
			else {
				int l, r;
				cin >> l >> r;
				cout << query(r) - query(l - 1) << endl;
			}
		}
		return 0;
	}
	```	

### 单点修改矩阵求和

这个也比较简单，类比一维的树状数组，可以得到 $t(x,y)$ 表示 $(x - \text{lowbit}(x) + 1, y - \text{lowbit}(y) + 1) \to (x, y)$ 的子矩阵和，然后 $\log^2$ 求一下，询问直接二维前缀和减一减就行了。

??? note "Code"
	```cpp
	// author : black_trees
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#pragma GCC target("avx,sse2,sse3,sse4,mmx")
	#pragma GCC optimize("Ofast", "inline", "-ffast-math")
	
	#define endl '\n'
	
	using namespace std;
	using i64 = long long;
	
	const int si = (1 << 12) + 1;
	
	int n, m;
	i64 t[si][si];
	
	inline int lowbit(int x) {
		return x & -x;
	}
	void add(int x, int y, int z) {
		int yy = y;
		while(x <= n) {
			y = yy;
			while(y <= m) {
				t[x][y] += z;
				y += lowbit(y);
			}
			x += lowbit(x);
		}
	}
	i64 query(int x, int y) {
		i64 ret = 0;
		int yy = y;
		while(x) {
			y = yy;
			while(y) {
				ret += t[x][y];
				y -= lowbit(y);
			}
			x -= lowbit(x);
		}
		return ret;
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		// cin.exceptions(cin.failbit | cin.badbit);
	
		cin >> n >> m;
		// loj133 这题初始 A 是零矩阵，所以不用建树了。
		int opt;
		while(cin >> opt) {
			if(opt == 1) {
				int x, y, k;
				cin >> x >> y >> k;
				add(x, y, k);
			}
			else {
				int a, b, c, d;
				cin >> a >> b >> c >> d;
				cout << query(c, d) + query(a - 1, b - 1) - query(c, b - 1) - query(a - 1, d) << endl;
			}
		}
	
		return 0;
	}
	```
注意不要忘了记录一下 $y$ 的值，每次循环结束重新赋值一次。

### 矩阵修改单点查询

这个直接维护二维的差分数组就行，套路和一维一样。

??? note "Code"
	```cpp
	// author : black_trees
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#pragma GCC target("avx,sse2,sse3,sse4,mmx")
	#pragma GCC optimize("Ofast", "inline", "-ffast-math")
	
	#define endl '\n'
	
	using namespace std;
	using i64 = long long;
	
	const int si = (1 << 12) + 1;
	
	int n, m;
	i64 t[si][si];
	
	inline int lowbit(int x) {
		return x & -x;
	}
	void add(int x, int y, int z) {
		int yy = y;
		while(x <= n) {
			y = yy;
			while(y <= m) {
				t[x][y] += z;
				y += lowbit(y);
			}
			x += lowbit(x);
		}
	}
	i64 query(int x, int y) {
		i64 ret = 0;
		int yy = y;
		while(x) {
			y = yy;
			while(y) {
				ret += t[x][y];
				y -= lowbit(y);
			}
			x -= lowbit(x);
		}
		return ret;
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		// cin.exceptions(cin.failbit | cin.badbit);
	
		int opt;
	
		cin >> n >> m;
		// loj134 这题初始 A 是零矩阵，所以不用构造差分矩阵然后建树了。
		while(cin >> opt) {
			if(opt == 1) {
				int a, b, c, d, k;
				cin >> a >> b >> c >> d >> k;
				add(a, b, k), add(a, d + 1, -k), add(c + 1, b, -k), add(c + 1, d + 1, k);
			}
			else {
				int x, y;
				cin >> x >> y;
				cout << query(x, y) << endl;
			}
		}
	
		return 0;
	}
	```

### 矩阵修改矩阵查询

类比区间修改区间查询的套路，算每个 $c(u,v)$ 在求二维前缀 $(1, 1) \to (i, j)$ 的答案的时候被算了多少次，然后推个式子拆掉 Sigma，常数项和变量分开求和（参变分离）就行。

$$
a(i, j ) = \sum\limits_{x =1}^{i}\sum\limits_{y = 1}^{j} c(i,j)
$$

$$
\sum\limits_{x = 1}^{i}\sum\limits_{y = 1}^{j} a(i, j) = \sum\limits_{x = 1}^{i}\sum\limits_{y = 1}^j\sum\limits_{u = 1}^x \sum\limits_{v = 1}^{y} c(u, v)
$$

$$
\Rightarrow\sum\limits_{u = 1}^{i}\sum\limits_{v = 1}^j c(u,v)\cdot (ij - (u-1)j - i(v - 1) + (u - 1)(i - 1))
$$

$$
\Rightarrow\sum\limits_{u = 1}^i \sum\limits_{v =1 }^j c(u,v) \cdot (i j - uj + j -iv + i +ui - u - i + 1)
$$

$$
\Rightarrow\sum\limits_{u = 1}^i \sum\limits_{v = 1}^j c(u, v) \cdot(i -u + 1)\cdot(j - v + 1)
$$

$$
\Rightarrow(i + 1)(j + 1)\sum\limits_{u = 1}^i \sum\limits_{v =1 }^j c(u,v) - (j + 1)\sum\limits_{u = 1}^i \sum\limits_{v =1 }^j c(u,v) \cdot u - (i + 1)\sum\limits_{u = 1}^i \sum\limits_{v =1 }^j c(u,v) \cdot v + \sum\limits_{u = 1}^i \sum\limits_{v =1 }^j c(u,v) \cdot uv
$$

所以维护四个树状数组就行了，注意可以用一下内存连续+循环展开+尽量不做过多的乘法，减小常数（似乎树状数组本来常数就很小？）

修改类似二维差分，询问类似二维前缀和。

这份代码目前在 Loj 上是最优解 1244ms（8/27/22）。

??? info "ScreenShots"
	[![vW7Vy9.png](https://s1.ax1x.com/2022/08/28/vW7Vy9.png)](https://imgse.com/i/vW7Vy9)

??? note "Code"
	```cpp
	// author : black_trees
	
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#pragma GCC target("avx,sse2,sse3,sse4,mmx")
	#pragma GCC optimize("Ofast", "inline", "-ffast-math")
	
	#define endl '\n'
	
	using namespace std;
	using i64 = long long;
	
	template <typename __Tp> void read(__Tp &x) {
	    int f = x = 0; char ch = getchar();
	    for (; !isdigit(ch); ch = getchar()) if (ch == '-') f = 1;
	    for (; isdigit(ch); ch = getchar()) x = (x << 1) + (x << 3) + (ch ^ 48);
	    if (f) x = -x;
	}
	// void read(modint &x) { int __value; read(__value); x = __value; return; } 
	void read(char &ch) { for (ch = getchar(); isspace(ch); ch = getchar()); }
	template <typename __Tp1, typename ...__Tp2> void read(__Tp1 &x, __Tp2 &... y) { read(x), read(y...); }
	template <typename __Tp> void write(__Tp x) {
	    if (x < 0) putchar('-'), x = -x;
	    if (x > 9) write(x / 10);
	    putchar(x % 10 + 48);
	}
	void write(char ch) { putchar(ch); }
	// void write(modint x) { write(x.val()); }
	void write(const char *s) { for (; *s; ++s) putchar(*s); }
	template <typename __Tp1, typename ...__Tp2> void write(__Tp1 x, __Tp2 ... y) { write(x), write(y...); }
	
	
	const int si = (1 << 12) + 1;
	
	int n, m;
	i64 t[si][si][4]; // 内存连续
	i64 ret[4] = {0};
	
	inline int lowbit(int x) {
		return x & -x;
	}
	void add(int x, int y, int k) {
		int xx = x, yy = y;
		while(x <= n) {
			y = yy;
			while(y <= m) {
				t[x][y][0] += k, t[x][y][1] += k * xx, t[x][y][2] += k * yy, t[x][y][3] += k * xx * yy;
				y += lowbit(y);
			}
			x += lowbit(x);
		}
	}
	i64 query(int x, int y) {
		int xx = x, yy = y;
		
		ret[0] = ret[1] = ret[2] = ret[3] = 0;
		while(x) {
			y = yy;
			while(y) {
				ret[0] += t[x][y][0];
				ret[1] += t[x][y][1];
				ret[2] += t[x][y][2];
				ret[3] += t[x][y][3];
				y -= lowbit(y);
			}
			x -= lowbit(x);
		}
		return ((xx + 1) * (yy + 1) * ret[0]) - ((yy + 1) * ret[1]) - ((xx + 1) * ret[2]) + ret[3];
	}
	
	int main() {	
	
		// cin.tie(0) -> sync_with_stdio(false);
		// cin.exceptions(cin.failbit | cin.badbit);
	
		read(n, m);
	
		int opt; 
		int a, b, c, d, k;
	
		while(~scanf("%d", &opt)) {
			if(opt == 1) {
				read(a, b, c, d, k);
				add(a, b, k), add(a, d + 1, -k), add(c + 1, b, -k), add(c + 1, d + 1, k);
			}
			else {
				read(a, b, c, d);
				write( query(c, d) - query(a - 1, d) - query(c, b - 1) + query(a - 1, b - 1) , endl);
			}
		}
	
		return 0;
	}
	
	```

感觉区间修区间查，矩阵修矩阵查都运用了某种比较经典的套路，似乎可以记下来以后用？

**大概就是在求和/求乘积或者干什么的时候，分离包含多变量的项，把每个变量都提出来单独维护，让变量和变量之间相互独立，不同时维护多个变量导致思路过于复杂。**


### 区间最值

显然最大最小值没有区间可减性，所以不能用前缀和减。

然后虽然可以求最值，但是实际上挺麻烦而且不是很通用。

静态最值不如直接写 st 表，动态最值就算只支持单点修也挺麻烦。

个人觉得没有必要，直接 segment tree 就行了。

不过如果只是问前缀最值+单点修改那肯定就写树状数组，实现过于简单懒得写，可以看看这道题里写的：[Optimal Partition](../../../../rec/atcf-for-2022-04/#cf1668dcf1667b-optimal-partition) 

## 应用

### 求逆序对

先离散化，在值域上建立树状数组（就是用树状数组维护桶）。

扫描 $n \to 1$，对于 $a(i)$，它能和 $a(i + 1) \sim a(n)$ 构成的逆序对个数就是当前树状数组上 $[1,a(i) - 1]$ 的前缀和。

累加答案之后，执行 $\text{add}(a(i), 1)$ 即可。

复杂度 $O(n \log n)$，比 Merge Sort 好写多了。

```cpp
for(int i = n; i >= 1; --i) {
	ans += query(a[i] - 1);
	add(a[i], 1);
}
cout << ans << endl;
```

### 三元逆序对

求满足 $i < j < k, a(i) > a(j) > a(k)$ 的 $(i,j,k)$ 的个数。

和二元逆序对本质一样，离散化之后维护两个树状数组，一个维护逆序对，一个维护顺序对，最后对于每个位置乘法原理一下即可。