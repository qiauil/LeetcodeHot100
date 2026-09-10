# 爬楼梯（Climbing Stairs）

给定一个整数 `n`，表示到达楼梯顶部一共有 `n` 级台阶。每次你可以爬：

- `1` 级台阶
- `2` 级台阶

请返回到达楼梯顶部一共有多少种不同的爬法。

## 示例 1

```text
输入：n = 2

输出：2
```

解释：

1. `1 + 1 = 2`
2. `2 = 2`

## 示例 2

```text
输入：n = 3

输出：3
```

解释：

1. `1 + 1 + 1 = 3`
2. `1 + 2 = 3`
3. `2 + 1 = 3`

------

# 核心思路

这道题最重要的状态转移关系是：

```text
ways(n) = ways(n - 1) + ways(n - 2)
```

原因很简单：如果我们最终要到达第 `n` 级台阶，那么最后一步只有两种可能：

- 从第 `n - 1` 级走 `1` 步到达 `n`
- 从第 `n - 2` 级走 `2` 步到达 `n`

因此，到达第 `n` 级的方法数，就是这两种情况的方法数之和。

这其实就是一个 **Fibonacci（斐波那契）型递推关系**。

------

# 1. 递归

## 思路

站在任意位置 `i` 时，我们都有两个选择：

- 爬 `1` 级，来到 `i + 1`
- 爬 `2` 级，来到 `i + 2`

因此，可以通过递归搜索所有可能的路线。

如果：

- 正好到达 `n`，说明找到一种合法方案，返回 `1`
- 超过 `n`，说明当前路线无效，返回 `0`

于是：

```text
dfs(i) = dfs(i + 1) + dfs(i + 2)
```

这种方法本质上是在构建一棵二叉递归树，并枚举所有可能的走法。

## 算法步骤

1. 从第 `0` 级开始。

2. 定义递归函数 `dfs(i)`，表示从当前位置 `i` 出发，到达顶部的方法数。

3. 如果 `i == n`，返回 `1`。

4. 如果 `i > n`，返回 `0`。

5. 否则分别尝试走 `1` 级和 `2` 级：

   ```text
   dfs(i + 1) + dfs(i + 2)
   ```

6. 最终返回 `dfs(0)`。

## 代码

```python
class Solution:
    def climbStairs(self, n: int) -> int:

        def dfs(i: int) -> int:
            # 正好到达顶部，找到一种合法方案
            if i == n:
                return 1

            # 超过顶部，当前方案无效
            if i > n:
                return 0

            # 分别尝试走 1 步和 2 步
            return dfs(i + 1) + dfs(i + 2)

        return dfs(0)
```

原代码中的写法：

```python
if i >= n:
    return i == n
```

也是正确的，因为 Python 中：

```python
True == 1
False == 0
```

所以：

```python
return i == n
```

实际上等价于：

```python
if i == n:
    return 1
else:
    return 0
```

不过在面试中，显式写成 `return 1` 和 `return 0` 通常更加直观。

## 复杂度

- 时间复杂度：**O(2ⁿ)**
- 空间复杂度：**O(n)**

空间复杂度来自递归调用栈。最深情况下，每次只走 `1` 级，因此递归深度约为 `n`。

### 为什么时间复杂度是 O(2ⁿ)？

因为每个状态大致都会产生两个递归分支：

```text
dfs(i)
├── dfs(i + 1)
└── dfs(i + 2)
```

存在大量重复计算。例如：

```text
dfs(0)
├── dfs(1)
│   ├── dfs(2)
│   └── dfs(3)
└── dfs(2)
```

这里的 `dfs(2)` 被计算了多次。

这也是下一种方法——记忆化搜索——要解决的问题。

------

# 2. 动态规划：自顶向下（Top-Down / Memoization）

## 思路

普通递归效率低的根本原因是：

> 同一个状态被反复计算。

例如，只要我们到达位置 `i`，那么：

