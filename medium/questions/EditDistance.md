# 编辑距离（Edit Distance）

给定两个仅由小写英文字母组成的字符串 `word1` 和 `word2`。

你可以对 `word1` 执行任意次数的以下三种操作：

- 在任意位置插入一个字符
- 删除任意位置的一个字符
- 替换任意位置的一个字符

返回将 `word1` 转换成 `word2` 所需的最少操作次数。

------

## 核心思路与状态定义

设：

- `m = len(word1)`
- `n = len(word2)`
- `dfs(i, j)` 或 `dp[i][j]` 表示：将后缀 `word1[i:]` 转换成后缀 `word2[j:]` 所需的最少操作次数。

在位置 `(i, j)`：

### 字符相同

如果：

```
word1[i] == word2[j]
```

当前字符不需要修改，两个指针同时向后移动：

```
dp[i][j] = dp[i + 1][j + 1]
```

### 字符不同

可以选择三种操作：

1. 删除 `word1[i]`

   删除后，`word1` 的指针向后移动，`word2` 的指针不变：

   ```
   dp[i + 1][j]
   ```

2. 向 `word1` 插入 `word2[j]`

   插入的字符已经和 `word2[j]` 匹配，因此 `word2` 的指针向后移动，而 `word1` 的原字符还没有处理：

   ```
   dp[i][j + 1]
   ```

3. 将 `word1[i]` 替换为 `word2[j]`

   替换后两个字符已经匹配，两个指针同时向后移动：

   ```
   dp[i + 1][j + 1]
   ```

因此状态转移方程为：

```
dp[i][j] = 1 + min(
    dp[i + 1][j],      # 删除
    dp[i][j + 1],      # 插入
    dp[i + 1][j + 1],  # 替换
)
```

------

## 1. 递归

### 思路

递归函数 `dfs(i, j)` 表示：

> 将 `word1[i:]` 转换成 `word2[j:]` 所需的最少操作次数。

如果当前字符相同，不需要执行操作，直接处理剩余部分。

如果当前字符不同，则分别尝试插入、删除和替换，并取三种选择中的最小值。

### 边界情况

如果 `word1` 已经处理完：

```
i == m
```

那么必须将 `word2[j:]` 中的所有剩余字符插入 `word1`，需要：

```
n - j
```

次操作。

如果 `word2` 已经处理完：

```
j == n
```

那么必须删除 `word1[i:]` 中的所有剩余字符，需要：

```
m - i
```

次操作。

### 算法步骤

1. 定义递归函数 `dfs(i, j)`。
2. 如果 `i == m`，返回 `n - j`。
3. 如果 `j == n`，返回 `m - i`。
4. 如果两个当前字符相同，返回 `dfs(i + 1, j + 1)`。
5. 否则分别计算插入、删除和替换的结果。
6. 取三者中的最小值，并加上当前操作的代价 `1`。
7. 从 `dfs(0, 0)` 开始递归。

### 代码

```
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        def dfs(i: int, j: int) -> int:
            # word1 已处理完，只能插入 word2 剩余的字符
            if i == m:
                return n - j

            # word2 已处理完，只能删除 word1 剩余的字符
            if j == n:
                return m - i

            # 当前字符相同，不需要执行操作
            if word1[i] == word2[j]:
                return dfs(i + 1, j + 1)

            # 删除 word1[i]
            delete_cost = dfs(i + 1, j)

            # 在 word1 中插入 word2[j]
            insert_cost = dfs(i, j + 1)

            # 将 word1[i] 替换成 word2[j]
            replace_cost = dfs(i + 1, j + 1)

            # 加 1 表示执行当前这一次操作
            return 1 + min(delete_cost, insert_cost, replace_cost)

        return dfs(0, 0)
```

### 复杂度分析

- 时间复杂度：指数级，宽松上界为 `O(3^(m+n))`
- 空间复杂度：`O(m+n)`

递归树中每个状态最多产生三个分支，而且没有缓存重复子问题，因此效率很低。空间复杂度来自递归调用栈，其最大深度为 `O(m+n)`。

------

## 2. 自顶向下动态规划（记忆化搜索）

### 思路

普通递归会重复计算大量相同的状态。例如，经过不同的操作顺序后，可能多次到达同一个 `(i, j)`。

由于一个子问题完全由 `i` 和 `j` 决定，因此可以使用字典保存已经计算过的结果：

```
memo[(i, j)]
```

它表示将 `word1[i:]` 转换成 `word2[j:]` 所需的最少操作次数。

这就是记忆化搜索，也叫自顶向下动态规划。

