## 定义

矩阵是啥应该不用说了吧。

一般表示的时候用大写字母表示矩阵。用小写字母加一个二元组下标表示矩阵里的元素。

比如 $A$ 为一个 $n \times m$ 的矩阵：

这里 $n\times m$ 指 $n$ 行 $m$ 列。

$$A=\begin{bmatrix} a_{1,1} & \cdots & a_{1,m} \\ \vdots & \ddots & \vdots \\ a_{n,1} & \cdots & a_{n,m}\end{bmatrix}$$

$a_{i,j}$ 就表示矩阵 $A$ 当中第 $i$ 行第 $j$ 列的元素。

（线性代数里不是经常说行列式吗，所以是 行 $\times$ 列 啊（划掉））

也可以简记为 $A=(a_{i,j})$。

向量：注意这里的向量和几何里的向量的不同。

一般把 $n$ 个实数组成的 $n$ 元组称为向量。

如果它表示为一个 $1\times n$ 的矩阵，则称为行向量，如果是 $n \times 1$ ，则称为列向量：

行向量：$(a_1,a_2,a_3,...,a_n)$

列向量：$\begin{bmatrix}a_1 \\ a_2 \\ \vdots \\ a_n \end{bmatrix}$

一般都是用列向量（方便一点），一般会用黑体斜体表示列向量。

有兴趣的可以了解一下 $n$ 维欧几里德空间（可以考虑看看《线性代数》）。

单位矩阵（$I$）：

对于一个 $n \times n$ 的矩阵 $A$ ，如果满足 $\forall i ,a_{i,i}=1,\text{Others}=0$，那么称 $A$ 是一个单位矩阵，一般记作 $I$

比如：

$$I=\begin{bmatrix}1 &0 & 0 \\ 0& 1 & 0 \\ 0 & 0& 1\end{bmatrix}$$

## 标量乘法

$\alpha$ 是一个标量，$A$ 是一个 $n \times m$ 的矩阵，

则 $\alpha A=B$ 是一个 $n \times m$ 的矩阵，且 $B_{i,j}=\alpha\times A_{i,j}$。

满足结合律，分配律。

## 矩阵加法

$A$ 是一个 $n \times m$ 的矩阵，$B$ 是一个 $n\times m$ 的矩阵，

则他们进行矩阵加法 $A+B$ 得到的结果矩阵 $C$ 是一个 $n \times m$ 的矩阵。

且 $C_{i,j}=A_{i,j}+B_{i,j}$。

满足结合律，交换律。

## 矩阵乘法

重头戏。

$A$ 是一个 $n \times m$ 的矩阵，$B$ 是一个 $m \times k$ 的矩阵。

注意，$A$ 的列数和 $B$ 的行数必须相等！

那么他们进行矩阵乘法 $A \times B$ 的到的结果矩阵 $C$ 是一个 $n \times k$ 的矩阵。

且满足：

$$C_{i,j}=\sum\limits_{k=1}^m A_{i,k}\times B_{k,j}$$

形象的解释就是，$C_{i,j}$ 等于 $A$ 的第 $i$ 行和 $B$ 的第 $j$ 列一一对应地乘起来。

注意：矩阵乘法**不一定**满足交换律！！

但是它满足**结合律**。

```cpp
struct Matrix{
    int a[si][si];
    Matrix(){ memset(a,0,sizeof a); }
    inline Matrix operator * (const Matrix &B)const{
		Matrix C,A=*this;
        for(register int i=1;i<=cnt;++i){
			for(register int j=1;j<=cnt;++j){
				for(register int k=1;k<=cnt;++k){
					C.a[i][j]+=A.a[i][k]*B.a[k][j];
                }
            }
        } return C;
        // 最好循环的时候不要用 si。
        // 用一个设定好的常数或者题目给的变量会比较好。
        // 但是如果乘法不止需要适用于一对 n,m,k，那么就最好用 si - 1。
        // 为啥不会有影响呢？因为构造函数里把没有用到的设置成 0 了。
    }
};
```

如果被卡常了，可以考虑手动展开内层循环。

要求取模的话手动加上就行。

## 矩阵快速幂

因为矩阵乘法满足结合律，所以一个矩阵的 $k$ 次幂定义为：

$$A^k=\begin{matrix}\underbrace{A \times A\times A \dots \times A}\\k \text{ times}\end{matrix}$$

