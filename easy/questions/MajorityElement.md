# Majority Element（多数元素）

## 题目描述

给定一个长度为 `n` 的整数数组 `nums`，返回其中的 **多数元素（Majority Element）**。

多数元素指的是在数组中出现次数 **严格超过 `⌊n / 2⌋` 次** 的元素。

题目保证：**数组中一定存在多数元素。**

### 示例 1

```text
输入：nums = [5,5,1,1,1,5,5]

输出：5
```

`5` 一共出现了 `4` 次，而：

```text
⌊7 / 2⌋ = 3
```

因此 `5` 是多数元素。

### 示例 2

```text
输入：nums = [2,2,2]

输出：2
```

### 约束条件

```text
1 <= nums.length <= 50,000
-1,000,000,000 <= nums[i] <= 1,000,000,000
```

### Follow-up

能否做到：

```text
时间复杂度：O(n)
空间复杂度：O(1)
```

答案是可以的，最经典的方法就是 **Boyer-Moore Voting Algorithm（Boyer-Moore 投票算法）**。

------

# 核心观察

多数元素出现次数超过整个数组的一半。

也就是说，如果把：

- 一个多数元素
- 一个非多数元素

两两配对并一起消除，那么最终 **多数元素一定会剩下来**。

这正是 Boyer-Moore 投票算法的核心思想。

例如：

```text
nums = [5, 5, 1, 1, 1, 5, 5]
```

可以想象不断进行：

```text
5 和 1 抵消
5 和 1 抵消
5 和 1 抵消
```

因为 `5` 的数量严格超过其他所有数字数量的总和的一半，所以无论如何抵消，最后留下来的候选者一定是 `5`。

------

# 解法一：Boyer-Moore Voting Algorithm

这是这道题最重要、也是面试中最推荐掌握的解法。

## 思路

维护两个变量：

```python
candidate
count
```

其中：

- `candidate`：当前认为可能是多数元素的候选者
- `count`：当前候选者相对于其他元素的“净票数”

遍历数组时：

```text
遇到 candidate：
    count += 1

遇到其他元素：
    count -= 1
```

如果：

```text
count == 0
```

说明之前的候选者和其他元素已经完全抵消，因此我们可以选择当前元素作为新的候选者。

由于真正的多数元素出现次数超过 `n / 2`，它最终一定能够在这种抵消过程中存活下来。

------

## 算法步骤

1. 初始化：

```python
candidate = None
count = 0
```

1. 遍历数组中的每个 `num`。
2. 如果：

```python
count == 0
```

将：

```python
candidate = num
```

1. 如果：

```python
num == candidate
```

则：

```python
count += 1
```

否则：

```python
count -= 1
```

1. 遍历结束后返回 `candidate`。

------

## Python 实现

```python
from typing import List


class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        candidate = None
        count = 0

        for num in nums:
            # 当前候选者的票数已经被完全抵消，
            # 因此选择当前数字作为新的候选者
            if count == 0:
                candidate = num

            # 相同元素投赞成票，不同元素互相抵消
            if num == candidate:
                count += 1
            else:
                count -= 1

        return candidate
```

也可以把最后的判断压缩为：

```python
class Solution:
    def majorityElement(self, nums):
        candidate = count = 0

        for num in nums:
            if count == 0:
                candidate = num

            count += 1 if num == candidate else -1

        return candidate
```

其中：

```python
1 if num == candidate else -1
```

是 Python 的条件表达式，相当于：

```python
if num == candidate:
    count += 1
else:
    count -= 1
```

------

## 为什么 Boyer-Moore 一定正确？

这是理解这个算法最关键的地方。

假设多数元素是：

```text
M
```

由于 `M` 出现次数超过一半：

```text
count(M) > n / 2
```

因此所有非 `M` 元素的总数量一定小于 `M` 的数量。

Boyer-Moore 中的：

```text
count -= 1
```

可以理解为：

> 用一个不同元素和一个 candidate 进行配对，然后把它们一起消除。

即使我们不断让：

```text
一个 M + 一个非 M
```

互相抵消，由于 `M` 的数量更多，最终一定仍然至少会剩下一个 `M`。

