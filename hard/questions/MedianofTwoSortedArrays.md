# 寻找两个正序数组的中位数

给定两个大小分别为 `m` 和 `n` 的整数数组 `nums1` 和 `nums2`，两个数组都已经按照**升序排列**。

请返回这两个数组中所有元素组成的数据集合的**中位数**。

要求算法的时间复杂度为：

$O(\log(m+n))$

------

## 示例 1

```text
输入：nums1 = [1, 2], nums2 = [3]

输出：2.0
```

解释：

合并后的有序数组为：

```text
[1, 2, 3]
```

中间元素是 `2`，因此中位数为：

```text
2
```

------

## 示例 2

```text
输入：nums1 = [1, 3], nums2 = [2, 4]

输出：2.5
```

解释：

合并后的有序数组为：

```text
[1, 2, 3, 4]
```

总共有偶数个元素，因此中位数是中间两个元素 `2` 和 `3` 的平均值：

$\frac{2+3}{2}=2.5$

------

# 解法一：暴力合并 + 排序

## 思路

最直接的方法就是：

1. 将两个数组合并。
2. 对合并后的数组进行排序。
3. 根据数组长度的奇偶性计算中位数。

如果数组总长度为奇数，中间的元素就是中位数。

如果数组总长度为偶数，则取中间两个元素的平均值。

这种方法非常容易理解，但它完全没有利用题目中非常重要的条件：

> `nums1` 和 `nums2` 本身已经是有序数组。

因此它并不是这道题真正希望我们使用的方法。

------

## 算法步骤

设合并后的数组长度为 `total`：

1. 将 `nums1` 和 `nums2` 合并。
2. 对合并后的数组排序。
3. 如果 `total` 为奇数：
   - 返回 `merged[total // 2]`。
4. 如果 `total` 为偶数：
   - 返回中间两个元素的平均值。

------

## Python 实现

```python
from typing import List


class Solution:
    def findMedianSortedArrays(
        self,
        nums1: List[int],
        nums2: List[int]
    ) -> float:

        # 合并两个数组
        merged = nums1 + nums2

        # 对合并后的数组进行排序
        merged.sort()

        total = len(merged)

        # 偶数个元素：取中间两个元素的平均值
        if total % 2 == 0:
            left_middle = merged[total // 2 - 1]
            right_middle = merged[total // 2]

            return (left_middle + right_middle) / 2.0

        # 奇数个元素：直接返回中间元素
        return float(merged[total // 2])
```

------

## 复杂度分析

设：

- `m = len(nums1)`
- `n = len(nums2)`

合并需要：

$O(m+n)$

排序需要：

$O((m+n)\log(m+n))$

因此总体时间复杂度：

$\boxed{O((m+n)\log(m+n))}$

额外空间复杂度：

$\boxed{O(m+n)}$

------

# 解法二：双指针

## 思路

因为两个数组本身已经有序，所以实际上没有必要：

```text
合并 → 再排序
```

我们可以模仿 **归并排序（Merge Sort）** 中合并两个有序数组的过程。

分别使用两个指针：

```text
i → nums1
j → nums2
```

每次比较：

```python
nums1[i]
nums2[j]
```

取较小的那个作为当前合并后的下一个元素，并移动对应指针。

不过，我们甚至不需要真的创建一个完整的合并数组。

原因是：

> 中位数只与中间的一个或两个元素有关。

因此，只需要模拟合并过程直到数组中间位置即可。

------

## 为什么需要记录两个值？

假设：

```text
median1 = 当前取出的元素
median2 = 上一次取出的元素
```

如果总长度是奇数：

```text
median = median1
```

如果总长度是偶数：

```text
median = (median1 + median2) / 2
```

例如：

```text
nums1 = [1, 3]
nums2 = [2, 4]
```

模拟合并：

```text
1
2
3
```

走到中间附近时：

```text
median2 = 2
median1 = 3
```

因此：

$\frac{2+3}{2}=2.5$

