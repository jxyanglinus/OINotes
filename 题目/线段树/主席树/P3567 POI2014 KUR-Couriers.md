---
tags:
  - 可持久化
  - 线段树
  - 主席树
---
[P3567 POI2014 KUR-Couriers](https://www.luogu.com.cn/problem/P3567)
# Solution
- 出现次数严格大于一半的数只可能有一个。
- 对于区间 $[l,\:r]$，其中点为 $mid$：
	- 若 $[l,\:mid]$ 中所有数出现的次数乘 $2$ 之后大于整个查询区间的长度，那么 $[l,\:mid]$ 就有可能存在出现次数严格大于一半的数，另一半绝对不可能。
	- 对于 $[mid + 1,\:r]$ 也同理。
	- 若两边都不存在，则直接返回 $0$。
	- 最后区间长度为 $1$ 的时候就找到了答案。
# Code
```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int maxn = 5e5 + 5;
int n, m;
int idcnt;
int arr[maxn];
int tree[maxn << 5], rootid[maxn << 5], lson[maxn << 5], rson[maxn << 5];

void update(int &root, int pre, int pos, int nowl, int nowr) {
    root = ++idcnt;
    tree[root] = tree[pre] + 1;
    lson[root] = lson[pre];
    rson[root] = rson[pre];
    if (nowl == nowr) return;
    int mid = (nowl + nowr) >> 1;
    if (mid >= pos) update(lson[root], lson[pre], pos, nowl, mid);
    else update(rson[root], rson[pre], pos, mid + 1, nowr);
}

int query(int l, int r, int len, int nowl, int nowr) {
    if (nowl == nowr) return nowl;
    int mid = (nowl + nowr) >> 1;
    if (2 * (tree[lson[r]] - tree[lson[l]]) > len) return query(lson[l], lson[r], len, nowl, mid);
    if (2 * (tree[rson[r]] - tree[rson[l]]) > len) return query(rson[l], rson[r], len, mid + 1, nowr);
    return 0;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &arr[i]);
    }
    for (int i = 1; i <= n; i++) {
        update(rootid[i], rootid[i - 1], arr[i], 1, n);
    }
    while (m--) {
        int l, r;
        scanf("%d%d", &l, &r);
        printf("%d\n", query(rootid[l - 1], rootid[r], r - l + 1, 1, n));
    }
    return 0;
}
```
