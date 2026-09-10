# 将有序数组转换为二叉搜索树

给定一个整数数组 `nums`，其中元素按**升序排列**，请将其转换为一棵**高度平衡（height-balanced）二叉搜索树（BST）**。

一棵二叉树是**高度平衡**的，当且仅当对于树中的每一个节点，其左子树和右子树的高度差不超过 `1`。

------

## 核心思路

这道题同时利用了两个性质：

1. **二叉搜索树（BST）**
   - 左子树中的所有节点值都小于当前节点。
   - 右子树中的所有节点值都大于当前节点。
2. **高度平衡**
   - 希望左右子树包含的节点数量尽可能接近。

由于 `nums` 本身已经按升序排列，因此最自然的做法是：

- 选择当前区间的**中间元素**作为根节点。
- 中间元素左边的部分构造左子树。
- 中间元素右边的部分构造右子树。
- 对左右两部分重复相同过程。

例如：

```text
nums = [-10, -3, 0, 5, 9]
              ↑
            中间元素
```

选择 `0` 作为根节点：

```text
          0
        /   \
   [-10,-3] [5,9]
```

然后继续分别取左右区间的中间元素。

这种“取中点 + 递归划分”的结构本质上类似于**二分查找**和**分治算法（Divide and Conquer）**。

------

# 1. DFS：数组切片

## 思路

每次选择数组中间元素作为当前节点，然后直接使用 Python 的数组切片：

```python
nums[:mid]
nums[mid + 1:]
```

分别构造左右子树。

这种写法非常直观，也是最容易想到的方法。

## 算法步骤

1. 如果当前数组为空，返回 `None`。
2. 找到数组中间下标 `mid`。
3. 创建值为 `nums[mid]` 的根节点。
4. 使用 `nums[:mid]` 递归构造左子树。
5. 使用 `nums[mid + 1:]` 递归构造右子树。
6. 返回根节点。

## Python 代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def sortedArrayToBST(
        self,
        nums: List[int]
    ) -> Optional[TreeNode]:

        # Base Case：
        # 当前数组为空，说明这里没有节点
        if not nums:
            return None

        # 选择中间元素作为根节点
        mid = len(nums) // 2
        root = TreeNode(nums[mid])

        # 中间元素左边的部分构造左子树
        root.left = self.sortedArrayToBST(nums[:mid])

        # 中间元素右边的部分构造右子树
        root.right = self.sortedArrayToBST(nums[mid + 1:])

        return root
```

## 为什么一定是平衡的？

每次选择中间元素之后，左右两边的元素数量最多只会相差 `1`。

例如长度为 `7`：

```text
[0, 1, 2, 3, 4, 5, 6]
          ↑

左边 3 个
右边 3 个
```

长度为 `6`：

```text
[0, 1, 2, 3, 4, 5]
       ↑

左边 2 个
右边 3 个
```

因此不断递归之后，左右子树高度最多只会相差 `1`。

## 时间复杂度

这里有一个非常重要的 Python 细节。

虽然一共只创建了 `n` 个节点，但：

```python
nums[:mid]
nums[mid + 1:]
```

会创建新的 `list`。

一次切片本身需要复制元素。

递归关系大致为：

```text
T(n) = 2T(n/2) + O(n)
```

因此：

**时间复杂度：**

```text
O(n log n)
```

### 空间复杂度

递归深度约为：

```text
O(log n)
```

但由于不断创建数组切片，总的额外切片空间可能达到：

```text
O(n log n)
```

如果只讨论某一时刻同时存在的切片和递归调用，通常也可以得到较小的界；但从面试角度来说，关键结论是：

> 这种方法会产生不必要的数组复制，因此不是最优实现。

此外，最终生成的树本身需要：

```text
O(n)
```

空间。

------

# 2. DFS + 左右边界（推荐 / 最优）

这是这道题最推荐掌握的方法。

## 思路

上一种方法真正的问题并不是递归，而是：

```python
nums[:mid]
```

会复制数组。

实际上我们完全不需要创建新的数组，只需要记录：

```text
当前处理 nums 的哪一段
```

因此定义：

```python
helper(left, right)
```

表示：

> 使用 `nums[left:right + 1]` 构造一棵高度平衡 BST。

例如：

```text
nums = [-10, -3, 0, 5, 9]

helper(0, 4)
```

找到：

```python
mid = (0 + 4) // 2 = 2
```

因此：

```text
nums[2] = 0
```

成为根节点。

随后：

```python
helper(0, 1)   # 左子树
helper(3, 4)   # 右子树
```

整个过程中始终使用原数组，没有任何切片。

------

## 算法步骤

1. 定义辅助函数 `helper(left, right)`。
2. 如果：

```python
left > right
```

说明当前区间为空，返回 `None`。

3. 找到当前区间的中点：

```python
mid = (left + right) // 2
```

1. 创建节点：

```python
root = TreeNode(nums[mid])
```

1. 构造左子树：

```python
root.left = helper(left, mid - 1)
```

1. 构造右子树：

```python
root.right = helper(mid + 1, right)
```

1. 返回当前根节点。
2. 最终调用：

```python
helper(0, len(nums) - 1)
```

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
    def sortedArrayToBST(
        self,
        nums: List[int]
    ) -> Optional[TreeNode]:

        def helper(left: int, right: int) -> Optional[TreeNode]:

            # Base Case：
            # left > right 表示当前区间为空
            if left > right:
                return None

            # 选择当前区间的中间元素
            mid = (left + right) // 2

            # 中间元素作为当前子树的根节点
            root = TreeNode(nums[mid])

            # nums[left : mid] 构造左子树
            root.left = helper(left, mid - 1)

            # nums[mid + 1 : right + 1] 构造右子树
            root.right = helper(mid + 1, right)

            return root

        return helper(0, len(nums) - 1)
```