------

## Python 实现

```python
from typing import List


class Solution:
    def findMedianSortedArrays(
        self,
        nums1: List[int],
        nums2: List[int]
    ) -> float:

        len1 = len(nums1)
        len2 = len(nums2)

        # 两个数组各自的指针
        i = 0
        j = 0

        # median1：当前取出的元素
        # median2：上一次取出的元素
        median1 = 0
        median2 = 0

        # 我们只需要走到中间位置即可
        for _ in range((len1 + len2) // 2 + 1):

            # 当前 median1 会成为“上一个元素”
            median2 = median1

            # 两个数组都还有元素
            if i < len1 and j < len2:

                if nums1[i] <= nums2[j]:
                    median1 = nums1[i]
                    i += 1
                else:
                    median1 = nums2[j]
                    j += 1

            # nums2 已经遍历完
            elif i < len1:
                median1 = nums1[i]
                i += 1

            # nums1 已经遍历完
            else:
                median1 = nums2[j]
                j += 1

        total = len1 + len2

        # 奇数长度
        if total % 2 == 1:
            return float(median1)

        # 偶数长度
        return (median1 + median2) / 2.0
```

------

## 复杂度分析

最多只需要扫描两个数组的一半左右，因此渐进复杂度仍然为：

$\boxed{O(m+n)}$

不需要额外数组，因此空间复杂度为：

$\boxed{O(1)}$

------

# 解法三：寻找第 k 小元素

## 核心思想

这道题其实可以转换成另一个经典问题：

> 如何在两个有序数组中找到第 `k` 小的元素？

如果能够高效解决这个问题，那么中位数自然也就可以得到。

设两个数组总长度为：

```python
total = m + n
```

如果 `total` 为奇数，中位数就是：

$\frac{total+1}{2}$

位置上的元素。

如果 `total` 为偶数，则中位数是：

$\frac{total}{2}$

和：

$\frac{total}{2}+1$

这两个位置元素的平均值。

------

# 如何寻找第 k 小元素？

假设：

```text
A = [...]
B = [...]
```

我们想寻找两个数组合起来之后的：

```text
第 k 小元素
```

核心思想是：

> 每次尝试排除大约 `k / 2` 个不可能成为答案的元素。

例如：

```text
A = [1, 3, 5, 7, ...]
B = [2, 4, 6, 8, ...]
```

假设寻找第 `k` 小元素。

我们可以比较：

```text
A 的第 k/2 个元素
B 的第 k/2 个元素
```

如果：

```text
A[k/2 - 1] <= B[k/2 - 1]
```

那么可以确定：

```text
A 前 k/2 个元素
```

都不可能是第 `k` 小元素。

因此可以直接把它们排除。

与此同时：

```text
k -= 被排除的元素数量
```

不断重复这个过程。

------

## 为什么可以排除？

假设：

```text
A[i - 1] <= B[j - 1]
```

并且：

```text
i ≈ k / 2
j ≈ k / 2
```

那么对于 `A[i - 1]` 来说，最多只有：

```text
A 前 i 个元素
+
B 前 j - 1 个元素
```

位于它前面。

因此：

```text
A[0:i]
```

不可能包含第 `k` 小元素。

于是可以安全地排除。

------

# 递归算法

定义：

```python
get_kth(A, B, k)
```

表示：

> 返回两个有序数组合并之后的第 `k` 小元素。

注意这里的 `k` 是：

```text
从 1 开始计数
```

也就是说：

```text
k = 1
```

表示最小元素。

算法流程：

1. 保证 `A` 是较短的数组。

2. 如果 `A` 为空：

   ```python
   return B[k - 1]
   ```

3. 如果：

   ```python
   k == 1
   ```

   返回：

   ```python
   min(A[0], B[0])
   ```

4. 每次查看大约第 `k // 2` 个元素。

5. 比较两个数组对应位置的值。

6. 排除其中一个数组的前一部分。

7. 减小 `k`。

