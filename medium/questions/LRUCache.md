# LRU Cache（最近最少使用缓存）

## 题目

实现一个 **LRU（Least Recently Used，最近最少使用）缓存**类 `LRUCache`。

需要支持以下操作：

- `LRUCache(int capacity)`：初始化一个容量为 `capacity` 的 LRU 缓存。
- `int get(int key)`：
  - 如果 `key` 存在，返回对应的 `value`；
  - 否则返回 `-1`。
- `void put(int key, int value)`：
  - 如果 `key` 已经存在，更新它对应的 `value`；
  - 如果 `key` 不存在，则插入新的 `(key, value)`；
  - 如果插入后缓存容量超过 `capacity`，删除**最近最少使用**的 key。

只要某个 key 被执行过 `get` 或 `put`，就认为它刚刚被使用过，也就是成为 **Most Recently Used（MRU，最近使用）** 的元素。

要求：

> `get()` 和 `put()` 的平均时间复杂度都必须达到 **O(1)**。

------

# 一、理解 LRU

LRU 的核心思想是：

> **当缓存满了以后，优先淘汰最长时间没有被访问过的数据。**

例如容量为 3：

```
LRU                         MRU
 ↓                           ↓
[1] <-> [2] <-> [3]
```

这里：

- `1` 最久没有被访问，因此是 LRU；
- `3` 最近刚被访问，因此是 MRU。

如果此时执行：

```
get(1)
```

那么 `1` 刚刚被访问过，因此应该移动到 MRU：

```
LRU                         MRU
 ↓                           ↓
[2] <-> [3] <-> [1]
```

如果接下来：

```
put(4, value)
```

由于缓存已经满了，需要删除最久没有使用的 `2`：

```
LRU                         MRU
 ↓                           ↓
[3] <-> [1] <-> [4]
```

因此，这道题真正需要解决两个问题：

1. **如何快速找到一个 key？**
2. **如何快速维护 key 的使用顺序？**

这也是为什么最经典的解法是：

```
Hash Map + Doubly Linked List
哈希表   + 双向链表
```

------

# 二、方法一：暴力解法——List

## 思路

最简单的方法是使用一个 Python `list` 保存所有 `(key, value)`。

规定：

```
左边 = Least Recently Used
右边 = Most Recently Used
```

例如：

```
[[1, 10], [2, 20], [3, 30]]
   ↑                   ↑
  LRU                 MRU
```

每当访问某个 key：

1. 找到它；
2. 从当前位置删除；
3. 添加到 list 最后。

这样就表示它刚刚被使用过。

当插入新元素并且缓存已经满了：

```
self.cache.pop(0)
```

直接删除第一个元素，也就是 LRU。

------

## 算法

### `get(key)`

遍历整个 list：

- 如果找到 key：
  1. 删除当前位置的元素；
  2. 添加到 list 尾部；
  3. 返回 value。
- 如果找不到，返回 `-1`。

### `put(key, value)`

首先遍历 list 查找 key。

如果存在：

1. 删除旧元素；
2. 更新 value；
3. 添加到 list 尾部。

如果不存在：

1. 如果缓存已经满了，删除第一个元素；
2. 把新的 `[key, value]` 添加到尾部。

------

## 代码

```
class LRUCache:

    def __init__(self, capacity: int):
        # cache[0] 是 LRU
        # cache[-1] 是 MRU
        self.cache = []
        self.capacity = capacity

    def get(self, key: int) -> int:
        # 需要遍历整个 list 查找 key
        for i in range(len(self.cache)):
            if self.cache[i][0] == key:

                # 找到元素后将其从当前位置删除
                item = self.cache.pop(i)

                # 放到最后，表示它刚刚被使用
                self.cache.append(item)

                return item[1]

        return -1

    def put(self, key: int, value: int) -> None:
        # 首先检查 key 是否已经存在
        for i in range(len(self.cache)):
            if self.cache[i][0] == key:

                # 删除旧位置
                item = self.cache.pop(i)

                # 更新 value
                item[1] = value

                # 移动到 MRU 位置
                self.cache.append(item)

                return

        # key 不存在，并且缓存已经满了
        if len(self.cache) == self.capacity:
            # 删除 LRU
            self.cache.pop(0)

        # 新元素一定是 MRU
        self.cache.append([key, value])
```

