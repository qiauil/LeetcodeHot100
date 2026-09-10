# 二叉树的最大深度（Maximum Depth of Binary Tree）

给定一棵二叉树的根节点 `root`，返回这棵二叉树的**最大深度**。

二叉树的最大深度定义为：

> 从根节点出发，到最远叶子节点的最长路径上所包含的**节点数量**。

例如：

```text
        3
       / \
      9  20
         / \
        15  7
```

最长路径可以是：

```text
3 -> 20 -> 15
```

包含 `3` 个节点，因此最大深度为：

```text
3
```

------

# 1. 递归 DFS

## 思路

这道题最自然的解法是使用**递归深度优先搜索（DFS）**。

对于任意一个节点：

```text
当前节点所在子树的最大深度
=
1 + max(左子树最大深度, 右子树最大深度)
```

这里的 `1` 代表**当前节点本身**。

如果当前节点是 `None`，说明这是一棵空树，它的深度为：

```text
0
```

因此递归关系可以写成：

```text
maxDepth(root)
=
0                                           root is None
1 + max(maxDepth(root.left),
        maxDepth(root.right))               otherwise
```

例如：

```text
        1
       / \
      2   3
     /
    4
```

从节点 `1` 开始：

```text
maxDepth(1)
= 1 + max(maxDepth(2), maxDepth(3))

maxDepth(2)
= 1 + max(maxDepth(4), maxDepth(None))
= 1 + max(1, 0)
= 2

maxDepth(3)
= 1

所以：

maxDepth(1)
= 1 + max(2, 1)
= 3
```

------

## 算法步骤

1. 如果 `root` 为 `None`，返回 `0`。

2. 递归计算左子树最大深度：

   ```python
   left_depth = self.maxDepth(root.left)
   ```

3. 递归计算右子树最大深度：

   ```python
   right_depth = self.maxDepth(root.right)
   ```

4. 返回：

   ```python
   1 + max(left_depth, right_depth)
   ```

------

## 代码

```python
# Definition for a binary tree node.
# 二叉树节点定义
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        # Base case:
        # 空树没有任何节点，因此最大深度为 0
        if not root:
            return 0

        # 分别递归计算左右子树的最大深度
        left_depth = self.maxDepth(root.left)
        right_depth = self.maxDepth(root.right)

        # 当前节点本身贡献一层，因此需要 +1
        return 1 + max(left_depth, right_depth)
```

也可以简写成：

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        return 1 + max(
            self.maxDepth(root.left),
            self.maxDepth(root.right)
        )
```

------

## 复杂度分析

设：

- `n` 为树中的节点总数
- `h` 为树的高度

### 时间复杂度

```text
O(n)
```

因为每个节点只会被访问一次。

### 空间复杂度

```text
O(h)
```

这里的空间主要来自**递归调用栈**。

对于平衡二叉树：

```text
h = O(log n)
```

因此空间复杂度为：

```text
O(log n)
```

对于极端退化的二叉树，例如：

```text
1
 \
  2
   \
    3
     \
      4