8. 继续递归。

------

## Python 实现

这里不直接使用：

```python
A[i:]
```

来创建新的切片，而是使用：

```python
a_start
b_start
```

记录当前数组的起始位置。

这样可以避免每次递归创建新数组。

```python
from typing import List


class Solution:

    def get_kth(
        self,
        a: List[int],
        m: int,
        b: List[int],
        n: int,
        k: int,
        a_start: int = 0,
        b_start: int = 0
    ) -> int:

        # 始终保证 a 是剩余元素较少的数组
        if m > n:
            return self.get_kth(
                b, n,
                a, m,
                k,
                b_start,
                a_start
            )

        # 如果 a 已经为空，
        # 那么第 k 小元素直接位于 b 中
        if m == 0:
            return b[b_start + k - 1]

        # 第 1 小元素就是两个数组当前首元素中的较小值
        if k == 1:
            return min(
                a[a_start],
                b[b_start]
            )

        # 尝试从两个数组中分别检查约 k // 2 个元素
        i = min(m, k // 2)
        j = min(n, k // 2)

        a_value = a[a_start + i - 1]
        b_value = b[b_start + j - 1]

        if a_value > b_value:
            # b 的前 j 个元素不可能包含第 k 小元素
            return self.get_kth(
                a,
                m,
                b,
                n - j,
                k - j,
                a_start,
                b_start + j
            )

        # a 的前 i 个元素不可能包含第 k 小元素
        return self.get_kth(
            a,
            m - i,
            b,
            n,
            k - i,
            a_start + i,
            b_start
        )

    def findMedianSortedArrays(
        self,
        nums1: List[int],
        nums2: List[int]
    ) -> float:

        total = len(nums1) + len(nums2)

        # 对于奇数长度，left == right
        # 对于偶数长度，它们分别表示两个中间位置
        left = (total + 1) // 2
        right = (total + 2) // 2

        left_value = self.get_kth(
            nums1,
            len(nums1),
            nums2,
            len(nums2),
            left
        )

        right_value = self.get_kth(
            nums1,
            len(nums1),
            nums2,
            len(nums2),
            right
        )

        return (left_value + right_value) / 2.0
```

------

## `//` 运算符说明

Python 中：

```python
k // 2
```

表示**整数除法（floor division）**。

例如：

```python
5 // 2   # 2
6 // 2   # 3
```

而：

```python
5 / 2
```

得到：

```python
2.5
```

因为数组索引必须使用整数，所以这里需要：

```python
//
```

------

## 复杂度分析

每次递归都会排除相当一部分候选元素，使 `k` 持续缩小。

因此时间复杂度为：

$\boxed{O(\log(m+n))}$

递归调用会占用调用栈，因此额外空间复杂度为：

$\boxed{O(\log(m+n))}$

------

# 解法四：二分查找分割线（最优解）

这是这道题最经典、也是面试中最值得掌握的解法。

时间复杂度：

$\boxed{O(\log(\min(m,n)))}$

空间复杂度：

$\boxed{O(1)}$

------

# 核心思想：不要找中位数，而是找正确的分割线

我们希望把两个数组分别切一刀：

```text
A: [ ........ | ........ ]
B: [ ........ | ........ ]
```

然后把左右两部分看成：

```text
左半部分
右半部分
```

我们希望满足两个条件。

------

## 条件一：左右元素数量正确

左边必须包含整个数组大约一半的元素。

------

## 条件二：左边所有元素都 <= 右边所有元素

因为两个数组内部本身已经有序，所以实际上只需要检查分割线附近的四个元素：

```text
Aleft   | Aright
Bleft   | Bright
```

正确分割必须满足：

```python
Aleft <= Bright
```

以及：

```python
Bleft <= Aright
```

一旦满足这两个条件：

> 整个左半部分中的所有元素，都不会大于整个右半部分中的任何元素。

这时，中位数一定就在这四个边界元素之间。

------

# 为什么只需要检查四个元素？

