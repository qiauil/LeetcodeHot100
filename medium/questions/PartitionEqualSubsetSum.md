# Partition Equal Subset Sum（分割等和子集）

给定一个只包含正整数的数组 `nums`。

请判断是否可以将数组划分为两个子集 `subset1` 和 `subset2`，使得：

```
sum(subset1) == sum(subset2)
```

如果可以，返回 `True`；否则返回 `False`。

------

## 核心思路：把问题转化为 Subset Sum

假设数组所有元素之和为：

```
total = sum(nums)
```

如果能够将数组划分为两个和相等的子集，那么每个子集的和一定是：

```
target = total // 2
```

因此首先有一个非常重要的判断：

- 如果 `total` 是奇数，不可能被平均分成两份，直接返回 `False`。
- 如果 `total` 是偶数，问题就转化为了：

> **能否从 `nums` 中选择若干个数字，使它们的和恰好等于 `target = total // 2`？**

这就是经典的 **Subset Sum（子集和）** 问题，同时也是 **0/1 背包问题** 的一个典型变体。

所谓 **0/1**，指的是：

> 每一个数字只能选择一次，不能重复选择。

例如：

```
nums = [1, 5, 11, 5]

total = 22
target = 11
```

我们可以找到子集：

```
[11]
```

或者：

```
[1, 5, 5]
```

它们的和都是 `11`，所以答案为 `True`。

------

# 1. 递归 Recursion

## 思路

对于每一个位置 `i`，当前数字 `nums[i]` 都只有两种选择：

1. **不选它**
2. **选择它**

因此可以用 DFS 搜索所有可能的组合。

定义：

```
dfs(i, target)
```

表示：

> 从下标 `i` 开始的数字中，能否选出一些数字，使它们的和等于当前剩余的 `target`？

对于 `nums[i]`：

### 不选择当前数字

```
dfs(i + 1, target)
```

### 选择当前数字

```
dfs(i + 1, target - nums[i])
```

只要其中一种情况成功，就返回 `True`。

------

## 终止条件

如果：

```
target == 0
```

说明已经成功凑出了目标值，可以立即返回：

```
True
```

如果：

```
i == len(nums)
```

说明所有数字都已经考虑完了，却仍然没有凑出目标值，返回：

```
False
```

如果：

```
target < 0
```

说明选择的数字已经超过目标值。

由于题目中的数字全部为正整数，后面继续选数字只会让结果更小，所以可以直接返回 `False`。

------

## 代码

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        # 总和为奇数，不可能平均分成两个整数和
        if total % 2 != 0:
            return False

        target = total // 2

        def dfs(i: int, remain: int) -> bool:
            # 已经成功凑出 target
            if remain == 0:
                return True

            # 数字已经使用完，或者当前和已经超过 target
            if i >= len(nums) or remain < 0:
                return False

            # 选择 1：不使用 nums[i]
            skip = dfs(i + 1, remain)

            # 选择 2：使用 nums[i]
            take = dfs(i + 1, remain - nums[i])

            return skip or take

        return dfs(0, target)
```

------

## 复杂度

设：

- `n = len(nums)`
- `target = sum(nums) // 2`

每个数字都有「选」和「不选」两个选择，因此：

```
时间复杂度：O(2^n)
空间复杂度：O(n)
```

空间主要来自递归调用栈。

### 为什么是 `2^n`？

如果有 `n` 个数字，每一个数字都有：

```
选 / 不选
```

两个状态。

因此最多产生：

```
2 × 2 × ... × 2 = 2^n
```

种组合。

这也是为什么纯递归通常无法通过较大的测试数据。

------

# 2. Top-Down DP：递归 + Memoization

## 思路

观察刚才的 DFS：

```
dfs(i, remain)
```

实际上，同一个状态可能会被计算很多次。

例如：

```
dfs(5, 10)
```

可能通过不同的递归路径反复出现。

但只要：

