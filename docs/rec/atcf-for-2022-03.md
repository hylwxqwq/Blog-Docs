## 三月好题改错

### CF1646E Power Board

`Mar/05/2022`

> 给你一个 $n\times m$ 的矩阵，且 $a_{i,j}=i^j$。求这个矩阵当中有多少不同的数。
>
> 1e6。

暴力明显爆炸。

考虑容斥之类的做法来去重。

发现两个不同行 $(i,j)$ 会生成重复的数，当且仅当 $\exist d,a,b \ |\ i=d^a,j=d^b$。

比如 $2,4,8$ 这种。

列一个表：

$$\begin{bmatrix}2^1&2^2&2^3&\dots&2^m \\4^1&4^2&4^3&\dots&4^m\\8^1&8^2&8^3&\dots&8^m\end{bmatrix}$$

化成 $2$ 的次幂形式： 

$$\begin{bmatrix}2^1&2^2&2^3&\dots&2^m \\2^2&2^4&2^6&\dots&2^{2m}\\2^3&2^6&2^9&\dots&2^{3m}\end{bmatrix}$$

考虑指数：

$$\begin{bmatrix}1&2&3&\dots&m \\2&4&6&\dots&{2m}\\3&6&9&\dots&{3m}\end{bmatrix}$$

如果只考虑 $2,4,8,16$ 这种的话，答案就是上面第三个表当中不同的数的个数。

因为我们去掉底数的过程就是求了一个 $\log$ ，所以表里面的数是 $\log$ 级别的。

写一个暴力即可查询。

转化到原题，就是对于每一个不是某一个整数的大于等于 $2$ 的整数次幂的数求一次。

