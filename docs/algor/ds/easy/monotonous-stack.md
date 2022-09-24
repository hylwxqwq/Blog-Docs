
## 概述

单调栈，顾名思义，就是栈内元素以某种方式单调的栈。

单调队列可以用来维护滑动窗口最值，而单调栈可以维护一个序列中每个元素的 NGE，NLE。

NGE = Next Greater Element, L 同理。

就是维护对于 $\forall a_{i}$，它左边的第一个使得 $a_{j} > a_{i}$ 的 $j$。

## 应用

NGE 的暴力做法是对于每个询问 $O(n)$ 跑一遍，复杂度 $O(nm)$ 不能接受。

考虑维护一个栈表示“当前还未确定 NGE” 的元素集合。

然后让栈从栈底到栈顶单调递减。

这样，如果有一个元素入栈时，破坏了栈内单调性，那么不断弹出栈内的元素，直到栈顶不小于这个元素，然后再让这个元素入栈。

被这个元素弹出的所有元素的 NGE 就是这个新进栈元素的下标，原因显然。

如果处理完整个序列之后，栈不为空，那么说明栈中元素没有 NGE。

??? note "实现（[洛谷模板题](https://www.luogu.com.cn/problem/P5788)）："
    ```cpp
    // author : black_trees
    
    #include <stack>
    #include <cstdio>
    #include <cstring>
    #include <iostream>
    #include <algorithm>
    
    #define endl '\n'
    #define meow(x) cerr << #x << " = " << x
    
    using namespace std;
    using i64 = long long;
    
    const int si = 3e6 + 10;
    const int inf = 0x3f3f3f3f;
    
    int ans[si];
    int a[si], n;
    std::stack<int> mns; // ele that waiting to cal nxt greater ele. 
    
    int main() {    
    
        cin.tie(0) -> sync_with_stdio(false);
        cin.exceptions(cin.failbit | cin.badbit);
    
        cin >> n;
        for(int i = 1; i <= n; ++i)
            cin >> a[i];
        for(int i = 1; i <= n; ++i) {
            while(!mns.empty() && a[i] > a[mns.top()])
                ans[mns.top()] = i, mns.pop();
            mns.push(i);
        }
        for(int i = 1; i <= n; ++i) 
            cout << ans[i] << " ";
        cout << endl;
    
        return 0;
    }
    ```

复杂度 $O(n)$。

单调栈还可以用来[优化 DP](../../dp/opt/monotonous-stack-optimize.md)。

## 习题

??? note "[POJ3250 Bad Hair Day](http://poj.org/problem?id=3250)"
    有 $N$ 头牛从左到右排成一排，每头牛有一个高度 $h_i$，设左数第 $i$ 头牛与「它右边第一头高度 $≥h_i$」的牛之间有 $c_i$ 头牛，试求 $\sum_{i=1}^{N} c_i$。

比较基础的应用有这一题，就是个单调栈的简单应用，记录每头牛被弹出的位置，如果没有被弹出过则为最远端，稍微处理一下即可计算出题目所需结果。

另外，单调栈也可以用于离线解决 RMQ 问题。

我们可以把所有询问按右端点排序，然后每次在序列上从左往右扫描到当前询问的右端点处，并把扫描到的元素插入到单调栈中。这样，每次回答询问时，单调栈中存储的值都是位置 $\le r$ 的、可能成为答案的决策点，并且这些元素满足单调性质。此时，单调栈上第一个位置 $\ge l$ 的元素就是当前询问的答案，这个过程可以用二分查找实现。使用单调栈解决 RMQ 问题的时间复杂度为 $O(q\log q + q\log n)$，空间复杂度为 $O(n)$。