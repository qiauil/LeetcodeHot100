# 两数相加（Add Two Numbers）

给定两个**非空**链表 `l1` 和 `l2`，它们分别表示两个非负整数。

数字按照**逆序**存储。例如，数字 `321` 在链表中表示为：

```text
1 -> 2 -> 3
```

链表中的每个节点只存储一位数字。

可以假设这两个数字都不会包含前导零，除非数字本身就是 `0`。

请返回两个数字之和，并同样使用链表表示结果。

## 约束

- `1 <= l1.length, l2.length <= 100`
- `0 <= Node.val <= 9`

------

# 1. 递归解法

## 思路

这道题本质上是在模拟我们手算整数加法的过程。

由于链表中的数字是**逆序存储**的，因此链表头节点恰好对应个位：

```text
342  ->  2 -> 4 -> 3
465  ->  5 -> 6 -> 4
```

所以我们可以直接从链表头开始逐位相加，而不需要像普通整数加法那样从末尾开始。

每一位需要处理三个值：

1. `l1` 当前节点的数字
2. `l2` 当前节点的数字
3. 上一位产生的进位 `carry`

例如：

```text
2 + 5 + 0 = 7
4 + 6 + 0 = 10
3 + 4 + 1 = 8
```

最终得到：

```text
7 -> 0 -> 8
```

也就是：

```text
342 + 465 = 807
```

递归非常适合这个过程，因为对于当前位计算完成后，我们只需要把问题转换成：

> “剩余两个链表 + 当前产生的进位”

也就是一个结构完全相同的子问题。

------

## 递归状态

定义：

```python
add(l1, l2, carry)
```

表示：

> 将从 `l1` 和 `l2` 当前节点开始的数字，加上上一位传入的 `carry`，返回结果链表。

### 递归终止条件

只有在：

```text
l1 已经结束
l2 已经结束
carry == 0
```

三个条件同时满足时，才说明整个加法真正结束。

否则即使两个链表都结束了，只要还有进位，就还需要创建一个节点。

例如：

```text
5 + 5 = 10
```

对应：

```text
5
+
5
```

当前位产生：

```text
0
```

并留下：

```text
carry = 1
```

因此最后还需要额外生成节点 `1`。

------

## 算法步骤

定义递归函数：

```python
add(l1, l2, carry)
```

对于每一层递归：

1. 如果 `l1`、`l2` 都为空，并且 `carry == 0`：
   - 返回 `None`
2. 获取两个链表当前节点的值：
   - 如果节点不存在，则当作 `0`
3. 计算当前位的总和：

```python
total = v1 + v2 + carry
```

1. 得到：
   - 当前结果位
   - 下一位进位
2. 递归处理下一个节点
3. 创建当前结果节点，并把递归结果连接到 `next`

------

## Python 中的 `divmod`

这里可以使用 Python 内置函数：

```python
divmod(a, b)
```

它会一次返回：

```python
(a // b, a % b)
```

例如：

```python
divmod(17, 10)
```

返回：

```python
(1, 7)
```

因此：

```python
carry, digit = divmod(total, 10)
```

就等价于：

```python
carry = total // 10
digit = total % 10
```

在这种“商表示进位、余数表示当前数字”的题目中，`divmod` 非常方便。

------

## 代码

```python
# 单链表节点定义
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def add(
        self,
        l1: Optional[ListNode],
        l2: Optional[ListNode],
        carry: int
    ) -> Optional[ListNode]:

        # 递归终止：
        # 两个链表都处理完，并且没有剩余进位
        if not l1 and not l2 and carry == 0:
            return None

        # 如果某个链表已经结束，则这一位视为 0
        v1 = l1.val if l1 else 0
        v2 = l2.val if l2 else 0

        # divmod(total, 10) 返回：
        # (total // 10, total % 10)
        # 即：(新的进位, 当前位数字)
        carry, digit = divmod(v1 + v2 + carry, 10)

        # 递归处理下一位
        next_node = self.add(
            l1.next if l1 else None,
            l2.next if l2 else None,
            carry
        )

        # 当前节点连接递归得到的后续结果
        return ListNode(digit, next_node)

    def addTwoNumbers(
        self,
        l1: Optional[ListNode],
        l2: Optional[ListNode]
    ) -> Optional[ListNode]:

        return self.add(l1, l2, 0)
```

