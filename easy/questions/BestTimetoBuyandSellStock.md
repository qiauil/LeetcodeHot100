~~~python
# 买卖股票的最佳时机（Best Time to Buy and Sell Stock）

给定一个整数数组 `prices`，其中 `prices[i]` 表示 NeetCoin 在第 `i` 天的价格。

你可以选择某一天买入一枚 NeetCoin，并在**未来的另一天**将其卖出。

返回你能够获得的最大利润。你也可以选择**不进行任何交易**，此时利润为 `0`。

---

## 核心思路

这道题最重要的限制是：

> **必须先买入，再卖出。**

因此，我们不能简单地找整个数组中的最小值和最大值，因为最大值可能出现在最小值之前。

例如：

```text
prices = [10, 7, 5, 8, 3]
```

虽然：

```text
最大值 = 10
最小值 = 3
```

但不能在价格为 `3` 时买入，再回到过去以 `10` 卖出。

正确的思考方式应该是：

> 对于每一个卖出日期，只考虑它之前出现过的最低买入价格。

这也是后面最优解的核心。

---

# 1. 暴力枚举（Brute Force）

## 思路

最直接的方法是枚举所有可能的“买入日 + 卖出日”。

对于每一天：

* 假设在这一天买入；
* 检查之后的每一天；
* 计算如果在未来某一天卖出，可以得到多少利润；
* 记录所有合法交易中的最大利润。

例如：

```text
prices = [7, 1, 5, 3, 6, 4]
```

如果在第 2 天以 `1` 买入，那么之后可以：

```text
以 5 卖出 → 利润 4
以 3 卖出 → 利润 2
以 6 卖出 → 利润 5
以 4 卖出 → 利润 3
```

因此目前最大利润为 `5`。

## 算法步骤

1. 初始化 `res = 0`，用于保存最大利润。
2. 枚举每一天 `i`，将其作为买入日。
3. 枚举所有 `j > i` 的日期，将其作为卖出日。
4. 计算：

```python
prices[j] - prices[i]
```

5. 更新最大利润。
6. 返回 `res`。

## 代码

```python
from typing import List


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # 最大利润，初始化为 0
        # 这样即使所有交易都会亏钱，也可以选择不交易
        res = 0

        # 枚举买入日期
        for i in range(len(prices)):
            buy = prices[i]

            # 卖出日期必须晚于买入日期
            for j in range(i + 1, len(prices)):
                sell = prices[j]

                # 计算当前交易利润，并更新最大利润
                res = max(res, sell - buy)

        return res
```

## 复杂度分析

* 时间复杂度：**O(n²)**
* 空间复杂度：**O(1)**

因为对于每一个买入日期，都需要检查后面的所有卖出日期。

---

# 2. 双指针（Two Pointers）

## 思路

我们希望：

```text
低价买入 → 之后高价卖出
```

可以使用两个指针：

```text
l = 买入日期
r = 卖出日期
```

其中始终保持：

```text
l < r
```

也就是说，卖出一定发生在买入之后。

假设：

```text
prices[l] < prices[r]
```

说明当前交易能够赚钱：

```python
profit = prices[r] - prices[l]
```

于是更新最大利润。

但如果：

```text
prices[r] <= prices[l]
```

说明在 `r` 这一天的价格更低。

那么 `r` 显然是一个比 `l` 更好的买入位置。

因此可以直接：

```python
l = r
```

然后继续向右寻找未来的卖出机会。

---

## 为什么可以直接把 `l` 移动到 `r`？

假设：

```text
prices[l] = 7
prices[r] = 3
```

对于任何未来价格 `x`：

如果从 `l` 买入：

```text
利润 = x - 7
```

如果从 `r` 买入：

```text
利润 = x - 3
```

显然：

```text
x - 3 >= x - 7
```

所以一旦遇到更低的价格，旧的买入位置就再也没有价值了。

这也是这个算法能够做到 **O(n)** 的关键。

---

## 算法步骤

初始化：

```text
l = 0
r = 1
maxP = 0
```

然后不断移动 `r`：

