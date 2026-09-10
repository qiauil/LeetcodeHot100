# Binary Tree Right Side View（二叉树的右视图）

给定一棵二叉树的根节点 `root`，返回从二叉树**右侧观察时能够看到的节点值**，结果按照**从上到下**的顺序排列。

例如：

```
        1
       / \
      2   3
       \   \
        5   4
```

从右侧观察，可以看到：

```
1 → 3 → 4
```

因此返回：

```
[1, 3, 4]
```

------

## 核心思路

这道题的本质是：

> **找到二叉树每一层最右侧的节点。**

可以使用两种经典方法：

1. **DFS（深度优先搜索）**
   - 优先访问右子树。
   - 每一层第一次访问到的节点，就是这一层最右侧的节点。
2. **BFS（广度优先搜索 / 层序遍历）**
   - 一层一层遍历。
   - 每一层最后访问到的节点，就是这一层最右侧的节点。

面试中两种方法都值得掌握。

------

# 方法一：DFS（深度优先搜索）

## 思路

如果我们从右侧观察二叉树，那么对于每一个深度 `depth`，我们只关心：

> **从右边开始搜索时，第一个遇到的节点。**

因此 DFS 时按照下面的顺序搜索：

```
当前节点
   ↓
右子树
   ↓
左子树
```

也就是：

```
Root → Right → Left
```

这样一来，**每进入一个新的深度时，第一个访问到的节点一定是这一层最靠右、能够被看到的节点。**

例如：

```
        1
       / \
      2   3
     /   /
    4   5
```

DFS 的访问顺序大致为：

```
1 → 3 → 5 → 2 → 4
```

第一次到达：

```
depth = 0 → 1
depth = 1 → 3
depth = 2 → 5
```

所以：

```
res = [1, 3, 5]
```

之后再访问 `2` 和 `4` 时，对应的深度已经记录过了，因此不会覆盖答案。

------

## 算法步骤

1. 创建空列表 `res`，保存最终右视图。

2. 定义 DFS 函数：

   ```
   dfs(node, depth)
   ```

3. 如果 `node` 为空，直接返回。

4. 如果：

   ```
   depth == len(res)
   ```

   说明这是我们

   第一次到达这个深度

   。

5. 将当前节点加入 `res`。

6. 优先递归搜索右子树。

7. 再递归搜索左子树。

8. 从：

   ```
   dfs(root, 0)
   ```

   开始搜索。

9. 返回 `res`。

------

## Python 实现

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        # res[i] 保存深度 i 能够从右侧看到的节点
        res = []

        def dfs(node, depth):
            # 到达空节点，结束递归
            if not node:
                return

            # 如果 depth == len(res)，说明这是第一次访问这一层
            # 因为我们优先访问右子树，所以当前节点就是这一层最右侧的节点
            if depth == len(res):
                res.append(node.val)

            # 一定要先访问右子树
            dfs(node.right, depth + 1)

            # 然后再访问左子树
            dfs(node.left, depth + 1)

        dfs(root, 0)

        return res
```

------

## 为什么 `depth == len(res)`？

这是这个 DFS 解法最重要的一行：

```
if depth == len(res):
    res.append(node.val)
```

假设：

```
res = [1, 3]
```

那么：

```
len(res) = 2
```

说明：

```
depth 0 已经找到答案
depth 1 已经找到答案
depth 2 还没有找到答案
```

因此，当：

```
depth == 2
```

时：

```
depth == len(res)
```

成立。

这意味着：

> 当前节点是 DFS 第一次访问到深度 `2` 的节点。

又因为 DFS 是：

```
右 → 左
```

所以它自然就是这一层最靠右的节点。

这个技巧在很多二叉树 DFS 题目中都非常有用：

> **`depth == len(result)` 可以用来判断是否第一次到达某个深度。**

------

## 时间复杂度

```
O(n)
```

其中 `n` 是二叉树节点数量。

每个节点最多被访问一次。

------

## 空间复杂度

```
O(h)
```

其中 `h` 是树的高度，主要来自递归调用栈。

最坏情况下，二叉树退化成链表：

```
1
 \
  2
   \
    3
     \
      ...
