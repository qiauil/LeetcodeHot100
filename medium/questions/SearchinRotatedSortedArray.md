# 搜索旋转排序数组

给定一个长度为 `n`、原本按升序排列的数组。该数组经过了 `1` 到 `n` 次旋转。

例如：

```
原数组：[1, 2, 3, 4, 5, 6]

旋转 1 次：[6, 1, 2, 3, 4, 5]
旋转 3 次：[4, 5, 6, 1, 2, 3]
旋转 6 次：[1, 2, 3, 4, 5, 6]
```

给定旋转后的排序数组 `nums` 和整数 `target`：

- 如果 `target` 存在于 `nums` 中，返回它的下标。
- 如果不存在，返回 `-1`。

可以假设 `nums` 中的所有元素都互不相同。

使用 `O(n)` 时间完成搜索非常简单。请设计一个时间复杂度为 `O(log n)` 的算法。

## 示例

```
输入：nums = [4, 5, 6, 7, 0, 1, 2], target = 0
输出：4
输入：nums = [4, 5, 6, 7, 0, 1, 2], target = 3
输出：-1
输入：nums = [1], target = 1
输出：0
```

## 前置知识

解决本题之前，建议掌握以下内容：

- **二分查找**：将搜索范围不断缩小一半，从而达到 `O(log n)` 的时间复杂度。
- **数组和下标**：理解如何通过下标访问数组元素。
- **有序数组的性质**：旋转后的数组虽然整体不再有序，但可以被分成两个分别递增的子数组。
- **旋转点**：旋转后数组中最小元素所在的位置，也是两个递增子数组的分界点。

例如：

```
nums = [4, 5, 6, 7, 0, 1, 2]
                     ↑
                   旋转点
```

它可以分成：

```
[4, 5, 6, 7] 和 [0, 1, 2]
```

------

## 方法一：暴力搜索

### 思路

最直接的方法是从左到右依次检查数组中的每个元素。

- 如果当前元素等于 `target`，返回当前下标。
- 如果遍历完整个数组仍未找到，返回 `-1`。

这种方法一定可以得到正确答案，但没有利用数组的有序性质。

### 代码

```
from typing import List


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        # enumerate(nums) 会同时返回元素的下标和值
        for index, value in enumerate(nums):
            if value == target:
                return index

        return -1
```

也可以使用原答案中的 `range` 写法：

```
from typing import List


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        # range(len(nums)) 依次生成 0 到 len(nums) - 1
        for i in range(len(nums)):
            if nums[i] == target:
                return i

        return -1
```

### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

------

## 方法二：先找旋转点，再分别搜索两部分

### 思路

旋转排序数组可以看作两个递增数组拼接在一起：

```
[4, 5, 6, 7] + [0, 1, 2]
```

因此可以：

1. 使用二分查找找到最小元素，也就是旋转点。
2. 对旋转点左边的子数组进行普通二分查找。
3. 如果没有找到，再对右边的子数组进行二分查找。

虽然最多进行了三次二分查找，但总时间复杂度仍然是 `O(log n)`，因为常数次数不会改变渐进复杂度。

### 如何找到旋转点

设搜索区间为 `[left, right]`，中点为 `mid`。

比较 `nums[mid]` 和 `nums[right]`：

- 如果 `nums[mid] > nums[right]`，说明最小元素一定在 `mid` 的右边，因此令：

  ```
  left = mid + 1
  ```

- 否则，说明最小元素可能就是 `mid`，也可能在 `mid` 左边，因此令：

  ```
  right = mid
  ```

注意这里不能写成 `right = mid - 1`，因为 `mid` 本身可能就是最小元素。

### 代码

