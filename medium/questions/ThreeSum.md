# 三数之和（3Sum）

## 题目描述

给定一个整数数组 `nums`，请返回所有满足以下条件的三元组：

```text
[nums[i], nums[j], nums[k]]
```

其中：

```text
nums[i] + nums[j] + nums[k] == 0
```

并且下标：

```text
i、j、k
```

两两不同。

最终结果中**不能包含重复的三元组**。

返回的三元组顺序以及结果列表中的顺序均不限。

---

## 示例 1

```text
输入：
nums = [-1, 0, 1, 2, -1, -4]

输出：
[[-1, -1, 2], [-1, 0, 1]]
```

解释：

```text
nums[0] + nums[1] + nums[2]
= -1 + 0 + 1
= 0
```

以及：

```text
nums[0] + nums[3] + nums[4]
= -1 + 2 + (-1)
= 0
```

虽然可能有不同的下标组合得到相同的数值三元组，但最终只保留不同的三元组：

```text
[-1, -1, 2]
[-1, 0, 1]
```

---

## 示例 2

```text
输入：
nums = [0, 1, 1]

输出：
[]
```

不存在三个数之和等于 `0`。

---

## 示例 3

```text
输入：
nums = [0, 0, 0]

输出：
[[0, 0, 0]]
```

唯一的三元组之和为：

```text
0 + 0 + 0 = 0
```

---

## 约束

- `3 <= nums.length <= 3000`
- `-10^5 <= nums[i] <= 10^5`

---

# 一、暴力枚举

## 核心思路

最直接的方法是枚举所有可能的三元组。

由于三个下标必须不同，可以直接枚举：

```text
i < j < k
```

这样天然保证三个下标不重复。

对于每一组三个位置：

```python
nums[i] + nums[j] + nums[k]
```

如果结果等于 `0`，就说明找到了一个合法三元组。

然而，数组中可能存在重复数字，因此不同的下标组合可能得到相同的三元组。

例如：

```text
nums = [-1, -1, 0, 1, 1]
```

可能通过不同的下标得到：

```text
[-1, 0, 1]
```

多次。

因此可以使用：

```python
set
```

保存结果，从而自动去重。

---

## 为什么先排序？

原解法在枚举之前执行：

```python
nums.sort()
```

排序以后，同一个三元组中的三个数字天然按照非递减顺序排列。

例如不会同时出现：

```text
(-1, 0, 1)
(0, -1, 1)
(1, 0, -1)
```

而都会统一表示为：

```text
(-1, 0, 1)
```

这样非常方便使用集合去重。

---

## 算法步骤

1. 对 `nums` 排序。
2. 创建集合 `res` 保存唯一三元组。
3. 使用三层循环枚举：
   - `i`
   - `j > i`
   - `k > j`
4. 如果：

```python
nums[i] + nums[j] + nums[k] == 0
```

则把：

```python
(nums[i], nums[j], nums[k])
```

加入集合。
5. 最后把 tuple 转换回 list。
6. 返回结果。

---

## Python 实现

```python
from typing import List


class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        # 排序后，相同三元组会拥有统一的元素顺序
        nums.sort()

        # set 用于自动去除重复三元组
        res = set()

        n = len(nums)

        # 枚举第一个数
        for i in range(n):
            # 枚举第二个数
            for j in range(i + 1, n):
                # 枚举第三个数
                for k in range(j + 1, n):
                    if nums[i] + nums[j] + nums[k] == 0:
                        # list 不能放进 set，因此转换为 tuple
                        res.add((nums[i], nums[j], nums[k]))

        # 题目要求返回 List[List[int]]
        return [list(triplet) for triplet in res]
```

---

## Python 数据结构说明：`set`

Python 的 `set` 是集合类型，具有两个重要特点：

1. 元素唯一；
2. 平均情况下插入和查找都是 `O(1)`。

例如：

