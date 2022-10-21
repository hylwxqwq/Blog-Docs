
## Hash

说 Hash table 之前先简单的提一嘴 Hash。

Hash 的思想就是 mapping，或者说 “映射”。

它的作用就是 “缩小信息域”，把原来很分散或者范围很大的东西压缩一下，方便处理。

最常见的应用就是离散化，把 $a_i$ 映射成 $val(a_i)$，那从 $\{a\} \to val$ 的这个映射就利用了 Hash 的思想。

一般 Hash 都会有一个 Hash function，用来把对应值映射到一个新的位置上去，比如离散化当中的 Hash function 就是 $val$。

## Hash table

考虑这样一个问题，给你一个序列 $a$，统计每个数出现了多少次，$0\le a_i \le 1e5$。

显然会想到直接用桶计数，那么扩展一下问题，令 $a_i \in [-10^{18},10^{18}]$？

直接统计没法做了，当然你可以离散化之后扫一遍，或者直接利用 STL Map 来实现。

STL Map 本质上是一颗红黑树，但是 gnu_pbds 里面有两个数据结构叫 cc/gp_hash_table，这两个东西就是 Hash table 作为底层的，只不过应对 Hash 冲突的方法不同。

考虑设 Hash function : $H(x) = (x \mod p) + 1$，$p$ 是一个较大的但是小于 $n$ 的质数。

这个 Hash function 会把序列分成 $p$ 组，在序列随机给出的情况下每组的大小是均匀的。

我们把这 $p$ 组数扔到邻接表上。

对于序列 $a$，我们从头开始一个一个插入，对于 $a_i$，我们从 $head(H(a_i))$ 这个位置开始找，然后在这个链表里面开始找，如果找不到 $a_i$，我们就在这组链表的开头插入一个节点并将其储存的值加一，找到了就直接在对应位置加一。

因为数据随机，所以这个的期望复杂度近似 $O(n)$，也就是说每次插入操作均摊 $O(1)$

个人感觉在尾部插入也差不多？不过在头部插入会好写一点（毕竟链式前向星就这么写）。

这种方法叫拉链法，也叫开散列法，它的思想是把所有 Hash 冲突的元素扔到一个组里面。

还有一种方法是探查法，直接把所有元素存在一个表当中，如果发生 Hash 冲突则根据某种方式继续进行探查，找到一个空的位置把它放进去。

常用的探查方法是线性探查法，就是如果在 $pos$ 发生冲突，就不断找 $pos + 1, pos + 2, \dots$ 直到不冲突。

这里只实现了拉链法。

```cpp
// 瞎写的，没编译没测过，仅供参考。
const int si = 1e5 + 10;
int n, a[si], tot = 0;
int head[si], val[si], cnt[si], Next[si];

int H(int x) { return (x % p) + 1; }
bool insert(int x) {
	bool exist = false;
	int u = H(x); 
	for(int i = head[u]; ~i; i = Next[i]) {
		if(val[i] == x) {
			cnt[i]++, exist = true;
			break;
		}
	}
	if(exist) return true;
	++tot, Next[tot] = head[u], val[tot] = x, cnt[tot] = 1, head[u] = tot;
	return false;
}
int query(int x) {
	int u = H(x);
	for(int i = head[u]; ~i; i = Next[i])
		if(val[i] == x) return cnt[i];
	return 0;
}
// 记得初始化。
```