------

## 递归过程示例

假设：

```python
nums = [-10, -3, 0, 5, 9]
```

第一次：

```text
left = 0
right = 4
mid = 2

root = 0
           0
```

左边：

```text
[-10, -3]
```

右边：

```text
[5, 9]
```

继续递归：

```text
helper(0, 1)
mid = 0
root = -10
```

以及：

```text
helper(3, 4)
mid = 3
root = 5
```

最终可能得到：

```text
        0
       / \
    -10   5
      \    \
      -3    9
```

这是一棵合法的高度平衡 BST。

注意：答案**不唯一**。

例如下面这样的树也可能合法：

```text
        0
       / \
     -3   9
     /   /
   -10  5
```

------

## 时间复杂度

每个数组元素只会被访问一次，并创建一个对应的 `TreeNode`：

```text
O(n)
```

因此：

**时间复杂度：**

```text
O(n)
```

------

## 空间复杂度

因为每次都从中间划分，所以递归树的高度约为：

```text
log₂ n
```

递归调用栈：

```text
O(log n)
```

如果**不计算最终返回的树**：

```text
O(log n)
```

如果把最终生成的树也计算在空间复杂度中：

```text
O(n)
```

因为最终需要创建 `n` 个节点。

面试中通常更推荐这样表达：

> Auxiliary Space（额外空间）：`O(log n)`
> Output Space（输出树）：`O(n)`

------

# 3. 迭代 DFS

## 思路

递归实际上依赖 Python 的**函数调用栈（Call Stack）**。

例如：

```python
helper(left, right)
```

调用另一个：

```python
helper(left, mid - 1)
```

Python 会自动保存当前函数的状态。

如果不使用递归，我们可以自己维护一个：

```python
stack
```

来模拟递归过程。

栈中每个元素保存：

```text
(node, left, right)
```

表示：

> `node` 需要使用 `nums[left:right + 1]` 这一段进行填充。

------

## 算法步骤

1. 如果数组为空，返回 `None`。
2. 创建一个临时根节点。
3. 将：

```text
(root, 0, n - 1)
```

放入栈。

4. 不断从栈顶取出：

```text
(node, left, right)
```

1. 找到中间下标 `mid`。
2. 将：

```python
nums[mid]
```

赋给当前节点。

7. 如果左区间存在：

- 创建左孩子；
- 将左孩子及其区间加入栈。

1. 如果右区间存在：
   - 创建右孩子；
   - 将右孩子及其区间加入栈。
2. 栈为空后返回根节点。

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
    def sortedArrayToBST(
        self,
        nums: List[int]
    ) -> Optional[TreeNode]:

        if not nums:
            return None

        # 先创建一个占位根节点
        # 后面再根据区间中点修改它的值
        root = TreeNode(0)

        # 每个元素：
        # (需要填充的节点, 左边界, 右边界)
        stack = [(root, 0, len(nums) - 1)]

        while stack:

            # DFS 使用 stack.pop()
            node, left, right = stack.pop()

            # 找到当前区间的中点
            mid = (left + right) // 2

            # 当前节点使用中间元素
            node.val = nums[mid]

            # 如果左区间非空
            if left <= mid - 1:
                node.left = TreeNode(0)

                stack.append(
                    (node.left, left, mid - 1)
                )

            # 如果右区间非空
            if mid + 1 <= right:
                node.right = TreeNode(0)

                stack.append(
                    (node.right, mid + 1, right)
                )

        return root
```

------

## 为什么 `stack.pop()` 是 DFS？

Python：

```python
list.pop()
```

默认删除并返回列表的**最后一个元素**。

因此：

```python
stack.append(x)
stack.pop()
```

形成的是：

```text
LIFO
Last In, First Out
后进先出
```

这正是栈的特点，因此可以模拟 DFS。

例如：

```python
stack = []

stack.append("A")
stack.append("B")
stack.append("C")