------

## 复杂度分析

设缓存最多包含 `n` 个元素。

### 时间复杂度

`get()`：

```
O(n)
```

因为需要遍历 list 查找 key。

`put()`：

```
O(n)
```

因为同样可能需要遍历整个 list。

此外，Python：

```
list.pop(i)
```

如果 `i` 不是最后一个位置，后面的元素需要移动，所以本身也可能是 `O(n)`。

### 空间复杂度

```
O(n)
```

------

## 为什么这个方法不满足题目要求？

题目明确要求：

```
get() -> O(1)
put() -> O(1)
```

而 list 无法在 `O(1)` 时间内根据 key 找到元素。

因此我们需要一个支持：

```
key -> element
```

快速查找的数据结构。

自然想到：

```
dict
```

但 `dict` 又不方便维护 LRU 顺序。

因此需要：

> **Hash Map + Doubly Linked List**

------

# 三、最重要的解法：Hash Map + Doubly Linked List

这是这道题面试中最应该掌握的解法。

核心数据结构：

```
Hash Map
    +
Doubly Linked List
```

它们分别解决不同的问题。

| 数据结构           | 作用                    | 复杂度       |
| ------------------ | ----------------------- | ------------ |
| Hash Map           | 根据 key 快速找到 Node  | O(1) average |
| Doubly Linked List | 删除 / 插入 / 移动 Node | O(1)         |

------

# 四、为什么需要 Hash Map？

Python 字典：

```
cache = {}
```

可以做到平均：

```
查找：O(1)
插入：O(1)
删除：O(1)
```

因此可以建立：

```
key -> Node
```

例如：

```
cache = {
    1: Node(1, 100),
    2: Node(2, 200),
    3: Node(3, 300)
}
```

这样：

```
cache[2]
```

就能平均 `O(1)` 找到 key `2` 对应的链表节点。

------

# 五、为什么需要双向链表？

找到 Node 以后，我们还需要把它移动到 MRU。

假设：

```
LRU                         MRU
 ↓                           ↓
[1] <-> [2] <-> [3] <-> [4]
```

执行：

```
get(2)
```

需要变成：

```
LRU                         MRU
 ↓                           ↓
[1] <-> [3] <-> [4] <-> [2]
```

也就是：

1. 从当前位置删除 `2`；
2. 把 `2` 插入链表尾部。

双向链表可以在已知 Node 的情况下，在 `O(1)` 时间删除节点。

------

# 六、为什么单向链表不够方便？

删除：

```
A -> B -> C
```

如果想删除 `B`，必须修改：

```
A.next = C
```

所以需要知道 `B` 前面的 `A`。

单向链表只有：

```
B.next
```

却没有：

```
B.prev
```

因此不方便直接删除任意 Node。

双向链表：

```
A <-> B <-> C
```

Node 同时保存：

```
node.prev
node.next
```

所以可以直接：

```
node.prev.next = node.next
node.next.prev = node.prev
```

完成删除。

时间复杂度：

```
O(1)
```

------

# 七、使用 Dummy Nodes 简化代码

这里使用两个虚拟节点：

```
left                                    right
 ↓                                        ↓
[DUMMY] <-> [LRU] <-> ... <-> [MRU] <-> [DUMMY]
```

规定：

```
left.next  = LRU
right.prev = MRU
```

因此：

### 找 LRU

```
self.left.next
```

### 找 MRU

```
self.right.prev
```

### 插入新的 MRU

永远插入：

```
right 前面
```

Dummy Node 的好处是避免处理大量边界情况。

例如，如果没有 dummy node，就需要频繁判断：

