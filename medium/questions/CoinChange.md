# 零钱兑换（Coin Change）

## 题目描述

给定一个整数数组 `coins`，其中每个元素表示一种硬币的面额；再给定一个整数 `amount`，表示目标金额。

请返回凑出目标金额所需要的最少硬币数量。如果无法恰好凑出该金额，则返回 `-1`。

每种硬币都可以使用无限次。

### 示例

```
输入：coins = [1, 2, 5], amount = 11
输出：3
解释：11 = 5 + 5 + 1
输入：coins = [2], amount = 3
输出：-1
输入：coins = [1], amount = 0
输出：0
```

------

## 核心分析

对于任意金额 `a`，可以尝试选择一枚面额为 `coin` 的硬币，然后继续解决剩余金额：

```
a - coin
```

因此状态转移关系为：

```
凑出金额 a 的最少硬币数
= min(凑出金额 a - coin 的最少硬币数 + 1)
```

也就是：

\[ dp[a] = \min_{coin \in coins}(dp[a-coin] + 1) \]

其中只有当 `a - coin >= 0` 时，这个转移才有效。

------

# 方法一：自底向上的动态规划（推荐）

## 思路

定义：

```
dp[a] = 凑出金额 a 所需要的最少硬币数量
```

从较小的金额开始计算，逐步得到更大金额的答案。

如果已经知道凑出 `a - coin` 最少需要多少枚硬币，那么再添加一枚面额为 `coin` 的硬币，就可以凑出金额 `a`：

```
dp[a] = min(dp[a], dp[a - coin] + 1)
```

### 初始状态

```
dp[0] = 0
```

凑出金额 `0` 不需要使用任何硬币。

其他状态暂时设置为 `amount + 1`，表示“当前无法凑出”。

为什么可以使用 `amount + 1`？

如果存在面额为 `1` 的硬币，那么凑出 `amount` 最多需要 `amount` 枚硬币。因此，任何大于 `amount` 的结果都可以表示“不可能”。

即使没有面额为 `1` 的硬币，这个值仍然可以安全地作为无效标记。

## 算法步骤

1. 创建长度为 `amount + 1` 的数组 `dp`。
2. 将所有元素初始化为 `amount + 1`。
3. 设置 `dp[0] = 0`。
4. 依次计算金额 `1` 到 `amount`：
   - 枚举每一种硬币；
   - 如果当前金额不小于硬币面额，则更新当前状态。
5. 如果 `dp[amount]` 仍然是初始值，返回 `-1`；否则返回 `dp[amount]`。

## Python 代码

```
from typing import List


class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        # dp[a] 表示凑出金额 a 所需要的最少硬币数量。
        #
        # amount + 1 是一个不可能成为有效答案的值，
        # 因此用它表示“当前金额还无法凑出”。
        dp = [amount + 1] * (amount + 1)

        # 凑出金额 0 不需要任何硬币。
        dp[0] = 0

        # 从小金额开始计算，逐步得到更大金额的答案。
        for current_amount in range(1, amount + 1):
            for coin in coins:
                # 只有剩余金额非负时，才能选择这枚硬币。
                if current_amount >= coin:
                    dp[current_amount] = min(
                        dp[current_amount],
                        dp[current_amount - coin] + 1
                    )

        # 如果状态没有被更新，说明无法凑出目标金额。
        return -1 if dp[amount] == amount + 1 else dp[amount]
```

## 示例推演

对于：

```
coins = [1, 2, 5]
amount = 5
```

动态规划数组的变化如下：

```
dp[0] = 0
dp[1] = 1                  # 1
dp[2] = 1                  # 2
dp[3] = 2                  # 1 + 2
dp[4] = 2                  # 2 + 2
dp[5] = 1                  # 5
```

所以答案为 `1`。

## 复杂度分析

设：

- `n` 为硬币种类数量，即 `len(coins)`；
- `t` 为目标金额，即 `amount`。

时间复杂度：

\[ O(n \times t) \]

需要计算 `t` 个状态，每个状态都要检查 `n` 种硬币。

空间复杂度：

\[ O(t) \]

需要一个长度为 `amount + 1` 的动态规划数组。

------

# 方法二：自顶向下的动态规划（递归 + 记忆化搜索）

## 思路

先使用递归函数回答下面的问题：

```
dfs(a) = 凑出金额 a 所需要的最少硬币数量
```

对于每一种硬币，都尝试：

```
1 + dfs(a - coin)
```

单纯递归会反复计算相同的金额。例如，多个递归分支都可能需要计算 `dfs(6)`。

因此，可以使用字典 `memo` 保存已经计算过的结果：

```
memo[a] = dfs(a) 的计算结果
```

当再次遇到同一个金额时，直接返回缓存结果。