假设数组内部已经有序：

```text
A:
... <= Aleft | Aright <= ...

B:
... <= Bleft | Bright <= ...
```

由于：

```python
Aleft <= Aright
Bleft <= Bright
```

天然成立，因此真正需要检查的只有两个**跨数组关系**：

```python
Aleft <= Bright
Bleft <= Aright
```

只要它们成立，就可以保证：

```text
max(左半部分) <= min(右半部分)
```

这正是正确分割所需要的条件。

------

# 为什么只在较短数组上二分？

假设：

```text
A = 较短数组
B = 较长数组
```

我们在 `A` 中选择一个分割位置之后，由于左侧总元素数量已经确定，因此：

> `B` 的分割位置也随之确定。

也就是说，我们只需要搜索一个变量。

如果：

```text
A 的分割位置 = i
```

那么：

```text
B 的分割位置 = j
```

可以直接计算出来。

因此整个问题变成：

> 在较短数组中二分搜索正确的分割位置。

搜索范围大小是：

$\min(m,n)$

所以时间复杂度是：

$O(\log(\min(m,n)))$

------

# 分割索引的定义

原始实现使用的是一种比较特殊、但非常常见的索引定义：

```text
i = A 左边最后一个元素的下标
j = B 左边最后一个元素的下标
```

因此：

```text
Aleft  = A[i]
Aright = A[i + 1]

Bleft  = B[j]
Bright = B[j + 1]
```

如果：

```python
half = total // 2
```

那么：

```python
j = half - i - 2
```

这里的 `-2` 很容易产生困惑。

原因是 `i` 和 `j` 表示的是：

> 左边最后一个元素的**下标**

而不是：

> 左边元素的**数量**。

例如：

```text
i = 2
```

意味着 A 左边实际上有：

```text
3 个元素
```

也就是：

```text
i + 1
```

个。

于是：

```text
(i + 1) + (j + 1) = half
```

整理得到：

```text
i + j + 2 = half
```

因此：

```python
j = half - i - 2
```

这也是这份代码中 `-2` 的来源。

------

# 边界情况：为什么需要正负无穷？

考虑：

```text
A = [1, 2]
```

如果分割线位于数组最左边：

```text
| 1 2
```

那么：

```text
Aleft
```

不存在。

为了让比较逻辑仍然成立，我们把：

```python
Aleft = -∞
```

这样：

```python
Aleft <= Bright
```

永远不会因为左边没有元素而失败。

同理，如果分割线位于数组最右边：

```text
1 2 |
```

那么：

```text
Aright
```

不存在。

此时令：

```python
Aright = +∞
```

Python 中可以写成：

```python
float("-infinity")
float("infinity")
```

也可以使用更简洁的：

```python
float("-inf")
float("inf")
```

例如：

```python
float("-inf") < -1000000000
# True

float("inf") > 1000000000
# True
```

因此无穷值非常适合处理这种数组边界情况。

------

# 如何根据分割线得到中位数？

## 总长度为奇数

例如：

```text
1 2 | 3 4
```

左侧比右侧少一个元素。

中位数就是右侧最小元素：

```python
min(Aright, Bright)
```

------

## 总长度为偶数

例如：

```text
1 2 | 3 4
```

中间的两个元素分别是：

```text
左边最大值
右边最小值
```

所以：

```python
left_max = max(Aleft, Bleft)
right_min = min(Aright, Bright)
```

中位数：

```python
(left_max + right_min) / 2
```

------

# Python 实现