### 算法步骤

1. 定义字典 `memo` 保存已经计算过的状态。
2. 如果其中一个字符串已经处理完，返回对应的剩余长度。
3. 如果 `(i, j)` 已经计算过，直接返回缓存结果。
4. 如果当前字符相同，递归处理 `(i + 1, j + 1)`。
5. 如果当前字符不同，尝试插入、删除和替换。
6. 将计算结果保存到 `memo[(i, j)]`。
7. 返回 `dfs(0, 0)`。

### 代码

```
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # key 是 (i, j)，value 是对应状态的最小编辑距离
        memo = {}

        def dfs(i: int, j: int) -> int:
            # word1 已处理完，需要插入 word2 剩余的全部字符
            if i == m:
                return n - j

            # word2 已处理完，需要删除 word1 剩余的全部字符
            if j == n:
                return m - i

            # 避免重复计算相同状态
            if (i, j) in memo:
                return memo[(i, j)]

            if word1[i] == word2[j]:
                # 当前字符相同，不产生编辑代价
                memo[(i, j)] = dfs(i + 1, j + 1)
            else:
                delete_cost = dfs(i + 1, j)
                insert_cost = dfs(i, j + 1)
                replace_cost = dfs(i + 1, j + 1)

                memo[(i, j)] = 1 + min(
                    delete_cost,
                    insert_cost,
                    replace_cost,
                )

            return memo[(i, j)]

        return dfs(0, 0)
```

也可以使用 Python 标准库中的 `functools.cache` 自动完成缓存：

```
from functools import cache


class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        @cache
        def dfs(i: int, j: int) -> int:
            if i == m:
                return n - j

            if j == n:
                return m - i

            if word1[i] == word2[j]:
                return dfs(i + 1, j + 1)

            return 1 + min(
                dfs(i + 1, j),      # 删除
                dfs(i, j + 1),      # 插入
                dfs(i + 1, j + 1),  # 替换
            )

        return dfs(0, 0)
```

### `functools.cache` 说明

`@cache` 是一个装饰器，会根据函数参数自动缓存函数的返回值。

例如，第一次调用：

```
dfs(2, 3)
```

时会正常计算结果。以后再次调用 `dfs(2, 3)`，Python 会直接返回之前缓存的结果，不再重复递归。

因为这里的参数 `i` 和 `j` 都是整数，可以安全地作为缓存键。

### 复杂度分析

- 时间复杂度：`O(mn)`
- 空间复杂度：`O(mn)`

一共有至多 `(m+1)(n+1)` 个不同状态，每个状态只计算一次。

空间包括：

- 记忆化缓存：`O(mn)`
- 递归调用栈：`O(m+n)`

整体由缓存占用主导，因此为 `O(mn)`。

------

## 3. 自底向上动态规划

### 思路

递归和记忆化搜索从 `(0, 0)` 开始解决问题。自底向上动态规划则反过来，先计算较短后缀的答案，再逐步得到完整字符串的答案。

定义：

```
dp[i][j]
```

表示将 `word1[i:]` 转换成 `word2[j:]` 所需的最少操作次数。

由于状态 `(i, j)` 依赖：

- `(i + 1, j)`
- `(i, j + 1)`
- `(i + 1, j + 1)`

所以需要按照从右下角到左上角的顺序填表。

### 边界初始化

如果 `word1` 已经处理完：

```
dp[m][j] = n - j
```

需要插入 `word2[j:]` 中的全部字符。

如果 `word2` 已经处理完：

```
dp[i][n] = m - i
```

需要删除 `word1[i:]` 中的全部字符。

### 算法步骤

1. 创建大小为 `(m+1) × (n+1)` 的二维数组。
2. 初始化最后一行，表示 `word1` 已经处理完。
3. 初始化最后一列，表示 `word2` 已经处理完。
4. 从右下角向左上角遍历。
5. 当前字符相同时，使用右下角状态。
6. 当前字符不同时，取删除、插入和替换中的最小值，再加 `1`。
7. 返回 `dp[0][0]`。

### 代码

```
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # 多出的一行和一列用于表示某个字符串已经处理完
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        # word1 已处理完：需要插入 word2 剩余字符
        for j in range(n + 1):
            dp[m][j] = n - j

        # word2 已处理完：需要删除 word1 剩余字符
        for i in range(m + 1):
            dp[i][n] = m - i

        # 从右下角向左上角填表
        for i in range(m - 1, -1, -1):
            for j in range(n - 1, -1, -1):
                if word1[i] == word2[j]:
                    # 当前字符相同，不需要操作
                    dp[i][j] = dp[i + 1][j + 1]
                else:
                    delete_cost = dp[i + 1][j]
                    insert_cost = dp[i][j + 1]
                    replace_cost = dp[i + 1][j + 1]

                    dp[i][j] = 1 + min(
                        delete_cost,
                        insert_cost,
                        replace_cost,
                    )

        return dp[0][0]
```