```python
s = set()

s.add((1, 2, 3))
s.add((1, 2, 3))

print(s)
```

结果仍然只有：

```text
{(1, 2, 3)}
```

因此非常适合用于去重。

---

## 为什么要使用 `tuple`？

下面的代码是不允许的：

```python
res.add([1, 2, 3])
```

因为 Python 的 `list` 是可变对象，因此不可哈希，不能作为 `set` 的元素。

而：

```python
tuple
```

是不可变对象，可以被哈希。

所以需要写：

```python
res.add((1, 2, 3))
```

最后再转换回：

```python
list
```

---

## 复杂度分析

设数组长度为 `n`，最终不同三元组数量为 `m`。

### 时间复杂度

三层循环：

```text
O(n^3)
```

排序需要：

```text
O(n log n)
```

因此总时间复杂度由三层循环主导：

```text
O(n^3)
```

### 空间复杂度

如果不计算输出：

```text
O(m)
```

用于集合保存不同三元组。

最坏情况下：

```text
m = O(n^2)
```

因此：

```text
O(n^2)
```

另外排序算法本身可能使用额外空间，具体取决于实现。

---

## 暴力法的问题

本题：

```text
n <= 3000
```

如果使用：

```text
O(n^3)
```

最坏情况下操作数量大约是：

```text
3000^3
= 27,000,000,000
```

也就是数百亿级别。

因此暴力法基本无法接受。

---

# 二、哈希表

## 核心思路

三数之和可以看成：

```text
a + b + c = 0
```

如果我们已经确定：

```text
a
b
```

那么第三个数一定是：

```text
c = -(a + b)
```

因此没有必要再使用第三层循环。

我们可以用哈希表记录每个数字当前还剩多少个，然后在确定前两个数字后直接查询：

```python
target = -(nums[i] + nums[j])
```

是否仍然存在。

这样可以把时间复杂度从：

```text
O(n^3)
```

降低到：

```text
O(n^2)
```

---

# 为什么使用“频率表”而不仅仅是集合？

因为数组中可能需要重复使用相同的数值，但必须对应不同的下标。

例如：

```text
[-1, -1, 2]
```

需要两个不同位置上的 `-1`。

如果只记录：

```text
-1 是否存在
```

就无法判断到底还有几个 `-1` 可以使用。

因此更准确的方法是记录：

```text
数字 -> 剩余出现次数
```

例如：

```python
count[-1] = 2
count[2] = 1
```

---

## 算法步骤

1. 对数组排序。
2. 使用哈希表统计每个数字出现次数。
3. 枚举第一个数字 `nums[i]`：
   - 先将它的可用次数减一；
   - 跳过重复的第一个数字。
4. 枚举第二个数字 `nums[j]`：
   - 将它的可用次数减一；
   - 跳过重复的第二个数字。
5. 计算：

```python
target = -(nums[i] + nums[j])
```

6. 如果：

```python
count[target] > 0
```

说明数组后面仍然存在一个可用的第三个数字。
7. 保存结果。
8. 每轮 `i` 结束后恢复第二层循环中减少掉的计数。

---

## Python 实现

```python
from typing import List
from collections import defaultdict


class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums.sort()

        # count[x] 表示数字 x 当前还有多少个可用
        count = defaultdict(int)

        for num in nums:
            count[num] += 1

        res = []

        for i in range(len(nums)):
            # 当前 nums[i] 已经作为第一个数使用，
            # 所以从剩余可用数量中移除
            count[nums[i]] -= 1

            # 跳过重复的第一个数字
            if i > 0 and nums[i] == nums[i - 1]:
                continue

            for j in range(i + 1, len(nums)):
                # 当前 nums[j] 作为第二个数使用
                count[nums[j]] -= 1

                # 跳过重复的第二个数字
                if j - 1 > i and nums[j] == nums[j - 1]:
                    continue

                # 需要的第三个数字
                target = -(nums[i] + nums[j])

                # target 仍然有剩余，说明可以使用不同下标组成三元组
                if count[target] > 0:
                    res.append([nums[i], nums[j], target])

            # 恢复这一轮 j 遍历中减少掉的计数，
            # 方便下一轮 i 正确使用
            for j in range(i + 1, len(nums)):
                count[nums[j]] += 1

        return res
```

