# 复制带随机指针的链表（Copy Linked List with Random Pointer）

## 题目描述

给定一个长度为 `n` 的链表头节点 `head`。

与普通单链表不同，每个节点除了包含 `next` 指针之外，还包含一个额外的 `random` 指针：

- `next` 指向链表中的下一个节点；
- `random` 可以指向链表中的**任意节点**，也可以为 `None`。

请创建该链表的一个**深拷贝（Deep Copy）**。

新的链表必须包含恰好 `n` 个**全新的节点**。对于每一个复制后的节点：

- `val` 与原节点相同；
- `next` 指向原节点 `next` 所对应的**复制节点**；
- `random` 指向原节点 `random` 所对应的**复制节点**。

特别注意：

> 新链表中的任何指针都不能指向原链表中的节点。

返回复制后链表的头节点。

题目示例通常使用 `[val, random_index]` 表示一个节点，其中：

- `val`：节点的值；
- `random_index`：该节点的 `random` 指针指向的节点下标；
- 如果 `random = None`，则对应 `random_index = null`。

------

# 一、核心难点

这道题真正困难的地方不是复制 `val` 或 `next`，而是复制 `random`。

例如：

```
A ──next──> B ──next──> C

A.random ─────────────> C
B.random ─────────────> A
C.random ─────────────> C
```

复制之后应该得到：

```
A' ──next──> B' ──next──> C'

A'.random ────────────> C'
B'.random ────────────> A'
C'.random ────────────> C'
```

而不能出现：

```
A'.random ────────────> C   # 错误：指回原链表
```

因此问题的核心可以概括成一句话：

> **给定一个原节点，如何快速找到它对应的复制节点？**

最自然的方法就是建立：

```
原节点 -> 复制节点
```

这样的映射关系。

------

# 二、方法一：递归 + 哈希表

## 思路

使用一个哈希表保存：

```
original_node -> copied_node
```

例如：

```
A -> A'
B -> B'
C -> C'
```

递归处理一个节点时：

1. 如果节点是 `None`，返回 `None`；
2. 如果这个节点已经复制过，直接返回对应的复制节点；
3. 否则创建复制节点；
4. **立即存入哈希表**；
5. 递归复制 `next`；
6. 递归复制 `random`。

这里第 4 步非常重要。

因为 `random` 可能形成环：

```
A.random -> B
B.random -> A
```

如果不先记录 `A -> A'` 就递归，那么可能无限递归。

## 代码

```
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':
        # key: 原节点
        # value: 对应的复制节点
        oldToCopy = {}

        def clone(node):
            # 空节点直接返回 None
            if node is None:
                return None

            # 已经复制过，直接复用
            # 这一步同时避免 random 指针形成环时无限递归
            if node in oldToCopy:
                return oldToCopy[node]

            # 创建新节点
            copy = Node(node.val)

            # 必须先加入哈希表，再递归
            oldToCopy[node] = copy

            # 分别复制 next 和 random
            copy.next = clone(node.next)
            copy.random = clone(node.random)

            return copy

        return clone(head)
```

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

每个节点只会真正创建一次。

空间主要来自：

- 哈希表：`O(n)`
- 递归调用栈：最坏 `O(n)`

### 算法理解

虽然题目叫“链表”，但加入 `random` 后，从结构上看它更像一个**图（Graph）**：

```
node
 ├── next
 └── random
```

因此这个递归解法，本质上就是：

> **DFS + 哈希表复制图。**

如果以后遇到 **Clone Graph** 一类问题，会发现思路几乎完全一样。

------

# 三、方法二：哈希表 + 两次遍历

这是最直观、最适合面试时首先写出的解法。

## 思路

分成两个阶段。

### 第一遍：只创建节点

遍历原链表：

```
A -> B -> C
```

创建：

```
A'
B'
C'
```

并建立：

```
A -> A'
B -> B'
C -> C'
```

此时暂时不处理指针。

### 第二遍：连接指针

对于原节点：

```
cur
```

对应复制节点：

