# Search a 2D Matrix II（搜索二维矩阵 II）

给定一个 `m × n` 的整数矩阵 `matrix` 和一个目标值 `target`，请编写一个高效算法判断 `target` 是否存在于矩阵中。

矩阵满足：

- 每一行从左到右**升序排列**
- 每一列从上到下**升序排列**

例如：

```text
matrix = [
    [1,  2,  4,  8],
    [10, 11, 12, 13],
    [14, 20, 30, 40]
]

target = 10
```

输出：

```text
True
```

------

# 一、核心观察

这道题最重要的地方在于：

> 应该从矩阵的**右上角**或**左下角**开始搜索。

例如从右上角开始：

```text
1   2   4  [8]
10  11  12  13
14  20  30  40
```

假设当前位置是：

```python
matrix[row][col]
```

由于矩阵的行和列都是递增的，我们可以根据当前位置与 `target` 的大小关系，直接排除**一整行或一整列**。

### 情况 1：当前位置等于 `target`

```text
matrix[row][col] == target
```

直接找到答案：

```python
return True
```

### 情况 2：当前位置大于 `target`

```text
matrix[row][col] > target
```

因为当前元素位于这一列相对靠上的位置，而下面的元素只会更大。

因此当前这一列不可能再找到更小的 `target`。

所以：

```python
col -= 1
```

向左移动一列。

### 情况 3：当前位置小于 `target`

```text
matrix[row][col] < target
```

由于当前元素位于这一行的最右侧，而它左边的所有元素都更小。

所以当前这一行不可能包含更大的 `target`。

因此：

```python
row += 1
```

向下移动一行。

------

# 方法一：从右上角开始搜索

## 思路

假设矩阵是：

```text
 1   2   4   8
10  11  12  13
14  20  30  40
```

从右上角 `8` 开始。

搜索：

```text
target = 10
```

### 第一步

当前位置：

```text
8
```

因为：

```text
8 < 10
```

而这一行中 `8` 左边的元素：

```text
1, 2, 4
```

全部比 `8` 更小，因此它们更不可能等于 `10`。

所以可以直接排除第一行：

```text
 1   2   4   8      ← 排除
10  11  12  13
14  20  30  40
```

向下：

```text
row += 1
```

------

### 第二步

来到：

```text
13
```

因为：

```text
13 > 10
```

并且 `13` 下方的元素：

```text
40
```

一定更大。

所以这一列不可能包含 `10`。

排除这一列：

```text
 1   2   4  | 8
10  11  12  |13
14  20  30  |40
            ↑
          排除
```

向左：

```text
col -= 1
```

------

### 第三步

来到：

```text
12
```

因为：

```text
12 > 10
```

继续向左。

------

### 第四步

来到：

```text
11
```

因为：

```text
11 > 10
```

继续向左。

------

### 第五步

来到：

```text
10
```

找到：

```python
return True
```

整个搜索路径实际上只有：

```text
8
↓
13
← 12
← 11
← 10
```

并不需要遍历整个矩阵。

------

# 算法步骤

设：

```python
rows = len(matrix)
cols = len(matrix[0])
```

从右上角开始：

```python
row = 0
col = cols - 1
```

然后不断判断：

```text
如果 current == target
    找到

如果 current > target
    向左

如果 current < target
    向下
```

直到：

```python
row >= rows
```

或者：

```python
col < 0
```

说明已经走出矩阵，目标值不存在。

------

# Python 代码

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        rows = len(matrix)
        cols = len(matrix[0])

        # 从右上角开始
        row = 0
        col = cols - 1

        # 只要当前位置仍然在矩阵范围内，就继续搜索
        while row < rows and col >= 0:
            current = matrix[row][col]

            if current == target:
                return True

            elif current > target:
                # 当前值太大：
                # 当前列下面的值只会更大，
                # 因此排除当前列，向左移动
                col -= 1

            else:
                # 当前值太小：
                # 当前行左边的值只会更小，
                # 因此排除当前行，向下移动
                row += 1

        # 走出矩阵仍未找到
        return False
```

------

# 为什么一定要从右上角开始？

这其实是这道题最核心的算法设计点。

右上角有一个非常特殊的性质：

```text
        ← 更小
1   2   4   8
            ↓
           更大
```

对于右上角元素：

- 左边的元素更小
- 下边的元素更大

因此当前位置与 `target` 比较之后，我们一定能确定一个搜索方向。

如果：

```text
current > target
```

我们需要更小的数：

```text
←
```

如果：

```text
current < target
```

我们需要更大的数：

```text
↓
```

所以每次比较都至少能排除：

- 一整行，或者
- 一整列

这就是该算法高效的原因。

------

# 为什么不能从左上角开始？

假设从：

```text
[1]  2   4   8
10  11  12  13
14  20  30  40
```

开始。

左上角有：

```text
        → 更大
        ↓ 更大
