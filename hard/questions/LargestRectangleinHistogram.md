# 柱状图中最大的矩形

给定一个整数数组 `heights`，其中 `heights[i]` 表示第 `i` 根柱子的高度，每根柱子的宽度均为 `1`。

请返回这些柱子能够组成的**最大矩形面积**。

------

## 前置知识

在解决这道题之前，建议熟悉以下概念：

- **单调栈（Monotonic Stack）**：利用栈维护元素的单调递增或单调递减关系。
- **Next Smaller Element 模式**：寻找某个元素左侧 / 右侧距离它最近的更小元素。
- **分治（Divide and Conquer）**：以区间中的最小值为分界点，将问题拆分为左右子问题。
- **数组遍历**：在遍历过程中维护边界、栈或其他状态。

这道题最核心的观察是：

> 对于任意一个矩形，都一定存在某根柱子是这个矩形中的“最矮柱子”。

因此，我们可以反过来考虑：

> 如果固定第 `i` 根柱子作为矩形中的最矮柱子，那么它最多能够向左、向右延伸多远？

一旦知道这个范围，就可以计算：

```text
面积 = heights[i] × 可延伸宽度
```

不同解法的主要区别，就在于**如何高效找到每根柱子的左右边界**。

------

# 1. 暴力解法

## 思路

对于每一根柱子，我们都假设它是某个矩形中的**最矮柱子**。

然后从当前位置分别向左和向右扩展：

- 只要相邻柱子的高度 `>= 当前柱子的高度`，矩形就仍然可以继续延伸。
- 遇到比当前柱子更矮的柱子时，扩展停止。

这样就可以得到以当前柱子为最低高度时能够形成的最大矩形。

例如：

```text
heights = [2, 1, 5, 6, 2, 3]
```

如果当前考虑高度为 `5` 的柱子：

```text
       █
     █ █
     █ █
     █ █
█    █ █ █ █
█ █  █ █ █ █
```

它右侧的 `6 >= 5`，因此可以继续扩展；

再往右遇到 `2 < 5`，停止。

左侧遇到 `1 < 5`，也停止。

因此：

```text
高度 = 5
宽度 = 2
面积 = 5 × 2 = 10
```

## 算法

对于每个下标 `i`：

1. 令 `height = heights[i]`。
2. 向右寻找第一个高度 `< height` 的柱子。
3. 向左寻找第一个高度 `< height` 的柱子。
4. 两个边界之间，就是当前高度能够覆盖的最大范围。
5. 计算面积并更新答案。

```python
from typing import List


class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        n = len(heights)
        max_area = 0

        for i in range(n):
            height = heights[i]

            # 向右扩展：
            # 只要柱子的高度 >= 当前 height，就仍然可以包含在矩形中
            right = i + 1
            while right < n and heights[right] >= height:
                right += 1

            # 向左扩展
            left = i - 1
            while left >= 0 and heights[left] >= height:
                left -= 1

            # 此时：
            # left 指向左侧第一个严格小于 height 的柱子
            # right 指向右侧第一个严格小于 height 的柱子
            #
            # 因此有效区间为：
            # [left + 1, right - 1]
            width = right - left - 1

            max_area = max(max_area, height * width)

        return max_area
```

### 为什么宽度是 `right - left - 1`？

假设：

```text
left = 1
right = 5
```

意味着真正能够使用的位置是：

```text
2, 3, 4
```

共有：

```text
5 - 1 - 1 = 3
```

个柱子。

因此：

```python
width = right - left - 1
```

这是这道题中非常重要的边界公式。

## 复杂度

- 时间复杂度：**O(n²)**
- 空间复杂度：**O(1)**

最坏情况下，例如：

```text
[1, 2, 3, 4, 5, 6, ...]
```

对于每一根柱子，都可能需要扫描很长的一段区间，因此总体达到 `O(n²)`。

------

# 2. 分治 + 线段树

## 思路

一个区间 `[L, R]` 中的最大矩形，一定属于下面三种情况之一。

首先找到区间中的最矮柱子：

```text
minIndex
```

那么最大矩形可能是：

1. 使用 `heights[minIndex]` 作为最低高度，并横跨整个 `[L, R]`。
2. 完全位于 `minIndex` 左边。
3. 完全位于 `minIndex` 右边。

