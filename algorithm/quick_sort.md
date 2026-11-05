# 从一路到三路：用 Python 彻底理解快速排序

快速排序（Quick Sort）是经典的分治算法之一。它平均时间复杂度为 \(O(n \log n)\)，实现简洁，而且在实际工程中通常具有很好的性能。

但“快速排序”并不只有一种写法。

根据 `partition`，也就是“分区”策略的不同，快速排序常见的实现可以分为：

- 一路快速排序
- 两路快速排序
- 三路快速排序

它们的整体框架几乎完全一致，真正不同的地方，在于：

> 如何根据基准值 `pivot` 对当前数组区间进行划分。

本文将使用 Python，从最基础的一路快速排序开始，逐步理解两路和三路快速排序，以及为什么需要这些优化。

---

## 一、快速排序的核心思想

假设我们有这样一个数组：

```python
nums = [5, 3, 8, 4, 2, 7, 1]
```

快速排序的基本思想可以概括为三个步骤：

1. 从数组中选择一个元素作为基准值 `pivot`
2. 根据 `pivot` 对数组进行分区
3. 对分区后的左右子数组递归执行快速排序

例如，如果选择 `5` 作为 `pivot`，我们希望得到类似下面的结构：

```text
小于 5 的元素 | 5 | 大于 5 的元素
```

例如：

```text
[3, 4, 2, 1] | 5 | [8, 7]
```

然后继续对：

```text
[3, 4, 2, 1]
```

以及：

```text
[8, 7]
```

分别执行快速排序。

因此，快速排序本质上是一种典型的：

```text
分治 Divide and Conquer
```

算法。

它的一般递归框架可以写成：

```python
def quick_sort(arr, left, right):
    if left >= right:
        return

    p = partition(arr, left, right)

    quick_sort(arr, left, p - 1)
    quick_sort(arr, p + 1, right)
```

可以看到，真正决定快速排序实现方式的，其实就是：

```python
partition()
```

接下来我们就从最简单的一路分区开始。

---

# 二、一路快速排序

## 1. 什么是一路快速排序？

一路快速排序又可以理解为：

> 使用一个指针从左向右扫描数组，并维护一个“小于 pivot 的区域”。

假设当前数组为：

```python
[5, 3, 8, 4, 2, 7, 1]
```

选择第一个元素：

```python
pivot = 5
```

作为基准。

我们希望扫描完成后，数组形成：

```text
< pivot | pivot | >= pivot
```

为了实现这一点，可以定义一个指针：

```python
j
```

使得区间：

```text
[left + 1 ... j]
```

始终表示：

```text
< pivot
```

而：

```text
[j + 1 ... i - 1]
```

表示：

```text
>= pivot
```

其中 `i` 是当前正在扫描的元素。

---

## 2. 一路分区过程

假设：

```text
[5, 3, 8, 4, 2, 7, 1]
```

初始状态：

```text
pivot = 5

[5, 3, 8, 4, 2, 7, 1]
    ↑
    i

j = left
```

扫描 `3`：

```text
3 < 5
```

因此把它放入“小于 pivot”的区域。

此时：

```text
[5, 3, 8, 4, 2, 7, 1]
       ↑
       i
```

然后扫描 `8`：

```text
8 >= 5
```

不用处理。

继续扫描 `4`：

```text
4 < 5
```

于是扩大 `< pivot` 区域，将 `4` 与对应位置交换。

最终扫描完成后，可能得到：

```text
[5, 3, 4, 2, 1, 7, 8]
             ↑
             j
```

此时：

```text
[left + 1 ... j] < pivot
```

最后交换：

```python
arr[left], arr[j] = arr[j], arr[left]
```

得到：

```text
[1, 3, 4, 2, 5, 7, 8]
             ↑
           pivot
```

此时 `5` 已经处于最终排序位置。

---

## 3. Python 实现

```python
def partition_one_way(arr, left, right):
    pivot = arr[left]

    # arr[left + 1 ... j] < pivot
    j = left

    for i in range(left + 1, right + 1):
        if arr[i] < pivot:
            j += 1
            arr[j], arr[i] = arr[i], arr[j]

    # 将 pivot 放到最终位置
    arr[left], arr[j] = arr[j], arr[left]

    return j
```

