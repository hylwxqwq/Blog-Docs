## 三月 Tricks 整理

### [ZJOI2007] 最大半联通子图

> 给定一张有向图 $G$。
>
> 称一个导出子图是“半连通的”，当且仅当对于导出子图当中的任意点对 $(u,v)$。
>
> 都有 $u$ 可以到达 $v$ 或者 $v$ 可以到达 $u$。
>
> 求 $G$ 的最大半连通子图的大小，和个数。

首先发现强连通分量必然是一个半联通子图。

所以先缩点得到一个 DAG。

发现最大半联通子图必然是 DAG 上的一条最长链。

并且这个最长链不能分岔。

那么在 DAG 上进行递推求最长路即可。

因为 Tarjan 缩点会得到逆拓扑序，所以不用拓扑排序了，倒过来就行。

题目要求记录方案，那么就在 $f_i$ 进行转移的时候记录一个 $g_i$ 表示 $f_i$ 这个状态对应的方案。

题目可能有重边，所以需要记录每一个节点上一次是从哪里转移的，避免重复计算。

初始化 $f_i=size_{scc_{i}},g_i=1$。

核心代码：

??? note "Code"
    ```cpp
    for(register int u=cnt;u>=1;--u) f[u]=scc[u].size(),g[u]=1;
    for(register int u=cnt;u>=1;--u){
    	for(register int i=ee[u].head;i;i=ee[i].Next){
    		int v=ee[i].ver; if(las[v]==u) continue; las[v]=u;
    		if(f[u]+(int)scc[v].size()>f[v]) 
                f[v]=f[u]+(int)scc[v].size(),g[v]=g[u];
    		else if(f[u]+(int)scc[v].size()==f[v]) 
                g[v]=(g[v]+g[u])%mod;
    	}
    } int res1=0,res2=0;
    for(register int i=1;i<=cnt;++i){
    	if(f[i]>res1) res1=f[i],res2=g[i];
    	else if(f[i]==res1) res2=(res2+g[i])%mod;
    } return printf("%d\n%d\n",res1,res2),0;
    ```

!!! Tricks
    1. Tarjan 缩完点得到的序列是逆拓扑序。
    
    2. DAG 上 DP 一般是往前推而不是往后找前驱的方式。
    
    3. DAG 上 DP 有可能需要考虑重边。

??? Warning

    还有，如果非要在 SCC 后利用原图建边，一定要手写一个 for 。
    
    如果是这样写只会清空 $e_0$
    
    ```cpp
    struct Edge{
        int head,ver,Next;
        inline void Init(){ head=-1; }
    }e[si_m]
       
    e->Init();
    ```
    
    二维的 `std::bitset<>` 之类的 STL 可以这么写（但是常数大），是因为它重载了 `[]` ，本质上它是一个**类似**这样的东西：
    
    ```cpp
    struct Bitset{
        bool bb[si];
    	inline bool* operator [] (int idx){ return &bb[idx]; }
    }b; 
    // 乱写的，理解就行
    ```
    
    也就是说它并不是一个 bitset 数组，仍然只是一个 bitset。

```cpp
Tag : 强连通分量/DAG上的DP/求方案
```

### Acwing361 观光奶牛

> 给定一张图，求图上的一个环使得
>
> “环上各点的权值之和”除以“环上各边的权值之和”最大。

01分数规划，把点权移动到边权上，

因为求的是 $\max\{\dfrac{\sum f_i}{\sum t_i}\}$。

二分答案之后变成判定 $mid$ 是否 $\le \dfrac{\sum f_i}{\sum t_i}$  。

也就是判断环上 $\sum (mid\times t_i -f_i)$ 是否 $\le 0$。

所以把边变成 $t\times mid-f$ 的形式跑 SPFA 最短路。

如果有负环则 $mid$ 不可行，如果正常求出最短路，证明 $mid$ 可行。