* 如果 `prices[r] > prices[l]`

  * 计算利润；
  * 更新 `maxP`。
* 否则：

  * 当前价格更低；
  * 将 `l = r`。
* 最后执行：

```python
r += 1
```

---

## 代码

```python
from typing import List


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # l：当前最优买入日期
        # r：当前尝试卖出的日期
        l, r = 0, 1

        # 最大利润
        maxP = 0

        while r < len(prices):

            # 当前卖出价格高于买入价格，可以获得利润
            if prices[l] < prices[r]:
                profit = prices[r] - prices[l]
                maxP = max(maxP, profit)

            else:
                # 当前价格更低，因此它是一个更好的买入位置
                l = r

            # 卖出指针继续向后移动
            r += 1

        return maxP
```

## 复杂度分析

* 时间复杂度：**O(n)**
* 空间复杂度：**O(1)**

数组只被扫描一次。

---

# 3. 动态规划 / 一次遍历（Dynamic Programming / One Pass）

## 思路

这一解法是面试中非常推荐的写法。

在遍历数组的过程中，我们只需要维护两个信息：

```text
1. 到目前为止见过的最低价格
2. 到目前为止能够获得的最大利润
```

假设当前价格是：

```python
sell
```

我们可以把今天想象成“卖出日”。

那么为了获得最大利润，我们一定希望在今天之前用最低价格买入。

因此：

```python
当前利润 = sell - minBuy
```

其中：

```python
minBuy
```

表示之前见过的最低价格。

每访问一天，就做两件事情：

```python
maxP = max(maxP, sell - minBuy)
minBuy = min(minBuy, sell)
```

---

## 状态含义

虽然这里被称为“动态规划”，但它实际上是一个经过空间优化后的 DP。

可以理解为：

```text
minBuy = 截止当前日期的最低价格
maxP   = 截止当前日期的最大利润
```

每一天的答案都只依赖之前的信息，因此不需要完整的 DP 数组。

---

## 算法步骤

1. 初始化：

```python
minBuy = prices[0]
maxP = 0
```

2. 遍历每个价格 `sell`。
3. 假设今天卖出：

```python
sell - minBuy
```

4. 更新最大利润：

```python
maxP = max(maxP, sell - minBuy)
```

5. 更新最低买入价格：

```python
minBuy = min(minBuy, sell)
```

6. 返回 `maxP`。

---

## 代码

```python
from typing import List


class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # 当前找到的最大利润
        maxP = 0

        # 截止目前为止出现过的最低价格
        minBuy = prices[0]

        for sell in prices:
            # 假设今天卖出：
            # 最好的买入价格就是之前见过的最低价格
            profit = sell - minBuy

            # 更新最大利润
            maxP = max(maxP, profit)

            # 更新最低买入价格
            minBuy = min(minBuy, sell)

        return maxP
```

## 复杂度分析

* 时间复杂度：**O(n)**
* 空间复杂度：**O(1)**

---

# 双指针和动态规划有什么区别？

这两个解法本质上其实非常接近。

双指针写法维护的是：

```text
l = 最优买入位置
r = 当前卖出位置
```

而一次遍历写法维护的是：

```text
minBuy = 最低买入价格
sell   = 当前卖出价格
```

例如：

```python
prices = [7, 1, 5, 3, 6, 4]
```

当扫描到：

```text
6
```

时：

```text
minBuy = 1
```

于是：

```text
profit = 6 - 1 = 5
```

双指针方法实际上也在做同样的事情，只不过它保存的是最低价格对应的**下标**：

```text
l = 1
```

而 DP / 一次遍历写法直接保存最低价格：

```text
minBuy = 1
```

所以从代码面试角度来说，第三种写法通常更加简洁。

---

# 示例推演

对于：

```python
prices = [7, 1, 5, 3, 6, 4]
```

一次遍历过程如下：

| 当前价格 `sell` | 当前最低价格 `minBuy` | 当前利润 | 最大利润 |
| ----------: | --------------: | ---: | ---: |
|           7 |               7 |    0 |    0 |
|           1 |               1 |    0 |    0 |
|           5 |               1 |    4 |    4 |
|           3 |               1 |    2 |    4 |
|           6 |               1 |    5 |    5 |
|           4 |               1 |    3 |    5 |