---

## Python 类说明：`defaultdict`

`defaultdict` 来自：

```python
from collections import defaultdict
```

普通字典：

```python
d = {}
```

如果直接读取一个不存在的 key：

```python
d[5]
```

会抛出：

```text
KeyError
```

而：

```python
count = defaultdict(int)
```

表示：

> 如果 key 不存在，就自动使用 `int()` 作为默认值。

而：

```python
int()
```

的结果是：

```text
0
```

因此：

```python
count[5]
```

会自动得到：

```text
0
```

这使得频率统计非常方便：

```python
for num in nums:
    count[num] += 1
```

不需要手动判断：

```python
if num not in count:
    count[num] = 0
```

---

## 复杂度分析

### 时间复杂度

排序：

```text
O(n log n)
```

主要枚举两层：

```text
O(n^2)
```

恢复计数同样在每轮 `i` 中执行一次 `O(n)` 遍历，因此整体仍然是：

```text
O(n^2)
```

### 空间复杂度

频率表最多保存 `n` 个不同数字：

```text
O(n)
```

不计算输出结果。

如果输出也计算在内，则为：

```text
O(n + m)
```

其中 `m` 是结果三元组数量。

---

## 对这种哈希表解法的评价

这种方法可以做到：

```text
O(n^2)
```

但代码明显比双指针复杂。

尤其是需要谨慎维护：

```text
count
```

的减少与恢复。

因此在面试中通常不如“双指针”解法自然。

它更适合帮助理解一个通用思想：

> 当已经确定两个数时，可以通过哈希表在 `O(1)` 平均时间内寻找第三个数。

---

# 三、排序 + 双指针

## 核心思路

这是本题最经典、最推荐的解法。

首先对数组排序。

然后固定第一个数字：

```python
a = nums[i]
```

剩下的问题变成：

> 在 `nums[i+1:]` 中找到两个数，使它们的和等于 `-a`。

也就是经典的：

```text
Two Sum II
```

问题。

由于数组已经排序，可以使用两个指针：

```python
l = i + 1
r = len(nums) - 1
```

计算：

```python
threeSum = a + nums[l] + nums[r]
```

然后根据结果移动指针。

---

# 为什么排序后可以使用双指针？

因为排序后：

```text
nums[l]
```

随着 `l` 向右移动，只会变大或保持不变。

而：

```text
nums[r]
```

随着 `r` 向左移动，只会变小或保持不变。

因此：

### 如果当前和太小

```python
threeSum < 0
```

我们需要让总和变大。

所以移动：

```python
l += 1
```

因为左指针右移后，数字会变大。

---

### 如果当前和太大

```python
threeSum > 0
```

我们需要让总和变小。

所以移动：

```python
r -= 1
```

因为右指针左移后，数字会变小。

---

### 如果刚好等于 0

```python
threeSum == 0
```

说明找到了答案。

记录当前三元组，然后两个指针都向中间移动，并跳过重复数字。

---

# 算法步骤

1. 对 `nums` 排序。
2. 枚举第一个数字 `nums[i]`。
3. 如果：

```python
nums[i] > 0
```

直接结束循环。
4. 如果当前第一个数字与上一个相同，跳过。
5. 设置：

```python
l = i + 1
r = len(nums) - 1
```

6. 当：

```python
l < r
```

时：
   - 计算三个数之和。
   - 和太小：`l += 1`
   - 和太大：`r -= 1`
   - 和等于 `0`：
     - 保存结果；
     - 左右指针都移动；
     - 跳过重复值。
7. 返回结果。

---

# Python 实现

