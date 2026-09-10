# Pascal's Triangle（杨辉三角）

给定一个整数 `rowIndex`，返回 Pascal's Triangle（杨辉三角）的第 `rowIndex` 行。

这里的行号从 **0 开始计数（0-indexed）**。

在杨辉三角中，每个位置的数字等于它正上方两个相邻数字之和。

例如：

```text
row 0:        [1]
row 1:       [1, 1]
row 2:      [1, 2, 1]
row 3:     [1, 3, 3, 1]
row 4:    [1, 4, 6, 4, 1]
```

如果：

```text
rowIndex = 3
```

则返回：

```text
[1, 3, 3, 1]
```

------

## 一、核心递推关系

杨辉三角最重要的规律是：

```text
triangle[i][j]
=
triangle[i - 1][j - 1]
+
triangle[i - 1][j]
```

也就是说，第 `i` 行内部位置 `j` 的值，等于上一行中：

- 左上方 `j - 1`
- 右上方 `j`

两个元素之和。

每一行的第一个元素和最后一个元素始终为：

```text
1
```

因此，如果上一行为：

```text
[1, 3, 3, 1]
```

下一行就是：

```text
[1, 4, 6, 4, 1]
```

因为：

```text
4 = 1 + 3
6 = 3 + 3
4 = 3 + 1
```

下面介绍几种常见解法。

------

# 解法一：动态规划——递归 / Top-Down

## 思路

为了得到第 `n` 行，我们可以先得到第 `n - 1` 行。

因为当前行中的每一个内部元素都依赖于上一行的两个相邻元素，所以这个问题天然具有递归结构：

```text
第 n 行
↓
先计算第 n - 1 行
↓
根据第 n - 1 行构造第 n 行
```

例如，要计算：

```text
[1, 3, 3, 1]
```

先递归得到：

```text
[1, 2, 1]
```

然后计算：

```text
1
1 + 2 = 3
2 + 1 = 3
1
```

得到：

```text
[1, 3, 3, 1]
```

------

## 算法步骤

1. 如果 `rowIndex == 0`，直接返回：

```text
[1]
```

1. 递归计算上一行：

```python
prev_row = self.getRow(rowIndex - 1)
```

1. 当前行第一个元素一定是 `1`。
2. 对于所有内部元素：

```text
cur_row[i]
=
prev_row[i - 1]
+
prev_row[i]
```

1. 最后再添加一个 `1`。
2. 返回当前行。

------

## Python 代码

```python
from typing import List


class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        # 基础情况：
        # 杨辉三角第 0 行只有一个元素 1
        if rowIndex == 0:
            return [1]

        # 递归计算上一行
        prev_row = self.getRow(rowIndex - 1)

        # 当前行第一个元素一定为 1
        cur_row = [1]

        # 根据上一行计算当前行的内部元素
        for i in range(1, rowIndex):
            cur_row.append(prev_row[i - 1] + prev_row[i])

        # 当前行最后一个元素也一定为 1
        cur_row.append(1)

        return cur_row
```

------

## 复杂度分析

设：

```text
n = rowIndex
```

第 `n` 行需要计算大约 `n` 个元素，第 `n - 1` 行需要计算约 `n - 1` 个元素，以此类推。

因此总计算量为：

```text
1 + 2 + 3 + ... + n
```

所以：

```text
时间复杂度：O(n²)
```

递归过程中需要保存调用栈，同时每一层都会创建对应的一行。

如果考虑递归执行过程中同时存在的所有行，额外空间可能达到：

```text
O(n²)
```

如果只讨论递归调用栈深度，则调用栈为：

```text
O(n)
```

------

## 面试讨论

这种写法很好地表达了杨辉三角的递归关系，但通常不是最优方案。

主要问题有两个：

1. 为了得到第 `n` 行，需要递归构造之前所有行。
2. 递归本身带来了额外的调用栈开销。

因此，如果题目只要求返回某一行，通常更推荐后面的空间优化 DP 或组合数学解法。

------

# 解法二：动态规划——Bottom-Up

## 思路

