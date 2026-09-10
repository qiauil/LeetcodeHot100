# 最大子数组和（Maximum Subarray）

## 题目描述

给定一个整数数组 `nums`，请找出一个具有最大和的 **连续子数组（subarray）**，并返回这个最大和。

**子数组** 是数组中一段连续且非空的元素序列。

---

## 示例 1

```text
输入：nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

输出：6
```

解释：

```text
[4, -1, 2, 1]
```

的元素和为：

```text
4 + (-1) + 2 + 1 = 6
```

这是所有连续子数组中的最大和。

---

## 示例 2

```text
输入：nums = [1]

输出：1
```

---

## 示例 3

```text
输入：nums = [5, 4, -1, 7, 8]

输出：23
```

解释：

```text
[5, 4, -1, 7, 8]
```

整个数组的和为：

```text
23
```

---

# 核心观察

这道题最关键的问题是：

> 当我们遍历到一个新的数字时，应该继续之前的子数组，还是从当前位置重新开始？

假设当前数字是：

```text
num
```

并且：

```text
curSum
```

表示：

> 以当前位置前一个元素结尾的最大子数组和。

那么当前有两种选择：

### 选择 1：继续之前的子数组

```text
curSum + num
```

### 选择 2：从当前位置重新开始

```text
num
```

因此：

```text
newCurSum = max(
    num,
    curSum + num
)
```

这就是 Kadane's Algorithm 的核心状态转移。

---

# 1. 暴力枚举

## 思路

最直接的方法是枚举所有连续子数组。

对于每一个起点 `i`：

- 从 `i` 开始向右扩展；
- 累加新的元素；
- 每得到一个新的连续子数组，就更新最大和。

例如：

```text
nums = [1, -2, 3]
```

所有连续子数组：

```text
[1]        -> 1
[1, -2]    -> -1
[1, -2, 3] -> 2
[-2]       -> -2
[-2, 3]    -> 1
[3]        -> 3
```

最终答案为：

```text
3
```

---

## Python 实现

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        # 至少有一个元素，因此可以直接用 nums[0] 初始化答案
        res = nums[0]

        # 枚举子数组起点
        for i in range(len(nums)):
            cur_sum = 0

            # 枚举子数组终点
            for j in range(i, len(nums)):
                cur_sum += nums[j]

                # 更新全局最大和
                res = max(res, cur_sum)

        return res
```

---

## 时间与空间复杂度

- 时间复杂度：`O(n^2)`
- 空间复杂度：`O(1)`

---

# 2. 前缀和

## 思路

定义前缀和：

```text
prefix[i]
```

表示：

```text
nums[0] + nums[1] + ... + nums[i - 1]
```

那么子数组：

```text
nums[left:right]
```

的和可以通过：

```text
prefix[right] - prefix[left]
```

快速计算。

例如：

```text
nums = [2, -1, 3]

prefix = [0, 2, 1, 4]
```

子数组：

```text
nums[1:3] = [-1, 3]
```

其和为：

```text
prefix[3] - prefix[1]
= 4 - 2
= 2
```

---

## 基础实现

虽然使用前缀和可以把每个子数组的求和操作降低到 `O(1)`，但是仍然需要枚举：

```text
O(n^2)
```

个子数组。

因此总时间复杂度仍然是：

```text
O(n^2)
```

---

## Python 实现

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        n = len(nums)

        # prefix[i] 表示 nums[0:i] 的和
        prefix = [0] * (n + 1)

        for i in range(n):
            prefix[i + 1] = prefix[i] + nums[i]

        res = nums[0]

        # 枚举所有子数组
        for left in range(n):
            for right in range(left + 1, n + 1):
                cur_sum = prefix[right] - prefix[left]
                res = max(res, cur_sum)

        return res
```

---

## 时间与空间复杂度

- 时间复杂度：`O(n^2)`
- 空间复杂度：`O(n)`

---

# 3. Kadane's Algorithm

## 推荐程度

**这是最大子数组和最经典、最重要的解法。**

时间复杂度：

```text
O(n)
```

额外空间：

```text
O(1)
```

---

## 核心状态

定义：

```text
curSum
```

表示：

> 以当前位置结尾的最大连续子数组和。

当处理当前数字：

