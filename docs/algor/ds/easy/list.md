
## 概述

这里说的链表是双向链表，它支持在任意位置插入或者删除元素，但是只能按照顺序访问。

一个双向链表的结点通常由元素值 $val$ ，指向下一个节点的指针 $next$ 和指向上一个节点的指针 $prev$

一般来说我们会额外建立表头 $head$ 和表尾 $tail$ 指针，并将它们首尾相连，来避免不必要的问题。

## 实现

### 初始化

初始化链表只需要把 $head$ 的 $next$ 指向 $tail$ ， $tail$ 的 $prev$ 指向 $head$ 即可

```cpp
inline void init(){
	head=new node(),tail=new node();
	head->next=tail,tail->prev=head;
} // init new List
```

### 插入元素

在节点 $p$ 之后插入一个节点 $q$ ，并且节点值为 $val$ 。

我们只需要让 $p$ 的 $next$ 节点的 $prev$ 指向 $q$ ，让 $q$ 和 $p$  以及原来的 $p.next$ 相连即可。

最后把 $p$ 的 $next$ 指向 $q$ ，注意是最后，不然你在前面操作的时候就会让 $q$ 的 $prev$ 指向自己的。

```cpp
inline void insert(node *p,int val){
	node *q;q=new node();
	q->val=val,p->next->prev=q;
	q->next=p->next,q->prev=p,p->next=q;
} // insert a element ,value is val, after p
```

### 删除元素

删除一个节点 $p$ 也比较简单，只需要把 $p$ 的 $prev$ 以及 $next$ 链接起来即可。

```cpp
inline void remove(node *p){
	p->prev->next=p->next,p->next->prev=p->prev;
	delete p;
} // remove p
```

### 清空链表

回收链表内存的话只需要让 $head$ 一步步逼近 $tail$ ，在逼近的过程中把访问过的 $head.prev$ 删除即可。

注意这里要先跳 $next$ 之后再删除 $prev$ ，不然很容易会访问无效内存的，但是我们开头的连接链表的首尾就完美的解决了这个问题。

```cpp
inline void reset(){
	while(head!=tail){
		head=head->next;
		delete head->prev;
	} delete tail;
} // clear
```