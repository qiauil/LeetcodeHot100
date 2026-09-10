# Binary Tree Maximum Path Sum（二叉树最大路径和）

## 题目描述

给定一棵**非空二叉树**的根节点 `root`，返回任意一条**非空路径**的最大路径和（Maximum Path Sum）。

二叉树中的一条**路径（Path）**是由若干节点组成的序列，其中：

- 相邻两个节点之间必须存在一条边。
- 同一个节点不能在路径中重复出现。
- 路径**不一定经过根节点**。
- 路径可以从任意节点开始，并在任意节点结束。

一条路径的**路径和（Path Sum）**，就是该路径上所有节点值的总和。

例如：

```text
       1
      / \
     2   3
```

路径：

```text
2 → 1 → 3
```

路径和为：

```text
2 + 1 + 3 = 6
```

------

# 核心思路

这道题最重要的是区分两个概念：

1. **以当前节点为起点、向下延伸的最大路径**
2. **经过当前节点的完整路径**

假设当前节点为 `node`：

```text
        node
       /    \
    left    right
```

一条经过 `node` 的完整路径，可以是：

```text
左子树 → node → 右子树
```

因此：

```text
经过当前节点的最大路径和
=
node.val + leftMax + rightMax
```

但是，当我们要把一个结果返回给当前节点的**父节点**时，就不能同时选择左右两边。

原因是：

如果左右两边都返回给父节点，就会产生“分叉路径”，而题目中的路径必须是一条连续的链，不能在多个地方分叉。

因此递归函数返回给父节点的结果只能是：

```text
node.val + max(leftMax, rightMax)
```

这个区别是整道题的核心。

------

# 1. DFS：重复计算版本

## 思路

对于树中的每一个节点，都把它当作一条路径的**最高点 / 转折点**。

例如：

```text
left subtree
      \
       node
      /
right subtree
```

更常见地表示为：

```text
left → node → right
```

对于每个节点，需要分别计算：

```text
左子树能够向下提供的最大路径和
右子树能够向下提供的最大路径和
```

然后计算：

```python
node.val + leftDown + rightDown
```

作为“经过当前节点的最大路径”。

辅助函数 `getMax(node)` 用来计算：

> 从 `node` 开始，只沿着一个方向向下走时，可以得到的最大路径和。

因此：

```python
node.val + max(leftDown, rightDown)
```

如果这个值小于 `0`，就返回 `0`。

因为对于上层节点来说：

```text
加上一个负数路径
```

一定比：

```text
完全不使用这条路径
```

更差。

------

## 算法步骤

1. 使用 DFS 遍历树中的所有节点。
2. 对于每个节点：
   - 调用 `getMax()` 计算左子树的最大向下路径。
   - 调用 `getMax()` 计算右子树的最大向下路径。
   - 使用：

```python
node.val + left + right
```

更新全局最大值。

1. `getMax(node)`：
   - 如果节点为空，返回 `0`。
   - 递归计算左右子树。
   - 只能选择其中较大的一个方向继续向下：

```python
node.val + max(left, right)
```

- 如果结果为负数，则返回 `0`。

------

## 代码

```python
# 二叉树节点定义
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        # 不能初始化为 0。
        # 因为整棵树的节点值可能全部是负数。
        res = float("-inf")

        def dfs(node):
            nonlocal res

            if not node:
                return

            # 分别计算左右子树能够提供的最大“向下路径”
            left = self.getMax(node.left)
            right = self.getMax(node.right)

            # 把当前节点看作路径的最高点 / 转折点。
            #
            # 路径可以同时使用左子树和右子树：
            #
            # left -> node -> right
            res = max(
                res,
                node.val + left + right
            )

            # 继续检查树中的每一个节点
            dfs(node.left)
            dfs(node.right)

        dfs(root)

        return res

    def getMax(self, root: Optional[TreeNode]) -> int:
        """
        返回：
        从 root 开始，只沿一个方向向下延伸时，
        能得到的最大路径和。

        如果结果为负，则返回 0，
        表示上层节点最好完全不使用这条路径。
        """

        if not root:
            return 0

        left = self.getMax(root.left)
        right = self.getMax(root.right)

        # 向上返回时只能选择左、右其中一个分支
        path = root.val + max(left, right)

        # 如果这整条路径贡献为负数，则直接舍弃
        return max(0, path)
```

------

## 时间复杂度

### 时间复杂度：`O(n²)`

这一版本并不是最优解。

虽然 `dfs()` 本身会访问每个节点一次，但对于每一个节点，我们又调用了：

```python
getMax(node.left)
getMax(node.right)
```

而 `getMax()` 本身还会递归遍历子树。

因此存在大量重复计算。

在退化成链表的二叉树中：

```text
1
 \
  2
   \
    3
     \
      4
```

需要计算：

```text
n + (n-1) + (n-2) + ... + 1
```

所以最坏情况下：

```text
O(n²)
```

------

## 空间复杂度：`O(n)`

空间主要来自递归调用栈。

如果二叉树退化成链表：

```text
1
 \
  2
   \
    3
     \
      ...
```

递归深度最多为：

