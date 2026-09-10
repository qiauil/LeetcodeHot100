# K 个一组翻转链表

给定一个单链表的头节点 `head` 和一个正整数 `k`。

你需要将链表中的节点按照每 `k` 个一组进行翻转：

- 翻转前 `k` 个节点；
- 再翻转接下来的 `k` 个节点；
- 依此类推；
- 如果最后剩余的节点数量不足 `k` 个，则保持它们原来的顺序不变。

返回处理后的链表。

注意：**只能修改节点的 `next` 指针，不能修改节点中存储的值。**

------

## 前置知识

在解决这道题之前，建议熟悉以下内容：

- **Linked List（链表）**
  - 链表节点结构
  - 链表遍历
  - 指针 / 引用操作
- **Linked List Reversal（链表翻转）**
  - 使用 `prev`、`curr`、`next` 三个指针翻转链表
- **Recursion（递归）**
  - 将问题拆分成更小的子问题
  - 递归终止条件
- **Dummy Node（虚拟头节点 / 哨兵节点）**
  - 在链表头部之前添加一个额外节点
  - 用于减少对头节点的特殊处理

------

# 核心难点

这道题本质上并不是单纯的“翻转链表”，而是：

> **确定每一组的边界 → 翻转这一组 → 重新连接前后链表。**

假设：

```text
1 -> 2 -> 3 -> 4 -> 5 -> 6
```

并且：

```text
k = 3
```

那么需要分成：

```text
[1 -> 2 -> 3] [4 -> 5 -> 6]
```

分别翻转后：

```text
3 -> 2 -> 1 -> 6 -> 5 -> 4
```

对于每一组，我们通常需要关注三个位置：

```text
groupPrev -> [ 当前 k 个节点 ] -> groupNext
```

其中：

- `groupPrev`：当前组之前的节点
- `groupNext`：当前组之后的第一个节点
- 当前组翻转完成之后，需要重新连接这两个边界

这也是这道题最容易出现指针错误的地方。

------

# 解法一：递归

## 思路

对于当前链表头 `head`，首先向后检查是否至少存在 `k` 个节点。

如果不足 `k` 个：

```text
直接返回 head
```

因为题目要求最后不足 `k` 个的节点保持原顺序。

如果当前至少存在 `k` 个节点，则可以：

1. 找到当前 `k` 个节点之后的节点；
2. 递归处理后面的链表；
3. 得到“后半部分已经处理完成的链表”；
4. 再翻转当前这 `k` 个节点；
5. 将当前组连接到递归结果上。

也就是说，递归思路可以理解为：

> **先处理后面的组，再处理当前组。**

------

## 一个简单例子

假设：

```text
1 -> 2 -> 3 -> 4 -> 5 -> 6
k = 3
```

第一次调用：

```text
reverseKGroup(1, 3)
```

确认：

```text
1 -> 2 -> 3
```

刚好有 3 个节点。

此时 `cur` 指向：

```text
4
```

于是递归：

```text
reverseKGroup(4, 3)
```

第二层递归处理：

```text
4 -> 5 -> 6
```

得到：

```text
6 -> 5 -> 4
```

然后回到第一层，将：

```text
1 -> 2 -> 3
```

翻转，并连接到：

```text
6 -> 5 -> 4
```

最终得到：

```text
3 -> 2 -> 1 -> 6 -> 5 -> 4
```

------

## 算法步骤

### 第一步：检查是否有 `k` 个节点

从当前 `head` 开始向后移动：

```python
cur = head
group = 0

while cur and group < k:
    cur = cur.next
    group += 1
```

循环结束后：

- 如果 `group < k`
  - 剩余节点不足 `k`
  - 不翻转
- 如果 `group == k`
  - 当前存在完整的一组
  - `cur` 此时正好指向下一组的第一个节点

------

### 第二步：递归处理后面的链表

```python
cur = self.reverseKGroup(cur, k)
```

递归完成以后：

```python
cur
```

表示：

> 后半部分经过 k-group 翻转之后的新头节点。

------

### 第三步：翻转当前 `k` 个节点

使用：

```python
tmp = head.next
head.next = cur
cur = head
head = tmp
```

反复执行 `k` 次。

