# Search Insert Position（搜索插入位置）

## 题目描述

给定一个**升序排列且元素互不相同**的整数数组 `nums`，以及一个目标值 `target`：

- 如果 `target` 存在于数组中，返回它的下标。
- 如果 `target` 不存在，返回它按照升序插入数组后应该所在的下标。

要求算法的时间复杂度必须为：

$O(\log n)$

例如：

```text
nums = [1, 3, 5, 6], target = 5
返回 2

nums = [1, 3, 5, 6], target = 2
返回 1

nums = [1, 3, 5, 6], target = 7
返回 4
```

------

## 1. 线性搜索 Linear Search

### 思路

最直接的方法是从左向右遍历数组，寻找第一个：

```text
nums[i] >= target
```

的位置。

为什么是 `>=`？

- 如果 `nums[i] == target`，说明已经找到了目标值。
- 如果 `nums[i] > target`，那么为了保持数组有序，`target` 应该插入到 `i` 的位置。

如果遍历完整个数组都没有找到这样的元素，则说明：

```text
target > nums 中的所有元素
```

因此应该插入数组末尾，也就是下标 `n`。

### 算法步骤

1. 从下标 `0` 开始遍历数组。
2. 如果发现 `nums[i] >= target`，返回 `i`。
3. 如果遍历结束仍未返回，返回 `len(nums)`。

### Python 实现

```python
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        for i in range(len(nums)):
            # 找到第一个大于或等于 target 的元素
            if nums[i] >= target:
                return i

        # target 比所有元素都大，插入数组末尾
        return len(nums)
```

### 时间与空间复杂度

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

### 评价

这个方法逻辑简单，但**不满足题目要求的 `O(log n)` 时间复杂度**。

由于数组已经有序，应该利用这个性质使用二分搜索。

------

# 2. 二分搜索 Binary Search - I

## 思路

因为数组已经排序，可以使用二分搜索。

这一版本除了维护二分搜索的左右边界之外，还额外维护一个：

```python
res
```

用于记录当前找到的、可能的插入位置。

初始时：

```python
res = len(nums)
```

表示默认情况下，`target` 应该插入数组末尾。

如果在搜索过程中发现：

```python
nums[mid] > target
```

那么 `mid` 就是一个可能的插入位置。

但它不一定是最靠左的插入位置，因此：

```python
res = mid
```

之后继续向左搜索。

------

## 算法步骤

初始化：

```python
l = 0
r = n - 1
res = n
```

之后不断检查中间位置 `mid`：

### 情况 1：找到目标

```python
nums[mid] == target
```

直接返回 `mid`。

### 情况 2：中间值大于目标

```python
nums[mid] > target
```

说明：

- `mid` 可以作为插入位置。
- 但左边可能还有更合适的位置。

因此：

```python
res = mid
r = mid - 1
```

### 情况 3：中间值小于目标

```python
nums[mid] < target
```

说明目标一定在右边：

```python
l = mid + 1
```

最后返回 `res`。

------

## Python 实现

```python
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        # 默认插入数组末尾
        res = len(nums)

        l, r = 0, len(nums) - 1

        while l <= r:
            mid = (l + r) // 2

            # 找到 target，直接返回
            if nums[mid] == target:
                return mid

            # nums[mid] 比 target 大
            # 当前 mid 是一个可能的插入位置
            if nums[mid] > target:
                res = mid

                # 继续向左寻找更小的合法位置
                r = mid - 1
            else:
                # nums[mid] < target
                # target 只能出现在右半部分
                l = mid + 1

        return res
```

------

## 时间与空间复杂度

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

------

# 3. 二分搜索 Binary Search - II

## 核心观察

实际上，我们并不需要额外使用：

```python
res
```

因为标准二分搜索结束之后，如果没有找到 `target`，左指针 `l` **天然就会停在正确的插入位置上**。

这是这道题最重要的理解之一。

------

## 为什么最后的 `l` 就是答案？

整个搜索过程中：

```python
nums[mid] < target
```

时，我们执行：

```python
l = mid + 1
```

因此 `l` 会不断越过所有：

```text
小于 target 的元素
```

而当：

```python
nums[mid] > target
```

时：

```python
r = mid - 1
```

右边界会向左移动。

最终循环结束时：

```python
l > r
```

此时 `l` 恰好表示：

> 第一个应该放置 `target` 的位置。

换一种理解：

```text
[ 所有 < target 的元素 ] target [ 所有 > target 的元素 ]
                           ↑
                           l
```

------

## 算法步骤

1. 初始化：

```python
l = 0
r = n - 1
```

1. 当 `l <= r` 时：
   - 计算 `mid`
   - 如果 `nums[mid] == target`，返回 `mid`
   - 如果 `nums[mid] > target`，搜索左半部分
   - 如果 `nums[mid] < target`，搜索右半部分
2. 如果最终没有找到目标，返回 `l`。

------

## Python 实现

```python
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums) - 1

        while l <= r:
            mid = (l + r) // 2

            # 找到 target
            if nums[mid] == target:
                return mid

            if nums[mid] > target:
                # target 在左边
                r = mid - 1
            else:
                # nums[mid] < target
                # target 在右边
                l = mid + 1

        # 如果 target 不存在，
        # l 正好是它应该插入的位置
        return l
```

