# 盛最多水的容器（Container With Most Water）

## 题目描述

给定一个整数数组 `heights`，其中：

```text
heights[i]
```

表示第 `i` 根竖线的高度。

你可以选择任意两根竖线，与 `x` 轴共同组成一个容器。

请返回这个容器能够盛放的**最大水量**。

### 约束

- `2 <= heights.length <= 100000`
- `0 <= heights[i] <= 10000`

---

# 一、暴力枚举

## 核心思路

最直接的方法是枚举所有可能的两根竖线。

假设我们选择下标：

```text
i < j
```

那么：

- 容器的宽度为：

```text
j - i
```

- 容器的高度由两根竖线中较短的那一根决定：

```text
min(heights[i], heights[j])
```

因此，这两根竖线构成的容器面积为：

```text
min(heights[i], heights[j]) * (j - i)
```

我们枚举所有 `(i, j)`，取其中最大值即可。

---

## 为什么高度取较短的一边？

例如两根竖线高度分别为：

```text
3 和 8
```

即使右边高度为 `8`，水面最多也只能达到高度 `3`。

如果超过 `3`，水会从较短的一侧溢出。

因此容器的有效高度一定是：

```python
min(heights[i], heights[j])
```

---

## 算法步骤

1. 初始化 `res = 0`，记录目前最大的面积。
2. 枚举左边界 `i`。
3. 枚举右边界 `j`，其中 `j > i`。
4. 对每一对 `(i, j)`：
   - 计算高度；
   - 计算宽度；
   - 计算面积；
   - 更新最大值。
5. 返回 `res`。

---

## Python 实现

```python
from typing import List


class Solution:
    def maxArea(self, heights: List[int]) -> int:
        res = 0
        n = len(heights)

        # 枚举左边界
        for i in range(n):
            # 枚举右边界
            for j in range(i + 1, n):
                # 容器高度由较短的竖线决定
                height = min(heights[i], heights[j])

                # 两根竖线之间的水平距离
                width = j - i

                # 当前容器面积
                area = height * width

                # 更新最大面积
                res = max(res, area)

        return res
```

---

## 复杂度分析

设数组长度为 `n`。

- **时间复杂度：`O(n^2)`**
  - 一共有约 `n(n-1)/2` 对竖线需要检查。

- **空间复杂度：`O(1)`**
  - 只使用了常数个额外变量。

---

## 暴力法的问题

本题：

```text
n <= 100000
```

如果使用 `O(n^2)`：

```text
100000² = 10,000,000,000
```

也就是最多需要检查约百亿级别的组合。

因此暴力法通常无法通过时间限制。

这说明我们必须寻找一种方法，避免检查所有可能的竖线组合。

---

# 二、双指针

## 核心思路

双指针是这道题最经典、最推荐的解法。

我们分别设置：

```python
l = 0
r = len(heights) - 1
```

也就是一开始选择：

- 最左边的竖线；
- 最右边的竖线。

这样一开始容器拥有**最大的宽度**。

每一步计算当前面积后，我们让两个指针逐渐向中间靠拢。

关键问题是：

> 应该移动哪一个指针？

答案是：

> **移动高度较短的那一侧。**

这是整道题最核心的贪心思想。

---

# 为什么必须移动较短的一边？

假设当前：

```text
heights[l] < heights[r]
```

当前面积为：

```text
heights[l] * (r - l)
```

因为左边更短，所以当前容器高度由：

```text
heights[l]
```

决定。

现在宽度已经是：

```text
r - l
```

如果我们移动右边较高的指针：

```text
r -= 1
```

那么新容器：

- 宽度一定变小；
- 左边界高度仍然是 `heights[l]`；
- 新的有效高度最多仍然不会超过 `heights[l]`。

也就是说：

```text
新面积 <= heights[l] * 更小的宽度
```

因此不可能比当前面积更大。

---

## 一个具体例子

假设：

```text
heights[l] = 3
heights[r] = 8
宽度 = 10
```

当前面积：

```text
3 * 10 = 30
```

如果移动右边：