因此最终候选者必然是多数元素。

------

## 一个完整例子

```text
nums = [5, 5, 1, 1, 1, 5, 5]
```

遍历过程：

| num  | candidate | count | 说明           |
| ---- | --------- | ----- | -------------- |
| 5    | 5         | 1     | 选择 5         |
| 5    | 5         | 2     | 相同，+1       |
| 1    | 5         | 1     | 不同，-1       |
| 1    | 5         | 0     | 不同，抵消     |
| 1    | 1         | 1     | 重新选择候选者 |
| 5    | 1         | 0     | 不同，抵消     |
| 5    | 5         | 1     | 重新选择 5     |

最终：

```text
candidate = 5
```

------

## 时间与空间复杂度

```text
时间复杂度：O(n)
空间复杂度：O(1)
```

只需要遍历一次数组，并且只使用两个额外变量。

这正好满足题目的 Follow-up 要求。

------

## 一个非常重要的前提

Boyer-Moore 找到的是：

> 如果多数元素存在，那么它一定是最终 candidate。

本题已经明确保证：

```text
majority element always exists
```

所以可以直接返回。

但是如果题目 **不保证多数元素存在**，则必须再扫描一次数组验证：

```python
class Solution:
    def majorityElement(self, nums):
        candidate = None
        count = 0

        # 第一遍：寻找候选者
        for num in nums:
            if count == 0:
                candidate = num

            count += 1 if num == candidate else -1

        # 第二遍：验证候选者
        if nums.count(candidate) > len(nums) // 2:
            return candidate

        return None
```

------

# 解法二：Hash Map / 哈希表

## 思路

最直观的优化方式是记录每个数字出现了多少次。

例如：

```text
nums = [5,5,1,1,1,5,5]
```

可以建立：

```text
5 -> 4
1 -> 3
```

然后找到出现次数最多的数字。

由于题目保证多数元素存在，因此出现次数最大的元素一定就是答案。

------

## Python 实现

```python
from typing import List
from collections import defaultdict


class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        count = defaultdict(int)

        result = nums[0]
        max_count = 0

        for num in nums:
            # 记录 num 出现的次数
            count[num] += 1

            # 如果当前数字成为出现次数最多的元素，
            # 更新答案
            if count[num] > max_count:
                result = num
                max_count = count[num]

        return result
```

事实上，因为多数元素一旦出现次数超过一半，就已经可以确定答案，因此还可以提前返回：

```python
from collections import defaultdict


class Solution:
    def majorityElement(self, nums):
        count = defaultdict(int)
        n = len(nums)

        for num in nums:
            count[num] += 1

            # 一旦超过一半，立即找到答案
            if count[num] > n // 2:
                return num
```

------

## `defaultdict(int)` 是什么？

`defaultdict` 来自：

```python
from collections import defaultdict
```

普通字典：

```python
count = {}
```

如果直接执行：

```python
count[num] += 1
```

而 `num` 之前不存在，会产生：

```text
KeyError
```

因此通常需要写：

```python
if num not in count:
    count[num] = 0

count[num] += 1
```

而：

```python
defaultdict(int)
```

会自动给不存在的 key 一个默认值。

因为：

```python
int()
```

返回：

```python
0
```

所以：

```python
count[num] += 1
```

可以直接使用。

例如：

```python
from collections import defaultdict

count = defaultdict(int)

count[5] += 1
count[5] += 1

print(count[5])
```

结果：

```text
2
```

------

## 也可以使用普通 `dict`

面试中也非常常见：

```python
class Solution:
    def majorityElement(self, nums):
        count = {}

        for num in nums:
            count[num] = count.get(num, 0) + 1

            if count[num] > len(nums) // 2:
                return num
```

这里：

```python
count.get(num, 0)
```

表示：

> 如果 `num` 存在，则返回对应 value；否则返回 `0`。

这是 Python 哈希表计数问题中非常常见的写法。

------

## 时间与空间复杂度

平均情况下：

```text
时间复杂度：O(n)
空间复杂度：O(n)
```

最坏情况下数组可能有 `O(n)` 个不同元素，因此哈希表最多保存 `O(n)` 个 key。

------

# 解法三：Sorting / 排序

