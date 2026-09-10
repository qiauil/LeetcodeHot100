# Linked List Cycle II（环形链表 II）

给定一个链表的头节点 `head`，如果链表中存在环，请返回**环开始的第一个节点**；如果链表不存在环，则返回 `None`。

要求：

- **不能修改链表**
- `pos` 只是评测系统为了描述链表结构而使用的辅助信息，**不会作为参数传入**

例如：

```
head = [3, 2, 0, -4]
pos = 1
```

链表实际结构可以理解为：

```
3 → 2 → 0 → -4
    ↑         ↓
    └─────────┘
```

因为尾节点 `-4` 的 `next` 指向索引为 `1` 的节点 `2`，所以环的入口节点是：

```
2
```

------

# 一、核心问题

这道题实际上包含两个问题：

1. **链表是否存在环？**
2. 如果存在环，**环的入口在哪里？**

一个很自然的做法是使用哈希集合记录访问过的节点。

但这会需要：

\[ O(n) \]

额外空间。

更优秀、也是这道题最经典的解法是：

> **Floyd's Cycle Detection Algorithm（Floyd 判圈算法）**

也叫：

> **快慢指针（Fast & Slow Pointers）**

它可以做到：

```
Time:  O(n)
Space: O(1)
```

而且不仅可以判断有没有环，还可以找到**环的入口节点**。

------

# 方法一：Floyd 快慢指针

## 核心思路

整个算法分成两个阶段：

### 第一阶段：判断是否存在环

使用两个指针：

```
slow
fast
```

它们都从：

```
head
```

开始。

每次：

```
slow = slow.next
fast = fast.next.next
```

也就是：

- `slow` 每次走 **1 步**
- `fast` 每次走 **2 步**

如果链表不存在环，那么 `fast` 最终一定会到达：

```
None
```

如果链表存在环，那么 `fast` 和 `slow` 最终一定会在环中相遇。

------

# 为什么快慢指针一定会相遇？

可以把链表想象成：

```
A → B → C → D → E
        ↑       ↓
        H ← G ← F
```

一旦：

```
slow
fast
```

都进入环中，它们就相当于两个运动员在环形跑道上跑步。

`fast` 每轮比 `slow` 多走：

```
2 - 1 = 1
```

步。

因此，`fast` 相对于 `slow` 每轮都会靠近一格。

最终一定会追上 `slow`。

所以：

```
slow == fast
```

就意味着：

```
链表中存在环
```

------

# 第一阶段代码

```
slow = head
fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next

    if slow == fast:
        # 找到了相遇点
        break
```

这里的条件：

```
while fast and fast.next:
```

非常重要。

因为我们需要执行：

```
fast.next.next
```

所以必须首先保证：

```
fast != None
```

并且：

```
fast.next != None
```

否则会访问空指针。

------

# 第二阶段：寻找环的入口

这是这道题最巧妙的部分。

当 `slow` 和 `fast` 在环内第一次相遇以后：

1. 保持其中一个指针停在相遇点
2. 将另一个指针重新放回 `head`
3. 两个指针之后都**每次只走一步**
4. 它们下一次相遇的位置，就是**环的入口**

也就是说：

```
slow = head

while slow != fast:
    slow = slow.next
    fast = fast.next

return slow
```

乍看之下非常神奇。

为什么这样一定会在环入口相遇？

这正是这道题最值得理解的地方。

------

# 为什么第二次相遇一定是环入口？

我们给几个距离定义变量。

假设链表结构为：

```
head
 ↓
● → ● → ● → ● → ●
        ↑         ↓
        ● ← ● ← ●
        ↑
      环入口
```

设：

```
a = head 到环入口的距离
b = 环入口到第一次相遇点的距离
c = 相遇点继续走回环入口的距离
```

因此整个环的长度：

\[ L=b+c \]

可以画成：

```
        a
head ───────→ 环入口
                │
                │ b
                ↓
              相遇点
                │
                │ c
                ↓
              环入口
```

------

## 第一次相遇时，slow 走了多少？

`slow` 从 `head` 出发：

1. 先走 `a` 步进入环
2. 再走 `b` 步到达相遇点

所以：

\[ slow=a+b \]

------

## fast 走了多少？

`fast` 的速度是 `slow` 的两倍，所以：

\[ fast=2(a+b) \]

但是因为两者最终处于**同一个节点**，`fast` 比 `slow` 多走的路程一定是若干个完整的环。

假设 `fast` 比 `slow` 多绕了 `k` 圈：

\[ fast=slow+kL \]

代入：

\[ 2(a+b)=a+b+kL \]

得到：

\[ a+b=kL \]

因此：

\[ a=kL-b \]

又因为：

\[ L=b+c \]

所以：

\[ a=k(b+c)-b \]

整理：

\[ a=(k-1)L+c \]