```
if head is None:
    ...

if tail is None:
    ...

if node == head:
    ...

if node == tail:
    ...
```

有了 dummy node 后，删除和插入逻辑基本完全统一。

------

# 八、Node 节点

```
class Node:
    def __init__(self, key, val):
        self.key = key
        self.val = val

        self.prev = None
        self.next = None
```

每个节点保存：

```
key
value
prev
next
```

这里一个非常重要的细节是：

> **Node 必须保存 key，而不能只保存 value。**

原因稍后会解释。

------

# 九、核心操作 1：删除 Node

假设：

```
A <-> B <-> C
```

删除 `B`：

```
prev = node.prev
nxt = node.next

prev.next = nxt
nxt.prev = prev
```

结果：

```
A <-> C
```

代码：

```
def remove(self, node):
    prev = node.prev
    nxt = node.next

    prev.next = nxt
    nxt.prev = prev
```

时间复杂度：

```
O(1)
```

------

# 十、核心操作 2：插入 MRU

我们规定：

```
right.prev = MRU
```

所以所有刚刚被使用的节点都插入 `right` 前面。

原来：

```
... <-> A <-> right
```

插入 `node`：

```
... <-> A <-> node <-> right
```

代码：

```
def insert(self, node):
    prev = self.right.prev
    nxt = self.right

    prev.next = node
    node.prev = prev

    node.next = nxt
    nxt.prev = node
```

时间复杂度：

```
O(1)
```

------

# 十一、`get()` 的实现

如果：

```
key not in self.cache
```

直接：

```
return -1
```

如果存在，则通过 Hash Map：

```
node = self.cache[key]
```

平均 `O(1)` 找到节点。

然后：

```
self.remove(node)
self.insert(node)
```

把它移动到 MRU。

最后：

```
return node.val
```

------

# 十二、`put()` 的实现

## 情况 1：key 已经存在

例如：

```
LRU                    MRU
 ↓                      ↓
[1] <-> [2] <-> [3]
```

执行：

```
put(2, 200)
```

因为 `2` 被访问了，所以需要：

1. 更新 value；
2. 将 `2` 移动到 MRU。

变成：

```
LRU                    MRU
 ↓                      ↓
[1] <-> [3] <-> [2]
```

------

## 情况 2：key 不存在

创建新的 Node：

```
node = Node(key, value)
```

然后：

```
self.cache[key] = node
self.insert(node)
```

因为刚插入的元素一定是 MRU。

------

## 情况 3：超过容量

如果：

```
len(self.cache) > self.cap
```

那么：

```
lru = self.left.next
```

找到 LRU。

然后同时从两个数据结构删除：

```
self.remove(lru)
del self.cache[lru.key]
```

这里也解释了：

> 为什么 Node 必须保存 key？

因为当我们通过链表找到 LRU：

```
lru = self.left.next
```

我们还必须从 Hash Map 删除：

```
del self.cache[lru.key]
```

如果 Node 没有保存 key，就无法 `O(1)` 知道应该删除 Hash Map 中的哪个 entry。

------

# 十三、完整代码

下面给出一个稍微整理后的版本。与原答案思路相同，但更新已有 key 时直接复用 Node，会更加直观。