```

此时：

```
h = n
```

所以最坏空间复杂度为：

```
O(n)
```

如果树比较平衡，则：

```
h = log n
```

递归栈空间约为：

```
O(log n)
```

> 如果把最终返回的 `res` 也计入空间复杂度，那么答案本身最多可能包含 `O(n)` 个节点，因此总空间可以写成 `O(n)`。

------

# 方法二：BFS（广度优先搜索）

## 思路

BFS 会按照层级遍历二叉树：

```
第 0 层
第 1 层
第 2 层
...
```

如果每一层按照：

```
左 → 右
```

进行遍历，那么：

> **这一层最后访问到的节点，就是这一层最右侧的节点。**

例如：

```
        1
       / \
      2   3
     / \   \
    4   5   6
```

层序遍历：

```
Level 0: 1
Level 1: 2, 3
Level 2: 4, 5, 6
```

每一层最后一个节点：

```
1
3
6
```

所以：

```
[1, 3, 6]
```

------

## 算法步骤

1. 创建结果数组 `res`。
2. 创建队列 `q`，首先加入 `root`。
3. 当队列不为空时：
   - 使用 `len(q)` 得到当前层的节点数量。
   - 遍历这一层的所有节点。
   - 记录这一层最后一个有效节点。
   - 将节点的左右子节点加入队列。
4. 当前层处理结束后，将最右节点加入 `res`。
5. 返回 `res`。

------

## 原答案写法

```
from collections import deque


class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        res = []

        # deque 是双端队列
        q = deque([root])

        while q:
            rightSide = None

            # 当前这一层有多少个节点
            qLen = len(q)

            for i in range(qLen):
                # 从队列左侧取出节点
                node = q.popleft()

                if node:
                    # 因为按照从左到右遍历，
                    # rightSide 最终会停留在这一层最右边的节点
                    rightSide = node

                    # 将下一层节点加入队列
                    q.append(node.left)
                    q.append(node.right)

            # 当前层存在有效节点
            if rightSide:
                res.append(rightSide.val)

        return res
```

------

## 更推荐的 BFS 写法

上面的代码可以正确运行，不过它会把：

```
None
```

也加入队列：

```
q.append(node.left)
q.append(node.right)
```

因此代码中还需要：

```
if node:
```

来额外判断。

面试中我更推荐下面这种写法：

```
from collections import deque


class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        # 空树没有右视图
        if not root:
            return []

        res = []
        q = deque([root])

        while q:
            # 当前层的节点数量
            level_size = len(q)

            for i in range(level_size):
                node = q.popleft()

                # 如果是当前层最后一个节点，
                # 那么它就是这一层最右侧的节点
                if i == level_size - 1:
                    res.append(node.val)

                # 只把真实存在的节点加入队列
                if node.left:
                    q.append(node.left)

                if node.right:
                    q.append(node.right)

        return res
```

这版逻辑更加直接：

```
if i == level_size - 1:
    res.append(node.val)
```

明确表达了：

> 当前节点是这一层最后一个节点，因此它就是右视图节点。

------

# Python：`collections.deque`

BFS 中通常使用：

```
from collections import deque
```

然后：

```
q = deque()
```

`deque` 是 Python 提供的**双端队列（double-ended queue）**。

它支持：

```
q.append(x)      # 从右侧加入
q.appendleft(x)  # 从左侧加入

q.pop()          # 从右侧删除
q.popleft()      # 从左侧删除
```

BFS 最常见的组合就是：

```
q.append(node)   # 入队
q.popleft()      # 出队
```

### 为什么不用普通 `list`？

如果使用：

```
queue = []