```text
num
```

时，有两个选择：

```text
1. 从 num 重新开始
2. 把 num 接在之前的最优子数组后面
```

因此：

```text
curSum = max(
    num,
    curSum + num
)
```

与此同时：

```text
res
```

记录所有位置中出现过的最大 `curSum`。

---

# 为什么可以丢弃负贡献前缀？

这是 Kadane 算法最值得理解的地方。

假设前面某段连续子数组的和是：

```text
-5
```

当前数字是：

```text
4
```

如果继续之前的子数组：

```text
-5 + 4 = -1
```

如果从 `4` 重新开始：

```text
4
```

显然：

```text
4 > -1
```

也就是说：

> 如果前面的累计和小于 0，它只会拖累后面的结果。

因此，一旦：

```text
curSum < 0
```

继续保留它就没有意义。

可以直接从当前元素重新开始。

---

# 状态转移公式

设：

```text
dp[i]
```

表示：

> 以 `nums[i]` 结尾的最大连续子数组和。

那么：

```text
dp[i] = max(
    nums[i],
    dp[i - 1] + nums[i]
)
```

最终答案：

```text
max(dp[i])
```

由于：

```text
dp[i]
```

只依赖：

```text
dp[i - 1]
```

因此不需要真的创建一个 `dp` 数组。

只用一个变量：

```text
curSum
```

即可。

---

# Python 实现

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        # curSum：
        # 以当前位置结尾的最大连续子数组和
        curSum = nums[0]

        # res：
        # 到目前为止出现过的全局最大子数组和
        res = nums[0]

        for num in nums[1:]:
            # 两种选择：
            # 1. 从当前 num 重新开始
            # 2. 延续之前的最大子数组
            curSum = max(
                num,
                curSum + num
            )

            # 更新全局答案
            res = max(res, curSum)

        return res
```

---

# 一个更直观的 Kadane 写法

有时也会看到下面这种写法：

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        res = nums[0]
        cur_sum = 0

        for num in nums:
            # 如果之前的和为负数，
            # 它只会拖累当前数字，因此直接丢弃
            if cur_sum < 0:
                cur_sum = 0

            cur_sum += num

            res = max(res, cur_sum)

        return res
```

这段代码与前面的状态转移：

```python
curSum = max(num, curSum + num)
```

本质上完全等价。

---

# 示例推演

考虑：

```text
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
```

初始：

```text
curSum = -2
res    = -2
```

---

## 处理 `1`

```text
max(
    1,
    -2 + 1
)

= max(1, -1)
= 1
```

所以：

```text
curSum = 1
res    = 1
```

---

## 处理 `-3`

```text
max(
    -3,
    1 + (-3)
)

= max(-3, -2)
= -2
```

```text
curSum = -2
res    = 1
```

---

## 处理 `4`

```text
max(
    4,
    -2 + 4
)

= max(4, 2)
= 4
```

说明之前的负和应该被丢弃。

```text
curSum = 4
res    = 4
```

---

## 处理 `-1`

```text
curSum = 4 + (-1) = 3
res    = 4
```

---

## 处理 `2`

```text
curSum = 3 + 2 = 5
res    = 5
```

---

## 处理 `1`

```text
curSum = 5 + 1 = 6
res    = 6
```

此时对应子数组：

```text
[4, -1, 2, 1]
```

---

## 处理 `-5`

```text
curSum = 6 - 5 = 1
res    = 6
```

---

## 处理 `4`

```text
curSum = 1 + 4 = 5
res    = 6
```

最终：

```text
6
```

---

# 4. 基于前缀和最小值的 O(n) 解法

## 思路

如果：

```text
prefix[i]
```

表示前 `i` 个元素的和，那么子数组：

```text
nums[left:i]
```

的和是：

```text
prefix[i + 1] - prefix[left]
```

为了让这个值最大，我们希望：

```text
prefix[left]
```

尽可能小。

因此遍历数组时，只需要维护：

```text
之前出现过的最小前缀和
```

即可。

---

## 核心公式

当前位置前缀和：

```text
prefix
```

之前最小前缀：

```text
min_prefix
```

那么以当前位置作为右端点时的最大子数组和就是：

```text
prefix - min_prefix
```

