[Peach Blossom Spring - 洛谷](https://www.luogu.com.cn/problem/UVA1496) 
[PDF](https://onlinejudge.org/external/14/p1496.pdf) 
## 分析
从[管道连接](https://www.luogu.com.cn/problem/P3264)推荐过来的，基本相当于双倍经验。

看 $k$ 范围特小，考虑最小斯坦纳树。根据题意，关键点分为前 $k$ 个和 后 $k$ 个两组。

定义 $g[S]$ 表示所选关键点点集为 $S$ 的时候所构成的斯坦纳树森林的最小边权和，发现 $g[S]$​ 合法的条件是 $S$ 在二进制表示中的前 $k$ 位里 $1$ 的个数和后 $k$ 位 $1$ 的个数相等（对应题目的实际意义就是 $k$ 个人都可以找到对应的房子）。

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 55, maxm = 1005, maxk = 10;
int n, m, k;
int f[maxn][1 << maxk], g[1 << maxk];
int head[maxn];
struct Edge {
	int to, weight, next;
} edge[maxm << 1];
struct Node {
	int id, dis;
	bool operator < (const Node &rhs) const {
		return dis > rhs.dis;
	}
};

int edgecnt;
inline void insertEdge(int u, int v, int w) {
	edge[++edgecnt].to = v;
	edge[edgecnt].weight = w;
	edge[edgecnt].next = head[u];
	head[u] = edgecnt;
}

bool visited[maxn];
inline void dijkstra(int S) {
	memset(visited, 0, sizeof(visited));
	priority_queue<Node> que;
	for (int i = 1; i <= n; i++) {
		if (f[i][S] < 0x3f3f3f3f) {
			que.push(Node{ i, f[i][S] });
		}
	}
	while (!que.empty()) {
		int u = que.top().id;
		que.pop();
		if (visited[u]) continue;
		visited[u] = true;
		for (int i = head[u]; i; i = edge[i].next) {
			int v = edge[i].to, w = edge[i].weight;
			if (f[v][S] > f[u][S] + w) {
				f[v][S] = f[u][S] + w;
				que.push(Node{ v, f[v][S] });
			}
		}
	}
}

inline int Popcount(int x) {
	int res = 0; // 统计 x 二进制下 1 的个数。（本来想按照之前某一套初赛题里面的写法来写，但是我写不来-_-）
	while (x) {
		if (x & 1) res++;
		x >>= 1;
	}
	return res;
}

int main() {
	int T;
	scanf("%d", &T);
	while (T--) {
		memset(head, 0, sizeof(head));
		edgecnt = 0; // 多测一定一定一定要清空。
		scanf("%d%d%d", &n, &m, &k);
		for (int i = 1; i <= m; i++) {
			int u, v, w;
			scanf("%d%d%d", &u, &v, &w);
			insertEdge(u, v, w);
			insertEdge(v, u, w);
		}
		memset(f, 0x3f, sizeof(f));
		memset(g, 0x3f, sizeof(g));
		for (int i = 1; i <= k; i++) {
			f[i][1 << (i - 1)] = 0; // 前 k 个关键点
		}
		for (int i = n - k + 1; i <= n; i++) {
			f[i][1 << (2 * k + i - n - 1)] = 0; // 后 k 个关键点
		}
		for (int i = 1; i <= n; i++) {
			f[i][0] = 0;
		}
		// 最小斯坦纳树正常操作
		for (int S = 1; S < (1 << (2 * k)); S++) {
			for (int i = 1; i <= n; i++) {
				for (int j = S & (S - 1); j; j = S & (j - 1)) {
					f[i][S] = min(f[i][S], f[i][S ^ j] + f[i][j]);
				}
			}
			dijkstra(S);
		}
		for (int S = 1; S < (1 << (2 * k)); S++) {
			// 前 k 位里 1 的个数与后 k 位里的 1 的个数相等时才可以转移。
			if (Popcount(S & ((1 << k) - 1)) == Popcount(S >> k)) {
				for (int i = 1; i <= n; i++) {
					g[S] = min(g[S], f[i][S]);
				}
			}
		}
		// 转移，合并子集
		for (int S = 1; S < (1 << (2 * k)); S++) {
			for (int i = S & (S - 1); i; i = S & (i - 1)) {
				g[S] = min(g[S], g[S ^ i] + g[i]);
			}
		}
		if (g[(1 << (2 * k)) - 1] == 0x3f3f3f3f) printf("No solution\n");
		else printf("%d\n", g[(1 << (2 * k)) - 1]);
	}
	return 0;
}
```
