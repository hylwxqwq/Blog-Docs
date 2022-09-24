
## 概述

树链剖分分三种，一种是轻重链剖分，一种是长链剖分，还有一种是实链剖分。

前者较为常用，中间的似乎可以拿来优化 DP（复杂度是 $\sqrt{n}$），后面的那一种用于 LCT。

下文用重链剖分代替树链剖分。

## 泛化

简单来说，重链剖分可以把树上的一类对于点权的操作和询问变为序列/区间上的操作和询问。

一般这些操作分两类：

1. 子树操作（子树加，子树查）
2. 路径操作（路径上加，路径上查）

重链剖分可以单次 $\text{O}(\log n)$ 的复杂度，快速将子树或者路径上的节点转化为一段或者多段区间。

再配合以线段树，树状数组等 $\log$ 数据结构可以以单次 $\text{O}(\log^2 n)$ 的复杂度完成信息的修改和查询。

## 预处理(剖分过程)

### 几个概念

定义 $hson_u$ 表示 $u$ 的所有儿子中，子树大小最大的那一个儿子，称为**重儿子**。

（多个相同任取一个即可）

其它的儿子称作 $u$ 的**轻儿子**。

定义**重边**为 $u$ 到 $hson_u$ 的一条边（$u$ 不需要是它的 $father$ 的重儿子）。

其他的边称作**轻边**。

若干条重边首尾相连形成的链称为**重链**。

落单的节点也当作重链，然后可以整棵树就被剖分成了**若干条不相交的重链**。

你知道处理子树信息可以利用 dfs 树和 dfs 序的性质，即是一棵子树内的 $dfn$ 值是连续的。

所以我们还需要预处理出 $dfn$，并记录每个 $dfn$ 值对应的节点编号 $rnk$ 方便线段树等数据结构的操作。

为了之后处理方便，还需要处理 $u$ 的子树大小 $siz_u$，父亲节点 $fat_u$，深度 $dep_u$，以及 $u$ 所在的重链上深度最小的节点 $top$ （即为链顶）。

也就是说我们需要处理这些东西：

$fat, dep, siz, hson, top, dfn, rnk$

