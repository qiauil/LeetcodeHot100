# 删除链表的倒数第 N 个节点（Remove Nth Node From End of List）

## 题目描述

给定一个单链表的头节点 `head` 和一个整数 `n`，删除链表中**倒数第 `n` 个节点**，并返回删除后的链表头节点。

---

## 示例 1

```text
输入：
head = [1, 2, 3, 4]
n = 2

输出：
[1, 2, 4]
```

倒数第 `2` 个节点是：

```text
3
```

删除后：

```text
1 -> 2 -> 4
```

---

## 示例 2

```text
输入：
head = [5]
n = 1

输出：
[]
```

唯一的节点就是倒数第 `1` 个节点，因此删除后链表为空。

---

## 示例 3

```text
输入：
head = [1, 2]
n = 2

输出：
[2]
```

倒数第 `2` 个节点就是头节点 `1`，删除后只剩：

```text
2
```

---

## 约束

- 链表节点总数记为 `sz`
- `1 <= sz <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= sz`

---

# 一、使用数组保存所有节点

## 核心思路

链表的问题在于：

> 不能像数组一样通过下标直接访问第几个节点。

所以最直接的方法是先把所有链表节点保存进数组。

例如：

```text
1 -> 2 -> 3 -> 4
```

保存后：

```python
nodes = [node1, node2, node3, node4]
```

如果链表长度是：

```text
N
```

那么倒数第 `n` 个节点对应从前往后的下标：

```text
N - n
```

例如：

```text
N = 4
n = 2
```

那么：

```text
removeIndex = 4 - 2 = 2
```

对应数组中的：

```text
nodes[2]
```

也就是节点 `3`。

---

## 删除链表节点的本质

单链表中，如果要删除：

```text
prev -> target -> next
```

只需要修改：

```python
prev.next = target.next
```

就可以变成：

```text
prev -> next
```

因此我们只需要找到被删除节点的前一个节点。

---

## 算法步骤

1. 遍历链表，把每个节点对象加入数组 `nodes`。
2. 计算需要删除节点的下标：

```python
removeIndex = len(nodes) - n
```

3. 如果：

```python
removeIndex == 0
```

说明需要删除头节点，直接返回：

```python
head.next
```

4. 否则让前一个节点跳过目标节点：

```python
nodes[removeIndex - 1].next = nodes[removeIndex].next
```

5. 返回原来的 `head`。

---

## Python 实现

```python
from typing import Optional


# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def removeNthFromEnd(
        self,
        head: Optional[ListNode],
        n: int
    ) -> Optional[ListNode]:

        nodes = []
        cur = head

        # 保存链表中的所有节点对象
        while cur:
            nodes.append(cur)
            cur = cur.next

        # 倒数第 n 个节点对应从前往后的 0-based 下标
        removeIndex = len(nodes) - n

        # 如果要删除的是头节点
        if removeIndex == 0:
            return head.next

        # 让前一个节点跳过目标节点
        nodes[removeIndex - 1].next = nodes[removeIndex].next

        return head
```

---

## Python 类型说明：`Optional[ListNode]`

代码中：

```python
head: Optional[ListNode]
```

表示：

```text
head 要么是一个 ListNode
要么是 None
```

它等价于比较现代的 Python 写法：

```python
head: ListNode | None
```

因为链表可能为空，所以链表头节点的类型通常会使用 `Optional`。

需要：

```python
from typing import Optional
```

---

## 复杂度分析

设链表长度为 `N`。

- **时间复杂度：`O(N)`**
  - 遍历链表一次。

- **空间复杂度：`O(N)`**
  - 数组 `nodes` 保存了所有节点引用。

---

## 这种方法的优缺点

优点：

- 非常直观；
- 下标计算容易理解；
- 不容易出现复杂的指针错误。

缺点：

- 使用了 `O(N)` 额外空间；
- 没有充分利用链表本身的结构。

因此面试中通常可以先作为基础思路，然后继续优化空间复杂度。

---

# 二、两次遍历（Two Pass）

## 核心思路

其实我们并不需要真的把所有节点保存下来。

只要知道链表长度：

```text
N
```

就可以计算要删除的节点从头开始的位置：

```text
N - n
```

所以可以：

1. 第一次遍历链表，计算长度 `N`；
2. 第二次遍历，找到目标节点的前一个节点；
3. 修改 `next` 指针完成删除。

这样就可以把空间复杂度降低到：

```text
O(1)
```

---

## 下标关系

假设链表长度为：

```text
N
```

倒数第：

```text
n
```

个节点，对应从头开始的 `0-based` 下标：

```text
N - n
```

例如：

```text
1 -> 2 -> 3 -> 4
N = 4
n = 2
```

目标节点下标：

