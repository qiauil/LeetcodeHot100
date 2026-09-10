# Subarray Sum Equals K（和为 K 的子数组）

给定一个整数数组 `nums` 和一个整数 `k`，返回数组中 **元素和等于 `k` 的连续非空子数组数量**。

> **Subarray（子数组）** 指数组中一段连续且非空的元素序列。

---

## 示例 1

```text
输入：
nums = [2, -1, 1, 2]
k = 2

输出：
4
```

满足条件的子数组为：

```text
[2]
[2, -1, 1]
[-1, 1, 2]
[2]
```

一共有：

```text
4
```

个。

---

## 示例 2

```text
输入：
nums = [4, 4, 4, 4, 4, 4]
k = 4

输出：
6
```

数组中的每一个单独的 `4` 都构成一个和为 `4` 的子数组，因此答案为 `6`。

---

## 约束

- `1 <= nums.length <= 20,000`
- `-1,000 <= nums[i] <= 1,000`
- `-10,000,000 <= k <= 10,000,000`

---

# 核心问题

这道题问的是：

> 有多少个连续子数组的元素和等于 `k`？

如果一个子数组从索引 `i` 开始，到索引 `j` 结束，那么它的和是：

```text
nums[i] + nums[i+1] + ... + nums[j]
```

最直接的方法是枚举所有起点和终点。

但更优的方法是利用：

> **前缀和 Prefix Sum**

将“某一段区间的和”转换成两个前缀和之差。

---

# 解法一：暴力枚举

## 思路

枚举每一个起点 `i`。

然后从 `i` 开始向右扩展终点 `j`，同时维护当前子数组的累计和。

例如：

```text
nums = [2, -1, 1, 2]
```

当：

```text
i = 0
```

依次检查：

```text
[2]
[2, -1]
[2, -1, 1]
[2, -1, 1, 2]
```

当：

```text
i = 1
```

依次检查：

```text
[-1]
[-1, 1]
[-1, 1, 2]
```

以此类推。

每当累计和等于 `k`，就将答案加 `1`。

---

## 算法步骤

1. 初始化：

```python
res = 0
```

2. 枚举每一个起点 `i`。
3. 设置：

```python
cur_sum = 0
```

4. 从 `i` 开始向右枚举终点 `j`：
   - 把 `nums[j]` 加入 `cur_sum`；
   - 如果 `cur_sum == k`，则 `res += 1`。
5. 返回 `res`。

---

## 代码

```python
from typing import List


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        res = 0

        # 枚举子数组起点
        for i in range(len(nums)):
            cur_sum = 0

            # 枚举子数组终点
            for j in range(i, len(nums)):
                cur_sum += nums[j]

                if cur_sum == k:
                    res += 1

        return res
```

---

## Python 命名说明

原代码中使用：

```python
sum = 0
```

虽然能够运行，但不推荐。

原因是 Python 自带一个内置函数：

```python
sum(...)
```

例如：

```python
sum([1, 2, 3])
```

结果为：

```text
6
```

如果写：

```python
sum = 0
```

会覆盖当前作用域中的内置函数名。

因此更推荐：

```python
cur_sum
```

或者：

```python
current_sum
```

---

## 复杂度分析

假设：

```text
n = len(nums)
```

一共有大约：

\[
\frac{n(n+1)}{2}
\]

个子数组。

因此时间复杂度为：

\[
O(n^2)
\]

空间复杂度为：

\[
O(1)
\]

---

# 解法二：前缀和 + 哈希表

这是这道题最重要、也是面试中最推荐的方法。

## 什么是前缀和？

定义：

```text
prefix[j]
```

表示：

```text
nums[0] + nums[1] + ... + nums[j]
```

例如：

```text
nums = [2, -1, 1, 2]
```

前缀和依次是：

```text
2
1
2
4
```

如果额外定义：

```text
prefix[-1] = 0
```

那么：

```text
索引：         -1   0   1   2   3
prefix：        0   2   1   2   4
```

---

# 如何利用前缀和计算子数组的和？

假设一个子数组是：

```text
nums[i ... j]
```

那么：

```text
prefix[j]
```

包含：

```text
nums[0] + ... + nums[i-1] + nums[i] + ... + nums[j]
```

而：

```text
prefix[i-1]
```

包含：

```text
nums[0] + ... + nums[i-1]
```

两者相减：

\[
prefix[j] - prefix[i-1]
\]

正好就是：

\[
nums[i] + nums[i+1] + \cdots + nums[j]
\]

也就是子数组：

```text
nums[i ... j]
```