与递归不同，我们也可以从第 `0` 行开始，一行一行构造整个杨辉三角：

```text
row 0
↓
row 1
↓
row 2
↓
...
↓
row rowIndex
```

每一行都根据上一行计算。

这是一种典型的 Bottom-Up Dynamic Programming（自底向上的动态规划）。

------

## 算法步骤

首先创建所有行。

第 `i` 行有：

```text
i + 1
```

个元素，并且最开始全部初始化为：

```text
1
```

例如，如果：

```text
rowIndex = 4
```

初始化结果：

```text
[
    [1],
    [1, 1],
    [1, 1, 1],
    [1, 1, 1, 1],
    [1, 1, 1, 1, 1]
]
```

然后从第 `2` 行开始更新内部元素：

```text
res[i][j]
=
res[i - 1][j - 1]
+
res[i - 1][j]
```

最终返回：

```python
res[rowIndex]
```

------

## Python 代码

```python
from typing import List


class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        # 创建整个杨辉三角
        # 第 i 行拥有 i + 1 个元素
        # 每行首尾元素都为 1，因此先全部初始化为 1
        res = [[1] * (i + 1) for i in range(rowIndex + 1)]

        # 第 0 行和第 1 行不需要计算内部元素
        for i in range(2, rowIndex + 1):

            # 只计算内部元素
            for j in range(1, i):
                res[i][j] = (
                    res[i - 1][j - 1]
                    + res[i - 1][j]
                )

        return res[rowIndex]
```

------

## 复杂度分析

我们需要计算整个杨辉三角：

```text
1 + 2 + 3 + ... + n
```

因此：

```text
时间复杂度：O(n²)
```

同时需要保存所有行，因此：

```text
空间复杂度：O(n²)
```

------

## Python 语法补充：二维 List Comprehension

这一行：

```python
res = [[1] * (i + 1) for i in range(rowIndex + 1)]
```

是 Python 的 List Comprehension（列表推导式）。

例如：

```python
rowIndex = 3
```

相当于：

```python
res = [
    [1] * 1,
    [1] * 2,
    [1] * 3,
    [1] * 4
]
```

最终得到：

```python
[
    [1],
    [1, 1],
    [1, 1, 1],
    [1, 1, 1, 1]
]
```

------

# 解法三：动态规划——空间优化 I

## 思路

前面的二维 DP 保存了整个杨辉三角。

但实际上，当我们计算当前行时，只需要上一行：

```text
previous row
      ↓
 current row
```

之前更早的行已经没有用了。

因此我们完全不需要保存：

```text
row 0
row 1
row 2
...
```

而只需要保存当前这一行。

------

## 一个非常直观的构造方法

假设当前行为：

```text
[1, 2, 1]
```

每一个元素都会分别贡献给下一行的两个位置。

对于第一个 `1`：

```text
[1, 1, 0, 0]
```

对于 `2`：

```text
[1, 3, 2, 0]
```

对于最后一个 `1`：

```text
[1, 3, 3, 1]
```

于是就得到了下一行。

------

## 算法步骤

1. 初始化：

```python
res = [1]
```

1. 重复 `rowIndex` 次。
2. 每一次创建一个比当前数组长度多 `1` 的数组：

```python
next_row = [0] * (len(res) + 1)
```

1. 对当前行每一个元素：

```python
next_row[j] += res[j]
next_row[j + 1] += res[j]
```

也就是说，一个元素分别向左下和右下贡献自己的值。

1. 更新：

```python
res = next_row
```

1. 最终返回 `res`。

------

## Python 代码

```python
from typing import List


class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        # 第 0 行
        res = [1]

        # 每一轮构造下一行
        for _ in range(rowIndex):

            # 下一行一定比当前行多一个元素
            next_row = [0] * (len(res) + 1)

            for j in range(len(res)):
                # 当前元素贡献给左下方
                next_row[j] += res[j]

                # 当前元素贡献给右下方
                next_row[j + 1] += res[j]

            # 进入下一轮
            res = next_row

        return res
```

------

## 复杂度分析

