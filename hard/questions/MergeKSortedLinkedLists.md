# 合并 K 个升序链表

给定一个包含 `k` 个链表的数组 `lists`，其中每个链表都已经按照**升序**排列。

请将所有链表合并成一个新的**升序链表**，并返回合并后链表的头节点。

------

## 示例

### 示例 1

```text
输入：
lists = [[1,2,4],[1,3,5],[3,6]]

输出：
[1,1,2,3,3,4,5,6]
```

### 示例 2

```text
输入：
lists = []

输出：
[]
```

### 示例 3

```text
输入：
lists = [[]]

输出：
[]
```

------

# 一、需要掌握的基础知识

在做这道题之前，最好熟悉以下内容：

- **单向链表（Singly Linked List）**
  - 链表节点结构
  - 链表遍历
  - 修改 `next` 指针
- **合并两个有序链表**
  - 这是本题绝大多数高效解法的核心子问题
- **最小堆 / 优先队列（Min Heap / Priority Queue）**
  - 用于快速找到 `k` 个候选节点中的最小值
- **分治（Divide and Conquer）**
  - 将 `k` 个链表不断两两合并
- **排序（Sorting）**
  - 最暴力的方法会先收集所有节点值，再统一排序

------

# 二、核心问题分析

假设：

- `k` = 链表的数量
- `n` = 所有链表中的**节点总数**

这道题真正需要解决的问题是：

> 每一次应该如何高效地从 `k` 个有序链表中找到当前最小的节点？

不同解法的区别，基本就在这里。

例如：

- 暴力排序：根本不利用链表已经有序这一条件
- 暴力扫描：每次检查 `k` 个链表的头节点
- 最小堆：用 `O(log k)` 找到当前最小节点
- 分治：避免每次从 `k` 个候选节点里找最小，而是不断进行“两两合并”

面试中最重要的通常是：

1. **最小堆：`O(n log k)`**
2. **分治：`O(n log k)`**

其中分治迭代版本通常非常值得掌握。

------

# 三、解法 1：暴力收集 + 排序

## 思路

最简单的方法是完全忽略链表已经有序这一特点。

我们可以：

1. 遍历所有链表
2. 把所有节点的值放入 Python 列表
3. 对这些值统一排序
4. 根据排序结果重新创建一个新的链表

这种方法非常直观，但没有利用输入数据“每个链表本身已经有序”的重要条件。

因此通常不会是面试中的理想答案。

------

## 算法步骤

1. 创建空数组 `nodes`
2. 遍历 `lists` 中的每一个链表
3. 遍历链表中的所有节点，将 `node.val` 加入 `nodes`
4. 调用 `nodes.sort()` 排序
5. 创建一个 dummy node（虚拟头节点）
6. 根据排序后的值逐个创建新节点
7. 返回 `dummy.next`

------

## Python 代码

```python
# 单向链表节点定义
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:

        nodes = []

        # 收集所有链表中的节点值
        for lst in lists:
            while lst:
                nodes.append(lst.val)
                lst = lst.next

        # 对所有节点值统一排序
        nodes.sort()

        # dummy 是虚拟头节点
        dummy = ListNode(0)
        cur = dummy

        # 根据排序后的值重新创建链表
        for value in nodes:
            cur.next = ListNode(value)
            cur = cur.next

        return dummy.next
```

------

## 复杂度分析

排序 `n` 个元素需要：

```text
O(n log n)
```

因此：

- **时间复杂度：`O(n log n)`**
- **空间复杂度：`O(n)`**

这里不仅需要保存所有节点值，还重新创建了 `n` 个链表节点。

------

## 优缺点

优点：

- 思路最简单
- 容易实现
- 不容易写错

缺点：

- 没有利用每条链表已经有序的特点
- 重新创建了全部节点
- 时间复杂度不如 `O(n log k)` 的最优方法

------

# 四、解法 2：每次扫描所有链表头节点

## 思路

因为每个链表本身已经有序，所以整个链表中当前还没有处理的最小元素，一定出现在某个链表的**头节点**。

因此我们可以：

