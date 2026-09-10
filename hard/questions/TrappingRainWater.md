# 接雨水（Trapping Rain Water）

给定一个由非负整数组成的数组 `height`，它表示一张柱状的高度图。`height[i]` 表示第 `i` 根柱子的高度，每根柱子的宽度均为 `1`。

请返回这些柱子之间总共能够接住多少单位的雨水。

------

## 前置知识

在解决这道题之前，建议熟悉以下知识：

- **数组（Array）**：数组遍历与元素访问。
- **前缀 / 后缀数组（Prefix / Suffix Array）**：从左右两个方向预先计算某个位置之前或之后的最大值。
- **双指针（Two Pointers）**：使用左右两个指针，并根据条件逐渐向中间移动。
- **单调栈（Monotonic Stack）**：利用栈维护具有单调性的元素或下标，从而高效寻找左右边界。

------

# 核心思路

对于任意位置 `i`，它上方能存多少水，只取决于：

- `i` 左侧最高的柱子；
- `i` 右侧最高的柱子；
- `height[i]` 自身的高度。

假设：

```
leftMax  = i 左侧（包含 i）最高的柱子
rightMax = i 右侧（包含 i）最高的柱子
```

那么位置 `i` 的最高水位由两边较矮的一侧决定：

```
waterLevel = min(leftMax, rightMax)
```

因此：

```
water[i] = min(leftMax, rightMax) - height[i]
```

例如：

```
leftMax = 5
rightMax = 3
height[i] = 1

        |
        |
|~~~~~~~|
|~~~|~~~|
|~~~|~~~|
|___|___|
5   1   3
```

虽然左边的墙高达 `5`，但右边只有 `3`，所以水位最高只能达到 `3`：

```
water = min(5, 3) - 1
      = 2
```

**这条公式是整道题最核心的结论。**

下面四种方法，本质上都在解决同一个问题：

> 如何高效地知道每个位置左右两边的最大高度？

------

# 1. 暴力解法（Brute Force）

## 思路

最直接的方法是：

对于每一个位置 `i`，分别向左和向右扫描，找到：

```
leftMax
rightMax
```

然后计算：

```
min(leftMax, rightMax) - height[i]
```

问题在于：**每处理一个位置，都需要重新扫描左右两侧。**

因此存在大量重复计算。

------

## 算法步骤

对于每个位置 `i`：

1. 从 `0` 到 `i` 找到 `leftMax`。
2. 从 `i` 到 `n - 1` 找到 `rightMax`。
3. 计算当前位置的雨水：

```
min(leftMax, rightMax) - height[i]
```

1. 累加到答案中。

------

## Python 实现

```
from typing import List


class Solution:
    def trap(self, height: List[int]) -> int:
        if not height:
            return 0

        n = len(height)
        res = 0

        for i in range(n):
            # 初始化为当前柱子的高度
            leftMax = height[i]
            rightMax = height[i]

            # 找到当前位置左侧（包含自己）的最高柱子
            for j in range(i):
                leftMax = max(leftMax, height[j])

            # 找到当前位置右侧（包含自己）的最高柱子
            for j in range(i + 1, n):
                rightMax = max(rightMax, height[j])

            # 两侧较矮的柱子决定最高水位
            water = min(leftMax, rightMax) - height[i]

            res += water

        return res
```

------

## 复杂度

- 时间复杂度：**O(n²)**
- 空间复杂度：**O(1)**

外层循环执行 `n` 次，而每次都可能再扫描接近 `n` 个元素，因此整体为 `O(n²)`。

------

# 2. 前缀最大值 + 后缀最大值

## 思路

暴力解法慢的原因是：

> 我们一直在重复计算某个位置左边和右边的最大值。

因此可以提前把它们全部算出来。

创建两个数组：

```
leftMax[i]  = height[0...i] 中的最大值
rightMax[i] = height[i...n-1] 中的最大值
```

之后，对于任意位置 `i`：

```
water[i] = min(leftMax[i], rightMax[i]) - height[i]
```

就可以直接 `O(1)` 得到结果。

这是一种非常经典的：

> **预计算（Precomputation）/ 空间换时间**

思想。

------

## 如何构造 `leftMax`

假设：

```
height = [3, 1, 2, 4]
```