## Python 代码

```
from typing import List


class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        # memo[a] 保存凑出金额 a 所需要的最少硬币数量。
        memo = {}

        # 使用 amount + 1 表示当前金额无法凑出。
        impossible = amount + 1

        def dfs(remaining: int) -> int:
            # 剩余金额为 0，说明已经成功凑出目标金额。
            if remaining == 0:
                return 0

            # 如果已经计算过，直接返回缓存结果。
            if remaining in memo:
                return memo[remaining]

            min_coins = impossible

            for coin in coins:
                if remaining >= coin:
                    # 使用当前硬币后，继续解决剩余金额。
                    subproblem = dfs(remaining - coin)

                    # 如果剩余金额可以凑出，则尝试更新答案。
                    if subproblem != impossible:
                        min_coins = min(min_coins, subproblem + 1)

            # 保存结果，无法凑出时也需要保存，
            # 避免以后再次计算这个失败状态。
            memo[remaining] = min_coins
            return min_coins

        result = dfs(amount)
        return -1 if result == impossible else result
```

## 复杂度分析

时间复杂度：

\[ O(n \times t) \]

金额 `0` 到 `amount` 中的每个状态最多计算一次，每次需要枚举所有硬币。

空间复杂度：

\[ O(t) \]

包括：

- 记忆化字典所占空间；
- 最坏情况下递归调用栈所占空间。

## Python 递归深度说明

Python 默认允许的递归深度通常在一千层左右。如果 `amount` 很大且硬币面额较小，自顶向下的方法可能触发：

```
RecursionError: maximum recursion depth exceeded
```

因此在实际面试和在线判题中，通常更推荐自底向上的动态规划，因为它不依赖递归调用栈。

------

# 方法三：纯递归暴力搜索

## 思路

对于当前剩余金额，尝试选择每一种硬币，然后递归处理剩余金额：

```
dfs(amount - coin)
```

最后，从所有可行选择中取硬币数量的最小值。

这种方法可以得到正确答案，但它会重复计算大量相同的子问题。

例如，计算 `dfs(10)` 时，不同的选择顺序都可能再次进入 `dfs(5)`。

## Python 代码

```
from typing import List


class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        impossible = amount + 1

        def dfs(remaining: int) -> int:
            # 已经恰好凑出目标金额。
            if remaining == 0:
                return 0

            min_coins = impossible

            # 尝试选择每一种硬币。
            for coin in coins:
                if remaining >= coin:
                    result = dfs(remaining - coin)

                    if result != impossible:
                        min_coins = min(min_coins, result + 1)

            return min_coins

        result = dfs(amount)
        return -1 if result == impossible else result
```

## 复杂度分析

设最小硬币面额为 `m`。

递归树的最大深度约为：

\[ \frac{t}{m} \]

每一层最多产生 `n` 个分支，因此时间复杂度上界约为：

\[ O\left(n^{t/m}\right) \]

如果只使用较宽松的上界，也可以写成：

\[ O(n^t) \]

空间复杂度主要来自递归调用栈：

\[ O(t/m) \]

最坏情况下，当最小硬币面额为 `1` 时，为：

\[ O(t) \]

纯递归通常只适合帮助理解状态转移，不适合直接提交。

------

# 方法四：广度优先搜索（BFS）

## 思路

可以把每一个金额看作图中的一个节点。

从金额 `current` 出发，每使用一枚硬币，就可以到达：

```
current + coin
```

每条边代表“使用了一枚硬币”。

因此，问题可以转化为：

> 在一个无权图中，寻找从金额 `0` 到金额 `amount` 的最短路径。

广度优先搜索会逐层访问节点：

```
第 1 层：使用 1 枚硬币能够得到的金额
第 2 层：使用 2 枚硬币能够得到的金额
第 3 层：使用 3 枚硬币能够得到的金额
...
```

第一次到达目标金额时，当前层数就是最少硬币数量。

## Python 代码

```
from collections import deque
from typing import List


class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        if amount == 0:
            return 0

        # 队列中保存当前已经凑出的金额。
        queue = deque([0])

        # seen[a] 表示金额 a 是否已经访问过。
        # 同一个金额不需要重复进入队列。
        seen = [False] * (amount + 1)
        seen[0] = True

        # steps 表示当前使用的硬币数量，也就是 BFS 层数。
        steps = 0

        while queue:
            steps += 1

            # 只处理当前层已有的节点。
            for _ in range(len(queue)):
                current = queue.popleft()

                for coin in coins:
                    next_amount = current + coin

                    # BFS 第一次到达目标时，一定使用了最少硬币。
                    if next_amount == amount:
                        return steps

                    # 超过目标金额，或者已经访问过，则跳过。
                    if next_amount > amount or seen[next_amount]:
                        continue

                    seen[next_amount] = True
                    queue.append(next_amount)

        # 队列耗尽后仍未到达目标，说明无法凑出。
        return -1
```

