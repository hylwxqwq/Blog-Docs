
## 泛化

这类问题的特点都是，一般不具有区间的可加可减性，且较难合并答案。

这也导致线段树和树状数组等算法很难处理这些询问。

所以最早在 codeforces 上出现类似的问题时，高手们想到了使用分块的思想。

并且这类问题是离线不带修的，所以他们就想到了对询问进行分块。

## 应用

主要的三个思想是：

1. 将询问离线下来。
2. 将询问分块。
   然后按照左端点所在的块的编号为第一关键字，右端点的位置为第二关键字排序。

3. 排序后，当前询问答案将在上一个询问的基础上得到，方式是暴力移动左右端点，计算变化的贡献。

（实际上还是对序列进行分块，不过分块后询问的左端点所处的块的编号将影响之后的操作）

归纳一下，只要当前区间 $[l, r]$ 的答案可以以 $\text{O}(1)$ 的复杂度扩展到相邻的区间 $[l,r-1],[l,r+1],[l-1,r],[l+1,r]$，那么就可以使用分块的思想，优化暴力扩展的过程，减少移动的总距离以降低时间复杂度。

下面给出一份模板并做解释说明：

???+ note "Code"
	```cpp
	 
	int n, m, unit; // unit 是块长，一般取根号 n。
	int a[si];
	
	struct Query {
		int l, r, id;
		bool operator < (const Query &b) const {
			if((l / unit) != (b.l / unit)) 
				return l < b.l;
			// 这里和写 (l / unit) < (b.l / unit) 是等价的。
			return r < b.r; 
		}
	}ask[si];
	
	int ans, res[si]; // 实时的答案，每个询问的答案
	
	inline void add(int pos) {} // 加上 pos 这个位置之后，答案 ans 会如何变化
	inline void sub(int pos) {} // 删掉 pos 这个位置之后，答案 ans 会如何变化
	
	int main(){
	
		cin >> n >> m, unit = sqrt(n);
		for(int i = 1; i <= n; ++i)
			cin >> a[i];
	
		for(int i = 1; i <= m; ++i) 
			cin >> ask[i].l >> ask[i].r, ask[i].id = i;
		sort(ask + 1, ask + 1 + m);
	
		int l = 1, r = 0; // 维护答案是闭区间 [l, r] 时的写法。
		for(int i = 1; i <= m; ++i) {
			Query &q = ask[i];
			while(l > q.l) add(--l);
			while(r < q.r) add(++r);
			while(l < q.l) sub(l++);
			while(r > q.r) sub(r--);
			// 这里的移动顺序是有讲究的，不能乱搞。
	
			res[q.id] = ans;
		}
	
		for(int i = 1; i <= m; ++i) 
			cout << res[i] << endl;
		return 0;
	}
	```

!!! Warning
	1. 此处维护的是**闭区间**，所以初始值为 `l = 1, r = 0`。
	2. 我移动指针的写法是比较标准的一种，不能乱改，具体为什么下面会说。

!!! note "关于指针移动的顺序"

	OI-wiki 上提到了所有的可能移动方式，并列出了哪一些是正确的，哪一些是错误的。

	我使用的是 `--l, ++r, l++, r--`。

	这和其它正确做法的共同点是，全部都是先扩大区间，然后再将 $l,r$ 缩小到下一个询问的区间 $[l\prime,r\prime]$。

	这样可以防止 $l > r + 1$ 的情况出现，如果这种情况出现，那么就会有一个元素被加入的次数是负数，这是不合法的。

	!!! note "为什么有 ++ 和 -- 的区别"
		前两次扩展是扩大区间，会让扫到的元素被加入。

		而 $l, r$ 本来在的位置是已经被加入过的，所以要先直接移动指针之后再加入 $l - 1$ 和 $r + 1$，应当写 `--l, ++r`。

		后两次扩展是缩小区间，会让扫到的元素被踢出。

		而 $l, r$ 本来在的位置是已经被加入过的，所以需要先踢出 $l, r$ 再移动指针，应当写 `l++, r--`。

这样的时间复杂度是 $\text{O}(n\sqrt{n})$ 的（假设 $n,m$ 同阶），

具体证明我是不会的（所以这里摘抄了 OI-wiki 的证明。

