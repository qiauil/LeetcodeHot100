# 二叉树的最近公共祖先（Lowest Common Ancestor of a Binary Tree）

给定一棵二叉树，以及树中的两个节点 `p` 和 `q`，找到它们的**最近公共祖先**（Lowest Common Ancestor，LCA）。

根据最近公共祖先的定义：

> 节点 `p` 和 `q` 的最近公共祖先，是树 `T` 中同时以 `p` 和 `q` 为后代的**最深节点**。这里规定：**一个节点也可以是它自己的后代**。

这里的「最近 / Lowest」指的是**在树中的深度最大、距离 `p` 和 `q` 最近的公共祖先**，而不是节点值最小。

例如：

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

- `LCA(5, 1) = 3`
- `LCA(5, 4) = 5`

第二个例子非常重要：因为一个节点可以是自己的后代，所以当 `5` 本身就是 `4` 的祖先时，`5` 就是最近公共祖先。

------

# 解法一：DFS + 状态记录

## 核心思路

我们可以使用 DFS 遍历整棵树，并让每个节点向它的父节点汇报两个信息：

```
我的子树里有没有 p？
我的子树里有没有 q？
```

因此，对于每个节点 `node`，我们维护：

```
[found_p, found_q]
```

例如：

```
[True, False]
```

表示当前节点的子树中找到了 `p`，但没有找到 `q`。

对于当前节点来说：

```
found_p = 左子树找到p or 右子树找到p or 当前节点就是p
found_q = 左子树找到q or 右子树找到q or 当前节点就是q
```

如果：

```
found_p == True
found_q == True
```

那么说明当前节点的子树已经同时包含 `p` 和 `q`。

由于 DFS 是**后序遍历**：

```
左子树 → 右子树 → 当前节点
```

因此最先满足这个条件的节点，就是最深的那个节点，也就是 LCA。

## 算法步骤

1. 定义 `dfs(node)`，返回 `[found_p, found_q]`。
2. 如果 `node` 为空，返回 `[False, False]`。
3. 递归搜索左子树。
4. 递归搜索右子树。
5. 根据左右子树以及当前节点计算 `found_p` 和 `found_q`。
6. 如果两者都为 `True`，并且还没有找到 LCA，那么当前节点就是 LCA。
7. 最终返回记录下来的 `lca`。

## 代码

```
# 二叉树节点定义
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None


class Solution:
    def lowestCommonAncestor(
        self,
        root: 'TreeNode',
        p: 'TreeNode',
        q: 'TreeNode'
    ) -> 'TreeNode':

        lca = None

        def dfs(node):
            nonlocal lca

            # 空节点：既没有找到 p，也没有找到 q
            if node is None:
                return [False, False]

            # 如果已经找到 LCA，就不需要继续搜索
            if lca is not None:
                return [False, False]

            # 后序遍历：先搜索左右子树
            left = dfs(node.left)
            right = dfs(node.right)

            # 当前子树是否包含 p
            found_p = (
                left[0]
                or right[0]
                or node is p
            )

            # 当前子树是否包含 q
            found_q = (
                left[1]
                or right[1]
                or node is q
            )

            # 第一个同时包含 p 和 q 的节点就是 LCA
            if found_p and found_q and lca is None:
                lca = node

            return [found_p, found_q]

        dfs(root)

        return lca
```

## `nonlocal` 说明

这里有一个比较值得注意的 Python 关键字：

```
nonlocal lca
```

`lca` 定义在外层函数：

```
def lowestCommonAncestor(...):
    lca = None
```

而我们希望内部函数：

```
def dfs(node):
```

能够**修改**这个变量。

因此需要：

```
nonlocal lca
```

如果没有 `nonlocal`，执行：

```
lca = node
```

时，Python 会认为你是在 `dfs()` 内创建了一个新的局部变量 `lca`，而不是修改外层的 `lca`。

------

# 解法二：递归 DFS（最优写法，推荐掌握）

这是这道题最经典、最适合面试的解法。

相比第一种方法，它不需要：

- `found_p`
- `found_q`
- 全局 / `nonlocal` 的 `lca`

而是直接利用**递归函数的返回值**传递信息。

## 核心思路

定义：

```
lowestCommonAncestor(root, p, q)
```

它可以理解成：

> 「在以 `root` 为根的这棵子树中寻找 `p` 和 `q`，如果发现有意义的节点，就把它返回给父节点。」

首先处理最关键的递归终止条件：

```
if root is None or root is p or root is q:
    return root
```

也就是说：

