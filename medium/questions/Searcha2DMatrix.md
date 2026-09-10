# 搜索二维矩阵

给定一个 `m × n` 的整数矩阵 `matrix` 和一个整数 `target`。矩阵满足：

- 每一行都按非递减顺序排列。
- 每一行的第一个元素都大于上一行的最后一个元素。

如果 `target` 存在于矩阵中，返回 `True`；否则返回 `False`。

要求设计时间复杂度为 `O(log(m × n))` 的算法。

------

## 方法一：暴力搜索

### 思路

依次检查矩阵中的每个元素：

- 如果当前元素等于 `target`，立即返回 `True`。
- 如果遍历完整个矩阵仍未找到目标值，返回 `False`。

这个方法没有利用矩阵已经排序的性质。

### 代码

```
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        # 逐行遍历矩阵
        for row in matrix:
            # 遍历当前行中的每个元素
            for value in row:
                if value == target:
                    return True

        return False
```

### 复杂度分析

- 时间复杂度：`O(m × n)`
- 空间复杂度：`O(1)`

其中，`m` 是矩阵的行数，`n` 是矩阵的列数。

------

## 方法二：阶梯搜索

### 思路

题目给出的排序条件能够保证：

- 每一行从左到右递增。
- 每一列从上到下递增。

例如，从矩阵的右上角开始：

- 如果当前值大于 `target`，那么当前值下方的元素只会更大，因此应该向左移动。
- 如果当前值小于 `target`，那么当前值左侧的元素只会更小，因此应该向下移动。
- 如果当前值等于 `target`，返回 `True`。

每次移动都会排除一整行或一整列，因此搜索路线看起来像沿着阶梯移动。

### 为什么每一列也是递增的？

根据题意：

```
下一行的第一个元素 > 上一行的最后一个元素
```

而一行中的其他元素都不小于该行的第一个元素，所以对于任意一列，都有：

```
matrix[r + 1][c] > matrix[r][c]
```

因此可以安全地使用阶梯搜索。

### 代码

```
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        # 防止访问空矩阵的 matrix[0] 时发生错误
        if not matrix or not matrix[0]:
            return False

        rows, cols = len(matrix), len(matrix[0])

        # 从右上角开始搜索
        row, col = 0, cols - 1

        while row < rows and col >= 0:
            value = matrix[row][col]

            if value == target:
                return True
            elif value > target:
                # 当前值过大，向左移动
                col -= 1
            else:
                # 当前值过小，向下移动
                row += 1

        return False
```

### 复杂度分析

- 时间复杂度：`O(m + n)`
- 空间复杂度：`O(1)`

搜索过程中最多向下移动 `m` 次、向左移动 `n` 次。

### 补充说明

阶梯搜索是一种很实用的通用方法，但它没有达到题目要求的 `O(log(m × n))` 时间复杂度。

对于接近方阵的矩阵，例如 `m = n`，其复杂度为 `O(n)`；而一维二分搜索只需要 `O(log(n²)) = O(log n)`。

------

## 方法三：两次二分搜索

### 思路

可以将搜索过程分成两个阶段：

1. 二分查找目标值可能出现在哪一行。
2. 在候选行中再次进行二分查找。

对于任意一行：

- 如果 `target` 大于该行最后一个元素，目标只能在更下面的行。
- 如果 `target` 小于该行第一个元素，目标只能在更上面的行。
- 否则，目标值只可能位于当前行。

### 算法步骤

1. 设置行搜索区间 `top = 0`、`bottom = rows - 1`。
2. 对所有行进行二分查找：
   - 若 `target > matrix[row][-1]`，搜索下面的行。
   - 若 `target < matrix[row][0]`，搜索上面的行。
   - 否则，当前行就是唯一的候选行。
3. 如果不存在候选行，返回 `False`。
4. 在候选行中进行普通二分查找。
5. 找到目标值时返回 `True`，否则返回 `False`。

### 代码

```
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        if not matrix or not matrix[0]:
            return False

        rows, cols = len(matrix), len(matrix[0])

        # 第一次二分搜索：寻找候选行
        top, bottom = 0, rows - 1
        candidate_row = -1

        while top <= bottom:
            row = top + (bottom - top) // 2

            if target < matrix[row][0]:
                # 目标值小于当前行的最小值，搜索上方
                bottom = row - 1
            elif target > matrix[row][-1]:
                # 目标值大于当前行的最大值，搜索下方
                top = row + 1
            else:
                # target 位于当前行的取值范围内
                candidate_row = row
                break

        if candidate_row == -1:
            return False

        # 第二次二分搜索：在候选行中查找目标值
        left, right = 0, cols - 1

        while left <= right:
            mid = left + (right - left) // 2
            value = matrix[candidate_row][mid]

            if value == target:
                return True
            elif value < target:
                left = mid + 1
            else:
                right = mid - 1

        return False
```

### 复杂度分析

- 第一次二分搜索：`O(log m)`
- 第二次二分搜索：`O(log n)`
- 总时间复杂度：`O(log m + log n)`，等价于 `O(log(m × n))`
- 空间复杂度：`O(1)`

因为：

```
log m + log n = log(m × n)
```

------

## 方法四：一次二分搜索（推荐）

### 思路

由于：

- 每一行内部有序；
- 下一行的第一个元素大于上一行的最后一个元素；

所以如果按行展开矩阵，便会得到一个完整的有序数组。

例如：

```
matrix = [
    [1,  3,  5],
    [7,  9, 11],
    [13, 15, 17]
]
```

