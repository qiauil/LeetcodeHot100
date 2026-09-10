# Perfect Squares（完全平方数）

给定一个整数 `n`，返回**和为 `n` 的完全平方数的最少数量**。

**完全平方数（Perfect Square）**是某个整数的平方。例如：

```
1, 4, 9, 16, 25, ...
```

也就是说：

- `1 = 1²`
- `4 = 2²`
- `9 = 3²`
- `16 = 4²`

例如：

```
n = 12

12 = 4 + 4 + 4
```

因此答案为 `3`。

而：

```
n = 13

13 = 9 + 4
```

因此答案为 `2`。

------

## 一、核心思路

这道题本质上可以理解为：

> 对于一个数字 `n`，不断选择一个不超过它的完全平方数，并求出剩余部分最少需要多少个完全平方数。

例如对于：

```
n = 12
```

第一次可以选择：

```
1² = 1  → 剩余 11
2² = 4  → 剩余 8
3² = 9  → 剩余 3
```

所以：

```
f(12) = 1 + min(
    f(11),
    f(8),
    f(3)
)
```

一般来说：

\[ dp[n] = 1 + \min_{i^2 \le n} dp[n-i^2] \]

这就是这道题最重要的**状态转移关系**。

从这个关系出发，可以得到：

1. 暴力递归
2. Top-Down 动态规划（记忆化搜索）
3. Bottom-Up 动态规划
4. BFS
5. 数学方法

面试中最值得掌握的是 **Top-Down DP、Bottom-Up DP 和 BFS**；如果知道数论解法，则属于很好的加分项。

------

# 1. 暴力递归

## 思路

我们希望把 `n` 表示成尽可能少的完全平方数之和。

对于当前的 `target`，尝试减去所有满足：

```
i * i <= target
```

的完全平方数。

然后递归求解剩余部分：

```
dfs(target - i * i)
```

因为当前已经使用了一个完全平方数，所以：

```
1 + dfs(target - i * i)
```

最后从所有选择中取最小值。

例如：

```
dfs(12)

选择 1：
1 + dfs(11)

选择 4：
1 + dfs(8)

选择 9：
1 + dfs(3)
```

因此：

```
dfs(12) = min(
    1 + dfs(11),
    1 + dfs(8),
    1 + dfs(3)
)
```

------

## 算法

1. 定义递归函数 `dfs(target)`。
2. 如果 `target == 0`，返回 `0`。
3. 初始化 `res = target`，因为最坏情况下可以全部使用 `1²`。
4. 枚举所有满足 `i² <= target` 的完全平方数。
5. 递归计算：

```
1 + dfs(target - i * i)
```

1. 取所有可能结果中的最小值。

------

## 代码

```
class Solution:
    def numSquares(self, n: int) -> int:

        def dfs(target):
            # Base Case：
            # target = 0 时，不需要任何完全平方数
            if target == 0:
                return 0

            # 最坏情况：
            # target = 1² + 1² + ... + 1²
            res = target

            # 枚举所有不超过 target 的完全平方数
            i = 1
            while i * i <= target:
                square = i * i

                # 使用当前 square 后，
                # 递归解决剩余部分
                res = min(
                    res,
                    1 + dfs(target - square)
                )

                i += 1

            return res

        return dfs(n)
```

------

## 为什么这种方法很慢？

因为存在大量**重复子问题（Overlapping Subproblems）**。

例如：

```
dfs(12)
├── dfs(11)
│   ├── dfs(10)
│   └── dfs(7)
├── dfs(8)
│   ├── dfs(7)
│   └── dfs(4)
└── dfs(3)
```

可以看到：

```
dfs(7)
```

可能被计算很多次。

随着 `n` 增大，这种重复计算会非常严重。

### 复杂度

设每个状态最多尝试约 `√n` 个完全平方数，递归深度最坏为 `n`。

粗略来看：

- **时间复杂度：指数级**
- **空间复杂度：O(n)**，主要来自递归调用栈