这里实际上使用的是一种稍微特殊的链表翻转方式。

假设：

```text
当前组：1 -> 2 -> 3
后半部分：6 -> 5 -> 4
```

一开始：

```text
head = 1
cur = 6
```

第一次：

```text
1 -> 6
```

此时：

```text
cur = 1
head = 2
```

第二次：

```text
2 -> 1 -> 6
```

第三次：

```text
3 -> 2 -> 1 -> 6
```

于是当前组完成翻转。

------

## 代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def reverseKGroup(
        self,
        head: Optional[ListNode],
        k: int
    ) -> Optional[ListNode]:

        # cur 用于检查从 head 开始是否至少存在 k 个节点
        cur = head
        group = 0

        # 向后走最多 k 步
        while cur and group < k:
            cur = cur.next
            group += 1

        # 只有完整的 k 个节点才进行翻转
        if group == k:

            # cur 当前指向下一组的第一个节点
            # 先递归处理后面的链表
            cur = self.reverseKGroup(cur, k)

            # 翻转当前这一组的 k 个节点
            while group > 0:
                # 保存下一个尚未处理的节点
                tmp = head.next

                # 当前节点连接到已经处理好的链表
                head.next = cur

                # cur 更新为新的链表头
                cur = head

                # head 移动到下一个待翻转节点
                head = tmp

                group -= 1

            # cur 是当前 k-group 翻转之后的新头节点
            head = cur

        # 如果不足 k 个节点，则直接返回原 head
        return head
```

------

## 时间复杂度

设链表长度为 `n`。

每个节点只会被：

- 检查一次；
- 翻转一次。

因此总时间复杂度为：

```text
O(n)
```

------

## 空间复杂度

递归每处理一组 `k` 个节点，就会增加一层递归调用。

递归深度大约为：

```text
n / k
```

因此辅助空间复杂度为：

```text
O(n / k)
```

如果忽略 `k`，使用更宽松的上界，也可以写成：

```text
O(n)
```

这里的空间主要来自 **递归调用栈**。

------

# 解法二：迭代

## 思路

迭代解法通常更加推荐，因为：

- 时间复杂度仍然是 `O(n)`；
- 不使用递归；
- 额外空间复杂度只有 `O(1)`。

核心思想是：

> 每次找到完整的一组 `k` 个节点，只翻转这一组，然后重新连接链表。

为了方便处理第一组，我们使用一个 **dummy node（虚拟头节点）**。

------

# Dummy Node 是什么？

原链表：

```text
1 -> 2 -> 3 -> 4
```

添加 dummy 后：

```text
dummy -> 1 -> 2 -> 3 -> 4
```

例如：

```python
dummy = ListNode(0, head)
```

其中：

```python
dummy.val = 0
dummy.next = head
```

`dummy` 的值实际上不重要，它的作用只是：

> 保证原链表的头节点前面始终存在一个节点。

这样，即使第一组翻转之后头节点发生变化，也可以统一写：

```python
groupPrev.next = kth
```

而不需要专门处理：

```text
“如果当前组是第一组怎么办？”
```

------

# 关键指针

假设当前链表为：

```text
groupPrev
    |
    v
dummy -> 1 -> 2 -> 3 -> 4 -> 5
              ^
              kth
```

如果：

```text
k = 3
```

那么：

```python
groupPrev = dummy
kth = 3
groupNext = 4
```

因此当前要翻转的区域为：

```text
1 -> 2 -> 3
```

整个结构可以抽象成：

```text
groupPrev -> [1 -> 2 -> 3] -> groupNext
```

翻转之后：

```text
groupPrev -> [3 -> 2 -> 1] -> groupNext
```

------

# `getKth()` 的作用

辅助函数：

```python
def getKth(self, curr, k):
    while curr and k > 0:
        curr = curr.next
        k -= 1
    return curr
```

作用是：

> 从 `curr` 开始向后移动 `k` 次，找到当前组的第 `k` 个节点。

注意这里 `curr` 通常是：

```python
groupPrev
```

也就是当前组之前的节点。

例如：

```text
dummy -> 1 -> 2 -> 3
```

调用：

```python
getKth(dummy, 3)
```

移动过程：

```text
dummy
 ↓ 1
 ↓ 2
 ↓ 3