```

假如：

```text
1 < target
```

我们知道需要找更大的数。

但问题在于：

```text
应该向右？
还是向下？
```

两个方向都可能存在答案。

例如搜索：

```text
target = 4
```

应该向右。

但搜索：

```text
target = 10
```

应该向下。

所以仅仅比较：

```text
1 < target
```

无法排除任何一个方向。

这就是为什么左上角不是一个好的起点。

------

# 为什么左下角也可以？

左下角同样具有“一边越来越大、一边越来越小”的性质。

例如：

```text
            ↑ 更小
1   2   4   8
10  11  12  13
[14]20  30  40
 →
更大
```

从左下角开始：

- 向右：数字变大
- 向上：数字变小

因此：

```text
current < target
```

说明需要更大的值：

```text
→ 向右
```

而：

```text
current > target
```

说明需要更小的值：

```text
↑ 向上
```

所以左下角同样可以作为起点。

------

# 左下角版本

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        rows = len(matrix)
        cols = len(matrix[0])

        # 从左下角开始
        row = rows - 1
        col = 0

        while row >= 0 and col < cols:
            current = matrix[row][col]

            if current == target:
                return True

            elif current > target:
                # 当前值太大，需要寻找更小的值
                # 向上移动
                row -= 1

            else:
                # 当前值太小，需要寻找更大的值
                # 向右移动
                col += 1

        return False
```

这两个版本本质完全相同。

面试时选择自己最容易记住的一种即可。

------

# 复杂度分析

设矩阵大小为：

```text
m × n
```

从右上角开始：

```text
row = 0
col = n - 1
```

整个过程中：

- `row` 只会增加，最多增加 `m` 次
- `col` 只会减少，最多减少 `n` 次

因此最多走：

[
m+n
]

步左右。

所以时间复杂度：

[
O(m+n)
]

额外空间复杂度：

[
O(1)
]

因为只使用了：

```text
row
col
current
rows
cols
```

等常数数量的变量。

------

# 方法二：每一行分别进行二分搜索

另一种比较容易想到的方法是：

> 每一行都是有序数组，所以可以对每一行做 Binary Search（二分搜索）。

例如：

```text
1   2   4   8       Binary Search
10  11  12  13      Binary Search
14  20  30  40      Binary Search
```

如果某一行找到了：

```python
return True
```

否则最终：

```python
return False
```

------

# Python 代码：逐行二分搜索

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        for row in matrix:
            left = 0
            right = len(row) - 1

            while left <= right:
                mid = (left + right) // 2

                if row[mid] == target:
                    return True

                elif row[mid] < target:
                    # target 更大，搜索右半部分
                    left = mid + 1

                else:
                    # target 更小，搜索左半部分
                    right = mid - 1

        return False
```

------

# 二分搜索回顾

对于一个有序数组：

```text
1  3  5  7  9
```

搜索：

```text
target = 7
```

维护：

```python
left
right
```

计算中点：

```python
mid = (left + right) // 2
```

然后比较：

```python
nums[mid]
```

如果：

```python
nums[mid] < target
```

说明左边，包括 `mid`，都不可能是答案：

```python
left = mid + 1
```

如果：

```python
nums[mid] > target
```

说明右边，包括 `mid`，都不可能是答案：

```python
right = mid - 1
```

这就是标准 Binary Search。

------

# 逐行二分搜索的复杂度

一共有：

```text
m
```

行。

每一行：

```text
n
```

个元素。

对长度为 `n` 的数组进行二分搜索需要：

[
O(\log n)
]

所以总时间复杂度：

[
O(m\log n)
]

空间复杂度：

[
O(1)
]

如果矩阵接近正方形，例如：

```text
n × n
```

那么复杂度为：

[
O(n\log n)
]

而右上角搜索为：

[
O(n)
]

因此这道题通常更推荐右上角 / 左下角搜索。

------

# 方法三：暴力搜索

最直接的方法是检查矩阵中的每个元素：

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        for row in matrix:
            for value in row:
                if value == target:
                    return True

        return False
```

时间复杂度：

[
O(mn)
]

空间复杂度：

[
O(1)
]

这个方法虽然正确，但完全没有利用矩阵已经排序这一条件，因此一般不是面试官期待的答案。

------

# 三种方法比较

| 方法                | 时间复杂度   | 空间复杂度 | 是否利用行有序 | 是否利用列有序 | 推荐程度 |
| ------------------- | ------------ | ---------- | -------------- | -------------- | -------- |
| 暴力遍历            | `O(mn)`      | `O(1)`     | ❌              | ❌              | ⭐        |
| 逐行二分            | `O(m log n)` | `O(1)`     | ✅              | 基本没有       | ⭐⭐⭐      |
| 右上角 / 左下角搜索 | `O(m + n)`   | `O(1)`     | ✅              | ✅              | ⭐⭐⭐⭐⭐    |

对于这道题，最推荐：

```text
右上角搜索
```

因为它同时利用了：

```text
行递增
+
列递增
```

两个条件。

------

# 常见错误

## 1. 误以为整个矩阵可以直接看成一个排序数组

这里需要特别注意。

题目只保证：

```text
每行递增
每列递增
```

并**没有保证**：

```text
上一行最后一个元素
<
下一行第一个元素
```

例如下面的矩阵完全符合题目要求：

```text
1   4   7
2   5   8
3   6   9
```