因此：

```text
answer(L, R)
=
max(
    heights[minIndex] × (R - L + 1),
    answer(L, minIndex - 1),
    answer(minIndex + 1, R)
)
```

这就是一个典型的分治结构。

问题在于：

> 如何快速找到任意区间中的最小柱子？

如果每次都线性扫描寻找最小值，那么复杂度仍然可能退化到 `O(n²)`。

因此，可以使用**线段树（Segment Tree）**维护：

```text
某个区间中最小高度柱子的下标
```

这样每次查询最小值下标只需要：

```text
O(log n)
```

------

## 线段树是什么？

线段树是一种用于处理**区间查询**的数据结构。

常见功能包括：

```text
区间最小值
区间最大值
区间求和
区间 GCD
```

这里我们需要的是：

> Range Minimum Query，简称 RMQ。

但线段树中存储的不是最小高度本身，而是：

```text
最小高度对应的下标
```

例如：

```text
heights = [2, 1, 5, 6, 2, 3]

query(2, 5)
```

对应区间：

```text
[5, 6, 2, 3]
```

最小值为：

```text
2
```

其原数组下标为：

```text
4
```

所以：

```python
query(2, 5) == 4
```

------

## 算法

1. 根据 `heights` 建立线段树。
2. 每个节点存储该区间中最小柱子的下标。
3. 定义递归函数：

```python
solve(L, R)
```

1. 如果：

```text
L > R
```

返回 `0`。

1. 如果：

```text
L == R
```

返回：

```python
heights[L]
```

1. 查询 `[L, R]` 中最小柱子的下标 `minIndex`。
2. 分别计算：
   - 横跨整个区间的面积；
   - 左子区间最大面积；
   - 右子区间最大面积。
3. 返回三者最大值。

```python
class MinIndexSegmentTree:
    def __init__(self, heights):
        self.heights = heights
        self.n = len(heights)

        # 线段树通常需要大约 4n 空间
        self.tree = [0] * (4 * self.n)

        if self.n > 0:
            self._build(1, 0, self.n - 1)

    def _build(self, node, left, right):
        """
        tree[node] 保存区间 [left, right]
        中高度最小的柱子的下标。
        """
        if left == right:
            self.tree[node] = left
            return

        mid = (left + right) // 2

        self._build(node * 2, left, mid)
        self._build(node * 2 + 1, mid + 1, right)

        left_index = self.tree[node * 2]
        right_index = self.tree[node * 2 + 1]

        if self.heights[left_index] <= self.heights[right_index]:
            self.tree[node] = left_index
        else:
            self.tree[node] = right_index

    def query(self, query_left, query_right):
        """
        返回 [query_left, query_right]
        中最小高度柱子的下标。
        """
        return self._query(
            1,
            0,
            self.n - 1,
            query_left,
            query_right
        )

    def _query(self, node, left, right, query_left, query_right):

        # 当前区间和查询区间没有交集
        if right < query_left or left > query_right:
            return -1

        # 当前区间完全包含在查询区间中
        if query_left <= left and right <= query_right:
            return self.tree[node]

        mid = (left + right) // 2

        left_index = self._query(
            node * 2,
            left,
            mid,
            query_left,
            query_right
        )

        right_index = self._query(
            node * 2 + 1,
            mid + 1,
            right,
            query_left,
            query_right
        )

        if left_index == -1:
            return right_index

        if right_index == -1:
            return left_index

        if self.heights[left_index] <= self.heights[right_index]:
            return left_index

        return right_index


class Solution:
    def largestRectangleArea(self, heights):
        if not heights:
            return 0

        segment_tree = MinIndexSegmentTree(heights)

        def solve(left, right):
            if left > right:
                return 0

            if left == right:
                return heights[left]

            # 找到当前区间中最矮柱子的下标
            min_index = segment_tree.query(left, right)

            # 情况 1：
            # 使用最矮柱子横跨整个区间
            whole_area = (
                heights[min_index] *
                (right - left + 1)
            )

            # 情况 2：
            # 最大矩形完全位于左边
            left_area = solve(
                left,
                min_index - 1
            )

            # 情况 3：
            # 最大矩形完全位于右边
            right_area = solve(
                min_index + 1,
                right
            )

            return max(
                whole_area,
                left_area,
                right_area
            )

        return solve(0, len(heights) - 1)
```