??? note "Code"
    ```cpp
    /*
     * @Author: black_trees <black_trees@foxmail.com>
     * @Date: 2022-03-08 08:30:06
     * @Last Modified by: black_trees <black_trees@foxmail.com>
     * @Last Modified time: 2022-03-08 09:12:38
     */
    
    /*
     * READ THIS BEFORE YOU WRITE YOUR CODE
     * ====================================
     * @ Read the description carefully!
     * @ Pay attention to the data domain.
     * @ Keep calm.
     * @ Write down your thought on your draft.
     * @ Keep your mind clear.
     * @ Stabilize the coding speed.
     * ====================================
     * READ THIS AGAIN BEFORE YOU START !!!
     */
    
    #include<bits/stdc++.h>
    using namespace std;
    using ldb=long double;
    
    constexpr int si_n=1e3+10;
    constexpr int si_m=5e3+10;
    constexpr ldb eps=1e-4;
    int L,P,tot=0;
    int f[si_n],t[si_m];
    double dis[si_n];
    int cnt[si_n];
    bool vis[si_n];
    std::queue<int>q;
    struct Edge{ int head,Next,ver,w; }e[si_m];
    inline void add(int u,int v,int w){ e[++tot].ver=v,e[tot].Next=e[u].head,e[u].head=tot,e[tot].w=w; }
    inline bool spfa(ldb mid){
        for(register int i=1;i<=L;++i) cnt[i]=dis[i]=0,q.push(i),vis[i]=true;
        while(!q.empty()){
            int u=q.front(); q.pop();
            vis[u]=false;
            for(register int i=e[u].head;i;i=e[i].Next){
                int v=e[i].ver; ldb w=e[i].w*1.0*mid-1.0*f[u];
                if(dis[v]>dis[u]+w){
                    dis[v]=dis[u]+w,cnt[v]=cnt[u]+1;
                    if(cnt[u]>=L) return true;
                    if(!vis[v]) q.push(v),vis[v]=true;
                }
            }
        } return false;
    }
    
    int main(){
        cin>>L>>P;
        for(register int i=1;i<=L;++i) cin>>f[i];
        for(register int i=1,u,v;i<=P;++i) cin>>u>>v>>t[i],add(u,v,t[i]);
        ldb l=0.0,r=1000.0;
        while(r-l>eps){
            ldb mid=(l+r)/2;
            if(spfa(mid)) l=mid;
            else r=mid;
        } return printf("%.2Lf\n",r),0;
    }
    
    ```

```cpp
Tag : SPFA/01分数规划
```

### Acwing1165 单词环

01分数规划，把每个单词的前两个字母和后两个字母连边，单词长度看作边权。

这样子点的数量就降下来了。

要求的是 $\max\{\dfrac{\sum len}{n}\}$，$n$ 是选的单词的个数。

二分答案后转化为求 $mid$ 是否 $\le \dfrac{\sum len}{n}$。

也就是 $\sum len_i- n\times mid=\sum(len_i-mid)$ 是否 $\ge 0$

所以把边权化成 $len-mid$ 的形式跑 SPFA 最长路。

判断有没有正环即可判定 $mid$ 的可行性。

??? note "Code"
    ```cpp
    /*
     * @Author: black_trees <black_trees@foxmail.com>
     * @Date: 2022-03-08 09:30:27
     * @Last Modified by: black_trees <black_trees@foxmail.com>
     * @Last Modified time: 2022-03-08 10:34:42
     */
    
    /*
     * READ THIS BEFORE YOU WRITE YOUR CODE
     * ====================================
     * @ Read the description carefully!
     * @ Pay attention to the data domain.
     * @ Keep calm.
     * @ Write down your thought on your draft.
     * @ Keep your mind clear.
     * @ Stabilize the coding speed.
     * ====================================
     * READ THIS AGAIN BEFORE YOU START !!!
     */
    
    #include<bits/stdc++.h>
    using namespace std;
    using ldb=long double;
    
    constexpr int si_n=7e2+10;
    constexpr int si_m=1e5+10;
    constexpr int eps=1e-4;
    int n,tot=0,cnt=0,num=0;
    std::map<pair<char,char>,int>rec;
    inline void add(char c,char cc){ if(rec.find({c,cc})==rec.end()) rec[{c,cc}]=++cnt;  }
    struct Edge{ 
        int head,ver,Next,w; 
        inline void Init(){ head=-1; }
    }e[si_m];
    inline void add(int u,int v,int w){ e[++tot].ver=v,e[tot].Next=e[u].head,e[u].head=tot,e[tot].w=w; }
    ldb dis[si_n]; bool vis[si_n]; int Cnt[si_n];
    std::queue<int>q;
    inline bool spfa(ldb mid){
        for(register int i=1;i<=cnt;++i) vis[i]=true,q.push(i),Cnt[i]=0,dis[i]=0;
        while(!q.empty()){
            int u=q.front(); q.pop();
            vis[u]=false;
            for(register int i=e[u].head;i;i=e[i].Next){
                int v=e[i].ver; ldb w=e[i].w*1.0-mid;
                if(dis[v]<dis[u]+w){
                    dis[v]=dis[u]+w,Cnt[v]=Cnt[u]+1;
                    if(++num>10000) return true;
                    if(Cnt[v]>=cnt) return true;
                    if(!vis[v]) q.push(v),vis[v]=true;
                }
            }
        } return false;
    }
    
    int main(){
        while(~scanf("%d",&n) && n){ 
            e->Init(); num=0,tot=0,cnt=0; rec.clear();
            for(register int i=1;i<=n;++i){
                string s; cin>>s;
                add(s[0],s[1]),add(s[s.size()-2],s[s.size()-1]);
                int u=rec[{s[0],s[1]}],v=rec[{s[s.size()-2],s[s.size()-1]}];
                add(u,v,s.size());
            } ldb l=0.0,r=1000.0;
            if(!spfa(0)){ puts("No solution"); continue; }
            while(r-l>eps){
                ldb mid=(l+r)/2;
                if(spfa(mid)) l=mid;
                else r=mid;
            } printf("%.2Lf\n",r);
        }
        return 0;
    }
    
    // 不知道是不是 WA 的，不管了
    ```

