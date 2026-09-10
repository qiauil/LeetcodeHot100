# Rotate Image（旋转图像）

给定一个 `n × n` 的整数矩阵 `matrix`，将其**顺时针旋转 90°**。

要求必须进行**原地修改（in-place）**，不能额外创建另一个二维矩阵来完成最终解法。

例如：

```text
原矩阵：

1 2 3
4 5 6
7 8 9

顺时针旋转 90° 后：

7 4 1
8 5 2
9 6 3
```

------

## 一、核心坐标变化

理解这道题最重要的是掌握元素旋转后的坐标变化。

原矩阵中的元素：

```text
matrix[i][j]
```

顺时针旋转 90° 后，会移动到：

```text
matrix[j][n - 1 - i]
```

也就是：

$(i,j)\rightarrow(j,n-1-i)$

例如，在 `3 × 3` 矩阵中：

```text
(0, 0) -> (0, 2)
(0, 1) -> (1, 2)
(0, 2) -> (2, 2)

(1, 0) -> (0, 1)
(1, 1) -> (1, 1)
(1, 2) -> (2, 1)
```

这个坐标关系是暴力解法的基础，也可以帮助理解后面的原地算法。

------

# 方法一：额外矩阵模拟旋转

> 这个方法最直观，但**不满足题目要求的 O(1) 额外空间**。
> 非常适合作为理解旋转坐标关系的第一步。

## 思路

创建一个新的 `n × n` 矩阵 `rotated`。

遍历原矩阵中的每个元素：

```text
matrix[i][j]
```

按照旋转规则放入：

```text
rotated[j][n - 1 - i]
```

最后再将 `rotated` 复制回原来的 `matrix`。

例如：

```text
1 2 3
4 5 6
7 8 9
```

元素 `1`：

```text
原坐标：(0, 0)
新坐标：(0, 2)
```

元素 `2`：

```text
原坐标：(0, 1)
新坐标：(1, 2)
```

元素 `7`：

```text
原坐标：(2, 0)
新坐标：(0, 0)
```

最终得到：

```text
7 4 1
8 5 2
9 6 3
```

## 算法步骤

1. 令 `n = len(matrix)`。
2. 创建一个新的 `n × n` 矩阵 `rotated`。
3. 遍历所有 `(i, j)`。
4. 执行：

```python
rotated[j][n - 1 - i] = matrix[i][j]
```

1. 将 `rotated` 中的元素复制回 `matrix`。

## Python 代码

```python
from typing import List


class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        n = len(matrix)

        # 创建额外的 n × n 矩阵
        rotated = [[0] * n for _ in range(n)]

        # 根据旋转后的坐标关系填充新矩阵
        for i in range(n):
            for j in range(n):
                rotated[j][n - 1 - i] = matrix[i][j]

        # 将结果复制回原矩阵
        for i in range(n):
            for j in range(n):
                matrix[i][j] = rotated[i][j]
```

## Python 语法说明：二维列表初始化

这里使用：

```python
rotated = [[0] * n for _ in range(n)]
```

生成一个 `n × n` 的二维列表。

例如 `n = 3`：

```python
$    [0, 0, 0],
    [0, 0, 0],
    [0, 0, 0]$
```

这里有一个 Python 面试中值得注意的细节。

**不要写成：**

```python
rotated = [[0] * n] * n
```

因为这样得到的多行实际上引用的是**同一个列表对象**。

例如：

```python
matrix = [[0] * 3] * 3
matrix[0][0] = 1
```

结果可能是：

```text
$    [1, 0, 0],
    [1, 0, 0],
    [1, 0, 0]$
```

因此，创建二维数组通常更安全的方式是：

```python
[[0] * n for _ in range(n)]
```

## 复杂度

时间复杂度：

$O(n^2)$

因为需要遍历整个矩阵。

空间复杂度：

$O(n^2)$

因为创建了一个额外的 `n × n` 矩阵。

------

# 方法二：四个元素一组原地旋转

## 思路

这是一个真正满足题目 **in-place** 要求的方法。

可以把矩阵想象成一层一层的正方形：

```text
a a a a a
a b b b a
a b c b a
a b b b a
a a a a a
```

