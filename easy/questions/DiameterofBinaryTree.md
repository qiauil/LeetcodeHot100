# 二叉树的直径（Diameter of Binary Tree）

二叉树的**直径（diameter）\**定义为：树中\**任意两个节点之间最长路径的长度**。

这条路径**不一定经过根节点**。

两个节点之间路径的**长度**，指的是路径中包含的**边（edge）的数量**，而不是节点数量。同时，一条合法路径不能重复经过同一个节点。

给定二叉树的根节点 `root`，返回这棵树的直径。

------

## 前置知识

在解决这道题之前，建议熟悉以下内容：

- **二叉树基础**
  - 节点、左右子节点、父子关系
- **深度优先搜索（DFS）**
  - 尤其是递归 DFS
- **二叉树高度的计算**
  - 如何递归计算一棵子树的高度
- **后序遍历（Post-order Traversal）**
  - 先处理左右子树，再处理当前节点

------

# 一、核心概念：为什么直径和高度有关？

对于任意一个节点：

```text
        node
       /    \
    left    right
```

如果最长路径**经过当前节点 `node`**，那么这条路径一定是：

```text
左子树中的某个最深节点
        ↓
      node
        ↓
右子树中的某个最深节点
```

因此：

```text
经过当前节点的最长路径长度
= 左子树高度 + 右子树高度
```

这里的“高度”可以理解为：

> 从当前节点的某个子节点出发，向下能走的最长路径中包含多少个节点。

例如：

```text
      1
     / \
    2   3
   /
  4
```

对于节点 `1`：

- 左子树高度 = 2，对应 `2 -> 4`
- 右子树高度 = 1，对应 `3`

所以经过节点 `1` 的路径长度：

```text
2 + 1 = 3
```

对应路径：

```text
4 -> 2 -> 1 -> 3
```

确实有 `3` 条边。

因此整道题的关键是：

> **在计算每个节点高度的同时，检查 `leftHeight + rightHeight`，并维护其中的最大值。**

------

# 二、暴力解法

## 思路

对于树中的每一个节点，我们都计算：

```text
左子树高度 + 右子树高度
```

得到**经过这个节点的最长路径**。

但最长路径也可能完全位于左子树或右子树，因此当前子树的直径应该是下面三者的最大值：

```text
1. 经过当前节点的最长路径
2. 左子树的直径
3. 右子树的直径
```

也就是：

```text
diameter(root)
=
max(
    height(root.left) + height(root.right),
    diameter(root.left),
    diameter(root.right)
)
```

------

## 算法步骤

对于当前节点 `root`：

1. 如果 `root` 为空，返回 `0`。
2. 计算左子树高度 `leftHeight`。
3. 计算右子树高度 `rightHeight`。
4. 计算经过当前节点的直径：

```python
leftHeight + rightHeight
```

1. 递归计算左子树和右子树自己的直径。
2. 返回三者中的最大值。

------

## 代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        # 空树没有任何边，因此直径为 0
        if not root:
            return 0

        # 分别计算左右子树高度
        left_height = self.maxHeight(root.left)
        right_height = self.maxHeight(root.right)

        # 经过当前节点的最长路径
        diameter_through_root = left_height + right_height

        # 最长路径也可能完全位于某一棵子树中
        diameter_in_subtree = max(
            self.diameterOfBinaryTree(root.left),
            self.diameterOfBinaryTree(root.right)
        )

        return max(diameter_through_root, diameter_in_subtree)

    def maxHeight(self, root: Optional[TreeNode]) -> int:
        """返回以 root 为根节点的子树高度。"""
        if not root:
            return 0

        return 1 + max(
            self.maxHeight(root.left),
            self.maxHeight(root.right)
        )
```

------

## 为什么这个方法比较慢？

问题出在：

```python
self.maxHeight(...)
```

会被反复调用。

例如对于根节点，我们会计算一次整棵左子树的高度。

之后递归进入左子树时，又会重新计算其中很多节点的高度。

因此很多节点会被重复访问。

对于极端情况：

```text
1
 \
  2
   \
    3
     \
      4
       \
        ...