stack.pop()
```

得到：

```text
"C"
```

------

## 时间复杂度

每个节点只被处理一次：

```text
O(n)
```

------

## 空间复杂度

栈主要保存尚未处理的子树。

由于树是高度平衡的，栈大小通常为：

```text
O(log n)
```

因此：

**额外空间：**

```text
O(log n)
```

**输出树：**

```text
O(n)
```

------

# 递归 DFS 与迭代 DFS 对比

| 方法           | 时间复杂度   | 额外空间   | 特点                   |
| -------------- | ------------ | ---------- | ---------------------- |
| DFS + 数组切片 | `O(n log n)` | 较高       | 最直观，但存在数组复制 |
| DFS + 左右边界 | `O(n)`       | `O(log n)` | **最推荐**             |
| Iterative DFS  | `O(n)`       | `O(log n)` | 避免递归，但代码更复杂 |

面试中优先推荐：

```text
DFS + 左右边界
```

因为它：

- 思路简单；
- 代码短；
- 时间复杂度最优；
- 没有数组切片；
- 非常容易证明正确性。

------

# Python / 数据结构知识点

## 1. `TreeNode`

LeetCode 中通常已经定义：

```python
class TreeNode:
    def __init__(
        self,
        val=0,
        left=None,
        right=None
    ):
        self.val = val
        self.left = left
        self.right = right
```

它表示二叉树中的一个节点。

例如：

```python
node = TreeNode(5)
```

对应：

```text
    5
   / \
None None
```

可以添加左右节点：

```python
node.left = TreeNode(3)
node.right = TreeNode(7)
```

得到：

```text
    5
   / \
  3   7
```

------

## 2. `Optional[TreeNode]`

代码中：

```python
Optional[TreeNode]
```

来自 Python 的 `typing` 模块。

它表示返回值可能是：

```python
TreeNode
```

也可能是：

```python
None
```

近似理解为：

```text
TreeNode | None
```

现代 Python 也可以写成：

```python
TreeNode | None
```

------

## 3. `//` 整数除法

```python
mid = (left + right) // 2
```

Python 中：

```python
//
```

表示整数除法。

例如：

```python
5 // 2
```

结果为：

```python
2
```

这非常适合计算数组中间下标。

------

# 常见错误

## 1. 左右边界 Off-by-One Error

这是本题最常见的错误。

假设：

```python
mid = (left + right) // 2
```

`nums[mid]` 已经被用作当前根节点，因此不能再次包含在左右子树中。

### 错误

```python
root.left = helper(left, mid)
```

这里把 `mid` 又包含到了左子树。

在某些情况下区间不会缩小，从而产生无限递归。

### 正确

```python
root.left = helper(left, mid - 1)
root.right = helper(mid + 1, right)
```

核心原则：

```text
left subtree  → [left, mid - 1]

root           → mid

right subtree → [mid + 1, right]
```

------

# 2. Base Case 写错

正确的终止条件是：

```python
if left > right:
    return None
```

为什么不是：

```python
left == right
```

就结束？

因为当：

```python
left == right
```

时，当前区间实际上还有一个元素。

例如：

```text
left = 3
right = 3
```

对应：

```python
nums[3]
```

这个元素应该被创建为一个叶子节点。

因此：

```text
left == right
```

仍然要继续执行。

只有：

```text
left > right
```

才表示区间真正为空。

------

# 3. 偶数长度数组应该选择哪个中点？

例如：

```text
[1, 2, 3, 4]
```

严格来说有两个“中间元素”：

```text
2, 3
```

可以选择偏左中点：

```python
mid = (left + right) // 2
```

也可以选择偏右中点：

```python
mid = (left + right + 1) // 2
```

两种方式都能生成合法的高度平衡 BST。

例如选择 `2`：

```text
      2
     / \
    1   3
         \
          4
```

选择 `3`：

```text
      3
     / \
    1   4
     \
      2
```

两棵树都满足高度平衡要求。

因此这道题的答案并不唯一。

面试中一般直接使用：

```python
mid = (left + right) // 2
```

即可。

------

# 关键算法总结

这道题最重要的是看到：

```text
Sorted Array
      +
Balanced BST
      ↓
Middle Element as Root
```

因为数组已经有序：

```text
左半部分 < 中间元素 < 右半部分
```

天然满足 BST 的顺序要求。

而选择中间元素又能保证：

```text
左边节点数量 ≈ 右边节点数量
```

从而保证树的高度平衡。

因此可以记住这个模板：

```python
def build(left, right):
    if left > right:
        return None

    mid = (left + right) // 2

    root = TreeNode(nums[mid])

    root.left = build(left, mid - 1)
    root.right = build(mid + 1, right)

    return root
```

它实际上是一个非常典型的**分治（Divide and Conquer）模板**：

```text
1. Divide
   从中点把问题分成左右两部分

2. Conquer
   递归解决左右子问题

3. Combine
   将左右子树连接到当前根节点
```

------

# 面试推荐版本

```python
class Solution:
    def sortedArrayToBST(
        self,
        nums: List[int]
    ) -> Optional[TreeNode]:

        def build(left, right):

            # 当前区间为空
            if left > right:
                return None

            # 中间元素作为根节点
            mid = (left + right) // 2
            root = TreeNode(nums[mid])

            # 分治构造左右子树
            root.left = build(left, mid - 1)
            root.right = build(mid + 1, right)

            return root

        return build(0, len(nums) - 1)
```

**时间复杂度：**

```text
O(n)
```

**额外空间复杂度：**

```text
O(log n)
```

**核心记忆：**

```text
有序数组 → 取中点 → 左右递归 → 平衡 BST
```