!!! Trick
    01 分数规划的时候如果结合了图论，可以根据化出来的和式改变边权（在 SPFA 里面改就行，不用重新建图）。

```cpp
Tag : SPFA/01分数规划
```

### Acwing350 巡逻

考虑 $K=1$ 的情况。

发现只需要在直径的两个端点之间连接一条边。

然后这条直径就可以只走一次。

必然是最优的。

考虑 $K=2$ 的情况。

为了不重复，也就是使得被减少的边尽量的多。

在 “抛掉” 原来选中的直径的树上再求一次直径。

注意这里不是真的去除，而是说使得这条直径不会被选。

所以一个 Tricky 的做法就是把选中的直径上的所有边全部设置成 $-1$。

但是注意，如果设置成 $-1$ ，就不能用 2-BFS 求直径了，就只能 DP。

但是为了求方案方便，第一次求直径要用 2-BFS

$\text{Trick:}$ 求直径的时候如果有负边权不能使用 2-BFS。

但是 2-BFS 相对于 dp 更容易求方案。

??? note "Code"
    ```cpp
    /*
     * @Author: black_trees <black_trees@foxmail.com>
     * @Date: 2022-03-17 15:06:41
     * @Last Modified by: black_trees <black_trees@foxmail.com>
     * @Last Modified time: 2022-03-17 20:26:14
     */
    
    /*
     * READ THIS BEFORE YOU WRITE YOUR CODE
     * ====================================
     * @ Read the description carefully!
     * @ Pay attention to the data domain.
     * @ Keep calm.
     * @ Write down your thought on your draft.
     * @ Keep your mind clear.
     * @ Stabilize the coding speed.
     * ====================================
     * READ THIS AGAIN BEFORE YOU START !!!
     */
    
    #include<queue>
    #include<cstring>
    #include<cstdio>
    #include<iostream>
    #include<algorithm>
    using namespace std;
    
    constexpr int si_n=1e5+10;
    constexpr int si_m=2e5+10;
    int n,K,tot=0;
    int head[si_n];
    struct Edge{ int ver,Next,w; }e[si_m];
    inline void add(int u,int v,int w){ e[tot]=(Edge){v,head[u],w},head[u]=tot++; }
    struct Node{ int nu,ans; };
    std::queue<Node>q;
    bool vis[si_n];
    int pos=0,qos=0,ans=-1;
    inline void Bfs(int st,int &qwq){
        memset(vis,false,sizeof vis),vis[st]=true,q.push((Node){st,0}); 
        while(!q.empty()){
            Node u=q.front(); q.pop();
            if(ans<u.ans) qwq=u.nu,ans=u.ans;
            for(register int i=head[u.nu];~i;i=e[i].Next){
                int v=e[i].ver,w=e[i].w;
                if(!vis[v]) q.push((Node){v,ans+w}),vis[v]=true;;
            }
        } return ;
    }
    int dis[si_n];
    inline void dfs(int u,int fa){
        for(register int i=head[u];~i;i=e[i].Next){
            int v=e[i].ver,w=e[i].w;
            if(v==fa) continue;
            dfs(v,u);
            ans=max(ans,dis[u]+dis[v]+w);   
            dis[u]=max(dis[u],dis[v]+w);
        } return ;
    }
    inline bool Rev(int u,int fa){
        if(u==pos || u==qos) return true;
        int ok=0;
        for(register int i=head[u];~i;i=e[i].Next){
            int v=e[i].ver,&w=e[i].w;
            if(v==fa) continue;
            if(Rev(v,u)) ok++,e[i].w=e[i^1].w=-1;
        } return ok==1?true:false;
    }
    int main(){
        memset(head,-1,sizeof head),memset(dis,0,sizeof dis);
        cin>>n>>K; int res=(n<<1);
        for(register int i=1;i<n;++i){
            int u,v; cin>>u>>v;
            add(u,v,1),add(v,u,1);
        }
        
    // for(register int i=1;i<=n;++i){
    //     for(register int j=head[i];~j;j=e[j].Next){
    //         cout<<i<<" "<<e[j].ver<<" "<<e[j].w<<endl;
    //     }
    // }
    
        Bfs(1,pos);
        ans=-1;
        Bfs(pos,qos);
        res-=ans;
    
    // cout<<ans<<endl;
    // cout<<res+ans<<endl;
    // cout<<pos<<" "<<qos<<endl;
    
        if(K==1){ res--; return cout<<res<<endl,0; } 
        Rev(1,0);
    
    // for(register int i=1;i<=n;++i){
    //     for(register int j=head[i];~j;j=e[j].Next){
    //         cout<<i<<" "<<e[j].ver<<" "<<e[j].w<<endl;
    //     }
    // }
    
    // cout<<ans<<endl;
        
        ans=-1;
        dfs(1,0);
    
    // for(register int i=1;i<=n;++i) cout<<dis[i]<<" "; cout<<endl;
    // cout<<ans<<endl;
    
        res-=ans; 
        return cout<<res<<endl,0;
    }
    
    // If there exist negtive edge(s) on Tree, don't use 2-BFS to calc diameter.
    // should use dp.
    
    ```