```python
from typing import List


class Solution:
    def findMedianSortedArrays(
        self,
        nums1: List[int],
        nums2: List[int]
    ) -> float:

        A = nums1
        B = nums2

        total = len(A) + len(B)
        half = total // 2

        # 二分搜索必须放在较短的数组上
        if len(A) > len(B):
            A, B = B, A

        # i 表示 A 左侧最后一个元素的下标
        #
        # 注意：
        # i = -1 也是合法状态，
        # 表示 A 左半部分为空。
        l = 0
        r = len(A) - 1

        while True:

            # A 中的分割位置
            i = (l + r) // 2

            # 根据左边总元素数量推导出 B 的分割位置
            j = half - i - 2

            # -----------------------------
            # 获取分割线两侧的四个边界元素
            # -----------------------------

            # 如果 A 左边为空，用 -∞
            Aleft = (
                A[i]
                if i >= 0
                else float("-inf")
            )

            # 如果 A 右边为空，用 +∞
            Aright = (
                A[i + 1]
                if i + 1 < len(A)
                else float("inf")
            )

            # 如果 B 左边为空，用 -∞
            Bleft = (
                B[j]
                if j >= 0
                else float("-inf")
            )

            # 如果 B 右边为空，用 +∞
            Bright = (
                B[j + 1]
                if j + 1 < len(B)
                else float("inf")
            )

            # -----------------------------
            # 判断当前分割是否正确
            # -----------------------------

            if Aleft <= Bright and Bleft <= Aright:

                # 总长度为奇数
                if total % 2 == 1:
                    return float(min(Aright, Bright))

                # 总长度为偶数
                left_max = max(Aleft, Bleft)
                right_min = min(Aright, Bright)

                return (left_max + right_min) / 2.0

            # Aleft 太大：
            # 说明 A 左边元素取得太多，
            # 分割线需要向左移动
            elif Aleft > Bright:
                r = i - 1

            # Bleft > Aright
            #
            # 说明 A 左边取得太少，
            # 需要把 A 的分割线向右移动
            else:
                l = i + 1
```

------

# 用例分析

考虑：

```text
nums1 = [1, 3]
nums2 = [2, 4]
```

总长度：

```text
4
```

因此：

```python
half = 2
```

两个数组长度相同：

```text
A = [1, 3]
B = [2, 4]
```

------

## 第一次二分

```python
l = 0
r = 1
```

所以：

```python
i = 0
```

然后：

```python
j = half - i - 2
  = 2 - 0 - 2
  = 0
```

于是：

```text
A:

1 | 3
↑   ↑
L   R


B:

2 | 4
↑   ↑
L   R
```

因此：

```python
Aleft = 1
Aright = 3

Bleft = 2
Bright = 4
```

检查：

```python
Aleft <= Bright
1 <= 4
# True
```

以及：

```python
Bleft <= Aright
2 <= 3
# True
```

所以找到了正确分割。

由于元素总数为偶数：

```python
left_max = max(1, 2)
# 2

right_min = min(3, 4)
# 3
```

最终：

```python
(2 + 3) / 2
```

得到：

```text
2.5
```

------

# 一个更容易理解的分割写法

上面的最优解代码中：

```python
i
j
```

表示的是：

> 左半部分最后一个元素的数组下标。

因此会出现：

```python
j = half - i - 2
```

这种比较难记的公式。

面试时，也可以采用另一种通常更加直观的定义：

> `partitionA` 和 `partitionB` 表示左半部分包含多少个元素。

这样公式会更容易理解。