那么：

```
leftMax = [3, 3, 3, 4]
```

因为：

```
leftMax[i] = max(leftMax[i - 1], height[i])
```

------

## 如何构造 `rightMax`

同样：

```
height   = [3, 1, 2, 4]
rightMax = [4, 4, 4, 4]
```

从右向左计算：

```
rightMax[i] = max(rightMax[i + 1], height[i])
```

------

## 算法步骤

1. 创建长度为 `n` 的 `leftMax`。
2. 从左向右计算每个位置左侧最大高度。
3. 创建长度为 `n` 的 `rightMax`。
4. 从右向左计算每个位置右侧最大高度。
5. 遍历数组，根据：

```
min(leftMax[i], rightMax[i]) - height[i]
```

计算雨水。

6. 返回总和。

------

## Python 实现

```
from typing import List


class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height)

        if n == 0:
            return 0

        # leftMax[i]：从 0 到 i 的最高柱子
        leftMax = [0] * n

        # rightMax[i]：从 i 到 n-1 的最高柱子
        rightMax = [0] * n

        # 构造前缀最大值
        leftMax[0] = height[0]

        for i in range(1, n):
            leftMax[i] = max(
                leftMax[i - 1],
                height[i]
            )

        # 构造后缀最大值
        rightMax[n - 1] = height[n - 1]

        for i in range(n - 2, -1, -1):
            rightMax[i] = max(
                rightMax[i + 1],
                height[i]
            )

        # 根据左右最高柱子计算每个位置的雨水
        res = 0

        for i in range(n):
            water_level = min(
                leftMax[i],
                rightMax[i]
            )

            res += water_level - height[i]

        return res
```

------

## Python：`range(n - 2, -1, -1)`

这里值得特别注意：

```
for i in range(n - 2, -1, -1):
```

Python 的 `range` 形式为：

```
range(start, stop, step)
```

其中 `stop` **不包含在范围中**。

因此：

```
range(n - 2, -1, -1)
```

表示：

```
n-2, n-3, ..., 2, 1, 0
```

之所以写 `-1` 而不是 `0`，就是为了让索引 `0` 也被遍历到。

------

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

虽然进行了三次遍历：

```
构造 leftMax   → O(n)
构造 rightMax  → O(n)
计算结果        → O(n)
```

但：

```
O(n) + O(n) + O(n) = O(n)
```

------

# 3. 单调栈（Monotonic Stack）

## 思路

前两种方法是：

> **逐列计算雨水。**

单调栈则换了一个角度：

> **逐层计算一段凹槽中的雨水。**

当我们遇到一个比栈顶柱子更高的柱子时，说明可能出现：

```
左墙   凹槽   右墙
 |             |
 |      ~~~~~~~|
 |      ~~~~~~~|
 |      |      |
```

此时：

- 当前柱子 `i`：**右墙**
- 被弹出的柱子：**凹槽底部**
- 弹出之后新的栈顶：**左墙**

有了：

```
左墙 + 底部 + 右墙
```

就可以计算这一层能够存多少水。

------

## 为什么使用“单调栈”？

栈中保存的是**柱子的下标**，并且柱子的高度大体保持：

```
从栈底到栈顶单调递减
```

当出现更高的柱子时：

```
height[i] >= height[stack[-1]]
```

说明当前柱子可以作为右边界，于是不断弹出较矮的柱子并计算积水。

------

## 水量如何计算？

假设：

```
left   = 左墙高度
mid    = 底部高度
right  = 右墙高度
```

水的高度：

```
h = min(left, right) - mid
```

水的宽度：

```
w = right_index - left_index - 1
```

因此：

```
water = h * w
```

这里的 `-1` 是因为左右两根墙本身不能算作中间储水区域。

------

## Python 实现