1. 查看所有非空链表的头节点
2. 找到其中最小的节点
3. 将它连接到答案链表
4. 将对应链表的指针向后移动一位
5. 重复这个过程

例如当前有：

```text
L1: 1 -> 4 -> 7
L2: 2 -> 5 -> 8
L3: 3 -> 6 -> 9
```

只需要比较：

```text
1, 2, 3
```

找到 `1`。

取出 `1` 后：

```text
L1: 4 -> 7
L2: 2 -> 5 -> 8
L3: 3 -> 6 -> 9
```

下一轮比较：

```text
4, 2, 3
```

找到 `2`。

不断重复即可。

问题在于：

> 每取出一个节点，都需要扫描最多 `k` 个链表。

所以效率比较低。

------

## Python 代码

```python
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:

        dummy = ListNode(0)
        cur = dummy

        while True:
            # min_index 表示当前最小头节点来自哪个链表
            min_index = -1

            # 扫描所有链表的当前头节点
            for i in range(len(lists)):

                # 当前链表已经为空
                if lists[i] is None:
                    continue

                # 第一个有效节点，或者找到了更小的节点
                if (
                    min_index == -1
                    or lists[i].val < lists[min_index].val
                ):
                    min_index = i

            # 所有链表都已经为空
            if min_index == -1:
                break

            # 把当前最小节点直接接到结果链表
            cur.next = lists[min_index]

            # 对应链表向后移动
            lists[min_index] = lists[min_index].next

            # 结果链表尾指针向后移动
            cur = cur.next

        return dummy.next
```

------

## 复杂度分析

一共有 `n` 个节点。

每取出一个节点，需要检查最多 `k` 个链表头节点：

```text
O(k)
```

总共执行 `n` 次：

```text
O(nk)
```

因此：

- **时间复杂度：`O(nk)`**
- **额外空间复杂度：`O(1)`**

这里直接复用了原链表节点。

------

# 五、解法 3：依次合并链表

## 思路

另一种方法是：

> 不一次处理 `k` 个链表，而是每次只合并两个链表。

例如有：

```text
L1
L2
L3
L4
```

可以：

```text
merge(L1, L2) -> M1

merge(M1, L3) -> M2

merge(M2, L4) -> Answer
```

这样问题就转化为了一个非常经典的问题：

> 如何合并两个升序链表？

------

# 六、核心子问题：合并两个升序链表

这个操作需要非常熟练，因为后面的分治解法也会不断使用它。

假设：

```text
l1: 1 -> 4 -> 7
l2: 2 -> 3 -> 8
```

比较：

```text
1 vs 2
```

选择 `1`：

```text
result: 1
l1: 4 -> 7
l2: 2 -> 3 -> 8
```

然后：

```text
4 vs 2
```

选择 `2`：

```text
result: 1 -> 2
```

不断重复。

当其中一个链表为空后，可以直接把另一个链表剩余部分全部接到结果后面。

------

## Dummy Node 是什么？

这里经常会使用：

```python
dummy = ListNode()
tail = dummy
```

`dummy` 是一个**虚拟头节点**。

它不属于真正结果的一部分，主要目的是：

> 避免对“第一个节点”进行特殊处理。

例如如果不用 dummy，我们就需要判断：

```python
if head is None:
    head = node
else:
    tail.next = node
```

有了 dummy 后，所有节点都可以统一写成：

```python
tail.next = node
tail = tail.next
```

最后：

```python
return dummy.next
```

即可。

这是链表题中非常常见的技巧。

------

## Python 代码

```python
class Solution:
    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:

        if len(lists) == 0:
            return None

        # 依次将前面合并好的结果和下一个链表合并
        for i in range(1, len(lists)):
            lists[i] = self.mergeList(
                lists[i - 1],
                lists[i]
            )

        return lists[-1]

    def mergeList(self, l1, l2):
        dummy = ListNode()
        tail = dummy

        # 两个链表都还有节点时进行比较
        while l1 and l2:
            if l1.val < l2.val:
                tail.next = l1
                l1 = l1.next
            else:
                tail.next = l2
                l2 = l2.next

            tail = tail.next

        # 其中一个链表已经为空，
        # 直接接上另一个链表剩余的部分
        if l1:
            tail.next = l1
        else:
            tail.next = l2

        return dummy.next
```