```cpp
Tag : 树的直径/方案
```

### Acwing354 天天爱跑步

这题和雨天的尾巴很像。

都可以对于每个节点维护一颗动态开点线段树然后线段树合并。

不过这题的答案不是最值，有区间可减性。

所以可以不用线段树，直接在每个节点记录相关的操作。

然后 dfs 一次，对于每个节点，递归进子树完之后的答案减去递归之前的就是这个节点的答案。

??? note "Code"
    ```cpp
    /*
     * READ THIS BEFORE YOU WRITE YOUR CODE
     * ====================================
     * @ Read the description carefully!
     * @ Pay attention to the data domain.
     * @ Keep calm.
     * @ Write down your thought on your draft.
     * @ Keep your mind clear.
     * @ Stabilize the coding speed.
     * ====================================
     * Common Bugs
     * ====================================
     * @ Unuse some written function (forget write dfs(v) in dfs(u)) ?
     * @ long long or precision ERROR ?
     * @ Output Format (%lld,%llu) ?
     * @ Special cases (n=1),(root is not 1) ?
     * @ Clear the array (head,vis) ?
     * @ Wrong variable name (i, but written j) ?
     * ====================================
     * READ THIS AGAIN BEFORE YOU START !!!
     */
    #pragma GCC optimize("Ofast", "inline", "-ffast-math")
    #pragma GCC target("avx,sse2,sse3,sse4,mmx")
    #include<queue>
    #include<vector>
    #include<cstring>
    #include<iostream>
    using namespace std;
    
    constexpr int si=3e5+10;
    int n,m;
    int tot=0,head[si];
    struct Edge{ int ver,Next;}e[si<<1];
    inline void add(int u,int v){ e[tot]=(Edge){v,head[u]},head[u]=tot++;}
    std::queue<int>q; std::vector<int>a1[si],b1[si],a2[si],b2[si];
    int c1[si<<1],c2[si<<1],ans[si];
    int w[si],v[si];
    
    int f[si][20],dep[si];
    inline void Bfs(){
        q.push(1),dep[1]=1;
        while(!q.empty()){
            int u=q.front();q.pop();
            for(register int i=head[u];~i;i=e[i].Next){
                int v=e[i].ver;
                if(dep[v]) continue;
                dep[v]=dep[u]+1,f[v][0]=u;
                for(register int j=1;j<=19;++j) f[v][j]=f[f[v][j-1]][j-1];
                q.push(v);
            }
        }   
    }
    inline int Lca(int u,int v){
        if(dep[u]<dep[v]) swap(u,v);
        for(register int i=19;i>=0;--i){
            if(dep[f[u][i]]>=dep[v]) u=f[u][i];
        } if(u==v) return u;
        for(register int i=19;i>=0;--i){
            if(f[u][i]!=f[v][i]) u=f[u][i],v=f[v][i];
        } return f[u][0];
    }
    inline void dfs(int x){
        int val1=c1[dep[x]+w[x]], val2=c2[w[x]-dep[x]+n];v[x]=1;
        for(register int i=head[x];~i;i=e[i].Next){
            int y=e[i].ver;
            if(v[y]) continue;
            dfs(y);
        }
        for(register int i=0;i<a1[x].size();i++) c1[a1[x][i]]++;
        for(register int i=0;i<b1[x].size();i++) c1[b1[x][i]]--;
        for(register int i=0;i<a2[x].size();i++) c2[a2[x][i]+n]++;
        for(register int i=0;i<b2[x].size();i++) c2[b2[x][i]+n]--;
        ans[x]+=c1[dep[x]+w[x]]-val1+c2[w[x]-dep[x]+n]-val2;
    }
    
    int main(){
        cin>>n>>m,memset(head,-1,sizeof head);
        for(register int i=1;i<n;++i){
            int x,y; cin>>x>>y;
            add(x,y),add(y,x);
        }
        for(register int i=1;i<=n;++i) cin>>w[i];
        Bfs();
        for(register int i=1;i<=m;++i){
            int x,y; cin>>x>>y;
            int z=Lca(x,y);
            a1[x].push_back(dep[x]),b1[f[z][0]].push_back(dep[x]);
            a2[y].push_back(dep[x]-2*dep[z]),b2[z].push_back(dep[x]-2*dep[z]);
        } dfs(1);
        for(register int i=1;i<=n;++i) cout<<ans[i]<<" "; cout<<endl;
        return 0;
    }
    
    ```