```text
4 - 2 = 2
```

即：

```text
3
```

---

## 算法步骤

1. 第一次遍历链表，统计节点数量 `N`。
2. 计算：

```python
removeIndex = N - n
```

3. 如果：

```python
removeIndex == 0
```

说明删除头节点，返回：

```python
head.next
```

4. 否则再次从头开始遍历。
5. 停在目标节点的前一个节点。
6. 执行：

```python
cur.next = cur.next.next
```

7. 返回 `head`。

---

## Python 实现

原始代码可以进一步简化，使“走到前一个节点”的含义更直接：

```python
from typing import Optional


# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def removeNthFromEnd(
        self,
        head: Optional[ListNode],
        n: int
    ) -> Optional[ListNode]:

        # 第一次遍历：计算链表长度
        N = 0
        cur = head

        while cur:
            N += 1
            cur = cur.next

        # 目标节点从头开始的 0-based 下标
        removeIndex = N - n

        # 删除头节点
        if removeIndex == 0:
            return head.next

        # 第二次遍历：
        # 走到目标节点的前一个节点
        cur = head

        for _ in range(removeIndex - 1):
            cur = cur.next

        # 跳过目标节点
        cur.next = cur.next.next

        return head
```

---

## 为什么循环是 `removeIndex - 1` 次？

如果目标节点下标是：

```text
removeIndex
```

我们真正需要停在：

```text
removeIndex - 1
```

的位置。

例如：

```text
1 -> 2 -> 3 -> 4
```

要删除 `3`：

```text
removeIndex = 2
```

需要停在节点 `2`：

```text
下标 1
```

从头节点下标 `0` 出发，只需要移动：

```text
1 次
```

也就是：

```python
range(removeIndex - 1)
```

---

## 复杂度分析

- **时间复杂度：`O(N)`**
  - 虽然遍历两次，但：

```text
O(N) + O(N) = O(N)
```

- **空间复杂度：`O(1)`**

---

# 三、递归

## 核心思路

递归有一个天然特点：

> 可以先一路走到链表末尾，然后在递归返回的过程中从后往前处理节点。

因此非常适合“从链表尾部开始计数”。

例如：

```text
1 -> 2 -> 3 -> 4
```

递归深入顺序：

```text
1
2
3
4
None
```

而递归返回顺序：

```text
4
3
2
1
```

这正好相当于：

```text
从尾部向前数
```

---

## 为什么使用 `n = [n]`？

原代码中：

```python
return self.rec(head, [n])
```

传入的不是整数：

```python
n
```

而是：

```python
[n]
```

这是因为 Python 中整数是不可变对象。

如果递归函数中写：

```python
n -= 1
```

只会修改当前函数自己的局部变量，不会让其他递归层共享同一个计数器。

而列表是可变对象：

```python
n = [2]
```

每一层递归都引用同一个列表，因此：

```python
n[0] -= 1
```

可以让所有递归层共享同一个倒计时。

---

## 递归过程

核心代码：

```python
head.next = self.rec(head.next, n)
n[0] -= 1

if n[0] == 0:
    return head.next

return head
```

含义是：

1. 先递归处理后面的链表；
2. 返回时计数减一；
3. 当计数变成 `0`：
   - 当前节点就是倒数第 `n` 个节点；
   - 返回 `head.next`；
   - 相当于让上一层直接连接当前节点的下一个节点；
4. 其他节点正常返回自己。

---

## Python 实现

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def rec(self, head, n):
        # 递归到链表末尾
        if not head:
            return None

        # 先处理后面的链表
        head.next = self.rec(head.next, n)

        # 在递归回溯过程中，从尾部开始计数
        n[0] -= 1

        # 当前节点正好是倒数第 n 个节点
        if n[0] == 0:
            # 返回下一个节点，相当于删除当前节点
            return head.next

        return head

    def removeNthFromEnd(self, head, n):
        # 用列表包装 n，让所有递归层共享同一个可变计数器
        return self.rec(head, [n])
