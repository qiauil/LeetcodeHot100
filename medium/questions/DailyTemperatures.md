# 每日温度（Daily Temperatures）

给定一个整数数组 `temperatures`，其中 `temperatures[i]` 表示第 `i` 天的温度。

请返回一个数组 `result`，其中 `result[i]` 表示：从第 `i` 天开始，需要等待多少天才能遇到温度更高的一天。

如果未来不存在温度更高的日子，则令 `result[i] = 0`。

例如：

```
输入：temperatures = [73, 74, 75, 71, 69, 72, 76, 73]
输出：             [1,  1,  4,  2,  1,  1,  0,  0]
```

------

## 解法一：暴力枚举

### 思路

对于每一天，依次向后检查未来的温度：

- 如果找到一个温度严格高于当天温度的日子，就记录两个日期之间的距离。
- 如果检查到数组末尾仍未找到，则结果为 `0`。

这种方法直观且容易实现，但每一天都可能检查大量后续元素，因此效率较低。

### 算法步骤

1. 创建结果数组 `res`，初始值全部为 `0`。
2. 对每个位置 `i`：
   - 从下一天 `i + 1` 开始向后搜索。
   - 找到第一个满足 `temperatures[j] > temperatures[i]` 的位置 `j`。
   - 将等待天数 `j - i` 保存到 `res[i]`。
3. 返回 `res`。

### 代码

```
from typing import List


class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        res = [0] * n

        for i in range(n):
            # 从当前日期的下一天开始寻找
            for j in range(i + 1, n):
                if temperatures[j] > temperatures[i]:
                    # 两个下标之差就是需要等待的天数
                    res[i] = j - i
                    break

        return res
```

### 复杂度分析

- 时间复杂度：`O(n²)`
  - 最坏情况下，每一天都需要检查几乎所有后续日期。
- 空间复杂度：
  - 不考虑返回结果：`O(1)`
  - 包含返回结果：`O(n)`

------

## 解法二：单调栈

### 思路

我们需要解决的问题本质上是：

> 对于数组中的每个元素，找到它右侧第一个严格大于它的元素。

这类“下一个更大元素”问题通常可以使用单调栈解决。

栈中保存尚未找到更高温度的日期。遍历到第 `i` 天时：

- 如果当前温度高于栈顶日期的温度，说明第 `i` 天就是栈顶日期等待的第一个更暖和的日子。
- 弹出栈顶，并用下标之差计算等待天数。
- 继续比较新的栈顶，因为当前温度可能同时解决多个较早日期。
- 最后将当前日期压入栈中。

栈中温度从栈底到栈顶呈单调不增，因此这种结构称为“单调栈”。

### 算法步骤

1. 创建长度为 `n` 的结果数组 `res`，初始值全部为 `0`。
2. 创建栈 `stack`，保存尚未找到更高温度的日期下标。
3. 从左到右遍历温度数组：
   - 当栈不为空，并且当前温度严格高于栈顶日期的温度时：
     - 弹出栈顶下标 `previous_day`。
     - 设置 `res[previous_day] = i - previous_day`。
   - 将当前日期下标 `i` 压入栈中。
4. 遍历结束后，仍留在栈中的日期不存在更暖和的未来日期，其结果保持为 `0`。
5. 返回 `res`。

### 代码

```
from typing import List


class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        res = [0] * n

        # 栈中保存日期下标，而不是温度本身
        stack = []

        for i, current_temperature in enumerate(temperatures):
            # 当前温度可以为栈顶日期提供答案
            while (
                stack
                and current_temperature > temperatures[stack[-1]]
            ):
                previous_day = stack.pop()
                res[previous_day] = i - previous_day

            # 当前日期等待未来某个更暖和的日子
            stack.append(i)

        return res
```

### 示例过程

对于：

```
temperatures = [73, 74, 75, 71, 69, 72]
```

部分处理过程如下：

```
当前温度 73：栈为空，将下标 0 压栈
当前温度 74：74 > 73，弹出 0，res[0] = 1
当前温度 75：75 > 74，弹出 1，res[1] = 1
当前温度 71：不能解决 75，将下标 3 压栈
当前温度 69：不能解决 71，将下标 4 压栈
当前温度 72：依次解决温度 69 和 71 对应的日期
```

### 复杂度分析

- 时间复杂度：`O(n)`
  - 每个下标最多入栈一次、出栈一次。
- 空间复杂度：`O(n)`
  - 最坏情况下，所有下标都会保存在栈中。

### Python 相关说明

#### `enumerate`

```
for i, temperature in enumerate(temperatures):
```

`enumerate` 可以同时获得元素的下标和值，相当于：

```
for i in range(len(temperatures)):
    temperature = temperatures[i]
```

#### `stack[-1]`

Python 列表的负数下标表示从末尾访问元素：

```
stack[-1]
```

表示栈顶元素，但不会将它删除。

#### `list.pop()`

```
previous_day = stack.pop()
```

`pop()` 会删除并返回列表的最后一个元素，因此可以用 Python 的列表模拟栈。

#### Python 中的短路求值

```
while stack and current_temperature > temperatures[stack[-1]]:
```

Python 会先判断 `stack` 是否为空。只有当栈不为空时，才会执行后面的 `stack[-1]`，从而避免访问空列表。

------

## 解法三：动态规划与跳跃查询

### 思路

暴力解法会逐天向后检查。实际上，我们可以复用已经计算出的答案，从而跳过一些没有必要检查的日期。

由于第 `i` 天需要使用右侧日期的计算结果，因此要从右向左遍历数组。

假设正在计算第 `i` 天，并检查到第 `j` 天：