```text
从 i 到顶部的方法数
```

永远是固定的。

因此，第一次计算完 `dfs(i)` 后，可以将结果保存起来。以后再次遇到 `i` 时，直接使用之前的结果。

这种技术叫做：

**Memoization（记忆化）**

也叫：

**Top-Down Dynamic Programming（自顶向下动态规划）**

它仍然保留了递归的思考方式，但是通过缓存消除了重复计算。

## 算法步骤

1. 创建数组 `cache`，用于保存每个位置的答案。

2. 定义 `dfs(i)`。

3. 如果正好到达 `n`，返回 `1`。

4. 如果超过 `n`，返回 `0`。

5. 如果 `cache[i]` 已经计算过，直接返回缓存值。

6. 否则计算：

   ```text
   dfs(i + 1) + dfs(i + 2)
   ```

7. 将结果保存到 `cache[i]`。

8. 返回 `dfs(0)`。

## 代码

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        # cache[i] 表示从第 i 级出发，到达顶部的方法数
        # -1 表示这个状态还没有计算过
        cache = [-1] * n

        def dfs(i: int) -> int:
            # 正好到达顶部
            if i == n:
                return 1

            # 超过顶部
            if i > n:
                return 0

            # 如果之前已经计算过，直接使用缓存
            if cache[i] != -1:
                return cache[i]

            # 计算当前状态
            cache[i] = dfs(i + 1) + dfs(i + 2)

            return cache[i]

        return dfs(0)
```

## `[-1] * n` 是什么？

这是 Python 中非常常见的列表初始化方式：

```python
cache = [-1] * n
```

例如：

```python
n = 5
cache = [-1] * n
```

结果是：

```python
[-1, -1, -1, -1, -1]
```

这里使用 `-1` 表示：

> 这个状态还没有被计算过。

因为合法的爬楼梯方案数量一定不会是负数，所以 `-1` 可以安全地作为特殊标记。

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

虽然代码中仍然存在两个递归调用：

```python
dfs(i + 1) + dfs(i + 2)
```

但是每个 `i` 实际上只会被真正计算一次。

总共只有：

```text
0, 1, 2, ..., n
```

大约 `n` 个状态，因此时间复杂度下降为 `O(n)`。

空间主要包括：

- `cache`：O(n)
- 递归调用栈：O(n)

------

# 3. 动态规划：自底向上（Bottom-Up）

## 思路

前面的递归是从：

```text
0 → n
```

不断向后探索。

我们也可以反过来思考：

> 到达第 `i` 级台阶的方法数是多少？

如果定义：

```text
dp[i] = 到达第 i 级台阶的方法数
```

那么到达 `i` 的最后一步只有两种可能：

```text
从 i - 1 走 1 步
从 i - 2 走 2 步
```

因此：

```text
dp[i] = dp[i - 1] + dp[i - 2]
```

这就是标准的动态规划状态转移方程。

## 初始状态

```text
dp[1] = 1
dp[2] = 2
```

因为：

到第 1 级：

```text
1
```

只有一种方法。

到第 2 级：

```text
1 + 1
2
```

有两种方法。

## 算法步骤

1. 如果 `n <= 2`，直接返回 `n`。

2. 创建数组：

   ```python
   dp = [0] * (n + 1)
   ```

3. 初始化：

   ```python
   dp[1] = 1
   dp[2] = 2
   ```

4. 从 `3` 开始计算：

   ```python
   dp[i] = dp[i - 1] + dp[i - 2]
   ```

5. 返回 `dp[n]`。

## 代码

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        # 处理最小规模的情况
        if n <= 2:
            return n

        # dp[i] 表示到达第 i 级台阶的方法数
        dp = [0] * (n + 1)

        # 基础状态
        dp[1] = 1
        dp[2] = 2

        # 根据状态转移方程逐步计算
        for i in range(3, n + 1):
            dp[i] = dp[i - 1] + dp[i - 2]

        return dp[n]
```

