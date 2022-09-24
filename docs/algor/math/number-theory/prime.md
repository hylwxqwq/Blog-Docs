
## 质数的定义

简单来说就是除了 $1$ 和它本身以外没有任何因数。

本文记 $\text{PRIME}(n)$ 表示 $n$ 为质数。

有两个重要结论：

+ 结论 1：对于足够大的 $n \in \mathbb{N}^*$，$[1,n]$ 中的素数个数约为 $\frac{n}{\ln n}$ 个 （**对数结论**）
+ 结论 2：若 $\lnot \text{PRIME}(n)$，则 $\exists T \in [2,\sqrt{n}]$，使得 $T|n$。（**根号结论**）

此处记 $\sqrt{n} = \lfloor\sqrt{n}\rfloor$。

## 判断是否为质数

由根号结论可以得到一个比较显然的质数判断算法：

令 $x\in[2,\sqrt{n}]$，判断是否存在 $x | n$。若存在 $\Rightarrow \lnot \text{PRIME}(n)$。

复杂度 $O(\sqrt{n})$。

```cpp
bool is_prime(int n) {
	if(n < 2) return false;
	for(int i = 2; i * i <= n; ++i) 
		if(n % i == 0) return false;
	return true;
}
```

## 质数筛法

泛化：用来求 $[1,n]$ 中的素数集合。

### 埃氏筛

基于一个显然的结论：$\forall x \in \mathbb{N}^*$，$\lnot \text{PRIME}(kx),(k = 2, 3, 4,\dots)$

一个 simple 的想法：枚举 $2 \to n$，如果 $i$ 没有被标记为合数，则 $\text{PRIME}(i)$，然后标记所有的 $ki$。如果 $i$ 已经被标记，那么标记所有的 $ki$，初始的时候 $2$ 没有被标记。复杂度 $O(n\times\sum\frac{n}{i})=O(n^2)$。

发现这个做法会重复标记一个数很多次，比如 $12$ 就会被 $2, 3, 4, 6$ 都标记一次。

那么考虑一个简单的优化：显然对于 $\forall rx,r\in[2,x)$，$rx$ 必然会在被 $x$ 标记到的时候提前被标记到。因为 $r$ 肯定比 $x$ 小，那么 $r$ 肯定就标记过了 $rx$，所以我们可以直接从 $k = x$ 开始标记 $kx$。

复杂度 $O(\sum\limits_{r\le n \and \text{PRIME}(r)}\dfrac{n}{r}) = O(n \log \log n)$。

```cpp
bool vis[si];
int m, prime[si];
void get_primes(int n) {
	m = 0;
	memset(vis, false, sizeof vis);
	for(int i = 2; i <= n; ++i) {
		if(!vis[i]) prime[++m] = i;
		for(int j = i * i; j <= n; ++j) 
			vis[j] = true;
	}
}
```

### 线性筛

又叫欧拉筛。

考虑到埃氏筛还是会重复标记很多数。

还是举一个例子：$12 = 2 \times 6 = 3 \times 4$，这个例子下，$12$ 会被 $2,3$ 各筛一次。

问题所在就是，一个数并不能唯一确定一个数来标记它。

思索一下，可以使用一个数的最小质因子来标记它，因为质因子不会再拆分成别的因数，使用最小的质因子是因为确定起来方便。

记 $mp(n)$ 表示 $n$ 的最小质因子，显然，如果 $\text{PRIME}(i) \Rightarrow mp(i) = i$。

那么当扫到某一个数 $i$ 且 $i$ 没有被标记的时候，令 $mp(i) = i$ ，记录质数 $i$。然后对于 $i$ （不管它是不是质数）枚举所有比 $mp(i)$ 小或者等于 $mp(i)$ 的质数 $j$（从已经确定的质数集合里面选），标记 $i\times j$ 为合数，并令 $mp(i\times j) = j$。

换一种说法，就是**从大到小累积质因子**，这样**能唯一确定每个数的组成方式**，比如 $12$ 就是 $3 \times 2 \times 2$。

筛出 $12$ 的过程是；先扫描到 $2$，此时合法的 $j$ 只能是 $2$，所以标记 $4$ 为合数，然后令 $mp(4)=2$，扫描到 $3$，发现 $2,3$ 可以充当 $j$，所以标记 $6,9$，$mp(6) = 2,mp(9) = 3$。扫到 $4$ 的时候，发现只有 $2$ 可以充当 $j$，于是标记 $8$， $mp(8)=2$，当扫描到 $6$ 的时候，就标记了 $12$，并令 $mp(12) = 2$。

因为每个合数 $i \times j$ 只会被他的最小质因子 $j$ 筛一次，所以复杂度是 $O(n)$ 的。

这样还顺便求了每个数的最小质因子。

```cpp
int vis[si];
int m = 0, prime[si];

void get_primes(int n) {
	m = 0;
	memset(vis, 0, sizeof vis);
	for(int i = 2; i <= n; ++i) {
		if(vis[i] == 0) { 
			vis[i] = i;
			prime[++m] = i;
		}
		for(int j = 1; j <= m; ++j) {
			if(prime[j] > vis[i] || prime[j] * i > n) 
				break;
			vis[prime[j] * i] = prime[j];
		}
	}
}
```

## 质因数分解

泛化：分解一个数的所有质因子

+ 算术基本定理：任何一个大于 $1$ 的正整数都可以唯一分解为有限个质数的乘积。

  也叫唯一分解定理，可以写成 $N = p_1^{c1}\times p_2^{c2}\times p_3^{c3}\times \dots p_m^{cm}, c_i \in \mathbb{N}^*, p_i < p_{i + 1},\text{PRIME}(p_i)$。

+ 试除法：

  上面的定理基本没太大的用处，考虑一个埃氏筛的变式，结合根号结论。

  想法：扫一遍 $d\in[2,\sqrt{n}]$，若 $d|n$，不断的除掉 $n$ 中的 $d$，记录 $c$ 即可。

  因为从小到大，所以每次能整除 $n$ 的 $d$ 必然是质数，是合数的之前都除掉了。

  复杂度 $O(n)$。

```cpp
int c[si]; // exponential
int m = 0, p[si]; // prime factor

void divide(int n) {
	m = 0;
	for(int i = 2; i * i <= n; ++i) {
		if(n % i == 0) {
			p[++m] = i, c[m] = 0;
			while(n % i == 0) n /= i, c[m]++;
		}
	}
	if(n > 1) p[++m] = n, c[m] = 1;
}
```

