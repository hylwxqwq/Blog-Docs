## 01背包

> 给你 $n$ 个物品，$m$ 的容量，每个物品有体积 $v_i$ 和价值 $w_i$，问你能获得的价值 $\max$。

考虑设状态 $f_{i,j}$ ，其表示集合：“只从前 $i$ 个里面选，用的空间不超过 $j$ 的所有方案”，属性为选择的 $\sum w_i$ 的 Max

 那么集合 $f_{i,j}$ 就可以划分为两个部分，前 $i$ 个当中选第 $i$ 个物品的所有情况和前 $i$ 个当中不选第 $i$ 个物品的所有情况。

前者是 $f_{i-1,j-v_{i}}+w_i$，后者是 $f_{i-1,j}$。

状态转移方程：$f_{i,j}=\max\limits_{1 \le i \le n,0 \le j \le m} \begin{cases}f_{i-1,j-v_{i}}+w_i & j \ge v_i\\ f_{i-1,j} & \text{otherwise.}\end{cases}$

```cpp
    // 注意这里的状态设计为 “至多”，所以f的初值全部为0。
    // 如果是 f[0][0]=0,memset(f,0xcf,sizeof f) 那就是“恰好”，需要在最后扫描取最大值。
    for(register int i=1;i<=n;++i){
        for(register int j=0;j<=m;++j){
            f[i][j]=f[i-1][j];
        }
        for(register int j=v[i];j<=m;++j){
            f[i][j]=max(f[i][j],f[i-1][j-v[i]]+w[i]);
        }
    } return printf("%d\n",f[n][m]),0; 
```

这里要特别强调方程里的 $j \ge v_i$ 这个条件，它是集合 $f_{i,j}$ 存在选择当前物品这个子集的充要条件。

不然的话 $f_{i,j}$ 连这个子集都没有，怎么转移？

但是也不能不转移，另外一个子集是无论如何都存在的，所以它是肯定需要转移的啊。

所以这里有种更好理解的写法：

```cpp
    for(register int i=1;i<=n;++i){
        for(register int j=0;j<=m;++j){
            f[i][j]=f[i-1][j];
            if(j>=v[i]) f[i][j]=max(f[i][j],f[i-1][j-v[i]]+w[i]);
        }
    } return printf("%d\n",f[n][m]),0;
```

另外不要忘了，$j$ 是已经使用的空间，是可能为 $0$ 的，不要漏了。『**条件的取值范围**』

还有状态的初值（边界，比如下标为 $0$ 的一系列东西是不是会有特殊的处理）『**状态的边界**』

然后这个东西是可以优化的，根据第二种写法不难发现，每次 $f_{i,j}$ 都会继承上一个状态 $f_{i-1,j}$。

并且 $f_{i}$ 系的状态只和 $f_{i-1}$ 系的状态有关系，所以可以考虑去掉第一维。

但是为了保证无后效性，也就是保证不会出现整个集合推子集的情况。

我们需要研究下方程。

考虑当前循环到 $j$，那么需要的是上一段状态 $f_{i-1}$ 当中的 $j-v_i$，也就是在 $j$ 的前面。

我们需要让每个物品都只被拿一次，所以要使用**倒序循环**。

这样，$j$ 前面的部分都是 $i-1$ 阶段的，后面的都是 $i$ 阶段的。

```cpp
    for(register int i=1;i<=n;++i){
        for(register int j=m;j>=v[i];--j){
            f[j]=max(f[j],f[j-v[i]]+w[i]);
        }
    } return printf("%d\n",f[m]),0;
```

这样子你甚至不需要额外的初始化，只需要让 $f[0]=0$ 即可，因为之后的都会覆盖。

## 完全背包

> 给你 $n$ 种物品，$m$ 的容量，每种物品有体积 $v_i$ 和价值 $w_i$，每种物品可以有无穷多个，问你能获得的价值 $\max$。

状态属性和表示集合与 01 背包一样，不同的地方再于集合的划分。

这次不是选或者不选了，而是第 $i$ 种物品选 $n$ 个，（肯定不会拿无穷多个，每种最多也就 $\lfloor \frac{m}{v_i} \rfloor = s$ 个）。

暴力做法就是枚举 $s$ 次进行决策，$\text{O}(n^3)$。

方程：

$$
f_{i,j} = \max\limits_{0\le k\le s}\{f_{i-1,j-kv_i}+kw_i\}
$$

