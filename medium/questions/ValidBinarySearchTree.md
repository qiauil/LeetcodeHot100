# 验证二叉搜索树（Valid Binary Search Tree）

给定一棵二叉树的根节点 `root`，判断它是否是一棵**有效的二叉搜索树（Binary Search Tree, BST）**。

如果是，返回 `True`；否则返回 `False`。

一棵有效的二叉搜索树需要满足：

- 对于任意节点，其**左子树中的所有节点值**都必须**严格小于**当前节点值。
- 对于任意节点，其**右子树中的所有节点值**都必须**严格大于**当前节点值。
- 左右子树本身也必须是二叉搜索树。

> **关键点：BST 的限制针对的是整棵子树，而不仅仅是直接的左右子节点。**

例如：

```
        10
       /
      5
       \
        15
```

虽然：

```
5 < 10
15 > 5
```

每个节点与自己的直接父节点看起来都满足要求，但 `15` 位于 `10` 的左子树中，却有：

```
15 > 10
```

所以这**不是**一棵有效的 BST。

------

# 方法一：暴力递归

## 思路

最直接的思路是：

对于树中的**每一个节点**，都检查：

1. 左子树中的所有节点是否都 `< node.val`
2. 右子树中的所有节点是否都 `> node.val`
3. 左子树本身是否也是 BST
4. 右子树本身是否也是 BST

例如对于：

```
        10
       /  \
      5    15
```

检查节点 `10` 时：

```
左子树所有节点 < 10
右子树所有节点 > 10
```

然后再分别以 `5` 和 `15` 为根节点重复整个过程。

这个方法逻辑非常直观，但问题在于：**同一个节点可能会被重复访问很多次。**

------

## 算法步骤

对于当前节点 `root`：

1. 如果 `root` 为空，返回 `True`。
2. 遍历整个左子树，检查所有值是否 `< root.val`。
3. 遍历整个右子树，检查所有值是否 `> root.val`。
4. 如果任何一个检查失败，返回 `False`。
5. 递归检查左子树本身是否是 BST。
6. 递归检查右子树本身是否是 BST。
7. 全部满足则返回 `True`。

------

## Python 实现

```
# 二叉树节点定义
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    # 检查左子树节点：
    # 左子树中的所有值都必须小于当前节点值
    left_check = staticmethod(lambda val, limit: val < limit)

    # 检查右子树节点：
    # 右子树中的所有值都必须大于当前节点值
    right_check = staticmethod(lambda val, limit: val > limit)

    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        # 空树认为是有效 BST
        if not root:
            return True

        # 检查：
        # 1. 左子树所有节点 < root.val
        # 2. 右子树所有节点 > root.val
        if (
            not self.isValid(root.left, root.val, self.left_check)
            or not self.isValid(root.right, root.val, self.right_check)
        ):
            return False

        # 继续递归检查左右子树本身是否为 BST
        return (
            self.isValidBST(root.left)
            and self.isValidBST(root.right)
        )

    def isValid(
        self,
        root: Optional[TreeNode],
        limit: int,
        check
    ) -> bool:
        """检查整棵子树是否都满足指定的大小关系。"""

        if not root:
            return True

        # 当前节点不满足限制
        if not check(root.val, limit):
            return False

        # 左右子树中的所有节点都必须满足同一个限制
        return (
            self.isValid(root.left, limit, check)
            and self.isValid(root.right, limit, check)
        )
```

## `staticmethod` 和 `lambda` 说明

这里有两个比较函数：

```
left_check = staticmethod(lambda val, limit: val < limit)
right_check = staticmethod(lambda val, limit: val > limit)
```

`lambda` 是 Python 中定义简单匿名函数的方式：

```
lambda val, limit: val < limit
```

基本等价于：

```
def left_check(val, limit):
    return val < limit
```

`staticmethod` 则表示这个函数属于类的命名空间，但调用它时**不需要传入 `self`**。

不过对于代码面试而言，这种写法稍微有些复杂。直接写两个普通辅助函数通常会更容易阅读。

