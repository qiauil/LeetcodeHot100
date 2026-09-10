# 二叉树的层序遍历（Binary Tree Level Order Traversal）

给定一棵二叉树的根节点 `root`，返回该二叉树的**层序遍历**结果。

返回结果是一个二维列表，其中每个子列表包含树中**同一层的所有节点值**，并且节点按照**从左到右**的顺序排列。

例如，对于下面的二叉树：

```
        3
       / \
      9   20
         /  \
        15   7
```

层序遍历结果为：

```
[
    [3],
    [9, 20],
    [15, 7]
]
```

------

## 方法一：深度优先搜索（DFS）

### 核心思路

虽然题目要求的是**按层遍历**，但并不意味着一定要使用 BFS。

DFS 会沿着一条路径不断向下搜索，因此我们只需要在递归过程中额外记录：

> 当前节点位于二叉树的第几层 `depth`。

然后让：

```
res[depth]
```

专门保存第 `depth` 层的节点。

例如：

```
        3          depth = 0
       / \
      9   20       depth = 1
         /  \
        15   7     depth = 2
```

最终：

```
res[0] = [3]
res[1] = [9, 20]
res[2] = [15, 7]
```

DFS 每访问一个节点，就把它放进对应深度的列表中。

------

### 一个关键细节：为什么 `len(res) == depth`？

代码中有这样一个判断：

```
if len(res) == depth:
    res.append([])
```

这是 DFS 解法中最值得理解的地方。

假设：

```
res = [
    [3],
    [9, 20]
]
```

此时：

```
len(res) == 2
```

意味着目前只创建了：

```
depth = 0
depth = 1
```

两层。

如果 DFS 第一次访问：

```
depth = 2
```

就会发现：

```
len(res) == depth
```

因此说明：

> 我们第一次到达这一层，需要为这一层创建一个新的列表。

于是：

```
res.append([])
```

之后才能：

```
res[depth].append(node.val)
```

------

### 算法步骤

1. 创建结果列表 `res`。
2. 定义递归函数 `dfs(node, depth)`。
3. 如果 `node` 为空，直接返回。
4. 如果当前 `depth` 对应的层还没有被创建，则向 `res` 添加一个空列表。
5. 将当前节点值加入：

```
res[depth]
```

1. 递归访问左子树，深度变为 `depth + 1`。
2. 递归访问右子树，深度变为 `depth + 1`。
3. 从：

```
dfs(root, 0)
```

开始遍历。

9. 返回 `res`。

------

### Python 实现

```
# 二叉树节点的定义
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val        # 当前节点的值
#         self.left = left      # 左子节点
#         self.right = right    # 右子节点


class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        # res[d] 用来保存第 d 层的所有节点
        res = []

        def dfs(node, depth):
            # 递归终止条件
            if not node:
                return

            # 第一次访问这一层时，
            # 为这一层创建一个新的列表
            if len(res) == depth:
                res.append([])

            # 将当前节点加入对应层
            res[depth].append(node.val)

            # 继续访问下一层
            # 先左后右可以保证同一层节点从左到右排列
            dfs(node.left, depth + 1)
            dfs(node.right, depth + 1)

        # 根节点位于第 0 层
        dfs(root, 0)

        return res
```

### 为什么 DFS 也能保证从左到右？

因为递归顺序是：

```
dfs(node.left, depth + 1)
dfs(node.right, depth + 1)
```

也就是**永远先访问左子树，再访问右子树**。

因此，对于同一个 `depth`，左侧节点会先被加入：

```
res[depth]
```

右侧节点随后加入，所以最终仍然满足从左到右的顺序。

------

### 时间复杂度

**O(n)**

其中 `n` 是二叉树节点总数。

每个节点恰好访问一次，因此：

```
O(n)
```

------

### 空间复杂度

**O(n)**

这里可以分成两部分理解：

- 返回结果 `res` 本身需要 `O(n)` 空间。
- DFS 递归调用栈需要 `O(h)` 空间，其中 `h` 是树的高度。

因此整体空间复杂度为：

```
O(n)
```

