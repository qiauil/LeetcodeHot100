# 寻找旋转排序数组中的最小值

给定一个长度为 `n` 的数组，它原本按升序排列，之后被旋转了 `1` 到 `n` 次。

例如，数组 `nums = [1, 2, 3, 4, 5, 6]`：

- 旋转 `4` 次后，可能变为 `[3, 4, 5, 6, 1, 2]`。
- 旋转 `6` 次后，仍然是 `[1, 2, 3, 4, 5, 6]`。

旋转 `4` 次表示将数组末尾的四个元素移动到数组开头；旋转 `n` 次后，数组会恢复原状。

假设旋转排序数组 `nums` 中的所有元素都互不相同，请返回数组中的最小元素。

使用 `O(n)` 时间复杂度解决这道题非常简单。你能否设计一个时间复杂度为 `O(log n)` 的算法？

------

## 方法一：暴力遍历

### 思路

旋转只会改变元素的位置，不会改变数组中包含的元素。

因此，最直接的方法是遍历整个数组，记录遇到的最小值。Python 内置函数 `min()` 已经实现了这一过程。

### 代码

```
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        # min() 会遍历整个数组并返回其中的最小值
        return min(nums)
```

### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

### Python 函数说明：`min()`

`min(iterable)` 用于返回可迭代对象中的最小元素。例如：

```
min([3, 4, 5, 1, 2])  # 返回 1
```

虽然代码中只写了一行，但 `min()` 仍然需要检查数组中的每个元素，因此时间复杂度是 `O(n)`，不满足题目要求的 `O(log n)`。

------

## 方法二：二分查找并记录当前最小值

### 核心思路

旋转排序数组可以被看成两个升序区间。例如：

```
[4, 5, 6, 7, 0, 1, 2]
 └──升序──┘  └升序┘
```

数组的最小值位于第二个升序区间的起点，也就是旋转点。

在每轮二分查找中：

- 如果当前搜索区间已经整体有序，那么该区间的最小值就是 `nums[left]`。
- 如果 `nums[mid] >= nums[left]`，说明从 `left` 到 `mid` 是有序的，旋转点位于右半部分。
- 否则，说明旋转点位于左半部分。

搜索过程中同时维护已经见过的最小值。

### 算法步骤

1. 使用 `res` 保存当前找到的最小值。
2. 初始化左右边界 `left = 0`、`right = n - 1`。
3. 当 `left <= right` 时：
   - 如果 `nums[left] < nums[right]`，当前区间已经严格递增：
     - 使用 `nums[left]` 更新答案；
     - 结束搜索。
   - 计算中间位置 `mid`。
   - 使用 `nums[mid]` 更新答案。
   - 如果左半部分有序，将搜索范围移动到右半部分。
   - 否则，将搜索范围移动到左半部分。
4. 返回 `res`。

### 代码

```
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        # 先将第一个元素作为当前最小值
        res = nums[0]

        left, right = 0, len(nums) - 1

        while left <= right:
            # 当前区间已经严格递增，
            # 因此 nums[left] 就是该区间的最小值
            if nums[left] < nums[right]:
                res = min(res, nums[left])
                break

            mid = left + (right - left) // 2

            # nums[mid] 也可能就是整个数组的最小值
            res = min(res, nums[mid])

            if nums[mid] >= nums[left]:
                # 左半部分有序，旋转点只能位于右半部分
                left = mid + 1
            else:
                # nums[mid] < nums[left]，
                # 说明旋转点位于左半部分或 mid 位置
                right = mid - 1

        return res
```

### 复杂度分析

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

### 为什么可以使用 `right = mid - 1`？

在这个写法中，执行边界更新之前，已经通过下面这行代码记录了 `nums[mid]`：

```
res = min(res, nums[mid])
```

所以即使 `mid` 恰好是最小值，随后将它排除出搜索范围也不会丢失答案。

如果不额外使用 `res` 保存答案，就不能随意排除 `mid`。

------

## 方法三：二分查找旋转点（推荐）

### 核心思路

这是一种更加简洁的二分查找写法。它不需要额外维护答案，而是不断缩小搜索区间，直到区间中只剩下最小元素。

关键是比较：

```
nums[mid] 和 nums[right]
```

由于所有元素互不相同，因此只有两种情况。

### 情况一：`nums[mid] < nums[right]`

例如：

```
[6, 7, 1, 2, 3, 4, 5]
       ↑           ↑
      mid        right
```

从 `mid` 到 `right` 是一个升序区间，说明最小值可能是 `nums[mid]`，也可能位于 `mid` 左边。

因此：

```
right = mid
```

这里不能写成 `right = mid - 1`，因为 `mid` 本身可能就是最小值。

### 情况二：`nums[mid] > nums[right]`

例如：

```
[4, 5, 6, 7, 0, 1, 2]
          ↑           ↑
         mid        right
```

`nums[mid]` 大于 `nums[right]`，说明旋转点一定在 `mid` 的右边，因此 `mid` 不可能是最小值。

所以：

```
left = mid + 1
```

### 算法步骤

