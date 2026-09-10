# 二叉树的中序遍历（Binary Tree Inorder Traversal）

给定二叉树的根节点 `root`，返回其节点值的**中序遍历（Inorder Traversal）**结果。

中序遍历的访问顺序是：

**左子树 → 当前节点 → 右子树**

例如，对于下面这棵树：

```text
    2
   / \
  1   3
```

中序遍历结果为：

```text
[1, 2, 3]
```

如果这棵树恰好是一棵**二叉搜索树（Binary Search Tree, BST）**，那么中序遍历得到的节点值会按照升序排列。

------

## 1. 递归深度优先搜索（Recursive DFS）

### 思路

中序遍历本身就具有非常自然的递归结构：

1. 先遍历左子树；
2. 再访问当前节点；
3. 最后遍历右子树。

因此可以定义一个递归函数 `inorder(node)`，表示：

> 对以 `node` 为根节点的子树执行中序遍历。

当 `node` 为空时，说明已经走到了叶子节点之外，直接返回即可。

### 算法步骤

1. 创建结果数组 `res`。
2. 定义递归函数 `inorder(node)`。
3. 如果 `node` 为 `None`，直接返回。
4. 递归遍历 `node.left`。
5. 将当前节点的值 `node.val` 加入结果数组。
6. 递归遍历 `node.right`。
7. 从根节点 `root` 开始递归。
8. 返回结果数组。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        res = []

        def inorder(node):
            # 递归终止条件：空节点不需要处理
            if not node:
                return

            # 1. 遍历左子树
            inorder(node.left)

            # 2. 访问当前节点
            res.append(node.val)

            # 3. 遍历右子树
            inorder(node.right)

        inorder(root)
        return res
```

### 为什么递归顺序很重要？

对于二叉树 DFS，三种经典遍历方式的区别，本质上就是：

> **什么时候处理当前节点。**

```text
前序遍历 Preorder:
当前节点 → 左子树 → 右子树

中序遍历 Inorder:
左子树 → 当前节点 → 右子树

后序遍历 Postorder:
左子树 → 右子树 → 当前节点
```

也就是说，在代码中，`res.append(node.val)` 的位置决定了遍历类型。

中序遍历对应：

```python
inorder(node.left)
res.append(node.val)
inorder(node.right)
```

### 复杂度分析

假设二叉树共有 `n` 个节点，高度为 `h`。

- **时间复杂度：`O(n)`**
  - 每个节点恰好访问一次。
- **辅助空间复杂度：`O(h)`**
  - 主要来自递归调用栈。
  - 平衡二叉树中：`O(log n)`
  - 极端退化成链表时：`O(n)`
- **输出数组空间：`O(n)`**
  - 因为最终需要存储所有节点。

> 面试中通常把输出数组所需空间排除在“额外空间”之外，因此更准确的说法是：递归方法的 **auxiliary space 为 `O(h)`**，而不是统一写成 `O(n)`。

------

# 2. 迭代深度优先搜索（Iterative DFS）

### 思路

递归方法实际上依赖 Python 的**函数调用栈（call stack）**记录：

> 遍历完左子树以后，我应该回到哪个节点？

我们也可以自己维护一个 `stack`，显式模拟递归调用栈。

中序遍历要求：

```text
尽可能向左走
↓
没有左节点了
↓
处理当前节点
↓
进入右子树
↓
再次尽可能向左走
```

因此核心逻辑可以概括为：

```text
一路向左压栈
→ 弹出一个节点
→ 访问它
→ 转向它的右子树
```

### 算法步骤

1. 创建结果数组 `res`。

2. 创建栈 `stack`。

3. 使用 `cur` 指向当前节点，初始为 `root`。

4. 只要：

   - `cur` 不为空，或者
   - `stack` 不为空

   就继续遍历。

5. 当 `cur` 不为空时：

   - 将 `cur` 压入栈；
   - 然后令 `cur = cur.left`；
   - 不断向左走。

6. 当无法继续向左时：

   - 从栈中弹出一个节点；
   - 将其值加入结果；
   - 转向它的右子树。

7. 重复以上过程。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        res = []
        stack = []

        # cur 表示当前正在访问的节点
        cur = root

        # 只要还有节点尚未处理，就继续循环
        while cur or stack:

            # 第一阶段：
            # 不断向左走，并把经过的节点保存到栈中
            while cur:
                stack.append(cur)
                cur = cur.left

            # 当前已经无法继续向左
            # 栈顶节点就是接下来应该访问的节点
            cur = stack.pop()
            res.append(cur.val)

            # 当前节点处理完成后，进入它的右子树
            cur = cur.right

        return res
```