```
class Node:
    def __init__(self, key: int, val: int):
        self.key = key
        self.val = val

        # 双向链表指针
        self.prev = None
        self.next = None


class LRUCache:

    def __init__(self, capacity: int):
        self.capacity = capacity

        # Hash Map:
        # key -> 对应的链表 Node
        self.cache = {}

        # Dummy Nodes
        #
        # left.next  永远指向 LRU
        # right.prev 永远指向 MRU
        self.left = Node(0, 0)
        self.right = Node(0, 0)

        # 初始化：
        # left <-> right
        self.left.next = self.right
        self.right.prev = self.left

    def _remove(self, node: Node) -> None:
        """
        从双向链表中删除 node。
        已知 node 的情况下时间复杂度 O(1)。
        """
        prev_node = node.prev
        next_node = node.next

        prev_node.next = next_node
        next_node.prev = prev_node

    def _insert_mru(self, node: Node) -> None:
        """
        将 node 插入 right 前面，
        使其成为 Most Recently Used。
        """
        prev_node = self.right.prev

        # prev_node <-> node
        prev_node.next = node
        node.prev = prev_node

        # node <-> right
        node.next = self.right
        self.right.prev = node

    def get(self, key: int) -> int:
        # Hash Map 平均 O(1) 查找
        if key not in self.cache:
            return -1

        node = self.cache[key]

        # get 也算一次访问，
        # 所以需要把它移动到 MRU
        self._remove(node)
        self._insert_mru(node)

        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            # key 已存在：
            # 更新 value，并移动到 MRU
            node = self.cache[key]
            node.val = value

            self._remove(node)
            self._insert_mru(node)

        else:
            # key 不存在：
            # 创建新的节点
            node = Node(key, value)

            # 同时加入 Hash Map 和 Linked List
            self.cache[key] = node
            self._insert_mru(node)

            # 如果超过容量，删除 LRU
            if len(self.cache) > self.capacity:
                # left.next 永远是 LRU
                lru = self.left.next

                # 从链表删除
                self._remove(lru)

                # 从 Hash Map 删除
                del self.cache[lru.key]
```

------

# 十四、完整执行示例

假设：

```
cache = LRUCache(2)
```

初始：

```
left <-> right
```

执行：

```
cache.put(1, 1)
```

得到：

```
left <-> [1] <-> right
           ↑
          MRU
```

执行：

```
cache.put(2, 2)
```

得到：

```
left <-> [1] <-> [2] <-> right
           ↑       ↑
          LRU     MRU
```

执行：

```
cache.get(1)
```

返回：

```
1
```

由于 `1` 刚刚被访问：

```
left <-> [2] <-> [1] <-> right
           ↑       ↑
          LRU     MRU
```

执行：

```
cache.put(3, 3)
```

首先插入：

```
left <-> [2] <-> [1] <-> [3] <-> right
```

超过容量，因此删除：

```
left.next
```

也就是 `2`：

```
left <-> [1] <-> [3] <-> right
           ↑       ↑
          LRU     MRU
```

所以：

```
cache.get(2)
```

返回：

```
-1
```

------

# 十五、复杂度分析

设容量为 `n`。

## `get()`

Hash Map 查找：

```
O(1) average
```

双向链表删除：

```
O(1)
```

双向链表插入：

```
O(1)
```

因此：

```
get(): O(1) average
```

## `put()`

Hash Map 查找 / 插入 / 删除：

```
O(1) average
```

双向链表插入 / 删除：

```
O(1)
```

因此：

```
put(): O(1) average
```

## 空间复杂度

Hash Map 和 Doubly Linked List 最多保存 `n` 个真实节点：

```
O(n)
```

------

# 十六、Python 内置实现：`OrderedDict`

Python 标准库提供：

```
collections.OrderedDict
```

它是一种**维护 key 顺序的字典**。

首先需要：

```
from collections import OrderedDict
```

------

## `OrderedDict.move_to_end()`

```
cache.move_to_end(key)
```

可以把指定 key 移动到最后。

例如：

```
from collections import OrderedDict

d = OrderedDict()

d[1] = "A"
d[2] = "B"
d[3] = "C"

d.move_to_end(1)
```

顺序从：

```
1, 2, 3
```

变成：

```
2, 3, 1
```

所以非常适合表示：

```
LRU -> MRU
```

------

## `OrderedDict.popitem()`

```
cache.popitem(last=False)
```

表示删除**最前面的元素**。

而我们的最前面正好是 LRU。

如果：

```
cache.popitem(last=True)
```

则删除最后面的元素。

`last=True` 是默认值。

------

# 十七、`OrderedDict` 完整实现

