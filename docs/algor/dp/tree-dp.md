## 普通树形DP

树形DP的固有特征就是使用 dfs 递归实现。

一般来说都是 dfs 到儿子，然后**上传信息**更新父亲，最后得出答案。

常见的辅助数组是：

+ `siz[u]`：记录以 $u$ 为根的子树的大小
+ `dep[u]` ：记录以某个节点（通常是 $1$）为根的时候 $u$ 的深度。
+ `dis[u]`：$u$ 能向下走的最远距离 / 以 $u$ 为根的子树当中所有节点到 $u$ 的距离和。

（除了换根DP会在第二次 dfs 的时候用父亲更新儿子）

一般来说，状态都是设计成这种 ：$f_{u}$ 表示节点 $u$ blablabla，表示以 $u$ 为根的子树 blablabla……

### 例子

> 没有上司的舞会：给你一棵树，如果选了某一个节点的父亲节点，那么就不能选这个节点，节点有点权，问你可以选择的点权和的最大值。

设 $f[u][0/1]$ 表示以 $u$ 为根的子树， $u$ 参加或者不参加舞会的所有情况，属性为：取得的最大权值。

这样子设计才能够表示出题目中**所有的限制关系**：从属，是否参加。

根据状态设计，分 $f[u][0]$ 和 $f[u][1]$ 两种情况来讨论。

如果 $u$ 不参加舞会，那么它的儿子们都可以参加舞会。

注意到 $r_i$ 可能是负数，所以要在儿子参加和不参加里面取个最大值然后再求和。

$f[u][0]=\sum \max(f[v][1],f[v][0])$

然后如果 $u$ 参加了舞会，那么它的儿子节点就都不会参加，但是它自己要参加。

所以 $f[u][1]=\sum f[v][0]+r_u$

然后我们DP完了之后就在 $f[root][0]$ 和 $f[root][1]$ 之间取个最大值即可。

从这道题不难发现，树形DP的重点就在于如何利用状态表示节点和节点之间的从属，限制等关系。

并且设计状态的时候一般都是问什么设什么（和线性DP差不多）。

??? note "Code"
    ```cpp
    
    #include<bits/stdc++.h>
    using namespace std;
    
    const int si=6e3+10;
    struct Tree{
        int ver,head,Next;
    }e[si*2];
    int root=0,cnt=0;
    void add(int u,int v){
        e[++cnt].ver=v,e[cnt].Next=e[u].head;
        e[u].head=cnt;
    }
    
    int r[si];
    int f[si][2];
    bool nrt[si];
    
    void dp(int u,int fa){
        f[u][0]=0;
        f[u][1]=r[u];
        for(register int i=e[u].head;i;i=e[i].Next){
            int v=e[i].ver;
            if(v==fa) continue;
            dp(v,u);
            f[u][0]+=max(f[v][1],f[v][0]);
            f[u][1]+=f[v][0];
        }
    }
    
    int n;
    int main(){
        scanf("%d",&n);
        for(register int i=1;i<=n;++i){
            scanf("%d",&r[i]);
        }
        for(register int i=1;i<n;++i){
            int u,v;
            scanf("%d%d",&u,&v);
            add(u,v),add(v,u);
            nrt[v]=true;
        }
        for(register int i=1;i<=n;++i){
            if(!nrt[i]){
                root=i;break;
            }
        }
        dp(root,0);
        int res=max(f[root][0],f[root][1]);
        printf("%d\n",res);
    }
    ```

## 树上背包

这个见 背包问题 。

## 换根DP

### 泛化

这类问题的显著特点是两次dfs，一次收集信息，一次进行转移。

有的时候暴力需要每个点进行一次 $\text{O}(n)$ 的 dfs，复杂度 $\text{O}(n^2)$。

此时一般都可以使用换根DP进行 $\text{O}(n)$ 的计算。

解决时一般分三步：

1. 确定一个根节点 $rt$

2. 从 $rt$ 出发，向下 dfs，并用儿子更新父亲的信息。

3. 第二次 dfs，一般还是从最开始确定的根节点 $rt$ 开始，不过这个时候需要用**以父亲** $u$ **为整棵树的根的答案**，结合信息来更新**以儿子** $v$ **为整棵树的根的答案**。

   具体来说，此时进行换根DP的时候，会设 $f_u$ 表示以 $u$ 为根的时候的答案，然后从最开始选定的根节点向下转移。

   因为每次只会向当前节点的儿子节点转移，所以只需要对信息进行一些整合、计算就可以得到儿子节点的答案。

   （此处所说的“父亲”，“儿子”，都是在最开始选定的根节点 $rt$ 为整棵树的根的情况下定义的）

