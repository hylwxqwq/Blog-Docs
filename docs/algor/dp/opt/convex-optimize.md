## 泛化

这一类问题和单调队列优化DP有共同的地方。

可以用类似的思想思考其中的一部分内容。

考虑单调队列优化DP里面那个 1D1D 的式子。

$$f_{i}=\min\limits_{L(i)\le j \le R(i)}\{f_j+val(i,j)\}$$

其中 $val(i,j)$ 是关于 $i,j$ 的多项式。

并且可以分裂为两个分别只和 $i,j$ 相关的部分。

但如果 $val(i,j)$ 当中有**同时和** $i,j$ **相关的部分**（乘积和做除法），那么单调队列优化DP就不再适用。

此时就需要使用斜率优化DP。

而决策集合的上下界 $[L(i),R(i)]$ 的变化则决定了辅助斜率优化使用的算法。

如果决策集合是**单调变化**的，可以直接使用**单调队列**。

否则需要根据情况使用**二分**或者**平衡树**，这里的选择则是取决于决策集合的变化是在**头尾放入**还是在**任意位置插入**。

## 例题

> $n$ 个任务排成一个序列在一台机器上等待完成（顺序不得改变），这 $n$ 个任务被分成若干批，每批包含相邻的若干任务。  
> 从零时刻开始，这些任务被分批加工，第 $i$ 个任务单独完成所需的时间为 $t_i$。在每批任务开始前，机器需要启动时间 $s$，而完成这批任务所需的时间是各个任务需要时间的总和（同一批任务将在同一时刻完成）。  
> 每个任务的费用是它的完成时刻乘以一个费用系数 $c_i$。请确定一个分组方案，使得总费用最小。

### 做法1 - 暴力

> 保证 $n \le 500,1\le s \le 50,1\le t_i,c_i \le 100$

考虑最暴力的方式，设 $f_{i,j}$ 表示前 $i$ 个任务，分成 $j$ 批执行的所有方案，属性是总费用最小。

枚举上一批任务的最后一个点即可。

$f_{i,j}=\min\limits_{0\le k < i}\{f_{k,j-1}+c[k+1\sim i]\times (t[k+1\sim i]+s\times j)\}$

处理前缀和可以做到 $\text{O}(n^3)$

### 做法2 - 费用提前计算

> 保证 $n\le 5000,1\le s \le 50,1\le t_i,c_i \le 100$

发现上一个方程的瓶颈在于 $j$ ，需要知道启动了多少次，因为 $s$ 造成的贡献会和启动次数成正比。

发现如果一批任务确定被分在一起，那么之后的所有任务的结束时间自然都会增加 $s$。

所以可以在转移的时候先把这个 $s$ 加到后面（费用提前计算）

因为题目没有分多少组的限制，设 $f_i$ 表示把前 $i$ 个任务分成若干个批次的所有方案，属性为最小花费。

枚举上一组的最后端点可以得到：

$f_i=\min\limits_{0\le j < i}\{f_j+s\times c[j+1\sim n]+t[1\sim i] \times c[j+1 \sim i]\}$

预处理前缀和可以做到 $\text{O}(n^2)$

### 做法3 - 斜率优化

> 保证 $n \le 3\times 10^5,1\le s \le 512,1\le t_i,c_i \le 512$

要求时间复杂度 $\text{O}(n)$。

发现这个方程就是 1D1D 的形式，用前缀和的方式重新写一下这个方程：

$f_i=\min\limits_{0\le j < i}\{f_j+s\times (sumc[n]-sumc[j])+sumt[i]\times(sumc[i]-sumc[j])\}$ 

发现后面的 $val(i,j)$ 这个多项式是 

$s\times (sumc[n]-sumc[j])+sumt[i]\times(sumc[i]-sumc[j])$

把 $i$ 相关项， $j$ 相关项，$i,j$ 相关项分离：

常项：$s\times sumc[n]$

$i$ 相关项： $sumt[i]\times sumc[i]$

$j$ 相关项：$-s\times sumc[j]$

$i,j$ 相关项： $-sumt[i]\times sumc[j]$

把外层循环固定时候的常量提出去

$f_i=s\times sumc[n]+sumt[i]\times sumc[i]+\min\limits_{0 \le j < i}\{f_j -s\times sumc[j]-sumt[i]\times sumc[j]\}$

把 $\min$ 去掉，把关于**决策点** $j$ 的看作**变量**。

$f_j=(s+sumt[i])\times sumc[j]+f_i-sumt[i]\times sumc[i]-s\times sumc[n]$