```python
from typing import List


class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        res = []

        # 双指针依赖有序数组
        nums.sort()

        for i, a in enumerate(nums):
            # 如果第一个数字已经大于 0，
            # 后面的数字都 >= a > 0，
            # 三个数不可能再加出 0
            if a > 0:
                break

            # 跳过重复的第一个数字，
            # 防止生成相同三元组
            if i > 0 and a == nums[i - 1]:
                continue

            # 在 i 的右侧寻找另外两个数字
            l = i + 1
            r = len(nums) - 1

            while l < r:
                threeSum = a + nums[l] + nums[r]

                if threeSum < 0:
                    # 当前和太小，需要增大
                    l += 1

                elif threeSum > 0:
                    # 当前和太大，需要减小
                    r -= 1

                else:
                    # 找到合法三元组
                    res.append([a, nums[l], nums[r]])

                    # 两边同时向中间移动
                    l += 1
                    r -= 1

                    # 跳过重复的左侧数字
                    #
                    # 条件顺序写成 l < r 在前，
                    # 可以先确认指针仍然有效，再访问 nums[l]
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1

                    # 也可以显式跳过右侧重复数字。
                    # 不是绝对必须，但逻辑更对称、也更易读。
                    while l < r and nums[r] == nums[r + 1]:
                        r -= 1

        return res
```

---

# 为什么固定第一个数后，本质上变成 Two Sum？

原条件是：

```text
a + b + c = 0
```

固定：

```text
a = nums[i]
```

以后：

```text
b + c = -a
```

于是问题变成：

> 在排序数组剩余区间中找到两个数，使它们之和等于 `-a`。

这就是为什么 3Sum 经常被理解为：

```text
排序
+
固定一个数
+
双指针解决 Two Sum
```

这是非常重要的面试模式。

类似的思路还可以推广到：

```text
4Sum
k-Sum
```

---

# 为什么 `a > 0` 时可以直接 `break`？

数组已经排序。

如果：

```python
a = nums[i] > 0
```

那么后面的元素满足：

```text
nums[l] >= a > 0
nums[r] >= a > 0
```

因此：

```text
a + nums[l] + nums[r] > 0
```

不可能等于 `0`。

所以可以直接：

```python
break
```

结束整个外层循环。

---

# 为什么不能写成 `a >= 0`？

因为：

```text
[0, 0, 0]
```

本身就是合法答案。

如果写：

```python
if a >= 0:
    break
```

当：

```text
a = 0
```

时就会提前退出，从而漏掉：

```text
[0, 0, 0]
```

因此必须是：

```python
if a > 0:
    break
```

---

# 去重是这道题的核心难点

3Sum 真正容易出错的地方通常不是寻找三个数，而是：

> 如何避免重复三元组？

排序之后，去重就变得非常容易。

需要处理两种重复：

1. 第一个数字重复；
2. 找到答案后，左右指针遇到重复数字。

---

## 1. 跳过重复的第一个数字

例如：

```text
[-1, -1, 0, 1]
```

第一次固定：

```text
a = -1
```

已经搜索了所有以 `-1` 开头的三元组。

第二次又遇到：

```text
a = -1
```

如果重新搜索，就很可能生成完全相同的结果。

因此：

```python
if i > 0 and nums[i] == nums[i - 1]:
    continue
```

直接跳过。

---

## 2. 找到答案后跳过重复的左右值

例如：

```text
[-2, 0, 0, 0, 2, 2]
```

如果找到：

```text
[-2, 0, 2]
```

之后只简单执行：

```python
l += 1
r -= 1
```

可能又遇到：

```text
0
2
```

从而重复加入：

```text
[-2, 0, 2]
```

所以找到答案以后，应跳过连续的相同数字。

例如：

```python
while l < r and nums[l] == nums[l - 1]:
    l += 1
```

右侧也可以写：

```python
while l < r and nums[r] == nums[r + 1]:
    r -= 1
```

