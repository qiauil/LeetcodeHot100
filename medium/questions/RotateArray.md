# Rotate Array — 旋转数组

给定一个整数数组 `nums`，将数组向右旋转 `k` 步，其中 `k` 是非负整数。

## 示例 1

```text
输入：nums = [1,2,3,4,5,6,7,8], k = 4

输出：[5,6,7,8,1,2,3,4]
```

解释：

```text
向右旋转 1 步：[8,1,2,3,4,5,6,7]
向右旋转 2 步：[7,8,1,2,3,4,5,6]
向右旋转 3 步：[6,7,8,1,2,3,4,5]
向右旋转 4 步：[5,6,7,8,1,2,3,4]
```

## 示例 2

```text
输入：nums = [1000,2,4,-3], k = 2

输出：[4,-3,1000,2]
```

解释：

```text
向右旋转 1 步：[-3,1000,2,4]
向右旋转 2 步：[4,-3,1000,2]
```

## 约束条件

- `1 <= nums.length <= 100,000`
- `-(2^31) <= nums[i] <= 2^31 - 1`
- `0 <= k <= 100,000`

------

## 前置知识

在解决这道题之前，最好熟悉以下几个概念：

- **数组原地操作（In-place Array Manipulation）**：通过交换或移动元素直接修改原数组，而不是创建完整的新数组。
- **模运算（Modular Arithmetic）**：使用 `%` 处理 `k` 大于数组长度的情况。
- **双指针反转数组**：通过左右两个指针向中间移动，实现子数组原地反转。
- **环状遍历（Cyclic Traversal）**：按照元素最终位置不断跳转，并沿着一个“环”移动元素。

其中最重要的一个预处理是：

```python
k %= n
```

因为数组长度为 `n` 时：

```text
向右旋转 n 次 = 完全没有变化
```

例如：

```text
nums = [1, 2, 3, 4]

旋转 4 次 = [1, 2, 3, 4]
旋转 5 次 = 旋转 1 次
旋转 9 次 = 旋转 1 次
```

所以真正有效的旋转次数始终是：

```python
k % n
```

------

# 方法一：暴力模拟

## 思路

最直接的方法就是完全按照题目描述进行模拟。

每次向右旋转一步：

1. 保存最后一个元素。
2. 将其他元素全部向右移动一位。
3. 将原来的最后一个元素放到数组开头。

重复这个过程 `k` 次即可。

例如：

```text
[1,2,3,4,5]

保存 5

右移：
[1,1,2,3,4]

再把 5 放到开头：
[5,1,2,3,4]
```

这种方法非常容易想到，也很适合作为面试时最初的 brute force 解法，但效率较低。

## 算法步骤

1. 令 `n = len(nums)`。
2. 使用 `k %= n` 得到真正需要旋转的次数。
3. 重复 `k` 次：
   - 保存 `nums[n - 1]`。
   - 从右向左移动数组元素。
   - 将保存的元素放到 `nums[0]`。

```python
from typing import List


class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        不返回任何值，直接原地修改 nums。
        """

        n = len(nums)

        # 如果 k >= n，只需要保留真正有效的旋转次数
        k %= n

        while k:
            # 保存当前数组的最后一个元素
            tmp = nums[n - 1]

            # 必须从右向左移动。
            # 如果从左向右，会覆盖掉还没有使用的数据。
            for i in range(n - 1, 0, -1):
                nums[i] = nums[i - 1]

            # 原来的最后一个元素移动到数组开头
            nums[0] = tmp

            k -= 1
```

## 为什么必须从右向左移动？

假设：

```text
nums = [1,2,3,4]
```

如果从左向右：

```python
nums[1] = nums[0]
```

数组变成：

```text
[1,1,3,4]
```

这时原来的 `2` 已经丢失了。

因此，在原地向右移动数组时，要从数组末尾开始：

```python
for i in range(n - 1, 0, -1):
```

这里的：

```python
range(start, stop, step)
```

表示从 `start` 开始，每次增加 `step`，直到到达 `stop` 之前。

所以：

```python
range(n - 1, 0, -1)
```

会依次产生：

```text
n - 1, n - 2, ..., 2, 1
```

## 复杂度

- 时间复杂度：`O(nk)`
- 空间复杂度：`O(1)`

每旋转一次需要移动大约 `n` 个元素，一共旋转 `k` 次。

因此，当：

