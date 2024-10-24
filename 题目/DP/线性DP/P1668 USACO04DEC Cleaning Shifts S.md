---
tags:
  - 动态规划
  - 线性DP
  - 线段树
  - 区间DP
---
[P1668 USACO04DEC Cleaning Shifts S - 洛谷](https://www.luogu.com.cn/problem/P1668) 
## 分析
这道题最快的做法应该是贪心，但是线段树优化 DP 也可以做。

首先看到此题，可以想到一个很暴力的区间 DP：$f[i][j]$ 表示在 $[i, j]$ 时段内最少需要的奶牛数量。对于每头牛的空闲时段 $[l, r]$，其每个子区间答案均为 $1$；对于其他的部分，像[[P1880 NOI1995 石子合并]] 一样转移就行了。时间效率 $O(t^3)$，可以拿 $35$ 分。

优化一下，把区间 DP 优化为线性 DP，令 $f[i]$ 表示覆盖时段 $[1, i]$ 所需要的最少奶牛数量。首先显然覆盖一个区间至少需要一头奶牛。如果一头奶牛的覆盖区间为 $[l, r]$，其中 $l = 1$，那么 $f[r] = 1$，这是 $f[r]$ 的最优解。而对于 $l \not= 1$ 的情况，则可以和前面的状态合并：$f[r] = \min_{l - 1 \le i < r}\{f[i] + 1\}$。为保证转移的顺序，应该先按照右端点排序。现在时间效率 $O(nt)$，仍然不够。

继续优化，注意到上面的方程需要找到 $[l - 1, r)$ 的最小状态。~~Look familiar?~~ 我们考虑使用**线段树优化**，用一棵维护最小值的线段树来存状态。再来想想这棵线段树需要哪些功能：根据上面的方程可知需要区间查询，然后针对每个状态的更新，只需要单点修改即可。由于引入了线段树，时间效率优化为了 $O(n \log t)$，可通过本题。

## 代码
暴力区间 DP
```cpp
#include <bits/stdc++.h>  
using namespace std;  
  
const int maxn = 25005;  
const int maxt = 8005;  
int n, t;  
int f[maxt][maxt];  
  
int main() {  
    scanf("%d%d", &n, &t);  
    memset(f, 0x3f, sizeof(f));  
    for (int i = 1; i <= n; i++) {  
        int l, r;  
        scanf("%d%d", &l, &r);  
        for (int j = l; j <= r; j++) {  
            for (int k = j; k <= r; k++) {  
                f[j][k] = 1;  
            }  
        }  
    }  
    for (int len = 1; len <= t; len++) {  
        for (int i = 1, j; (j = i + len - 1) <= t; i++) {  
            for (int k = i; k < j; k++) {  
                f[i][j] = min(f[i][j], f[i][k] + f[k + 1][j]);  
            }  
        }  
    }  
    printf("%d", f[1][t] == 0x3f3f3f3f ? -1 : f[1][t]);  
    return 0;  
}
```
未优化的线性 DP
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 2.5e4 + 5;
const int maxt = 1e6 + 5;
int n, t;
int f[maxt];
struct Node {
	int l, r;
	bool operator < (const Node &rhs) const {
		return r < rhs.r;
	}
} node[maxn];

int main() {
	scanf("%d%d", &n, &t);
	memset(f, 0x3f, sizeof(f));
	for (int i = 1; i <= n; i++) {
		scanf("%d%d", &node[i].l, &node[i].r);
		if (node[i].l == 1) f[node[i].r] = 1;
	}
	sort(node + 1, node + 1 + n);
	for (int i = 1; i <= n; i++) {
		for (int j = node[i].l - 1; j < node[i].r; j++) {
			f[node[i].r] = min(f[node[i].r], f[j] + 1);
		}
	}
	printf("%d", f[t] == 0x3f3f3f3f ? -1 : f[t]);
	return 0;
}
```
线段树优化 DP
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 2.5e4 + 5;
const int maxt = 1e6 + 5;
int n, t;
struct SegmentTree {
	int left, right;
	int minv;
} tree[maxt << 2];
struct Node {
	int l, r;
	bool operator < (const Node &rhs) const {
		return r < rhs.r;
	}
} node[maxn];

inline void pushup(int root) {
	tree[root].minv = min(tree[root << 1].minv, tree[root << 1 | 1].minv);
}

void build(int root, int l, int r) {
	tree[root].left = l;
	tree[root].right = r;
	if (l == r) {
		tree[root].minv = 0x3f3f3f3f;
		return;
	}
	int mid = (l + r) >> 1;
	build(root << 1, l, mid);
	build(root << 1 | 1, mid + 1, r);
	pushup(root);
}

void update(int root, int pos, int val) {
	if (tree[root].left == tree[root].right) {
		tree[root].minv = val;
		return;
	}
	int mid = (tree[root].left + tree[root].right) >> 1;
	if (mid >= pos) update(root << 1, pos, val);
	else update(root << 1 | 1, pos, val);
	pushup(root);
}

int query(int root, int l, int r) {
	if (tree[root].left >= l && tree[root].right <= r) {
		return tree[root].minv;
	}
	int mid = (tree[root].left + tree[root].right) >> 1;
	int ans = 0x3f3f3f3f;
	if (mid >= l) ans = min(ans, query(root << 1, l, r));
	if (mid < r) ans = min(ans, query(root << 1 | 1, l, r));
	return ans;
}

int main() {
	scanf("%d%d", &n, &t);
	build(1, 1, t);
	for (int i = 1; i <= n; i++) {
		scanf("%d%d", &node[i].l, &node[i].r);
		if (node[i].l == 1) update(1, node[i].r, 1);
	}
	sort(node + 1, node + 1 + n);
	for (int i = 1; i <= n; i++) {
		update(1, node[i].r, min(query(1, node[i].r, node[i].r), query(1, node[i].l - 1, node[i].r - 1) + 1));
	}
	printf("%d", query(1, t, t) == 0x3f3f3f3f ? -1 : query(1, t, t));
	return 0;
}
```
