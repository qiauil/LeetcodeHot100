# Swap Nodes in Pairs（两两交换链表中的节点）

给定一个链表的头节点 `head`，请将链表中**每两个相邻节点进行交换**，并返回交换后的链表头节点。

要求：

- **不能修改节点内部的值**
- 必须真正改变节点之间的 `next` 指针关系
- 如果链表节点数为奇数，最后一个节点保持不变

例如：

```
输入：

1 → 2 → 3 → 4

输出：

2 → 1 → 4 → 3
```

如果链表长度为奇数：

```
输入：

1 → 2 → 3 → 4 → 5

输出：

2 → 1 → 4 → 3 → 5
```

------

# 一、核心问题

这道题的关键不在于交换：

```
1 和 2 的值
```

而是要真正交换两个：

```
ListNode
```

也就是说，原来：

```
prev → first → second → next_pair
```

交换以后必须变成：

```
prev → second → first → next_pair
```

因此每交换一组节点，本质上需要修改 **3 条 `next` 指针**。

这也是这道题最核心的地方。

------

# 方法一：迭代 + Dummy Node

这是最推荐的解法。

时间复杂度：

\[ O(n) \]

空间复杂度：

\[ O(1) \]

------

## 为什么需要 Dummy Node？

考虑最开始：

```
1 → 2 → 3 → 4
```

交换第一组：

```
1, 2
```

以后，新的头节点会变成：

```
2
```

也就是说：

```
head
```

本身会发生变化。

这种：

> **对链表头节点进行修改**

的问题，非常适合使用：

```
dummy = ListNode(0, head)
```

这样可以人为地在链表最前面增加一个虚拟节点：

```
dummy → 1 → 2 → 3 → 4
```

现在交换第一组和交换后面的任意一组节点，操作就完全统一了。

------

# 二、每次到底需要哪些指针？

假设当前结构：

```
prev → first → second → next_pair
```

其中：

```
first = prev.next
second = first.next
```

例如：

```
dummy → 1 → 2 → 3 → 4
 ↑       ↑   ↑
prev   first second
```

我们的目标是变成：

```
prev → second → first → next_pair
```

也就是：

```
dummy → 2 → 1 → 3 → 4
```

------

# 三、三步完成交换

假设：

```
prev → first → second → next_pair
```

要变成：

```
prev → second → first → next_pair
```

需要修改三条连接。

------

## 第一步：让 `first` 指向下一组

原来：

```
first → second → next_pair
```

执行：

```
first.next = second.next
```

变成：

```
first ─────────→ next_pair

second → next_pair
```

以：

```
1 → 2 → 3 → 4
```

为例：

```
first = 1
second = 2
```

执行后：

```
1 → 3 → 4

2 → 3 → 4
```

此时还没有完成交换。

------

## 第二步：让 `second` 指向 `first`

执行：

```
second.next = first
```

得到：

```
2 → 1 → 3 → 4
```

但是这时前面的：

```
prev
```

仍然还指向：

```
first
```

所以链表前半部分还没有接上。

------

## 第三步：让 `prev` 指向 `second`

执行：

```
prev.next = second
```

最终：

```
prev → second → first → next_pair
```

也就是：

```
dummy → 2 → 1 → 3 → 4
```

交换完成。

------

# 四、为什么顺序很重要？

三条关键语句是：

```
first.next = second.next
second.next = first
prev.next = second
```

这个顺序非常重要。

原因是链表没有数组下标。

如果你提前覆盖了某条：

```
next
```

指针，却没有保存后续节点的位置，就可能把后半条链表“弄丢”。

链表题中有一个非常重要的原则：

> **修改 `next` 之前，要确保之后仍然能够找到所有需要访问的节点。**

在这个算法中：

```
first = prev.next
second = first.next
```

已经保存好了当前需要交换的两个节点。

而：

```
second.next
```

就是下一组的起点，因此首先执行：

```
first.next = second.next
```

可以安全地把 `first` 接到下一组。

------

# Python 代码