```text
n = 100000
k = 100000
```

时，这种方法会非常慢。

------

# 方法二：使用额外数组

## 思路

与其一步一步移动，不如直接计算：

> 每个元素最终应该去哪里。

假设一个元素原来位于索引 `i`。

向右旋转 `k` 步以后，它的新位置为：

```python
(i + k) % n
```

例如：

```text
nums = [1,2,3,4,5]
k = 2
```

索引：

```text
0  1  2  3  4
```

新位置：

```text
0 -> (0 + 2) % 5 = 2
1 -> (1 + 2) % 5 = 3
2 -> (2 + 2) % 5 = 4
3 -> (3 + 2) % 5 = 0
4 -> (4 + 2) % 5 = 1
```

于是：

```text
1 -> index 2
2 -> index 3
3 -> index 4
4 -> index 0
5 -> index 1
```

最终：

```text
[4,5,1,2,3]
```

这里 `% n` 非常重要，因为数组索引到达末尾以后需要重新绕回索引 `0`。

## 算法步骤

1. 创建长度为 `n` 的临时数组 `tmp`。

2. 对每个索引 `i`：

   ```python
   tmp[(i + k) % n] = nums[i]
   ```

3. 将 `tmp` 的内容写回 `nums`。

```python
from typing import List


class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        不返回任何值，直接修改 nums。
        """

        n = len(nums)
        k %= n

        # 创建一个长度为 n 的临时数组
        tmp = [0] * n

        for i in range(n):
            # nums[i] 向右移动 k 位后的最终位置
            new_index = (i + k) % n
            tmp[new_index] = nums[i]

        # 将 tmp 的全部内容写回原数组
        nums[:] = tmp
```

## Python：`nums[:] = tmp`

这里值得特别注意。

如果写：

```python
nums = tmp
```

只是让局部变量 `nums` 指向另一个列表，并没有真正修改调用者传进来的原数组。

而：

```python
nums[:] = tmp
```

表示：

> 替换 `nums` 中的所有元素。

因此原来的 `list` 对象仍然存在，只是内容发生了变化。

例如：

```python
a = [1, 2, 3]
b = a

a[:] = [4, 5, 6]

print(b)
```

结果：

```text
[4, 5, 6]
```

但如果：

```python
a = [1, 2, 3]
b = a

a = [4, 5, 6]

print(b)
```

结果仍然是：

```text
[1, 2, 3]
```

这也是 LeetCode 中很多要求 **modify in-place** 的题目需要特别注意的 Python 细节。

## 复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

相比暴力方法已经快很多，但需要创建一个完整的新数组。

------

# 方法三：环状替换 / Cyclic Traversal

## 思路

方法二告诉我们，一个位于索引 `i` 的元素最终应该移动到：

```python
(i + k) % n
```

那么是否可以不创建额外数组，而是直接把元素移动到正确位置？

可以。

我们从一个位置开始：

```text
current
```

把这个位置的元素移动到：

```python
(current + k) % n
```

但是那个位置原本也有一个元素，所以必须先把被覆盖的元素保存下来。

然后继续把这个被替换出来的元素送到它自己的目标位置。

不断重复，就形成一个环。

例如：

```text
nums = [1,2,3,4,5,6]
k = 2
```

从索引 `0` 开始：

```text
0 -> 2 -> 4 -> 0
```

这是一个环。

对应元素：

```text
1 -> index 2
3 -> index 4
5 -> index 0
```

但这个环只处理了：

```text
0, 2, 4
```

还有：

```text
1, 3, 5
```

于是从索引 `1` 再开始一个环：

```text
1 -> 3 -> 5 -> 1
```

最终处理完所有元素。

------

## 为什么有时会存在多个环？

这取决于：

```text
n 和 k 的最大公约数
```

环的数量实际上等于：

```text
gcd(n, k)
```

例如：

```text
n = 6
k = 2
```

因为：

```text
gcd(6, 2) = 2
```

所以存在两个环：

```text
0 -> 2 -> 4 -> 0
1 -> 3 -> 5 -> 1
```

而如果：

```text
n = 7
k = 3
```

因为：

```text
gcd(7, 3) = 1
```

整个数组会形成一个大环。

面试中通常不需要证明这一点，但知道它有助于理解为什么不能只从索引 `0` 处理一次。

------

## 算法步骤

1. 计算 `k %= n`。

2. 使用 `count` 记录已经移动了多少个元素。

