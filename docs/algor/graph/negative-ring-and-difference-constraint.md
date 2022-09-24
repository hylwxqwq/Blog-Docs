## 负环

大概就是图上的一个环，环上所有边的权值之和是负数。

正常跑最短路的话就会在上面无限转下去。

处理的时候可以利用 Bellmanford 和 SPFA 的性质来判断。

现在在正常的最短路上用下面的两种方式之一进行判定：

1. 如果有一个点被迭代了不少于 $n$ 次（入队不少于 $n$ 次），那么图中必然存在负环。
2. 如果源点到某一个点的最短路有不少于 $n$ 条边，那么图中必然存在负环。

1 很好理解，被入队了不少于 $n$ 次，就说明无论如何迭代，始终存在至少一条边满足三角形不等式，在最短路当中对应的就是负环。

2 也差不多，一个 $n$ 个点，$n$ 条边的联通图必然是存在环的（好像描述有点问题），最短路是个环，那么必然是出现了负环。

通常来说第二种做法效率更高，比如

```cpp
1->2, 2->3, 3->4, 4->5, ..., n-1->n, n->1
```

这种图，1 的做法就要迭代 $\text{O}(n^2)$  级别次，2 只需要 $n$ 次。

还有一种优化是把 `std::queue` 换成 `std::stack`。

另外一种**不一定正确**的卡时 trick 是，当所有节点的总入队次数超过某个设定值的时候，就直接**认为**图中存在负环。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	constexpr int si_n=5e2+10;
	constexpr int si_m=5e3+2e2+10;
	int n,m,q;
	int T,tot=0;
	struct Edge{ int head,Next,ver,w; }e[si_m];
	inline void add(int u,int v,int w){ e[++tot].ver=v,e[tot].Next=e[u].head,e[u].head=tot,e[tot].w=w; }
	int dis[si_n],cnt[si_n];
	bool vis[si_n];
	std::queue<int>Q;
	inline bool spfa(int s){
		memset(dis,0,sizeof dis),memset(cnt,0,sizeof cnt),memset(vis,false,sizeof vis);
		for(register int i=1;i<=n;++i){
			Q.push(i),vis[i]=true;
		} cnt[s]=0; // 全部入队，相当于建立一个超级源点。
		while(!Q.empty()){
			int u=Q.front(); Q.pop();
			vis[u]=false;
			for(register int i=e[u].head;i;i=e[i].Next){
				int v=e[i].ver,w=e[i].w;
				if(dis[v]>dis[u]+w){
					dis[v]=dis[u]+w;
					cnt[v]=cnt[u]+1;
					if(cnt[v]>=n) return true;
					if(!vis[v]) Q.push(v),vis[v]=true;
				}
			}
		} return false;
	}
	```

还有，如果只是判定负环的话，$dis$ 初始化成多少都是没有问题的。

换成判断正环的话，求最长路即可。

## 差分约束

> 给定一个不等式组，每个不等式形如 $x_i \le x_j +C_k$ ，其中 $C_k$ 是常数（正负均可）， $i,j$ 是自变量。
>
> 问一组可行解 $x_1,x_2 \dots x_n$。

不等式长的很像三角形不等式，所以考虑利用图论分析。

比如 $x_i\le x_j +C_k$，可以看作 $j\to i$ 的路径上有一条权值是 $C_k$ 的边。

最终满足条件时，实际上就是所有类似 $x_i > x_j+C_k$ 的三角形不等式都不成立，也就是求完最短路之后的情况。

此时，从源点出发到每个点的 $dis_i$ 就是对应的 $x_i$，$dis$ 就是一组可行解。

如果图中存在负环，那么在环上转一圈之后必然会出现 $x_i \le x_i +\sum C_k , \sum C_k <0$ 的情况，此时矛盾（这个不等式相当于 $x_i < x_i$），无解。

如果现在要求的不等式变成了 $x_i \ge x_j + C_k$ ，跑最长路即可，无解变成判断正环。

当然，为了保证所有的不等式都的得到满足，需要找到一个能够从它出发，经过所有**边**的源点进行 SPFA，这个建立一个超级源点就行了。

不过，如果题目要求了类似 $\forall i,x_i>0$ 的要求，且还要求最小的话，就需要连边而不是全部放到队列里面了，边权根据题目判断，比如前面的例子就需要给每一个点连 $0 \to i,w=1$，然后让 $dis_0=0$ 。

如果出现 $x_i \le C_k$ 这种条件，让 $x_i$ 和超级源点连 $C_k$ 的边就可以了。

$< >$ 可以用 $+-1$ 来变化成 $\le \ge$。

$=$ 等价于 $\le \land\ge$ 。

如果在题目里遇到 $\ge \le$ 同时出现，变换方向，移动 $C_k$ 即可。