```cpp
Tag : 树上差分
```

### Acwing369 北大ACM队的远足

> 给定一张有向图，有两次乘车的机会，可以再任意地方上下车，但是下车了就用了一次机会。
>
> 每次连续乘车的举例不能超过 $q$。
>
> 求从 $s \to t$ 要经过的 “危险路段” 的长度最小是多少。
>
> “危险路段”定义为 $s\to t$ 的必经边。

$s \to t$ 的必经边的意思就是，任意一条从 $s\to t$ 的路径，他们都包含的边。

首先考虑一个求出有向图的桥（必经边）的算法。

可以使用支配树先求出必经点集然后做。

不过有一种比较好写的 $\text{O}(n)$ DP 做法。

设 $dp_s[i]$ 表示从 $s$ 到 $i$ 的路径条数。

$dp_t[i]$ 表示反图上从 $t$ 到 $i$ 的路径条数。

经过推导不难发现两个充要条件：

1. 一个点 $x$ 是必经点，当且仅当 $dp_s[i] \times dp_t[i] = dp_s[t]$。
2. 一条边 $(u\to v)$ 是必经边，当且仅当 $dp_s[u] \times dp_t[v] = dp_s[t]$。

但是 $dp_s,dp_t$ 是指数级别的，`__uint128_t` 都不一定可以应付。

所以考虑类似哈希的思想，给他们取模。

用类似双哈希的思想，同时记录两种模数意义下的值。

然后两个值相等，当且仅当他们在两种意义下的值都一样。

一般取 $mod1=1e9+7,mod2=1e9+9$。

然后考虑求出 $s \to t$ 的最短路。

明显的，这些必经边肯定在任意的一条最短路上都出现了，所以随便选哪一条都一样。

然后考虑如何统计答案。

经过严谨的举例，画图论证可以发现，直接统计的话非常容易算重复。

意思是两个连续段重合的时候会非常难算，要分特别多类讨论。

所以考虑一个非常有意思的 Trick ：

!!! Trick
    **以每一个点为 “分割点”，分别 DP，计算两边的答案** $ds,dt$，重合的问题就得到了解决。

等价于把问题转化成了求只覆盖一次，桥边总长度减去可以覆盖的最长的桥边的长度。

然后每一个点的答案就是 $ds_i + dt_i$，枚举所有点更新即可，这个 Trick 在 _Acwing341 最优贸易_ 那一题也用到了。