```
copy = oldToCopy[cur]
```

那么：

```
copy.next = oldToCopy[cur.next]
copy.random = oldToCopy[cur.random]
```

这样所有新节点的指针都只会指向**新节点**。

------

## 代码

```
"""
# Node 定义
class Node:
    def __init__(
        self,
        x: int,
        next: 'Node' = None,
        random: 'Node' = None
    ):
        self.val = int(x)
        self.next = next
        self.random = random
"""


class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':

        # 原节点 -> 复制节点
        #
        # 提前加入 None -> None，
        # 后面处理 next/random 时就不需要额外判断 None。
        oldToCopy = {None: None}

        # ---------- 第一遍 ----------
        # 创建所有复制节点
        cur = head

        while cur:
            copy = Node(cur.val)
            oldToCopy[cur] = copy

            cur = cur.next

        # ---------- 第二遍 ----------
        # 设置复制节点的 next 和 random
        cur = head

        while cur:
            copy = oldToCopy[cur]

            # 原 next 指向谁，
            # 新 next 就指向“谁的复制节点”
            copy.next = oldToCopy[cur.next]

            # random 同理
            copy.random = oldToCopy[cur.random]

            cur = cur.next

        # head 为 None 时，也能正确返回 None
        return oldToCopy[head]
```

------

## 为什么 `{None: None}` 很方便？

如果没有：

```
oldToCopy = {None: None}
```

那么可能需要写：

```
copy.next = oldToCopy[cur.next] if cur.next else None
copy.random = oldToCopy[cur.random] if cur.random else None
```

加入：

```
None -> None
```

之后可以直接：

```
copy.next = oldToCopy[cur.next]
copy.random = oldToCopy[cur.random]
```

这是一个很常见的小技巧。

------

## Python：字典中的对象为什么可以作为 key？

这里：

```
oldToCopy[cur] = copy
```

使用 `Node` 对象本身作为 Python `dict` 的 key。

在这道题默认的 `Node` 定义下，这是可以的。

我们需要的不是：

```
节点的 val -> 复制节点
```

而是：

```
节点对象本身 -> 复制节点
```

因为不同节点完全可能拥有相同的 `val`。

例如：

```
Node(7) -> Node(7)
```

它们虽然值一样，但仍然是两个不同节点。

所以绝不能写成：

```
oldToCopy[cur.val] = copy
```

------

## 复杂度

第一遍 `O(n)`，第二遍 `O(n)`：

```
O(n) + O(n) = O(n)
```

因此：

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

------

# 四、方法三：哈希表 + 一次遍历

## 思路

方法二需要两次遍历，因为第一次必须先创建所有节点。

但 Python 的 `collections.defaultdict` 可以做到：

> 当访问不存在的 key 时，自动创建默认 value。

因此可以写：

```
oldToCopy[cur]
oldToCopy[cur.next]
oldToCopy[cur.random]
```

即使对应复制节点还没有创建，`defaultdict` 也可以自动创建。

这样创建节点和连接指针就可以在同一次遍历中完成。

------

## `collections.defaultdict` 说明

普通字典：

```
d = {}

d["abc"]       # KeyError
```

而：

```
from collections import defaultdict

d = defaultdict(lambda: Node(0))
```

当第一次访问：

```
d["abc"]
```

时，会自动执行：

```
Node(0)
```

并大致产生：

```
d["abc"] = Node(0)
```

然后返回这个对象。

------

## 代码

```
import collections


class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':

        # 当访问一个不存在的原节点时，
        # 自动为它创建一个临时复制节点。
        oldToCopy = collections.defaultdict(lambda: Node(0))

        # None 必须特殊处理，否则访问 oldToCopy[None]
        # 会错误地创建一个 Node(0)。
        oldToCopy[None] = None

        cur = head

        while cur:
            # 当前节点的复制节点可能已经因为
            # 某个 random 指针而提前创建。
            copy = oldToCopy[cur]

            # 填入正确的值
            copy.val = cur.val

            # 如果 cur.next / cur.random 对应节点尚未创建，
            # defaultdict 会自动创建。
            copy.next = oldToCopy[cur.next]
            copy.random = oldToCopy[cur.random]

            cur = cur.next

        return oldToCopy[head]
```