`std::bitset` 即可。 思路来自：<https://codeforces.com/blog/entry/100584?#comment-892898> [@Suri429](https://codeforces.com/profile/Suri429)

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	 
	#define int long long 
	constexpr int si=1e6+10;
	int s=1,n,m;
	int a[si],b[si];
	std::bitset<si*20>c;
	#undef int 
	 
	int main(){
	#define int long long 
		scanf("%lld%lld",&n,&m);
		for(register int i=1;i<=20;++i){
			for(register int j=1;j<=m;++j) c[i*j]=1;
			a[i]=c.count();
		}
		for(register int i=2;i<=n;++i){
			if(b[i]==1) continue;
			int k=1;b[i]=1;
			for(register int j=i*i;j<=n;j=j*i) b[j]=1,k=k+1;
			s=s+a[k];
		} return printf("%lld",s),0;
	#undef int 
	}
	```

```cpp
Tag : 暴力/bitset/思维
```

### CF1647E Madoka and the Sixth-graders

`Mar/22/2022`

> 给 $n$ 张桌子，初始的时候桌子上坐着的学生的编号分别为 $a_1, a_2, \dots , a_n$。
>
> 且 $a$ 是一个 $n$ 的排列，然后给定几组有向关系 $(i,j)$，表示每过一节课，桌子 $i$ 的学生会移动到桌子 $j$。
>
> 如果一个桌子上有超过一个人，留下编号最小的，然后踢出其他的。
>
> 门外的学生编号为 $n+1,n+2,\dots$。
>
> 如果有空的位置，按照位置编号从小到大排序，然后让门外的学生依次进来填上。
>
> 给定最终的状态 $b$ ，求出满足条件的字典序最小的 $a$。
>
> 1e5。
>
> 保证每节课总是有学生被踢出，学生的编号互不相同。

发现题目保证了 $p$ 当中必然有重复元素。

所以每一轮必然会有人被开除，等价于每一轮必然有人进入。

所以教室里的所有数的最大值一定是单调上升的。

可以求出当前经过的轮数 $k =$ `(*max_element(b + 1, b + 1 + n) - n) / (n - set<int>(p + 1, p + 1 + n).size())`

因为类似样例一当中的 “双向边” 很不好处理。

所以当务之急是经过 $k$ 轮之后，原来在某一个点的 stu 会在哪一个点出现。

设这个点为 $dest[i]$

并且先不考虑被开除的情况。

这个如果直接递推是 $\text{O}(nk)$ 的，必然会 TLE。

然后发现这个东西关于 $2$ 的正整数次幂具有**划分性**。

也就是设 $t[i][j]$ 表示 $i$ 跳 $2^j$ 步可以到的位置，那么必然可以得到 $t[i][j] = t[ t[i][j - 1] ][j - 1]$.

更一般的， $t[i][j] = t[ t[ t[ t[i][j-3] ][j-3] ][j-2] ][j-1]$ ，这样可以一直嵌套下去。

那么这个递推就可以直接利用**倍增优化**。

所以把 $k$ 二进制拆分，枚举 $k$ 的每一位，如果当前位是 $1$， 那么给每一个 $dest$ 跳 $2^j$ 步，$j$ 是当前位。

初始化 $to[i][0] = p[i]$。

对于因为 $p[i]$ 有重复的，所以 $dest[i]$ 也必然会有重复的。

假设某一个点 $i$，有三个点（实际上个数任意，此处举例需要） $A, B, C$ 可以经过 $k$ 轮到达它，

那么留在这个地方的（$b[i]$）必然是这三点过来的学生当中编号最小的，

所以最开始（$k = 0$）的时候，$A, B, C$ 当中必然有一个点的学生是 $b[i]$。

为了让字典序最小，考虑让 $A, B, C$ 当中编号最小的一个点初始坐上 $b[i]$，也就是让这个点的 $a = b[i]$。

然后现在就已经填好了一些点。

维护剩下的，编号在 $1 \sim n$ 当中的学生集合 $S$（为什么是 $1 \sim n$ 原因显然）。

然后从前往后扫描 $a$ 当中所有还没有答案的点，对于当中的某一个点 $i$ ，它经过 $k$ 轮去到的地方是 $dest[i]$。

那么它的 $a[i]$ 必然要大于所有 $dest = dest[i]$ 的点当中编号最小的那个点初始的 $a$，也就是 $b[dest[i]]$。

所以在集合 $S$ 当中二分一个大于 $b[dest[i]]$ 的，最小的点填上去就可。

??? note "Code"
	```cpp
	#include <set>
	#include <cmath>
	#include <bitset>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	#include <unordered_set>
	
	using namespace std;
	
	const int si = 1e5 + 10;
	
	int n;
	int a[si], b[si], p[si];
	int t[si][51];
	int dest[si];
	int tmp[si];
	
	int main() {
		cin >> n;
		int mx = -1; bitset<si> vis;
		for(int i = 1; i <= n; ++i) cin >> p[i], vis[p[i]] = true;
		for(int i = 1; i <= n; ++i) cin >> b[i], mx = max(mx, b[i]);
		
		int k = (mx - n) / (n - vis.count()); 
		// int k = (*max_element(b + 1, b + 1 + n) - n) / (n - unordered_set<int>(p + 1, p + 1 + n).size());
	
		int lg = (int)(log(k) / log(2)) + 1;
		
		cerr << k << " " << lg << endl;
		
		for(int i = 1; i <= n; ++i) t[i][0] = p[i];
		for(int j = 1; j <= lg; ++j) {
			for(int i = 1; i <= n; ++i) {
				t[i][j] = t[t[i][j - 1]][j - 1];
			}
		}
		
		for(int i = 1; i <= n; ++i) dest[i] = i;
		
		for(int j = lg; j >= 0; --j) {
			if(k >> j & 1) {
				for(int i = 1; i <= n; ++i) {
					dest[i] = t[dest[i]][j];
					if(t[dest[i]][j] == 0) cerr << "Error on #" << i << ", " << j << endl;
				}
			}
		}
		
		// for(int i = 1; i <= n; ++i) cout << dest[i] << endl;
		
		sort(p + 1, p + 1 + n);
		int m = unique(p + 1, p + 1 + n) - p - 1;
		
		// cout << m << endl;
		// for(int i = 1; i <= m; ++i) cout << p[i] << endl;
		
		memset(a, 0x3f, sizeof a);
		memset(tmp, 0x3f, sizeof tmp);
		set<int>rest;
		for(int i = 1; i <= n; ++i) {
			if(tmp[dest[i]] == 0x3f3f3f3f) tmp[dest[i]] = i;
			rest.insert(i);
		}
		
		// for(int i = 1; i <= n; ++i) cout << tmp[i] << endl;
		
		for(int i = 1; i <= m; ++i) {
			if(tmp[p[i]] == 0x3f3f3f3f) continue;
			a[tmp[p[i]]] = b[p[i]];
			// cerr << b[p[i]] << "$";
			auto pos = rest.find(b[p[i]]);
			if(pos == rest.end()) cerr << "Error";
			rest.erase(pos);
		}
		for(int i = 1; i <= n; ++i) {
			if(a[i] == 0x3f3f3f3f) {
				auto pos = rest.lower_bound(b[dest[i]]);
				// cerr << b[dest[i]] << "#";
				if(pos == rest.end()) cerr << "Error";
				a[i] = *pos, rest.erase(pos); 
			}
		}
		
		for(int i = 1; i <= n; ++i) cout << a[i] <<" ";
		cout << endl;
		return 0;
	}
	```

```cpp
Tag : 倍增/图论
```

### CF1650G Counting Shortcuts

`Mar/24/2022`

> 给定一张无权无向图，问题从某个点 $s$ 到某个点 $t$ 的好路径的条数对 $1e9+7$ 取模。
>
> 一个路径是好的，当且仅当这个路径的长度和 $s,t$ 之间的最短路的长度差不超过 $1$。
>
> $n,m$ 2e5。

首先考虑求出 $s$ 出发到每一个点的最短路。

然后 DP 即可。

方程比较显然，直接看代码即可：

??? note "Code"
	```cpp
			dijkstra(s); 
			// Bfs(s), ReBfs(s);
			vector<pair<int, int> > Dis;
			for(int i = 1; i <= n; ++i) {
				Dis.push_back({dis[i], i});
				dp[i][0] = dp[i][1] = 0;
			} 
			sort(Dis.begin(), Dis.end());
			dp[s][0] = 1, dp[s][1] = 0; 
			for(auto x : Dis) {
				int u = x.second;
				for(int i = head[u]; ~i; i = e[i].Next) {
					int v = e[i].ver;
					if(dis[u] == dis[v] + 1)
						dp[u][0] = (dp[u][0] + dp[v][0]) % mod;
				}
			}
			for(auto x : Dis) {
				int u = x.second;
				for(int i = head[u]; ~i; i = e[i].Next) {
					int v = e[i].ver;
					if(dis[u] == dis[v] + 1) 
						dp[u][1] = (dp[v][1] + dp[u][1]) % mod;
					else if(dis[u] == dis[v])
						dp[u][1] = (dp[v][0] + dp[u][1]) % mod;
				}
			} 
			cout << (dp[t][0] + dp[t][1]) % mod << endl;	
	```

```cpp
Tag : DP/图论/最短路
```

### CF1654E Arithmetic Operations

`Mar/21/2022`

> 给一个数列 $a$ ，每次操作可以任意修改一个位置的一个数，修改后的数可以是正整数，负整数和零。
>
> 问使得 $a$ 成为等差数列的最小的操作次数。
>
> 1e5。

把每个元素看成一个二维平面上的点 $(i,a_i)$。

然后要求的可以转化为求这个平面上最多有多少个点共线，然后用 $n$ 减去这个值即可。

**也就是把等差数列转换成平面直角坐标系上的一条直线。**

但是这个东西好像是根号分治，不会，所以先空着了。

### CF1657F Words On Tree

`Mar/24/2022`

> 给一棵树，给 $q$ 个三元组 $(x_i,y_i,s_i)$，$x_i,y_i$ 是节点编号， $s_i$ 是字符串，长度和树上 $(x_i,y_i)$ 的简单路径长度一致。
>
> 构造一棵树，每个节点上有一个字符，使得它满足这 $q$ 个三元组的约束。
>
> 对于每一个三元组，需要满足，路径 $(x_i,y_i)$ 上的字符串要么是 $s_i$ ，要么是 $reverse(s_i)$。
>
> 9s, 4e5。

考虑暴力 2-SAT。

但是这样每个节点的候选值有 26 个，没法做。

那么把每个三元组这样处理：

把 $s_i$ 放到这个路径上，然后把 $rev(s_i)$ 放到这个路径上。

然后就可以发现每个节点的候选集合大小最大为 $2$。

（对每个节点，对经过它的三元组留下的候选集合求交就可以得到）

然后就可以 2-SAT 了。

但是这样直接实现会非常复杂。

考虑是否存在另外一种更好的写法。

扫描每一个串，然后内层扫描路径上的每一个节点，如果当前节点没有候选集合，给他加上。

然后考虑这样的一个过程：

```cpp
cand[i][0]             cand[i][1] (当前节点的候选集合)
    
    
    