的和。

---

# 核心公式

我们希望：

\[
prefix[j] - prefix[i-1] = k
\]

移项：

\[
prefix[i-1] = prefix[j] - k
\]

这句话非常重要。

它意味着：

> 当我们已经走到位置 `j`，当前前缀和为 `cur_sum` 时，只需要知道以前出现过多少次 `cur_sum - k`。

如果以前某个位置的前缀和等于：

```python
cur_sum - k
```

那么从那个位置之后开始，到当前位置结束的子数组，其和一定等于 `k`。

---

# 为什么需要“频率”而不只是判断是否存在？

考虑：

```text
nums = [0, 0, 0]
k = 0
```

前缀和会多次出现：

```text
0, 0, 0, 0
```

一个相同的前缀和值可能对应多个不同起点。

所以不能只记录：

```python
prefix_sum 是否出现过
```

而必须记录：

```python
prefix_sum 出现过多少次
```

因此哈希表保存的是：

```text
前缀和 -> 出现次数
```

---

# 逐步示例

考虑：

```text
nums = [2, -1, 1, 2]
k = 2
```

初始化：

```python
cur_sum = 0
res = 0
prefix_sums = {0: 1}
```

---

## 处理第一个 `2`

```text
cur_sum = 2
```

计算：

```text
diff = cur_sum - k
     = 2 - 2
     = 0
```

哈希表中：

```text
0 出现过 1 次
```

所以找到一个子数组：

```text
[2]
```

于是：

```text
res = 1
```

然后记录：

```text
prefix_sums[2] = 1
```

---

## 处理 `-1`

```text
cur_sum = 1
```

需要寻找：

```text
1 - 2 = -1
```

哈希表中没有：

```text
-1
```

因此没有新子数组。

记录：

```text
prefix_sums[1] = 1
```

---

## 处理 `1`

```text
cur_sum = 2
```

需要寻找：

```text
2 - 2 = 0
```

哈希表中：

```text
0 出现过 1 次
```

因此找到：

```text
[2, -1, 1]
```

答案：

```text
res = 2
```

然后：

```text
prefix_sums[2]
```

从 `1` 增加到 `2`。

---

## 处理最后一个 `2`

```text
cur_sum = 4
```

需要寻找：

```text
4 - 2 = 2
```

此时：

```text
prefix_sums[2] = 2
```

说明此前有 **两个位置** 的前缀和等于 `2`。

所以当前能够形成两个新的合法子数组：

```text
[-1, 1, 2]
[2]
```

于是：

```text
res += 2
```

最终：

```text
res = 4
```

---

# 算法步骤

1. 初始化：

```python
res = 0
cur_sum = 0
prefix_sums = {0: 1}
```

2. 遍历每一个 `num`：
   - 更新当前前缀和：

```python
cur_sum += num
```

   - 计算我们希望以前出现过的前缀和：

```python
diff = cur_sum - k
```

   - 如果 `diff` 出现过 `x` 次：

```python
res += x
```

   - 将当前 `cur_sum` 的出现次数加 `1`。
3. 返回 `res`。

---

# 代码

```python
from typing import List


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        res = 0
        cur_sum = 0

        # key   = 前缀和
        # value = 该前缀和出现过多少次
        #
        # 0: 1 表示在遍历数组之前，
        # 已经存在一个“空前缀”，其和为 0
        prefix_sums = {0: 1}

        for num in nums:
            # 当前前缀和
            cur_sum += num

            # 如果以前存在一个前缀和等于 cur_sum - k，
            # 那么从那个前缀之后到当前位置的子数组和就是 k
            diff = cur_sum - k

            # diff 可能出现过多次，
            # 每一次都对应一个不同的合法子数组
            res += prefix_sums.get(diff, 0)

            # 最后再记录当前前缀和
            prefix_sums[cur_sum] = prefix_sums.get(cur_sum, 0) + 1

        return res
```

---

# Python：`dict.get()`

这道题中使用：

```python
prefix_sums.get(diff, 0)
```

Python 字典的：

```python
dict.get(key, default)
```

表示：

- 如果 `key` 存在，返回对应的 value；
- 如果不存在，返回 `default`。

例如：

```python
d = {"a": 3}

d.get("a", 0)
```

返回：

```text
3
```

而：

```python
d.get("b", 0)
```

返回：

```text
0
```

所以：

```python
prefix_sums.get(diff, 0)
```

非常适合表示：

> `diff` 之前出现过多少次？如果一次都没有出现，就当作 `0`。

---

# 为什么必须初始化 `{0: 1}`？