完整快速排序：

```python
def quick_sort_one_way(arr):
    def partition(left, right):
        pivot = arr[left]

        j = left

        for i in range(left + 1, right + 1):
            if arr[i] < pivot:
                j += 1
                arr[j], arr[i] = arr[i], arr[j]

        arr[left], arr[j] = arr[j], arr[left]

        return j

    def sort(left, right):
        if left >= right:
            return

        p = partition(left, right)

        sort(left, p - 1)
        sort(p + 1, right)

    sort(0, len(arr) - 1)

    return arr
```

测试：

```python
nums = [5, 3, 8, 4, 2, 7, 1]

quick_sort_one_way(nums)

print(nums)
```

输出：

```text
[1, 2, 3, 4, 5, 7, 8]
```

---

# 三、一路快速排序存在什么问题？

一路快速排序对于普通数据表现不错。

但是考虑下面这个数组：

```python
[5, 5, 5, 5, 5, 5, 5]
```

如果我们的条件是：

```python
if arr[i] < pivot:
```

那么所有等于 `pivot` 的元素都会被划分到右侧。

第一次 partition 之后可能形成：

```text
5 | 5 5 5 5 5 5
```

递归：

```text
5 | 5 5 5 5 5
```

再递归：

```text
5 | 5 5 5 5
```

也就是说，每次只能减少一个元素。

此时递归树变成：

```text
n
|
n - 1
|
n - 2
|
n - 3
|
...
```

快速排序就会退化为：

\[
O(n^2)
\]

这就是一路快速排序面对大量重复元素时的一个明显问题。

为了解决这个问题，我们可以使用：

> 两路快速排序。

---

# 四、两路快速排序

## 1. 两路快速排序的核心思想

两路快速排序仍然选择一个 `pivot`。

但它使用两个指针：

```text
i →
← j
```

其中：

- `i` 从左向右扫描
- `j` 从右向左扫描

我们的目标是：

```text
左边 <= pivot
右边 >= pivot
```

更准确地说，在扫描过程中：

- `i` 寻找一个 `>= pivot` 的元素
- `j` 寻找一个 `<= pivot` 的元素

然后交换：

```python
arr[i], arr[j] = arr[j], arr[i]
```

继续扫描。

---

## 2. 举个例子

数组：

```text
[4, 7, 3, 4, 6, 2, 4, 5]
```

选择：

```text
pivot = 4
```

初始：

```text
[4, 7, 3, 4, 6, 2, 4, 5]
    ↑                 ↑
    i                 j
```

`i` 从左边找第一个：

```text
>= 4
```

找到：

```text
7
```

`j` 从右边找第一个：

```text
<= 4
```

找到：

```text
4
```

交换：

```text
[4, 4, 3, 4, 6, 2, 7, 5]
```

然后继续向中间扫描。

接下来可能找到：

```text
左边：4
右边：2
```

交换：

```text
[4, 4, 3, 2, 6, 4, 7, 5]
```

最终当：

```text
i >= j
```

时停止。

然后交换：

```python
arr[left], arr[j] = arr[j], arr[left]
```

把 `pivot` 放入最终位置。

---

## 3. Python 实现

```python
def partition_two_way(arr, left, right):
    pivot = arr[left]

    i = left + 1
    j = right

    while True:

        # 从左向右寻找 >= pivot 的元素
        while i <= right and arr[i] < pivot:
            i += 1

        # 从右向左寻找 <= pivot 的元素
        while j >= left + 1 and arr[j] > pivot:
            j -= 1

        if i >= j:
            break

        arr[i], arr[j] = arr[j], arr[i]

        i += 1
        j -= 1

    arr[left], arr[j] = arr[j], arr[left]

    return j
```

完整实现：