```text
新的右边高度 = 100
新的宽度 = 9
```

虽然新的右边高度变成了 `100`，但是左边仍然只有 `3`。

所以有效高度仍然是：

```text
min(3, 100) = 3
```

新面积：

```text
3 * 9 = 27
```

仍然更小。

因此保留较短的一侧没有意义。

要想让面积变大，虽然宽度必然减少，但我们至少需要尝试找到一个**更高的短板**。

所以应该移动：

```text
较短的一侧
```

---

# 双指针为什么不会错过最优答案？

这部分是面试中非常重要的解释。

假设：

```text
heights[l] <= heights[r]
```

当前面积：

```text
A = heights[l] * (r - l)
```

对于所有右边界：

```text
k < r
```

如果仍然固定左边界 `l`，那么：

```text
面积 = min(heights[l], heights[k]) * (k - l)
```

由于：

```text
min(heights[l], heights[k]) <= heights[l]
```

并且：

```text
k - l < r - l
```

所以：

```text
min(heights[l], heights[k]) * (k - l)
<
heights[l] * (r - l)
```

也就是说：

> 在当前 `r` 已经是最远右边界的情况下，固定较短的 `l`，再尝试任何更靠左的右边界都不可能得到更大的面积。

因此可以安全地丢弃当前 `l`：

```python
l += 1
```

这正是双指针算法成立的数学依据。

---

# 算法步骤

1. 初始化：

```python
l = 0
r = len(heights) - 1
res = 0
```

2. 当：

```python
l < r
```

时：

- 计算当前高度：

```python
min(heights[l], heights[r])
```

- 计算当前宽度：

```python
r - l
```

- 计算面积并更新答案。

3. 如果：

```python
heights[l] <= heights[r]
```

说明左边是短板：

```python
l += 1
```

否则：

```python
r -= 1
```

4. 当两个指针相遇后结束。

5. 返回 `res`。

---

# Python 实现

```python
from typing import List


class Solution:
    def maxArea(self, heights: List[int]) -> int:
        # 左右指针分别从数组两端开始
        l = 0
        r = len(heights) - 1

        # 记录最大容器面积
        res = 0

        while l < r:
            # 容器高度由较短的一边决定
            height = min(heights[l], heights[r])

            # 两根竖线之间的距离
            width = r - l

            # 当前容器面积
            area = height * width

            # 更新最大值
            res = max(res, area)

            # 移动较短的一侧
            #
            # 原因：
            # 宽度下一步一定会减少，
            # 只有尝试找到更高的短板，才有机会获得更大的面积。
            if heights[l] <= heights[r]:
                l += 1
            else:
                r -= 1

        return res
```

---

# 复杂度分析

- **时间复杂度：`O(n)`**
  - 左指针只会从左向右移动；
  - 右指针只会从右向左移动；
  - 每个位置最多被访问一次。

- **空间复杂度：`O(1)`**
  - 只使用了几个变量。

---

# 为什么相等时移动哪一边都可以？

代码中使用：

```python
if heights[l] <= heights[r]:
    l += 1
else:
    r -= 1
```

如果：

```text
heights[l] == heights[r]
```

那么移动左边或移动右边都可以。

因为当前两侧一样高。

假设高度都是：

```text
h
```

那么固定其中任意一侧，再缩小宽度，都不可能得到更大的面积，除非后面找到更高的边界。

因此：

```python
l += 1
```

或者：

```python
r -= 1
```

都不会影响算法正确性。

---

# Python 类型说明：`List[int]`

代码中的：

```python
from typing import List
```

以及：

```python
heights: List[int]
```

属于 Python 的**类型标注（Type Hint）**。

它表示：

```text
heights 是一个元素类型为 int 的列表
```

例如：

```python
[1, 8, 6, 2, 5]
```

符合：

```python
List[int]
```

这种类型标注主要用于：

- 提高代码可读性；
- IDE 自动补全；
- 静态类型检查；
- LeetCode 等平台的函数签名。

它不会改变代码本身的运行逻辑。

如果使用 Python 3.9 及以后，也可以写成：

```python
heights: list[int]
```

