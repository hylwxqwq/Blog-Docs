## 前缀函数

对于一个字符串 $s$，定义其前缀函数为它对应前缀的 border 长度：

$$
\pi[i] = \max\{j\}, (s[1\dots j] = s[i - j + 1 \dots i], j < i)
$$

若不存在这样的 $j$，则 $\pi[i] = 0$。

可以理解成以 $i$ 结尾的 $s$ 的**非前缀**子串和 $s$ 的前缀能够匹配的最大长度。

这个东西在很多串串题里面都会有用，所以接下来提一下这个东西的计算。

最暴力的做法就是，考虑对于 $i$，枚举所有以 $i$ 结尾的非前缀子串，并将其和对应前缀比较。

直接做是 $O(n^3)$ 的，如果用 Hash 优化一下就是 $O(n^2)$ 的。

但是存在一种更加优秀的方法可以 $O(n + m)$ 计算它，就是下面要说的名为 Knuth-Morris-Pratt 的算法。

## KMP

首先注意到一个点是，每次 $\pi[i]$ 的变化一定是在 $[0,1]$ 之间的，最多加一不然不变。

这个性质比较显然，但是很重要。

然后我们定义 $\pi[i]$ 的”候选项“为满足 $s[1\dots j] = s[i - j + 1 \dots i], j < i$ 的所有 $j$。

观察可以发现另外一个 Theorem：

> 如果 $j$ 是 $\pi[i]$ 的一个候选项，那么 $\pi[j]$ 一定是 $\pi[i]$ 候选项且 $\forall k \in (\pi[j], j)$，$k$ 都不是 $\pi[i]$ 的候选项。
>
> 即是，$\pi[i]$ 的候选项一定是 $\pi[j], \pi[\pi[j]], \dots$。

前一段比较显然，后一段可以用反证法证明，只要说明这个一定和 $\pi$ 定义中的 $\max$ 相矛盾即可。

然后我们可以根据第一个 Observation 看出另外一个 Theorem：

> 如果 $\pi[i]$ 的一个候选项是 $j$，那么 $\pi[i - 1]$ 的一个候选项是 $j - 1$。
>
> （两个字符串 $s[i - j + 1\dots i]$ 和 $s[1 \dots j]$ 相等的前提是 $s[i - j + 1, \dots i - 1] = s[1\dots j - 1]$）

于是结合这两个引理，我们不难想到一个算法：

初始化 $\pi[1] = 0$，从 $i = 2$ 开始计算，初始令 $j = 0$，

对于每个 $i$ ，假设此时 $\pi[1\sim i - 1]$ 都已经求出来，然后每次我们都让 $\pi[i]$ 的候选项是 $\pi[i - 1] + 1, \pi[\pi[i - 1]] + 1 \dots$ 然后求最大值即可。

当然求最大值的时候需要判断时候合法，所以实现的时候我们一般倒着来，从 $j = \pi[i - 1]$ 开始，不断跳 $\pi$ 直到合法，然后记录此时的 $j + 1$ 作为 $\pi[i]$，如果最后 $j = 0$ 了，证明不存在候选项，$\pi[i] = 0$。

因为 $j$ 初始值就是上一层的前缀函数 $\pi[i - 1]$，所以每次我们实际上是在**尝试**让 $j + 1$ 成为新的候选项，如果当前的候选项不合法，之后跳 $\pi$ 找下一个候选项的步骤就叫做“失配”。

实现：

```cpp
Next[1] = 0; // 记录 pi 的同时作为失配指针，Next 的意义可以理解为“下一个候选项”。
for(int i = 2, j = 0; i <= n; ++i) {
    while(j > 0 && s[i] != s[j + 1]) j = Next[j];
    if(s[i] == s[j + 1]) j ++;
    Next[i] = j;
}
```

显然每次 $j$ 最多增加 $1$，而跳 $\pi$ 的次数显然不会超过当前层开始的时候 $j$ 的值和 `while` 结束以后的 $j$ 的差值。

所以复杂度 $O(n)$。

### 单模式匹配

可以考虑再设一个 $f[i]$，表示文本串的以 $i$ 结尾的子串（注意不是非前缀了）和模式串的前缀能匹配的最大长度。

如果 $f[i]$ 和模式串的长度相等，那么证明模式串在文本串的 $[i - n + 1, i]$ 这个位置出现了。

求法和 Next 类似，不过每次匹配的时候我们是让模式串的前缀和文本串的一个以 $i$ 结尾的子串进行匹配，如果无法匹配就在模式串上跳 $\pi$，直到找到一个合法的模式串前缀可以和文本串的子串匹配。

