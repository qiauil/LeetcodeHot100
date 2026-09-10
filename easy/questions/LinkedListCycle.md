# Linked List Cycle（环形链表）

给定一个链表的头节点 `head`，判断该链表中是否存在环。

如果链表中存在某个节点，使得从该节点开始不断沿着 `next` 指针向后移动，最终能够再次回到这个节点，那么我们就认为链表中存在环。

题目内部会使用 `pos` 表示链表尾节点的 `next` 指针连接到哪个节点，其中 `pos` 是该节点在链表中的索引。

**注意：`pos` 并不会作为参数传入函数。**

如果链表中存在环，返回 `True`；否则返回 `False`。

------

## 解法一：哈希集合 Hash Set

### 思路

一个非常直接的想法是：

> **记录所有已经访问过的节点。**

我们从 `head` 开始不断沿着 `next` 指针向后遍历。

如果某一次访问到的节点之前已经访问过，说明链表又回到了之前的位置，因此链表中一定存在环。

例如：

```text
1 → 2 → 3 → 4
        ↑     ↓
        ← ← ←
```

遍历顺序可能是：

```text
1 → 2 → 3 → 4 → 3
```

当第二次遇到节点 `3` 时，就可以确定链表存在环。

相反，如果最终走到了 `None`，说明链表正常结束，因此不存在环。

### 算法步骤

1. 创建一个空的哈希集合 `seen`，用于保存已经访问过的节点。
2. 使用指针 `cur` 从 `head` 开始遍历链表。
3. 对于当前节点：
   - 如果 `cur` 已经存在于 `seen` 中，说明出现了重复节点，返回 `True`。
   - 否则，将 `cur` 加入 `seen`。
4. 将 `cur` 移动到 `cur.next`。
5. 如果最终 `cur == None`，说明不存在环，返回 `False`。

```python
# 单链表节点定义
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        # 保存已经访问过的“节点对象”
        seen = set()

        cur = head

        while cur:
            # 如果当前节点之前已经访问过，
            # 说明链表绕回了之前的位置，存在环
            if cur in seen:
                return True

            # 记录当前节点
            seen.add(cur)

            # 继续访问下一个节点
            cur = cur.next

        # 能走到 None，说明链表正常结束
        return False
```

### Python 中的 `set`

`set` 是 Python 提供的哈希集合，核心特点是：

- 元素不能重复；
- 判断某个元素是否存在通常只需要平均 `O(1)` 时间；
- 添加元素通常也是平均 `O(1)` 时间。

这里用到：

```python
cur in seen
```

用于判断节点是否已经访问过。

以及：

```python
seen.add(cur)
```

用于记录当前节点。

需要特别注意，这里保存的是 **`ListNode` 节点对象本身**，而不是：

```python
cur.val
```

因为不同节点完全可能拥有相同的值。

例如：

```text
1 → 2 → 1 → None
```

两个值为 `1` 的节点并不是同一个节点，因此不能仅通过节点值判断是否存在环。

### 复杂度分析

假设链表中一共涉及 `n` 个不同节点。

- **时间复杂度：`O(n)`**
  - 每个节点最多被访问一次。
  - 哈希集合的查找和插入平均为 `O(1)`。
- **空间复杂度：`O(n)`**
  - 最坏情况下需要把所有节点都保存进 `seen`。

------

# 解法二：快慢指针 / Floyd 判圈算法

这是这道题更经典、也更推荐在面试中掌握的解法。

### 核心思路

我们同时维护两个指针：

- `slow`：每次走 **1 步**
- `fast`：每次走 **2 步**

如果链表不存在环，那么 `fast` 由于移动得更快，最终一定会先走到链表末尾：

```text
1 → 2 → 3 → 4 → None
slow →

fast    →
```

如果链表存在环，那么一旦两个指针都进入环中，它们就会一直在环里移动。

由于：

```text
fast 每轮走 2 步
slow 每轮走 1 步
```

因此 `fast` 每轮实际上都会比 `slow` **多靠近 1 步**。

最终 `fast` 一定会追上 `slow`。

可以把它想象成两个人在圆形跑道上跑步：

```text
slow：1 格 / 秒
fast：2 格 / 秒
```

只要两个人一直在圆形跑道上跑，跑得快的人最终一定会套圈追上跑得慢的人。

------

## 为什么相遇就说明一定存在环？

如果链表没有环，那么节点只能一路向后：

```text
A → B → C → D → None
```

两个指针不可能在移动之后重新回到同一个节点。

而如果存在环：

```text
A → B → C → D
        ↑     ↓
        F ← E
```

`slow` 和 `fast` 进入环以后，会不断沿着：

```text
C → D → E → F → C → ...
```

循环移动。

由于 `fast` 比 `slow` 每轮多走一步，所以二者之间的距离会不断缩小，最终相遇。

------

## 算法步骤

