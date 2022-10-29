## 边双联通分量 e-DCC

割边/桥：如果在无向图 $G=(V,E)$ 当中去掉一条边 $(u,v)$ 后，$G$ 分裂为两个不联通的子图，则称边 $(u,v)$ 是无向图 $G$ 的一个桥。

边双联通分量：不含桥边的极大联通子图。

类似 SCC，引入 $G$ 的一颗搜索树 $T$ 和 $dfn,low$。

$low$ 的定义也是从某个节点向上最高（在 $T$ 当中）能到达的节点的 $dfn$。

那么要找到桥边，实际上就是找到搜索树上相邻的两个节点 $(u,v)$ ，使得 $u$ 是 $v$ 的父亲且 $dfn_u<low_v$，也就是 $v$ 无论如何都没有办法走到 $u$ 或者 $u$ 更上面的节点。

这是一个无向图，所以 $v$ 和 $v$ 的子树都是没有边能到达 $u$ 以及更上面的子图的。

因为无向图的 dfs 树以外的边都不是横叉边，所以只需要考虑“回边”的影响。

那么必然可以证明，$(u,v)$ 一定是桥边。

这里也可以同时得到一个性质，**桥边一定是树边**。

而且，**任意一个简单环中的边都不是桥边**。

去掉桥边之后，剩下的连通块就是一个个 e-DCC。

缩点也比较容易，直接把所有去掉桥边之后的连通块分别合并成一个节点即可。

??? note "Code"
	```cpp
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	
	const int si = 2e5 + 10;
	
	int n, m, q;
	
	// 原图
	int head[si], tot1 = 0;
	struct Edge {
	    int ver, Next;
	}e[si << 2]; 
	inline void add1(int u, int v) {
	    e[tot1] = (Edge){v, head[u]}, head[u] = tot1++;
	}
	
	// 缩完点之后的图
	// 如果原来的图是连通图的话
	// 可以证明缩完点之后必然是一棵树。
	int Head[si], tot2 = 0;
	struct Tree {
	    int ver, Next;
	}t[si << 2];
	inline void add2(int u, int v) {
	    t[tot2] = (Tree){v, Head[u]}, Head[u] = tot2++;
	}
	
	// E-dcc 的个数.
	int cnt = 0; 
	
	int dfn[si], low[si], tim = 0;
	// 是否是桥
	bool bridge[si << 2];
	int c[si];
	
	// in_edge 是用来消除重边的影响的。
	// 表示当前状态是从哪一条边过来的。
	void tarjan(int u, int in_edge) {
	    dfn[u] = low[u] = ++tim;
	    for(int i = head[u]; ~i; i = e[i].Next) {
	        int v = e[i].ver;
	        if(!dfn[v]) {
	            tarjan(v, i);
	            low[u] = min(low[u], low[v]);
	            if(dfn[u] < low[v]) bridge[i] = bridge[i ^ 1] = true; 
	        }
	        else if((i ^ 1) != in_edge) low[u] = min(low[u], dfn[v]);
	    }
	}
	
	// 去掉桥边的连通块染色
	void dfs(int u, int col) {
	    c[u] = col;
	    for(int i = head[u]; ~i; i = e[i].Next) {
	        int v = e[i].ver;
	        if(c[v] || bridge[i]) continue;
	        dfs(v, col);
	    }
	} 
	void Construct() {
	    for(int i = 1; i <= n; ++i){
	        for(int j = head[i]; ~j; j = e[j].Next) {
	            int v = e[j].ver;
	            if(c[i] == c[v]) continue;
	            // 只需要加一次，遍历到反向边的时候会自动补全成无向边
	            add2(c[i], c[v]);
	        }
	    }
	}
	
	int main() {
		memset(head, -1, sizeof head);
	    memset(Head, -1, sizeof Head);
	    memset(bridge, false, sizeof bridge);
	    cin >> n >> m;
	    for(int i = 1; i <= m; ++i) {
	        int u, v;
	        cin >> u >> v;
	        add1(u, v), add1(v, u);
	    }
	    for(int i = 1; i <= n; ++i) if(!dfn[i]) tarjan(i, -1);
	    for(int i = 1; i <= n; ++i) if(!c[i]) ++cnt, dfs(i, cnt);
	    Construct();
	}
	```

$in\_edge$ 表示递归到当前节点所经过的那一条边的编号。

$in\_edge$ 存在的原因是，如果按照正常搜索树的更新方式，$fa_u$ 的 $dfn$ 必然不会用来更新 $u$ 的 $low$。

但是如果出现重边的话，那 $(fa_u,u)$ 必然不是桥边，那么就需要用 $fa_u$ 的 $dfn$ 更新 $low_u$。

可以证明的一个推论：**对于一个无向连通图，Edcc 缩完点之后必然会形成一棵树**。

因为只要图上带环，就可以继续缩点，而树是无向连通无环图，所以得证。

## 点双联通分量 v-DCC

割点：如果对于无向图 $G=(V,E)$，$\exist x \in V$，使得删除 $x$ 之后，$G$ 分裂成两个及以上的不联通的子图，则称节点 $x$ 是无向图 $G$ 的一个割点。

