---
tags:
  - 线段树
  - 贪心
---
[P1607 USACO09FEB Fair Shuttle G](https://www.luogu.com.cn/problem/P1607)

# Solution
- 典型的贪心，思路近似于[借教室](https://www.luogu.com.cn/problem/P1083)。
- 按区间右端点从小到大排序：
![[为何按右端点排序.png]]
- 需要注意的是，奶牛在 $S_i$ 上车，在 $E_i$ 下车，那么到 $E_i$ 的时候，奶牛下车后就会把空间空出来，所以更新的时候只更新 $[S_i,\:E_i-1]$。
- 车子装不下的时候，可以让一部分奶牛先上车，所以在上车的时候取剩余空位与 $M_i$ 中的较小值。
# Code
```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int maxn = 50005;
int k, n, c;
struct CowGroup {
    int start, ending, number;
    bool operator < (const CowGroup &that) const {
        return ending < that.ending;
    }
} cow[maxn];
int rootid, idcnt;
int tree[maxn << 1], tag[maxn << 1], lson[maxn << 1], rson[maxn << 1];

inline void pushup(int root) {
    tree[root] = max(tree[lson[root]], tree[rson[root]]);
}

inline void pushdown(int root) {
    if (tag[root] == 0) return;
    if (!lson[root]) lson[root] = ++idcnt;
    if (!rson[root]) rson[root] = ++idcnt;
    tree[lson[root]] += tag[root];
    tree[rson[root]] += tag[root];
    tag[lson[root]] += tag[root];
    tag[rson[root]] += tag[root];
    tag[root] = 0;
}

void update(int &root, int l, int r, int val, int nowl, int nowr) {
    if (!root) root = ++idcnt;
    if (nowl >= l && nowr <= r) {
        tree[root] += val;
        tag[root] += val;
        return;
    }
    int mid = (nowl + nowr) >> 1;
    if (mid >= l) update(lson[root], l, r, val, nowl, mid);
    if (mid < r) update(rson[root], l, r, val, mid + 1, nowr);
    pushup(root);
}

int query(int root, int l, int r, int nowl, int nowr) {
    if (!root) return 0;
    if (nowl >= l && nowr <= r) {
        return tree[root];
    }
    pushdown(root);
    int mid = (nowl + nowr) >> 1;
    int ans = 0;
    if (mid >= l) ans = max(ans, query(lson[root], l, r, nowl, mid));
    if (mid < r) ans = max(ans, query(rson[root], l, r, mid + 1, nowr));
    return ans;
}

int main() {
    scanf("%d%d%d", &k, &n, &c);
    for (int i = 1; i <= k; i++) {
        scanf("%d%d%d", &cow[i].start, &cow[i].ending, &cow[i].number);
    }
    sort(cow + 1, cow + 1 + k);
    int ans = 0;
    for (int i = 1; i <= k; i++) {
        int rest = query(rootid, cow[i].start, cow[i].ending - 1, 1, n);
        int select = min(c - rest, cow[i].number);
        ans += select;
        update(rootid, cow[i].start, cow[i].ending - 1, select, 1, n);
    }
    printf("%d", ans);
    return 0;
}
```
