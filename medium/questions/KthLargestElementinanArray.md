# 数组中的第 K 个最大元素

给定一个未排序的整数数组 `nums` 和一个整数 `k`，请返回数组中第 `k` 大的元素。

这里的“第 `k` 大”指数组按从小到大排序后所处的位置，并不是第 `k` 个不同的元素。因此，重复元素也会被正常计数。

进阶：你能否在不对整个数组排序的情况下解决这个问题？

## 关键下标转换

假设数组长度为 `n`：

- 按升序排列时，第 `k` 大元素的下标为 `n - k`
- 按降序排列时，第 `k` 大元素的下标为 `k - 1`

例如：

```
nums = [3, 2, 1, 5, 6, 4]
升序   = [1, 2, 3, 4, 5, 6]

第 2 大元素是 5
升序下标 = 6 - 2 = 4
降序下标 = 2 - 1 = 1
```

------

## 方法一：排序

### 思路

将整个数组按升序排列后：

- 最大元素位于最后一个位置
- 第 2 大元素位于倒数第 2 个位置
- 第 `k` 大元素位于下标 `n - k`

这是最容易理解和实现的方法，但由于需要对整个数组排序，时间复杂度为 `O(n log n)`。

### 算法步骤

1. 将数组按非递减顺序排序。
2. 计算第 `k` 大元素的下标 `n - k`。
3. 返回该位置上的元素。

### 代码

```
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 原地升序排序
        nums.sort()

        # 第 k 大元素在升序数组中的下标为 n - k
        return nums[len(nums) - k]
```

### Python 函数说明：`list.sort()`

`nums.sort()` 会对列表进行原地排序：

- 默认按升序排列
- 不会创建一个新的列表
- 返回值为 `None`
- 会改变原数组 `nums`

如果不希望改变原数组，可以使用：

```
sorted_nums = sorted(nums)
```

`sorted(nums)` 会返回一个新的有序列表。

### 复杂度分析

- 时间复杂度：`O(n log n)`
- 空间复杂度：Python 的 `list.sort()` 使用 Timsort，最坏情况下需要 `O(n)` 的辅助空间

------

## 方法二：大小为 K 的最小堆

### 思路

我们并不需要知道所有元素的完整顺序，只需要维护当前遇到的 `k` 个最大元素。

最小堆非常适合完成这个任务：

- 堆顶始终是堆中最小的元素
- 如果堆的大小始终保持为 `k`，堆中保存的就是目前最大的 `k` 个元素
- 其中最小的那个元素，也就是堆顶，正好是第 `k` 大元素

处理每个元素时：

1. 将元素加入最小堆。
2. 如果堆的大小超过 `k`，删除堆顶的最小元素。
3. 遍历结束后，堆顶就是第 `k` 大元素。

### 代码

```
import heapq
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        min_heap = []

        for num in nums:
            # 将当前元素加入最小堆
            heapq.heappush(min_heap, num)

            # 只保留最大的 k 个元素
            if len(min_heap) > k:
                heapq.heappop(min_heap)

        # 堆顶是这 k 个最大元素中最小的一个，
        # 因而就是整个数组中的第 k 大元素
        return min_heap[0]
```

也可以直接使用 Python 提供的 `nlargest`：

```
import heapq
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 返回最大的 k 个元素，并取其中最小的一个
        return heapq.nlargest(k, nums)[-1]
```

### Python 堆函数说明

Python 的 `heapq` 模块实现的是最小堆：

- `heapq.heappush(heap, value)`：将元素加入堆，时间复杂度为 `O(log k)`
- `heapq.heappop(heap)`：删除并返回堆顶最小元素，时间复杂度为 `O(log k)`
- `heapq.heapreplace(heap, value)`：先删除堆顶，再加入新元素
- `heapq.nlargest(k, iterable)`：返回可迭代对象中最大的 `k` 个元素

`heapq` 直接使用普通列表保存堆，不需要创建专门的堆对象。

### 复杂度分析

设 `n` 为数组长度：

- 时间复杂度：`O(n log k)`
- 空间复杂度：`O(k)`

当 `k` 远小于 `n` 时，该方法通常比完整排序更合适。

------

## 方法三：快速选择