```

最后返回：

```text
3
```

如果链表不足 `k` 个节点，那么移动过程中：

```python
curr
```

会变成：

```python
None
```

因此可以通过：

```python
if not kth:
    break
```

判断剩余节点不足 `k` 个。

------

# 迭代算法步骤

## 第一步：建立 dummy

```python
dummy = ListNode(0, head)
groupPrev = dummy
```

`groupPrev` 永远表示：

> 当前 k-group 前面的那个节点。

------

## 第二步：找到当前组的第 k 个节点

```python
kth = self.getKth(groupPrev, k)
```

如果：

```python
kth is None
```

说明剩余节点不足 `k` 个：

```python
if not kth:
    break
```

此时不翻转剩余节点。

------

## 第三步：记录下一组的位置

```python
groupNext = kth.next
```

假设：

```text
dummy -> 1 -> 2 -> 3 -> 4 -> 5
```

当前：

```text
k = 3
```

那么：

```text
kth = 3
groupNext = 4
```

------

# 为什么 `prev = groupNext`？

翻转普通链表时，通常写：

```python
prev = None
curr = head
```

但是这道题需要让当前组翻转之后，最后一个节点直接连接后面的链表。

因此使用：

```python
prev = groupNext
```

假设：

```text
1 -> 2 -> 3 -> 4
```

只翻转：

```text
1 -> 2 -> 3
```

希望最终得到：

```text
3 -> 2 -> 1 -> 4
```

所以翻转开始时：

```python
prev = 4
curr = 1
```

第一次：

```text
1 -> 4
```

第二次：

```text
2 -> 1 -> 4
```

第三次：

```text
3 -> 2 -> 1 -> 4
```

这样，当前组和后面的链表自动完成连接。

这是这个解法中非常巧妙的一点。

------

# 第四步：翻转当前组

初始化：

```python
prev = groupNext
curr = groupPrev.next
```

然后：

```python
while curr != groupNext:
    tmp = curr.next
    curr.next = prev
    prev = curr
    curr = tmp
```

这是经典的链表翻转模板：

```text
保存 next
↓
修改 curr.next
↓
prev 前进
↓
curr 前进
```

------

# 第五步：重新连接当前组

翻转之前：

```text
groupPrev -> 1 -> 2 -> 3 -> groupNext
```

翻转之后，内部已经变成：

```text
3 -> 2 -> 1 -> groupNext
```

但：

```python
groupPrev.next
```

仍然指向原来的：

```text
1
```

因此需要修改：

```python
groupPrev.next = kth
```

变成：

```text
groupPrev -> 3 -> 2 -> 1 -> groupNext
```

------

# 为什么要保存原来的第一个节点？

原来的：

```text
1
```

翻转后会变成当前组的最后一个节点：

```text
3 -> 2 -> 1
          ^
```

而它正好应该成为下一轮的：

```python
groupPrev
```

因此在修改：

```python
groupPrev.next
```

之前，需要先保存：

```python
tmp = groupPrev.next
```

然后：

```python
groupPrev.next = kth
groupPrev = tmp
```

所以：

```text
原来的组头
```

会变成：

```text
下一组之前的节点
```

------

# 完整代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def reverseKGroup(
        self,
        head: Optional[ListNode],
        k: int
    ) -> Optional[ListNode]:

        # dummy 节点用于统一处理链表头部
        dummy = ListNode(0, head)

        # groupPrev 表示当前 k-group 前面的节点
        groupPrev = dummy

        while True:

            # 找到当前组的第 k 个节点
            kth = self.getKth(groupPrev, k)

            # 如果剩余节点不足 k 个，则不再翻转
            if not kth:
                break

            # 当前组之后的第一个节点
            groupNext = kth.next

            # curr：当前准备翻转的节点
            # prev：当前节点翻转后应该指向的节点
            #
            # prev 从 groupNext 开始，
            # 可以让当前组翻转后自动连接后半部分
            prev = groupNext
            curr = groupPrev.next

            # 翻转当前 k 个节点
            while curr != groupNext:

                # 保存原来的下一个节点，
                # 防止修改 curr.next 后丢失后续链表
                tmp = curr.next

                # 反转指针
                curr.next = prev

                # prev 和 curr 同时向前移动
                prev = curr
                curr = tmp

            # 原来的第一个节点，
            # 翻转之后会成为当前组最后一个节点
            oldGroupStart = groupPrev.next

            # kth 原本是第 k 个节点，
            # 翻转之后变成当前组的新头节点
            groupPrev.next = kth

            # 原来的第一个节点现在位于组尾，
            # 下一轮它就是新的 groupPrev
            groupPrev = oldGroupStart

        # dummy.next 永远指向真正的链表头
        return dummy.next

    def getKth(self, curr, k):
        """
        从 curr 开始向后走 k 步。

        如果存在第 k 个节点，则返回该节点；
        如果剩余节点不足 k 个，则返回 None。
        """

        while curr and k > 0:
            curr = curr.next
            k -= 1

        return curr
```