最重要的是这个关系：

```
从 head 到环入口的距离

=

从第一次相遇点继续走到环入口的距离
+ 若干个完整环
```

因此，如果：

- 一个指针从 `head` 出发
- 另一个指针从第一次相遇点出发
- 两者速度都变成每次一步

那么当第一个指针恰好走：

```
a
```

步来到环入口时，

第二个指针也恰好走：

```
a
```

步，并最终来到环入口。

所以两者一定会在：

```
环入口
```

相遇。

------

# Python 代码

```
from typing import Optional


# LeetCode 通常已经提供 ListNode 定义
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow = head
        fast = head

        # 第一阶段：
        # 使用快慢指针判断链表是否存在环
        while fast and fast.next:
            slow = slow.next          # 每次走 1 步
            fast = fast.next.next     # 每次走 2 步

            # 快慢指针在环中相遇
            if slow == fast:
                # 第二阶段：
                # 将 slow 重新放到链表头部
                slow = head

                # 两个指针现在都每次走 1 步
                # 下一次相遇的位置就是环入口
                while slow != fast:
                    slow = slow.next
                    fast = fast.next

                return slow

        # fast 能走到 None，说明不存在环
        return None
```

------

# 更简洁的写法

```
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow = fast = head

        # 找第一次相遇点
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

            if slow == fast:
                break
        else:
            # while 正常结束：
            # fast 到达链表末尾，说明无环
            return None

        # 找环入口
        slow = head

        while slow != fast:
            slow = slow.next
            fast = fast.next

        return slow
```

这个版本使用了 Python 中比较特别的：

```
while ... else ...
```

语法。

------

# Python 语法说明：`while ... else`

Python 的循环可以带 `else`：

```
while condition:
    ...
else:
    ...
```

这里的 `else` 会在：

> 循环**没有通过 `break` 提前退出**时执行。

例如：

```
while fast and fast.next:
    ...

    if slow == fast:
        break
else:
    return None
```

有两种情况。

### 情况一：存在环

发生：

```
slow == fast
```

执行：

```
break
```

因此：

```
else
```

不会执行。

------

### 情况二：不存在环

最终：

```
fast == None
```

或者：

```
fast.next == None
```

导致 `while` 条件自然变成 `False`。

没有执行 `break`。

这时会进入：

```
else:
    return None
```

不过在代码面试中，如果你不熟悉 `while ... else`，完全没有必要使用它。

前面的第一种写法通常更加直观。

------

# 为什么比较的是节点，而不是节点的值？

代码中：

```
if slow == fast:
```

比较的是：

> 两个指针是否指向**同一个节点对象**

而不是：

```
slow.val == fast.val
```

因为链表中完全可能存在重复值。

例如：

```
1 → 2 → 3 → 2 → 5
```

这里存在两个：

```
val = 2
```

的不同节点。

它们的值虽然一样，但不是同一个节点。

所以判断快慢指针是否相遇时，我们关心的是：

```
是不是同一个节点
```

而不是：

```
节点中的值是否相同
```

这也是链表题中非常重要的概念。

------

# `Optional[ListNode]` 是什么意思？

函数定义：

```
def detectCycle(
    self,
    head: Optional[ListNode]
) -> Optional[ListNode]:
```

这里的：

```
Optional[ListNode]
```

表示：

```
这个值可以是 ListNode
也可以是 None
```

它基本等价于：

```
ListNode | None
```

在较新的 Python 版本中，也可以写：

```
def detectCycle(
    self,
    head: ListNode | None
) -> ListNode | None:
```

因为：

- 如果存在环，我们返回一个 `ListNode`
- 如果不存在环，我们返回 `None`

所以使用：

```
Optional[ListNode]
```

非常合适。

------

# 关于题目中的 `pos`

题目特别说明：

> `pos` 不作为参数进行传递。

这个地方很容易产生误解。

例如题目展示：

```
head = [3, 2, 0, -4]
pos = 1
```

并不代表你的函数会收到：

```
detectCycle(head, pos)
```

真正的函数只有：

```
detectCycle(head)
```

`pos` 只是 LeetCode 的评测系统用来构造链表。

例如：

```
pos = 1
```

表示链表尾节点指向索引：

```
1
```

对应的节点。

也就是：

```
3 → 2 → 0 → -4
    ↑         ↓
    └─────────┘
```

你的算法并不知道 `pos` 是多少。

必须通过：

```
链表自身的 next 指针关系
```

来发现环以及环的入口。

------

# 示例推演

考虑：

```
3 → 2 → 0 → -4
    ↑         ↓
    └─────────┘
```

环入口：

```
2
```

初始化：

```
slow = 3
fast = 3
```

------

## 第一轮

`slow` 走一步：

```
slow = 2
```

`fast` 走两步：

```
fast = 0
```

