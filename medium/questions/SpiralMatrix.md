# Spiral Matrix（螺旋矩阵）

给定一个 `m × n` 的整数矩阵 `matrix`，请按照**顺时针螺旋顺序（Spiral Order）**返回矩阵中的所有元素。

例如：

```text
matrix =
[
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
]
```

遍历顺序为：

```text
1 → 2 → 3
        ↓
4 → 5   6
↑       ↓
7 ← 8 ← 9
```

因此返回：

```python
[1, 2, 3, 6, 9, 8, 7, 4, 5]
```

------

# 核心思路

这道题本质上是在模拟如下循环移动：

```text
向右 → 向下 → 向左 → 向上 → 向右 → ...
```

每完成一条边的遍历，下一次能够遍历的区域就会缩小。

常见的实现方式有三种：

1. **递归**
   - 把每一条边看成一个递归子问题。
2. **四边界迭代**
   - 维护 `top / bottom / left / right` 四个边界。
   - 最直观，也最适合面试。
3. **方向 + 步数迭代**
   - 不显式维护四个边界，而是记录每个方向还能走多少步。
   - 代码更紧凑，但理解成本稍高。

面试中通常最推荐 **方法二：四边界迭代**。

------

# 1. 递归解法

## 思路

可以把整个螺旋遍历看成不断处理越来越小的矩形。

对于当前尚未访问的矩形区域，我们维护：

- `row`：当前矩形还有多少行
- `col`：当前方向需要访问多少个元素
- `(r, c)`：当前所在位置
- `(dr, dc)`：当前移动方向

方向 `(dr, dc)` 可以表示为：

```text
向右：(0, 1)
向下：(1, 0)
向左：(0, -1)
向上：(-1, 0)
```

每次递归做两件事：

1. 沿当前方向连续访问 `col` 个元素。
2. 顺时针转向，然后递归处理剩余的更小矩形。

------

## 为什么递归参数是 `dfs(col, row - 1, ...)`

假设当前矩形大小是：

```text
row × col
```

我们首先沿当前方向访问 `col` 个元素。

例如第一次是访问最上面一行：

```text
→ → → → →
```

这一整条边访问完成后，其中一个维度已经减少了 `1`。

然后因为我们进行了 90° 转向，所以：

```text
原来的 row
```

会成为下一轮对应的：

```text
col
```

因此递归参数变为：

```python
dfs(col, row - 1, ...)
```

可以把它理解为：

> 每转 90°，矩形的“宽”和“高”的角色互换，同时刚刚走完的一条边被移除。

------

## 顺时针旋转方向

当前方向为：

```python
(dr, dc)
```

顺时针旋转 90° 后：

```python
(dc, -dr)
```

例如：

```text
向右
(0, 1)

↓

向下
(1, 0)
```

代入公式：

```python
(dc, -dr)
= (1, 0)
```

同理：

```text
(1, 0)   → (0, -1)
向下         向左

(0, -1)  → (-1, 0)
向左         向上

(-1, 0)  → (0, 1)
向上         向右
```

------

## 代码

```python
from typing import List


class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        m, n = len(matrix), len(matrix[0])
        res = []

        def dfs(row, col, r, c, dr, dc):
            """
            row, col:
                当前剩余矩形的两个维度

            r, c:
                当前所在的位置

            dr, dc:
                当前移动方向
            """

            # 如果某个维度变成 0，
            # 说明已经没有元素需要访问
            if row == 0 or col == 0:
                return

            # 沿当前方向访问 col 个元素
            for _ in range(col):
                r += dr
                c += dc

                res.append(matrix[r][c])

            # 顺时针旋转方向：
            # (dr, dc) -> (dc, -dr)
            #
            # 同时：
            # - 行列角色互换
            # - 已经完整处理掉一条边，所以 row - 1
            dfs(
                col,
                row - 1,
                r,
                c,
                dc,
                -dr
            )

        # 从矩阵左上角的左边开始：
        #
        # (0, -1)
        #
        # 第一次移动方向为向右 (0, 1)，
        # 因此第一次移动就会进入 matrix[0][0]
        dfs(m, n, 0, -1, 0, 1)

        return res
```

------

## 为什么从 `(0, -1)` 开始？

因为我们希望递归函数内部统一采用：

```python
r += dr
c += dc

res.append(matrix[r][c])
```

即：

> **先移动，再访问。**

所以把起点设置在矩阵左上角 `(0, 0)` 的左侧：

```text
起点
 ↓
(*) → [0][0] → [0][1] → [0][2]
```

也就是：

```python
(r, c) = (0, -1)
```

第一次：

```python
dr = 0
dc = 1
```

因此：