------

# 复杂度分析

## 时间复杂度

```text
O(n)
```

虽然每一轮都会先通过 `getKth()` 寻找第 `k` 个节点，然后再翻转这一组，但每个节点只会参与常数次操作。

粗略来看，每一组会：

```text
检查 k 个节点
+
翻转 k 个节点
```

因此总操作量约为：

```text
2n
```

忽略常数后：

```text
O(n)
```

------

## 空间复杂度

```text
O(1)
```

除了几个指针变量：

```python
dummy
groupPrev
kth
groupNext
prev
curr
tmp
```

之外，没有额外使用与链表长度相关的数据结构。

------

# 两种方法对比

| 方法 | 时间复杂度 | 空间复杂度                    | 特点                                     |
| ---- | ---------- | ----------------------------- | ---------------------------------------- |
| 递归 | `O(n)`     | `O(n / k)`，最坏可视为 `O(n)` | 思路简洁，但需要理解递归返回后的连接过程 |
| 迭代 | `O(n)`     | `O(1)`                        | 面试中通常更加推荐，空间效率更高         |

如果是在代码面试中，我会优先掌握 **迭代版本**。

因为它集中考察了链表题最重要的能力：

```text
定位区间边界
+
局部翻转
+
重新连接链表
```

------

# 常见错误

## 1. 翻转最后不足 `k` 个的节点

例如：

```text
1 -> 2 -> 3 -> 4 -> 5
k = 3
```

正确结果：

```text
3 -> 2 -> 1 -> 4 -> 5
```

而不是：

```text
3 -> 2 -> 1 -> 5 -> 4
```

因此每次翻转之前，必须先确认：

```text
剩余节点数量 >= k
```

------

## 2. 翻转完成后没有重新连接 `groupPrev`

翻转之后：

```text
1 -> 2 -> 3
```

内部已经变成：

```text
3 -> 2 -> 1
```

但前半部分仍然可能指向：

```text
1
```

因此必须执行：

```python
groupPrev.next = kth
```

因为 `kth` 是翻转之后当前组的新头节点。

------

## 3. 丢失当前组原来的第一个节点

假设：

```text
1 -> 2 -> 3
```

翻转后：

```text
3 -> 2 -> 1
```

原来的：

```text
1
```

现在成为了组尾。

它应该作为下一轮的：

```python
groupPrev
```

因此必须在修改连接之前保存：

```python
oldGroupStart = groupPrev.next
```

然后：

```python
groupPrev = oldGroupStart
```

------

## 4. 修改 `curr.next` 之前没有保存原来的 `next`

链表翻转中最经典的错误就是直接：

```python
curr.next = prev
curr = curr.next
```

这会出问题，因为：

```python
curr.next
```

已经被修改了。

原来的后续链表可能因此丢失。

正确方式一定是：

```python
tmp = curr.next
curr.next = prev
prev = curr
curr = tmp
```

可以记成：

```text
先保存
再断开
再移动
```

------

## 5. `getKth()` 的 off-by-one 错误

因为：

```python
groupPrev
```

位于当前组之前，所以应该从它开始向后移动 **k 次**。

例如：

```text
dummy -> 1 -> 2 -> 3
k = 3
```

从 `dummy`：