因此，纯递归主要用于帮助理解状态转移，实际面试中通常不会作为最终答案。

------

# 2. 动态规划：Top-Down（记忆化搜索）

## 思路

暴力递归最大的问题是：

> 相同的 `target` 被重复计算。

因此，我们可以使用一个 `memo`：

```
memo[target] = dfs(target)
```

第一次计算 `dfs(target)` 时正常递归。

以后再次遇到相同的 `target`：

```
if target in memo:
    return memo[target]
```

直接返回之前计算好的结果。

这种：

> **递归 + 缓存**

的方式通常称为：

- Memoization
- Top-Down Dynamic Programming
- 记忆化搜索

------

## 算法

1. 创建字典 `memo`。
2. 定义 `dfs(target)`。
3. 如果 `target == 0`，返回 `0`。
4. 如果 `target` 已经存在于 `memo` 中，直接返回。
5. 枚举所有满足 `i² <= target` 的完全平方数。
6. 计算：

```
1 + dfs(target - i * i)
```

1. 保存最小结果：

```
memo[target] = res
```

1. 返回 `dfs(n)`。

------

## 代码

```
class Solution:
    def numSquares(self, n: int) -> int:
        # memo[target]：
        # 和为 target 所需要的最少完全平方数数量
        memo = {}

        def dfs(target):
            # Base Case
            if target == 0:
                return 0

            # 如果已经计算过，直接返回
            if target in memo:
                return memo[target]

            # 最坏情况全部使用 1²
            res = target

            i = 1

            # 枚举所有完全平方数
            while i * i <= target:
                square = i * i

                res = min(
                    res,
                    1 + dfs(target - square)
                )

                i += 1

            # 缓存当前结果
            memo[target] = res

            return res

        return dfs(n)
```

------

## 复杂度

一共有：

```
0, 1, 2, ..., n
```

约 `n` 个不同状态。

对于每个 `target`，最多枚举：

```
1², 2², ..., floor(√target)²
```

即大约 `√n` 个完全平方数。

因此：

- **时间复杂度：O(n√n)**
- **空间复杂度：O(n)**

空间包括：

- `memo`：O(n)
- 最坏情况下递归调用栈：O(n)

> 原题解析中将这一方法的时间复杂度写成了 `O(n*n)`，更准确的复杂度是 **O(n√n)**。

------

# 3. 动态规划：Bottom-Up

这是这道题最经典、最推荐掌握的解法之一。

## 思路

Top-Down 是：

```
从 n 开始
↓
递归寻找更小的问题
```

Bottom-Up 则反过来：

```
先解决 0
↓
解决 1
↓
解决 2
↓
...
↓
最终解决 n
```

定义：

```
dp[target]
```

表示：

> 和为 `target` 所需要的最少完全平方数数量。

Base Case：

```
dp[0] = 0
```

因为组成 `0` 不需要任何数字。

对于每个 `target`，枚举所有：

```
square = s * s
```

只要：

```
square <= target
```

就可以：

```
dp[target] = min(
    dp[target],
    1 + dp[target - square]
)
```

------

## 状态转移方程

核心公式：

\[ dp[target] = \min_{s^2 \le target} \left( 1 + dp[target-s^2] \right) \]

例如：

```
target = 12
```

可以选择：

```
1² = 1
2² = 4
3² = 9
```

因此：

```
dp[12] = min(
    1 + dp[11],
    1 + dp[8],
    1 + dp[3]
)
```

------

## 代码

```
class Solution:
    def numSquares(self, n: int) -> int:
        # dp[target] 表示：
        # 和为 target 所需要的最少完全平方数数量
        #
        # 最坏情况下全部使用 1²，
        # 所以初始化为 n
        dp = [n] * (n + 1)

        # 组成 0 不需要任何完全平方数
        dp[0] = 0

        # 从小到大计算每一个 target
        for target in range(1, n + 1):

            s = 1

            # 只枚举不超过 target 的完全平方数
            while s * s <= target:
                square = s * s

                dp[target] = min(
                    dp[target],
                    1 + dp[target - square]
                )

                s += 1

        return dp[n]
```