虽然我们只保存一行，但还是需要逐行计算：

```text
1 + 2 + ... + n
```

因此：

```text
时间复杂度：O(n²)
空间复杂度：O(n)
```

这里 `O(n)` 的空间主要来自最终结果数组和 `next_row`。

------

## 面试理解：Push 和 Pull

这种写法可以理解成一种 **Push（向外贡献）** 思路。

传统递推是：

```text
当前位置
=
左上
+
右上
```

这是从上一行“取值”。

而这里换一个角度：

```text
当前元素
→ 左下
→ 右下
```

每个元素把自己的值“推送”到下一行的两个位置。

两种理解本质完全相同。

------

# 解法四：动态规划——原地更新 / 空间优化 II

## 思路

前一个空间优化方案仍然需要：

```python
res
next_row
```

两个数组。

我们实际上还可以只使用一个数组。

核心技巧是：

> 从右向左更新数组。

假设当前有效数据是：

```text
[1, 3, 3, 1]
```

下一行中的某个元素应该满足：

```text
row[j]
=
旧 row[j]
+
旧 row[j - 1]
```

问题在于，如果从左向右更新：

```text
row[1]
row[2]
row[3]
```

当我们更新 `row[1]` 以后，它已经不是上一行的数据了。

随后计算 `row[2]` 时，就会错误地使用这个已经修改过的值。

因此必须：

```text
从右向左
```

更新。

------

## 为什么一定要从右向左？

假设上一行是：

```text
[1, 3, 3, 1]
```

要计算：

```text
[1, 4, 6, 4, 1]
```

对于中间位置：

```text
6 = 3 + 3
```

如果从左向右：

```python
row[1] = row[1] + row[0]
```

数组先变成：

```text
[1, 4, 3, 1]
```

然后：

```python
row[2] = row[2] + row[1]
```

计算的却是：

```text
3 + 4 = 7
```

而正确答案应该是：

```text
3 + 3 = 6
```

因为原来的 `row[1]` 已经被覆盖。

如果从右向左，就不会出现这个问题。

------

## Python 代码

```python
from typing import List


class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        # 最终答案长度一定是 rowIndex + 1
        # 首尾始终为 1，因此可以全部初始化为 1
        row = [1] * (rowIndex + 1)

        # 逐行计算
        for i in range(1, rowIndex):

            # 必须从右向左更新
            # range(i, 0, -1) 表示：
            # i, i-1, ..., 1
            for j in range(i, 0, -1):
                row[j] += row[j - 1]

        return row
```

------

## 复杂度分析

需要执行大约：

```text
1 + 2 + ... + n
```

次更新，因此：

```text
时间复杂度：O(n²)
空间复杂度：O(n)
```

如果按照很多面试题对「额外空间」的定义，不将返回结果本身算入额外空间，那么这个方案甚至可以描述为：

```text
额外空间复杂度：O(1)
```

因为所有计算都直接发生在最终返回的数组上。

------

## Python 函数补充：range(start, stop, step)

这里：

```python
range(i, 0, -1)
```

表示从 `i` 开始，每次减 `1`，直到 `1`。

例如：

```python
list(range(4, 0, -1))
```

得到：

```python
[4, 3, 2, 1]
```

注意：

```text
stop = 0
```

本身不会被包含。

因此：

```python
range(i, 0, -1)
```

恰好可以让 `j` 从 `i` 遍历到 `1`。

------

# 解法五：组合数学

## 思路

这是这道题最值得掌握的优化。

杨辉三角的第 `n` 行其实就是一组二项式系数：

```text
C(n, 0),
C(n, 1),
C(n, 2),
...
C(n, n)
```

例如第 `4` 行：

```text
[1, 4, 6, 4, 1]
```

实际上就是：

```text
C(4,0) = 1
C(4,1) = 4
C(4,2) = 6
C(4,3) = 4
C(4,4) = 1
```

组合数公式为：

```text
         n!
C(n,k) = ─────────
         k!(n-k)!
```

但如果每个元素都直接计算阶乘，会产生大量重复计算。

