# Palindrome Linked List｜回文链表

给定一个单链表的头节点 `head`，如果该链表是**回文链表（Palindrome）**，返回 `True`；否则返回 `False`。

所谓**回文**，指一个序列从前往后读和从后往前读完全相同。

例如：

- `1 -> 2 -> 3 -> 2 -> 1`
- `2 -> 2`
- `1 -> 2 -> 1`

都是回文序列。

------

## 示例

### 示例 1

```text
Input: head = [1,2,3,2,1]

Output: true
```

### 示例 2

```text
Input: head = [2,2]

Output: true
```

### 示例 3

```text
Input: head = [2,1]

Output: false
```

------

## 约束

- `1 <= 链表长度 <= 100,000`
- `0 <= Node.val <= 9`

### Follow-up

你能否做到：

- 时间复杂度：`O(n)`
- 额外空间复杂度：`O(1)`

其中 `n` 为链表节点数量。

------

# 解法一：转换为数组

## 思路

判断回文最自然的方法，是同时比较序列最左边和最右边的元素。

对于数组来说，这非常容易，因为数组支持通过下标直接访问任意位置，例如：

```python
arr[0]
arr[-1]
```

但是单链表只能沿着 `next` 指针**从前向后遍历**，不能直接从尾部向前访问。

因此，一个最简单的解决办法是：

1. 先遍历链表；
2. 将所有节点的值保存到数组中；
3. 再使用左右双指针判断数组是否为回文。

例如：

```text
链表：

1 -> 2 -> 3 -> 2 -> 1

转换成：

[1, 2, 3, 2, 1]

 ↑           ↑
left       right
```

然后不断比较：

```text
arr[left] == arr[right]
```

并让两个指针向中间移动。

------

## 算法步骤

1. 遍历整个链表，将每个节点的值存入数组 `arr`。
2. 初始化：
   - `left = 0`
   - `right = len(arr) - 1`
3. 当 `left < right` 时：
   - 如果 `arr[left] != arr[right]`，说明不是回文，返回 `False`。
   - 否则：
     - `left += 1`
     - `right -= 1`
4. 如果所有对应位置都相同，则返回 `True`。

------

## Python 代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        arr = []

        # 遍历链表，将所有节点值存入数组
        cur = head
        while cur:
            arr.append(cur.val)
            cur = cur.next

        # 左右双指针
        left, right = 0, len(arr) - 1

        while left < right:
            # 对称位置的元素不同，则不是回文
            if arr[left] != arr[right]:
                return False

            left += 1
            right -= 1

        return True
```

------

## Python 知识点：`list.append()`

```python
arr.append(cur.val)
```

`append(x)` 会将元素 `x` 添加到 Python 列表的末尾。

例如：

```python
arr = []

arr.append(1)
arr.append(2)

print(arr)
```

结果：

```text
[1, 2]
```

一般情况下，`append()` 的均摊时间复杂度为 `O(1)`。

------

## 复杂度分析

假设链表长度为 `n`。

### 时间复杂度

```text
O(n)
```

第一次遍历链表需要 `O(n)`。

之后左右双指针最多比较约 `n / 2` 次，因此仍然是 `O(n)`。

总复杂度：

```text
O(n)
```

### 空间复杂度

```text
O(n)
```

因为需要额外数组保存所有 `n` 个节点的值。

------

# 解法二：递归

## 思路

单链表只能从前向后遍历，但**递归的回溯过程**可以帮助我们模拟“从后向前遍历”。

例如：

```text
1 -> 2 -> 3 -> 2 -> 1
```

递归调用过程：

```text
rec(第1个节点)
    rec(第2个节点)
        rec(第3个节点)
            rec(第4个节点)
                rec(第5个节点)
                    rec(None)
```

当递归到达链表末尾之后，开始逐层返回：

```text
第5个节点
第4个节点
第3个节点
第2个节点
第1个节点
```

也就是说，在递归**回溯**过程中，我们实际上能够按照：

```text
尾节点 -> ... -> 头节点
```

的顺序访问链表。

同时维护另一个指针：

```python
self.cur
```

从链表头部开始正常向后移动。

于是可以实现：

```text
self.cur        recursion node