```python
def quick_sort_two_way(arr):
    def partition(left, right):
        pivot = arr[left]

        i = left + 1
        j = right

        while True:

            while i <= right and arr[i] < pivot:
                i += 1

            while j >= left + 1 and arr[j] > pivot:
                j -= 1

            if i >= j:
                break

            arr[i], arr[j] = arr[j], arr[i]

            i += 1
            j -= 1

        arr[left], arr[j] = arr[j], arr[left]

        return j

    def sort(left, right):
        if left >= right:
            return

        p = partition(left, right)

        sort(left, p - 1)
        sort(p + 1, right)

    sort(0, len(arr) - 1)

    return arr
```

---

# 五、为什么两路快速排序更适合重复元素？

考虑：

```text
[5, 5, 5, 5, 5, 5, 5]
```

一路快速排序可能会把所有等于 `5` 的元素都留在同一边。

但是两路快速排序中：

```text
i →
← j
```

两个指针都会在遇到：

```text
arr[i] == pivot
```

以及：

```text
arr[j] == pivot
```

时停止。

随后交换，并继续向中间移动。

虽然交换两个相等元素本身看起来没有意义，但是：

> 它可以保证两个指针继续前进，使等于 pivot 的元素被更加均匀地分布到左右两个子区间。

因此对于大量重复元素：

```text
5 5 5 5 5 5 5
```

分区更有可能变成类似：

```text
5 5 5 | 5 | 5 5 5
```

而不是：

```text
5 | 5 5 5 5 5 5
```

递归树会更加平衡。

---

# 六、两路快速排序还有什么问题？

两路快速排序已经可以很好地处理重复元素。

但是考虑一种极端情况：

```python
nums = [5] * 1_000_000
```

这里的所有元素完全相同。

两路快速排序虽然可以保持递归区间相对平衡，但这些：

```text
5
```

仍然会不断参与后续递归。

换句话说：

> 我们明明已经知道这些元素全部等于 pivot，却仍然需要继续排序它们。

有没有办法一次 partition 就直接处理所有等于 `pivot` 的元素？

当然可以。

这就是：

> 三路快速排序。

---

# 七、三路快速排序

三路快速排序也叫：

```text
3-Way Quick Sort
```

它通常使用：

> Dutch National Flag Partition

也就是著名的“荷兰国旗问题”分区思想。

---

## 1. 三路快速排序的核心思想

不同于一路和两路快速排序将数组分成两个部分，三路快速排序直接把数组分成：

```text
< pivot | == pivot | > pivot
```

这三个区域。

例如：

```text
[4, 7, 4, 2, 4, 6, 4, 1]
```

选择：

```text
pivot = 4
```

一次 partition 后，我们希望得到：

```text
[2, 1] | [4, 4, 4, 4] | [7, 6]
```

注意：

中间：

```text
== pivot
```

的这一部分已经完全排好。

因此递归的时候，只需要处理：

```text
< pivot
```

以及：

```text
> pivot
```

两个区域。

---

# 八、三路分区中的三个指针

通常定义三个指针：

```text
lt
i
gt
```

维护下面几个区间：

```text
arr[left + 1 ... lt] < pivot

arr[lt + 1 ... i - 1] == pivot

arr[i ... gt - 1]      未处理

arr[gt ... right]      > pivot
```

可以画成：

```text
pivot | < pivot | == pivot | 未处理 | > pivot
       ---------  ----------          ---------
          lt         i                  gt
```

其中：

```python
lt = left
i = left + 1
gt = right + 1
```

---

# 九、三路快速排序的扫描规则

对于：

```python
arr[i]
```

有三种情况。

---

## 情况一：arr[i] < pivot

如果：

```python
arr[i] < pivot
```

那么它应该进入：

```text
< pivot
```

区域。

因此：

```python
arr[lt + 1], arr[i] = arr[i], arr[lt + 1]

lt += 1
i += 1
```

---

## 情况二：arr[i] > pivot

如果：

```python
arr[i] > pivot
```

那么它应该进入：

```text
> pivot
```

区域。

执行：

```python
arr[i], arr[gt - 1] = arr[gt - 1], arr[i]

gt -= 1
```

这里特别注意：

```python
i
```

不能立即加一。

为什么？

因为从：

```python
gt - 1
```

换过来的元素还没有被检查。

它可能：

```text
< pivot
```

也可能：

```text
== pivot
```

