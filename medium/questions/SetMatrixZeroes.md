# Set Matrix Zeroes（矩阵置零）

给定一个 `m x n` 的整数矩阵 `matrix`，如果某个元素为 `0`，则将该元素所在的**整行和整列**全部设置为 `0`。

要求：必须对原矩阵进行 **原地修改（in-place）**。

------

## 一、问题的关键点

这道题真正需要注意的是：

> **不能在第一次扫描矩阵时，看到 `0` 就立即把对应行列改成 `0`。**

因为这样会产生“连锁反应”。

例如：

```text
1 1 1
1 0 1
1 1 1
```

如果扫描到中间的 `0` 后，立即把第二行和第二列全部改成 `0`：

```text
1 0 1
0 0 0
1 0 1
```

之后继续扫描时，会遇到很多**新生成的 `0`**。如果把这些新生成的 `0` 也当成原始的 `0`，最终整个矩阵都可能被错误地置零。

因此，这道题的核心思想是：

> **先记录哪些行和列需要变成 `0`，再统一执行修改。**

不同解法之间的主要区别，就是“如何保存这些标记”。

------

# 1. 暴力解法：复制矩阵

## 思路

最直接的方法是创建一个原矩阵的副本。

遍历原矩阵时：

- 只根据**原矩阵**判断哪些位置原本就是 `0`
- 所有修改都写入副本
- 最后再把副本复制回原矩阵

这样，新产生的 `0` 不会影响后续判断。

------

## 算法步骤

1. 记录矩阵的行数 `ROWS` 和列数 `COLS`。
2. 创建矩阵副本 `mark`。
3. 遍历原矩阵中的每个位置 `(r, c)`。
4. 如果：

```python
matrix[r][c] == 0
```

则：

- 将 `mark` 的第 `r` 行全部设置为 `0`
- 将 `mark` 的第 `c` 列全部设置为 `0`

1. 遍历结束后，将 `mark` 的所有值复制回 `matrix`。

------

## Python 实现

```python
from typing import List


class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        ROWS, COLS = len(matrix), len(matrix[0])

        # 创建原矩阵的深拷贝
        # 每一行都需要单独复制
        mark = [row[:] for row in matrix]

        for r in range(ROWS):
            for c in range(COLS):
                # 只根据原矩阵中的 0 做判断
                if matrix[r][c] == 0:

                    # 将对应整行置 0
                    for j in range(COLS):
                        mark[r][j] = 0

                    # 将对应整列置 0
                    for i in range(ROWS):
                        mark[i][c] = 0

        # 将结果复制回原矩阵，实现原地修改
        for r in range(ROWS):
            for c in range(COLS):
                matrix[r][c] = mark[r][c]
```

------

## 关于 `row[:]`

这里：

```python
row[:]
```

表示对一个 Python 列表进行**浅拷贝**。

例如：

```python
row = [1, 2, 3]
copy_row = row[:]
```

此时 `copy_row` 是一个新的列表。

对于本题的二维整数矩阵来说：

```python
mark = [row[:] for row in matrix]
```

相当于创建了一份足够使用的二维矩阵副本。

不要写成：

```python
mark = matrix
```

因为这样只是让两个变量引用同一个矩阵，并没有真正复制。

------

## 复杂度分析

假设矩阵有：

- `m` 行
- `n` 列

最坏情况下，每个位置都是 `0`。

每发现一个 `0`，需要修改：

```text
O(m + n)
```

最多有：

```text
m × n
```

个位置，因此最坏时间复杂度为：

```text
O(mn(m + n))
```

空间复杂度为：

```text
O(mn)
```

因为复制了整个矩阵。

> 这个方法虽然直观，但无论时间还是空间都不是理想解法。

------

# 2. 使用两个标记数组

## 思路

实际上，我们不需要复制整个矩阵。

只需要知道：

- 哪些行需要被置零
- 哪些列需要被置零

因此，可以分别创建两个布尔数组：

```python
rows
cols
```

其中：

```python
rows[r]
```

表示第 `r` 行是否需要变成 `0`。

```python
cols[c]
```