```
from typing import Optional


class ListNode:
    def __init__(
        self,
        val: int = 0,
        next: Optional["ListNode"] = None
    ):
        self.val = val
        self.next = next


class Solution:
    def swapPairs(
        self,
        head: Optional[ListNode]
    ) -> Optional[ListNode]:

        # Dummy 节点放在真正的 head 前面
        # 这样即使第一组交换后 head 改变，也能统一处理
        dummy = ListNode(0, head)

        # prev 始终指向“当前需要交换的一组节点”的前一个节点
        prev = dummy

        # 必须至少剩下两个节点才能交换
        while prev.next and prev.next.next:
            # 当前要交换的两个节点
            first = prev.next
            second = first.next

            # 原结构：
            # prev -> first -> second -> next_pair

            # 1. first 接到下一组
            first.next = second.next

            # 2. second 接到 first 前面
            second.next = first

            # 3. prev 接到交换后的 second
            prev.next = second

            # 交换后：
            # prev -> second -> first -> next_pair

            # 下一轮中，first 已经成为这一组的第二个节点
            # 所以它就是下一组之前的 prev
            prev = first

        return dummy.next
```

------

# 五、完整示例推演

考虑：

```
1 → 2 → 3 → 4
```

加入 dummy：

```
dummy → 1 → 2 → 3 → 4
```

初始：

```
prev = dummy
first = 1
second = 2
```

------

## 第一组交换

原来：

```
dummy → 1 → 2 → 3 → 4
 ↑       ↑   ↑
prev   first second
```

执行：

```
first.next = second.next
```

也就是：

```
1.next = 3
```

接着：

```
second.next = first
```

也就是：

```
2.next = 1
```

再执行：

```
prev.next = second
```

于是：

```
dummy → 2 → 1 → 3 → 4
```

现在：

```
prev = first
```

所以：

```
dummy → 2 → 1 → 3 → 4
              ↑
             prev
```

------

## 第二组交换

当前：

```
prev = 1
first = 3
second = 4
```

原结构：

```
1 → 3 → 4
```

交换以后：

```
1 → 4 → 3
```

整体变成：

```
dummy → 2 → 1 → 4 → 3
```

最后：

```
return dummy.next
```

返回：

```
2
```

最终链表：

```
2 → 1 → 4 → 3
```

------

# 六、为什么交换以后 `prev = first`？

这是非常值得理解的一个细节。

交换之前：

```
prev → first → second → next_pair
```

交换以后：

```
prev → second → first → next_pair
```

注意：

```
first
```

已经变成这一组的**最后一个节点**。

因此下一组：

```
next_pair
```

前面的节点正好就是：

```
first
```

所以执行：

```
prev = first
```

下一轮就可以继续处理：

```
first → next_first → next_second
```

------

# 七、为什么循环条件是：

```
while prev.next and prev.next.next:
```

因为要进行一次交换，至少需要两个节点。

当前：

```
prev
```

指向这一组前面的节点。

第一节点：

```
prev.next
```

第二节点：

```
prev.next.next
```

所以只有：

```
prev.next != None
```

并且：

```
prev.next.next != None
```

时才能交换。

如果只剩：

```
一个节点
```

那么循环直接结束。

最后一个节点自然保持原来的位置。

------

# 八、奇数长度链表示例

例如：

```
1 → 2 → 3 → 4 → 5
```

前两组交换：

```
2 → 1 → 4 → 3 → 5
```

最后：

```
prev → 5 → None
```

此时：

```
prev.next
```

存在，但是：

```
prev.next.next
```

是：

```
None
```

所以：

```
while prev.next and prev.next.next:
```

条件失败。

节点：

```
5
```

保持不变。

最终：

```
2 → 1 → 4 → 3 → 5
```

------

# 九、Dummy Node 是什么？

`dummy node`，也叫：

- sentinel node
- 虚拟头节点
- 哨兵节点

它通常不是实际链表数据的一部分。

例如原链表：

```
1 → 2 → 3
```