```

计算高度的工作量大致为：

```text
n + (n - 1) + (n - 2) + ... + 1
```

因此时间复杂度为：

```text
O(n²)
```

------

## 复杂度

设：

- `n` = 节点数量
- `h` = 树的高度

时间复杂度：

```text
O(n²)
```

最坏情况下，每个节点都会重复计算很多次高度。

空间复杂度：

```text
O(h)
```

递归调用栈最多有 `h` 层。

最坏情况下树退化为链表：

```text
O(n)
```

------

# 三、推荐解法：一次 DFS

这是面试中最推荐的解法。

## 核心思路

暴力解法的问题是：

> 我们为了计算直径，不断重复计算子树高度。

实际上，可以让 DFS **只计算一次高度**。

在计算高度的过程中，顺便计算经过当前节点的直径：

```python
left + right
```

并不断更新全局最大值。

------

## DFS 到底返回什么？

这是这道题最重要的地方。

DFS 返回：

```text
当前子树的高度
```

而不是直径。

对于当前节点：

```python
left = dfs(root.left)
right = dfs(root.right)
```

那么：

```python
left + right
```

是**经过当前节点的直径**。

而返回给父节点的值应该是：

```python
1 + max(left, right)
```

因为父节点只能选择：

```text
当前节点 → 左边
```

或者：

```text
当前节点 → 右边
```

不可能同时走左右两边，否则就无法继续向父节点延伸。

------

## 算法步骤

对于每个节点：

1. DFS 计算左子树高度 `left`。
2. DFS 计算右子树高度 `right`。
3. 当前节点作为路径最高点时：

```python
left + right
```

1. 用这个值更新全局最大直径。
2. 返回当前子树高度：

```python
1 + max(left, right)
```

由于每个节点只访问一次，因此时间复杂度是：

```text
O(n)
```

------

## 代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        # 记录目前找到的最大直径
        res = 0

        def dfs(node):
            nonlocal res

            # 空节点高度为 0
            if not node:
                return 0

            # 后序遍历：先计算左右子树高度
            left = dfs(node.left)
            right = dfs(node.right)

            # 经过当前节点的路径：
            # 左侧最长路径 + 右侧最长路径
            res = max(res, left + right)

            # 返回当前子树的高度给父节点
            return 1 + max(left, right)

        dfs(root)

        return res
```

------

# 四、理解 `nonlocal`

这段代码中有：

```python
res = 0

def dfs(node):
    nonlocal res
```

`nonlocal` 是 Python 中用于**嵌套函数**的关键字。

这里：

```python
diameterOfBinaryTree()
```

是外层函数，而：

```python
dfs()
```

是内层函数。

`res` 定义在外层函数中：

```python
res = 0
```

如果内层函数希望修改它：

```python
res = max(res, left + right)
```

就需要声明：

```python
nonlocal res
```

否则 Python 会把 `res` 当作 `dfs()` 内部的局部变量。

可以理解为：

> `nonlocal res`：我这里使用的 `res` 不是 `dfs()` 自己的局部变量，而是外层函数中的那个 `res`。

------

# 五、为什么 `left + right` 是“边数”？

这一点非常容易混淆。

假设：

```text
      1
     / \
    2   3
   /
  4
```

对于节点 `1`：

```python
left = 2
right = 1
```

因为：

```text
左子树高度：2 -> 4，共 2 个节点
右子树高度：3，共 1 个节点
```

最长路径：

```text
4 -> 2 -> 1 -> 3
```

节点数是：

```text
4
```

边数是：

```text
3
```

而：

```python
left + right
= 2 + 1
= 3
```

正好就是边数。

所以这里不需要：

```python
left + right + 1
```

因为题目要求的是**边的数量**。

------

# 六、为什么 DFS 返回高度，而不是直径？

这是非常重要的面试考点。

假设：

```text
        A
       /
      B
     / \
    C   D
```

在节点 `B`：

```python
left + right
```