---

# 为什么原答案只跳过左侧重复值也能工作？

原代码只写了：

```python
while l < r and nums[l] == nums[l - 1]:
    l += 1
```

没有显式跳过右侧重复值。

在这道题中，这种写法仍然可以正确去重，因为对于：

```text
固定的 a
固定的 nums[l]
```

满足：

```text
a + nums[l] + nums[r] = 0
```

的 `nums[r]` 数值是唯一确定的。

跳过重复的 `nums[l]` 已经足以阻止同一三元组再次加入。

不过在面试代码中，同时跳过左右重复值：

```python
while l < r and nums[l] == nums[l - 1]:
    l += 1

while l < r and nums[r] == nums[r + 1]:
    r -= 1
```

通常更直观，也更容易解释。

---

# 示例演示

假设：

```text
nums = [-1, 0, 1, 2, -1, -4]
```

排序后：

```text
[-4, -1, -1, 0, 1, 2]
```

---

## 第一轮

固定：

```text
a = -4
```

左右指针：

```text
l -> -1
r -> 2
```

当前和：

```text
-4 + (-1) + 2 = -3
```

太小，所以：

```python
l += 1
```

继续搜索。

最终不会找到答案。

---

## 第二轮

固定：

```text
a = -1
```

此时：

```text
l -> -1
r -> 2
```

和为：

```text
-1 + (-1) + 2 = 0
```

找到：

```text
[-1, -1, 2]
```

移动两个指针。

接着可能得到：

```text
l -> 0
r -> 1
```

和为：

```text
-1 + 0 + 1 = 0
```

找到：

```text
[-1, 0, 1]
```

---

## 第三轮

下一个 `i` 对应的值仍然是：

```text
-1
```

由于：

```python
nums[i] == nums[i - 1]
```

直接跳过。

这样就避免产生重复三元组。

最终结果：

```text
[
    [-1, -1, 2],
    [-1, 0, 1]
]
```

---

# 复杂度分析

## 时间复杂度

排序：

```text
O(n log n)
```

外层固定一个数字：

```text
O(n)
```

每一轮内部双指针最多移动：

```text
O(n)
```

因此主要复杂度为：

```text
O(n^2)
```

由于：

```text
O(n^2) > O(n log n)
```

所以整体：

```text
O(n^2)
```

---

## 空间复杂度

如果不计算输出结果：

```text
O(1)
```

额外辅助空间。

不过：

```python
nums.sort()
```

底层排序算法本身可能使用额外空间，因此严格来说应根据语言和排序实现计算。

如果把输出结果算在内，则还需要：

```text
O(m)
```

其中：

```text
m = 不同合法三元组的总数量
```

最坏情况下：

```text
m = O(n^2)
```

---

# Python 函数说明：`enumerate`

代码：

```python
for i, a in enumerate(nums):
```

等价于同时获得：

- 当前元素下标 `i`
- 当前元素值 `a`

例如：

```python
nums = [10, 20, 30]

for i, value in enumerate(nums):
    print(i, value)
```

输出：

```text
0 10
1 20
2 30
```

因此：

```python
for i, a in enumerate(nums):
```

比：

```python
for i in range(len(nums)):
    a = nums[i]
```

更加简洁。

---

# 常见错误

## 1. 忘记排序

双指针算法依赖一个关键性质：

```text
左指针右移 -> 数字不会变小
右指针左移 -> 数字不会变大
```

只有在数组有序时，这个性质才成立。

因此必须先：

```python
nums.sort()
```

否则：

```python
threeSum < 0
```

时移动 `l` 并不能保证总和变大。

---

## 2. 忘记跳过重复的第一个数字

错误写法：

```python
for i, a in enumerate(nums):
    l, r = i + 1, len(nums) - 1
```

如果数组中存在：

```text
[-1, -1, ...]
```

那么会对同一个 `a = -1` 搜索多次，从而产生重复结果。