## `range(3, n + 1)` 的含义

Python 中：

```python
range(start, end)
```

包含 `start`，但**不包含 `end`**。

因此：

```python
range(3, n + 1)
```

实际上遍历：

```text
3, 4, 5, ..., n
```

例如：

```python
list(range(3, 6))
```

得到：

```python
[3, 4, 5]
```

因此这里必须写 `n + 1`，否则不会计算 `dp[n]`。

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

------

# 4. 动态规划：空间优化

## 思路

观察状态转移：

```text
dp[i] = dp[i - 1] + dp[i - 2]
```

计算当前状态时，我们实际上只需要知道前两个状态：

```text
dp[i - 1]
dp[i - 2]
```

更早的状态：

```text
dp[i - 3]
dp[i - 4]
...
```

已经不会再被使用。

因此，没有必要保存整个 `dp` 数组，只需要使用两个变量。

这是一种非常常见的动态规划优化：

> 如果当前状态只依赖固定数量的前置状态，就可以考虑压缩 DP 空间。

## 更容易理解的写法

可以把两个变量命名为：

```text
prev1 = dp[i - 1]
prev2 = dp[i - 2]
```

## 代码

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n <= 2:
            return n

        # prev2 = dp[i - 2]
        # prev1 = dp[i - 1]
        prev2 = 1
        prev1 = 2

        for i in range(3, n + 1):
            # 当前状态
            current = prev1 + prev2

            # 将两个变量向前移动
            prev2 = prev1
            prev1 = current

        return prev1
```

这和原答案中的：

```python
one, two = 1, 1

for i in range(n - 1):
    temp = one
    one = one + two
    two = temp

return one
```

本质完全一样。

不过在面试中，我更推荐 `prev1 / prev2 / current` 版本，因为变量与状态转移：

```text
dp[i - 1]
dp[i - 2]
dp[i]
```

之间的对应关系更加清楚。

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(1)**

这通常也是本题面试中最推荐的解法。

------

# 5. 与 Fibonacci 数列的关系

爬楼梯问题实际上就是 Fibonacci 数列的一个变体。

如果 Fibonacci 定义为：

```text
F(0) = 0
F(1) = 1
F(n) = F(n - 1) + F(n - 2)
```

那么爬楼梯答案为：

```text
ways(n) = F(n + 1)
```

例如：

| `n`  | 爬楼梯方法数 | Fibonacci |
| ---- | ------------ | --------- |
| 1    | 1            | F(2) = 1  |
| 2    | 2            | F(3) = 2  |
| 3    | 3            | F(4) = 3  |
| 4    | 5            | F(5) = 5  |
| 5    | 8            | F(6) = 8  |

因此，本题实际上可以看成：

> 求第 `n + 1` 个 Fibonacci 数。

------

# 6. 矩阵快速幂

## 思路

由于本题满足：

```text
ways(n) = ways(n - 1) + ways(n - 2)
```

我们可以利用 Fibonacci 数列的矩阵表达：

```text
| F(n + 1) |   | 1  1 |   | F(n)     |
| F(n)     | = | 1  0 | × | F(n - 1) |
```

定义矩阵：

```text
M = | 1  1 |
    | 1  0 |
