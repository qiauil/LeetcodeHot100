# 最长递增子序列（Longest Increasing Subsequence, LIS）

给定一个整数数组 `nums`，返回其中**最长严格递增子序列**的长度。

**子序列（subsequence）\**是指：从原序列中删除任意数量（也可以一个都不删除）的元素，并且\**不改变剩余元素之间的相对顺序**，得到的新序列。

例如：

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]

一个最长递增子序列可以是：
[2, 3, 7, 101]

长度为 4。
```

注意这里要求的是**严格递增**：

```
[1, 2, 2, 3]
```

不能把两个 `2` 同时放进严格递增子序列中。

------

# 1. 递归：枚举选与不选

## 核心思路

对于数组中的每一个位置 `i`，我们实际上只有两个选择：

```
1. 不选择 nums[i]
2. 选择 nums[i]
```

但是第二种选择有一个前提：

```
nums[i] > 上一个被选择的元素
```

因此，我们可以通过 DFS 枚举所有可能的子序列。

定义：

```
dfs(i, j)
```

表示：

- `i`：当前正在考虑的位置；
- `j`：上一个被加入递增子序列的元素下标；
- 如果当前还没有选择任何元素，则令 `j = -1`。

------

## 状态转移

对于 `nums[i]`：

### 情况一：不选择

```
dfs(i + 1, j)
```

上一个选择的元素仍然是 `j`。

### 情况二：选择

只有满足：

```
j == -1 or nums[j] < nums[i]
```

才能选择 `nums[i]`。

此时：

```
1 + dfs(i + 1, i)
```

因为选择了当前元素，所以子序列长度增加 `1`，同时最后一个被选择的位置变成 `i`。

最终取两种选择的最大值。

------

## 代码

```
from typing import List


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:

        def dfs(i: int, j: int) -> int:
            # 所有元素都已经考虑完
            if i == len(nums):
                return 0

            # 情况 1：不选择 nums[i]
            lis = dfs(i + 1, j)

            # 情况 2：选择 nums[i]
            # j == -1 表示目前还没有选择任何元素
            if j == -1 or nums[j] < nums[i]:
                lis = max(
                    lis,
                    1 + dfs(i + 1, i)
                )

            return lis

        return dfs(0, -1)
```

------

## 复杂度

**时间复杂度：**

\[ O(2^n) \]

因为每个元素基本都有「选择 / 不选择」两个分支。

**空间复杂度：**

\[ O(n) \]

主要来自递归调用栈。

> 原题给出的 `O(2n)` 应当是排版错误，正确的是 `O(2^n)`。

------

# 2. 自顶向下动态规划：二维状态

## 核心思路

上一种递归存在大量重复计算。

例如：

```
dfs(i, j)
```

可能从不同的递归路径重复到达。

因此，可以把已经计算过的：

```
(i, j)
```

对应答案保存下来，也就是**记忆化搜索（Memoization）**。

这是从递归自然过渡到动态规划的一种常见方式：

```
暴力递归
    ↓
发现重复子问题
    ↓
加入缓存
    ↓
Top-Down Dynamic Programming
```

------

## 为什么 `memo` 需要 `n + 1` 列？

状态中的：

```
j
```

可能取：

```
-1, 0, 1, 2, ..., n-1
```

总共 `n + 1` 种可能。

但是数组不能使用 `-1` 表示一个独立的 DP 状态，因为：

```
memo[i][-1]
```

在 Python 中表示数组的最后一个元素。

所以统一使用：

```
j + 1
```

进行映射：

```
j = -1  → 0
j = 0   → 1
j = 1   → 2
...
j = n-1 → n
```

------

## 代码

```
from typing import List


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)

        # memo[i][j + 1] 保存 dfs(i, j) 的结果
        memo = [[-1] * (n + 1) for _ in range(n)]

        def dfs(i: int, j: int) -> int:
            # 所有元素已经处理完
            if i == n:
                return 0

            # 如果已经计算过，直接返回
            if memo[i][j + 1] != -1:
                return memo[i][j + 1]

            # 情况 1：不选择 nums[i]
            lis = dfs(i + 1, j)

            # 情况 2：选择 nums[i]
            if j == -1 or nums[j] < nums[i]:
                lis = max(
                    lis,
                    1 + dfs(i + 1, i)
                )

            # 缓存答案
            memo[i][j + 1] = lis

            return lis

        return dfs(0, -1)