本题给的不是 DAG，所以还需要先拓扑排序。

??? note "Code"
    ```cpp
    #include <queue>
    #include <cstring>
    #include <iostream>
    #include <algorithm>
    
    using namespace std;
    
    const int N = 1e5 + 10;
    const int M = 2e5 + 10; 
    const int mod = 1e9 + 7;
    // 本题不需要双取模
    
    int ver[M*2], edge[M*2], nxt[M*2], head[N], tot;
    int f[2][N], deg[2][N], dis[N], pre[N], n, m, s, t, Q;
    
    bool bri[M*2];
    int a[N], b[N], cnt; 
    int sum[N], sum_bri[N], ds[N], dt[N], ds_min[N];
    int occur[N], first_occur[N];
    
    queue<int> q;
    
    void add(int u, int v, int w) {
        ver[++tot] = v, edge[tot] = w, nxt[tot] = head[u], head[u] = tot;
    }
    
    void topsort(int s, int bit) {
        if(bit == 0) { 
            memset(dis, 0x3f, sizeof(dis));
            dis[s] = 0;
        }
        f[bit][s] = 1;
        for(int i = 1; i <= n; i++)
            if(deg[bit][i] == 0) q.push(i);
        while(!q.empty()) {
            int u = q.front(); q.pop();
            for(int i = head[u]; i; i = nxt[i])
                if((i & 1) == bit) {
                    int v = ver[i];
                    f[bit][v] = (f[bit][v] + f[bit][u]) % mod; 
                    if(bit == 0 && dis[v] > dis[u] + edge[i]) {  
                        dis[v] = dis[u] + edge[i];
                        pre[v] = i;
                    }
                    if(--deg[bit][v] == 0) q.push(v);
                }
        }
    }
    
    int main() {
        int T; 
        cin >> T;
        while(T--) {
            memset(head, 0, sizeof(head)), memset(deg, 0, sizeof(deg));
            memset(f, 0, sizeof(f)), tot = 1; cnt = 0;
            memset(bri, 0, sizeof(bri)), memset(occur, 0, sizeof(occur));
            
            cin >> n >> m >> s >> t >> Q;
            s++; t++;
            
            for(int i = 1; i <= m; i++) {
                int u, v, w;
                cin >> u >> v >> w;
                u++, v++;
                add(u, v, w), add(v, u, w);
                deg[0][v]++, deg[1][u]++; 
            }
            
            topsort(s, 0);
            
            if(f[0][t] == 0) { 
            	puts("-1"); 
            	continue; 
            }
            
            topsort(t, 1);
            
            for(int i = 2; i <= tot; i += 2) {
                int u = ver[i ^ 1], v = ver[i];
                if((long long)f[0][u] * f[1][v] % mod == f[0][t]) {
                    bri[i] = true;
                }
            }
    
            for(int u = 1; u <= n; u++) {
                for(int i = head[u]; i; i = nxt[i]) {
                    if(i & 1) continue; 
                    int v = ver[i];
                    if(occur[v] == u) {
                        bri[i] = false;
                        bri[first_occur[v]] = false;
                    } else {
                        occur[v] = u;
                        first_occur[v] = i;
                    }
                }
            }
            while(t != s) {
                a[++cnt] = edge[pre[t]];
                b[cnt] = bri[pre[t]];
                t = ver[pre[t] ^ 1];
            }
            
            for(int i = 1; i <= cnt; i++) {
                sum[i] = sum[i - 1] + a[i]; 
                sum_bri[i] = sum_bri[i - 1] + (b[i] ? a[i] : 0);
            }
            ds_min[0] = 1 << 30;
            for(int i = 1, j = 0; i <= cnt; i++) { 
                while(sum[i] - sum[j] > Q) j++;
                ds[i] = sum_bri[j];
                if(j > 0 && b[j]) ds[i] -= min(a[j], Q - (sum[i] - sum[j]));
                ds_min[i] = min(ds[i], ds_min[i - 1] + (b[i] ? a[i] : 0)); 
            }
            for(int i = cnt, j = cnt + 1; i; i--) { 
                while(sum[j - 1] - sum[i - 1] > Q) j--;
                dt[i] = sum_bri[cnt] - sum_bri[j - 1];
                if(j <= cnt && b[j]) dt[i] -= min(a[j], Q - (sum[j - 1] - sum[i - 1]));
            }
    
            int ans = 1 << 30;
            for(int i = 1; i <= cnt; i++)
                ans = min(ans, dt[i] + ds_min[i - 1]);
    
            for(int i = 1, j = 0; i <= cnt; i++) {
                while(sum[i] - sum[j] > 2 * Q) j++;
                int temp = sum_bri[j];
                if(j > 0 && b[j]) temp -= min(a[j], 2 * Q - (sum[i] - sum[j]));
                ans = min(ans, temp + sum_bri[cnt] - sum_bri[i]);
            }
            cout << ans << endl;
        }
    }
    ```