------

## 递归过程示例

假设：

```text
l1 = 2 -> 4 -> 3
l2 = 5 -> 6 -> 4
```

第一次：

```text
2 + 5 + 0 = 7

digit = 7
carry = 0
```

第二次：

```text
4 + 6 + 0 = 10

digit = 0
carry = 1
```

第三次：

```text
3 + 4 + 1 = 8

digit = 8
carry = 0
```

最终：

```text
7 -> 0 -> 8
```

------

## 时间复杂度与空间复杂度

设：

```text
m = l1 的长度
n = l2 的长度
```

### 时间复杂度

```text
O(max(m, n))
```

因为最多需要处理较长链表的每一个节点，并可能额外处理一次最终进位。

也可以宽松地写成：

```text
O(m + n)
```

但这里 `O(max(m, n))` 更准确地描述了实际循环/递归次数。

### 空间复杂度

如果只计算**额外辅助空间**：

```text
O(max(m, n))
```

原因是递归调用栈最多有 `max(m, n) + 1` 层。

另外，返回结果链表本身需要：

```text
O(max(m, n))
```

空间。

> 面试中通常需要区分：
>
> - **辅助空间**：算法额外使用的内存
> - **输出空间**：最终答案本身占用的内存

------

# 2. 迭代解法

## 思路

迭代解法仍然是在模拟竖式加法，只不过不用递归，而是同时维护两个链表指针。

每次循环：

```text
l1 当前数字
+
l2 当前数字
+
carry
```

然后：

```python
digit = total % 10
carry = total // 10
```

创建新节点保存 `digit`，最后让两个输入链表指针继续向后移动。

------

## 为什么使用 Dummy Node？

构造链表时经常会使用一个**哑节点（dummy node）**。

例如：

```python
dummy = ListNode()
cur = dummy
```

之后每生成一个节点：

```python
cur.next = ListNode(digit)
cur = cur.next
```

最终结果链表的真正头节点是：

```python
dummy.next
```

### Dummy Node 的作用

如果不用 `dummy`，我们就需要专门判断：

```text
“现在创建的是不是结果链表的第一个节点？”
```

可能会写成：

```python
if head is None:
    head = new_node
else:
    cur.next = new_node
```

使用 `dummy` 后，就可以统一处理所有节点，不需要对头节点做特殊判断。

这是链表题中非常常见的技巧。

------

## 算法步骤

1. 创建：
   - `dummy`：结果链表之前的哑节点
   - `cur`：指向当前结果链表尾部
   - `carry = 0`
2. 只要满足下面任意条件，就继续循环：

```text
l1 还有节点
或者
l2 还有节点
或者
carry != 0
```

也就是：

```python
while l1 or l2 or carry:
```

1. 获取当前数字：

```python
v1 = l1.val if l1 else 0
v2 = l2.val if l2 else 0
```

1. 计算当前位：

```python
total = v1 + v2 + carry
carry = total // 10
digit = total % 10
```

1. 创建结果节点：

```python
cur.next = ListNode(digit)
```

1. 所有指针向后移动
2. 返回：

```python
dummy.next
```

------

## 代码

```python
# 单链表节点定义
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def addTwoNumbers(
        self,
        l1: Optional[ListNode],
        l2: Optional[ListNode]
    ) -> Optional[ListNode]:

        # dummy 是哑节点，不属于最终答案的一部分
        dummy = ListNode()

        # cur 始终指向结果链表的最后一个节点
        cur = dummy

        # 保存上一位产生的进位
        carry = 0

        # 只要：
        # 1. l1 还有节点
        # 2. l2 还有节点
        # 3. 或者仍然存在进位
        # 就需要继续处理
        while l1 or l2 or carry:

            # 某个链表结束后，对应数字视为 0
            v1 = l1.val if l1 else 0
            v2 = l2.val if l2 else 0

            # 当前位总和
            total = v1 + v2 + carry

            # 更新进位
            carry = total // 10

            # 当前结果数字
            digit = total % 10

            # 把当前数字加入结果链表
            cur.next = ListNode(digit)

            # cur 移动到新创建的节点
            cur = cur.next

            # 输入链表指针向后移动
            if l1:
                l1 = l1.next

            if l2:
                l2 = l2.next

        # dummy 本身不是答案，因此返回 dummy.next
        return dummy.next
```

