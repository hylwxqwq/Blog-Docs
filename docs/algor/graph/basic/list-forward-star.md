## 关于链式前向星

我的写法很奇怪，巨 TM 奇怪：

```cpp
int tot=0;
struct Edge{ int head,ver,Next,w; }e[si_m<<1];
inline void add(int u,int v,int w){ e[++tot].ver=v,e[tot].Next=e[u].head,e[u].head=tot,e[tot].w=w; }
// 边的编号从 1 开始

// in main or other function:
for(register int i=e[u].head;i;i=e[i].Next){
	//...
}
// clear
for(register int i=0;i<=tot;++i) e[i].head=0; tot=0;
```

正常人一般会把 head 提出来方便多测的清空。

但是我会写 `for(register int i=0;i<=tot;++i) e[i].head=0;`

这样子是没问题的，但我之前犯过一个错误，我 `for` 清空的时候写的是 `e[i].head=-1`。

但是遍历的时候是 `i=...;i;i=...` 这种。

然后就挂了，调了一个下午，用 GDB 发现前向星的写法臭了（两三年了都没出事，也没发现…… /ch

用 `e[i].head=-1` 清空的话这么写：

```cpp
int tot=0;
int head[si_n];
struct Edge{ int ver,Next,w; }e[si_m<<1];
inline void add(int u,int v,int w){  e[tot]=(Edge){v,head[u],w},head[u]=tot++; }
// 边的编号从 0 开始

// 
for(register int i=head[u];~i;i=e[i].Next){
	//...
}
// clear
memset(head,-1,sizeof head); tot=0; // 不管有没有多测都要memset -1.
```

网络流的时候要成对变换，正常人应该会像上面这样子写。

如果非要用第一种，就初始化 `tot=1`，从 2 开始给边编号。