------

## 为什么这个方法可能很慢？

假设 `k` 个链表长度比较接近。

第一次：

```text
L1 + L2
```

可能处理大约：

```text
2n/k
```

个节点。

第二次：

```text
(L1 + L2) + L3
```

处理：

```text
3n/k
```

个节点。

之后越来越长：

```text
4n/k
5n/k
...
```

前面已经合并好的节点会被**反复遍历**。

最坏情况下总复杂度可达到：

- **时间复杂度：`O(nk)`**
- **额外空间复杂度：`O(1)`**

这正是后面分治法需要解决的问题。

------

# 七、解法 4：最小堆 / 优先队列

## 思路

回顾解法 2。

我们每次都需要找到：

> `k` 个链表当前头节点中的最小值。

暴力方法需要扫描：

```text
O(k)
```

如果使用**最小堆（Min Heap）**，就可以把这个操作优化到：

```text
O(log k)
```

------

## 堆中保存什么？

我们不需要把所有 `n` 个节点都放进堆。

因为每个链表本身已经有序，只需要维护：

> 每个链表当前还没有处理的第一个节点。

因此堆的大小最多只有：

```text
k
```

例如：

```text
L1: 1 -> 4 -> 7
L2: 2 -> 5 -> 8
L3: 3 -> 6 -> 9
```

一开始只把：

```text
1, 2, 3
```

放进最小堆。

弹出 `1` 后，将 `1.next = 4` 加进去：

```text
2, 3, 4
```

弹出 `2` 后加入 `5`：

```text
3, 4, 5
```

这样不断重复。

------

## 算法步骤

1. 创建一个最小堆
2. 将每个非空链表的头节点加入堆
3. 创建 dummy node 和尾指针 `cur`
4. 不断执行：
   - 弹出堆中最小节点
   - 将它接到结果链表
   - 如果这个节点存在 `next`
   - 将 `next` 加入堆
5. 当堆为空时结束
6. 返回 `dummy.next`

------

# 八、Python 的 `heapq`

Python 标准库提供：

```python
import heapq
```

`heapq` 实现的是：

> **最小堆（min-heap）**

例如：

```python
heap = []

heapq.heappush(heap, 5)
heapq.heappush(heap, 2)
heapq.heappush(heap, 8)

print(heapq.heappop(heap))
```

输出：

```text
2
```

常见操作：

```python
heapq.heappush(heap, value)
```

向堆中加入元素。

```python
heapq.heappop(heap)
```

删除并返回最小元素。

二者复杂度都是：

```text
O(log k)
```

其中 `k` 是当前堆中元素数量。

> 注意：Python 的 `heapq` **默认就是最小堆**。如果想模拟最大堆，常见技巧才是存储负数，例如 `-value`。

------

# 九、为什么需要 `NodeWrapper`？

下面这种写法：

```python
heapq.heappush(heap, node)
```

不一定可行。

原因在于当 Python 的堆需要比较两个 `ListNode` 时，它不知道：

```python
node1 < node2
```

究竟应该如何比较。

因此可以定义一个包装类：

```python
class NodeWrapper:
    def __init__(self, node):
        self.node = node

    def __lt__(self, other):
        return self.node.val < other.node.val
```

这里最重要的是：

```python
__lt__
```

------

## `__lt__` 是什么？

`__lt__` 是 Python 的特殊方法（dunder method）。

`lt` 表示：

```text
less than
```

也就是：

```text
<
```

当 Python 执行：

```python
a < b
```

时，本质上可以调用：

```python
a.__lt__(b)
```

所以定义：

```python
def __lt__(self, other):
    return self.node.val < other.node.val
```

等价于告诉 Python：

> 比较两个 `NodeWrapper` 时，根据其中链表节点的 `val` 进行比较。

这样 `heapq` 就知道哪个节点应该排在前面。

------

## Python 代码

