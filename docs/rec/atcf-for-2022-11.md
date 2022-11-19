
## 十一月 CF AT 丢人做题记录

### ABC277E

> 给你一张简单无向图，每个边 $(u_i, v_i)$ 有一个属性 $a_i$。
>
> 其中如果 $a_i$ 为 $1$ 表示这条边是可以通行的，否则是不可通行的。
>
> 有 $K$ 个特殊点，如果你处于这个特殊点，你可以进行一次转换，使得所有的 $a_i$ 取反。
>
> 问你从 $1 \to n$ 的最短路（无解则答案为 `-1`）。
>
> $2\le n \le 2e5, 1\le m \le 1e5, 0\le K \le n$。

呃呃，感觉是比较板子的分层图最短路。

就是你看到有一些边要在特殊情况下才能通行或者这些边的边权会改变之类的时候，就考虑把边分类。

并且拆点，把图分成多层，如果遇到可以进行不同层转移的节点（比如本题中的特殊点），就在不同层之间连边（这条边的代价取决于在层之间转移的代价）。

之后问题就可以转化成普通的最短路问题了。

对应到这道题上就是考虑把 0/1 状态的边分开连，同层边的边权设置为 $1$，因为我们转换一次不耗费任何代价，所以不同层之间的边的边权是 $0$。

类似这样：

[![znFhE6.png](https://s1.ax1x.com/2022/11/18/znFhE6.png)](https://imgse.com/i/znFhE6)

然后连完边直接求最短路即可。

```cpp
// author : black_trees

#include <cmath>
#include <queue>
#include <cstdio>
#include <utility>
#include <cstring>
#include <iostream>
#include <algorithm>

#define endl '\n'

using namespace std;
using i64 = long long;

const int si = 2e5 + 10;

int n, m;
int tot = 0, head[si << 1];
struct Edge { int ver, Next, w; } e[si << 2];
inline void add(int u, int v, int w) { e[tot] = (Edge){v, head[u], w}, head[u] = tot++; }

std::priority_queue<std::pair<int, int> >q;
bool vis[si << 1]; int dis[si << 1];
void dijkstra(int s) {
	memset(vis, false, sizeof vis), memset(dis, 0x3f, sizeof dis);
	dis[s] = 0,q.push({dis[s], s});
	while(!q.empty()) {
		int u = q.top().second; q.pop();
		if(vis[u]) continue; vis[u] = true;
		for(int i = head[u]; ~i; i = e[i].Next){
			int v = e[i].ver, w = e[i].w;
			if(dis[v] > dis[u] + w) 
				dis[v] = dis[u] + w, q.push({-dis[v], v});
		}
	}
}

int main() {

	cin.tie(0) -> sync_with_stdio(false);
	cin.exceptions(cin.failbit | cin.badbit);

	memset(head, -1, sizeof head);

	int K;
	cin >> n >> m >> K;
	for(int i = 1; i <= m; ++i) {
		int u, v, w;
		cin >> u >> v >> w;
		if(w == 0) add(u + n, v + n, 1), add(v + n, u + n, 1);
		if(w == 1) add(u, v, 1), add(v, u, 1);
	}
	for(int i = 1; i <= K; ++i) {
		int gt; cin >> gt;
		add(gt, gt + n, 0), add(gt + n, gt, 0);
	}
	dijkstra(1);

	int ans = min(dis[n], dis[n + n]);
	if(ans < 0x3f3f3f3f) cout << ans << endl;
	else cout << "-1" << endl;

	return 0;
}
```