## 复杂度

- 时间复杂度：**O(n)**
- 空间复杂度：**O(n)**

### 与两次遍历版本比较

虽然这个版本只遍历一次，但它的渐进复杂度并没有改善：

```
Two Pass: O(n)
One Pass: O(n)
```

因此面试中通常没有必要为了“一次遍历”强行选择这个版本。

**两次遍历版本逻辑更加直接，也更容易解释和验证。**

------

# 五、方法四：节点穿插法 —— O(1) 额外空间

这是这道题最经典的空间优化解法。

## 核心思想

哈希表存在的目的，是为了快速实现：

```
A -> A'
```

如果不用哈希表，我们能不能通过链表结构本身建立这种对应关系？

可以。

把复制节点直接放在原节点后面：

```
原来：

A -> B -> C

变成：

A -> A' -> B -> B' -> C -> C'
```

于是有一个非常重要的关系：

```
A.next == A'
B.next == B'
C.next == C'
```

也就是说：

> **每个原节点的复制节点，就是 `original.next`。**

这实际上利用链表本身代替了哈希表。

------

## Step 1：穿插复制节点

原链表：

```
A -> B -> C
```

处理后：

```
A -> A' -> B -> B' -> C -> C'
```

代码核心：

```
copy = Node(cur.val)

copy.next = cur.next
cur.next = copy
```

------

## Step 2：设置 random

假设：

```
A.random = C
```

由于复制节点紧跟在原节点后：

```
C.next = C'
```

因此：

```
A'.random = C'
```

可以转换为：

```
A.next.random = A.random.next
```

这就是整个算法最关键的一行：

```
l1.next.random = l1.random.next
```

------

## Step 3：拆分链表

目前：

```
A -> A' -> B -> B' -> C -> C'
```

需要重新拆成：

```
A -> B -> C

A' -> B' -> C'
```

拆分过程中要同时：

1. 恢复原链表；
2. 建立复制链表。

------

## 完整代码

```
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':

        if head is None:
            return None

        # ==================================================
        # Step 1：在每个原节点后插入它的复制节点
        #
        # A -> B -> C
        #
        # 变成：
        #
        # A -> A' -> B -> B' -> C -> C'
        # ==================================================

        l1 = head

        while l1 is not None:
            l2 = Node(l1.val)

            # 新节点先连接原来的下一个节点
            l2.next = l1.next

            # 原节点再连接复制节点
            l1.next = l2

            # 跳到下一个原节点
            l1 = l2.next

        # 复制链表的头一定是 head 后面的节点
        newHead = head.next

        # ==================================================
        # Step 2：设置复制节点的 random
        # ==================================================

        l1 = head

        while l1 is not None:

            if l1.random is not None:
                # l1.next 是 l1 的复制节点
                #
                # l1.random.next 是
                # l1.random 对应的复制节点
                l1.next.random = l1.random.next

            # 跳过复制节点，到下一个原节点
            l1 = l1.next.next

        # ==================================================
        # Step 3：拆开两个链表
        #
        # A -> A' -> B -> B'
        #
        # 恢复：
        # A -> B
        #
        # 得到：
        # A' -> B'
        # ==================================================

        l1 = head

        while l1 is not None:
            l2 = l1.next

            # 恢复原链表
            l1.next = l2.next

            # 连接复制链表
            if l2.next is not None:
                l2.next = l2.next.next

            # 前往下一个原节点
            l1 = l1.next

        return newHead
```

------

## 复杂度

需要三次线性遍历：

```
O(n) + O(n) + O(n) = O(n)
```

因此：

- 时间复杂度：**O(n)**
- 额外空间复杂度：**O(1)**

这里说的是 **extra space / auxiliary space**。

复制出来的 `n` 个节点本身属于题目要求的输出，因此通常**不计算在额外空间复杂度中**。