```

------

## 复杂度

状态数量大约为：

```
n × n
```

每个状态只会真正计算一次。

因此：

**时间复杂度：**

\[ O(n^2) \]

**空间复杂度：**

\[ O(n^2) \]

另外递归调用栈需要：

\[ O(n) \]

不过总体仍然是：

\[ O(n^2) \]

------

# 3. 自顶向下动态规划：一维状态

二维状态虽然很好理解，但实际上还可以进一步简化。

## 核心思路

重新定义：

```
dfs(i)
```

表示：

> **以 `nums[i]` 作为第一个元素的最长递增子序列长度。**

如果已经选择了 `nums[i]`，那么接下来我们只需要寻找：

```
j > i
```

并且：

```
nums[j] > nums[i]
```

的元素。

例如：

```
nums = [2, 5, 3, 7]
        ↑
        i
```

从 `2` 开始：

```
2 → 5 → 7
2 → 3 → 7
```

因此：

```
dfs(i) = 1 + max(dfs(j))
```

其中必须满足：

```
j > i and nums[j] > nums[i]
```

------

## 状态转移

默认情况下，每个元素自己就能构成长度为 `1` 的递增子序列：

```
lis = 1
```

然后枚举右边所有位置：

```
for j in range(i + 1, n):
```

如果：

```
nums[i] < nums[j]
```

就可以尝试：

```
1 + dfs(j)
```

------

## 代码

```
from typing import List


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)

        if n == 0:
            return 0

        # memo[i]：
        # 以 nums[i] 开始的最长递增子序列长度
        memo = [-1] * n

        def dfs(i: int) -> int:
            # 已经计算过
            if memo[i] != -1:
                return memo[i]

            # nums[i] 自己至少可以组成长度为 1 的子序列
            lis = 1

            # 尝试寻找下一个更大的元素
            for j in range(i + 1, n):
                if nums[i] < nums[j]:
                    lis = max(
                        lis,
                        1 + dfs(j)
                    )

            memo[i] = lis
            return lis

        # LIS 不一定从 nums[0] 开始
        return max(dfs(i) for i in range(n))
```

------

## 一个非常重要的理解

为什么最后是：

```
max(dfs(i) for i in range(n))
```

而不是：

```
dfs(0)
```

因为最长递增子序列**不一定从第一个元素开始**。

例如：

```
[100, 1, 2, 3, 4]
```

如果强制从 `100` 开始，答案只有：

```
[100]
```

长度 `1`。

但真正的 LIS 是：

```
[1, 2, 3, 4]
```

长度 `4`。

------

## 复杂度

每一个 `i` 都可能枚举后面所有的 `j`：

\[ O(n^2) \]

因此：

**时间复杂度：**

\[ O(n^2) \]

**空间复杂度：**

\[ O(n) \]

------

# 4. 自底向上动态规划：二维 DP

这一方法本质上是第 2 种方法的迭代版本。

## DP 状态

可以理解：

```
dp[i][j + 1]
```

表示：

> 当前考虑 `nums[i:]`，并且之前最后一个选择的元素位置是 `j` 时，能够得到的最长递增子序列长度。

由于：

```
dp[i]
```

依赖：

```
dp[i + 1]
```

所以必须：

```
从右向左
```

进行计算。

------

## 状态转移

不选择 `nums[i]`：

```
dp[i + 1][j + 1]
```

选择 `nums[i]`：

```
1 + dp[i + 1][i + 1]
```

当然，只有满足：

```
j == -1 or nums[j] < nums[i]
```

时才能选择。

因此：

```
dp[i][j + 1] = max(
    不选择,
    选择
)
```

------

## 代码

```
from typing import List


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)

        # 多增加一行表示 i == n 的 base case
        dp = [[0] * (n + 1) for _ in range(n + 1)]

        # 从右往左计算
        for i in range(n - 1, -1, -1):

            # j 只可能出现在 i 之前
            # 同时需要考虑 j == -1
            for j in range(i - 1, -2, -1):

                # 情况 1：不选择 nums[i]
                lis = dp[i + 1][j + 1]

                # 情况 2：选择 nums[i]
                if j == -1 or nums[j] < nums[i]:
                    lis = max(
                        lis,
                        1 + dp[i + 1][i + 1]
                    )

                dp[i][j + 1] = lis

        # 对应初始状态 dfs(0, -1)
        return dp[0][0]
