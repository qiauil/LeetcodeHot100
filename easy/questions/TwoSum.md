# Two Sum（两数之和）

## 题目描述

给定一个整数数组 `nums` 和一个整数 `target`，请返回两个下标 `i` 和 `j`，使得：

```
nums[i] + nums[j] == target
```

并且：

```
i != j
```

可以假设：**每组输入都恰好存在唯一一对下标满足条件。**

返回答案时，要求**较小的下标在前**。

------

# 1. 暴力枚举（Brute Force）

## 思路

最直接的方法就是检查数组中所有不同的元素组合。

对于每个位置 `i`，再检查它后面的每一个位置 `j`。如果：

```
nums[i] + nums[j] == target
```

就说明找到了答案。

这种方法非常直观，但效率较低，因为最坏情况下几乎需要检查所有的元素对。

## 算法步骤

1. 使用下标 `i` 遍历数组。
2. 对于每个 `i`，使用 `j` 从 `i + 1` 开始遍历。
3. 检查 `nums[i] + nums[j]` 是否等于 `target`。
4. 如果相等，返回 `[i, j]`。
5. 题目保证一定存在唯一解，因此理论上一定能够找到答案。

```
from typing import List


class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        # 枚举第一个元素的位置
        for i in range(len(nums)):
            # j 从 i + 1 开始：
            # 1. 避免使用同一个元素两次
            # 2. 避免重复检查，例如 (0, 1) 和 (1, 0)
            for j in range(i + 1, len(nums)):
                if nums[i] + nums[j] == target:
                    return [i, j]

        # 题目保证一定存在答案，因此正常情况下不会执行到这里
        return []
```

## 为什么 `j` 从 `i + 1` 开始？

如果让 `j` 从 `0` 开始，会产生两个问题：

第一，可能检查同一个元素：

```
i = 2, j = 2
```

但题目要求：

```
i != j
```

第二，会重复检查同一对元素：

```
(i=1, j=3)
(i=3, j=1)
```

因此从 `i + 1` 开始既可以避免重复，也可以保证两个下标不同。

## 时间与空间复杂度

- 时间复杂度：**O(n²)**
- 空间复杂度：**O(1)**

最坏情况下，我们需要检查大约：

```
n × (n - 1) / 2
```

组元素，因此是 `O(n²)`。

------

# 2. 排序 + 双指针（Sorting + Two Pointers）

## 思路

如果数组已经按照从小到大的顺序排列，那么可以使用两个指针：

```
左指针                     右指针
   ↓                           ↓
[小, 小, 小, ..., 大, 大, 大]
```

计算：

```
nums[left] + nums[right]
```

然后根据结果移动指针：

- 如果和太小，说明需要更大的数，因此移动左指针；
- 如果和太大，说明需要更小的数，因此移动右指针；
- 如果正好等于 `target`，则找到答案。

问题在于：**题目要求返回原数组中的下标。**

因此不能简单地直接排序 `nums`，而是需要保存：

```
(元素值, 原始下标)
```

然后按照元素值排序。

## 算法步骤

1. 将数组转换为：

```
[value, original_index]
```

1. 根据 `value` 排序。
2. 设置：
   - 左指针 `i = 0`
   - 右指针 `j = n - 1`
3. 计算两个位置元素之和：
   - 等于 `target`：找到答案；
   - 小于 `target`：`i += 1`；
   - 大于 `target`：`j -= 1`。
4. 返回两个元素在原数组中的下标。

```
from typing import List


class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        # A 中保存：
        # [元素值, 元素在原数组中的下标]
        A = []

        for i, num in enumerate(nums):
            A.append([num, i])

        # Python 对列表排序时，
        # 默认首先根据第一个元素排序
        A.sort()

        left = 0
        right = len(nums) - 1

        while left < right:
            current_sum = A[left][0] + A[right][0]

            if current_sum == target:
                index1 = A[left][1]
                index2 = A[right][1]

                # 题目要求较小下标在前
                return [min(index1, index2), max(index1, index2)]

            elif current_sum < target:
                # 当前和太小，需要更大的数
                left += 1

            else:
                # 当前和太大，需要更小的数
                right -= 1

        return []
```

## 为什么双指针可以工作？

假设数组已经排序：

```
[2, 4, 6, 8, 11]
 ↑             ↑
left         right
```

如果：

```
2 + 11 < target
```

那么 `2` 和数组中任何比 `11` 更小的元素相加，只会得到更小的结果。

所以 `2` 不可能组成答案。

因此可以安全地执行：

```
left += 1
```

类似地，如果：

```
2 + 11 > target
```

那么 `11` 太大了，需要尝试更小的右侧元素：

```
right -= 1
```

这就是双指针能够把原本可能需要 `O(n²)` 的搜索压缩成 `O(n)` 扫描的原因。

## `enumerate()` 说明

Python 的：

```
enumerate(nums)
```

可以同时获得元素的：

```
下标 + 值
```

例如：

```
nums = [10, 20, 30]

for i, num in enumerate(nums):
    print(i, num)
```

输出：

```
0 10
1 20
2 30
```