然后更新：

```text
min_prefix = min(min_prefix, prefix)
```

---

## Python 实现

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        prefix = 0

        # 在还没取任何元素之前，前缀和为 0
        min_prefix = 0

        res = nums[0]

        for num in nums:
            # 当前前缀和
            prefix += num

            # 当前右端点能够得到的最大子数组和
            res = max(
                res,
                prefix - min_prefix
            )

            # 更新历史最小前缀和
            min_prefix = min(
                min_prefix,
                prefix
            )

        return res
```

---

## 时间与空间复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

---

# 为什么最小前缀和方法成立？

假设：

```text
prefix[r + 1]
```

是到位置 `r` 为止的前缀和。

对于任何以 `r` 结尾的子数组：

```text
nums[l:r+1]
```

其和为：

```text
prefix[r + 1] - prefix[l]
```

其中：

```text
prefix[r + 1]
```

已经固定。

要让结果最大，就应该让：

```text
prefix[l]
```

最小。

因此：

> 对于每一个右端点，只需要知道之前最小的前缀和。

---

# Kadane vs 最小前缀和

两个方法都能做到：

```text
时间：O(n)
空间：O(1)
```

它们只是观察角度不同。

| 方法       | 核心思路                     |
| ---------- | ---------------------------- |
| Kadane     | 判断是否值得继续之前的子数组 |
| 最小前缀和 | 当前前缀减去之前最小前缀     |
| 暴力法     | 枚举所有连续子数组           |
| 基础前缀和 | 用前缀和 O(1) 求某段区间和   |

面试中更推荐：

```text
Kadane's Algorithm
```

因为：

- 状态简单；
- 代码短；
- 经典程度高；
- 很容易扩展到类似动态规划问题。

---

# 最大子数组和与最大乘积子数组的区别

这是两个很适合一起理解的问题。

---

## 最大子数组和

状态只需要：

```text
curMax
```

因为加法不会出现：

```text
一个非常小的负数
突然因为再次相加负数而变成一个很大的正数
```

状态转移：

```text
curMax = max(
    num,
    curMax + num
)
```

---

## 最大乘积子数组

必须同时维护：

```text
curMax
curMin
```

因为：

```text
负数 × 负数 = 正数
```

状态转移：

```text
newMax = max(
    num,
    oldMax * num,
    oldMin * num
)

newMin = min(
    num,
    oldMax * num,
    oldMin * num
)
```

因此可以把 Maximum Product Subarray 看成：

> Kadane 思想在乘法场景下的扩展版本。

---

# 常见错误

## 1. 把答案初始化成 0

错误：

```python
res = 0
```

例如：

```text
nums = [-3, -2, -5]
```

正确答案是：

```text
-2
```

如果初始化为 `0`，会错误返回：

```text
0
```

因此应该：

```python
res = nums[0]
```

---

## 2. 认为遇到负数就必须重新开始

错误思路：

```text
负数一定不好
```

实际上负数可以暂时降低当前和，但后面可能还有更大的正数。

例如：

```text
[5, -1, 5]
```

最大子数组是整个：

```text
[5, -1, 5]
```

总和：

```text
9
```

因此真正需要判断的是：

```text
之前整个累计和是否为负
```

而不是：

```text
当前数字是否为负
```

---

## 3. 返回最后的 `curSum`

错误：

```python
return curSum
```

`curSum` 只表示：

```text
以最后一个元素结尾的最大子数组和
```

而全局最大值可能在更早的位置出现。

例如：

```text
nums = [5, -1, -10]
```

最终：

```text
curSum = -6
```

但正确答案是：

```text
5
```

因此必须维护：

```python
res = max(res, curSum)
```

---

# Python 细节：`nums[1:]`

代码中：

```python
for num in nums[1:]:
```

表示从：

```text
nums[1]
```

开始遍历到数组末尾。

例如：

```python
nums = [10, 20, 30, 40]

nums[1:]
```

得到：

```python
[20, 30, 40]
```

这是 Python 的 **切片（slice）**。

需要注意：

```python
nums[1:]
```

会创建一个新的列表，因此严格来说需要额外的 `O(n)` 临时空间。

在算法分析中很多面试场景仍会把核心算法称为 `O(1)` 额外空间，但如果希望完全避免切片，可以写：

```python
for i in range(1, len(nums)):
    num = nums[i]