```

------

## 复杂度

**时间复杂度：**

\[ O(n^2) \]

**空间复杂度：**

\[ O(n^2) \]

这种写法虽然可以帮助理解「递归 → DP」的转换，但实际面试中通常不会优先使用，因为存在更简单的一维 DP。

------

# 5. 自底向上动态规划：一维 DP

这是最经典、也最适合面试讲解的 LIS 动态规划解法之一。

## DP 定义

定义：

```
lis[i]
```

表示：

> **以 `nums[i]` 开始的最长递增子序列长度。**

每一个元素自己至少能够组成一个长度为 `1` 的递增子序列：

```
lis = [1] * n
```

如果：

```
nums[i] < nums[j]
```

那么：

```
nums[i] → nums[j] → ...
```

可以组成新的递增子序列。

因此：

```
lis[i] = max(
    lis[i],
    1 + lis[j]
)
```

因为 `lis[i]` 依赖右边的 `lis[j]`，所以我们从右向左计算。

------

## 代码

```
from typing import List


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        n = len(nums)

        if n == 0:
            return 0

        # lis[i]：
        # 以 nums[i] 开始的最长递增子序列长度
        lis = [1] * n

        # 从右往左
        for i in range(n - 1, -1, -1):

            # 枚举所有可能接在 nums[i] 后面的元素
            for j in range(i + 1, n):

                # 严格递增
                if nums[i] < nums[j]:
                    lis[i] = max(
                        lis[i],
                        1 + lis[j]
                    )

        # LIS 可以从任意位置开始
        return max(lis)
```

------

## 示例

对于：

```
nums = [1, 4, 2, 3]
```

从右向左：

```
3:
lis = [1, 1, 1, 1]

2:
2 < 3
lis = [1, 1, 2, 1]

4:
右边没有比 4 更大的
lis = [1, 1, 2, 1]

1:
1 → 4      长度 2
1 → 2 → 3  长度 3

lis = [3, 1, 2, 1]
```

最终：

```
max(lis) == 3
```

------

## 另一种同样常见的 DP 定义

面试中其实更常看到：

```
dp[i]
```

定义成：

> **以 `nums[i]` 结尾的最长递增子序列长度。**

这样就从左向右计算：

```
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0

        n = len(nums)

        # dp[i]：
        # 以 nums[i] 结尾的 LIS 长度
        dp = [1] * n

        for i in range(n):
            for j in range(i):
                if nums[j] < nums[i]:
                    dp[i] = max(
                        dp[i],
                        dp[j] + 1
                    )

        return max(dp)
```

这两种定义本质完全一样：

```
以 i 开始 → 从右往左
以 i 结尾 → 从左往右
```

面试时选择自己最容易解释的一种即可。

------

## 复杂度

**时间复杂度：**

\[ O(n^2) \]

**空间复杂度：**

\[ O(n) \]

------

# 6. 线段树（Segment Tree）

## 核心思路

前面的 `O(n²)` DP 中有一个操作非常耗时。

如果定义：

```
dp[i] = 以 nums[i] 结尾的 LIS 长度
```

那么我们其实是在计算：

\[ dp[i] = 1 + \max(dp[j]) \]

其中：

\[ nums[j] < nums[i] \]

换句话说，每看到一个数字 `x`，我们想快速回答：

> 所有比 `x` 小的数字中，最大的 LIS 长度是多少？

如果可以快速进行：

```
区间最大值查询
```

就能加速这一过程。

线段树正好适合：

```
Range Maximum Query
+
Point Update
```

即：

```
区间最大值查询 + 单点更新
```

------

# 6.1 为什么需要坐标压缩？

数组中的值可能非常大：

```
[-1000000000, 8, 1000000000]
```

显然不能建立一个大小为：

```
2,000,000,001
```

的线段树。

因此先把数值映射到相对排名：

```
原数组：
[100, 20, 50]

排序：
[20, 50, 100]

压缩：
100 → 2
20  → 0
50  → 1

得到：
[2, 0, 1]
```

我们不关心数字真正是多少，只关心：

```
谁比谁大
```

------

# 6.2 `bisect_left`

Python 标准库提供：

```
from bisect import bisect_left
```

调用：

```
bisect_left(arr, x)
```

返回：

> 在有序数组 `arr` 中，第一个 `>= x` 的位置。

例如：

```
arr = [1, 3, 5, 7]

bisect_left(arr, 5)
# 2