```
i 相同
remain 相同
```

那么后面可选择的数字完全一样，答案自然也完全一样。

所以可以把计算过的状态保存起来：

```
memo[i][remain]
```

表示：

> 从下标 `i` 开始，能否凑出 `remain`。

这就是 **Memoization（记忆化搜索）**。

------

## 状态数量

`i` 最多有 `n` 种：

```
0 ~ n - 1
```

`remain` 最多有：

```
0 ~ target
```

因此总共最多：

```
n × target
```

种不同状态。

这就将原来的指数级复杂度：

```
O(2^n)
```

降低到了：

```
O(n × target)
```

------

## 代码

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False

        target = total // 2
        n = len(nums)

        # -1：还没有计算
        # True / False：已经计算过
        memo = [[-1] * (target + 1) for _ in range(n)]

        def dfs(i: int, remain: int) -> bool:
            # 成功凑出 target
            if remain == 0:
                return True

            # 没有数字可选，或者已经超过 target
            if i >= n or remain < 0:
                return False

            # 当前状态已经计算过
            if memo[i][remain] != -1:
                return memo[i][remain]

            # 不选择 nums[i]
            skip = dfs(i + 1, remain)

            # 选择 nums[i]
            take = dfs(i + 1, remain - nums[i])

            memo[i][remain] = skip or take

            return memo[i][remain]

        return dfs(0, target)
```

------

## 复杂度

```
时间复杂度：O(n × target)
空间复杂度：O(n × target)
```

另外还有：

```
O(n)
```

的递归调用栈，不过被 DP 表的 `O(n × target)` 主导。

------

## 面试理解

Top-Down DP 本质上就是：

```
暴力递归
    ↓
发现重复子问题
    ↓
加入 Memo
    ↓
Dynamic Programming
```

所以如果面试时不知道如何直接写 DP，通常可以先写出递归，再问自己：

> **递归函数中哪些参数决定了一个子问题？**

这里答案是：

```
(i, remain)
```

因此它们就是 DP 状态。

------

# 3. Bottom-Up DP：二维 DP

## 状态定义

现在不再使用递归，而是直接建立 DP 表。

定义：

```
dp[i][j]
```

表示：

> 使用前 `i` 个数字，能否组成和 `j`？

这里：

```
i = 0 ~ n
j = 0 ~ target
```

------

## Base Case

对于任何数量的数字：

```
dp[i][0] = True
```

因为目标和为 `0` 时，我们什么都不选即可：

```
{} → sum = 0
```

因此：

```
for i in range(n + 1):
    dp[i][0] = True
```

------

## 状态转移

当前数字是：

```
num = nums[i - 1]
```

为什么是 `i - 1`？

因为：

```
dp 第 i 行 = 前 i 个数字
```

而 Python 数组下标从 `0` 开始，因此第 `i` 个数字是：

```
nums[i - 1]
```

------

对于目标 `j`，有两种选择。

### 不选择当前数字

那么能不能凑出 `j`，完全取决于前 `i - 1` 个数字：

```
dp[i - 1][j]
```

### 选择当前数字

如果：

```
num <= j
```

选择 `num` 后，只需要之前能够凑出：

```
j - num
```

即：

```
dp[i - 1][j - num]
```

所以：

```
dp[i][j] = (
    dp[i - 1][j]
    or
    dp[i - 1][j - num]
)
```

------

## 代码

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False

        target = total // 2
        n = len(nums)

        # dp[i][j]:
        # 使用前 i 个数字，能否凑出和 j
        dp = [
            [False] * (target + 1)
            for _ in range(n + 1)
        ]

        # 和为 0 永远可以通过“什么都不选”实现
        for i in range(n + 1):
            dp[i][0] = True

        for i in range(1, n + 1):
            num = nums[i - 1]

            for j in range(1, target + 1):
                # 不选择当前数字
                dp[i][j] = dp[i - 1][j]

                # 如果当前数字可以放入
                if num <= j:
                    # 或者选择当前数字
                    dp[i][j] = (
                        dp[i][j]
                        or dp[i - 1][j - num]
                    )

        return dp[n][target]
```