```text
O(n)
```

如果树比较平衡，则递归深度约为：

```text
O(log n)
```

因此最坏空间复杂度：

```text
O(n)
```

------

# 2. DFS：最优解

这是更推荐掌握的解法。

## 核心思想

对于每个节点，我们实际上可以在**同一次 DFS** 中完成两件事情：

### ① 计算经过当前节点的最大完整路径

这条路径可以同时使用左右两边：

```text
left → node → right
```

因此：

```python
node.val + leftMax + rightMax
```

这个结果用于更新**全局最大路径和**。

------

### ② 计算返回给父节点的最大向下路径

父节点如果想继续使用当前节点，只能选择一个方向：

```text
       parent
          |
        node
       /
    left
```

或者：

```text
       parent
          |
        node
           \
           right
```

不能是：

```text
       parent
          |
        node
       /    \
    left    right
```

因为这样路径就分叉了。

所以返回：

```python
node.val + max(leftMax, rightMax)
```

------

## 为什么负数路径要直接丢掉？

假设：

```text
        10
       /
     -20
```

如果使用左子树：

```text
10 + (-20) = -10
```

如果不用：

```text
10
```

显然：

```text
10 > -10
```

因此：

```python
leftMax = max(leftMax, 0)
rightMax = max(rightMax, 0)
```

可以理解为：

> 如果一个子树对最终路径产生负贡献，那就完全不选它。

------

## 算法步骤

定义：

```python
dfs(node)
```

表示：

> 从 `node` 开始，只沿一个方向向下走时，可以得到的最大路径和。

对于每个节点：

### 第一步：递归求左右子树

```python
leftMax = dfs(node.left)
rightMax = dfs(node.right)
```

### 第二步：去掉负贡献

```python
leftMax = max(leftMax, 0)
rightMax = max(rightMax, 0)
```

### 第三步：计算经过当前节点的完整路径

```python
node.val + leftMax + rightMax
```

更新全局答案：

```python
res = max(
    res,
    node.val + leftMax + rightMax
)
```

### 第四步：向父节点返回一个方向

```python
return node.val + max(leftMax, rightMax)
```

------

# 推荐代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        # 用 root.val 初始化，而不是 0。
        #
        # 因为所有节点都可能是负数，例如：
        #
        #     -3
        #
        # 此时答案应该是 -3，而不是 0。
        res = [root.val]

        def dfs(node):
            """
            返回：
            从 node 开始，只选择一个方向向下延伸，
            能够得到的最大路径和。
            """

            if not node:
                return 0

            # 计算左、右子树能够提供的最大向下路径
            leftMax = dfs(node.left)
            rightMax = dfs(node.right)

            # 如果某一侧为负数，就不使用该侧
            leftMax = max(leftMax, 0)
            rightMax = max(rightMax, 0)

            # 经过当前节点的完整路径可以同时包含：
            #
            # 左子树 + 当前节点 + 右子树
            #
            # 这个路径不能继续返回给父节点，
            # 但可以用来更新全局答案。
            currentPath = node.val + leftMax + rightMax

            res[0] = max(res[0], currentPath)

            # 返回给父节点时，只能选择其中一个方向，
            # 否则路径就会发生分叉。
            return node.val + max(leftMax, rightMax)

        dfs(root)

        return res[0]
```

------

# 更 Pythonic 的写法：使用 `nonlocal`

上面的代码使用：

```python
res = [root.val]
```

然后通过：

```python
res[0]
```

修改答案。

这是因为 Python 的内部函数如果直接写：

```python
res = ...
```

默认会创建一个新的局部变量，而不是修改外层函数中的 `res`。

一种更自然的 Python 写法是使用：

```python
nonlocal
```

------

## `nonlocal` 说明

例如：

```python
def outer():
    x = 10

    def inner():
        nonlocal x
        x = 20
```

这里：

```python
nonlocal x
```

表示：

> `x` 不是 `inner()` 自己的局部变量，而是来自外层 `outer()` 作用域。

因此这道题可以写成：

```python
class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        res = root.val

        def dfs(node):
            nonlocal res

            if not node:
                return 0

            # 左侧最大向下路径
            leftMax = max(dfs(node.left), 0)

            # 右侧最大向下路径
            rightMax = max(dfs(node.right), 0)

            # 当前节点作为转折点时的最大完整路径
            res = max(
                res,
                node.val + leftMax + rightMax
            )

            # 返回给父节点时只能选择一个方向
            return node.val + max(leftMax, rightMax)

        dfs(root)

        return res
```

从 Python 代码风格上来说，这个版本通常更加清晰。

------

# 时间与空间复杂度

## 时间复杂度：`O(n)`

每个节点只会被 DFS 访问一次。

每次访问节点时进行的工作都是：

```text
O(1)
```

因此：

```text
O(n)
```

其中 `n` 是二叉树的节点数量。

------

## 空间复杂度：`O(n)`

主要消耗来自递归栈。

最坏情况下，二叉树退化为链表：

```text
1
 \
  2
   \
    3
     \
      ...
