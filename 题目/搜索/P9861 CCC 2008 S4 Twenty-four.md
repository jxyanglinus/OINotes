[P9861 CCC 2008 S4 Twenty-four - 洛谷](https://www.luogu.com.cn/problem/P9861) 
## 分析

对于 $4$ 张牌，我们枚举其全排列即可。对于中间的 $3$ 个符号也是如此。

在处理除法时需要注意，除法运算除数不能为 $0$，且参与除法运算的两个数必须是可以整除的数。

然后，即使枚举了所有牌和符号的排列，计算的顺序也会对结果造成影响。设 $4$ 张牌分别是 $A, B, C, D$，则有以下计算顺序：
$$
\begin{aligned}
(((AB)C)D) \\
((A(BC))D) \\
(A((BC)D)) \\
(A(B(CD))) \\
((AB)(CD))
\end{aligned}
$$
## 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

int T;
int ans;
int arr[10];

int calc(int x, int y, int opt, bool &state) {
	int ans = 0;
	state = true;
	switch (opt) {
	case 1:
		ans = x + y;
		break;
	case 2:
		ans = x - y;
		break;
	case 3:
		ans = x * y;
		break;
	case 4:
		if (y == 0 || x % y != 0) {
			state = false;
			break;
		}
		ans = x / y;
		break;
	}
	return ans;
}

bool visited[10];
int order[10];
void dfs(int cur) {
	if (cur == 4) {
		bool state1 = false, state2 = false, state3 = false;
		for (int i = 1; i <= 4; i++) {
			for (int j = 1; j <= 4; j++) {
				for (int k = 1; k <= 4; k++) {
					int val = calc(calc(calc(order[1], order[2], i, state1), order[3], j, state2), order[4], k, state3);
					if (state1 && state2 && state3 &&  val <= 24) ans = max(ans, val);
					val = calc(calc(order[1], order[2], i,state1), calc(order[3], order[4], j, state2), k, state3);
					if (state1 && state2 && state3 &&  val <= 24) ans = max(ans, val);
					val = calc(calc(order[1], calc(order[2], order[3], i, state1), j, state2), order[4], k, state3);
					if (state1 && state2 && state3 &&  val <= 24) ans = max(ans, val);
					val = calc(order[1], calc(order[2], calc(order[3], order[4], i, state1), j, state2), k, state3);
					if (state1 && state2 && state3 &&  val <= 24) ans = max(ans, val);
					val = calc(order[1], calc(calc(order[2], order[3], i, state1), order[4], j, state2), k, state3);
					if (state1 && state2 && state3 &&  val <= 24) ans = max(ans, val);
				}
			}
		}
		return;
	}
	for (int i = 1; i <= 4; i++) {
		if (!visited[i]) {
			visited[i] = true;
			order[cur + 1] = arr[i];
			dfs(cur + 1);
			order[cur + 1] = 0;
			visited[i] = false;
		}
	}
}

int main() {
	scanf("%d", &T);
	while (T--) {
		ans = 0;
		for (int i = 1; i <= 4; i++) {
			scanf("%d", &arr[i]);
		}
		dfs(0);
		printf("%d\n", ans);
	}
	return 0;
}
```