------

## 复杂度

```
时间复杂度：O(n × target)
空间复杂度：O(n × target)
```

------

# 4. 二维 DP → 两个一维数组

## 思路

观察二维 DP 的状态转移：

```
dp[i][j]
```

只依赖：

```
dp[i - 1][j]
dp[i - 1][j - num]
```

也就是说：

> **当前这一行只依赖上一行。**

因此没有必要保存整个二维矩阵。

只需要保存：

```
上一行 dp
当前行 next_dp
```

即可。

------

## 代码

原答案这里有一个值得修正的小问题：

```
nextDp[0]
```

必须始终保持为 `True`。

因为：

```
和为 0 永远可以通过什么都不选得到。
```

如果直接复用两个数组但没有维护 `nextDp[0] = True`，第一次交换数组之后可能让 `dp[0]` 变成 `False`，从而导致后面的状态计算错误。

更安全、也更容易理解的写法如下：

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False

        target = total // 2

        # dp[j]：
        # 使用之前处理过的数字，能否凑出和 j
        dp = [False] * (target + 1)
        dp[0] = True

        for num in nums:
            # 当前这一轮从上一轮 dp 复制
            # 对应“不选择 num”
            next_dp = dp.copy()

            for j in range(num, target + 1):
                # 如果选择 num，
                # 需要上一轮能够凑出 j - num
                next_dp[j] = (
                    dp[j]
                    or dp[j - num]
                )

            dp = next_dp

        return dp[target]
```

------

## 复杂度

```
时间复杂度：O(n × target)
空间复杂度：O(target)
```

虽然每一轮有：

```
dp.copy()
```

但复制长度也是 `target`，所以整体时间复杂度仍然是：

```
O(n × target)
```

------

# 5. Hash Set DP

## 思路

还可以不用数组，而使用 Python 的：

```
set
```

记录当前所有能够组成的子集和。

例如：

```
dp = {0}
```

表示目前只能组成：

```
0
```

假设遇到数字：

```
num = 3
```

对于原来能够组成的每个和 `t`：

- 不选择 `3` → 仍然是 `t`
- 选择 `3` → 产生 `t + 3`

------

例如：

```
dp = {0, 1, 5}
num = 3
```

新的可能和包括：

```
不选 3：
0, 1, 5

选择 3：
3, 4, 8
```

因此：

```
next_dp = {0, 1, 3, 4, 5, 8}
```

------

## Python `set` 说明

`set` 是 Python 中的 **哈希集合**：

```
dp = set()
```

或者：

```
dp = {0}
```

它有两个非常重要的特点：

### 自动去重

```
{1, 1, 1, 2}
```

最终就是：

```
{1, 2}
```

不同选择可能产生相同的 subset sum，所以这里非常适合使用 `set`。

### 平均 O(1) 查询和插入

例如：

```
target in dp
dp.add(x)
```

平均时间复杂度通常都是 `O(1)`。

------

## 代码

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False

        target = total // 2

        # 当前所有可以形成的 subset sum
        dp = {0}

        for num in nums:
            next_dp = set()

            for current_sum in dp:
                # 不选择当前数字
                next_dp.add(current_sum)

                # 选择当前数字
                new_sum = current_sum + num

                if new_sum == target:
                    return True

                # 因为 nums 都是正数，
                # 超过 target 的状态没有继续保存的必要
                if new_sum < target:
                    next_dp.add(new_sum)

            dp = next_dp

        return target in dp
```

------

## 复杂度

理论上最多只需要保留：

```
0 ~ target
```

这些状态，因此：

```
时间复杂度：O(n × target)
空间复杂度：O(target)
```

不过 `set` 的常数开销通常比 Boolean Array 更大。

所以在这道题中：

