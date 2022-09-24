## 二月 Tricks 整理

### Acwing97 约数之和

> 求 $A^B$ 的约数之和，要取模。

把 $A$ 分解质因数，然后指数乘上 $B$ 可以得到以下式子：

$sum=(1+p_1+p_1^2+p_1^3+\dots+p_1^{B\times c_1})\times(1+p_2+p_2^2+p_2^3+\dots+p_2^{B\times c_2})\times\dots \times (1+p_n+p_n^2+p_n^3+\dots+p_n^{B\times c_n})$

$p_i$ 表示 $A$ 的第 $i$ 个质因子，$c_i$ 表示 $p_i$ 头上的指数。

每一个小括号里面可以用等比数列求和，但是取模（只）对于除法没有分配律，分子分母不能分开取模然后除。

所以考虑用分治法求每个小括号里的内容然后相乘取模。

把每个式子从中间拆开，然后把后面大一点的式子提取一个公因数出来使得它和前面的式子一样，然后就可以写成递归了。

??? note "Code"
	```cpp
	/*
	 * @Author: black_trees 
	 * @Date: 2022-01-29 10:12:50 
	 * @Last Modified by: black_trees
	 * @Last Modified time: 2022-01-29 10:20:59
	 */
	
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	const int mod=9901;
	const int N=5e6+7;
	int a,b,q[N],f[N],ans,tot,cnt;
	inline int qpow(int a,int b,int p){
	    int res=1%p;
	    for(;b;b>>=1){
	        if(b&1) res=res*a%p;
	        a=a*a%p;
	    } return res;
	}
	inline void split(){
	    for(register int i=2;i*i<=a;i++){
	        if(a%i==0){
	            q[++cnt]=i;
	            while(a%i==0) f[cnt]++,a/=i;
	        }
	    } if(a>1) q[++cnt]=a,f[cnt]++;
	}
	inline void solve(){
	    if(a==0){ puts("0"); return; }
	    if(b==0){ puts("1"); return; }
	    split(),ans=1;
	    for(register int i=1;i<=cnt;i++){
	        if((q[i]-1)%mod==0){
	            ans=(ans*(b*f[i]+1)%mod)%mod;
	            continue;
	        }
	        int x=qpow(q[i],b*f[i]+1,mod);x=(x-1+mod)%mod;
	        int y=qpow(q[i]-1,mod-2,mod);ans=(ans*x*y)%mod;
	    } printf("%lld",ans); return;
	}
	signed main(){
	    #ifndef ONLINE_JUDGE
	        freopen("Input.txt","r",stdin);
	        freopen("Output.txt","w",stdout);
	    #endif
	    cin>>a>>b; return solve(),0;
	}
	
	```

!!! Tricks
	取模时可以利用分治法对等比数列求和。

```cpp
Tag : 分治/等比数列
```

### Acwing11 背包问题求方案数

求方案数，先背包，然后 dp 的时候记录一个 $g$ 数组，表示使用不超过当前的空间的方案有多少种。

根据 $f$ 转移的两种情况分开讨论就行。

不知道为啥，我写个 $tmp$ 存 $f$ 没转移的 $val$ 是错的。

但是先比较，记录 $g$ 之后再更新 $f$ 就是对的。

??? note "Code"
	```cpp
	/*
	 * @Author: black_trees 
	 * @Date: 2022-02-02 16:02:50 
	 * @Last Modified by: black_trees
	 * @Last Modified time: 2022-02-02 17:18:10
	 */
	
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	constexpr int si=1e3+10;
	constexpr int p=1e9+7;
	int n,m;
	int v[si],w[si];
	int f[si],g[si];
	
	signed main(){
	    #ifndef ONLINE_JUDGE
	        freopen("Input.txt","r",stdin);
	        freopen("Output.txt","w",stdout);
	    #endif
	    scanf("%lld%lld",&n,&m);
	    for(register int i=1;i<=n;++i){
	        scanf("%lld%lld",&v[i],&w[i]);
	    }
	    for(register int i=0;i<=m;++i){
	        g[i]=1;
	    }
	    for(register int i=1;i<=n;++i){
	        for(register int j=m;j>=v[i];--j){
	            int tmp=f[j-v[i]]+w[i];
	            if(tmp>f[j]) f[j]=tmp,g[j]=g[j-v[i]];
	            else if(tmp==f[j]) g[j]=(g[j]+g[j-v[i]])%p;
	        }
	    } return printf("%lld\n",g[m]%p),0;
	}
	```