```python
r = 0
c = 0
```

刚好进入第一个元素。

------

## 复杂度分析

设：

- `m` = 行数
- `n` = 列数

### 时间复杂度

```text
O(m × n)
```

每个元素只会被访问一次。

### 空间复杂度

如果**不计算返回结果**：

```text
O(min(m, n))
```

主要来自递归调用栈。

每经过两次转向，实际上就相当于剥掉矩阵的一层，因此递归深度与较短的矩阵维度同阶。

如果把输出数组 `res` 也计算进去：

```text
O(m × n)
```

因为最终需要保存所有元素。

------

# 2. 四边界迭代法

## 思路

这是这道题最经典、最直观的解法。

我们维护四个边界：

```text
top
 ↓
+-------------+
|             |
|             |
|             |
+-------------+
              ↑
            bottom
```

以及：

```text
left         right
 ↓             ↓
+-------------+
|             |
|             |
+-------------+
```

这里代码采用的是类似 Python 切片的**左闭右开区间**：

```python
top <= row < bottom
left <= col < right
```

因此：

```python
left = 0
right = n

top = 0
bottom = m
```

注意：

```python
right
bottom
```

本身都不属于当前有效区域。

------

## 每一轮遍历

每一层按照以下顺序：

### ① 上边：左 → 右

```text
→ → → → →
```

访问：

```python
matrix[top][left:right]
```

然后：

```python
top += 1
```

------

### ② 右边：上 → 下

```text
        ↓
        ↓
        ↓
```

访问：

```python
matrix[top:bottom][right - 1]
```

然后：

```python
right -= 1
```

------

### ③ 下边：右 → 左

```text
← ← ← ← ←
```

访问完成后：

```python
bottom -= 1
```

------

### ④ 左边：下 → 上

```text
↑
↑
↑
```

访问完成后：

```python
left += 1
```

然后进入下一层。

------

## 代码

```python
from typing import List


class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        res = []

        # 使用左闭右开的边界：
        #
        # 有效列范围：[left, right)
        # 有效行范围：[top, bottom)
        left, right = 0, len(matrix[0])
        top, bottom = 0, len(matrix)

        while left < right and top < bottom:

            # 1. 遍历上边：左 -> 右
            for col in range(left, right):
                res.append(matrix[top][col])

            # 上边已经处理完
            top += 1

            # 2. 遍历右边：上 -> 下
            for row in range(top, bottom):
                res.append(matrix[row][right - 1])

            # 右边已经处理完
            right -= 1

            # 非常重要：
            # 上边和右边处理之后，
            # 剩余矩形可能已经不存在。
            #
            # 如果不检查，单行/单列矩阵可能产生重复元素。
            if not (left < right and top < bottom):
                break

            # 3. 遍历下边：右 -> 左
            for col in range(right - 1, left - 1, -1):
                res.append(matrix[bottom - 1][col])

            # 下边已经处理完
            bottom -= 1

            # 4. 遍历左边：下 -> 上
            for row in range(bottom - 1, top - 1, -1):
                res.append(matrix[row][left])

            # 左边已经处理完
            left += 1

        return res
```

------

# `range()` 在这道题中的使用

这道题大量使用 Python 的：

```python
range(start, stop, step)
```

其生成范围遵循：

```text
包含 start
不包含 stop
```

即：

```python
[start, stop)
```

例如：

```python
range(0, 4)
```

产生：

```python
0, 1, 2, 3
```

------

## 正向遍历

```python
for col in range(left, right):
```

表示：

```text
left → right - 1
```

------

## 反向遍历

比较容易出错的是：

```python
for col in range(right - 1, left - 1, -1):
```

因为 `stop` 不包含。

如果希望访问：

```text
right - 1
right - 2
...
left
```

那么必须写：

```python
range(right - 1, left - 1, -1)
```

而不是：

```python
range(right - 1, left, -1)
```

否则 `left` 本身不会被访问。

同理：

```python
range(bottom - 1, top - 1, -1)
```

表示：

```text
bottom - 1 → top
```

------

# 为什么中间一定要再次检查边界？

这是四边界解法中最重要的细节之一。

假设矩阵只有一行：

```text
[1, 2, 3, 4]
```

第一次遍历上边：

```text
1 → 2 → 3 → 4
```

此时：

```python
top += 1
```

剩余矩形实际上已经不存在。

如果继续执行“下边从右向左”，就会再次加入：

```text
4 → 3 → 2 → 1
```

造成重复。

因此完成：

```text
上边
右边
```

之后，需要再次检查：

```python
if not (left < right and top < bottom):
    break
```

这是为了处理矩阵最终退化成：

- 单行
- 单列

