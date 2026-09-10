# 排序链表（Sort List）

## 题目描述

给你链表的头节点 `head`，请将其按**升序**排列，并返回排序后的链表。

例如：

```
输入：

4 -> 2 -> 1 -> 3

输出：

1 -> 2 -> 3 -> 4
```

------

# 一、核心思路

如果这是一个数组：

```
[4, 2, 1, 3]
```

我们可以直接使用很多经典排序算法。

但链表和数组有一个非常重要的区别：

- 数组支持通过下标 `O(1)` 随机访问；
- 链表只能从前往后遍历；
- 链表交换节点本身比较麻烦；
- 但链表**拆分和合并非常方便**，只需要修改指针。

因此，对于链表排序，**归并排序（Merge Sort）**通常是最自然的选择。

归并排序的核心思想是：

```
不断把链表分成两半

        4 -> 2 -> 1 -> 3
              ↓
       4 -> 2     1 -> 3
         ↓           ↓
       4   2       1   3

然后从下往上合并：

       2 -> 4     1 -> 3
              ↓
       1 -> 2 -> 3 -> 4
```

整个过程分成三个核心步骤：

1. **找到链表中点**
2. **递归排序左右两半**
3. **合并两个有序链表**

------

# 二、方法一：自顶向下归并排序

这是这道题最经典、也最推荐首先掌握的方法。

## 思路

对于：

```
4 -> 2 -> 1 -> 3
```

先拆成：

```
4 -> 2

1 -> 3
```

继续拆：

```
4    2

1    3
```

单个节点本身就是有序链表。

然后开始合并：

```
4 + 2
↓
2 -> 4
```

以及：

```
1 + 3
↓
1 -> 3
```

最后：

```
2 -> 4

1 -> 3
```

合并得到：

```
1 -> 2 -> 3 -> 4
```

因此整个算法可以写成：

```
sortList(head):

    1. 如果只有 0/1 个节点
       直接返回

    2. 找到链表中点

    3. 从中间断开
       得到 left 和 right

    4. left = sortList(left)
       right = sortList(right)

    5. merge(left, right)
```

------

# 三、如何找到链表中点：快慢指针

这里需要使用链表中非常经典的技巧：

> **Fast & Slow Pointers（快慢指针）**

设置：

```
slow = head
fast = head
```

然后：

```
slow = slow.next
fast = fast.next.next
```

也就是说：

- `slow` 每次走一步；
- `fast` 每次走两步。

当 `fast` 到达链表末尾时，`slow` 大约正好走到链表中间。

例如：

```
1 -> 2 -> 3 -> 4 -> 5
S
F
```

移动：

```
1 -> 2 -> 3 -> 4 -> 5
     S
          F
```

再次移动：

```
1 -> 2 -> 3 -> 4 -> 5
          S
                    F
```

此时 `slow` 就位于中间节点附近。

------

## 一个容易出错的地方

在这道题中，我们希望把：

```
1 -> 2 -> 3 -> 4
```

拆成：

```
1 -> 2

3 -> 4
```

所以更方便的写法是：

```
slow = head
fast = head.next
```

然后：

```
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

结束后：

```
slow
 ↓
1 -> 2 -> 3 -> 4
     ↑
```

`slow.next` 就是右半部分的头：

```
right = slow.next
```

然后：

```
slow.next = None
```

把链表真正断开：

```
1 -> 2 -> None

3 -> 4 -> None
```

------

# 四、为什么必须执行 `slow.next = None`？

这是递归归并排序里一个非常重要的细节。

假设：

```
1 -> 2 -> 3 -> 4
```

我们找到：

```
right = slow.next
```

但没有执行：

```
slow.next = None
```

那么左半部分仍然是：

```
1 -> 2 -> 3 -> 4
```

而不是：

```
1 -> 2
```

下一次递归：

```
sortList(head)
```

处理的仍然可能是原来的链表，导致递归无法正确缩小问题规模。

所以：

```
slow.next = None
```

本质上是在真正执行：

> **Divide（分治中的“分”）**

------

# 五、如何合并两个有序链表

现在假设已经有：

```
list1:

1 -> 4 -> 7
```

和：

```
list2:

2 -> 3 -> 8
```

我们需要合并成：

```
1 -> 2 -> 3 -> 4 -> 7 -> 8
```

这其实就是经典的 **Merge Two Sorted Lists**。

------

## Dummy Node

这里通常使用一个虚拟头节点：

```
dummy = ListNode()
tail = dummy
```

`dummy` 本身不属于最终答案，它只是帮助我们统一处理链表头。

例如：

```
dummy
  ↓
 -1
```

然后不断：

```
tail.next = ...
tail = tail.next
```

最终真正的链表头就是：

```
dummy.next
```

------

## 合并过程

假设：

```
left:
1 -> 4 -> 7

right:
2 -> 3 -> 8
```

比较：

```
1 < 2
```

所以选择 `1`：

```
dummy -> 1
```

然后比较：

```
4 vs 2
```

选择 `2`：

```
dummy -> 1 -> 2
```

继续：

```
4 vs 3
```

选择 `3`：

```
dummy -> 1 -> 2 -> 3
```

不断重复即可。

当其中一个链表为空后，另外一个链表本身已经有序，因此可以直接整体接到后面：

```
tail.next = left if left else right
```

------

# 六、完整代码：递归归并排序

```python
"""
# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
"""


class Solution:
    def sortList(self, head: Optional[ListNode]) -> Optional[ListNode]:

        # Base Case:
        # 空链表或者只有一个节点，本身已经有序
        if head is None or head.next is None:
            return head

        # ==================================================
        # Step 1：使用快慢指针找到链表中点
        # ==================================================

        slow = head
        fast = head.next

        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        # slow.next 是右半部分的头节点
        right = slow.next

        # 从中间断开链表
        slow.next = None

        # 左半部分仍然从 head 开始
        left = head

        # ==================================================
        # Step 2：递归排序左右两半
        # ==================================================

        left = self.sortList(left)
        right = self.sortList(right)

        # ==================================================
        # Step 3：合并两个已经排好序的链表
        # ==================================================

        return self.merge(left, right)

    def merge(
        self,
        left: Optional[ListNode],
        right: Optional[ListNode]
    ) -> Optional[ListNode]:

        # Dummy Node：
        # 避免单独处理“第一个节点”
        dummy = ListNode()
        tail = dummy

        # 两个链表都还有节点时，不断选择较小的节点
        while left and right:

            if left.val <= right.val:
                tail.next = left
                left = left.next
            else:
                tail.next = right
                right = right.next

            # tail 永远指向当前合并链表的最后一个节点
            tail = tail.next

        # 此时最多只有一个链表还有剩余节点。
        # 剩余部分本身已经有序，可以直接接上。
        tail.next = left if left else right

        return dummy.next
```

------

# 七、完整过程示例

假设：

```
4 -> 2 -> 1 -> 3
```

## 第一层递归

拆成：

```
4 -> 2

1 -> 3
```

------

## 第二层递归

左边：

```
4 -> 2
```

拆成：

```
4

2
```

右边：

```
1 -> 3
```

拆成：

```
1

3
```

此时所有链表长度都为 `1`，触发：

```
if head is None or head.next is None:
    return head
```

------

## 开始向上合并

首先：

```
merge(4, 2)

=> 2 -> 4
```

然后：

```
merge(1, 3)

=> 1 -> 3
```

现在得到：

```
2 -> 4

1 -> 3
```

最后：

```
merge(
    2 -> 4,
    1 -> 3
)
```

比较过程：

```
2 vs 1  -> 选择 1
2 vs 3  -> 选择 2
4 vs 3  -> 选择 3
```

最后剩下：

```
4
```

直接接到尾部。

最终：

```
1 -> 2 -> 3 -> 4
```

------

# 八、时间复杂度分析

归并排序每次都会把链表大致分成两半：

```
                    n
               /         \
             n/2         n/2
            /   \       /   \
          n/4   n/4   n/4   n/4
          ...