而不需要导入：

```python
List
```

---

# 一个完整示例

假设输入：

```text
heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]
```

初始：

```text
l = 0
r = 8
```

对应高度：

```text
1 和 7
```

面积：

```text
min(1, 7) * 8 = 8
```

左边更短，因此：

```text
l += 1
```

现在：

```text
l = 1
r = 8
```

高度：

```text
8 和 7
```

面积：

```text
min(8, 7) * 7
= 7 * 7
= 49
```

右边较短，所以：

```text
r -= 1
```

之后继续重复这个过程。

最终最大面积为：

```text
49
```

---

# 常见错误

## 1. 移动了较高的一边

这是最常见的错误。

例如：

```text
heights[l] = 3
heights[r] = 8
```

当前高度受：

```text
3
```

限制。

如果移动高度为 `8` 的右指针：

- 宽度变小；
- 短板仍然是左边的 `3`；
- 面积不可能变大。

因此正确策略是：

```python
l += 1
```

也就是移动较短的一边。

---

## 2. 把本题和 Trapping Rain Water 混淆

这两道题都涉及柱子和水，但问题完全不同。

### Container With Most Water

只选择：

```text
两根竖线
```

形成一个容器。

我们求的是：

```text
某一对边界形成的最大面积
```

公式：

```text
min(left_height, right_height) * width
```

---

### Trapping Rain Water

则要求计算：

```text
整个数组中所有位置一共能够积多少水
```

它涉及每个位置左边和右边的最高柱子。

因此两题虽然视觉上类似，但思路和计算目标完全不同。

---

## 3. 宽度写成 `r - l + 1`

两根竖线的位置分别为：

```text
l
r
```

它们之间的水平距离是：

```python
r - l
```

不是：

```python
r - l + 1
```

例如下标：

```text
0 和 3
```

两条线之间距离为：

```text
3
```

而不是 `4`。

所以正确面积公式是：

```python
area = min(heights[l], heights[r]) * (r - l)
```

---

## 4. 把柱子中间的高度考虑进面积

本题中，容器面积只由：

```text
两边界的较短高度
×
两边界之间的距离
```

决定。

中间柱子的高度：

```text
完全不影响这个容器的面积计算
```

例如：

```text
[8, 100, 100, 100, 7]
```

如果选择首尾两根线：

```text
8 和 7
```

容器面积仍然是：

```text
7 * 4 = 28
```

中间即使有更高的柱子，也不会改变这两个边界形成的容器面积。

---

# 解法对比

| 方法 | 时间复杂度 | 空间复杂度 | 面试推荐程度 |
|---|---:|---:|---|
| 暴力枚举 | `O(n^2)` | `O(1)` | 仅适合理解题意 |
| 双指针 | `O(n)` | `O(1)` | **强烈推荐** |

---

# 面试中的核心理解

这道题最重要的不是记住代码，而是理解：

> 为什么每次可以安全地丢弃较短的那一边？

因为当前面积为：

```text
min(heights[l], heights[r]) * (r - l)
```

假设：

```text
heights[l] <= heights[r]
```

那么当前高度已经被：

```text
heights[l]
```

限制。

如果仍然保留 `l`，无论怎样向左移动 `r`：

- 宽度只会变小；
- 有效高度最多仍然是 `heights[l]`。

所以不可能得到更大的面积。

因此：

```text
固定当前较短边的所有剩余组合都可以直接排除。
```

这就是为什么算法能从：

```text
O(n^2)
```

优化到：

```text
O(n)
```

---

# 推荐记忆模板

面试中可以直接记住下面这个核心模板：

```python
class Solution:
    def maxArea(self, heights: list[int]) -> int:
        l, r = 0, len(heights) - 1
        res = 0

        while l < r:
            res = max(
                res,
                min(heights[l], heights[r]) * (r - l)
            )

            if heights[l] <= heights[r]:
                l += 1
            else:
                r -= 1

        return res
```

真正需要记住的核心只有一句：

```text
宽度一定越来越小，所以必须移动短板，尝试用更高的边界补偿宽度损失。
```