## 复杂度

- 构建线段树：`O(n)`
- 每次最小值查询：`O(log n)`
- 分治过程中最多进行 `O(n)` 次查询

因此总体：

- 时间复杂度：**O(n log n)**
- 空间复杂度：**O(n)**

### 面试角度

这个方法非常适合展示：

- 分治思想；
- Range Minimum Query；
- Segment Tree。

但对于本题而言，它并不是最简洁的做法。

面试中最值得掌握的方案通常还是后面的：

> **单调栈 O(n)**。

------

# 3. 单调栈：分别寻找左右最近更矮柱子

## 核心思路

对于每根柱子 `i`，我们希望找到：

```text
left[i]
```

表示：

> 左边距离 `i` 最近的、高度严格小于 `heights[i]` 的柱子下标。

以及：

```text
right[i]
```

表示：

> 右边距离 `i` 最近的、高度严格小于 `heights[i]` 的柱子下标。

那么当前柱子能够作为最低高度覆盖的区间就是：

```text
(left[i], right[i])
```

注意两个边界本身不能包含，因为它们都比当前柱子矮。

所以宽度是：

```python
right[i] - left[i] - 1
```

面积：

```python
heights[i] * (right[i] - left[i] - 1)
```

为了快速得到最近的更矮元素，可以使用**单调递增栈**。

------

## 什么是单调栈？

普通栈只要求：

```text
后进先出
```

单调栈额外维护某种顺序，例如：

```text
栈底 → 栈顶

1, 2, 5, 7
```

对应高度严格递增。

本题中，我们通常在栈里存：

```text
柱子的下标
```

而不是柱子的高度。

因为通过下标：

```python
heights[stack[-1]]
```

既可以获得高度，也可以获得位置。

------

## 寻找左侧最近更矮柱子

遍历：

```python
for i in range(n):
```

如果栈顶柱子的高度：

```python
>= heights[i]
```

说明栈顶不可能是当前柱子的“最近严格更矮元素”，因此将它弹出：

```python
while stack and heights[stack[-1]] >= heights[i]:
    stack.pop()
```

弹出之后：

- 如果栈为空：左侧不存在更矮柱子，记为 `-1`。
- 否则：栈顶就是左侧最近更矮柱子。

------

## 寻找右侧最近更矮柱子

完全类似，只需要从右向左遍历。

如果不存在右侧更矮柱子，我们使用：

```python
n
```

作为虚拟边界。

------

## 代码

```python
from typing import List


class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        n = len(heights)

        # left[i]:
        # i 左边最近的、严格比 heights[i] 小的柱子下标
        left = [-1] * n

        stack = []

        for i in range(n):

            # 删除所有 >= 当前高度的柱子
            # 最终栈顶才会是严格更矮的柱子
            while stack and heights[stack[-1]] >= heights[i]:
                stack.pop()

            if stack:
                left[i] = stack[-1]

            stack.append(i)

        # right[i]:
        # i 右边最近的、严格比 heights[i] 小的柱子下标
        right = [n] * n

        stack = []

        for i in range(n - 1, -1, -1):

            while stack and heights[stack[-1]] >= heights[i]:
                stack.pop()

            if stack:
                right[i] = stack[-1]

            stack.append(i)

        max_area = 0

        for i in range(n):
            # 可使用区间：
            # left[i] + 1 ... right[i] - 1
            width = right[i] - left[i] - 1

            area = heights[i] * width

            max_area = max(max_area, area)

        return max_area
```

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

------

## 为什么 while 循环不会导致 O(n²)？

这是理解单调栈时非常重要的一点。

虽然代码中存在：

```python
for ...
    while ...
```

但不能简单认为这是 `O(n²)`。

每一个下标：

```text
最多入栈一次
最多出栈一次
```

因此所有 `pop()` 加起来最多执行 `n` 次。

整个过程实际上是：

```text
O(n)
```

这种分析方式叫做：

> **摊还分析（Amortized Analysis）**。

------

# 4. 单调栈：一次遍历，保存 `(start_index, height)`

这是一个更紧凑的 `O(n)` 解法。

## 核心思路

栈中不只保存高度，而是保存：

```python
(start_index, height)
```

其中：