- 如果走到空节点 → 返回 `None`
- 如果找到 `p` → 返回 `p`
- 如果找到 `q` → 返回 `q`

然后分别搜索左右子树：

```
left = self.lowestCommonAncestor(root.left, p, q)
right = self.lowestCommonAncestor(root.right, p, q)
```

接下来只有三种情况。

### 情况一：左右都找到了节点

```
left != None
right != None
```

说明：

```
        root
       /    \
   p/q      q/p
```

`p` 和 `q` 分别位于当前节点两侧。

因此：

```
root
```

就是最近公共祖先。

------

### 情况二：只有左边找到了

```
        root
       /
   p 和 q
```

此时：

```
left != None
right == None
```

我们不需要在当前节点做任何决定，只需要：

```
return left
```

把左边找到的结果继续向上传递。

------

### 情况三：只有右边找到了

同理：

```
return right
```

因此最后可以非常简洁地写成：

```
return left if left else right
```

## 代码

```
# 二叉树节点定义
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None


class Solution:
    def lowestCommonAncestor(
        self,
        root: 'TreeNode',
        p: 'TreeNode',
        q: 'TreeNode'
    ) -> 'TreeNode':

        # Base Case：
        # 1. 搜索到空节点
        # 2. 当前节点就是 p
        # 3. 当前节点就是 q
        #
        # 如果找到 p 或 q，直接将它向上传递
        if root is None or root is p or root is q:
            return root

        # 分别搜索左右子树
        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)

        # 左右两边都有结果：
        # 说明 p 和 q 分别位于当前节点两侧，
        # 所以当前节点就是 LCA
        if left is not None and right is not None:
            return root

        # 只有一边有结果：
        # 将非空结果继续向上传递
        return left if left is not None else right
```

## 为什么找到 `p` 或 `q` 就可以直接返回？

这是这道题最容易产生疑问的地方。

假设：

```
        5 (p)
       / \
      6   2
         / \
        7   4 (q)
```

这里：

```
p = 5
q = 4
```

当递归到 `5` 时：

```
root is p
```

所以直接：

```
return 5
```

我们甚至没有继续搜索 `5` 的子树。

为什么这样仍然正确？

因为题目保证 `p` 和 `q` 都存在于树中。

既然：

```
当前 root == p
```

那么无论 `q` 在 `p` 的子树中还是其他地方，都不存在比 `p` 更深、同时又是 `p` 自己祖先的节点。

如果 `q` 在 `p` 的子树中：

```
p
|
...
|
q
```

那么 LCA 必然就是 `p`。

所以遇到 `p` 或 `q` 时可以立即返回。

------

# 解法三：BFS + Parent Map

## 核心思路

二叉树的节点通常只有：

```
node.left
node.right
```

也就是说，我们可以从父节点走到子节点，却不能直接：

```
node.parent
```

从子节点回到父节点。

我们可以通过 BFS 遍历整棵树，并人为建立：

```
child -> parent
```

的映射。

例如：

```
        3
       / \
      5   1
```

建立：

```
parent[3] = None
parent[5] = 3
parent[1] = 3
```

有了这个 `parent` 哈希表之后，问题就变得类似于：

> 给定两个节点，从它们不断向父节点移动，找到第一个公共节点。

## 算法步骤

1. 使用 BFS 遍历二叉树。
2. 使用哈希表 `parent` 保存每个节点的父节点。
3. 找到 `p` 和 `q` 后即可停止 BFS。
4. 从 `p` 开始不断向上移动，将 `p` 的所有祖先加入 `ancestors` 集合。
5. 从 `q` 开始不断向上移动。
6. `q` 遇到的第一个属于 `ancestors` 的节点，就是 LCA。

## 代码

```
from collections import deque


# 二叉树节点定义
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None


class Solution:
    def lowestCommonAncestor(
        self,
        root: 'TreeNode',
        p: 'TreeNode',
        q: 'TreeNode'
    ) -> 'TreeNode':

        # parent[node] 表示 node 的父节点
        # 根节点没有父节点
        parent = {
            root: None
        }

        # BFS 队列
        queue = deque([root])

        # 直到 p 和 q 都已经被发现
        while p not in parent or q not in parent:

            node = queue.popleft()

            if node.left is not None:
                parent[node.left] = node
                queue.append(node.left)

            if node.right is not None:
                parent[node.right] = node
                queue.append(node.right)

        # 保存 p 到 root 路径上的所有节点
        ancestors = set()

        current = p

        while current is not None:
            ancestors.add(current)
            current = parent[current]

        # 从 q 开始向 root 移动
        current = q

        # 第一个同时属于 p 的祖先集合的节点
        # 就是最近公共祖先
        while current not in ancestors:
            current = parent[current]

        return current
```