因为所有满足结合律的元算都可以使用快速幂求解。

所以可以利用矩阵乘法的结合律写出一个矩阵快速幂算法：

```cpp
Matrix Ans,A;
inline Matrix Qpow(int b){
	for(;b;b>>=1){
		if(b&1) Ans=Ans*A;
        A=A*A;
    } return Ans;
} // 有的时候根据情况需要初始化一下 Ans.
```

## 矩阵乘法优化递推

> 给你一个数字 $n$，求出 $Fib_n \text{ mod }998244353 ,n \le 1e18$。
>
> $Fib$ 是斐波那契数列。

看到 $n$ 的范围发现直接递推明显爆炸。

所以考虑把 $Fib_i,Fib_{i-1}$ 表示成一个行向量 $\begin{bmatrix} Fib_i & Fib_{i-1} \end{bmatrix}$

然后我们想把递推式子表示成矩阵乘法的形式再利用矩阵快速幂进行高速递推：

$\begin{bmatrix} Fib_{i-1} & Fib_{i-2} \end{bmatrix} \times ? = \begin{bmatrix} Fib_i & Fib_{i-1} \end{bmatrix}$ 

不妨设这个 $?$ 为一个矩阵 $base$。

根据矩阵乘法的定义，$base$ 应该是一个 $2\times 2$ 的矩阵。

考虑列出原来的递推式：$Fib_{n}=Fib_{n-1}+Fib_{n-2}$。

发现结果矩阵的 $(1,1)$ 这个位置是 $Fib_i$，而这个位置 $C_{1,1}$ 应该是等于 $A_{1,1}\times B_{1,1}+A_{1,2}\times B_{2,1}$

也就是 $Fib_{i-1} \times B_{1,1}+Fib_{i-2}\times B_{2,1}$ 

所以 $B_{1,1}$ 和 $B_{2,1}$ 都是 $1$ ：$\begin{bmatrix}1\\1\end{bmatrix}$

同理可以得到 $B_{2,1}$ 和 $B_{2,2}$ ：$\begin{bmatrix}1\\0\end{bmatrix}$

所以 $base=\begin{bmatrix}1 & 1\\ 1 & 0\end{bmatrix}$

原式可以化为 $\begin{bmatrix} Fib_{i-1} & Fib_{i-2} \end{bmatrix} \times \begin{bmatrix}1 & 1\\ 1 & 0\end{bmatrix} = \begin{bmatrix} Fib_i & Fib_{i-1} \end{bmatrix}$。

那么设 $Ans=\begin{bmatrix} 1 & 1 \end{bmatrix}=\begin{bmatrix} Fib_2 & Fib_1\end{bmatrix}$

所以 $Fib_n$ 就是 $Ans \times base^{n-2}$ 的 $(1,1)$。

写一个矩阵快速幂即可。

广义斐波那契数列也是一样的。

然后举一个 OI-Wiki 上的例子

$$ f_{1} = f_{2} = 0 \\ f_{n} = 7f_{n-1}+6f_{n-2}+5n+4\times 3^n $$

发现 $f_n$ 和 $f_{n-1}, f_{n-2}, n$ 有关，于是考虑构造一个矩阵描述状态。

但是如果矩阵仅有这三个元素 $\begin{bmatrix}f_n& f_{n-1}& n\end{bmatrix}$ 是难以构造出转移方程的，因为乘方运算和 $+1$ 无法用矩阵描述。

于是考虑构造一个更大的矩阵。

$$ \begin{bmatrix}f_n& f_{n-1}& n& 3^n & 1\end{bmatrix} $$

我们希望构造一个递推矩阵可以转移到

$$  \begin{bmatrix} f_{n+1}& f_{n}& n+1& 3^{n+1} & 1 \end{bmatrix} $$

转移矩阵即为

$$ \begin{bmatrix} 7 & 1 & 0 & 0 & 0\\ 6 & 0 & 0 & 0 & 0\\ 5 & 0 & 1 & 0 & 0\\ 12 & 0 & 0 & 3 & 0\\ 5 & 0 & 1 & 0 & 1 \end{bmatrix} $$

再举一个例子：$f(i) = (f(i - 1) + \dfrac{p}{100}) / 2$。

（要取模的）

