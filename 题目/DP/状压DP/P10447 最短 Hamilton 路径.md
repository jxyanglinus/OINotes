---
tags:
  - 动态规划
  - 状压DP
  - 常数优化
---
[P10447 最短 Hamilton 路径 - 洛谷](https://www.luogu.com.cn/problem/P10447)
## Analysis
Hamilton 路径的定义是从 $0$ 到 $n-1$ 不重不漏地经过每个点恰好一次。
这是一个经典の[旅行商问题](https://baike.baidu.com/item/%E6%97%85%E8%A1%8C%E5%95%86%E9%97%AE%E9%A2%98/7737042)
最容易直接想到的方法就是全排列，然后算每种情况的路径长度。但是这种方法的时间效率为 $O(n \times n!)$，肯定 TLE ~~评测机给你测飞起来~~。
再看数据范围，$n \le 20$，考虑状压 DP，指数级复杂度，能过。
先考虑设 $f[i]$ 表示状态 $i$ 下的最短路径。然而这是不够的，因为我们不仅要经过所有的点，而且要从 $0$ 开始，到 $n - 1$ 结束。
所以再添加一维，设 $f[i][j]$ 表示在 $i$ 状态下最后经过的点是 $j$ 的最短路径。
这要怎么转移呢？
$$
f[i][j] = \min_{k \in i, j \in i}\{f[i \oplus (1 << j)][k] + dist[k][j]\}
$$
代码如下：
```cpp
f[1][0] = 0; // 令只经过点 0 的状态的最短距离为 0
// 枚举状态
for (int i = 2; i < (1 << n); i++) {
    for (int j = 0; j < n; j++) {
        // 枚举每个点
        if (!(i & (1 << j))) continue; // 如果当前状态没有经过 j 这个点，就不继续计算了。
        for (int k = 0; k < n; k++) {
            if (!(i & (1 << k))) continue; // 同上
            // min 函数里的第二项：没有经过点 j 且最后一个点为点 k 的最短路径 + 点 k 到点 j 的距离
            f[i][j] = min(f[i][j], f[i ^ (1 << j)][k] + dist[k][j]);
        }
    }
}
```
另外还有一项常数优化：我们要求的路径，是从 $0$ 开始的，因此，无论怎么走，第一个点永远是 $0$，路径上一定会经过 $0$，所以我们可以只在状态 $i$ 为奇数时计算（这表示当前状态经过了 $0$）。这样一来，我们 DP 的计算量就直接减少了一半。
```cpp
f[1][0] = 0;
// 枚举状态
for (int i = 3; i < (1 << n); i += 2) {
    for (int j = 0; j < n; j++) {
        // 枚举每个点
        if (!(i & (1 << j))) continue; // 如果当前状态没有经过 j 这个点，就不继续计算了。
        for (int k = 0; k < n; k++) {
            if (!(i & (1 << k))) continue; // 同上
            // min 函数里的第二项：没有经过点 j 且最后一个点为点 k 的最短路径 + 点 k 到点 j 的距离
            f[i][j] = min(f[i][j], f[i ^ (1 << j)][k] + dist[k][j]);
        }
    }
}
```
## Code
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 20;
int n;
int dist[maxn][maxn];
int f[1 << maxn][maxn];

int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            scanf("%d", &dist[i][j]);
        }
    }
    for (int i = 0; i < (1 << n); i++) {
        for (int j = 0; j < n; j++) {
            f[i][j] = 0x3f3f3f3f;
        }
    }
    f[1][0] = 0;
    // 枚举状态
    for (int i = 3; i < (1 << n); i += 2) {
        for (int j = 0; j < n; j++) {
            // 枚举每个点
            if (!(i & (1 << j))) continue; // 如果当前状态没有经过 j 这个点，就不继续计算了。
            for (int k = 0; k < n; k++) {
                if (!(i & (1 << k))) continue; // 同上
                // min 函数里的第二项：没有经过点 j 且最后一个点为点 k 的最短路径 + 点 k 到点 j 的距离
                f[i][j] = min(f[i][j], f[i ^ (1 << j)][k] + dist[k][j]);
            }
        }
    }
    printf("%d", f[(1 << n) - 1][n - 1]);
    return 0;
}
```
