[P2744 USACO5.3 量取牛奶Milk Measuring - 洛谷](https://www.luogu.com.cn/problem/P2744) 
## 分析
方法一：

这道题看到之后很容易想到背包 DP，但是直接写完全背包不太好维护所选的物品。然后就想到通过搜索暴力枚举各种组合，然后通过 DP 判断方案是否可行。然而，题目中 $p \le 100$，直接暴搜肯定会超时。题目中要求所选物品数量最少，且字典序最小，那么我们先排个序，然后通过迭代加深控制深度，既保证数量，又保证字典序，还避免大量无意义的搜索。不过，事实上，这样并没有从本质上优化复杂度，仍是指数级。迭代加深可以通过控制深度来避免无意义搜索，但是在深度很深的时候，迭代深搜反而会做更多的重复计算，只是题目数据很水（排完序后基本前 $3$ 个桶就可以出答案），所以即便如此也可以通过，而且迭代深搜还挺快。

方法二：

还是完全背包，但是我们可以想一下使用状压的方式来记录所选物品，不过本题 $p$ 取值过大，数组显然不能开 $O(2^{100})$，怎么办呢？

这个时候，可以使用二进制神器：`bitset`！

我们可以对每一个体积，用一个长度为 $100$ 的 `bitset` 维护恰好能量出当前体积的物品的状态，然后用背包问题来进行处理。这样空间是 $O(Q \frac{p}{32})$，完全可以接受。

## 代码
迭代加深
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 105, maxm = 20005;
int Vol, n;
int arr[maxn], f[maxm];

int sel[maxn];
void dpCalc(int all) {
	memset(f, 0, sizeof(f));
	f[0] = 1;
	for (int i = 1; i <= all; i++) {
		for (int j = sel[i]; j <= Vol; j++) {
			f[j] = f[j] || f[j - sel[i]];
		}
	}
	if (f[Vol]) {
		printf("%d ", all);
		for (int i = 1; i <= all; i++) {
			printf("%d ", sel[i]);
		}
		exit(0);
	}
}

void dfs(int cur, int chosen, int all) {
	if (cur > n) return;
	if (n - cur < all - chosen) return; // 剩下的物品不够选了，直接返回
	if (chosen == all) {
		dpCalc(all);
		return;
	}
	sel[chosen + 1] = arr[cur + 1];
	dfs(cur + 1, chosen + 1, all);
	sel[chosen + 1] = 0;
	dfs(cur + 1, chosen, all);
}

int main() {
	scanf("%d%d", &Vol, &n);
	for (int i = 1; i <= n; i++) {
		scanf("%d", &arr[i]);
	}
	sort(arr + 1, arr + 1 + n);
	// special situation.
	for (int i = 1; i <= n; i++) {
		if (Vol % arr[i] == 0) {
			printf("1 %d", arr[i]);
			return 0;
		}
	}
	for (int i = 2; i <= n; i++) {
		dfs(0, 0, i);
	}
	return 0;
}
```

bitset 法
```cpp
#include <bits/stdc++.h>  
using namespace std;  
  
const int maxq = 20005, maxn = 105;  
int arr[maxn];  
int n, Vol;  
bitset<100> chosen[maxq];  
  
int main() {  
    freopen("water.in", "r", stdin);  
    freopen("water.out", "w", stdout);  
    scanf("%d%d", &Vol, &n);  
    for (int i = 0; i < n; i++) {  
        scanf("%d", &arr[i]);  
    }  
    sort(arr, arr + n);  
    for (int i = 1; i <= Vol; i++) {  
        chosen[i].set();
    }  
    for (int i = 0; i < n; i++) {  
        for (int j = Vol; j >= arr[i]; j--) {  
            for (int k = arr[i]; k <= j; k += arr[i]) {  
                if (chosen[j - k].count() + 1 < chosen[j].count()) {  
                    chosen[j] = chosen[j - k];  
                    chosen[j].set(i);  
                } else if (chosen[j - k].count() + 1 == chosen[j].count()) {  
                    bitset<100> temp = chosen[j - k];  
                    temp.set(i);  
                    for (int pos = 0; pos < 100; pos++) {  
                        if (temp.test(pos) ^ chosen[j].test(pos)) {  
                            if (!chosen[j].test(pos)) {  
                                chosen[j] = temp;  
                            }  
                            break;  
                        }  
                    }  
                }  
            }  
        }  
    }  
    printf("%d ", (int)chosen[Vol].count());  
    for (int i = 0; i < 100; i++) {  
        if (chosen[Vol].test(i)) {  
            printf("%d ", arr[i]);  
        }  
    }  
    return 0;  
}
```
### 关于 bitset 的使用
[[bitset 使用]]