第1个节点   vs   第5个节点
第2个节点   vs   第4个节点
第3个节点   vs   第3个节点
...
```

如果所有对应节点的值都相同，则链表是回文。

------

## 算法步骤

1. 使用 `self.cur` 指向链表头节点。
2. 定义递归函数 `rec(node)`。
3. 递归不断调用：

```python
rec(node.next)
```

直到链表末尾。

4. 在递归返回时：

- `node` 从链表尾部向前移动；
- `self.cur` 从链表头部向后移动。

1. 比较：

```python
self.cur.val == node.val
```

1. 如果不同，返回 `False`。
2. 如果相同，将：

```python
self.cur = self.cur.next
```

1. 如果所有比较均成功，则返回 `True`。

------

## Python 代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        # 左侧指针，从链表头开始向后移动
        self.cur = head

        def rec(node):
            # 先递归到链表末尾
            if node is not None:
                if not rec(node.next):
                    return False

                # 回溯时，node 从后往前，
                # self.cur 从前往后
                if self.cur.val != node.val:
                    return False

                # 左侧指针向后移动
                self.cur = self.cur.next

            return True

        return rec(head)
```

------

## 为什么这个方法有效？

关键在于理解递归的执行顺序。

假设：

```text
1 -> 2 -> 3
```

调用：

```python
rec(1)
```

实际进入顺序是：

```text
rec(1)
rec(2)
rec(3)
rec(None)
```

但比较操作发生在：

```python
rec(node.next)
```

返回之后。

因此比较顺序实际上是：

```text
3
2
1
```

这就是递归在这里能够模拟“反向遍历”的原因。

------

## 复杂度分析

### 时间复杂度

```text
O(n)
```

每个节点只被递归处理一次。

### 空间复杂度

```text
O(n)
```

虽然没有显式创建数组，但递归调用需要使用**调用栈（Call Stack）**。

递归深度最多达到 `n`，因此空间复杂度为：

```text
O(n)
```

------

## Python 注意事项：递归深度

这道题中有：

```text
n <= 100,000
```

因此 Python 中递归解法实际上存在一个非常重要的问题：

> Python 默认允许的递归深度通常只有约 1000 层。

如果链表非常长，代码可能触发：

```python
RecursionError
```

因此在 Python 面试中，这个方法更适合作为一种**算法思路**，但通常不应该作为本题的最佳实际实现。

------

# 解法三：栈

## 思路

栈（Stack）的特点是：

```text
Last In First Out
LIFO
后进先出
```

例如依次压入：

```text
1
2
3
```

栈中：

```text
顶部
3
2
1
底部
```

之后依次弹出的顺序就是：

```text
3, 2, 1
```

因此，我们可以：

1. 第一次遍历链表，将所有值压入栈；
2. 第二次从链表头部开始遍历；
3. 每访问一个节点，就从栈顶弹出一个值；
4. 比较两者是否相同。

这样就相当于：

```text
链表：头 -> 尾
栈：  尾 -> 头
```

从而判断回文。

------

## 算法步骤

1. 创建一个空栈。
2. 遍历链表，将所有节点值压入栈。
3. 再次从头遍历链表。
4. 每访问一个节点：
   - 使用 `stack.pop()` 取出栈顶元素；
   - 与当前节点值进行比较。
5. 如果不同，则返回 `False`。
6. 如果全部匹配，则返回 `True`。

------

## Python 代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        stack = []

        # 第一次遍历：
        # 将所有节点值压入栈
        cur = head
        while cur:
            stack.append(cur.val)
            cur = cur.next

        # 第二次遍历：
        # 链表从前往后，栈从后往前
        cur = head
        while cur:
            if cur.val != stack.pop():
                return False

            cur = cur.next

        return True
```

原答案还可以写成更紧凑的形式：

```python
cur = head

while cur and cur.val == stack.pop():
    cur = cur.next

