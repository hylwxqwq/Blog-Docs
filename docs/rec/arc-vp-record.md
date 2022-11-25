
不管怎么样，反正听了 zjk 的建议，vp ARC 打着玩就当思维训练了。

## ARC080

> CSP2022 考前一天和 [hfy](https://www.cnblogs.com/Sakurajima-Mai/p/16836914.html) 还有 wcx 随机跳了一场 ARC 打着玩，就当开拓思维了。
> 
> 发现远古 ARC 和 ABC 是成对出现的，相当于 div1 和 div2。
> 
> 感觉 CDE 都是可做题，CD 比较平凡，E 是小清新思维题，感觉需要擅长观察结论才弄得出来。
> F 不太会，以后可以看看


### C - 4-adjacent

> 给你一个序列 $a$，你要重排这个序列，使得相邻两项乘积都是 4 的倍数。
> 
> 如果有解输出 `Yes`,否则输出 `No`。

比较简单，考虑分类，一类是奇数，一类是 $4$ 的倍数，另外一类是不是 $4$ 的倍数的偶数。

注意到奇数的两边不是边界就一定要是 $4$，如果没有 $2 \times \text{odd}$ 形式的数，那只要奇数个数小于等于 $4$ 的倍数的个数 $+ 1$ 即可。

如果有 $2 \times \text{odd}$ 形式的数，那么奇数的个数必须要是 $\le$ $4$ 的倍数的个数的。

然后就没了。

??? note "Code"

    ```cpp
    int c = 0, cc = 0, ccc = 0;
    cin >> n;
    for(int i = 1; i <= n; ++i) {
        cin >> a[i];
        if(a[i] & 1) c++;
        else if(a[i] % 4 == 0) ccc++;
        else if(a[i] % 2 == 0) cc++;
    }
    if((c > ccc && cc) || c - 1 > ccc) 
        cout << "No" << endl;
    else cout << "Yes" << endl;
    ```

### D - Grid Coloring

> 给你一个序列 $a$，长度为 $q$，给定一个 $n\times m$ 的网格。
> 
> 保证 $\sum a_i = n \times m$，现在要求你染 $q$ 个 4-联通块出来，其中颜色 $i$ 要有 $a_i$ 个格子，构造方案。
> 
> $100$。

笨比题，蛇形染色即可，显然一定有解。

### E - Young Maids

题目名字好怪。

> 给你一个 $1 \sim n, (n \equiv 0 (\mod 2))$ 的排列 $p$，你每次可以取出相邻的两个元素，把他们按照原来的顺序扔到一个队列的头部。
> 
> 问你最后可能得到的字典序最小的排列是什么。
> 
> 数据范围 $2e5$。

感觉很小清新，需要一定观察能力？

首先正着显然不好搞，**考虑倒过来观察合法解的形状**（感觉和 221025C 的罪人挽歌那题想法类似）。

首先注意到如果取了 $(x, y)$ 这两个位置， 那么 $[x + 1, y - 1]$ 这个区间是不能和其它区间组合的，也就是不能跨区间操作。

由此可以发现每次取到的是一个奇数项 + 一个偶数项的形式（不然一定没法把任意一个 $[x + 1, y - 1]$ 取完（因为这样长度不是偶数））。

要字典序最小，谈就需要靠后取到的奇数项尽量小，因为这是一个排列所以我们不用考虑偶数项大小（不会有相同的）。

然后每次我们把当前的可以取的区间拿出来放进一个堆，关键字是奇数项最小值。

每次贪心地取堆顶，决策完之后断三个区间出来，插入回去，不断取直到堆为空，然后就做完了。

维护原序列奇数项的区间最小值和偶数项的区间最小值直接 RMQ，中间空出来的用 $+\infty$ 补全就行。

??? note "Code"   
    ```cpp
    // author : black_trees
    
    #include <set>
    #include <cmath>
    #include <stack>
    #include <queue>
    #include <cstdio>
    #include <vector>
    #include <cstring>
    #include <iostream>
    #include <algorithm>
    
    using namespace std;
    using i64 = long long;
    
    const int si = 2e5 + 10;
    const int inf = 0x3f3f3f3f;
    
    int n;
    int a[si], b[si], p[si];
    
    class Segment_Tree {
        private:
            struct node {
                int l, r;
                int minv;
            } t[si << 2];
            inline void pushup(int p) {
                t[p].minv = min(t[p << 1].minv, t[p << 1 | 1].minv);
            }
        public:
            void build(int p, int l, int r) {
                t[p].l = l, t[p].r = r;
                if(l == r) { t[p].minv = b[l]; return; }
                int mid = (l + r) >> 1;
                build(p << 1, l, mid), build(p << 1 | 1, mid + 1, r);
                pushup(p);
            }
            int query(int p, int ql, int qr) {
                int ret = inf, l = t[p].l, r = t[p].r;
                if(ql <= l && r <= qr) return t[p].minv;
                int mid = (l + r) >> 1;
                if(ql <= mid) ret = min(ret, query(p << 1, ql, qr));
                if(qr > mid) ret = min(ret, query(p << 1 | 1, ql, qr));
                return ret;
            }
    } tr[2];
    
    struct Interval {
        int l, r;
        int odd, even;
        bool operator < (const Interval &t) const {
            return odd > t.odd;
        }
    };
    std::priority_queue<Interval> q;
    
    void Insert(int l, int r) {
        if(l > r) return;
        int x = tr[l & 1].query(1, l, r), y = tr[l & 1 ^ 1].query(1, p[x] + 1, r);
        q.push({l, r, x, y});
    }
    void Assign(Interval t)  {
        Insert(t.l, p[t.odd] - 1), Insert(p[t.odd] + 1, p[t.even] - 1), Insert(p[t.even] + 1, t.r);
    }
    
    int main() {
    
        cin.tie(0) -> sync_with_stdio(false);
        cin.exceptions(cin.failbit | cin.badbit);
    
        cin >> n;
        for(int i = 1; i <= n; ++i)
            cin >> a[i], p[a[i]] = i;
    
        memset(b, 0x3f, sizeof b);
        for(int i = 1; i <= n; i += 2)
            b[i] = a[i];
        tr[1].build(1, 1, n);
        memset(b, 0x3f, sizeof b);
        for(int i = 2; i <= n; i += 2) 
            b[i] = a[i];
        tr[0].build(1, 1, n);
        
        Insert(1, n);
        while(!q.empty()) {
            Interval u = q.top(); q.pop();
            cout << u.odd << " " << u.even << " ";
            Assign(u);
        }
    
        return 0;
    }
    ```
### F - Unknown.

我不会。

## ARC076

> 这次是 NOIP2022 考前和 hfy 一起进行脑洞打开。
>
> 因为懒所以盒盒代码。

### C - Reconciled?

> 给你 $n$ 只带编号的狗，$m$ 只带编号的猴。
>
> 要求出满足没有任意的狗/猴连通块的排列方式。
>
> $1\le n, m\le 1e5$。

简单题，注意到 $|n - m| \ge 2$ 必定无解。

然后 $n = m$ 的时候答案是 $n!m!$。

如果是 $|n - m| = 1$，答案是 $2n!m!$。

### D - Built?

> 定义两个点的 $dis$ 为 $\min(|x_1 - x_2|, |y_1 - y_2|))$。
>
> 给定 $n,(1\le n \le 1e9)$ 个点，求使得这些点联通的代价最小值。
>
> 联通两点的代价是 $dis$。

本质是求最小 dis 生成树，直接加边是 $O(n^2)$ 的不能接受。

注意到其实我们可以对 $x, y$ 分别排个序，然后对于一个点，让他和它在排列上的左右两点连边就行了。

然后一遍 MST 完事。

### E - Connected?

> 给出一个 $R\times C$ 的棋盘，其中 $1$ 到 $n$ 之间的每个正整数都会在棋盘上出现两次，
>
> 给定第 $i$ 个数出现的位置，要求把每一对相同的数用线（粗细忽略不计）连起来，且线不能相交也不能越过棋盘边界，求是否能完成。
>
> $R,C 1e8, n 1e5$。

注意到只有两端在边界的线才会对答案造成影响，其它的在里面转几下就行了。

所以只需要考虑判所有两端都在边界的点对形成的连线是否相交。

注意到这个实际上可以钦定一个转圈的方向（顺时针，逆时针），把这个东西当成括号序列。然后判断一下括号序列是否合法就行。

### F - Unknown

我不会。