```
from typing import List


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1

        # 第一次二分查找：寻找最小元素的下标
        while left < right:
            mid = left + (right - left) // 2

            if nums[mid] > nums[right]:
                # 最小元素一定在 mid 右侧
                left = mid + 1
            else:
                # mid 可能就是最小元素，因此不能排除 mid
                right = mid

        pivot = left

        def binary_search(left: int, right: int) -> int:
            """在 nums[left:right + 1] 中执行普通二分查找。"""
            while left <= right:
                mid = left + (right - left) // 2

                if nums[mid] == target:
                    return mid
                elif nums[mid] < target:
                    left = mid + 1
                else:
                    right = mid - 1

            return -1

        # 在旋转点左侧搜索
        result = binary_search(0, pivot - 1)
        if result != -1:
            return result

        # 左侧没有找到，再搜索旋转点及其右侧
        return binary_search(pivot, len(nums) - 1)
```

### 复杂度分析

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

虽然代码中定义了一个内部函数，但它没有使用递归，也没有创建与输入规模相关的数据结构，因此空间复杂度仍为 `O(1)`。

------

## 方法三：两次二分查找

### 思路

这个方法同样先找到旋转点，但找到旋转点后，不需要对两部分都进行搜索。

由于旋转点右侧是一个递增数组，可以先判断目标值是否满足：

```
nums[pivot] <= target <= nums[-1]
```

- 如果满足，目标值只可能位于右侧。
- 否则，目标值只可能位于左侧。

确定搜索范围后，只进行一次普通二分查找。

### 算法步骤

1. 使用二分查找找到旋转点 `pivot`。
2. 根据目标值确定应该搜索哪个递增区间。
3. 在选定的区间内执行普通二分查找。
4. 找到则返回下标，否则返回 `-1`。

### 代码

```
from typing import List


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1

        # 第一次二分查找：寻找旋转点
        while left < right:
            mid = left + (right - left) // 2

            if nums[mid] > nums[right]:
                left = mid + 1
            else:
                right = mid

        pivot = left

        # 根据 target 的取值确定搜索范围
        left, right = 0, len(nums) - 1

        if nums[pivot] <= target <= nums[right]:
            # target 可能在旋转点右侧
            left = pivot
        else:
            # target 可能在旋转点左侧
            right = pivot - 1

        # 第二次二分查找：在选定的有序区间中查找 target
        while left <= right:
            mid = left + (right - left) // 2

            if nums[mid] == target:
                return mid
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        return -1
```

### 复杂度分析

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

第一次二分查找旋转点需要 `O(log n)`，第二次搜索目标值也需要 `O(log n)`：

```
O(log n) + O(log n) = O(log n)
```

------

## 方法四：一次二分查找

### 思路

这是本题更常见、也更值得掌握的解法。

旋转后的数组整体不一定有序，但对于任意中点 `mid`，下面两部分中至少有一部分一定是有序的：

```
[left, mid] 或 [mid, right]
```

因此，每轮二分查找可以：

1. 判断左半部分还是右半部分有序。
2. 判断 `target` 是否位于有序的那一半。
3. 根据判断结果排除另一半。

### 情况一：左半部分有序

如果：

```
nums[left] <= nums[mid]
```

说明 `[left, mid]` 是递增的。

接着判断目标值是否位于：

```
nums[left] <= target < nums[mid]
```

- 如果在这个范围中，向左搜索。
- 否则，向右搜索。

### 情况二：右半部分有序

如果左半部分无序，那么 `[mid, right]` 一定有序。

判断目标值是否位于：

```
nums[mid] < target <= nums[right]
```

- 如果在这个范围中，向右搜索。
- 否则，向左搜索。

### 代码

```
from typing import List


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1

        while left <= right:
            mid = left + (right - left) // 2

            # 找到目标值
            if nums[mid] == target:
                return mid

            # 情况一：左半部分 [left, mid] 有序
            if nums[left] <= nums[mid]:
                # target 位于左侧有序区间
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1

            # 情况二：右半部分 [mid, right] 有序
            else:
                # target 位于右侧有序区间
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1

        return -1
```

### 复杂度分析

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

### 为什么每次至少有一半有序

旋转数组只有一个位置破坏了原来的递增关系，也就是最大元素与最小元素的连接处。

因此，当我们从中点将当前区间分成两部分时，旋转点最多只能出现在其中一部分。另一部分不包含旋转点，所以一定保持递增。

例如：

