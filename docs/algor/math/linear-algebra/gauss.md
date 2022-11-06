
## 概述

简单来说就是用来解线性方程组的一个算法。

线性方程组就是每个变量的最高次数都是一次的 $n$ 元方程组。

类似这样：

$$
\begin{cases}
    2x_1+4x_2+5x_3 = 23\\
    3x_1+5x_2+2x_3 = 14\\
    4x_1+6x_2+3x_3 = 24
\end{cases}
$$

（随手写的不一定有解）

小学数学学过，解二元一次方程组需要用到消元法，注意到消元法的本质是对变量的系数进行运算，所以我们可以直接写出方程组的系数矩阵：

$$
\begin{bmatrix}   
    2 & 4 & 5 \\
    3 & 5 & 2 \\
    4 & 6 & 3
\end{bmatrix} 
$$

线性方程组的等号右边都是一个常数，于是我们把这个常数也写进去：

$$
\begin{bmatrix}   
    2 & 4 & 5 \ |\ 23\\
    3 & 5 & 2 \ |\ 14 \\
    4 & 6 & 3 \ |\ 24
\end{bmatrix} 
$$

这就是一个 $n\times (n + 1)$ 的增广矩阵，我们要做的就是把这个增广矩阵通过一些变换变成这样的形式：

$$
\begin{bmatrix}
    1 & c_{1,2} &c_{1,3} \ &|\ x\\
    0 & 1 & c_{2,3}\ &|\ y\\
    0 & 0 & 1\ &|\ z\\
\end{bmatrix}
$$

其中 $c$ 是一个常数，$x,y,z$ 都是常数。

这样的矩阵被叫做方程组的上三角形矩阵。

这个上三角形矩阵可以再化简成一个简化阶梯形矩阵（方式是从下往上代入）：

$$
\begin{bmatrix}
    1 & 0 & 0\ |\ a\\
    0 & 1 & 0\ |\ b\\
    0 & 0 & 1\ |\ c\\
\end{bmatrix}
$$

这个简化阶梯形矩阵就直接表示了方程组的唯一解 $x_1 = a, x_2 = b, x_3 = c$。

把增广矩阵变成上三角矩阵，再变成简化阶梯形矩阵的过程就是高斯消元。

具体分三步：

1. 扫描 $i \in [1, n]$，对于 $i$，从 $[i,n]$ 行当中   找到一个 $x_{1} \sim x_{i-1}$ 的系数都为零，$x_i$ 的系数不为零的行，把它交换到第 $i$ 行（注意这里不能从 $[1,i)$ 里面找，不然解的系数就乱了）。
2. 对于每个 $i$，扫描 $j \in [1, n]$，对于 $\forall j \not ={i}$，利用当前第 $i$ 行的系数消掉第 $j$ 行的 $x_i$ 的系数，将其变为零（具体方式是利用初等行变换，令第 $i$ 行乘上一个系数使得 $c_{i,i}$ 变成 $c_{j, i}$ 之后，让第 $j$ 行整体减去第 $i$ 行）。
3. 回代写出简化阶梯形矩阵。

当然一个很显然的事情是方程组是不一定有唯一解的。

首先如果消元的时候出现了 $0 = d$ 的这种方程，方程显然无解。

然后注意到第一步不一定能找到一个满足 $x_1 \sim x_{i - 1}$ 的系数都为 $0$ 的行，于是我们每次只需要找到一个 $x_i$ 的系数不为零的行交换到第 $i$ 行就行，如果方程有唯一解，这对解是没有影响的。

显然方程组的解不一定是整数，所以要考虑精度问题，假设我们当前要用第 $i$ 行消掉其它行 $x_i$ 的系数，设第 $i$ 行的 $x_i$ 的系数为 $c_{i, i} = r$。

那么对于 $\forall j \in [1,n], (j \not ={i})$，$c_{j, i}$ 就会变成 $0$，$c_{j, k}, (k \not ={i})$ 就会变成 $c_{j,k} -\dfrac{c_{j, i}}{r} \times c_{i, k}$。

