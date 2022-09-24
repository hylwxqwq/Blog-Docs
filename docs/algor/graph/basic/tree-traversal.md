
## 深度优先遍历

### 对于二叉树

#### 前序遍历

先根后左右。

```cpp
void dfs(int u) {
	if(u == -1) return;
	order[++cnt] = u;
	dfs(u.lson), dfs(u.rson);
}
```

#### 中序遍历

先左，后根，再右。

```cpp
void dfs(int u) {
	if(u == -1) return;
	dfs(u.lson);
	order[++cnt] = u;
	dfs(u.rson);
}
```

#### 后序遍历

先左右，后根。

```cpp
void dfs(int u) {
	if(u == -1);
	dfs(u.lson), dfs(u.rson);
	order[++cnt] = u;
}
```

### 性质

1. 前序的第一个是 root，后序的最后一个是 root。
2. 先确定根节点，然后根据中序遍历，在根左边的为左子树，根右边的为右子树。
3. 对于每一个子树可以看成一个全新的树，仍然遵循上面的规律。

### 对于多叉树

就是分别递归每个儿子，不撞南墙不回头，类似二叉树的前序遍历。

实际顺序由加边顺序决定。

```cpp
void dfs(int u, int fa) {
	order[++cnt] = u;
	for(int i = head[u]; ~i; i = e[i].Next) {
		int v = e[i].ver;
		if(v == fa) continue;
		dfs(v, u);
	}
}
```

## 广度优先遍历

一层一层遍历，用队列。

```cpp
void bfs() {
	q.push(root);
	while(!q.empty()) {
		int u = q.front();
		q.pop();
		order[++cnt] = u, vis[u] = true;
		for(int i = head[u]; ~i; i = e[u].Next) {
			int v = e[i].ver;
			if(!vis[v]) q.push(v);
		}
	}
}
```

## 关于树的一点点基础知识

树是一个无向连通图，$n$ 个节点 $n - 1$ 条边，不存在回路。

满二叉树：每一层节点个数都满的二叉树（第 $i$ 层有 $2^{i - 1}$ 个节点）

完全二叉树：满二叉树的子集，$n$ 个节点的完全二叉树深度为 $\lfloor\log_2n\rfloor + 1$。

对任意一颗二叉树，度为0的结点（即叶子结点）总是比度为2的结点多一个