### 思路

快速选择（Quick Select）与快速排序类似，但它不会递归处理分区后的两侧，而是只处理目标下标所在的一侧。

首先选择一个基准值 `pivot`，然后重新排列数组，使得：

- 小于或等于 `pivot` 的元素位于左侧
- 大于 `pivot` 的元素位于右侧
- `pivot` 最终位于它在有序数组中的正确位置

假设 `pivot` 最终位于下标 `p`：

- 如果 `p == target`，说明已经找到答案
- 如果 `p > target`，只需要继续搜索左侧
- 如果 `p < target`，只需要继续搜索右侧

对于第 `k` 大元素，其在升序数组中的目标下标为：

```
target = len(nums) - k
```

快速选择不保证每次都恰好排除一半元素，但在基准值选择比较均匀时，能够快速缩小搜索范围。

### 算法步骤

1. 将第 `k` 大转换为升序下标 `target = n - k`。
2. 在当前区间中选择最右侧元素作为基准值。
3. 对数组进行分区，并得到基准值的最终下标 `p`。
4. 根据 `p` 和 `target` 的关系，只搜索一侧区间。
5. 当 `p == target` 时返回答案。

### 代码

```
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 第 k 大对应升序数组中的第 n-k 个下标
        target = len(nums) - k

        def quick_select(left: int, right: int) -> int:
            # 选择区间最右侧元素作为基准值
            pivot = nums[right]

            # insert 表示下一个“小于等于 pivot”的元素
            # 应该被放置的位置
            insert = left

            for i in range(left, right):
                if nums[i] <= pivot:
                    nums[insert], nums[i] = nums[i], nums[insert]
                    insert += 1

            # 将 pivot 放到最终位置
            nums[insert], nums[right] = nums[right], nums[insert]

            if insert == target:
                return nums[insert]

            if insert > target:
                # 目标在左侧
                return quick_select(left, insert - 1)

            # 目标在右侧
            return quick_select(insert + 1, right)

        return quick_select(0, len(nums) - 1)
```

### 复杂度分析

- 平均时间复杂度：`O(n)`
- 最坏时间复杂度：`O(n²)`
- 平均递归栈空间复杂度：`O(log n)`
- 最坏递归栈空间复杂度：`O(n)`

最坏情况通常发生在基准值选择非常不均匀时。例如，对已经有序的数组始终选择最右侧元素作为基准值，每次可能只能排除一个元素。

该实现会在原数组上交换元素，因此会改变 `nums` 的元素顺序。

------

## 方法四：Hoare 分区快速选择

### 思路

前面的基础快速选择使用 Lomuto 分区：从一侧扫描数组，并把所有 `<= pivot` 的元素集中到左侧。当数组中存在大量等于 `pivot` 的元素时，分区可能非常不均衡。

Hoare 分区使用两个指针：

- `i` 从左向右寻找第一个 `>= pivot` 的元素。
- `j` 从右向左寻找第一个 `<= pivot` 的元素。
- 当 `i < j` 时，交换这两个位置上的元素。
- 当两个指针相遇或交错时，返回分界线 `j`。

扫描时使用严格比较 `< pivot` 和 `> pivot`。因此，遇到等于 `pivot` 的元素时两个指针都会停下；交换后，下一轮扫描又会继续向中间推进。重复元素通常会被分散到分界线两侧，而不是全部堆积到同一侧。

循环结束后，区间满足：

```
[left, j]      中的元素 <= pivot
[j + 1, right] 中的元素 >= pivot
```

Hoare 分区返回的是左右区间的**分界线**，不一定是基准元素的最终下标。这是它与 Lomuto 分区最重要的区别。

### Python 实现