我们可以利用相邻组合数之间的关系：

```text
C(n,k)
=
C(n,k-1) × (n-k+1) / k
```

因此，只要知道前一个值，就可以在 `O(1)` 时间计算下一个值。

------

## 公式推导

因为：

```text
C(n,k-1)
=
n! / ((k-1)! × (n-k+1)!)
```

而：

```text
C(n,k)
=
n! / (k! × (n-k)!)
```

两者相除：

```text
C(n,k)
────────
C(n,k-1)

=
n-k+1
───────
   k
```

所以：

```text
C(n,k)
=
C(n,k-1)
×
(n-k+1)
────────
   k
```

------

## 算法步骤

1. 第一个组合数一定是：

```text
C(n,0) = 1
```

所以：

```python
row = [1]
```

1. 对：

```text
i = 1 ... rowIndex
```

使用：

```text
C(n,i)
=
C(n,i-1)
×
(n-i+1)
─────────
    i
```

1. 将得到的新组合数加入数组。

------

## Python 代码

```python
from typing import List


class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        # C(n, 0) 永远等于 1
        row = [1]

        for i in range(1, rowIndex + 1):
            # 根据前一个组合数计算当前组合数：
            #
            # C(n, i)
            # = C(n, i - 1) * (n - i + 1) / i
            #
            # Python 中使用 // 做整数除法
            next_value = (
                row[-1]
                * (rowIndex - i + 1)
                // i
            )

            row.append(next_value)

        return row
```

------

## 示例

假设：

```text
rowIndex = 4
```

最开始：

```text
row = [1]
```

### i = 1

```text
1 × (4 - 1 + 1) / 1
= 4
```

得到：

```text
[1, 4]
```

### i = 2

```text
4 × (4 - 2 + 1) / 2
= 4 × 3 / 2
= 6
```

得到：

```text
[1, 4, 6]
```

### i = 3

```text
6 × 2 / 3
= 4
```

得到：

```text
[1, 4, 6, 4]
```

### i = 4

```text
4 × 1 / 4
= 1
```

最终：

```text
[1, 4, 6, 4, 1]
```

------

## 复杂度分析

只需要遍历目标行一次：

```text
时间复杂度：O(n)
```

最终需要保存：

```text
n + 1
```

个数字，因此：

```text
空间复杂度：O(n)
```

如果不计算返回数组本身，则：

```text
额外空间复杂度：O(1)
```

这是这几种方法中时间复杂度最优的方法。

------

# Python 相关知识补充

## 1. `List[int]`

代码中：

```python
def getRow(self, rowIndex: int) -> List[int]:
```

使用的是 Python Type Hint（类型注解）。

其中：

```python
rowIndex: int
```

表示参数 `rowIndex` 预期为整数。

而：

```python
-> List[int]
```

表示函数预期返回一个整数列表。

通常需要：

```python
from typing import List
```

在较新的 Python 中也可以直接写：

```python
def getRow(self, rowIndex: int) -> list[int]:
```

------

## 2. `row[-1]`

Python 支持负数索引。

```python
row[-1]
```

表示数组最后一个元素。

例如：

```python
row = [1, 4, 6]
```

那么：

```python
row[-1]
```

就是：

```text
6
```

因此组合数学方案中：

```python
row[-1] * ...
```

就是使用刚刚计算出的前一个组合数。

------

## 3. `/` 和 `//`

Python 中：

```python
/
```

表示普通除法，结果通常是浮点数。

例如：

```python
6 / 3
```

结果：

```text
2.0
```

而：

```python
//
```

表示整除。

例如：

```python
6 // 3
```

结果：

```text
2
```

组合数一定是整数，因此这里使用：

```python
//
```

避免产生不必要的浮点数。

------

# 常见错误

## 1. 原地 DP 时从左向右更新

这是这道题最经典的陷阱之一。

如果写成：

```python
for j in range(1, i + 1):
    row[j] += row[j - 1]
```

就是错误的。

因为 `row[j - 1]` 很可能已经在这一轮被修改。

正确方式必须：