```

则：

```text
M^n
```

中会包含 Fibonacci 数。

因为爬楼梯答案是：

```text
F(n + 1)
```

所以：

```text
M^n[0][0]
```

正好对应答案。

关键在于，矩阵的 `n` 次方可以通过 **Binary Exponentiation（二进制快速幂）** 在 `O(log n)` 次乘法内完成。

## 代码

```python
class Solution:
    def climbStairs(self, n: int) -> int:

        def matrix_mult(A, B):
            """
            计算两个 2 x 2 矩阵的乘积
            """

            return [
                [
                    A[0][0] * B[0][0] + A[0][1] * B[1][0],
                    A[0][0] * B[0][1] + A[0][1] * B[1][1],
                ],
                [
                    A[1][0] * B[0][0] + A[1][1] * B[1][0],
                    A[1][0] * B[0][1] + A[1][1] * B[1][1],
                ],
            ]

        def matrix_pow(matrix, power):
            """
            使用快速幂计算 matrix ^ power
            """

            # 单位矩阵，相当于普通乘法中的数字 1
            result = [
                [1, 0],
                [0, 1],
            ]

            base = matrix

            while power > 0:
                # 如果当前二进制最低位是 1，
                # 将当前 base 乘到答案中
                if power % 2 == 1:
                    result = matrix_mult(result, base)

                # base 不断平方
                base = matrix_mult(base, base)

                # 相当于二进制右移一位
                power //= 2

            return result

        fibonacci_matrix = [
            [1, 1],
            [1, 0],
        ]

        result = matrix_pow(fibonacci_matrix, n)

        return result[0][0]
```

## 为什么快速幂是 O(log n)？

例如计算：

```text
x^13
```

普通方法需要：

```text
x × x × x × ... × x
```

大约进行 `13` 次操作。

但因为：

```text
13 = 1101₂
```

可以利用不断平方：

```text
x
x²
x⁴
x⁸
```

只需要大约：

```text
log₂(n)
```

次迭代。

代码中的：

```python
power //= 2
```

就是不断把指数减半。

## 单位矩阵

代码中：

```python
result = [
    [1, 0],
    [0, 1],
]
```

叫做 **Identity Matrix（单位矩阵）**。

它在线性代数中的作用类似普通数字里的 `1`：

```text
1 × x = x
```

对应矩阵：

```text
I × A = A
```

因此，在进行矩阵快速幂时，结果需要从单位矩阵开始。

## 复杂度

- 时间复杂度：**O(log n)**
- 空间复杂度：**O(1)**

因为矩阵始终只有 `2 × 2` 大小，所以矩阵本身占用常数空间。

------

# 7. 数学公式：Binet Formula

## 思路

Fibonacci 数列存在一个闭式公式，称为：

**Binet's Formula（比内公式）**

定义：

```text
φ = (1 + √5) / 2
ψ = (1 - √5) / 2
```

那么：

```text
F(n) = (φⁿ - ψⁿ) / √5
```

由于：

```text
ways(n) = F(n + 1)
```

所以可以直接通过公式计算答案。

## 代码

```python
import math


class Solution:
    def climbStairs(self, n: int) -> int:
        # √5
        sqrt5 = math.sqrt(5)

        # 黄金比例
        phi = (1 + sqrt5) / 2

        # 黄金比例的共轭
        psi = (1 - sqrt5) / 2

        # 爬楼梯答案对应 Fibonacci(n + 1)
        n += 1

        # 浮点计算可能存在精度误差，因此使用 round
        return round((phi ** n - psi ** n) / sqrt5)
```

## `math.sqrt`

需要首先：

```python
import math
```

然后：

```python
math.sqrt(5)
```

用于计算：

```text
√5
```

例如：

```python
math.sqrt(9)
```

结果为：

```python
3.0
```

注意返回值通常是浮点数。

## `round`

```python
round(x)
```

用于将一个浮点数四舍五入为最接近的整数。

这里之所以需要 `round`，是因为浮点数计算可能产生类似：

```text
7.999999999999998
```

而理论答案实际上应该是：

```text
8
```

## 一个重要的面试注意点

虽然从数学模型来看，这种方法非常漂亮，但在实际编程面试中，一般**不推荐把 Binet 公式作为主要解法**。

原因是：

> Python 的普通浮点数精度有限。

当 `n` 很大时：

```python
phi ** n
```

可能产生数值精度误差。

因此，在要求严格整数精度时：

- 空间优化 DP 更可靠
- 矩阵快速幂也更可靠

另外，将 `phi ** n` 简单描述为严格意义上的 `O(1)` 或 `O(log n)` 都需要依赖具体数值计算模型。在普通算法面试的简化模型下，通常会把这种方法视作常数额外空间，但不要过度依赖它做复杂度优势论证。

------

# 常见错误

## 1. Base Case 的 Off-by-One Error

这是这道题最常见的问题之一。

如果定义：

```text
dp[i] = 到达第 i 级的方法数
```

那么：

```text
dp[1] = 1
dp[2] = 2
```

之后应该从：

```text
i = 3
```

开始计算。

错误写法：

```python
# 错误：过早开始循环，会覆盖基础状态
for i in range(1, n + 1):
    dp[i] = dp[i - 1] + dp[i - 2]
