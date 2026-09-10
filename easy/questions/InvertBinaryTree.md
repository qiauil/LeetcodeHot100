# Invert Binary Tree（翻转二叉树）

给定一棵二叉树的根节点 `root`，请将这棵二叉树进行**翻转（invert / mirror）**，并返回翻转后的根节点。

所谓翻转二叉树，就是对树中的**每一个节点**，交换它的左子节点和右子节点。

例如：

```text
原始二叉树：               翻转后：

        4                     4
      /   \                 /   \
     2     7               7     2
    / \   / \             / \   / \
   1   3 6   9           9   6 3   1
```

这道题的核心操作其实非常简单：

```python
node.left, node.right = node.right, node.left
```

真正需要考虑的是：**如何遍历整棵树，保证每个节点都执行一次这个交换操作。**

常见做法有三种：

1. BFS（广度优先搜索）
2. 递归 DFS（深度优先搜索）
3. 迭代 DFS（使用显式栈）

------

# 1. BFS：广度优先搜索

## 思路

BFS（Breadth-First Search）按照**从上到下、逐层遍历**的方式访问二叉树。

因此我们可以：

1. 从根节点开始。
2. 取出当前节点。
3. 交换当前节点的左右子节点。
4. 将它的左右子节点加入队列。
5. 不断重复，直到队列为空。

因为每一个节点都会恰好被访问一次，所以最终整棵树都会被翻转。

这里其实不需要特别关心节点属于哪一层，因为我们的目标只是确保：

> 每一个节点都进行一次 `left / right` 交换。

------

## 算法步骤

1. 如果 `root` 为空，直接返回 `None`。
2. 创建一个队列，并将 `root` 放入队列。
3. 当队列不为空时：
   - 从队列头部取出一个节点 `node`。
   - 交换 `node.left` 和 `node.right`。
   - 如果新的 `node.left` 存在，将它加入队列。
   - 如果新的 `node.right` 存在，将它加入队列。
4. 所有节点处理完成后，返回 `root`。

------

## Python 实现

```python
from collections import deque
from typing import Optional

# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        # 空树不需要翻转
        if not root:
            return None

        # BFS 使用队列
        queue = deque([root])

        while queue:
            # 从队列左侧取出当前节点
            node = queue.popleft()

            # 交换当前节点的左右子节点
            node.left, node.right = node.right, node.left

            # 将交换后的左右子节点加入队列
            if node.left:
                queue.append(node.left)

            if node.right:
                queue.append(node.right)

        return root
```

------

## `collections.deque` 说明

这里使用：

```python
from collections import deque
```

`deque` 是 Python 标准库中的**双端队列（double-ended queue）**。

在 BFS 中通常需要不断执行：

```python
queue.popleft()
```

也就是从队列头部删除元素。

使用 `deque` 时：

```python
queue.popleft()
```

时间复杂度为：

```text
O(1)
```

而如果使用普通 Python `list`：

```python
queue.pop(0)
```

由于后面的元素需要整体向前移动，时间复杂度是：

```text
O(n)
```

因此在 Python 中实现 BFS 时，通常应该优先使用：

```python
deque
```

而不是：

```python
list + pop(0)
```

------

## 复杂度分析

假设二叉树共有 `n` 个节点。

### 时间复杂度

```text
O(n)
```

每个节点只会进入队列一次、离开队列一次，并执行一次左右子节点交换。

------

### 空间复杂度

```text
O(w)
```

其中 `w` 是二叉树的最大宽度。

因为 BFS 的队列最多可能同时保存某一层的大量节点。

最坏情况下：

```text
O(n)
```

例如对于一棵较完整的二叉树，最后一层可能包含大约一半的节点。

------

# 2. 递归 DFS：深度优先搜索

## 思路

翻转一棵二叉树，可以递归地理解为：

> 翻转当前节点，然后翻转它的左子树和右子树。

因为一棵二叉树的左右子树本身仍然是二叉树，所以这个问题具有非常自然的递归结构。

对于任意一个节点：

```text
1. 交换 left 和 right
2. 翻转新的 left subtree
3. 翻转新的 right subtree
```

例如：

```text
        root
       /    \
     left   right
```

交换之后：

```text
        root
       /    \
    right   left
```

然后再分别递归处理这两棵子树。

------

## 递归的两个关键部分

大多数二叉树递归题都可以从两个方面思考。

### 1. Base Case

什么时候停止递归？

如果当前节点为空：

```python
if not root:
    return None
```

说明已经走到了树的末端。

------

### 2. Recursive Case

当前节点应该做什么？

这道题只有两个动作：

```python
root.left, root.right = root.right, root.left
```

然后：

```python
self.invertTree(root.left)
self.invertTree(root.right)
```

------

