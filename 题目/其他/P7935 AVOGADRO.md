---
tags:
  - 队列
  - 拓扑
---
[P7935 AVOGADRO - 洛谷](https://www.luogu.com.cn/problem/P7935)
## Analysis
~~我好菜，黄题都不会做~~
题目给出了一个排列和两个序列。在排列中，元素不会重复，这点很重要。
删除后的三行在升序排序后完全相同，即三行所保留的数字相同。只要我们寻找出两个序列里面不存在的数，然后在排列里面删除，**同时，删除掉的那一列上的数字已经不能使用了，那么利用它们又可以继续删除别的数。**（考虑使用队列实现）

正确性。。。好证！考虑当前的三个序列必然数量相同，在数量相同的前提下，有一个是排列，那么如果另外两个序列有了相同的数，必然它没有另外一个数。那么这样删下去肯定会使得其变为只出现一次或者不出现。

不难发现本题使用了拓扑排序的思想：先有一些对象可用，使它们进队，然后经过操作又能让其他对象可用并进队。

另外还有一种方法：人为指定一个遍历次数，每轮遍历看看那些需要被删除。因为遍历到一定次数之后数据就不会再变化，所以可以指定一个合适的遍历次数（这需要对数据有个较为精确的估计，过大过小都不好）。
## Code
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1e5 + 5;
int n;
int appear[2][maxn];
struct Col {
    int a, b, c;
    bool del;
} col[maxn];
queue<int> que;
bool visited[maxn];

inline bool compare(const Col &c1, const Col &c2) {
    return c1.a < c2.a;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &col[i].a);
    }
    for (int i = 1; i <= n; i++) {
        scanf("%d", &col[i].b);
        appear[0][col[i].b]++;
    }
    for (int i = 1; i <= n; i++) {
        scanf("%d", &col[i].c);
        appear[1][col[i].c]++;
    }
    sort(col + 1, col + 1 + n, compare);
    for (int i = 1; i <= n; i++) {
        if (!appear[0][i] || !appear[1][i]) {
            que.push(i);
        }
    }
    int ans = 0;
    while (!que.empty()) {
        int u = que.front();
        que.pop();
        if (visited[u]) continue;
        visited[u] = true;
        appear[0][col[u].b]--;
        appear[1][col[u].c]--;
        if (!appear[0][col[u].b]) que.push(col[u].b);
        if (!appear[1][col[u].c]) que.push(col[u].c);
        ans++;
    }
    printf("%d", ans);
    return 0;
}
```