??? note "Proof & Analysis"

	以下的情况在 $n$ 和 $m$ 同阶的前提下讨论。
	
	首先是分块这一步，这一步的时间复杂度是 $O(\sqrt{n}\cdot\sqrt{n}\log\sqrt{n}+n\log n)=O(n\log n)$;
	
	接着就到了莫队算法的精髓了，下面我们用通俗易懂的初中方法来证明它的时间复杂度是 $O(n\sqrt{n})$；
	
	证：令每一块中 $L$ 的最大值为 $\max_1,\max_2,\max_3, \cdots , \max_{\lceil\sqrt{n}\rceil}$。
	
	由第一次排序可知，$\max_1 \le \max_2 \le \cdots \le \max_{\lceil\sqrt{n}\rceil}$。
	
	显然，对于每一块暴力求出第一个询问的时间复杂度为 $O(n)$。
	
	考虑最坏的情况，在每一块中，$R$ 的最大值均为 $n$，每次修改操作均要将 $L$ 由 $\max_{i - 1}$ 修改至 $\max_i$ 	或由 $\max_i$ 修改至 $\max_{i - 1}$。
	
	考虑 $R$：因为 $R$ 在块中已经排好序，所以在同一块修改完它的时间复杂度为 $O(n)$。对于所有块就是 $O(n\sqrt{n})$。
	
	重点分析 $L$：因为每一次改变的时间复杂度都是 $O(\max_i-\max_{i-1})$ 的，所以在同一块中时间复杂度为 $O(\sqrt{n}\	cdot(\max_i-\max_{i-1}))$。
	
	将每一块 $L$ 的时间复杂度合在一起，可以得到：
	
	对于 $L$ 的总时间复杂度为
	
	$$
	\begin{aligned}
	& O(\sqrt{n}(\max{}_1-1)+\sqrt{n}(\max{}_2-\max{}_1)+\sqrt{n}(\max{}_3-\max{}_2)+\cdots+\sqrt{n}(\	max{}_{\lceil\sqrt{n}\rceil}-\max{}_{\lceil\sqrt{n}\rceil-1))} \\
	= & O(\sqrt{n}\cdot(\max{}_1-1+\max{}_2-\max{}_1+\max{}_3-\max{}_2+\cdots+\max{}_{\lceil\sqrt{n}\	rceil-1}-\max{}_{\lceil\sqrt{n}\rceil-2}+\max{}_{\lceil\sqrt{n}\rceil}-\max{}_{\lceil\sqrt{n}\rceil-	1)}) \\
	= & O(\sqrt{n}\cdot(\max{}_{\lceil\sqrt{n}\rceil-1}))\\
	\end{aligned}
	$$
	
	（裂项求和）
	
	由题可知 $\max_{\lceil\sqrt{n}\rceil}$ 最大为 $n$，所以 $L$ 的总时间复杂度最坏情况下为 $O(n\sqrt{n})$。
	
	综上所述，莫队算法的时间复杂度为 $O(n\sqrt{n})$；
	
	但是对于 $m$ 的其他取值，如 $m<n$，分块方式需要改变才能变的更优。
	
	怎么分块呢？
	
	我们设块长度为 $S$，那么对于任意多个在同一块内的询问，挪动的距离就是 $n$，一共 $\displaystyle \frac{n}{S}$ 	个块，移动的总次数就是 $\displaystyle \frac{n^2}{S}$，移动可能跨越块，所以还要加上一个 $mS$ 	的复杂度，总复杂度为 $\displaystyle O\left(\frac{n^2}{S}+mS\right)$	，我们要让这个值尽量小，那么就要将这两个项尽量相等，发现 $S$ 取 $\displaystyle \frac{n}{\sqrt{m}}$ 	是最优的，此时复杂度为 $\displaystyle O\left(\frac{n^2}{\displaystyle \frac{n}{\sqrt{m}}}+m\left(\frac{n	}{\sqrt{m}}\right)\right)=O(n\sqrt{m})$。
	
	事实上，如果块长度的设定不准确，则莫队的时间复杂度会受到很大影响。例如，如果 $m$ 与 $\sqrt n$ 同阶，并且块长误设为 	$\sqrt n$，则可以很容易构造出一组数据使其时间复杂度为 $O(n \sqrt n)$ 而不是正确的 $O(n)$。
	
	莫队算法看起来十分暴力，很大程度上是因为莫队算法的分块排序方法看起来很粗糙。我们会想到通过看上去更精细的排序方法对所有	区间排序。一种方法是把所有区间 $[l, r]$ 看成平面上的点 $(l, r)$，并对所有点建立曼哈顿最小生成树，每次沿着曼哈顿最小	生成树的边在询问之间转移答案。这样看起来可以改善莫队算法的时间复杂度，但是实际上对询问分块排序的方法的时间复杂度上界已	经是最优的了。
	
	假设 $n, m$ 同阶且 $n$ 是完全平方数。我们考虑形如 $[a \sqrt n, b \sqrt n](1 \le a, b \le \sqrt n)$ 	的区间，这样的区间一共有 $n$ 个。如果把所有的区间看成平面上的点，则两点之间的曼哈顿距离恰好为两区间的转移代价，并且任	意两个区间之间的最小曼哈顿距离为 $\sqrt n$，所以处理所有询问的时间复杂度最小为 $O(n \sqrt n)$	。其它情况的数据构造方法与之类似。
	
	莫队算法还有一个特点：当 $n$ 不变时，$m$ 越大，处理每次询问的平均转移代价就越小。一些其他的离线算法也具有同样的特点（如求 LCA 的 Tarjan 算法），但是莫队算法的平均转移代价随 $m$ 的变化最明显。