------

## 示例分析

假设：

```text
nums = [1, 3, 5, 6]
target = 2
```

第一次：

```text
l = 0
r = 3
mid = 1
nums[mid] = 3
```

因为：

```text
3 > 2
```

所以：

```text
r = 0
```

第二次：

```text
l = 0
r = 0
mid = 0
nums[mid] = 1
```

因为：

```text
1 < 2
```

所以：

```text
l = 1
```

此时：

```text
l = 1
r = 0
```

循环结束。

返回：

```python
1
```

正好是：

```text
[1, 2, 3, 5, 6]
    ↑
```

------

## 时间与空间复杂度

- 时间复杂度：`O(log n)`
- 空间复杂度：`O(1)`

------

# 4. Lower Bound：更通用的二分搜索写法

## 思路

这道题本质上是在寻找：

> **第一个大于或等于 `target` 的元素的位置。**

这种操作在算法中通常称为：

```text
lower bound
```

数学上可以理解为寻找最小的下标 `i`，使得：

$nums[i] \ge target$

如果不存在这样的元素，则返回：

```python
len(nums)
```

这正好就是本题需要的答案。

------

## 搜索区间

这个版本和前面的二分搜索有一个重要区别。

初始化为：

```python
l = 0
r = len(nums)
```

注意：

```python
r = len(nums)
```

而不是：

```python
len(nums) - 1
```

因为这里使用的是**左闭右开区间**：

```text
[l, r)
```

例如数组：

```text
[1, 3, 5, 6]
```

我们搜索的候选位置实际上有：

```text
0  1  2  3  4
```

其中 `4` 表示：

```text
插入数组末尾
```

因此必须允许 `r = n`。

------

## 算法步骤

初始化：

```python
l = 0
r = n
```

循环条件：

```python
while l < r:
```

计算：

```python
mid = l + (r - l) // 2
```

如果：

```python
nums[mid] >= target
```

说明：

```text
mid 有可能就是答案
```

但左边可能还有更小的合法位置，所以：

```python
r = mid
```

注意这里不能写：

```python
r = mid - 1
```

因为当前的 `mid` 本身仍然是候选答案。

如果：

```python
nums[mid] < target
```

那么 `mid` 不可能是答案，因此：

```python
l = mid + 1
```

最终：

```python
l == r
```

该位置就是 lower bound。

------

## Python 实现

```python
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        # 搜索区间为 [l, r)
        l, r = 0, len(nums)

        while l < r:
            mid = l + (r - l) // 2

            if nums[mid] >= target:
                # mid 可能就是答案，
                # 因此保留 mid，继续搜索左侧
                r = mid
            else:
                # nums[mid] < target
                # mid 一定不是答案
                l = mid + 1

        return l
```

原答案中的：

```python
elif nums[mid] < target:
```

其实没有必要，因为前面的条件已经是：

```python
if nums[mid] >= target:
```

剩下的情况必然就是：

```python
nums[mid] < target
```

所以直接使用 `else` 更简洁。

------

# 5. 两种二分搜索模板的区别

这也是代码面试中非常值得掌握的内容。

## 模板一：闭区间 `[l, r]`

```python
l, r = 0, len(nums) - 1

while l <= r:
    mid = (l + r) // 2

    if nums[mid] < target:
        l = mid + 1
    elif nums[mid] > target:
        r = mid - 1
    else:
        return mid
```

特点：

```text
搜索范围 = [l, r]
```

所以：

```python
l == r
```

时仍然存在一个需要检查的元素，因此循环条件必须是：

```python
l <= r
```

------

## 模板二：左闭右开 `[l, r)`

```python
l, r = 0, len(nums)

while l < r:
    mid = l + (r - l) // 2

    if nums[mid] >= target:
        r = mid
    else:
        l = mid + 1
```

特点：

```text
搜索范围 = [l, r)
```

当：

```python
l == r
```

时区间已经为空，所以循环结束。

这种模板特别适合寻找：

- 第一个 `>= target`
- 第一个 `> target`
- 第一个满足某个条件的位置

也就是常说的：

```text
Boundary Binary Search
```

或者：

```text
Binary Search on the Answer Boundary
```

------

# 6. Python 内置二分搜索：`bisect_left`

Python 标准库提供了：

```python
bisect
```

模块，可以直接对有序列表执行二分搜索。

本题可以使用：

```python
bisect.bisect_left(nums, target)
```

------

## `bisect_left()` 的含义

```python
bisect.bisect_left(nums, target)
```

返回：

> 将 `target` 插入 `nums` 时，在保持数组有序的前提下，最靠左的合法插入位置。

换句话说，它寻找的是：

```text
第一个 >= target 的位置
```

也就是：

```text
lower bound
```

------

## Python 实现

```python
from typing import List
import bisect


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        # 返回第一个 >= target 的位置
        return bisect.bisect_left(nums, target)
```

------

## `bisect_left()` 示例