### Python 二维列表说明

下面的写法通过列表推导式创建了 `m+1` 个相互独立的列表：

```
dp = [[0] * (n + 1) for _ in range(m + 1)]
```

不要写成：

```
dp = [[0] * (n + 1)] * (m + 1)
```

后一种写法会让所有行引用同一个列表。修改其中一行时，其他行也会同时改变。

### 复杂度分析

- 时间复杂度：`O(mn)`
- 空间复杂度：`O(mn)`

需要计算 `m × n` 个主要状态，每个状态只执行常数次操作。

------

## 4. 动态规划：两行空间优化

### 思路

在二维动态规划中，`dp[i][j]` 只依赖：

- 下一行的 `dp[i+1][j]`
- 当前行右侧的 `dp[i][j+1]`
- 下一行右下角的 `dp[i+1][j+1]`

因此不需要保存整个二维表，只需要保存：

- `next_row`：下一行，即原来的第 `i+1` 行
- `current_row`：当前正在计算的第 `i` 行

为了进一步减少空间，可以让数组长度取决于较短的字符串。

### 为什么可以交换两个字符串

编辑距离具有对称性：

```
distance(word1, word2) = distance(word2, word1)
```

从 `word1` 到 `word2` 的一次插入，相当于反方向转换时的一次删除；替换操作则在两个方向上完全对称。

因此可以交换两个字符串，使第二个字符串更短，从而令一维数组长度为 `min(m,n)+1`。

### 代码

```
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # 让 word2 成为较短的字符串，减小数组长度
        if m < n:
            word1, word2 = word2, word1
            m, n = n, m

        # next_row 表示下一行 dp[i+1][...]
        # 当 word1 已处理完时，需要插入 word2 剩余的字符
        next_row = [n - j for j in range(n + 1)]

        for i in range(m - 1, -1, -1):
            current_row = [0] * (n + 1)

            # word2 已处理完，需要删除 word1 剩余字符
            current_row[n] = m - i

            for j in range(n - 1, -1, -1):
                if word1[i] == word2[j]:
                    current_row[j] = next_row[j + 1]
                else:
                    delete_cost = next_row[j]
                    insert_cost = current_row[j + 1]
                    replace_cost = next_row[j + 1]

                    current_row[j] = 1 + min(
                        delete_cost,
                        insert_cost,
                        replace_cost,
                    )

            # 当前行将成为下一轮的“下一行”
            next_row = current_row

        return next_row[0]
```

这里每轮直接创建新的 `current_row`，状态含义比较清晰。原答案中的：

```
dp = nextDp[:]
```

使用的是列表切片，它会创建 `nextDp` 的浅拷贝。由于列表中只有整数，浅拷贝完全足够。

### 复杂度分析

- 时间复杂度：`O(mn)`
- 空间复杂度：`O(min(m,n))`

虽然只保存两行，但常数空间仍然是两倍的较短字符串长度，渐进空间复杂度依然为 `O(min(m,n))`。

------

## 5. 动态规划：单数组空间优化

### 思路

两行动态规划还可以继续压缩成一个数组。

更新某个位置前，数组中的值表示下一行；更新后，则表示当前行：

```
更新前 dp[j]     = dp[i+1][j]
更新后 dp[j]     = dp[i][j]
更新后 dp[j+1]   = dp[i][j+1]
```

还缺少右下角的旧值：

```
dp[i+1][j+1]
```

由于这个值在原数组中可能已经被覆盖，因此使用额外变量 `diagonal` 保存它。

### 单数组更新过程

在计算当前状态时：

- `dp[j]`：尚未更新，表示下一行同列，即删除操作
- `dp[j+1]`：已经更新，表示当前行右侧，即插入操作
- `diagonal`：保存下一行右侧的旧值，即替换或匹配操作

每次覆盖 `dp[j]` 前，先将旧值保存到 `old_dp_j`。完成更新后：

```
diagonal = old_dp_j
```

为下一次循环准备新的右下角旧值。

### 代码

