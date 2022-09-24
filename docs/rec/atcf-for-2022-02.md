## 二月好题改错

### CF1638E Colorful Operations

`Feb/22/2022`

> 给你一个序列 $a$，支持三种操作
> + `0 l r v` 将 $[l,r]$ 涂成颜色 $v$
> + `1 v x` 将所有颜色 $v$ 的元素加上 $x$
> + `2 x` 查询 $a_x$ 
>
> 1e6.

教会我ODT的题。

首先可以直接利用 ODT 维护每个段。

操作0就是 `assign`

对于操作1，考虑对每个颜色维护一个 $tag$ 。

表示这个颜色当前一共加了多少。

然后 `assign` 的时候，对于每一个被改变颜色的块，先利用树状数组把他的值加上 $tag$ ，推平之后还要减去新颜色的 $tag$。

操作 2 利用树状数组 + 差分询问之后再加上当前颜色的 $tag$ 即可。 

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	constexpr int si=1e6+10;
	int n,q;
	int a[si];
	int tag[si];
	
	inline int lowbit(int x){ return x&-x; }
	struct BIT{
		int	n,c[si],ans=0;
		inline void modify(int x,int y){ for(register int i=x;i<=n;i+=lowbit(i)) c[i]+=y;}
		inline void add(int l,int r,int v){ modify(l,v),modify(r+1,-v); }
		inline int query(int x){ans=0; for(register int i=x;i;i-=lowbit(i)) ans+=c[i]; return ans; }
	}bitr;
	
	struct node{
		int l,r;
		mutable int val;
		node(const int &il,const int &ir,const int &iv):l(il),r(ir),val(iv){}
		inline bool operator < (const node &b)const{ return l<b.l; }
	}; std::set<node>odt;
	inline std::set<node>::iterator split(int pos){
		if(pos>n) return odt.end();
		std::set<node>::iterator it=--odt.upper_bound((node){pos,0,0});
		if(it->l==pos) return it;
		int l=it->l,r=it->r,v=it->val;
		odt.erase(it),odt.insert(node{l,pos-1,v});
		return odt.insert((node){pos,r,v}).first;
	}
	inline void assign(int l,int r,int v){
		std::set<node>::iterator itr=split(r+1),itl=split(l);
		for(std::set<node>::iterator it=itl;it!=itr;++it){
			bitr.add(it->l,it->r,tag[it->val]);	
		} odt.erase(itl,itr),odt.insert((node){l,r,v});
		bitr.add(l,r,-tag[v]);
	}
	inline int Tag(int pos){
		std::set<node>::iterator it=odt.lower_bound(node{pos,0,0});
		if(it!=odt.end() && it->l==pos) return tag[it->val];
		--it; return tag[it->val];
	}
	
	signed main(){
		scanf("%lld%lld",&n,&q),odt.insert((node){1,n,1});
		bitr.n=n; string op; int l,r,c;
		while(q--){	cin>>op;
			if(op=="Color") scanf("%lld%lld%lld",&l,&r,&c),assign(l,r,c);
			if(op=="Add") scanf("%lld%lld",&l,&c),tag[l]+=c;
			if(op=="Query") scanf("%lld",&c),printf("%lld\n",Tag(c)+bitr.query(c));
		} return 0;	
	}
	
	```

```cpp
Tag : ODT/树状数组/差分
```

### CF1635E Cars

`Feb/23/2022`

> 在一个数轴上有一些车，每个车都有初始的位置和方向，方向不会改变，开始运动后速度不变。
> 称两辆车相关，当且仅当他们一定会相遇，
> 称两辆车不相关，当且仅当他们一定不会相遇。
> 给你 $m$ 组车之间相关不相关的关系，构造一个初始方案，无解输出-1。
>
> 2e5.

发现两辆车想要有关系，必须方向相反。

这里用二分图染色判定即可（需要考虑不联通的情况）

然后我们考虑任意一对车 $(u,v)$

假设 $u$ 的方向是 Right，$v$ 的方向是 Left。

那么如果 $(u,v)$ 相关，那么 $u$ 在数轴上面的位置必定小于 $v$

如果不相关， $u$ 在数轴上面的位置必定大于 $v$

第一种关系记作 $u \to v$

第二种关系记作 $v\to u$

这是一个严格偏序关系，所以可以联系到 DAG，使用拓扑排序判断这个严格偏序关系是否成立即可。

初始位置用拓扑序求解。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	constexpr int si=2e5+10;
	int n,m;
	std::vector<int>GO[si],G[si];
	int col[si],ord[si],x[si];
	bool vis[si],have_sol=true;
	struct node{ int ty,u,v; }a[si];
	inline void dfs(int u,int cur){
		col[u]=cur,vis[u]=true;
		for(int &v:GO[u]){
			if(vis[v] && col[v]==cur) have_sol=false;
			if(vis[v]) continue; dfs(v,cur==1?2:1);
		}
	}
	
	signed main(){
		scanf("%lld%lld",&n,&m);
		for(register int i=1;i<=m;++i){
			scanf("%lld%lld%lld",&a[i].ty,&a[i].u,&a[i].v);
			GO[a[i].u].push_back(a[i].v),GO[a[i].v].push_back(a[i].u);
		} for(register int i=1;i<=n;++i) if(!vis[i]) dfs(i,1);
		if(!have_sol) return puts("NO"),0;
		for(register int i=1;i<=m;++i){
			int &u=a[i].u,&v=a[i].v;
			if(a[i].ty==1){ if(col[u]==1) G[u].push_back(v),++ord[v]; else G[v].push_back(u),++ord[u]; }
			else{ if(col[u]==1) G[v].push_back(u),++ord[u]; else G[u].push_back(v),++ord[v]; }
		} std::queue<int>q; for(register int i=1;i<=n;++i) if(ord[i]==0) q.push(i);
		int tot=0; while(!q.empty()){
			int u=q.front(); q.pop(),x[u]=++tot;
			for(int &v:G[u]) if(!--ord[v]) q.push(v);
		} if(tot!=n) return puts("NO"),0;
		puts("YES"); for(register int i=1;i<=n;++i){
			if(col[i]==1) putchar('L'); else putchar('R'); 
			printf(" %lld\n",x[i]);
		} return 0;
	}
	```