```

树实际上退化成了链表，此时：

```text
h = n
```

空间复杂度为：

```text
O(n)
```

------

## 面试中如何理解这个递归

这类二叉树递归题通常都可以使用一个非常重要的思考方式：

> **先假设递归函数已经能够正确解决左右子树的问题，然后思考当前节点应该如何利用左右子树的答案。**

在这里：

```python
self.maxDepth(root.left)
```

可以理解为：

> “假设它已经帮我算出了左子树最大深度。”

同样：

```python
self.maxDepth(root.right)
```

代表右子树最大深度。

当前节点需要做的事情只有：

```python
1 + max(left_depth, right_depth)
```

这也是很多二叉树递归题的核心模式：

```text
解决当前节点
=
解决左子树
+
解决右子树
+
合并结果
```

------

# 2. 迭代 DFS：使用栈

## 思路

递归 DFS 本质上依赖的是 Python 的**函数调用栈**。

我们也可以自己维护一个显式的 `stack`，从而将递归 DFS 改成迭代 DFS。

但是这里仅仅保存节点还不够，因为题目要求最大深度。

因此栈中的每个元素同时保存：

```text
(node, depth)
```

也就是：

- 当前节点
- 当前节点所在的深度

例如：

```python
(root, 1)
```

表示根节点的深度为 `1`。

每次从栈中取出一个节点：

1. 更新当前最大深度。
2. 将左孩子压入栈，深度变为 `depth + 1`。
3. 将右孩子压入栈，深度变为 `depth + 1`。

------

## 算法步骤

1. 初始化栈：

   ```python
   stack = [(root, 1)]
   ```

2. 初始化答案：

   ```python
   res = 0
   ```

3. 当栈不为空时：

   - 弹出 `(node, depth)`
   - 如果节点存在：
     - 更新最大深度
     - 将左右孩子以及 `depth + 1` 压入栈

4. 返回 `res`。

------

## 代码

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        # 栈中保存：
        # (节点, 该节点所在的深度)
        stack = [(root, 1)]

        # 当前找到的最大深度
        res = 0

        while stack:
            # DFS 使用栈，因此从末尾弹出元素
            node, depth = stack.pop()

            # 由于代码允许把 None 放进栈，
            # 所以取出来之后需要检查
            if node:
                # 更新目前见过的最大深度
                res = max(res, depth)

                # 左右子节点的深度都是当前深度 + 1
                stack.append((node.left, depth + 1))
                stack.append((node.right, depth + 1))

        return res
```

------

## 一个稍微更干净的版本

原代码会把：

```python
None
```

也压入栈中，然后取出来以后再判断。

也可以先处理空树，然后只把真正存在的节点加入栈：

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        stack = [(root, 1)]
        max_depth = 0

        while stack:
            node, depth = stack.pop()

            max_depth = max(max_depth, depth)

            # 只把真正存在的孩子加入栈
            if node.left:
                stack.append((node.left, depth + 1))

            if node.right:
                stack.append((node.right, depth + 1))

        return max_depth
```

这种写法通常更容易理解。

------

## Python：`list.pop()`

这里使用：

```python
node, depth = stack.pop()
```

Python 的：

```python
list.pop()
```

默认会删除并返回 list 的**最后一个元素**。

例如：

```python
stack = [1, 2, 3]

x = stack.pop()

print(x)
# 3

print(stack)
# [1, 2]
```

因此：

```python
append()
```

配合：

```python
pop()
```

就可以把 Python 的 `list` 当成一个**栈（Stack）**使用：

```text
Last In, First Out
LIFO
后进先出
```

------

## 复杂度分析

### 时间复杂度

```text
O(n)
```

每个节点最多访问一次。

### 空间复杂度

最坏情况：

```text
O(n)
```

不过更精确地说，DFS 栈的大小和树的结构有关。

对于平衡树，栈通常只需要保存：

```text
O(h)
```

级别的节点，其中：

```text
h = O(log n)
```

但使用这种写法分析面试复杂度时，一般写最坏情况：

```text
O(n)
```

是完全可以接受的。

------

# 3. BFS：层序遍历

## 思路

还有一种非常直观的思路：使用**广度优先搜索（Breadth-First Search, BFS）**。

BFS 的特点是：

> 一层一层地遍历二叉树。

例如：

```text
        1          <- 第 1 层
       / \
      2   3        <- 第 2 层
     / \
    4   5          <- 第 3 层