首先旋转最外层：

```text
a
```

然后旋转：

```text
b
```

不断向中心缩小。

对于每一层，我们每次处理四个对应位置：

```text
左上 -> 右上
右上 -> 右下
右下 -> 左下
左下 -> 左上
```

但是在实际赋值时，如果直接：

```python
top_right = top_left
```

就会覆盖原来的 `top_right`。

因此需要先暂存一个元素。

------

## 四个位置的移动关系

假设当前层：

```text
top = l
bottom = r
```

对于偏移量 `i`：

```text
          top-left
              ↓
      +---------------+
      | x           x |
      |               |
      |               |
      | x           x |
      +---------------+
```

顺时针旋转后：

```text
top-left     -> top-right
top-right    -> bottom-right
bottom-right -> bottom-left
bottom-left  -> top-left
```

赋值时我们反过来看“当前位置的值来自哪里”：

```text
top-left     <- bottom-left
bottom-left  <- bottom-right
bottom-right <- top-right
top-right    <- 原 top-left
```

这样只需要保存原来的 `top-left`。

------

## 算法步骤

维护两个边界：

```python
l = 0
r = n - 1
```

它们表示当前需要处理的正方形层。

对于每一层：

```python
while l < r:
```

遍历当前层顶部的每一个位置。

需要注意：

```python
range(r - l)
```

而不是：

```python
range(r - l + 1)
```

因为每次旋转的是**四个元素组成的一组**。如果把最后一个位置也算进去，会重复旋转角上的元素。

完成一层后：

```python
l += 1
r -= 1
```

继续处理内层。

------

## Python 代码

```python
from typing import List


class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        # 当前层的左右边界
        l, r = 0, len(matrix) - 1

        # 从最外层逐渐向内处理
        while l < r:
            # 当前这一层需要旋转 r - l 组元素
            for i in range(r - l):
                top = l
                bottom = r

                # 1. 暂存左上角
                top_left = matrix[top][l + i]

                # 2. 左下 -> 左上
                matrix[top][l + i] = matrix[bottom - i][l]

                # 3. 右下 -> 左下
                matrix[bottom - i][l] = matrix[bottom][r - i]

                # 4. 右上 -> 右下
                matrix[bottom][r - i] = matrix[top + i][r]

                # 5. 原来的左上 -> 右上
                matrix[top + i][r] = top_left

            # 进入下一层
            l += 1
            r -= 1
```

------

## 如何理解这些下标

这是这个方法最容易出错的地方。

当前四个位置分别为：

```python
# 左上
matrix[top][l + i]

# 右上
matrix[top + i][r]

# 右下
matrix[bottom][r - i]

# 左下
matrix[bottom - i][l]
```

可以记成：

| 位置 | 行           | 列      |
| ---- | ------------ | ------- |
| 左上 | `top`        | `l + i` |
| 右上 | `top + i`    | `r`     |
| 右下 | `bottom`     | `r - i` |
| 左下 | `bottom - i` | `l`     |

观察规律：

- 顶边：**行不变，列增加**
- 右边：**列不变，行增加**
- 底边：**行不变，列减少**
- 左边：**列不变，行减少**

这个规律比死记四个公式更可靠。

------

## 为什么 `range(r - l)`？

例如当前处理一个 `4 × 4` 矩阵：

```text
a b c d
e f g h
i j k l
m n o p
```

最外层：

```text
l = 0
r = 3
```

于是：

```python
range(r - l)
```

等于：

```python
range(3)
```

也就是：

```text
i = 0
i = 1
i = 2
```

对应三组四元素旋转。

如果执行四次，最后一组会和第一组产生重叠。

------

## 复杂度

时间复杂度：

$O(n^2)$

虽然代码看起来是“层 × 边”，但整个矩阵中的元素本质上只被常数次处理。

空间复杂度：

$O(1)$

只使用：

```text
l
r
i
top
bottom
top_left
```

这些常数数量的变量。

------

# 方法三：上下翻转 + 转置

这是这道题最值得掌握的方法之一，也是我更推荐面试时使用的方法。

代码短，而且数学结构非常清晰。

## 核心思路