```python
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 第 k 大元素对应升序排列后的下标 n - k
        target = len(nums) - k
        left = 0
        right = len(nums) - 1

        while left < right:
            # 选择当前区间最左侧元素作为基准值
            pivot = nums[left]

            # i、j 从区间外侧开始，方便使用先移动后判断的写法
            i = left - 1
            j = right + 1

            while True:
                # 从左向右寻找第一个 >= pivot 的元素
                i += 1
                while nums[i] < pivot:
                    i += 1

                # 从右向左寻找第一个 <= pivot 的元素
                j -= 1
                while nums[j] > pivot:
                    j -= 1

                # 指针相遇或交错，j 就是左右区间的分界线
                if i >= j:
                    break

                # nums[i] 应该位于右侧，nums[j] 应该位于左侧
                nums[i], nums[j] = nums[j], nums[i]

            # 注意：j 是分界线，而不是 pivot 的最终下标
            if target <= j:
                right = j
            else:
                left = j + 1

        # left == right 时，目标位置已经被定位
        return nums[target]
```

### 为什么它更适合重复元素

假设数组中的元素全部相同：

```
[5, 5, 5, 5, 5]
```

Lomuto 分区可能每轮只能排除一个元素，区间长度变化为：

```
5 -> 4 -> 3 -> 2 -> 1
```

Hoare 分区中的两个指针会持续向中间移动，分界线通常接近中间，区间长度大致变为：

```
5 -> 3 -> 2 -> 1
```

不过，Hoare 分区并没有专门建立“等于 pivot”的连续区域。它只是倾向于把重复元素分散到两侧，因此大量重复元素场景下通常表现良好，但三路分区会更加直接。

### 复杂度分析

- 平均时间复杂度：`O(n)`
- 最坏时间复杂度：`O(n²)`
- 额外空间复杂度：`O(1)`

该实现使用循环而不是递归，并且会原地修改数组。

------

## 方法五：三路分区快速选择

### 思路

三路分区也叫 Dutch National Flag Partition（荷兰国旗分区）。它会在一次扫描中将当前区间划分为三个连续部分：

```
小于 pivot | 等于 pivot | 大于 pivot
```

使用三个指针维护区间：

- `[left, less - 1]`：小于 `pivot`。
- `[less, scan - 1]`：等于 `pivot`。
- `[scan, greater]`：尚未检查。
- `[greater + 1, right]`：大于 `pivot`。

扫描完成后：

- 如果 `target < less`，答案在左侧。
- 如果 `target > greater`，答案在右侧。
- 如果 `less <= target <= greater`，目标落在等值区间中，可以直接返回 `pivot`。

对于全部元素相同的数组，第一次扫描就会发现整个区间都等于 `pivot`，因此无需继续缩小区间。这是三路分区处理大量重复元素时最明显的优势。

### Python 实现

```python
import random
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 转换为升序排列后的目标下标
        target = len(nums) - k
        left = 0
        right = len(nums) - 1

        while left <= right:
            # 随机选择 pivot，降低有序数组上持续产生极端分区的概率
            pivot = nums[random.randint(left, right)]

            less = left       # 小于 pivot 区域的下一个写入位置
            scan = left       # 当前正在检查的位置
            greater = right   # 大于 pivot 区域的下一个写入位置

            while scan <= greater:
                if nums[scan] < pivot:
                    # 当前元素属于左侧“小于 pivot”的区域
                    nums[less], nums[scan] = nums[scan], nums[less]
                    less += 1
                    scan += 1

                elif nums[scan] > pivot:
                    # 当前元素属于右侧“大于 pivot”的区域
                    nums[scan], nums[greater] = nums[greater], nums[scan]
                    greater -= 1

                    # 交换到 scan 位置的元素尚未检查，
                    # 因此这里不能执行 scan += 1

                else:
                    # 当前元素等于 pivot，保留在中间区域
                    scan += 1

            # 分区完成：
            # [left, less - 1]       < pivot
            # [less, greater]        == pivot
            # [greater + 1, right]   > pivot
            if target < less:
                right = less - 1
            elif target > greater:
                left = greater + 1
            else:
                # target 位于等值区间，答案就是 pivot
                return pivot

        # 题目保证 1 <= k <= len(nums)，正常情况下不会执行到这里
        raise ValueError("k 超出有效范围")
```

### 为什么交换大元素后不能移动 `scan`

执行下面的交换后：

```python
nums[scan], nums[greater] = nums[greater], nums[scan]
```

原本位于 `greater` 的元素被交换到了 `scan`，但这个新元素还没有经过比较。因此只能令 `greater -= 1`，不能同时令 `scan += 1`。下一轮循环必须继续检查当前位置。

