# Maximum Product Subarray（乘积最大子数组）

给定一个整数数组 `nums`，请找出数组中**乘积最大的连续非空子数组**，并返回该乘积。

**子数组（subarray）** 是数组中一段连续、非空的元素序列。

可以假设最终答案能够存储在 **32 位整数**范围内。

> 注意：如果子数组只有一个元素，那么它的乘积就是这个元素本身。

------

## 1. 暴力枚举

### 思路

这道题比“最大子数组和”稍微复杂一些，因为乘积会受到两个特殊情况的强烈影响：

- **负数**：会使乘积符号翻转。原本很大的正数乘上负数会变成很小的负数，而原本很小的负数再乘一个负数可能变成很大的正数。
- **0**：任何跨过 `0` 的子数组乘积都会变成 `0`。

最直接的做法就是枚举所有连续子数组。

对于每一个起点 `i`：

1. 从 `nums[i]` 开始。
2. 不断向右扩展终点 `j`。
3. 维护当前子数组的乘积。
4. 更新全局最大值。

由于每一种可能的连续子数组都会被检查，因此一定可以得到正确答案。

### 算法步骤

1. 用 `nums[0]` 初始化答案 `res`。
2. 枚举每个起点 `i`。
3. 设置 `cur = nums[i]`。
4. 不断向右扩展：
   - `cur *= nums[j]`
   - 使用 `cur` 更新 `res`
5. 返回 `res`。

```
from typing import List


class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        # 当前找到的最大乘积
        res = nums[0]

        # 枚举子数组起点
        for i in range(len(nums)):
            cur = nums[i]
            res = max(res, cur)

            # 枚举子数组终点
            for j in range(i + 1, len(nums)):
                # 当前子数组 nums[i:j+1] 的乘积
                cur *= nums[j]

                # 更新最大乘积
                res = max(res, cur)

        return res
```

### 复杂度

- 时间复杂度：**O(n²)**
- 空间复杂度：**O(1)**

### 小结

这种方法的优点是简单直观，非常适合理解题意，但当数组很长时效率较低。

例如数组长度为 `n` 时，一共有大约

\[ \frac{n(n+1)}{2} \]

个连续子数组，所以需要平方级别的时间。

------

# 2. 动态规划 / Kadane 思想

这是这道题最重要、也是面试中最推荐掌握的解法。

## 核心思路

经典的 **Kadane Algorithm（最大子数组和）** 只需要维护：

> “以当前位置结尾的最大和”。

但对于乘积来说，只维护最大值是不够的。

原因是：

\[ 负数 \times 负数 = 正数 \]

例如：

```
[-2, 3, -4]
```

当我们处理到 `-4` 时：

- 前面的最大乘积可能是 `3`
- 前面的最小乘积可能是 `-6`

此时：

```
3 × -4 = -12
-6 × -4 = 24
```

反而是之前的**最小乘积**产生了新的最大乘积。

因此，在每一个位置都必须同时维护：

- `curMax`：以当前位置结尾的最大乘积
- `curMin`：以当前位置结尾的最小乘积

------

## 状态转移

假设当前元素为：

```
num
```

对于以 `num` 结尾的子数组，只有三种可能：

### 1. 从当前位置重新开始

```
num
```

### 2. 接在之前最大乘积后面

```
num * curMax
```

### 3. 接在之前最小乘积后面

```
num * curMin
```

所以新的最大值为：

```
curMax = max(
    num,
    num * oldCurMax,
    num * oldCurMin
)
```

新的最小值为：

```
curMin = min(
    num,
    num * oldCurMax,
    num * oldCurMin
)
```

------

## 为什么必须保存旧的 `curMax`？

因为计算新的 `curMax` 和 `curMin` 时，两者都必须基于**上一轮的状态**。

如果先写：

```
curMax = ...
```

然后再使用：

```
curMax * num
```

计算 `curMin`，这里使用的已经是更新后的 `curMax`，就会导致错误。