先展开式子：$f(i) = \dfrac{1}{2} f(i - 1) + \dfrac{p}{200}$

如果初始矩阵是 $1\times 2$ 的感觉上完全不够，因为没法很好的处理这个“加上一个常数”的东西。

发现**转移矩阵里的数的本质是初始矩阵的数的某个系数**。

所以本着“**递推式里需要啥，就在初始矩阵里放啥**”的思想，我们放一个 $1$ 在初始矩阵里，然后每次转移都让 $1$ 的系数为 $\dfrac{p}{200}$ 即可。

初始矩阵：

$$\begin{bmatrix}f(i), f(i - 1), 1\end{bmatrix}$$

得到的矩阵：

$$\begin{bmatrix}f(i + 1), f(i), 1\end{bmatrix}$$

转移矩阵：

$$\begin{bmatrix} \frac{1}{2} & 1 & 0 \\ 0 & 0 & 0 \\ \frac{p}{200} & 0 & 1 \end{bmatrix}$$

## 矩阵的一些常见应用

### 恰好 K 条边 最短路

首先用邻接矩阵 $A$ 存图。

然后 $A[i,j]$ 就可以看做 $i \to j$ 经过恰好一条边的最短路。

考虑求出经过恰好两条边的最短路 $B$。

可以发现 $B[i,j]=\min\limits_{1\le k \le n}\{A[i,k]+A[k,j]\}$

这里就是用了类似 Floyd 的枚举中间点思想（其本质是 dp）。

类似的可以得到经过恰好 $K$ 条边的最短路。

设 $A^{r}$ 表示经过恰好 $r$ 条边的最短路矩阵。

可以得到 $A^{r}[i,j]=\min\limits_{1\le k \le n}\{A^p[i,k]+A^q[k,j]\},r=p+q$。

发现这个运算就是个类似于矩阵乘法的东西，我们将其定义为 $\oplus$。

把 $\sum$ 换成 $\min$ ，把 $\times$ 换成 $+$ 就能看出来。

刚好这个东西仍然满足结合律，我们把式子写一下：

$$
A^{r} = A^{r-1} \oplus A^1
$$

发现 $A^r$ 就等于 $A^1$ 在 $\oplus$ 意义下的 $r$ 次幂。

那么就可以用矩阵快速幂 $n^3\log K$ 求 $A^K$ 了。

### 矩阵表达修改

这里还是直接贴的 OI-Wiki 的例子，因为我懒得写了。

#### 「THUSCH 2017」大魔法师

??? note "题目描述"
	>    大魔法师小 L 制作了 $n$ 个魔力水晶球，每个水晶球有水、火、土三个属性的能量值。
	>
	>    小 L 把这 $n$ 个水晶球在地上从前向后排成一行，然后开始今天的魔法表演。
	>
	>    我们用 $A_i,\ B_i,\ C_i$ 分别表示从前向后第 $i$ 个水晶球（下标从 $1$ 开始）的水、火、土的能量值。
	>
	>    小 L 计划施展 $m$ 次魔法。每次，他会选择一个区间 $[l, r]$，然后施展以下 $3$ 大类、$7$ 种魔法之一：
	>
	>    1.  魔力激发：令区间里每个水晶球中 **特定属性** 的能量爆发，从而使另一个 **特定属性** 	的能量增强。具体来说，有以下三种可能的表现形式：
	>        - 火元素激发水元素能量：令 $A_i = A_i + B_i$。
	>        
	>        - 土元素激发火元素能量：令 $B_i = B_i + C_i$。
	>        
	>        -   水元素激发土元素能量：令 $C_i = C_i + A_i$。
	>            
	>            **需要注意的是，增强一种属性的能量并不会改变另一种属性的能量，例如 $A_i = A_i + B_i$ 并不会使 $B_i$ 增加或减少。	**
	>        
	>    2.  魔力增强：小 L 挥舞法杖，消耗自身 $v$ 点法力值，来改变区间里每个水晶球的 **特定属性** 	的能量。具体来说，有以下三种可能的表现形式：
	>        - 火元素能量定值增强：令 $A_i = A_i + v$。
	>        - 水元素能量翻倍增强：令 $B_i=B_i \cdot v$。
	>        - 土元素能量吸收融合：令 $C_i = v$。
	>        
	>    3. 魔力释放：小 L 将区间里所有水晶球的能量聚集在一起，融合成一个新的水晶球，然后送给场外观众。
	>
	>      生成的水晶球每种属性的能量值等于区间内所有水晶球对应能量值的代数和。**	需要注意的是，魔力释放的过程不会真正改变区间内水晶球的能量**。
	>
	>      值得一提的是，小 L 制造和融合的水晶球的原材料都是定制版的 OI 工厂水晶，所以这些水晶球有一个能量阈值 $998244353$	。当水晶球中某种属性的能量值大于等于这个阈值时，能量值会自动对阈值取模，从而避免水晶球爆炸。
	>
	>      小 W 为小 L（唯一的）观众，围观了整个表演，并且收到了小 L 在表演中融合的每个水晶球。小 W 想知道，这些水晶球蕴涵的三种属性的能量值分别是多少。