## 奇偶性优化

没有优化的莫队的指针移动可能是这样的（感性理解）：

```
----------------------->|
		   |<-----------|
		   |--o-------o----->
o 是我们当前所讨论的询问左右端点。
```

然后在第三次移过来的时候才会更新答案。

出现这种情况的原因是，当多个询问的右端点不在同一块时，$r$ 指针需要多次往返移动，多走了很多不必要的路程。

所以可以考虑，对于编号为奇数的块，块内按 $r$ 升序排序，编号为偶数的块，块内按 $r$ 降序排序。

画图就可以发现，这样去掉了很多不必要的路程

比如上面的这个图，本来在第三次扫过我们所讨论的询问时才更新的答案，到第二次扫过来就更新了。

```
----------------------->|
		   |<-o-------o-|
```

实测跑的飞快，可以优化 $30\% \sim 35\%$ 的速度。

??? note "实现"
	```cpp
		bool operator < (const Query &b) const {
			if((l / unit) != (b.l / unit)) 
				return l < b.l;
			if((l / unit) & 1)
				return r < b.r;
			return r > b.r;
		} // 奇偶性优化的写法
	```

## 例题

用两道板子题看一看莫队的具体实现和一些细节。

### 20220429 C 组模拟赛 T3

???+ note "题目描述"
	给你一个序列 $a$，长度为 $n$，$q$ 次询问。

	每次询问一个区间 $[l,r]$ ，求这个区间里出现次数为奇数的数的个数。

	$n,q\le 10^5, |a_i| \le 10^9$。
??? note "题解"
	莫队板子，甚至比小 Z 的袜子还板子。

	考虑直接莫队，问题在于如何记录贡献/答案的变化。

	先离散化 $a_i$ 方便统计。

	开一个变量 $ans$，记录实时的答案，另外开一个数组 $cnt[i]$ 记录每一个数的出现次数。

	如果变动 $cnt[i]$ 之后，$cnt[i]$ 变为奇数，$ans+1$，反之 $ans-1$ 即可。

??? note "Code"
	```cpp
	
	#include <cmath>
	#include <bitset>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const int si = 1e5 + 10;
	
	int n, Q, unit;
	int a[si];
	struct Query {
		int l, r, id;
		bool operator < (const Query &b) const {
			if((l / unit) != (b.l / unit)) 
				return l < b.l;
			if((l / unit) & 1) 
				return r < b.r;
			return r > b.r; 
		} 
	}ask[si];
	
	int cnt[si];
	int res[si], ans = 0;
	
	inline void add(int pos) { 
		cnt[a[pos]] ++;
		if(cnt[a[pos]] & 1) ans ++;
		else ans --;
	}
	inline void sub(int pos) { 
		cnt[a[pos]] --;
		if(cnt[a[pos]] & 1) ans ++;
		else ans --;
	}
	
	int main() {	
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		std::vector<int> v; v.clear();
		cin >> n, unit = sqrt(n);
		for(int i = 1; i <= n; ++i)
			cin >> a[i], v.emplace_back(a[i]);
		sort(v.begin(), v.end()), v.erase(unique(v.begin(), v.end()), v.end());
		for(int i = 1; i <= n; ++i)
			a[i] = lower_bound(v.begin(), v.end(), a[i]) - v.begin();
	
		cin >> Q;
		for(int i = 1; i <= Q; ++i) 
			cin >> ask[i].l >> ask[i].r,
			ask[i].id = i;
		sort(ask + 1, ask + 1 + Q);
	
		int l = 1, r = 0;
		for(int i = 1; i <= Q; ++i) {
			Query &q = ask[i];
			while(l > q.l) add(--l);
			while(r < q.r) add(++r);
			while(l < q.l) sub(l++);
			while(r > q.r) sub(r--);
	
			res[q.id] = ans;
		}
	
		for(int i = 1; i <= Q; ++i)
			cout << res[i] << endl;
		return 0;
	}
	```

### [国家集训队] 小 Z 的袜子