```python
from typing import List


class Solution:
    def findMedianSortedArrays(
        self,
        nums1: List[int],
        nums2: List[int]
    ) -> float:

        # 始终在较短数组上二分搜索
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1

        m = len(nums1)
        n = len(nums2)

        left = 0
        right = m

        # 左半部分需要包含的元素数量
        #
        # +1 的作用是：
        # 如果总长度为奇数，让左半部分多一个元素，
        # 这样中位数始终可以从左侧最大值取得。
        half = (m + n + 1) // 2

        while left <= right:

            # partition1 表示 nums1 左边有多少个元素
            partition1 = (left + right) // 2

            # nums2 左边应该有多少个元素
            partition2 = half - partition1

            # nums1 分割线附近的元素
            max_left_1 = (
                nums1[partition1 - 1]
                if partition1 > 0
                else float("-inf")
            )

            min_right_1 = (
                nums1[partition1]
                if partition1 < m
                else float("inf")
            )

            # nums2 分割线附近的元素
            max_left_2 = (
                nums2[partition2 - 1]
                if partition2 > 0
                else float("-inf")
            )

            min_right_2 = (
                nums2[partition2]
                if partition2 < n
                else float("inf")
            )

            # 找到正确分割
            if (
                max_left_1 <= min_right_2
                and max_left_2 <= min_right_1
            ):

                # 奇数个元素：
                # 左半部分比右半部分多一个，
                # 中位数就是左侧最大值
                if (m + n) % 2 == 1:
                    return float(
                        max(max_left_1, max_left_2)
                    )

                # 偶数个元素：
                # 中位数是左侧最大值和右侧最小值的平均
                left_max = max(
                    max_left_1,
                    max_left_2
                )

                right_min = min(
                    min_right_1,
                    min_right_2
                )

                return (left_max + right_min) / 2.0

            # nums1 左侧取得太多
            elif max_left_1 > min_right_2:
                right = partition1 - 1

            # nums1 左侧取得太少
            else:
                left = partition1 + 1
```

这两个版本的本质完全相同，只是**分割位置的定义不同**。

对于代码面试，我通常更推荐第二种写法，因为：

```text
partition1 = A 左边元素数量
partition2 = B 左边元素数量
```

于是天然有：

```python
partition1 + partition2 == half
```

因此：

```python
partition2 = half - partition1
```

比：

```python
j = half - i - 2
```

更容易推导，也更不容易出现 off-by-one 错误。

------

# 最优解复杂度分析

假设较短数组长度为：

```text
min(m, n)
```

我们只在这个数组上进行二分搜索。

因此时间复杂度：

$\boxed{O(\log(\min(m,n)))}$

整个过程中只使用几个变量：

```text
i
j
Aleft
Aright
Bleft
Bright
...
```

没有创建额外数组。

因此空间复杂度：

$\boxed{O(1)}$

------

# 四种解法对比

| 解法        | 时间复杂度          | 空间复杂度           | 是否满足题目要求 | 面试价值 |
| ----------- | ------------------- | -------------------- | ---------------- | -------- |
| 合并 + 排序 | `O((m+n) log(m+n))` | `O(m+n)`             | ❌                | 低       |
| 双指针      | `O(m+n)`            | `O(1)`               | ❌                | 中       |
| 第 k 小元素 | `O(log(m+n))`       | `O(log(m+n))` 递归栈 | ✅                | 高       |
| 二分分割线  | `O(log(min(m,n)))`  | `O(1)`               | ✅                | **最高** |

------

# 面试时最值得掌握的思考路径

这道题真正困难的地方不是「中位数」本身，而是如何利用：

```text
两个数组已经有序
```

这个条件。

可以按照下面的思考过程推进：

```text
暴力：
合并 + 排序
        ↓
已经有序，不需要重新排序
        ↓
双指针模拟 merge
        ↓
但题目要求 O(log(m+n))
        ↓
必须使用二分思想
        ↓
把中位数转换成第 k 小元素
        ↓
或者寻找满足条件的分割线
        ↓
只在较短数组上二分
```

------

# 常见错误

## 1. 在较长数组上进行二分

最优解应该：

> 始终在较短数组上进行二分搜索。

如果在较长数组上搜索，那么根据分割位置计算出来的另一个数组的分割点可能：

```text
< 0
```

或者：

```text
> 数组长度
```

从而导致复杂的边界问题甚至数组越界。

因此通常先写：

```python
if len(nums1) > len(nums2):
    nums1, nums2 = nums2, nums1
```

------

## 2. 混用不同的 partition 定义

这是这道题最容易出现的 bug 之一。

一种写法定义：

```text
i = 左边最后一个元素的下标
```

另一种写法定义：

```text
i = 左边元素的数量
```

这两种写法都正确，但对应公式完全不同。

例如第一种：