!!! Trick
	DP 类问题求方案数时，需要在转移成功的时候记录方案。

	同时需要在当前状态和转移过来的状态的 DP 值相等时，将两个的方案累加。


```cpp
Tag : 背包/方案数
```

### Acwing529 [NOIP2017] 宝藏

发现本题的要求就是求出一颗生成树，使得按照要求得到的权值和最小。

并且，题目中要求的“从最初的宝藏屋到当前道路的起点经过的宝藏屋个数”，就是以最初的宝藏屋为root，到当前道路起点的深度。

设 $f[i,msk]$ 表示集合：当前打通的最大深度为 $i$ ，打通了状态为 $msk$ 的宝藏屋的所有方案。属性为权值和的 Min。

那么考虑枚举**上一层**（按照“最后”进行集合划分）的状态 $qwq$，

那么可以得到一个转移方程：$f_{i,msk}=\min\{f_{i,msk},f_{i-1,qwq}+(i-1)\times cost(qwq,msk)\}$

其中 $cost(qwq,msk)$ 表示从 $qwq$ 转移到 $msk$ 增加的道路长度的最小值。

但是这里要求只能扩展一层，所以还要判 $qwq$ 是否能只通过一层扩展得到 $msk$。

具体怎么判定？

1.  $qwq$ 一定要是 $msk$ 的子集。
2. $msk$ 当中的所有元素一定处于集合 $S_{qwq}$，其中 $S_{qwq}$ 表示 $qwq$ 往下扩展一层之后所的到的所有新节点和 $qwq$ 本身组成的集合。

这个可以预处理做。

然后考虑如何处理 $cost$。

当预处理出所有的可行状态的对之后，我们记 $r(msk,i)$ 表示从状态 $msk$ 所有的的节点出发，扩展到节点 $i$ 的最短路。

这个可以在处理可行状态的时候转移出来。

然后考虑枚举所有可行对的状态之间的差异，比如当前是 $i,j$ 这两个状态，只需要枚举他们之前的差异，然后对相应的 $r$ 求和即可。

那么就可以进行转移了，初始化让每个节点分别为根，让对应状态的权值为 $0$，其他的赋值为 $+\infty$ 即可。

??? note "Code"
	```cpp
	/*
	 * @Author: black_trees <black_trees@foxmail.com>
	 * @Date: 2022-02-08 10:59:15
	 * @Last Modified by: black_trees <black_trees@foxmail.com>
	 * @Last Modified time: 2022-02-08 11:21:12
	 */
	
	#include<bits/stdc++.h>
	using namespace std;
	
	constexpr int inf=0x3f3f3f3f;
	constexpr int si=15;
	constexpr int stasi=1<<13;
	int n,m;
	int a[si][si];
	int f[si][stasi];
	int all[stasi],road[stasi][si];
	std::vector<int>valid[stasi],cost[stasi];
	
	int main(){
		scanf("%d%d",&n,&m),memset(a,0x3f,sizeof a);
		for(register int i=1;i<=m;++i){
			int u,v,w;
			scanf("%d%d%d",&u,&v,&w);
			a[u][v]=a[v][u]=min(a[u][v],w);
		} memset(road,0x3f,sizeof road);
		for(register int msk=0;msk<(1<<n);++msk){
			all[msk]=msk;
			for(register int i=1;i<=n;++i){
				if(msk>>(i-1)&1){
					road[msk][i]=0;
					for(register int j=1;j<=n;++j){
						if(a[i][j]==inf) continue;
						all[msk]|=1<<(j-1),road[msk][j]=min(road[msk][j],a[i][j]);
					}
				}
			}
		}
		for(register int msk=0;msk<(1<<n);++msk){
			for(register int mks=0;mks<msk;++mks){
				if((msk&mks)==mks && (msk&all[mks])==msk){
					valid[msk].push_back(mks); int su=0;
					for(register int i=1;i<=n;++i){
						if((msk^mks)>>(i-1)&1) su+=road[mks][i];
					} cost[msk].push_back(su);
				}
			}
		} memset(f,0x3f,sizeof f);
		for(register int i=1;i<=n;++i) f[1][1<<(i-1)]=0;
		int res=f[1][(1<<n)-1];
		for(register int i=2;i<=n;++i){
			for(register int msk=1;msk<(1<<n);++msk){
				for(register int j=0;j<(int)valid[msk].size();++j){
					int k=valid[msk][j];
					f[i][msk]=min(f[i][msk],f[i-1][k]+(i-1)*cost[msk][j]);
				}
			} res=min(res,f[i][(1<<n)-1]);
		} return printf("%d\n",res),0;
	}
	
	```