```text
start_index
```

表示：

> 这个高度作为矩形最低高度时，最早能够从哪里开始。

假设：

```text
stack = [(1, 2), (3, 5)]
```

其中：

```text
(3, 5)
```

表示高度 `5` 的矩形可以从下标 `3` 开始延伸。

当遇到一个更矮的柱子时，例如：

```text
当前高度 = 2
```

那么高度 `5` 无法继续向右扩展。

于是我们可以立刻计算：

```text
height = 5
start = 3
当前位置 = i

面积 = 5 × (i - 3)
```

更重要的是：

当前更矮的柱子可以继承被弹出柱子的 `start`。

也就是说：

```python
start = popped_start
```

因为如果之前的较高柱子能够从那个位置开始，那么现在这个更矮的柱子当然也可以。

------

## 举例

```text
heights = [2, 1]
```

最开始：

```text
i = 0
h = 2

stack = [(0, 2)]
```

来到：

```text
i = 1
h = 1
```

因为：

```text
2 > 1
```

弹出：

```text
(0, 2)
```

计算：

```text
面积 = 2 × (1 - 0) = 2
```

然后：

```python
start = 0
```

把高度 `1` 放入：

```text
(0, 1)
```

这意味着：

> 高度 1 虽然出现在下标 1，但它实际上可以从下标 0 开始组成矩形。

最终可以得到：

```text
面积 = 1 × 2 = 2
```

------

## 代码

```python
from typing import List


class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        max_area = 0

        # 每个元素：
        # (这个高度最早可以开始的位置, 高度)
        stack = []

        for i, height in enumerate(heights):
            start = i

            # 当前柱子更矮时，
            # 栈顶那些较高柱子无法继续向右延伸
            while stack and stack[-1][1] > height:
                index, previous_height = stack.pop()

                # previous_height 的右边界就是 i - 1
                width = i - index

                max_area = max(
                    max_area,
                    previous_height * width
                )

                # 当前更矮的柱子可以继承之前的起点
                start = index

            stack.append((start, height))

        # 遍历结束之后，栈里的柱子右侧没有更矮柱子，
        # 因此它们都可以一直延伸到数组末尾
        n = len(heights)

        for index, height in stack:
            width = n - index

            max_area = max(
                max_area,
                height * width
            )

        return max_area
```

### `enumerate()` 说明

这里使用了：

```python
for i, height in enumerate(heights):
```

Python 的：

```python
enumerate(iterable)
```

会同时返回：

```text
下标 + 元素
```

例如：

```python
nums = [10, 20, 30]

for i, x in enumerate(nums):
    print(i, x)
```

得到：

```text
0 10
1 20
2 30
```

因此：

```python
for i, height in enumerate(heights):
```

等价于：

```python
for i in range(len(heights)):
    height = heights[i]
```

但通常更符合 Python 的写法习惯。

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

------

# 5. 单调栈：标准最优写法

这是这道题最值得在面试中掌握的版本。

它只需要：

- 一次遍历；
- 一个栈；
- 不需要额外的 `left` / `right` 数组。

## 核心思想

我们维护一个栈：

```python
stack
```

其中保存柱子的：

```text
下标
```

并保证对应高度按照单调递增关系排列。

也就是说：

```python
heights[stack[0]]
<= heights[stack[1]]
<= heights[stack[2]]
...
```

当遇到一个更矮的柱子时：

```python
heights[i] < heights[stack[-1]]
```

说明栈顶柱子：

> 已经不能继续向右扩展。

因此，此时正好可以计算以栈顶柱子作为最低高度的最大矩形。

------

## 最关键的问题：弹栈后宽度怎么算？

假设：

```python
height = heights[stack.pop()]
```

当前下标为：

```text
i
```

当前柱子更矮，所以：

```text
右侧最近更矮柱子 = i
```

弹栈之后，如果栈不为空：

```python
stack[-1]
```

就是左侧最近更矮柱子。

因此有效矩形范围是：

```text
stack[-1] + 1
...
i - 1
```

宽度：

```python
i - stack[-1] - 1
```

所以：

```python
width = i - stack[-1] - 1
```

------

## 如果弹栈后 stack 为空呢？

说明左侧不存在任何更矮柱子。

那么当前高度可以从：

```text
0
```