## `deque` 说明

这里：

```
from collections import deque
```

`deque` 是 Python `collections` 模块提供的**双端队列**。

BFS 中通常推荐：

```
queue = deque()

queue.append(x)     # 从右边加入
queue.popleft()     # 从左边取出
```

其中：

```
popleft()
```

时间复杂度为：

```
O(1)
```

不建议使用普通 `list`：

```
queue = []
queue.pop(0)
```

因为 `pop(0)` 删除第一个元素后，需要移动后面的元素，时间复杂度为：

```
O(n)
```

因此在 Python 面试题中：

```
BFS → 优先想到 collections.deque
```

------

# 三种方法复杂度比较

设二叉树共有 `n` 个节点，高度为 `h`。

| 方法               | 时间复杂度 | 额外空间复杂度 |
| ------------------ | ---------- | -------------- |
| DFS + 两个 Boolean | `O(n)`     | `O(h)`         |
| 递归 DFS（推荐）   | `O(n)`     | `O(h)`         |
| BFS + Parent Map   | `O(n)`     | `O(n)`         |

这里原答案统一写成了：

```
Space Complexity: O(n)
```

作为最坏情况是正确的，但对于递归 DFS，更准确的表达是：

```
O(h)
```

因为额外空间主要来自**递归调用栈**。

如果树是平衡二叉树：

```
h = O(log n)
```

所以空间复杂度：

```
O(log n)
```

如果树极度倾斜：

```
1
 \
  2
   \
    3
     \
      ...
```

那么：

```
h = n
```

最坏空间复杂度就是：

```
O(n)
```

因此面试中可以说：

> Space complexity is `O(h)` due to the recursion stack, where `h` is the height of the tree. In the worst case, it becomes `O(n)`.

------

# 常见错误

## 1. 混淆节点引用和节点值

题目要求寻找的是**指定节点 `p` 和 `q`**的 LCA，而不是寻找：

```
p.val
q.val
```

对应的值。

因此推荐：

```
root is p
root is q
```

而不是：

```
root.val == p.val
root.val == q.val
```

因为理论上不同节点可能拥有相同的 `val`。

例如：

```
        3
       / \
      5   5
```

左右两个 `5` 是两个不同的 `TreeNode` 对象。

因此：

```
node.val == p.val
```

无法准确判断当前节点是不是 `p`。

------

## 2. 忘记「节点可以是自己的祖先」

假设：

```
        p
       /
      ...
     /
    q
```

那么：

```
LCA(p, q) = p
```

不是 `p` 的父节点。

这也是为什么最优递归解法中的：

```
if root is p or root is q:
    return root
```

如此重要。

------

## 3. 不理解递归返回值的含义

第二种方法最容易死记硬背：

```
left = dfs(root.left)
right = dfs(root.right)

if left and right:
    return root

return left or right
```

更好的理解方式是：

> `dfs(node)` 返回的是：**从当前子树中找到的、值得继续向上传递的节点。**

因此：

```
left=None, right=None
→ 什么都没找到

left=p, right=None
→ 左边找到一个目标，继续把 p 往上传

left=None, right=q
→ 右边找到一个目标，继续把 q 往上传

left=p, right=q
→ 两边各找到一个目标
→ 当前节点就是 LCA
```

可以记成：

```
        当前节点
        /     \
     有结果   有结果
        ↓
    当前节点是 LCA
```

而：

```
        当前节点
        /
      有结果
        ↓
    把结果继续向上传
```

------

# 面试推荐写法

这道题最推荐掌握**解法二：递归 DFS**。

核心代码实际上只有：

```
class Solution:
    def lowestCommonAncestor(self, root, p, q):

        if root is None or root is p or root is q:
            return root

        left = self.lowestCommonAncestor(root.left, p, q)
        right = self.lowestCommonAncestor(root.right, p, q)

        if left and right:
            return root

        return left if left else right
```

可以把整个算法浓缩成三个判断：

```
① 当前节点是 p / q
   → 返回当前节点

② 左右子树都有结果
   → 当前节点就是 LCA

③ 只有一边有结果
   → 把这一边的结果继续向上传
```

面试时尤其需要理解第二点：

> **左右子树同时返回非空值，意味着 `p` 和 `q` 在当前节点处分叉，因此当前节点就是最近公共祖先。**

这也是这道题最核心的递归思想。