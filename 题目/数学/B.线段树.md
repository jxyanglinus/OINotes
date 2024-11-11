---
tags:
  - 线段树
  - 数学
  - 思维题
  - 记忆化
  - 考试
---
Link：[B. 线段树 - 2024考前集训测试30 - 比赛 - 天立NEW](http://47.108.49.170:8000/contest/40/problem/2)
## 分析
好题！

首先最基础的 $20$ 分，直接对于每一个询问，直接跑线段树建树流程，效率 $O(Tn)$。然而，本题 $1 \le T \le 10^4, 1 \le x \le y \le n \le 10^{18}$，显然需要找规律。

首先容易想到一个性质，就是对于长度一样的区间，线段树的节点数和构型是一样的，只是节点编号不同。

线段树是一棵完全二叉树，假设根节点编号为 $x$，则其左儿子编号为 $2x$，右儿子编号为 $2x+1$，依次类推，发现以 $x$ 为根的子树的节点编号均为一个关于 $x$ 的一次函数。显然以 $x$ 为根的子树的节点编号和也是一个关于 $x$ 的一次函数。

对于区间 $[l, r]$，令 $len \gets r - l + 1$，设 $f(len, x) = k_{len}x + b_{len}$，则可以得到：$f(len, x)=f(\left\lceil \frac{len}{2} \right\rceil, 2x) + f(\left\lfloor \frac{len}{2} \right\rfloor, 2x + 1) + x$。（之所以左子树向上取整，右子树向下取整，是因为线段树建树的流程里，左子树部分的长度就是向上取整了的，式子摆在这里：$mid = (l + r) >> 1$，左子树 $[l, mid]$，右子树 $[mid + 1, r]$）。
进一步又可以推出：
$$
\begin{aligned}
k_{len} &= 2k_{\left\lceil \frac{len}{2} \right\rceil} + 2k_{\left\lfloor \frac{len}{2} \right\rfloor + 1} \\
b_{len} &= b_{\left\lceil \frac{len}{2} \right\rceil} + b_{\left\lfloor \frac{len}{2} \right\rfloor} + k_{\left\lfloor \frac{len}{2} \right\rfloor}
\end{aligned}
$$
结合前面的性质，我们可以通过分治处理出各个长度的 $k, b$，并记忆化存储下来。
## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int mod = 1e9 + 7;
unordered_map<ll, ll> k, b;

void dfs(ll len) {
	if (k[len]) return;
	int lpart = (len + 1) >> 1, rpart = len >> 1; // 向上取整，向下取整
	dfs(lpart), dfs(rpart); // 左子树和右子树
	k[len] = (2 * k[lpart] + 2 * k[rpart] + 1) % mod; // 公式，前面推的
	b[len] = (b[lpart] + b[rpart] + k[rpart]) % mod;
}

ll query(ll root, ll L, ll R, ll l, ll r) {
	if (l >= L && r <= R) {
		ll len = r - l + 1;
		dfs(len);
		return (k[len] * root % mod + b[len]) % mod;
	}
	ll mid = (l + r) >> 1;
	ll ans = 0;
	if (mid >= L) (ans += query((root << 1) % mod, L, R, l, mid)) %= mod;
	if (mid < R) (ans += query((root << 1 | 1) % mod, L, R, mid + 1, r)) %= mod;
	return ans;
}

int main() {
	freopen("segt.in", "r", stdin);
	freopen("segt.out", "w", stdout);
	int T;
	scanf("%d", &T);
	k[1] = 1, b[1] = 0; // 注意初始化，长度为 1 时的 k 和 b。
	while (T--) {
		ll n, x, y;
		scanf("%lld%lld%lld", &n, &x, &y);
		printf("%lld\n", query(1, x, y, 1, n));
	}
	return 0;
}
```