一直延伸到：

```text
i - 1
```

因此：

```python
width = i
```

综合起来：

```python
width = i if not stack else i - stack[-1] - 1
```

------

## 为什么遍历 `range(n + 1)`？

正常数组下标只有：

```text
0 ... n - 1
```

但这里我们故意再多遍历一次：

```python
for i in range(n + 1):
```

当：

```python
i == n
```

时，我们把它想象成一个：

```text
高度 = 0
```

的虚拟柱子。

这个虚拟柱子的作用就是：

> 强制把栈中剩下的所有柱子全部弹出并计算面积。

因此不需要在主循环之后再单独写一次清栈逻辑。

这个技巧通常称为：

> **Sentinel（哨兵）技巧**。

------

## 推荐代码

```python
from typing import List


class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        n = len(heights)

        max_area = 0

        # 栈中保存柱子的下标
        # 对应的高度保持单调递增
        stack = []

        # 多遍历一次：
        # i == n 时相当于加入高度为 0 的虚拟柱子，
        # 用于强制处理栈中的剩余元素
        for i in range(n + 1):

            # i == n：
            # 相当于当前高度为 0，因此所有剩余柱子都应该弹出
            #
            # heights[stack[-1]] >= heights[i]：
            # 当前柱子比栈顶矮或相等，
            # 栈顶柱子的右边界已经确定
            while stack and (
                i == n or
                heights[stack[-1]] >= heights[i]
            ):
                height = heights[stack.pop()]

                if not stack:
                    # 左侧不存在更矮柱子，
                    # 当前矩形可以从下标 0 开始
                    width = i
                else:
                    # stack[-1] 是左侧最近的更矮柱子
                    #
                    # 有效范围：
                    # stack[-1] + 1 ... i - 1
                    width = i - stack[-1] - 1

                max_area = max(
                    max_area,
                    height * width
                )

            # 注意：
            # 当 i == n 时这个下标本身不会再被访问 heights[i]，
            # 所以即使将 n 放入栈中也不会产生问题。
            stack.append(i)

        return max_area
```

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

虽然存在嵌套的 `while`，但是每个柱子：

```text
最多入栈一次
最多出栈一次
```

所以总操作次数仍然是线性的。

------

# 6. 推荐的更简洁版本：真正加入哨兵 0

上一个版本通过：

```python
i == n
```

模拟虚拟高度 `0`。

在面试中，还可以直接在数组末尾添加一个：

```python
0
```

代码通常更容易解释。

```python
from typing import List


class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        # 添加高度为 0 的哨兵，
        # 确保最后所有柱子都会被弹出并计算
        heights = heights + [0]

        stack = []
        max_area = 0

        for i, height in enumerate(heights):

            while stack and heights[stack[-1]] >= height:
                current_height = heights[stack.pop()]

                # 弹出后：
                # i 是右侧第一个更矮柱子
                #
                # stack[-1]（若存在）
                # 是左侧第一个更矮柱子
                if stack:
                    width = i - stack[-1] - 1
                else:
                    width = i

                max_area = max(
                    max_area,
                    current_height * width
                )

            stack.append(i)

        return max_area
```

这个版本的优点是：

```text
不用在 while 条件中额外判断 i == n
```

因此逻辑上可以统一理解为：

> 每次遇到一个更矮的柱子，就结算所有不能再继续向右延伸的柱子。

------

# 7. 单调栈解法的完整推演

使用经典例子：

```text
heights = [2, 1, 5, 6, 2, 3]
```

使用尾部哨兵：

```text
[2, 1, 5, 6, 2, 3, 0]
```

------

### i = 0，height = 2

栈为空：

```text
push 0
```

得到：

```text
stack = [0]
```

高度：

```text
[2]
```

------

### i = 1，height = 1

当前：

```text
1 < 2
```

弹出 `0`：

```text
height = 2
```

栈变为空，因此：

```text
width = i = 1

area = 2 × 1 = 2
```

然后压入 `1`：

```text
stack = [1]
```

------

### i = 2，height = 5

```text
5 > 1
```

直接压入：

```text
stack = [1, 2]
```

对应高度：

```text
[1, 5]
```

------

### i = 3，height = 6

```text
6 > 5
```

压入：

```text
stack = [1, 2, 3]
```

对应高度：