```python
import heapq


# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class NodeWrapper:
    def __init__(self, node):
        self.node = node

    # 定义两个 NodeWrapper 之间的“小于”关系
    # heapq 会利用它维护最小堆
    def __lt__(self, other):
        return self.node.val < other.node.val


class Solution:
    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:

        if not lists:
            return None

        min_heap = []

        # 每个非空链表先放入头节点
        for lst in lists:
            if lst is not None:
                heapq.heappush(
                    min_heap,
                    NodeWrapper(lst)
                )

        dummy = ListNode(0)
        cur = dummy

        while min_heap:
            # 取出当前所有候选节点中的最小节点
            wrapper = heapq.heappop(min_heap)
            node = wrapper.node

            # 接到结果链表
            cur.next = node
            cur = cur.next

            # 当前节点来自某个有序链表
            # 它的下一个节点现在成为新的候选节点
            if node.next:
                heapq.heappush(
                    min_heap,
                    NodeWrapper(node.next)
                )

        return dummy.next
```

------

## 复杂度分析

每个节点：

- 进入堆一次
- 离开堆一次

堆中最多只有 `k` 个元素，因此一次 push/pop：

```text
O(log k)
```

总共有 `n` 个节点：

```text
O(n log k)
```

因此：

- **时间复杂度：`O(n log k)`**
- **空间复杂度：`O(k)`**

这是本题的标准最优解之一。

------

# 十、Python 中另一种更常见的 Heap 写法

实际面试中，也可以避免自己定义 `NodeWrapper`。

一种很常见的写法是把：

```python
(node.val, i, node)
```

放进堆中。

其中：

- `node.val`：负责按照节点值排序
- `i`：链表编号，用作 tie-breaker
- `node`：真正的链表节点

为什么需要 `i`？

假设两个节点的值相同：

```text
1
1
```

如果堆元素只有：

```python
(node.val, node)
```

Python 比较完第一个元素发现：

```text
1 == 1
```

之后可能尝试比较：

```python
node1 < node2
```

而 `ListNode` 默认不能比较。

加入唯一的 `i` 后：

```python
(node.val, i, node)
```

Python 可以通过 `i` 打破平局，因此不需要比较 `ListNode`。

```python
import heapq


class Solution:
    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:

        min_heap = []

        # (节点值, 链表编号, 节点)
        for i, node in enumerate(lists):
            if node:
                heapq.heappush(
                    min_heap,
                    (node.val, i, node)
                )

        dummy = ListNode(0)
        tail = dummy

        while min_heap:
            value, i, node = heapq.heappop(min_heap)

            tail.next = node
            tail = tail.next

            if node.next:
                heapq.heappush(
                    min_heap,
                    (node.next.val, i, node.next)
                )

        return dummy.next
```

这个版本在 Python 面试中也非常常见。

------

# 十一、解法 5：分治法（递归）

## 思路

这一方法和**归并排序 Merge Sort** 的思想非常相似。

如果有：

```text
L1 L2 L3 L4 L5 L6 L7 L8
```

不要按照：

```text
(((((((L1 + L2) + L3) + L4) + ...)))
```

依次合并。

而是：

```text
L1 + L2
L3 + L4
L5 + L6
L7 + L8
```

得到：

```text
M1 M2 M3 M4
```

然后：

```text
M1 + M2
M3 + M4
```

得到：

```text
N1 N2
```

最后：

```text
N1 + N2
```

得到最终结果。

结构类似：

```text
          最终结果
         /       \
       N1         N2
      /  \       /  \
    M1   M2    M3   M4
   / \   / \   / \   / \
 L1 L2 L3 L4 L5 L6 L7 L8
```

这就是分治。

------

## 为什么分治更高效？

假设一共有：

```text
k
```

个链表。

每一轮链表数量大约减半：

```text
k
k / 2
k / 4
k / 8
...
1
```

所以一共有：

```text
log₂ k
```

层。

而每一层中，所有链表节点总共只会被处理一次，因此每层：

```text
O(n)
```

总复杂度：

```text
O(n log k)
```

------

## 算法步骤

### Base Case

如果：