```python
half = (m + n) // 2

j = half - i - 2
```

第二种可能写成：

```python
half = (m + n + 1) // 2

partitionB = half - partitionA
```

绝对不能：

> 使用一种定义的 `half`，再搭配另一种定义的 `i/j` 公式。

否则非常容易出现：

```text
off-by-one error
```

即「差 1」的错误。

------

# 3. 没有正确处理空 partition

例如：

```text
A = [1, 2]

| 1 2
```

此时左边没有元素。

不能直接访问：

```python
A[-1]
```

因为 Python 中：

```python
A[-1]
```

并不会报错，而是表示：

> 数组最后一个元素。

这反而可能导致一种非常隐蔽的逻辑错误。

应该显式使用：

```python
float("-inf")
```

或者：

```python
float("inf")
```

处理不存在的边界。

------

# 4. 混淆奇数和偶数情况

总长度为奇数时，中位数只有：

```text
1 个元素
```

总长度为偶数时，中位数由：

```text
2 个元素
```

决定。

而且具体应该取：

```text
max(left)
```

还是：

```text
min(right)
```

取决于你采用的 partition 定义。

因此：

> 不要死记最后的 median 公式，要先明确你的左右分区到底各有多少个元素。

------

# 5. 忘记数组本身已经有序

如果直接：

```python
merged = nums1 + nums2
merged.sort()
```

虽然可以得到正确答案，但实际上浪费了：

```text
两个输入数组已经排序
```

这个核心条件。

时间复杂度会变成：

$O((m+n)\log(m+n))$

而最优算法可以做到：

$O(\log(\min(m,n)))$

------

# 记忆最优解

如果需要在面试前快速复习，可以只记住下面这几句话：

```text
1. 永远在短数组上二分。

2. 找一个 partition，使左右两边元素数量相等
   （奇数时一侧多一个）。

3. 正确 partition 满足：

   Aleft <= Bright
   Bleft <= Aright

4. Aleft 太大：
   partition 向左。

5. Bleft 太大：
   partition 向右。

6. 找到以后：
   偶数：
   (max(left) + min(right)) / 2

   奇数：
   根据 partition 定义取
   max(left) 或 min(right)。
```

------

# 核心模板

推荐面试时记住这一版：

```python
from typing import List


class Solution:
    def findMedianSortedArrays(
        self,
        nums1: List[int],
        nums2: List[int]
    ) -> float:

        # 保证 nums1 是较短数组
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1

        m = len(nums1)
        n = len(nums2)

        left = 0
        right = m

        # 左半部分元素数量
        half = (m + n + 1) // 2

        while left <= right:

            partition1 = (left + right) // 2
            partition2 = half - partition1

            max_left_1 = (
                nums1[partition1 - 1]
                if partition1 > 0
                else float("-inf")
            )

            min_right_1 = (
                nums1[partition1]
                if partition1 < m
                else float("inf")
            )

            max_left_2 = (
                nums2[partition2 - 1]
                if partition2 > 0
                else float("-inf")
            )

            min_right_2 = (
                nums2[partition2]
                if partition2 < n
                else float("inf")
            )

            # 找到正确 partition
            if (
                max_left_1 <= min_right_2
                and max_left_2 <= min_right_1
            ):

                # 奇数
                if (m + n) % 2 == 1:
                    return float(
                        max(max_left_1, max_left_2)
                    )

                # 偶数
                return (
                    max(max_left_1, max_left_2)
                    + min(min_right_1, min_right_2)
                ) / 2.0

            # nums1 左侧太大
            if max_left_1 > min_right_2:
                right = partition1 - 1

            # nums1 左侧太小
            else:
                left = partition1 + 1
```

这份模板的核心只有三个公式真正需要记住：

```python
half = (m + n + 1) // 2

partition2 = half - partition1
```

以及：

```python
max_left_1 <= min_right_2
and
max_left_2 <= min_right_1
```

理解这三个关系之后，这道题就不需要依靠死记代码了。