顺时针旋转 90° 可以拆成两个操作：

1. **上下翻转矩阵**
2. **沿主对角线转置**

即：

```text
Reverse vertically
        +
Transpose
        =
Rotate 90° clockwise
```

------

## 第一步：上下翻转

原矩阵：

```text
1 2 3
4 5 6
7 8 9
```

执行：

```python
matrix.reverse()
```

得到：

```text
7 8 9
4 5 6
1 2 3
```

注意这里并不是把每一行：

```text
1 2 3
```

变成：

```text
3 2 1
```

而是**交换行的位置**：

```text
第一行 <-> 最后一行
```

------

## 第二步：转置

转置的含义是：

$matrix[i][j]\leftrightarrow matrix[j][i]$

也就是沿着主对角线：

```text
↘
```

交换两边的元素。

对：

```text
7 8 9
4 5 6
1 2 3
```

进行转置：

```text
7 4 1
8 5 2
9 6 3
```

这正是顺时针旋转 90° 的结果。

------

## Python 代码

```python
from typing import List


class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        # 第一步：上下翻转
        # 第一行和最后一行交换，第二行和倒数第二行交换……
        matrix.reverse()

        # 第二步：沿主对角线进行转置
        n = len(matrix)

        for i in range(n):
            # 只遍历主对角线右上方的元素
            # 防止同一对元素被交换两次
            for j in range(i + 1, n):
                matrix[i][j], matrix[j][i] = (
                    matrix[j][i],
                    matrix[i][j],
                )
```

也可以更紧凑地写：

```python
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        matrix.reverse()

        for i in range(len(matrix)):
            for j in range(i + 1, len(matrix)):
                matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```

------

# 为什么“上下翻转 + 转置”成立？

这是一个非常值得理解的地方。

原元素坐标：

$(i,j)$

首先上下翻转。

第 `i` 行变成：

$n-1-i$

因此：

$(i,j)
\rightarrow
(n-1-i,j)$

然后转置，交换行列：

$(n-1-i,j)
\rightarrow
(j,n-1-i)$

最终得到：

$(i,j)
\rightarrow
(j,n-1-i)$

这正好就是**顺时针旋转 90°** 的坐标公式。

所以：

```text
上下翻转 + 转置
```

并不是一个需要死记的技巧，它实际上直接推导出了旋转的坐标变化。

------

# Python 函数说明：`list.reverse()`

代码中：

```python
matrix.reverse()
```

调用的是 Python `list` 的原地反转方法。

例如：

```python
nums = [1, 2, 3]
nums.reverse()

print(nums)
```

结果：

```python
[3, 2, 1]
```

它直接修改原列表，而不是创建一个新的列表。

因此：

```python
matrix.reverse()
```

会原地修改：

```python
matrix
```

空间复杂度可以视为：

$O(1)$

需要注意，它和：

```python
matrix[::-1]
```

不完全相同。

```python
matrix[::-1]
```

会创建一个新的列表，而：

```python
matrix.reverse()
```

直接修改原列表。

对于强调 **in-place** 的面试题，一般更推荐：

```python
matrix.reverse()
```

------

# Python 语法说明：多变量交换

代码中：

```python
matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```

是 Python 常见的交换写法。

传统写法可能是：

```python
temp = matrix[i][j]
matrix[i][j] = matrix[j][i]
matrix[j][i] = temp
```

Python 可以直接写成：

```python
a, b = b, a
```

右侧会先求值，因此不会出现“第一个赋值覆盖第二个值”的问题。

------

# 为什么转置时只遍历 `j > i`？

代码：

```python
for i in range(n):
    for j in range(i + 1, n):
```

只遍历主对角线的一侧。

例如：

```text
x a b
  x c
    x
```

只处理：

```text
a
b
c
```

原因是 `(i, j)` 和 `(j, i)` 本身就是一对。

例如：

```python
matrix[0][1], matrix[1][0] = ...
```

已经完成了一次交换。

如果后面又处理：

```python
matrix[1][0], matrix[0][1] = ...
```

它们就会再次交换回来。

最终相当于什么都没做。

所以通常写：

```python
for j in range(i + 1, n):
```

而不是：

