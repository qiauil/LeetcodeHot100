# 编辑距离（Edit Distance）

给定两个仅由小写英文字母组成的字符串 `word1` 和 `word2`。

你可以对 `word1` 执行任意次数的以下三种操作：

* 在任意位置插入一个字符
* 删除任意位置的一个字符
* 替换任意位置的一个字符

返回将 `word1` 转换成 `word2` 所需的最少操作次数。

---

## 核心思路与状态定义

设：

* `m = len(word1)`
* `n = len(word2)`
* `dfs(i, j)` 或 `dp[i][j]` 表示：将 `word1` 的前 `i` 个字符转换成 `word2` 的前 `j` 个字符所需的最少操作次数。

也就是：

```python
word1[:i] -> word2[:j]
```

需要的最少操作数。

注意：

```python
dp[i][j]
```

中的 `i` 和 `j` 表示的是“字符数量”，因此当前正在考虑的字符分别是：

```python
word1[i - 1]
word2[j - 1]
```

而不是 `word1[i]` 和 `word2[j]`。

例如：

```python
word1 = "horse"
word2 = "ros"
```

那么：

```python
dp[3][2]
```

表示：

```text
"hor" -> "ro"
```

所需要的最少操作次数。

---

### 字符相同

如果：

```python
word1[i - 1] == word2[j - 1]
```

说明两个前缀的最后一个字符已经相同，不需要执行任何操作。

因此只需要考虑前面的部分：

```python
dp[i][j] = dp[i - 1][j - 1]
```

也就是说：

```text
word1[:i]     -> word2[:j]
     ↑                  ↑
   相同字符            相同字符

问题缩小为：

word1[:i-1] -> word2[:j-1]
```

---

### 字符不同

如果：

```python
word1[i - 1] != word2[j - 1]
```

则可以选择三种操作。

### 1. 删除 `word1[i - 1]`

删除 `word1` 当前前缀的最后一个字符后：

```text
word1[:i]
    ↓ 删除最后一个字符
word1[:i-1]
```

此时还需要将：

```python
word1[:i - 1]
```

转换成：

```python
word2[:j]
```

所以对应状态为：

```python
dp[i - 1][j]
```

因此：

```python
delete_cost = dp[i - 1][j]
```

可以理解为：

```text
word1[:i] -> word2[:j]
     |
     | 删除 word1 最后一个字符
     ↓
word1[:i-1] -> word2[:j]
```

删除操作会消耗 `word1` 中的一个字符，因此：

```text
i 减 1
j 不变
```

---

### 2. 向 `word1` 插入 `word2[j - 1]`

如果选择插入操作，那么插入的字符会直接匹配：

```python
word2[j - 1]
```

也就是说，目标字符串的最后一个字符已经通过这次插入解决了。

因此在执行这次插入之前，只需要解决：

```python
word1[:i] -> word2[:j - 1]
```

所以对应状态为：

```python
dp[i][j - 1]
```

因此：

```python
insert_cost = dp[i][j - 1]
```

可以理解为：

```text
word1[:i] -> word2[:j-1]
                  |
                  | 插入 word2[j-1]
                  ↓
             word2[:j]
```

例如：

```text
"ab" -> "abc"
```

如果最后一步是插入 `'c'`，那么在插入 `'c'` 之前，只需要解决：

```text
"ab" -> "ab"
```

也就是目标字符串少一个字符。

因此插入操作：

```text
i 不变
j 减 1
```

可以记忆为：

> 插入会满足目标字符串中的一个字符。

---

### 3. 将 `word1[i - 1]` 替换成 `word2[j - 1]`

如果执行替换操作：

```text
word1[i - 1] -> word2[j - 1]
```

那么两个字符串当前的最后一个字符就都处理完了。

因此只需要继续处理：

```python
word1[:i - 1] -> word2[:j - 1]
```

对应：

```python
dp[i - 1][j - 1]
```

因此：

```python
replace_cost = dp[i - 1][j - 1]
```

替换操作会同时处理两个字符串中的一个字符，因此：

```text
i 减 1
j 减 1
```

---

因此状态转移方程为：

