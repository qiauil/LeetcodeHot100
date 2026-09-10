# 两个链表的相交节点（Intersection of Two Linked Lists）

给定两个单链表的头节点 `headA` 和 `headB`，请返回两个链表开始相交的节点。如果两个链表完全不相交，则返回 `None`。

这里的**相交**指的是两个链表从某个节点开始，共享完全相同的节点对象，而不仅仅是节点的值相同。

例如：

```
A: 4 → 1
        \
         8 → 4 → 5
        /
B: 5 → 6 → 1
```

两个链表的相交节点是值为 `8` 的那个**同一个节点对象**。

需要注意：函数执行结束后，两个链表必须保持原来的结构，不能修改链表。

------

## 自定义判题说明

LeetCode 的判题器实际上会使用以下信息构造两个链表：

- `intersectVal`：相交节点的值。如果没有相交节点，则为 `0`
- `listA`：第一个链表
- `listB`：第二个链表
- `skipA`：从 `listA` 头节点开始跳过多少个节点后到达相交节点
- `skipB`：从 `listB` 头节点开始跳过多少个节点后到达相交节点

这些参数**不会直接传入你的函数**。

判题器会根据这些数据构造真正共享节点的链表，然后只把：

```
headA
headB
```

传给 `getIntersectionNode()`。

因此，我们真正需要判断的是：

```
nodeA == nodeB
```

而不是：

```
nodeA.val == nodeB.val
```

------

# 方法一：暴力枚举

## 思路

最直接的方法是：

对于链表 A 中的每一个节点，都遍历一次链表 B，检查链表 B 中是否存在与它完全相同的节点。

如果：

```
nodeA == nodeB
```

那么说明它们实际上是同一个节点对象，也就是两个链表的交点。

这种方法比较容易理解，但时间复杂度较高。

------

## 算法步骤

1. 从 `headA` 开始遍历链表 A。
2. 对于 A 中的每一个节点：
   - 从 `headB` 开始完整遍历一次链表 B。
3. 如果发现两个节点引用相同，则返回该节点。
4. 如果全部检查结束仍未找到，则返回 `None`。

------

## Python 实现

```
# 单链表节点定义
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None


class Solution:
    def getIntersectionNode(
        self,
        headA: ListNode,
        headB: ListNode
    ) -> Optional[ListNode]:

        # 遍历链表 A 中的每一个节点
        while headA:
            cur = headB

            # 对当前的 A 节点，遍历整个链表 B
            while cur:
                # 注意：这里比较的是节点本身，而不是节点值
                if headA == cur:
                    return headA

                cur = cur.next

            headA = headA.next

        # 没有交点
        return None
```

------

## 复杂度分析

假设：

- 链表 A 长度为 `m`
- 链表 B 长度为 `n`

时间复杂度：

```
O(m × n)
```

因为对于 A 中的每一个节点，都可能需要完整扫描一次 B。

空间复杂度：

```
O(1)
```

没有使用额外的数据结构。

------

# 方法二：哈希集合

## 思路

暴力解法的问题在于，我们不断重复遍历链表 B。

一种自然的优化方式是使用 `set`。

先将链表 A 中所有节点存入集合：

```
node_set
```

然后遍历链表 B。

对于 B 中的每个节点，只需要判断：

```
if cur in node_set:
```

即可知道这个节点是否也出现在 A 中。

由于 Python `set` 的查找操作平均是 `O(1)`，因此整体时间复杂度可以降低到 `O(m + n)`。

------

## 算法步骤

1. 创建一个集合 `node_set`。
2. 遍历链表 A，将所有节点加入集合。
3. 遍历链表 B。
4. 对于 B 中的每个节点：
   - 如果节点存在于集合中，返回该节点。
5. 如果遍历结束仍没有找到，则返回 `None`。

------

## Python 实现

```
# 单链表节点定义
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None


class Solution:
    def getIntersectionNode(
        self,
        headA: ListNode,
        headB: ListNode
    ) -> Optional[ListNode]:

        # 用于保存链表 A 中出现过的节点
        node_set = set()

        cur = headA

        # 将 A 中所有节点存入 set
        while cur:
            node_set.add(cur)
            cur = cur.next

        # 遍历 B
        cur = headB

        while cur:
            # 如果当前节点也存在于 A 中，
            # 那么它就是两个链表共享的节点
            if cur in node_set:
                return cur

            cur = cur.next

        return None
```

------

## Python `set` 补充

`set` 是 Python 中的哈希集合。

常见操作：

```
s = set()

s.add(x)      # 添加元素

x in s        # 判断元素是否存在
```

平均情况下：

```
s.add(x)
```

和：

```
x in s
```

时间复杂度都是：

```
O(1)
```

因此 `set` 非常适合处理：

> “这个对象以前是否出现过？”

这一类问题。

这里 `ListNode` 对象默认可以按照对象身份进行哈希，因此可以直接：

```
node_set.add(cur)
```

------

## 复杂度分析

时间复杂度：

```
O(m + n)
```

因为：

- 遍历 A 一次：`O(m)`
- 遍历 B 一次：`O(n)`

空间复杂度：