可以考虑把每个子集的式子展开： 

整个集合 $f_{i,j}$：

$$
f_{i,j} = \max(f_{i-1,j},f_{i-1,j-v_i}+w_i,f_{i-1,j-
2v_i}+2w_i,\dots f_{i-1,j-sv_i}+sw_i)\cdots \text{Formula A}
$$

选一个 $i$ 之后的子集 $f_{i,j-v_i}$：

$$
f_{i,j-v_i} = \max(f_{i-1,j-v_i},f_{i-1,j-2v_i}+w_i,\dots,f_{i-1,j-sv_i}+(s-1)w_i)\cdots \text{Formula B}
$$

后面的式子也是一样递推定义，我们现在只考虑这两个式子 $\text{A,B}$。

发现括号里的式子非常的类似，给式子 $\text{B}$ 的 $\max$ 中的每一项加上 $w_i$ 就变成了式子 $\text{A}$，

所以可以有：

$$
f_{i,j}=\max \begin{cases}f_{i,j-v_i}+w_i & j \ge v_i \\ f_{i-1,j} & \text{otherwise.}\end{cases}
$$

注意第一种情况是 $f_{i,j-v_i}$ 而不是 $f_{i-1,j-v_i}$。

类似于01背包，它可以化成一维，不过因为每个物品可以无限使用，并且状态可能在同阶段进行转移。所以要使用**正序循环**。（和01背包略有不同，画图即可）

```cpp
    for(register int i=1;i<=n;++i){
        for(register int j=v[i];j<=m;++j){
            f[j]=max(f[j],f[j-v[i]]+w[i]);
        }
    } return printf("%d\n",f[m]),0;
```

## 多重背包

> 给你 $n$ 种物品，你有 $m$ 的容量，每种物品有 $c_i$ 个，体积 $v_i$，价值 $w_i$ ，问你能够获得的价值 Max

第一种方法是考虑直接把每种物品拆成 $c_i$ 个不同物品进行 01背包。

```cpp
    for(register int i=1;i<=n;++i){
        for(register int j=1;j<=c[i];++j){
            for(register int k=m;k>=v[i];--k){
                f[k]=max(f[k],f[k-v[i]]+w[i]);
            }
        }
    } return printf("%d\n",f[m]),0;
```

和完全背包的第一种暴力做法基本一样。

我们先把这种形式下的方程写出来：

$f_{i,j}$ 表示在前 $i$ 种物品里选，空间不超过 $j$ 的所有方案，属性为价值和 Max。

那么考虑枚举的就是每种物品选多少个 （$cnt$）。

所以有方程：

$$
f_{i,j}=\max\limits_{0 \le cnt \le c_i}\{f_{i-1,j-cnt\times v_i}+cnt\times w_i\}
$$

滚动数组之后可以得到方程（$f_{i-1,j}$ 直接被继承了，所以不用让 $cnt=0$ 了）：

$$
f_j = \max\limits_{1\le cnt \le c_i}\{f_{j-cnt\times v_i} + cnt \times w_i\}
$$

把式子展开：

$$
f_{j}=\max(f_{j},f_{j-v_i}+w_i,\dots,f_{j-c_i\times v_i}+c_i \times w_i) \cdots \text{Formula A}
$$

依旧观察 $f_{j-v_i}$ 项，看是否可以优化：

（把 $f_{j-v_i}$ 当作 $f_{j}$ 代入最上面的方程，此处先不考虑背包够不够用 ）

$$
f_{j-v_i}=\max\limits_{1 \le cnt \le c_i} \{f_{j-v_i-cnt\times v_i}+cnt\times w_i\}
$$

$$
\Rightarrow f_{j-v_i}=\max(f_{j-v_i},f_{j-2v_i}+w_i,\dots,f_{j-c_i\times v_i}+(c_i-1)\times w_i,f_{j-(c_i+1)v_i}+c_i\times w_i) \cdots \text{Formula B}
$$

然后你会发现，这里没法像完全背包那样优化了，为什么呢？因为式子 $\text{B}$ 不只是比式子 $\text{A}$ 每项少了 $w_i$。

因为式子 $\text{B}$ 的决策集合里没有 $f_{j}$ 这一项，却多了 $f_{j-(c_i+1)v_i}$ 这一项，

所以我们是没办法直接从式子 $\text{B}$ 推到式子 $\text{A}$ 去的。