表示第 `c` 列是否需要变成 `0`。

整个过程分为两个阶段：

```text
第一遍：记录
第二遍：修改
```

这也是这道题非常重要的一种通用算法思想：

> **Read first, write later.**
>
> 先收集信息，再执行修改。

这样就不会受到修改过程中产生的新 `0` 的干扰。

------

## 算法步骤

### 第一步：记录需要置零的行和列

创建：

```python
rows = [False] * ROWS
cols = [False] * COLS
```

遍历矩阵，如果：

```python
matrix[r][c] == 0
```

则：

```python
rows[r] = True
cols[c] = True
```

------

### 第二步：统一修改矩阵

再次遍历矩阵。

对于位置 `(r, c)`：

如果：

```python
rows[r] or cols[c]
```

说明：

- 当前行需要置零

或者：

- 当前列需要置零

因此：

```python
matrix[r][c] = 0
```

------

## Python 实现

```python
from typing import List


class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        ROWS, COLS = len(matrix), len(matrix[0])

        # rows[r] == True：
        # 第 r 行需要被全部置为 0
        rows = [False] * ROWS

        # cols[c] == True：
        # 第 c 列需要被全部置为 0
        cols = [False] * COLS

        # 第一遍：只记录原矩阵中出现 0 的行和列
        for r in range(ROWS):
            for c in range(COLS):
                if matrix[r][c] == 0:
                    rows[r] = True
                    cols[c] = True

        # 第二遍：根据之前记录的信息统一修改
        for r in range(ROWS):
            for c in range(COLS):
                if rows[r] or cols[c]:
                    matrix[r][c] = 0
```

------

## 为什么这里使用 `or`？

条件是：

```python
if rows[r] or cols[c]:
```

因为只要满足下面任意一个条件：

- 第 `r` 行原本存在 `0`
- 第 `c` 列原本存在 `0`

当前位置就应该被置为 `0`。

因此使用逻辑或 `or`。

------

## Python：`[False] * n`

例如：

```python
rows = [False] * 5
```

得到：

```python
[False, False, False, False, False]
```

这是 Python 中非常常见的初始化固定长度数组的方法。

由于 `False` 是不可变对象，因此这里使用：

```python
[False] * n
```

完全没有问题。

但如果初始化嵌套列表：

```python
matrix = [[0] * n] * m
```

就需要特别小心，因为所有行可能会引用同一个列表。

------

## 复杂度分析

第一次遍历：

```text
O(mn)
```

第二次遍历：

```text
O(mn)
```

因此总时间复杂度：

```text
O(mn)
```

额外空间：

```text
rows: O(m)
cols: O(n)
```

所以：

```text
O(m + n)
```

### 总结

```text
时间复杂度：O(mn)
空间复杂度：O(m + n)
```

相比暴力解法，这已经非常高效。

------

# 3. 空间优化：使用矩阵第一行和第一列作为标记数组

## 核心思想

上一种方法使用了：

```python
rows
cols
```

两个额外数组。

但仔细观察可以发现：

我们需要存储的只是：

```text
这一行是否需要置零
这一列是否需要置零
```

实际上完全可以利用矩阵本身来保存这些信息。

具体来说：

- 使用**第一列**记录哪些行需要置零
- 使用**第一行**记录哪些列需要置零

例如：

```python
matrix[r][0] = 0
```

表示：

> 第 `r` 行最终应该全部变成 `0`

而：

```python
matrix[0][c] = 0
```

表示：

> 第 `c` 列最终应该全部变成 `0`

这样就不需要额外的 `rows` 和 `cols` 数组了。

------

# 关键问题：`matrix[0][0]`

第一行和第一列有一个共同的位置：

```python
matrix[0][0]
```

问题在于：

```text
它到底表示第一行需要置零，
还是第一列需要置零？
```

一个元素无法同时独立保存两个布尔状态。

因此，我们需要额外保存其中一个状态。

本题代码选择：

```python
rowZero
```

专门记录：

> **第一行原本是否存在 `0`。**

与此同时：

```python
matrix[0][0]
```

负责记录：

> **第一列是否需要变成 `0`。**