bisect_left(arr, 4)
# 2
```

因为位置 `2` 是：

```
第一个 >= 4 的元素
```

即 `5`。

在这里，我们使用它得到每个数字在去重排序数组中的排名。

------

# 6.3 代码

```
from typing import List
from bisect import bisect_left


class SegmentTree:
    def __init__(self, N: int):
        self.n = N

        # 将叶子节点数量扩展到 2 的幂
        # 例如：
        # N = 5 → self.n = 8
        while self.n & (self.n - 1):
            self.n += 1

        # 下标 1 为根节点
        self.tree = [0] * (2 * self.n)

    def update(self, i: int, val: int) -> None:
        """
        更新位置 i 的最大 LIS 长度。
        """

        # 叶子节点的位置
        i += self.n

        # 使用 max 更稳妥：
        # 相同数字可能多次出现
        self.tree[i] = max(self.tree[i], val)

        # 从叶子向上更新父节点
        i >>= 1

        while i >= 1:
            self.tree[i] = max(
                self.tree[i << 1],        # 左孩子
                self.tree[i << 1 | 1]     # 右孩子
            )

            i >>= 1

    def query(self, left: int, right: int) -> int:
        """
        查询闭区间 [left, right] 中的最大值。
        """

        if left > right:
            return 0

        result = 0

        # 转换到叶子区域
        left += self.n
        right += self.n + 1

        # 此处实际使用的是 [left, right) 半开区间
        while left < right:

            # left 是右孩子：
            # 当前节点需要被单独计算
            if left & 1:
                result = max(result, self.tree[left])
                left += 1

            # right 是右孩子：
            # right - 1 需要被单独计算
            if right & 1:
                right -= 1
                result = max(result, self.tree[right])

            left >>= 1
            right >>= 1

        return result


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0

        # ---------- 坐标压缩 ----------
        sorted_nums = sorted(set(nums))

        compressed = [
            bisect_left(sorted_nums, num)
            for num in nums
        ]

        # 实际只需要不同数字的数量
        seg_tree = SegmentTree(len(sorted_nums))

        answer = 0

        for num in compressed:

            # 查询所有严格小于当前数字的位置：
            #
            # [0, num - 1]
            #
            # 注意不能包含 num 本身，
            # 因为题目要求 strictly increasing
            best_before = seg_tree.query(0, num - 1)

            cur_lis = best_before + 1

            # 当前数字作为结尾时的最大 LIS
            seg_tree.update(num, cur_lis)

            answer = max(answer, cur_lis)

        return answer
```

------

## 关于位运算

线段树实现中出现了几种常见位运算。

### `x << 1`

相当于：

```
x * 2
```

在线段树中：

```
i << 1
```

表示左孩子：

```
2i
```

------

### `x << 1 | 1`

相当于：

```
2 * x + 1
```

表示右孩子：

```
2i + 1
```

------

### `x >> 1`

相当于：

```
x // 2
```

表示父节点。

------

### `x & 1`

用于判断奇偶：

```
x & 1 == 1
```

说明 `x` 是奇数。

------

### `x & (x - 1)`

这是一个很常见的技巧：

```
x & (x - 1) == 0
```

当且仅当正整数 `x` 是：

```
2 的幂
```

例如：

```
1  = 2^0
2  = 2^1
4  = 2^2
8  = 2^3
16 = 2^4
```

------

## 复杂度

坐标压缩排序：

\[ O(n\log n) \]

每个元素进行：

- 一次查询：`O(log n)`
- 一次更新：`O(log n)`

因此总时间复杂度：

\[ O(n\log n) \]

空间复杂度：

\[ O(n) \]

------

# 7. 动态规划 + 二分查找

这是这道题中最重要的优化解法之一，通常也是面试中的最佳答案。

时间复杂度：

\[ O(n\log n) \]

而且实现比线段树简单得多。

------

## 核心思想

维护一个数组：

```
dp
```

其中：

```
dp[i]
```

表示：

> 所有长度为 `i + 1` 的递增子序列中，**最小的结尾元素**。

例如：

```
dp = [2, 3, 7]
```

含义是：

```
长度 1 的递增子序列，最优结尾是 2
长度 2 的递增子序列，最优结尾是 3
长度 3 的递增子序列，最优结尾是 7
```

这里所谓「最优」，就是：

```
结尾越小越好
```

因为较小的结尾更容易被之后的数字继续扩展。

------

# 为什么结尾越小越好？

假设现在有两个长度同为 `3` 的递增子序列：

```
[1, 5, 10]
[2, 4, 6]
```

它们的结尾分别是：

```
10
6
```

如果以后来了：

```
7
```

那么：

```
10 → 7  ×
6  → 7  ✓
```

所以对于相同长度的递增子序列，我们只需要保留：

```
最小的尾部元素
```

------

# 处理每一个数字

假设当前：

```
dp = [2, 5, 8]
```

来了：

```
10
```

因为：

```
10 > dp[-1]
```

可以直接扩展：

```
dp = [2, 5, 8, 10]
```

LIS 长度增加。

------

但如果来了：

```
6
```

不能扩展：

```
2 < 5 < 8 < 6
```

不成立。

但是可以把：

```
8
```

替换成：

```
6
```

得到：

```
dp = [2, 5, 6]
```

长度没有变化，但尾部更小，更有利于以后扩展。

------

## 为什么使用 `bisect_left`？

我们需要找到：

> `dp` 中第一个 `>= 当前数字` 的位置。

这正是：

```
bisect_left(dp, num)
```

的作用。

例如：

```
dp = [2, 5, 8]