```

递归深度为：

```text
O(n)
```

因此最坏空间复杂度：

```text
O(n)
```

如果是一棵平衡二叉树，则递归深度为：

```text
O(log n)
```

------

# 一个完整例子

假设：

```text
        -10
        /  \
       9    20
           /  \
          15   7
```

首先看节点 `15`：

```text
left = 0
right = 0

完整路径 = 15
返回父节点 = 15
```

节点 `7`：

```text
完整路径 = 7
返回父节点 = 7
```

来到节点 `20`：

```text
leftMax = 15
rightMax = 7
```

经过 `20` 的完整路径：

```text
15 → 20 → 7
```

路径和：

```text
15 + 20 + 7 = 42
```

因此：

```text
res = 42
```

但是返回给 `-10` 时不能返回：

```text
15 + 20 + 7
```

因为这样意味着父节点连接到一个已经分叉的路径。

只能选择：

```python
20 + max(15, 7)
```

所以返回：

```text
35
```

最终答案：

```text
42
```

------

# 常见错误

## 1. 把答案初始化为 `0`

错误：

```python
res = 0
```

考虑：

```text
-3
```

合法路径必须至少包含一个节点，所以答案应该是：

```text
-3
```

但如果：

```python
res = 0
```

最终可能错误地返回：

```text
0
```

而树中根本不存在路径和为 `0` 的非空路径。

正确写法：

```python
res = root.val
```

或者：

```python
res = float("-inf")
```

------

# 2. 没有丢弃负数子树

错误：

```python
leftMax = dfs(node.left)
rightMax = dfs(node.right)
```

之后直接：

```python
node.val + leftMax + rightMax
```

如果：

```text
leftMax = -10
```

显然不应该加入这条路径。

应该写：

```python
leftMax = max(dfs(node.left), 0)
rightMax = max(dfs(node.right), 0)
```

本质上是在做：

```text
有正贡献 → 加入路径

有负贡献 → 不加入路径
```

------

# 3. 向父节点返回左右两条路径

这是这道题最常见、也最重要的错误。

错误：

```python
return node.val + leftMax + rightMax
```

原因是这会产生非法的“多次分叉”。

例如：

```text
           A
           |
           B
         /   \
        C     D
```

如果 `B` 把：

```text
C → B → D
```

整体返回给 `A`，那么再连接 `A` 后：

```text
      A
      |
      B
     / \
    C   D
```

这已经不是一条简单路径，而是一个 Y 型结构。

正确返回值：

```python
return node.val + max(leftMax, rightMax)
```

因为返回给父节点的路径只能是：

```text
A → B → C
```

或者：

```text
A → B → D
```

------

# 4. 忘记单节点本身也可以是一条路径

路径不要求：

```text
至少有两个节点
```

一个节点本身就是合法路径。

例如：

```text
    -5
   /  \
 -10  -20
```

最大路径就是：

```text
-5
```

而不是把任何负数子节点加进来。

------

# 5. 混淆「最大路径」与「向下路径」

这是理解这道题最关键的地方。

递归过程中其实同时存在两个不同的答案。

### 最大完整路径

可以使用左右两个方向：

```text
left → node → right
```

计算：

```python
node.val + leftMax + rightMax
```

用途：

```text
更新全局最大值
```

------

### 最大向下路径

只能使用其中一个方向：

```text
node → left
```

或者：

```text
node → right
```

计算：

```python
node.val + max(leftMax, rightMax)
```

用途：

```text
返回给父节点
```

可以记成：

```text
更新答案时：可以选两边

向上返回时：只能选一边
```

------

# 面试时的思考模板

遇到这道题，可以按照下面的方式快速推导。

首先问自己：

```text
如果当前节点是整条路径的最高点，最大路径是多少？
```

答案：

```python
node.val + leftMax + rightMax
```

然后问：

```text
如果父节点要继续使用当前节点，
当前节点能向父节点提供什么？
```

只能提供：

```python
node.val + max(leftMax, rightMax)
```

最后问：

```text
如果某一边是负数，还应该使用吗？
```

不应该：

```python
max(subtree, 0)
```

所以最终模板就是：

```python
def dfs(node):
    if not node:
        return 0

    left = max(dfs(node.left), 0)
    right = max(dfs(node.right), 0)

    # 完整路径：左右都能用
    res = max(res, node.val + left + right)

    # 返回父节点：只能选一边
    return node.val + max(left, right)
```

------

# 总结

这道题本质上是一道经典的**树形 DFS / 后序遍历（Postorder Traversal）**问题。

必须同时维护两种状态：

```text
1. 经过当前节点的完整最大路径
2. 返回给父节点的单边最大路径
```

对应公式：

```python
# 当前节点作为路径转折点
currentPath = node.val + leftMax + rightMax

# 返回给父节点
return node.val + max(leftMax, rightMax)
```

并且需要过滤负贡献：

```python
leftMax = max(leftMax, 0)
rightMax = max(rightMax, 0)
```

最终推荐记住：

```text
左右两边都可以用于“更新答案”，
但是只能选择一边“返回父节点”。
```

最优复杂度：

```text
时间复杂度：O(n)
空间复杂度：O(n) 最坏情况
```