------

## 复杂度分析

### 时间复杂度：`O(n²)`

最坏情况下，例如树退化成链表：

```
1
 \
  2
   \
    3
     \
      ...
```

每个节点都需要重新扫描自己的子树，因此总访问次数大约为：

```
n + (n - 1) + (n - 2) + ... + 1
```

即：

```
O(n²)
```

### 空间复杂度：`O(n)`

主要来自递归调用栈。

最坏情况下树退化成链表，递归深度达到 `n`。

------

# 方法二：DFS + 上下界 ⭐ 推荐

## 核心思路

这是这道题最重要、也是面试中最推荐的方法。

BST 的要求不能简单理解成：

```
左孩子 < 父节点 < 右孩子
```

更准确的理解应该是：

> **树中的每一个节点都有一个由所有祖先共同决定的合法取值区间。**

对于根节点：

```
(-∞, +∞)
```

因为没有任何祖先限制它。

假设当前节点为：

```
node
```

合法区间为：

```
(left, right)
```

那么必须满足：

```
left < node.val < right
```

接下来：

### 进入左子树

左孩子必须小于当前节点，因此：

```
原区间：
(left, right)

新范围：
(left, node.val)
```

### 进入右子树

右孩子必须大于当前节点，因此：

```
原区间：
(left, right)

新范围：
(node.val, right)
```

这样，每向下一层，我们都在不断**收紧合法范围**。

------

## 示例

考虑：

```
          10
         /  \
        5    15
            / \
           6   20
```

开始：

```
10 的范围：
(-∞, +∞)
```

进入右子树：

```
15 的范围：
(10, +∞)
```

再进入 `15` 的左子树：

```
6 的范围：
(10, 15)
```

但是：

```
6 < 10
```

因此：

```
6 ∉ (10, 15)
```

立即发现这不是 BST。

这也说明了为什么**只比较父节点是不够的**。

如果只比较：

```
6 < 15
```

看起来完全合法。

但实际上 `6` 位于 `10` 的右子树，因此还必须满足：

```
6 > 10
```

------

## 算法步骤

1. 从根节点开始 DFS。
2. 根节点的合法范围为：

```
(-∞, +∞)
```

1. 对于每个节点，检查：

```
left < node.val < right
```

1. 如果不满足，直接返回 `False`。
2. 递归检查左子树：

```
(left, node.val)
```

1. 递归检查右子树：

```
(node.val, right)
```

1. 所有节点都满足要求，则返回 `True`。

------

## Python 实现

```
# 二叉树节点定义
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:

        def valid(node, lower, upper):
            # 空节点不会违反 BST 规则
            if not node:
                return True

            # 当前节点必须严格位于合法区间内
            if not (lower < node.val < upper):
                return False

            # 左子树：
            # 所有节点必须 < 当前节点
            # 所以上界更新为 node.val
            left_valid = valid(
                node.left,
                lower,
                node.val
            )

            # 右子树：
            # 所有节点必须 > 当前节点
            # 所以下界更新为 node.val
            right_valid = valid(
                node.right,
                node.val,
                upper
            )

            return left_valid and right_valid

        # 根节点一开始没有任何上下界限制
        return valid(
            root,
            float("-inf"),
            float("inf")
        )
```

实际面试中可以写得更简洁：

```
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def dfs(node, lower, upper):
            if not node:
                return True

            if not (lower < node.val < upper):
                return False

            return (
                dfs(node.left, lower, node.val)
                and
                dfs(node.right, node.val, upper)
            )

        return dfs(root, float("-inf"), float("inf"))
```

------

## `float("-inf")` 和 `float("inf")`

Python 可以通过：

```
float("-inf")
float("inf")
```

分别表示：

```
负无穷
正无穷
```

例如：

```
float("-inf") < -999999999999
# True

float("inf") > 999999999999
# True
```

因此它们非常适合表示根节点最开始的合法范围：

```
(-∞, +∞)
```

这样就不需要猜测题目中节点值可能出现的最大值和最小值。