!!! Trick
	严格偏序关系可以利用拓扑排序来判定/求解。

参考 <https://zhuanlan.zhihu.com/p/470044525>

```cpp
Tag : 拓扑排序/二分图/偏序关系
```

### CF1635F Closest Pair

`Feb/25/2022`

> 给你两个序列 $x,w$ ,定义一个点对 $(i,j)$ 的权值为 $|x_i-x_j|\times (w_i+w_j)$
> $q$ 组询问，每次询问区间 $[l,r]$ 当中的点对权值的最小值。
>
> 3e5.

考虑把 $x$ 当作 $x$ 轴，$w$ 当作 $y$ 轴。

那么把每个点 $(x_i,w_i)$ 放到坐标系里面。

如果两个点形成的正方形当中有别的点，那么这两个点必定不是最优的。

形式化的，如果对于一个点对 $(i,j)$ ，$\exist k \ | \ x_i<x_k<x_y \ \land \ w_i<w_k<w_j$

那么 $(i,j)$ 必定不是最优的。

另外一边的情况一样。

所以对于每一个 $i$ ，利用单调栈向后处理出所有的可能最优点对 $(i,j)$。

然后把这个点对看作是一个答案区间 $[i,j]$，权值就是这个点对的贡献。

那么离线之后，对于每个询问，就是求询问的区间当中包含（或者交）的答案区间的权值的最小值。