```

如果我们能够统计一共处理了多少层，那么：

```text
层数 = 最大深度
```

因此：

> 每处理完整的一层，就让 `level += 1`。

当队列为空时，说明整棵树已经遍历完成，此时 `level` 就是最大深度。

------

## 算法步骤

1. 如果 `root` 为空，最终返回 `0`。

2. 创建队列 `q`。

3. 将根节点加入队列。

4. 初始化：

   ```python
   level = 0
   ```

5. 当队列不为空时：

   - 使用 `len(q)` 获取**当前层的节点数量**

   - 恰好处理这些节点

   - 将它们的孩子加入队列

   - 当前层处理完毕后：

     ```python
     level += 1
     ```

6. 返回 `level`。

------

## 代码

```python
from collections import deque

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        # BFS 通常使用 deque 作为队列
        q = deque()

        if root:
            q.append(root)

        level = 0

        while q:
            # len(q) 是当前这一层的节点数量
            # 必须先固定下来，因为遍历过程中还会加入下一层节点
            level_size = len(q)

            # 只处理当前层
            for _ in range(level_size):
                # 从队列左边取出节点
                node = q.popleft()

                # 将下一层节点加入队列右边
                if node.left:
                    q.append(node.left)

                if node.right:
                    q.append(node.right)

            # 当前一整层已经处理完成
            level += 1

        return level
```

------

# Python：`collections.deque`

BFS 中通常使用：

```python
from collections import deque
```

创建：

```python
q = deque()
```

然后：

```python
q.append(x)
```

从右边加入元素：

```text
[A, B] -> [A, B, X]
```

使用：

```python
q.popleft()
```

从左边删除并返回元素：

```text
[A, B, X]

popleft()

-> A
-> [B, X]
```

这正好符合队列：

```text
First In, First Out
FIFO
先进先出
```

------

## 为什么 BFS 不建议直接使用 Python `list`？

理论上你可以这样写：

```python
q = []

q.append(node)
node = q.pop(0)
```

但是：

```python
list.pop(0)
```

的时间复杂度是：

```text
O(n)
```

因为删除第一个元素之后，后面的元素都需要向前移动。

而：

```python
deque.popleft()
```

的时间复杂度是：

```text
O(1)
```

因此在 Python 中：

```text
DFS Stack → list
BFS Queue → deque
```

是非常常见的搭配。

------

# BFS 中 `for _ in range(len(q))` 为什么重要？

这一段代码：

```python
for _ in range(len(q)):
```

是层序遍历非常重要的写法。

假设当前队列中有：

```text
[2, 3]
```

也就是说当前层有两个节点。

因此：

```python
len(q) == 2
```

接下来处理节点 `2` 时，可能向队列中加入：

```text
4, 5
```

队列变成：

```text
[3, 4, 5]
```

然后处理节点 `3`。

注意：

```text
4 和 5 属于下一层
```

不能在当前循环中继续处理。

因此我们必须在进入 `for` 循环时固定当前层节点数。

例如：

```python
level_size = len(q)

for _ in range(level_size):
    ...
```

这是二叉树 BFS 层序遍历中非常重要的模板。

------

# 复杂度分析

## 时间复杂度

```text
O(n)
```

每个节点只会：

- 入队一次
- 出队一次

因此总时间复杂度为：

```text
O(n)
```

------

## 空间复杂度

```text
O(n)
```

BFS 的空间复杂度主要来自队列。

队列最多可能保存某一层的所有节点。

对于一棵完全二叉树，最后一层大约可能包含：

```text
n / 2
```

个节点，因此：

```text
O(n)
```

------

# 三种方法对比

| 方法     | 核心数据结构  | 时间复杂度 | 空间复杂度  | 特点             |
| -------- | ------------- | ---------- | ----------- | ---------------- |
| 递归 DFS | Python 调用栈 | `O(n)`     | `O(h)`      | 最简洁、最推荐   |
| 迭代 DFS | `list` 栈     | `O(n)`     | 最坏 `O(n)` | 避免递归         |
| BFS      | `deque` 队列  | `O(n)`     | `O(n)`      | 天然按层计算深度 |

对于这道题，面试中通常优先推荐：

```text
递归 DFS
```

因为它最直接地对应递归定义：

```text
树的最大深度
=
1 + max(左子树深度, 右子树深度)
```

------

# 常见错误

## 1. 混淆 Depth 和 Height

在树的相关题目中，经常会出现两个概念：

```text
Depth
Height
```

### Depth

通常表示：

> 从根节点到当前节点的距离。

例如：

```text
        A
       /
      B
     /
    C