```

---

# 完全 O(1) 额外空间的推荐实现

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        cur_sum = nums[0]
        res = nums[0]

        for i in range(1, len(nums)):
            num = nums[i]

            # 要么延续之前的子数组，
            # 要么从当前元素重新开始。
            cur_sum = max(
                num,
                cur_sum + num
            )

            res = max(
                res,
                cur_sum
            )

        return res
```

---

# 如果还需要返回最大子数组本身

有些面试官可能会继续追问：

> 如果不仅要返回最大和，还需要返回对应的子数组怎么办？

这时只需要额外记录：

- 当前子数组起点；
- 最优子数组的左右边界。

---

## Python 实现

```python
from typing import List, Tuple


class Solution:
    def maxSubArrayWithRange(
        self,
        nums: List[int]
    ) -> Tuple[int, List[int]]:
        cur_sum = nums[0]
        res = nums[0]

        # 当前候选子数组起点
        cur_left = 0

        # 最优子数组边界
        best_left = 0
        best_right = 0

        for i in range(1, len(nums)):
            # 如果从当前元素重新开始更好
            if nums[i] > cur_sum + nums[i]:
                cur_sum = nums[i]
                cur_left = i
            else:
                cur_sum += nums[i]

            # 更新全局最优解
            if cur_sum > res:
                res = cur_sum
                best_left = cur_left
                best_right = i

        return (
            res,
            nums[best_left:best_right + 1]
        )
```

例如：

```text
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
```

返回：

```text
最大和：6

子数组：
[4, -1, 2, 1]
```

---

# 面试时推荐的推导过程

## 第一步：先提出暴力法

可以先说：

> 枚举每一个起点和终点，并维护区间和。

复杂度：

```text
O(n^2)
```

---

## 第二步：寻找重复计算

暴力法会重复计算大量相邻区间。

例如：

```text
[1, 2, 3]
```

计算：

```text
[1, 2]
```

之后再计算：

```text
[1, 2, 3]
```

实际上只需要：

```text
之前的和 + 3
```

这提示我们可以维护：

```text
以当前位置结尾的最优状态
```

---

## 第三步：定义状态

定义：

```text
curSum =
以当前位置结尾的最大连续子数组和
```

处理：

```text
num
```

时只有两个选择：

```text
num
curSum + num
```

于是：

```text
curSum = max(
    num,
    curSum + num
)
```

---

## 第四步：解释为什么负数前缀可以丢弃

如果：

```text
curSum < 0
```

那么对于任何后续数字：

```text
x
```

都有：

```text
curSum + x < x
```

因此：

> 一个负的累计和永远不会帮助未来得到更大的连续子数组和。

所以可以从当前位置重新开始。

---

# 推荐面试代码

如果只记一个版本，推荐：

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        # 以当前位置结尾的最大子数组和
        cur_sum = nums[0]

        # 全局最大子数组和
        res = nums[0]

        for i in range(1, len(nums)):
            num = nums[i]

            # 当前只有两种选择：
            # 1. 从当前元素重新开始
            # 2. 延续之前的子数组
            cur_sum = max(
                num,
                cur_sum + num
            )

            # 更新全局答案
            res = max(
                res,
                cur_sum
            )

        return res
```

---

# 时间与空间复杂度

对于 Kadane 算法：

```text
时间复杂度：O(n)
空间复杂度：O(1)
```

因为：

- 数组只遍历一次；
- 只维护有限几个变量。

---

# 总结

最大子数组和的核心状态是：

```text
curSum =
以当前位置结尾的最大连续子数组和
```

状态转移：

```text
curSum = max(
    nums[i],
    curSum + nums[i]
)
```

全局答案：

```text
res = max(
    res,
    curSum
)
```

最重要的直觉是：

> **如果之前的累计和是负数，那么它只会拖累后面的子数组，因此应该直接丢弃。**

最终复杂度：

```text
时间：O(n)
空间：O(1)
```

这就是经典的：

```text
Kadane's Algorithm
```

它也是理解 Maximum Product Subarray、最大环形子数组和以及许多连续区间动态规划问题的重要基础。