```
O(m)
```

因为最多需要存储链表 A 的全部节点。

------

# 方法三：双指针 + 长度对齐

## 核心观察

假设两个链表相交：

```
A:

a1 → a2 → a3
          \
           c1 → c2 → c3

B:

b1 → b2 → b3 → b4
               /
             c1 → c2 → c3
```

一旦两个链表进入公共部分：

```
c1 → c2 → c3
```

之后的所有节点都完全相同。

因此，从相交节点到链表尾部的距离一定相同。

真正的问题在于：

> 两个链表在相交之前的长度可能不同。

所以我们可以先计算两个链表的长度。

假设：

```
A 长度 = 8
B 长度 = 5
```

那么让 A 的指针先向前走：

```
8 - 5 = 3
```

步。

这样两个指针距离链表末尾的距离就相同了。

之后同时向前移动，第一次相遇的位置就是交点。

------

## 算法步骤

1. 分别计算两个链表长度 `m` 和 `n`。
2. 找出较长的链表。
3. 让较长链表的指针先前进 `|m - n|` 步。
4. 此时两个指针距离链表尾部的距离相同。
5. 两个指针同时向前移动。
6. 当：

```
l1 == l2
```

时返回该节点。

如果不存在交点，最终两个指针都会变成：

```
None
```

此时同样有：

```
l1 == l2
```

循环结束并返回 `None`。

------

## Python 实现

```
# 单链表节点定义
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None


class Solution:
    def getIntersectionNode(
        self,
        headA: ListNode,
        headB: ListNode
    ) -> Optional[ListNode]:

        # 辅助函数：计算链表长度
        def getLength(head):
            length = 0
            cur = head

            while cur:
                length += 1
                cur = cur.next

            return length

        # 分别计算两个链表的长度
        m = getLength(headA)
        n = getLength(headB)

        l1 = headA
        l2 = headB

        # 保证 l1 始终指向较长的链表
        if m < n:
            m, n = n, m
            l1, l2 = headB, headA

        # 让较长链表先走长度差
        while m - n:
            m -= 1
            l1 = l1.next

        # 此时两个指针距离链表末尾相同
        # 同时向前移动
        while l1 != l2:
            l1 = l1.next
            l2 = l2.next

        # 如果有交点，l1 就是交点
        # 如果没有交点，l1 == l2 == None
        return l1
```

------

## 为什么长度对齐后一定能找到交点？

假设：

```
A 独有部分长度 = a
B 独有部分长度 = b
公共部分长度 = c
```

那么：

```
m = a + c
n = b + c
```

假设：

```
a > b
```

那么：

```
m - n
= (a + c) - (b + c)
= a - b
```

让 A 先走：

```
a - b
```

步后：

```
A 剩余距离 = b + c
B 剩余距离 = b + c
```

两个指针就处于同一起跑线上了。

之后同时前进，必然同时进入公共部分。

------

## 复杂度分析

时间复杂度：

```
O(m + n)
```

虽然代码中会进行多次遍历，但所有遍历加起来仍然只是与两个链表长度成线性关系。

空间复杂度：

```
O(1)
```

只使用了几个指针和整数变量。

------

# 方法四：双指针交换链表

这是这道题最经典、也最推荐在面试中掌握的方法。

## 核心思路

我们不显式计算链表长度。

设置两个指针：

```
l1 = headA
l2 = headB
```

两个指针分别向前移动。

关键操作是：

> 当一个指针走到自己链表末尾后，让它从另一个链表的头部重新开始。

也就是说：

```
l1: A → B
l2: B → A
```

因此：

```
l1 总共走的路径 = A + B
l2 总共走的路径 = B + A
```

两者最终走过的总距离相同。

这样就自动消除了两个链表长度不同造成的偏移。

------

## 直观理解

假设：

```
A = A独有部分 + 公共部分
B = B独有部分 + 公共部分
```

记：

```
A 独有部分长度 = a
B 独有部分长度 = b
公共部分长度 = c
```

指针 `l1` 的路径是：

```
a + c + b
```

然后到达公共部分入口。

指针 `l2` 的路径是：

```
b + c + a
```

然后到达公共部分入口。

因为：

```
a + c + b
=
b + c + a
```

所以两个指针最终会同时到达交点。

这其实就是：

> 用交换链表的方式，隐式补偿两个链表的长度差。

------

## 算法步骤

1. 初始化：

```
l1 = headA
l2 = headB
```

1. 当：

```
l1 != l2
```

时不断移动。

1. 对于 `l1`：
   - 如果当前不是 `None`，向前走一步；
   - 如果已经是 `None`，跳到 `headB`。
2. 对于 `l2`：
   - 如果当前不是 `None`，向前走一步；
   - 如果已经是 `None`，跳到 `headA`。
3. 最终：
   - 如果存在交点，两者会在交点相遇；
   - 如果不存在交点，两者最终都会变成 `None`。

------

## Python 实现