> Hash Set 写法很直观，但通常不是性能最好的实现。

------

# 6. 最优的常规 DP：一维 0/1 背包

这是这道题最推荐掌握的标准解法。

## 核心思想

定义：

```
dp[j]
```

表示：

> 使用目前处理过的数字，能否组成和 `j`。

初始化：

```
dp[0] = True
```

因为不选择任何数字即可得到和 `0`。

对于每一个：

```
num
```

如果之前可以组成：

```
j - num
```

那么加上当前的 `num` 后，就可以组成：

```
j
```

所以：

```
dp[j] = dp[j] or dp[j - num]
```

------

# 最重要的问题：为什么必须倒序遍历？

这是这道题非常重要的面试考点。

必须写：

```
for j in range(target, num - 1, -1):
```

也就是：

```
target → num
```

从右往左遍历。

------

## 如果从左向右会发生什么？

假设：

```
nums = [2]
target = 4
```

开始：

```
dp = [True, False, False, False, False]
```

如果从左向右：

```
j = 2
```

因为：

```
dp[0] == True
```

所以：

```
dp[2] = True
```

接下来：

```
j = 4
```

现在程序看到：

```
dp[2] == True
```

于是：

```
dp[4] = True
```

相当于使用了：

```
2 + 2
```

但数组中明明只有一个 `2`。

也就是说，当前数字在同一轮被重复使用了。

这就从：

```
0/1 Knapsack
```

错误地变成了：

```
Unbounded / Complete Knapsack
```

------

## 为什么倒序不会重复使用？

如果从：

```
target → num
```

倒序更新，那么：

```
dp[j - num]
```

在本轮还没有被修改。

所以它代表的仍然是：

> **使用当前数字之前的状态。**

这样当前 `num` 就最多只会使用一次。

------

## 代码

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        # 奇数不可能平分
        if total % 2 != 0:
            return False

        target = total // 2

        # dp[j] 表示是否能组成和 j
        dp = [False] * (target + 1)

        # 什么都不选择，可以组成 0
        dp[0] = True

        for num in nums:
            # 必须倒序！
            # 保证每个 num 最多使用一次
            for j in range(target, num - 1, -1):
                dp[j] = dp[j] or dp[j - num]

            # 可选优化：已经找到 target，可以提前结束
            if dp[target]:
                return True

        return dp[target]
```

------

## `range(target, num - 1, -1)` 说明

Python 的：

```
range(start, stop, step)
```

不包含 `stop`。

因此：

```
range(target, num - 1, -1)
```

产生：

```
target, target-1, ..., num
```

例如：

```
list(range(10, 4, -1))
```

结果：

```
[10, 9, 8, 7, 6, 5]
```

所以如果希望循环能够包含：

```
num
```

`stop` 必须写成：

```
num - 1
```

------

## 复杂度

```
时间复杂度：O(n × target)
空间复杂度：O(target)
```

其中：

```
target = sum(nums) / 2
```

严格来说，这是一种 **Pseudo-polynomial Time（伪多项式时间）** 算法。

原因是复杂度取决于：

```
数字的值 / 数字总和
```

而不仅仅是输入元素个数 `n`。

------

# 7. Bitset DP

这是一个非常巧妙的版本，尤其适合 Python。

代码非常短：

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False

        target = total // 2

        # bit 0 = 1，表示 sum = 0 可以达到
        dp = 1

        for num in nums:
            # 原状态：不选择 num
            # 左移 num 位：选择 num
            dp |= dp << num

        # 检查第 target 位是不是 1
        return (dp & (1 << target)) != 0
```

------

# Bitset 到底在做什么？

这是整个答案里最抽象的部分。

核心思想是：

> 用一个整数的不同二进制位表示不同的 subset sum 是否能够达到。

假设：

```
dp 的第 j 个 bit = 1
```

就表示：

```
sum = j 可以组成
```

------

## 初始状态