最终答案：

```text
5
```

即：

```text
价格 1 时买入
价格 6 时卖出
```

利润：

```text
6 - 1 = 5
```

---

# 常见错误

## 1. 卖出发生在买入之前

题目要求：

```text
buy day < sell day
```

也就是说，必须先买再卖。

下面这种写法是错误的：

```python
# 错误示例
for i in range(len(prices)):
    for j in range(i):
        # j < i
        # 相当于卖出日期早于买入日期
        profit = prices[j] - prices[i]
```

正确的暴力枚举应该保证：

```python
j > i
```

例如：

```python
for i in range(len(prices)):
    for j in range(i + 1, len(prices)):
        profit = prices[j] - prices[i]
```

---

## 2. 直接用全局最大值减全局最小值

下面的思路是错误的：

```python
max(prices) - min(prices)
```

例如：

```python
prices = [10, 8, 7, 2]
```

会得到：

```text
10 - 2 = 8
```

但价格 `10` 出现在价格 `2` 之前。

这意味着：

```text
先以 2 买入
再以 10 卖出
```

实际上是在“穿越时间”。

因此不能只考虑价格大小，还必须考虑日期顺序。

---

## 3. 返回负利润

例如：

```python
prices = [7, 6, 4, 3, 1]
```

股票一直下跌。

任何交易都会亏钱，因此最优选择是：

```text
不交易
```

答案应该是：

```text
0
```

所以最大利润通常初始化为：

```python
maxP = 0
```

而不是一个负数。

---

# Python 相关知识点

## `List[int]`

代码中的：

```python
def maxProfit(self, prices: List[int]) -> int:
```

使用了 Python 的类型提示（type hint）。

其中：

```python
List[int]
```

表示：

```text
由整数组成的列表
```

需要：

```python
from typing import List
```

例如：

```python
prices: List[int] = [7, 1, 5, 3, 6, 4]
```

在较新的 Python 版本中，也可以直接写：

```python
def maxProfit(self, prices: list[int]) -> int:
```

---

## `range()`

例如：

```python
range(len(prices))
```

假设：

```python
len(prices) == 5
```

那么它产生：

```text
0, 1, 2, 3, 4
```

而：

```python
range(i + 1, len(prices))
```

表示从 `i` 后面的那个位置开始遍历。

因此特别适合用于：

```text
卖出日期必须在买入日期之后
```

这样的场景。

---

## `min()` 和 `max()`

Python 内置函数：

```python
min(a, b)
```

返回较小值。

例如：

```python
minBuy = min(minBuy, sell)
```

表示：

> 在之前的最低价格和当前价格中，保存较低的那个。

类似地：

```python
maxP = max(maxP, profit)
```

表示：

> 在之前的最大利润和当前利润中，保存较大的那个。

---

# 面试中推荐的解法

通常建议直接使用一次遍历：

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = prices[0]
        max_profit = 0

        for price in prices:
            # 如果今天卖出，可以获得的利润
            max_profit = max(
                max_profit,
                price - min_price
            )

            # 更新历史最低买入价格
            min_price = min(
                min_price,
                price
            )

        return max_profit
```

核心可以概括为一句话：

> **遍历每一个卖出日期，并记录它之前出现过的最低买入价格。**

或者用一个非常适合面试时口述的表达：

```text
At every day, I treat the current price as the selling price.
I keep track of the minimum price seen so far as the best buying price.
Then I update the maximum profit using current_price - min_price.
```

对应中文就是：

> 遍历每一天时，我把当前价格看作卖出价格，同时维护此前出现过的最低价格作为最佳买入价格，然后使用“当前价格 − 最低买入价格”更新最大利润。

这个解法达到：

```text
时间复杂度：O(n)
空间复杂度：O(1)
```

也是这道题最值得掌握的解法。xxxxxxxxxx2 1# Wrong: can return negative2return maxPrice - minPrice  # Could be negative if maxPrice found before minPricepython
~~~