```

正确写法：

```python
# 正确：保留 dp[1] 和 dp[2]
for i in range(3, n + 1):
    dp[i] = dp[i - 1] + dp[i - 2]
```

------

## 2. 没有处理 `n = 1` 和 `n = 2`

例如直接写：

```python
dp = [0] * (n + 1)
dp[1] = 1
dp[2] = 2
```

当：

```text
n = 1
```

时，数组只有：

```python
[0, 0]
```

此时访问：

```python
dp[2]
```

就会出现：

```text
IndexError
```

因此通常先处理：

```python
if n <= 2:
    return n
```

再执行一般逻辑。

------

## 3. 没有明确 DP 状态的含义

动态规划题里，一个很重要的习惯是，在写状态转移之前先明确：

```text
dp[i] 到底表示什么？
```

本题最自然的定义是：

```text
dp[i] = 到达第 i 级台阶的方法数
```

然后再推导：

```text
dp[i] = dp[i - 1] + dp[i - 2]
```

面试时建议按照下面的顺序思考：

```text
1. 定义状态
2. 推导状态转移
3. 确定 base case
4. 确定遍历顺序
5. 考虑空间优化
```

------

# 面试中的推荐解题路径

如果面试官给出这道题，一个很自然的思考过程是：

```text
暴力递归
    ↓
发现重复子问题
    ↓
Memoization
    ↓
Bottom-Up DP
    ↓
发现只依赖前两个状态
    ↓
O(1) 空间优化
```

最终推荐实现：

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n <= 2:
            return n

        prev2 = 1  # dp[i - 2]
        prev1 = 2  # dp[i - 1]

        for i in range(3, n + 1):
            current = prev1 + prev2

            prev2 = prev1
            prev1 = current

        return prev1
```

复杂度：

```text
时间：O(n)
空间：O(1)
```

这是本题在代码面试中通常最值得掌握的版本。

------

# 算法总结

这道题虽然简单，但包含了一个非常典型的动态规划演化过程：

```text
递归
    ↓
重复子问题
    ↓
记忆化搜索
    ↓
动态规划
    ↓
滚动变量 / 状态压缩
```

核心递推式始终没有变化：

```text
dp[i] = dp[i - 1] + dp[i - 2]
```

不同方案之间真正变化的是：

> **如何计算并保存这些状态。**

| 方法         | 时间复杂度     | 空间复杂度 | 面试推荐程度 |
| ------------ | -------------- | ---------- | ------------ |
| 暴力递归     | O(2ⁿ)          | O(n)       | 适合解释思路 |
| Top-Down DP  | O(n)           | O(n)       | 推荐         |
| Bottom-Up DP | O(n)           | O(n)       | 推荐         |
| 空间优化 DP  | **O(n)**       | **O(1)**   | **最推荐**   |
| 矩阵快速幂   | O(log n)       | O(1)       | 进阶         |
| Binet 公式   | 取决于数值模型 | O(1)       | 数学拓展     |

对于实际面试，最重要的是能够清楚解释：

```text
为什么 dp[i] = dp[i - 1] + dp[i - 2]
```

并能够从 `O(n)` 空间进一步优化到 `O(1)` 空间。