代表：

```text
C -> B -> D
```

这是一个完整的路径。

但是当父节点 `A` 想要使用 `B` 提供的信息时，它只能继续一条方向：

```text
A -> B -> C
```

或者：

```text
A -> B -> D
```

不能使用：

```text
C -> B -> D
```

之后又回到 `B -> A`。

因为这样会重复经过 `B`。

因此 DFS 返回的必须是：

```python
1 + max(left, right)
```

即：

> 从当前节点向下走的最长单边路径。

而：

```python
left + right
```

只是用于更新最终答案。

可以记成：

```text
DFS 返回：单边最大贡献

更新答案：左边贡献 + 右边贡献
```

这是很多树形 DFS 问题都会使用的模式。

------

# 七、时间和空间复杂度

## 时间复杂度

```text
O(n)
```

每个节点只访问一次。

每个节点所做的额外操作都是常数时间：

```python
max(...)
```

因此总时间：

```text
O(n)
```

------

## 空间复杂度

```text
O(h)
```

其中 `h` 是树的高度，主要来自递归调用栈。

### 平衡二叉树

如果树比较平衡：

```text
h = O(log n)
```

因此：

```text
Space = O(log n)
```

### 极端退化树

例如：

```text
1
 \
  2
   \
    3
     \
      4
```

树高度：

```text
h = n
```

因此：

```text
Space = O(n)
```

------

# 八、迭代 DFS

递归 DFS 会使用 Python 的函数调用栈。

我们也可以显式维护一个 `stack`，模拟递归过程。

由于父节点必须等到左右子节点都计算完成以后，才能得到：

```python
leftHeight + rightHeight
```

因此这里需要执行的是**后序遍历**：

```text
左子树
→ 右子树
→ 当前节点
```

------

## 核心思路

对于每个节点，保存：

```python
(height, diameter)
```

其中：

```text
height
```

表示这棵子树的高度。

```text
diameter
```

表示这棵子树内部的最大直径。

对于当前节点：

```python
left_height, left_diameter
right_height, right_diameter
```

可以得到：

```python
height = 1 + max(left_height, right_height)
```

以及：

```python
diameter = max(
    left_height + right_height,
    left_diameter,
    right_diameter
)
```

------

## 代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        # 题目通常允许空树，这里单独处理更加安全
        if not root:
            return 0

        # 模拟递归调用栈
        stack = [root]

        # mp[node] = (以 node 为根的子树高度, 子树最大直径)
        #
        # 给 None 一个默认值，可以避免反复判断左右子节点是否为空
        mp = {
            None: (0, 0)
        }

        while stack:
            # 查看栈顶，但暂时不弹出
            node = stack[-1]

            # 如果左子树还没有处理，先处理左子树
            if node.left and node.left not in mp:
                stack.append(node.left)

            # 左子树完成后，再处理右子树
            elif node.right and node.right not in mp:
                stack.append(node.right)

            else:
                # 左右子树都已经计算完，可以处理当前节点
                node = stack.pop()

                left_height, left_diameter = mp[node.left]
                right_height, right_diameter = mp[node.right]

                # 当前子树高度
                height = 1 + max(left_height, right_height)

                # 当前子树最大直径有三种可能：
                # 1. 经过当前节点
                # 2. 完全在左子树
                # 3. 完全在右子树
                diameter = max(
                    left_height + right_height,
                    left_diameter,
                    right_diameter
                )

                mp[node] = (height, diameter)

        return mp[root][1]
```

------

## 复杂度

时间复杂度：

```text
O(n)
```

每个节点只会被最终处理一次。

空间复杂度：

```text
O(n)
```

因为：

```python
mp
```

需要保存每个节点的结果。

此外：

```python
stack
```

最坏情况下也可能达到 `O(n)`。

------

# 九、Python 中使用节点作为字典 Key

迭代版本中有：

```python
mp[node] = (height, diameter)
```

也就是直接把：

```python
TreeNode
```

对象作为 Python 字典的 Key。

一般情况下，普通 Python 类的实例是可以被哈希的，因此可以作为：

```python
dict
```

的 key。

例如：

```python
node = TreeNode(1)

