
## 概述

> 给一张带权图，问从某个点到其它所有点的最短距离。

如果是无权图可以用 BFS。

如果是只有 0/1 边权的可以使用双端队列 BFS。

如果是带权的单源最短路，那么就需要根据图的性质选择 Dijkstra, Spfa, BellmanFord。

如果是稠密图，并且要求全源最短路，在 $n$ 比较小的情况下使用 Floyd.

Johnson 不常用，不提了。

单源最短路的时候一般使用 $dis_i$ 表示从起点 $s$ 到 $i$ 的最短路。

全源的时候一般用 $dis_{i,j}$ 表示从 $i\to j$ 的最短路。

三角形不等式是 $dis_v>dis_u+w(u,v)$。

一般会在满足不等式的时候令 $dis_v=dis_u+w(u,v)$。

## Dijkstra 算法

思路很简单，首先用一个数组 $vis$ 标记某个节点是否被更新过。

初始化令 $dis_s=0$，其它的设成 $+ \infty$。

每次找找到一个没有被标记的， $dis$ 最小的节点 $x$ ，标记这个节点。

然后扫描 $x$ 的所有出边，利用三角形不等式更新它能到达的所有节点的 $dis$ 。

直到所有节点被标记完为止。

复杂度 $\text{O}(n^2)$。

优化：考虑利用一个小根堆，每次取出根进行更新即可，当某个节点被三角形不等式更新的时候将其放入堆。

??? note "Code"
	```cpp
	std::priority_queue<pair<int,int>>q;
	inline void dijkstra(int s){
	    memset(vis,false,sizeof vis),memset(dis,0x3f,sizeof dis);
	    dis[s]=0,q.push({dis[s],s});
	    while(!q.empty()){
	        int u=q.top().second; q.pop();
	        if(vis[u]) continue; vis[u]=true;
	        for(register int i=e[u].head;i;i=e[i].Next){
	            int v=e[i].ver,w=e[i].w;
	            if(dis[v]>dis[u]+w) dis[v]=dis[u]+w,q.push({-dis[v],v}); //利用相反数把大根堆->小根堆
	            // 一定要先更新 dis[v] 再 q.push
	        }
	    }
	}
	```

在非 DAG 上出现负边权就不能用。

## Bellman Ford 算法

思想也很简单。

扫描所有边，如果扫描到的这条边满足三角形不等式，更新对应节点的 $dis$。

重复迭代，直到没有更新操作发生，复杂度 $\text{O}(nm)$。

## Spfa

队列优化的 BellmanFord，有负边权也没有影响。

考虑使用一个队列，最开始的时候队列只包含起点。

每次取出队头，扫描队头的所有出边，如果满足三角形不等式则更新。

如果被更新的节点不在队列里面，把被更新的节点插入队尾。

直到队列为空，复杂度 $\text{O}(km)$。 $k$ 是个比较小的常数。

??? note "Code"
	```cpp
	std::queue<int>q;
	inline void spfa(int s){
	    memset(vis,false,sizeof vis),memset(dis,0x3f,sizeof dis);
	    dis[s]=0,q.push(s),vis[s]=true;
	    while(!q.empty()){
	        int u=q.front(); q.pop();
	        vis[u]=false;
	        for(register int i=e[u].head;i;i=e[i].Next){
	            int v=e[i].ver,w=e[i].w;
	            if(dis[v]>dis[u]+w){
	                dis[v]=dis[u]+w;
	                if(!vis[v]) q.push(v),vis[v]=true;
	            }
	        }
	    }
	}
	```

这个算法容易被卡，比如菊花图和蒲公英就随便卡SPFA。

所以给一个优化。

### Spfa + SLF + Swap

至少是我觉得最有效的优化。

虽然还是被 @fstqwq 学长在 zhihu 上和一堆神仙叉爆了。

考虑用一个双端队列优化，每次将入队结点的 $dis$ 和队首比较，如果更大则插入至队尾，否则插入队首。

这是普通的 SLF，更加优秀的方式是加上 swap，每次检查队头是否小于队尾，如果不是的话交换队头和队尾。

直接改一下队列的实现即可。

??? note "Code"
	```cpp
	struct SLF_Swap{
		std::deque<int>dq;
		SLF_Swap(){ dq.clear(); }
		inline void push(int x){
			if(!dq.empty()){
				if(dis[x]<dis[dq.front()]) dq.push_front(x);
				else dq.push_back(x);
				if(dis[dq.front()]>dis[dq.back()]) swap(dq.front(),dq.back());
	            // 这里的两重 if 可以保证只会在至少有两个元素的时候才交换。
			} else dq.push_back(x);
		}
		inline void pop(){
			dq.pop_front();
			if(!dq.empty() && dis[dq.front()]>dis[dq.back()]) swap(dq.front(),dq.back());
		}
		inline int size(){ return dq.size(); }
		inline int front(){ return dq.front(); }
		inline bool empty(){ return !dq.size(); }
	}q;
	```