因此需要先保存：

```
tmp = curMax * num
```

或者更清晰地直接保存：

```
prevMax = curMax
prevMin = curMin
```

------

## 推荐写法

```
from typing import List


class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        # 全局最大乘积
        res = nums[0]

        # curMax：以上一个位置结尾的最大乘积
        # curMin：以上一个位置结尾的最小乘积
        curMax = 1
        curMin = 1

        for num in nums:
            # 保存更新前的状态
            prevMax = curMax
            prevMin = curMin

            # 当前位置可以：
            # 1. 自己重新开始一个子数组
            # 2. 接在之前最大乘积之后
            # 3. 接在之前最小乘积之后
            curMax = max(
                num,
                num * prevMax,
                num * prevMin
            )

            curMin = min(
                num,
                num * prevMax,
                num * prevMin
            )

            # 更新全局答案
            res = max(res, curMax)

        return res
```

原答案中的写法稍微节省了一个变量：

```
from typing import List


class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        res = nums[0]
        curMin, curMax = 1, 1

        for num in nums:
            # 保存旧 curMax * num
            # 因为下面 curMax 会立即被更新
            tmp = curMax * num

            curMax = max(
                num * curMax,
                num * curMin,
                num
            )

            curMin = min(
                tmp,
                num * curMin,
                num
            )

            res = max(res, curMax)

        return res
```

------

## `0` 为什么不需要特殊处理？

假设：

```
nums = [2, 3, 0, 4]
```

处理 `0` 时：

```
curMax = max(0, oldMax * 0, oldMin * 0)
       = 0

curMin = 0
```

下一轮处理 `4`：

```
curMax = max(4, 0 * 4, 0 * 4)
       = 4
```

也就是说：

> `num` 本身作为候选值，使算法能够自动在 `0` 之后重新开始一个新的子数组。

因此不需要写：

```
if num == 0:
    ...
```

------

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(1)**

------

## 面试中应该记住的核心

这道题最关键的一句话是：

> **因为负数可能把最小乘积翻转成最大乘积，所以必须同时维护当前最大乘积和当前最小乘积。**

如果面试时能解释清楚这一点，基本就抓住了题目的核心。

------

# 3. 前缀积 + 后缀积

## 思路

还有一种非常简洁的 O(n) 解法。

关键观察是：

> 在一个不包含 `0` 的连续区间中，最大乘积子数组要么是整个区间，要么通过去掉第一个负数之前的部分，或者去掉最后一个负数之后的部分得到。

考虑一个没有 `0` 的区间。

### 情况 1：负数数量是偶数

例如：

```
[-2, 3, -4]
```

负数有两个。

整个区间的乘积为正数：

```
(-2) × 3 × (-4) = 24
```

通常整个区间就是最优选择。

------

### 情况 2：负数数量是奇数

例如：

```
[-2, 3, 4]
```

整个乘积为负：

```
-24
```

如果想得到正数乘积，就必须去掉一个负数。

只有两种值得考虑的方式：

- 去掉**第一个负数及其之前的元素**
- 去掉**最后一个负数及其之后的元素**

换句话说：

> 最优结果一定能够通过某个前缀积或者后缀积得到。

因此可以：

- 从左向右计算前缀积
- 从右向左计算后缀积

同时维护最大值。

------

## `0` 的处理

`0` 会把数组天然分成多个独立区间。

例如：

```
[2, 3, 0, -2, -4]
```

可以看作：

```
[2, 3]
[-2, -4]
```

扫描时只需要遇到 `0` 后重新开始计算乘积即可。

------

## 算法

```
from typing import List


class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        n = len(nums)
        res = nums[0]

        # 分别维护从左向右和从右向左的乘积
        prefix = 0
        suffix = 0

        for i in range(n):
            # 如果上一轮乘积为 0，
            # 相当于从当前位置重新开始
            prefix = nums[i] * (prefix or 1)

            # 从右往左扫描
            suffix = nums[n - 1 - i] * (suffix or 1)

            res = max(res, prefix, suffix)

        return res
```