mp = {}
mp[node] = 10
```

之后：

```python
mp[node]
```

就可以得到：

```text
10
```

这里实际上是根据对象身份来区分不同节点，而不是根据：

```python
node.val
```

来区分。

这很重要，因为二叉树中完全可能存在：

```text
两个 val 都为 1 的不同节点
```

因此不能简单写：

```python
mp[node.val]
```

否则会发生冲突。

------

# 十、常见错误

## 1. DFS 返回了直径，而不是高度

错误：

```python
def dfs(node):
    left = dfs(node.left)
    right = dfs(node.right)

    return left + right
```

这里返回的是：

```text
经过当前节点的直径
```

但父节点真正需要的是：

```text
当前节点能够向下提供的最长单边路径
```

正确写法：

```python
return 1 + max(left, right)
```

而：

```python
left + right
```

只用于更新答案：

```python
res = max(res, left + right)
```

可以记住：

```text
return max(left, right) + 1
update answer with left + right
```

------

## 2. 认为最长路径一定经过根节点

错误思路：

```python
left = height(root.left)
right = height(root.right)

return left + right
```

这只计算了：

```text
经过根节点的最长路径
```

但真正的直径可能完全位于某棵子树中。

例如：

```text
          1
         /
        2
       / \
      3   4
     /     \
    5       6
```

最长路径可能是：

```text
5 -> 3 -> 2 -> 4 -> 6
```

根本不经过节点 `1`。

因此必须在**每一个节点**上检查：

```python
left + right
```

------

## 3. 把节点数量当成边数量

题目要求：

```text
diameter = number of edges
```

例如：

```text
1 -> 2 -> 3 -> 4
```

这里：

```text
节点数量 = 4
边数量 = 3
```

最终答案应该是：

```text
3
```

而不是：

```text
4
```

------

# 十一、面试中最值得掌握的版本

推荐直接掌握下面这个版本：

```python
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        diameter = 0

        def dfs(node):
            nonlocal diameter

            if not node:
                return 0

            left_height = dfs(node.left)
            right_height = dfs(node.right)

            # 当前节点作为路径最高点时的直径
            diameter = max(
                diameter,
                left_height + right_height
            )

            # 返回当前节点向下能够提供的最长路径长度
            return 1 + max(left_height, right_height)

        dfs(root)

        return diameter
```

面试时可以这样概括核心逻辑：

```text
对每个节点，我通过 DFS 得到左右子树的高度。

如果最长路径经过当前节点，那么路径长度就是：
leftHeight + rightHeight。

所以我在 DFS 过程中维护全局最大值。

DFS 本身需要返回当前子树的高度给父节点，
因此返回 1 + max(leftHeight, rightHeight)。

每个节点只处理一次，所以时间复杂度是 O(n)，
递归栈空间复杂度是 O(h)。
```

------

# 十二、这道题背后的通用 DFS 模式

这道题实际上体现了一个非常重要的树形 DFS 模板：

```python
def dfs(node):
    if not node:
        return base_value

    left = dfs(node.left)
    right = dfs(node.right)

    # 使用左右子树信息更新全局答案
    update_answer(left, right)

    # 返回父节点真正需要的信息
    return something
```

关键在于区分两个概念：

```text
① 当前节点要贡献给最终答案什么？
② 当前节点要返回给父节点什么？
```

在本题中：

```text
更新答案：
left + right

返回父节点：
1 + max(left, right)
```

这种“**DFS 返回单边信息，同时用左右两边共同更新全局答案**”的思想非常常见。

例如很多类似的树问题都会使用同样的模式：

```text
Binary Tree Maximum Path Sum
Longest Univalue Path
Maximum path-related problems
```

因此，比起单纯记住代码，更值得记住的是：

> **父节点通常只能接收一条向下延伸的路径，但当前节点在计算全局最优解时，可以同时组合左右两条路径。**