如果把 $f_j$ 当作 $y$ ，$sumc[j]$ 看作 $x$。

可以得到一条直线 $y=kx+b$

其中 $b=f_i-sumt[i]\times sumc[i]-s\times sumc[n]$

> **实际上这条直线应当是这样的形式**：
>
> $y$ 是只和 $j$ 相关的项，然后 $kx$ 就是把 $i\times j$ 这种项放过来，其中 $k$ 对应 $i$ 的一次项，$x$ 对应 $j$ 的一次项。
>
> 剩下的部分就是 $b$ 了。
>
> 如果遇到两个 $i\times j$ 项，可以考虑把他们拆开，然后再分别放到对应位置去。
>
> 比如 $a_ia_j+b_ib_j$ 这种，就把他拆成 $(a_i+b_i)(a_j+b_j)-a_ib_i-a_jb_j$。
>
> 第一部分放到 $kx$ 项，第二部分放到 $b$ 项，第三部分放到 $y$ 项即可。

> 可以发现，斜率优化处理 $i\times j$ 的方式就是把它当作直线方程中的 $kx$。

把 $j=0\to n-1$ 的 **每个**二元组 $(sumc[j],f_j)$ 当作平面直角坐标系上的一个点 $(x,y)$（这就是要去掉 $\min$ 的原因）

（实际上就是把直线中 $x$ 位置的式子当作坐标系里的 $x$ 坐标，$y$ 位置的式子当作坐标系里的 $y$ 坐标）

我们可以得到这样的一个东西：