return not cur
```

但在代码面试中，前一种显式判断的写法通常更容易解释，也更不容易出现理解错误。

------

## Python 知识点：`list.pop()`

Python 的 `list` 可以直接作为栈使用。

压栈：

```python
stack.append(x)
```

弹栈：

```python
x = stack.pop()
```

例如：

```python
stack = []

stack.append(1)
stack.append(2)
stack.append(3)

print(stack.pop())  # 3
print(stack.pop())  # 2
```

因此：

```python
append()
```

对应：

```text
push
```

而：

```python
pop()
```

对应：

```text
pop
```

当从列表尾部进行 `append()` 和 `pop()` 时，它们的均摊时间复杂度通常都是 `O(1)`。

------

## 复杂度分析

### 时间复杂度

```text
O(n)
```

遍历链表两次：

```text
O(n) + O(n) = O(n)
```

### 空间复杂度

```text
O(n)
```

栈中保存了所有节点的值。

------

# 解法四：快慢指针 + 原地反转链表

这是本题最重要的解法，也是满足 Follow-up：

```text
时间 O(n)
空间 O(1)
```

的标准方案。

------

## 核心思路

如果能够把链表后半部分反转：

```text
原链表：

1 -> 2 -> 3 -> 2 -> 1
```

找到中点后，将后半部分：

```text
3 -> 2 -> 1
```

反转成：

```text
1 -> 2 -> 3
```

然后只需要同时遍历：

```text
前半部分：
1 -> 2 -> 3

反转后的后半部分：
1 -> 2 -> 3
```

逐个比较即可。

因此整个算法可以拆成三个经典链表操作：

```text
1. 快慢指针寻找中点
2. 反转后半部分
3. 同时遍历两部分进行比较
```

------

# 第一步：快慢指针寻找中点

使用两个指针：

```python
slow = head
fast = head
```

其中：

```text
slow 每次走 1 步
fast 每次走 2 步
```

代码：

```python
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

为什么这样能够找到中点？

因为：

```text
fast 的速度 = slow 的 2 倍
```

所以当 `fast` 到达链表末尾时，`slow` 正好大约走完整个链表长度的一半。

------

## 偶数长度链表

例如：

```text
1 -> 2 -> 2 -> 1
```

开始时：

```text
slow
 ↓
 1 -> 2 -> 2 -> 1
 ↑
fast
```

最终：

```text
          slow
           ↓
1 -> 2 -> 2 -> 1
                    fast = None
```

此时 `slow` 位于后半部分的第一个节点。

------

## 奇数长度链表

例如：

```text
1 -> 2 -> 3 -> 2 -> 1
```

最终：

```text
          slow
           ↓
1 -> 2 -> 3 -> 2 -> 1
                    ↑
                   fast
```

此时 `slow` 正好指向中间节点：

```text
3
```

------

# 第二步：反转后半部分链表

这是非常经典的链表反转模板。

假设：

```text
slow

 ↓
3 -> 2 -> 1 -> None
```

我们希望变成：

```text
None <- 3 <- 2 <- 1

                  ↑
                 prev
```

代码：

```python
prev = None

while slow:
    nxt = slow.next
    slow.next = prev
    prev = slow
    slow = nxt
```

------

## 链表反转模板详解

每一次循环完成三件事。

### 1. 保存下一个节点

```python
nxt = slow.next
```

这是必须的。

因为下一步我们会修改：

```python
slow.next
```

如果不提前保存原来的下一个节点，就会丢失剩余链表。

------

### 2. 反转指针

```python
slow.next = prev
```

例如：

```text
3 -> 2
```

变成：

```text
3 <- 2
```

------

### 3. 两个指针向前移动

```python
prev = slow
slow = nxt
```

最终：

```python
prev
 ↓
1 -> 2 -> 3 -> None
```

因此 `prev` 就是反转后的链表头节点。

------

# 第三步：比较两个链表

此时：

```python
left = head
right = prev
```

然后逐个比较：

```python
while right:
    if left.val != right.val:
        return False

    left = left.next
    right = right.next
```

这里有一个很重要的细节：

```python
while right:
```

而不是：

```python
while left and right:
```

我们的目标实际上是：

> 检查反转后的后半部分是否完全匹配前半部分。

------

