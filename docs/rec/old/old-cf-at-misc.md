## CF & AT 杂题

因为没啥时间，也不太想写题解，所以一些杂题之类的简单题解就放在这里了。

（是时候改改强迫症了）

一般只会有CF和AT的，整场比赛的题解会放在 Codeforces Solution 里面。

其实大部分都是口胡的，没时间写代码，以后再补（

### CF 1473E

定义一个路径的贡献为下面的那个式子，问 $1 \to \forall i$ 的路径的贡献的最小值

发现柿子：$\sum\limits_{i=1}^kw_{e_i} -\max\limits_{i=1}^kw_{e_i} + \min\limits_{i=1}^kw_{e_i}$

实际上就是在维护所有边权去掉最大加上一个最小。

也就是少一个边，另外一个边算两次。

那么就是说，对于每一个决策都会有去掉不加上，不去掉不加上，加上不去掉，加上然后去掉四种情况，

所以把每个点拆成 $4$ 个状态，问题就可以转化为在分层图上求最短路，因为你多加的是 $\min$ 而且求的也是 $\min$ ，所以这个做法是对的。

### CF 516E （By $\text{d}\color{red}{\text{reamoon}}$）

*3100 的大毒瘤数论。

不太会，之后再重新问问出题人咋做的（

### ABC 209E

博弈论+图论

必须要和上一个单词重合三个以上的字母的单词接龙，给定词典，且大小写敏感。

问输赢或平。

你发现我们不会关心除了前三个或者后三个字符以外的东西。

而且要接龙就必须完全重合，所以我们考虑先字符串hash一下前三位和后三位，然后把前三位向后三位连边。

然后我们会得到一张有向图。

考虑怎么维护先手必胜态和先手必败态。

如果当前状态能去往的状态都是必胜态或者无法去往任意状态，则它是先手必败态。

如果它能到达至少一个必败态，那么他是必胜态。

其他的都是平局。

那么建反图拓扑更新即可，更新不到的都是平局。

另外，只要一个点确定了必胜或者必败态，那么它也可以入队，因为不会再被更改了。

### ABC 191D

给定一个圆，问圆上和圆内整点个数。

是某个HAOI的超级加强版（

这题卡精度特别吓人，YL当时卡 $1e-14$ 过了。

我卡 $1e-10$ 爆了，本来是忘了这个题的，结果DM的毒瘤卡精度课讲了这个。

就回来写一下。

发现圆心坐标和半径都是浮点数，不过不超过 $10^5$ 而且最多有四位小数点。

所以我们考虑先给每个点的 $x,y$ 都乘上 $10^4$ 变成整数。

然后考虑勾股定理+枚举判断合法上下界就可以了。

### CF 261D

没做，空着。

### CF 1526C2

问一个序列的最长子序列，使子序列的任意位置的前缀和不小于 $0$。

大傻逼题，秒了。

考虑用一个优先队列维护，一个一个吃，如果吃死了就把最小的吐出来然后继续吃。

最后输出 size 走人。

### CF 1006F

dm 讲的一道双向BFS。

懒得写所以转了别人的sol

题意是有一个 $n\times m$ 的地图，然后从左上角走到右下角，问最后异或的值等于 $k$ 的路径有多少条。

 思路就是折半搜索，可以降低很多时间复杂度，因为当地图小的时候，$n+m$ 就不满足折半的条件了，

所以这里用 $n+m+2$ 作为条件，首先我们先搜索步数为 $n+m+2$ 的一半的值为搜索终点，记录当前的异或值，然后再从终点开始搜索，

当搜索到 $n+m+2$ 的一半的时候，我们判断当前点的与 $k$ 的异或值是否有标记过，有的话就说明这整个一条路的异或值等于 $k$ ，直接加给 $ans$ 就好了。

原文链接：https://blog.csdn.net/Charles_Zaqdt/article/details/81077919

### ARC 084 D/B

dm 讲的一道数论BFS。

找出数位和最小的 $k$ 的倍数是多少。

2e5

发现可能会炸 `__int128` .

所以我们考虑对数位进行操作。

发现如果从 $1$ 开始，只用两种操作就可以得到任意一个数（其实真实情况一种就可以了）

1. $+1$ 且 cost 为 $1$
2. $\times 10$ 且 cost 为 $0$。

所以其实这是个 01BFS。

令每一个状态都对 $k$ 取模，然后从 $1$ 开始。

如果当前这个操作是 cost 为 $1$ 的操作，扔到队尾，反之扔到队头。

然后01BFS即可。

### ABC 203D

一道 01 二分的板子？

