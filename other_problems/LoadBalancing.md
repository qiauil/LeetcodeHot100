# 两类经典负载均衡问题：连续划分与任意划分

在工程实践中，我们经常会遇到这样的负载均衡问题：

给定一组任务，每个任务都有一个正整数权重，例如：

```python
L = [7, 2, 5, 10, 8]
```

这些数字可以表示任务运行时间、文件大小、计算量、请求数量等。

现在希望把这些任务分给 `k` 个处理单元，使得每个处理单元承担一部分任务，并希望：

> **负载最重的处理单元，其总负载尽可能小。**

用数学语言来说，我们希望最小化：

$$
\max(S_1, S_2, \dots, S_k)
$$

其中 \(S_i\) 表示第 \(i\) 个分组中所有元素之和。

根据“分组是否必须保持原数组中的连续性”，这个问题可以分成两类：

1. 每个新数组必须是原数组的连续子数组；
2. 每个新数组可以由原数组中的任意元素组成。

这两个问题的目标看起来完全一样，但它们的算法性质有很大差异。

---

## 问题一：连续子数组划分

### 问题定义

给定一个正整数数组：

$$
L = [a_1, a_2, \dots, a_n]
$$

以及整数：

$$
1 \le k \le n
$$

要求将数组划分成恰好 `k` 个**非空连续子数组**：

$$
P_1, P_2, \dots, P_k
$$

并满足：

- 每个元素恰好属于一个分组；
- 每个分组都是原数组中的连续区间；
- 所有分组保持原数组原有顺序；
- 所有分组覆盖整个数组。

目标是最小化：

$$
\min \max_{1 \le i \le k}
\left(
\sum_{x \in P_i} x
\right)
$$

例如：

```text
L = [7, 2, 5, 10, 8]
k = 2
```

一种划分方法是：

```text
[7, 2, 5] | [10, 8]
```

两个分组的和分别是：

```text
14, 18
```

最大值为：

```text
18
```

另一种划分：

```text
[7, 2] | [5, 10, 8]
```

两个分组的和分别为：

```text
9, 23
```

最大值为 `23`。

因此前一种更好。

事实上，这个例子的最优答案就是：

```text
18
```

### 核心思路：二分答案

这个问题有一个很漂亮的解法：

> 不直接寻找“应该在哪里切”，而是猜测“最大允许负载”是多少。

假设我们猜测：

```text
limit = 18
```

那么问题就变成：

> 能不能把整个数组划分成不超过 `k` 个连续子数组，并且每一个子数组的和都不超过 `18`？

这个可行性问题比原问题简单很多。

### 如何判断一个 limit 是否可行

因为数组中的元素都是正数，我们可以从左到右贪心扫描：

> 当前分组尽可能多地放入元素，只有在继续加入下一个元素会超过 `limit` 时才切分。

例如：

```text
L = [7, 2, 5, 10, 8]
limit = 18
```

从左向右：

```text
7 + 2 + 5 = 14
```

如果继续加入 `10`：

```text
14 + 10 = 24 > 18
```

所以必须切开：

```text
[7, 2, 5]
```

然后处理剩余元素：

```text
10 + 8 = 18
```

最终得到：

```text
[7, 2, 5] | [10, 8]
```

一共只需要两个分组，因此 `limit = 18` 是可行的。

如果：

```text
limit = 17
```

那么会得到：

```text
[7, 2, 5] | [10] | [8]
```

需要三个分组。

如果题目要求最多只能划分成两个分组，那么 `17` 就不可行。

### 为什么可以使用二分搜索

这个问题存在很重要的单调性。

如果 `limit = X` 可以完成划分，那么任何：

$$
Y > X
$$

也一定可以完成划分。

相反，如果 `limit = X` 不可行，那么任何：

$$
Y < X
$$

也一定不可行。

因此，可行性随着 `limit` 增大，会呈现类似：

```text
False False False False True True True True
```

这样的结构。