创建：

```
dummy = ListNode(0, head)
```

得到：

```
dummy → 1 → 2 → 3
```

其中：

```
dummy.val
```

是多少通常并不重要。

真正重要的是：

```
dummy.next
```

指向真正的链表头节点。

------

## Dummy Node 的主要作用

Dummy node 最常用于：

> **头节点可能发生变化的链表问题。**

例如：

- 删除链表中的节点
- 合并链表
- 反转链表的一部分
- 两两交换节点
- 按组反转链表

因为使用 dummy 后：

```
头节点
```

和：

```
普通中间节点
```

可以使用同一套逻辑处理。

这是链表题中非常值得形成习惯的技巧。

------

# 十、为什么最后返回 `dummy.next`？

开始：

```
dummy → 1 → 2 → 3 → 4
```

交换后：

```
dummy → 2 → 1 → 4 → 3
```

真正的链表头节点已经变成：

```
2
```

而：

```
dummy.next
```

始终指向交换后的真正头节点。

因此：

```
return dummy.next
```

而不是：

```
return head
```

因为原来的：

```
head
```

仍然指向节点：

```
1
```

但 `1` 已经不是新链表的头节点。

------

# 十一、复杂度分析

## 时间复杂度

\[ O(n) \]

每个节点最多被访问和修改常数次。

虽然每次处理两个节点，但仍然需要处理整个链表。

所以：

\[ O(n) \]

------

## 空间复杂度

\[ O(1) \]

只使用了几个指针：

```
dummy
prev
first
second
```

并没有创建与链表长度相关的数据结构。

注意：

```
dummy = ListNode(...)
```

虽然创建了一个额外节点，但它只是**常数个额外节点**，因此额外空间仍然是：

\[ O(1) \]

------

# 方法二：递归

这道题也有一个非常优雅的递归写法。

虽然代码很短，但空间复杂度会变成：

\[ O(n) \]

因为递归调用需要使用调用栈。

------

# 十二、递归的核心思路

考虑：

```
1 → 2 → 3 → 4
```

我们可以先只关注：

```
1 → 2
```

这两个节点。

交换：

```
1 → 2
```

得到：

```
2 → 1
```

剩余部分：

```
3 → 4
```

其实就是一个**规模更小的相同问题**：

> 把从节点 `3` 开始的链表两两交换。

所以可以递归调用：

```
self.swapPairs(head.next.next)
```

整体逻辑：

```
第一对节点
+
递归处理剩余链表
```

------

# 十三、递归代码

```
from typing import Optional


class Solution:
    def swapPairs(
        self,
        head: Optional[ListNode]
    ) -> Optional[ListNode]:

        # Base Case：
        # 没有节点或者只有一个节点，不需要交换
        if not head or not head.next:
            return head

        # 当前的两个节点：
        #
        # first -> second -> rest
        #
        first = head
        second = head.next

        # 递归交换 second 后面的剩余链表
        first.next = self.swapPairs(second.next)

        # second 放到 first 前面
        second.next = first

        # 当前这一段交换以后：
        #
        # second -> first -> swapped_rest
        #
        return second
```

------

# 十四、递归过程推演

考虑：

```
1 → 2 → 3 → 4
```

第一次调用：

```
swapPairs(1)
```

当前：

```
first = 1
second = 2
```

但我们先需要知道：

```
3 → 4
```

交换以后是什么。

于是调用：

```
swapPairs(3)
```

------

## 第二层递归

当前：

```
3 → 4
```

得到：

```
first = 3
second = 4
```

继续处理：

```
swapPairs(None)
```

Base Case：

```
return None
```

然后：

```
3.next = None
4.next = 3
```

得到：

```
4 → 3
```

返回：

```
4
```

------

## 回到第一层

现在已经知道：

```
3 → 4
```

交换以后是：

```
4 → 3
```

所以：

```
1.next = 4
```

然后：

```
2.next = 1
```

得到：

```
2 → 1 → 4 → 3
```

最后：