这个 trick 不是很常见，但是只要一出就能卡到人。

这道题要求你求出一个 $n\times n$ 的矩阵当中的所有 $k\times k$ 大小的子矩阵的贡献的最小值。

一个矩阵的贡献定义为把矩阵所有元素从小到大排序之后的第 $\lfloor \dfrac{k^2}{2} \rfloor +1$ 大的数（可以理解为**中位数**）

大小是 $800^2$。

问题可以通过二分转化为判定：存不存在一个子矩阵，使得这个矩阵的贡献比 $x$ 小？

然后我们让 $x$ 尽量小，当这个矩阵的贡献就是 $x$ 的时候，答案就是 $x$。

你考虑去二分这个数字 $x$，然后对于你二分到的矩阵，令：

$$\text{submatrix}[i][j]=\begin{cases}0 & ,\text{submatrix}[i][j]\le x\\1 & ,\text{submatrix}[i][j] > x\end{cases}$$

如果说这个子矩阵的和是 $<\lfloor \dfrac{k^2}{2} \rfloor +1$  的，那么很明显比 $x$ 大的数就一定少于  $\lfloor \dfrac{k^2}{2} \rfloor +1$  个。

那么它的贡献肯定比 $x$ 小，满足要求，反之则不满足。

然后写个二维前缀和搞搞就行了。

### CF 1059D

给一堆点，是否有一个和 $x$ 轴相切的圆使得每一个点都被它包含。

问这个圆的半径最小是多少。

首先你发现如果存在一个点对 $((x_1,y_1),(x_2,y_2))$ 满足 $y_1\times y_2 <0$ ，那么就很明显无解。

然后我们把所有点转到 $x$ 轴上方。

对于每一个你二分到的 $r$ ，我们可以扫描一遍点集合，对于每一个点  $(x,y)$ 利用勾股定理求出一个区间，使得区间里的每一个点到 $(x,y)$ 的距离在区间 $[y,r]$ 当中。

然后对于所有的区间求一个交集，如果交集为空集那么这个 $r$ 不合法，反之合法。

然后我们使用实数二分，但是因为这题卡精度，所以我们不使用 $eps$ 来判断，而是直接按次数循环。

这题精度要求 $10^{-6}$ ，所以你大概循环个 $10^3$ 次就差不多了。

### AGC 006D

也是一道非常妙的 01 二分。

定义一个数字金字塔，每一个位置是它下一层的三个节点的**中位数**。

给定最后一层，求第一层最有，$2e5$ 层。

发现要 $\text{O}(n \log n)$ 才行。

然后你考虑二分顶层节点 $x$ 的数值。

对于最后一层的节点，我们令所有的 $\le x$ 的节点变成 $0$ ，反之为 $1$ （和 ABC203D 一个思路）。

然后你考虑向上递推的过程。

你发现，因为 $x$ 是最顶层节点，所以如果底层有连续的两个 $1$ 或者连续的两个 $0$ ，他们就会一直向上走，而且不断往中间靠。

所以最靠近中间的那一组就是塔顶的 $0$ 或者 $1$ 。

如果你推出来塔顶是 $0$ ,那么就满足情况，反之不满足。

然后你再特判一下根本没有连续的数字的情况即可。

### CF 865C

一道期望+二分。

也比较妙。

帮 PWK 修了他的 [题解](https://weizheng.blog.luogu.org/solution-cf865c)，所以题意就用他的了：

你是一个 up 主，你要录一段速通 RA2YR 的视频，

战役有 $n$ 关，每关你有 $p_i\%$ 的可能性花 $F_i$ 的时间通过，$(1-p_i)\%$ 的可能性花掉 $S_i$ 的时间通过（失误了），其中保证 $F_i>S_i$。

但因为你想“速通”，你不希望你的视频长度超过 $m$。

当你认为自己接下来可能录不完（接着录录完不如重新录）时，你会重新开始录，求录完整个游戏的时间期望。

考虑设 $f_{i,j}$ 表示打过第 $i$ 关，使用 $j$ 的时间的期望。

然后因为是有时间限制的，所以你会推出一个这样的柿子（也是PWK的）：

$$ f(i,j)=\begin{cases}
0 ,& i=n,j\le m\\
f(0,0),& j >m \\ 
\min(f(0,0),e1+e2) ,& \text{otherwise.}
\end{cases} $$

其中 $e_1=p_{i+1} \times f(i+1,j+f_{i+1})+f_{i+1}))   $。

$e_2=p_{i+1}\times(1-p_{i+1})\times(f(i+1,j+S_{i+1})+S_{i+1})$ 。