3. 从 `start = 0` 开始一个环。

4. 保存当前位置的元素到 `prev`。

5. 计算：

   ```python
   next_idx = (current + k) % n
   ```

6. 将 `prev` 放到 `next_idx`，同时保存原本位于 `next_idx` 的元素。

7. 不断继续，直到回到 `start`。

8. 如果还没有处理完全部元素，则从下一个 `start` 开始新的环。

```python
from typing import List


class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        不返回任何值，直接原地修改 nums。
        """

        n = len(nums)
        k %= n

        # count：已经移动到正确位置的元素数量
        count = 0

        # start：当前环的起始位置
        start = 0

        while count < n:
            current = start

            # prev 保存当前准备移动的元素
            prev = nums[start]

            while True:
                # 当前元素应该移动到的位置
                next_idx = (current + k) % n

                # 将 prev 放到目标位置，
                # 同时把目标位置原来的元素保存到 prev
                nums[next_idx], prev = prev, nums[next_idx]

                current = next_idx
                count += 1

                # 回到起始位置，说明当前环已经完成
                if current == start:
                    break

            # 当前环结束，尝试从下一个位置开始
            start += 1
```

## Python：多重赋值

这里：

```python
nums[next_idx], prev = prev, nums[next_idx]
```

相当于：

```python
tmp = nums[next_idx]
nums[next_idx] = prev
prev = tmp
```

Python 在执行赋值之前，会先计算右边所有表达式，因此不会出现覆盖问题。

这也是 Python 中非常常见的交换写法：

```python
a, b = b, a
```

------

## 复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

虽然有两层 `while`，但并不是 `O(n²)`。

关键在于：

```python
count
```

最多只会增加到 `n`。

也就是说，每一个元素只会被处理一次，因此整体仍然是 `O(n)`。

------

# 方法四：三次反转

这是这道题中非常经典、也非常适合面试的解法。

## 思路

向右旋转 `k` 步，本质上就是：

> 把数组最后 `k` 个元素移动到最前面。

假设：

```text
nums = [1,2,3,4,5,6,7]
k = 3
```

目标是：

```text
[5,6,7,1,2,3,4]
```

可以把数组看成两部分：

```text
A = [1,2,3,4]
B = [5,6,7]
```

我们想得到：

```text
B + A
```

首先整体反转：

```text
[7,6,5,4,3,2,1]
```

现在相当于：

```text
reverse(B) + reverse(A)
```

接下来分别再反转这两部分：

```text
reverse([7,6,5]) = [5,6,7]
reverse([4,3,2,1]) = [1,2,3,4]
```

最终：

```text
[5,6,7,1,2,3,4]
```

也就是说：

```text
reverse(A + B)
= reverse(B) + reverse(A)
```

然后：

```text
reverse(reverse(B)) = B
reverse(reverse(A)) = A
```

因此通过三次反转即可完成旋转。

------

## 算法步骤

1. 计算 `k %= n`。
2. 反转整个数组。
3. 反转前 `k` 个元素。
4. 反转剩余 `n-k` 个元素。

```python
from typing import List


class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        不返回任何值，直接原地修改 nums。
        """

        n = len(nums)
        k %= n

        def reverse(left: int, right: int) -> None:
            """
            原地反转 nums[left:right + 1]
            """
            while left < right:
                nums[left], nums[right] = nums[right], nums[left]

                left += 1
                right -= 1

        # 第一步：反转整个数组
        reverse(0, n - 1)

        # 第二步：反转前 k 个元素
        reverse(0, k - 1)

        # 第三步：反转剩余元素
        reverse(k, n - 1)
```

------

## 过程示例

```text
nums = [1,2,3,4,5,6,7,8]
k = 3
```

第一步，整体反转：

```text
[8,7,6,5,4,3,2,1]
```

第二步，反转前 `3` 个：

```text
[6,7,8,5,4,3,2,1]
```

第三步，反转后面的部分：

```text
[6,7,8,1,2,3,4,5]
```

这正是向右旋转 `3` 步的结果。

------

## 双指针反转

`reverse()` 是一个非常值得记住的模板：

```python
def reverse(left, right):
    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1
```

逻辑就是：

```text
L               R
↓               ↓
1  2  3  4  5  6

交换：

6  2  3  4  5  1
   ↑        ↑
   L        R

继续交换：

6  5  3  4  2  1
      ↑  ↑
      L  R
```