- 如果 `temperatures[j] > temperatures[i]`，那么 `j` 就是答案。
- 如果第 `j` 天不够暖，但 `res[j] > 0`，说明我们已经知道第 `j` 天之后的下一个更暖日期，可以直接跳到 `j + res[j]`。
- 如果 `res[j] == 0`，说明第 `j` 天之后没有任何日期比第 `j` 天更暖。由于第 `j` 天本身又不比第 `i` 天暖，因此第 `i` 天也不可能在后面找到更暖的日子。

### 算法步骤

1. 创建结果数组 `res`，初始值全部为 `0`。
2. 从倒数第二天开始，从右向左遍历。
3. 对于每个位置 `i`：
   - 从下一天 `j = i + 1` 开始检查。
   - 如果第 `j` 天不比第 `i` 天暖：
     - 如果 `res[j] == 0`，停止搜索。
     - 否则跳到 `j + res[j]`。
   - 如果最终找到更暖的第 `j` 天，则令 `res[i] = j - i`。
4. 返回 `res`。

### 代码

```
from typing import List


class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        res = [0] * n

        # 最后一天之后没有其他日期，所以从倒数第二天开始
        for i in range(n - 2, -1, -1):
            j = i + 1

            while j < n and temperatures[j] <= temperatures[i]:
                # 第 j 天之后不存在比第 j 天更暖的日期。
                # 同时，第 j 天又不比第 i 天暖，因此可以结束搜索。
                if res[j] == 0:
                    j = n
                    break

                # 利用已经计算出的答案直接向后跳跃
                j += res[j]

            if j < n:
                res[i] = j - i

        return res
```

### 为什么 `res[j] == 0` 时可以停止？

当前已知：

```
temperatures[j] <= temperatures[i]
```

而 `res[j] == 0` 表示在 `j` 之后，不存在温度严格高于 `temperatures[j]` 的日期。

更准确地说，在本题的跳跃过程中，能够到达 `j` 说明沿途所有候选日期都没有超过 `temperatures[i]`。此时继续搜索不会产生有效答案，因此可以停止。

### 复杂度分析

- 时间复杂度：通常可视为 `O(n)`
  - 已计算的结果允许算法跳过大量无效位置。
  - 相比单调栈，该方法的线性复杂度证明不够直观，因此面试中通常更推荐单调栈。
- 空间复杂度：
  - 不考虑返回结果：`O(1)`
  - 包含返回结果：`O(n)`

------

## 常见错误

### 1. 使用 `>=` 而不是 `>`

题目要求的是未来第一个更暖和的日子，也就是温度必须严格升高。

```
# 错误：相同温度也会被认为是更暖
while stack and current_temperature >= temperatures[stack[-1]]:
    index = stack.pop()
```

正确写法：

```
# 正确：只有温度严格升高时才弹栈
while stack and current_temperature > temperatures[stack[-1]]:
    index = stack.pop()
    res[index] = i - index
```

例如：

```
temperatures = [70, 70, 71]
```

第一天不能把第二天当作更暖的一天；第一天和第二天都应该等待到第三天。

------

### 2. 混淆栈中保存的是温度还是下标

栈中既可以保存 `(温度, 下标)`，也可以只保存下标。

保存二元组的写法：

```
stack = []

for i, temperature in enumerate(temperatures):
    while stack and temperature > stack[-1][0]:
        previous_temperature, previous_day = stack.pop()
        res[previous_day] = i - previous_day

    stack.append((temperature, i))
```

只保存下标的写法：

```
stack = []

for i, temperature in enumerate(temperatures):
    while stack and temperature > temperatures[stack[-1]]:
        previous_day = stack.pop()
        res[previous_day] = i - previous_day

    stack.append(i)
```

只保存下标通常更简洁，因为可以通过：

```
temperatures[stack[-1]]
```

访问栈顶日期的温度。

------

### 3. 等待天数出现差一错误

等待天数应直接使用两个日期的下标之差：

```
res[i] = j - i
```

例如，第 `2` 天需要等到第 `5` 天：

```
等待天数 = 5 - 2 = 3
```

使用单独的计数器时，很容易因初始值错误而多算或少算一天：

```
# 容易出现差一错误
count = 0
j = i + 1

while j < n and temperatures[j] <= temperatures[i]:
    j += 1
    count += 1
```

因此，找到目标位置后直接计算下标之差更加可靠。

------

### 4. 只使用一次 `if`，没有持续弹栈

当前温度可能同时是多个历史日期遇到的第一个更高温度，因此必须使用 `while`：

```
# 错误：最多只能处理一个日期
if stack and current_temperature > temperatures[stack[-1]]:
    previous_day = stack.pop()
    res[previous_day] = i - previous_day
```

正确写法：

```
# 正确：持续处理所有温度低于当前温度的栈顶日期
while stack and current_temperature > temperatures[stack[-1]]:
    previous_day = stack.pop()
    res[previous_day] = i - previous_day
```

------

## 解法总结

| 解法         | 时间复杂度    | 额外空间 | 特点                     |
| ------------ | ------------- | -------- | ------------------------ |
| 暴力枚举     | `O(n²)`       | `O(1)`   | 最直观，但效率较低       |
| 单调栈       | `O(n)`        | `O(n)`   | 标准解法，面试中最推荐   |
| 动态规划跳跃 | 通常为 `O(n)` | `O(1)`   | 复用右侧答案，思路较巧妙 |

这道题最值得掌握的是单调栈解法。它不仅适用于每日温度，还常用于：

- 下一个更大元素
- 下一个更小元素
- 股票价格跨度
- 柱状图中的最大矩形
- 接雨水问题中的部分解法