```

因此递归深度大约是：

```
log n
```

每一层递归中，所有 `merge` 操作加起来需要处理 `n` 个节点：

```
第 1 层：O(n)
第 2 层：O(n)
第 3 层：O(n)
...
共 log n 层
```

因此：

**时间复杂度：O(n log n)**

------

# 九、空间复杂度分析

合并链表时，我们并没有创建长度为 `n` 的额外数组。

`merge()` 只是重新连接原有节点的 `next` 指针。

所以 `merge` 本身：

```
O(1) extra space
```

但是递归版本存在递归调用栈。

链表不断减半，因此递归深度：

```
O(log n)
```

所以整体：

**空间复杂度：O(log n)**

主要来自递归调用栈。

------

# 十、为什么归并排序特别适合链表？

这是这道题非常值得理解的地方。

## 数组中的归并排序

对于数组：

```
[1, 4, 7]

[2, 3, 8]
```

合并时通常需要额外数组：

```
[1, 2, 3, 4, 7, 8]
```

所以往往需要：

```
O(n)
```

额外空间。

------

## 链表中的归并排序

链表不需要复制元素。

例如：

```
1 -> 4 -> 7

2 -> 3 -> 8
```

我们只需要修改：

```
node.next
```

就可以重新组织节点。

因此：

> **链表非常适合做 Merge。**

------

# 十一、为什么快速排序通常不是这道题的首选？

快速排序非常适合数组，因为数组可以：

```
nums[i]
nums[j]
```

进行 `O(1)` 随机访问。

链表做不到这一点。

同时，快速排序需要围绕 pivot 做 partition，在链表上实现通常没有归并排序自然。

另外快速排序：

- 平均：`O(n log n)`
- 最坏：`O(n²)`

而归并排序可以稳定保证：

```
O(n log n)
```

因此：

> 对于链表排序，看到题目后首先应该考虑 **Merge Sort**。

------

# 十二、方法二：自底向上归并排序

递归版本虽然非常直观，但空间复杂度为：

```
O(log n)
```

因为存在递归调用栈。

如果题目进一步要求：

> 能否做到 `O(1)` 额外空间？

可以使用：

> **Bottom-Up Merge Sort（自底向上归并排序）**

最终达到：

- 时间复杂度：**O(n log n)**
- 额外空间复杂度：**O(1)**

------

# 十三、自底向上的核心思想

递归版本是：

```
不断拆小：

8
↓
4 + 4
↓
2 + 2 + 2 + 2
↓
1 + 1 + 1 + 1 + 1 + 1 + 1 + 1

然后向上 merge
```

自底向上则反过来：

```
一开始每个节点看成长度为 1 的有序链表
```

例如：

```
4 -> 2 -> 1 -> 3
```

### 第一轮：合并长度为 1 的链表

```
[4] [2] [1] [3]

↓

[2 -> 4] [1 -> 3]
```

此时：

```
size = 1
```

------

### 第二轮：合并长度为 2 的链表

```
[2 -> 4]

[1 -> 3]

↓

[1 -> 2 -> 3 -> 4]
```

此时：

```
size = 2
```

下一轮：

```
size = 4
```

已经达到链表长度，排序完成。

因此：

```
size = 1
size = 2
size = 4
size = 8
...
```

每轮：

```
size *= 2
```

------

# 十四、Bottom-Up 中的关键操作：split

我们需要能够从：

```
1 -> 2 -> 3 -> 4 -> 5 -> 6
```

按照长度 `2` 切出：

```
1 -> 2

3 -> 4