### 例子

> [POI2008]STA-Station：给你一颗无根无权树，请你找到一个节点，使得以这个节点为根的时候，树的节点的深度之和最大。

首先以 $1$ 为根，考虑进行一次 dfs，求出此时每个点的深度 $depth$，子树大小 $siz$，子树到这个点的距离和 $dis$。

那么设 $f_u$ 表示以 $u$ 为根的时候，树的所有节点的深度之和。

考虑从 $u$ 的父亲 $fa$ 转移过来。

首先，$f_u$ 肯定要先加上 $dis_u$，这一部分本来就是以 $u$ 为根，所以深度和自然是到 $u$ 的距离和（因为无权）。

然后发现 $u$ 的上面那部分需要统计。

明显的，我们可以利用 $f_{fa}$ 求出上面的部分，但是 $f_{fa}$ 本身就包含了 $u$ 的子树信息。

所以要给 $f_{fa}$ 减去 $dis_u$ ，不过 $fa$ 和 $u$ 在以 $1$ 为根的时候是有深度差的，所以要再减去 $siz_u(\times 1)$。

刚好，这个深度差又会造成上面的那一部分在以 $u$ 为根的时候的深度比以 $fa$ 为根的时候增加 $1$。

所以要加上 $siz_1-siz_u$。

[![treedp-1.png](https://s4.ax1x.com/2022/02/11/HU9DIS.png)](https://imgtu.com/i/HU9DIS)

整理下式子：$f_u=f_{fa}-dis_u+dis_u-siz_u+siz_1-siz_u=f_{fa}+siz_1-2siz_u$。

所以实际上只需要记录 $siz$ 就可以了。

但是你为了转移，肯定要在换根之前先求出 $f_1$ 。这个利用 $depth$ 暴力加即可。

??? note "Code"
    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
    
    #define int long long
    constexpr int si=1e6+10;
    int n;
    struct Tree{
        int head,ver,Next;
    }e[si<<1];
    int tot=0;
    inline void add(int u,int v){
        e[++tot].ver=v,e[tot].Next=e[u].head;
        e[u].head=tot;
    }
    int f[si],siz[si],depth[si];
    inline void dfs1(int u,int fa){
        siz[u]=1;
        for(register int i=e[u].head;i;i=e[i].Next){
            int v=e[i].ver;
            if(v==fa) continue;
            depth[v]=depth[u]+1;
            dfs1(v,u),siz[u]+=siz[v];
        } return ;
    }
    inline void dfs2(int u,int fa){
        for(register int i=e[u].head;i;i=e[i].Next){
            int v=e[i].ver;
            if(v==fa) continue;
            f[v]=f[u]+siz[1]-2*siz[v];
            dfs2(v,u);
        } return ;
    }
    
    signed main(){
        scanf("%lld",&n);
        for(register int i=1;i<n;++i){
            int u,v; scanf("%lld%lld",&u,&v);
            add(u,v),add(v,u);
        } depth[1]=0;
        dfs1(1,0);
        for(register int i=1;i<=n;++i){
            f[1]+=depth[i];
        } dfs2(1,0);
        int res=0,ans=-114514;
        for(register int i=1;i<=n;++i){
            if(f[i]>ans) ans=f[i],res=i;
        } return printf("%lld\n",res),0;
    }
    ```

再总结一下：

1. 找一个根节点 $rt$
2. 统计此时的信息
3. 换根，利用父亲更新儿子的答案，转移之前要先初始化第一遍 dfs 时候的根 $rt$ 的答案。

### 另外一个例子

> [USACO12FEB]Nearby Cows G：给你一棵 $n$ 个点的树，点带权，
> 
> 对于每个节点 $i$，求出距离它不超过 $K$ 的所有节点的权值和 $m_i$，
> 
> $K$ 给定，并且 $n\times K \le 2\times 10^6, K \le 20$。

我们仍然考虑先 dfs 一遍 求出所需要的信息，而这里问的是节点权值和，又有 $K$ 的限制。

那么先找一个根节点，比如 $1$ ，然后设 $f_{u,k}$ 表示以 $u$ 为根的子树当中，和 $u$ 的距离不超过 $k$ 的节点的权值和。

这里 $f_{u,k}=\sum f_{v,k-1}$，初始化 $\forall f_{u,i}=a_u$。

然后考虑进行换根。

设 $dis_{u,k}$ 表示以 $u$ 为根的时候，$K=k$ 的时候的 $m_u$。

那么考虑从 $dis_{fa}$ 转移。

简单推一下可以发现，$dis_{u,k}=f_{u,k}+dis_{fa,k-1}-f_{u,k-2}$。

然后我们处理出 $f$ 之后，先把 $f$ 复制给 $dis$ ，那么这个时候 $dis_1$ 其实就已经算出来了，换根完后输出即可。

不过要注意的一个点是，如果是用儿子更新父亲的信息 （`dfs1`） ，那么要先 dfs 然后再更新。

如果是用父亲更新儿子的信息（`dfs2`），需要先更新再 dfs。

??? note "Code1"
    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
    
    #define int long long
    constexpr int si=1e5+10;
    int n,k;
    struct Tree{
        int head,ver,Next;
    }e[si<<1];
    int tot=0;
    inline void add(int u,int v){
        e[++tot].ver=v,e[tot].Next=e[u].head;
        e[u].head=tot;
    }
    int f[si][22],dis[si][22];
    int a[si];
    
    inline void dfs1(int u,int fa){
        for(register int i=0;i<=k;++i){
            f[u][i]=a[u];
        }
        for(register int i=e[u].head;i;i=e[i].Next){
            int v=e[i].ver;
            if(v==fa) continue;
            dfs1(v,u);
            for(register int j=1;j<=k;++j){
                f[u][j]+=f[v][j-1];
            }
        }
    }
    inline void dfs2(int u,int fa){
        for(register int i=e[u].head;i;i=e[i].Next){
            int v=e[i].ver;
            if(v==fa) continue;
            dis[v][1]+=f[u][0];
            for(register int j=2;j<=k;++j){
                dis[v][j]+=dis[u][j-1]-f[v][j-2];
            } dfs2(v,u);
        }
    }
    
    signed main(){
        scanf("%lld%lld",&n,&k);
        for(register int i=1;i<n;++i){
            int u,v;
            scanf("%lld%lld",&u,&v);
            add(u,v),add(v,u);
        }
        for(register int i=1;i<=n;++i){
            scanf("%lld",&a[i]);
        } dfs1(1,0);
        for(register int i=1;i<=n;++i){
            for(register int j=0;j<=k;++j){
                dis[i][j]=f[i][j];
            }
        } dfs2(1,0);
        for(register int i=1;i<=n;++i){
            printf("%lld\n",dis[i][k]);
        } return 0;
    }
    ```

当然 $f$ 也可以这么求：

初始化 $f_{u, 0} = a_u$，然后 $f_{u,k}=\sum f_{v,k-1}$。

此时求的是距离恰好为 $k$，所以还需要做一遍前缀和。

这种写法的实现：

??? note "Code2"
    ```cpp
    int dp[si][22]; 
    // 以 1 为根, u 的子树，距离不超过 i，cnt。
    // 初始是恰好，之后做前缀和。
    int ans[si][22];
    // 以 u 为根的答案，ans[u][1] 直接预处理，ans[u][2] 开始再换根 dp。
    
    // 第一遍 dfs
    void dfs1(int u, int fa) {
        dp[u][0] = a[u];
        for(int i = head[u]; ~i; i = e[i].Next) {
            int v = e[i].ver;
            if(v == fa) continue;
            dfs1(v, u);
            for(int j = 1; j <= k; ++j)
                dp[u][j] += dp[v][j - 1];
        }
    }
    // 前缀和，也可以不用 dfs。
    void dfs2(int u, int fa) {
        for(int i = 1; i <= k; ++i) 
            dp[u][i] += dp[u][i - 1];
        for(int i = head[u]; ~i; i = e[i].Next) {
            int v = e[i].ver;
            if(v == fa) continue;
            dfs2(v, u);
        }
    }
    // 换根
    void dfs3(int u, int fa) {  
        for(int i = head[u]; ~i; i = e[i].Next) {
            int v = e[i].ver;
            if(v == fa) continue;
            ans[v][1] = dp[v][1] + a[u];
            for(int j = 2; j <= k; ++j)
                ans[v][j] = dp[v][j] + ans[u][j - 1] - dp[v][j - 2];
            dfs3(v, u);
        }
    }
    void solve() {
        memset(dp, 0, sizeof dp);
        memset(ans, 0, sizeof ans);
        dfs1(1, 0), dfs2(1, 0);
        memcpy(ans[1], dp[1], sizeof ans[1]);
        dfs3(1, 0);
        for(int i = 1; i <= n; ++i)
            cout << ans[i][k] << " " [i != n];
        cout << endl; return; 
    }
    ```