的情况。

------

# 复杂度分析

### 时间复杂度

```text
O(m × n)
```

虽然代码中有四个 `for` 循环，但是每个矩阵元素最终只会被访问一次。

因此不是：

```text
O(4mn)
```

即使写成：

```text
O(4mn)
```

在渐进复杂度中也会去掉常数：

```text
O(mn)
```

------

### 空间复杂度

不计算返回结果：

```text
O(1)
```

只维护了：

```python
left
right
top
bottom
```

几个变量。

如果包括返回数组：

```text
O(m × n)
```

------

# 3. 方向 + 步数迭代法

## 思路

这个方法同样是迭代，但不维护：

```python
top
bottom
left
right
```

而是维护两个信息：

1. 当前移动方向
2. 当前方向还能走多少步

方向始终按照：

```text
右 → 下 → 左 → 上
```

循环。

------

## 方向数组

定义：

```python
directions = [
    (0, 1),   # 右
    (1, 0),   # 下
    (0, -1),  # 左
    (-1, 0)   # 上
]
```

令：

```python
d = 0
```

那么：

```python
directions[d]
```

就是当前方向。

方向变化：

```python
d = (d + 1) % 4
```

因此：

```text
0 → 1 → 2 → 3 → 0 → ...
```

对应：

```text
右 → 下 → 左 → 上 → 右 → ...
```

------

# 为什么步数是 `[n, m - 1]`？

定义：

```python
steps = [n, m - 1]
```

其中：

```python
steps[0]
```

表示当前水平方向还能走多少步。

```python
steps[1]
```

表示当前垂直方向还能走多少步。

假设矩阵为：

```text
m = 3
n = 4
```

第一次向右：

```text
→ → → →
```

需要访问：

```text
4 个元素
```

所以：

```python
steps[0] = n = 4
```

之后向下。

但是右上角已经在刚才访问过了，因此向下只剩：

```text
m - 1
```

个新元素。

所以：

```python
steps[1] = m - 1
```

------

# 步数为什么不断减少？

假设矩阵是：

```text
1   2   3   4
5   6   7   8
9  10  11  12
```

移动长度依次是：

```text
向右：4
向下：2
向左：3
向上：1
向右：2
...
```

可以发现：

```text
水平方向：
4 → 3 → 2 → ...

垂直方向：
2 → 1 → 0 → ...
```

每完成一次某个维度方向的遍历，就：

```python
steps[...] -= 1
```

------

# `d & 1` 是什么意思？

代码中有：

```python
steps[d & 1]
```

这里：

```python
&
```

是 Python 的**按位与运算符（bitwise AND）**。

对于：

```python
d = 0, 1, 2, 3
```

有：

```text
d      二进制      d & 1

0       00          0
1       01          1
2       10          0
3       11          1
```

因此：

```python
d & 1
```

等价于判断：

```text
d 是偶数 → 0
d 是奇数 → 1
```

而方向恰好是：

```text
d = 0：右    水平
d = 1：下    垂直
d = 2：左    水平
d = 3：上    垂直
```

所以：

```python
steps[d & 1]
```

可以非常简洁地选择：

```text
水平步数 / 垂直步数
```

从可读性角度，也可以写成：

```python
steps[d % 2]
```

两者在这里效果相同。

面试中如果担心位运算影响代码可读性：

```python
d % 2
```

通常会更加直观。

------

# 代码

```python
from typing import List


class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        res = []

        # 四个方向按照顺时针排列：
        # 右、下、左、上
        directions = [
            (0, 1),
            (1, 0),
            (0, -1),
            (-1, 0)
        ]

        # steps[0]：水平方向剩余步数
        # steps[1]：垂直方向剩余步数
        #
        # 第一次向右需要走 n 个元素；
        # 随后向下时，右上角已经访问过，
        # 因此只需要走 m - 1 个元素。
        steps = [
            len(matrix[0]),
            len(matrix) - 1
        ]

        # 从 matrix[0][0] 左边开始
        r, c = 0, -1

        # d = 0 表示首先向右
        d = 0

        # d & 1：
        # d = 0 / 2 -> 0 -> 水平方向
        # d = 1 / 3 -> 1 -> 垂直方向
        while steps[d & 1] > 0:

            # 当前方向需要移动的次数
            for _ in range(steps[d & 1]):
                dr, dc = directions[d]

                r += dr
                c += dc

                res.append(matrix[r][c])

            # 当前维度的可用长度减少 1
            steps[d & 1] -= 1

            # 顺时针切换到下一个方向
            d = (d + 1) % 4

        return res
```

------

# 复杂度分析

### 时间复杂度

```text
O(m × n)
```