现在：

```
slow = 2
fast = 0
```

------

## 第二轮

`slow`：

```
2 → 0
```

所以：

```
slow = 0
```

`fast`：

```
0 → -4 → 2
```

所以：

```
fast = 2
```

------

## 第三轮

`slow`：

```
0 → -4
```

`fast`：

```
2 → 0 → -4
```

于是：

```
slow = fast = -4
```

第一次相遇。

------

## 第二阶段

现在：

```
slow = head = 3
fast = -4
```

两个指针每次都只走一步。

第一步：

```
slow: 3 → 2
fast: -4 → 2
```

于是：

```
slow == fast
```

两者都来到：

```
2
```

所以：

```
return slow
```

返回环入口节点。

------

# 复杂度分析

## 时间复杂度

\[ O(n) \]

第一阶段，快慢指针最多只会在线性数量的节点范围内移动。

第二阶段，再从头节点和相遇点向前移动，最多也是：

\[ O(n) \]

因此总时间复杂度仍然是：

\[ O(n) \]

不是：

\[ O(n)+O(n)=O(2n) \]

在 Big-O 中忽略常数，所以：

\[ O(n) \]

------

## 空间复杂度

\[ O(1) \]

只使用了两个指针：

```
slow
fast
```

没有额外的数据结构。

------

# 方法二：Hash Set

理解 Floyd 算法之前，也可以先从一个更直观的方法开始。

## 思路

使用一个集合：

```
visited
```

保存所有已经访问过的节点。

遍历链表时：

```
如果当前节点以前出现过
→ 说明当前节点就是环入口
```

因为从 `head` 开始依次向后遍历：

> 第一个被访问第二次的节点，一定就是进入环之后重新遇到的第一个节点，也就是环入口。

------

# Python 代码

```
from typing import Optional


class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        visited = set()

        current = head

        while current:
            # 当前节点以前访问过
            # 说明这里就是环入口
            if current in visited:
                return current

            # 记录当前节点
            visited.add(current)

            current = current.next

        # 能到达 None，说明不存在环
        return None
```

------

# Python 数据结构说明：`set`

Python 的：

```
set
```

是哈希集合。

常见操作：

```
visited.add(node)
```

把元素加入集合。

判断：

```
node in visited
```

平均时间复杂度为：

\[ O(1) \]

因此整个链表只需要遍历一次。

------

# Hash Set 方法复杂度

时间复杂度：

\[ O(n) \]

空间复杂度：

\[ O(n) \]

因为最坏情况下需要把所有节点放入：

```
visited
```

------

# 两种方法比较

| 方法           | 时间复杂度 | 空间复杂度 | 是否修改链表 | 推荐程度 |
| -------------- | ---------- | ---------- | ------------ | -------- |
| Hash Set       | `O(n)`     | `O(n)`     | ❌            | ⭐⭐⭐      |
| Floyd 快慢指针 | `O(n)`     | `O(1)`     | ❌            | ⭐⭐⭐⭐⭐    |

面试中更推荐：

```
Floyd 快慢指针
```

因为它达到了：

```
O(n) Time
O(1) Space
```

------

# 常见错误

## 1. 只检测有没有环，却不知道如何找到入口

很多人熟悉：

```
slow = slow.next
fast = fast.next.next
```

并且知道：

```
slow == fast
```

说明有环。

但这只解决了：

```
Linked List Cycle
```

也就是“有没有环”。

本题进一步要求：

```
环从哪里开始
```

所以第一次相遇之后不能直接：

```
return slow
```

第一次相遇点**不一定是环入口**。

正确操作是：

```
slow = head

while slow != fast:
    slow = slow.next
    fast = fast.next

return slow
```

------

# 2. 把第一次相遇点当成环入口

例如：

```
head → ● → ● → A → ● → ● → ●
              ↑             ↓
              └────── X ←───┘
```

假设：

```
A = 环入口
X = 快慢指针第一次相遇点
```

通常：

```
A != X
```

所以不能第一次相遇就返回。

第一次相遇的作用只是：

1. 证明存在环
2. 为第二阶段寻找环入口提供一个特殊位置

------

# 3. 第二阶段仍然让 fast 每次走两步

这是一个非常常见的错误。

第一阶段：

```
slow = slow.next
fast = fast.next.next
```

第二阶段必须改成：

```
slow = slow.next
fast = fast.next
```

也就是说：

> 第二阶段两个指针速度必须一样。

否则前面的数学关系不成立。

------

# 4. 忘记处理无环链表

错误代码：

```
while slow != fast:
    ...
```

如果没有先判断链表是否有环，就可能产生：

- 空指针错误
- 无限循环
- 错误结果

必须先通过：

```
while fast and fast.next:
```

检查。

如果 `fast` 能走到链表末尾：

```
None
```

就说明：

