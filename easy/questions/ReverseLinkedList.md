# Reverse Linked List｜反转单链表

## 题目描述

给定一个单链表的头节点 `head`，请将整个链表反转，并返回反转后的新头节点。

### 示例 1

```text
输入：head = [0,1,2,3]

输出：[3,2,1,0]
```

原链表：

```text
0 -> 1 -> 2 -> 3 -> None
```

反转后：

```text
3 -> 2 -> 1 -> 0 -> None
```

### 示例 2

```text
输入：head = []

输出：[]
```

### 约束条件

- 链表长度满足：`0 <= length <= 1000`
- 节点值满足：`-1000 <= Node.val <= 1000`

------

## 前置知识

在解决这道题之前，建议熟悉以下内容：

### 1. 链表基础

单链表中的每个节点通常包含两个部分：

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val      # 当前节点保存的值
        self.next = next    # 指向下一个节点
```

例如：

```text
0 -> 1 -> 2 -> None
```

实际上表示：

```text
Node(0).next = Node(1)
Node(1).next = Node(2)
Node(2).next = None
```

### 2. 指针操作

这道题的核心并不是交换节点中的值，而是**修改每个节点的 `next` 指针方向**。

原本：

```text
A -> B
```

反转之后：

```text
A <- B
```

也就是：

```python
B.next = A
```

### 3. 递归基础

递归解法需要理解：

- 函数调用栈
- 递归的终止条件
- 递归返回时的“回溯”过程

------

# 解法一：迭代

在实际代码面试中，**迭代通常是这道题最推荐的解法**。

原因是：

- 时间复杂度 `O(n)`
- 额外空间复杂度 `O(1)`
- 不需要递归栈
- 思路非常经典，很多链表题都会复用这个技巧

------

## 核心思路

迭代反转链表的本质是：

> 从左向右遍历链表，并逐个把当前节点的 `next` 指针反过来。

假设当前链表为：

```text
0 -> 1 -> 2 -> 3 -> None
```

我们希望逐渐把它变成：

```text
None <- 0 <- 1 <- 2 <- 3
```

但是存在一个非常重要的问题。

假设当前：

```text
curr = 1
```

此时：

```text
1 -> 2
```

如果直接执行：

```python
curr.next = prev
```

那么 `1` 原本指向 `2` 的关系就会丢失。

因此，在改变 `curr.next` 之前，必须先保存：

```python
temp = curr.next
```

------

## 三个关键指针

迭代过程中通常维护三个变量：

```text
prev
curr
temp
```

它们分别表示：

### `prev`

已经完成反转的链表部分的头节点。

### `curr`

当前正在处理的节点。

### `temp`

临时保存 `curr` 原本的下一个节点。

这个变量非常重要，因为修改：

```python
curr.next
```

之后，就无法再通过 `curr` 找到原来的下一个节点。

------

## 过程演示

初始：

```text
prev = None
curr = 0

None    0 -> 1 -> 2 -> 3
 ↑      ↑
prev   curr
```

### 第一次循环

首先保存：

```python
temp = curr.next
```

得到：

```text
temp = 1
```

然后反转：

```python
curr.next = prev
```

变成：

```text
None <- 0    1 -> 2 -> 3
```

移动两个指针：

```python
prev = curr
curr = temp
```

此时：

```text
None <- 0    1 -> 2 -> 3
          ↑  ↑
        prev curr