相反，当 `nums[scan] < pivot` 时，`less` 到 `scan - 1` 原本就是已经处理好的等值区域。交换后，来到 `scan` 位置的元素一定等于 `pivot`，所以可以安全地同时移动 `less` 和 `scan`。

### 复杂度分析

- 期望时间复杂度：`O(n)`
- 最坏时间复杂度：`O(n²)`
- 额外空间复杂度：`O(1)`

如果当前区间包含大量等于 `pivot` 的元素，三路分区可以一次排除整个等值区间。该实现会原地修改数组。

------

## 方法六：三数取中优化的快速选择

### 思路

这个版本仍然使用快速选择，不过进行了两个工程上的优化：

- 使用“三数取中”的方式选择基准值，减少连续选到极端值的概率
- 使用循环代替递归，使额外空间复杂度降低到 `O(1)`

该实现按照降序进行分区：

- 大于 `pivot` 的元素移动到左侧
- 小于 `pivot` 的元素移动到右侧

因此，第 `k` 大元素的目标下标为：

```
target = k - 1
```

需要注意，三数取中能够改善实际表现，但并没有从理论上消除 `O(n²)` 的最坏情况。因此，“优化版”比“最优版”更准确。

### 代码

```
from typing import List


class Solution:
    def partition(
        self,
        nums: List[int],
        left: int,
        right: int
    ) -> int:
        mid = (left + right) // 2

        # 把中间元素移动到 left + 1
        nums[mid], nums[left + 1] = nums[left + 1], nums[mid]

        # 对 left、left + 1、right 三个位置进行调整，
        # 使 nums[left] >= nums[left + 1] >= nums[right]
        if nums[left] < nums[right]:
            nums[left], nums[right] = nums[right], nums[left]

        if nums[left + 1] < nums[right]:
            nums[left + 1], nums[right] = nums[right], nums[left + 1]

        if nums[left] < nums[left + 1]:
            nums[left], nums[left + 1] = nums[left + 1], nums[left]

        # 选择三者的中位数作为 pivot
        pivot = nums[left + 1]

        i = left + 1
        j = right

        while True:
            # 从左向右寻找第一个 <= pivot 的元素
            while True:
                i += 1
                if nums[i] <= pivot:
                    break

            # 从右向左寻找第一个 >= pivot 的元素
            while True:
                j -= 1
                if nums[j] >= pivot:
                    break

            if i > j:
                break

            # 将较小元素移到右侧，较大元素移到左侧
            nums[i], nums[j] = nums[j], nums[i]

        # 将 pivot 放到最终位置
        nums[left + 1], nums[j] = nums[j], nums[left + 1]
        return j

    def quickSelect(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums) - 1

        while True:
            # 区间只剩一个或两个元素时，直接处理
            if right <= left + 1:
                if (
                    right == left + 1
                    and nums[right] > nums[left]
                ):
                    nums[left], nums[right] = nums[right], nums[left]

                return nums[target]

            pivot_index = self.partition(nums, left, right)

            if pivot_index == target:
                return nums[pivot_index]

            if pivot_index > target:
                # 目标位于左侧
                right = pivot_index - 1
            else:
                # 目标位于右侧
                left = pivot_index + 1

    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 降序排列下，第 k 大元素的下标是 k - 1
        return self.quickSelect(nums, k - 1)
```

### 三数取中说明

三数取中通常会考察以下三个位置：

- 当前区间的第一个元素
- 当前区间的中间元素
- 当前区间的最后一个元素

选择三者中的中位数作为基准值，可以降低基准值恰好是当前区间最小值或最大值的概率。

例如：

```
三个候选值：[1, 9, 5]
其中位数为 5，因此选择 5 作为 pivot。
```

它主要改善实际运行效率，但最坏时间复杂度依然是 `O(n²)`。

### 复杂度分析

- 平均时间复杂度：`O(n)`
- 最坏时间复杂度：`O(n²)`
- 额外空间复杂度：`O(1)`

该算法会原地修改数组。

------

## 方法七：随机基准值快速选择

在面试中，随机化快速选择通常比复杂的三数取中实现更容易解释，也更不容易写错。

随机选择基准值可以显著降低持续出现最坏分区的概率。