最开始只有：

```
sum = 0
```

可以达到。

因此：

```
dp = 1
```

二进制：

```
...000001
```

第 `0` 位为 `1`：

```
bit 0 = 1
```

表示：

```
sum 0 可以达到
```

------

## 假设遇到 num = 3

执行：

```
dp << 3
```

原来：

```
000001
```

左移三位：

```
001000
```

原来的：

```
sum = 0
```

被转换成：

```
sum = 3
```

然后执行：

```
dp |= dp << 3
```

得到：

```
001001
```

现在：

```
bit 0 = 1
bit 3 = 1
```

表示：

```
sum = 0
sum = 3
```

都可以组成。

------

## 再加入 num = 2

当前：

```
dp = 001001
```

表示：

```
{0, 3}
```

执行：

```
dp << 2
```

相当于所有已有的和都增加 `2`：

```
{2, 5}
```

然后：

```
dp |= dp << 2
```

最终表示：

```
{0, 2, 3, 5}
```

这正对应：

```
不选任何数字：0
只选 2：2
只选 3：3
选择 2 + 3：5
```

------

# 位运算符说明

Bitset 解法使用了三个 Python 位运算。

### `<<`：左移

```
dp << num
```

相当于：

> 把所有已经能够组成的 sum 都增加 `num`。

------

### `|`：按位 OR

```
dp |= dp << num
```

相当于合并：

```
不选 num 可以达到的 sum
+
选择 num 可以达到的 sum
```

------

### `&`：按位 AND

最后：

```
dp & (1 << target)
```

是为了检查：

```
dp 的第 target 位是否为 1
```

也就是：

> 能不能组成 `target`。

------

## Bitset 复杂度需要更准确地理解

原答案将其简单写为：

```
时间：O(n × target)
空间：O(target)
```

作为一般性的上界理解没有问题，但 Python 的实现实际上更有意思。

Python 的 `int` 是 **任意精度整数**。因此：

```
dp << num
dp | ...
```

会一次性对一整块机器字进行底层运算，而不是在 Python 层一个一个地遍历 `target` 个 Boolean。

可以粗略理解为：

```
时间复杂度：O(n × target / word_size)
空间复杂度：O(target) bits
```

实际运行中，Bitset 往往比普通 Python 的二维/一维循环快很多。

不过面试时如果面试官主要考标准 DP，优先解释：

```
一维 0/1 背包
```

通常会更加清晰。

------

# 各种方法之间的关系

实际上，这 7 种写法并不是 7 个完全不同的算法，而是同一个问题不断优化的过程：

```
暴力递归
    ↓
发现重复状态
    ↓
Top-Down Memoization
    ↓
Bottom-Up 二维 DP
    ↓
观察当前行只依赖上一行
    ↓
两个一维数组
    ↓
利用倒序更新
    ↓
一个一维数组
    ↓
利用二进制位并行表示状态
    ↓
Bitset
```

这是学习 DP 时非常值得掌握的一条思维路径。

------

# 方法对比

| 方法           | 时间复杂度      | 空间复杂度       | 特点                        |
| -------------- | --------------- | ---------------- | --------------------------- |
| 递归           | `O(2^n)`        | `O(n)`           | 最直观，但效率低            |
| Top-Down Memo  | `O(n × target)` | `O(n × target)`  | 最容易从递归推导            |
| 二维 Bottom-Up | `O(n × target)` | `O(n × target)`  | DP 状态最清晰               |
| 两个一维数组   | `O(n × target)` | `O(target)`      | 展示空间优化过程            |
| Hash Set       | `O(n × target)` | `O(target)`      | 思路直观，Python 写起来方便 |
| 一维 0/1 背包  | `O(n × target)` | `O(target)`      | **面试最推荐**              |
| Bitset         | 位运算优化      | `O(target)` bits | Python 中非常精巧且通常很快 |

------

# 面试最推荐掌握的写法