```text
[1, 5, 6]
```

------

### i = 4，height = 2

首先：

```text
2 < 6
```

弹出高度 `6`。

此时：

```text
stack[-1] = 2
```

所以：

```text
width
= 4 - 2 - 1
= 1

area
= 6 × 1
= 6
```

继续比较：

```text
2 < 5
```

弹出高度 `5`。

此时：

```text
stack[-1] = 1
```

所以：

```text
width
= 4 - 1 - 1
= 2

area
= 5 × 2
= 10
```

当前最大值变为：

```text
10
```

然后：

```text
2 > 1
```

停止弹栈，压入下标 `4`：

```text
stack = [1, 4]
```

------

### i = 5，height = 3

```text
3 > 2
```

压入：

```text
stack = [1, 4, 5]
```

------

### i = 6，height = 0

这是哨兵。

因为：

```text
0
```

比所有正常柱子都矮，所以不断弹栈。

最终所有剩余面积都会被计算。

最终：

```text
max_area = 10
```

------

# 8. `>` 和 `>=` 到底应该怎么选？

这是本题中非常常见的细节问题。原答案也特别提醒了严格不等号和非严格不等号的区别。

例如：

```text
[2, 2, 2]
```

当当前高度与栈顶高度相同时，可以选择：

```python
>
```

也可以选择：

```python
>=
```

但这会影响：

> 相同高度柱子在栈中的保留方式。

------

## 使用 `>=`

```python
while stack and heights[stack[-1]] >= height:
    ...
```

遇到相同高度时，会弹出之前的柱子。

结果是：

> 同一高度通常只保留更靠右的柱子，但旧柱子的面积已经被计算。

这种方式很常见。

------

## 使用 `>`

```python
while stack and heights[stack[-1]] > height:
    ...
```

相同高度会同时留在栈中。

这种写法也可以正确工作，但边界逻辑要保持一致。

------

## 面试中最重要的不是死记符号

真正需要理解的是：

> 你的单调栈究竟是“严格递增”还是“非严格递增”？

以及：

> 相同高度的柱子由哪一个负责覆盖更大的范围？

如果左右边界处理不一致，就容易出现错误。

------

# 9. 常见错误

## 9.1 宽度少减或多减了 1

如果：

```text
left
```

和：

```text
right
```

分别表示左右两侧最近的**更矮柱子**，

那么它们本身不能包含在矩形中。

所以：

```python
width = right - left - 1
```

而不是：

```python
right - left
```

也不是：

```python
right - left + 1
```

这是本题最常见的错误之一。

------

## 9.2 忘记处理栈中剩余元素

例如：

```text
[1, 2, 3, 4]
```

整个数组一直递增。

如果只在遇到更矮柱子时计算面积，那么遍历结束时：

```text
stack
```

里面仍然有很多柱子。

这些柱子都应该能够延伸到数组末尾。

解决方式有两种：

### 方法一

遍历结束后手动清栈。

### 方法二

添加一个：

```text
0
```

高度的哨兵。

第二种通常更加简洁。

------

## 9.3 栈为空时仍然访问 `stack[-1]`

错误写法：

```python
width = i - stack[-1] - 1
```

如果栈已经为空：

```python
stack[-1]
```

就会报错。

正确处理：

```python
if stack:
    width = i - stack[-1] - 1
else:
    width = i
```

栈为空意味着：

> 左侧不存在更矮柱子，因此矩形能够一直延伸到下标 `0`。

------

## 9.4 没有考虑单元素数组

例如：

```text
heights = [5]
```

答案应该是：

```text
5
```

因为：

```text
height = 5
width = 1
area = 5
```

同样，对于：

```text
[5, 5, 5]
```

答案应该是：

```text
5 × 3 = 15
```

好的单调栈实现不应该需要为这些情况单独写特殊判断。

------

# 10. Python 中需要特别理解的语法

## `stack[-1]`

Python 中：

```python
stack[-1]
```

表示列表最后一个元素。

对于栈而言，它就是：

```text
栈顶元素
```

例如：

```python
stack = [1, 4, 7]

stack[-1]
```

结果：

```text
7
```

------

## `list.append()`

```python
stack.append(i)
```

表示将元素添加到列表末尾。

当 `list` 作为栈使用时，它相当于：