```python
l == r
```

说明范围里只剩一个链表：

```python
return lists[l]
```

------

### Divide

计算中点：

```python
mid = l + (r - l) // 2
```

然后递归处理：

```python
left = divide(lists, l, mid)
right = divide(lists, mid + 1, r)
```

------

### Conquer

左右两边分别已经变成一个有序链表：

```text
left
right
```

再调用：

```text
merge two sorted lists
```

把它们合并。

------

## Python 代码

```python
class Solution:
    def mergeKLists(self, lists):

        if not lists:
            return None

        return self.divide(
            lists,
            0,
            len(lists) - 1
        )

    def divide(self, lists, left, right):

        # 范围为空
        if left > right:
            return None

        # 只剩一个链表，不需要继续拆分
        if left == right:
            return lists[left]

        mid = left + (right - left) // 2

        # 分别合并左右两部分
        l1 = self.divide(lists, left, mid)
        l2 = self.divide(lists, mid + 1, right)

        # 将两个已经排好序的链表合并
        return self.mergeTwoLists(l1, l2)

    def mergeTwoLists(self, l1, l2):

        dummy = ListNode(0)
        cur = dummy

        while l1 and l2:
            if l1.val <= l2.val:
                cur.next = l1
                l1 = l1.next
            else:
                cur.next = l2
                l2 = l2.next

            cur = cur.next

        # 剩余部分已经有序，可以直接连接
        if l1:
            cur.next = l1
        else:
            cur.next = l2

        return dummy.next
```

------

## 复杂度分析

一共有约：

```text
log k
```

层。

每一层总共处理：

```text
n
```

个节点。

所以：

- **时间复杂度：`O(n log k)`**

递归树的最大递归深度为：

```text
O(log k)
```

因此：

- **额外空间复杂度：`O(log k)`**

这里不考虑输出链表本身，因为原来的链表节点被直接复用了。

------

# 十二、解法 6：分治法（迭代）

## 思路

这一方法和递归分治完全一样，只是不再通过递归完成。

每一轮都将链表**两两合并**。

例如：

```text
L1 L2 L3 L4 L5 L6
```

第一轮：

```text
merge(L1, L2) -> M1
merge(L3, L4) -> M2
merge(L5, L6) -> M3
```

现在：

```text
M1 M2 M3
```

第二轮：

```text
merge(M1, M2) -> N1
merge(M3, None) -> M3
```

现在：

```text
N1 M3
```

第三轮：

```text
merge(N1, M3)
```

得到最终答案。

------

## 算法步骤

1. 如果 `lists` 为空，返回 `None`
2. 当：

```python
len(lists) > 1
```

时不断执行：

3. 创建：

```python
mergedLists = []
```

1. 每两个链表为一组：
   - `lists[i]`
   - `lists[i + 1]`
2. 如果最后只有一个链表没有配对，则让第二个链表为 `None`
3. 合并后放入 `mergedLists`
4. 令：

```python
lists = mergedLists
```

1. 最终只剩一个链表，返回：

```python
lists[0]
```

------

## Python 代码

```python
class Solution:

    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:

        if not lists:
            return None

        # 每一轮将链表数量大约减半
        while len(lists) > 1:

            merged_lists = []

            # 每两个链表为一组进行合并
            for i in range(0, len(lists), 2):

                l1 = lists[i]

                # 如果链表数量为奇数，
                # 最后一个链表没有配对对象
                l2 = (
                    lists[i + 1]
                    if i + 1 < len(lists)
                    else None
                )

                merged_lists.append(
                    self.mergeList(l1, l2)
                )

            # 进入下一轮
            lists = merged_lists

        return lists[0]

    def mergeList(self, l1, l2):

        dummy = ListNode()
        tail = dummy

        while l1 and l2:

            if l1.val < l2.val:
                tail.next = l1
                l1 = l1.next

            else:
                tail.next = l2
                l2 = l2.next

            tail = tail.next

        # 直接连接剩余链表
        if l1:
            tail.next = l1
        else:
            tail.next = l2

        return dummy.next
```

------

## 复杂度分析