## 思路

如果将数组排序，那么多数元素一定会覆盖数组的中间位置：

```python
len(nums) // 2
```

因此排序之后直接返回中间元素即可。

------

## 为什么中间位置一定是多数元素？

假设：

```text
n = 7
```

多数元素至少出现：

```text
4 次
```

如果多数元素是 `5`，排序后它可能是：

```text
[1, 1, 1, 5, 5, 5, 5]
```

也可能是：

```text
[1, 1, 5, 5, 5, 5, 9]
```

或者：

```text
[5, 5, 5, 5, 8, 9, 10]
```

无论它从哪里开始，由于它占据超过一半的位置，因此一定会覆盖：

```python
n // 2
```

这个索引。

------

## Python 实现

```python
class Solution:
    def majorityElement(self, nums):
        # 原地排序
        nums.sort()

        # 多数元素一定占据中间位置
        return nums[len(nums) // 2]
```

------

## Python 的 `list.sort()`

```python
nums.sort()
```

会直接修改原数组。

例如：

```python
nums = [3, 1, 2]

nums.sort()

print(nums)
```

结果：

```text
[1, 2, 3]
```

如果不希望修改原数组，则可以使用：

```python
sorted_nums = sorted(nums)
```

`sorted()` 会返回一个新的列表。

------

## 时间与空间复杂度

时间复杂度：

```text
O(n log n)
```

Python 的排序实现使用 **Timsort**。

额外空间复杂度并不能简单认为永远是 `O(1)`。Python 的 `list.sort()` 在实际实现中可能使用额外辅助空间，因此面试中更安全的表述是：

```text
空间复杂度取决于排序算法；
若只讨论某些原地排序算法，可达到 O(1)；
Python 内置排序可能使用 O(n) 级别的辅助空间。
```

------

# 解法四：Brute Force / 暴力枚举

## 思路

对于数组中的每一个数字：

```text
num
```

重新扫描整个数组，统计它出现了多少次。

如果：

```text
count > n // 2
```

说明它就是多数元素。

------

## Python 实现

```python
class Solution:
    def majorityElement(self, nums):
        n = len(nums)

        for num in nums:
            # 对当前 num，再完整扫描一次数组
            count = sum(1 for x in nums if x == num)

            if count > n // 2:
                return num
```

------

## `sum(1 for x in nums if x == num)` 是什么？

这是一个 **生成器表达式（generator expression）**。

```python
1 for x in nums if x == num
```

表示：

> 遍历 `nums`，每找到一个等于 `num` 的元素，就产生一个 `1`。

例如：

```python
nums = [5, 1, 5, 2, 5]
num = 5
```

生成的效果类似：

```text
1, 1, 1
```

然后：

```python
sum(...)
```

得到：

```text
3
```

所以：

```python
sum(1 for x in nums if x == num)
```

本质上是在统计 `num` 出现的次数。

实际上 Python 还可以直接写：

```python
nums.count(num)
```

例如：

```python
count = nums.count(num)
```

------

## 时间与空间复杂度

外层遍历：

```text
O(n)
```

每次又完整扫描数组：

```text
O(n)
```

因此：

```text
时间复杂度：O(n²)
空间复杂度：O(1)
```

这种方法主要用于说明最基础的思路，不推荐作为面试最终答案。

------

# 解法五：Bit Manipulation / 位运算

## 思路

整数可以表示成二进制。

例如：

```text
5 = 0101
```

由于多数元素出现超过一半次数，因此对于多数元素的每一位：

> 如果这一位是 `1`，那么数组中超过一半的数字在这一位也一定是 `1`。

因此我们可以分别统计每个二进制位上有多少个 `1`。

如果某一位：

```text
出现 1 的次数 > n / 2
```

那么多数元素对应的这一位一定是 `1`。

------

## Python 实现

题目中的数值范围：

```text
-10^9 <= nums[i] <= 10^9
```

可以使用 32 位有符号整数处理。