------

# 复杂度分析

设：

```text
m = l1 的长度
n = l2 的长度
```

## 时间复杂度

```text
O(max(m, n))
```

我们最多遍历较长链表一次，并可能由于最终进位额外执行一次循环。

## 空间复杂度

如果不计算输出链表：

```text
O(1)
```

因为迭代解法只维护：

```text
dummy
cur
carry
v1
v2
total
```

等少量变量，辅助空间不会随着输入规模增长。

如果计算返回结果所需空间，则结果链表需要：

```text
O(max(m, n))
```

------

# 递归与迭代的比较

| 方法 | 时间复杂度    | 辅助空间      | 特点                               |
| ---- | ------------- | ------------- | ---------------------------------- |
| 递归 | `O(max(m,n))` | `O(max(m,n))` | 代码结构直观，与问题递归定义一致   |
| 迭代 | `O(max(m,n))` | `O(1)`        | 更节省空间，更适合作为面试标准解法 |

通常更推荐**迭代解法**。

原因不是它的时间复杂度更好，而是递归解法需要额外的调用栈空间。

另外，Python 默认递归深度通常在千级左右。虽然本题链表长度最多只有 `100`，不会出现问题，但对于长度没有严格限制的链表题，迭代通常更加稳妥。

------

# 常见错误

## 1. 忘记处理最后的进位

例如：

```text
l1 = 9 -> 9 -> 9
l2 = 1
```

对应：

```text
999 + 1 = 1000
```

计算到两个链表都结束后：

```text
carry = 1
```

因此还需要额外创建节点 `1`。

错误：

```python
while l1 or l2:
    ...
```

正确：

```python
while l1 or l2 or carry:
    ...
```

------

## 2. 使用 `l1 and l2`

错误写法：

```python
while l1 and l2:
```

这意味着：

> 只有两个链表都还有节点时才继续。

如果：

```text
l1 = 2 -> 4 -> 3
l2 = 5
```

处理完第一位之后 `l2` 就为空，循环会直接结束，导致 `l1` 剩余部分没有被处理。

正确写法是：

```python
while l1 or l2 or carry:
```

------

## 3. 链表结束后没有把数字当作 `0`

因为两个链表长度可能不同，所以不能直接写：

```python
v1 = l1.val
v2 = l2.val
```

否则当其中一个链表先结束时会访问 `None.val`。

应该写成：

```python
v1 = l1.val if l1 else 0
v2 = l2.val if l2 else 0
```

这个写法实际上把：

```text
243
+
  5
```

理解为：

```text
243
+
005
```

------

## 4. 混淆当前数字和进位

对于：

```python
total = v1 + v2 + carry
```

应该是：

```python
digit = total % 10
carry = total // 10
```

例如：

```text
8 + 7 + 1 = 16
```

那么：

```text
当前位 digit = 6
下一位 carry = 1
```

而不是反过来。

------

# 面试中的核心总结

这道题最重要的观察是：

> **链表采用逆序存储，使得链表的遍历方向和整数加法的计算方向完全一致。**

因此不需要：

- 反转链表
- 转换成整数
- 使用栈

只需要同时遍历两个链表，并维护一个 `carry`。

可以把整个算法浓缩成三个公式：

```python
total = v1 + v2 + carry
digit = total % 10
carry = total // 10
```

以及一个关键循环条件：

```python
while l1 or l2 or carry:
```

如果在面试中采用迭代解法，一个比较清晰的解释方式是：

> “我会同时遍历两个链表。因为数字是逆序存储的，所以链表头就是个位，可以直接模拟竖式加法。对于长度不一致的情况，把不存在的节点视为 0。每次计算当前位和进位，并通过 dummy node 构造结果链表。循环直到两个链表都结束且没有剩余进位。”