这一点非常重要。

初始化：

```python
prefix_sums = {0: 1}
```

表示：

> 在开始遍历数组之前，存在一个和为 `0` 的空前缀。

考虑：

```text
nums = [2]
k = 2
```

处理 `2` 后：

```text
cur_sum = 2
```

需要寻找：

```text
cur_sum - k = 0
```

如果：

```python
prefix_sums = {0: 1}
```

那么能够找到：

```text
[2]
```

这个从索引 `0` 开始的子数组。

如果没有初始化：

```python
{0: 1}
```

那么从数组开头开始、且前缀和正好等于 `k` 的子数组就会漏掉。

---

# `{0: 1}` 的数学含义

也可以这样理解。

定义前缀和：

```text
prefix[0] = 0
```

然后：

```text
prefix[i+1] = nums[0] + ... + nums[i]
```

那么子数组：

```text
nums[0 ... j]
```

的和就是：

```text
prefix[j+1] - prefix[0]
```

而：

```text
prefix[0] = 0
```

所以我们必须事先把这个“起始前缀”记录下来。

因此：

```python
{0: 1}
```

并不是特殊技巧，而是前缀和定义自然推导出来的结果。

---

# 为什么必须“先查询，再更新”？

正确顺序：

```python
cur_sum += num

res += prefix_sums.get(cur_sum - k, 0)

prefix_sums[cur_sum] = prefix_sums.get(cur_sum, 0) + 1
```

不能交换成：

```python
cur_sum += num

prefix_sums[cur_sum] = prefix_sums.get(cur_sum, 0) + 1

res += prefix_sums.get(cur_sum - k, 0)
```

原因是：

> 当前前缀和只能作为未来子数组的“左边界”，不能作为当前子数组自己的“历史前缀”。

---

## 特别是当 `k = 0`

假设：

```text
nums = [1]
k = 0
```

当前：

```text
cur_sum = 1
```

如果先把：

```text
prefix_sums[1]
```

加入哈希表，然后再找：

```text
cur_sum - k = 1
```

就会把刚刚加入的当前前缀自己算进去。

这实际上对应一个：

```text
空子数组
```

但题目明确要求：

```text
subarray 必须 non-empty
```

所以这是错误的。

因此顺序一定是：

```text
1. 查询以前的前缀和
2. 再记录当前前缀和
```

---

# 为什么不能使用普通滑动窗口？

很多“连续子数组”问题可以用：

```text
left / right
```

双指针滑动窗口解决。

但这道题：

```text
nums[i]
```

可能是负数。

因此窗口和并不具有单调性。

---

## 如果数组全是正数

例如：

```text
nums = [1, 2, 3, 4]
```

扩大窗口：

```text
sum
```

一定增加。

缩小窗口：

```text
sum
```

一定减少。

因此可以根据：

```text
sum < k
sum > k
```

决定向哪个方向移动。

---

## 但存在负数时

例如：

```text
nums = [3, -2, 5]
```

加入一个新元素：

```text
-2
```

窗口和反而会减少。

删除一个负数：

```text
-2
```

窗口和反而会增加。

所以：

```text
扩大窗口
```

不一定让和变大。

```text
缩小窗口
```

也不一定让和变小。

于是普通滑动窗口失去了判断依据。

这就是为什么本题需要：

> **前缀和 + 哈希表**

而不是普通双指针滑动窗口。

---

# 复杂度分析

## 时间复杂度

数组只遍历一次。

每个元素执行：

- 一次前缀和更新；
- 一次哈希表查询；
- 一次哈希表更新。

Python 字典平均查询 / 更新复杂度为：

\[
O(1)
\]

因此总时间复杂度：

\[
\boxed{O(n)}
\]

---

## 空间复杂度

最坏情况下，每一个前缀和都不同。

哈希表中最多保存：

\[
O(n)
\]

个不同前缀和。

因此：

\[
\boxed{O(n)}
\]

---

# 更简洁的 `defaultdict` 写法

Python 还可以使用：

```python
collections.defaultdict
```

进一步简化代码。

## `defaultdict`

```python
from collections import defaultdict

count = defaultdict(int)
```

如果访问一个不存在的 key：

```python
count[x]
```

会自动返回：

```text
0
```

因此不需要反复写：

```python
dict.get(key, 0)
```

---

## 代码