而这正是二分搜索最适合处理的形式。

### 二分搜索的上下界

答案的下界一定是：

```python
low = max(L)
```

原因是数组中的最大元素无论如何都必须被放进某一个分组，所以最大分组和不可能比它还小。

答案的上界可以设为：

```python
high = sum(L)
```

因为最坏情况下，我们可以把所有元素都放在同一组中。

因此答案一定落在：

$$
[\max(L), \sum L]
$$

之间。

### Python 实现

```python
def split_array_contiguous(nums, k):
    """
    将 nums 划分成最多 k 个连续非空子数组，
    使得子数组和的最大值最小。

    返回最小的最大子数组和。
    """

    def can_split(limit):
        groups = 1
        current_sum = 0

        for x in nums:
            if current_sum + x <= limit:
                current_sum += x
            else:
                groups += 1
                current_sum = x

                if groups > k:
                    return False

        return True

    low = max(nums)
    high = sum(nums)

    while low < high:
        mid = (low + high) // 2

        if can_split(mid):
            high = mid
        else:
            low = mid + 1

    return low
```

测试：

```python
L = [7, 2, 5, 10, 8]

print(split_array_contiguous(L, 2))
```

输出：

```text
18
```

### 为什么“最多 k 组”等价于“恰好 k 组”

代码里的 `can_split()` 判断的是：

> 是否能够划分成不超过 `k` 个分组。

而原问题要求的是恰好 `k` 个非空分组。

在所有元素均为正数、并且 `k <= n` 的情况下，这两者对于求最小最大负载是等价的。

如果某个 `limit` 可以将数组划分成少于 `k` 组，那么我们总可以继续把某些长度大于 1 的连续分组拆开。

由于元素均为正数，拆分以后每个新分组的和都不会大于原来的分组和，因此仍然不会超过 `limit`。

所以：

> 能划分成不超过 `k` 组，就一定也能划分成恰好 `k` 组。

### 如何恢复具体划分

前面的二分搜索只计算了最优的最大负载。

如果还希望得到具体的分组，可以在找到最优 `limit` 后，再扫描一次数组。

```python
def build_contiguous_partition(nums, k, limit):
    result = []
    current = []
    current_sum = 0

    for i, x in enumerate(nums):
        remaining_items = len(nums) - i
        remaining_groups = k - len(result)

        if (
            current
            and (
                current_sum + x > limit
                or remaining_items == remaining_groups
            )
        ):
            result.append(current)
            current = []
            current_sum = 0

        current.append(x)
        current_sum += x

    if current:
        result.append(current)

    return result
```

使用：

```python
L = [7, 2, 5, 10, 8]
k = 2

best = split_array_contiguous(L, k)
groups = build_contiguous_partition(L, k, best)

print(best)
print(groups)
```

输出可能是：

```text
18
[[7, 2, 5], [10, 8]]
```

### 时间复杂度

设：

```text
n = len(L)
S = sum(L)
```

每一次判断 `limit` 是否可行，都需要扫描一次数组：

$$
O(n)
$$

而二分搜索大约需要：

$$
O(\log S)
$$

次。

因此总时间复杂度为：

$$
O(n \log S)
$$

如果只计算最优值，额外空间复杂度为：

$$
O(1)
$$

这使得该算法即使面对很大的数组也非常实用。

---

## 问题二：任意子集划分

### 问题定义

现在取消连续性的限制。

仍然给定：

$$
L = [a_1, a_2, \dots, a_n]
$$

以及整数 `k`。

我们需要将所有元素划分到 `k` 个非空集合：

$$
P_1, P_2, \dots, P_k
$$

满足：

- 每个元素恰好属于一个集合；
- 不要求元素在原数组中连续；
- 元素可以任意重新组合。

目标仍然是：

$$
\min
\max_{1 \le i \le k}
\left(
\sum_{x \in P_i} x
\right)
$$

例如：

```text
L = [7, 2, 5, 10, 8]
k = 2
```