这样两个状态就被区分开了。

------

# 算法步骤

## 第一步：扫描矩阵并建立标记

初始化：

```python
rowZero = False
```

遍历所有元素。

如果发现：

```python
matrix[r][c] == 0
```

首先：

```python
matrix[0][c] = 0
```

表示：

> 第 `c` 列需要置零。

然后判断当前是不是第一行。

如果：

```python
r > 0
```

则：

```python
matrix[r][0] = 0
```

表示：

> 第 `r` 行需要置零。

如果：

```python
r == 0
```

则不能通过 `matrix[0][0]` 保存第一行状态，因此：

```python
rowZero = True
```

------

## 第二步：处理内部矩阵

注意，此时不能立刻处理第一行和第一列。

因为它们现在承担着“标记数组”的职责。

所以只处理：

```text
第 1 ~ m-1 行
第 1 ~ n-1 列
```

即：

```python
for r in range(1, ROWS):
    for c in range(1, COLS):
```

如果：

```python
matrix[r][0] == 0
```

说明当前行要置零。

或者：

```python
matrix[0][c] == 0
```

说明当前列要置零。

因此：

```python
matrix[r][c] = 0
```

------

## 第三步：处理第一列

如果：

```python
matrix[0][0] == 0
```

说明第一列需要全部置零。

于是：

```python
for r in range(ROWS):
    matrix[r][0] = 0
```

------

## 第四步：处理第一行

如果：

```python
rowZero
```

为 `True`，说明第一行原本存在 `0`。

于是：

```python
for c in range(COLS):
    matrix[0][c] = 0
```

------

# Python 实现

```python
from typing import List


class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        ROWS, COLS = len(matrix), len(matrix[0])

        # 单独记录第一行是否原本包含 0
        rowZero = False

        # 第一遍：
        # 使用第一行和第一列作为 marker
        for r in range(ROWS):
            for c in range(COLS):
                if matrix[r][c] == 0:

                    # 第一行记录：第 c 列需要置零
                    matrix[0][c] = 0

                    if r > 0:
                        # 第一列记录：第 r 行需要置零
                        matrix[r][0] = 0
                    else:
                        # 当前 0 在第一行中
                        rowZero = True

        # 第二遍：
        # 根据第一行和第一列的 marker
        # 更新内部区域
        #
        # 注意：不能从 0 开始，
        # 因为第一行和第一列现在还保存着 marker 信息
        for r in range(1, ROWS):
            for c in range(1, COLS):
                if matrix[r][0] == 0 or matrix[0][c] == 0:
                    matrix[r][c] = 0

        # matrix[0][0] 用来表示：
        # 第一列是否需要置零
        if matrix[0][0] == 0:
            for r in range(ROWS):
                matrix[r][0] = 0

        # rowZero 单独表示：
        # 第一行是否需要置零
        if rowZero:
            for c in range(COLS):
                matrix[0][c] = 0
```

------

# 示例分析

假设：

```text
matrix =

1  1  1
1  0  1
1  1  1
```

第一次扫描到：

```text
matrix[1][1] = 0
```

于是设置：

```python
matrix[0][1] = 0
matrix[1][0] = 0
```

矩阵暂时变为：

```text
1  0  1
0  0  1
1  1  1
```

这里第一行和第一列已经变成了我们的“标记数组”。

可以理解为：

```text
第一行：
   ↓
1  0  1
   ↑
第 1 列需要置零
```

以及：

```text
第一列：

1
0 ← 第 1 行需要置零
1
```

之后只处理内部区域。

最终得到：

```text
1  0  1
0  0  0
1  0  1
```

------

# 为什么一定要最后处理第一行和第一列？

这是空间优化解法最容易出错的地方之一。

第一行和第一列在算法运行过程中并不是普通数据，而是：

```text
marker / 标记数组
```

例如：

```python
matrix[0][2] == 0
```

可能并不是因为这个位置本身最终需要被处理，而是它在告诉我们：

> 第 `2` 列需要全部置零。

如果过早执行：

```python
matrix[0][c] = 0
```

就可能把本来不是 marker 的位置也改成 `0`，导致信息污染。