```
from typing import List


class Solution:
    def trap(self, height: List[int]) -> int:
        if not height:
            return 0

        # 保存柱子的下标，而不是高度
        stack = []

        res = 0

        for i in range(len(height)):

            # 当前柱子足够高时，
            # 可以尝试把之前较矮的柱子作为“凹槽底部”
            while stack and height[i] >= height[stack[-1]]:

                # 弹出的柱子是凹槽底部
                bottom_index = stack.pop()
                bottom_height = height[bottom_index]

                # 如果栈空了，说明没有左墙
                # 无法形成封闭区域
                if not stack:
                    break

                # 新的栈顶就是左墙
                left_index = stack[-1]

                left_height = height[left_index]
                right_height = height[i]

                # 水的高度由左右两墙中较矮的一侧决定
                water_height = (
                    min(left_height, right_height)
                    - bottom_height
                )

                # 左右墙之间的水平距离
                water_width = i - left_index - 1

                # 当前这一层的水量
                res += water_height * water_width

            # 保存当前柱子的下标
            stack.append(i)

        return res
```

------

## 为什么栈里存下标而不是高度？

因为计算宽度时需要：

```
i - left_index - 1
```

如果只保存：

```
height[i]
```

我们虽然知道柱子有多高，却不知道柱子在哪里，也就无法计算水平距离。

这是单调栈题目中非常常见的设计：

> **如果后续计算既需要值，又需要位置，通常在栈中保存 index。**

通过：

```
height[index]
```

随时可以获得对应的高度。

------

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

虽然代码中存在：

```
for ...
    while ...
```

但并不是 `O(n²)`。

原因是每根柱子：

- 最多入栈一次；
- 最多出栈一次。

因此所有 `push + pop` 操作加起来最多是 `O(n)`。

这是分析单调栈复杂度时非常重要的思想。

------

# 4. 双指针（Two Pointers）—— 推荐掌握

## 思路

这是这道题非常经典、同时也是空间复杂度最优的解法。

前缀 / 后缀数组方法需要：

```
leftMax[]
rightMax[]
```

但实际上，我们并不需要同时保存所有位置的最大值。

只需要维护：

```
leftMax  = 当前左侧见过的最高柱子
rightMax = 当前右侧见过的最高柱子
```

然后使用两个指针：

```
l →               ← r
```

不断向中间移动。

------

## 最关键的问题：为什么可以只处理较矮的一边？

假设当前：

```
leftMax < rightMax
```

那么对于左边当前位置来说，虽然我们可能还不知道右侧**真正的最高柱子**是多少，但这并不重要。

因为我们已经知道：

```
右边至少存在一根高度为 rightMax 的柱子
```

并且：

```
rightMax > leftMax
```

所以左侧的水位一定受：

```
leftMax
```

限制。

也就是说，此时左边的水量已经可以确定：

```
leftMax - height[l]
```

不需要知道右边之后还有没有更高的柱子。

反过来，如果：

```
rightMax <= leftMax
```

那么右边的水量已经可以确定。

------

## 双指针的核心不变量

可以把算法记成一句话：

> **哪一侧的最大高度更小，就处理哪一侧。**

因为较矮的一侧决定了当前能够确定的水位。

```
leftMax < rightMax
    ↓
处理左边

leftMax >= rightMax
    ↓
处理右边
```

------

## 算法步骤

初始化：

```
l = 0
r = n - 1

leftMax = height[l]
rightMax = height[r]
```

然后不断比较：

```
leftMax < rightMax ?
```

如果成立：

```
移动 l
更新 leftMax
计算左侧雨水
```

否则：

```
移动 r
更新 rightMax
计算右侧雨水
```

直到两个指针相遇。

------

## Python 实现

```
from typing import List


class Solution:
    def trap(self, height: List[int]) -> int:
        if not height:
            return 0

        # 左右两个指针
        l = 0
        r = len(height) - 1

        # 当前左右两边见过的最高柱子
        leftMax = height[l]
        rightMax = height[r]

        res = 0

        while l < r:

            # 左边的最高墙更矮
            # → 左边决定当前水位
            if leftMax < rightMax:
                l += 1

                # 更新左侧最大高度
                leftMax = max(leftMax, height[l])

                # 当前柱子能够接住的水
                res += leftMax - height[l]

            else:
                # 右边的最高墙更矮（或相等）
                # → 右边决定当前水位
                r -= 1

                # 更新右侧最大高度
                rightMax = max(rightMax, height[r])

                # 当前柱子能够接住的水
                res += rightMax - height[r]

        return res
```

------

## 为什么这里不会出现负数？

注意代码的顺序：

```
leftMax = max(leftMax, height[l])
res += leftMax - height[l]
```

更新之后必然：