???+ note "题目描述"

	作为一个生活散漫的人，小 Z 每天早上都要耗费很久从一堆五颜六色的袜子中找出一双来穿。终于有一天，小 Z 再也无法忍受这恼人的找袜子过程，于是他决定听天由命……

	具体来说，小 Z 把这 $N$ 只袜子从 $1$ 到 $N$ 编号，然后从编号 $L$ 到 $R$ 的所有袜子中抽出两只。

	尽管小 Z 并不在意两只袜子是不是完整的一双，甚至不在意两只袜子是否一左一右，他却很在意袜子的颜色，毕竟穿两只不同色的袜子会很尴尬。

	你的任务便是告诉小 Z，他有多大的概率抽到两只颜色相同的袜子。当然，小 Z 希望这个概率尽量高，所以他可能会询问多个 $[L,R]$ 以方便自己选择。

	然而数据中有 $L=R$ 的情况，请特判这种情况，输出 **0/1**。

??? note "题解"
	
	可以算的上莫队的起源题目。

	考虑一个区间 $[l,r]$ 的答案应当是什么： 方案数一共有 $\text{C}^{2}_{r - l + 1} = \dfrac{(r-l+1)\times(r-l)}{2}$ 种。

	而假设这个区间里颜色 $c$ 的袜子有 $num[c]$ 种，每种颜色的答案就是 $\text{C}^{2}_{num[c]} = \dfrac{num[c]\times(num[c]-1)}{2}$ 种。

	所以能抽到同色的总方案数是 $\sum_c \dfrac{num[c]\times(num[c]-1)}{2}$ 种。

	那么一个区间的答案就是 $\dfrac{\sum_c \dfrac{num[c]\times(num[c]-1)}{2}}{\dfrac{(r-l+1)\times(r-l)}{2}}$。

	题目要求约分，所以对于每个询问记录它的分子和分母，最后约分即可，题目要求的 $L = R$ 的情况也需要特判，原因显然。

	那么发现这个东西是可以 $\text{O}(1)$ 转移到相邻区间的，所以直接莫队即可。

??? note "Code"

	```cpp
	
	#include <cmath>
	#include <cstdio>
	#include <cstring>
	#include <iostream>
	#include <algorithm>
	
	#define meow(x) cerr << #x << " = " << x
	
	using namespace std;
	using i64 = long long;
	
	const i64 si = 5e4 + 10;
	
	i64 n, m, unit;
	i64 c[si], cnt[si];
	struct Query {
		i64 l, r, id;
		bool operator < (const Query &b) const {
			if((l / unit) != (b.l / unit)) 
				return l < b.l;
			if((l / unit) & 1)
				return r < b.r;
			return r > b.r;
		}
	}ask[si];
	
	i64 sum = 0;
	i64 nume[si], deno[si];
	i64 gcd(i64 a, i64 b) {
		return b ? gcd(b, a % b) : a;
	}
	
	void add(i64 pos) {
		i64 now = c[pos];
		sum -= (cnt[now] * (cnt[now] - 1)) / 2;
		cnt[now] ++;
		sum += (cnt[now] * (cnt[now] - 1)) / 2;
	}
	void sub(i64 pos) {
		i64 now = c[pos];
		sum -= (cnt[now] * (cnt[now] - 1)) / 2;
		cnt[now] --;
		sum += (cnt[now] * (cnt[now] - 1)) / 2;
	}
	// 更新答案把原来的减去，然后加上新的即可。
	
	int main() {	
	
		// freopen("1.in", "r", stdin);
		// freopen("1.ans", "w", stdout);
	
		cin.tie(0) -> sync_with_stdio(false);
		cin.exceptions(cin.failbit | cin.badbit);
	
		memset(cnt, 0, sizeof cnt);
	
		cin >> n >> m, unit = sqrt(n);
		for(int i = 1; i <= n; ++i)
			cin >> c[i];
	
		for(int i = 1; i <= m; ++i) 
			cin >> ask[i].l >> ask[i].r, ask[i].id = i;
		sort(ask + 1, ask + 1 + m);
	
		i64 l = 1, r = 0;
		for(int i = 1; i <= m; ++i) {
			Query &q = ask[i];
	
			if(q.l == q.r) {
				nume[q.id] = 0, deno[q.id] = 1;
				continue;
			}
	
			while(l > q.l) add(--l);
			while(r < q.r) add(++r);
			while(l < q.l) sub(l++);
			while(r > q.r) sub(r--);
	
			nume[q.id] = sum, deno[q.id] = (r - l + 1) * (r - l) / 2;
			if(sum == 0) deno[q.id] = 1ll;
		}
	
		for(int i = 1; i <= m; ++i) {
			if(nume[i] != 0) {
				i64 com = gcd(nume[i], deno[i]);
				nume[i] /= com, deno[i] /= com;
			}
			cout << nume[i] << "/" << deno[i] << endl;
		}
	
		return 0;
	}
	```