正确处理：

```python
if i > 0 and nums[i] == nums[i - 1]:
    continue
```

---

## 3. 找到答案后没有处理重复值

找到：

```python
threeSum == 0
```

以后，如果只加入结果但不合理移动或去重，就可能不断添加相同三元组。

正确做法至少需要：

```python
l += 1
r -= 1
```

并跳过连续重复值。

---

## 4. 错误地使用 `a >= 0` 提前结束

错误：

```python
if a >= 0:
    break
```

这会漏掉：

```text
[0, 0, 0]
```

正确：

```python
if a > 0:
    break
```

---

## 5. 找到答案后只移动一个指针

假设：

```python
threeSum == 0
```

如果只移动：

```python
l += 1
```

而不处理 `r`，虽然通过额外的逻辑仍可能最终得到正确答案，但更容易重复搜索相同组合。

标准做法是：

```python
l += 1
r -= 1
```

因为当前：

```text
nums[l]
nums[r]
```

这一对已经使用过。

---

## 6. 去重条件访问顺序不够安全

更推荐写：

```python
while l < r and nums[l] == nums[l - 1]:
    l += 1
```

而不是：

```python
while nums[l] == nums[l - 1] and l < r:
    l += 1
```

原因是 Python 的：

```python
and
```

采用**从左到右短路求值**。

把边界判断：

```python
l < r
```

放在前面，可以先确认指针处于合法搜索区间，再访问数组元素。

这是双指针题中值得养成的编码习惯。

---

# 解法对比

| 方法 | 时间复杂度 | 额外空间复杂度 | 面试推荐程度 |
|---|---:|---:|---|
| 暴力枚举 | `O(n^3)` | `O(m)` | 不推荐 |
| 哈希表 | `O(n^2)` | `O(n)` | 可以掌握 |
| 排序 + 双指针 | `O(n^2)` | `O(1)`* | **最推荐** |

> `*` 不计算输出和排序算法自身使用的额外空间。

---

# 面试中最值得掌握的思维模式

这道题可以记成：

```text
3Sum
=
固定一个数
+
在剩余有序数组中做 Two Sum
```

具体来说：

```text
a + b + c = 0
```

固定：

```text
a
```

得到：

```text
b + c = -a
```

然后使用双指针寻找：

```text
b
c
```

---

# 推荐记忆模板

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        res = []

        for i, a in enumerate(nums):
            if a > 0:
                break

            if i > 0 and a == nums[i - 1]:
                continue

            l, r = i + 1, len(nums) - 1

            while l < r:
                total = a + nums[l] + nums[r]

                if total < 0:
                    l += 1
                elif total > 0:
                    r -= 1
                else:
                    res.append([a, nums[l], nums[r]])
                    l += 1
                    r -= 1

                    while l < r and nums[l] == nums[l - 1]:
                        l += 1

                    while l < r and nums[r] == nums[r + 1]:
                        r -= 1

        return res
```

核心记忆点可以概括为：

```text
排序
→ 固定第一个数
→ 两端夹逼
→ 和小左移
→ 和大右移
→ 找到答案后跳过重复值
```

---

# 进一步总结：这道题为什么值得掌握？

3Sum 不只是考一道固定题型，它包含了几个非常常见的面试技巧：

```text
1. 排序后利用单调性
2. 双指针减少搜索空间
3. 固定一个变量，把高维问题降维
4. 去重
5. 利用提前终止条件优化搜索
```

尤其重要的是：

```text
k-Sum -> 固定一个数 -> (k-1)-Sum
```

这种降维思想。

例如：

```text
4Sum
```

可以固定一个数字，把问题变成：

```text
3Sum
```

继续固定以后，又可以变成：

```text
Two Sum
```

因此，真正值得记住的不只是 3Sum 的代码，而是：

> **排序 + 固定元素 + 双指针 + 去重**

这一整套通用解题模式。