最终得到：

```text
6 5 4 3 2 1
```

这种技巧在数组、字符串以及回文相关问题中非常常见。

------

## `k = 0` 会不会有问题？

假设：

```python
k = 0
```

代码会执行：

```python
reverse(0, -1)
```

但由于：

```python
left = 0
right = -1
```

条件：

```python
left < right
```

一开始就是 `False`，所以不会进行任何操作。

因此原代码仍然可以正常工作。

------

## 复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

三个反转操作分别处理：

```text
n
k
n-k
```

个元素，因此总操作量为：

```text
n + k + (n-k) = 2n
```

忽略常数后仍然是：

```text
O(n)
```

### 面试推荐度

这个方法通常是本题最值得掌握的解法：

```text
时间 O(n)
空间 O(1)
代码简单
逻辑清晰
容易解释
```

相比 Cyclic Traversal，三次反转通常更加容易正确实现。

------

# 方法五：Python 切片一行写法

## 思路

Python 支持非常方便的列表切片。

向右旋转 `k` 步，可以理解为：

```text
最后 k 个元素
+
前 n-k 个元素
```

因此可以直接写：

```python
nums[:] = nums[-k:] + nums[:-k]
```

完整代码：

```python
from typing import List


class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        n = len(nums)
        k %= n

        nums[:] = nums[-k:] + nums[:-k]
```

不过这里存在一个 Python 特有的边界问题。

当：

```python
k = 0
```

时：

```python
nums[-0:]
```

其实等价于：

```python
nums[0:]
```

也就是整个数组。

同时：

```python
nums[:-0]
```

等价于：

```python
nums[:0]
```

即空数组。

所以最终仍然得到原数组：

```python
nums[:] = nums[:] + []
```

结果虽然正确，但理解这个切片行为很重要。

------

## Python 切片说明

假设：

```python
nums = [1,2,3,4,5]
k = 2
```

那么：

```python
nums[-k:]
```

即：

```python
nums[-2:]
```

得到：

```text
[4,5]
```

而：

```python
nums[:-k]
```

即：

```python
nums[:-2]
```

得到：

```text
[1,2,3]
```

拼接：

```python
nums[-k:] + nums[:-k]
```

得到：

```text
[4,5,1,2,3]
```

然后：

```python
nums[:] = ...
```

再把结果写回原数组。

------

## 复杂度

虽然代码只有一行，但它并不是 `O(1)` 空间。

Python 切片：

```python
nums[-k:]
nums[:-k]
```

会创建新的列表，而且：

```python
+
```

也会创建新的列表。

因此：

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

------

# 方法对比

| 方法             | 时间复杂度 | 额外空间 | 是否原地修改   | 面试推荐 |
| ---------------- | ---------- | -------- | -------------- | -------- |
| 暴力模拟         | `O(nk)`    | `O(1)`   | 是             | ★        |
| 额外数组         | `O(n)`     | `O(n)`   | 最终写回原数组 | ★★★      |
| Cyclic Traversal | `O(n)`     | `O(1)`   | 是             | ★★★★     |
| 三次反转         | `O(n)`     | `O(1)`   | 是             | ★★★★★    |
| Python 切片      | `O(n)`     | `O(n)`   | 最终写回原数组 | ★★★      |

------

# 面试时建议的思考路径

如果面试官给出这道题，可以按照下面的思路逐步优化：

```text
1. 最直接：
   每次把最后一个元素移到开头
           ↓
       O(nk)

2. 能不能一次确定每个元素最终的位置？
           ↓
   new_index = (i + k) % n
           ↓
       O(n) 时间
       O(n) 空间

3. 能不能做到 O(1) 额外空间？
           ↓
   Cyclic Replacement
   或
   三次 Reverse

4. 最终推荐：
   Three Reversals
   O(n) time
   O(1) space
```

其中最重要的两个公式/模板值得记住：

```python
# 元素右移 k 位后的最终位置
new_index = (i + k) % n
```

以及：

```python
# 原地反转区间
while left < right:
    nums[left], nums[right] = nums[right], nums[left]
    left += 1
    right -= 1
```

对于这道题，如果面试官明确要求：

```text
Can you solve it in-place with O(1) extra space?
```

优先考虑 **三次反转法**。它和 Cyclic Traversal 都满足要求，但通常更容易写对，也更容易在面试中解释清楚。
