[Zuma - 洛谷](https://www.luogu.com.cn/problem/CF607B)
## Analysis
题意：每一步在当前序列中一个回文的、连续的子串，并将这个子串移除，求最小步数。
不难想到使用区间DP，设 $f[i][j]$ 表示 $[i,j]$ 中的最小步数。
则可得临界状态：
$$
f[i][i] = 1​
$$
$$
f[i][i + 1] = 1,color[i]=color[i+1]
$$
$$
f[i][i + 1] = 2, color[i] \neq color[i+1]
$$
转移方程：
$$
f[i][j] = f[i + 1][j - 1], color[i] = color[j]
$$
$$
f[i][j] = \max_{k = i}^{j - 1}f[i][k] + f[k + 1][j]
$$
**值得注意的是，在以上两个方程中，即使满足第一个方程的条件，仍然需要执行第二个操作，因为 $f[i + 1][j + 1]$ 不一定比 $f[i][k] + f[k + 1][j]$ 小。**
## Code
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 505;
int n;
int arr[maxn];
int f[maxn][maxn];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &arr[i]);
    }
    memset(f, 0x3f, sizeof(f));
    for (int i = 1; i <= n; i++) {
        f[i][i] = 1;
    }
    for (int i = 1; i < n; i++) {
        if (arr[i] == arr[i + 1]) f[i][i + 1] = 1;
        else f[i][i + 1] = 2;
    }
    for (int len = 3; len <= n; len++) {
        for (int i = 1, j; (j = i + len - 1) <= n; i++) {
            if (arr[i] == arr[j]) f[i][j] = min(f[i][j], f[i + 1][j - 1]);
            for (int k = i; k < j; k++) {
                f[i][j] = min(f[i][j], f[i][k] + f[k + 1][j]);
            }
        }
    }
    printf("%d", f[1][n]);
    return 0;
}
```
