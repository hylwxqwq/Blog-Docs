
## 例题

### Cleaning Shifts

> 农夫约翰雇佣他的 $N$ 头奶牛帮他进行牛棚的清理工作。
>
> 他将全天分为了很多个班次，其中第 $M$ 个班次到第 $E$ 个班次（包括这两个班次）之间必须都有牛进行清理。
> 
> 这 $N$ 头牛中，第 $i$ 头牛可以从第 $a_i$ 个班次工作到第 $b_i$ 个班次，同时，它会索取 $c_i$ 的佣金。
> 
> 请你安排一个合理的清理班次，使得 $[M,E]$ 时间段内都有奶牛在清理，并且所需支付给奶牛的报酬最少。
>
> $1\le N \le 1e3, 0 \le M,E \le 86399, a_i,b_i \in [M,E]$

提取题目要素之后可以设计出一个状态：

$dp_i$ 表示 $[M,i]$ 这一段全部清理完的所有方案，属性为花费最小值。

可以考虑先按照右端点排序，从牛的工作时间的右端点进行转移，可以设计出方程：

$$dp_{b_i} = \min\limits_{a_i -1 \le j < b_i}\{dp_j\} + c_i$$

枚举 $j$ 找到最小值即可。

其实瓶颈就在于这个枚举的过程，所以考虑对这个枚举的过程进行优化。

发现决策集合就是 $\{dp_j\} \ | \ j \in [a_1 - 1, b_i]$，

观察它的变化，当 $i$ 增加时，$b_i$ 是严格不下降的，因为我们排了序。

但 $a_i$ 的变化很可能非常不均匀，所以我们似乎没法做类似单调队列这种直接维护决策集合的优化。

但是，这里是询问 $dp$ 在**某一段上的最小值**，且 $dp$ **随时会发生更新**。

所以可以利用直接线段树维护 $dp$ 数组（本质上是维护它的第一维）。

支持决策后单点修改，转移的时候区间询问 $\min$ 就行了。

并且，本题实际上可以不开 $dp$ 数组，直接在线段树上执行修改即可。

??? note "Code"
	```cpp
	#include <iostream>
	#include <cstdio>
	#include <cstring>
	#include <algorithm>
	const int N=2e5+10;
	const int INF=0x3f3f3f3f;
	using namespace std;
	int read()
	{
	    int x=0,f=0,c=getchar();
	    while(c<'0'||c>'9'){if(c=='-')f=1;c=getchar();}
	    while(c>='0'&&c<='9'){x=x*10+c-'0';c=getchar();}
	    return f?-x:x;
	}
	
	int t[N<<2];
	
	void add(int x,int l,int r,int pos,int val)
	{
	//  cout<<x<<" "<<l<<" "<<r<<" "<<pos<<" "<<val<<endl;
	    if(l==r){ t[x]=min(t[x],val); return;}//注意 最小值意义下的赋值 
	    int mid=(l+r)>>1;
	    if(pos<=mid) add(x*2,l,mid,pos,val);
	    else add(x*2+1,mid+1,r,pos,val);
	    t[x]=min(t[x*2],t[x*2+1]);
	}
	
	int query(int x,int l,int r,int L,int R)
	{
	    if(L<=l&&R>=r){ return t[x];}
	    int mid=(l+r)>>1;
	    int ans=INF;
	    if(L<=mid) ans=min(ans,query(x*2,l,mid,L,R));
	    if(R>mid) ans=min(ans,query(x*2+1,mid+1,r,L,R));
	    return ans;
	}
	
	struct Node
	{
	    int x,y,z;
	}p[N];
	bool cmp(Node x,Node y){return x.y<y.y;}
	int n,L,R,b[N],tot,f[N],cnt;
	int main()
	{
	    n=read(); L=read(); R=read();
	    for(int i=1;i<=n;i++) 
	    {
	        int x=read(),y=read(),z=read();
	        if(y<L||x>R) continue;
	        b[++tot]=x; b[++tot]=y;
	        b[++tot]=x+1; b[++tot]=y+1;
	        p[++cnt]=(Node){x,y,z};
	    }   
	    b[++tot]=L; b[++tot]=R;
	    sort(b+1,b+tot+1);
	    tot=unique(b+1,b+tot+1)-(b+1);
	
	    sort(p+1,p+cnt+1,cmp);  
	
	    for(int i=1;i<=cnt;i++)
	    {
	        p[i].x=lower_bound(b+1,b+tot+1,p[i].x)-b,
	        p[i].y=lower_bound(b+1,b+tot+1,p[i].y)-b;
	    }
	    L=lower_bound(b+1,b+tot+1,L)-b;
	    R=lower_bound(b+1,b+tot+1,R)-b;
	    memset(f,0x3f,sizeof f);memset(t,0x3f,sizeof t);
	    add(1,0,tot,L-1,0);
	    for(int i=1;i<=cnt;i++)
	    {
	        f[p[i].y]=min(f[p[i].y],query(1,0,tot,p[i].x-1,p[i].y-1)+p[i].z);//右边界
	        add(1,0,tot,p[i].y,f[p[i].y]); 
	    }
	    int ans=INF;
	    for(int i=R;i<=tot;i++)
	        ans=min(ans,f[i]);
	    if(ans==INF){puts("-1");return 0;}
	    printf("%d",ans);
	    return 0;
	}
	// 直接贺的，毕竟是嘴巴做的题（
	// 作者：juruoHBr
	// 链接：https://www.acwing.com/solution/content/92559/
	```

