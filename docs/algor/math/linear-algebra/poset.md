## 非严格偏序，自反偏序

设 $\preccurlyeq$ 是集合 $S$ 上的二元关系，如果 $\preccurlyeq$ 满足：

1. 自反性：$\forall a \in S,$ 有 $a \preccurlyeq a$
2. 反对称性：$\forall a,b \in S,a\preccurlyeq b \land b \preccurlyeq a,$  则 $a=b$。
3. 传递性： $\forall a,b,c \in S, a \preccurlyeq b \land b \preccurlyeq c,$ 则 $a \preccurlyeq c$

则称 $\preccurlyeq$ 是 $S$ 上的非严格偏序或自反偏序。

类似图论里的自环和无向边。

## 严格偏序，反自反偏序

设 $\prec$ 是集合 $S$ 上的二元关系，如果 $\prec$ 满足：

1. 反自反性：$\forall a \in S,$ 有 $a \not\prec a$
2. 非对称性：$\forall a,b \in S,a\prec b \Rightarrow b \not\prec a,$。
3. 传递性： $\forall a,b,c \in S, a \prec b \land b \prec c,$ 则 $a \prec c$

则称 $\prec$ 是 $S$ 上的严格偏序或反自反偏序。

类似图论里的有向边。

一个集合上的严格偏序关系图实际上就是一个 DAG（有向无环）。

并且这个图的传递闭包是它自己。

所以遇到严格偏序的判定（是否成立）时，可以利用拓扑排序解决。

如果排序完了之后，仍有 $deg \not= 0$ 的点，则这个严格偏序关系不成立。

如果成立，要求构造方案时，一般需要用到拓扑序。