```python
from typing import List
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_count = defaultdict(int)

        # 空前缀
        prefix_count[0] = 1

        cur_sum = 0
        res = 0

        for num in nums:
            cur_sum += num

            # 有多少个历史前缀满足：
            #
            # cur_sum - old_prefix = k
            #
            # 即：
            #
            # old_prefix = cur_sum - k
            res += prefix_count[cur_sum - k]

            # 当前前缀供未来使用
            prefix_count[cur_sum] += 1

        return res
```

这种写法非常简洁，也是一个很适合面试的 Python 版本。

---

# 推荐的面试版本

如果面试中只写一个版本，推荐：

```python
from typing import List


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_count = {0: 1}

        cur_sum = 0
        res = 0

        for num in nums:
            cur_sum += num

            # old_prefix = cur_sum - k
            res += prefix_count.get(cur_sum - k, 0)

            # 当前前缀和供未来位置使用
            prefix_count[cur_sum] = prefix_count.get(cur_sum, 0) + 1

        return res
```

这段代码非常短，但最好能够完整解释下面这条公式：

\[
cur\_sum - old\_prefix = k
\]

因此：

\[
old\_prefix = cur\_sum - k
\]

这就是整个算法的核心。

---

# 一个更通用的前缀和理解方式

以后看到：

> 连续子数组的和

可以先考虑：

```text
prefix sum
```

因为：

```text
区间和 = 右边前缀和 - 左边前缀和
```

也就是：

\[
sum(i...j) = prefix[j+1] - prefix[i]
\]

如果题目要求：

```text
sum(i...j) = k
```

那么：

\[
prefix[j+1] - prefix[i] = k
\]

变形：

\[
prefix[i] = prefix[j+1] - k
\]

于是就自然得到：

> 遍历到当前前缀和时，在哈希表中寻找 `current_prefix - k`。

这个模式在很多题目中都会出现。

---

# 常见错误

## 1. 忘记初始化 `{0: 1}`

错误：

```python
prefix_count = {}
```

正确：

```python
prefix_count = {0: 1}
```

否则会漏掉：

```text
从索引 0 开始
```

且和正好等于 `k` 的子数组。

---

## 2. 只记录是否出现，而不是出现次数

错误思路：

```python
prefix_sums = set()
```

因为同一个前缀和可能出现多次。

每一次出现都对应一个不同的潜在子数组起点。

所以必须记录：

```text
prefix_sum -> frequency
```

而不是只保存：

```text
prefix_sum 是否存在
```

---

## 3. 先更新哈希表，再查询

错误：

```python
prefix_count[cur_sum] += 1
res += prefix_count[cur_sum - k]
```

正确：

```python
res += prefix_count.get(cur_sum - k, 0)
prefix_count[cur_sum] = prefix_count.get(cur_sum, 0) + 1
```

必须保证：

> 当前前缀和不能被当成“过去的前缀和”。

---

## 4. 尝试使用普通滑动窗口

由于存在：

```text
负数
```

窗口和没有单调性。

所以不能通过：

```text
sum > k → 缩小窗口
sum < k → 扩大窗口
```

来保证正确性。

---

## 5. 忘记题目问的是“数量”

题目不是要求返回：

```text
子数组本身
```

也不是：

```text
起始下标
```

而是：

```text
合法子数组的总数量
```

所以当某个历史前缀和出现过：

```text
x
```

次时，需要：

```python
res += x
```

而不是：

```python
res += 1
```

---

# 方法对比

| 方法 | 核心思想 | 时间复杂度 | 空间复杂度 | 推荐度 |
|---|---|---:|---:|---|
| 暴力枚举 | 枚举所有起点和终点 | `O(n²)` | `O(1)` | 一般 |
| 前缀和 + 哈希表 | 查找 `cur_sum - k` 的历史出现次数 | `O(n)` | `O(n)` | **最推荐** |

---

# 面试总结

这道题最重要的推导过程是：

```text
某个子数组和为 k
        ↓
右前缀和 - 左前缀和 = k
        ↓
左前缀和 = 当前前缀和 - k
        ↓
需要快速知道某个历史前缀和出现过多少次
        ↓
使用 Hash Map
```

最终模板：

```python
prefix_count = {0: 1}
cur_sum = 0
res = 0

for num in nums:
    cur_sum += num

    res += prefix_count.get(cur_sum - k, 0)

    prefix_count[cur_sum] = prefix_count.get(cur_sum, 0) + 1
```

建议重点记住三个关键点：

```text
1. prefix_count = {0: 1}
2. 查找 cur_sum - k
3. 必须先查找，再更新当前 cur_sum
```

时间复杂度：

\[
\boxed{O(n)}
\]

空间复杂度：

\[
\boxed{O(n)}
\]