```
from collections import OrderedDict


class LRUCache:

    def __init__(self, capacity: int):
        # OrderedDict 会维护 key 的顺序
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1

        # get 也属于一次访问，
        # 将 key 移动到最后，使其成为 MRU
        self.cache.move_to_end(key)

        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            # 已存在的 key 被再次访问，
            # 移动到 MRU
            self.cache.move_to_end(key)

        # 新增或者更新 value
        self.cache[key] = value

        # 超过容量，删除最前面的 LRU
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

------

# 十八、`OrderedDict` 解法复杂度

平均时间复杂度：

```
get(): O(1)
put(): O(1)
```

空间复杂度：

```
O(n)
```

从实际 Python 开发角度，`OrderedDict` 的实现非常简洁。

但如果这是**算法面试**，面试官通常更希望看到：

```
Hash Map + Doubly Linked List
```

因为这才真正考察了你是否理解 LRU 如何做到 `O(1)`。

------

# 十九、常见错误

## 1. 忘记 `get()` 也要更新使用顺序

这是最常见的错误之一。

错误理解：

```
只有 put 才算使用
```

实际上题目明确规定：

```
get 和 put 都算一次访问
```

因此：

```
get(key)
```

成功以后必须：

```
self._remove(node)
self._insert_mru(node)
```

否则 LRU 顺序就错了。

------

## 2. 双向链表指针更新不完整

删除：

```
A <-> B <-> C
```

必须同时修改：

```
A.next = C
C.prev = A
```

也就是：

```
node.prev.next = node.next
node.next.prev = node.prev
```

少修改任何一边都会破坏链表。

------

## 3. 插入节点时漏掉某个指针

插入：

```
A <-> node <-> right
```

实际上涉及四个连接：

```
A.next = node
node.prev = A

node.next = right
right.prev = node
```

这也是双向链表题目最容易写错的地方之一。

------

## 4. Node 中没有保存 key

错误设计：

```
class Node:
    def __init__(self, value):
        self.value = value
```

当缓存满了以后，我们通过：

```
lru = self.left.next
```

找到了需要删除的 Node。

但接下来还必须：

```
del self.cache[lru.key]
```

如果 Node 不保存 key，就无法在 `O(1)` 时间知道 Hash Map 应该删除哪一个 entry。

所以：

```
class Node:
    def __init__(self, key, value):
        self.key = key
        self.value = value
```

是非常重要的设计。

------

## 5. 只从链表删除，却忘记从 Hash Map 删除

淘汰 LRU 时需要同步维护两个数据结构：

```
self._remove(lru)
del self.cache[lru.key]
```

因为：

```
Hash Map + Linked List
```

共同表示同一份缓存状态。

只删除其中一个会造成数据不一致。

------

# 二十、面试中最值得记住的思维过程

这道题不建议单纯背代码，更重要的是记住下面的推导：

```
题目要求 get(key) = O(1)
            ↓
需要快速根据 key 找数据
            ↓
Hash Map


题目要求维护 LRU 顺序
            ↓
访问一个节点后需要把它移动到 MRU
缓存满以后需要删除 LRU
            ↓
需要 O(1) 删除 / 插入节点
            ↓
Doubly Linked List


最终：
Hash Map + Doubly Linked List
```

然后规定：

```
left                                    right
 ↓                                        ↓
Dummy <-> LRU <-> ... <-> MRU <-> Dummy
```

记住两个关键位置：

```
self.left.next
# LRU

self.right.prev
# MRU
```

再把所有链表操作压缩成两个 helper：

```
remove(node)
insert_mru(node)
```

那么：

```
get(key)
```

本质就是：

```
找到 node
→ remove(node)
→ insert_mru(node)
→ return value
```

而：

```
put(key, value)
```

本质就是：

```
如果存在：
    更新 node
    → remove
    → insert_mru

如果不存在：
    创建 node
    → insert_mru

如果超过容量：
    left.next 就是 LRU
    → 从链表删除
    → 从 Hash Map 删除
```

掌握这个结构以后，LRU Cache 的代码基本就可以从逻辑直接推导出来，而不需要逐行背诵。