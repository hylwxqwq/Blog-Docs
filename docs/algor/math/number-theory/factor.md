
## 定义

若 $n$ 除以 $d$ 的余数为 $0$，则称 $d$ 能整除 $n$，或者 $d$ 为 $n$ 的约数，记作 $d|n,(n,d \in \mathbb{Z})$

+ 唯一分解定理的三个推论：
  1. 若 $n \in \mathbb{N}^*$，则 $n$ 的正约数集合为 $\{x | x = p_1^{b_1}p_2^{b_2}\dots p_m^{b_m},b_i \le c_i\}$。
  2. $n$ 的正约数个数为 $\prod\limits_{i = 1}^m(c_i+ 1)$
  3. $n$ 的正约数之和为 $(1+p_1+p_1^2+p_1^3+\dots +p_1^{c_1})\times\dots(1+p_m+p_m^2+p_m^3+\dots +p_m^{c_m}) = \prod\limits_{i = 1}^m(\sum\limits_{j = 1}^{c_i} p_i^{j})$

## 求正约数集合

### 试除法

考虑到 $n = a\times b$，其中 $a,b$ 为一对同时出现的因子。结合根号结论，扫描 $a\in[1,\sqrt{n}]$，若 $a | n$，则同时记录 $a,n/a$ 即可。

复杂度显然是根号，由此可以得到一个推论：整数 $n$ 的约数上界为 $2\sqrt{n}$ （**约数上界结论**）。

```cpp
int m, div[si];
void get_factors(int n) {
	m = 0;
	for(int i = 1; i * i <= n; ++i)
		if(n % i == 0) {
			div[++m] = i; 
			if(i * i != n) div[++m] = n / i;
		}
}
```

### 倍数法

试除法有局限性，只能求一个数的正约数集合，倍数法则可以求 $[1,n]$ 所有数的正约数集合。

基于一个比较显然的想法，对于任意一个 $d\in[1,n]$，以 $d$ 为约数之一的数的集合必然是 $\{kd,k=2,3,4\dots,\lfloor n/d\rfloor\}$

那么直接把 $d$ 扔到里面即可，复杂度 $O(n+n/2+n/3+\dots+1) = O(n\log n)$。

又可以导出一个结论：$1\sim n$ 所有数的约数个数之和约为 $n\log n$ 个（**约数个数和结论**）。

```cpp
std::vector<int> fact[si];
void get_factors(int n) {
	for(int i = 1; i <= n; ++i)
		for(int j = 1; j <= n / i; ++j)
			fact[i * j].emplace_back(i);
}
```

## 最大公约数

定义如字面意义。

+ 定理：$\forall a,b \in \mathbb{N},\gcd(a,b)\times \operatorname{lcm}(a,b)=a\times b$。

+ 更相减损术

  1. $\forall a,b \in \mathbb{N},a\ge b$ 有 $\gcd(a,b) = \gcd(b, a-b) = \gcd(a, a-b)$
  2. $\forall a,b \in \mathbb{N}$，有 $\gcd(2a,2b) = 2\gcd(a,b)$。

  第二者证明显然。

  $\text{Proof of 1:}$

  > 对于 $\forall d$，$d |	a, d|b$，可以有 $d|(a-b)$，因为，设 $a = r_1\times d,b= r_2\times d$。
  >
  > 显然 $r1-r2\ge 0$，所以 $a-b = (r_1-r_2)d$。
  >
  > 那么 $(a,b)$ 的公约数集合必然与 $(a,a-b)$ 相同，可以得到这两个集合的最大值相等，即是 $\gcd$ 相等。 

+ 欧几里得算法

  定理：$\forall a,b \in \mathbb{N}\Rightarrow\gcd(a,b)=\gcd(b,a\mod b)$

  $\text{Proof}:$

  > 考虑类比上面的证明去证明公约数集合相等。
  >
  > 不妨令 $a\ge b$，令 $a = qb+r$，显然 $d|a,d|qb \Rightarrow d|(a-qb)$。
  >
  > 那么设 $d|a,d|b$，则有 $d|qb\Rightarrow d|(a-qb) \Rightarrow d|r$。
  >
  > 因为 $d|r,d|b$，同上理即可。 

  当 $b=0$ 时，原式等于 $a$，可以写出以下代码求 $\gcd$（本质上是一种辗转相除）

  ```cpp
  int gcd(int a, int b) {
  	return b ? gcd(b, a % b) : a;
  }
  ```

  复杂度 $O(\log(a+b))$。

## 互质

就是 $\gcd(a,b)=1,a,b\in\mathbb{N}$，则称 $a,b$ 互质。

注意两两互质是 $\gcd(a,b)=\gcd(b,c)=\gcd(a,c) = 1$。

$\gcd(a,b,c)=1$ 为“$a,b,c$ 互质”。

