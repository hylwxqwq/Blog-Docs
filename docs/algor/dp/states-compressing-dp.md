## 泛化

思想就是把**一个表示“存在”的 “集合” 转换成一个二进制数**。

然后进行对应的转移。

## 具体细节

比如设 $f_{msk}$ 表示获得状态为 $msk$ 的物品所需的代价。

其中如果 $msk$ 的第 $i$ 位是 $1$ ，那么表示第 $i$ 个物品已经被取得。

通常需要对 $msk$ 进行一些位运算的操作：

1. 检查 $msk$ 的第 $i$ 位是不是 $1$ ：
   + `(msk>>(i-1)&1)==1` 则为 $1$。
   + `(msk&(1<<(i-1)))!=0` 则为 $1$。
2. 将 $msk$ 的第 $i$ 位设置为 $1$ ：`msk|=(1<<(i-1))`
3. 将 $msk$ 的第 $i$ 位设置为 $0$ ：`msk&=(~(1<<(i-1)))`
3. 将 $msk$ 的第 $i$ 位取反：`msk^=(1<<(i-1))`。

其它的可以看位运算的部分。比如 `lowbit` 和 `popcnt` 之类的。

枚举状态一般这么写：

```cpp
for(register int msk=0;msk<(1<<n);++msk) // n 是位数。
```

一种常见的的优化复杂度的方式就是把合法的状态（决策）全部处理出来存到一个辅助数组里面。

循环的时候就可以这样：

```cpp
for(register int i=1;i<=cnt;++i) // cnt 是合法状态个数。
```

普通状压一般分两种，一种是基于联通性的状压DP（棋盘类），一种是集合类的状压DP。

前一种的典型就是“[POJ2411]莫德里安的梦想“，“[SCOI2005]互不侵犯”和“[NOI2001]炮兵阵地”。

这类问题一般都需要处理每一行的合法状态，然后进行转移，转移的阶段**一般**都是 “行”。

后一种的典型就是“[NOIP2016]愤怒的小鸟”，“[NOIP2017]宝藏”

这类问题一般需要记录当前状态已经处理了哪些事件，转移的时候**一般**以状态 $msk$ 作为阶段。

这两种的共同点就是，**某个变量的数据范围一般会很小**。

## 例题

### [P1879 [USACO06NOV]Corn Fields G](https://www.luogu.com.cn/problem/P1879)

棋盘类，最好写也是最经典的状压题之一。

题意：要求你在 $n \times m$ 的矩阵上放一些物品，有些位置不能放，你不能让两个物品挨着，求方案数并取模。

$n,m \le 13$

首先发现这一题的 $n,m$ 都是 $\le 13$ 的。

所以我们考虑状压。

先考虑没有不能放的限制，我们用二进制**预处理**出一行里所有的**可行**状态 $sta$。

这样子可以少枚举一层，不然会爆炸。

如果说我们处理出来的情况是我们处理到的那一行的原来的状态的子集，那么这个状态就是可行的。

意思就是说，比如你是这样子的：

```
原来的状态： 1 0 1 1 0 0 1 0 0 1 1 1
处理的状态： 1 0 1 0 0 0 1 0 0 1 0 1
```

也就是说，处理的状态当中放了草的位置在原来的地方都是可以种草的。那么就是可行的。

然后我们设 $f_{i,j}$ 表示考虑第 $i$ 行，你考虑处理出来的第 $j$ 个状态的方案数。

如果说这第 $j$ 个状态是可行的，那么这里就会有方案。

反之如果不可行，那么就不会转移到它，方案数是 $0$。

考虑枚举上一行的所有可行状态 $k$ ，然后方程就是 $f_{i,j}=f_{i,j}+f_{i-1,k} , \text{if} \ sta(j)\& sta (k) =0$。

$\ sta(j)\& sta (k) =0$ 是因为你需要判断上下有没有相邻的。

处理可行状态 $sta$ 的话只需要枚举所有的 $2^n$ 个状态，看他有没有两位是相邻的即可。 

