# 合并两个有序链表（Merge Two Sorted Lists）

给定两个已经按升序排列的单链表 `list1` 和 `list2` 的头节点。

请将这两个链表合并成一个新的**升序链表**。合并时应直接复用原链表中的节点，通过修改节点之间的 `next` 指针完成拼接，而不是创建一份新的节点副本。

返回合并后链表的**头节点**。

------

## 一、递归解法

### 核心思路

递归合并两个有序链表时，每一步只需要比较两个链表当前的头节点。

由于两个链表本身已经有序：

- 如果 `list1.val <= list2.val`，那么 `list1` 当前节点一定应该放在合并结果的最前面。
- 否则，`list2` 当前节点应该放在最前面。

选定当前较小节点后，只需要递归处理“剩下的部分”，并将递归结果连接到当前节点的 `next` 即可。

可以把问题理解为：

```text
merge(list1, list2)
=
较小的头节点
+
merge(剩余的 list1, 剩余的 list2)
```

### 算法步骤

1. 如果 `list1` 为空：
   - 直接返回 `list2`。
2. 如果 `list2` 为空：
   - 直接返回 `list1`。
3. 比较两个头节点：
   - 如果 `list1.val <= list2.val`：
     - 当前结果的头节点应该是 `list1`。
     - 递归合并 `list1.next` 和 `list2`。
     - 将递归结果赋给 `list1.next`。
   - 否则：
     - 当前结果的头节点应该是 `list2`。
     - 递归合并 `list1` 和 `list2.next`。
     - 将递归结果赋给 `list2.next`。
4. 最终递归会在某一个链表为空时结束。

### Python 代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def mergeTwoLists(
        self,
        list1: ListNode,
        list2: ListNode
    ) -> ListNode:

        # 递归终止条件：
        # 如果其中一个链表为空，直接返回另一个链表即可。
        if not list1:
            return list2

        if not list2:
            return list1

        # list1 当前节点更小，
        # 因此 list1 应该成为当前合并结果的头节点。
        if list1.val <= list2.val:
            list1.next = self.mergeTwoLists(list1.next, list2)
            return list1

        # 否则 list2 当前节点更小。
        else:
            list2.next = self.mergeTwoLists(list1, list2.next)
            return list2
```

### 递归过程示例

假设：

```text
list1: 1 -> 3 -> 5
list2: 2 -> 4 -> 6
```

第一次比较：

```text
1 < 2
```

所以结果一定从 `1` 开始：

```text
1 -> merge(3 -> 5, 2 -> 4 -> 6)
```

下一次比较：

```text
3 > 2
```

于是：

```text
1 -> 2 -> merge(3 -> 5, 4 -> 6)
```

继续：

```text
1 -> 2 -> 3 -> merge(5, 4 -> 6)
```

最终：

```text
1 -> 2 -> 3 -> 4 -> 5 -> 6
```

### 时间复杂度

设：

- `n` 为 `list1` 的长度
- `m` 为 `list2` 的长度

每个节点最多被处理一次，因此：

```text
时间复杂度：O(n + m)
```

### 空间复杂度

递归本身需要使用调用栈。

最坏情况下，递归深度可以达到：

```text
n + m
```

因此：

```text
空间复杂度：O(n + m)
```

这里额外空间主要来自**递归调用栈**，而不是新创建的链表节点。

------

## 二、迭代解法

### 核心思路

迭代方法通常更适合作为面试中的首选解法，因为：

- 时间复杂度同样是 `O(n + m)`
- 额外空间复杂度只有 `O(1)`
- 不存在递归栈过深的问题

我们维护一个指针 `node`，始终指向当前已经合并链表的最后一个节点。

每次比较 `list1` 和 `list2` 当前的头节点：

- 谁更小，就把谁连接到 `node.next`
- 然后让对应链表向前移动
- `node` 本身也向前移动

直到其中一个链表为空。

------

## Dummy Node：虚拟头节点

这里会使用一个非常常见的链表技巧：

```python
dummy = ListNode()
```

`dummy` 被称为：

```text
dummy node
虚拟头节点
哨兵节点
```

它本身并不是最终结果的一部分。

它的主要作用是避免对“第一个节点”进行特殊处理。

例如，如果没有 `dummy`，我们可能需要先判断：

```python
head = list1
```

还是：

```python
head = list2
```

之后再继续处理。

使用 `dummy` 后，可以统一写成：

```python
node.next = ...
```

最后返回：

```python
dummy.next
```

即可得到真正的头节点。

这是链表题中非常重要、非常常见的技巧。

------

## 算法步骤

1. 创建一个虚拟头节点：

```python
dummy = ListNode()
```

1. 创建 `node` 指针，让它指向当前合并链表的尾部：

```python
node = dummy
```

1. 当 `list1` 和 `list2` 都不为空时：
   - 比较 `list1.val` 与 `list2.val`
   - 将较小节点接到：

```python
node.next
```

- 对应链表向前移动
- `node` 也向前移动

1. 当循环结束时，至少有一个链表已经为空。
2. 将另一个链表剩余的部分直接连接到结果尾部。
3. 返回：

```python
dummy.next
```

------

## Python 代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def mergeTwoLists(
        self,
        list1: ListNode,
        list2: ListNode
    ) -> ListNode:

        # 创建虚拟头节点。
        # dummy 本身不会出现在最终结果中。
        dummy = ListNode()

        # node 始终指向当前合并链表的最后一个节点。
        node = dummy

        # 只有两个链表都不为空时，
        # 才需要继续比较它们的头节点。
        while list1 and list2:

            if list1.val < list2.val:
                # list1 当前节点更小，
                # 将它接到结果链表尾部。
                node.next = list1

                # list1 向前移动。
                list1 = list1.next

            else:
                # list2 当前节点更小或相等，
                # 将它接到结果链表尾部。
                node.next = list2

                # list2 向前移动。
                list2 = list2.next

            # 更新结果链表的尾指针。
            node = node.next

        # 此时至少有一个链表已经为空。
        #
        # 如果 list1 还有剩余节点，就接上 list1；
        # 否则接上 list2。
        #
        # 因为剩余部分本身已经有序，
        # 所以无需继续逐个比较。
        node.next = list1 or list2

        # dummy 是虚拟节点，
        # dummy.next 才是真正的结果链表头节点。
        return dummy.next
```