```
nums = [4, 5, 6, 7, 0, 1, 2]
left = 0, mid = 3, right = 6

左半部分：[4, 5, 6, 7]    有序
右半部分：[7, 0, 1, 2]    包含旋转点
```

再例如：

```
nums = [6, 7, 0, 1, 2, 4, 5]
left = 0, mid = 3, right = 6

左半部分：[6, 7, 0, 1]    包含旋转点
右半部分：[1, 2, 4, 5]    有序
```

------

## Python 语法说明

### `List[int]`

```
from typing import List

nums: List[int]
```

`List[int]` 是类型注解，表示 `nums` 应该是一个元素类型为整数的列表。

它主要用于提高代码的可读性以及帮助编辑器进行类型检查，不会改变算法的运行方式。

在 Python 3.9 及以上版本中，也可以写成：

```
nums: list[int]
```

某些在线判题平台会自动导入 `List`；如果在本地运行，则通常需要手动添加：

```
from typing import List
```

### `//` 整数除法

```
mid = (left + right) // 2
```

`//` 表示向下取整的整数除法，用于计算中间下标。

也可以写成：

```
mid = left + (right - left) // 2
```

后者在整数可能溢出的编程语言中更加安全。虽然 Python 的整数通常不会溢出，但这种写法具有更好的通用性。

### 连续比较

Python 支持连续比较：

```
nums[pivot] <= target <= nums[right]
```

它等价于：

```
nums[pivot] <= target and target <= nums[right]
```

------

## 常见错误

### 1. 判断左半部分有序时使用严格小于号

一次二分查找中应该写：

```
nums[left] <= nums[mid]
```

而不是：

```
nums[left] < nums[mid]
```

当搜索区间只剩下一个或两个元素时，可能出现 `left == mid`。此时使用 `<` 会错误地认为左半部分无序。

使用 `<=` 可以正确处理这种边界情况。

### 2. 混淆“寻找旋转点”和“寻找目标值”

寻找旋转点时使用的是：

```
if nums[mid] > nums[right]:
    left = mid + 1
else:
    right = mid
```

它的目标是找到数组中的最小元素。

寻找 `target` 时使用的则是普通二分查找：

```
if nums[mid] < target:
    left = mid + 1
else:
    right = mid - 1
```

两者的搜索目标和区间更新方式不同，不能混用。

### 3. 寻找旋转点时错误地排除 `mid`

当：

```
nums[mid] <= nums[right]
```

只能说明旋转点位于 `[left, mid]` 中，而 `mid` 自己也可能是最小元素。

因此应该写：

```
right = mid
```

不能写成：

```
right = mid - 1
```

### 4. 忽略旋转后仍与原数组相同的情况

数组旋转 `n` 次后会恢复原状，此时旋转点为 `0`。

例如：

```
nums = [1, 2, 3, 4, 5]
```

寻找旋转点的算法应该返回下标 `0`，后续搜索也必须能够正常执行。

### 5. 忽略单元素数组

例如：

```
nums = [1], target = 1
```

正确答案应该是 `0`。

又如：

```
nums = [1], target = 2
```

正确答案应该是 `-1`。

这也是判断有序部分时使用 `nums[left] <= nums[mid]` 的重要原因之一。

------

## 方法总结

| 方法                 | 核心思路                                 | 时间复杂度 | 空间复杂度 |
| -------------------- | ---------------------------------------- | ---------- | ---------- |
| 暴力搜索             | 依次检查每个元素                         | `O(n)`     | `O(1)`     |
| 找旋转点后搜索两部分 | 找到最小元素，再依次搜索两个有序区间     | `O(log n)` | `O(1)`     |
| 两次二分查找         | 找到旋转点，判断目标所在区间，再搜索一次 | `O(log n)` | `O(1)`     |
| 一次二分查找         | 每轮判断哪一半有序并排除另一半           | `O(log n)` | `O(1)`     |

面试中推荐优先掌握“一次二分查找”。它不需要显式寻找旋转点，代码更紧凑，也能直接体现对旋转排序数组性质的理解。