1. 初始化 `left = 0`、`right = n - 1`。
2. 当 `left < right` 时：
   - 计算中间位置 `mid`。
   - 如果 `nums[mid] < nums[right]`，令 `right = mid`。
   - 否则，令 `left = mid + 1`。
3. 循环结束时，`left == right`，并且它们都指向最小元素。
4. 返回 `nums[left]`。

### 代码

```
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        left, right = 0, len(nums) - 1

        # 循环结束条件是 left == right
        while left < right:
            # 这种写法可以避免某些语言中的整数溢出问题
            mid = left + (right - left) // 2

            if nums[mid] < nums[right]:
                # 最小值可能位于 mid，
                # 因此不能排除 mid
                right = mid
            else:
                # 因为元素互不相同，这里实际上是 nums[mid] > nums[right]
                # 最小值一定在 mid 的右边
                left = mid + 1

        # left 和 right 最终指向同一个位置，即最小元素的位置
        return nums[left]
```

### 复杂度分析

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

------

## 正确性说明

在二分查找过程中，始终维持下面的不变量：

> 数组的最小值一定存在于闭区间 `[left, right]` 中。

每轮搜索都会安全地缩小这个区间：

- 当 `nums[mid] < nums[right]` 时，保留 `[left, mid]`。
- 当 `nums[mid] > nums[right]` 时，保留 `[mid + 1, right]`。

每次至少排除一个元素，因此最终一定会得到：

```
left == right
```

此时区间中只剩一个元素，根据不变量，它一定是数组中的最小值。

------

## 示例推演

以数组为例：

```
nums = [4, 5, 6, 7, 0, 1, 2]
```

| `left` | `mid` | `right` | 比较结果                    | 边界更新    |
| ------ | ----- | ------- | --------------------------- | ----------- |
| 0      | 3     | 6       | `nums[3] = 7 > nums[6] = 2` | `left = 4`  |
| 4      | 5     | 6       | `nums[5] = 1 < nums[6] = 2` | `right = 5` |
| 4      | 4     | 5       | `nums[4] = 0 < nums[5] = 1` | `right = 4` |

最终：

```
left == right == 4
nums[4] == 0
```

所以最小值为 `0`。

------

## 常见错误

### 1. 错误地排除 `mid`

当：

```
nums[mid] < nums[right]
```

最小值有可能正好位于 `mid`，因此必须写成：

```
right = mid
```

不能写成：

```
right = mid - 1
```

相反，当：

```
nums[mid] > nums[right]
```

可以确定 `mid` 不是最小值，因此应该写成：

```
left = mid + 1
```

------

### 2. 循环条件与边界更新不匹配

方法三应该使用：

```
while left < right:
```

因为循环目标是让两个边界最终重合。

如果使用 `left <= right`，却仍然采用 `right = mid` 这样的边界更新方式，可能导致死循环或数组越界。

------

### 3. 没有考虑数组未旋转的情况

数组可能旋转了 `n` 次，此时它与原数组相同：

```
[1, 2, 3, 4, 5]
```

方法三不需要单独处理这种情况，仍然可以正确找到最小值。

如果希望提前结束，也可以加入：

```
if nums[left] < nums[right]:
    return nums[left]
```

由于题目保证元素互不相同，所以只要 `nums[left] < nums[right]`，当前区间就是严格递增的。

不过这个检查只是一种优化，并不是方法三正确运行的必要条件。

------

### 4. 忽略“元素互不相同”的条件

本题保证所有元素唯一，因此比较 `nums[mid]` 和 `nums[right]` 时，不需要处理相等的情况。

如果数组允许存在重复元素，例如：

```
[2, 2, 2, 0, 1, 2]
```

可能出现：

```
nums[mid] == nums[right]
```

这时无法判断最小值位于哪一侧，通常需要执行：

```
right -= 1
```

允许重复元素时，最坏情况下每次只能排除一个元素，所以时间复杂度可能退化为 `O(n)`。

------

## Python 相关说明

### `List[int]`

```
from typing import List
```

`List[int]` 是类型提示，表示参数 `nums` 应该是一个由整数构成的列表：

```
def findMin(self, nums: List[int]) -> int:
```

最后的 `-> int` 表示该函数预期返回一个整数。

类型提示主要用于提高代码可读性和辅助静态检查，通常不会影响程序运行。

### 中间位置的计算

代码使用：

```
mid = left + (right - left) // 2
```

在 Python 中也可以直接写：

```
mid = (left + right) // 2
```

Python 整数不会发生传统意义上的固定宽度溢出，因此两种写法都安全。

不过第一种写法在 Java、C++ 等使用固定宽度整数的语言中更稳妥，也是代码面试中常见的标准写法。

------

## 总结

推荐使用方法三：

```
class Solution:
    def findMin(self, nums: List[int]) -> int:
        left, right = 0, len(nums) - 1

        while left < right:
            mid = left + (right - left) // 2

            if nums[mid] < nums[right]:
                right = mid
            else:
                left = mid + 1

        return nums[left]
```

需要记住的核心边界规则是：

```
nums[mid] < nums[right]  →  right = mid
nums[mid] > nums[right]  →  left = mid + 1
```

第一种情况保留 `mid`，因为它可能是最小值；第二种情况排除 `mid`，因为它一定不是最小值。