```text
第 1 步 -> 1
第 2 步 -> 2
第 3 步 -> 3
```

所以返回：

```text
3
```

这正是当前组的第 `k` 个节点。

------

## 6. 不使用 dummy node 导致头节点逻辑复杂

第一组翻转之前：

```text
head -> 1 -> 2 -> 3
```

翻转之后：

```text
head -> 3 -> 2 -> 1
```

意味着：

```python
head
```

本身需要更新。

如果使用：

```text
dummy -> 1 -> 2 -> 3
```

则只需要统一修改：

```python
groupPrev.next
```

第一组和后面的所有组可以使用完全相同的代码。

因此，在涉及：

```text
删除头节点
插入头节点
翻转头部
```

这类链表题中，dummy node 往往非常有用。

------

# Python 相关说明

## `Optional[ListNode]`

函数定义中：

```python
head: Optional[ListNode]
```

来自 Python 的类型标注系统。

通常需要：

```python
from typing import Optional
```

`Optional[ListNode]` 表示：

```text
head 可以是一个 ListNode
或者
head 可以是 None
```

等价于较新的 Python 写法：

```python
ListNode | None
```

它主要用于增强代码可读性和类型检查，并不会改变算法本身。

------

## `ListNode`

LeetCode 通常会预先定义：

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

一个节点包含两个核心字段：

```python
self.val
```

表示节点存储的数据。

```python
self.next
```

表示下一个节点。

例如：

```python
node1 = ListNode(1)
node2 = ListNode(2)

node1.next = node2
```

表示：

```text
1 -> 2
```

这道题明确要求不能修改：

```python
node.val
```

只能通过修改：

```python
node.next
```

来改变链表结构。

------

# 面试中建议记住的核心模板

对于这道题，最值得记住的并不是整段代码，而是下面这个结构：

```python
while True:
    # 1. 找到当前组尾部
    kth = getKth(groupPrev, k)

    if not kth:
        break

    # 2. 记录下一组
    groupNext = kth.next

    # 3. 翻转当前组
    prev = groupNext
    curr = groupPrev.next

    while curr != groupNext:
        tmp = curr.next
        curr.next = prev
        prev = curr
        curr = tmp

    # 4. 重新连接
    oldGroupStart = groupPrev.next
    groupPrev.next = kth
    groupPrev = oldGroupStart
```

可以把整个过程浓缩成四个词：

```text
Find
↓
Reverse
↓
Reconnect
↓
Advance
```

也就是：

```text
找到边界
↓
翻转区间
↓
重新连接
↓
移动到下一组
```

------

# 一个重要的指针不变量

迭代解法中，理解下面这个不变量会非常有帮助：

在每一轮开始时：

```python
groupPrev.next
```

一定是：

> 当前准备翻转的这一组的第一个节点。

而一轮结束以后：

```python
groupPrev
```

会更新成：

> 刚刚翻转完成的这一组的最后一个节点。

例如：

```text
dummy -> 1 -> 2 -> 3 -> 4 -> 5 -> 6
```

翻转第一组后：

```text
dummy -> 3 -> 2 -> 1 -> 4 -> 5 -> 6
                  ^
                  groupPrev
```

下一轮：

```python
groupPrev.next
```

自然就是：

```text
4
```

于是可以继续处理：

```text
4 -> 5 -> 6
```

正是因为维护了这个不变量，整个算法才能不断向后推进。

------

# 总结

这道题最重要的三个知识点是：

1. **先确认剩余节点至少有 `k` 个，再翻转。**
2. **翻转一个局部链表时，必须保存当前组前后的边界。**
3. **翻转完成后，原来的组头会变成组尾。**

尤其是迭代版本中的这几个变量：

```python
groupPrev
kth
groupNext
curr
prev
```

可以对应理解为：

```text
groupPrev   当前组之前
kth         当前组最后一个节点
groupNext   下一组开始
curr        当前正在处理的节点
prev        当前节点翻转后应该指向的位置
```

如果能清楚地画出：

```text
groupPrev -> [ k 个节点 ] -> groupNext
```

并理解翻转之后要变成：

```text
groupPrev -> [ 翻转后的 k 个节点 ] -> groupNext
```

这道题的主要难点就基本解决了。