------

## Python：`[n] * (n + 1)`

这里：

```
dp = [n] * (n + 1)
```

表示创建一个长度为：

```
n + 1
```

的列表，每个元素初始值都是 `n`。

例如：

```
n = 4

dp = [4] * 5
```

得到：

```
[4, 4, 4, 4, 4]
```

这里用 `n` 作为初始值，是因为最坏情况下任何数字都可以全部由 `1²` 组成：

```
n = 5

5 = 1 + 1 + 1 + 1 + 1
```

所以答案一定不会超过 `n`。

------

## 复杂度

外层循环执行 `n` 次。

对于每个 `target`，内层最多枚举 `√target` 个完全平方数。

因此：

- **时间复杂度：O(n√n)**
- **空间复杂度：O(n)**

这通常是面试中最容易解释、实现也非常稳定的方案。

------

# 4. BFS（广度优先搜索）

## 思路

这道题还可以转换成一个**最短路径问题**。

把数字看作图中的节点：

```
0, 1, 2, 3, ..., n
```

如果两个数字之间相差一个完全平方数，就可以认为存在一条边。

例如从 `0` 出发：

```
0
├── +1  → 1
├── +4  → 4
└── +9  → 9
```

下一层继续：

```
1 → 2, 5, 10, ...
4 → 5, 8, 13, ...
9 → 10, 13, ...
```

每走一条边，就相当于：

> 使用了一个完全平方数。

因此问题变成：

> 从 `0` 到 `n` 最少需要走多少步？

而在**无权图（Unweighted Graph）**中寻找最短路径，BFS 正是标准算法。

------

## BFS 的层级含义

例如：

```
Level 0:
0

Level 1:
1, 4, 9, ...

Level 2:
两个完全平方数可以组成的数字

Level 3:
三个完全平方数可以组成的数字
```

所以：

> BFS 第一次访问到 `n` 时，对应的层数一定就是最少完全平方数数量。

------

## 算法

1. Queue 中首先放入 `0`。
2. 使用 `seen` 记录已经访问过的数字。
3. BFS 一层一层搜索。
4. 对当前数字 `cur`，尝试加入所有完全平方数：

```
cur + 1
cur + 4
cur + 9
...
```

1. 如果：

```
nxt == n
```

直接返回当前层数。

6. 否则，如果 `nxt` 没访问过，就加入队列。

------

## 代码

```
from collections import deque


class Solution:
    def numSquares(self, n: int) -> int:
        # BFS 队列
        q = deque([0])

        # 防止重复访问同一个数字
        seen = {0}

        # res 表示当前使用了多少个完全平方数
        res = 0

        while q:
            # 进入 BFS 的下一层
            res += 1

            # 当前层的节点数量
            level_size = len(q)

            for _ in range(level_size):
                cur = q.popleft()

                s = 1

                # 尝试加上所有可能的完全平方数
                while cur + s * s <= n:
                    nxt = cur + s * s

                    # BFS 第一次到达 n，
                    # 一定对应最短路径
                    if nxt == n:
                        return res

                    # 没访问过才加入队列
                    if nxt not in seen:
                        seen.add(nxt)
                        q.append(nxt)

                    s += 1

        return res
```

------

## Python：`collections.deque`

BFS 中通常不要使用普通 `list`：

```
q.pop(0)
```

因为从 Python `list` 的开头删除元素需要移动后面的元素，时间复杂度为：

```
O(n)
```

Python 提供了：

```
from collections import deque
```

`deque` 是 **double-ended queue（双端队列）**。

常用操作：

```
q.append(x)      # 从右侧加入
q.appendleft(x)  # 从左侧加入

q.pop()          # 从右侧删除
q.popleft()      # 从左侧删除
```

其中：

```
append()
popleft()
```

都可以认为是 **O(1)**。

所以 BFS 中通常使用：

```
deque
```

作为队列。

------