```

------

### 第二次循环

保存：

```python
temp = curr.next
```

也就是：

```text
temp = 2
```

反转：

```python
curr.next = prev
```

得到：

```text
None <- 0 <- 1    2 -> 3
```

继续移动：

```text
prev = 1
curr = 2
```

------

不断重复后：

```text
None <- 0 <- 1 <- 2 <- 3
```

最后：

```text
curr = None
prev = 3
```

因此 `prev` 就是新链表的头节点。

------

## 算法步骤

1. 初始化：

```python
prev = None
curr = head
```

1. 当 `curr` 不为空时，不断执行：

```python
temp = curr.next
curr.next = prev
prev = curr
curr = temp
```

1. 循环结束后：

```python
curr == None
```

此时：

```python
prev
```

正好指向反转后的新头节点。

1. 返回：

```python
prev
```

------

## Python 代码

```python
# Definition for singly-linked list.
# 单链表节点定义
#
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        # prev 指向已经完成反转的链表部分
        # 一开始还没有任何节点被反转，因此是 None
        prev = None

        # curr 表示当前正在处理的节点
        curr = head

        while curr:
            # 1. 保存 curr 原本的下一个节点
            # 因为马上要修改 curr.next
            temp = curr.next

            # 2. 将当前节点的 next 指针反转
            curr.next = prev

            # 3. prev 向前移动到当前节点
            prev = curr

            # 4. curr 移动到原链表中的下一个节点
            curr = temp

        # 当 curr == None 时，
        # prev 就是反转后的新头节点
        return prev
```

------

## 一个非常重要的代码模板

建议记住下面这四行：

```python
temp = curr.next
curr.next = prev
prev = curr
curr = temp
```

它们可以理解成：

```text
保存下一步
→ 反转当前指针
→ 扩大已反转区域
→ 继续处理下一节点
```

这是链表题中非常常见的“指针翻转模板”。

------

## 时间复杂度

```text
O(n)
```

其中 `n` 是链表节点数量。

每个节点只访问一次。

------

## 空间复杂度

```text
O(1)
```

只使用了：

```text
prev
curr
temp
```

几个额外变量。

它们占用的空间不会随着链表长度增加。

------

# 解法二：递归

## 核心思路

递归解法可以概括为一句话：

> 先反转后面的链表，再把当前节点接到反转后的链表尾部。

假设链表为：

```text
1 -> 2 -> 3 -> None
```

对于节点 `1`：

我们暂时不处理 `1`，而是先递归地反转：

```text
2 -> 3
```

递归完成后：

```text
3 -> 2
```

此时局部结构实际上还是：

```text
1 -> 2
     ↑
     |
     3
```

更准确地说：

```text
1 -> 2 <- 3
```

现在需要让：

```text
2 -> 1
```

也就是：

```python
head.next.next = head
```

然后切断：

```text
1 -> 2
```

原来的连接：

```python
head.next = None
```

最终：

```text
3 -> 2 -> 1 -> None
```

------

## 为什么 `head.next.next = head` 可以反转指针？

这是递归解法最容易令人困惑的一行：

```python
head.next.next = head
```

假设：

```text
head = 2
head.next = 3
```

原本：

```text
2 -> 3
```

那么：

```python
head.next.next
```

相当于：

```python
3.next
```

因此：

```python
head.next.next = head
```

实际上就是：

```python
3.next = 2
```

于是：

```text
2 <- 3
```

指针方向就被反转了。

------

## 为什么必须执行 `head.next = None`？

考虑：

```text
1 -> 2
```

执行：

```python
head.next.next = head
```

之后：

```text
1 <-> 2
```

因为原来的：

```text
1 -> 2
```

仍然存在。

与此同时又增加了：

```text
2 -> 1
```

因此产生了一个环：

```text
1 -> 2
^    |
|____|
```

所以必须执行：

```python
head.next = None
```

将：

```text
1 -> 2
```

断开。

最终得到：

```text
2 -> 1 -> None
```

------

## 递归过程示例

链表：

```text
1 -> 2 -> 3 -> 4
```

执行：

```python
reverseList(1)
```

首先继续递归：

```text
reverseList(1)
    reverseList(2)
        reverseList(3)
            reverseList(4)