```python
import bisect

nums = [1, 3, 5, 6]

print(bisect.bisect_left(nums, 5))
# 2

print(bisect.bisect_left(nums, 2))
# 1

print(bisect.bisect_left(nums, 7))
# 4

print(bisect.bisect_left(nums, 0))
# 0
```

------

## `bisect_left` 和 `bisect_right`

Python 还有一个：

```python
bisect.bisect_right()
```

两者在数组存在重复元素时区别非常重要。

假设：

```python
nums = [1, 2, 2, 2, 4]
```

执行：

```python
bisect.bisect_left(nums, 2)
```

得到：

```text
1
```

即：

```text
第一个 2 的位置
```

而：

```python
bisect.bisect_right(nums, 2)
```

得到：

```text
4
```

即：

```text
最后一个 2 后面的位置
```

可以记成：

```text
bisect_left  → 第一个 >= target 的位置
bisect_right → 第一个 > target 的位置
```

虽然本题说明数组中的元素互不相同，因此两者在目标不存在时通常都会得到相同插入位置，但从语义上来说：

```python
bisect_left
```

更加符合本题要求。

------

## 时间与空间复杂度

`bisect_left()` 内部使用二分搜索，因此：

- 时间复杂度：`O(log n)`
- 额外空间复杂度：`O(1)`

需要注意：

```python
bisect.insort()
```

虽然寻找插入位置是 `O(log n)`，但真正往 Python `list` 中插入元素需要移动后面的元素，因此实际插入操作是：

$O(n)$

而本题只要求**返回插入位置**，所以 `bisect_left()` 是 `O(log n)`。

------

# 7. 推荐掌握的版本

对于这道题，如果是在代码面试中，推荐优先掌握 **Lower Bound 版本**：

```python
from typing import List


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums)

        while l < r:
            mid = l + (r - l) // 2

            if nums[mid] >= target:
                r = mid
            else:
                l = mid + 1

        return l
```

这个版本的价值不仅在于能解决本题，更重要的是它可以推广到很多二分搜索问题。

可以把它记成一个模板：

```python
l, r = 0, n

while l < r:
    mid = l + (r - l) // 2

    if mid 满足条件:
        r = mid
    else:
        l = mid + 1

return l
```

核心思想是：

> 找到**第一个满足某个条件的位置**。

------

# 8. 常见错误 Common Pitfalls

## 8.1 二分搜索边界的 Off-by-One Error

二分搜索最常见的问题不是算法思想，而是边界。

尤其容易混淆：

```python
while l <= r
```

和：

```python
while l < r
```

它们不能随意替换。

如果搜索区间是：

```text
[l, r]
```

通常使用：

```python
while l <= r
```

更新方式通常是：

```python
r = mid - 1
l = mid + 1
```

如果搜索区间是：

```text
[l, r)
```

通常使用：

```python
while l < r
```

在 lower bound 中：

```python
r = mid
```

而不是：

```python
r = mid - 1
```

------

## 8.2 忘记“插入数组末尾”的情况

例如：

```text
nums = [1, 3, 5, 6]
target = 10
```

正确答案是：

```text
4
```

即：

```python
len(nums)
```

因为：

```text
[1, 3, 5, 6, 10]
             ↑
             index = 4
```

不能返回：

```text
3
```

因为 `3` 是最后一个元素的下标，而不是末尾插入位置。

------

## 8.3 `r = mid` 和 `r = mid - 1` 混用

这是 lower bound 中非常重要的细节。

如果条件是：

```python
nums[mid] >= target
```

此时 `mid` 本身仍然可能是答案。

因此使用：

```python
r = mid
```

如果写成：

```python
r = mid - 1
```

就有可能错误地把真正答案排除掉。

------

## 8.4 不理解为什么可以返回 `l`

对于：

```python
while l <= r:
```

这种版本，循环结束意味着：

```text
r < l
```

并且可以理解为：

```text
0 ... r | l ... n-1
```

其中：

```text
nums[0:r+1] < target
nums[l:] > target
```

因此：

```text
l
```

恰好就是 `target` 应该出现的位置。

这是比“背代码”更值得掌握的性质。

------

# 9. 面试总结

这道题表面上是“搜索插入位置”，本质上考察的是：

```text
Binary Search / Lower Bound
```

最值得掌握的定义是：

$\text{lower\_bound}(target) = \text{第一个满足 } nums[i] \ge target \text{ 的位置}$

而本题要求的插入位置恰好就是这个位置。

因此可以建立下面的联系：

```text
Search Insert Position
        ↓
寻找第一个 >= target 的位置
        ↓
Lower Bound
        ↓
Binary Search
```

推荐记住两个版本：

```python
# 标准二分搜索
l, r = 0, n - 1
while l <= r:
    ...
return l
```

以及更加通用的：

```python
# Lower Bound
l, r = 0, n
while l < r:
    mid = l + (r - l) // 2

    if nums[mid] >= target:
        r = mid
    else:
        l = mid + 1

return l
```

如果面试官允许使用 Python 标准库，则可以直接：

```python
import bisect

return bisect.bisect_left(nums, target)
```

其中 `bisect_left()` 本质上就是在寻找：

```text
第一个 >= target 的位置
```
