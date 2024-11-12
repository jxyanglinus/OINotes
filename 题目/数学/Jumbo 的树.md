---
tags:
  - 考试
  - 数学
  - 期望
  - 概率
---
Link：[B. Jumbo的树 - 2024考前集训测试32 - 比赛 - 天立NEW](http://47.108.49.170:8000/contest/42/problem/2) 
## 题目描述
“Jumbo 树论好强啊，能不能教教我啊。” mengbier 跪求 Jumbo 赐教。

气场强大的 Jumbo 不慌不忙，从 Robbery 肚子里掏出一个题：

给你一棵根为 $1$ 的有根树，开始每个点都是黑色的，每轮操作会随机选一个黑点，然后把这个点到根路径上的所有点染白，问要把所有点全部染白期望需要几轮？

才疏学浅的 mengbier 看得目瞪口呆，然后 Jumbo 用中指扶了扶眼镜说：“你先一眼秒了它，再来找我吧！”

但 mengbier 能力有限，显然~~不~~能一眼秒，他十分绝望，~~都快要做出标题的表情了~~。

于是他来找你――一名乐于助人的  OIer， mengbier 相信你一定可以帮助他求得 Jumbo 的树论教学。
## 输入格式
第一行一个正整数 $n \: (n \le 10^7)$，表示从 Robbery 肚子里掏出来的题给你的树的点数。

第二行 $n - 1$ 个正整数，第 $i$ 个数 $prt_i$ 表示 $i + 1$ 号节点的父节点（保证父节点编号小于子节点编号）。
## 输出格式
输出一行一个正整数  $ans$，表示答案在 $\mod 998244353$ 意义下的值。
## 分析
求所有点被染色的期望轮数，可以求每一个点的期望，然后加起来。

根据染色规则，如果子树内的任意一点被染色，子树的根也就被染色了，所以，只有直接把子树的根染色才有贡献，而如果是因为子树中的点被染色而使得根被染色，那么根就不用再次染色，也就没有贡献。所以设节点 $u$ 的子树大小为 $size_u$，则这个点的贡献是 $\frac{1}{size_u}$。最后的答案就是 $\displaystyle\sum_{u = 1}^{n}\frac{1}{size_u}$。

此外，由于题目保证父节点编号小于子节点编号，所以可以从 $n$ 到 $1$ 遍历算出子树大小，而不用 DFS。（这么大的数据，DFS 容易爆栈）

最后就是，因为要取模，所以要求逆元。又因为数据很大，最好线性求逆元（时限够大的话用费马小定理也不是不行）。
## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

template<typename T> inline void read(T &x) {
	x = 0; signed op = 1; char ch = getchar();
	for (; !isdigit(ch); ch = getchar()) if (ch == '-') op = -1;
	for (; isdigit(ch); ch = getchar()) x = x * 10 + ch - '0';
	x *= op;
}

template<typename T> void write(T x) {
	if (x < 0) putchar('-'), x = -x;
	if (x > 9) write(x / 10);
	putchar(x % 10 + '0');
}

typedef long long ll;
const ll maxn = 1e7 + 5;
const ll mod = 998244353;
ll n;
ll father[maxn], size[maxn];
ll inv[maxn];

int main() {
	freopen("jumbo.in", "r", stdin);
	freopen("jumbo.out", "w", stdout);
	read(n);
	size[1] = 1;
	for (ll i = 2; i <= n; i++) {
		ll pre;
		read(pre);
		father[i] = pre;
		size[i] = 1;
	}
	for (ll i = n; i > 1; i--) {
		size[father[i]] += size[i];
	}
	inv[0] = inv[1] = 1;
	for (ll i = 2; i <= n; i++) {
		inv[i] = mod - mod / i * inv[mod % i] % mod;
	}
	ll ans = 0;
	for (ll i = 1; i <= n; i++) {
		ans = (ans + inv[size[i]]) % mod;
	}
	write(ans);
	return 0;
}
```
