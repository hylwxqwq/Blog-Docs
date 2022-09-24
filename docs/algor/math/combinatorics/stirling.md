
## 第二类斯特林数

定义：将 $n$ 个不同元素分到 $k$ 个完全相同的非空盒子里面的方案数为 $\begin{Bmatrix}n \\ k\end{Bmatrix}$。

递推式： $\begin{Bmatrix}n \\ k\end{Bmatrix} = \begin{Bmatrix}n - 1 \\ k - 1\end{Bmatrix} + k \begin{Bmatrix}n - 1 \\ k\end{Bmatrix}$。

证明：就是考虑新加入的这个球是放到原有的还是新建一个。

边界：$\begin{Bmatrix}n \\ 0\end{Bmatrix} = [n = 0]$。