## Floyd 算法

不能有负环，因为 Floyd 要求最短路必须存在。

但是 Floyd 可以判断负环，

先初始化所有 $dis = +\infty,dis[i][i] = 0$。

只要跑完之后存在 $dis[i][i] < 0$，即存在负环。

考虑动态规划。

设 $dis_{i,j,k}$ 表示从 $i \to j$ ，经过若干个标号不超过 $k$ 的节点的最短路长度。

可以分两个部分转移，第一个部分是 $dis_{i,j,k-1}$，第二个部分是 $i\to k \to j$

前者是直接从 $i \to j$ ，经过节点编号不超过 $k-1$，后者是先从 $i$ 到 $k$ 之后再到 $j$ 。

所以 $dis_{i,j,k}=\min(dis_{i,j,k-1},dis_{i,k,k-1}+dis_{k,j,k-1})$

这里 $k$ 是阶段，所以放在最外层。

发现这玩意儿只和上一层有关，经过分析可以发现，能滚动数组优化，可以直接去掉第三维。

但是方程要稍微改变一下

```cpp
for(register int k=1;k<=n;++k){
    for(register int i=1;i<=n;++i){
        for(register int j=1;j<=n;++j){
            dis[i][j]=min(dis[i][j],dis[i][k]+dis[k][j]);
        }
    }
} // 不要忘记初始化.
```

一般需要用到非板子的Floyd 的时候，都需要考虑使用 $k$ 这个东西的性质。

比如要求最小环，恰好 $X$ 条边的最短路的时候就需要用这个考虑。

更多的时候是把 $k$ 当作**中间**点。

## 最短路的一些扩展应用

或者说是一些比较有意思的技巧。

另外一部分会在题目总结里面提到。

### Floyd 处理传递闭包

> 给你一些元素和一些具有传递性的二元关系，通过传递性推导出更多的关系。

最简单的例子就是 $A<B$ 这种关系。

设 $f_{i,j}$ 表示 $i,j$ 之间是否有这种二元的传递关系。

如果满足 $i < j$ （此处的小于代指二元关系）。

那么 $f_{i,j}=true$，反之 $f_{i,j}=false$。

跑一遍 Floyd 就可以得到所有能推出的关系。

```cpp
for(register int k=1;k<=n;++k){
    for(register int i=1;i<=n;++i){
        for(register int j=1;j<=n;++j){
            f[i][j]|=(f[i][k]&&f[k][j]);
        }
    }
}
```

### Floyd 处理无向图最小环

> 给你一个无向图，求图上的最小环，要求环至少是三元环。

考虑 Floyd 外层循环刚刚开始的时候 $dis_{i,j}$ 是什么。

明显，是：“经过若干个编号不超过 $k-1$ 的节点，由 $i \to j$ 的最短路”。

把 $k$ 当作中间点，从 $i\to j \to k \to i$ 就是一个环。

用式子表达这个就是 $dis_{i,j}+a_{j,k}+a_{k,i}$ （此时还没有对 $k$ 这一层的 $dis$ 进行更新）

对所有的这个式子取最小值，就可以得到答案。

虽然对于每个 $k$ ，这个算法只求的标号不超过 $k$ 的节点构成的最小环，但是之后的 $k$ 是会考虑到的，所以算法是正确的。

如果要输出方案的话，记录 $pos_{i,j}$ 表示使 $dis_{i,j}$ 最后发生更新的 $k$， dp 完之后搞一下递归输出即可。

??? note "Code"
	```cpp
	std::vector<int>ans_path;
	inline void gopath(int u,int v){
	    if(pos[u][v]==0) return;
	    gopath(u,pos[u][v]),ans_path.push_back(pos[u][v]),gopath(pos[u][v],v);
	} // go through the path from u to v;
	
	signed main(){
	    cin>>n>>m; memset(a,0x3f,sizeof a);
	    for(register int i=1;i<=n;++i) a[i][i]=0;
	    for(register int i=1;i<=m;++i){
	        int u,v,w; cin>>u>>v>>w;
	        a[u][v]=min(a[u][v],w),a[v][u]=a[u][v];
	    } memcpy(dis,a,sizeof a); int ans=0x3f3f3f3f3f3f3f3f,tmp=ans;
	    for(register int k=1;k<=n;++k){
	        for(register int i=1;i<k;++i){ // 注意是dp之前，此时 dis 还是 k-1 的时候的状态。
	            for(register int j=i+1;j<k;++j){
	                if(a[j][k]<tmp/2 && a[k][i]<tmp/2 && ans>dis[i][j]+a[j][k]+a[k][i]){
	                    ans=dis[i][j]+a[j][k]+a[k][i];
	                    ans_path.clear(),ans_path.push_back(i),gopath(i,j);
	                    ans_path.push_back(j),ans_path.push_back(k);
	                } // 不判的话 a[j][k]+a[k][i] 有可能爆，导致答案出错。
	            }
	        }  // 更新最小环取min的过程
	        for(register int i=1;i<=n;++i){
	            for(register int j=1;j<=n;++j){
	                if(dis[i][j]>dis[i][k]+dis[k][j]) pos[i][j]=k,dis[i][j]=dis[i][k]+dis[k][j];
	            }
	        } // 正常的 Floyd
	    } if(ans==0x3f3f3f3f3f3f3f3f) return puts("No solution."),0;
	    for(auto x:ans_path) cout<<x<<" ";
	    return puts(""),0;
	}
	```