## 为什么需要 `seen`？

例如：

```
0 → 1 → 5
0 → 4 → 5
```

数字 `5` 可以通过不同路径到达。

如果没有：

```
seen
```

那么 `5` 会被重复加入 Queue，产生大量重复计算。

因此：

```
seen = set()
```

用来保证每个状态最多进入队列一次。

------

## 复杂度

最多访问 `n` 个不同状态。

每个状态最多尝试约 `√n` 个完全平方数。

因此：

- **时间复杂度：O(n√n)**
- **空间复杂度：O(n)**

------

# 5. 数学方法

这是效率最高、但需要额外数论知识的方法。

## 核心数学结论

首先需要知道两个重要定理。

### 拉格朗日四平方定理

**Lagrange's Four Square Theorem：**

> 每一个正整数都可以表示成最多四个整数平方之和。

因此这道题的答案一定属于：

```
1
2
3
4
```

也就是说：

```
numSquares(n)
```

永远不会超过 `4`。

------

### 勒让德三平方定理

进一步有：

> 一个正整数不能表示为三个整数平方之和，当且仅当它可以写成：

\[ 4^k(8m+7) \]

因此，如果不断把 `n` 中的因子 `4` 除掉：

```
while n % 4 == 0:
    n //= 4
```

最终得到：

```
n % 8 == 7
```

那么答案一定是：

```
4
```

------

## 如何判断答案是 1、2、3、4？

### 情况一：答案是 1

如果：

```
n = a²
```

那么答案就是：

```
1
```

例如：

```
16 = 4²
```

------

### 情况二：答案是 4

如果：

\[ n = 4^k(8m+7) \]

答案一定是：

```
4
```

------

### 情况三：答案是 2

尝试寻找：

\[ n = a^2 + b^2 \]

枚举 `a`：

```
a = 1, 2, ..., √n
```

然后判断：

```
n - a * a
```

是不是完全平方数。

如果是，答案就是 `2`。

------

### 情况四：剩下只能是 3

因为四平方定理告诉我们答案最多为 `4`。

我们已经排除了：

```
1
2
4
```

所以剩下的答案必然是：

```
3
```

------

## 代码

```
import math


class Solution:
    def numSquares(self, n: int) -> int:

        # 判断一个数字是不是完全平方数
        def is_square(num):
            root = math.isqrt(num)
            return root * root == num

        # 情况 1：
        # n 本身就是完全平方数
        if is_square(n):
            return 1

        # 保存原始 n
        reduced = n

        # 去掉所有因子 4
        while reduced % 4 == 0:
            reduced //= 4

        # Legendre 三平方定理：
        # 如果 reduced ≡ 7 (mod 8)，
        # 则必须使用 4 个平方数
        if reduced % 8 == 7:
            return 4

        # 检查是否可以表示成两个平方数之和
        i = 1

        while i * i <= n:
            remainder = n - i * i

            if is_square(remainder):
                return 2

            i += 1

        # 已经排除了 1、2、4
        # 根据四平方定理，答案只能是 3
        return 3
```

------

## Python：`math.isqrt()`

原答案使用：

```
int(math.sqrt(num))
```

也可以工作，但在这里更推荐：

```
math.isqrt(num)
```

`math.isqrt(num)` 会返回：

\[ \lfloor\sqrt{num}\rfloor \]

而且整个计算过程使用整数运算。

例如：

```
import math

math.isqrt(16)  # 4
math.isqrt(17)  # 4
math.isqrt(24)  # 4
math.isqrt(25)  # 5
```

判断完全平方数：

```
root = math.isqrt(num)

if root * root == num:
    # num 是完全平方数
```

相比浮点数版本：

```
int(math.sqrt(num))
```

`math.isqrt()` 不涉及浮点精度问题，因此对于整数平方判断更加合适。

------

## 复杂度

去除因子 `4`：

```
O(log n)
```

判断一个平方数：

```
O(1)
```

检查两个平方数之和最多枚举：

```
√n
```

次。

因此整体：