甚至还可能：

```text
> pivot
```

所以必须继续处理当前位置 `i`。

---

## 情况三：arr[i] == pivot

如果：

```python
arr[i] == pivot
```

它已经位于正确区域。

因此只需要：

```python
i += 1
```

---

# 十、Python 实现三路快速排序

```python
def quick_sort_three_way(arr):
    def sort(left, right):
        if left >= right:
            return

        pivot = arr[left]

        # arr[left + 1 ... lt] < pivot
        lt = left

        # arr[gt ... right] > pivot
        gt = right + 1

        # arr[lt + 1 ... i - 1] == pivot
        i = left + 1

        while i < gt:

            if arr[i] < pivot:
                arr[lt + 1], arr[i] = arr[i], arr[lt + 1]

                lt += 1
                i += 1

            elif arr[i] > pivot:
                arr[i], arr[gt - 1] = arr[gt - 1], arr[i]

                gt -= 1

            else:
                i += 1

        # 将 pivot 放入 == pivot 区域
        arr[left], arr[lt] = arr[lt], arr[left]

        # 只递归处理小于和大于 pivot 的部分
        sort(left, lt - 1)
        sort(gt, right)

    sort(0, len(arr) - 1)

    return arr
```

测试：

```python
nums = [4, 7, 4, 2, 4, 6, 4, 1]

quick_sort_three_way(nums)

print(nums)
```

输出：

```text
[1, 2, 4, 4, 4, 4, 6, 7]
```

---

# 十一、三路快速排序为什么适合大量重复元素？

考虑：

```python
nums = [5, 5, 5, 5, 5, 5, 5]
```

三路快速排序第一次 partition 就会直接形成：

```text
< 5 | == 5 | > 5
```

实际上：

```text
< 5
```

为空：

```text
> 5
```

也为空：

而：

```text
== 5
```

包含全部元素。

因此：

```text
[5, 5, 5, 5, 5, 5, 5]
```

只需要扫描一次。

时间复杂度接近：

\[
O(n)
\]

这正是三路快速排序面对大量重复元素时最明显的优势。

---

# 十二、一路、两路、三路快速排序对比

现在可以把三种快速排序放在一起比较。

| 类型 | 分区方式 | 重复元素表现 |
|---|---|---|
| 一路快排 | `< pivot` / `>= pivot` | 较差 |
| 两路快排 | 左右双指针 | 较好 |
| 三路快排 | `<` / `=` / `>` | 最好 |

从结构上看：

一路：

```text
< pivot | pivot | >= pivot
```

两路：

```text
<= pivot | pivot | >= pivot
```

三路：

```text
< pivot | == pivot | > pivot
```

三者最大的区别，并不在递归框架。

它们本质上的区别都在：

```text
partition
```

也就是：

> 如何组织 pivot 周围的元素。

---

# 十三、快速排序的时间复杂度

无论是哪一种快速排序，在理想情况下，每一次 partition 都可以把数组大致分成两半：

```text
n
      /    \
   n/2      n/2
   / \      / \
 n/4 n/4  n/4 n/4
```

递归深度大约为：

\[
\log n
\]

每一层大约扫描：

\[
n
\]

个元素。

所以整体复杂度为：

\[
O(n \log n)
\]

---

## 最坏情况

如果每次 pivot 都是当前区间的最小值或最大值：

```text
n
|
n - 1
|
n - 2
|
...
```

递归深度就会达到：

\[
n
\]

因此复杂度变为：

\[
O(n^2)
\]

这也是快速排序最著名的最坏情况。

---

# 十四、为什么不建议固定选择第一个元素作为 pivot？

前面的代码为了方便理解，都使用：

```python
pivot = arr[left]
```

但是对于已经有序的数组：

```python
[1, 2, 3, 4, 5, 6, 7]
```

如果每次都选择第一个元素：

```text
1
```

那么第一次 partition：

```text
1 | 2 3 4 5 6 7
```

第二次：

```text
2 | 3 4 5 6 7
```

递归树严重失衡。

最终复杂度退化为：

\[
O(n^2)
\]

---

# 十五、随机化 pivot

