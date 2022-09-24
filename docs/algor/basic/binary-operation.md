## 基础运算符

`and,or,xor` 的显著特点就是**二进制下不进位**。

下面说的“按位”，指的是把参加运算的两个二进制数变成同位数的，然后每一位对应进行操作。

+ 按位与 `&` ，`and`

特点：二元运算符，二进制下按位进行运算，全 $1$ 则 $1$ ，反之为 $0$。

性质：$a \& b \le \min(a,b)$（手推即可）。

+ 按位或 `|` ，`or`

特点：二元运算符，二进制下按位进行运算，有 $1$ 则 $1$ ，全 $0$ 则 $0$。

性质：$a | b \ge \max(a,b)$

有个怪想法：既然不管是啥 | 1 都是 1，那是不是可以很快的把某个数变成 $2^n-1$ 的形式？

+ 按位异或 `^` ，`xor`

特点：二元运算符，二进制下按位进行运算，相反则 $1$ ，反之则 $0$。

相当于二进制下的不进位加法。

$x \operatorname{xor} y = k \ \Rightarrow x \operatorname{xor} k =y \ \Rightarrow y \operatorname{xor} k=x$ 

可以用于成对变换（图论当中常用）。

令 $n$ 为偶数，那么 $n \operatorname{xor} 1 = n+1$。

令 $n$ 为奇数，那么 $n \operatorname{xor} 1 =n-1$。

+ 按位取反 `~` ，`not`

特点：一元运算符，二进制下的每一位都取反（包括符号位）。

+ 左移 `<<`

特点：一元运算符，二进制下把每一位向左平移，高位越界则舍弃，低位用 $0$ 补足。

公式$1 << n =2^n,x<<n=x\times 2^n$公式

+ 算术右移 `>>`

特点：一元运算符，二进制下把每一位向右平移，低位越界则舍弃，高位用**符号位**补足。

$n>>1=\lfloor \frac{n}{2}\rfloor$ 。 GNU 的 GCC和G++都使用算术右移，逻辑右移高位用 $0$ 补足。

## 一些技巧

### lowbit

$\operatorname{lowbit}(n)$ 表示二进制下 $n$ 的最低的一个 $1$ 和它后面的 $0$ 组成的数。

公式：$\operatorname{lowbit}(n)=n \& -n$。

### popcnt

$\operatorname{popcnt}(n)$ 表示二进制下 $n$ 有多少位是 $1$ 。

可以直接用 

```cpp
int __builtin_popcount(unsigned int x)
int __builtin_popcountll(unsigned long long x)
```

毕竟现在的 NOI Linux 2.0 支持 C++14 和 G++ 9.8.3。

不过也可以手写

```cpp
int res=0;
while(n){
    ++res;
    n-=lowbit(n);
}
```

### 状压

把状态表示为一个二进制数之后进行操作（当然可以用 bitset，但是肯定没有直接拿一个数来做好）。

假设最低位为第一位。

```cpp
   (highest)      (lowest)
	  |				  |				
	  V				  V	
bit : 9 8 7 6 5 4 3 2 1                
num : 0 1 1 1 0 0 0 1 1
```

那么可以有以下操作：

+ 检测第 $k$ 位是 $0/1$ ：`(n&(1<<(k-1)))!=0` -> 1
+ 给第 $k$ 位取反：`n^=(1<<(k-1))`
+ 将第 $k$ 位赋值为 $1$：`n|=(1<<(k-1))`
+ 将第 $k$ 位赋值为 $0$：`n&=(~(1<<(k-1)))`

因为 C++ 默认是 $0$ 开头的，所以要 $-1$。

不过终态仍旧是 $1<<n$ （和最低位为 $0$ 的不减一写法是一样的）

说白了，如果以 $0$ 做最低位，不 $-1$ 。以 $1$ 做最低位，要 $-1$ ，但是；这两种写法写的时候，对应的位置实际上是一样的。

```cpp
// C++ Version
bool isPowerOfTwo(int n) { return n > 0 && (n & (n - 1)) == 0; }
// C++ Version
int modPowerOfTwo(int x, int mod) { return x & (mod - 1); }
// C++ Version
int Abs(int n) {
  return (n ^ (n >> 31)) - (n >> 31);
  /* n>>31 取得 n 的符号，若 n 为正数，n>>31 等于 0，若 n 为负数，n>>31 等于 -1
     若 n 为正数 n^0=n, 数不变，若 n 为负数有 n^(-1)
     需要计算 n 和 -1 的补码，然后进行异或运算，
     结果 n 变号并且为 n 的绝对值减 1，再减去 -1 就是绝对值 */
}
// C++ Version
// 如果 a>=b,(a-b)>>31 为 0，否则为 -1
int max(int a, int b) { return b & ((a - b) >> 31) | a & (~(a - b) >> 31); }
int min(int a, int b) { return a & ((a - b) >> 31) | b & (~(a - b) >> 31); }
```

### 快速幂

> 给定 $a,b \le 10^{9}$ ，$p$ 为一个大质数，求 $a^b \operatorname{mod} p$。

考虑二进制拆分思想，把 $b$ 在二进制下（一共 $k$ 位）的每一个 $1$ 单独拆开：

$b= \sum\limits_{i=1}^k c_i \times 2^{i-1}$

把式子展开之后放到指数位置：

$a^b=\prod\limits_{i=1}^{k} a^{c_k\times2^{k-1}}$

然后发现 $a^{2^i}=(a^{2^{i-1}})^2$ ，所以这个式子可以递推。

因为 $k \le \log(b)$ ，所以复杂度是 $\text{O}(\log(b))$。

```cpp
inline int qpow(int a,int b,int p){
    int ans=1%p; // attention
    for(;b;b>>=1){
        if(b&1) ans=(long long)ans*a%p; // b 的当前位是 1 ，累乘答案。
        a=(long long)a*a%p; // 递推。
    } return ans;
}
```

### 龟速乘

> 如果 $10^{18}$？

发现快速幂的时候乘法容易爆，所以考虑写一个类似“64位整数乘法”的东西。

仍然考虑二进制拆分，也可以直接递推，不过上面是乘，这里是累加。

```cpp
#define int long long
inline int turtle_mul(int a,int b,int p){
    int ans=0;
    for(;b;b>>=1){
        if(b&1) ans=(ans+a)%p;
        a=a*2%p;
    } return ans;
}
#undef int 
```

### 快速乘

发现龟速乘的时候时间复杂度是 $\log$ 的。

所以我们想一想又没有更优秀的做法。

骆可强神仙在2009年的集训队论文里提到了一个有意思的做法。

因为 $a\times b \operatorname{mod} p=a\times b -\lfloor \frac{a\times b}{p} \rfloor \times p$ 。

考虑用 `long double ` 存 $\frac{a\times b}{p}$ ，那么直接看整数部分即可。

然后发现上面的减法当中，前者和后者的差不会大于 $p$ ，所以分别用 `long long` 存，只需要关心较低位就行。

整数运算溢出就相当于取模，所以完全可以这么干（不过在 $p$ 大了之后会有精度问题）。

```cpp
#define int long long
inline int qmul(int a,int b,int p){
    a%=p,b%=p;
    int c=(a*b)/p;
    int ans=a*b-c*p;
    if(ans<0) ans+=p;
    else if(ans>=p) ans-=p;
    return ans;
}
#undef int 
```