如果面试官问**辅助空间（auxiliary space）**，则递归栈是：

```
O(h)
```

对于平衡二叉树：

```
h = O(log n)
```

对于完全退化成链表的二叉树：

```
h = O(n)
```

------

# 方法二：广度优先搜索（BFS）——更经典的层序遍历

## 核心思路

BFS（Breadth First Search，广度优先搜索）天然适合解决层序遍历问题，因为 BFS 本身就是：

> 一层一层地向外扩展。

这里需要使用一个**队列（Queue）**。

基本过程：

```
当前层节点
    ↓
从队列取出
    ↓
记录节点值
    ↓
把它们的孩子加入队列
    ↓
下一层节点
```

例如：

```
        3
       / \
      9   20
         /  \
        15   7
```

队列变化大致如下：

```
[3]

取出 3
加入 9, 20

[9, 20]

取出 9, 20
加入 15, 7

[15, 7]
```

因此队列天然保存着接下来需要访问的节点。

------

## 为什么需要 `qLen`？

这是 BFS 解法最重要的细节。

假设当前队列：

```
q = [9, 20]
```

这两个节点属于**同一层**。

所以我们首先记录：

```
qLen = len(q)
```

此时：

```
qLen = 2
```

然后只处理这两个节点：

```
for _ in range(qLen):
```

在处理它们的时候，新加入的 `15` 和 `7` 属于**下一层**，不能在当前循环里继续处理。

也就是说：

> `qLen` 相当于给当前这一层划了一条边界。

这是 BFS 层序遍历的核心技巧。

------

## 算法步骤

1. 如果 `root` 为空，返回 `[]`。
2. 创建队列 `q`，并将根节点加入队列。
3. 当队列不为空时：
   - 使用 `len(q)` 记录当前层节点数量。
   - 创建 `level` 保存当前层节点。
   - 恰好处理当前层的所有节点。
   - 将每个节点的左右孩子加入队列。
4. 将 `level` 加入最终结果。
5. 重复以上过程直到队列为空。
6. 返回结果。

------

## 推荐的 Python 实现

原答案把 `None` 节点也加入了队列：

```
q.append(node.left)
q.append(node.right)
```

然后取出之后再判断：

```
if node:
```

这种写法可以工作，但会让队列中出现不必要的 `None`。

面试中更推荐只把**真实存在的节点**加入队列：

```
from collections import deque


class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        # 空树
        if not root:
            return []

        res = []

        # deque 用作 BFS 队列
        q = deque([root])

        while q:
            # 当前队列中的节点数量
            # 就是当前这一层的节点数量
            level_size = len(q)

            # 保存当前层节点
            level = []

            # 只处理当前这一层
            for _ in range(level_size):
                # 从队列左侧取出节点
                node = q.popleft()

                level.append(node.val)

                # 将下一层节点加入队列
                if node.left:
                    q.append(node.left)

                if node.right:
                    q.append(node.right)

            # 当前层处理完毕
            res.append(level)

        return res
```

------

# Python `collections.deque` 说明

BFS 中经常使用：

```
from collections import deque
```

`deque` 全称是：

> double-ended queue，双端队列。

它允许我们高效地从两端添加或删除元素。

这里主要使用两个操作：

```
q.append(node)
```

将元素加入队列**右侧**。

以及：

```
node = q.popleft()
```

从队列**左侧**删除并返回一个元素。

所以：

```
append()
                  ↓
[ A, B, C ] ← new node
  ↑
popleft()
```

这正好符合普通队列：

> First In, First Out（FIFO，先进先出）

的特点。

### 为什么不用 Python `list.pop(0)`？

虽然也可以写：

```
node = q.pop(0)
```

但不推荐。

对于 Python `list`：

```
pop(0)
```

删除第一个元素之后，后面的所有元素都需要向前移动，因此时间复杂度是：

```
O(n)
```

而 `deque.popleft()` 的时间复杂度是：

```
O(1)
```

因此在 Python 中实现 BFS 时：

```
collections.deque
```

几乎是标准选择。

------

# BFS 时间与空间复杂度