由于矩阵的结合律和分配律成立，单点修改可以自然地推广到区间，即推出矩阵后直接用线段树维护区间矩阵乘积即可。

下面将举几个例子。

$A_i = A_i + v$ 的转移

$$ \begin{bmatrix} A & B & C & 1 \end{bmatrix} \begin{bmatrix} 1 & 0 & 0 & 0\\ 0 & 1 & 0 & 0\\ 0 & 0 & 1 & 0\\ v & 0 & 0 & 1\\ \end{bmatrix}= \begin{bmatrix} A+v & B & C & 1\\ \end{bmatrix} $$

$B_i=B_i \cdot v$ 的转移

$$ \begin{bmatrix} A & B & C & 1\end{bmatrix}\begin{bmatrix}1 & 0 & 0 & 0\\0 & v & 0 & 0\\0 & 0 &1& 0\\0 & 0 & 0 & 1\\\end{bmatrix}=\begin{bmatrix}A & B \cdot v & C & 1\\\end{bmatrix}$$

??? note "Code"
	```cpp
	#include<cstdio>
	#include<iostream>
	#include<cstring>
	using namespace std;
	
	constexpr int mod=998244353;
	constexpr int maxn=260000;
	int n,m;
	
	template<class T>inline void read(T &a){
		register char ch;
		while (ch=getchar(),(ch<'0' || ch>'9') && ch!='-');
		register bool f=(ch=='-');
		register T x=f?0:ch-'0';
		while(ch=getchar(),ch>='0' && ch<='9') x=(x<<3)+(x<<1)+(ch^48);
		a=f?-x:x;
	}
	struct Matrix{
		int a[5][5];
		Matrix(){ memset(a,0,sizeof(a)); }
		inline void unit_init(){
			memset(a,0,sizeof(a));
			for(register int i=1;i<=4;i++) a[i][i]=1;
		}
		inline Matrix operator * (const Matrix& M){
			Matrix res;
			for(register int i=1;i<=4;i++){
				for(register int j=1;j<=4;j++){
					res.a[i][j]=(res.a[i][j]+1ll*a[i][1]*M.a[1][j])%mod;
					res.a[i][j]=(res.a[i][j]+1ll*a[i][2]*M.a[2][j])%mod;
					res.a[i][j]=(res.a[i][j]+1ll*a[i][3]*M.a[3][j])%mod;
					res.a[i][j]=(res.a[i][j]+1ll*a[i][4]*M.a[4][j])%mod;
				}
			} return res;
		}
		inline Matrix operator + (const Matrix &M){
			Matrix res;
			for(register int i=1;i<=4;i++){
				for(register int j=1;j<=4;j++){
					res.a[i][j]=(M.a[i][j]+a[i][j])%mod;
				}
			} return res;
		}
	}ans,unit,ex_unit;
	struct Segment_Tree{ Matrix Mat,tag; }t[maxn<<2];
	inline void init_1(){ unit.a[2][1]=1; }
	inline void init_2(){ unit.a[3][2]=1; }
	inline void init_3(){ unit.a[1][3]=1; }
	inline void init_4(int v){ unit.a[4][1]=v; }
	inline void init_5(int v){ unit.a[2][2]=v; } 
	inline void init_6(int v){ unit.a[3][3]=0,unit.a[4][3]=v; }
	inline void pushdown(int p){
		t[p<<1].tag=t[p<<1].tag*t[p].tag;
		t[p<<1|1].tag=t[p<<1|1].tag*t[p].tag;
		t[p<<1].Mat=t[p<<1].Mat*t[p].tag;
		t[p<<1|1].Mat=t[p<<1|1].Mat*t[p].tag;
		t[p].tag.unit_init();
	}
	inline void pushup(int p){
		for(register int i=1;i<=4;i++){
			t[p].Mat.a[1][i]=1ll*(t[p<<1].Mat.a[1][i]+t[p<<1|1].Mat.a[1][i]); 
			t[p].Mat.a[1][i]-=(t[p].Mat.a[1][i]>=mod)?mod:0;
		}
	}
	inline void built(int l,int r,int p){
		t[p].tag=ex_unit;
		if(l==r){
			read(t[p].Mat.a[1][1]);read(t[p].Mat.a[1][2]);read(t[p].Mat.a[1][3]);
			t[p].Mat.a[1][4]=1;
			return;
		} int mid=(l+r)/2;
		built(l,mid,p<<1),built(mid+1,r,p<<1|1),pushup(p);
	}
	inline void update(int l,int r,int ql,int qr,int p,Matrix M){
		if(ql<=l && r<=qr){
			t[p].Mat=t[p].Mat*M;
			t[p].tag=t[p].tag*M;
			return;
		} pushdown(p); int mid=(r+l)>>1;
		if(mid>=ql) update(l,mid,ql,qr,p<<1,M);
		if(qr>mid) update(mid+1,r,ql,qr,p<<1|1,M);
		pushup(p);
	}
	inline void query(int l,int r,int ql,int qr,int p){
		if(ql<=l && r<=qr){
			for(register int i=1;i<=3;i++){
				ans.a[1][i]=ans.a[1][i]+t[p].Mat.a[1][i];
				ans.a[1][i]-=(ans.a[1][i]>=mod)?mod:0;
			} return ;
		} int mid=(l+r)>>1; pushdown(p);
		if(mid>=ql) query(l,mid,ql,qr,p<<1);
		if(qr>mid) query(mid+1,r,ql,qr,p<<1|1);
	}
	
	int main(){
		ex_unit.unit_init(),read(n),built(1,n,1),read(m);
		for(register int i=1;i<=m;i++){
			unit=ex_unit; int opt,l,r,v;
			read(opt);read(l);read(r);
			if(opt==1) init_1(),update(1,n,l,r,1,unit);
			if(opt==2) init_2(),update(1,n,l,r,1,unit);
			if(opt==3) init_3(),update(1,n,l,r,1,unit);
			if(opt==4) read(v),init_4(v),update(1,n,l,r,1,unit);
			if(opt==5) read(v),init_5(v),update(1,n,l,r,1,unit);  
			if(opt==6) read(v),init_6(v),update(1,n,l,r,1,unit);  
			if(opt==7) memset(ans.a,0,sizeof(ans.a)),query(1,n,l,r,1),printf("%d %d %d\n",ans.a[1][1],ans.a[1][2],ans	.a[1][3]);
		} return 0;
	}
	```