------

## Python：`x or 1` 是什么意思？

这一写法利用了 Python 中的 **truthy / falsy（真值判断）**。

在 Python 中：

```
0
```

会被视为 `False`。

因此：

```
0 or 1
```

得到：

```
1
```

而：

```
5 or 1
```

得到：

```
5
```

所以：

```
prefix or 1
```

等价于：

```
prefix if prefix != 0 else 1
```

因此：

```
prefix = nums[i] * (prefix or 1)
```

实际上表示：

```
if prefix == 0:
    prefix = nums[i]
else:
    prefix *= nums[i]
```

也就是：

> 如果前面的乘积被 `0` 截断了，就从当前位置重新开始。

面试中如果担心 `or` 写法不够直观，也可以写成：

```
if prefix == 0:
    prefix = 1

prefix *= nums[i]
```

------

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(1)**

------

## 与动态规划解法的比较

两种方法复杂度完全相同：

| 方法                       | 时间 | 空间 | 特点                           |
| -------------------------- | ---- | ---- | ------------------------------ |
| `curMax / curMin` 动态规划 | O(n) | O(1) | 通用、逻辑清晰、最推荐         |
| 前缀积 + 后缀积            | O(n) | O(1) | 代码非常简洁，但证明稍微不直观 |

代码面试中，通常更推荐 **`curMax / curMin` 动态规划**，因为它展示了非常清晰的状态转移思想。

------

# 4. 按 `0` 分段 + 滑动窗口

## 思路

由于任何包含 `0` 的乘积都会变为 `0`，因此可以先把数组按照 `0` 拆成多个不包含 `0` 的区间。

例如：

```
[2, -3, 0, -2, -4, 0, 5]
```

拆成：

```
[2, -3]
[-2, -4]
[5]
```

对于一个不包含 `0` 的区间：

- 如果负数数量是偶数，可以保留所有负数。
- 如果负数数量是奇数，就必须少保留一个负数，使窗口中的负数数量变成偶数。

然后使用双指针维护一个负数数量满足条件的窗口。

------

## 算法步骤

1. 按照 `0` 将数组分成若干不含 `0` 的区间。

2. 对于每个区间：

   - 统计负数数量 `negs`

   - 如果是偶数：

     ```
     need = negs
     ```

   - 如果是奇数：

     ```
     need = negs - 1
     ```

3. 使用窗口 `[j, i]`：

   - 向右扩展 `i`
   - 更新乘积 `prod`
   - 更新负数数量

4. 如果窗口中的负数超过 `need`：

   - 不断移动左边界 `j`
   - 将移出的元素从乘积中除掉

5. 使用合法窗口更新答案。

------

## 代码

```
from typing import List


class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        segments = []
        cur = []

        # 使用负无穷初始化，
        # 保证负数数组也能够正确处理
        res = float("-inf")

        # ---------- 按 0 分割数组 ----------
        for num in nums:
            # 单独的 0 本身也可能是答案
            res = max(res, num)

            if num == 0:
                if cur:
                    segments.append(cur)
                cur = []
            else:
                cur.append(num)

        # 别忘了最后一个区间
        if cur:
            segments.append(cur)

        # ---------- 处理每一个非零区间 ----------
        for sub in segments:
            # 统计整个区间的负数数量
            total_negs = sum(1 for x in sub if x < 0)

            # 最优窗口允许包含的负数数量
            need = (
                total_negs
                if total_negs % 2 == 0
                else total_negs - 1
            )

            prod = 1
            window_negs = 0
            j = 0

            for i in range(len(sub)):
                prod *= sub[i]

                if sub[i] < 0:
                    window_negs += 1

                # 窗口中负数过多，
                # 不断收缩左边界
                while window_negs > need:
                    prod //= sub[j]

                    if sub[j] < 0:
                        window_negs -= 1

                    j += 1

                # 确保窗口非空
                if j <= i:
                    res = max(res, prod)

        return res
```