## 如何理解 `stack`？

假设有这样一棵树：

```text
        4
       /
      2
     /
    1
```

程序首先一路向左：

```text
stack = [4]
stack = [4, 2]
stack = [4, 2, 1]
```

此时 `1` 没有左孩子，所以弹出：

```text
访问 1
stack = [4, 2]
```

然后 `1` 没有右孩子，于是再次弹出：

```text
访问 2
stack = [4]
```

因此，栈本质上保存的是：

> **左子树遍历结束之后还需要回来处理的祖先节点。**

这正是递归调用栈所做的事情。

------

## `while cur or stack` 为什么必须同时检查两个条件？

一个非常重要的细节是：

```python
while cur or stack:
```

不能简单写成：

```python
while cur:
```

因为 `cur` 可能已经变成 `None`，但栈中仍然存在尚未访问的祖先节点。

例如：

```text
    2
   /
  1
```

一路向左之后：

```text
cur = None
stack = [2, 1]
```

虽然 `cur` 为空，但显然遍历还没有结束。

------

### 复杂度分析

- **时间复杂度：`O(n)`**
  - 每个节点最多：
    - 入栈一次；
    - 出栈一次。
  - 因此总操作数量仍然是线性的。
- **辅助空间复杂度：`O(h)`**
  - 栈中最多保存从根节点到某个叶子的路径。
  - 平衡树：`O(log n)`
  - 最坏情况：`O(n)`
- **输出空间：`O(n)`**

------

# 3. Morris Traversal

## 思路

前两个方法都需要额外空间记录：

> 遍历完左子树之后应该回到哪个节点？

递归使用的是：

```text
系统调用栈
```

迭代方法使用的是：

```text
自己维护的 stack
```

Morris Traversal 则提出了一个比较巧妙的想法：

> 能不能暂时修改树中的指针，让左子树自己“指回”当前节点？

这样就不再需要额外的栈。

Morris Traversal 可以将辅助空间降低到：

```text
O(1)
```

------

## 中序前驱（Inorder Predecessor）

理解 Morris Traversal 的关键是**中序前驱**。

假设当前节点为：

```text
cur
```

并且它存在左子树。

那么当前节点的中序前驱，就是：

> **左子树中最右边的节点。**

例如：

```text
        5
       /
      2
       \
        4
```

对于节点 `5`：

```text
左子树 = 以 2 为根的子树
```

其中最右边的节点是：

```text
4
```

因此：

```text
4 是 5 的中序前驱
```

在正常的中序遍历中：

```text
... → 4 → 5 → ...
```

所以 Morris Traversal 暂时令：

```python
4.right = 5
```

这样遍历完成左子树之后，就可以通过这条临时指针返回 `5`。

这条临时指针通常称为：

**thread（线索）**。

------

## 算法步骤

令当前节点为 `cur`。

### 情况一：`cur` 没有左孩子

如果：

```python
cur.left is None
```

由于中序遍历顺序是：

```text
左 → 当前 → 右
```

当前节点没有左子树，因此可以直接访问：

```python
res.append(cur.val)
```

然后进入右子树：

```python
cur = cur.right
```

------

### 情况二：`cur` 有左孩子

找到左子树中最右边的节点：

```python
prev = cur.left

while prev.right and prev.right != cur:
    prev = prev.right
```

这里有两种情况。

#### 第一次遇到 `cur`

如果：

```python
prev.right is None
```

说明左子树还没有遍历。

创建临时链接：

```python
prev.right = cur
```

然后进入左子树：

```python
cur = cur.left
```

------