- **时间复杂度：O(√n)**
- **空间复杂度：O(1)**

> 原题解析中这里写成了 `O(n)`，更准确地说，该实现的时间复杂度是 **O(√n)**，并不是真正意义上的常数时间。

------

# 各种方法对比

| 方法         | 时间复杂度 | 空间复杂度 | 面试推荐度 |
| ------------ | ---------- | ---------- | ---------- |
| 暴力递归     | 指数级     | O(n)       | ⭐          |
| Top-Down DP  | O(n√n)     | O(n)       | ⭐⭐⭐⭐       |
| Bottom-Up DP | O(n√n)     | O(n)       | ⭐⭐⭐⭐⭐      |
| BFS          | O(n√n)     | O(n)       | ⭐⭐⭐⭐       |
| 数学         | O(√n)      | O(1)       | ⭐⭐⭐⭐⭐      |

如果是在代码面试中，一个比较自然的思考过程是：

```
暴力递归
    ↓
发现重复子问题
    ↓
Memoization / Top-Down DP
    ↓
Bottom-Up DP
```

如果面试官继续问：

> Can you do better?

可以进一步讨论：

```
BFS → 将问题理解为最短路径

数学 → Lagrange + Legendre 定理
```

------

# 常见错误

## 1. 没有发现重复子问题

直接使用：

```
dfs(target)
```

会导致大量重复计算。

例如：

```
dfs(5)
```

可能从不同递归路径被重复调用很多次。

解决方式是加入：

```
memo
```

将递归转化为记忆化搜索。

------

## 2. DP 的 Base Case 写错

最重要的初始化是：

```
dp[0] = 0
```

它表示：

> 和为 `0` 时，需要 `0` 个完全平方数。

这个状态直接决定：

```
dp[1] = 1 + dp[0] = 1
dp[4] = 1 + dp[0] = 1
dp[9] = 1 + dp[0] = 1
```

如果 `dp[0]` 初始化错误，后面的状态都会受到影响。

------

## 3. 忘记只枚举完全平方数

应该枚举：

```
1², 2², 3², ...
```

也就是：

```
i = 1

while i * i <= target:
    square = i * i
    ...
    i += 1
```

而不是把：

```
1, 2, 3, 4, 5, ...
```

全部当作候选值。

------

## 4. 复杂度误判为 O(n²)

无论 DP 还是 BFS，每个状态并不是枚举 `n` 个数字，而只需要枚举：

```
1², 2², ..., floor(√n)²
```

候选数量大约是：

\[ \sqrt n \]

所以：

```
n 个状态 × √n 个候选
```

得到：

\[ O(n\sqrt n) \]

这是面试中很值得注意的复杂度分析。

------

# 面试总结

这道题最核心的 DP 定义可以记住：

```
dp[i] = 和为 i 所需要的最少完全平方数数量
```

状态转移：

```
dp[i] = min(
    dp[i],
    1 + dp[i - square]
)
```

其中：

```
square = 1², 2², 3², ...
square <= i
```

Base Case：

```
dp[0] = 0
```

因此 Bottom-Up DP 的核心模板实际上只有：

```
dp = [n] * (n + 1)
dp[0] = 0

for target in range(1, n + 1):
    s = 1

    while s * s <= target:
        square = s * s

        dp[target] = min(
            dp[target],
            1 + dp[target - square]
        )

        s += 1

return dp[n]
```

如果从图论角度理解，则是：

> **每加入一个完全平方数相当于走一步，问题转化为从 `0` 到 `n` 的无权最短路径，因此可以使用 BFS。**

如果从数学角度理解，则利用：

> **任何正整数最多只需要 4 个完全平方数，并可以通过勒让德三平方定理判断答案是否必须为 4。**

所以这道题实际上很好地串联了三类非常重要的面试知识：

```
递归 / DP
      ↓
状态转移与重复子问题

BFS
      ↓
无权图最短路径

数论
      ↓
Lagrange 四平方定理
Legendre 三平方定理
```