我们希望这个式子计算的时候精度损失尽量小，也就是要让除法那里的精度损失更小，因为浮点数的绝对值越小精度越高，所以我们只需要让 $r$ 尽量大即可，那么就可以对精度有一个小优化：每次找到一个 $x_i$ 系数绝对值最大的行用来消去别的行的系数。

如果消元完了之后，如果 $\exists i, \exists c_{i, j} \not ={0}, j \not ={i}$ 的 $i$，那么证明我们找不到方程的唯一解，但是矩阵的形式会变成这种样子：

$$
\begin{bmatrix}
    1 & 3 & 0\ |\ 7 \\
    0 & 0 & 0\ |\ 0 \\
    0 & 0 & 6\ |\ 1
\end{bmatrix}
$$

例子中 $i = 1$ 就是不满足条件的行，方程的形式是这样的：

$$
\begin{cases}
    x_1 = 7 - 3x_2\\
    x_3 = \dfrac{1}{6}
\end{cases}
$$

可以发现，不管 $x_2$ 是什么实数，都有一个 $x_1$ 可以使得方程有解。

我们将 $x_2$ 这种变量称为自由元（自己那一行系数全为 $0$），$x_1,x_3$ 称为主元。

如果原方程组有 $k$ 行的系数全是 $0$，就证明原方程有 $k$ 个自由元，$n - k$ 个主元，原方程组有无数个解。

??? note "[模板：[SDOI2006]线性方程组](https://www.luogu.com.cn/problem/P2455)"

    ```cpp

    // author : black_trees

    #include <cmath>
    #include <cstdio>
    #include <cstring>
    #include <iomanip>
    #include <iostream>
    #include <algorithm>

    using namespace std;
    using i64 = long long;
    using ldb = long double;

    const int si = 50 + 10;
    const ldb eps = 1e-5;

    int n;
    ldb c[si][si], d[si], x[si];

    int Gauss() {
        for(int i = 1; i <= n; ++i) {
            int l = i;
            for(int j = i + 1; j <= n; ++j) 
                if(fabs(c[j][i]) > fabs(c[l][i]))
                    l = j;
            if(l != i) {
                for(int j = 1; j <= n; ++j) 
                    swap(c[i][j], c[l][j]);
                swap(d[i], d[l]);
            }
            if(fabs(c[i][i]) >= eps) {
                for(int j = 1; j <= n; ++j) {
                    if(j == i) continue;
                    ldb rte = c[j][i] / c[i][i];
                    for(int k = 1; k <= n; ++k)
                        c[j][k] -= rte * c[i][k];
                    d[j] -= rte * d[i];
                }
            }
        }
        bool nosol = false, infsol = false;
        for(int i = 1; i <= n; ++i) {
            int j = 1;
            while(fabs(c[i][j]) < eps && j <= n)
                j ++;
            j += (fabs(d[i]) < eps);
            if(j > n + 1) infsol = true;
            if(j == n + 1) nosol = true;	
        }
        if(nosol)  return 0;
        if(infsol) return 1;
        for(int i = n; i >= 1; --i) {
            for(int j = i + 1; j <= n; ++j)
                d[i] -= x[j] * c[i][j];
            x[i] = d[i] / c[i][i];
        }
        for(int i = 1; i <= n; ++i)
            cout << "x" << i << "=" << fixed << setprecision(2) << x[i] << endl;
        return 2;
    }

    int main() {

        cin.tie(0) -> sync_with_stdio(false);
        cin.exceptions(cin.failbit | cin.badbit);
        
        cin >> n;
        for(int i = 1; i <= n; ++i) {
            for(int j = 1; j <= n; ++j)
                cin >> c[i][j];
            cin >> d[i];
        }
        int ret = Gauss(); 
        if(ret == 2) return 0;
        if(ret == 0) cout << "-1" << endl; // 先判无解
        if(ret == 1) cout << "0" << endl;
        return 0;
    }
    ```

## 应用

### 异或方程组

Pigeon.

### 消除 dp 后效性

Pigeon.