因此正确顺序必须是：

```text
1. 建立 marker
2. 根据 marker 修改内部矩阵
3. 修改第一列
4. 修改第一行
```

可以记成：

> **标记区域最后修改。**

------

# 复杂度分析

第一遍扫描：

```text
O(mn)
```

第二遍扫描内部区域：

```text
O(mn)
```

最后处理第一行和第一列：

```text
O(m + n)
```

总体仍然是：

```text
O(mn)
```

额外只使用了：

```python
rowZero
```

一个布尔变量。

因此：

```text
空间复杂度：O(1)
```

最终：

```text
时间复杂度：O(mn)
空间复杂度：O(1)
```

这是这道题的最优解法。

------

# 常见错误

## 1. 扫描过程中立即把整行整列置零

错误思路：

```python
if matrix[r][c] == 0:
    # 立即修改整行和整列
```

问题在于：

> 新产生的 `0` 会被误认为原始的 `0`。

从而产生错误的连锁修改。

正确的方法是：

```text
先标记
再修改
```

------

## 2. 忘记单独记录第一行

空间优化解法中：

```python
matrix[0][0]
```

位于：

```text
第一行 ∩ 第一列
```

所以不能同时独立表示：

```text
第一行是否需要置零
第一列是否需要置零
```

因此必须额外使用：

```python
rowZero
```

保存其中一个状态。

------

## 3. 过早修改第一行或第一列

第一行和第一列承担着 marker 的作用。

如果在内部矩阵处理完成之前修改它们，就可能丢失原来的标记信息。

因此必须：

```text
内部元素 → 第一列 → 第一行
```

------

## 4. 第二遍仍然从 `(0, 0)` 开始遍历

错误：

```python
for r in range(ROWS):
    for c in range(COLS):
```

在空间优化方案的第二阶段，这样做会修改 marker 本身。

应该跳过第一行和第一列：

```python
for r in range(1, ROWS):
    for c in range(1, COLS):
```

------

# 三种解法对比

| 方法                  | 时间复杂度   | 空间复杂度 | 面试推荐程度 |
| --------------------- | ------------ | ---------- | ------------ |
| 复制整个矩阵          | `O(mn(m+n))` | `O(mn)`    | 较低         |
| 两个标记数组          | `O(mn)`      | `O(m+n)`   | 高           |
| 第一行/第一列作为标记 | `O(mn)`      | `O(1)`     | **最高**     |

------

# 面试中的思路演进

这道题很适合按照下面的顺序向面试官展示思考过程。

首先指出不能直接修改：

```text
直接修改会导致新产生的 0 污染后续判断。
```

然后提出最自然的办法：

```text
我可以先记录哪些行和列需要被置零。
```

于是得到：

```text
rows + cols
```

的 `O(m+n)` 空间解法。

接着继续优化：

```text
rows 和 cols 本质上只是两个 marker 数组。
矩阵本身已经有 m 行和 n 列，
因此可以使用第一列表示 rows，
第一行表示 cols。
```

最后发现：

```text
matrix[0][0]
```

发生状态冲突，所以使用额外的一个布尔变量解决。

整个优化过程可以概括为：

```text
复制所有数据
    ↓
只保存必要状态
    ↓
复用输入空间保存状态
```

这是非常常见的空间优化思路。

------

# 一个值得记住的算法模式：原地标记（In-place Marking）

这道题最值得学习的并不仅仅是代码本身，而是这种技巧：

> **当额外数组只是用来保存某种简单状态时，可以考虑复用输入数组的一部分来保存状态。**

类似思路经常出现在数组和矩阵题中：

```text
额外 hash/set/boolean array
        ↓
寻找输入数据中可以充当 marker 的位置
        ↓
将 O(n) 空间优化为 O(1)
```

但使用这种方法时通常需要特别考虑：

```text
被拿来作为 marker 的位置，
它自己原来的状态应该如何保存？
```

本题中的：

```python
rowZero
```

就是专门解决这个问题的。

因此，看到类似题目时可以形成一个条件反射：

> **能不能把输入数组本身当作额外存储空间？**