每个元素访问一次。

### 空间复杂度

不计算返回结果：

```text
O(1)
```

只有：

```python
directions
steps
r
c
d
```

这些固定大小的数据。

包括返回数组：

```text
O(m × n)
```

------

# 三种方法对比

| 方法        | 时间复杂度 | 额外空间      | 优点                     | 缺点                        |
| ----------- | ---------- | ------------- | ------------------------ | --------------------------- |
| 递归        | `O(mn)`    | `O(min(m,n))` | 思路巧妙、代码简洁       | 参数抽象，面试中不容易解释  |
| 四边界      | `O(mn)`    | `O(1)`        | 最直观、最好证明正确性   | 边界更新需要谨慎            |
| 方向 + 步数 | `O(mn)`    | `O(1)`        | 代码紧凑，不需要四个边界 | `steps` 和 `d & 1` 不够直观 |

这里所谓第三种方法的 **Optimal**，主要是指：

```text
O(mn) 时间 + O(1) 额外空间
```

但第二种四边界方法实际上也已经具有：

```text
O(mn) 时间 + O(1) 额外空间
```

所以第三种方法在**渐进复杂度上并不优于第二种**。

它更多是一种更紧凑的实现方式。

对于代码面试，我会优先选择：

> **四边界迭代法。**

因为它在复杂度已经最优的情况下，逻辑最清楚，也最容易向面试官解释和证明。

------

# 常见错误

## 1. 没有在中途重新检查边界

最容易出问题的是：

```python
# 上边
...

# 右边
...

# 没检查边界

# 直接继续下边
...
```

当矩阵最后只剩：

```text
一行
```

或者：

```text
一列
```

时，会导致重复访问。

因此四边界方法中：

```python
if not (left < right and top < bottom):
    break
```

非常重要。

------

## 2. 错误处理非正方形矩阵

不要假设：

```python
m == n
```

测试时一定要考虑：

```text
1 × n
m × 1
2 × 5
5 × 2
```

例如：

```python
[
    [1, 2, 3, 4]
]
```

以及：

```python
[
    [1],
    [2],
    [3],
    [4]
]
```

都是非常重要的边界测试。

------

## 3. 反向 `range()` 写错

例如想遍历：

```text
3 → 2 → 1 → 0
```

应该：

```python
range(3, -1, -1)
```

而不是：

```python
range(3, 0, -1)
```

因为后者不会包含 `0`。

这是 Python 面试代码中很常见的 off-by-one error（差一错误）。

------

## 4. 方向旋转错误

如果使用方向 `(dr, dc)`，顺时针旋转公式是：

```python
(dr, dc) -> (dc, -dr)
```

对应：

```text
右 → 下 → 左 → 上 → 右
```

如果写成：

```python
(-dc, dr)
```

则实际上是在逆时针旋转。

------

# 面试中推荐的思考过程

看到 Spiral Matrix，可以首先想到：

> 每完成最外层的一条边，这条边以后就不能再次访问，因此可以不断缩小矩阵的有效边界。

于是定义：

```python
left
right
top
bottom
```

然后按照：

```text
上 → 右 → 下 → 左
```

遍历。

每走完一条边就把对应边界向内缩：

```python
top += 1
right -= 1
bottom -= 1
left += 1
```

最关键的特殊情况是：

```text
剩余区域可能变成单行或单列
```

所以在遍历完上边和右边之后，需要重新检查：

```python
left < right and top < bottom
```

这样就可以保证每个元素：

> **恰好访问一次。**

因此最终：

```text
时间复杂度：O(mn)
额外空间复杂度：O(1)
```

------

# 推荐记忆的标准版本

```python
from typing import List


class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        res = []

        left, right = 0, len(matrix[0])
        top, bottom = 0, len(matrix)

        while left < right and top < bottom:

            # 上：左 -> 右
            for col in range(left, right):
                res.append(matrix[top][col])
            top += 1

            # 右：上 -> 下
            for row in range(top, bottom):
                res.append(matrix[row][right - 1])
            right -= 1

            # 防止单行 / 单列情况下重复访问
            if left >= right or top >= bottom:
                break

            # 下：右 -> 左
            for col in range(right - 1, left - 1, -1):
                res.append(matrix[bottom - 1][col])
            bottom -= 1

            # 左：下 -> 上
            for row in range(bottom - 1, top - 1, -1):
                res.append(matrix[row][left])
            left += 1

        return res
```

这版代码的核心可以压缩成一句话记忆：

```text
走上边 → top++
走右边 → right--
检查边界
走下边 → bottom--
走左边 → left++
```

只要能够熟练解释这五步，这道题在代码面试中基本就掌握了。