## 完整代码

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        # --------------------------------
        # 1. 快慢指针寻找链表中点
        # --------------------------------
        slow = head
        fast = head

        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        # --------------------------------
        # 2. 原地反转后半部分链表
        # --------------------------------
        prev = None

        while slow:
            nxt = slow.next       # 保存下一个节点
            slow.next = prev      # 反转当前指针
            prev = slow           # prev 前进一步
            slow = nxt            # slow 前进一步

        # 此时 prev 是反转后半部分的头节点

        # --------------------------------
        # 3. 比较前半部分和反转后的后半部分
        # --------------------------------
        left = head
        right = prev

        while right:
            if left.val != right.val:
                return False

            left = left.next
            right = right.next

        return True
```

------

# 为什么奇数长度也可以直接从 `slow` 开始反转？

例如：

```text
1 -> 2 -> 3 -> 2 -> 1
```

找到中点：

```text
          slow
           ↓
1 -> 2 -> 3 -> 2 -> 1
```

从 `slow` 开始反转后：

```text
1 -> 2 -> 3
```

然后与链表头部比较：

```text
left:   1 -> 2 -> 3
right:  1 -> 2 -> 3
```

比较：

```text
1 == 1
2 == 2
3 == 3
```

仍然完全成立。

也就是说：

> 奇数长度时，中间节点虽然被包含在后半部分中，但它最终只会和自己对应的位置进行比较，因此不会影响结果。

------

# 复杂度分析

## 时间复杂度

```text
O(n)
```

三个阶段分别为：

寻找中点：

```text
O(n)
```

反转后半链表：

```text
O(n / 2)
```

比较：

```text
O(n / 2)
```

因此：

```text
O(n) + O(n / 2) + O(n / 2)
= O(n)
```

------

## 空间复杂度

```text
O(1)
```

只使用了几个指针变量：

```python
slow
fast
prev
nxt
left
right
```

无论链表有多少节点，所使用的额外变量数量都是固定的，因此额外空间复杂度为：

```text
O(1)
```

------

# 常见陷阱

## 1. 奇数长度链表的中点处理

快慢指针代码：

```python
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

对于奇数长度：

```text
1 -> 2 -> 3 -> 2 -> 1
```

`slow` 最终会停在：

```text
3
```

也就是正中间节点。

如果直接从 `slow` 开始反转：

```text
3 -> 2 -> 1
```

虽然中间节点也被包含进来，但算法仍然正确。

需要避免因为想手动跳过中间节点，而引入不必要的 off-by-one 错误。

------

## 2. 反转链表时忘记保存 `next`

错误示例：

```python
slow.next = prev
slow = slow.next
```

问题在于：

```python
slow.next
```

已经被修改成了 `prev`。

因此原来的剩余链表就找不到了。

正确模式一定是：

```python
nxt = slow.next
slow.next = prev
prev = slow
slow = nxt
```

这个模板非常值得直接记住。

------

## 3. 原地反转会修改原链表

上面的 `O(1)` 解法会修改原始链表结构。

例如原来：

```text
1 -> 2 -> 3 -> 2 -> 1
```

执行过程中后半部分被反转。

在 LeetCode 这道题中通常可以接受，但在真实工程或者某些面试追问中，面试官可能会问：

> 如果要求函数执行结束后，链表结构完全保持不变怎么办？

解决方法是：

1. 找到后半部分；
2. 反转后半部分；
3. 判断回文；
4. **再次反转后半部分，将链表恢复。**

------

# 推荐版本：判断后恢复原链表

这是一个更加完整、工程上更安全的版本。