如果代码面试只准备一个版本，建议熟练掌握下面这一版：

```
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False

        target = total // 2

        dp = [False] * (target + 1)
        dp[0] = True

        for num in nums:
            # 倒序遍历，保证 num 只使用一次
            for j in range(target, num - 1, -1):
                dp[j] = dp[j] or dp[j - num]

            # 提前找到答案
            if dp[target]:
                return True

        return False
```

可以将整个算法记忆成三个步骤：

```
1. total 是奇数 → False

2. 问题转化：
   是否存在 subset sum = total // 2？

3. 0/1 背包：
   dp[j] = dp[j] OR dp[j - num]
   并且 j 必须从右向左遍历
```

------

# 常见错误

## 1. 忘记检查总和是否为奇数

首先一定要：

```
if sum(nums) % 2:
    return False
```

因为如果：

```
total = 11
```

两个子集不可能分别拥有：

```
5.5
```

的整数和。

------

## 2. 一维 DP 从左向右更新

错误：

```
for j in range(num, target + 1):
    dp[j] = dp[j] or dp[j - num]
```

这可能会让同一个数字在同一轮被多次使用。

正确：

```
for j in range(target, num - 1, -1):
    dp[j] = dp[j] or dp[j - num]
```

记忆：

```
0/1 背包 → 倒序
完全背包 → 正序
```

这是非常常见的面试知识点。

------

## 3. 二维 DP 中混淆 `i` 和 `i - 1`

如果定义：

```
dp[i][j]
```

表示：

```
前 i 个数字
```

那么当前考虑的实际数组元素是：

```
nums[i - 1]
```

因为 Python 的数组从 `0` 开始。

------

## 4. 忘记 `dp[0] = True`

无论是二维还是一维 DP，都需要理解：

```
sum = 0
```

永远可以通过：

```
什么都不选
```

得到。

因此一维 DP 必须初始化：

```
dp[0] = True
```

二维 DP 则是：

```
dp[i][0] = True
```

------

## 5. 两数组空间优化时破坏 `dp[0]`

如果使用：

```
dp
next_dp
```

两个数组反复交换，必须确保：

```
next_dp[0] = True
```

始终成立。

这也是原始第 4 个解法中比较容易被忽略的细节。更推荐直接：

```
next_dp = dp.copy()
```

这样逻辑更加安全。

------

## 6. `target < 0` 为什么可以直接返回 False？

这是因为题目明确给的是：

```
positive integers
```

全部都是正整数。

如果：

```
remain < 0
```

后面继续选择任何数字都只会让 `remain` 更小，不可能重新回到 `0`。

如果题目允许负数，那么这个剪枝就不一定成立。

------

## 7. 整数溢出

Python 的：

```
int
```

支持任意精度整数，因此：

```
total = sum(nums)
```

通常不需要担心整数溢出。

但在 C++ / Java 等语言中，如果数组元素和可能很大，则需要注意数据类型。例如：

```
long long total;
```

或者 Java：

```
long total;
```

因此这个问题主要是其他语言需要注意，Python 中通常不是问题。

------

# 一个值得记住的 DP 模板

对于大量类似：

> 每个数字只能使用一次，判断能否组成某个目标和

的问题，都可以考虑下面这个模板：

```
dp = [False] * (target + 1)
dp[0] = True

for num in nums:
    for j in range(target, num - 1, -1):
        dp[j] = dp[j] or dp[j - num]
```

它本质上就是：

```
0/1 Knapsack
```

以后遇到以下类型的问题都应该对这个模式保持敏感：

```
Can Partition...
Subset Sum...
Target Sum...
Choose each item at most once...
Can we form sum X...
```

其中这道 **Partition Equal Subset Sum** 最关键的转化就是：

```
Equal Partition
        ↓
Subset Sum = total / 2
        ↓
0/1 Knapsack
```

这三步如果能够在面试中迅速识别出来，这道题基本就已经解决了一大半。