5 -> 6
```

因此可以定义：

```
def split(head, size):
```

它负责：

1. 从 `head` 开始走 `size` 个节点；
2. 把这一段链表断开；
3. 返回下一段链表的头节点。

------

# 十五、完整代码：自底向上归并排序

```
class Solution:
    def sortList(self, head: Optional[ListNode]) -> Optional[ListNode]:

        if head is None or head.next is None:
            return head

        # ==================================================
        # Step 1：计算链表长度
        # ==================================================

        n = 0
        cur = head

        while cur:
            n += 1
            cur = cur.next

        # dummy.next 永远指向当前链表头
        dummy = ListNode(0, head)

        # ==================================================
        # Step 2：
        # size = 1, 2, 4, 8, ...
        #
        # 每一轮把两个长度为 size 的有序链表合并
        # ==================================================

        size = 1

        while size < n:

            prev = dummy
            cur = dummy.next

            while cur:

                # 第一段链表
                left = cur

                # 切出长度为 size 的 left
                right = self.split(left, size)

                # 切出长度为 size 的 right
                cur = self.split(right, size)

                # 合并 left 和 right
                merged_head, merged_tail = self.merge(left, right)

                # 接回整个链表
                prev.next = merged_head

                # prev 移动到本次合并后的尾部
                prev = merged_tail

            # 下一轮合并更长的链表
            size *= 2

        return dummy.next

    def split(
        self,
        head: Optional[ListNode],
        size: int
    ) -> Optional[ListNode]:

        if head is None:
            return None

        # 从 head 开始，一共保留 size 个节点
        for _ in range(size - 1):
            if head.next is None:
                break
            head = head.next

        # 下一段链表的头
        next_head = head.next

        # 断开当前链表
        head.next = None

        return next_head

    def merge(
        self,
        left: Optional[ListNode],
        right: Optional[ListNode]
    ):

        dummy = ListNode()
        tail = dummy

        while left and right:

            if left.val <= right.val:
                tail.next = left
                left = left.next
            else:
                tail.next = right
                right = right.next

            tail = tail.next

        # 接上剩余部分
        tail.next = left if left else right

        # 找到合并后的尾节点
        while tail.next:
            tail = tail.next

        return dummy.next, tail
```

------

# 十六、为什么 Bottom-Up 可以做到 O(1) 额外空间？

递归版本：

```
sort(n)
  sort(n/2)
    sort(n/4)
      ...
```

需要保存递归调用栈：

```
O(log n)
```

而 Bottom-Up 完全使用：

```
while
```

循环控制：

```
size = 1
size = 2
size = 4
size = 8
...
```

没有递归调用栈。

整个过程中只使用：

```
dummy
prev
cur
left
right
tail
```

这些固定数量的指针。

因此：

**额外空间复杂度：O(1)**

------

# 十七、两种方法对比

| 方法             | 时间复杂度   | 额外空间   | 特点                       |
| ---------------- | ------------ | ---------- | -------------------------- |
| 自顶向下归并排序 | `O(n log n)` | `O(log n)` | **最直观，最推荐首先掌握** |
| 自底向上归并排序 | `O(n log n)` | `O(1)`     | 满足严格空间优化要求       |

面试中建议掌握顺序：

```
Top-Down Merge Sort
        ↓
先写出正确的 O(n log n)

        ↓ 面试官追问空间优化

Bottom-Up Merge Sort
        ↓
O(n log n) Time
O(1) Extra Space
```

------

# 十八、常见错误

## 1. 忘记递归 Base Case

错误：

```
def sortList(self, head):
    # 直接继续拆
```

必须有：

```
if head is None or head.next is None:
    return head
```

否则无法结束递归。

------

## 2. 找到中点之后没有断开链表

错误：

```
right = slow.next

left = self.sortList(head)
right = self.sortList(right)
```

因为：

```
head
 ↓
1 -> 2 -> 3 -> 4
```

仍然没有断开。

正确：

```
right = slow.next
slow.next = None
```

得到：

```
1 -> 2

3 -> 4
```

然后才能分别递归。

------

## 3. 快慢指针初始化不合适

在这道题中推荐：

```
slow = head
fast = head.next
```

这样对于：

```
1 -> 2
```

最终：

```
slow = 1
```

可以顺利拆成：

```
1