------

## `list1 or list2` 的 Python 含义

代码中的：

```python
node.next = list1 or list2
```

是一个比较 Python 化的写法。

在 Python 中，`or` 不一定返回 `True` 或 `False`，而是返回其中某个操作数。

这里：

```python
list1 or list2
```

等价于：

```python
if list1:
    node.next = list1
else:
    node.next = list2
```

因为循环结束后至少有一个链表是 `None`。

例如：

```python
list1 = None
list2 = 4 -> 5 -> 6
```

那么：

```python
list1 or list2
```

返回的就是 `list2`。

因此可以很简洁地把剩余链表连接到结果后面。

------

## 时间复杂度

每个节点最多被访问并连接一次，因此：

```text
时间复杂度：O(n + m)
```

其中：

- `n` 是 `list1` 的长度
- `m` 是 `list2` 的长度

------

## 空间复杂度

迭代方法只使用了固定数量的额外指针：

```python
dummy
node
list1
list2
```

并没有随着输入规模增长而创建额外的数据结构。

因此：

```text
空间复杂度：O(1)
```

注意：这里是在**复用原链表节点**，而不是创建新的链表节点。

------

# 递归与迭代对比

| 方法 | 时间复杂度 | 空间复杂度 | 特点                       |
| ---- | ---------- | ---------- | -------------------------- |
| 递归 | `O(n + m)` | `O(n + m)` | 代码简洁，逻辑直观         |
| 迭代 | `O(n + m)` | `O(1)`     | 空间更优，更适合工程和面试 |

通常情况下，**迭代解法更值得优先掌握**。

递归解法则非常适合帮助理解这道题的递归结构。

------

# 为什么可以直接连接剩余链表？

假设主循环结束时：

```text
list1: None
list2: 5 -> 7 -> 9
```

当前已经得到：

```text
1 -> 2 -> 3 -> 4
```

由于：

1. `list2` 原本就是有序链表；
2. 当前已经处理过的所有节点都不大于 `list2` 当前节点；

因此可以直接：

```text
1 -> 2 -> 3 -> 4 -> 5 -> 7 -> 9
```

不需要继续逐节点比较。

这也是：

```python
node.next = list1 or list2
```

能够成立的原因。

------

# 常见错误

## 1. 没有正确处理空链表

输入完全可能是：

```python
list1 = None
list2 = 1 -> 2 -> 3
```

或者：

```python
list1 = None
list2 = None
```

迭代写法中的：

```python
while list1 and list2:
```

天然能够处理这种情况。

如果一开始 `list1` 为空，循环不会执行，最后：

```python
node.next = list1 or list2
```

会直接连接 `list2`。

递归写法则需要明确的终止条件：

```python
if not list1:
    return list2

if not list2:
    return list1
```

------

## 2. 忘记连接剩余节点

这是这道题最常见的错误之一。

例如：

```text
list1: 1 -> 2
list2: 3 -> 4 -> 5
```

比较完 `1` 和 `2` 后：

```text
list1 = None
list2 = 3 -> 4 -> 5
```

如果此时直接返回结果，那么 `3 -> 4 -> 5` 就会丢失。

因此必须：

```python
node.next = list1 or list2
```

------

## 3. 忘记移动 `node`

下面的代码是不完整的：

```python
node.next = list1
list1 = list1.next
```

完成连接后，还必须：

```python
node = node.next
```

否则 `node` 会一直停留在原来的位置，后续连接会不断覆盖：

```python
node.next
```

------

## 4. 错误地移动链表指针

正确顺序通常是：

```python
node.next = list1
list1 = list1.next
node = node.next
```

关键是要确保：

- 已经先保存当前节点到结果链表中；
- 然后再让 `list1` 或 `list2` 前进。

------

# 面试中可以这样理解

这道题本质上使用的是一个非常经典的思想：

**双指针 + 贪心选择。**

两个指针分别指向：

```text
list1 当前最小的未处理节点
list2 当前最小的未处理节点
```

因为两个链表已经有序，所以：

```text
min(list1.val, list2.val)
```

一定就是整个“尚未处理部分”中最小的节点。

因此每一步选择较小节点都是安全的，不需要回头修改之前的选择。

这个思想和数组中的：

```text
Merge Sorted Array
归并排序中的 Merge 阶段
```

本质上是一样的。

区别只是：

- 数组通常通过索引移动；
- 链表通过 `next` 指针移动。

------

# 推荐记忆模板

对于“合并两个有序链表”，迭代代码可以浓缩成下面这个模板：

```python
dummy = node = ListNode()

while list1 and list2:
    if list1.val < list2.val:
        node.next = list1
        list1 = list1.next
    else:
        node.next = list2
        list2 = list2.next

    node = node.next

node.next = list1 or list2

return dummy.next
```

其中最值得记住的三个部分是：

```python
# 1. Dummy Node
dummy = node = ListNode()

# 2. 每次连接较小节点
node.next = ...

# 3. 最后接上剩余部分
node.next = list1 or list2
```

这三个模式在很多链表题中都会反复出现。