可以将它想象成：

```
[1, 3, 5, 7, 9, 11, 13, 15, 17]
```

不需要真的创建这个一维数组。只要把一维下标转换为二维坐标，就可以直接访问原矩阵。

对于一维下标 `mid`：

```
row = mid // cols
col = mid % cols
```

其中：

- `//` 是整除运算，用于确定元素位于第几行。
- `%` 是取模运算，用于确定元素位于该行的第几列。

### 下标转换示例

假设矩阵有 `4` 列，一维下标为 `6`：

```
row = 6 // 4  # 1
col = 6 % 4   # 2
```

因此，一维数组中的第 `6` 个下标对应：

```
matrix[1][2]
```

### 算法步骤

1. 将矩阵看作一个长度为 `rows × cols` 的有序数组。
2. 设置搜索区间：
   - `left = 0`
   - `right = rows × cols - 1`
3. 计算中间下标 `mid`。
4. 将 `mid` 转换为二维坐标：
   - `row = mid // cols`
   - `col = mid % cols`
5. 比较 `matrix[row][col]` 与 `target`：
   - 相等：返回 `True`
   - 当前值小于目标值：搜索右半部分
   - 当前值大于目标值：搜索左半部分
6. 搜索区间为空时返回 `False`。

### 代码

```
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        if not matrix or not matrix[0]:
            return False

        rows, cols = len(matrix), len(matrix[0])

        # 将矩阵视为长度为 rows * cols 的一维有序数组
        left, right = 0, rows * cols - 1

        while left <= right:
            mid = left + (right - left) // 2

            # 将一维下标转换为二维坐标
            row = mid // cols
            col = mid % cols
            value = matrix[row][col]

            if value == target:
                return True
            elif value < target:
                # 目标值在右半部分
                left = mid + 1
            else:
                # 目标值在左半部分
                right = mid - 1

        return False
```

### 复杂度分析

- 时间复杂度：`O(log(m × n))`
- 空间复杂度：`O(1)`

这个方法只进行一次二分搜索，代码简洁，并且完全满足题目的时间复杂度要求，因此通常是面试中的最佳答案。

------

## Python 语法与类型说明

### `List[List[int]]`

```
from typing import List
```

`List` 用于类型标注：

```
matrix: List[List[int]]
```

表示 `matrix` 是一个二维整数列表。

在 Python 3.9 及以上版本，也可以写成：

```
matrix: list[list[int]]
```

类型标注主要用于提高代码的可读性，并帮助编辑器和静态检查工具发现类型错误。它不会改变程序的运行逻辑。

### 整除运算符 `//`

```
row = mid // cols
```

`//` 返回整数商。例如：

```
7 // 3 == 2
```

在本题中，它用来计算一维下标对应的行号。

### 取模运算符 `%`

```
col = mid % cols
```

`%` 返回除法的余数。例如：

```
7 % 3 == 1
```

在本题中，它用来计算元素在当前行中的列号。

### `matrix[row][-1]`

Python 支持负数下标：

```
matrix[row][-1]
```

表示当前行的最后一个元素，等价于：

```
matrix[row][len(matrix[row]) - 1]
```

------

## 常见错误

### 1. 混淆行列转换公式

一维下标转换为二维坐标时，应该使用矩阵的列数 `cols`：

```
row = mid // cols
col = mid % cols
```

不能写成：

```
row = mid // rows
col = mid % rows
```

因为矩阵按行存储，每经过 `cols` 个元素才会进入下一行。

------

### 2. 二分搜索的边界错误

当搜索区间是闭区间 `[left, right]` 时：

```
while left <= right:
```

更新方式应该是：

```
left = mid + 1
right = mid - 1
```

如果只写成 `left = mid` 或 `right = mid`，可能导致搜索区间无法缩小，从而出现死循环。

------

### 3. 没有处理空矩阵

直接执行：

```
len(matrix[0])
```

需要保证 `matrix` 至少包含一行。因此应先检查：

```
if not matrix or not matrix[0]:
    return False
```

这里利用了 Python 的短路求值：

- 如果 `not matrix` 为 `True`，Python 不会继续计算 `not matrix[0]`。
- 因此不会在空矩阵上访问 `matrix[0]`。

------

### 4. 真的创建一个展开后的数组

虽然一次二分搜索将矩阵“视为”一维数组，但没有必要真的执行：

```
flattened = [value for row in matrix for value in row]
```

这样会产生：

- `O(m × n)` 的额外时间开销；
- `O(m × n)` 的额外空间开销。

通过 `mid // cols` 和 `mid % cols` 可以直接访问原矩阵，保持 `O(1)` 空间复杂度。

------

## 方法对比

| 方法         | 时间复杂度         | 空间复杂度 | 是否满足题目要求 |
| ------------ | ------------------ | ---------- | ---------------- |
| 暴力搜索     | `O(m × n)`         | `O(1)`     | 否               |
| 阶梯搜索     | `O(m + n)`         | `O(1)`     | 否               |
| 两次二分搜索 | `O(log m + log n)` | `O(1)`     | 是               |
| 一次二分搜索 | `O(log(m × n))`    | `O(1)`     | 是               |

## 总结

这道题最重要的观察是：矩阵按行展开后仍然是一个完整的有序数组。

因此，最直接且最优雅的解法是一次二分搜索：

```
row = mid // cols
col = mid % cols
```

这两个公式让我们可以在不真正展开矩阵的情况下，将普通的一维二分搜索应用到二维矩阵中。