```python
class Solution:
    def majorityElement(self, nums):
        n = len(nums)

        # bit[i] 表示所有数字中，
        # 第 i 个二进制位为 1 的数字数量
        bit = [0] * 32

        for num in nums:
            for i in range(32):
                # 取出 num 的第 i 位
                bit[i] += (num >> i) & 1

        result = 0

        for i in range(32):
            if bit[i] > n // 2:
                if i == 31:
                    # 第 31 位是 32 位有符号整数的符号位
                    result -= 1 << i
                else:
                    result |= 1 << i

        return result
```

------

## 位运算解释

### `1 << i`

表示将二进制 `1` 向左移动 `i` 位。

例如：

```python
1 << 0
```

得到：

```text
0001 = 1
1 << 2
```

得到：

```text
0100 = 4
```

因此：

```python
1 << i
```

可以用来表示：

> 第 `i` 位为 `1` 的整数。

------

### `num >> i`

表示将 `num` 向右移动 `i` 位。

例如：

```text
num = 5

5 = 0101
```

如果：

```python
5 >> 2
```

得到：

```text
0001
```

这样就可以把我们想检查的 bit 移动到最低位。

------

### `(num >> i) & 1`

用来获得：

```text
num 的第 i 个二进制位
```

因为：

```text
x & 1
```

只会保留最低位。

结果只可能是：

```text
0
```

或者：

```text
1
```

------

### `result |= 1 << i`

这里：

```python
|=
```

是按位 OR 后赋值。

等价于：

```python
result = result | (1 << i)
```

作用是：

> 将 `result` 的第 `i` 位设置为 `1`。

------

## 为什么需要特别处理第 31 位？

32 位有符号整数通常使用 **二进制补码（Two's Complement）** 表示负数。

其中最高位：

```text
bit 31
```

代表符号位，其权重不是：

```text
+2^31
```

而是：

```text
-2^31
```

所以代码写成：

```python
if i == 31:
    result -= 1 << i
```

而不是：

```python
result |= 1 << i
```

这样才能正确构造负数。

------

## 时间与空间复杂度

我们对每一个数字检查固定的 32 个 bit：

```text
时间复杂度：O(32n) = O(n)
```

额外数组固定只有 32 个元素：

```text
空间复杂度：O(32) = O(1)
```

这里 `32` 是常数，因此在渐进复杂度中被忽略。

虽然复杂度满足：

```text
O(n) time
O(1) space
```

但相比 Boyer-Moore，这种方法实现更复杂，在本题中通常不是首选。

------

# 解法六：Randomization / 随机化

## 思路

由于多数元素出现次数超过数组的一半，因此随机从数组中选择一个元素：

```python
random.choice(nums)
```

选中多数元素的概率一定：

```text
> 50%
```

如果随机选到了多数元素，扫描数组验证即可。

否则继续随机。

------

## Python 实现

```python
import random


class Solution:
    def majorityElement(self, nums):
        n = len(nums)

        while True:
            # 从 nums 中随机选择一个元素
            candidate = random.choice(nums)

            # 验证它是否为多数元素
            if nums.count(candidate) > n // 2:
                return candidate
```

------

## `random.choice()`

需要：

```python
import random
```

然后：

```python
random.choice(nums)
```

会随机返回序列中的一个元素。

例如：

```python
nums = [1, 2, 3]

candidate = random.choice(nums)
```

`candidate` 可能是：

```text
1
2
3
```

中的任意一个。

------

## `nums.count(candidate)`

Python `list.count()` 会遍历整个列表，统计某个元素出现了多少次。

例如：

```python
nums = [2, 1, 2, 3, 2]

nums.count(2)
```

返回：

```text
3
```

因此它的时间复杂度是：

```text
O(n)
```

------

## 为什么期望时间是 O(n)？

假设随机一次选中多数元素的概率：

```text
p > 1/2
```

那么平均尝试次数约为：

```text
1 / p < 2
```

也就是说，期望情况下只需要常数次随机尝试。

每次验证需要：

```text
O(n)
```

所以：

```text
期望时间复杂度：O(n)
空间复杂度：O(1)
```

注意这里是：

```text
Expected O(n)
```

而不是严格的最坏情况 `O(n)`。

理论上随机算法可能连续很多次都没有选中多数元素，因此最坏情况下没有确定的有限运行次数上界。

所以面试中一般仍然优先使用 Boyer-Moore。