可以使用 Fenwick Tree 或者 Segment Tree 维护。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	constexpr int si=3e5+10;
	constexpr int inf=4e18+1;
	int n,q;
	struct Fenwick{
		int a[si];
		Fenwick(){memset(a,0x7f,sizeof a);}
		inline int lowbit(int x){ return x&-x; }
		inline void modify(int x,int v){x=n-x+1;while(x<=n)a[x]=min(a[x],v),x+=lowbit(x);}
		inline int sum(int x){int ans=inf;x=n-x+1;while(x)ans=min(ans,a[x]),x-=lowbit(x);return ans;}	
	}tr;
	int x[si],w[si];
	int Stack[si],Top;
	std::vector<std::pair<int,int>>a[si];
	int ans[si];
	
	signed main(){
		cin>>n>>q; for(register int i=1;i<=n;++i) cin>>x[i]>>w[i];
		for(register int i=1;i<=q;++i){
			int l,r; cin>>l>>r; a[r].push_back(make_pair(l,i));
		} for(register int i=1;i<=n;++i){
			while(Top && w[i]<=w[Stack[Top]]) tr.modify(Stack[Top],(x[i]-x[Stack[Top]])*(w[i]+w[Stack[Top]])),--Top;
			if(Top) tr.modify(Stack[Top],(x[i]-x[Stack[Top]])*(w[i]+w[Stack[Top]]));
			Stack[++Top]=i; for(auto x:a[i]) ans[x.second]=tr.sum(x.first);
		} for(register int i=1;i<=q;++i) cout<<ans[i]<<endl;
		return 0;
	}
	```

!!! Trick
	最优性问题的时候可以考虑某个答案如何才可能是最优的，以此排除冗杂决策。

```cpp
Tag : 思维/树状数组/线段树
```

### ABC239F Construct Highway

`Feb/28/2022`

> 给你一颗树，只告诉你一部分边和每个点的度数。
>
> 构造这棵树。
>
> 2e5.

首先，如果给出的度数之和不等于 $2n-2$ ，明显无解。

然后发现给出的这些边会把所有的节点分成若干个连通块。

所以考虑在连通块之间连边。

如果同一个连通块里面的两个点连边，那也是无解的。

最特殊的是只需要一个度数的连通块，因为最后必须是一棵树，它想要联通，就必定不能和另外一个需要一个度数的连通块连边。

除非只剩最后两个连通块。

否则最后构造出来的会是个森林。

所以把所有连通块分成两组，一组是只需要一个度数的连通块，另外一组是需要至少两个的。

然后就让第一组的不断和第二组的连边，第二组组内排序从小到大，如果第二组的某个连通块在连完之后变成了只需要一个的，扔进第一组。

其它情况就是无解了。

利用 dsu 维护即可。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	constexpr int si=2e5+10;
	int n,m,sum=0; int deg[si];
	std::vector<int>f[si];
	struct Mycmp{ inline bool operator ()(int x,int y)const{ return f[x].size()<f[y].size(); } };
	struct Dsu{
		int pa[si];
		inline int root(int x){ if(pa[x]!=x) return pa[x]=root(pa[x]); return pa[x]; }
		inline bool same(int x,int y){ return root(x)==root(y); }
		inline void Union(int x,int y){ int rx=root(x),ry=root(y);if(rx==ry) return; pa[rx]=ry; }
	}dsu; std::multiset<int,Mycmp>s;
	std::vector<pair<int,int>>ans;
	
	signed main(){ 
		cin>>n>>m; for(register int i=1;i<=n;++i) cin>>deg[i],sum+=deg[i],dsu.pa[i]=i;
		if(sum!=2*n-2) return puts("-1"),0;
		f[0].push_back(0); for(register int i=1;i<=m;++i){
			int u,v; cin>>u>>v; --deg[u],--deg[v]; 
			if(dsu.same(u,v)) return puts("-1"),0; dsu.Union(u,v);	
		} for(register int i=1;i<=n;++i) if(deg[i]<0) return puts("-1"),0; else while(deg[i]--) f[dsu.root(i)].push_back(i);
		for(register int i=1;i<=n;++i) if(f[i].size()>0) s.insert(i);
		while(!s.empty()){
			std::multiset<int,Mycmp>::iterator it=s.begin(),itt=s.upper_bound(0);
			int rmp=*it;
			if(f[*it].size()>1) return puts("-1"),0; s.erase(it);
			if(!f[rmp].size()) continue;
			if(itt==s.end()){
				std::multiset<int,Mycmp>::iterator yt=std::prev(s.end()); int ls=*yt; 
				ans.push_back({f[rmp][0],*prev(f[ls].end())}),s.erase(yt);
			}
			else{
				int ls=*itt; ans.push_back({f[rmp][0],*prev(f[ls].end())});
				s.erase(itt),f[ls].pop_back(),s.insert(ls);
			}
		} for(std::pair<int,int>x:ans) cout<<x.first<<" "<<x.second<<endl; return 0;
	}
	```

```cpp
Tag : 并查集/构造/思维
```

### CF1642E/CF1641C Anonymity Is Important

`Feb/24/2022`

> 给你一个病人的序列，初始不知道任何关于病人的信息。
>
> 现在有 $q$ 次操作，第一种是 `0 l r 0/1`  表示确定 $[l,r]$ 当中有没有病人
>
> 第二种是询问 `1 x` ，要求输出在当前已知信息下，病人 $x$ 的状态（没病，有病，不确定）。
>
> 2e5.	

这题有很多种做法，我的做法有两种，一种是类似 jiangly 做法的一个 `std::set` 维护。

另外一种是 odt。

+ set 做法：