```python
if word1[i - 1] == word2[j - 1]:
    dp[i][j] = dp[i - 1][j - 1]
else:
    dp[i][j] = 1 + min(
        dp[i - 1][j],      # 删除
        dp[i][j - 1],      # 插入
        dp[i - 1][j - 1],  # 替换
    )
```

可以记忆为：

```text
删除：消耗 word1 一个字符
      (i - 1, j)

插入：满足 word2 一个字符
      (i, j - 1)

替换：两个字符同时处理
      (i - 1, j - 1)
```

---

# 1. 递归

## 思路

递归函数：

```python
dfs(i, j)
```

表示：

> 将 `word1` 的前 `i` 个字符转换成 `word2` 的前 `j` 个字符所需的最少操作次数。

也就是：

```python
word1[:i] -> word2[:j]
```

与常见的后缀递归不同，这里的递归是从完整字符串开始，不断缩短前缀：

```text
dfs(m, n)
   ↓
dfs(i - 1, j)
dfs(i, j - 1)
dfs(i - 1, j - 1)
```

---

## 边界情况

如果：

```python
i == 0
```

说明 `word1` 的前缀为空字符串：

```text
"" -> word2[:j]
```

此时只能通过插入得到 `word2[:j]`。

因此需要：

```python
j
```

次操作。

即：

```python
if i == 0:
    return j
```

---

如果：

```python
j == 0
```

说明目标字符串为空：

```text
word1[:i] -> ""
```

此时只能删除 `word1[:i]` 中的全部字符。

因此需要：

```python
i
```

次操作。

即：

```python
if j == 0:
    return i
```

---

## 算法步骤

1. 定义递归函数 `dfs(i, j)`。
2. 如果 `i == 0`，返回 `j`。
3. 如果 `j == 0`，返回 `i`。
4. 比较 `word1[i - 1]` 和 `word2[j - 1]`。
5. 如果两个字符相同，返回 `dfs(i - 1, j - 1)`。
6. 如果不同，分别尝试删除、插入和替换。
7. 取三者中的最小值，再加上当前操作代价 `1`。
8. 从 `dfs(m, n)` 开始递归。

---

## 代码

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        def dfs(i: int, j: int) -> int:
            # word1 前缀为空，只能插入 word2 的前 j 个字符
            if i == 0:
                return j

            # word2 前缀为空，只能删除 word1 的前 i 个字符
            if j == 0:
                return i

            # 当前最后一个字符相同，不需要执行操作
            if word1[i - 1] == word2[j - 1]:
                return dfs(i - 1, j - 1)

            # 删除 word1[i - 1]
            delete_cost = dfs(i - 1, j)

            # 插入 word2[j - 1]
            insert_cost = dfs(i, j - 1)

            # 将 word1[i - 1] 替换成 word2[j - 1]
            replace_cost = dfs(i - 1, j - 1)

            return 1 + min(
                delete_cost,
                insert_cost,
                replace_cost,
            )

        return dfs(m, n)
```

---

## 复杂度分析

* 时间复杂度：指数级，宽松上界为 `O(3^(m+n))`
* 空间复杂度：`O(m+n)`

递归过程中，同一个 `(i, j)` 状态可能被重复计算很多次，因此普通递归效率很低。

---

# 2. 自顶向下动态规划（记忆化搜索）

## 思路

普通递归会重复计算大量相同状态。

由于一个子问题完全由：

```python
(i, j)
```

决定，因此可以用：

```python
memo[(i, j)]
```

保存结果。

这里：

```python
memo[(i, j)]
```

表示：

> 将 `word1[:i]` 转换成 `word2[:j]` 所需要的最少操作次数。

---

## 算法步骤

1. 定义字典 `memo`。
2. 如果 `i == 0`，返回 `j`。
3. 如果 `j == 0`，返回 `i`。
4. 如果 `(i, j)` 已经计算过，直接返回缓存值。
5. 如果当前最后两个字符相同，处理 `(i - 1, j - 1)`。
6. 如果不同，尝试删除、插入和替换。
7. 保存计算结果。
8. 返回 `dfs(m, n)`。

---

## 代码

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        memo = {}

        def dfs(i: int, j: int) -> int:
            if i == 0:
                return j

            if j == 0:
                return i

            if (i, j) in memo:
                return memo[(i, j)]

            if word1[i - 1] == word2[j - 1]:
                memo[(i, j)] = dfs(i - 1, j - 1)

            else:
                delete_cost = dfs(i - 1, j)
                insert_cost = dfs(i, j - 1)
                replace_cost = dfs(i - 1, j - 1)

                memo[(i, j)] = 1 + min(
                    delete_cost,
                    insert_cost,
                    replace_cost,
                )

            return memo[(i, j)]

        return dfs(m, n)
```