如果把输出也计算进去，总空间当然仍然需要 `O(n)`。

------

# 六、方法五：利用 random 指针暂存复制节点

这个方法同样可以做到：

- 时间：`O(n)`
- 额外空间：`O(1)`

不过相比节点穿插法，它更加巧妙，也更难理解。

一般面试中掌握上一种 **Interleaving / Weaving** 方法就已经足够。

## 核心思想

上一种方法把：

```
original -> copy
```

关系临时存储在：

```
original.next
```

这个方法则把它临时存储在：

```
original.random
```

对于原节点 `A`：

```
A.random = R
```

创建 `A'` 后：

```
A'.next = R
A.random = A'
```

于是：

```
A.random
```

现在可以直接找到：

```
A'
```

而原来的 `random` 暂时被保存在：

```
A'.next
```

中。

------

## 代码

```
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':

        if head is None:
            return None

        # ==================================================
        # Step 1：
        # 创建复制节点，并暂时放入原节点的 random 中
        # ==================================================

        l1 = head

        while l1:
            l2 = Node(l1.val)

            # 暂存原来的 random
            l2.next = l1.random

            # 原节点的 random 暂时指向复制节点
            l1.random = l2

            l1 = l1.next

        # 此时 head.random 就是 head 的复制节点
        newHead = head.random

        # ==================================================
        # Step 2：设置复制节点的 random
        # ==================================================

        l1 = head

        while l1:
            # l2 是 l1 的复制节点
            l2 = l1.random

            # l2.next 当前保存的是：
            # l1 原来的 random 节点
            #
            # 而 original_random.random
            # 现在恰好是 original_random 的复制节点。
            l2.random = l2.next.random if l2.next else None

            l1 = l1.next

        # ==================================================
        # Step 3：
        # 恢复原 random，并连接复制链表的 next
        # ==================================================

        l1 = head

        while l1 is not None:
            l2 = l1.random

            # 恢复原节点的 random
            l1.random = l2.next

            # 下一个原节点的 random
            # 就是“下一个复制节点”
            l2.next = l1.next.random if l1.next else None

            l1 = l1.next

        return newHead
```

## 复杂度

- 时间复杂度：**O(n)**
- 额外空间复杂度：**O(1)**
- 如果计算输出节点：`O(n)`

### 面试建议

这个解法虽然漂亮，但它临时改变了 `random` 指针的语义：

```
原本：random -> 随机节点
临时：random -> 自己的复制节点
```

同时又使用复制节点的 `next` 保存原来的 `random`。

因此代码可读性明显低于经典的节点穿插法。

除非面试官继续追问其他 `O(1)` 空间方法，否则通常没有必要主动写这个版本。

------

# 七、五种方法对比

| 方法              | 时间复杂度 | 额外空间 | 特点                     |
| ----------------- | ---------- | -------- | ------------------------ |
| 递归 + Hash Map   | `O(n)`     | `O(n)`   | 本质类似 DFS 复制图      |
| Hash Map 两次遍历 | `O(n)`     | `O(n)`   | **最直观，最推荐先掌握** |
| Hash Map 一次遍历 | `O(n)`     | `O(n)`   | 利用 `defaultdict`       |
| 节点穿插法        | `O(n)`     | `O(1)`   | **经典空间优化解法**     |
| 利用 random 暂存  | `O(n)`     | `O(1)`   | 巧妙，但可读性较差       |

对于代码面试，建议重点掌握两个：

```
1. Hash Map 两次遍历
2. 节点穿插 O(1) 空间
```

通常这两种已经覆盖：

```
基础解法 → O(n) space

        ↓ 优化

进阶解法 → O(1) extra space
```

------

# 八、常见错误

## 1. 为同一个原节点创建多个复制节点

错误：

```
copy.random = Node(original.random.val)
```

假设：

```
A.random -> C
B.random -> C
```

上面的代码会分别创建两个不同的 `C'`：

```
A'.random -> C1'
B'.random -> C2'
```