思路之后补，先放代码

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	int n,q;
	
	signed main(){
		cin>>n>>q; std::set<int>s; for(register int i=-1;i<=n;++i) s.insert(i);
		std::map<int,int>rec; while(q--){ int op; cin>>op;
			if(op==0){ 
				int l,r,x; cin>>l>>r>>x; --l;
				if(!x){
					std::set<int>::iterator it=s.lower_bound(l);
					while(*it<r) it=s.erase(it);
				}	
				else{
					std::map<int,int>::iterator it=rec.lower_bound(l);
					if(it!=rec.end() && it->second<=r) continue;
					rec[l]=r,it=rec.find(l);
					while(it!=rec.begin() && r<=std::prev(it)->second) rec.erase(std::prev(it));
					}
			}
			else{ int x;cin>>x;--x;
				if(!s.count(x)) puts("NO");
				else{
					auto qwq=s.find(x);
					int l=*std::prev(qwq),r=*std::next(qwq);
					auto it=rec.upper_bound(l);
					if(it!=rec.end() && it->second<=r) puts("YES");
					else puts("N/A");
				}
			}
		} return 0;
	}
	```

+ odt 做法：

首先考虑利用 odt 维护整个序列，初始插入一个 $(1,n,-1)$ 的节点表示整个序列都不确定。

发现 $0$ 的优先级是最高的，$1$ 不能覆盖它。

所以如果是 `0 l r 0` ，直接暴力 `assign` 即可。

但是如果是 `0 l r 1` ，就要考虑不覆盖 $(l,r,0)$ 的节点。

直接暴力即可，用链表维护的话复杂度应该没太大问题。

如果最后询问的位置的颜色是 $1$ 并且这个块的长度是 $1$ ，答案就是 YES

如果询问到 $0$，答案是 NO

如果询问到 $-1$ 或者块长不为 $1$ 的颜色为 $1$ 的块，答案是 N/A。

另外一种 odt 套 odt 的做法似乎没有必要。

```cpp
Tag : 暴力DS/ODT/染色
```

### ABC241F Skate

`Feb/28/2022`

> 给一个矩阵，某些位置有障碍，每次移动必须要在最后撞到一个障碍，
>
> 问从 $(s_x,s_y)$ 到 $(g_x,g_y)$ 的最小步数。
>
> 矩阵1e9，障碍**1e5**，起点终点不重合。

细节略微有点多的 BFS。

因为每一步最后只会停在障碍的上下左右四个方向。

所以把障碍的上下左右四个方向都建一个点，分别和可以到它的点连边。

然后 BFS 即可。

数据范围已经提示了要从障碍入手了。

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	constexpr int si=1e6+10;
	std::map<int,std::set<int>>row,col;
	std::map<std::pair<int,int>,int>rec;
	std::vector<int>G[si];
	int dis[si];
	int sx,sy,gx,gy;
	int h,w,n,tot=0;
	inline void node(int x,int y){ if(!(x<1 || x>h || y<1 || y>w) && !rec.count({x,y})) rec[{x,y}]=++tot; }
	inline void add(int x,int y){ G[x].push_back(y); }
	inline void bfs(int s){
		for(register int i=1;i<=tot;++i) dis[i]=-1;
		std::queue<int>q; q.push(s),dis[s]=0;
		while(!q.empty()){
			int u=q.front();q.pop();
			for(int v:G[u]){
				if(!~dis[v]) dis[v]=dis[u]+1,q.push(v);
			}
		}
	}
	
	signed main(){
		cin>>h>>w>>n>>sx>>sy>>gx>>gy;
		for(register int i=1,u,v;i<=n;++i) cin>>u>>v,row[u].insert(v),col[v].insert(u),node(u-1,v),node(u+1,v),node(u,v+1),node(u,v-1); node(sx,sy),node(gx,gy);
		for(std::pair<pair<int,int>,int>iter:rec){
			int x=iter.first.first,y=iter.first.second;
			std::_Rb_tree_const_iterator<int> it;
			it=row[x].lower_bound(y); if(it!=row[x].begin()) add(rec[{x,y}],rec[{x,*std::prev(it)+1}]);
			it=row[x].upper_bound(y); if(it!=row[x].end()) add(rec[{x,y}],rec[{x,*it-1}]);
			it=col[y].lower_bound(x); if(it!=col[y].begin()) add(rec[{x,y}],rec[{*std::prev(it)+1,y}]);
			it=col[y].upper_bound(x); if(it!=col[y].end()) add(rec[{x,y}],rec[{*it-1,y}]);
		} bfs(rec[{sx,sy}]);
		return printf("%lld\n",dis[rec[{gx,gy}]]),0;
	}
	```

!!! Trick
	多读一读数据范围

```cpp
Tag : BFS/思维 
```