```

---

## 一个递归示例

假设：

```text
1 -> 2 -> 3 -> 4
n = 2
```

初始：

```text
n[0] = 2
```

递归到末尾后开始返回。

处理 `4`：

```text
n[0] = 1
```

保留 `4`。

处理 `3`：

```text
n[0] = 0
```

说明：

```text
3
```

就是倒数第 `2` 个节点。

于是返回：

```text
3.next
```

也就是：

```text
4
```

结果链表变成：

```text
1 -> 2 -> 4
```

---

## 复杂度分析

- **时间复杂度：`O(N)`**
- **空间复杂度：`O(N)`**
  - 递归调用栈最多有 `N` 层。

---

## 递归解法的评价

这种思路比较巧妙，但面试中通常不是最推荐的答案。

主要原因：

- 使用 `O(N)` 调用栈；
- 使用可变列表包装 `n` 的技巧不如双指针直观；
- 链表很长时递归还可能遇到 Python 的递归深度限制。

本题节点数最多只有：

```text
30
```

因此不会有实际问题。

---

# 四、双指针：一次遍历

## 核心思路

这是本题最经典、最推荐的解法。

目标是找到：

> 倒数第 `n` 个节点的前一个节点。

如果能让两个指针始终保持固定距离，就可以在一次遍历中定位它。

---

# 快慢指针的关键关系

创建两个指针：

```text
left
right
```

让：

```text
right
```

先向前移动 `n` 步。

这样：

```text
left 和 right
```

之间就保持了 `n` 个节点的距离。

之后两个指针一起向前移动。

当：

```text
right == None
```

时，`left` 就会恰好停在：

```text
待删除节点的前一个节点
```

---

# 为什么需要 Dummy Node？

如果要删除的恰好是头节点，例如：

```text
1 -> 2
n = 2
```

目标节点是：

```text
1
```

但头节点前面没有真实节点。

所以我们人为创建一个虚拟节点：

```text
dummy -> 1 -> 2
```

这样即使删除头节点，也可以统一写成：

```python
left.next = left.next.next
```

无需单独处理。

---

## Dummy Node 是链表题中非常重要的技巧

```python
dummy = ListNode(0, head)
```

此时：

```text
dummy.next == head
```

如果删除原头节点：

```python
dummy.next = dummy.next.next
```

那么新的头节点自然就是：

```python
dummy.next
```

因此最终统一返回：

```python
dummy.next
```

---

# 双指针具体过程

假设：

```text
head = 1 -> 2 -> 3 -> 4
n = 2
```

建立：

```text
dummy -> 1 -> 2 -> 3 -> 4
  ^
 left

right -> 1
```

先让 `right` 移动 `2` 步：

```text
right -> 3
```

此时：

```text
left  -> dummy
right -> 3
```

然后一起移动：

```text
left  -> 1
right -> 4
```

再移动：

```text
left  -> 2
right -> None
```

此时：

```text
left.next -> 3
```

正好是需要删除的节点。

执行：

```python
left.next = left.next.next
```

链表变成：

```text
1 -> 2 -> 4
```

---

# 为什么 `right` 必须先走 `n` 步？

这是最容易出现 off-by-one 错误的地方。

我们的目标是让 `left` 最后停在：

```text
目标节点的前一个节点
```

而不是目标节点本身。

由于：

```text
left
```

从 `dummy` 开始，而：

```text
right
```

从 `head` 开始，

让 `right` 再前进：

```text
n
```

步后，两者之间正好形成所需距离。

---

## 算法步骤

1. 创建虚拟节点：

```python
dummy = ListNode(0, head)
```

2. 初始化：

```python
left = dummy
right = head
```

3. 让 `right` 先向前移动 `n` 次。
4. 当 `right` 不为空时：
   - `left = left.next`
   - `right = right.next`
5. 此时：

```python
left.next
```

就是目标节点。
6. 删除：

```python
left.next = left.next.next
```

7. 返回：

```python
dummy.next
```

---

# Python 实现

```python
from typing import Optional


# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def removeNthFromEnd(
        self,
        head: Optional[ListNode],
        n: int
    ) -> Optional[ListNode]:

        # 虚拟头节点，统一处理“删除原头节点”的情况
        dummy = ListNode(0, head)

        left = dummy
        right = head

        # 先让 right 向前移动 n 步
        for _ in range(n):
            right = right.next

        # 两个指针同步向前移动
        #
        # 当 right 到达 None 时，
        # left 正好停在待删除节点的前一个节点
        while right:
            left = left.next
            right = right.next

        # 删除目标节点
        left.next = left.next.next

        # dummy.next 始终代表删除后的真正头节点
        return dummy.next
```

---

# 为什么原代码修改 `n` 也可以？

原代码使用：

```python
while n > 0:
    right = right.next
    n -= 1
```

这没有问题，因为函数后续已经不需要原始的 `n`。

不过从可读性角度，面试代码中也可以写成：

```python
for _ in range(n):
    right = right.next
```

这样：

- 不修改输入参数；
- 意图更清晰；
- 更容易看出“right 先走 n 步”。

---

# 复杂度分析

- **时间复杂度：`O(N)`**
  - 每个指针最多遍历一次链表。

- **空间复杂度：`O(1)`**
  - 只使用常数个指针和一个 dummy 节点。

因此这是本题最优、最推荐的解法。

---

# 为什么说这是“一次遍历”？

虽然：

```text
right
```

先走了 `n` 步，然后两个指针一起移动，

但整个过程中：

```text
right
```

从头到尾只经过一次链表。

因此总操作量仍然是线性的：

```text
O(N)
```

不需要像 Two Pass 解法那样：

```text
先完整遍历一次
再从头重新遍历
```

---

# 常见错误

## 1. 忘记处理删除头节点

例如：

```text
head = [1, 2]
n = 2
```

需要删除：

```text
1
```

如果没有 dummy 节点，很容易需要单独判断：

```python
if n == length:
    return head.next