因此：

```
for i, num in enumerate(nums):
```

通常比：

```
for i in range(len(nums)):
    num = nums[i]
```

更加简洁。

## `sort()` 说明

对于：

```
A = [
    [5, 0],
    [2, 1],
    [8, 2]
]
```

执行：

```
A.sort()
```

之后：

```
[
    [2, 1],
    [5, 0],
    [8, 2]
]
```

Python 默认会按照子列表中的第一个元素进行比较；如果第一个元素相同，再比较第二个元素。

## 时间与空间复杂度

- 时间复杂度：**O(n log n)**
- 空间复杂度：**O(n)**

其中排序占主要开销：

```
O(n log n)
```

双指针扫描本身只需要：

```
O(n)
```

因此总体为：

```
O(n log n)
```

------

# 3. 哈希表：两次遍历（Hash Map — Two Pass）

## 思路

对于当前位置的数字：

```
n = nums[i]
```

如果它是答案的一部分，那么另外一个数字一定应该是：

```
target - n
```

这个值通常称为 **complement（补数 / 配对值）**。

例如：

```
target = 9
当前元素 = 2
```

那么我们需要寻找：

```
9 - 2 = 7
```

因此问题就变成了：

> 能否快速判断数组中是否存在 `7`？

哈希表非常适合解决这个问题，因为平均情况下，字典中的查找和插入都是 `O(1)`。

## 算法步骤

第一遍：

把所有数字以及对应下标保存到哈希表：

```
值 -> 下标
```

第二遍：

对于每个元素 `nums[i]`：

1. 计算：

```
diff = target - nums[i]
```

1. 检查 `diff` 是否存在于哈希表。
2. 同时确保找到的下标不是当前下标 `i`。
3. 如果满足条件，则返回答案。

```
from typing import List


class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        # 哈希表：
        # 数值 -> 下标
        indices = {}

        # 第一遍：记录所有元素的位置
        for i, n in enumerate(nums):
            indices[n] = i

        # 第二遍：寻找当前元素所需要的补数
        for i, n in enumerate(nums):
            diff = target - n

            # diff 必须存在，并且不能使用当前元素自己
            if diff in indices and indices[diff] != i:
                j = indices[diff]

                # 保证较小下标在前
                return [min(i, j), max(i, j)]

        return []
```

## 为什么要检查 `indices[diff] != i`？

考虑：

```
nums = [3, 4, 5]
target = 6
```

对于：

```
nums[0] = 3
```

有：

```
diff = 6 - 3 = 3
```

哈希表中确实存在 `3`。

但是这个 `3` 就是：

```
nums[0]
```

也就是我们正在使用的元素本身。

如果不检查下标，就可能错误地返回：

```
[0, 0]
```

因此两次遍历版本需要额外保证：

```
indices[diff] != i
```

## 哈希表 / `dict` 说明

Python 中通常使用：

```
dict
```

作为哈希表。

例如：

```
indices = {}

indices[5] = 2
indices[10] = 7
```

此时：

```
indices[5]
```

得到：

```
2
```

判断某个 key 是否存在：

```
if 5 in indices:
```

平均情况下：

- 插入：`O(1)`
- 查找：`O(1)`
- 删除：`O(1)`

因此哈希表是 Two Sum 这类“快速寻找对应值”问题的经典工具。

## 时间与空间复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

虽然遍历了两次数组：

```
O(n) + O(n)
```

但常数会被忽略，因此仍然是：

```
O(n)
```

------

# 4. 哈希表：一次遍历（Hash Map — One Pass）

## 思路

这是这道题最经典、也是面试中通常最推荐的解法。

核心思想是：

> 在处理当前数字时，只去寻找之前已经见过的数字。

假设当前：

```
n = nums[i]
```

我们需要：

```
diff = target - n
```

如果 `diff` 已经存在于哈希表，那么说明之前已经看到过一个数字，可以与当前数字组成答案。

如果不存在，则把当前数字加入哈希表，供后面的元素使用。

这样只需要扫描一次数组。

## 算法步骤

1. 创建哈希表：

```
数字 -> 下标
```

1. 从左到右遍历数组。
2. 对于当前数字 `n`，计算：

```
diff = target - n
```

1. 如果 `diff` 已经存在于哈希表：
   - 返回之前 `diff` 的下标；
   - 返回当前下标 `i`。
2. 如果不存在：
   - 把当前数字和下标加入哈希表。

```
from typing import List


class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        # 保存之前已经遍历过的元素：
        # 数值 -> 下标
        prev_map = {}

        for i, n in enumerate(nums):
            # 当前元素需要寻找的配对值
            diff = target - n

            # 如果之前已经出现过 diff，
            # 那么 diff + n == target
            if diff in prev_map:
                # prev_map 中的下标一定小于当前 i，
                # 因此天然满足较小下标在前
                return [prev_map[diff], i]

            # 当前元素暂时没有找到配对，
            # 将它记录下来供后面的元素使用
            prev_map[n] = i
```

------

# 一次遍历示例

假设：

```
nums = [2, 7, 11, 15]
target = 9
```

初始：