------

## 复杂度分析

### 时间复杂度：`O(n)`

每个节点只访问一次。

如果有 `n` 个节点：

```
O(n)
```

### 空间复杂度：`O(h)`

其中 `h` 是树的高度，空间主要来自递归调用栈。

平衡二叉树：

```
h = O(log n)
```

因此：

```
O(log n)
```

最坏情况下树退化成链表：

```
h = O(n)
```

因此：

```
O(n)
```

所以更精确地说，空间复杂度是：

```
O(h)
```

而不是简单地写成 `O(n)`。

------

# 方法三：BFS + 上下界

## 思路

DFS 的核心其实并不是“深度优先”，而是：

> 每个节点都需要携带一个由祖先决定的合法范围。

因此完全可以把递归 DFS 改成 BFS。

我们使用队列保存：

```
(node, lower_bound, upper_bound)
```

例如：

```
(root, -∞, +∞)
```

每取出一个节点：

```
lower < node.val < upper
```

如果不满足，返回 `False`。

然后：

```
左孩子：
(node.left, lower, node.val)

右孩子：
(node.right, node.val, upper)
```

------

## Python 实现

```
from collections import deque


class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True

        # 队列中的元素：
        # (节点, 合法下界, 合法上界)
        queue = deque([
            (root, float("-inf"), float("inf"))
        ])

        while queue:
            # 从队列左侧取出一个节点
            node, lower, upper = queue.popleft()

            # 当前节点必须严格处于合法区间
            if not (lower < node.val < upper):
                return False

            # 左孩子的上界变成当前节点值
            if node.left:
                queue.append(
                    (node.left, lower, node.val)
                )

            # 右孩子的下界变成当前节点值
            if node.right:
                queue.append(
                    (node.right, node.val, upper)
                )

        return True
```

------

## `collections.deque`

`deque` 是 Python `collections` 模块提供的**双端队列（double-ended queue）**：

```
from collections import deque
```

创建：

```
queue = deque()
```

从右边加入：

```
queue.append(x)
```

从左边删除：

```
x = queue.popleft()
```

因此 BFS 中通常使用：

```
queue.append(...)
queue.popleft()
```

而不推荐普通 `list` 的：

```
queue.pop(0)
```

因为 Python `list.pop(0)` 删除第一个元素之后，需要移动后面的所有元素，时间复杂度为：

```
O(n)
```

而：

```
deque.popleft()
```

是：

```
O(1)
```

因此 `deque` 是 Python 实现 BFS 时最常用的数据结构之一。

------

## 复杂度分析

### 时间复杂度：`O(n)`

每个节点进入和离开队列一次。

### 空间复杂度：`O(n)`

BFS 的队列在最坏情况下可能同时保存大量节点。

例如一棵完全二叉树最底层可能包含约：

```
n / 2
```

个节点，因此：

```
O(n)
```

------

# 方法四：中序遍历 ⭐ 值得掌握

还有一个非常经典的 BST 性质：

> **对一棵有效 BST 进行中序遍历，得到的节点值序列一定严格递增。**

中序遍历顺序是：

```
Left → Root → Right
```

例如：

```
        5
       / \
      3   7
     / \ / \
    2  4 6  8
```

中序遍历：

```
2 → 3 → 4 → 5 → 6 → 7 → 8
```

严格递增。

因此，我们只需要进行一次中序遍历，并记录前一个访问的节点值 `prev`。

每次访问当前节点时检查：

```
node.val > prev
```

如果出现：

```
node.val <= prev
```

就说明不是 BST。

------

## Python 实现

```
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        prev = float("-inf")

        def inorder(node):
            nonlocal prev

            if not node:
                return True

            # 1. 遍历左子树
            if not inorder(node.left):
                return False

            # 2. 访问当前节点
            # BST 的中序遍历必须严格递增
            if node.val <= prev:
                return False

            # 更新前一个访问的节点值
            prev = node.val

            # 3. 遍历右子树
            return inorder(node.right)

        return inorder(root)
```

