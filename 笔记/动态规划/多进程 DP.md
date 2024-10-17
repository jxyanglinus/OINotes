~~我好像还没怎么见过这样的题。~~ 不过，做任何 DP，深刻充分的理解永远是通用的题解。

# 例子
有两个对象，共同完成一批任务，其中一个对象做任务时，另一个对象可以在同时完成别的任务；对于每个任务，两对象完成的时间不同，求最快什么时候完成。

## 看看例题
1. [P2224 HNOI2001 产品加工 - 洛谷](https://www.luogu.com.cn/problem/P2224) 
## 分析
**令 $f[i][j]$ 表示做到第 $i$ 件物品，机器 $A$ 耗时为 $j$ 时，$B$ 机器所耗费的最短时间。** 设 $t[i][0/1/2]$ 表示第 $i$ 件物品由 $A$，$B$，或 $A, B$ 共同加工的时间，则：
$$
\begin{aligned}
f[i][j] = min\{f[i - 1][j - t[i][0]], f[i - 1][j] + t[i][1], f[i - 1][j - t[i][2]] + t[i][2]\}\\
\end{aligned}
$$
$f[i - 1][j - t[i][0]]$ 表示这个任务由 $A$ 完成。

$f[i - 1][j] + t[i][1]$ 表示这个任务由 $B$ 完成。

$f[i - 1][j - t[i][2]] + t[i][2]$ 表示这个任务由 $A, B$ 共同完成。

初始化：$f[0][0] = 0$，其他均设为正无穷。

本题数据范围比较大，二维数组空间过大，所以要用滚动数组滚掉一维。（滚动数组注意当前状态的含义与合法性）
## 代码
```cpp
#include <bits/stdc++.h>  
using namespace std;  
  
const int maxn = 6e3 + 5;  
int f[maxn * 5];  
int n;  
int cost[maxn][3];  
int up;  
  
int main() {  
    scanf("%d", &n);  
    for (int i = 1; i <= n; i++) {  
        for (int j = 0; j < 3; j++) {  
            scanf("%d", &cost[i][j]);  
        }  
        up += max(cost[i][0], max(cost[i][1], cost[i][2])); // 记录完成时间的上界  
    }  
    memset(f, 0x3f, sizeof(f));  
    f[0] = 0;  
    // f[i][j] = min{ f[i - 1][j - cost[i][0]], f[i - 1][j] + cost[i][1], f[i - 1][j - cost[i][2]] + cost[i][2] }  
    for (int i = 1; i <= n; i++) {  
        for (int j = up; j >= 0; j--) {  
            int temp = f[j]; // 要注意的地方：f[j] 发生了变化，所以要暂存下来参与决策。              f[j] = 0x3f3f3f3f; //相当于每次初始化  
            if (cost[i][0] && j >= cost[i][0]) {  
                f[j] = min(f[j], f[j - cost[i][0]]);  
            }  
            if (cost[i][1]) {  
                f[j] = min(f[j], temp + cost[i][1]); // here is temp.  
            }  
            if (cost[i][2] && j >= cost[i][2]) {  
                f[j] = min(f[j], f[j - cost[i][2]] + cost[i][2]);  
            }  
        }  
    }  
    int ans = 0x3f3f3f3f;  
    for (int i = 0; i <= up; i++) {  
        ans = min(ans, max(f[i], i));  // 完成时间为 A, B 中较大的那个
    }  
    printf("%d", ans);  
    return 0;  
}
```

2. [P1800 software - 洛谷](https://www.luogu.com.cn/problem/P1800)
## 分析
和上一道题比较类似，但又不太一样。

首先可以发现，交付时间为两个软件完成时间较长的那个，而我们要求这个较长的时间最短——二分答案！

然后我们就二分交付时间，然后用 DP 检验这个时间是否合法。

**设 $f[i][j]$ 表示到第 $i$ 个技术人员的时候，第一个软件已经完成了 $j$ 个模块，第二个软件可以完成的模块数量**，则：
$$
\begin{aligned}
f[i][j] = \style\nolimit\max_{0 \le k \le min(\frac{day}{d1[i]}, j)}\{f[i - 1][j - k] + (\frac{day - d1[i] \times k}{d2[i]})\}
\end{aligned}
$$

其中 $day$ 表示二分查找枚举的交付日期。

DP 完之后，若 $f[m] \ge m$，则表示程序一做完 $m$ 个模块的时候，程序二也至少能完成任务。

## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 105;
int n, m;
int f[maxn];
int dev1[maxn], dev2[maxn];

bool check(int day) {
    f[0] = 0; // 滚动数组压维了
    for (int i = 1; i <= m; i++) {
        f[i] = -0x3f3f3f3f;
    }
    for (int i = 1; i <= n; i++) {
        for (int j = m; j >= 0; j--) {
            for (int k = 0; k <= min(day / dev1[i], j); k++) {
                f[j] = max(f[j], f[j - k] + (day - k * dev1[i]) / dev2[i]);
            }
        }
    }
    return f[m] >= m;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d%d", &dev1[i], &dev2[i]);
    }
    int l = 0, r = 114514;
    while (l < r) {
        int mid = (l + r) >> 1;
        if (check(mid)) r = mid;
        else l = mid + 1;
    }
    printf("%d", l);
    return 0;
}
```