```
prev_map = {}
```

处理：

```
i = 0
n = 2
```

需要：

```
diff = 9 - 2 = 7
```

但是：

```
7 不在 prev_map 中
```

因此保存：

```
prev_map = {
    2: 0
}
```

继续：

```
i = 1
n = 7
```

计算：

```
diff = 9 - 7 = 2
```

发现：

```
2 在 prev_map 中
```

并且：

```
prev_map[2] = 0
```

因此返回：

```
[0, 1]
```

------

# 为什么一定要“先查找，再插入”？

这点非常重要。

正确顺序：

```
if diff in prev_map:
    return [prev_map[diff], i]

prev_map[n] = i
```

而不是先执行：

```
prev_map[n] = i
```

考虑：

```
nums = [3]
target = 6
```

如果先把当前 `3` 放入哈希表：

```
prev_map = {3: 0}
```

再计算：

```
diff = 6 - 3 = 3
```

就会误以为找到了答案：

```
[0, 0]
```

这违反了：

```
i != j
```

而“先查找，再插入”天然保证哈希表中只包含**当前位置之前的元素**，因此不可能使用同一个元素两次。

------

# 时间与空间复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

我们只遍历一次数组，每次进行一次哈希表查找和插入。

由于 Python `dict` 的查找和插入平均为 `O(1)`：

```
n × O(1) = O(n)
```

------

# 四种方法对比

| 方法           | 时间复杂度 | 空间复杂度 | 特点                 |
| -------------- | ---------- | ---------- | -------------------- |
| 暴力枚举       | O(n²)      | O(1)       | 最直观，但速度慢     |
| 排序 + 双指针  | O(n log n) | O(n)       | 利用有序数组和双指针 |
| 哈希表两次遍历 | O(n)       | O(n)       | 思路直接             |
| 哈希表一次遍历 | **O(n)**   | **O(n)**   | 最推荐、最经典       |

对于这道题，面试中通常优先选择：

```
Hash Map + One Pass
```

因为它同时具有：

- `O(n)` 时间复杂度；
- 代码简洁；
- 不需要排序；
- 天然避免重复使用当前元素；
- 天然按照较小下标在前返回答案。

------

# 常见错误

## 1. 使用同一个元素两次

错误写法：

```
# 错误：
# 当 nums[i] * 2 == target 时，
# 可能错误返回 [i, i]
if diff in indices:
    return [i, indices[diff]]
```

对于两次遍历方法，需要额外判断：

```
if diff in indices and indices[diff] != i:
    return [i, indices[diff]]
```

更推荐使用一次遍历：

```
if diff in prev_map:
    return [prev_map[diff], i]

prev_map[n] = i
```

因为 `prev_map` 中只保存之前访问过的元素。

------

## 2. 返回数字而不是下标

题目要求返回：

```
indices
```

而不是：

```
values
```

例如：

```
nums = [2, 7, 11, 15]
target = 9
```

正确答案：

```
[0, 1]
```

而不是：

```
[2, 7]
```

------

## 3. 重复数字的处理

例如：

```
nums = [3, 3]
target = 6
```

正确答案是：

```
[0, 1]
```

一次遍历版本可以自然处理这种情况：

第一次：

```
n = 3
prev_map = {}
```

没有找到另一个 `3`，于是：

```
prev_map = {3: 0}
```

第二次：

```
n = 3
diff = 3
```

发现：

```
3 in prev_map
```

于是返回：

```
[0, 1]
```

------

## 4. 补数计算错误

正确公式：

```
diff = target - nums[i]
```

因为我们要求：

```
nums[i] + diff = target
```

移项得到：

```
diff = target - nums[i]
```

不要错误地写成：

```
diff = nums[i] - target
```

------

# 面试中的推荐写法

如果面试官直接让你解决 Two Sum，可以优先写下面这个版本：

```
from typing import List


class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        seen = {}  # 数值 -> 之前出现的位置

        for i, num in enumerate(nums):
            complement = target - num

            if complement in seen:
                return [seen[complement], i]

            seen[num] = i
```

可以这样解释核心逻辑：

> 对于当前数字 `num`，我需要找到 `target - num`。
> 我使用哈希表保存之前出现过的数字以及对应下标。
> 如果补数已经出现，就直接返回它的下标和当前下标；否则将当前数字加入哈希表。
> 每个元素只处理一次，所以时间复杂度是 `O(n)`，额外空间复杂度是 `O(n)`。

------

# 算法总结

Two Sum 最值得掌握的并不仅仅是这道题本身，而是一个非常常见的算法模式：

```
寻找两个元素满足某个关系
        ↓
固定当前元素
        ↓
计算“我需要的另一个元素”
        ↓
使用 Hash Map 快速寻找它
```

在 Two Sum 中：

```
当前值 = x
目标值 = target

需要寻找的值：
target - x
```

因此可以把原本的：

```
“枚举所有两个数的组合”
```

转化为：

```
“遍历每个数 + O(1) 查询它需要的另一个数”
```

这正是时间复杂度从：

```
O(n²)
```

降低到：

```
O(n)
```

的关键。