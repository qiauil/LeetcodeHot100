# Symmetric Tree｜对称二叉树

给定一棵二叉树的根节点 `root`，判断这棵树是否关于中心轴**镜像对称**。

例如，一棵对称二叉树需要满足：

- 左子树的根节点与右子树的根节点值相同；
- 左子树的左子树与右子树的右子树互为镜像；
- 左子树的右子树与右子树的左子树互为镜像。

因此，这道题真正要判断的不是“左右子树是否相同”，而是：

> **左右子树是否互为镜像。**

------

## 一、核心思路：比较“镜像节点对”

假设现在有两个节点 `left` 和 `right`，它们应该位于整棵树中互相镜像的位置。

它们互为镜像需要同时满足：

1. `left` 和 `right` 都为空；
2. 或者二者都不为空，并且：
   - `left.val == right.val`
   - `left.left` 与 `right.right` 互为镜像
   - `left.right` 与 `right.left` 互为镜像

可以把镜像关系表示为：

```text
        left                 right
       /    \               /     \
      A      B             C       D

镜像比较：

A <--------> D
B <--------> C
```

最重要的比较顺序是：

```python
left.left   <-> right.right
left.right  <-> right.left
```

而不是：

```python
left.left   <-> right.left
left.right  <-> right.right
```

后者是在判断两棵树是否结构完全相同，而不是判断是否镜像。

------

# 方法一：递归 DFS

## 思路

这是本题最自然、也通常是面试中最推荐的解法。

定义一个递归函数：

```python
dfs(left, right)
```

它表示：

> 判断 `left` 和 `right` 两棵子树是否互为镜像。

对于每一对节点，只需要考虑三种情况：

### 情况 1：两个节点都为空

```python
if not left and not right:
    return True
```

两个空位置天然是对称的。

------

### 情况 2：只有一个节点为空

```python
if not left or not right:
    return False
```

例如：

```text
    left        right
     3           3
    /             \
   4               None
```

对应位置一个有节点、一个没有节点，因此一定不对称。

------

### 情况 3：两个节点都存在

需要同时检查：

```python
left.val == right.val
```

以及：

```python
dfs(left.left, right.right)
dfs(left.right, right.left)
```

只有三个条件全部满足，这两个节点对应的子树才互为镜像。

------

## 算法步骤

1. 从根节点的左右孩子开始比较：

   ```python
   dfs(root.left, root.right)
   ```

2. 如果两个节点都为空，返回 `True`。

3. 如果只有一个节点为空，返回 `False`。

4. 如果两个节点的值不同，返回 `False`。

5. 递归检查两组镜像位置：

   - `left.left` 和 `right.right`
   - `left.right` 和 `right.left`

6. 两组递归检查都为 `True` 时，当前子树才是镜像的。

------

## Python 代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        # 空树可以认为是对称的
        if not root:
            return True

        def dfs(left, right):
            # 情况 1：
            # 两个对应位置都为空，说明这一对位置是对称的
            if not left and not right:
                return True

            # 情况 2：
            # 只有一个为空，结构不对称
            if not left or not right:
                return False

            # 情况 3：
            # 两个节点都存在：
            # 1. 当前节点值必须相同
            # 2. 外侧节点必须镜像
            # 3. 内侧节点必须镜像
            return (
                left.val == right.val
                and dfs(left.left, right.right)
                and dfs(left.right, right.left)
            )

        # 从根节点左右子树开始进行镜像比较
        return dfs(root.left, root.right)
```

------

## 为什么这里是“交叉比较”？

这是这道题最关键的地方。

假设：

```text
           root
          /    \
       left    right
       /  \    /   \
      A    B  C     D