但是注意到滚动数组之后，方程的样子很类似 $f_i = \max\limits_{L(i)\le j \le R(i)}{f_j + val(i,j)}$ 的形式。

我们又可以发现一个很有意思的性质：

[![LpIk1e.png](https://s1.ax1x.com/2022/04/08/LpIk1e.png)](https://imgtu.com/i/LpIk1e)

决策集合的变化长的咋这么像单调队列呢？

所以可以考虑滚动数组之后利用单调队列优化。

因为滚动数组之后是倒推的，所以被单调队列先弹出的是 $f_j$，进来的是的 $f_{j-(c_i+1)v_i}$。

然后你发现，对于同一个 $i$ 可以把所有的 $j$ 分成 $v_i$ 组，依据是除以 $v_i$ 的余数

+ $\overline{0}$ 剩余系：$\{0,v_i,2v_i\dots\}$
+ $\overline{1}$ 剩余系：$\{1,v_i+1,2v_i+1\}$
+ $\dots$
+ $\overline{v_i-1}$ 剩余系 $\{v_i-1,2v_i-1,3v_i-1\dots\}$

然后在每一个剩余系里面利用单调队列单独转移，可以发现他们互不干扰。

简单来说，对于每一个 $i$ ，循环余数 $u \in [0,v_i)$。

然后对于每一个固定的 $u$ ，倒序循环倍数 $p=\lfloor \frac{m-u}{v_i}\rfloor \to 0$

假设当前状态是 $j=u+p\times v_i$，那么 $f_{j}=\max\limits_{p-c_i\le k \le p-1}\{f_{u+k\times v_i}+(p-k)\times w_i\}$

下界是 $p-c_i$ 很好理解，因为你只能拿 $c_i$ 个，但是为啥上限是 $p-1$ 而不是 $p$ 呢？

因为这里滚动数组了，相当于 $f_{i,j}$ 直接继承了 $f_{i-1,j}$ 然后开始取 $\max$。

现在把 $i,u$ **看成定值**，也就是只考虑内层循环 $p$ 。倒序枚举的时候，如果 $p$ 减少 $1$ ，那么 $k$ 的范围也会相对向低方向滑动一个单位。

然后再看方程里面： $f_{u+k\times v_i}-kw_i+pw_i$ ，加号前面的部分是**关于 $k$ 的**，后半部分是**关于 $p$ 的**。

因为 $k$ 是在**单调递减**，所以维护一个决策点 $k$ 单调递减，队头为 $f_{u+k\times v_i}-kw_i$ 的**最大值**的单调队列。

每次单调队列的时候，先把队头超过 $p-1$ 的弹掉，然后取队头为最优决策进行转移。

最后把新加入的决策 $k=p-c_i-1$ 插入队列的尾部，检查单调性，排除冗杂。

为什么是 $k = p-c_i -1$ 呢？因为原来的下界是 $p - c_i$，而每次入队的都是 $cnt$ 小 $1$ 的，也就是倍数 $k$ 小 $1$ 的，自然就是 $p-c_i-1$。

因为里面两层的循环相当于遍历了 $[0,m]$，所以复杂度 $\text{O}(nm)$。

```cpp
/*
 * @Author: black_trees 
 * @Date: 2022-02-01 12:49:34 
 * @Last Modified by: black_trees
 * @Last Modified time: 2022-02-01 13:21:38
 */
#include<bits/stdc++.h>
using namespace std;

constexpr int si_n=1e3+10;
constexpr int si_v=2e4+10;
int n,m;
int v[si_n],w[si_n],c[si_n];
int f[si_v],q[si_v];

int main(){
    #ifndef ONLINE_JUDGE
        freopen("Input.txt","r",stdin);
        freopen("Output.txt","w",stdout);
    #endif
    scanf("%d%d",&n,&m);
    for(register int i=1;i<=n;++i){
        scanf("%d%d%d",&v[i],&w[i],&c[i]);
    }
    auto calc=[&](int u,int i,int k)->int{
        return f[u+k*v[i]]-k*w[i];
    };
    for(register int i=1;i<=n;++i){
        for(register int u=0;u<v[i];++u){
            int head=1,tail=0; memset(q,0,sizeof q);
            int mxp=(m-u)/v[i];
            // 从这开始，在第三层以内，认为 i 和 u 是固定的
            for(register int k=mxp-1;k>=max(mxp-c[i],0);--k){
                while(head<=tail && calc(u,i,q[tail])<=calc(u,i,k)) --tail;
                q[++tail]=k;
            } // 把初始的决策集合插入
            // 此处的决策集合可以理解为图中 "f[j]" 系的决策候选集合。
            for(register int p=(m-u)/v[i];p>=0;--p){
                while(head<=tail && q[head]>p-1) ++head; // 过期的弹掉
                if(head<=tail) f[u+p*v[i]]=max(f[u+p*v[i]],calc(u,i,q[head])+p*w[i]); // 单调队列非空的时候才可以有max来转移。
                if(p-c[i]-1>=0){
                    while(head<=tail && calc(u,i,q[tail])<=calc(u,i,p-c[i]-1)) --tail;
                    q[++tail]=p-c[i]-1;
                } // 如果新加入的决策小于0了，那么就不用加了（都没了还加啥）
            }
        }
    } return printf("%d\n",f[m]),0;
}
```

上面写的似乎有点瑕疵 （$f_{j-(c_i+1)v_i}$ 这个状态似乎不能存在），不过能理解“决策集合的单调变化”就行了。

## 分组背包

> 给你 $n$ 组物品，每组有 $c_i$ 个，体积分别为 $v_{i,j}$ ，价值分别为 $w_{i,j}$ ，你有 $m$ 的空间，每组最多选一个，求价值Max

发现就是01背包和多重背包的变体。

我们考虑先套上 01 背包的板子，然后在里面进行决策。

```cpp
for 组（物品）
  for 体积
    for 决策
      if 满足条件
        转移
```

不过这里一定要注意，$j$ 要全部扫一遍，因为决策里面的那几个物品的体积是会变的。

如果是二维状态，就在第三层循环外面先 `f[i][j]=f[i-1][j];`。

如果是一维状态，就只需要在第三层里多一个特判.

```cpp
    for(register int i=1;i<=n;++i){ // 枚举组
        for(register int j=m;j>=0;--j){ // 枚举给每一个组分多少空间
            // 需要保证每组只选一个，所以要像 01 背包一样倒序循环。
            for(register int k=1;k<=c[i];++k){ // 枚举这组选哪一个
                if(j>=v[i][k]) f[j]=max(f[j],f[j-v[i][k]]+w[i][k]);
            }
        }
    } return printf("%d\n",f[m]),0;
```

## 树上背包

> 物品之间具有依赖关系，且依赖关系组成一棵树的形状。如果选择一个物品，则必须选择它的父节点。
> 
> 你有 $n$ 个物品，$m$ 的空间，问价值 Max

我们可以很容易地写出状态：设 $dp_{u, i}$ 表示以 $u$ 为根的子树中，选不超过 $i$ 的空间的所有方案，属性为权值和最大值。

然后可以对集合进行一个划分：一半是选 $u$，一半是不选 $u$。

考虑对这两个部分各自转移，但实际上除了选/不选 $u$ 的决策以外，他们的决策转移方式是相同的。

发现转移只需要枚举分配给 $u$ 所有的儿子以及以它们为根的子树的空间 $j$，

然后对于每个儿子枚举一个 $k_v$，表示这个儿子 $v$ 以及它的子树分到的空间。

对于每个 $j$，合法的转移状态是一组满足 $\sum_v k_v = j$ 的 $k$。

这个 $\sum_v k_v = j$ 怎么满足呢？ 其实我们只需要关心当前扫描到的儿子分配了多少空间即可，因为我们并不关心当前答案是怎么来的。

所以两重循环就可以了。

最后还需要记得决策强制选 $u$。

```cpp
inline void dfs(int u,int fa){
    for(register int i=0;i<(int)g[u].size();++i){
        int ver=g[u][i];
        if(ver==fa) continue;
        dfs(ver,u);
    }
    for(register int i=0;i<(int)g[u].size();++i){
        int ver=g[u][i];
        if(ver==fa) continue;
        for(register int j=m-v[u];j>=0;--j){ // 枚举排除 u 自己的，总共可以往下分配的空间
            for(register int k=0;k<=j;++k){ // 可以给当前子树分配的空间
                f[u][j]=max(f[u][j],f[u][j-k]+f[ver][k]);
            }
        }
    }
    // 上面的循环也可以放到 dfs 后面去

    for(register int i=m;i>=v[u];--i){ // 强制选 u 的转移。
        f[u][i]=f[u][i-v[u]]+w[u];
    }
    for(register int i=0;i<v[u];++i){ // 连 u 的空间都无法满足，0
        f[u][i]=0;
    }
}
```

这实际上是一个分组背包问题。

对于每一个分配给 $u$ 的子树的空间 $j$ ：

物品组数是 $|son(u)|$，每组都有 $j - v_u$ 个物品，每组的每个物品 $k$ 的价值是 $dp_{ver, k}$，体积是 $k$。

背包总容积是 $j - v_u$（必须选 $u$），每组只能选一个物品（每个儿子只能选一个状态 $dp_{ver,k}$ 转移到 $dp_{u, j}$）。

最后问能取到的权值和的 $\max$。

所以枚举分配的空间那里需要**倒序**循环，保证每个物品只取一次。

## 状态的 “恰好”“至多”“至少”

最主要的区别就是他们的字面意思，使用对应状态的时候要想清楚这种对应的状态要怎么写。

初始值和终态要根据状态本身的**定义**来写。

这里假定需要求解的问题是普通的01背包，状态设计为二维，无滚动数组，只需要价值最大即可，没有恰好装满的条件。

+ 恰好
  
  完整的状态是：考虑从**前** $i$ 个物品里面选，恰好用 $j$ 的空间的所有方案，权值和的最大值。
  
  因为 $f_0$ 系的状态就是所有的考虑从前 $0$ 个物品里选（不选）的情况。
  
  所以在“恰好”的约束下，只有 $f_{0,0}=0$ ,也就是恰好用 $0$ 的空间才是合法的。
  
  其他的不合法条件都要设置为 $-\infty$ ，（不仅表示“不合法”，也是为了之后转移）
  
  所以会这么写：
  
  ```cpp
  memset(f,0xcf,sizeof f),f[0][0]=0;
  ```
  
   那么，在dp完了之后，我们需要扫描 $f_n$ 系的状态，因为装不满也有可能是最值（当前问题没有恰好的限制，如果有，直接输出终态即可）。
  
  （不论有没有滚动数组优化都需要）

+ 至多
  
  完整的状态是：考虑从前 $i$ 个物品里面选，用不超过 $j$ 的空间的所有方案，权值和的最大值。
  
  因为这里的限制是不超过，所以 $f_0$ 系的所有状态都是合法的。
  
  你想，你不选任何物品，那自然所有可能的空间的不会超过啊。
  
  所以初始值全部设置为 $0$。
  
  那么，在dp完了之后，直接输出终态 $f_{n,m}$ 即可，（毕竟状态设计的是“所有方案”）

因为至多和恰好都适用于 “最大”，所以放在上面，而至少适用于“最小”，所以单独分离（不过恰好也是可以做的，道理一样）

> 问题：你至少需要用 $j$ 的空间，求价值的最小值。

+ 至少 
  
  完整的状态是：考虑从前 $i$ 个物品当中选，至少用 $j$ 的空间的所有方案，权值和的最小值。
  
  状态转移方程是和上面差不多的，不过初始化和转移方式需要改变。
  
  因为当你不选的时候，只有 $f_{0,0}=0$ 是合法的，所以其他的设置为 $+\infty$。
  
  ```cpp
  memset(f,0x3f,sizeof f),f[0][0]=0;
  ```
  
  但是转移就略有不同了。
  
  如果说，你枚举到的 $j$ 没有办法满足当前 $v_i$ 的需求，不应该只是继承上一轮的状态。
  
  因为，在不考虑访问无效下标的情况下，$f_{i,j}=\min\{f_{i-1,j-v_i}\},(j-v_i<0)$ 也是合法的转移。
  
  至少需要负数的空间，那你不选也是满足的啊，$f$ 又是记录最小值，自然需要把 $f_{i-1,j-v_i}=0$ 啊。
  
  所以状态转移方程需要变成这样：（代码写的是二维背包不滚动的情况，其他道理一样）
  
  ```cpp
  memset(f,0x3f,sizeof f),f[0][0][0]=0;
  for(register int i=1;i<=n;++i){
      for(register int j=0;j<=O2_need;++j){
          for(register int k=0;k<=N_need;++k){
              f[i][j][k]=f[i-1][j][k];
              f[i][j][k]=min(f[i][j][k],f[i-1][max(0,j-a[i])][max(0,k-b[i])]+c[i]);
          }
      }
  } return printf("%d\n",f[n][O2_need][N_need]),0;
  ```