```

当：

```text
head = 4
```

由于：

```text
head.next == None
```

直接返回：

```text
4
```

此时 `4` 会成为最终的新头节点。

------

### 回到节点 3

执行：

```python
3.next.next = 3
```

也就是：

```python
4.next = 3
```

然后：

```python
3.next = None
```

得到：

```text
4 -> 3
```

------

### 回到节点 2

执行：

```python
2.next.next = 2
```

相当于：

```python
3.next = 2
```

得到：

```text
4 -> 3 -> 2
```

------

### 回到节点 1

执行：

```python
1.next.next = 1
```

得到：

```text
4 -> 3 -> 2 -> 1
```

最终返回：

```text
4
```

------

## 算法步骤

1. 如果链表为空，直接返回 `None`。
2. 如果当前节点是最后一个节点，它就是反转后的新头节点。
3. 递归反转：

```python
head.next
```

之后的链表。

1. 让当前节点的下一个节点指回来：

```python
head.next.next = head
```

1. 将当前节点变成新的尾节点：

```python
head.next = None
```

1. 返回递归得到的新头节点。

------

## Python 代码

推荐使用下面这个稍微更简洁的版本：

```python
# Definition for singly-linked list.
#
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        # Base Case：
        # 空链表，或者已经到达链表最后一个节点
        if not head or not head.next:
            return head

        # 递归反转 head 后面的链表
        new_head = self.reverseList(head.next)

        # 让原来的下一个节点指向当前节点
        #
        # 原本：
        # head -> head.next
        #
        # 修改后：
        # head <- head.next
        head.next.next = head

        # 当前 head 将成为反转后链表的尾部
        # 因此必须断开它原来的 next 指针
        head.next = None

        # new_head 始终是原链表的最后一个节点，
        # 也就是反转后的头节点
        return new_head
```

------

## 原答案代码的写法

原答案的逻辑也是正确的：

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head:
            return None

        # 如果 head 本身就是最后一个节点，
        # 那么它就是当前子链表反转后的头节点
        newHead = head

        if head.next:
            # 先递归反转后半部分
            newHead = self.reverseList(head.next)

            # 让下一个节点指回当前节点
            head.next.next = head

        # 当前节点最终会成为尾节点
        head.next = None

        return newHead
```

不过在面试中，下面这个 Base Case：

```python
if not head or not head.next:
    return head
```

通常会让递归逻辑更加清晰。

------

## 时间复杂度

```text
O(n)
```

每个节点都会进入一次递归调用。

------

## 空间复杂度

```text
O(n)
```

虽然没有显式创建数组、字典等数据结构，但是递归调用会占用 **Call Stack（调用栈）**。

对于：

```text
1 -> 2 -> 3 -> ... -> n
```

调用栈深度最多为：

```text
n
```

因此空间复杂度是：

```text
O(n)
```

------

# Python 相关知识说明

## `Optional[ListNode]`

代码中经常看到：

```python
head: Optional[ListNode]
```

这是 Python 的类型提示。

`Optional[ListNode]` 表示：

```text
head 可能是一个 ListNode
```

也可能是：

```python
None
```

它基本等价于：

```python
ListNode | None
```

在 Python 3.10+ 中，也可以写成：

```python
def reverseList(
    self,
    head: ListNode | None
) -> ListNode | None:
```

需要注意：

> 类型提示主要用于提高代码可读性和静态检查，并不会自动限制 Python 运行时变量的类型。

------

## `while curr`

代码：

```python
while curr:
```

实际上可以理解为：

```python
while curr is not None:
```

因为链表遍历结束时：

```python
curr = None
```

所以这种写法在 Python 链表题中非常常见。

------

## `self.reverseList(...)`

因为 `reverseList` 是：

```python
class Solution
```

中的实例方法，所以递归调用时需要通过：

```python
self.reverseList(...)
```

调用当前对象上的同一个方法。

------

# 常见错误

## 1. 修改指针之前没有保存下一个节点

错误写法：

```python
curr.next = prev
curr = curr.next
```

问题在于：

执行：

```python
curr.next = prev
```

之后，`curr.next` 已经不再指向原链表中的下一个节点。