---

也可以使用 Python 的 `functools.cache`：

```python
from functools import cache


class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        @cache
        def dfs(i: int, j: int) -> int:
            if i == 0:
                return j

            if j == 0:
                return i

            if word1[i - 1] == word2[j - 1]:
                return dfs(i - 1, j - 1)

            return 1 + min(
                dfs(i - 1, j),      # 删除
                dfs(i, j - 1),      # 插入
                dfs(i - 1, j - 1),  # 替换
            )

        return dfs(m, n)
```

---

## 复杂度分析

* 时间复杂度：`O(mn)`
* 空间复杂度：`O(mn)`

一共有最多：

```text
(m + 1) × (n + 1)
```

个不同状态。

每个状态只会计算一次。

---

# 3. 自底向上动态规划

## 思路

定义：

```python
dp[i][j]
```

表示：

> 将 `word1` 的前 `i` 个字符转换成 `word2` 的前 `j` 个字符所需的最少操作次数。

也就是：

```python
word1[:i] -> word2[:j]
```

由于：

```python
dp[i][j]
```

依赖：

```python
dp[i - 1][j]
dp[i][j - 1]
dp[i - 1][j - 1]
```

因此需要从左上角向右下角填表。

可以把依赖关系画成：

```text
                 j-1           j
            +-----------+-----------+
      i-1   |  replace  |  delete   |
            |   match   |           |
            +-----------+-----------+
       i    |  insert   | dp[i][j]  |
            +-----------+-----------+
```

计算 `dp[i][j]` 时：

* 上方已经计算完成
* 左方已经计算完成
* 左上方也已经计算完成

因此遍历顺序是：

```text
左上 -> 右下
```

---

## 边界初始化

二维数组大小为：

```text
(m + 1) × (n + 1)
```

这里多出的一行和一列用于表示空字符串。

---

如果 `word2` 为空：

```python
dp[i][0]
```

表示：

```text
word1[:i] -> ""
```

只能删除 `word1` 的全部 `i` 个字符。

因此：

```python
dp[i][0] = i
```

---

如果 `word1` 为空：

```python
dp[0][j]
```

表示：

```text
"" -> word2[:j]
```

只能插入 `word2` 的全部 `j` 个字符。

因此：

```python
dp[0][j] = j
```

---

初始 DP 表类似：

```text
             ""    r    o    s
          +----+----+----+----+
       "" |  0 |  1 |  2 |  3 |
          +----+----+----+----+
        h |  1 |    |    |    |
          +----+----+----+----+
        o |  2 |    |    |    |
          +----+----+----+----+
        r |  3 |    |    |    |
          +----+----+----+----+
        s |  4 |    |    |    |
          +----+----+----+----+
        e |  5 |    |    |    |
          +----+----+----+----+
```

第一列表示：

```text
word1[:i] -> ""
```

第一行表示：

```text
"" -> word2[:j]
```

---

## 算法步骤

1. 创建 `(m + 1) × (n + 1)` 的二维数组。
2. 初始化第一列：

```python
dp[i][0] = i
```

3. 初始化第一行：

```python
dp[0][j] = j
```

4. 从 `i = 1`、`j = 1` 开始从左上向右下遍历。
5. 比较：

```python
word1[i - 1]
word2[j - 1]
```

6. 如果相同：

```python
dp[i][j] = dp[i - 1][j - 1]
```

7. 如果不同：

```python
dp[i][j] = 1 + min(
    dp[i - 1][j],
    dp[i][j - 1],
    dp[i - 1][j - 1],
)
```

8. 最终返回：

```python
dp[m][n]
```

---