1. 初始化两个指针：

```python
slow = head
fast = head
```

1. 当：

```python
fast
```

和：

```python
fast.next
```

都不为 `None` 时继续循环。

1. 每一轮：
   - `slow` 向前移动一步；
   - `fast` 向前移动两步。
2. 如果：

```python
slow == fast
```

说明两个指针指向了同一个节点，因此存在环。

1. 如果循环因为 `fast` 或 `fast.next` 为 `None` 而结束，说明链表不存在环。

```python
# 单链表节点定义
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        # slow 每次移动一步
        # fast 每次移动两步
        slow, fast = head, head

        # fast 要移动两步，因此必须同时保证：
        # 1. fast 不为 None
        # 2. fast.next 不为 None
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

            # 比较的是两个指针是否指向同一个节点
            if slow == fast:
                return True

        # fast 能够走到链表末尾，说明不存在环
        return False
```

### 复杂度分析

- **时间复杂度：`O(n)`**
- **空间复杂度：`O(1)`**

快慢指针不需要额外保存访问过的节点，因此空间复杂度优于哈希集合。

------

# 为什么 Floyd 算法是 `O(n)`？

这个地方在面试中有时会被追问。

假设：

- 从链表头到环入口的距离为 `a`
- 环的长度为 `c`

`slow` 进入环之前最多走 `a` 步。

进入环以后，`fast` 相对于 `slow` 每轮会前进：

```text
2 - 1 = 1
```

个节点。

因此，它们最多再经过 `c` 次移动就会相遇。

总移动次数不会超过一个线性量级：

```text
O(a + c)
```

而：

```text
a + c <= n
```

因此总体时间复杂度为：

```text
O(n)
```

------

# 两种方法对比

| 方法     | 时间复杂度 | 空间复杂度 | 特点                 |
| -------- | ---------- | ---------- | -------------------- |
| 哈希集合 | `O(n)`     | `O(n)`     | 最直观，容易想到     |
| 快慢指针 | `O(n)`     | `O(1)`     | 空间更优，面试更推荐 |

如果面试官只要求判断是否存在环，通常优先使用 **Floyd 快慢指针算法**。

哈希集合法则非常适合作为首先提出的直观方案，然后再进一步优化空间复杂度：

```text
Hash Set：O(n) space
        ↓
思考是否真的需要保存所有节点
        ↓
Fast & Slow Pointers：O(1) space
```

这种从简单方案逐步优化的思路本身也是面试中很好的表达方式。

------

# 常见错误

## 1. 没有检查 `fast.next`

错误写法：

```python
while fast:
    fast = fast.next.next
```

如果链表为：

```text
1 → 2 → 3 → None
```

某一时刻：

```python
fast = 3
fast.next = None
```

此时执行：

```python
fast.next.next
```

就相当于：

```python
None.next
```

会产生错误。

因此应该写：

```python
while fast and fast.next:
```

Python 中的 `and` 会进行**短路求值（short-circuit evaluation）**。

也就是说：

```python
fast and fast.next
```

如果 `fast` 已经是 `None`，Python 就不会继续计算：

```python
fast.next
```

从而避免异常。

------

## 2. 比较节点值，而不是节点对象

错误：

```python
if slow.val == fast.val:
    return True
```

例如：

```text
1 → 2 → 3 → 2 → None
```

这里虽然有两个节点的值都是 `2`，但它们是两个不同的节点。

因此：

```python
slow.val == fast.val
```

并不能说明两个指针相遇。

真正需要判断的是：

```python
slow == fast
```

即：

> 两个指针是否指向同一个 `ListNode` 节点。

------

## 3. 在移动指针之前判断 `slow == fast`

因为一开始：

```python
slow = head
fast = head
```

所以如果直接这样写：

```python
while fast and fast.next:
    if slow == fast:
        return True

    slow = slow.next
    fast = fast.next.next
```

第一次循环时：

```python
slow == fast == head
```

会直接错误返回 `True`。

因此通常要：

```python
slow = slow.next
fast = fast.next.next
```

之后再判断：

```python
if slow == fast:
```

------

# 面试中可以如何解释

一种比较清晰的表达方式是：

> 我首先可以使用一个 Hash Set 保存所有访问过的节点。如果遍历过程中再次遇到同一个节点，就说明存在环。这个方法时间复杂度是 `O(n)`，但需要 `O(n)` 的额外空间。
>
> 我们可以进一步优化空间。使用 Floyd's Cycle Detection Algorithm，维护一个 slow pointer 和一个 fast pointer。slow 每次走一步，fast 每次走两步。如果不存在环，fast 最终会到达 `None`；如果存在环，两者进入环后，由于 fast 每轮都会相对 slow 前进一个节点，因此最终一定会相遇。
>
> 所以可以在 `O(n)` 时间和 `O(1)` 额外空间内完成判断。