可以划分成：

```text
[10, 5] 和 [8, 7, 2]
```

两个分组的负载分别为：

```text
15 和 17
```

因此最大负载为：

```text
17
```

注意：

```text
[10, 5]
```

并不是原数组中的连续子数组。

所以这种划分在问题一中不允许，在问题二中却完全合法。

### 这个问题为什么更难

这个问题实际上对应经典的：

> **Multiprocessor Scheduling Problem**

也可以看作：

> **Makespan Minimization**

如果把每个元素理解成一个任务的执行时间，把 `k` 个分组理解成 `k` 台相同的机器，那么我们希望最小化：

> 所有机器中最晚完成工作的那台机器的完成时间。

这里有一个非常重要的计算复杂度差异。

对于连续划分问题，我们可以利用数组顺序构造出一个高效的可行性判断。

但对于任意划分问题，每一个任务都可以被分配给任意一个分组。

粗略来看，每个元素都有 `k` 种选择，因此可能存在：

$$
k^n
$$

种分配方案。

即使只有：

```text
k = 2
```

它也和经典的 Partition Problem 紧密相关。

因此，一般形式下，这个问题是 NP-hard 的。

这意味着我们通常不能期待存在一个像问题一那样简单的多项式时间算法，在任意输入上都快速求出最优解。

实际工程中通常有两类做法：

- 数据规模较大：使用贪心或近似算法；
- 数据规模较小且必须最优：使用回溯、动态规划或 Branch and Bound。

### 工程中常用的方法：LPT

一种简单而且非常实用的算法是：

> 每次把当前最大的任务分配给当前总负载最小的机器。

这个算法叫做：

> **Longest Processing Time First，LPT**

首先将任务从大到小排序：

```text
[10, 8, 7, 5, 2]
```

假设有两台机器。

初始状态：

```text
machine 1 = 0
machine 2 = 0
```

放入 `10`：

```text
machine 1 = 10
machine 2 = 0
```

放入 `8`：

```text
machine 1 = 10
machine 2 = 8
```

接下来 `7` 放入当前负载更小的第二台机器：

```text
machine 1 = 10
machine 2 = 15
```

然后 `5` 放入第一台：

```text
machine 1 = 15
machine 2 = 15
```

最后 `2` 可以放入任意一台：

```text
machine 1 = 17
machine 2 = 15
```

最终最大负载为：

```text
17
```

在这个例子中，LPT 恰好找到了最优解。

### Python 实现 LPT

使用 Python 的最小堆非常适合实现这个算法。

堆中始终保存：

```text
(当前负载, 机器编号)
```

于是每次都可以快速找到当前负载最小的机器。

```python
import heapq


def partition_lpt(nums, k):
    """
    使用 LPT 贪心算法将 nums 分给 k 个组。

    返回：
    max_load: 最大组负载
    groups: 每个分组包含的元素
    """

    if k <= 0:
        raise ValueError("k must be positive")

    if k > len(nums):
        raise ValueError("k cannot exceed number of items")

    nums = sorted(nums, reverse=True)

    heap = [(0, i) for i in range(k)]
    heapq.heapify(heap)

    groups = [[] for _ in range(k)]

    for x in nums:
        load, machine = heapq.heappop(heap)

        groups[machine].append(x)
        load += x

        heapq.heappush(heap, (load, machine))

    loads = [sum(group) for group in groups]

    return max(loads), groups
```

使用：

```python
L = [7, 2, 5, 10, 8]

max_load, groups = partition_lpt(L, 2)

print(max_load)
print(groups)
```

可能得到：

```text
17
[[10, 5, 2], [8, 7]]
```

两组负载分别为：

```text
17 和 15
```

### LPT 的时间复杂度

首先需要排序：

$$
O(n \log n)
$$

随后，每个元素都需要进行一次堆的弹出和插入操作。

堆中有 `k` 个元素，因此单次操作复杂度为：

$$
O(\log k)
$$