s[j]					s[len - j - 1] (rev(s)[j])	 (当前扫描到的串在这个点的候选集合)
```

如果 $s_j$ 不等于 $cand[i][0]$ ，那么证明选 $s[j]$ 就不能选 $cand[i][0]$ 。

所以连接 $s[j] \to cand[i][1]$ ，然后把对应的逆否命题 $cand[i][0] \to s[len-j-1]$ 链接上。

其它三种情况同理。

这个时候再跑 2-SAT 就可以了。

因为时限 $9s$ 所以随便搞。

??? note "Code"
	```cpp
	
	#include <cmath>
	#include <stack>
	#include <vector>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define c0 qwq
	#define c1 qaq
	
	using namespace std;
	
	const int si =16 * (1e5 + 10);
	
	int n, q;
	
	int head[si], tot = 0;
	struct Edge {
		int ver, Next;
	}e[si << 1];
	inline void add(int u, int v) {
		e[tot] = (Edge){v, head[u]}, head[u] = tot++;
	}
	
	int dep[si], f[si][20];
	int lg;
	void dfs(int u, int fa) {
		dep[u] = dep[fa] + 1, f[u][0] = fa;
		for(int i = 1; i <= lg; ++i) f[u][i] = f[f[u][i-1]][i-1];
		for(int i = head[u]; ~i; i = e[i].Next) {
			int v = e[i].ver;
			if(v == fa) continue;
			dfs(v, u);
		}
	}
	int Lca(int u, int v) {
		if(dep[u] < dep[v]) swap(u, v);
		for(int i = lg; i >= 0; --i) if(dep[f[u][i]] >= dep[v]) u = f[u][i];
		if(u == v) return u;
		for(int i = lg; i >= 0; --i) if(f[u][i] != f[v][i]) u = f[u][i], v = f[v][i];
		return f[u][0];
	}
	
	vector<int> G[si];
	inline void Add(int u, int v) {
		G[u].emplace_back(v);
	}
	
	int tim = 0;
	int dfn[si], low[si];
	stack<int>s;
	bool ins[si];
	int cnt = 0;
	int c[si];
	void tarjan(int u) {
		dfn[u] = low[u] = ++tim;
		s.push(u), ins[u] = true;
		for(int v : G[u]) {
			if(!dfn[v]) tarjan(v), low[u] = min(low[u], low[v]);
			else if(ins[v]) low[u] = min(low[u], dfn[v]); 
		}
		if(dfn[u] == low[u]) {
			int x; ++cnt;
			do {
				x = s.top(), s.pop();
				c[x] = cnt, ins[x] = false;
			} while(u != x);
		}
	}
	
	int m;
	char cand[si][2];
	inline int Node(int u, bool op) {
		if(op) return u;
		else return u + (n + m);
	}
	
	int main() {
		cin >> n >> q, lg = (int)(log(n) / log(2)) + 1;
		memset(head, -1, sizeof head);
		memset(ins, false, sizeof ins);
		for(int i = 1; i < n; ++i) {
			int u, v;
			cin >> u >> v;
			add(u, v), add(v, u);
		} 
		dfs(1, 0);
		int now = 0;
		m = q;
		while(q--) {
			++now;
			int u, v; string s;
			cin >> u >> v, cin >> s;
			int len = (int)s.size(); 
			int lca = Lca(u, v);
			vector<int> path, tmp;
			
			while(u != lca) path.emplace_back(u), u = f[u][0];
			path.emplace_back(lca);
			while(v != lca) tmp.emplace_back(v), v = f[v][0];
			reverse(tmp.begin(), tmp.end());
			for(auto x : tmp) path.emplace_back(x);
			
			// for(auto x : path) cout << x << ' '; cout << endl;
			
			for(int i = 0; i < len; ++i) {
				int x = path[i];
				char c0 = s[i], c1 = s[len - i - 1];
				
				if(!cand[x][0] && !cand[x][1]) cand[x][0] = c0, cand[x][1] = c1;
				
				if(cand[x][0] != c0) {
					Add(Node(now, 0), Node(x + m, 1));
					Add(Node(x + m, 0), Node(now, 1));
				}
				if(cand[x][1] != c0) {
					Add(Node(now, 0), Node(x + m, 0));
					Add(Node(x + m, 1), Node(now, 1));
				}
				if(cand[x][0] != c1) {
					Add(Node(now, 1), Node(x + m, 1));
					Add(Node(x + m, 0), Node(now, 0));
				}
				if(cand[x][1] != c1) {
					Add(Node(now, 1), Node(x + m, 0));
					Add(Node(x + m, 1), Node(now, 0));
				}
			}
		}
		for(int i = 1; i <= 2 * (n + m); ++i) {
			if(!dfn[i]) tarjan(i);
		}
		for(int i = 1; i <= n; ++i) {
			if(!cand[i][0]) cand[i][0] = cand[i][1] = 'a';
		}
		for(int i = 1; i <= n + m; ++i) {
			if(c[Node(i, 0)] == c[Node(i, 1)]) 
				return puts("NO"), 0;
		}
		puts("YES");
		for(int i = 1; i <= n; ++i) {
			putchar(
				cand[i][
					c[Node(i + m, 0)] 
						>
					c[Node(i + m, 1)]
				]
			);
		}
		puts("");
		return 0;
	}
	```

```cpp
Tag : LCA/思维/2-SAT
```

### CF1656D K-good

`Mar/25/2022`

> 一个正整数 $n$ 是 $k$ Good 的，当且仅当 $n$ 可以被分成 $k$ 个不同正整数的和，且这 $k$ 个正整数模 $k$ 意义下相互不同余。
>
> 给定 $n$ （1e18），求出任意一个合法的，$\ge 2$ 的 $k$ ，使得 $n$ 是 $k$ Good 的。

考虑把 $n$ 按照 $1,2,3,4,\dots$ 的方式摊到 $k$ 个地方，然后把剩下的值加上，使得条件成立。

然后可以列出一个线性同余方程：$n \equiv \dfrac{k(k+1)}{2} (\operatorname{mod} k)$。

然后发现 $n$ 还要满足 $\ge \dfrac{k(k+1)}{2}$ 才可以。

所以现在就得到了判定的两个条件。

可以发现，$2^k$ 形式的数必然无解，其他必然有解。

奇数直接令 $k = 2$ 即可。

然后有一个我还暂时不会证明的结论。

排除完无解情况之后。

把一个数的所有 $2$ 因子提出来组成 $2^k$。

然后答案必然是 $2^{k+1}$ 和 $\dfrac{n}{2^k}$ 的最小值。

??? node "Code"
	```cpp
	#include <cmath>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	using i64 = long long;
	using i128 = __uint128_t;
	
	int T;
	
	inline i64 solve(i64 n) {
		i64 tmp = n;
		i64 cnt = 1;
		
		while(tmp % 2ll == 0ll) {
			tmp /= 2ll,
			cnt *= 2; 
		}
		
		if(tmp == 1ll) return -1ll;
		return min(tmp, 2*cnt);
	}
	
	int main() {
		cin >> T;
		while(T--) {
			i64 n;
			cin >> n;
			cout << solve(n) << endl;
		}
		return 0;
	}
	```

```cpp
Tag : 数学/数论/同余
```

### CF1656E Equal Tree Sums

> 给定一棵树，要求你给每个节点一个权值，使得去掉树中任意一个节点之后，
>
> 任意两个连通块之内的和是相等的。
>
> 1e5。

`Mar/25/2022`

结论：黑白染色，一种颜色的节点赋值为 $-deg(u)$，另外一种 $+deg(u)$。

考虑一个点对于和它相连的所有顶点，在删除之后做的贡献即可。

更好一点的证明：

[![qN62wV.png](https://s1.ax1x.com/2022/03/25/qN62wV.png)](https://imgtu.com/i/qN62wV)

??? note "Code"
	```cpp
	
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	
	const int si = 1e5 + 10;
	
	int n, T;
	
	int head[si], tot = 0;
	struct Edge {
		int ver, Next;
	}e[si << 1];
	int deg[si];
	inline void add(int u, int v) {
		e[tot] = (Edge){v, head[u]}, head[u] = tot++;
	}
	
	int c[si];
	
	void dfs(int u, int fa, int col) {
		if(c[u] != -1) return;
		c[u] = col;
		for(int i = head[u]; ~i; i = e[i].Next) {
			int v = e[i].ver;
			if(v == fa) continue;
			dfs(v, u, 1 - col);
		}
	}
	
	int main() {
		cin >> T;
		while(T--) {
			cin >> n;
			memset(c, -1, sizeof c), memset(deg, 0, sizeof deg);
			tot = 0, memset(head, -1, sizeof head);
			for(int i = 1; i < n; ++i) {
				int u, v;
				cin >> u >> v;
				add(u, v), add(v, u);
				++deg[u], ++deg[v];
			}
			dfs(1, 0, 1);
			for(int i = 1; i <= n; ++i) {
				if(c[i] == 0) cout << -deg[i] << " ";
				else cout << deg[i] << " ";
			}
			cout << endl;
		}
		return 0;
	}
	```

```cpp
Tag : 图论/思维
```

### ABC243F **Lottery**

`Mar/25/2022`

> Takahashi is participating in a lottery.
>
> Each time he takes a draw, he gets one of the $N$ prizes available. Prize $i$ is awarded with probability $\dfrac{w_i}{\sum^{N}_{j=1} w_j}$ , The results of the draws are independent of each other.
>
> What is the probability that he gets exactly $M$ different prizes from $K$ draws? Find it modulo $998244353$.
>
> $1\le N,M,K \le 50$。
>
> $M \le N$。

定义 $P(i) = \dfrac{w_i}{\sum^{N}_{j=1} w_j}$

可以发现，如果一种元素被选了 $c_i$ 次，那么它的概率就是 $P(i)^{c_i}$

然后考虑用可重集的排列数公式算。

但是这里要稍微魔改一下。

那么可以得到一个柿子：

$$\dfrac{K!}{\prod\limits_{i = 1}^{N} c_i!} \times \prod\limits_{i=1}^{N} P(i)^{c_i}, \sum c_i = K,  |\{i\ |\ c_i \not= 0\}|=M$$

这个就是答案。

其中 $c_i$ 表示 $i$ 这种物品被选了多少次，可以是 $0$。

$K!$  是常数，所以提出来：

$$K!\times \dfrac{\prod\limits_{i=1}^{N} P(i)^{c_i}}{\prod\limits_{i = 1}^{N} c_i!}$$

发现后面这个部分可以化成

$$\prod\limits_{i = 1}^{n} \dfrac{P(i)^{c_i}}{c_i}$$

必然可以递推。

然后设 $dp[i][j][k]$ 表示 $N = i, M = j, K = k$ 的时候的这个东西。

考虑枚举每一种物品选多少个即可转移。

复杂度 $\text{O}(NMK^2)$。

??? note "Code"
	```cpp
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	using i64 = long long;
	
	const int si = 50 + 10;
	constexpr int mod = 998244353;
	
	int N, M, K;
	i64 dp[si][si][si];
	i64 fact[si];
	int w[si], sum = 0;
	
	inline i64 qpow(i64 a, int b) {
		i64 ans = 1 % mod;
		for(; b; b >>= 1) {
			if(b & 1) ans = ans * a % mod;
			a = a * a % mod;
		}
		return ans;
	}
	inline i64 inv(int a) {
		return qpow(a, mod - 2) % mod;
	}
	
	int p[si];
	
	int main() {
		cin >> N >> M >> K;
		for(int i = 1; i <= N; ++i) {
			cin >> w[i]; 
			sum += w[i];
		}
		
		fact[0] = 1;
		for(int i = 1; i <= K; ++i) {
			fact[i] = fact[i - 1] * i * 1ll % mod;
		}
		
		for(int i = 1; i <= N; ++i) {
			p[i] = ((1ll * w[i] % mod) * inv(sum)) % mod;
		}
		
		dp[0][0][0] = 1;
		for(int i = 1; i <= N; ++i) {
			for(int j = 0; j <= M; ++j) {
				for(int k = 0; k <= K; ++k) {
					for(int c = 0; c <= k; ++c) {
						if(j - (c > 0) >= 0) 
							dp[i][j][k] = (dp[i][j][k] + dp[i - 1][j - (c > 0)][k - c] * qpow(p[i], c) % mod * inv(fact[c]) % mod) % mod;
					}
				}
			}
		}
		
		cout << dp[N][M][K] * fact[K] % mod << endl;
		return 0;
	}
	```

```cpp
Tag : 组合数学/DP/递推
```