```
# 单链表节点定义
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None


class Solution:
    def getIntersectionNode(
        self,
        headA: ListNode,
        headB: ListNode
    ) -> Optional[ListNode]:

        # 两个指针分别从两个链表头开始
        l1 = headA
        l2 = headB

        # 如果存在交点：
        # 最终 l1 和 l2 会在交点相遇
        #
        # 如果不存在交点：
        # 最终 l1 == l2 == None
        while l1 != l2:

            # l1 走完 A 后，从 B 的头重新开始
            if l1:
                l1 = l1.next
            else:
                l1 = headB

            # l2 走完 B 后，从 A 的头重新开始
            if l2:
                l2 = l2.next
            else:
                l2 = headA

        return l1
```

也可以使用更紧凑的 Python 写法：

```
class Solution:
    def getIntersectionNode(
        self,
        headA: ListNode,
        headB: ListNode
    ) -> Optional[ListNode]:

        l1, l2 = headA, headB

        while l1 != l2:
            l1 = l1.next if l1 else headB
            l2 = l2.next if l2 else headA

        return l1
```

------

# Python 三元表达式补充

上面的代码使用了：

```
l1 = l1.next if l1 else headB
```

这是 Python 的条件表达式，也称为三元表达式。

语法为：

```
value_if_true if condition else value_if_false
```

所以：

```
l1 = l1.next if l1 else headB
```

等价于：

```
if l1:
    l1 = l1.next
else:
    l1 = headB
```

这里：

```
if l1
```

实际上是在判断：

```
l1 is not None
```

因为 `None` 在布尔环境中会被视为 `False`。

------

# 为什么“双指针交换链表”不会死循环？

这是这个方法一个很重要的面试问题。

假设两个链表不相交。

两个指针分别走：

```
A → B
```

和：

```
B → A
```

因此它们最多都走：

```
m + n
```

个节点。

最终：

```
l1 = None
l2 = None
```

于是：

```
l1 == l2
```

循环退出。

所以不存在无限循环的问题。

------

# 四种方法对比

| 方法           | 时间复杂度 | 空间复杂度 | 是否推荐   |
| -------------- | ---------- | ---------- | ---------- |
| 暴力枚举       | `O(mn)`    | `O(1)`     | 不推荐     |
| Hash Set       | `O(m+n)`   | `O(m)`     | 可以       |
| 长度对齐双指针 | `O(m+n)`   | `O(1)`     | 推荐       |
| 交换链表双指针 | `O(m+n)`   | `O(1)`     | **最推荐** |

面试中比较理想的思考路线通常是：

```
暴力枚举
    ↓
Hash Set 优化重复查找
    ↓
发现其实不需要存储节点
    ↓
利用链表尾部对齐
    ↓
双指针交换链表
```

最终通常应该写出方法四。

------

# 常见错误

## 1. 比较节点值，而不是节点本身

这是本题最常见的错误。

错误：

```
if nodeA.val == nodeB.val:
    return nodeA
```

因为完全不同的两个节点也可能拥有相同的值：

```
A: 1 → 2 → 3

B: 9 → 2 → 8
```

这里两个 `2` 的值相同，但它们未必是同一个节点。

正确判断应该是：

```
if nodeA == nodeB:
```

或者更强调对象身份时，可以理解为：

```
nodeA is nodeB
```

本题判断的是：

> 是否共享同一个节点对象。

而不是：

> 是否存在相同的节点值。

------

## 2. 没有考虑两个链表不相交

在双指针交换链表方法中，如果没有交点，最终：

```
l1 == None
l2 == None
```

因此循环：

```
while l1 != l2:
```

会自然结束。

最终：

```
return l1
```

实际上就是：

```
return None
```

因此并不需要额外写：

```
if no_intersection:
    return None
```

------

## 3. 错误地修改链表结构

题目明确要求：

> 函数返回后，两个链表必须保持原来的结构。

因此不要尝试：

```
cur.next = ...
```

来改变链表。

这道题所有主流解法实际上都只需要移动指针，不需要修改 `next`。

------

## 4. 不理解为什么返回 `l1`

最终代码：

```
while l1 != l2:
    ...

return l1
```

退出循环只有两种情况。

### 情况一：存在交点

```
l1 == l2 == intersection_node
```

所以：

```
return l1
```

就是返回交点。

### 情况二：不存在交点

```
l1 == l2 == None
```

所以：

```
return l1
```

就是：

```
return None
```

因此一行代码同时处理了两种情况。

------

# 面试推荐写法

如果面试官直接要求最优方案，可以优先写下面这个版本：

```
class Solution:
    def getIntersectionNode(
        self,
        headA: ListNode,
        headB: ListNode
    ) -> Optional[ListNode]:

        p1, p2 = headA, headB

        while p1 != p2:
            # p1 走完 A 后改走 B
            p1 = p1.next if p1 else headB

            # p2 走完 B 后改走 A
            p2 = p2.next if p2 else headA

        # 有交点时是交点
        # 无交点时是 None
        return p1
```

面试中可以用一句话解释核心：

> 两个指针分别走 `A + B` 和 `B + A`，这样它们走过的总长度相同，相当于自动抵消了两个链表的长度差；如果存在交点，就会在交点相遇，否则最终同时到达 `None`。

这也是本题最值得记住的核心思想。