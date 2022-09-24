
## 概述

又叫做字典树，思想是把字符串拆分成一个一个字符，然后将字符放到树的“边”上（**字符指针**），利用节点储存字符串结束等信息。

普通 Trie 一般用于检索字符串是否存在，或者作为 AC 自动机的一部分，用于多模式匹配。

变种的 01Trie 可以实现诸如查询和 $x$ 异或值最大的数，维护异或和等**位运算相关操作**，尤其是异或。

结构一般长这样：

[![LRaR9f.png](https://s1.ax1x.com/2022/04/22/LRaR9f.png)](https://imgtu.com/i/LRaR9f)

## Trie

Trie 支持两种操作，插入和查询。

首先明确 Trie 的结构到底是怎么样的，Trie 将字符储存于树的边上，并在节点上有一个 $end$ 标记，表示当前节点是否是一个字符串的结尾。

实现时利用一个数组 $tr[x,ch]$，表示节点 $x$ 的 $ch$ 字符指针指向的节点编号。

类比动态开点线段树的思想，Trie 只会在一个节点被建立时给予一个编号。

初始的时候 Trie 只有一个节点 $root$，并且所有的 $tr$ 都指向 $\text{NULL}$ （空）。

### 插入字符串

将一个新的字符串 $s$ 插入到 Trie 当中。

初始时令一个指针 $p = root$，然后从最低位开始扫描字符串 $s$。 

如果当前扫描到的字符为 $s_i$，令 $p = tr[p,s_i]$。

如果 $tr[p,s_i]$ 不存在，新建一个节点 $q$，令 $tr[p, s_i] = q$，然后把 $p$ 跳到 $q$。

反之直接把 $p$ 跳过去即可。

然后令 $i = i + 1$，扫描下一个字符。

重复这个过程，直到 $s$ 被扫描完毕，在当前的节点 $p$ 上打上一个标记 $end[p] = \text{true}$。

放一张早年做的比较糙的 GIF：

[![cSnwzn.gif](https://z3.ax1x.com/2021/03/27/cSnwzn.gif)](https://imgtu.com/i/cSnwzn)

### 询问

查询一个字符串 $s$ 是否存在于 Trie 当中。

类似 `Insert` 操作，设一个指针 $p = root$，然后扫描 $s_i$。

不断跳 $p = tr[p, s_i]$，如果还没有扫完 $s$，就已经出现了 $tr[p,s_i]$ 指向 $\text{NULL}$ 的情况，则返回 $\text{false}$。

如果已经扫描完了 $s$，且 $end[p] = \text{true}$，返回 $\text{true}$。

反之返回 $\text{false}$。

### 代码实现

??? note "普通 Trie 树"
    ```cpp
    // 定义 NULL 为 0，字符集为 a~z。
    int tr[si][27];
    bool exist[si];
    int tot, root;
    
    void init() {
        memset(tr, 0, sizeof tr);
        memset(exist, false, sizeof exist);
        tot = 0, root = ++tot;
    }
    void insert(string s) {
        int p = root;
        for(int i = 0; i < (int)s.size(); ++i) {
            int ch = (int) (s[i] - 'a') + 1;
            if(!tr[p][ch])
                tr[p][ch] = ++tot;
            p = tr[p][ch];
        }
        exist[p] = true;
    }
    bool query(string s) {
        int p = root;
        for(int i = 0; i < (int)s.size(); ++i) {
            int ch = (int) (s[i] - 'a') + 1;
            if(!tr[p][ch])
                return false;
            p = tr[p][ch];
        }
        return exist[p];
    }
    ```

### 一些性质

1. 两个字符串的最长公共前缀是他们的尾标记所在的两个节点的 lca 到根的路径组成的字符串。

   我认为 LCP 也可以这么暴力求。

2. 可以把尾标记改成 `int`，可以用来统计词频。

3. 也可以方便的统计某个前缀在所有串当中出现的次数（结合 2）。

4. 是 AC 自动机的一部分。

## 01Trie

类似于普通的 Trie，只不过将字符集变为了 $\{0,1\}$ 以维护一些二进制相关的信息。

### 求与 x 异或的最值

#### 询问最值

将所有数二进制拆分，包括 $x$。

然后把这些二进制数**从高位到低位**插入进 Trie。

然后从高到低扫描 $x$ 二进制下的每一位，同时维护一个指针 $p$ 往下跳。

假设 $x$ 当前的这一位为 $k$，那么想要让异或和更大，就是让越高的位尽可能的为 $1$。

所以我们需要尽量往 $k \operatorname{xor} 1$ （相反的）的指针走，如果当前节点没有 $k \operatorname{xor} 1$ 这个指针，那么就只好走 $k$ 这个指针。

跳到叶子节点之后，从 $p$ 到 $root$ 的路径组成的数就是所求的，和 $x$ 异或起来的值最大的数，

在插入每个数之后，在对应结束的节点打一个标记，记录从这个节点到根的路径组成的数是多少即可。

异或起来最小同理。

#### 代码实现

??? note "01Trie"
    ```cpp
    using i64 = long long; 
    
    const int si = 1e5 + 10; 
    const int k = 32; 
    int tr[k * si][2]; 
    i64 value[k * si]; 
    int tot = 0, root = ++tot; 
    
    int newnode() {
        tr[++tot][0] = tr[tot][1] = value[tot] = 0;
        return tot;
    }
    int cacid(int num, int pos) {
        return (num >> pos) & 1; 
    }
    void insert(int num) {
        int p = root; 
        for(int i = 32; i >= 0; --i) {
            int ch = cacid(num, i); 
            if(!tr[p][ch]) 
                tr[p][ch] = newnode();
            p = tr[p][ch]; 
        }
        value[p] = num; 
    }
    // 查询异或 x 最大的一个。
    i64 query(i64 num) {
        int p = root; 
        for(int i = 32; i >= 0; --i) {
            int ch = cacid(num, i); 
            if(tr[p][ch ^ 1])
                p = tr[p][ch ^ 1]; 
            else
                p = tr[p][ch]; 
        }
        return value[p]; 
    }
    ```

### 维护异或和

01 Trie 可以用来维护一些数的异或和，支持插入，删除，还有全局加一操作。

我出的一道题里面有用到一个 Trick：异或和的二进制下某一位的值，取决于所有数的这一位的 $1$ 的个数的**奇偶性**。

也就是，异或和 $xorv$ 的第 $k$ 位是 $1$，当且仅当所有数当中有 **奇数个** 数的第 $k$ 为是 $1$。

那么利用它来考虑维护全局异或和。

不过维护异或和的时候和一般的 01Trie 不一样，此时的 01Trie 应当是**从低位到高位插入**的，以方便全局加一操作的处理。

#### 信息上传

用类似线段树的思想，我们自底向上收集信息。

设 $xorv_i$ 表示节点 $i$ 的子树所维护的异或和。

设 $wei_i$ 指节点 $i$ 到其父亲节点这条边上数值的数量（权值），也就是这条边 被所有插入的数 拆成二进制之后 在树上代表的路径 覆盖的次数。

不过我们实际上**不需要知道具体**维护了哪些数，维护了多少个，

也就是说，我们只需要知道 $wei$ 的**奇偶性**就行了。

那么，对于一个非叶子节点 $p$，可以有以下的过程：

首先，令 $wei_p = wei_{tr[p][0]} + wei_{tr[p][1]}$。

如果 $p$ 的 $0$ 指针指向的节点不为 $\text{NULL}$，那么让 $xorv_p = xorv_p \operatorname{xor} (xorv_{tr[p][0]}<<1)$，也就是**先对齐**每一位，然后异或起来。

如果 $p$ 的 $1$ 指针指向的节点不为 $\text{NULL}$，那么让 $xorv_p = xorv_p \operatorname{xor} ((xorv_{tr[p][1]} << 1) \operatorname{or} (wei_{tr[p][1]} \operatorname{and} 1))$。

也就是先对齐每一位，然后看奇偶性。

#### 插入 / 删除

插入直接递归，如果遇到空节点，那么新建即可。

为了之后全局加一**进位**方便，我们强制让每一个数都变成 `MaxDepth` 位，再插入进 Trie 当中。

`MaxDepth` 是你选择的 Trie 树的最大深度，一般比要插入的数的最高位数多出两三位。

当深度到达 `MaxDepth` 之后，令 $wei_p + 1$，然后不断向上传递信息即可。

这样子就能保证，每插入一个数 $x$，$x$ 的二进制表示 在 Trie 树上代表的路径 上的所有节点的 $wei$ 都会 $+1$。  

删除操作也是类似的，只不过变成了让最后一位（最高位）所对应的节点的 $wei -1$，再向上收集信息。

#### 全局加一

思考一下，在二进制下的加一操作实质上是什么？

$$(10011)_2 + 1 = (10100)_2 \\ (10110)_2 + 1 = (10111)_2$$

实际上就是：从最低位开始，找到第一个 $0$，将其取反，然后将它后面（到最低位）的所有 $1$ 变成 $0$。

对应到 01Trie 上，就是从 $root$ 开始递归下去，找到第一个非空的 $tr[p][0]$ 指针，然后交换 $tr[p][0],tr[p][1]$（取反）。

沿着**交换之后**的 $tr[p][0]$ 递归下去操作即可。

#### 代码实现

??? note "维护异或和"
    ```cpp
    #include <bits/stdc++.h>
    
    using namespace std;
    
    const int si = 1e4 + 10;
    const int MaxDepth = 21;
    
    int tr[si * (MaxDepth + 1)][2];
    int wei[si * (MaxDepth + 1)], xorv[si * (MaxDepth + 1)];
    int tot = 0, root = ++tot;
    // 其实这里 root 可以不用赋值，递归开点的时候会自动给编号的。
    
    int newnode() {
        tr[++tot][0] = tr[tot][1] = wei[tot] = xorv[tot] = 0;
        return tot;
    }
    void maintain(int p) {
        wei[p] = xorv[p] = 0;
        // 为了应对不断的删除和插入，每次维护 p 的时候都令 wei, xorv = 0。
        // 也就是每次都**重新收集一次信息**，而不是从原来的基础上修改。
        if(tr[p][0]) {
            wei[p] += wei[tr[p][0]];
            xorv[p] ^= (xorv[tr[p][0]] << 1);
            // 因为儿子所维护的异或和实际上比 p 少一位，
            // 如果要按位异或就要让儿子的异或和左移一位，和 p 对齐。
        }
        if(tr[p][1]) {
            wei[p] += wei[tr[p][1]];
            xorv[p] ^= (xorv[tr[p][1]] << 1) | (wei[tr[p][1]] & 1);
            // 利用奇偶性计算。
        }
        wei[p] = wei[p] & 1;
        // 每插入一次或者删除一次，奇偶性都会变化。
    }
    // 类似线段树的 pushup，从底向上收集信息。
    // 换种说法，是更新节点 p 的信息。
    void insert(int &p, int x, int depth) {
        if(!p) 
            p = newnode();
        if(depth > MaxDepth) {
            wei[p] += 1;
            return;
        }
        insert(tr[p][x & 1], x >> 1, depth + 1);
        // 从低到高位插入，所以是 x >> 1。
        maintain(p);
    }
    // 插入元素 x。
    void remove(int p, int x, int depth) {
        // 不知道是不是应该写 > MaxDepth - 1 还是 > MaxDepth ？
        if(depth == MaxDepth) {
            wei[p] -= 1;
            return;
        }
        remove(tr[p][x & 1], x >> 1, depth + 1);
        maintain(p);
    }
    // 删除元素 x，但是 x 不能是不存在的元素。
    // 否则会访问空节点 0 然后继续往下，会出错。
    void addall(int p) {
        swap(tr[p][0], tr[p][1]);
        if(tr[p][0]) 
            addall(tr[p][0]);
        maintain(p);
        // 交换后下面都被更改了，需要再次 maintain。
    }
    // 全部加一
    
    int main() {
        int n;
        cin >> n;
        std::vector<int> v(n + 1);
        for(int i = 1; i <= n; ++i) {
            cin >> v[i],
            insert(root, v[i], 0);
        }
        cout << xorv[root] << endl;
        // 查询总异或和
        int m;
        cin >> m;
        for(int i = 1; i <= m; ++i) {
            int x, y;
            cin >> y >> x;
            if(y == 0)
                remove(root, x, 0);
                // remove 和 addall 混用时小心 remove 掉不存在的元素！
            else
                addall(root);
            cout << xorv[root] << endl;
        }
    }
    ```

### 01Trie 合并

合并两颗 01Trie。

和线段树合并是比较类似的，用同样的思想思考即可。

就是把两个节点中的一个的信息合并到另外一个上，并返回合并之后的编号。

（当然，这里也可以根据情况把他们合并到一个新的节点上）。

如果有一个节点为空，那么就返回另外一个节点的编号即可。


??? note "Merge"
    ```cpp
    int merge(int p, int q) {
        if(!p) 
            return q;
        if(!q) 
            return p;
        wei[p] += wei[q], xorv[p] ^= xorv[q];
        tr[p][0] = merge(tr[p][0], tr[q][0]);
        tr[p][1] = merge(tr[p][1], tr[q][1]);
        return p;
    }
    ```

---

参考资料：[字典树 (Trie) - OI Wiki (oi-wiki.org)](https://oi-wiki.org/string/trie/)