------

## `nonlocal` 说明

这里：

```
prev = float("-inf")

def inorder(node):
    nonlocal prev
```

`prev` 定义在外层函数 `isValidBST()` 中，而内部函数 `inorder()` 需要**修改**这个变量。

因此需要：

```
nonlocal prev
```

如果没有 `nonlocal`，当你写：

```
prev = node.val
```

Python 会认为你正在 `inorder()` 内创建一个新的局部变量 `prev`。

`nonlocal` 的含义可以理解为：

> `prev` 不是当前函数的局部变量，请使用外层函数中的那个 `prev`。

------

## 复杂度

时间复杂度：

```
O(n)
```

空间复杂度：

```
O(h)
```

其中 `h` 是树的高度，来自递归调用栈。

------

# 常见错误

## 1. 只和直接父节点比较

这是本题最常见的错误。

错误思路类似：

```
if node.left and node.left.val >= node.val:
    return False

if node.right and node.right.val <= node.val:
    return False
```

它只能保证：

```
左孩子 < 当前节点 < 右孩子
```

却不能保证整棵子树满足要求。

例如：

```
        10
       /
      5
       \
        15
```

局部关系：

```
5 < 10
15 > 5
```

全部正确。

但 `15` 位于 `10` 的左子树，因此还必须：

```
15 < 10
```

显然不满足。

所以 DFS 解法中必须把祖先的限制继续向下传递。

------

## 2. 使用 `<=` / `>=` 作为合法条件

题目要求：

```
左子树 < 当前节点
右子树 > 当前节点
```

是**严格不等式**。

因此：

```
        5
       / \
      3   5
```

不是有效 BST，因为存在重复的 `5`。

DFS 中应该检查：

```
lower < node.val < upper
```

而不是：

```
lower <= node.val <= upper
```

中序遍历也必须是：

```
严格递增
```

所以判断条件为：

```
if node.val <= prev:
    return False
```

------

## 3. 边界值问题

在一些语言中，可能会写：

```
Integer.MIN_VALUE
Integer.MAX_VALUE
```

作为初始上下界。

但如果节点本身允许等于这些值，就可能产生边界问题。

例如 Java 中通常可以使用更大的类型：

```
Long.MIN_VALUE
Long.MAX_VALUE
```

Python 不存在固定大小整数溢出的问题，因为 Python 的 `int` 可以自动扩展。

本题中可以直接使用：

```
float("-inf")
float("inf")
```

作为初始上下界。

------

# 面试总结

这道题最核心的认知是：

> **BST 的合法性不是“节点与父节点之间的局部关系”，而是“节点必须满足所有祖先共同施加的范围限制”。**

因此最推荐的解法是：

```
DFS + 合法上下界
```

核心代码实际上只有：

```
def valid(node, lower, upper):
    if not node:
        return True

    if not (lower < node.val < upper):
        return False

    return (
        valid(node.left, lower, node.val)
        and
        valid(node.right, node.val, upper)
    )
```

可以重点记住范围如何变化：

```
当前节点：
(lower, upper)

          node
         /    \
        /      \
(lower,node)  (node,upper)
```

三种主要方法的区别可以概括为：

| 方法         | 时间    | 空间   | 核心思想                   | 面试推荐 |
| ------------ | ------- | ------ | -------------------------- | -------- |
| 暴力递归     | `O(n²)` | `O(h)` | 对每个节点重新扫描左右子树 | ⭐⭐       |
| DFS + 上下界 | `O(n)`  | `O(h)` | 将祖先限制向下传递         | ⭐⭐⭐⭐⭐    |
| BFS + 上下界 | `O(n)`  | `O(n)` | 用队列携带合法区间         | ⭐⭐⭐⭐     |
| 中序遍历     | `O(n)`  | `O(h)` | BST 中序遍历严格递增       | ⭐⭐⭐⭐⭐    |

如果是代码面试，**优先掌握 DFS + 上下界**；同时最好知道**中序遍历严格递增**这个 BST 的重要性质。