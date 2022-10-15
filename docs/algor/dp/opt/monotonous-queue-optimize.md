## 泛化

这一类问题的常见特征是 1D1D， 也就是一维状态一维转移。

内层循环的**取值范围由外层循环决定**。

并且在外层循环变量**固定 或者 单调变化**的情况下，**内层循环所枚举的决策集合是单调变化的**。

不一定非的要头和尾增加（减少）的一模一样，只要保证经过的位置不会再被经过一次就好（甚至可以是头一直不动，尾一直增加）。

这时候，因为内层循环变量的取值区间单调变化，就可以利用单调队列的思想进行优化。

使用一个单调队列维护决策点的可能取值，并且**保持队头是最优解**，如果队头超过了限制，弹出，如果队尾新加入的元素比已经在队列里面的更加优秀，排除冗杂决策。

然后利用队头元素进行决策即可。

## 一般形式

这一类问题的一般方程形式：

$$f_i=\min\limits_{L(i)\le j \le R(i)}\{f_j+val(i,j)\}$$

其中 $L(i),R(i)$ 是关于 $i$ 的一次函数，用来限制决策点 $j$ 的范围（决策集合上下界）。

所以一般我们都直接让 $i$ 单调变化，然后考虑决策集合（$f_{i, j}$ 的决策集合是所有可以转移到它的合法状态组成的集合）上下界的变化。

但如果有两维状态的时候，通常这里就不只是关于 $i$ 的了，可能还会有 $j$ 在里面。

所以这种情况下需要固定 $i$ 去考虑决策集合的变化（两个变量搞在一起肯定难受）

$val$ 则是关于 $i,j$ 的一个多项式，并且可以把它**分成两个部分**，一个只与 $i$ 相关，一个只与 $j$ 相关。

前一部分在 $i$ 固定的时候是常量，后一部分利用单调队列维护即可。

## 例题

### 问题 1

> 单调队列优化多重背包

见《背包DP》。

### 问题 2

> [POJ1821 Fence]：你有 $P$ 个工匠，第 $i$ 个工匠只能粉刷包括 $S_i$ 这一段木板，长度为 $L_i$ 的区间，并获得 $L_i \times p_i$ 的报酬，现在有 $n$ 块木板，问可以获得的最大报酬。
>
> $1\le n \le 16000,1\le m\le 100$

首先把工匠按 $S_i$ 排序，使得每个工匠粉刷的木板在上一个的**后面**。

这样就可以方便的进行 DP。

设 $f_{i,j}$ 表示前 $j$ 块木板，前 $i$ 个人粉刷的所有情况，属性为报酬的最大值。

先考虑特殊的。

1. 如果第 $i$ 个人不粉刷，$f_{i,j}=f_{i-1,j}$
2. $j$ 空着不粉刷，$f_{i,j}=f_{i,j-1}$。

需要考虑空着不粉刷的情况是因为 $\sum L_i$ 有可能小于 $n$ （题目没有保证大于等于）。

因为DP一般是以last作为分界的，考虑last就可，前面的会被依次递推。

然后考虑枚举上一个人最后刷到了哪里，设这个位置为 $k$。

然后 $[k+1,j]$ 就必须是第 $i$ 个人来处理。

因为题目有限制，所以 $k$ 是有取值范围的。

可以得到方程：

$$f_{i,j}=\max\limits_{j-L_i\le k \le S_i-1}\{f_{i-1,k}+P_i\times(j-k)\},j \ge S_i$$

复杂度 $O(n^3)$ 不能接受，发现方程是 1D1D 的形式，且 $k$ 的变化下界在 $i$ 固定的时候是一个关于 $j$ 的一次函数，上界是一个常数。

那么就证明，当 $i$ 固定，$j$ 单调变化的时候，$k$ 也是单调变化的。

并且 $P_i\times(j-k)$ 是一个关于 $j,k$ 的多项式，还可以拆分成两部分（没有 $j,k$ 的乘积项）。

于是把这一部分拆开：

$$f_{i,j}=P_i\times j + \max\limits_{j-L_i\le k \le S_i-1}\{f_{i-1,k}-P_i\times k\},j \ge L_i$$

所以对于每一个 $i$，维护一个 $f_{i-1,k}-P_i\times k$ 的最大值的队列，并且保证 $k$ 单调递增。

开始的时候把初始的决策先插入进去，然后枚举 $j$，对于每一个 $f_{i,j}$ 取队头更新，并同步维护单调性以及合法性即可。

因为每个元素只会入队出队一次，复杂度 $\text{O}(nP)$。

??? note "Code"
    ```cpp
    #include<cstdio>
    #include<iostream>
    #include<algorithm>
    #include<cmath>
    #include<deque>
    using namespace std;
    const int si_n=16000+100;
    const int si_m=110;
    int n,m,i,j,k,f[si_m][si_n];
    struct node{
        int p,l,s;
    	bool operator < (const node &b)const{
    		return s<b.s;
    	}
    }a[si_n];
    deque<int>q; // 偷懒 deque, 数组写法也是一样的。
    inline void init(){
        ios::sync_with_stdio(false),cin>>m>>n;
        for(register int i=1;i<=n;i++) cin>>a[i].l>>a[i].p>>a[i].s;
        sort(a+1,a+1+n);
    }
    inline int cal(int i,int k){
        return f[i-1][k]-a[i].p*k;
    }
    inline void work(){
        for(register int i=1;i<=n;i++){
            for(register int k=max(0,a[i].s-a[i].l);k<a[i].s;k++){
                while(q.size() && cal(i,q.back())<=cal(i,k)) q.pop_back();
                q.push_back(k);
            } // 插入初始决策
            for(register int j=1;j<=m;j++) {
                f[i][j]=max(f[i-1][j],f[i][j-1]); // 特殊情况
                if(j>=a[i].s){
                    while(q.size() && q.front()<j-a[i].l) q.pop_front();
                    if(q.size()) f[i][j]=max(f[i][j],a[i].p*j+cal(i,q.front())); // 队列非空才可以转移
                } // 转移
            }
        } cout<<f[n][m]; return ;
    }
    
    int main(){
        init(),work();
        return 0;
    }
    ```

## 模板

一般单调队列优化转移的时候的模板：

```cpp
循环 
    插入初始决策
循环
    先把过时的删除（维护合法性）
    如果队列非空
    	转移，（决策）
        排除冗杂，加入新状态。（维护单调性）
        （后两步有的时候不需要，比如这个题）
```