这显然不是正确的深拷贝。

正确方法：

```
copy.random = oldToCopy[original.random]
```

保证：

```
A'.random ─┐
           ├──> C'
B'.random ─┘
```

------

## 2. 使用节点值作为哈希表 key

错误：

```
oldToCopy[cur.val] = copy
```

因为：

```
A.val = 7
B.val = 7
```

完全合法。

此时两个不同节点都会使用：

```
oldToCopy[7]
```

导致映射冲突。

应该使用节点对象：

```
oldToCopy[cur] = copy
```

------

## 3. 忘记处理 `None`

例如：

```
copy.random = oldToCopy[cur.random]
```

如果：

```
cur.random is None
```

而字典中没有 `None`，就可能产生 `KeyError`。

一个很方便的处理方式：

```
oldToCopy = {None: None}
```

------

## 4. O(1) 解法中错误地处理 random

节点穿插之后：

```
A -> A' -> B -> B' -> C -> C'
```

如果：

```
A.random = C
```

不能写：

```
A.next.random = A.random
```

因为这会得到：

```
A'.random -> C
```

新链表又指回了原链表。

正确的是：

```
A.next.random = A.random.next
```

因为：

```
A.random      = C
A.random.next = C'
```

所以：

```
A'.random = C'
```

------

## 5. 拆分时没有恢复原链表

节点穿插法中，最终必须把：

```
A -> A' -> B -> B' -> C -> C'
```

恢复、拆分为：

```
A -> B -> C
```

和：

```
A' -> B' -> C'
```

因此拆分阶段必须同时维护两个链表。

------

# 九、面试中最重要的推导过程

这道题不建议单纯背代码，更值得记住的是下面的推导。

### 第一步：发现问题

普通链表复制：

```
A -> B -> C
```

只需要复制 `next`。

但现在：

```
A.random -> C
```

复制 `A` 时，需要找到：

```
C 对应的 C'
```

因此我们需要解决：

```
original node -> copied node
```

------

### 第二步：最自然的解决方案是 Hash Map

直接建立：

```
oldToCopy[original] = copy
```

于是：

```
copy.next = oldToCopy[original.next]
copy.random = oldToCopy[original.random]
```

得到 `O(n)` 时间、`O(n)` 空间。

------

### 第三步：思考能否去掉 Hash Map

Hash Map 唯一的重要作用其实就是：

```
给我 original，我能找到 copy。
```

那么能不能不用 Hash Map，也让：

```
original -> copy
```

成为 `O(1)` 操作？

可以，把复制节点直接放在原节点旁边：

```
A -> A'
B -> B'
C -> C'
```

于是：

```
copy_of_A = A.next
copy_of_B = B.next
copy_of_C = C.next
```

Hash Map 就被链表自身的结构替代了。

进一步：

```
copy(original.random)
```

自然就变成：

```
original.random.next
```

因此得到整道题最关键的关系：

```
original.next.random = original.random.next
```

这也是 `O(1)` 空间解法最值得真正理解的地方。

------

# 十、最终推荐模板

如果面试没有要求 `O(1)` 额外空间，优先写两次遍历 Hash Map：

```
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':
        oldToCopy = {None: None}

        # Pass 1: 创建所有节点
        cur = head
        while cur:
            oldToCopy[cur] = Node(cur.val)
            cur = cur.next

        # Pass 2: 连接 next 和 random
        cur = head
        while cur:
            copy = oldToCopy[cur]

            copy.next = oldToCopy[cur.next]
            copy.random = oldToCopy[cur.random]

            cur = cur.next

        return oldToCopy[head]
```

如果面试官进一步要求：

> **Can you solve it without extra space?**

再给出节点穿插法：

```
A -> B -> C

↓

A -> A' -> B -> B' -> C -> C'

↓

设置复制节点 random

↓

A -> B -> C
A' -> B' -> C'
```

这样从 `O(n)` 额外空间自然优化到 `O(1)` 额外空间，比直接背诵空间优化代码更容易在面试中完整地推导出来。