??? note "Code"
	```cpp
	
	#include<bits/stdc++.h>
	using namespace std;
	
	const int si=14;
	const int stasi=4096+10; // 可行状态一定在 2n 范围以内.
	const int bitsi=4096+10;
	const int p=100000000;
	inline int mod(int x,int p){ return x<0?(x+p)-(((x+p)/p)*p):x-((x/p)*p);}
	int n,m,cnt=0;
	int f[si][stasi];
	int sta[stasi],yard[si];
	
	inline void init(int n){
		for(register int i=0;i<=n;++i){ // 不要忘了都不放 (0) 也是可行的
			if((i&(i<<1))!=0 || (i&(i>>1))!=0) continue;// 记得打括号
			sta[++cnt]=i; // 合法状态
			// printf("%d\n",sta[cnt]);
		}
	}
	inline bool valid(int l,int s){
		if(!((yard[l]&sta[s])==sta[s])) return false; // 状态符合第 l 行的情况
		return true;
	}
	
	int main(){
		memset(f,0,sizeof f),scanf("%d%d",&n,&m);
		for(register int i=1;i<=n;++i){
			for(register int j=1,k;j<=m;++j){
				scanf("%1d",&k);
				if(k) yard[i]+=(1<<(m-j)); // 把每一行的状态转成一个二进制数
			}
			// printf("%d\n",yard[i]);
		} init((1<<m)-1); // 去掉最后的的一个，不然会多一个状态。
		for(register int i=1;i<=cnt;++i){
			if(valid(1,i)) f[1][i]=mod(1,p); // 不要忘记这里也要判合法
		}
		for(register int i=2;i<=n;++i){
			for(register int j=1;j<=cnt;++j){ // 枚举当前层状态
				if(!valid(i,j)) continue; // 状态是否符合当前行的情况
				for(register int k=1;k<=cnt;++k){ // 枚举上一层状态
					if((sta[j]&sta[k])!=0) continue; // 上下不合法
					f[i][j]=mod(f[i][j]+f[i-1][k],p);
				}
			}
		} int res=0;
		for(register int i=1;i<=cnt;++i){
			res=mod(res+f[n][i],p);
		} return printf("%d\n",mod(res,p)),0;
	}
	```

### [POJ2411 **Mondriaan's Dream**](http://poj.org/problem?id=2411)

棋盘类。

给你一个 $n\times m$ 的矩阵。

你可以用 $1\times 2$ 的长方形去填充它，可以竖着也可以横着。

问恰好填满的方案数。

$1\le n,m\le 11$ 。

仔细思考可以发现，如果当前只考虑第 $i$ 行，那么无非就是三种情况：

1. 用 $1\times 2$ 的（`==`） 填充
2. 用 $2\times 1$ 的上一半填充。
3. 用 $2\times 1$ 的下半部分填充（并且上一行的对应位置是 $2\times 1$ 的上一半）。

最麻烦的就是第二种情况。

所以考虑状压，设 $msk$ 表示某一行的状态，第 $i$ 位为 $1$ 则表示对应的位置上放的是 $2\times 1$ 的上一半。

所以，上一行的状态 $msk_{i-1}$ 要想转移到当前行 $msk_i$，必须满足 $msk_{i-1} \operatorname{and} msk_i =0$。

但是也需要考虑第一种情况放不放的了。那么先把上一行的状态 $\operatorname{or}$ 过来，那么下半部分的位置就确定了。

因为是 $1\times 2$ 的，所以上下两个状态进行按位或之后，需要满足：任意 $0$ 的连通块里，$0$ 的个数是偶数个。

那么先预处理所有行内合法的状态。

然后枚举行，再枚举上一行的状态和当前行的状态进行转移即可。  

??? note "Code"
	```cpp
	#include<bits/stdc++.h>
	using namespace std;
	
	#define int long long
	constexpr int si=20;
	constexpr int stasi=1<<12;
	int n,m;
	int f[si][stasi];
	bool vis[stasi];
	
	signed main(){
		while(scanf("%lld%lld",&n,&m)!=EOF && n && m){
			for(register int msk=0;msk<(1<<m);++msk){
				bool ff=false,cnt=false;
				for(register int i=1;i<=m;++i){
					if(msk>>(i-1)&1) ff|=cnt,cnt=false;
					else cnt^=1;
				} vis[msk]=ff|cnt?0:1;
			} memset(f,0,sizeof f),f[0][0]=1;
			for(register int i=1;i<=n;++i){
				for(register int j=0;j<(1<<m);++j){
					for(register int k=0;k<(1<<m);++k){
						if((j&k)!=0 || !vis[j|k]) continue;
						f[i][j]+=f[i-1][k];
					}
				}
			} printf("%lld\n",f[n][0]);
		} return 0;
	}  
	```