```

使用：

```python
dummy = ListNode(0, head)
```

则所有删除情况都可以统一处理。

---

## 2. `right` 只前进 `n - 1` 步

错误：

```python
for _ in range(n - 1):
    right = right.next
```

这会导致最终：

```text
left
```

停在目标节点本身，而不是它的前一个节点。

随后执行：

```python
left.next = left.next.next
```

就会删除错误的节点。

标准模板是：

```python
left = dummy
right = head

for _ in range(n):
    right = right.next
```

---

## 3. `left` 从 `head` 开始

如果写：

```python
left = head
right = head
```

那么删除头节点时就很难统一处理。

使用：

```python
left = dummy
```

可以让 `left` 有机会停在头节点前面。

这正是 dummy 节点存在的核心意义。

---

## 4. 返回原来的 `head`

如果删除的不是头节点：

```python
return head
```

当然没有问题。

但如果删除的就是原头节点，那么：

```python
head
```

仍然指向已经被移除的旧节点。

所以使用 dummy 节点时应该统一返回：

```python
return dummy.next
```

这样无论删除哪个节点都正确。

---

## 5. 删除节点时修改错指针

正确：

```python
left.next = left.next.next
```

假设：

```text
left -> A -> B
```

删除 `A` 后：

```text
left -> B
```

不要写成：

```python
left = left.next.next
```

这只会修改局部变量 `left` 自己指向哪里，并不会修改链表结构。

---

# Python / 链表概念说明：变量保存的是节点引用

假设：

```python
cur = head
```

这里不是复制整个链表。

`cur` 和 `head` 都引用同一个节点对象。

因此：

```python
cur.next = cur.next.next
```

实际上会修改链表本身的连接关系。

而：

```python
cur = cur.next
```

只是让变量 `cur` 改为引用下一个节点，不会修改链表。

这一区别是链表题非常重要的基础。

---

# 解法对比

| 方法 | 时间复杂度 | 空间复杂度 | 是否需要多次完整遍历 | 面试推荐 |
|---|---:|---:|---|---|
| 数组保存节点 | `O(N)` | `O(N)` | 否 | 基础思路 |
| 两次遍历 | `O(N)` | `O(1)` | 是 | 推荐 |
| 递归 | `O(N)` | `O(N)` | 否 | 了解即可 |
| 双指针 + Dummy | `O(N)` | `O(1)` | **否** | **最推荐** |

---

# 面试中最值得掌握的思维

这道题最核心的技巧有两个。

## 1. 固定距离双指针

当题目出现：

```text
倒数第 k 个节点
```

可以考虑：

```text
让一个指针先走 k 步
然后两个指针同步移动
```

这种“固定间隔”技巧非常常见。

---

## 2. Dummy Node

当链表题可能涉及：

```text
删除头节点
插入到头节点之前
头节点本身可能变化
```

时，非常值得优先考虑 dummy 节点。

典型模板：

```python
dummy = ListNode(0, head)
```

最后返回：

```python
dummy.next
```

这通常可以显著减少边界条件判断。

---

# 推荐记忆模板

```python
class Solution:
    def removeNthFromEnd(
        self,
        head: Optional[ListNode],
        n: int
    ) -> Optional[ListNode]:

        dummy = ListNode(0, head)

        left = dummy
        right = head

        # 保持 n 个节点的间隔
        for _ in range(n):
            right = right.next

        # 一起移动
        while right:
            left = left.next
            right = right.next

        # left.next 就是需要删除的节点
        left.next = left.next.next

        return dummy.next
```

可以把整个算法记成：

```text
dummy
→ right 先走 n 步
→ left/right 同步移动
→ right 到末尾
→ left.next 就是目标
→ 跳过目标节点
```

---

# 总结

这道题最推荐：

```text
Dummy Node + 双指针
```

核心逻辑是：

```text
right 比 left 提前保持固定距离
```

从而把：

```text
“倒数第 n 个节点”
```

转化成：

```text
“当 right 到达链表末尾时，left 所在的位置”
```

更重要的是，这道题体现了两个非常通用的链表技巧：

```text
1. Dummy Node 统一处理头节点边界
2. 快慢 / 固定间距双指针定位倒数位置
```

这两个技巧在后续大量链表面试题中都会反复出现。
