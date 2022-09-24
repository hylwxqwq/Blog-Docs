
## 概述

感觉就是一个简单的小 Trick，单独提出来写一下。

本质上就是说，你把二分直接放到线段树上来搞，然后注意一下啥时候走左子树啥时候走右子树啥时候返回就行了。

一般需要问题存在一定的单调性。

## 例题

### 静态全局第 k 大

简单题，全局第 $k$ 大，直接开一个权值线段树。

然后你考虑什么时候走左子树，显然就是左子树的和（元素个数）大于等于 $k$，证明第 $k$ 大在这边。

否则你把左子树大小 $sz$ 从 $k$ 里面减掉，在右子树询问第 $k - sz$ 大。

然后递归一下就完了。

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

### 单点修改区间查询 lower_bound

也是比较简单的题，只不过这次是在线段树上面直接二分。

考虑设一个 $ask(ql,qr,d)$ 函数，表示当前我询问 $[ql, qr]$ 的所有数当中第一个大于等于 $d$ 的数的下标。

而且还有一个非常重要的第二个定义：定义 $ask(ql, qr, d)$ 表示，**我能肯定潜在的答案必然在区间** $[ql, qr]$ 内，不管是否真的存在这个元素，反正如果可能出现就必然在 $[ql, qr]$ 里面。

所以我们递归的时候要修改 $[ql, qr]$，此时就不能沿用普通线段树的递归和边界条件了。

首先如果当前线段树节点维护 $[l, r]$，会有以下三种情况:

1. $[ql, qr] \subset [l, mid]$，直接递归左子树就行了，因为 $[ql, qr]$ 的线度比 $[l, mid]$ 小，所以不用改变。
2. $[ql, qr] \subset (mid, r]$，同理递归右子树。
3. $[ql, qr] \subset [l, r] \land (qr > mid) \land (ql \le mid)$，也就是分成了两半，因为要第一个所以我显然优先递归左子树，同时 $(ql, qr) \to (ql, mid)$，如果左子树答案不存在，不能直接返回，还要再跑右子树试一试。

然后考虑边界情况应该是什么，其实就是 $l = r$ 的时候会找到最终的潜在答案（$-1$，或者就是 $l$）。

所以你如果找到了答案，最后一定是会缩小到区间长度 $= 1$ 的。

```cpp
int ask(int p, int ql, int qr, int d) {
	int l = t[p].l, r = t[p].r;
	if(l == r) {
		if(t[p].mx >= d) 
			return l;
		return -1; 
	}
	int mid = (l + r) >> 1;
	if(qr <= mid) 
		return ask(p << 1, ql, qr, d);	
	if(ql > mid) 
		return ask(p << 1 | 1, ql, qr, d);
	int pos = ask(p << 1, ql, mid, d);
	if(~pos) 
		return pos;
	return ask(p << 1 | 1, mid + 1, qr, d);
}
```

但是这个其实跑的很慢。

有一个小优化：我们考虑 $l = ql, r = qr$ 的时候，我直接判一下 $t(2p).mx,t(2p + 1).mx$ 是不是 $\ge d$，这样用 $O(1)$ 换掉了可能的很多次递归。

```cpp
int ask(int p, int ql, int qr, int d) {
	int l = t[p].l, r = t[p].r;
	if(l == ql && r == qr) {
		if(t[p].mx < d) return -1;
		if(l == r) return l;
		int mid = (l + r) >> 1;
		if(t[p << 1].mx >= d) 
			return ask(p << 1, ql, mid, d);
		return ask(p << 1 | 1, mid + 1, qr, d);
	}
	int mid = (l + r) >> 1;
	if(qr <= mid) 
		return ask(p << 1, ql, qr, d);	
	if(ql > mid) 
		return ask(p << 1 | 1, ql, qr, d);
	int pos = ask(p << 1, ql, mid, d);
	if(~pos) 
		return pos;
	return ask(p << 1 | 1, mid + 1, qr, d);
}
```

### [省选联考 2020 A/B 卷] 冰火战士

很好，我还没做。

### 静态区间第 k 大

就是用主席树，把全局变成区间。

然后在主席树上面二分就行了。

具体不会，之后写。

### 动态区间第 k 大

就是额外用一个树状数组去维护主席树的信息支持单点修改就行了。

然后和静态区间第 k 大没啥差别。

具体不会，之后写。
