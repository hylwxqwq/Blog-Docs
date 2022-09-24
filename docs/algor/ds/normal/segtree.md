# 线段树

## 泛化

线段树是一种常用的维护区间信息的数据结构。

它要求所维护的信息具有区间可加性（容易按照区间划分，合并）

比如 $\sum, \prod, \min, \max$ 这些信息。

??? tip "更严谨的定义"
	线段树维护的信息应当是一个满足幺半群性质的信息。

	一个幺半群 $M = (S, \oplus, e)$ 满足这些性质：

	（$\oplus$ 是定义在集合 $S$ 上的二元运算）
	
	1. $\oplus$ 关于 $S$ 封闭。
	2. $\oplus$ 存在结合律即 $\forall a, b, c \in S, (a \oplus b) \oplus c = a\oplus (b \oplus c)$。
	3. $\oplus$ 存在幺元，即 $\exists e \in S, \forall x \in S, (e \oplus x = x) \lor (x \oplus e = x)$。

一般支持单点和区间的信息修改，区间的信息查询。

## 普通线段树

线段树基于分治思想，它先将序列从中间一分为二，

然后对于产生的这两段区间，继续进行从中间一分为二的过程。

直到最后分出来的这一段区间长度等于 $1$。

直观的来看，结构大概长这样：