每行递增：

```text
1 < 4 < 7
2 < 5 < 8
3 < 6 < 9
```

每列也递增：

```text
1 < 2 < 3
4 < 5 < 6
7 < 8 < 9
```

但是如果按一维展开：

```text
1, 4, 7, 2, 5, 8, 3, 6, 9
```

显然并不是有序数组。

所以不能直接对整个 `m × n` 矩阵做普通的一维二分搜索。

这和另一道常见题 **Search a 2D Matrix** 很容易混淆。

------

# 与 Search a 2D Matrix 的区别

代码面试中，这两道题非常容易被混在一起。

## Search a 2D Matrix

通常条件是：

```text
每一行内部递增
并且下一行第一个元素 > 上一行最后一个元素
```

例如：

```text
1   3   5   7
10  11  16  20
23  30  34  60
```

整个矩阵从一维角度来看其实是：

```text
1 3 5 7 10 11 16 20 23 30 34 60
```

完全有序。

所以可以：

```text
把二维坐标映射成一维
+
Binary Search
```

时间复杂度：

[
O(\log(mn))
]

------

## Search a 2D Matrix II

本题只保证：

```text
每行递增
每列递增
```

例如：

```text
1   4   7
2   5   8
3   6   9
```

不能直接展开成有序数组。

最经典的方法是：

```text
从右上角开始
```

时间复杂度：

[
O(m+n)
]

这个区别非常值得记住。

------

# Python 边界条件说明

如果题目没有保证矩阵一定非空，更稳健的写法是：

```python
if not matrix or not matrix[0]:
    return False
```

完整版本：

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        # 防止 matrix = [] 或 matrix = [[]]
        if not matrix or not matrix[0]:
            return False

        rows = len(matrix)
        cols = len(matrix[0])

        # 从右上角开始
        row = 0
        col = cols - 1

        while row < rows and col >= 0:
            current = matrix[row][col]

            if current == target:
                return True

            if current > target:
                # 太大：向左寻找更小的数
                col -= 1
            else:
                # 太小：向下寻找更大的数
                row += 1

        return False
```

这里：

```python
if not matrix
```

检查的是：

```text
[]
```

而：

```python
if not matrix[0]
```

检查的是：

```text
[[]]
```

这样后面访问：

```python
matrix[0]
```

或者：

```python
matrix[row][col]
```

时就不会出现越界问题。

------

# 为什么这个算法可以称为“消除法”

这个算法其实和二分搜索有一个共同思想：

> 每一次比较之后，都要尽可能排除不可能包含答案的搜索空间。

二分搜索每次：

```text
排除一半
```

而这道题每次：

```text
排除一整行
或
排除一整列
```

例如当前在右上角：

```text
? ? ? X
? ? ? ?
? ? ? ?
```

如果：

```text
X < target
```

那么 `X` 左边整行全部：

```text
< X < target
```

所以整行都可以扔掉：

```text
x x x x
? ? ? ?
? ? ? ?
```

如果：

```text
X > target
```

那么 `X` 下方整列全部：

```text
> X > target
```

所以整列都可以扔掉：

```text
? ? ? x
? ? ? x
? ? ? x
```

不断缩小这个候选区域，最终要么找到 `target`，要么候选区域为空。

------

# 面试中的推荐解释方式

如果面试官问：

> Why do you start from the top-right corner?

可以把思路组织成：

```text
右上角是一个特殊的位置：

它左边的数字都比它小，
它下面的数字都比它大。

因此：

如果 current > target，
那么当前这一列都可以排除，向左移动。

如果 current < target，
那么当前这一行都可以排除，向下移动。

每一步都会排除一行或者一列，
所以最多移动 m + n 次。
```

对应英文可以简洁表达为：

```text
I start from the top-right corner because it gives us two monotonic
directions.

Everything to the left is smaller, and everything below is larger.

So if the current value is greater than the target, I can eliminate
the current column and move left.

If it is smaller than the target, I can eliminate the current row
and move down.

Since we move left at most n times and down at most m times, the
time complexity is O(m + n).
```

------

# 最终推荐代码

面试时建议优先写这个版本：

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        if not matrix or not matrix[0]:
            return False

        rows = len(matrix)
        cols = len(matrix[0])

        # 从右上角开始搜索
        row = 0
        col = cols - 1

        while row < rows and col >= 0:
            if matrix[row][col] == target:
                return True

            if matrix[row][col] > target:
                # 当前值太大，排除当前列
                col -= 1
            else:
                # 当前值太小，排除当前行
                row += 1

        return False
```

记忆方式可以浓缩成：

```text
Start: 右上角

太大 -> 左移
太小 -> 下移
相等 -> 找到
```

复杂度：

```text
Time:  O(m + n)
Space: O(1)
```

这道题真正需要掌握的不是代码本身，而是识别出：

```text
右上角 / 左下角
```

属于一种非常有用的 **“单调性搜索起点”**。

当二维结构中两个方向分别呈现：

```text
一个方向越来越大
另一个方向越来越小
```

时，就应该考虑能否通过比较一次就排除一整行或一整列。