2
```

如果处理不当，很容易导致长度为 `2` 的链表无法正确缩小。

------

## 4. Merge 后忘记连接剩余节点

例如：

```
left:
1 -> 2

right:
3 -> 4 -> 5
```

当 `left` 用完后：

```
1 -> 2
```

还必须把：

```
3 -> 4 -> 5
```

整体接上。

因此需要：

```
tail.next = left if left else right
```

------

## 5. 不必要地创建大量新节点

排序链表通常没有必要把：

```
4 -> 2 -> 1 -> 3
```

复制成另一套节点。

我们可以直接重新连接原节点：

```
tail.next = left
```

或者：

```
tail.next = right
```

归并排序真正改变的是：

```
节点之间的 next 关系
```

而不是重新创建所有节点。

------

# 十九、面试中最重要的推导过程

这道题最值得掌握的是从题目条件自然推导出归并排序。

### 第一步：这是链表排序

目标显然是：

```
O(n log n)
```

常见 `O(n log n)` 排序：

```
Merge Sort
Quick Sort
Heap Sort
```

------

### 第二步：考虑数据结构特性

这是链表。

链表：

```
随机访问       不方便
交换节点       不方便
拆分           方便
合并           非常方便
```

因此：

```
Merge Sort
```

非常适合。

------

### 第三步：把问题拆成三个子问题

归并排序只需要解决：

```
① 如何找到中点？
② 如何排序两半？
③ 如何合并两个有序链表？
```

答案分别是：

```
① Fast & Slow Pointers

② Recursion

③ Merge Two Sorted Lists
```

组合起来：

```
Fast/Slow Pointer
        +
    Recursion
        +
Merge Two Sorted Lists
        ↓
    Sort List
```

------

### 第四步：分析复杂度

每次减半：

```
递归深度 = O(log n)
```

每层合并所有节点：

```
每层工作 = O(n)
```

所以：

```
O(n) × O(log n)
=
O(n log n)
```

递归栈：

```
O(log n)
```

因此得到：

```
Time:  O(n log n)
Space: O(log n)
```

------

### 第五步：进一步消除递归空间

如果要求：

```
O(1) extra space
```

那么问题实际上变成：

> 能不能不递归执行 Merge Sort？

可以。

把：

```
Top-Down
```

改成：

```
Bottom-Up
```

按照：

```
1 -> 2 -> 4 -> 8 -> ...
```

逐渐扩大待合并链表长度，就不再需要递归栈。

最终：

```
Time:  O(n log n)
Space: O(1)
```

------

# 二十、最终推荐模板

如果面试没有明确要求 `O(1)` 额外空间，推荐优先写下面这个版本：

```
class Solution:
    def sortList(self, head: Optional[ListNode]) -> Optional[ListNode]:

        # 0 / 1 个节点已经有序
        if head is None or head.next is None:
            return head

        # ---------- 1. 找中点 ----------
        slow = head
        fast = head.next

        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        # ---------- 2. 从中间断开 ----------
        right = slow.next
        slow.next = None

        left = head

        # ---------- 3. 分别排序 ----------
        left = self.sortList(left)
        right = self.sortList(right)

        # ---------- 4. 合并 ----------
        dummy = ListNode()
        tail = dummy

        while left and right:
            if left.val <= right.val:
                tail.next = left
                left = left.next
            else:
                tail.next = right
                right = right.next

            tail = tail.next

        # 接上剩余节点
        tail.next = left if left else right

        return dummy.next
```

可以把这道题压缩成一个非常适合面试记忆的框架：

```
Sort List
│
├── Divide
│     └── Fast & Slow Pointer 找中点
│
├── Conquer
│     ├── sortList(left)
│     └── sortList(right)
│
└── Merge
      └── Merge Two Sorted Lists
```

也就是：

> **链表排序 → 想到 Merge Sort → 快慢指针拆分 → 递归排序 → 双指针合并。**