例如：

```text
0 -> 1 -> 2
```

假设：

```text
curr = 1
prev = 0
```

执行：

```python
curr.next = prev
```

之后：

```text
0 <- 1    2
```

此时已经无法通过：

```python
curr.next
```

找到 `2`。

正确写法应该是：

```python
temp = curr.next
curr.next = prev
prev = curr
curr = temp
```

------

## 2. 递归解法忘记执行 `head.next = None`

例如：

```text
1 -> 2
```

执行：

```python
head.next.next = head
```

之后：

```text
1 <-> 2
```

如果不执行：

```python
head.next = None
```

链表就会形成环。

遍历：

```python
1 -> 2 -> 1 -> 2 -> ...
```

将永远无法结束。

------

## 3. 返回了错误的节点

迭代结束后：

```python
curr = None
```

因此不能返回：

```python
return curr
```

正确答案是：

```python
return prev
```

因为此时：

```text
prev = 原链表最后一个节点
```

也就是新链表的头节点。

------

## 4. 尝试交换节点值而不是修改指针

例如先把所有值存入数组，再反向写回：

```python
values = []
```

虽然某些情况下也能得到相同的节点值顺序，但这并不是标准意义上的“反转链表”。

这道题主要考察的是：

```text
Pointer Manipulation
```

也就是：

**修改节点之间的连接关系。**

------

# 两种解法对比

| 解法 | 时间复杂度 | 空间复杂度 | 特点                   |
| ---- | ---------- | ---------- | ---------------------- |
| 迭代 | `O(n)`     | `O(1)`     | 最推荐，经典三指针     |
| 递归 | `O(n)`     | `O(n)`     | 写法简洁，但使用调用栈 |

面试中如果没有额外要求，通常优先写：

```text
迭代解法
```

因为它既满足：

```text
O(n)
```

时间复杂度，也满足：

```text
O(1)
```

额外空间复杂度。

------

# 面试理解重点

这道题真正需要掌握的不是代码本身，而是下面三个思想。

## 1. 修改指针前，先保存原来的下一节点

链表没有数组那样的下标。

一旦丢失：

```python
curr.next
```

原来指向的位置，就可能再也无法访问后面的链表。

因此链表修改操作中经常遵循：

```text
先保存
→ 再修改
```

------

## 2. 用 `prev` 表示已经处理好的部分

迭代过程中可以把链表分成两个区域：

```text
已经反转的部分 | 尚未处理的部分
```

例如：

```text
None <- 0 <- 1    2 -> 3 -> 4
              ↑    ↑
             prev curr
```

每一次循环，就是把：

```text
curr
```

从“未处理区域”移动到“已反转区域”。

这其实是一种非常重要的算法思想：

> 维护一个循环不变量（Loop Invariant）。

在每次循环开始时：

```text
prev 指向的链表一定已经正确反转。
```

然后处理一个新的 `curr`，继续保持这个性质。

------

## 3. 链表题的核心经常是“保存引用”

很多链表题困难的原因，并不是算法复杂，而是：

> 修改一个指针之后，可能会失去访问其他节点的方法。

因此看到类似：

```python
curr.next = ...
```

时应该养成一个习惯：

先问自己：

```text
我修改 curr.next 之后，
还需不需要访问原来的 curr.next？
```

如果答案是需要，就应该提前保存：

```python
next_node = curr.next
```

------

# 推荐记忆版本

如果面试中需要快速写出答案，可以记住下面这个标准模板：

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        prev = None
        curr = head

        while curr:
            next_node = curr.next
            curr.next = prev
            prev = curr
            curr = next_node

        return prev
```

核心只有四步：

```text
1. 保存 next
2. 反转 next
3. prev 前进
4. curr 前进
```

即：

```python
next_node = curr.next
curr.next = prev
prev = curr
curr = next_node
```

这四行不仅适用于 Reverse Linked List，也是后续大量链表题的重要基础。