### The Battle Of Chibi

> 求一个长度为 $n$ 的序列 $A$ 的长度为 $m$ 的严格递增子序列个数。
>
> 答案对 $1e9+7$ 取模，$1\le n \le 1000, |a_i| \le 1e9$。

严格递增子序列这个东西类似 LIS，本题中多了 $m$ 这个限制。

所以就设 $dp_{i,j}$ 表示长度为 $i$，从 $1\sim j$ 选，由 $A_j$ 结尾的严格递增子序列个数。

根据“最后一个不同点”的划分依据，枚举上一个是从哪里转移过来的即可。

可以得到：

$$dp_{i,j} = \sum\limits_{a_{k} < a_j \operatorname{and} k < j} dp_{i-1,k}$$

初始化 $a_0 = +\infty,f_{0,0} = 1$，其余 $f$ 为 $0$。

发现当外层循环都固定的时候，如果 $j + 1$，那么 $k$ 的取值范围就会从 $[0,j)$ 变到 $[0,j+1)$。

那么决策集合就只会多 $dp_{i-1,j}$ 这个决策。

然后要做的就是在决策集合里查询所有满足 $a_j > a_k$ 的 $dp_{i-1,k}$ 的和。

发现直接枚举只需要不断判断 $a_j > a_k$ ，也就是**只在判断关键码的大小关系**。

所以可以在决策集合里按照 $a_i$ 排序，离散化一下，设  $a_j$ 离散化之后的值为 $val(a_j)$ 。

询问时只需要询问 $a_j$ 离散化之后的位置 $val(a_j)$ 的前缀和即可。

每次对于 $dp_{i,j}$ 的决策**进行完之后**，**再插入** $dp_{i-1,j}$ **这个决策**，以保证方程中 $k < j$ 这个条件被满足。

实际上就是类似可持久化 Trie 的“**依次插入**”的思想，以强制去掉（直接不加入它们）$j$ 后面的部分的方式使得 $k < j$ 始终被满足。 

这个过程可以使用平衡树。

不过，因为 $n$ 很小，所以可以对 $A$ 离散化，然后直接**建立一个树状数维护所有离散化后的位置**。

插入决策 $dp_{i-1,j}$ 的操作，就令 $j$ 这个位置加上 $dp_{i-1,j}$ 即可。

那么查询直接在树状数组上求出 $val(a_j)$ 这个位置的前缀和就行了，

不过需要注意要让 $a_0$ 离散化之后的值 $1$ 也被算到树状数组里去。

因为我们优化的前提是 “假定外层循环固定”，所以树状数组每次循环只会维护上一个阶段的所有状态。

也就是 $dp_{i - 1}$ 系的所有状态。