------

## Python：生成器表达式

代码中：

```
sum(1 for x in sub if x < 0)
```

用于统计负数数量。

可以理解成：

```
count = 0

for x in sub:
    if x < 0:
        count += 1
```

其中：

```
1 for x in sub if x < 0
```

是一个 **generator expression（生成器表达式）**。

每遇到一个负数就产生一个 `1`，最后由 `sum()` 全部加起来。

例如：

```
sub = [-2, 3, -4, 5]

sum(1 for x in sub if x < 0)
```

相当于：

```
1 + 1 = 2
```

------

## Python：为什么可以使用 `//=`？

因为这里的 `prod` 始终是窗口中所有整数的完整乘积，而 `sub[j]` 本来就是这个乘积中的一个因子。

例如：

```
prod = 2 × (-3) × 4 = -24
```

移除 `2`：

```
prod //= 2
```

得到：

```
-12
```

也就是：

```
(-3) × 4
```

不过这种写法依赖“当前元素一定是乘积的精确因子”。

------

## 复杂度

原代码会额外保存所有非零区间，因此：

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

理论上也可以不显式保存这些区间，把额外空间优化到 O(1)，但这个解法本身已经比动态规划方法复杂，因此通常没有必要进一步优化。

------

# 常见错误

## 1. 只维护当前最大乘积

错误思路：

```
curMax = max(num, curMax * num)
```

这类似最大子数组和，但对于乘积不成立。

例如：

```
[-2, 3, -4]
```

处理到 `3`：

```
curMax = 3
curMin = -6
```

处理 `-4`：

```
3 × -4 = -12
-6 × -4 = 24
```

真正的最大值来自之前的最小值。

因此必须同时维护：

```
curMax
curMin
```

------

## 2. 忘记 `0` 会截断乘积

例如：

```
[2, 3, 0, 4, 5]
```

最大乘积不能跨过 `0`。

实际上数组可以理解为两个独立区间：

```
[2, 3]

[4, 5]
```

但在动态规划解法中不需要显式分段，因为：

```
max(num, num * curMax, num * curMin)
```

中的 `num` 可以自然地重新开始子数组。

------

## 3. 更新 `curMax` 后再计算 `curMin`

错误示例：

```
curMax = max(num, num * curMax, num * curMin)

curMin = min(num, num * curMax, num * curMin)
```

第二行中的 `curMax` 已经不是上一轮的 `curMax` 了。

正确方法是先保存旧状态：

```
prevMax = curMax
prevMin = curMin

curMax = max(num, num * prevMax, num * prevMin)
curMin = min(num, num * prevMax, num * prevMin)
```

这是动态规划实现中非常常见的一类错误：

> **如果多个新状态都依赖旧状态，就不要在保存旧值之前原地覆盖状态。**

------

# 面试推荐解法

优先掌握下面这一版即可：

```
from typing import List


class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        res = nums[0]
        curMax = curMin = 1

        for num in nums:
            prevMax = curMax
            prevMin = curMin

            # 以当前位置结尾：
            # - 从 num 重新开始
            # - 接在之前最大乘积后面
            # - 接在之前最小乘积后面
            curMax = max(
                num,
                num * prevMax,
                num * prevMin
            )

            curMin = min(
                num,
                num * prevMax,
                num * prevMin
            )

            res = max(res, curMax)

        return res
```

可以把整个算法浓缩成三个问题来记：

1. **为什么需要 `curMax`？**
   记录以当前位置结尾的最大乘积。
2. **为什么需要 `curMin`？**
   因为负数可能把最小乘积翻转成最大乘积。
3. **为什么候选项里一定有 `num`？**
   因为有时候应该放弃之前的子数组，从当前位置重新开始；这也让算法能够自然处理 `0`。

最终复杂度：

\[ \boxed{\text{Time } O(n),\quad \text{Space } O(1)} \]