```
import random
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 转换为升序数组中的目标下标
        target = len(nums) - k
        left = 0
        right = len(nums) - 1

        while left <= right:
            # 随机选择 pivot，降低连续出现最坏分区的概率
            pivot_index = random.randint(left, right)

            # 将 pivot 移到最右侧，方便统一执行分区
            nums[pivot_index], nums[right] = (
                nums[right],
                nums[pivot_index],
            )
            pivot = nums[right]

            insert = left

            # 将所有 <= pivot 的元素移动到左侧
            for i in range(left, right):
                if nums[i] <= pivot:
                    nums[insert], nums[i] = nums[i], nums[insert]
                    insert += 1

            # 将 pivot 放到最终位置
            nums[insert], nums[right] = nums[right], nums[insert]

            if insert == target:
                return nums[insert]
            elif insert > target:
                right = insert - 1
            else:
                left = insert + 1

        # 在题目保证输入合法时，不会执行到这里
        raise ValueError("k 超出有效范围")
```

复杂度：

- 期望时间复杂度：`O(n)`
- 最坏时间复杂度：`O(n²)`
- 额外空间复杂度：`O(1)`

------

## 方法对比

| 方法                | 时间复杂度                | 额外空间             | 是否修改原数组 | 特点                                     |
| ------------------- | ------------------------- | -------------------- | -------------- | ---------------------------------------- |
| 排序                | `O(n log n)`              | Python 中最坏 `O(n)` | 是             | 最简单、最稳定                           |
| 大小为 `k` 的最小堆 | `O(n log k)`              | `O(k)`               | 否             | 适合 `k` 较小或流式数据                  |
| Lomuto 快速选择     | 平均 `O(n)`，最坏 `O(n²)` | 平均 `O(log n)`      | 是             | 实现直观，但大量重复元素时容易退化       |
| Hoare 快速选择      | 平均 `O(n)`，最坏 `O(n²)` | `O(1)`               | 是             | 双向扫描，通常能将重复元素分散到两侧     |
| 三路分区快速选择    | 期望 `O(n)`，最坏 `O(n²)` | `O(1)`               | 是             | 一次排除整个等值区间，最适合大量重复元素 |
| 三数取中快速选择    | 平均 `O(n)`，最坏 `O(n²)` | `O(1)`               | 是             | 降低选到极端 pivot 的概率                |
| 随机快速选择        | 期望 `O(n)`，最坏 `O(n²)` | `O(1)`               | 是             | 实现简洁，降低连续出现极端分区的概率     |

## 常见错误

### 1. 混淆第 K 大和第 K 小

在升序数组中：

```
第 k 小的下标 = k - 1
第 k 大的下标 = n - k
```

在降序数组中：

```
第 k 大的下标 = k - 1
```

这是最常见的下标偏移错误之一。

### 2. 错误地使用最大堆

如果维护最大的 `k` 个元素，应使用大小为 `k` 的最小堆。

原因是我们需要随时删除这 `k` 个元素中最小的元素。最小堆可以在 `O(log k)` 时间内完成这一操作。

最大堆通常需要：

1. 将全部 `n` 个元素加入堆。
2. 连续弹出 `k` 次最大元素。

这种方法一般需要 `O(n)` 空间，不如大小为 `k` 的最小堆节省空间。

### 3. 快速选择的基准值选择不当

如果始终使用最右侧元素作为基准值，那么对于已经升序或降序排列的数组，每次分区都可能非常不均匀：

```
n + (n - 1) + (n - 2) + ... + 1 = O(n²)
```

可以采用以下方式降低风险：

- 随机选择基准值
- 使用三数取中
- 使用三路分区处理大量重复元素

### 4. 忽略重复元素

题目要求的是排序后的第 `k` 个元素，而不是第 `k` 个不同的元素。

例如：

```
nums = [5, 5, 4, 3], k = 2
```

第 2 大元素仍然是 `5`，而不是 `4`。

### 5. 忘记算法会修改原数组

以下操作都会修改 `nums`：

- `nums.sort()`
- 原地快速选择
- 原地分区

如果题目要求保留原数组，可以先复制：

```
copied_nums = nums.copy()
```