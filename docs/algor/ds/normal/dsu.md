
## 概述

dsu 大概就是拿来维护一系列元素所属集合的数据结构。

支持合并集合，查询所属集合的操作。

变种就扩展域和带权两种，还有两种比较常见的小 trick。

下面就简单提两句。

## 基本操作

首先设 $pa(i)$ 表示元素 $i$ 所属集合的代表元素。初始 $pa(i) = i$。

### 查询所属集合

这个比较简单，就直接从当前节点一直向上跳，然后直到找到一个 $pa(i) = i$ 的节点即可。

```cpp
int root(int x) {
	if(pa[x] != x) return root(pa[x]);
	return pa[x];
}
```

有一个简单的优化是，每次查询完一个元素之后，把它的 $pa(i)$ 直接指向实际的集合根。

这东西叫路径压缩。

```cpp
int root(int x) {
	if(pa[x] != x) pa[x] = root(pa[x]);
	return pa[x];
}
```

### 合并两个集合

就直接把一个集合的根的 $pa$ 设置成另外一个集合的根即可。

```cpp
void Merge(int x, int y) {
	int rx = root(x), ry = root(y);
	if(rx == ry) return;
	pa[ry] = rx; return;
}
```

但是注意到我们希望合并之后的跳 $pa$ 路径长度更小，所以我们可以考虑启发式合并。

具体来说维护集合的 $siz$ 或者 $dep$，然后启发式合并一下即可。

```cpp
void Merge(int x, int y) {
	int rx = root(x), ry = rooot(y);
	if(rx == ry) return; 
	if(siz[rx] < siz[ry]) 
		pa[rx] = ry, siz[rx] += siz[ry];
	else pa[ry] = rx, siz[ry] += siz[rx];
}
```

如果单独使用路径压缩或启发式合并，复杂度都是 1log。

如果同时使用，复杂度是 $O(\alpha(n))$，近乎 $O(1)$。

## 变种

### 带权并查集

这个就是维护了一下节点到父节点的一个距离一样的东西。

没啥好说的。

### 扩展域并查集

其本质是有多个传递性信息的时候对于每个节点拆多个点出来维护。

然后在维护的时候判断矛盾和从属关系。

## 应用

### 维护传递性信息

比如给你一堆不等式形如 $x_1 = x_2$ 或 $x_1 \not= x_2$ 这种。

问你是否可能满足要求。发现等号是有传递性的，直接把所有等于的扔一个集合里面，扫一遍不等的看是否满足即可。

这种问题也可以用 2-SAT 来做，但是 2-SAT 适用性更广泛一点。

### 奇怪的连边技巧 

这是之前做 GSS4 的时候 gjh 教我的。

直接看这里吧：[link](../../../rec/interesting/segtree-tricks.md#gss4)