```

如果整棵树对称，那么：

```text
A 和 D 对称
B 和 C 对称
```

所以递归关系必须是：

```python
dfs(left.left, right.right)
dfs(left.right, right.left)
```

可以记忆为：

> **外侧和外侧比，内侧和内侧比。**

也就是：

```text
left.left   ↔ right.right    # 外侧
left.right  ↔ right.left     # 内侧
```

------

## 复杂度分析

设二叉树一共有 `n` 个节点，高度为 `h`。

### 时间复杂度

```text
O(n)
```

最坏情况下，每个节点都需要被访问一次。

### 空间复杂度

递归调用栈的深度取决于树高：

```text
O(h)
```

如果二叉树比较平衡：

```text
h = O(log n)
```

因此空间复杂度约为：

```text
O(log n)
```

如果树退化成类似链表的结构：

```text
h = O(n)
```

最坏空间复杂度为：

```text
O(n)
```

因此，更准确的表达是：

> 空间复杂度为 `O(h)`，最坏情况下为 `O(n)`。

------

# 方法二：迭代 DFS

## 思路

递归 DFS 本质上依赖 Python 的**函数调用栈**保存后续需要处理的节点。

我们也可以自己创建一个 `stack`，显式保存：

```python
(left_node, right_node)
```

每个 tuple 都表示：

> 这两个节点应该互为镜像。

初始时放入：

```python
(root.left, root.right)
```

每次弹出一对节点并进行比较。

如果当前节点匹配，则继续加入下一层的两组镜像节点：

```python
(left.left, right.right)
(left.right, right.left)
```

------

## 算法步骤

1. 创建栈：

   ```python
   stack = [(root.left, root.right)]
   ```

2. 当栈不为空时：

   - 弹出一组镜像节点；
   - 如果两个都为空，继续处理下一组；
   - 如果只有一个为空，返回 `False`；
   - 如果值不同，返回 `False`。

3. 将下一层需要比较的两组节点压入栈：

   ```python
   (left.left, right.right)
   (left.right, right.left)
   ```

4. 如果所有节点都检查完成，没有发现不对称情况，返回 `True`。

------

## Python 代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True

        # stack 中的每个元素都是一对应该互为镜像的节点
        stack = [(root.left, root.right)]

        while stack:
            # DFS 使用栈，所以从末尾弹出
            left, right = stack.pop()

            # 两个节点都为空，当前这一对是对称的
            if not left and not right:
                continue

            # 一个为空、另一个不为空，或者节点值不同
            # 都说明整棵树不对称
            if not left or not right or left.val != right.val:
                return False

            # 加入下一层需要比较的镜像节点
            #
            # 外侧节点：
            stack.append((left.left, right.right))

            # 内侧节点：
            stack.append((left.right, right.left))

        return True
```

------

## Python：`list` 为什么可以作为 Stack？

Python 没有专门内置一个叫 `Stack` 的类。

通常直接使用 `list`：

```python
stack = []
```

入栈：

```python
stack.append(x)
```

出栈：

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
print(stack.pop())  # 1
```

这是典型的：

```text
LIFO
Last In, First Out
后进先出
```

因此：

```python
list.append()
list.pop()
```

非常适合实现 DFS。

这两个操作在列表尾部执行时，平均时间复杂度都是：

```text
O(1)
```

------

## 复杂度分析

### 时间复杂度

```text
O(n)
```

每个节点最多被处理一次。

### 空间复杂度

```text
O(n)
```

最坏情况下，栈中可能同时保存 `O(n)` 个待处理节点对。

------

# 方法三：BFS / 队列

## 思路

除了 DFS，也可以使用 BFS。

DFS 使用：

```text
Stack
```

BFS 使用：

```text
Queue
```

我们仍然存储：

```python
(left, right)
```

这样的镜像节点对。

区别只是：

- DFS：后加入的节点先处理；
- BFS：先加入的节点先处理。

对于本题来说，实际上**并不要求真正按照“层”来处理**，因为我们关心的只是每一组镜像节点是否匹配。

因此 BFS 不需要：

```python
for _ in range(len(queue)):
```

这一层循环。

直接不断从队首取出节点对即可。

------

## Python 代码

```python
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True

        # queue 中保存应该互为镜像的节点对
        queue = deque([(root.left, root.right)])

        while queue:
            # BFS：从队首取出节点
            left, right = queue.popleft()

            # 两个位置都为空，当前节点对对称
            if not left and not right:
                continue

            # 结构不同或者节点值不同
            if not left or not right or left.val != right.val:
                return False

            # 下一层的外侧节点
            queue.append((left.left, right.right))

            # 下一层的内侧节点
            queue.append((left.right, right.left))

        return True
```

------

## Python：`collections.deque`

BFS 通常使用：

```python
from collections import deque
```

然后创建队列：

```python
queue = deque()
```

在队尾加入元素：

```python
queue.append(x)
```

从队首删除元素：

```python
x = queue.popleft()
```

例如：

```python
from collections import deque

queue = deque()

queue.append(1)
queue.append(2)
queue.append(3)

print(queue.popleft())  # 1
print(queue.popleft())  # 2
print(queue.popleft())  # 3
```

这是：

```text
FIFO
First In, First Out
先进先出
```

------

## 为什么 BFS 不推荐使用 `list.pop(0)`？

理论上也可以写：

```python
queue = [1, 2, 3]
queue.pop(0)
```

但 Python 的 `list` 是动态数组。

删除第一个元素后，后面的元素需要整体向前移动：

```text
[1, 2, 3, 4]

删除 1

[2, 3, 4]
 ↑  ↑  ↑