```cpp
Tag : 有向图的必经边/DP/拓扑排序
```

### Acwing386 社交网络

要求的式子是：

$$I(v) = \sum\limits_{s \not=v,t\not=v}\dfrac{C_{s,t}(v)}{C_{s,t}}$$

下面的是 $s \to t$ 的最短路条数

上面的是 $s\to t$ 的最短路经过 $v$ 的条数。

注意这里是有序点对，要算两次。

数据范围是 $n = 1e2$。

然后 $s\to v\to t$，加上多源询问，不难想到 Floyd 解决。

实际上要做的就是在 Floyd 跑最短路的时候最短路计数即可。

然后上面的部分只需要判定 $dis_{s,v} + dis_{v,t} = dis_{s,t}$ 即可。

然后答案累加 $cnt_{s,v} \times cnt_{v,t}$。

??? note "Code"
    ```cpp
    
    #include <cstdio>
    #include <cstring>
    #include <iostream>
    #include <algorithm>
    
    using namespace std;
    using i64 = long long;
    
    const int si = 1e2 + 10;
    
    int n, m;
    int dis[si][si];
    i64 cnt[si][si];
    
    int main() {
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
                }
            }
        }
        for(int k = 1; k <= n; ++k) {
            double ans = 0.0;
            for(int i = 1; i <= n; ++i) {
                if(i == k) continue;
                for(int j = 1; j <= n; ++j) {
                    // * s -> t 不一定是无序数对，t -> s 也可以，所以不能从 i + 1 开始.
                    if(i == j || j == k) continue;
                    if(dis[i][j] == dis[i][k] + dis[k][j]) {
                        ans += ((1.0 * (cnt[i][k] * cnt[k][j])) / (1.0 * cnt[i][j]));
                    }
                    // ! 惨痛教训 ：有除法且 1.0 × 某个数的时候要用括号括起来，不然会 /1.0 然后乘上原来要除的数。
                }
            }
            printf("%.3lf\n", ans);
        }
        return 0;
    }
    ```

```cpp
Tag : 最短路计数/经过指定点最短路计数
```

### *Acwing389 直径

> 求树的直径的必经边。

发现要求的就是所有直径的公共边。

可以先求出直径之后对于每一个点判断它是不是两条直径的交点。

然后分别找所有交点点对里面最靠中间的一对就行了。

还有一种更好的做法。

$dfs$ 记录当前路径上的公共边条数。

记 $las$ 表示最后使直径更新的节点。

然后如果出现类似 $d[u] = x, d[v1] + w1 = y, d[v2] + w2 = y$ 的情况

假设 $v1$ 是更先访问的，	

1. 如果 $x > y$, 那么需要把 $u -> v1 -> subtree(v1)$ 当中的边从 $cnt$ 里减去 。
2. 如果 $x \le y$, 那么需要把 $d[u]$ 往下走对应的边从 $cnt$ 里减去。