每一轮所有节点整体会被处理一次：

```text
O(n)
```

链表数量每轮减半：

```text
k -> k/2 -> k/4 -> ... -> 1
```

因此有：

```text
O(log k)
```

轮。

最终：

- **时间复杂度：`O(n log k)`**
- **辅助空间复杂度：`O(k)`**

这里的 `O(k)` 主要来自每一轮创建的：

```python
merged_lists
```

如果进一步原地操作 `lists`，辅助空间还可以降低。

------

# 十三、Heap 与 Divide and Conquer 的比较

这两个方法都是：

```text
O(n log k)
```

也是本题最重要的两种解法。

| 方法           | 时间复杂度   | 额外空间   | 特点                   |
| -------------- | ------------ | ---------- | ---------------------- |
| 暴力排序       | `O(n log n)` | `O(n)`     | 最简单，但浪费有序信息 |
| 扫描所有头节点 | `O(nk)`      | `O(1)`     | 简单但慢               |
| 依次合并       | `O(nk)`      | `O(1)`     | 容易想到，但重复遍历   |
| Min Heap       | `O(n log k)` | `O(k)`     | 非常经典               |
| 递归分治       | `O(n log k)` | `O(log k)` | 类似 Merge Sort        |
| 迭代分治       | `O(n log k)` | `O(k)`     | 面试中非常推荐         |

------

# 十四、面试中推荐掌握的答案：迭代分治

如果让我选择一个作为面试中的主答案，我会优先掌握：

```text
Divide and Conquer + Merge Two Sorted Lists
```

尤其是**迭代版本**。

原因是：

- 时间复杂度最优：`O(n log k)`
- 不需要额外的数据结构知识
- 核心逻辑建立在经典的“两链表合并”之上
- 很容易解释为什么是 `O(n log k)`
- 不涉及 Python 对 heap 元素比较规则的问题
- 很好地体现了分治思想

可以把整个思路记成一句话：

> **不要从左到右一个个合并，而是像 Merge Sort 一样，两两合并。**

------

# 十五、为什么「依次合并」是 `O(nk)`，而「两两合并」是 `O(n log k)`？

这是本题最值得真正理解的地方。

假设：

```text
k = 8
```

而且每个链表都有：

```text
m
```

个节点。

所以：

```text
n = 8m
```

------

## 依次合并

第一次：

```text
L1 + L2
```

处理约：

```text
2m
```

个节点。

第二次：

```text
(L1 + L2) + L3
```

处理：

```text
3m
```

第三次：

```text
4m
```

依次：

```text
2m + 3m + 4m + ... + 8m
```

总量近似：

```text
O(mk²)
```

因为：

```text
n = mk
```

所以：

```text
O(nk)
```

------

## 两两合并

第一层：

```text
L1 + L2
L3 + L4
L5 + L6
L7 + L8
```

总共处理：

```text
8m = n
```

第二层：

```text
M1 + M2
M3 + M4
```

仍然总共：

```text
n
```

第三层：

```text
N1 + N2
```

还是：

```text
n
```

层数：

```text
log₂ k
```

所以：

```text
n × log k
```

即：

```text
O(n log k)
```

这也是为什么**平衡地合并**比**从左到右不断累加合并**更高效。

------

# 十六、常见错误

## 1. 没有处理空链表

输入中完全可能存在：

```python
lists = [
    None,
    head1,
    None,
    head2
]
```

所以不能直接假设：

```python
lists[i].val
```

一定存在。

Heap 解法中应该先判断：

```python
if node:
    ...
```

------

## 2. 忘记移动被选择链表的指针

例如暴力扫描方法找到：

```python
lists[min_index]
```

之后一定要执行：

```python
lists[min_index] = lists[min_index].next
```

否则下一轮依然会选择同一个节点，从而进入死循环。

------

## 3. Heap 中直接存 `ListNode`

下面的代码可能出问题：

```python
heapq.heappush(heap, node)
```

因为 Python 的 `ListNode` 默认不支持：

```python
node1 < node2
```

可以使用：

```python
NodeWrapper
```

或者更常见地使用：