## 代码

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # dp[i][j] 表示：
        # word1 的前 i 个字符转换成
        # word2 的前 j 个字符需要的最少操作数
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        # word2 为空：
        # word1 的前 i 个字符全部删除
        for i in range(m + 1):
            dp[i][0] = i

        # word1 为空：
        # 插入 word2 的前 j 个字符
        for j in range(n + 1):
            dp[0][j] = j

        # 从左上角向右下角填表
        for i in range(1, m + 1):
            for j in range(1, n + 1):

                if word1[i - 1] == word2[j - 1]:
                    # 当前最后一个字符相同，不需要额外操作
                    dp[i][j] = dp[i - 1][j - 1]

                else:
                    # 删除 word1 当前最后一个字符
                    delete_cost = dp[i - 1][j]

                    # 插入 word2 当前最后一个字符
                    insert_cost = dp[i][j - 1]

                    # 替换当前最后一个字符
                    replace_cost = dp[i - 1][j - 1]

                    dp[i][j] = 1 + min(
                        delete_cost,
                        insert_cost,
                        replace_cost,
                    )

        return dp[m][n]
```

---

## Python 二维列表说明

正确写法：

```python
dp = [[0] * (n + 1) for _ in range(m + 1)]
```

不要写成：

```python
dp = [[0] * (n + 1)] * (m + 1)
```

后一种写法会让所有行引用同一个列表。

---

## 复杂度分析

* 时间复杂度：`O(mn)`
* 空间复杂度：`O(mn)`

一共需要计算大约 `m × n` 个主要状态，每个状态只进行常数次操作。

---

# 4. 动态规划：两行空间优化

## 思路

二维 DP 中：

```python
dp[i][j]
```

只依赖：

```python
dp[i - 1][j]      # 上方
dp[i][j - 1]      # 左方
dp[i - 1][j - 1]  # 左上方
```

因此计算第 `i` 行时，只需要保留：

* `previous_row`：上一行，即原来的第 `i - 1` 行
* `current_row`：当前正在计算的第 `i` 行

不需要保留整个二维数组。

---

## 为什么可以交换两个字符串

编辑距离具有对称性：

```text
distance(word1, word2)
=
distance(word2, word1)
```

从 `word1` 到 `word2` 的插入，对反方向来说就是删除。

替换操作本身也是对称的。

因此可以让 `word2` 成为较短字符串，使数组长度为：

```text
min(m, n) + 1
```

---

## 代码

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # 让 word2 成为较短字符串
        if m < n:
            word1, word2 = word2, word1
            m, n = n, m

        # dp[0][j] = j
        # 空字符串变成 word2[:j]，需要插入 j 次
        previous_row = list(range(n + 1))

        for i in range(1, m + 1):
            current_row = [0] * (n + 1)

            # dp[i][0] = i
            current_row[0] = i

            for j in range(1, n + 1):

                if word1[i - 1] == word2[j - 1]:
                    current_row[j] = previous_row[j - 1]

                else:
                    delete_cost = previous_row[j]
                    insert_cost = current_row[j - 1]
                    replace_cost = previous_row[j - 1]

                    current_row[j] = 1 + min(
                        delete_cost,
                        insert_cost,
                        replace_cost,
                    )

            previous_row = current_row

        return previous_row[n]
```

---

## 状态对应关系

二维版本中：

```text
previous_row[j]
=
dp[i - 1][j]
```

表示删除。

```text
current_row[j - 1]
=
dp[i][j - 1]
```

表示插入。

```text
previous_row[j - 1]
=
dp[i - 1][j - 1]
```

表示替换或字符匹配。

---

## 复杂度分析

* 时间复杂度：`O(mn)`
* 空间复杂度：`O(min(m,n))`

---

# 5. 动态规划：单数组空间优化

## 思路

两行 DP 还可以继续压缩为一个数组。

假设当前正在计算第 `i` 行。

更新 `dp[j]` 之前：

```python
dp[j]
```

仍然保存上一行的值：

```text
dp[i - 1][j]
```

也就是删除操作需要的状态。

---

更新：

```python
dp[j - 1]
```

之后，它已经变成当前行的值：