一共处理 `n` 个任务：

$$
O(n \log k)
$$

因此总复杂度为：

$$
O(n \log n + n \log k)
$$

由于通常：

$$
k \le n
$$

所以一般可以简单写成：

$$
O(n \log n)
$$

### LPT 一定能得到最优解吗

不能。

这是第二类问题和第一类问题最重要的区别之一。

LPT 是一个非常实用的近似算法，但不保证对所有输入都返回全局最优解。

例如：

```text
L = [3, 3, 3, 4, 4, 5]
k = 2
```

按照 LPT 排序：

```text
5, 4, 4, 3, 3, 3
```

算法可能得到：

```text
[5, 4, 3] = 12
[4, 3, 3] = 10
```

最大负载为：

```text
12
```

但实际上存在更好的划分：

```text
[5, 3, 3] = 11
[4, 4, 3] = 11
```

所以最优答案是：

```text
11
```

这说明贪心算法虽然快，但无法保证最优。

### 如果必须求精确解

如果数组规模较小，可以使用：

> 回溯 + 剪枝。

核心思想是逐个决定每个任务属于哪个分组。

最直接的暴力搜索需要考虑大量组合，因此必须加入剪枝。

一个常见策略是：

1. 先把任务从大到小排序；
2. 依次尝试将当前任务放入不同分组；
3. 维护当前已知的最好答案；
4. 如果某个搜索分支已经不可能更优，就立即停止；
5. 对负载相同的分组进行对称状态剪枝。

Python 实现如下：

```python
def partition_exact(nums, k):
    """
    使用回溯 + 剪枝求任意划分问题的精确解。

    适合 n 较小的情况。
    """

    if k <= 0:
        raise ValueError("k must be positive")

    if k > len(nums):
        raise ValueError("k cannot exceed number of items")

    nums = sorted(nums, reverse=True)

    loads = [0] * k
    groups = [[] for _ in range(k)]

    best_max = sum(nums)
    best_groups = None

    def dfs(index):
        nonlocal best_max, best_groups

        if index == len(nums):
            current_max = max(loads)

            if current_max < best_max:
                best_max = current_max
                best_groups = [g[:] for g in groups]

            return

        x = nums[index]

        seen_loads = set()

        for i in range(k):
            # 相同负载的机器是对称状态，没有必要重复尝试
            if loads[i] in seen_loads:
                continue

            seen_loads.add(loads[i])

            new_load = loads[i] + x

            # 已经不可能优于当前最优结果
            if new_load >= best_max:
                continue

            loads[i] += x
            groups[i].append(x)

            dfs(index + 1)

            groups[i].pop()
            loads[i] -= x

            # 所有空机器彼此等价
            if loads[i] == 0:
                break

    dfs(0)

    return best_max, best_groups
```

使用：

```python
L = [7, 2, 5, 10, 8]

best, groups = partition_exact(L, 2)

print(best)
print(groups)
```

可能输出：

```text
17
[[10, 5, 2], [8, 7]]
```

这里得到的是精确最优解。

### 为什么要先从大到小排序

代码中首先执行：

```python
nums = sorted(nums, reverse=True)
```

原因在于：

> 大任务对最终结果影响最大。

如果优先处理大任务，那么不合理的分配往往会更早暴露出来。

例如某个大任务加入当前分组以后已经使负载超过当前最优解，那么整个搜索分支就可以立即被剪掉。

因此，在组合搜索问题中：

> 优先处理约束最强、影响最大的元素

通常能显著提高剪枝效率。

### 一个非常重要的理论下界

对于任意划分问题，最优答案 `OPT` 至少满足两个条件。

首先：

$$
OPT \ge \max(L)
$$

因为最大的单个任务必须完整地分配给某一台机器。

其次：

$$
OPT \ge
\left\lceil
\frac{\sum L}{k}
\right\rceil
$$

因为总负载是：

$$
\sum L
$$

即使所有任务能够完美平均分配，每台机器