```
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # 让 dp 数组基于较短的字符串
        if m < n:
            word1, word2 = word2, word1
            m, n = n, m

        # word1 已处理完时，需要插入 word2 剩余的字符
        dp = [n - j for j in range(n + 1)]

        for i in range(m - 1, -1, -1):
            # 保存旧的 dp[n]，即下一行右下角的值
            diagonal = dp[n]

            # word2 已处理完，需要删除 word1 剩余字符
            dp[n] = m - i

            for j in range(n - 1, -1, -1):
                # 覆盖 dp[j] 之前，保存下一行同列的旧值
                old_dp_j = dp[j]

                if word1[i] == word2[j]:
                    # diagonal 表示 dp[i+1][j+1]
                    dp[j] = diagonal
                else:
                    dp[j] = 1 + min(
                        dp[j],      # 删除：旧的 dp[i+1][j]
                        dp[j + 1],  # 插入：新的 dp[i][j+1]
                        diagonal,   # 替换：旧的 dp[i+1][j+1]
                    )

                # 为下一次向左移动保存新的对角线旧值
                diagonal = old_dp_j

        return dp[0]
```

### 复杂度分析

- 时间复杂度：`O(mn)`
- 空间复杂度：`O(min(m,n))`

这是该动态规划转移下的最优空间复杂度。

------

## 示例分析

假设：

```
word1 = "horse"
word2 = "ros"
```

一种最优转换过程是：

```
horse
rorse   将 h 替换为 r
rose    删除第二个 r
ros     删除 e
```

总共需要 `3` 次操作，因此答案是：

```
3
```

需要注意，最优操作序列可能不唯一，但最少操作次数是确定的。

------

## 常见错误

### 1. 混淆插入和删除对应的状态

将 `word1` 转换成 `word2` 时：

- 删除 `word1[i]`：`i` 向后移动

  ```
  dfs(i + 1, j)
  ```

- 插入 `word2[j]`：相当于当前目标字符已经匹配，`j` 向后移动

  ```
  dfs(i, j + 1)
  ```

可以记忆为：

> 删除会消耗源字符串中的字符，插入会满足目标字符串中的字符。

### 2. 边界条件返回错误

如果 `word1` 已经处理完，不能直接返回 `0`，因为还要插入 `word2` 中剩余的字符：

```
if i == m:
    return n - j
```

同理，如果 `word2` 已经处理完，需要删除 `word1` 中剩余的字符：

```
if j == n:
    return m - i
```

### 3. 忘记计算当前操作

字符不同时，三个子问题只表示“执行当前操作之后”的剩余代价。

因此必须加上当前操作本身的代价：

```
1 + min(delete_cost, insert_cost, replace_cost)
```

### 4. 二维数组没有多创建一行和一列

动态规划数组必须是：

```
(m + 1) × (n + 1)
```

多出的一行和一列用来表示字符串已经处理完的情况。

如果只创建 `m × n`，访问 `dp[m][j]` 或 `dp[i][n]` 时会越界。

### 5. 字符相同时仍然加 `1`

如果：

```
word1[i] == word2[j]
```

不需要执行任何操作，因此应该直接使用：

```
dp[i][j] = dp[i + 1][j + 1]
```

不能额外加 `1`。

### 6. 单数组优化时覆盖了对角线值

单数组更新过程中，`dp[i+1][j+1]` 很容易在原地更新时丢失。

必须在覆盖数组元素之前，用一个额外变量保存旧的对角线值。否则替换和字符匹配对应的状态会出错。

### 7. Python 递归深度限制

记忆化搜索虽然时间复杂度已经优化到 `O(mn)`，但仍然使用递归。

Python 默认允许的递归深度有限。当字符串很长时，可能出现：

```
RecursionError: maximum recursion depth exceeded
```

在代码面试和在线评测中，自底向上的动态规划通常更加稳定。

------

## 方法对比

| 方法           | 时间复杂度                    | 空间复杂度    | 特点                           |
| -------------- | ----------------------------- | ------------- | ------------------------------ |
| 普通递归       | 指数级，宽松上界 `O(3^(m+n))` | `O(m+n)`      | 容易理解，但存在大量重复计算   |
| 记忆化搜索     | `O(mn)`                       | `O(mn)`       | 代码直观，但受递归深度限制     |
| 二维动态规划   | `O(mn)`                       | `O(mn)`       | 状态最清晰，适合面试讲解       |
| 两行动态规划   | `O(mn)`                       | `O(min(m,n))` | 节省空间，逻辑仍较容易理解     |
| 单数组动态规划 | `O(mn)`                       | `O(min(m,n))` | 空间最优，但原地更新较容易出错 |

面试中通常建议先写出二维动态规划，清楚说明状态定义、边界条件和转移方程；如果面试官继续要求优化空间，再将其压缩为一维数组。