```
leftMax >= height[l]
```

所以：

```
leftMax - height[l] >= 0
```

右侧同理。

因此这里不需要额外写：

```
max(0, ...)
```

------

# 四种方法对比

| 方法        | 时间复杂度 | 空间复杂度 | 核心思想                             |
| ----------- | ---------- | ---------- | ------------------------------------ |
| 暴力        | O(n²)      | O(1)       | 每个位置重新寻找左右最大值           |
| 前缀 + 后缀 | O(n)       | O(n)       | 提前保存每个位置左右最大值           |
| 单调栈      | O(n)       | O(n)       | 找到左墙、凹槽底部和右墙，逐层计算   |
| **双指针**  | **O(n)**   | **O(1)**   | 只维护当前左右最大值，处理较矮的一侧 |

面试中建议至少熟练掌握：

**前缀 / 后缀数组 → 双指针**

因为这两个方法之间存在非常自然的优化关系：

```
暴力
 ↓
发现重复计算 leftMax / rightMax
 ↓
前缀 + 后缀数组
 ↓
发现不需要保存所有历史最大值
 ↓
双指针
 ↓
O(n) 时间 + O(1) 空间
```

单调栈则代表另一条思考路线，也非常值得掌握，因为它可以迁移到大量类似问题。

------

# 常见错误

## 1. 错误计算边界位置

最左边和最右边的柱子不可能储水。

因为储水至少需要：

```
左墙 + 右墙
```

边界位置缺少其中一侧。

不过在正确的公式或双指针实现中，通常不需要专门跳过边界，因为它们自然会贡献 `0`。

------

## 2. 使用当前柱子的高度，而不是左右最大高度

错误：

```
water = min(height[l], height[r]) - height[i]
```

这里的 `height[l]` 和 `height[r]` 不一定是两边最高的墙。

真正决定水位的是：

```
water = min(leftMax, rightMax) - height[i]
```

所以要区分：

```
height[l]  → 当前柱子的高度

leftMax    → 到目前为止左侧最高柱子的高度
```

------

## 3. 出现负数雨水

一般公式最好理解为：

```
water = max(
    0,
    min(leftMax, rightMax) - height[i]
)
```

因为雨水不可能是负数。

不过如果 `leftMax` 和 `rightMax` 的定义**包含当前位置 `i`**，那么：

```
leftMax >= height[i]
rightMax >= height[i]
```

因此：

```
min(leftMax, rightMax) - height[i] >= 0
```

这种实现中实际上不需要额外的 `max(0, ...)`。

------

## 4. 双指针移动了错误的一侧

双指针最重要的规则：

```
leftMax < rightMax
→ 移动左指针

否则
→ 移动右指针
```

不要简单理解成：

```
height[l] < height[r]
```

对于当前这份代码来说，判断依据是：

```
leftMax < rightMax
```

因为真正决定水位的是**两边目前已知的最大高度**。

------

# 面试理解：从公式推导双指针

如果面试时需要解释，可以按照下面的逻辑推导。

首先，任意位置：

```
water[i]
= min(leftMax[i], rightMax[i]) - height[i]
```

直接计算所有 `leftMax[i]` 和 `rightMax[i]`：

```
时间 O(n)
空间 O(n)
```

接下来考虑是否能减少空间。

假设当前：

```
leftMax < rightMax
```

那么左侧当前位置的水位一定受 `leftMax` 限制。

因为右侧已经至少存在一堵高度为：

```
rightMax
```

的墙，而且：

```
rightMax > leftMax
```

所以不管右侧尚未探索的区域中是否存在更高的墙：

```
min(leftMax, 右侧真正最大值)
```

都一定等于：

```
leftMax
```

因此左边当前的位置已经可以安全计算。

右侧同理。

最终，我们就不再需要：

```
leftMax[]
rightMax[]
```

两个数组，而只需要：

```
leftMax
rightMax
```

两个变量。

于是空间复杂度从：

```
O(n)
```

优化到：

```
O(1)
```

而时间复杂度仍然保持：

```
O(n)
```

这也是这道题最值得掌握的优化过程：

> **先找到正确公式，再通过预计算消除重复计算，最后观察哪些历史信息实际上不需要完整保存，从而进一步优化空间。**