### 经过恰好 K 条边最短路

首先用邻接矩阵 $A$ 存图。

然后 $A[i,j]$ 就可以看做 $i \to j$ 经过恰好一条边的最短路。

考虑求出经过恰好两条边的最短路 $B$。

可以发现 $B[i,j]=\min\limits_{1\le k \le n}\{A[i,k]+A[k,j]\}$

这里就是用了类似 Floyd 的枚举中间点思想。

类似的可以得到经过恰好 $K$ 条边的最短路。

设 $A^{qwq}$ 表示经过恰好 $qwq$ 条边的最短路。

可以得到 $A^{qwq}[i,j]=\min\limits_{1\le k \le n}\{A^p[i,k]+A^q[k,j]\},qwq=p+q$

发现这玩意儿就是个类似于矩阵乘法的东西。

把 $\sum$ 换成 $\min$ ，把 $\times$ 换成 $+$。

刚好这个东西仍然满足结合律，所以可以用矩阵快速幂 $n^3\log K$ 求 $A^K$。

## 最短路计数

这个玩意儿 Floyd，SPFA，Dijkstra 都是可以做的。

在跑最短路的时候记录一个数组 $cnt[v]$ 表示从 $s \to v$ 的最短路径条数。

然后每次最短路被更新的时候就更新 $cnt$。

额外的，如果最短路长度没有被更新，但是三角不等式中的 $>$ 变成了 $=$，

那么给 $cnt$ 加上当前转移过来的点的 $cnt$。

这东西和DAG上的路径计数比较像，只不过有最短路的限制。

当然，路径条数这个玩意儿是指数级别的，一般都会要求取模。

$Floyd$ 的实现略微有点不同，因为是枚举中间点，所以需要再用一次乘法原理。

??? note "Code"
	```cpp
	Floyd:
	==============================================
	memset(dis, 0x3f, sizeof dis);
	memset(cnt, 0, sizeof cnt);
		
	cin >> n >> m;
	for(int i = 1; i <= m; ++i) {
		int u, v, w;
		cin >> u >> v >> w;
	    dis[u][v] = dis[v][u] = min(dis[u][v], w);
		cnt[u][v] = cnt[v][u] = 1;
	}
	
	for(int i = 1; i <= n; ++i) dis[i][i] = 0;
	
	for(int k = 1; k <= n; ++k) {
		for(int i = 1; i <= n; ++i) {
			for(int j = 1; j <= n; ++j) {
				if(dis[i][j] > dis[i][k] + dis[k][j]) {
					dis[i][j] = dis[i][k] + dis[k][j];
					cnt[i][j] = 1ll * cnt[i][k] * cnt[k][j];
				}
				else if(dis[i][j] == dis[i][k] + dis[k][j]) {
					cnt[i][j] += 1ll * cnt[i][k] * cnt[k][j];
				}
	            // 乘法原理计数。
			}
		}
	}
	
	Dijkstra:
	==============================================
	void dijkstra(int s) {
		memset(vis, false, sizeof vis);	
		memset(dis, 0x3f, sizeof dis);
		q.push({dis[s] = 0, s}), cnt[1] = 1;
	
		while(!q.empty()) {
			int u = q.top().second;
			q.pop();
			if(vis[u]) continue;
			vis[u] = true;
			for(int i = head[u]; ~i; i = e[i].Next) {
				int v = e[i].ver, w = e[i].w;
				if(dis[v] > dis[u] + w) {
					dis[v] = dis[u] + w;
					cnt[v] = cnt[u] % mod;
					q.push({-dis[v], v});
				}
				else if(dis[v] == dis[u] + w) {
					cnt[v] = (cnt[v] + cnt[u]) % mod;
				} 
			}
		}
	}
	
	int main() {
		memset(head, -1, sizeof head);
		memset(cnt, 0, sizeof cnt);
	
		cin >> n >> m;
		for(int i = 1; i <= m; ++i) {
			int u, v;
			cin >> u >> v;
			add(u, v), add(v, u);
		}
	
		dijkstra(1);
	
		for(int i = 1; i <= n; ++i) {
			cout << cnt[i] << endl;
		}
		return 0;
	}
	```