```text
dp[i][j - 1]
```

也就是插入操作需要的状态。

---

还有一个状态：

```text
dp[i - 1][j - 1]
```

也就是左上角。

但原来的 `dp[j - 1]` 已经被覆盖，因此需要额外使用变量：

```python
diagonal
```

保存旧的左上角值。

---

## 单数组更新过程

计算：

```python
dp[i][j]
```

时：

```text
dp[j]
```

更新前表示：

```text
dp[i - 1][j]
```

即删除。

---

```text
dp[j - 1]
```

更新后表示：

```text
dp[i][j - 1]
```

即插入。

---

```text
diagonal
```

表示：

```text
dp[i - 1][j - 1]
```

即替换或字符匹配。

---

每次覆盖：

```python
dp[j]
```

之前，需要：

```python
old_dp_j = dp[j]
```

保存旧值。

完成当前位置计算后：

```python
diagonal = old_dp_j
```

这样下一轮 `j + 1` 时，`diagonal` 就正好表示新的左上角。

---

## 代码

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # 让 dp 数组基于较短字符串
        if m < n:
            word1, word2 = word2, word1
            m, n = n, m

        # dp[0][j] = j
        dp = list(range(n + 1))

        for i in range(1, m + 1):

            # 进入新的一行之前：
            # dp[0] 旧值 = dp[i-1][0]
            # 它正是第一个位置需要的左上角
            diagonal = dp[0]

            # dp[i][0] = i
            dp[0] = i

            for j in range(1, n + 1):

                # 保存旧的 dp[i-1][j]
                old_dp_j = dp[j]

                if word1[i - 1] == word2[j - 1]:
                    # 左上角
                    dp[j] = diagonal

                else:
                    dp[j] = 1 + min(
                        dp[j],      # 删除：旧 dp[i-1][j]
                        dp[j - 1],  # 插入：新 dp[i][j-1]
                        diagonal,   # 替换：旧 dp[i-1][j-1]
                    )

                # 为下一列保存新的左上角
                diagonal = old_dp_j

        return dp[n]
```

---

## 复杂度分析

* 时间复杂度：`O(mn)`
* 空间复杂度：`O(min(m,n))`

---

# 示例分析

假设：

```python
word1 = "horse"
word2 = "ros"
```

最终要求的是：

```python
dp[5][3]
```

因为：

```text
len("horse") = 5
len("ros") = 3
```

也就是：

```text
"horse" -> "ros"
```

一种最优操作过程：

```text
horse
rorse   将 h 替换为 r
rose    删除第二个 r
ros     删除 e
```

总共 `3` 次操作。

所以：

```python
dp[5][3] = 3
```

---

# 常见错误

## 1. 混淆 `i`、`j` 和字符串下标

这里：

```python
dp[i][j]
```

表示的是：

```text
前 i 个字符
前 j 个字符
```

所以当前字符是：

```python
word1[i - 1]
word2[j - 1]
```

而不是：

```python
word1[i]
word2[j]
```

这是前缀 DP 中最常见的下标错误。

---

## 2. 混淆插入和删除对应的状态

删除：

```python
dp[i - 1][j]
```

因为删除会消耗 `word1` 中的一个字符。

---

插入：

```python
dp[i][j - 1]
```

因为插入会满足 `word2` 中的一个字符。

可以记忆为：

> 删除消耗源字符串，插入满足目标字符串。

---

## 3. 边界初始化错误

必须有：

```python
dp[i][0] = i
```

因为：

```text
word1[:i] -> ""
```

需要删除 `i` 个字符。

同时：

```python
dp[0][j] = j
```

因为：

```text
"" -> word2[:j]
```

需要插入 `j` 个字符。

---

## 4. 字符相同时仍然加 `1`

如果：

```python
word1[i - 1] == word2[j - 1]
```

两个当前字符已经相同，不需要任何操作。

所以：

```python
dp[i][j] = dp[i - 1][j - 1]
```

不能写成：

```python
1 + dp[i - 1][j - 1]
```

---

## 5. 忘记当前操作的代价

如果字符不同：

```python
dp[i - 1][j]
dp[i][j - 1]
dp[i - 1][j - 1]
```

只是