[![cSn8qf.png](https://z3.ax1x.com/2021/03/27/cSn8qf.png)](https://imgtu.com/i/cSn8qf)

可以发现，每一个节点都维护了序列上的一段区间 $[l, r]$。

并且对于一个维护 $[L, R]$ 的节点，它的左儿子维护 $[L,mid]$，右儿子维护 $(mid, R]$。

其中 $mid = \lfloor \dfrac{L + R}{2}\rfloor$。

去掉最后一层节点之后的线段树是一棵满二叉树，所以可以利用二倍标记法来确定一个节点的左右儿子编号。

形式上来说，对于非叶子节点 $p$，其左右儿子节点分别是 $p \times 2, p\times 2 + 1$。

但是像上图那样完美的结构只会在序列长度等于 $2^k$ 时出现，很多时候最后一层是填不满的，维护区间长度为 $1$ 的节点可能会跑到倒数第二层去。

比如这样（图中有些标注的区间是闭区间，有些是开区间，但无伤大雅）：

[![WyI5q0.png](https://z3.ax1x.com/2021/07/24/WyI5q0.png)](https://imgtu.com/i/WyI5q0)

可以发现，最后一层的那个节点的编号应当为 $31$，当 $n$ 增大时，这个数字会越来越接近 $4 \times n$。

所以开 $4$ 倍空间是必要操作，严谨证明可以自查。

为了方便处理，让每一个节点维护以下的信息即可：

+ 它维护的区间 $l, r$。
+ 它维护的信息的值 $dat$。

一般情况下使用结构体维护，方便模块化的编写和调试。

### 基本操作

#### 上传信息

这个操作用于把儿子节点的信息上传到父亲节点。

因为一般的线段树都是自底向上更新，所以需要这样的一个步骤。

就是直接把左右儿子的信息拉上来到父亲节点。

根据维护信息的不同做一点修改就行。

这个一般取决于询问问的是什么，如果问和就是加起来，如果问最值就是取最值。

??? note "Code"
	```cpp
	inline void pushup(int p) {
		t[p].dat_sum = t[p << 1].dat_sum + t[p << 1 | 1].dat_sum; // sum
		t[p].dat_min = min(t[p << 1].dat_min, t[p << 1 | 1].dat_min); // min 
	}
	```

可以发现这个操作就是把 lson 的信息和 rson 的信息合并起来，印证了线段树维护的信息应当满足结合律。

#### 建树

这个操作用于确定线段树的结构，每个节点维护的区间，并将线段树初始化为最初状态。

从根节点开始，左右儿子分别递归下去，同步记录两个值 $l, r$，表示当前节点应该维护的区间。

当递归到一个节点 $p$ 的时候，令 $\text{t[p].l} \gets l, \text{t[p].r} \gets r$ 即可。

当递归到叶子节点的时候，令这个节点维护的信息为 $a_l$，然后返回，不断上传信息即可。

```cpp
void build(int p, int l, int r) {
	t[p].l = l, t[p].r = r;
	if(l == r) {
		t[p].dat_sum = a[l];
		t[p].dat_min = a[l];
		return;
	}
	int mid = (l + r) >> 1;
	build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
	pushup(p); return;
}
```

#### 单点修改

这个操作用于修改某一个位置 $x$ 的值为 $v$。

可以类比建树的过程。

我们一直递归下去，找到维护这个位置 $x$ 的节点 $p$，令 $\text{t[p].dat} \gets v$，然后不断上传即可。

找 $x$ 只需要比较 $x$ 和 $mid$ 的大小。

如果 $x \le mid$，说明他在左子树，反之在右子树，递归下去即可。 

??? note "Code"
	```cpp
	void modify(int p, int x, int v) {
		if(l == r) {
			t[p].dat_sum = v;
			t[p].dat_min = v;
			return;  
		}
		int mid = (t[p].l + t[p].r) >> 1;
		if(x <= mid)
			modify(p << 1, x, v);
		else 
			modify(p << 1 | 1, x, v);
		pushup(p); return;
	}
	```

#### 区间查询

这个操作用于查询一个区间 $[ql, qr]$ 的信息，比如区间和或者区间最值。

还是从根节点递归下去，设 $nl = \text{t[p].l}, nr = \text{t[p].r}$。

然后如果当前节点 $p$ 满足 $[nl, nr] \subset [ql, qr]$。

那么就不用递归下去了，直接返回当前节点的信息即可（这是一个有用的小剪枝）。

否则分割递归下去，递归回来之后把左右子树的答案分别合并，然后返回这个值即可。

??? info "一张图理解下这个过程"
	[![vfT5Bn.png](https://s1.ax1x.com/2022/08/29/vfT5Bn.png)](https://imgse.com/i/vfT5Bn)
	
	节点标错了，凑合着看。

	解释：

	+ 先查询 $[2,6]$，访问到节点 $1$，发现不满足 $[nl, nr] \subset [ql, qr]$，于是向下递归，因为 $2(ql) \le 4(mid)$，所以左儿子 	$2$ 递归下去，因为 $6(qr) > 4(mid)$，所以右儿子 $4$ 号节点也要递归下去。

	+ 对于 $2$ 号节点，发现仍旧不满足 $[nl, nr] \subset [ql, qr]$，所以继续向下递归 $5,6$ 号节点。
	
	+ 对于 $5$ 号节点，发现 $2(ql) \le 1(mid)$ 不成立，不递归，发现 $6(qr) > 1(mid)$ 成立，所以递归 $10$，发现 $10$ 号节点满足了 $[nl, nr] \subset [ql, qr]$，直接返回。
	
	+ 对于 $6$ 号节点，发现 $[nl, nr] \subset [ql, qr]$，直接返回。
	
	+ 对于 $4$ 号节点，发现只满足 $2(ql) \le 6(mid)$，所以只递归 $7$ 号节点，然后发现 $7$ 满足 $[nl, nr] \subset [ql, qr]$，直接返回。
	
	+ 然后查询就做完了。

??? tip "这个小剪枝的复杂度证明"

	暂时咕咕咕掉了。

??? note "Code"
	```cpp
	i64 query_sum(int p, int l, int r) {
		i64 res = 0;
		if(l <= t[p].l && t[p].r <= r) 
			return t[p].dat_sum;
		int mid = (t[p].l + t[p].r) >> 1;
		if(l <= mid) 
			res += query_sum(p << 1, l, r);
		if(r > mid)
			res += query_sum(p << 1 | 1, l, r);
		return res; 
	}
	
	i64 query_min(int p, int l, int r) {
		i64 res = inf;
		if(l <= t[p].l && t[p].r <= r) 
			return t[p].dat_min;
		int mid = (t[p].l + t[p].r) >> 1;
		if(l <= mid) 
			res = min(res, query_sum(p << 1, l, r));
		if(r > mid)
			res = min(res, query_sum(p << 1 | 1, l, r));
		return res; 
	}
	```

答案有可能要从两边一起取过来，所以也能证明运算需要满足结合律。

### 延迟标记

考虑如果区间加区间和怎么办。

显然如果直接暴力递归到区间里的每个数对应的叶子节点然后向上更新，复杂度显然不能接受，单次操作就是 $O(n \log n)$ 的。

可以发现，有的时候我们并不需要递归到叶子节点，类似区间查询的那个小剪枝，我们如果递归到一个被询问区间完整包含的节点，直接在上面打一个“标记”，表示我摆烂了，不往下更新了，先给儿子节点欠着这个更新，等以后需要用到的时候再把标记 $O(1)$ 下放给儿子。

其本质是，我如果要对于一个节点进行更新，我首先要用它父亲的 tag 更新它的权值再考虑对他进行更新。

先把它父亲节点的 tag 全部 pushdown 下来（利用结合律），再把新的 tag （它对于它的儿子的）用结合律也打到它身上，。

这体现的就是一个 “时间上的结合律”，我是先有了父亲节点欠的更新，再有了现在新来的更新，我的 tag 也是，先有了父亲欠子树下放到我这里的 tag，再有我现在欠我的儿子（子树）的 tag。

类似上面区间询问的复杂度证明，可以知道这个操作是单次 $O(n \log n)$ 的。

??? note "Code"
	```cpp
	int len(int p) { return t[p].r - t[p].l + 1; }
	void pushdown(int p) {
		if(t[p].add) {
			t[p << 1].dat += (t[p].add * len(p << 1));
			t[p << 1 | 1].dat += (t[p].add * len(p << 1 | 1));
			t[p << 1].minv += t[p].add, t[p << 1 | 1].minv += t[p].add; // 水位线原理
			t[p << 1].add += t[p].add, t[p << 1 | 1].add += t[p].add;
			t[p].add = 0; // 设置为 Z 关于加法的幺元 0.
		}
	}
	```

???+ example "Luogu3372 线段树1"

	> 区间加区间求和，
	> 
	> $1\le n \le 10^5, 1\le a_i \le 2^{63} - 1$。

	就是线段树的板子题，写一个有 lazytag 的线段树即可。

	注意开 `long long`。

	??? note "Code"
		```cpp
		
		#include <cstdio>
		#include <cstring>
		#include <iostream>
		#include <algorithm>
		
		using namespace std;
		using i64 = long long;
		
		const int si = 1e5 + 10;
		
		int n, m;
		i64 a[si];
		
		class Segment_Tree {
		    private : 
		        struct Node {
		            int l, r;
		            i64 dat, tag;
		        }t[si << 2];
		        inline void pushup(int p) {
		            t[p].dat = t[p << 1].dat + t[p << 1 | 1].dat; 
		        }
		        inline void pushdown(int p) {
		            if(!t[p].tag) return;
		            t[p << 1].dat += 1ll * t[p].tag * (t[p << 1].r - t[p << 1].l + 1);
		            t[p << 1 | 1].dat += 1ll * t[p].tag * (t[p << 1 | 1].r - t[p << 1 | 1].l + 1);
		            t[p << 1].tag += t[p].tag, t[p << 1 | 1].tag += t[p].tag, t[p].tag = 0;
		        }
		    public :
		        void build(int p, int l, int r) {
		            t[p].l = l, t[p].r = r, t[p].tag = 0;
		            if(l == r) {
		                t[p].dat = a[l];
		                return;
		            }
		            int mid = (l + r) >> 1;
		            build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
		            pushup(p); return;
		        }
		        void update(int p, int l, int r, int v) {
		            if(l <= t[p].l && t[p].r <= r) {
		                t[p].dat += v * (t[p].r - t[p].l + 1);
		                t[p].tag += v; return;
		            }
		            pushdown(p); // 没到可以直接返回的时候，马上要递归子树了，也要 pushdown。
		            int mid = (t[p].l + t[p].r) >> 1;
		            if(l <= mid) 
		                update(p << 1, l, r, v);
		            if(r > mid) 
		                update(p << 1 | 1, l, r, v);
		            pushup(p); return;
		        } 
		        i64 query(int p, int l, int r) {
		            i64 res = 0;
		            if(l <= t[p].l && t[p].r <= r)
		                return t[p].dat;
		            pushdown(p); // 查询要查值，需要子树信息，必然要 pushdown。
		            int mid = (t[p].l + t[p].r) >> 1;
		            if(l <= mid) 
		                res += query(p << 1, l, r);
		            if(r > mid)
		                res += query(p << 1 | 1, l, r);
		            return res;
		        }
		};
		
		Segment_Tree tr;
		
		int main() {
		    
		    cin.tie(0) -> sync_with_stdio(false);
		    cin.exceptions(cin.failbit | cin.badbit);
		    
		    cin >> n >> m;
		    for(int i = 1; i <= n; ++i)
		        cin >> a[i];
		    tr.build(1, 1, n);
		    for(int i = 1; i <= m; ++i) {
		        int opt, l, r;
		        i64 v;
		        cin >> opt;
		        if(opt == 1) {
		            cin >> l >> r >> v;
		            tr.update(1, l, r, v);
		        }
		        else {
		            cin >> l >> r;
		            cout << tr.query(1, l, r) << endl;
		        }
		    }
		    
		    return 0;
		}
		```

这只是一个 tag 的情况，还可能有多个 tag，思考一下怎么弄。

一定要注意到一个事情，线段树本质上是在维护一个满足幺半群性质的信息，

如果我们想要打多个 tag，这些 tag 首先就必须在时间轴上满足结合律（可以合并），比如我们先打了一个运算 $\oplus$ 的标记 $x_0$ **然后** 打了一个 	$\otimes$ 的标记 $y_0$，当前节点已有的 tag 状态记录为 $(x_0, y_0)$。

然后我们又进行了一次 $\oplus$ 的标记 $x_1$，**再** 进行了一次 $\otimes$ 的操作 $y_1$，记录为 $(x_1,y_1)$。

那么我们就需要知道，怎么样合并 $(x_0, y_0), (x_1, y_1)$，也就是需要知道 $(x_0, y_0) \circ (x_1, y_1) = (x_2, y_2)$ 中的 $x_2, y_2$ 分别是什么。

$\circ$ 表示复合（合并）这两个 Tag，这里的下标表示的是时间轴上的位置。

显然两个 tag 没有什么关系的时候可以直接只考虑 $(x_0, y_0)$，这个比较 ez，不过如果两个 tag 之间相互有影响，就需要考虑 $(x_0, y_0)$ 和 $(y_0, x_0)$ 到底应该选哪一个（要判定先做什么运算），选择依据是合并前后的标记的顺序能否统一。

举个例子，区间加区间乘区间求和。

和线段树一差不多，只不过多了一个需要维护的运算：乘法。

所以这里应当考虑的是怎么把两个各自封闭又相互有影响的信息复合起来维护。

考虑如果只有区间加区间求和，tag 记录的就是儿子里面所有数要加多少。

如果在来一个区间乘怎么办，就考虑对这两个运算复合。

比如我先乘 $x$ 然后加上一个 $y$，儿子节点里的每个数 $value$ 就应当变成 $value \times x + y$。

然后如果继续复合就是 $(((value \times x) + y) \times z) + a\dots$ 这样（先加后乘也已经被包含在情况里面了）。

因为我们没法知道具体顺序，有可能是 $++\times+\times\times+++$这种的，你每次转换都需要新开一个 tag 记录。

我们肯定不想让每个节点的 tag 都有巨大多个，我们希望就只有两个，也就意味着我们只希望乘除法变换一次，即把乘都丢到一起，加都丢到一起。

所以观察一下这个式子形式，发现可以乘法分配律，我们在区间乘法打标记的时候给先加上的 add 乘上后面的 mul，然后 pushdown 的时候 add 就能单独拉出来加了。

下放标记的时候 add 需要先乘上父亲节点的 mul，然后再加上父亲节点的 add，因为父亲节点的 add 在区间乘打标记的时候已经乘过 mul 了，直接提出来加就行了。

更形式化的说，我们现在记录了两个 tag，一个加法，一个乘法，我们记为 $(add_i, mul_i)$。

假设当前已经存在对于当前节点作用的一个标记 $(add_0, mul_0)$，现在新来了一个标记 $(add_1, mul_1)$，我们要做的就是求出 $(add_0, mul_0)\circ (add_1, mul_1)$，并且要规定好是先 add 还是先 mul。

可以发现这里是在对着整体打标记，所以我们直接考虑当前区间维护的值在加入新的标记之后“怎么变化”了，并考虑怎么合并原有标记和新加入的标记。	

先假设先加后乘，显然有：

$$
\begin{aligned}
dat &= ((dat + add_0) \times mul_0 + add_1) \times mul_1\\
&= (dat \times mul_0 + add_0 \times mul_0 + add_1) \times mul_1\\
&= (dat \times mul_0 \times mul_1) + (add_0 \times mul_0 \times mul_1) + (add_1 \times mul_1)
\end{aligned}
$$

好像没法把合并后的标记也统一顺序（不能写成 $(dat + a)\times b$ 的形式）。

看看 $(mul_0, add_0)$ 的顺序如何。

$$
\begin{aligned}
dat &= (dat \times mul_0 + add_0) \times mul_1 + add_1\\
&= (dat \times mul_0 \times mul_1) + (add_0 \times mul_1 + add_1)\\
&= dat \times (mul_0 \times mul_1) + (add_0 \times mul_1 + add_1)
\end{aligned}
$$

然后可以很愉快的发现能把合并后的标记的顺序和已有的标记的顺序统一。

所以可以得到：$(mul_0, add_0) \circ (mul_1, add_1) = (mul_0 \times mul_1, add_0 \times mul_1 + add_1)$。

其本质就是，对于乘法标记我直接乘了下放，打乘法标记的时候顺便给加法乘一下，对于加法标记我直接先乘上父亲节点给的 mul，然后再加上父亲节点给的 add（这个再之前已经乘过了，乘法分配律）。

很显然这个对于四种加法乘法的组合顺序都满足（检查这个 tag 对于线段树信息的作用能否处理）。

加法之后加法就不考虑 mul 的处理，显然成立。乘法之后乘法同理。

先乘后加就是我们规定的形式，于是看看先加后乘的情况能不能满足：可以考虑最早存在的标记是 $(1, 0)$（幺元），然后再打了一个标记 $(1, add)$，再打了一个标记 $(mul, 0)$，展开式子之后显然成立。

所以这种 tag 方式是可行的。

???+ example "Luogu3373 线段树2"

	> 区间加区间乘，询问区间和对 $p$ 取模。
	> 
	> $1\le n, q\le 10^5$。

	就是刚刚已经说了的问题，这里直接给出实现。

	??? note "Code"
		```cpp
		
		#include <cstdio>
		#include <cstring>
		#include <iostream>
		#include <algorithm>
		
		#define meow(x) cerr << #x << " = " << x
		
		using namespace std;
		using i64 = long long;
		
		const int si = 2e5 + 10;
		// const i64 mod = 998244353ll;
		
		int mod;
		int n, m;
		i64 a[si];
		
		class Segment_Tree {
			private : 
				struct Node {
					int l, r;
					i64 dat, add, mul;
				}t[si << 2];
				inline void pushup(int p) {
					t[p].dat = (t[p << 1].dat + t[p << 1 | 1].dat) % mod;
				}
				inline void pushdown(int p) {
					if(!t[p].add && t[p].mul == 1) return;
					t[p << 1].dat = (t[p << 1].dat * t[p].mul + t[p].add * (t[p << 1].r - t[p << 1].l + 1)) % mod	;
					t[p << 1 | 1].dat = (t[p << 1 | 1].dat * t[p].mul + t[p].add * (t[p << 1 | 1].r - t[p << 1 | 1].l + 1)) % mod;

					t[p << 1].mul = (t[p << 1].mul * t[p].mul) % mod;
					t[p << 1 | 1].mul = (t[p << 1 | 1].mul * t[p].mul) % mod;

					t[p << 1].add = (t[p << 1].add * t[p].mul + t[p].add) % mod;
					t[p << 1 | 1].add = (t[p << 1 | 1].add * t[p].mul + t[p].add) % mod;
		
					t[p].add = 0, t[p].mul = 1;
				}
			public : 
				void build(int p, int l, int r) {
					t[p].l = l, t[p].r = r, t[p].mul = 1ll, t[p].add = 0ll;
					if(l == r) {
						t[p].dat = a[l] % mod;
						return;
					}
					int mid = (l + r) >> 1;
					build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
					pushup(p); return;
				}
				void update_add(int p, int l, int r, i64 v) {
					if(l <= t[p].l && t[p].r <= r) {
						t[p].add = (t[p].add + v) % mod;
						t[p].dat = (t[p].dat + v * (t[p].r - t[p].l + 1)) % mod;
						return;
					}
					pushdown(p);
					int mid = (t[p].l + t[p].r) >> 1;
					if(l <= mid)
						update_add(p << 1, l, r, v);
					if(r > mid)
						update_add(p << 1 | 1, l, r, v); 
					pushup(p); return;
				}
				void update_mul(int p, int l, int r, i64 v) {
					if(l <= t[p].l && t[p].r <= r) {
						t[p].add = (t[p].add * v) % mod;
						t[p].mul = (t[p].mul * v) % mod;
						t[p].dat = (t[p].dat * v) % mod;
						return;
					}
					pushdown(p);
					int mid = (t[p].l + t[p].r) >> 1;
					if(l <= mid) 
						update_mul(p << 1, l, r, v);
					if(r > mid)
						update_mul(p << 1 | 1, l, r, v);
					pushup(p); return;
				}
				i64 query(int p, int l, int r) {
					i64 res = 0ll;
					if(l <= t[p].l && t[p].r <= r)
						return t[p].dat % mod;
					pushdown(p);
					int mid = (t[p].l + t[p].r) >> 1;
					if(l <= mid) 
						res = (res + query(p << 1, l, r)) % mod;
					if(r > mid)
						res = (res + query(p << 1 | 1, l, r)) % mod;
					return res;
				}
		};
		
		Segment_Tree tr;
		// 不要到主函数里定义，容易爆栈。
		
		int main() {	
		
			cin.tie(0) -> sync_with_stdio(false);
			cin.exceptions(cin.failbit | cin.badbit);
		
			cin >> n >> m >> mod;
			for(int i = 1; i <= n; ++i)
				cin >> a[i];
		
			tr.build(1, 1, n);
			for(int i = 1; i <= m; ++i) {
				int opt, l, r;
				cin >> opt;
				if(opt == 2) {
					i64 v;
					cin >> l >> r >> v;
					tr.update_add(1, l, r, v);
				}
				if(opt == 1) {
					i64 v;
					cin >> l >> r >> v;
					tr.update_mul(1, l, r, v);
				}
				if(opt == 3) {
					cin >> l >> r;
					cout << tr.query(1, l, r) << endl;
				}
			}
		
			return 0;
		}
		```

总结一下，带 lazy 的线段树题一般就这几步：

1. 如果给一个区间整体打上标记，能否确定区间维护的值怎么变化
2. 如果给一个区间整体打上标记，能否确定 tag 怎么合并。
3. 形式化的说，懒标记线段树维护每个位置的信息幺半群 $(I, +)$ 和对信息的修改幺半群 $(D, *)$，要求 $D$ 对 $I$ 的作用满足分配率。

ref：

- [zhqwq 的博客 - 我 根 本 不 会 线 段 树|线段树再学习笔记](https://zhqwqsblog.blog.luogu.org/wo-gen-ben-fou-kuai-xian-duan-shu-xian-duan-shu-zai-xue-xi-bi-ji)
- [Github - ATcoder Library Documents](https://atcoder.github.io/ac-library/production/document_en/appendix.html)
- 粉兔的友情讲解，以及那句 「？你根本不是萌新」

## 动态开点

正常的线段树一般都是用二倍标记法去标记儿子的序号。

这样至少需要四倍空间才不会出现 RE。

但在某些特殊的情况下（比如后面的权值线段树），你需要维护下标的范围可能非常大，如果真的要全部建树建出来，空间就爆了。

此时可以利用一个类似线段树懒标记的思想，如果一个节点需要使用，那么我们才建立这个节点，反之就不用。

从实现上来讲，就是初始的时候只建立一个根节点代表整个区间。

当递归访问到的节点为空时，才新建一个节点，并给他一个编号。

此处编号的方式不同于原来的二倍标记法，只需要记录每一个节点的左右儿子的编号就可以了。

正常写法是把一个节点的区间信息直接附加到节点上。

在这里就直接作为函数的参数传递了。

一般空间要开大一点，比如 $\times 60$ 这种。

最好是先估算一下空间然后再开。

[![fBPrLR.png](https://z3.ax1x.com/2021/08/12/fBPrLR.png)](https://imgtu.com/i/fBPrLR)

一份区间加区间乘区间求和线段树的动态开点写法：

??? note "Code"
	```cpp
	#define ls t[p].lson
	#define rs t[p].rson
	class Segment_Tree {
		private : 
			struct Node {
				int lson, rson;
				i64 dat, add, mul;
				// 必须要新建节点时再初始化。
				// 否则编译器会提前处理这些赋值
				// 然后你的 binary 就会巨大无比
				// 是会出事的。
			}t[si * 60];
			// 空间一定要开够。
			int cnt_node = 0;
			inline int Newnode() {
				cnt_node ++;
				t[cnt_node].lson = t[cnt_node].rson = 0;
				t[cnt_node].dat = 0ll, t[cnt_node].add = 0ll, t[cnt_node].mul = 1ll;
				return cnt_node;
			}
			inline void pushup(int p) {
				t[p].dat = t[ls].dat + t[rs].dat;
			}
			// pushdown 也是需要传参的了。
			inline void pushdown(int p, int l, int r) {
				if(!t[p].add && t[p].mul == 1) 
					return;
				
				if(!ls) ls = Newnode();
				if(!rs) rs = Newnode();
				// 记得在这里也要动态开点。
	
				int mid = (l + r) >> 1;
	
				t[ls].dat = (t[ls].dat * t[p].mul + t[p].add * (mid - l + 1)) % mod;
				t[rs].dat = (t[rs].dat * t[p].mul + t[p].add * (r - mid)) % mod;
				t[ls].add = (t[ls].add * t[p].mul + t[p].add) % mod;
				t[rs].add = (t[rs].add * t[p].mul + t[p].add) % mod;
				t[ls].mul = (t[ls].mul * t[p].mul) % mod;
				t[rs].mul = (t[rs].mul * t[p].mul) % mod;
	
				t[p].add = 0ll, t[p].mul = 1ll;
			}
		public : 
			void update_add(int &p, int l, int r, int ql, int qr, i64 v) {
				if(l > r) return; // 递归边界。
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
				// 不存在直接返回 0 即可。
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
	```

## 权值线段树

属于一种变种的线段树。

维护的不是序列而是序列的值域，可以把它看作一个动态的桶（叶子节点就是桶）。

也就是说，它维护的一段 $[l,r]$ 区间的信息，是序列中权值在 $[l,r]$ 中的数的个数。

用它可以实现一些平衡树的功能。

因为一般维护的值域很大（$10^9$ 这种），所以可以离散化或者动态开点，个人推荐后一种。

### 正常操作

和正常的动态开点没区别。

??? note "Code"
	```cpp
	using i64 = long long;
	
	const int si = 1e5 + 10;
	
	int n, m;
	// m 是值域。
	#define ls t[p].lson
	#define rs t[p].rson
	struct Node {
		i64 dat;
		int lson, rson;
		// 这里不要提前赋值，newnode 时再赋值。
	}t[si * 60];
	int tot = 0, root = 0;
	inline int Newnode() {
		tot++;
		t[tot].dat = 0ll;
		t[tot].lson = 0, t[tot].rson = 0;	
		return tot;
	}
	inline void pushup(int p) {
		t[p].dat = t[ls].dat + t[rs].dat;
	}
	void modify(int &p, int l, int r, int x, int v) {
		if(l > r) return;
		if(!p) p = Newnode();
		if(l == r) {
			t[p].dat += v;
			return;
		}
		int mid = (l + r) >> 1;
		if(x <= mid) 
			modify(ls, l, mid, x, v);
		else 
			modify(rs, mid + 1, r, x, v);
		pushup(p); return;
	}
	i64 query(int p, int l, int r, int ql, int qr) {
		if(l > r) return 0ll;
		if(!p) return 0ll;
		if(ql <= l && r <= qr) 
			return t[p].dat;
		int mid = (l + r) >> 1;
		if(ql <= mid) 
			modify(ls, l, mid, ql, qr);
		if(qr > mid)
			modify(rs, mid + 1, r, ql, qr);
		pushup(p); return;
	}
	```

### 插入/删除

这个直接在对应位置单点加减一就行。

??? note "Code"
	```cpp
	void insert(int v) {
		modify(root, 1, m, v, 1);
	}
	void remove(int v) {
		modify(root, 1, m, v, -1);
	}
	```

### 动态全局第 k 大（线段树二分）

这里要用到线段树二分的思想。

假设要查询全局第 $k$ 大。

当前递归到的节点的左右儿子权值和分别为 $ls, rs$。

如果 $k \le ls$，那么递归查左子树。

反之令 $k \gets k - ls$，然后在右子树里查第 $k$ 大。

这个过程实际上就是在二分第 $k$ 大应当是多少。

可以把整颗线段树“压平”了来看。

??? note "Code"
	```cpp
	i64 kth(int p, int l, int r, int k) {
		if(l == r) 
			return t[p].dat;
		int mid = (l + r) >> 1;
		if(t[ls].dat >= k) 
			kth(ls, l, mid, k);
		else 
			kth(rs, mid + 1, r, k - t[ls].dat);
	}
	```

### 查询排名

查询某一个数 $x$ 的排名。

直接查线段树上 $[1,x)$ 的和然后加一即可。

??? note "Code"
	```cpp
	i64 rnk(int v) {
		return query(root, 1, m, 1, v - 1) + 1;
	}
	```

### 查询前驱后继

前驱直接先查询 $x$ 的排名 $rk$，然后查第 $rk - 1$ 大即可。  

后继类似。

??? note "Code"
	```cpp
	i64 pre(int v) {
		return kth(rnk(v) - 1);
	}
	i64 nex(int v) {
		return kth(t[root].dat - query(root, 1, m, v + 1, m) + 1);
	}
	```

## 线段树合并

有的时候需要对同一个值域在多颗不同的动态开点线段树上进行修改。

然后计算答案时需要综合它们的答案。

也就是我们需要两两合并这些线段树。

因为维护的值域相同，那么显然它们对于值域的子区间的划分也是一样。

所以我们可以同时从两颗线段树的树根开始，用 $p,q$ 两个指针**同步**向下递归。

可能会出现这四种情况：

1. $p$ 没有建立节点，而 $q$ 建立有节点。
2. $q$ 没有建立节点，而 $p$ 建立有节点。
3. $p,q$ 都没有建立这个节点。
4. $p,q$ 都有建立这个节点。


对于 $1,2$ ，直接把非空节点作为合并后的节点。

对于 $3$ ，既然你俩都没有，那合并后也没必要要这个点，也就是合并后仍旧没有这个节点。

对于 $4$ ，既然你们都有，那就合并你们的信息（类似于普通线段树的 Pushup），然后随便找一个节点作为合并之后的节点（一般是 $p$）

[![fBiU0I.png](https://z3.ax1x.com/2021/08/12/fBiU0I.png)](https://imgtu.com/i/fBiU0I)

??? note "Code"
	```cpp
	int merge(int p, int q, int l, int r) {
	    if(!p) return q; 
	    if(!q) return p;
	    if(l == r){
	        t[p].mx += t[q].mx;
	        return p;
	    }
	    int mid = (l + r) >> 1;
	    t[p].ls = merge(t[p].ls, t[q].ls, l, mid);
	    t[p].rs = merge(t[p].rs, t[q].rs, mid + 1, r);
	    pushup(p); return p;
	}
	```

还有一种节省空间的写法，叫节点回收。

具体来说，把合并后无用的节点的编号扔进一个栈里面。

合并需要新节点的时候再从栈里拿出来用。

## 扫描线

> 咕