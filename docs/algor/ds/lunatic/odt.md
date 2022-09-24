## 概述

一种暴力数据结构，基于 `std::set` 或者链表

基本的思想是**把所有相邻且相同的元素合并**成一个元素维护。

如果遇到区间类的操作就分裂，暴力处理。

在随机数据下可以做到优秀的 $\text{O}(n \log \log n)$，链表实现可以 $\text{O}(n \log n)$。

复杂度证明可以看 @hqztrue 的：<https://zhuanlan.zhihu.com/p/102786071>

## 具体实现

### 初始化

珂朵莉树的节点由三个元素构成，$(l,r,v)$。

分别表示这个节点维护的区间的左右端点和权值。

因为需要用 `std::set` 实现，所以还要重载运算符。

```cpp
struct node{
	int l,r;
	mutable int val; // if we need change a node's value which already in the set by using iterator, we have to use mutable.
	node(const int &il,const int &ir,const int &iv):l(il),r(ir),val(iv){}
	inline bool operator < (const node &b)const{ return l<b.l; }
}; std::set<node>odt;
```

初始化直接插入一个节点维护 $[1,n]$ 即可，权值根据题目判断。

### 区间分裂

基本操作之一，把 $[l,r]$ 分割成 $[l,pos),[pos,r]$

并返回代表后面那个区间的节点的迭代器。

先二分一下 $pos$ 的位置，然后删除原来的节点，新加入两个节点即可。

```cpp
inline std::set<node>::iterator split(int pos){
	if(pos>n) return odt.end(); // the position doesn't exist.
	std::set<node>::iterator it=--odt.upper_bound((node){pos,0,0}); // find the node that pos in;
	if(it->l==pos) return it; // if pos is the begin of the node, return;
	int l=it->l,r=it->r,v=it->val;
	odt.erase(it),odt.insert((node){l,pos-1,v}); 
	return odt.insert((node){pos,r,v}).first; // erase the original node, insert two node and return the left one's iterator.
} // split the node [l,r] to two smaller node [l,pos),[pos,r];
```

这里 $it$ 是个迭代器，所以要用指针访问里面的成员。

`std::set` 当中的 `insert()` 是有返回值的。

是一个 `std::pair<std::set<Template>::iterator,bool>`

前者是被插入的值的迭代器，后者表示是否插入成功。

`--upper_bound` 那里也可以使用 `std::prev`

### 区间推平

推平一段区间，也就是给某个区间全部赋值成某个值 $v$。

如果没有额外限制的话，先把 $l,r$ 从他们各自所属的节点当中 `split` 出来，然后把他们中间的这一段全部删除，再插入一个新的节点 $(l,r,v)$ 维护这个区间即可。

```cpp
inline void assign(int l,int r,int v){
	std::set<node>::iterator itr=split(r+1),itl=split(l); // because of [), so r+1. and **Remember, split(r+1) first. then split(l)**
	odt.erase(itl,itr),odt.insert((node){l,r,v});
} // change all element in the interval [l,r] to v;
```

注意一定要先 `split(r+1)` ，不然 $l$ 原来所属的节点有可能被删除导致 Runtime Error。

### 其它操作

其它的操作利用 `split` 和 `assign` 暴力搞就好。

基本都是这样：

```cpp
inline void example(int l,int r,int v){
	std::set<node>::iterator itr=split(r+1),itl=split(l);
	for(;itl!=itr;++itl){
		// blablabla...
	} return;
}
```