这些元素需要移动
```

因此：

```python
list.pop(0)
```

时间复杂度是：

```text
O(n)
```

而：

```python
deque.popleft()
```

时间复杂度是：

```text
O(1)
```

所以面试中写 BFS 时，应优先使用：

```python
collections.deque
```

------

## 复杂度分析

### 时间复杂度

```text
O(n)
```

每个节点最多访问一次。

### 空间复杂度

如果用 `w` 表示二叉树的最大宽度，那么 BFS 的空间复杂度可以写成：

```text
O(w)
```

最坏情况下：

```text
w = O(n)
```

因此最坏空间复杂度：

```text
O(n)
```

------

# 三种方法对比

| 方法     | 数据结构           | 时间复杂度 | 空间复杂度          | 特点               |
| -------- | ------------------ | ---------- | ------------------- | ------------------ |
| 递归 DFS | 递归调用栈         | `O(n)`     | `O(h)`              | 最简洁、最符合题意 |
| 迭代 DFS | `list` 作为 Stack  | `O(n)`     | `O(n)` 最坏         | 避免递归           |
| BFS      | `deque` 作为 Queue | `O(n)`     | `O(w)`，最坏 `O(n)` | 按队列顺序检查     |

其中：

```text
h = 树的高度
w = 树的最大宽度
```

对于代码面试，通常优先掌握：

```text
递归 DFS
```

因为它最直接地表达了：

> “两棵子树是否互为镜像”这一递归定义。

------

# 常见错误

## 1. 比较了错误的节点

最常见的错误是写成：

```python
dfs(left.left, right.left)
dfs(left.right, right.right)
```

这种比较是在判断：

> 左右两棵子树是不是完全相同。

而不是判断：

> 左右两棵子树是不是镜像。

镜像比较必须交叉：

```python
dfs(left.left, right.right)
dfs(left.right, right.left)
```

可以记住：

```text
外 ↔ 外
内 ↔ 内
```

------

## 2. 在检查 `None` 之前访问 `.val`

例如：

```python
if left.val != right.val:
```

如果：

```python
left is None
```

就会发生错误：

```text
AttributeError:
'NoneType' object has no attribute 'val'
```

因此正确顺序必须是：

```python
if not left and not right:
    return True

if not left or not right:
    return False

if left.val != right.val:
    return False
```

也就是说：

> **先检查节点是否存在，再访问节点属性。**

------

## 3. 只比较节点值，不比较树的结构

例如：

```text
        1
       / \
      2   2
     /     /
    3     3
```

左右两边虽然都出现：

```text
2, 3
```

但结构实际上是：

```text
左边的 3 在左侧
右边的 3 也在左侧
```

它们不是镜像结构。

所以仅比较节点值是不够的，还必须把：

```python
None
```

的位置一起考虑进去。

------

## 4. BFS 中不必要地逐层处理

原始 BFS 写法可能包含：

```python
while queue:
    for _ in range(len(queue)):
        ...
```

这种写法常见于：

- 二叉树层序遍历；
- 求每层最大值；
- 求每层平均值；
- 需要记录具体层数的问题。

但是本题只需要判断节点对是否互为镜像，并不关心：

```text
当前是第几层
```

因此可以简化为：

```python
while queue:
    left, right = queue.popleft()
```

这样代码会更加直接。

------

# 面试中的思考方式

看到“判断一棵树是否对称”，可以先把问题转换成：

> 如何判断两棵树是否互为镜像？

假设函数为：

```python
mirror(left, right)
```

那么镜像关系自然有：

```text
mirror(left, right)
=
left.val == right.val

AND

mirror(left.left, right.right)

AND

mirror(left.right, right.left)
```

再补充递归终止条件：

```text
两个都为空 → True

只有一个为空 → False
```

这样整个递归算法几乎就已经得到了。

------

# 推荐记忆模板

二叉树中经常会遇到这种“比较两棵子树”的题。

可以记住下面这个模板：

```python
def dfs(node1, node2):
    # 两个节点都为空
    if not node1 and not node2:
        return True

    # 一个为空，一个不为空
    if not node1 or not node2:
        return False

    # 比较当前节点 + 递归比较子树
    return (
        条件
        and dfs(...)
        and dfs(...)
    )
```

对于“相同二叉树（Same Tree）”，递归关系通常是：

```python
dfs(p.left, q.left)
dfs(p.right, q.right)
```

对于“对称二叉树（Symmetric Tree）”，递归关系则变成：

```python
dfs(left.left, right.right)
dfs(left.right, right.left)
```

两道题的核心区别就是：

```text
Same Tree:
左 ↔ 左
右 ↔ 右

Symmetric Tree:
左 ↔ 右
右 ↔ 左
```

------

# 推荐面试答案

如果面试官没有特殊要求，优先使用递归 DFS：

```python
class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True

        def dfs(left, right):
            if not left and not right:
                return True

            if not left or not right:
                return False

            return (
                left.val == right.val
                and dfs(left.left, right.right)
                and dfs(left.right, right.left)
            )

        return dfs(root.left, root.right)
```

核心只需要牢牢记住一句：

> **判断对称树，就是不断比较处于镜像位置上的两个节点：外侧对外侧，内侧对内侧。**