## 欧拉函数

+ 定义：$[1,n]$ 中与 $n$ 互质的数的个数为 $\varphi(n)$，称作欧拉函数。

+ 计算：$\varphi(n) = n \times\prod\limits_{\text{PRIME(p)}\and p|n}(1-\frac{1}{p})$。

  本质上是容斥原理，即对于 $n$ 的任意两个质因子 $p,q$，从 $[1,n]$ 中去除 $p,q$ 分别的倍数之后，还要把 $pq$ 的倍数加回来一次，

  对于整体应用这个结论可以得到 $\varphi(n)=n-\lfloor\frac{n}{p}\rfloor-\lfloor\frac{n}{q}\rfloor+\lfloor\frac{n}{pq}\rfloor=n(1-\frac{1}{p})(1-\frac{1}{q})$。

  所以分解质因数就可以 $O(\sqrt{n})$ 求 $\varphi(n)$。

### 欧拉函数的性质

+ 性质1：$\forall n > 1, 1\sim n$ 中与 $n$ 互质的数的和为 $n\cdot\varphi(n)/2$。

  由更相减损术，和 $n$ 互质的数必然成对出现，且均值为 $n/2$，证毕。

+ 性质2：欧拉函数为积性函数，且有：$\gcd(a,b)=1\Rightarrow \varphi(ab)=\varphi(a)\cdot\varphi(b)$。

  展开计算式就行了。

+ 性质3：（积性函数的性质）：在唯一分解定理背景下，若 $f$ 为积性函数，则有：$f(n)=\prod\limits_{i=1}^mf(p_i^{c_i})$

  显然任意的 $p_i^{c_i}$ 和 $p_j^{c_j}$ 必然互质，由积性函数的性质，对整体应用结论，可以得到原式。

+ 性质4：若 $p$ 为质数，若 $p|n$ 且 $p^2\not|n$，则 $\varphi(n)=\varphi(n/p)\varphi(p)=\varphi(n/p)\cdot(p-1)$。

  积性函数的性质，显然，常用于递推。

+ 性质5：若 $p$ 为质数，若 $p|n$ 且 $p^2|n$，则 $\varphi(n)= \varphi(n/p)\times p$

  因为 $n/p$ 和 $p$ 不互质，所以只能展开计算式得到，常用于递推。

+ 性质6：$\sum_{d|n}\varphi(d)=n$。

  很有意思的性质，先对 $n$ 分解质因数，令 $f(x)=\sum_{d|x}\varphi(d)$。

  显然 $f(p_i^{c_i}) = \varphi(1)+\varphi(p_i)+\varphi(p_i^{2})+\dots+\varphi(p_i^{c_i}) = p_i^{c_i}$（由性质 $5$ 可以发现是一个等比数列求和）。

  然后发现若 $\gcd(n,m)=1$，$f(nm)=(\sum_{d|n}\varphi(d))\cdot(\sum_{d|m}\varphi(d))=f(n)f(m)$。

  所以 $f$ 是积性函数，由积性函数性质可以得到原式成立。

### 筛法求欧拉函数

+ 埃氏筛

  考虑埃氏筛的过程，发现任意的 $m\in[2,n]$，$m$ 会被所有的质数 $d$ 筛一次（从平方开始，所以筛它的必然是质数）。

  然后根据欧拉函数的计算式：$\varphi(k)=\varphi(k)\times(\frac{i-1}{i}),k = i,2i,3i,\dots$ 即可。

  复杂度 $O(n\log n)$。

  ```cpp
  int phi[si];
  void calc_euler_func(int n) {
  	for(int i = 1; i <= n; ++i) phi[i] = i;
  	for(int i = 2; i <= n; ++i)
  		if(phi[i] == i)
  			for(int j = i; j <= n; j += i)
  				phi[j] = phi[j] / i * (i - 1);
  }
  ```

+ 欧拉筛

  考虑到每一个数只会被他的 $mp$ 筛一次，所以直接利用性质4，5递推即可。

  复杂度 $O(n)$。

  ```cpp
  int phi[si];
  int m = 0, prime[si], vis[si];
  void calc_euler_func(int n) {
  	m = 0, phi[1] = 1;
  	memset(vis, 0, sizeof vis);
  	for(int i = 2; i <= n; ++i) {
  		if(vis[i] == 0) { 
  			vis[i] = i, prime[++m] = i, phi[i] = i - 1; 
  		}
  		for(int j = 1; j <= m; ++i) {
  			if(prime[j] > vis[i] || prime[j] * vis[i] > n) break;
  			vis[prime[j] * i] = prime[j];
  			if(i % prime[j] == 0)
  				phi[prime[j] * i] = phi[i] * prime[j]; 
  			else 
  				phi[prime[j] * i] = phi[i] * (prime[j] - 1);
  		}
  	}
  }
  ```

  