num = 6

idx = bisect_left(dp, 6)
# idx = 2
```

于是：

```
dp[2] = 6
```

得到：

```
[2, 5, 6]
```

------

## 代码

```
from typing import List
from bisect import bisect_left


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        # dp[i] 表示：
        # 长度为 i + 1 的递增子序列中，
        # 最小可能的结尾元素
        dp = []

        for num in nums:

            # 找到第一个 >= num 的位置
            idx = bisect_left(dp, num)

            # num 比 dp 中所有元素都大：
            # 可以扩展当前最长递增子序列
            if idx == len(dp):
                dp.append(num)

            # 否则替换掉当前位置，
            # 让相同长度的子序列拥有更小的尾部
            else:
                dp[idx] = num

        return len(dp)
```

相比原答案，这个版本不需要单独维护：

```
LIS
```

因为始终有：

```
len(dp) == 当前 LIS 长度
```

代码会更加简洁。

------

# 示例：完整模拟二分查找算法

考虑：

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]
```

开始：

```
dp = []
```

处理 `10`：

```
dp = [10]
```

处理 `9`：

找到第一个：

```
>= 9
```

的位置，即 `10`：

```
dp = [9]
```

处理 `2`：

```
dp = [2]
```

处理 `5`：

```
5 > 2
```

因此：

```
dp = [2, 5]
```

处理 `3`：

用 `3` 替换 `5`：

```
dp = [2, 3]
```

处理 `7`：

```
dp = [2, 3, 7]
```

处理 `101`：

```
dp = [2, 3, 7, 101]
```

处理 `18`：

用 `18` 替换 `101`：

```
dp = [2, 3, 7, 18]
```

所以：

```
len(dp) == 4
```

最终答案：

```
4
```

------

# 一个非常重要的陷阱：`dp` 不一定是真正的 LIS

在二分查找方法中：

```
dp
```

主要是一个**辅助状态数组**。

它表示：

> 每一种长度的递增子序列能够达到的最小尾部。

它的长度一定等于 LIS 长度，但其中的元素**不保证就是原数组中的某一条真实最长递增子序列**。

例如：

```
nums = [4, 10, 4, 3, 8, 9]
```

算法可能产生：

```
dp = [3, 8, 9]
```

它的长度：

```
3
```

是正确答案。

但理解这个算法时，不应该把 `dp` 简单定义为：

```
当前已经找到的 LIS
```

更准确的定义始终是：

> `dp[i]` = 长度为 `i + 1` 的递增子序列中，最小的结尾值。

这是理解该算法最关键的一点。

------

# 为什么必须使用 `bisect_left`？

严格递增要求：

```
<
```

而不是：

```
<=
```

所以当出现重复数字时：

```
[2, 2, 2]
```

答案必须是：

```
1
```

使用：

```
bisect_left(dp, num)
```

会寻找：

```
第一个 >= num 的位置
```

因此：

```
2 → [2]
2 → 替换第一个 2 → [2]
2 → 替换第一个 2 → [2]
```

最终长度仍然是 `1`。

如果错误地寻找：

```
第一个 > num 的位置
```

那么重复数字可能会被当成新的递增长度：

```
[2, 2, 2]
```

导致错误结果。

------

# 8. 各种方法的关系

这道题非常适合用来理解「算法逐步优化」的过程：