```text
push
```

时间复杂度平均为：

```text
O(1)
```

------

## `list.pop()`

```python
index = stack.pop()
```

删除并返回列表最后一个元素。

对于栈而言，相当于：

```text
pop
```

例如：

```python
stack = [1, 4, 7]

x = stack.pop()
```

之后：

```text
x = 7
stack = [1, 4]
```

从 Python `list` 尾部执行 `pop()` 的时间复杂度为：

```text
O(1)
```

这也是 Python 中通常直接使用：

```python
list
```

实现栈，而不需要特殊 `Stack` 类的原因。

------

## `range(n + 1)`

例如：

```python
n = 3
```

那么：

```python
range(n + 1)
```

产生：

```text
0, 1, 2, 3
```

注意：

```text
3
```

已经超出了原数组的合法下标。

在本题中，这是故意设计的：

```text
i == n
```

代表一个虚拟的高度 `0` 柱子，用来清空栈。

------

# 11. 面试时如何解释最优解

可以将思路组织成下面几句话：

> 对每根柱子来说，如果我们知道它左侧和右侧第一个比它矮的柱子，那么就能够确定以它作为最低高度时可以形成的最大矩形。

> 暴力寻找左右边界需要 O(n²)，所以我使用一个单调递增栈保存柱子的下标。

> 当当前柱子比栈顶柱子更矮时，说明栈顶柱子的右边界已经确定，因此我将其弹出并立即计算面积。

> 弹栈之后新的栈顶就是它左侧最近的更矮柱子，所以宽度是 `i - stack[-1] - 1`；如果栈为空，那么它可以一直延伸到下标 0，因此宽度为 `i`。

> 最后加入一个高度为 0 的哨兵，用于强制处理所有仍然留在栈中的柱子。

> 每个元素最多入栈和出栈一次，所以时间复杂度为 O(n)，空间复杂度为 O(n)。

这基本已经覆盖了面试官最关心的几点：

```text
为什么用单调栈
栈维护什么
什么时候弹栈
为什么此时可以计算面积
左右边界是谁
宽度怎么算
为什么是 O(n)
为什么需要哨兵
```

------

# 12. 解法对比

| 解法                     | 核心思想                 | 时间复杂度 | 空间复杂度 | 面试推荐度 |
| ------------------------ | ------------------------ | ---------- | ---------- | ---------- |
| 暴力                     | 每根柱子向左右扩展       | O(n²)      | O(1)       | ★★☆☆☆      |
| 分治 + 线段树            | 用区间最小柱子切分问题   | O(n log n) | O(n)       | ★★★☆☆      |
| 两次单调栈               | 分别计算左右最近更矮元素 | O(n)       | O(n)       | ★★★★☆      |
| `(start, height)` 单调栈 | 弹栈时继承最左起点       | O(n)       | O(n)       | ★★★★☆      |
| 单调栈 + 哨兵            | 弹栈时直接结算矩形       | O(n)       | O(n)       | ★★★★★      |

面试准备时，建议至少掌握：

```text
1. 暴力解法
2. 左右边界单调栈
3. 一次遍历的最优单调栈
```

其中最应该熟练到能够直接写出的，是：

> **单调栈 + 哨兵 O(n) 解法。**

------

# 13. 最终推荐记忆模板

```python
from typing import List


class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        # 添加哨兵，确保最后能够清空栈
        heights = heights + [0]

        stack = []
        max_area = 0

        for i, height in enumerate(heights):

            # 当前高度更小时，
            # 栈顶柱子的右边界已经确定
            while stack and heights[stack[-1]] >= height:
                h = heights[stack.pop()]

                # 弹栈之后：
                # 当前 i 是右侧第一个更矮位置
                # stack[-1] 是左侧第一个更矮位置
                if stack:
                    width = i - stack[-1] - 1
                else:
                    width = i

                max_area = max(
                    max_area,
                    h * width
                )

            stack.append(i)

        return max_area
```

最关键的三个公式 / 结论：

```text
1. 当前更矮柱子出现 → 栈顶柱子的右边界确定

2. 弹栈后的栈顶 → 左侧最近的更矮柱子

3. width = right - left - 1
```

只要真正理解这三点，`Largest Rectangle in Histogram` 的单调栈解法就不需要死记代码。