```python
(node.val, index, node)
```

------

## 4. 误解 Python `heapq`

Python 的：

```python
heapq
```

默认是：

> **Min Heap**

也就是：

```python
heapq.heappop(heap)
```

会取出最小值。

如果要模拟 Max Heap，才经常写成：

```python
heapq.heappush(heap, -value)
```

然后取出时重新取负数。

------

## 5. 忘记使用 Dummy Node

对于链表构造题，非常推荐：

```python
dummy = ListNode()
tail = dummy
```

这样不需要单独判断：

> “当前加入的是不是结果链表的第一个节点？”

最终统一：

```python
return dummy.next
```

即可。

------

## 6. 分治时没有正确处理奇数个链表

例如：

```text
L1 L2 L3 L4 L5
```

最后的：

```text
L5
```

没有配对链表。

可以让：

```python
l2 = None
```

然后：

```python
mergeList(L5, None)
```

自然返回 `L5`。

------

## 7. 不小心丢失链表剩余部分

合并两个链表的时候：

```python
while l1 and l2:
```

结束之后，通常还有一个链表没有遍历结束。

千万不要忘记：

```python
tail.next = l1 if l1 else l2
```

实际上可以把：

```python
if l1:
    tail.next = l1
else:
    tail.next = l2
```

简写为：

```python
tail.next = l1 or l2
```

完整写法：

```python
def mergeTwoLists(self, l1, l2):
    dummy = ListNode()
    tail = dummy

    while l1 and l2:
        if l1.val <= l2.val:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next

        tail = tail.next

    # l1 和 l2 至少有一个已经是 None
    tail.next = l1 or l2

    return dummy.next
```

------

# 十七、面试记忆模板

这道题可以按照下面的逻辑快速回忆。

## 方法一：Min Heap

核心一句话：

> **Heap 中始终保存每个链表当前最小的候选节点。**

模板：

```python
heap = []

for each list:
    push its head

while heap:
    node = pop smallest

    append node to answer

    if node.next:
        push node.next
```

复杂度：

```text
Time:  O(n log k)
Space: O(k)
```

------

## 方法二：Divide and Conquer

核心一句话：

> **像 Merge Sort 一样，每轮将链表两两合并。**

模板：

```python
while len(lists) > 1:

    merged = []

    for every two lists:
        merged.append(
            mergeTwoLists(l1, l2)
        )

    lists = merged
```

复杂度：

```text
Time:  O(n log k)
Space: O(k)
```

------

# 十八、推荐面试代码

如果需要在代码面试中快速写出一个结构清晰、容易解释的版本，我推荐下面这个：

```python
class Solution:
    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:

        # Edge case：没有任何链表
        if not lists:
            return None

        # 每一轮把链表两两合并
        # k -> k/2 -> k/4 -> ... -> 1
        while len(lists) > 1:

            merged = []

            for i in range(0, len(lists), 2):

                l1 = lists[i]

                # 如果当前链表没有配对对象，
                # 让 l2 = None
                l2 = (
                    lists[i + 1]
                    if i + 1 < len(lists)
                    else None
                )

                merged.append(
                    self.mergeTwoLists(l1, l2)
                )

            lists = merged

        return lists[0]

    def mergeTwoLists(self, l1, l2):

        # Dummy node 避免处理头节点的特殊情况
        dummy = ListNode()
        tail = dummy

        # 每次选择较小的节点
        while l1 and l2:

            if l1.val <= l2.val:
                tail.next = l1
                l1 = l1.next

            else:
                tail.next = l2
                l2 = l2.next

            tail = tail.next

        # 直接接上剩余链表
        tail.next = l1 or l2

        return dummy.next
```

复杂度：

```text
Time:  O(n log k)
Space: O(k)
```

其中：

```text
n = 所有链表中的节点总数
k = 链表数量
```

最值得记住的是：

```text
merge two sorted lists
        +
balanced pairwise merging
        ↓
O(n log k)
```

也就是：

> **这道题本质上不是重新发明一种链表算法，而是思考如何高效地重复使用「合并两个有序链表」。**