```

如果按照节点数量计算：

```text
A depth = 1
B depth = 2
C depth = 3
```

有些教材也会按照**边的数量**计算：

```text
A depth = 0
B depth = 1
C depth = 2
```

因此具体题目中一定要看定义。

本题明确规定：

> 计算最长路径上的节点数量。

所以：

```text
空树深度 = 0
只有根节点 = 1
```

------

### Height

Height 一般表示：

> 从当前节点向下，到最远叶子节点的最长距离。

严格来说：

```text
depth
```

和：

```text
height
```

并不是完全相同的概念。

不过对于整棵树而言：

```text
tree maximum depth
```

和：

```text
root 的 height
```

在采用相同计数标准的情况下，数值是相同的。

面试中更重要的是保持定义一致。

------

## 2. 忘记处理空树

错误写法：

```python
def maxDepth(self, root):
    return 1 + max(
        self.maxDepth(root.left),
        self.maxDepth(root.right)
    )
```

问题在于：

当：

```python
root is None
```

时，代码会尝试访问：

```python
root.left
```

从而产生错误。

正确写法：

```python
def maxDepth(self, root):
    if not root:
        return 0

    return 1 + max(
        self.maxDepth(root.left),
        self.maxDepth(root.right)
    )
```

`if not root:` 是整个递归的 **base case（递归终止条件）**。

------

# 递归调用过程示例

假设：

```text
        1
       / \
      2   3
     /
    4
```

执行：

```python
maxDepth(1)
```

递归过程大致是：

```text
maxDepth(1)

├── maxDepth(2)
│   ├── maxDepth(4)
│   │   ├── maxDepth(None) → 0
│   │   └── maxDepth(None) → 0
│   │
│   │   maxDepth(4) → 1
│   │
│   └── maxDepth(None) → 0
│
│   maxDepth(2) → 2
│
└── maxDepth(3)
    ├── maxDepth(None) → 0
    └── maxDepth(None) → 0

    maxDepth(3) → 1
```

最终：

```text
maxDepth(1)
=
1 + max(2, 1)
=
3
```

注意递归真正计算结果的过程是：

```text
先一路向下
↓
遇到 None
↓
然后从下向上返回答案
```

因此这实际上也是一种非常典型的：

```text
Postorder / 后序思想
```

因为一个节点需要先知道：

```text
左子树结果
右子树结果
```

才能计算自己的答案。

------

# 面试重点总结

这道题本身比较简单，但它包含了几个非常重要的二叉树基础模板。

### 递归 DFS 模板

```python
def dfs(root):
    if not root:
        return base_value

    left = dfs(root.left)
    right = dfs(root.right)

    return combine(left, right)
```

在本题中：

```python
base_value = 0
```

而：

```python
combine(left, right)
```

就是：

```python
1 + max(left, right)
```

所以最终得到：

```python
def maxDepth(self, root):
    if not root:
        return 0

    return 1 + max(
        self.maxDepth(root.left),
        self.maxDepth(root.right)
    )
```

------

### BFS 层序遍历模板

```python
q = deque([root])

while q:
    level_size = len(q)

    for _ in range(level_size):
        node = q.popleft()

        if node.left:
            q.append(node.left)

        if node.right:
            q.append(node.right)
```

只要题目出现：

```text
level
层
每一层
最短层数
按层遍历
```

通常就应该想到 BFS。

------

### DFS 和 BFS 的选择

如果问题更像：

```text
“一个节点的答案如何由左右子树的答案得到？”
```

优先考虑：

```text
DFS / 递归
```

如果问题更像：

```text
“我要一层一层处理节点”
```

优先考虑：

```text
BFS
```

对于本题：

```text
maxDepth(node)
=
1 + max(maxDepth(left), maxDepth(right))
```

递归关系极其清晰，因此**递归 DFS 是面试中最值得优先掌握的解法**。
