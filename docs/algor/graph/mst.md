## 定义

> 给你一个带权无向图，求它的所有生成树当中权值和最小的一个。
>
> 带权无向图 $G$ 的生成树 $T$ 定义为包含 $G$ 的所有节点，由 $G$ 当中连接它们的 $n-1$ 条边构成的无向联通子图。

## 两个定理

1. 任意一颗最小生成树一定包含 $G$ 中最小的边（反证法即可）。 

2. 设一张无向图 $G=(V,E)$ ，从 $E$ 中选出 $k<|V|-1$条边构成 $G$ 的一个生成森林，然后再从剩余的 $|E|-k$ 条边中选出 $|V|-1-k$ 条边加入森林中，让它成为 $G$ 的生成树，并且选出的$\sum w$最小。

   那么，这个生成树一定包含 $|E|-k$ 条边里面连接生成森林的两个不连通节点的权值最小的边。

证明可以在 zhihu 看看 [@ciwei](https://www.zhihu.com/people/ciwei-78-83) 神仙的专栏，其中包括算法正确性的证明。

## Kruskal 算法

基本思想是维护图的最小生成森林。

开始的时候生成森林是空的，每一个节点就是一颗独立的树。

然后用结论 $2$ 维护森林，利用 dsu 维护联通性。

按照边权升序排个序，然后扫一遍每个边。

如果当前扫到的这条边所连的两个点 $(u,v)$ 已经联通了。那么跳过。

如果不是联通的，根据这一条：

>这个生成树一定包含 $|E|-k$ 条边里面连接生成森林的**两个不连通节点**的权值最小的边。

把这一条边加入到最小生成树里，顺便合并一下 $(u,v)$ 

??? note "Code"
    ```cpp
    struct Edge{ 
        int x,y,z;
        inline bool operator < (const Edge &b)const{ return z<b.z; }
    }a[si_m];
    struct Dsu{
        int pa[si_n];
        Dsu(){ for(register int i=1;i<=1e2+10;++i) pa[i]=i; }
        inline int root(int x){ if(pa[x]!=x) return pa[x]=root(pa[x]); return pa[x]; }
        inline bool same(int x,int y){ return root(x)==root(y); }
        inline void Union(int x,int y){ int rx=root(x),ry=root(y); if(rx==ry) return; pa[rx]=ry; }
    }dsu;
    
    int main(){
        cin>>n>>m;
        for(register int i=1;i<=m;++i){
            cin>>a[i].x>>a[i].y>>a[i].z;
        } sort(a+1,a+1+m); int ans=0;
        for(register int i=1;i<=m;++i){
            if(dsu.same(a[i].x,a[i].y)) continue;   
            dsu.Union(a[i].x,a[i].y),ans+=a[i].z;
        } cout<<ans<<endl;
        return 0;
    }
    ```

复杂度 $\text{O}(m \log m)$。

## Prim 算法

一般用于稠密图。

最开始确定 $1$ 号节点属于最小生成树。

每一次找到一条权值最小的，且满足它连接的其中一个点 $u$ 已经被选入最小生成树里，另一个点 $v$ 则未被选中的边。

具体实现：

维护一个数组 $dis$ ,如果 $u$ 没有被选入，那么 $dis_u$ 就等于 $u$ 和已经被选中点之间的连边中权值最小的边的权值。

反之 $dis_u$ 就等于 $u$ 被选中的时候选出来那条权值最小边的权值。

如何判是否选中呢？

维护一个 $vis$ 即可。从没有被选中的节点当中选出一个 $dis$ 最小的，标记它。

扫描和这个被选点的所有出边，更新另外一个端点的 $dis$ 。

最后生成树的权值和就是 $\sum\limits^{n}_{i=2} dis_i$。

??? note "Code"
    ```cpp
    inline void Prim(){
        memset(vis,false,sizeof vis),memset(dis,0x3f,sizeof dis);
        dis[1]=0;
        for(register int i=1;i<n;++i){
            int x=0;
            for(register int j=1;j<=n;++j){
                if(!vis[j] && (x==0 || dis[j]<dis[x])) x=j;
            } vis[x]=true;
            for(register int y=1;y<=n;++y) if(!vis[y]) dis[y]=min(dis[y],a[x][y]);
        }
    }
    int main(){
        cin>>n; memset(a,0x3f,sizeof a);
        for(register int i=1;i<n;++i){
            a[i][i]=0;
            for(register int j=1;j<=n;++j){
                int value; cin>>value;
                a[i][j]=a[j][i]=min(a[i][j],value);
            }
        } Prim(); int ans=0;
        for(register int i=2;i<=n;++i) ans+=dis[i];
        return printf("%d\n",ans),0;
    }
    ```

复杂度 $\text{O}(n^2)$，可以用二叉堆优化到 $\text{O}(m \log n)$，但不如直接 Kruskal。