#### 第二次遇到 `cur`

如果：

```python
prev.right == cur
```

说明：

```text
左子树已经遍历完成
```

于是恢复原来的树结构：

```python
prev.right = None
```

然后访问当前节点：

```python
res.append(cur.val)
```

最后进入右子树：

```python
cur = cur.right
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
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        res = []
        cur = root

        while cur:

            # 情况 1：
            # 当前节点没有左子树，因此可以直接访问
            if not cur.left:
                res.append(cur.val)
                cur = cur.right

            else:
                # 找到当前节点的中序前驱：
                # 左子树中最右边的节点
                prev = cur.left

                # prev.right == cur 表示这条临时链接
                # 是之前由 Morris Traversal 创建的
                while prev.right and prev.right != cur:
                    prev = prev.right

                # 第一次来到 cur：
                # 左子树还没有遍历
                if not prev.right:

                    # 创建 thread：
                    # 遍历完左子树之后可以通过它回到 cur
                    prev.right = cur

                    # 开始遍历左子树
                    cur = cur.left

                # 第二次回到 cur：
                # 说明左子树已经遍历完成
                else:
                    # 恢复原始树结构
                    prev.right = None

                    # 中序遍历此时才访问当前节点
                    res.append(cur.val)

                    # 继续遍历右子树
                    cur = cur.right

        return res
```

------

## Morris Traversal 为什么不会死循环？

注意这段代码：

```python
while prev.right and prev.right != cur:
    prev = prev.right
```

不能写成：

```python
while prev.right:
    prev = prev.right
```

原因是 Morris Traversal 自己创建了：

```python
prev.right = cur
```

如果第二次回来时没有检查：

```python
prev.right != cur
```

程序就会沿着临时链接继续向右走，形成循环。

因此：

```python
prev.right == cur
```

实际上是一个重要信号：

> **当前节点的左子树已经访问完成。**

------

## Morris Traversal 为什么仍然是 `O(n)`？

乍一看，我们对每个节点还需要：

```python
while prev.right:
```

寻找左子树最右节点，似乎可能达到 `O(n²)`。

实际上不会。

在整个 Morris Traversal 过程中，每条相关的树边最多只会被有限次数地经过：

- 创建 thread 时经过一次；
- 返回并删除 thread 时再经过一次。

因此所有操作累计仍然是线性的：

```text
O(n)
```

### 复杂度分析

- **时间复杂度：`O(n)`**
- **辅助空间复杂度：`O(1)`**
- **输出空间：`O(n)`**

需要注意：

> Morris Traversal 会**暂时修改原始树结构**。

虽然最终会恢复，但这也是它相对于递归和显式栈方法的重要区别。

------

# 三种方法对比

| 方法             | 时间复杂度 | 辅助空间 | 是否修改树 | 面试重要程度 |
| ---------------- | ---------- | -------- | ---------- | ------------ |
| 递归 DFS         | `O(n)`     | `O(h)`   | 否         | ★★★★★        |
| 迭代 DFS + Stack | `O(n)`     | `O(h)`   | 否         | ★★★★★        |
| Morris Traversal | `O(n)`     | `O(1)`   | 暂时修改   | ★★★          |

其中 `h` 表示树的高度。

对于平衡二叉树：

```text
h = O(log n)
```

对于完全退化的二叉树：

```text
h = O(n)
```

代码面试中建议优先掌握：

```text
递归 DFS
↓
显式 Stack
↓
Morris Traversal
```

其中第二种尤其重要，因为很多二叉树问题都会进一步询问：

> “如果不能使用递归，你会怎么做？”

------

# Python 中涉及的重要类和函数

## `TreeNode`

LeetCode 中的二叉树节点通常定义为：

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

每个节点包含三个主要属性：

```python
node.val
```

当前节点存储的值。

```python
node.left
```

左孩子。

```python
node.right
```

右孩子。

如果不存在对应孩子，则为：

```python
None
```

------

## `Optional[TreeNode]`

函数定义：

```python
def inorderTraversal(
    self,
    root: Optional[TreeNode]
) -> List[int]:
```

这里的：