??? note "Code"
    ```cpp
    
    #include <queue>
    #include <vector>
    #include <cstring>
    #include <iostream>
    #include <algorithm>
    
    using namespace std;
    using i64 = long long;
    
    const int si = 2e5 + 10;
    
    int n, m, cnt, las;
    vector<pair<int, int> > G[si];
    i64 d[si], ans;
    
    int dfs(int u, int fa, int dep) {
        int res = dep;
        // * ans = 直径长度
        // * cnt = 公共边条数（最终答案）
        for(auto &[v, w] : G[u]) {
            if(v == fa) continue;
            int cur = dfs(v, u, dep + 1);
            if(ans < d[u] + d[v] + w) {
                ans = d[u] + d[v] + w, las = dep, 
                cnt = res + cur - dep * 2;
                // ? 直径被更新，记录 las 并更新 cnt。
            }
            else if(ans == d[u] + d[v] + w) {
                if(d[v] + w >= d[u]) {
                    cnt = cur - las; 
                }   
                else {
                    cnt = res - las;
                }               
                // ? 出现类似 d[u] = x, d[v1] + w1 = y, d[v2] + w2 = y 的情况
                // ? 假设 v1 是更先访问的，
                // ? 1. 如果 x > y, 那么需要把 u -> v1 -> subtree(v1) 当中的边从 cnt 里减去 
                // ? 2. 如果 x <= y, 那么需要把 d[u] 往下走对应的边从 cnt 里减去。
    
                // * cur 是 2 对应的。
                // * res 是 1 对应的。
                // TODO : 具体的可能还要分析一下    
            }
            if(d[u] < d[v] + w) {
                d[u] = d[v] + w, res = cur;
            }
            else if(d[u] == d[v] + w) {
                res = dep;
            }
        } 
        return res;
        // ? 当前分支的公共边长度。
    }
    
    int main() {
        cin >> n;
        for(int i = 1; i < n; ++i) {
            int u, v, w; 
            cin >> u >> v >> w;
            G[u].emplace_back(v, w); 
            G[v].emplace_back(u, w);
        }
        dfs(1, 0, 1); 
        cout << ans << endl << cnt << endl;
        return 0;
    }
    ```

### Acwing390 逃学的小孩

可以发现，答案就是枚举直径上每一个点作为出发点的答案的最大值。

那么一个问题就是，求完 LCA 之后，如果树是带权的，该如何快速询问树上两点之间的距离？

仍旧是倍增，随着倍增 LCA 的 $f_{i,j}$ 同步记录一个 $dis_{i,j}$ 即可，查询时做一遍跳 LCA 的过程，

每次从 $u/v$ 往上跳 $2^i$ 步的时候，都让答案累加上 $dis_{u/v,i}$ 即可。

!!! Warning
    注意，**要先累加权值之后再跳**，不然会出问题。

这种类似的思想也可以在次小生成树里面体现，同步记录 $u$ 到 $2^i$ 级祖先路径上的最值/次最值。

然后用类似跳 LCA 的方式求任意两点 $(u,v)$ 的路径上的最值/次最值。

代码（此处并没有同步记录 $dis$，而是先求出所有的 $f$，然后再求 $dis$）：

??? note "Code"
    ```cpp
    int dep[si], f[si][20], lg;
    i64 dis[si][20];
    void dfs(int u, int fa) {
        dep[u] = dep[fa] + 1, f[u][0] = fa;
        for(int i = 1; i <= lg; ++i) 
            f[u][i] = f[f[u][i - 1]][i - 1];
        for(int i = head[u]; ~i; i = e[i].Next) {
            int v = e[i].ver, w = e[i].w;
            if(v == fa) 
                continue;
            dfs(v, u), dis[v][0] = w;
            // 初始化 dis。
            // 如果你是在dfs 过程中同步记录，需要先 dis[v][0] = w 然后再 dfs。
        }
    }
    i64 Dis(int u, int v) {
        i64 ret = 0;
        if(dep[u] < dep[v]) swap(u, v);
        for(int i = lg; i >= 0; --i) { 
            if(dep[f[u][i]] >= dep[v]) 
                ret += dis[u][i], u = f[u][i];
        }
        if(u == v) 
            return ret;
        for(int i = lg; i >= 0; --i) {
            if(f[u][i] != f[v][i])
                ret += dis[u][i], ret += dis[v][i],
                u = f[u][i], v = f[v][i];
            // 一定要先加上然后再跳！！！！！
            // 时隔一个月写严格次小生成树的时候，
            // 发现自己代码里有这个问题
                
        }
        return ret + dis[u][0] + dis[v][0];
        // 最后还没有会合，不要忘记加它们到 lca 的权值。
    }
    
    // in main()
    	lg = (int)(log(n) / log(2)) + 1;
        dfs(1, 0);
        for(int j = 1; j <= lg; ++j) {
            for(int i = 1; i <= n; ++i) {
                dis[i][j] = dis[i][j - 1] + dis[f[i][j - 1]][j - 1];
                // 从两个小段的信息合并到大段的信息。
            }
        }
    ```

??? note "另外一种做法"
    当然，也可以记录两个 $dep$ ，一个是把树看作无权树时候的 $dep$ ，另外一个是把树看作带权树时候的 $dep$。

    然后就可以继续用 $d[u] + d[v] - 2d[lca]$ 了。

```cpp
Tag ：树的直径/带权树上点对距离
```