!!! Trick
	在思考DP类的问题时可以先列出一个方程，不管里面的某些东西怎么求，等确定式子是对的（无后效性）之后再考虑计算这些东西。（逐层思考）

```cpp
Tag : 生成树/状压DP
```

### Acwing1073 树的中心

暴力做就是每个点 dfs 然后取 max。

但是明显 TLE，所以考虑换根DP。

第一次 dfs 处理出以 $1$ 为根的时候，每一个节点能往下走的距离的最大值和次大值。然后考虑处理出从每个节点往上走的最大值。

第二次 dfs 分类讨论进行转移即可。

统计的时候，如果最大值被更新，那么把次大值更新为原来的最大值。

反之令次大值和当前扫到的元素取 $\max$ 。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	constexpr int si_n=1e4+10;
	constexpr int si_m=si_n<<1;
	int n;
	struct node{
		int head,ver,Next,w;
	}e[si_m];
	int tot=0;
	inline void add(int u,int v,int w){
		e[++tot].ver=v,e[tot].w=w;
		e[tot].Next=e[u].head,e[u].head=tot;
	}
	int d[si_n],dis[si_n],up[si_n];
	int s[si_n];
	inline void dfs1(int u,int fa){
		for(register int i=e[u].head;i;i=e[i].Next){
			int v=e[i].ver,w=e[i].w;
			if(v==fa) continue;
			dfs1(v,u);
			if(d[v]+w>=d[u]) dis[u]=d[u],d[u]=d[v]+w,s[u]=v;
			else if(d[v]+w>dis[u]) dis[u]=d[v]+w;
		}
	}
	inline void dfs2(int u,int fa){
		for(register int i=e[u].head;i;i=e[i].Next){
			int v=e[i].ver,w=e[i].w;
			if(v==fa) continue;
			if(s[u]==v) up[v]=w+max(up[u],dis[u]);
			else up[v]=w+max(up[u],d[u]);
			dfs2(v,u);
		}
	}
	
	int main(){
		scanf("%d",&n);
		for(register int i=1;i<n;++i){
			int u,v,w;
			scanf("%d%d%d",&u,&v,&w);
			add(u,v,w),add(v,u,w);
		} dfs1(1,0),dfs2(1,0);
		int res=0x3f3f3f3f;
		for(register int i=1;i<=n;++i){
			res=min(res,max(d[i],up[i]));
		} return printf("%d\n",res),0;
	}
	```

!!! Tricks
	换根 DP 的方程如果出现 $\max$，换根的时候需要利用最大值和次大值进行转移。

	可以利用 `std::multiset`，也可以用 `std::vector` 记录。

```cpp
Tag : 换根DP
```

### CF708C Centroids

> 给定一颗树，你有一次将树改造的机会，改造的意思是删去一条边，再加入一条边，保证改造后还是一棵树。
>
> 请问有多少点可以通过改造，成为这颗树的重心？（如果以某个点为根，每个子树的大小都不大于 $\dfrac{n}{2}$ ，则称某个点为重心）

首先可以发现，如果一个节点 $u$ 不是重心，那么它的子树当中有且仅有一个的大小会超出 $\lfloor \frac{n}{2} \rfloor$。

那么我们改造的时候就一定是从这个超出 $\lfloor \frac{n}{2} \rfloor$ 的子树当中找到一个子树，使它和 $u$ 相连，满足以 $u$ 为根的所有子树都不大于 $\lfloor \frac{n}{2} \rfloor$。

并且这个找出来的子树大小必须是 $\le \lfloor \frac{n}{2} \rfloor$ 当中最大的一个，这样才能保证 $u$ 被改造为重心之后不可能出现矛盾。

所以我们只需要求出每一个子树当中的，小于等于 $\lfloor \frac{n}{2} \rfloor$ 的最大子树的大小就能快速进行判断了。

先考虑不换根的情况。

设 $f_{u}$ 表示 $u$ 的子树当中的，小于等于 $\lfloor \frac{n}{2} \rfloor$ 的最大子树的大小。

那么可以很容易的得到转移方程：

$f_{u}=\begin{cases} \max\{siz_v\} & siz_v \le \lfloor \frac{n}{2} \rfloor\\ \max\{f_{v}\} & \text{otherwise.} \end{cases}$

考虑换根，发现方程当中有出现 $\max$，所以我们需要记录 $f$ 的最大值和次大值。

这个方程比较套路，就是在原来的 $f$ 的基础上和 $n-siz_u$ 取 $\max$，然后注意最大值和次大值的讨论就可以。

??? note "Code"

	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	constexpr int si=1e6+10;
	constexpr int inf=0x3f3f3f3f;
	int t,n;
	int dp[si][2],pos[si];
	int dpp[si],ans[si];
	int siz[si],maxsiz[si];
	struct edge{
		int to,Next; 
	}e[si<<1];
	int head[si<<1];
	int tot=0;
	inline void add(int u,int v){
		e[++tot].to=v,e[tot].Next=head[u];
		head[u]=tot;
	} 
	inline void dfs1(int u,int fa){
		siz[u]=1;
		for(register int i=head[u];i;i=e[i].Next){
			if(e[i].to==fa) continue;
			dfs1(e[i].to,u);
			int v;
			siz[u]+=siz[e[i].to];
			if(siz[e[i].to]>siz[maxsiz[u]]) maxsiz[u]=e[i].to;
			if(siz[e[i].to]<=n/2) v=siz[e[i].to];
			else v=dp[e[i].to][0];
			if(dp[u][0]<v) dp[u][1]=dp[u][0],dp[u][0]=v,pos[u]=e[i].to;
			else if(dp[u][1]<v) dp[u][1]=v;
		}
	}
	inline void dfs2(int u,int fa){
		ans[u]=1;
		if(siz[maxsiz[u]]>n/2) ans[u]=(siz[maxsiz[u]]-dp[maxsiz[u]][0]<=n/2);
		else if(n-siz[u]>n/2) ans[u]=(n-siz[u]-dpp[u]<=n/2);
		for(register int i=head[u];i;i=e[i].Next){
			if(e[i].to==fa) continue;
			int v;
			if(n-siz[u]>n/2) v=dpp[u];
			else v=n-siz[u];
			dpp[e[i].to]=max(dpp[e[i].to],v);
			if(pos[u]==e[i].to) dpp[e[i].to]=max(dpp[e[i].to],dp[u][1]);
			else dpp[e[i].to]=max(dpp[e[i].to],dp[u][0]);
			dfs2(e[i].to,u);
		}
	}
	signed main(){
		scanf("%d",&n);
		for(register int i=1;i<n;i++){
			int u,v; scanf("%d%d",&u,&v);
			add(u,v),add(v,u);
		} dfs1(1,0),dfs2(1,0);
		for(register int i=1;i<=n;i++){
			printf("%d ",ans[i]);
		} return 0;
	}
	
	```