点双连通分量：不含割点的极大联通子图，但是注意，这里**不能直接求出割点之后去掉割点把连通块合并**。

因为割点本身就属于它连接的点双联通分量（同时属于）。

所以要特别注意。

仍然引入 $T$ ，$dfn$ 和 $low$ 。

考虑一个节点 $u$ 怎么才可能成为割点。

如果存在一条树边 $(u,v)$ 且 $u$ 是 $v$ 的父亲节点，且满足 $dfn_u \le low_v$，$u$ 不是根节点。

如果 $u$ 是根节点，那么需要满足两次以上 $dfn_u \le low_v$。

也就是 $v$ 和 $v$ 的子树里的节点最高只能到 $u$ ，到不了 $u$ 更上层的节点，那么 $u$ 必然会把 $v$ 和 $v$ 的子树以及 $u$ 上面的子图分开。

所以此时 $u$ 就是一个割点。

递归的同时维护一个栈，原理类似 SCC。

然后每次满足 $dfn_u \le low_v$ 的时候就弹出，直到 $v$ 出栈，被弹出的节点就组成一个 v-DCC。

（这里不需要判根，可以自己画图）

不要把 $u$ 弹出去了，如果 $u$ 是割点，那么之后访问到的 v-DCC 里面就会少 $u$ 这个点。

因为判定条件里面有 $=$，所以就不需要再判 $in\_edge$ 了，不过在没有特殊说明的情况下，一定要记住把重边直接在读入的时候判掉。

缩点的话唯一要注意的就是要把割点单独分裂出来，但是割点又要存在于它连接的所有v-DCC 当中。

所以要给割点一个新编号。

??? note "Code"
	```cpp
	#include <stack>
	#include <vector>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	
	const int si = 1e5 + 10;
	
	int n, m, root;
	
	// 原图
	int head[si], tot1 = 0;
	// 新图
	int Head[si], tot2 = 0;
	struct Edge {
		int ver, Next;
	}e[si << 2], g[si << 2];
	inline void add1(int u, int v) {
		e[tot1] = (Edge){v, head[u]}, head[u] = tot1++;
	}
	inline void add2(int u, int v) {
		g[tot2] = (Edge){v, Head[u]}, Head[u] = tot2++;
	}
	
	
	// Vdcc 的个数
	int cnt = 0;
	int dfn[si], low[si];
	int c[si], tim;
	// 割点的新编号
	int new_id[si];
	// 是否是割点
	bool cut[si];
	stack<int> s;
	vector<int> vdcc[si];
	
	void tarjan(int u) {
		dfn[u] = low[u] = ++tim;
		s.push(u);
	
		// 孤立点
		if(u == root && head[u] == -1) {
			vdcc[++cnt].emplace_back(u);
			return;
		}
	
		int flag = 0;
		for(int i = head[u]; ~i; i = e[i].Next) {
			int v = e[i].ver;
			if(!dfn[v]) {
				tarjan(v), low[u] = min(low[u], low[v]);
				
				if(dfn[u] <= low[v]) {
					++flag;
					// 根节点特判
					if(u != root || flag > 1) { // 注意这里是短路运算符，不要打反了。
						cut[u] = true;
					}
					int x; ++cnt;
					do {
						x = s.top(), s.pop();
						vdcc[cnt].emplace_back(x);
					} while(v != x);
					// 注意这里要是 v 不是 u
					// 如果 u 被弹出了，之后的连通块就会少 u。
					vdcc[cnt].emplace_back(u);
				}
			}
			else low[u] = min(low[u], dfn[v]);
		}
	}
	
	int num;
	void Construct() {
		num = cnt;
		for(int u = 1; u <= n; ++u) {
			if(cut[u]) new_id[u] = ++num;
		}
	
		for(int i = 1; i <= cnt; ++i) {
			for(int j : vdcc[i]) {
				if(cut[j]) add2(i, new_id[j]), add2(new_id[j], i);
				else c[j] = i;
			}
			// 如果是割点，就和这个割点所在的 v-Dcc 连边
			// 反之染色。
		}
	
		// 编号 1~cnt 的是 v-Dcc, 编号 > cnt 的是原图割点
	}
	
	int main() {
		memset(head, -1, sizeof head);
		memset(Head, -1, sizeof Head);
	
		cin >> n >> m;
		for(int i = 1; i <= m; ++i) {
			int u, v;
			cin >> u >> v;
	        // 判重边
			if(u == v) continue;
			add1(u, v), add1(v, u);
		}
	
		for(int i = 1; i <= n; ++i) {
			if(!dfn[i]) root = i, tarjan(i);
		}
		Construct();
		return 0;
	} 
	```

其实，如果原图是**无向连通图的话**，**vdcc 缩完点之后得到的也必然是一棵树**。

并且，如果把割点看作白点， vdcc 看作黑点，那么这颗树必然是**黑白相间**的。

也可以理解为一个二分图。

证明也比较简单，证明是树的话类似 edcc。

然后问题可以转化为，树中的所有边都是一边是割点，一边是一个 vdcc。

证明其它情况不存在即可。

（其实看 `Construct` 里的加边也能看出来）