所以这个过程就是，在 $t$ （文本串）以 $i - 1$ 结尾的子串和 $s[1, j]$ （模式串）已经匹配上了的时候，不断尝试扩展可以和文本串的以 $i$ 结尾的子串匹配的模式串的前缀长度，如果能扩展到整个模式串那就证明匹配成功了。

当然模式串的长度不一定要小于文本串，如果大于了显然是无法匹配成功的，但是在其他题里面就可以有别的应用，比如求模式串最长的出现在文本串里的前缀长度是多少之类的。

实现：

```cpp
Next[1] = 0;
for(int i = 2, j = 0; i <= n; ++i) {
    while(j > 0 && s[i] != s[j + 1]) j = Next[j];
    if(s[i] == s[j + 1]) j ++;
    Next[i] = j;
}
for(int i = 1, j = 0; i <= m; ++i) {
    while(j > 0 && (j == n || s[i] != s[j + 1])) j = Next[j];
    if(t[i] == s[j + 1]) ++j;
    f[i] = j;
    if(f[i] == n) orc[++cnt] = i - n + 1;
}
```

复杂度 $O(n + m)$。

注意求 $f$ 的时候是从 $i = 1$ 开始，并且如果 $j = n$ （$j + 1$ 这个位置不存在字符了），那么 $j$ 同样需要跳 $\pi$。

### CF1200E Compress Words

> 给你 $n$ 个字符串，答案串初始为空。第 $i$ 步将第 $i$ 个字符串加到答案串的后面，
> 但是尽量地去掉重复部分（即去掉一个最长的、是原答案串的后缀、也是第 $i$ 个串的前缀的字符串），求最后得到的字符串。
> $n \le 1e5, \sum len \le 1e6$。

这里就是和 KMP 中 $f$ 的定义类似的。

设答案串当前长度为 $N$，模式串的长度为 $M$，我们只需要求出答案串的后缀（即以 $N$ 结尾的子串）和新加入的串的前缀可以匹配的最大长度然后合并即可。

注意到每次合并的时候如果直接算整个串很亏，所以考虑前后各取 $\min(N,M)$ 长度的子串拿来匹配即可。

??? note "Code"
	```cpp
	// author : black_trees

	#include <cstdio>
	#include <vector>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	using i64 = long long;

	const int si = 1e5 + 10;
	const int si_len = 1e6 + 10;

	int n;
	string ans;

	int main() {	

		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);

		cin >> n >> ans;
		for(int i = 2; i <= n; ++i) {
			string nw; cin >> nw;
			int N = int(ans.size()), M = int(nw.size());
			int len = min(N, M);
			string s = nw.substr(0, len), t = ans.substr(N - len, len);
			s = ' ' + s, t = ' ' + t;
			vector<int> Next(len + 1), f(len + 1);
			Next[1] = 0;
			for(int j = 2, k = 0; j <= len; ++j) {
				while(k > 0 && s[j] != s[k + 1]) k = Next[k];
				if(s[j] == s[k + 1]) ++k;
				Next[j] = k;
			}
			for(int j = 1, k = 0; j <= len; ++j) {
				while(k > 0 && t[j] != s[k + 1]) k = Next[k];
				if(t[j] == s[k + 1]) ++k;
				f[j] = k;
			}
			int merge_len = f[len];
			nw.erase(0, merge_len);
			ans += nw;
		}	
		cout << ans << endl;

		return 0;
	}
	```

### POJ2406 Power Strings

> 求字符串的最小循环节，多测。
> $n \le 10^6$。

一个性质：如果 $(n - \pi[n]) | n$，则字符串存在循环节，最小循环节长度为 $\dfrac{n}{n - \pi[n]}$。

然后写起来就很简单了，不过 sb POJ 不支持高标准语法，CE 了好多次。。

??? note "Code"
	```cpp
	// author : black_trees

	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>

	#define endl '\n'

	using namespace std;
	// using i64 = long long;

	const int si = 1e6 + 10;

	char s[si];
	int n, Next[si];

	int main() {

		while(true) {
			scanf("%s", s);
			if(s[0] == '.') break;
			n = strlen(s);
			Next[1] = 0;
			for(int i = 2, j = 0; i <= n; ++i) {
				while(j > 0 && s[i - 1] != s[j]) j = Next[j];
				if(s[i - 1] == s[j]) j ++;
				Next[i] = j;
			}
			int len = 1;
			if(n % (n - Next[n]) == 0) len = n / (n - Next[n]);
			printf("%d\n", len);
		}

		return 0;
	}
	``` 