```python
for j in range(i, 0, -1):
    row[j] += row[j - 1]
```

即：

```text
从右向左更新
```

这是所有「一维数组压缩二维 DP」问题中都很常见的技巧。

面试中遇到原地 DP 时，可以主动思考：

> 当前状态依赖的是上一轮的数据，更新方向会不会导致旧状态被提前覆盖？

------

## 2. 组合数计算中的整数溢出

在 Python 中，整数具有任意精度，因此通常不需要担心普通整数溢出。

但在 Java、C++ 等语言中：

```text
C(n,k-1) × (n-k+1)
```

这个中间结果可能在除以 `k` 之前就已经超过 `int` 范围。

因此通常应该使用：

```text
long
```

或：

```text
long long
```

保存中间结果。

例如 C++ 中可以写成：

```cpp
long long value =
    previous * (n - k + 1) / k;
```

所以这并不是 Python 中的主要问题，但在跨语言面试中值得注意。

------

## 3. 行号从 0 开始

题目明确规定：

```text
0-indexed
```

因此：

```text
rowIndex = 0
```

返回：

```text
[1]
```

而：

```text
rowIndex = 1
```

返回：

```text
[1, 1]
```

不要把：

```text
rowIndex = 1
```

误认为杨辉三角的第一行：

```text
[1]
```

这是常见的 Off-by-One Error。

------

# 几种方案对比

| 方法              | 时间复杂度 | 空间复杂度     | 特点                       |
| ----------------- | ---------- | -------------- | -------------------------- |
| 递归 Top-Down     | `O(n²)`    | 最坏约 `O(n²)` | 递归关系直观，但效率一般   |
| 二维 Bottom-Up DP | `O(n²)`    | `O(n²)`        | 最容易理解的标准 DP        |
| 两行空间优化 DP   | `O(n²)`    | `O(n)`         | 不保存整个三角形           |
| 原地 DP           | `O(n²)`    | `O(n)`         | 面试常见，需要理解倒序更新 |
| 组合数学          | `O(n)`     | `O(n)`         | 时间最优，适合只求某一行   |

其中空间复杂度包含最终返回数组。

如果只计算「额外辅助空间」，那么：

```text
原地 DP：O(1)
组合数学：O(1)
```

------

# 面试推荐

如果面试官首先希望看到动态规划，可以先给出：

```text
二维 DP
```

然后指出：

> 由于计算当前行时只依赖上一行，因此不需要保存整个二维三角形。

于是优化到：

```text
O(n) 空间
```

接着进一步说明：

> 我们甚至可以直接在同一个数组中从右向左更新，这样不会覆盖仍然需要的上一轮数据。

得到原地 DP。

如果面试官继续问是否还能优化时间复杂度，就可以使用组合数学：

```text
C(n,k)
=
C(n,k-1)
×
(n-k+1)
─────────
    k
```

最终做到：

```text
O(n) 时间
```

因此比较完整的面试推导路线可以是：

```text
二维 DP
    ↓
只保留上一行
    ↓
原地倒序更新
    ↓
观察到杨辉三角本质是组合数
    ↓
O(n) 组合数学解法
```

------

# 最推荐代码：组合数学

如果题目只要求返回第 `rowIndex` 行，那么组合数学方案最简洁、时间复杂度也最好。

```python
from typing import List


class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        row = [1]

        for k in range(1, rowIndex + 1):
            # C(n, k)
            # = C(n, k - 1) * (n - k + 1) / k
            row.append(
                row[-1]
                * (rowIndex - k + 1)
                // k
            )

        return row
```

复杂度：

```text
时间复杂度：O(n)
空间复杂度：O(n)
额外辅助空间：O(1)
```

------

# 一句话记忆

这道题最值得记住两个技巧：

```text
DP 原地更新：
依赖左侧旧值 → 从右向左更新

Pascal Triangle：
第 n 行 = C(n,0), C(n,1), ..., C(n,n)
```

尤其是「**为了避免覆盖上一轮状态而反向遍历**」这个思想，在很多一维 DP 空间压缩题目中都会再次出现。