```python
for j in range(n):
```

主对角线上的：

```python
matrix[i][i]
```

也不需要交换，因为：

```python
matrix[i][i]
```

和自己交换没有任何意义。

------

## 复杂度

上下翻转：

$O(n)$

这里交换的是 `n` 个“行对象”。

矩阵转置需要处理约：

$\frac{n(n-1)}{2}$

个元素对，因此：

$O(n^2)$

整体时间复杂度：

$O(n^2)$

额外空间复杂度：

$O(1)$

------

# 三种方法对比

| 方法            | 时间复杂度 | 空间复杂度 | 是否满足题目要求 | 特点                       |
| --------------- | ---------- | ---------- | ---------------- | -------------------------- |
| 额外矩阵        | `O(n²)`    | `O(n²)`    | ❌                | 最直观，容易理解           |
| 四元素交换      | `O(n²)`    | `O(1)`     | ✅                | 真正模拟旋转，但下标较复杂 |
| 上下翻转 + 转置 | `O(n²)`    | `O(1)`     | ✅                | 代码最简洁，推荐掌握       |

面试中，如果没有额外限制，我通常会优先选择：

```text
上下翻转 + 转置
```

因为它同时具有：

- `O(n²)` 时间复杂度
- `O(1)` 额外空间
- 代码短
- 不容易出现复杂的索引错误
- 很容易通过坐标变换证明正确性

------

# 常见错误

## 1. 混淆顺时针和逆时针

顺时针 90°：

```text
1 2 3
4 5 6
7 8 9

↓

7 4 1
8 5 2
9 6 3
```

一种实现方式是：

```text
上下翻转
→
转置
```

而逆时针旋转 90° 可以使用另一组对应的翻转与转置操作。

面试中比死记操作顺序更稳妥的方法，是记住顺时针旋转的坐标公式：

$(i,j)\rightarrow(j,n-1-i)$

然后检查你的操作最终是否产生这个坐标变化。

------

## 2. 转置时遍历整个矩阵

错误写法：

```python
for i in range(n):
    for j in range(n):
        matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```

这里：

```text
(0, 1)
```

和：

```text
(1, 0)
```

会先交换一次，然后又交换回来。

正确写法：

```python
for i in range(n):
    for j in range(i + 1, n):
        matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```

只处理主对角线的一侧。

------

## 3. 四元素交换时下标计算错误

四元素方法最容易出现的问题就是下标混乱：

```python
matrix[top][l + i]
matrix[top + i][r]
matrix[bottom][r - i]
matrix[bottom - i][l]
```

与其纯粹背诵，建议记住四条边的变化：

```text
顶部：从左向右
右侧：从上向下
底部：从右向左
左侧：从下向上
```

因此偏移量分别表现为：

```text
l + i
top + i
r - i
bottom - i
```

------

## 4. 覆盖尚未保存的值

例如直接写：

```python
matrix[top][l + i] = matrix[bottom - i][l]
matrix[top + i][r] = matrix[top][l + i]
```

第二行读取的：

```python
matrix[top][l + i]
```

已经不是原来的左上角了，而是刚刚移动过来的左下角。

因此至少需要暂存一个值：

```python
top_left = matrix[top][l + i]
```

再依次完成四个位置的循环交换。

------

# 面试记忆总结

这道题可以浓缩成三个层次。

### 最基本的坐标公式

顺时针 90°：

```text
(i, j) -> (j, n - 1 - i)
```

### 最推荐的原地算法

```python
matrix.reverse()

for i in range(n):
    for j in range(i + 1, n):
        matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```

记忆为：

```text
上下翻转
+
主对角线转置
=
顺时针旋转 90°
```

### 如果面试官要求“不使用 reverse 等辅助方法”

使用：

```text
逐层处理
+
每次四个元素循环交换
```

同样可以做到：

```text
Time:  O(n²)
Space: O(1)
```

对于代码面试，这道题最重要的并不是记住某一段代码，而是能够解释：

> **为什么上下翻转以后再转置，会得到 `(i, j) -> (j, n - 1 - i)`。**

只要能够推导这个坐标变化，即使面试时忘记具体代码，也很容易重新写出来。