??? note "Code"
	```cpp
	
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	using namespace std;
	
	const int si = 1e3 + 10;
	constexpr int mod = 1e9 + 7;
	
	int n, m;
	int a[si];
	int dp[si][si];
	int t[si];
	inline int lowbit(int x) {
		return x & -x;
	}
	inline void add(int x, int v) {
		while(x <= n) {
			t[x] = (t[x] + v) % mod;
			x += lowbit(x);
		}
	}
	inline int que(int x) {
		int res = 0;
		while(x) {
			res = (res + t[x]) % mod;
			x -= lowbit(x);
		}
		return res;
	}
	
	void solve(int qwq) {
		memset(dp, 0, sizeof dp);
		cin >> n >> m;
		for(int i = 1; i <= n; ++i)
			cin >> a[i];
		std::vector<int> v;
		for(int i = 1; i <= n; ++i)
			v.push_back(a[i]);
		sort(v.begin(), v.end());
		v.erase(unique(v.begin(), v.end()), v.end());
	
		for(int i = 1; i <= n; ++i)
			a[i] = lower_bound(v.begin(), v.end(), a[i]) - v.begin() + 2;
		a[0] = 1;
	
		dp[0][0] = 1;
		for(int i = 1; i <= m; ++i) {
			memset(t, 0, sizeof t); 
			// 树状数组在每一轮只维护上一个阶段 i - 1 系列的状态。
			add(a[0], dp[i - 1][0]);
			// 初始决策。
			for(int j = 1; j <= n; ++j) {
				dp[i][j] = que(a[j] - 1);
				add(a[j], dp[i - 1][j]);
			}
		}
	
		int ans = 0;
		for(int i = 1; i <= n; ++i)
			ans = (ans + dp[m][i]) % mod;
		cout << "Case #" << qwq << ": " << ans << endl;
	}
	
	int main() {
		int T;
		cin >> T;
		int cnt = 0;
		while(T--)
			solve(++cnt);
		return 0;
	}
	```

## 泛化

DP 的优化大多都分为两种：

1. 对状态的优化
2. 对决策的优化

第一种主要是滚动数组，提取题目信息等层面的优化。

第二种就是斜率优化，DS优化，费用提前计算这种技巧性的优化。

其思想大多都是，在外层循环固定的条件下，

将利用枚举来在决策集合中转移的操作，优化为**直接在决策集合中找到最优/总和对应的状态或者信息**。

并根据决策集合的上下界变化，单调性质去选择对应的优化策略。

DS 优化主要用于**上下界不均匀的变化**（**插入**等变化方式），或者决策集合可能被**修改**而不是去除的情况。

上方的两道例题，恰好分别对应了这两种情况。

### 情况1

第一道是上下界不均匀变化（对右端点排序过后，左端点不一定是单调的）+ 决策集合中的元素可能随着转移被修改。

所以无法使用直接维护集合的方式，那么此时考虑的就是直接利用数据结构维护 $dp$ 数组的下标（某个维度）。

使用的数据结构需要根据方程本身的需求来定，比如方程需要区间修改，区间查询，就可以使用线段树。

如果方程需要查询第 $k$ 大，翻转区间，动态插入，就可以使用平衡树。

或者说 $dp$ 数组的下标太大难以维护，也可以考虑使用动态开点线段树等数据结构，或者先离散化。

以上这种情况很多时候都可以直接在数据结构上直接维护，$dp$ 数组都不用开，高级一点的就是两个数据结构之间相互滚动信息。

本情况还有一道经典题，是五月 Tricks 的 _Non-Equal Neighbors_。

### 情况2

第二道是看起来上下界均匀变化，但是决策集合是需要动态在任意位置插入（因为关键字是 $a_i$），所以也需要使用数据结构优化。

这种情况大多都是决策集合变化较为均匀，比如滑动窗口式，单调变化式。

但是决策集合里一般含有随时会变化的元素（比如 $dp$ 数组本身）。

所以导致单调队列等数据结构难以维护，此时就可以利用平衡树或者离散化后维护位置的树状数组/线段树来优化。

（其实维护位置本质上也是一种维护下标）

常见的技巧是决策后再插入新决策，以保证下标限制的成立。

并且这种情况的方程通常只依赖上一层的 $dp$ 值，数据结构维护的一般都是上一个阶段的值。

本情况还有两道经典题，是五月 Tricks 的 _The Bakery_，和四月好题的 _Optimal Partition_。