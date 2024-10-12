bitset 真的是二进制神器啊！处理 0/1 状态巨好用。
## 成员函数
- `set()` 将每一位都设置为 $1$。
- `set(pos)` 将第 `pos` 个改为 $1$。
- `reset()` 全部设置为 $0$。
- `reset(pos)` 将 `bitset` 中位于 `pos` 位置的值设为 $0$。
- `count()` 统计所有 $1$ 的个数。
- `test(pos)` 返回 `pos` 位的值。
- `size()` 返回 `bitset` 的长度
- `any()` 返回 `bitset` 中是否存在值为 $1$ 的位。
- `none()` 返回 `bitset` 中是否所有位都是 $0$。
- `all()` 返回 `bitset` 中是否所有位都是 1。
- `test(pos)` 返回 `bitset` 中位于 `pos` 位置的值。
- `flip(pos)` 将 `bitset` 中位于 `pos` 位置的值取反。
- `flip()` 将 `bitset` 中的所有值取反。
- `to_ulong()` 返回 `bitset` 转换成的无符号整数值。
- `to_ullong()` 返回 `bitset` 转换成的无符号长整数值。
## bitset 之间的运算
`bitset` 之间可以像整数一样进行位运算（`&`，`|`，`^`），例如可以用 `|` 将两个 `bitset` 合并。
## 例题
[[P2744 USACO5.3 量取牛奶Milk Measuring]] 