```
return 2
```

------

# 十五、为什么递归代码只有两条修改指针的语句？

核心代码：

```
first.next = self.swapPairs(second.next)
second.next = first
```

可以拆开理解。

原来：

```
first → second → rest
```

假设递归已经把：

```
rest
```

处理成：

```
swapped_rest
```

那么：

```
first.next = swapped_rest
```

得到：

```
first → swapped_rest
```

再执行：

```
second.next = first
```

得到：

```
second → first → swapped_rest
```

所以：

```
return second
```

因为 `second` 已经成为当前子链表的新头节点。

------

# 十六、递归复杂度

时间复杂度：

\[ O(n) \]

每个节点只处理常数次。

空间复杂度：

\[ O(n) \]

更精确一点，因为每次递归处理两个节点，所以递归深度大约是：

\[ \frac{n}{2} \]

因此调用栈空间是：

\[ O(n/2)=O(n) \]

所以如果面试官明确要求：

```
O(1) extra space
```

那么应该选择：

```
迭代 + Dummy Node
```

------

# 十七、两种方法比较

| 方法              | 时间复杂度 | 空间复杂度 | 特点                       | 推荐程度 |
| ----------------- | ---------- | ---------- | -------------------------- | -------- |
| 迭代 + Dummy Node | `O(n)`     | `O(1)`     | 指针操作清晰、满足最优空间 | ⭐⭐⭐⭐⭐    |
| 递归              | `O(n)`     | `O(n)`     | 代码非常简洁               | ⭐⭐⭐⭐     |

面试中如果没有特殊要求，我更推荐：

```
迭代 + Dummy Node
```

原因是：

- 时间复杂度最优
- 空间复杂度最优
- 能体现对链表指针的理解
- Dummy node 是高频链表技巧
- 更容易扩展到 `Reverse Nodes in k-Group`

------

# 十八、为什么不能直接交换 `val`？

题目明确要求：

> 不能修改节点内部的值。

例如：

```
1 → 2
```

不能简单写：

```
first.val, second.val = second.val, first.val
```

虽然打印结果看起来会变成：

```
2 → 1
```

但实际上：

```
两个节点对象的位置完全没有变化
```

只是它们内部存储的值发生了变化。

题目要求真正从：

```
Node1 → Node2
```

变成：

```
Node2 → Node1
```

所以必须修改：

```
next pointers
```

------

# 十九、常见错误

## 1. 交换值而不是交换节点

错误：

```
first.val, second.val = second.val, first.val
```

题目明确禁止。

正确做法是修改：

```
next
```

------

# 2. 忘记保存节点导致链表断裂

链表最大的风险之一就是：

> 修改 `next` 之后，把后面的链表弄丢。

例如原来：

```
1 → 2 → 3
```

如果随意执行：

```
2.next = 1
```

现在：

```
2 → 1 → 2 → 1 ...
```

而：

```
3
```

可能无法再访问。

所以调整指针时必须先明确所有需要的节点。

推荐固定写成：

```
first = prev.next
second = first.next

first.next = second.next
second.next = first
prev.next = second
```

------

# 3. 忘记更新 `prev`

完成：

```
1 → 2
```

交换以后：

```
2 → 1
```

下一轮：

```
3 → 4
```

前面的节点是：

```
1
```

所以必须：

```
prev = first
```

如果不更新，下一轮可能继续从错误位置开始。

------

# 4. 循环条件只检查一个节点

错误：

```
while prev.next:
```

因为可能只剩一个节点。

例如：

```
1 → 2 → 3
```

交换前两个以后只剩：

```
3
```

如果还尝试：

```
second = first.next
```

此时：

```
second = None
```

后面的：

```
second.next
```

就会报错。

因此需要确保至少还有两个节点：

```
while prev.next and prev.next.next:
```

------

# 5. 返回原来的 `head`

例如：

```
1 → 2 → 3 → 4
```

交换以后：

```
2 → 1 → 4 → 3
```

原来的：

```
head
```

仍然指向：