```
return None
```

------

# 5. 使用节点值判断是否相遇

错误：

```
if slow.val == fast.val:
```

正确：

```
if slow == fast:
```

因为我们要比较的是：

```
节点身份
```

而不是：

```
节点值
```

链表完全允许不同节点拥有相同的值。

------

# 6. 试图使用 `pos`

题目中的：

```
pos
```

不是函数参数。

因此不能：

```
return nodes[pos]
```

也不能假设：

```
pos
```

在代码中存在。

它只是评测系统描述输入的方法。

------

# 7. 修改链表来标记访问状态

有些人可能想到：

```
访问一个节点之后修改它的 next
```

或者：

```
修改节点的 value
```

用于表示“这个节点已经访问过”。

但题目明确要求：

> 不允许修改链表。

因此不能使用这种做法。

Floyd 算法完全不需要修改任何节点。

------

# 为什么 Floyd 算法这么重要？

Floyd 算法实际上是一类非常经典的：

> **双指针 + 不同速度**

问题。

其核心结构是：

```
slow = 1 step
fast = 2 steps
```

利用速度差发现：

```
重复状态 / 环
```

不仅可以应用于链表。

更一般地，如果一个状态不断执行：

```
state = f(state)
```

并且状态最终会出现循环，那么 Floyd 算法也可能适用。

因此有时它也被称为：

> **Cycle Detection Algorithm**

而不仅仅是链表算法。

------

# 为什么第二阶段要把其中一个指针放回 head？

可以把最关键的数学关系简化记成：

设：

```
a = head 到环入口
b = 环入口到相遇点
L = 环长度
```

第一次相遇时：

```
slow 走了 a + b
fast 走了 2(a + b)
```

两者路程差一定是完整的若干圈：

\[ 2(a+b)-(a+b)=kL \]

所以：

\[ a+b=kL \]

因此：

\[ a=kL-b \]

而：

```
L - b
```

就是：

```
相遇点 → 环入口
```

的距离。

所以：

```
head → 环入口
```

和：

```
相遇点 → 环入口
```

在“允许多绕若干圈”的意义下距离相同。

于是让两个指针：

```
一个从 head 开始
一个从 meeting point 开始
```

每次都走一步。

它们最终会在：

```
环入口
```

相遇。

------

# 面试中的推荐解释方式

如果面试官问：

> How would you solve this with constant extra space?

可以这样组织答案：

```
I would use Floyd's cycle detection algorithm with two pointers.

First, I use a slow pointer that moves one step at a time and a fast
pointer that moves two steps at a time.

If there is no cycle, the fast pointer will eventually reach null.

If there is a cycle, the two pointers will eventually meet inside
the cycle.

After they meet, I move one pointer back to the head. Then I move
both pointers one step at a time.

The node where they meet again is the entry point of the cycle.

This gives O(n) time and O(1) extra space.
```

如果继续追问：

> Why does the second meeting point correspond to the cycle entrance?

可以进一步解释：

```
Let a be the distance from the head to the cycle entrance,
and b be the distance from the entrance to the first meeting point.

At the first meeting, the fast pointer has traveled twice as far as
the slow pointer, and the difference between their traveled distances
must be a multiple of the cycle length.

This gives:

a + b = k * cycle_length

Therefore, the distance from the head to the cycle entrance is
equivalent to the distance from the meeting point to the cycle
entrance modulo the cycle length.

So if one pointer starts at the head and the other starts at the
meeting point, and both move one step at a time, they meet at the
cycle entrance.
```

------

# 最终推荐代码

面试中建议优先写这个版本：

```
from typing import Optional


class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow = head
        fast = head

        # Phase 1:
        # 判断是否存在环，并找到第一次相遇点
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

            if slow == fast:
                # Phase 2:
                # 从 head 和相遇点同时出发
                slow = head

                while slow != fast:
                    slow = slow.next
                    fast = fast.next

                # 第二次相遇的位置就是环入口
                return slow

        # fast 到达 None，说明不存在环
        return None
```

------

# 面试记忆总结

这道题可以浓缩成两个阶段。

### Phase 1：找相遇点

```
slow：每次 1 步
fast：每次 2 步
```

如果：

```
fast 到 None
```

则：

```
无环
```

如果：

```
slow == fast
```

则：

```
存在环
```

------

### Phase 2：找环入口

第一次相遇之后：

```
一个指针回到 head
另一个留在 meeting point
```

然后：

```
两个都每次走 1 步
```

下一次相遇：

```
= 环入口
```

因此可以记成：

```
第一次相遇：证明有环

第二次相遇：找到入口
```

复杂度：

```
Time:  O(n)
Space: O(1)
```

这道题最重要的面试知识点是：

> **Floyd 快慢指针不仅能够检测环，还能够通过第二次等速相遇找到环的入口。**
