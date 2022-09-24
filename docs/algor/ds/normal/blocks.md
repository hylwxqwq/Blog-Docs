## 原理

非常简单且暴力，以区间加，区间和为例。

考虑把序列分成 $\lceil\sqrt{n}\rceil$ 块，其中第 $i,(1 \le i \le \lfloor\sqrt{n}\rfloor)$ 段的边界是 $l=(i-1)\times \lfloor\sqrt{n}\rfloor +1,r=i \times \lfloor\sqrt{n}\rfloor$ 。

对于第 $\lceil\sqrt{n}\rceil$ 块，如果说第 $\lfloor\sqrt{n}\rfloor$ 块的右边界没有达到 $n$ ，那么就让第 $\lceil\sqrt{n}\rceil$ 块维护区间 $[i\times \lfloor \sqrt{n} \rfloor +1,n]$ 即可。

反之是不会有第 $\lceil\sqrt{n}\rceil$ 块的，因为此时 $\lfloor\sqrt{n}\rfloor=\lceil\sqrt{n}\rceil=\sqrt{n}$。

[![TFwTSK.png](https://s4.ax1x.com/2021/12/17/TFwTSK.png)](https://imgtu.com/i/TFwTSK)

然后预处理出每一个位置 $i$ 在分块之后属于哪一个块 （$\text{belong}_i$）。

对于每一个块，记录其左右边界以及块内和，并维护一个类似于懒标记的 $\text{add}$，表示这个块里的所有数都被加过多少。

每次操作区间 $[l,r]$ 的时候分情况讨论：

1. 如果 $\text{belong}_l=\text{belong}_r$ ，那么直接暴力修改区间 $[l,r]$。
2. 反之，把这个区间两边非整块的部分暴力修改，然后为所有整块的部分打上标记。

[![TFDAGq.png](https://s4.ax1x.com/2021/12/17/TFDAGq.png)](https://imgtu.com/i/TFDAGq)

查询的时候也比较类似：

1. 直接暴力扫一遍 $[l,r]$ ，求和即可（不要忘记把标记加上）
2. 非整块部分暴力扫，整块部分用块内和加上 $\text{add}$ 标记。

因为最多只有 $\lceil\sqrt{n}\rceil$ 块，所以复杂度是 $\text{O}((n+q)\times \sqrt{n})$。

## 习题

### LOJ6277. 数列分块入门 1

区间加单点查，只需要分块的时候记录 $\text{add}$ 配以暴力修改。

查询时带上标记即可。

??? note "Code"
    ```cpp
    #include <bits/stdc++.h>
    using namespace std;

    constexpr int si = 5e4 + 10;
    int n, q, t;
    int a[si], add[si];
    int L[si], R[si], belong[si];

    inline void change(int l, int r, int v) {
        int p = belong[l], q = belong[r];

        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                a[i] += v;
            }
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                a[i] += v;
            }

            for (register int i = L[q]; i <= r; ++i) {
                a[i] += v;
            }

            for (register int i = p + 1; i <= q - 1; ++i) {
                add[i] += v;
            }
        }
    }
    inline int query(int x) {
        return a[x] + add[belong[x]];
    }

    int main() {
        scanf("%d", &n), t = sqrt(n);

        for (register int i = 1; i <= n; ++i) {
            scanf("%d", &a[i]);
        }

        for (register int i = 1; i <= t; ++i) {
            L[i] = (i - 1) * t + 1, R[i] = i * t;
        }

        if (R[t] < n)
            ++t, L[t] = R[t - 1] + 1, R[t] = n;

        for (register int i = 1; i <= t; ++i) {
            for (register int j = L[i]; j <= R[i]; ++j) {
                belong[j] = i;
            }
        }

        for (register int i = 1; i <= n; ++i) {
            int op, l, r, c;
            scanf("%d%d%d%d", &op, &l, &r, &c);

            if (op == 0)
                change(l, r, c);
            else
                printf("%d\n", query(r));
        }

        return 0;
    }
    ```

### Luogu3372 【模板】线段树 1

区间加区间和，分块之后按照上面“原理”部分的做法来就行。

只需要注意一下什么时候要更新块内和，什么时候要打标记就行。

??? note "Code"
    ```cpp
    #include <bits/stdc++.h>
    using namespace std;
    
    #define int long long
    constexpr int si = 2e5 + 10;
    int n, m, t;
    int a[si], add[si], belong[si];
    int L[si], R[si], sum[si];
    
    inline void change(int l, int r, int v) {
        int p = belong[l], q = belong[r];
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                a[i] += v;
            }
    
            sum[p] += v * (r - l + 1);
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                a[i] += v;
            }
    
            sum[p] += v * (R[p] - l + 1);
    
            for (register int i = L[q]; i <= r; ++i) {
                a[i] += v;
            }
    
            sum[q] += v * (r - L[q] + 1);
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                add[i] += v;
            }
        }
    }
    inline int query(int l, int r) {
        int p = belong[l], q = belong[r];
        int res = 0;
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                res += a[i];
            }
    
            res += add[p] * (r - l + 1);
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                res += a[i];
            }
    
            res += add[p] * (R[p] - l + 1);
    
            for (register int i = L[q]; i <= r; ++i) {
                res += a[i];
            }
    
            res += add[q] * (r - L[q] + 1);
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                res += sum[i], res += add[i] * (R[i] - L[i] + 1);
            }
        }
    
        return res;
    }
    
    signed main() {
        scanf("%lld%lld", &n, &m), t = sqrt(n * 1.0);
    
        for (register int i = 1; i <= n; ++i) {
            scanf("%lld", &a[i]);
        }
    
        for (register int i = 1; i <= t; ++i) {
            L[i] = (i - 1) * sqrt(n * 1.0) + 1, R[i] = i * sqrt(n * 1.0);
        }
    
        if (R[t] < n)
            ++t, L[t] = R[t - 1] + 1, R[t] = n;
    
        for (register int i = 1; i <= t; ++i) {
            for (register int j = L[i]; j <= R[i]; ++j) {
                sum[i] += a[j], belong[j] = i;
            }
        }
    
        while (m--) {
            int op, l, r;
            scanf("%lld%lld%lld", &op, &l, &r);
    
            if (op == 1) {
                int k;
                scanf("%lld", &k);
                change(l, r, k);
            } else
                printf("%lld\n", query(l, r));
        }
    
        return 0;
    }
    ```

### LOJ6278. 数列分块入门 2

区间加，区间查询比 $c^2$ 小的数的个数。

首先继续分块，仍旧维护每一个块的 $\text{add}$ 标记。

考虑如何查询区间有多少个数比 $c^2$ 小。

沿用“小段朴素，大段维护”的思想，我们在查询时在两边的两个不完整块进行暴力扫描。

然后对于每一个被包含的整块，考虑在块内二分。

为了二分，我们需要额外多对于每一个块维护一个有序的序列 $v$ 。

当我们暴力修改小段的时候，这个小段所属的块的单调性就会改变，所以我们需要对这个块的 $v$ 进行重排。

如果是大段打标记的话，因为块内每一个数都被加了同一个数，所以单调性不会改变，不需要重排。

那么大段查询的时候只需要把 $c^2$ 减去每个块的标记之后然后二分就可以了。

初始化的时候不要忘记把元素扔进 $v$ 里面！

??? note "Code"
    ```cpp
    
    #include <bits/stdc++.h>
    using namespace std;
    
    constexpr int si = 5e4 + 10;
    constexpr int bsi = ceil(sqrt(si)) + 10;
    int n, t;
    int a[si], belong[si], add[si];
    int L[si], R[si];
    std::vector<int>v[bsi];
    
    inline void reset(int pos) {
        v[pos].clear();
    
        for (register int i = L[pos]; i <= R[pos]; ++i) {
            v[pos].push_back(a[i]);
        }
    
        sort(v[pos].begin(), v[pos].end());
    }
    inline void change(int l, int r, int v) {
        int p = belong[l], q = belong[r];
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                a[i] += v;
            }
    
            reset(p);
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                a[i] += v;
            }
    
            reset(p);
    
            for (register int i = L[q]; i <= r; ++i) {
                a[i] += v;
            }
    
            reset(q);
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                add[i] += v;
            }
        }
    }
    inline int query(int l, int r, int c) {
        int p = belong[l], q = belong[r];
        int res = 0, limit = c * c;
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                if (a[i] + add[p] < limit)
                    ++res;
            }
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                if (a[i] + add[p] < limit)
                    ++res;
            }
    
            for (register int i = L[q]; i <= r; ++i) {
                if (a[i] + add[q] < limit)
                    ++res;
            }
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                int x = limit - add[i];
                res += lower_bound(v[i].begin(), v[i].end(), x) - v[i].begin();
            }
        }
    
        return res;
    }
    
    int main() {
        scanf("%d", &n), t = sqrt(n);
    
        for (register int i = 1; i <= n; ++i) {
            scanf("%d", &a[i]);
        }
    
        for (register int i = 1; i <= t; ++i) {
            L[i] = (i - 1) * sqrt(n) + 1, R[i] = i * sqrt(n);
        }
    
        if (R[t] < n)
            ++t, L[t] = R[t - 1] + 1, R[t] = n;
    
        for (register int i = 1; i <= t; ++i) {
            for (register int j = L[i]; j <= R[i]; ++j) {
                belong[j] = i, v[i].push_back(a[j]);
                reset(i);
            }
        }
    
        for (register int i = 1; i <= n; ++i) {
            int op, l, r, c;
            scanf("%d%d%d%d", &op, &l, &r, &c);
    
            if (op == 0)
                change(l, r, c);
            else
                printf("%d\n", query(l, r, c));
        }
    
        return 0;
    }
    ```

### LOJ6279. 数列分块入门 3

区间加，区间询问前驱。

沿用上一个题的思想，我们小段暴力，大段二分。

但是如果直接对块内二分的话可能会因为没去重而爆炸。

所以我们考虑把上面那题的 `std::vector` 换成 `std::set` 。

那么只需要在每个块的 `std::set` 里面用 `lower_bound` 二分之后令迭代器减一，就可以了。

（因为 `lower_bound` 求的是大于等于 $x$ 的最小元素的迭代器，迭代器在 `std::set` 里面减一之后就是前驱了）

当然，二分的时候一定要使用容器本身的`lower_bound` ，不然效率会及其低下（具体可以看[这里](https://stackoverflow.com/questions/31821951/c-difference-between-stdlower-bound-and-stdsetlower-bound)）。

另外不要忘记初始化的时候把元素扔进 `std::set` 里面，也不要忘记判二分之后二分出 `s.begin()` 的情况。

??? note "Code"
    ```cpp
    
    #include <bits/stdc++.h>
    using namespace std;
    
    constexpr int si = 1e5 + 10;
    constexpr int bsi = ceil(sqrt(si)) + 10;
    int n, t;
    int a[si], add[si], belong[si];
    int L[si], R[si];
    std::set<int>s[bsi];
    
    inline void reset(int pos) {
        s[pos].clear();
    
        for (register int i = L[pos]; i <= R[pos]; ++i) {
            s[pos].insert(a[i]);
        }
    }
    inline void change(int l, int r, int c) {
        int p = belong[l], q = belong[r];
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                a[i] += c;
            }
    
            reset(p);
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                a[i] += c;
            }
    
            reset(p);
    
            for (register int i = L[q]; i <= r; ++i) {
                a[i] += c;
            }
    
            reset(q);
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                add[i] += c;
            }
        }
    }
    inline int query(int l, int r, int c) {
        int p = belong[l], q = belong[r];
        int res = -1;
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                if (a[i] + add[p] < c)
                    res = max(res, a[i] + add[p]);
            }
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                if (a[i] + add[p] < c)
                    res = max(res, a[i] + add[p]);
            }
    
            for (register int i = L[q]; i <= r; ++i) {
                if (a[i] + add[q] < c)
                    res = max(res, a[i] + add[q]);
            }
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                int x = c - add[i];
                std::set<int>::iterator it;
                it = s[i].lower_bound(x);
    
                if (it == s[i].begin())
                    continue;
    
                it--, res = max(res, *it + add[i]);
            }
        }
    
        return res;
    }
    
    int main() {
        scanf("%d", &n), t = sqrt(n);
    
        for (register int i = 1; i <= n; ++i) {
            scanf("%d", &a[i]);
        }
    
        for (register int i = 1; i <= t; ++i) {
            L[i] = (i - 1) * sqrt(n) + 1, R[i] = i * sqrt(n);
        }
    
        if (R[t] < n)
            ++t, L[t] = R[t - 1] + 1, R[t] = n;
    
        for (register int i = 1; i <= t; ++i) {
            for (register int j = L[i]; j <= R[i]; ++j) {
                belong[j] = i, s[i].insert(a[j]);
            }
        }
    
        for (register int i = 1; i <= n; ++i) {
            int op, l, r, c;
            scanf("%d%d%d%d", &op, &l, &r, &c);
    
            if (op == 0)
                change(l, r, c);
            else
                printf("%d\n", query(l, r, c));
        }
    
        return 0;
    }
    ```

### LOJ6280. 数列分块入门 4

区间加，区间求和并取模。

和 Luogu3372 基本没有区别，只是在查询的时候多了个取模而已。

??? note "Code"
    ```cpp
    #include <bits/stdc++.h>
    using namespace std;
    
    #define int long long
    constexpr int si = 2e5 + 10;
    int n, m, t;
    int a[si], add[si], belong[si];
    int L[si], R[si], sum[si];
    
    inline void change(int l, int r, int v) {
        int p = belong[l], q = belong[r];
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                a[i] += v;
            }
    
            sum[p] += v * (r - l + 1);
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                a[i] += v;
            }
    
            sum[p] += v * (R[p] - l + 1);
    
            for (register int i = L[q]; i <= r; ++i) {
                a[i] += v;
            }
    
            sum[q] += v * (r - L[q] + 1);
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                add[i] += v;
            }
        }
    }
    inline int query(int l, int r, int mod) {
        int p = belong[l], q = belong[r];
        int res = 0;
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                res = (res + a[i]) % mod;
            }
    
            res = (res + add[p] * (r - l + 1)) % mod;
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                res = (res + a[i]) % mod;
            }
    
            res = (res + add[p] * (R[p] - l + 1)) % mod;
    
            for (register int i = L[q]; i <= r; ++i) {
                res = (res + a[i]) % mod;
            }
    
            res = (res + add[q] * (r - L[q] + 1)) % mod;
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                res = (res + sum[i]) % mod, res = (res + add[i] * (R[i] - L[i] + 1)) % mod;
            }
        }
    
        return res % mod;
    }
    
    signed main() {
        scanf("%lld", &n), t = sqrt(n);
    
        for (register int i = 1; i <= n; ++i) {
            scanf("%lld", &a[i]);
        }
    
        for (register int i = 1; i <= t; ++i) {
            L[i] = (i - 1) * sqrt(n) + 1, R[i] = i * sqrt(n);
        }
    
        if (R[t] < n)
            ++t, L[t] = R[t - 1] + 1, R[t] = n;
    
        for (register int i = 1; i <= t; ++i) {
            for (register int j = L[i]; j <= R[i]; ++j) {
                sum[i] += a[j], belong[j] = i;
            }
        }
    
        for (register int i = 1; i <= n; ++i) {
            int op, l, r, c;
            scanf("%lld%lld%lld%lld", &op, &l, &r, &c);
    
            if (op == 0)
                change(l, r, c);
            else
                printf("%lld\n", query(l, r, c + 1));
        }
    
        return 0;
    }
    ```

### LOJ6281. 数列分块入门 5

区间开根号并向下取整，区间求和。

我们先考虑一个好写一点的做法：

还是分块，对于任意的被包含在修改区间的完整块直接全部暴力开根号，小块也是全部暴力开根号。

如果块内的最大值 $\le 1$ ，那么这个块就不用开根号。

所以每次暴力修改完一个块（或块的部分）之后需要更新一下区间最大值。

又发现对于 `int` 范围内的正整数，开 $5$ 次根号就可以 $=1$ 了。

则容易证明，每个块最多会被开根号 $4$ 次，所以可过。

??? note "Code"
    ```cpp
    
    #include <bits/stdc++.h>
    using namespace std;
    
    constexpr int si = 5e4 + 10;
    int n, t;
    int a[si], add[si], belong[si];
    int L[si], R[si], sum[si], maxv[si];
    
    inline void change(int l, int r) {
        int p = belong[l], q = belong[r];
    
        if (p == q) {
            if (maxv[p] == 0 || maxv[p] == 1)
                return;
    
            for (register int i = l; i <= r; ++i) {
                int rmp = a[i];
                a[i] = sqrt(a[i]), sum[p] -= (rmp - a[i]);
            }
    
            int mx = 0;
    
            for (register int i = L[p]; i <= R[p]; ++i) {
                mx = max(mx, a[i]);
            }
    
            maxv[p] = mx;
        } else {
            if (maxv[p] != 0 && maxv[p] != 1) {
                for (register int i = l; i <= R[p]; ++i) {
                    int rmp = a[i];
                    a[i] = sqrt(a[i]), sum[p] -= (rmp - a[i]);
                }
    
                int mx = 0;
    
                for (register int i = L[p]; i <= R[p]; ++i) {
                    mx = max(mx, a[i]);
                }
    
                maxv[p] = mx;
            }
    
            if (maxv[q] != 0 && maxv[q] != 1) {
                for (register int i = L[q]; i <= r; ++i) {
                    int rmp = a[i];
                    a[i] = sqrt(a[i]), sum[q] -= (rmp - a[i]);
                }
    
                int mx = 0;
    
                for (register int i = L[q]; i <= R[q]; ++i) {
                    mx = max(mx, a[i]);
                }
    
                maxv[q] = mx;
            }
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                if (maxv[i] == 0 || maxv[i] == 1)
                    continue;
    
                int mx = 0;
    
                for (register int j = L[i]; j <= R[i]; ++j) {
                    int rmp = a[j];
                    a[j] = sqrt(a[j]), sum[i] -= (rmp - a[j]);
                    mx = max(mx, a[j]);
                }
    
                maxv[i] = mx;
            }
        }
    }
    inline int query(int l, int r) {
        int p = belong[l], q = belong[r];
        int res = 0;
    
        if (p == q) {
            for (register int i = l; i <= r; ++i) {
                res += a[i];
            }
        } else {
            for (register int i = l; i <= R[p]; ++i) {
                res += a[i];
            }
    
            for (register int i = L[q]; i <= r; ++i) {
                res += a[i];
            }
    
            for (register int i = p + 1; i <= q - 1; ++i) {
                res += sum[i];
            }
        }
    
        return res;
    }
    
    int main() {
        scanf("%d", &n), t = sqrt(n);
    
        for (register int i = 1; i <= n; ++i) {
            scanf("%d", &a[i]);
        }
    
        for (register int i = 1; i <= t; ++i) {
            L[i] = (i - 1) * sqrt(n) + 1, R[i] = i * sqrt(n);
        }
    
        if (R[t] < n)
            ++t, L[t] = R[t - 1] + 1, R[t] = n;
    
        for (register int i = 1; i <= t; ++i) {
            for (register int j = L[i]; j <= R[i]; ++j) {
                belong[j] = i, sum[i] += a[j], maxv[i] = max(maxv[i], a[j]);
            }
        }
    
        for (register int i = 1; i <= n; ++i) {
            int op, l, r, c;
            scanf("%d%d%d%d", &op, &l, &r, &c);
    
            if (op == 0)
                change(l, r);
            else
                printf("%d\n", query(l, r));
        }
    
        return 0;
    }
    // it is better to use tag & rem the time of sqrt.
    ```

这里有另一种**写法**：

考虑把整块的开根号次数记录一下。

摊平 $\text{add}$ 的时候只需要暴力处理开 $1,2,3,4$ 次根号的情况，再加上上面的做法中的优化即可。

不过处理开超过 $5$ 次根号的情况需要注意区分 $0$ 和 $1$。