## `collections.deque` 说明

`deque` 是 Python 标准库提供的双端队列。

BFS 需要频繁从队首删除元素，因此应使用：

```
queue.popleft()
```

它的时间复杂度为：

\[ O(1) \]

如果使用普通列表：

```
queue.pop(0)
```

每次删除队首元素后，后面的元素都需要向前移动，时间复杂度为：

\[ O(k) \]

因此，使用 `deque` 实现 BFS 更高效。

## 为什么需要 `seen`？

同一个金额可能通过不同的硬币组合到达。例如，金额 `3` 可以通过：

```
1 + 2
2 + 1
1 + 1 + 1
```

但 BFS 第一次访问金额 `3` 时，已经找到了到达它的最短路径。之后不需要再次访问，否则会产生大量重复计算。

## 复杂度分析

金额只有 `0` 到 `amount`，因此最多访问 `t + 1` 个状态。每个状态需要尝试 `n` 种硬币。

时间复杂度：

\[ O(n \times t) \]

空间复杂度：

\[ O(t) \]

队列和 `seen` 数组最多保存 `O(t)` 个金额。

------

# 常见错误

## 1. 错误地使用贪心算法

一种常见误区是：每次都选择不超过剩余金额的最大硬币。

这种方法并不总能得到最优解。

例如：

```
coins = [1, 3, 4]
amount = 6
```

贪心算法会选择：

```
4 + 1 + 1
```

一共需要 `3` 枚硬币。

但最优答案是：

```
3 + 3
```

只需要 `2` 枚硬币。

因此，除非题目明确保证硬币系统具有特殊性质，否则不能直接使用贪心算法。

------

## 2. 忘记处理无法凑出的情况

错误写法：

```
return dp[amount]
```

如果目标金额无法凑出，这会返回用于表示无效状态的较大数值。

正确写法：

```
return -1 if dp[amount] == amount + 1 else dp[amount]
```

------

## 3. 错误地把 `dp` 初始化为 `0`

下面的初始化无法区分：

- 凑出金额需要 `0` 枚硬币；
- 当前金额尚未计算；
- 当前金额无法凑出。

```
# 不适合这道题
dp = [0] * (amount + 1)
```

更合适的初始化方式是：

```
dp = [amount + 1] * (amount + 1)
dp[0] = 0
```

------

## 4. 使用无穷大时没有正确处理无效状态

Python 可以使用：

```
float("inf")
```

表示正无穷：

```
dp = [float("inf")] * (amount + 1)
```

Python 的整数本身不会像 Java 或 C++ 的固定宽度整数一样发生普通的整数溢出。不过在 Java 中，如果使用 `Integer.MAX_VALUE` 表示无效状态，再执行：

```
1 + Integer.MAX_VALUE
```

就可能发生溢出。

因此，跨语言编写这道题时，使用 `amount + 1` 作为无效标记通常更加简单、安全。

------

## 5. 混淆“排列数量”和“最少硬币数量”

这道题只要求最少硬币数量，不关心硬币的选择顺序。

例如：

```
1 + 2 + 2
2 + 1 + 2
2 + 2 + 1
```

对于本题来说，它们都代表使用了三枚硬币。状态只需要记录“凑出某个金额所需要的最少数量”，不需要记录所有排列方式。

------

# 方法对比

| 方法             | 时间复杂度 | 空间复杂度 | 特点                                     |
| ---------------- | ---------- | ---------- | ---------------------------------------- |
| 纯递归           | 指数级     | `O(t)`     | 容易理解，但会大量重复计算               |
| 记忆化搜索       | `O(n × t)` | `O(t)`     | 写法接近递归定义，但可能遇到递归深度限制 |
| 自底向上动态规划 | `O(n × t)` | `O(t)`     | 稳定、直观，通常是最推荐的面试答案       |
| BFS              | `O(n × t)` | `O(t)`     | 将问题理解为无权图中的最短路径           |

# 面试总结

这道题的关键是识别出以下结构：

1. 目标是求最小值。
2. 每次可以从多种硬币中做出选择。
3. 选择一枚硬币后，问题会变成规模更小的同类问题。
4. 不同选择会产生重复子问题。
5. 因此可以使用动态规划或记忆化搜索。

面试中建议优先给出自底向上的动态规划解法：

```
dp[a] = min(dp[a], dp[a - coin] + 1)
```

同时需要解释清楚：

- `dp[a]` 的含义；
- `dp[0] = 0` 的原因；
- 为什么使用 `amount + 1` 表示不可达；
- 为什么时间复杂度是 `O(len(coins) × amount)`。