```python
class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        if not head or not head.next:
            return True

        # --------------------------------
        # 1. 找到前半部分的最后一个节点
        # --------------------------------
        slow = head
        fast = head

        # 这里使用 fast.next 和 fast.next.next，
        # 可以让 slow 停在“前半部分最后一个节点”
        while fast.next and fast.next.next:
            slow = slow.next
            fast = fast.next.next

        # --------------------------------
        # 2. 反转后半部分
        # --------------------------------
        second_half = self.reverse(slow.next)

        # 保存反转后的头节点，
        # 后面需要再次反转以恢复链表
        second_half_head = second_half

        # --------------------------------
        # 3. 比较两半
        # --------------------------------
        first_half = head
        result = True

        while second_half:
            if first_half.val != second_half.val:
                result = False
                break

            first_half = first_half.next
            second_half = second_half.next

        # --------------------------------
        # 4. 恢复原链表
        # --------------------------------
        slow.next = self.reverse(second_half_head)

        return result

    def reverse(self, head: Optional[ListNode]) -> Optional[ListNode]:
        """反转链表，并返回新的头节点。"""
        prev = None
        cur = head

        while cur:
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt

        return prev
```

这个版本仍然满足：

```text
时间复杂度：O(n)
空间复杂度：O(1)
```

但函数执行结束后，原链表结构也不会被破坏。

------

# 四种方法对比

| 方法                | 时间复杂度 | 空间复杂度 | 是否修改链表 | 推荐程度 |
| ------------------- | ---------- | ---------- | ------------ | -------- |
| 数组 + 双指针       | `O(n)`     | `O(n)`     | 否           | ⭐⭐⭐⭐     |
| 递归                | `O(n)`     | `O(n)`     | 否           | ⭐⭐       |
| 栈                  | `O(n)`     | `O(n)`     | 否           | ⭐⭐⭐      |
| 快慢指针 + 反转链表 | `O(n)`     | `O(1)`     | 是           | ⭐⭐⭐⭐⭐    |
| 反转后再恢复        | `O(n)`     | `O(1)`     | 最终恢复     | ⭐⭐⭐⭐⭐    |

------

# 面试中的思考路线

这道题非常适合按照“从简单到最优”的方式进行回答。

首先可以说：

> 最直接的方法是遍历链表，将所有节点值保存进数组，然后使用左右双指针判断回文。

复杂度：

```text
Time:  O(n)
Space: O(n)
```

如果面试官继续要求：

```text
Can you do it in O(1) extra space?
```

那么就进一步想到：

> 空间主要浪费在保存链表节点值。如果不保存这些值，就需要找到一种办法直接比较链表前半部分和后半部分。

于是自然得到：

```text
寻找中点
    ↓
反转后半部分
    ↓
比较两半
```

即：

```text
Fast / Slow Pointers
        +
Linked List Reversal
```

这也是本题最核心的算法组合。

------

# 本题需要掌握的三个基础模板

## 模板 1：遍历链表

```python
cur = head

while cur:
    # 使用 cur.val

    cur = cur.next
```

------

## 模板 2：快慢指针寻找中点

```python
slow = head
fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

核心规律：

```text
slow：一次走一步
fast：一次走两步
```

------

## 模板 3：反转单链表

```python
prev = None
cur = head

while cur:
    nxt = cur.next
    cur.next = prev

    prev = cur
    cur = nxt

return prev
```

这三个模板不仅会出现在本题中，也会频繁出现在其他链表题中。

------

# 面试记忆版

如果只想记住最优解，可以记成一句话：

```text
找中点 → 反转后半段 → 两边比较
```

对应代码骨架：

```python
# 1. 找中点
slow = fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next


# 2. 反转后半段
prev = None

while slow:
    nxt = slow.next
    slow.next = prev
    prev = slow
    slow = nxt


# 3. 比较
left = head
right = prev

while right:
    if left.val != right.val:
        return False

    left = left.next
    right = right.next

return True
```

------

# 总结

这道题主要考察三个知识点：

```text
1. 回文判断
2. 快慢指针
3. 链表原地反转
```

如果不考虑空间限制，最容易实现的是：

```text
链表 → 数组 → 双指针
```

如果要求：

```text
O(n) 时间
O(1) 额外空间
```

则应该使用：

```text
快慢指针寻找中点
        ↓
原地反转链表后半部分
        ↓
比较前后两部分
```

其中最值得熟练掌握的是链表反转模板：

```python
prev = None

while cur:
    nxt = cur.next
    cur.next = prev
    prev = cur
    cur = nxt
```

它是链表类面试题中最常见、也最重要的基础操作之一。