```
1
```

它已经不是链表头。

因此使用 dummy node 时：

```
return dummy.next
```

------

# 6. 指针调整顺序错误造成环

链表指针非常容易因为赋值顺序错误形成意外的环。

例如：

```
1 → 2 → 3
```

如果先：

```
second.next = first
```

变成：

```
1 → 2
↑   ↓
└───┘
```

暂时产生：

```
1 ↔ 2
```

的环。

如果后面处理不正确，就可能永久破坏链表。

因此面试时最好先在纸上写清楚：

```
prev → first → second → next_pair
```

目标：

```
prev → second → first → next_pair
```

然后一条一条修改。

------

# 二十、一个非常实用的链表思考模板

在做链表指针题时，不要一上来写代码。

先把局部结构画出来：

```
prev → first → second → next_pair
```

然后画目标：

```
prev → second → first → next_pair
```

接着问：

> 哪几条箭头发生了变化？

原来的三条：

```
prev → first
first → second
second → next_pair
```

新的三条：

```
prev → second
second → first
first → next_pair
```

于是代码自然就是：

```
first.next = second.next
second.next = first
prev.next = second
```

这种：

> **先画箭头变化，再写代码**

的方式，对于链表题非常有效。

------

# 二十一、与反转链表的关系

这道题其实可以理解为：

> **每两个节点作为一组，对每一组执行一次局部反转。**

普通 Reverse Linked List：

```
1 → 2 → 3 → 4

↓

4 → 3 → 2 → 1
```

本题：

```
1 → 2 | 3 → 4

↓

2 → 1 | 4 → 3
```

也就是：

```
k = 2
```

的分组反转。

因此它也是更复杂问题：

> **Reverse Nodes in k-Group**

的一个简化版本。

如果你能完全掌握本题的：

```
prev
first
second
next_pair
```

以及 dummy node 的用法，那么以后理解 `k` 个节点一组反转会容易很多。

------

# 面试中的推荐解释方式

如果面试官问：

> How would you swap the nodes without modifying their values?

可以这样回答：

```
I would use a dummy node and iterate through the list two nodes at a time.

For each pair, I keep three important references:
the node before the pair, the first node, and the second node.

If the structure is:

prev -> first -> second -> next

I rewire the pointers to:

prev -> second -> first -> next

This requires three pointer updates.

After swapping the pair, I move prev to first, because first becomes
the last node of the swapped pair.

Each node is processed a constant number of times, so the time
complexity is O(n), and the extra space is O(1).
```

------

# 最终推荐代码

面试时建议优先写：

```
from typing import Optional


class Solution:
    def swapPairs(
        self,
        head: Optional[ListNode]
    ) -> Optional[ListNode]:

        # 虚拟头节点，方便统一处理第一组节点
        dummy = ListNode(0, head)

        # prev 指向当前待交换节点对的前一个节点
        prev = dummy

        # 至少存在两个节点时才能交换
        while prev.next and prev.next.next:
            first = prev.next
            second = first.next

            # prev -> first -> second -> next
            #
            # 变为：
            #
            # prev -> second -> first -> next

            first.next = second.next
            second.next = first
            prev.next = second

            # first 现在是当前交换组的最后一个节点，
            # 下一轮让它作为 prev
            prev = first

        return dummy.next
```

------

# 面试记忆总结

这道题可以浓缩成一个局部结构：

```
Before:

prev → first → second → next
```

目标：

```
After:

prev → second → first → next
```

三条核心语句：

```
first.next = second.next
second.next = first
prev.next = second
```

然后：

```
prev = first
```

继续下一组。

记忆成：

```
first 接后面
second 接 first
prev 接 second
```

复杂度：

```
Time:  O(n)
Space: O(1)
```

这道题最重要的几个知识点是：

```
Dummy Node
+
链表指针重连
+
每两个节点局部反转
```

其中最值得养成的习惯是：

> **链表题先画出修改前后的局部箭头，再决定 `next` 指针应该如何赋值。**