## Python 实现

```python
from typing import Optional

# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        # Base Case：
        # 当前节点为空，说明已经到达子树末端
        if not root:
            return None

        # 交换当前节点的左右子节点
        root.left, root.right = root.right, root.left

        # 递归翻转交换之后的左子树
        self.invertTree(root.left)

        # 递归翻转交换之后的右子树
        self.invertTree(root.right)

        return root
```

------

## 为什么交换之后再递归不会有问题？

交换之前：

```text
root.left  = A
root.right = B
```

执行：

```python
root.left, root.right = root.right, root.left
```

之后：

```text
root.left  = B
root.right = A
```

因此接下来：

```python
self.invertTree(root.left)
self.invertTree(root.right)
```

实际上就是分别翻转原来的：

```text
B 子树
A 子树
```

两棵子树最终都会被处理，所以没有问题。

------

## Python 多变量交换

这行代码：

```python
root.left, root.right = root.right, root.left
```

是 Python 中非常常见的交换写法。

等价于：

```python
temp = root.left
root.left = root.right
root.right = temp
```

但 Python 的写法更加简洁。

可以将它理解为右侧先形成一个临时组合：

```python
(root.right, root.left)
```

然后再分别赋值给：

```python
root.left
root.right
```

因此不会出现第一个赋值覆盖掉原始值的问题。

------

## 复杂度分析

### 时间复杂度

```text
O(n)
```

每个节点恰好被递归访问一次。

------

### 空间复杂度

更加准确地说：

```text
O(h)
```

其中 `h` 是树的高度，因为额外空间主要来自**递归调用栈**。

如果二叉树比较平衡：

```text
h = O(log n)
```

因此空间复杂度为：

```text
O(log n)
```

如果二叉树极度倾斜，例如：

```text
1
 \
  2
   \
    3
     \
      4
```

那么：

```text
h = n
```

最坏空间复杂度为：

```text
O(n)
```

所以面试中可以回答：

```text
Space: O(h), worst case O(n)
```

这比单纯说 `O(n)` 更准确。

------

# 3. Iterative DFS：迭代深度优先搜索

## 思路

递归 DFS 实际上依赖的是程序内部的：

```text
Call Stack（函数调用栈）
```

我们也可以自己创建一个：

```python
stack = []
```

手动模拟递归过程。

过程如下：

```text
把 root 放入 stack

while stack 不为空:
    取出一个节点
    交换左右子节点
    将它的子节点放入 stack
```

由于栈具有：

```text
Last In, First Out
后进先出（LIFO）
```

因此程序会沿着某一条分支不断深入，这就是 DFS。

------

## 算法步骤

1. 如果 `root` 为空，返回 `None`。
2. 创建一个栈，将 `root` 放入其中。
3. 当栈不为空时：
   - 使用 `pop()` 取出栈顶节点。
   - 交换它的左右子节点。
   - 如果左子节点存在，将其压入栈。
   - 如果右子节点存在，将其压入栈。
4. 返回 `root`。

------

## Python 实现

```python
from typing import Optional

# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        # 空树直接返回
        if not root:
            return None

        # 使用 Python list 作为栈
        stack = [root]

        while stack:
            # pop() 删除并返回最后一个元素
            # 时间复杂度为 O(1)
            node = stack.pop()

            # 翻转当前节点
            node.left, node.right = node.right, node.left

            # 将子节点加入栈，之后继续处理
            if node.left:
                stack.append(node.left)

            if node.right:
                stack.append(node.right)

        return root
```

------

## Python `list` 为什么适合实现 Stack？

Python 的普通 `list` 非常适合作为栈使用：

```python
stack.append(x)
```

表示压栈（push）。

```python
stack.pop()
```

表示弹出栈顶元素（pop）。

这两个操作通常都是：

```text
O(1)
```

因此 DFS 中可以直接使用：

```python
stack = []
```

没有必要使用额外的数据结构。

------

## 一个容易混淆的细节：左右节点的压栈顺序

当前代码：

```python
if node.left:
    stack.append(node.left)

if node.right:
    stack.append(node.right)
```

因为 stack 是后进先出，所以：

```text
right 后进入
right 先处理
```

也就是说实际遍历顺序更加接近：

```text
root → right → left
```

但这道题中：

> **访问左边还是右边的先后顺序完全不影响最终结果。**

因为每一个节点只需要独立地完成一次左右交换。

如果一道 DFS 题要求：

```text
root → left → right
```

那么通常需要反过来压栈：

```python
stack.append(node.right)
stack.append(node.left)
```

因为：

```text
left 最后压入
left 最先弹出
```

这是迭代 DFS 中非常常见的面试细节。

------

## 复杂度分析

### 时间复杂度

```text
O(n)
```