[![Convenxtrick-1.png](https://s4.ax1x.com/2022/02/23/b99NUH.png)](https://imgtu.com/i/b99NUH)

发现直线 $y=kx+b$ 的斜率是 $(s+sumt[i])$

是一个关于 $i$ 的变量，也就是说，对于不同的 $i$ ，我们得到的斜率是不一样的。

当然如果出现 $sumt[i]$ 先增再减少导致有两个不同的 $i$ 的 $sumt[i]$ 一样，是没有影响的。

回想一下问题：最小化 $f_n$，对应到每个决策的问题就是最小化 $f_i$。

而 $f_i$ 处于截距 $b$ 之中，当 $y,x,k$ 固定的时候，只需要让 $b$ 最小，$f_i$ 就最小。

所以对于每一个 $i$ ，拿一条斜率为 $k$ 的直线从下往上平移，遇到的第一个点一定是最优解。

 [![convextrick-2.png](https://s4.ax1x.com/2022/02/23/b9PWj0.png)](https://imgtu.com/i/b9PWj0)

但是就算这样，还是需要枚举所有可能的 $j$ ，复杂度依旧是 $n^2$ 级别。

那么怎么优化？

随便找三个 $x$ 坐标递增的点 $i,j,k$

连接 $i\to j,j \to k$

发现只可能有如下的两种情况（三点共线的不算）。

[![3.png](https://s4.ax1x.com/2022/02/23/b9i4xI.png)](https://imgtu.com/i/b9i4xI)

发现第二种情况，$j$ 无论如何都不可能成为最优决策。

形式化的描述就是 $k_{i,j}>k_{j,k}$ ，那么 $j$ 不可能成为最优决策。

所以可以处理出所有的不可能成为最优决策的决策点，然后转移就在剩下的部分进行即可。

实际上，在本题就是在维护一个 $x$ 单调递增，$k$ 单调递增的 **下凸壳**，

[![4.png](https://s4.ax1x.com/2022/02/23/b9F3JH.png)](https://imgtu.com/i/b9F3JH)

具体如何维护这个下凸壳呢？

考虑从 $0 \to n-1$ 暴力扫一次，每次新增一个点都需要把它和上一个被加入的点，上上个被加入的点相互之间连边，如果这个点更优，去掉上一个，把这个点放进来。

GIF：

[![b9ECeP.gif](https://s4.ax1x.com/2022/02/23/b9ECeP.gif)](https://imgtu.com/i/b9ECeP)

因为在本题，斜率在 $i$ 增加的时候一定增加（因为 $sumt[i]$ 肯定单调上升）

所以可以利用一个单调队列，把前面已经决策过的删除，在后面进行决策。

也就是队头和队头加一的斜率小于 $s+sumt[i]$，就把队头弹出。

??? note "Code"
    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
    constexpr int si=3e5+10;
    
    #define ll long long
    ll f[si],sumt[si],sumc[si];   
    int q[si],n,S;
    
    int main() {   
        scanf("%d%d",&n,&S);  
        for(register int i=1;i<=n;i++){   
            int t,c; scanf("%d%d",&t,&c);   
            sumt[i]=sumt[i-1]+t,sumc[i]=sumc[i-1]+c;   
        } memset(f,0x3f,sizeof(f)),f[0]=0;   
        int head=1,tail=1; q[1]=0;   
        for(register int i=1;i<=n;i++){   
            while(head<tail && (f[q[head+1]]-f[q[head]])<=(S+sumt[i])*(sumc[q[head+1]]-sumc[q[head]])) head++;   
            f[i]=f[q[head]]-(S+sumt[i])*sumc[q[head]]+sumt[i]*sumc[i]+S*sumc[n];   
            while(head<tail && (f[q[tail]]-f[q[tail-1]])*(sumc[i]-sumc[q[tail]])>=(f[i]-f[q[tail]])*(sumc[q[tail]]- sumc[q[tail-1]])) tail--;   
            q[++tail]=i;   
        } return printf("%lld\n",f[n]),0;
    }
    ```

我这份代码是写的维护 $[head, tail)$ 这个半闭半开区间。

所以初始的时候 `head = 1, tail = 1`。

但是我如果直接改成维护闭区间 $[head, tail]$ 就会寄。

可能是代码移动端点的时候出了细节问题，之后再看吧。

复杂度 $\text{O}(n)$。

---

这题的 $\text{O}(n^2)$ 做法还有另外一个方程，

就是把当前批次任务的贡献带着 $S$ 也直接往后面加了：

$$dp_{i} = \min\limits_{0\le j < i}\{dp_j + (S + sumt[i] - sumt[j]) \times (sumc[n] - sumc[j])\}$$

这个也是可以斜率优化的，用上面总结的找直线方程的每一部分的方法即可。

性质也差不多。

### 做法4 - 二分 + 斜率优化

> 条件同3，$t_i$ 可能是负数。

发现 $sumt$ 不再是单调上升的，所以不能利用单调队列直接弹掉队头冗杂。

决策的时候就需要在整个凸壳上二分。

只需要找到这样的一个节点即可：

如果某个顶点左侧的线段斜率比 $k$ 小，右侧比 $k$ 大，那么这个点就是最优决策。

??? note "Code"
    ```cpp
    #include<bits/stdc++.h>
    using namespace std;
    constexpr int si=3e5+10;
    
    inline __int128 read(){
        __int128 x=0,f=1; char ch=getchar();
        while(ch<'0'||ch>'9'){
            if(ch=='-') f=-1;
            ch=getchar();
        }
        while(ch>='0'&&ch<='9') x=(x<<1)+(x<<3)+(ch^48),ch=getchar();
        return x*f;
    }
    inline void write(__int128 x){
        if(x<0) putchar('-'),x=-x;
        if(x>9) write(x/10);
        putchar(x%10+'0');
    }
    
    __int128  f[si],sumt[si],sumc[si];   
    int q[si],n,S,head,tail;
    inline int Mylower_bound(int slope){
        if(head==tail) return q[head];
        int l=head,r=tail;
        while(l<r){
            int mid=(l+r)>>1;
            if(f[q[mid+1]]-f[q[mid]]<=slope*(sumc[q[mid+1]]-sumc[q[mid]])) l=mid+1;
            else r=mid;
        } return q[l];
    }
    
    int main() {   
        scanf("%d%d",&n,&S);  
        for(register int i=1;i<=n;i++){   
            __int128 t,c; t=read(),c=read();  
            sumt[i]=sumt[i-1]+t,sumc[i]=sumc[i-1]+c;   
        } memset(f,0x3f,sizeof(f)),f[0]=0;   
        head=1,tail=1,q[1]=0;   
        for(register int i=1;i<=n;i++){  
            int qwq=Mylower_bound(S+sumt[i]);
            f[i]=f[qwq]-(S+sumt[i])*sumc[qwq]+sumt[i]*sumc[i]+S*sumc[n];   
            while(head<tail && (f[q[tail]]-f[q[tail-1]])*(sumc[i]-sumc[q[tail]])>=(f[i]-f[q[tail]])*(sumc[q[tail]]- sumc[q[tail-1]])) tail--;   
            q[++tail]=i;   
        } return write(f[n]),0;
    } // 本题会爆 long long, use __int128.
    ```

### 做法5 - 平衡树 + 斜率优化

> 如果 $c_i$ 也有可能是负数？

需要在凸壳的任意位置插入，查询，所以需要平衡树。

还不太会。