#### 「LibreOJ 6208」树上询问

??? note "题目描述"
	>    有一棵 $n$ 节点的树，根为 $1$ 号节点。每个节点有两个权值 $k_i, t_i$，初始值均为 $0$。 
	>    给出三种操作：  
	>
	>    1. $\operatorname{Add}( x , d )$ 操作：将 $x$ 到根的路径上所有点的 $k_i\leftarrow k_i + d$
	>
	>    2. $\operatorname{Mul}( x , d )$ 操作：将 $x$ 到根的路径上所有点的 $t_i\leftarrow t_i + d \times k_i$
	>
	>    3. $\operatorname{Query}( x )$ 操作：询问点 $x$ 的权值 $t_x$   
	>
	>       $n,~m \leq 100000, ~-10 \leq d \leq 10$

若直接思考，下放操作和维护信息并不是很好想。但是矩阵可以轻松地表达。

$$
\begin{aligned}
\begin{bmatrix}k & t & 1 \end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
d & 0 & 1
\end{bmatrix}
&=
\begin{bmatrix}k+d & t & 1 \end{bmatrix}\\
\begin{bmatrix}k & t & 1 \end{bmatrix}
\begin{bmatrix}
1 & d & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}
&=
\begin{bmatrix}k & t+d \times k & 1 \end{bmatrix}
\end{aligned}
$$