```python
Optional[TreeNode]
```

来自 Python 的 `typing` 模块。

它表示：

```python
TreeNode | None
```

也就是说 `root`：

- 可能是一个 `TreeNode`
- 也可能是 `None`

大致等价于现代 Python 中的：

```python
root: TreeNode | None
```

------

## `List[int]`

```python
List[int]
```

表示：

> 一个元素全部为整数的列表。

例如：

```python
[1, 2, 3]
```

现代 Python 也经常写成：

```python
list[int]
```

------

## `list.append()`

```python
res.append(node.val)
```

表示将一个元素加入列表末尾。

例如：

```python
res = [1, 2]

res.append(3)

print(res)
# [1, 2, 3]
```

单次 `append()` 的平均时间复杂度为：

```text
O(1)
```

------

## `list.pop()`

迭代解法中：

```python
cur = stack.pop()
```

`pop()` 默认删除并返回列表最后一个元素。

例如：

```python
stack = [1, 2, 3]

x = stack.pop()

print(x)
# 3

print(stack)
# [1, 2]
```

因此 Python 的 `list` 很适合实现一个栈：

```python
stack.append(x)   # push
stack.pop()       # pop
```

这两个操作的平均时间复杂度都是：

```text
O(1)
```

不要使用：

```python
stack.pop(0)
```

来实现普通栈，因为删除列表开头元素需要移动后面的元素，时间复杂度为：

```text
O(n)
```

------

# 常见错误

## 1. 递归时访问顺序写错

中序遍历必须是：

```python
inorder(node.left)
res.append(node.val)
inorder(node.right)
```

如果写成：

```python
res.append(node.val)
inorder(node.left)
inorder(node.right)
```

得到的是：

```text
前序遍历
```

如果写成：

```python
inorder(node.left)
inorder(node.right)
res.append(node.val)
```

得到的是：

```text
后序遍历
```

一个很好记的方法是看：

```python
res.append(node.val)
```

出现在什么位置。

------

## 2. 迭代解法忘记进入右子树

弹出节点并访问以后：

```python
cur = stack.pop()
res.append(cur.val)
```

必须继续：

```python
cur = cur.right
```

否则无法正确访问右子树。

完整逻辑应该始终记成：

```text
向左到底
→ pop
→ visit
→ 转向右子树
```

------

## 3. Morris Traversal 忘记恢复树结构

创建临时链接：

```python
prev.right = cur
```

之后，第二次返回当前节点时必须删除：

```python
prev.right = None
```

否则函数执行完成以后，原始二叉树结构就会被破坏。

------

## 4. Morris Traversal 过早访问当前节点

对于**中序遍历**，如果当前节点存在左子树，那么第一次遇到当前节点时不能立即访问。

因为访问顺序必须是：

```text
左子树
→ 当前节点
→ 右子树
```

因此 Morris Traversal 中：

```python
if not prev.right:
    prev.right = cur
    cur = cur.left
```

这里只创建临时链接，**不访问 `cur`**。

只有第二次回来时：

```python
else:
    prev.right = None
    res.append(cur.val)
```

才访问当前节点。

------

# 面试中的核心理解

这道题本身并不难，但它非常适合建立二叉树遍历的基础框架。

建议重点记住下面三个模板。

### 递归模板

```python
def inorder(node):
    if not node:
        return

    inorder(node.left)
    process(node)
    inorder(node.right)
```

### 迭代模板

```python
while cur or stack:
    while cur:
        stack.append(cur)
        cur = cur.left

    cur = stack.pop()
    process(cur)

    cur = cur.right
```

### Morris Traversal 核心思想

```text
没有左子树
→ 直接访问当前节点

有左子树
→ 找左子树最右节点

第一次遇到
→ 建立 predecessor.right → cur 的临时链接

第二次遇到
→ 删除临时链接
→ 访问 cur
```

如果是普通代码面试，**递归解法和显式栈解法应该做到能够直接默写**。Morris Traversal 更适合作为空间优化方法理解，它的关键不在背代码，而在理解：

> 利用当前节点的中序前驱建立临时返回路径，从而用树本身代替调用栈。