一张图（图源 [OI-Wiki](https://oi-wiki.org/graph/hld/)）：

[![O54ruj.png](https://s1.ax1x.com/2022/05/17/O54ruj.png)](https://imgtu.com/i/O54ruj)

### 几个性质

1.树上每个节点都属于且仅属于一条重链。

这个根据上面概念里说的：

> 落单的节点也当作重链，然后可以整棵树就被剖分成了若干条不相交的重链。

就可以很容易的得到。

2.子树内的 $dfn$ 是连续的

这个是 dfs 序的性质，比较容易得到。

推论： 子树内 $dfn$ 值的区间应当是 $[dfn_u, dfn_u + siz_u - 1]$。

这两个结论会用于子树操作的处理。

3.同一条重链上的 $dfn$ 是连续的

这个是重链剖分本身的性质，需要我们 dfs 处理时 **“优先”** dfs 重儿子，然后再 $dfn$ 轻儿子。

可以发现，按 $dfn$ 序排序之后的序列，实际上就是把一条条重链拼接在了一起。

所以重链剖分实际上就是 “树上问题” 序列化的一个工具。

4.经过一条轻边 $(u\to v)$的时候，$siz_v$ 的大小必然是 $siz_u$ 的二分之一，可能还少。

根据这个结论，把一条路径从 $\texttt{LCA}$ 拆开从两边分别往下跳重链，跳的次数会在 $\text{O}(\log n)$ 级别。

所以树上任意一条路径都可以拆成 $\text{O}(\log n)$ 条重链。

### 代码实现

```cpp
// 处理重儿子,父亲,深度,子树大小
void dfs1(int u, int fa) {
	int kot = 0;
	hson[u] = -1, siz[u] = 1;
	fat[u] = fa, dep[u] = dep[fa] + 1;

	for(int i = head[u]; ~i; i = e[i].Next) {
		int v = e[i].ver;
		if(v == fa) continue;
		
		dfs1(v, u), siz[u] += siz[v];
		
		if(siz[v] > kot)
			kot = siz[v], hson[u] = v;
	} 
}
// 处理 dfn,rnk,并进行重链剖分。
void dfs2(int u, int tp) {
	top[u] = tp, dfn[u] = ++tim, rnk[tim] = u;
	
	if(hson[u] == -1) return;
	dfs2(hson[u], tp);
	// 先 dfs 重儿子,保证重链上 dfn 连续,维持重链的性质
	
	for(int i = head[u]; ~i; i = e[i].Next) {
		int v = e[i].ver;
		if(v == fat[u] || v == hson[u]) continue;
		
		dfs2(v, v);
	}
}
```

## 子树操作

这个相对比较简单，直接利用 $dfn$ 的性质操作即可。

（基于上面的那个推论）

`tr` 是一颗线段树，你也可以在信息比较简单的时候用树状数组。

```cpp
void add_subtree(int u, int value) {
	tr.update(1, dfn[u], dfn[u] + siz[u] - 1, value);
	// 子树代表的区间的左右端点分别是 dfn[u], dfn[u] + siz[i] - 1;
} 
int query_subtree(int u) {
	return tr.query(1, dfn[u], dfn[u] + siz[u] - 1) % mod;
}
```

## 路径操作

这个也比较简单，考虑上面性质的第 4 条。

我们可以把树上任意一条路径拆成 $\text{O}(\log n)$ 条重链。

然后这个过程可以看成是从 $\texttt{LCA}$ 往两边走。

我们考虑它的逆过程，也就是类似倍增求 $\texttt{LCA}$ 的过程。

我们让**当前所在链顶深度更大的节点**不断向上跳重链，每次跳的时候对于 $u$ 和 $top_u$ 进行操作。

（比如 `tr.update(1, dfn[top[u]], dfn[u], value)` 这种）

然后当 $u, v$ 在同一条重链上的时候，我们直接令 $u$ 为深度更小的节点，然后对维护信息的数据结构上 $[dfn_u, dfn_v]$ 的区间进行操作即可。

```cpp
// 类似倍增 LCA 的跳重链过程
void add_path(int u, int v, int value) {
	while(top[u] != top[v]) {
		if(dep[top[u]] < dep[top[v]])
			swap(u, v);
		// 让链顶深度大的来跳
		
		tr.update(1, dfn[top[u]], dfn[u], value);
		// 把 u 到链顶的权值全部修改。
		u = fat[top[u]];	
		// 跳到链顶的父亲节点。	
	}
	
	if(dep[u] > dep[v]) swap(u, v);
	tr.update(1, dfn[u], dfn[v], value);
	// 一条重链上的 dfn 是连续的。
}
int query_path(int u, int v) {
	int ret = 0;
	while(top[u] != top[v]) {
		if(dep[top[u]] < dep[top[v]])
			swap(u, v);
	
		ret = (ret + tr.query(1, dfn[top[u]], dfn[u])) % mod;
		u = fat[top[u]];
	}
	
	if(dep[u] > dep[v]) swap(u, v);
	ret = (ret + tr.query(1, dfn[u], dfn[v])) % mod;
	return ret % mod;
}
```

## 求 LCA

类似路径操作的过程即可。

也是不断跳重链，然后跳到同一条重链上之后深度小的节点就是 $\texttt{LCA}$。

常数非常小。

原因是通常跳重链的时候，重链个数不会卡满 $\text{O}(\log n)$。

（除非拿完全二叉树）。

```cpp
int lca(int u, int v) {
	while(top[u] != top[v]) {
		if(dep[top[u]] < dep[top[v]]) 
			swap(u, v);
		u = fat[top[u]];
	}
	if(dep[u] > dep[v]) swap(u, v);
	return u;
}
```

----

提一嘴：其实我感觉 hld 和树上差分的关系相当于树状数组和前缀和的关系 XD。