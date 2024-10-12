# 错题：P2895 USACO08FEB Meteor Shower S

[P2895 USACO08FEB Meteor Shower S - 洛谷](https://www.luogu.com.cn/problem/P2895) 

## 错的地方

1. 贝茜行动坐标不能低于 $0$，但可以超 $300$！所以边界不能取 $300$。

2. （傻逼错点）题目说无解时输出 $-1$，~~然后我就忘了~~。

3. （傻逼错点）在算结果的时候没有排除未被访问到的点（取值为 $0$），结果好多测试点跑出来是 $0$。

4. （重点）在我原本的做法中，采用动态更新地图的方法：因为按照 BFS 的原理，入队的点的时间戳一定是不下降的，所以我就在每次处理队首的时候，把地图更新到当前队首的时间戳时的状态，然后再在当前地图上继续搜索。这样做的问题是：可能会有流星无法处理。

   - 以一下这组数据为例
```
5
0 0 2
3 0 0
1 2 5
2 2 4
1 4 4
```

   - 当时间戳为 $3$ 的时候，地图会被更新成这样
```
x x 2 3 4 
x 2 3 4 0 
x 3 4 0 0 
x x 0 0 0 
x 0 0 0 0  // x 表示被流星轰了之后无法行走，数字表示步数，0 表示没有访问到
```

   - 而当时间戳为 $4$ 的时候，会变成这样
```
x x 2 3 x 
x 2 x x x 
x x x x x 
x x x 0 0 
x 0 0 0 0 
```

   - 时间戳为 $4$ 的时候，坠落在 $(2, 2)$ 的流星会把 $(1, 2)$ 覆盖，即把第 $5$ 颗流星的位置给标记为毁坏，那么在继续走的时候，第五颗流星的位置也就不会过去了，同时现在也无法产生时间戳为 $5$ 的点了，那么上面两个时间戳为 $2$ 的点，按理来说最后是会被标记为毁坏的，最终答案应该为 $3$，但是由于没有把地图更新到时间戳为 $5$ 的状态，这两个点被认为是可被访问的，导致最终答案变成了 $2$。

   - 把错的代码贴出来
```cpp
#include <bits/stdc++.h>
using namespace std;

//#define __DEBUG__

const int maxn = 305;
const int maxm = 50005;
int n;
int ans = 0x3f3f3f3f;
struct Meteor {
	int x, y, tim;
} meteor[maxm];
enum Range {
	Minv = 0, Maxv = 400 // 错点：这里需要大于 300
};

inline bool compare(const Meteor &m1, const Meteor &m2) {
	return m1.tim < m2.tim;
}

int dist[maxn][maxn];
bool visited[maxn][maxn], blocked[maxn][maxn];
int dx[4] = { 0, 0, 1, -1 };
int dy[4] = { 1, -1, 0, 0 };
inline bool inArea(int x, int y) {
	return x >= Range::Minv && x <= Range::Maxv && y >= Range::Minv && y <= Range::Maxv;
}

int current = 1;
inline void UpdateMap(int tim) {
#ifdef __DEBUG__
	printf("* Now in UpdateMap. current = %d, tim = %d\n", current, tim);
#endif // __DEBUG__
	for (; current <= n; current++) {
		if (meteor[current].tim > tim) {
#ifdef __DEBUG__
			printf("* Breaked. current = %d, Meteor::tim = %d\n", current, meteor[current].tim);
#endif // __DEBUG__
			break;
		}
		blocked[meteor[current].x][meteor[current].y] = true;
		for (int i = 0; i < 4; i++) {
			int vx = meteor[current].x + dx[i], vy = meteor[current].y + dy[i];
			if (inArea(vx, vy)) blocked[vx][vy] = true;
		}
	}
#ifdef __DEBUG__
	printf("Current distance matrix:\n");
	for (int i = Range::Minv; i <= Range::Maxv; i++) {
		for (int j = Range::Minv; j <= Range::Maxv; j++) {
			if (!blocked[i][j]) printf("%d ", dist[i][j]);
			else printf("x ");
		}
		printf("\n");
	}
#endif // __DEBUG__
}

inline void Bfs() {
	queue<Meteor> que;
	que.push(Meteor { 0, 0, 0 });
	visited[0][0] = true;
	while (!que.empty()) {
		Meteor u = que.front();
		que.pop();
#ifdef __DEBUG__
		if (u.tim == 5) printf("* Final meteor captured: %d", u.tim); // 重量级：在被卡的测试点中，无法产生时间戳为 5 的点
#endif // __DEBUG__
		UpdateMap(u.tim);
		if (blocked[u.x][u.y]) {
			continue;
		}
		for (int i = 0; i < 4; i++) {
			int vx = u.x + dx[i], vy = u.y + dy[i];
			if (!inArea(vx, vy)) continue;
			if (!blocked[vx][vy] && !visited[vx][vy]) {
				dist[vx][vy] = dist[u.x][u.y] + 1;
				visited[vx][vy] = true;
				que.push(Meteor{ vx, vy, u.tim + 1 });
			}
		}
	}
}

int main() {
	scanf("%d", &n);
	for (int i = 1; i <= n; i++) {
		scanf("%d%d%d", &meteor[i].x, &meteor[i].y, &meteor[i].tim);
	}
	sort(meteor + 1, meteor + 1 + n, compare); // 按时间排序
	Bfs();
	for (int i = Range::Minv; i <= Range::Maxv; i++) {
		for (int j = Range::Minv; j <= Range::Maxv; j++) {
			if (blocked[i][j] || !visited[i][j]) continue; // 错点：这里一开始忘了要排除 visited[i][j] == 0 的点
			ans = min(ans, dist[i][j]);
		}
	}
	printf("%d", ans == 0x3f3f3f3f ? -1 : ans);
	return 0;
}
```

## 正解

预处理出每个点被砸中或被破坏的最小时间，在 BFS 的过程中，如果到达点的时间（步数）在其被破坏时间之间可以到达，如果到达一个没有破坏的点就是安全的点。对于不会受影响的点，将其设置为正无穷。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 505;
const int maxm = 50005;
int n;
int fall[maxn][maxn], dist[maxn][maxn];
bool visited[maxn][maxn];
struct Meteor {
    int x, y, tim;
} meteor[maxm];
enum Range {
    Low = 0, High = 400
};

inline bool inArea(int x, int y) {
    return x >= Range::Low && x <= Range::High && y >= Range::Low && y <= Range::High;
}

int dx[4] = {0, 0, -1, 1};
int dy[4] = {-1, 1, 0, 0};
inline void Bfs() {
    queue<Meteor> que;
    memset(dist, 0x3f, sizeof(dist));
    dist[0][0] = 0;
    que.push(Meteor{ 0, 0, 0 });
    visited[0][0] = true;
    while (!que.empty()) {
        Meteor u = que.front();
        que.pop();
        for (int i = 0; i < 4; i++) {
            int vx = u.x + dx[i], vy = u.y + dy[i];
            if (!inArea(vx, vy) || visited[vx][vy]) continue;
            if (fall[vx][vy] == 0x3f3f3f3f) {
                printf("%d", u.tim + 1);
                exit(0);
            }
            if (u.tim + 1 < fall[vx][vy]) {
                visited[vx][vy] = true;
                que.push(Meteor{vx, vy, u.tim + 1});
                dist[vx][vy] = min(dist[vx][vy], dist[u.x][u.y] + 1);
            }
        }
    }
    printf("-1");
}

int main() {
    scanf("%d", &n);
    memset(fall, 0x3f, sizeof(fall));
    for (int i = 1; i <= n; i++) {
        scanf("%d%d%d", &meteor[i].x, &meteor[i].y, &meteor[i].tim);
        fall[meteor[i].x][meteor[i].y] = min(meteor[i].tim, fall[meteor[i].x][meteor[i].y]);
        for (int j = 0; j < 4; j++) {
            int vx = meteor[i].x + dx[j], vy = meteor[i].y + dy[j];
            if (!inArea(vx, vy)) continue;
            fall[vx][vy] = min(meteor[i].tim, fall[vx][vy]); // 重叠时取最小值
        }
    }
    Bfs();
    return 0;
}
```