然后你发现 $f_{0,0}$ 会导致转移有环，所以把 $f_{0,0}$ 当作一个常数，从 $n$ 开始倒序枚举。

然后最优重开时间一定是等于期望时间的，所以我们可以二分他重开的时间。

然后利用 $f_{0,0}$ 的值判断可行性即可 （$f_{0,0}<mid$ ）。

### CF 1203F1

有 $i$ 个项目，每一个项目有一个限制和一个权值（权值可以为负数）。

你做一个项目的话，你的信誉需要达到限制 $a_i$，做完之后你的信誉会加上权值 $b_i$。

做完项目的时候信誉不能为 $0$ ，你的初始信誉为 $r$ ，问你能不能在满足条件的情况下做完所有项目。

我们考虑贪心。

你发现做完一个项目之后，你的信誉至少是 $a_i+b_i$。

所以我们可以按照 $a_i+b_i$ 升序排序，并且在此基础上让限制 $a_i$ 更大的排在前面。

然后从头开始模拟做项目的过程就可以了。

你发现 $b_i$ 的正负会导致贪心出现奇怪的错误，会 WA on test 5。

所以我们正负分开排序，循环两次然后判断可行性就可以了。

（upd on 22.10.03：这个本质上是一个 Exchange Argument。）

### CF 814D

有 $n$ 个圆，他们只能是相互包含，相离或者相切的。

将其分为两组，每组中，只有奇数次覆盖的才会算入面积，求可能的最大面积。 （翻译是我扔在luogu讨论区的）

我们考虑一个贪心，把所有覆盖次数是 $2$ 的圆全部扔到一组，剩下的一组。

证明略（

然后又因为包含关系是呈树形的，所以我们让被包含的成为父亲，外层的成为儿子，然后在树上跑一遍 $dfs$ 统计即可（不过关系是有可能变成森林的，所以要建一个超级源点）。

### CF 954G

一共有 $n$ 面墙，初始有 $a_i$ 个弓箭手在第 $i$ 面墙的位置上。一个在 $i$ 位置的弓箭手可以保护 $|i - j| \leq r$ 的所有墙 $j$ 。

你现在可以增派 $k$ 个弓箭手并且任意分配它们的位置。你需要最大化被数量最少的弓箭手保护的墙被弓箭手保护的数量。

$n \leq 5 \times 10^5, 0 \leq r \leq n, 0 \leq k \leq 10^{18} ,0 \leq a_i \leq 10^9$

（题意来自luogu讨论区）

设每一个位置的 defence lv 为 $d_i$

发现这个题的条件是 “最小值最大”。

所以很明显是二分答案。

考虑二分这个值 $mid$ ，每次 check 的时候考虑贪心。

如果有一个位置 $i$ 的  $d_i < mid$

我们在位置 $\min(i+r,n)$ 放 $mid-d_i$ 个 archer 就行了。

### CF 845E

有一个 $n$ 行 $m$ 列的网格。其中有 $k+1$ 个格子着火了。每个时刻，火会蔓延至相邻的格子（八联通）。

现在给出其中 $k$ 个着火的格子，请确定第 $k+1$ 格子，使得网格被烧完的用时最短。

你只需要输出最短用时。

$n,m \le 10^9,k\le 500$。

发现 $n,m$ 非常大，但是 $k$ 非常小。

所以我们从 $k$ 入手，考虑去计算每一个时刻，每一个点能蔓延到的位置。

发现这个东西每秒只扩展一层并且是八联通，所以可以 $\text{O}(1)$ 算这个矩形的顶点的坐标。

又发现这个“时间”没法很好的求出来，但是刚好具有单调性。

所以考虑二分。

每次二分的时候，我们计算出所有矩形覆盖的面积。

然后你去看最边上的那四个角（因为很明显扩展到他们的时间一定是每个方向上最晚的）的空余（如果有）有没有办法用一个点就能覆盖完就行。

因为值域比较大所以我们需要离散化。

计算面积可以使用二维差分或者扫描线。

### CF 547D

给坐标系当中的 $n$ 个整点红蓝染色，要求同一个直线（$y$ or $x$ 方向）上的红蓝节点个数相差不超过一。

问如何染色，多解任意即可。

保证有答案。

首先我们发现一定会有答案，又是要我们把点分成两个部分。

所以我们想到了二分图，手元一下样例可以发现确实是二分图。

所以直接对于同一直线上的点建边然后二分图染色即可。

可以先 $\text{O}(n \log n)$排序再建边，或者用一个 $\text{O}(n)$ 的方法建边，具体方法见提交记录：<https://codeforces.com/contest/547/submission/131984717>