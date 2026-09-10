# 在排序数组中查找元素的第一个和最后一个位置

给定一个按非递减顺序排列的整数数组 `nums` 和目标值 `target`，找出 `target` 在数组中的第一个位置和最后一个位置。

如果数组中不存在 `target`，返回 `[-1, -1]`。

题目要求时间复杂度必须为 `O(log n)`。[题目原文](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

## 示例

```
输入：nums = [5, 7, 7, 8, 8, 10], target = 8
输出：[3, 4]
输入：nums = [5, 7, 7, 8, 8, 10], target = 6
输出：[-1, -1]
输入：nums = [], target = 0
输出：[-1, -1]
```

------

## 方法一：线性扫描

### 思路

从左到右遍历数组：

- 第一次遇到 `target` 时，记录开始位置。
- 之后继续遍历，直到元素不再等于 `target`，即可确定结束位置。
- 如果始终没有找到目标值，返回 `[-1, -1]`。

这种方法能够得到正确答案，但没有充分利用数组有序的性质，也不满足题目的 `O(log n)` 要求。

### 代码

```
from typing import List


class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        first = -1
        last = -1

        for index, value in enumerate(nums):
            if value == target:
                # 只在第一次找到 target 时记录起点
                if first == -1:
                    first = index

                # 每次找到 target 都更新终点
                last = index

            # 因为数组有序，超过 target 后可以提前结束
            elif value > target:
                break

        return [first, last]
```

### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

------

## 方法二：普通二分搜索后向两侧扩展

### 思路

首先使用普通二分搜索找到任意一个等于 `target` 的位置。

找到之后：

- 不断向左移动，寻找第一个 `target`。
- 不断向右移动，寻找最后一个 `target`。

虽然查找任意一个目标值只需要 `O(log n)`，但是如果数组中存在大量重复的目标值，向两侧扩展可能需要 `O(n)`。

例如：

```
nums = [8, 8, 8, 8, 8, 8]
target = 8
```

找到中间的 `8` 后，仍然需要检查几乎整个数组。

### 代码

```
from typing import List


class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        left, right = 0, len(nums) - 1
        found = -1

        # 普通二分搜索：找到任意一个 target
        while left <= right:
            mid = left + (right - left) // 2

            if nums[mid] == target:
                found = mid
                break
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        if found == -1:
            return [-1, -1]

        # 从找到的位置向左右扩展
        first = found
        last = found

        while first > 0 and nums[first - 1] == target:
            first -= 1

        while last < len(nums) - 1 and nums[last + 1] == target:
            last += 1

        return [first, last]
```

### 复杂度分析

- 时间复杂度：最坏为 `O(n)`
- 空间复杂度：`O(1)`

因此，这个方法仍然不满足题目的时间复杂度要求。

------

## 方法三：两次二分搜索（推荐）

### 核心思路

普通二分搜索在找到 `target` 后会立即返回。但是本题需要确定左右边界，因此找到目标值后不能马上停止。

分别进行两次二分搜索：

1. 第一次寻找 `target` 的最左位置。
2. 第二次寻找 `target` 的最右位置。

两次搜索的区别只在于找到 `target` 后的处理方式。

### 寻找左边界

当 `nums[mid] == target` 时：

- 将 `mid` 记录为当前答案。
- 继续搜索左半部分。
- 因为左侧可能还存在另一个 `target`。

```
result = mid
right = mid - 1
```

### 寻找右边界

当 `nums[mid] == target` 时：

- 将 `mid` 记录为当前答案。
- 继续搜索右半部分。
- 因为右侧可能还存在另一个 `target`。

```
result = mid
left = mid + 1
```

### 代码

```
from typing import List


class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        def find_boundary(find_left: bool) -> int:
            """
            find_left 为 True：寻找 target 的最左位置
            find_left 为 False：寻找 target 的最右位置
            """
            left, right = 0, len(nums) - 1
            result = -1

            while left <= right:
                mid = left + (right - left) // 2

                if nums[mid] < target:
                    # 当前值太小，target 只能在右边
                    left = mid + 1

                elif nums[mid] > target:
                    # 当前值太大，target 只能在左边
                    right = mid - 1

                else:
                    # 找到了 target，先记录当前位置
                    result = mid

                    if find_left:
                        # 继续搜索左侧，寻找更靠左的 target
                        right = mid - 1
                    else:
                        # 继续搜索右侧，寻找更靠右的 target
                        left = mid + 1

            return result

        first = find_boundary(True)
        last = find_boundary(False)

        return [first, last]
```

### 执行过程示例

对于：

```
nums = [5, 7, 7, 8, 8, 10]
target = 8
```

寻找左边界时：

```
第一次：mid = 2，nums[mid] = 7
7 < 8，搜索右侧

第二次：mid = 4，nums[mid] = 8
记录位置 4，继续搜索左侧

第三次：mid = 3，nums[mid] = 8
记录位置 3，继续搜索左侧

最终左边界为 3
```

寻找右边界时：

```
第一次：mid = 2，nums[mid] = 7
7 < 8，搜索右侧

第二次：mid = 4，nums[mid] = 8
记录位置 4，继续搜索右侧

第三次：mid = 5，nums[mid] = 10
10 > 8，搜索左侧

最终右边界为 4
```

因此返回：

```
[3, 4]
```

### 复杂度分析

- 寻找左边界：`O(log n)`
- 寻找右边界：`O(log n)`
- 总时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

虽然进行了两次二分搜索，但：

```
O(log n) + O(log n) = O(log n)
```

常数系数在大 O 表示法中会被省略。

------

## 方法四：寻找插入位置

### 思路

还可以把问题转换为寻找两个插入位置：

- 左边界：第一个大于或等于 `target` 的位置。
- 右边界：第一个大于 `target` 的位置减一。

例如：

```
nums = [5, 7, 7, 8, 8, 10]
target = 8
```

第一个 `>= 8` 的位置是：

```
index = 3
```

第一个 `> 8` 的位置是：

```
index = 5
```

因此：

```
左边界 = 3
右边界 = 5 - 1 = 4
```

### `lower_bound` 的含义

下面的 `lower_bound(x)` 返回：

> 数组中第一个大于或等于 `x` 的元素位置。

如果所有元素都小于 `x`，则返回 `len(nums)`。

### 代码

```
from typing import List


class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        def lower_bound(value: int) -> int:
            """
            返回第一个满足 nums[index] >= value 的位置。
            搜索区间采用左闭右开形式 [left, right)。
            """
            left, right = 0, len(nums)

            while left < right:
                mid = left + (right - left) // 2

                if nums[mid] < value:
                    # mid 不满足条件，答案一定在右边
                    left = mid + 1
                else:
                    # mid 可能是答案，不能将它排除
                    right = mid

            return left

        # 第一个 >= target 的位置
        first = lower_bound(target)

        # first 越界或对应元素不等于 target，说明目标不存在
        if first == len(nums) or nums[first] != target:
            return [-1, -1]

        # 第一个 >= target + 1 的位置，
        # 也就是第一个严格大于 target 的位置
        last = lower_bound(target + 1) - 1

        return [first, last]
```

### 为什么 `target + 1` 有效？

因为题目中的数组元素都是整数：

```
第一个 >= target + 1 的元素
```

也就是：

```
第一个 > target 的元素
```

因此将其位置减一，就能得到最后一个 `target` 的位置。

### 复杂度分析

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

------

## 使用 Python 的 `bisect` 模块

Python 标准库的 `bisect` 模块提供了在有序数组中寻找插入位置的函数。

```
from bisect import bisect_left, bisect_right
```

- `bisect_left(nums, target)`：返回 `target` 的最左插入位置。
- `bisect_right(nums, target)`：返回 `target` 的最右插入位置，即所有 `target` 的右侧。

### 代码

```
from bisect import bisect_left, bisect_right
from typing import List


class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        # 第一个大于或等于 target 的位置
        first = bisect_left(nums, target)

        # 检查 target 是否真的存在
        if first == len(nums) or nums[first] != target:
            return [-1, -1]

        # bisect_right 返回最后一个 target 后面的位置
        last = bisect_right(nums, target) - 1

        return [first, last]
```

### 示例

```
nums = [5, 7, 7, 8, 8, 10]

bisect_left(nums, 8)   # 3
bisect_right(nums, 8)  # 5
```

因此目标范围是：

```
[3, 5 - 1]  # [3, 4]
```

### 面试建议

`bisect` 写法非常简洁，但面试官通常更希望候选人能够独立实现二分查找。建议优先掌握方法三或方法四，并将 `bisect` 作为补充方案。

------

## 常见错误

### 1. 找到目标值后立即返回

普通二分搜索通常这样写：

```
if nums[mid] == target:
    return mid
```

但本题需要查找边界。找到一个 `target` 并不意味着它就是第一个或最后一个。

寻找左边界时应继续向左：

```
result = mid
right = mid - 1
```

寻找右边界时应继续向右：

```
result = mid
left = mid + 1
```

------

### 2. 找到目标后没有先保存答案

下面的代码是不完整的：

```
if nums[mid] == target:
    right = mid - 1
```

继续向左搜索后，不一定还能找到另一个 `target`。因此必须先保存当前位置：

```
result = mid
right = mid - 1
```

------

### 3. 使用 `left = mid` 或 `right = mid`

当搜索区间采用闭区间 `[left, right]` 时，更新边界应该排除已经检查过的 `mid`：

```
left = mid + 1
right = mid - 1
```

如果写成：

```
left = mid
```

当 `left` 和 `mid` 相等时，搜索区间可能无法继续缩小，从而造成死循环。

------

### 4. 没有检查插入位置是否越界

在插入位置写法中：

```
first = lower_bound(target)
```

`first` 可能等于 `len(nums)`，因此不能直接访问：

```
nums[first]
```

必须先判断：

```
if first == len(nums) or nums[first] != target:
    return [-1, -1]
```

这里利用了 Python 的短路求值：当第一个条件为 `True` 时，不会继续访问 `nums[first]`。

------

### 5. 使用 `list.index()`

虽然可以写：

```
first = nums.index(target)
```

但 `list.index()` 会从头开始线性查找，其时间复杂度是 `O(n)`，不满足题目要求。

------

## 方法对比

| 方法                 | 时间复杂度  | 空间复杂度 | 满足题目要求 |
| -------------------- | ----------- | ---------- | ------------ |
| 线性扫描             | `O(n)`      | `O(1)`     | 否           |
| 二分搜索后向两侧扩展 | 最坏 `O(n)` | `O(1)`     | 否           |
| 两次边界二分搜索     | `O(log n)`  | `O(1)`     | 是           |
| 寻找两个插入位置     | `O(log n)`  | `O(1)`     | 是           |
| Python `bisect`      | `O(log n)`  | `O(1)`     | 是           |

## 总结

这道题的关键不是“找到一个等于 `target` 的元素”，而是找到两个边界：

```
第一个等于 target 的位置
最后一个等于 target 的位置
```

寻找左边界时：

```
result = mid
right = mid - 1
```

寻找右边界时：

```
result = mid
left = mid + 1
```

两次二分搜索是最直观、最适合面试讲解的解法，时间复杂度为 `O(log n)`，空间复杂度为 `O(1)`。