------

# 各种方法对比

| 方法             | 时间复杂度   | 空间复杂度     | 推荐程度 |
| ---------------- | ------------ | -------------- | -------- |
| Boyer-Moore      | `O(n)`       | `O(1)`         | ★★★★★    |
| Hash Map         | `O(n)`       | `O(n)`         | ★★★★☆    |
| Sorting          | `O(n log n)` | 取决于排序实现 | ★★★☆☆    |
| Brute Force      | `O(n²)`      | `O(1)`         | ★☆☆☆☆    |
| Bit Manipulation | `O(n)`       | `O(1)`         | ★★☆☆☆    |
| Randomization    | 期望 `O(n)`  | `O(1)`         | ★★☆☆☆    |

对于这道题，面试时建议至少熟练掌握：

```text
1. Hash Map
2. Boyer-Moore Voting Algorithm
```

其中 Boyer-Moore 是最值得重点理解的方案。

------

# 常见错误

## 1. 混淆 `>` 和 `>=`

多数元素的定义是：

```text
出现次数严格超过 n / 2
```

也就是：

```python
count > n // 2
```

而不是简单理解成：

```python
count >= n // 2
```

例如：

```text
nums = [1, 1, 2, 2]
```

此时：

```text
n = 4
n // 2 = 2
```

`1` 和 `2` 都出现：

```text
2 次
```

但是：

```text
2 不是 > 2
```

所以不存在多数元素。

如果错误地使用：

```python
count >= n // 2
```

就会把出现恰好一半次数的元素错误地判断成多数元素。

当然，本题保证多数元素一定存在，但理解定义本身仍然非常重要。

------

## 2. Python 中 `/` 和 `//` 的区别

Python 3 中：

```python
/
```

表示浮点除法。

例如：

```python
5 / 2
```

结果：

```text
2.5
```

而：

```python
//
```

表示向下取整的整数除法：

```python
5 // 2
```

结果：

```text
2
```

题目中的：

```text
⌊n / 2⌋
```

通常写成：

```python
n // 2
```

例如：

```python
if count > n // 2:
```

不过如果只是比较：

```python
count > n / 2
```

Python 本身也可以正确比较整数和浮点数，并不会产生类型错误。

使用：

```python
n // 2
```

主要是因为它更直接地对应题目中的：

```text
floor(n / 2)
```

------

## 3. 忘记 Boyer-Moore 的题目前提

如果题目保证：

```text
majority element always exists
```

那么：

```python
return candidate
```

即可。

如果题目没有这个保证，最终的 `candidate` 只是：

```text
potential majority element
```

还需要第二次遍历进行验证。

这是 Boyer-Moore 类型题目中非常常见的追问。

------

## 4. 把 Boyer-Moore 的 `count` 理解成真实出现次数

例如：

```python
count += 1 if num == candidate else -1
```

这里的 `count` **并不是 candidate 在数组中的真实出现次数**。

它代表的是：

```text
candidate 与其他元素经过抵消后的净票数
```

因此不能在算法结束后通过 `count` 判断 candidate 实际出现了多少次。

------

# 面试中的推荐回答方式

如果面试官先问最直观的方法，可以从 Hash Map 开始：

```text
我可以使用哈希表统计每个数字出现的次数。
这样时间复杂度是 O(n)，但是需要 O(n) 的额外空间。
```

如果面试官继续问：

```text
Can you do it in O(1) extra space?
```

就引出 Boyer-Moore：

```text
因为多数元素出现次数超过整个数组的一半，
所以可以让不同元素两两抵消。

维护一个 candidate 和 count：
遇到 candidate 时 count + 1，
遇到其他元素时 count - 1。

当 count 变成 0 时重新选择 candidate。

由于多数元素的数量比所有其他元素都有数量优势，
最终剩下的 candidate 一定是多数元素。
```

对应代码只需要：

```python
class Solution:
    def majorityElement(self, nums):
        candidate = None
        count = 0

        for num in nums:
            if count == 0:
                candidate = num

            count += 1 if num == candidate else -1

        return candidate
```

最终：

```text
Time:  O(n)
Space: O(1)
```

这是这道题最理想的面试答案。
