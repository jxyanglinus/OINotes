# Software

Link: [D. Software - 2024考前集训测试18](http://47.108.49.170:8000/contest/28/problem/4)，[P1800 software - 洛谷](https://www.luogu.com.cn/problem/P1800)

## 题目描述

一个软件开发公司同时要开发两个软件，并且要同时交付给用户，现在公司为了尽快完成这一任务，将每个软件划分成 $m$ 个模块，由公司里的技术人员分工完成，每个技术人员完成同一软件的不同模块的所用的天数是相同的，并且是已知的，但完成不同软件的一个模块的时间是不同的，每个技术人员在同一时刻只能做一个模块，一个模块只能由一个人独立完成而不能由多人协同完成。一个技术人员在整个开发期内完成一个模块以后可以接着做任一软件的任一模块。

写一个程序，求出公司最早能在什么时候交付软件。

## 分析

一开始想贪心，但是贪心保证不了正确性。这题是 DP。

然后再看题目，交付的时间是两个软件中开发周期长的那个，我们要求这个长的时间最短——这不是二分答案吗？我们二分枚举时间，然后用 DP 检验这个日期是否合法。

首先假设 $f[i][j]$ 表示到第 $i$ 个技术人员的时候，第一个软件已经完成了 $j$ 个模块，第二个软件可以完成的模块数量，则：
$$
f[i][j] = \style\nolimit\max_{0 \le k \le min(\frac{day}{d1[i]}, j)}\{f[i - 1][j - k] + (\frac{day - d1[i] \times k}{d2[i]})\}
$$

其中 $day$ 表示二分查找枚举的交付日期。

DP 完之后，若 $f[m] \ge m$，即程序一做完 $m$ 个模块的时候，程序二也至少能完成任务。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 105;
int n, m;
int f[maxn];
int dev1[maxn], dev2[maxn];

bool check(int day) {
    f[0] = 0;
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