!!! Trick
	先考虑有根的情况列出方程，然后再从换根的角度理解。

```cpp
Tag : 换根DP
```

### Acwing340 通信线路

> 可以将图中某条 $1\to n$ 的路径上的 $K$ 个边权设置为 $0$。
>
> 定义这条路径的花费为设置后路径上所有边权的最大值。
>
> 求花费最小值。

可以分层图最短路。

设 $dis[x,p]$ 表示从 $1$ 到 $x$，已经用了 $p$ 条免费机会的最小花费。

那么考虑一个“广义”的最短路，将每个节点都拆成 $K + 1$ 个（$(x,0),(x,1),\dots,(x,K)$）

同层之间直接连原来的边权，然后这一层和下一层连边就将边权设置为 $0$。

具体来说，对于 $(u,v)$ 这条原图中的边，连接 $(u,k),(v,k)$，边权为 $w(u,v)$。

然后连接 $(u,k),(v,k + 1)$，表示这条线路使用免费机会，所以边权为 $0$。

在这张图上跑一个最短路，最后答案在 $dis[n][i]$ 中取 $\min$ 即可。

实际上就是利用 SPFA DP。

或者利用 $01$ 二分，每次把不超过 $mid$ 的设置为 $0$ ，其它为 $1$。

然后利用 dijkstra Check $dis[n]$ 是否小于等于 $K$ 即可。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	constexpr int si=11010;
	constexpr int si_e=si<<1;
	int n,p,k;
	int tot=0,dis[si];
	bool vis[si];
	struct edge{ int head,next,ver,w; }e[si_e];
	inline void add(int u,int v,int w){ e[++tot].ver=v,e[tot].next=e[u].head,e[u].head=tot,e[tot].w=w; }
	std::priority_queue<pair<int,int>>q;
	inline bool dijkstra(int limit){
		memset(dis,0x3f,sizeof dis),memset(vis,false,sizeof vis);
		dis[1]=0,q.push({dis[1],1});
		while(!q.empty()){
			int u=q.top().second; q.pop();
			if(vis[u]) continue; vis[u]=true;
			for(register int i=e[u].head;i;i=e[i].next){
				int v=e[i].ver,w=e[i].w>limit?1:0;
				if(dis[v]>dis[u]+w) dis[v]=dis[u]+w,q.push({-dis[v],v});	
			}
		} return dis[n]<=k;
	}
	
	int main(){
		cin>>n>>p>>k; for(register int i=1,u,v,w;i<=p;++i) cin>>u>>v>>w,add(u,v,w),add(v,u,w);
		int l=0,r=1e6+1;
		while(l<r){
			int mid=(l+r)>>1;
			if(dijkstra(mid)) r=mid;
			else l=mid+1;
		} if(r==1e6+1) return puts("-1"),0;
		return printf("%d\n",r),0;
	}
	
	```

!!! Tricks
	1. 把最优问题转化成判定问题时，可以考虑利用 01 二分的思想更方便的 Check。

	2. 在无向图上 DP 的时候，因为转移可能有后效性，所以可以借助 SPFA 进行 DP。

	   此处会有后效性的原因有两个：
	   		1. 阶段并不是节点的编号顺序。

	   		2. 因为是无向图，无法像 DAG 那样很好的进行无后效性的 DP 转移。

	3. 可以把图上的节点根据不同的情况拆成多个节点，然后在新图上根据条件连边，问题就可以被转化为最普通的 SSSP。

```cpp
Tag : SPFA/01二分/分层图
```

### Acwing1148 秘密的牛奶运输

> 求无向联通带权图 $G$ 的严格次小生成树。

首先考虑非严格次小生成树怎么做。

先求出最小生成树，然后枚举每一条非树边 $(u,v)$，把这条非树边加入到最小生成树当中。

这时候必然会出现一个环，所以把这个环上除去 $w(u,v)$ 的最大值删去就可以了。

明显的，这个环上的最大值就是 MST 当中 $u \to v$ 的路径上的最大值。

可以在 MST 上用倍增求 LCA，然后记录 $u$ 到它的 $2^i$ 级祖先路径上的边权的严格最大值。

这个类似 $f$ 数组的记录，初始化 $w[u][0] = w(u,fa)$，然后倍增记录。

查询 MST 上 $(u,v)$ 路径上的最大值时，分别询问 $(u,lca),(v,lca)$ 路径上的严格最大值即可。

答案就是枚举完所有非树边被加入的情况的最小值。

然后发现这里和严格次小唯一的区别就是可能加入的边等于被删除的边的权值。

所以用类似换根 DP 里的 trick。

记录最大值的同时记录次大值，如果最大值等于被加入的非树边的权值，使用次大值代替最大值更新即可。

??? note "Code"
	```cpp
	#include <algorithm>
	#include <iostream>
	
	const int INF = 0x3fffffff;
	const long long INF64 = 0x3fffffffffffffffLL;
	
	struct Edge {
	  int u, v, val;
	
	  bool operator<(const Edge &other) const { return val < other.val; }
	};
	
	Edge e[300010];
	bool used[300010];
	
	int n, m;
	long long sum;
	
	class Tr {
	 private:
	  struct Edge {
	    int to, nxt, val;
	  } e[600010];
	
	  int cnt, head[100010];
	
	  int pnt[100010][22];
	  int dpth[100010];
	  // 到祖先的路径上边权最大的边
	  int maxx[100010][22];
	  // 到祖先的路径上边权次大的边，若不存在则为 -INF
	  int minn[100010][22];
	
	 public:
	  void addedge(int u, int v, int val) {
	    e[++cnt] = (Edge){v, head[u], val};
	    head[u] = cnt;
	  }
	
	  void insedge(int u, int v, int val) {
	    addedge(u, v, val);
	    addedge(v, u, val);
	  }
	
	  void dfs(int now, int fa) {
	    dpth[now] = dpth[fa] + 1;
	    pnt[now][0] = fa;
	    minn[now][0] = -INF;
	    for (int i = 1; (1 << i) <= dpth[now]; i++) {
	      pnt[now][i] = pnt[pnt[now][i - 1]][i - 1];
	      int kk[4] = {maxx[now][i - 1], maxx[pnt[now][i - 1]][i - 1],
	                   minn[now][i - 1], minn[pnt[now][i - 1]][i - 1]};
	      // 从四个值中取得最大值
	      std::sort(kk, kk + 4);
	      maxx[now][i] = kk[3];
	      // 取得严格次大值
	      int ptr = 2;
	      while (ptr >= 0 && kk[ptr] == kk[3]) ptr--;
	      minn[now][i] = (ptr == -1 ? -INF : kk[ptr]);
	    }
	
	    for (int i = head[now]; i; i = e[i].nxt) {
	      if (e[i].to != fa) {
	        maxx[e[i].to][0] = e[i].val;
	        dfs(e[i].to, now);
	      }
	    }
	  }
	
	  int lca(int a, int b) {
	    if (dpth[a] < dpth[b]) std::swap(a, b);
	
	    for (int i = 21; i >= 0; i--)
	      if (dpth[pnt[a][i]] >= dpth[b]) a = pnt[a][i];
	
	    if (a == b) return a;
	
	    for (int i = 21; i >= 0; i--) {
	      if (pnt[a][i] != pnt[b][i]) {
	        a = pnt[a][i];
	        b = pnt[b][i];
	      }
	    }
	    return pnt[a][0];
	  }
	
	  int query(int a, int b, int val) {
	    int res = -INF;
	    for (int i = 21; i >= 0; i--) {
	      if (dpth[pnt[a][i]] >= dpth[b]) {
	        if (val != maxx[a][i])
	          res = std::max(res, maxx[a][i]);
	        else
	          res = std::max(res, minn[a][i]);
	        a = pnt[a][i];
	      }
	    }
	    return res;
	  }
	} tr;
	
	int fa[100010];
	
	int find(int x) { return fa[x] == x ? x : fa[x] = find(fa[x]); }
	
	void Kruskal() {
	  int tot = 0;
	  std::sort(e + 1, e + m + 1);
	  for (int i = 1; i <= n; i++) fa[i] = i;
	
	  for (int i = 1; i <= m; i++) {
	    int a = find(e[i].u);
	    int b = find(e[i].v);
	    if (a != b) {
	      fa[a] = b;
	      tot++;
	      tr.insedge(e[i].u, e[i].v, e[i].val);
	      sum += e[i].val;
	      used[i] = 1;
	    }
	    if (tot == n - 1) break;
	  }
	}
	
	int main() {
	  std::ios::sync_with_stdio(0);
	  std::cin.tie(0);
	  std::cout.tie(0);
	
	  std::cin >> n >> m;
	  for (int i = 1; i <= m; i++) {
	    int u, v, val;
	    std::cin >> u >> v >> val;
	    e[i] = (Edge){u, v, val};
	  }
	
	  Kruskal();
	  long long ans = INF64;
	  tr.dfs(1, 0);
	
	  for (int i = 1; i <= m; i++) {
	    if (!used[i]) {
	      int _lca = tr.lca(e[i].u, e[i].v);
	      // 找到路径上不等于 e[i].val 的最大边权
	      long long tmpa = tr.query(e[i].u, _lca, e[i].val);
	      long long tmpb = tr.query(e[i].v, _lca, e[i].val);
	      // 这样的边可能不存在，只在这样的边存在时更新答案
	      if (std::max(tmpa, tmpb) > -INF)
	        ans = std::min(ans, sum - std::max(tmpa, tmpb) + e[i].val);
	    }
	  }
	  // 次小生成树不存在时输出 -1
	  std::cout << (ans == INF64 ? -1 : ans) << '\n';
	  return 0;
	} // from OI-wiki
	```

!!! Tricks
	1. 先思考非严格，再考虑严格（特殊 $\to$ 一般）

 	2. 非严格 $\to$ 严格 的过程就是去掉相等，在这种最值问题里可以利用记录次最值来解决。

```cpp
Tag : 严格次小/生成树/LCA/倍增
```