一个非常常见的优化是：

> 随机选择 pivot。

例如：

```python
import random
```

在 partition 之前：

```python
random_index = random.randint(left, right)

arr[left], arr[random_index] = (
    arr[random_index],
    arr[left],
)
```

这样：

```python
arr[left]
```

就变成了随机选出的 pivot。

---

## 两路随机快速排序

例如：

```python
import random


def quick_sort(arr):
    def partition(left, right):

        random_index = random.randint(left, right)

        arr[left], arr[random_index] = (
            arr[random_index],
            arr[left],
        )

        pivot = arr[left]

        i = left + 1
        j = right

        while True:

            while i <= right and arr[i] < pivot:
                i += 1

            while j >= left + 1 and arr[j] > pivot:
                j -= 1

            if i >= j:
                break

            arr[i], arr[j] = arr[j], arr[i]

            i += 1
            j -= 1

        arr[left], arr[j] = arr[j], arr[left]

        return j

    def sort(left, right):

        if left >= right:
            return

        p = partition(left, right)

        sort(left, p - 1)
        sort(p + 1, right)

    sort(0, len(arr) - 1)

    return arr
```

随机化不能从理论上完全消灭最坏情况，但可以极大降低持续遇到极端划分的概率。

因此随机快速排序的期望时间复杂度通常为：

\[
O(n \log n)
\]

---

# 十六、递归终止条件

快速排序代码中通常都会看到：

```python
if left >= right:
    return
```

这是因为：

当：

```text
left == right
```

时，当前区间只有一个元素。

例如：

```text
[5]
```

单个元素天然已经有序。

如果：

```text
left > right
```

则代表当前区间为空。

所以这两种情况下都不需要继续递归。

---

# 十七、快速排序是稳定排序吗？

快速排序通常：

> 不是稳定排序。

所谓稳定排序，是指两个值相等的元素，在排序前后的相对顺序保持不变。

例如：

```text
5A 3 5B
```

排序之后，如果仍然是：

```text
3 5A 5B
```

那么就是稳定的。

如果变成：

```text
3 5B 5A
```

那么就是不稳定的。

快速排序过程中会发生远距离交换，例如：

```python
arr[i], arr[j] = arr[j], arr[i]
```

因此相同元素的相对顺序可能发生变化。

所以经典快速排序属于：

```text
不稳定排序算法
```

---

# 十八、快速排序的空间复杂度

很多人会说：

```text
快速排序空间复杂度是 O(log n)
```

更准确地说，这是指：

> 平均情况下递归调用栈所占用的空间。

如果递归树比较平衡：

```text
depth ≈ log n
```

那么空间复杂度大约是：

\[
O(\log n)
\]

但如果快速排序退化：

```text
n
|
n - 1
|
n - 2
```

递归深度可能达到：

\[
O(n)
\]

因此最坏情况下递归栈空间复杂度是：

\[
O(n)
\]

---

# 十九、不要把“教学版快速排序”和“原地快速排序”混淆

初学快速排序时，经常会看到下面这种写法：

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr

    pivot = arr[0]

    left = [
        x for x in arr[1:]
        if x <= pivot
    ]

    right = [
        x for x in arr[1:]
        if x > pivot
    ]

    return (
        quick_sort(left)
        + [pivot]
        + quick_sort(right)
    )
```

这个版本非常适合理解思想。

但是它并不是经典意义上的原地快速排序。

因为：

```python
left = [...]
right = [...]
```

每次递归都会创建新的列表。

因此会消耗额外空间。

真正经典的快速排序一般是在原数组中直接交换：

```python
arr[i], arr[j] = arr[j], arr[i]
```

这种实现更加接近实际算法设计中的快速排序。

---

# 二十、什么时候选择一路、两路和三路？

可以给出一个比较实用的经验。

---

## 数据比较随机、重复值较少

可以使用：

```text
一路快速排序
```

它最大的优势是：

```text
实现简单
```

非常适合学习 partition 的基本思想。

---

## 数据存在较多重复元素

推荐：

```text
两路快速排序
```

因为它可以避免相等元素大量堆积在同一侧。

通常比一路快速排序更加稳健。

---

## 数据存在大量重复元素

推荐：

```text
三路快速排序
```

例如：

```python
[1, 1, 1, 1, 2, 2, 2, 3, 3]
```

或者：

```python
[5] * 1_000_000
```

三路快速排序能够在一次 partition 中直接跳过：

```text
== pivot
```

区域。

因此性能通常非常优秀。

---

# 二十一、最终版：随机三路快速排序

如果希望写一个比较稳健的快速排序版本，可以把：

```text
随机 pivot
```

和：

```text
三路 partition
```

结合起来。

代码如下：

```python
import random