每个节点只被访问一次。

------

### 空间复杂度

通常可以写成：

```text
O(h)
```

但具体取决于树的形状以及压栈顺序。

最坏情况下可能达到：

```text
O(n)
```

因此面试中写：

```text
Space: O(n) worst case
```

是安全的。

------

# 三种方法对比

| 方法          | 数据结构        | 时间复杂度 | 额外空间            | 特点       |
| ------------- | --------------- | ---------- | ------------------- | ---------- |
| BFS           | Queue           | `O(n)`     | `O(w)`，最坏 `O(n)` | 按层遍历   |
| Recursive DFS | Recursion Stack | `O(n)`     | `O(h)`，最坏 `O(n)` | 代码最简洁 |
| Iterative DFS | Stack           | `O(n)`     | 最坏 `O(n)`         | 避免递归   |

其中：

```text
n = 节点数量
h = 树的高度
w = 树的最大宽度
```

对于这道题来说，三种方法的核心逻辑完全一样：

```python
node.left, node.right = node.right, node.left
```

区别仅仅在于：

> **你用什么方式访问到树中的每一个节点。**

------

# 面试中推荐的解法

如果面试官没有限制，递归 DFS 通常是最自然、最容易表达的解法：

```python
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root:
            return None

        root.left, root.right = root.right, root.left

        self.invertTree(root.left)
        self.invertTree(root.right)

        return root
```

面试时可以简单解释：

```text
For every node, I swap its left and right children,
then recursively invert both subtrees.

Each node is visited exactly once, so the time complexity is O(n).
The recursion stack takes O(h) space, where h is the height of the tree.
```

对应中文思路：

```text
对于每个节点，我先交换它的左右子节点，
然后递归翻转它的左右子树。

每个节点只访问一次，因此时间复杂度是 O(n)。
递归栈的空间复杂度是 O(h)，其中 h 是树的高度。
```

------

# 常见错误与易混淆点

## 1. 忘记处理空树

必须考虑：

```python
root = None
```

因此递归 DFS 通常首先写：

```python
if not root:
    return None
```

否则后面访问：

```python
root.left
```

时就会发生错误。

------

## 2. 混淆交换前后的 `left` 和 `right`

执行：

```python
root.left, root.right = root.right, root.left
```

以后：

```text
root.left
```

已经是原来的右子树，而：

```text
root.right
```

已经是原来的左子树。

不过这道题中两边最终都需要递归，因此不会影响正确性。

------

## 3. 认为必须在交换之后才能将子节点加入 Stack / Queue

严格来说，这并不是必须的。

例如下面两种方式都可以正确遍历所有节点：

### 先交换，再加入

```python
node.left, node.right = node.right, node.left

stack.append(node.left)
stack.append(node.right)
```

### 先保存，再交换

```python
left = node.left
right = node.right

node.left, node.right = right, left

stack.append(left)
stack.append(right)
```

因为栈或队列保存的是**节点对象的引用**，并不会因为父节点的 `left` / `right` 指针发生交换而让节点消失。

真正需要保证的是：

> 左右两个子节点最终都被访问到。

因此原解析中“必须在交换之后压入，否则引用会错误”的说法并不准确。

------

## 4. 把 BFS 的空间复杂度简单理解成树高

BFS 保存的是：

```text
某一时刻待处理的节点
```

它的空间主要与树的**宽度**有关，而不是高度。

所以：

```text
BFS Space = O(w)
```

最坏：

```text
O(n)
```

------

## 5. 把递归 DFS 的空间复杂度永远写成 O(n)

更准确的表达是：

```text
O(h)
```

因为递归栈最多保存从根节点到当前节点的一条递归路径。

平衡树：

```text
O(log n)
```

极度倾斜的树：

```text
O(n)
```

这也是二叉树题中非常常见的复杂度分析方式。

------

# 核心总结

这道题本质上可以抽象为：

```text
遍历所有节点
+
对每个节点执行：
left ↔ right
```

因此重点并不在“翻转”操作本身，而在于熟悉二叉树的三种基本遍历框架：

### BFS 模板

```python
queue = deque([root])

while queue:
    node = queue.popleft()

    # process node

    if node.left:
        queue.append(node.left)

    if node.right:
        queue.append(node.right)
```

### Recursive DFS 模板

```python
def dfs(node):
    if not node:
        return

    # process node

    dfs(node.left)
    dfs(node.right)
```

### Iterative DFS 模板

```python
stack = [root]

while stack:
    node = stack.pop()

    # process node

    if node.left:
        stack.append(node.left)

    if node.right:
        stack.append(node.right)
```

掌握这三个模板之后，大量二叉树面试题实际上只是改变：

```text
# process node
```

这一部分的具体逻辑。