| 方法            | 核心思想          | 时间复杂度   | 空间复杂度 |
| --------------- | ----------------- | ------------ | ---------- |
| 暴力递归        | 每个元素选 / 不选 | `O(2^n)`     | `O(n)`     |
| Top-Down DP I   | 缓存 `(i, j)`     | `O(n²)`      | `O(n²)`    |
| Top-Down DP II  | `dfs(i)`          | `O(n²)`      | `O(n)`     |
| Bottom-Up DP I  | 二维 DP           | `O(n²)`      | `O(n²)`    |
| Bottom-Up DP II | 一维 DP           | `O(n²)`      | `O(n)`     |
| 线段树          | 区间最大值查询    | `O(n log n)` | `O(n)`     |
| DP + 二分查找   | 最小尾部          | `O(n log n)` | `O(n)`     |

其中面试最值得掌握的是：

```
① O(n²) 一维动态规划
② O(n log n) 二分查找
```

前者非常容易推导和解释，后者是标准最优解。

------

# 9. 常见错误

## 错误一：把子数组和子序列混淆

**子数组（subarray）**要求连续：

```
[1, 2, 3]
```

必须在原数组中连续出现。

**子序列（subsequence）**不要求连续，只要求保持相对顺序。

例如：

```
nums = [1, 100, 2, 100, 3]
```

可以得到子序列：

```
[1, 2, 3]
```

虽然它们在原数组中并不连续。

------

## 错误二：忘记是「严格」递增

题目要求：

```
nums[j] < nums[i]
```

不能使用：

```
nums[j] <= nums[i]
```

例如：

```
[1, 2, 2, 3]
```

最长严格递增子序列长度为：

```
3
```

而不是 `4`。

------

## 错误三：二分查找使用错误边界

严格递增 LIS 应寻找：

```
第一个 >= num 的位置
```

Python 中就是：

```
bisect_left(dp, num)
```

而不是寻找第一个严格大于 `num` 的位置。

------

## 错误四：错误理解二分算法中的 `dp`

不要理解成：

```
dp = 当前实际 LIS
```

而应该理解为：

```
dp[i]
=
长度为 i + 1 的递增子序列中
最小可能的结尾值
```

因此真正重要的是：

```
len(dp)
```

而不是 `dp` 本身。

------

## 错误五：DP 最终只返回最后一个状态

对于经典 `O(n²)` DP：

```
dp[i] = 以 nums[i] 结尾的 LIS 长度
```

最后必须：

```
return max(dp)
```

而不能：

```
return dp[-1]
```

因为 LIS 不一定以最后一个数组元素结束。

例如：

```
[1, 2, 3, 0]
```

DP 可能为：

```
[1, 2, 3, 1]
```

答案是：

```
3
```

而不是：

```
dp[-1] = 1
```

------

# 10. 面试中推荐的解题思路

如果面试官第一次给出这道题，可以先给出非常自然的：

\[ O(n^2) \]

动态规划。

定义：

```
dp[i]
```

表示：

> 以 `nums[i]` 结尾的最长递增子序列长度。

转移：

```
if nums[j] < nums[i]:
    dp[i] = max(dp[i], dp[j] + 1)
```

代码：

```
from typing import List


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if not nums:
            return 0

        n = len(nums)

        # dp[i]：
        # 以 nums[i] 结尾的 LIS 长度
        dp = [1] * n

        for i in range(n):
            for j in range(i):

                # nums[i] 可以接到 nums[j] 后面
                if nums[j] < nums[i]:
                    dp[i] = max(
                        dp[i],
                        dp[j] + 1
                    )

        return max(dp)
```

然后如果面试官继续问：

> Can you optimize it?

再推出：

```
DP + Binary Search
```

也就是：

```
from typing import List
from bisect import bisect_left


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        tails = []

        for num in nums:
            # 找到第一个 >= num 的尾部
            index = bisect_left(tails, num)

            if index == len(tails):
                # num 可以扩展当前最长长度
                tails.append(num)
            else:
                # 用更小的尾部替换原来的尾部
                tails[index] = num

        return len(tails)
```

面试中解释这个优化时，最关键的一句话是：

> `tails[i]` 表示所有长度为 `i + 1` 的递增子序列中，能够得到的最小结尾元素。我们总希望尾部尽可能小，因为越小的尾部越容易被后续元素扩展。

如果能清楚解释这一点，基本就掌握了 LIS 的 `O(n log n)` 解法。