### 时间复杂度：O(n)

每个节点：

- 进入队列一次；
- 离开队列一次；
- 被处理一次。

因此：

```
O(n)
```

### 空间复杂度：O(n)

队列最多可能同时保存一整层节点。

在一棵较为完整的二叉树中，最底层可能包含大约：

```
n / 2
```

个节点，因此最坏情况下：

```
O(n)
```

如果更精确地描述 BFS 的辅助空间，可以写成：

```
O(w)
```

其中 `w` 是二叉树的**最大宽度（maximum width）**。

因为：

```
w ≤ n
```

所以最坏情况仍然是：

```
O(n)
```

------

# DFS 与 BFS 对比

|                  | DFS        | BFS                 |
| ---------------- | ---------- | ------------------- |
| 核心数据结构     | 递归调用栈 | Queue               |
| 主要额外信息     | `depth`    | 当前层大小 `len(q)` |
| 时间复杂度       | O(n)       | O(n)                |
| 辅助空间         | O(h)       | O(w)                |
| 是否天然按层遍历 | 否         | **是**              |
| 代码思路         | 按深度归类 | 一层一层处理        |
| 面试推荐程度     | 很好       | **最经典**          |

其中：

```
h = height of tree       树的高度
w = maximum width        树的最大宽度
```

对于这道题，**BFS 是最自然、最标准的解法**。如果在代码面试中遇到“Level Order Traversal”，建议首先想到：

```
BFS
→ Queue
→ len(queue) 确定当前层大小
→ for 循环处理这一层
→ children 加入队列
```

这是一个非常值得形成条件反射的二叉树模板。

------

# 常见错误

## 1. BFS 没有区分不同层

错误思路可能只是不断：

```
while q:
    node = q.popleft()
```

然后直接记录节点。

这样虽然能够得到：

```
[3, 9, 20, 15, 7]
```

却无法得到题目要求的：

```
[
    [3],
    [9, 20],
    [15, 7]
]
```

解决方法就是在每轮 `while` 开始时：

```
level_size = len(q)
```

然后：

```
for _ in range(level_size):
```

只处理当前这一层。

------

## 2. 在处理当前层时动态使用 `len(q)`

不要写成类似：

```
for _ in range(len(q)):
```

然后误以为循环中的 `len(q)` 会持续代表当前层。

更清晰、更安全的模板是先固定：

```
level_size = len(q)

for _ in range(level_size):
    ...
```

因为在循环过程中我们会不断：

```
q.append(node.left)
q.append(node.right)
```

队列长度会发生变化，而新加入的节点属于**下一层**。

------

## 3. 忘记处理空树

如果：

```
root = None
```

正确结果应该是：

```
[]
```

因此 BFS 推荐一开始就写：

```
if not root:
    return []
```

DFS 中因为：

```
if not node:
    return
```

已经自然处理了空树，所以最终也会返回：

```
[]
```

------

## 4. 使用 `list.pop(0)` 实现 BFS

不要优先使用：

```
q = [root]
node = q.pop(0)
```

因为 `pop(0)` 是 `O(n)`。

Python BFS 更标准的写法是：

```
from collections import deque

q = deque([root])
node = q.popleft()
```

其中 `popleft()` 是 `O(1)`。

------

# 面试速记

看到：

```
Binary Tree
+
Level Order
```

优先想到：

```
BFS + Queue
```

核心模板可以压缩成：

```
q = deque([root])

while q:
    level = []

    for _ in range(len(q)):
        node = q.popleft()

        level.append(node.val)

        if node.left:
            q.append(node.left)

        if node.right:
            q.append(node.right)

    res.append(level)
```

真正需要记住的不是完整代码，而是：

```
while queue:
    当前层大小 = len(queue)

    for 当前层的每个节点:
        取出节点
        处理节点
        加入它的 children
```

**一句话总结：**

> BFS 用 `queue` 保存“接下来要访问的节点”，用 `len(queue)` 在每轮开始时锁定“当前这一层”；DFS 则用 `depth` 把节点放进对应的 `res[depth]`。