def quick_sort(arr):

    def sort(left, right):

        if left >= right:
            return

        # 随机选择 pivot
        random_index = random.randint(left, right)

        arr[left], arr[random_index] = (
            arr[random_index],
            arr[left],
        )

        pivot = arr[left]

        # arr[left + 1 ... lt] < pivot
        lt = left

        # arr[gt ... right] > pivot
        gt = right + 1

        # arr[lt + 1 ... i - 1] == pivot
        i = left + 1

        while i < gt:

            if arr[i] < pivot:

                arr[lt + 1], arr[i] = (
                    arr[i],
                    arr[lt + 1],
                )

                lt += 1
                i += 1

            elif arr[i] > pivot:

                arr[i], arr[gt - 1] = (
                    arr[gt - 1],
                    arr[i],
                )

                gt -= 1

            else:

                i += 1

        # 把 pivot 放入等于区域
        arr[left], arr[lt] = (
            arr[lt],
            arr[left],
        )

        # 中间等于 pivot 的部分无需继续排序
        sort(left, lt - 1)
        sort(gt, right)

    sort(0, len(arr) - 1)

    return arr
```

测试：

```python
nums = [
    5, 3, 5, 2, 8,
    5, 1, 5, 7, 2,
]

quick_sort(nums)

print(nums)
```

输出：

```text
[1, 2, 2, 3, 5, 5, 5, 5, 7, 8]
```

---

# 二十二、从一路到三路，我们到底优化了什么？

回顾整个过程。

最开始，一路快速排序关注的是：

```text
如何把小于 pivot 的元素放到左边？
```

于是得到：

```text
< pivot | pivot | >= pivot
```

但是它遇到大量重复元素时容易产生不平衡分区。

于是我们引入两路快速排序。

两路快速排序考虑：

```text
能不能同时从左右两边寻找放错位置的元素？
```

于是得到：

```text
i →
← j
```

这种方法可以让等于 pivot 的元素更加均匀地分散到左右两侧。

然后我们继续思考：

```text
如果大量元素本来就等于 pivot，
为什么还要继续递归处理它们？
```

于是诞生了三路快速排序：

```text
< pivot | == pivot | > pivot
```

中间区域一次 partition 后就已经完成排序。

因此，从一路到两路，再到三路，本质上是在不断优化：

> partition 的质量。

---

# 二十三、总结

快速排序的核心并不是“递归”本身。

真正值得深入理解的是：

```text
partition
```

整个算法可以概括为：

```text
选择 pivot
      ↓
执行 partition
      ↓
递归左右子区间
```

三种常见实现分别是：

```text
一路快速排序
< pivot | pivot | >= pivot
```

```text
两路快速排序
左右双指针向中间扫描
```

```text
三路快速排序
< pivot | == pivot | > pivot
```

它们之间并不是三种完全不同的算法，而是快速排序在不同数据分布下的不同分区策略。

如果只是学习快速排序：

```text
一路 → 两路 → 三路
```

是一个非常自然的学习顺序。

如果考虑实际使用时的鲁棒性，则通常可以进一步加入：

```text
随机 pivot
```

再搭配：

```text
两路 partition
```

或者：

```text
三路 partition
```

从而降低退化概率。

最后，可以用一句话概括三种快速排序：

> 一路快排解决“如何分区”，两路快排解决“重复元素导致分区失衡”，三路快排进一步解决“为什么还要继续处理已经等于 pivot 的元素”。

理解了这三个阶段，快速排序的整体设计思想也就基本掌握了。