queue.append(node)
queue.pop(0)
```

其中：

```
pop(0)
```

需要移动后面的元素，因此时间复杂度是：

```
O(n)
```

而：

```
deque.popleft()
```

时间复杂度是：

```
O(1)
```

因此在 Python 中实现 BFS 时：

> **优先使用 `collections.deque`。**

这是 Python 算法面试中非常常见的知识点。

------

# BFS 时间复杂度

每个节点只会：

- 入队一次；
- 出队一次。

因此：

```
O(n)
```

------

# BFS 空间复杂度

队列最多需要保存某一层的所有节点。

因此更加精确地说是：

```
O(w)
```

其中 `w` 是二叉树的最大宽度。

最坏情况下：

```
w = O(n)
```

所以通常写成：

```
O(n)
```

------

# DFS 与 BFS 对比

| 方法 | 核心思想                         | 时间复杂度 | 辅助空间 |
| ---- | -------------------------------- | ---------- | -------- |
| DFS  | 右子树优先，每层第一次访问的节点 | `O(n)`     | `O(h)`   |
| BFS  | 层序遍历，每层最后一个节点       | `O(n)`     | `O(w)`   |

其中：

```
h = tree height（树高）
w = maximum width（最大宽度）
```

对于这道题，我会更推荐优先掌握 **DFS 解法**，因为代码非常简洁，而且“**右优先 DFS + 每层第一次访问**”很好地体现了右视图的本质。

BFS 则更加直观：

> **右视图 = 每一层最右边的节点。**

------

# 常见错误

## 1. 只遍历右子节点

错误思路：

```
def dfs(node):
    if node:
        res.append(node.val)
        dfs(node.right)
```

右视图并不意味着：

> 一直沿着 `right` 指针走。

例如：

```
        1
       / \
      2   3
     /
    4
```

右子树在 `3` 之后结束了，但更深一层的 `4` 仍然能够从右侧看到。

正确答案：

```
[1, 3, 4]
```

因此 DFS 必须同时搜索：

```
dfs(node.right)
dfs(node.left)
```

只是**右子树优先**。

------

## 2. DFS 先遍历左子树，却仍然只记录第一次访问的节点

下面的逻辑：

```
if depth == len(res):
    res.append(node.val)
```

只有在：

```
Right → Left
```

的 DFS 顺序下，才能直接得到右视图。

如果改成：

```
dfs(node.left, depth + 1)
dfs(node.right, depth + 1)
```

那么每一层第一次访问到的节点会变成**最左侧节点**。

最终得到的实际上更接近：

```
Binary Tree Left Side View
```

------

## 3. BFS 忘记记录当前层节点数量

BFS 中这一句非常重要：

```
level_size = len(q)
```

它相当于给当前层划定一个边界。

例如：

```
queue = [2, 3]
```

说明这一轮只处理：

```
2, 3
```

即使处理 `2` 时又加入：

```
4, 5
```

这些属于**下一层**，不能在当前循环中继续处理。

因此通常使用：

```
level_size = len(q)

for i in range(level_size):
    node = q.popleft()
```

来确保 BFS 是严格按照层级进行处理的。

------

# 面试记忆模板

这道题可以记成两个非常简洁的模板。

### DFS

```
def dfs(node, depth):
    if not node:
        return

    # 第一次来到这一层
    if depth == len(res):
        res.append(node.val)

    # 右边优先
    dfs(node.right, depth + 1)
    dfs(node.left, depth + 1)
```

记忆：

```
Right First
+
First Node at Each Depth
=
Right Side View
```

### BFS

```
while q:
    level_size = len(q)

    for i in range(level_size):
        node = q.popleft()

        if i == level_size - 1:
            res.append(node.val)
```

记忆：

```
Level Order Traversal
+
Last Node of Each Level
=
Right Side View
```

------

# 总结

这道题最重要的不是记住代码，而是理解：

> **Binary Tree Right Side View 本质上是在寻找每一个深度最靠右的节点。**

因此有两个自然的观察角度：

```
DFS:
每层第一次从右边遇到的节点

BFS:
每层从左到右遍历时的最后一个节点
```

对应：

```
DFS → Right First + First at Depth
BFS → Level Order + Last at Level
```

这两个模式也可以迁移到很多类似的二叉树问题，例如 **Left Side View、每层最大值、层序遍历以及各种按深度收集节点的问题**。
