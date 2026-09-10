# 从前序遍历与中序遍历构造二叉树

给定两个整数数组 `preorder` 和 `inorder`：

- `preorder`：某棵二叉树的**前序遍历**
- `inorder`：同一棵二叉树的**中序遍历**
- 两个数组长度相同，并且所有节点值**互不相同**

请根据前序遍历和中序遍历重新构造这棵二叉树，并返回其根节点。

------

## 一、核心思路

这道题最关键的是理解两种遍历顺序：

```
前序遍历（Preorder）：
根 -> 左子树 -> 右子树

中序遍历（Inorder）：
左子树 -> 根 -> 右子树
```

因此：

> **前序遍历负责告诉我们“当前根节点是谁”，中序遍历负责告诉我们“左右子树的边界在哪里”。**

例如：

```
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]
```

因为 `preorder[0] = 3`，所以 `3` 一定是整棵树的根节点。

然后在 `inorder` 中找到 `3`：

```
              root
                ↓
inorder = [9,   3,   15, 20, 7]
           ↑          ↑
        左子树       右子树
```

所以：

```
左子树 inorder = [9]
右子树 inorder = [15, 20, 7]
```

左子树一共有 1 个节点，因此可以继续确定 `preorder`：

```
preorder = [3, 9, 20, 15, 7]
            ↑  ↑   ↑
           root 左   右子树开始
```

于是：

```
左子树 preorder = [9]
右子树 preorder = [20, 15, 7]
```

接下来对左右两部分重复同样的过程即可。

最终得到：

```
        3
       / \
      9   20
         /  \
        15   7
```

这也是这道题所有递归解法的基础。

------

# 方法一：DFS + 数组切片

## 思路

`preorder` 的第一个元素一定是当前子树的根节点。

找到这个根节点在 `inorder` 中的位置 `mid` 后，就可以把当前问题拆分为：

```
            root
           /    \
      left       right
```

由于 `inorder` 的结构是：

```
[left subtree] + root + [right subtree]
```

因此：

```
inorder[:mid]       # 左子树
inorder[mid + 1:]   # 右子树
```

如果 `mid` 左边有 `mid` 个节点，那么前序遍历中根节点之后的 `mid` 个节点，也一定属于左子树：

```
preorder[1 : mid + 1]
```

剩下的属于右子树：

```
preorder[mid + 1 :]
```

然后递归构造即可。

------

## 算法步骤

1. 如果 `preorder` 或 `inorder` 为空，说明当前子树不存在，返回 `None`。
2. `preorder[0]` 就是当前子树的根节点。
3. 在 `inorder` 中找到根节点的位置 `mid`。
4. 根据 `mid` 将两个数组分别划分为左子树和右子树。
5. 递归构造左子树。
6. 递归构造右子树。
7. 返回根节点。

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
    def buildTree(
        self,
        preorder: List[int],
        inorder: List[int]
    ) -> Optional[TreeNode]:

        # Base Case：
        # 没有节点时，当前子树为空
        if not preorder or not inorder:
            return None

        # 前序遍历的第一个节点一定是根节点
        root_val = preorder[0]
        root = TreeNode(root_val)

        # 找到根节点在中序遍历中的位置
        mid = inorder.index(root_val)

        # mid 左侧共有 mid 个节点，因此 preorder 中
        # 根节点之后的 mid 个节点属于左子树
        root.left = self.buildTree(
            preorder[1 : mid + 1],
            inorder[:mid]
        )

        # 剩余节点属于右子树
        root.right = self.buildTree(
            preorder[mid + 1 :],
            inorder[mid + 1 :]
        )

        return root
```

------

## Python：`list.index()`

这里使用了：

```
mid = inorder.index(root_val)
```

`list.index(x)` 会从左向右寻找元素 `x`，并返回它第一次出现的位置。

例如：

```
nums = [10, 20, 30]

nums.index(20)
# 1
```

需要注意，它并不是 `O(1)` 操作。

对于长度为 `n` 的列表：

```
list.index() = O(n)
```

这也是方法一最终达到 `O(n²)` 时间复杂度的重要原因。

------

## Python：数组切片

例如：

```
nums = [1, 2, 3, 4, 5]

nums[1:3]
# [2, 3]
```

Python 的切片采用：

```
[start, end)
```

也就是**左闭右开**。

因此：

```
preorder[1 : mid + 1]
```

包含索引：

```
1, 2, ..., mid
```

总共正好 `mid` 个元素。

另外，Python 的列表切片会创建新的列表，因此切片本身也需要额外的时间和空间。

------

## 复杂度

**时间复杂度：**

```
O(n²)
```

在最坏情况下，例如树退化成链表：

```
1
 \
  2
   \
    3
     \
      4
```

每次递归都需要：

```
inorder.index(...)
```

于是总搜索成本约为：

```
n + (n - 1) + (n - 2) + ... + 1
= O(n²)
```

**空间复杂度：**

```
O(n)
```

递归栈最坏深度为 `O(n)`。

严格考虑 Python 数组切片产生的临时列表时，最坏情况下还可能产生较大的额外切片开销。因此面试中更推荐下面的方法二。

------

# 方法二：哈希表 + DFS【推荐】

这是最适合作为面试主解法的方法。

## 思路

方法一有两个可以优化的问题：

```
1. inorder.index(root) 每次都需要 O(n) 搜索
2. 数组切片会不断创建新的 list
```

首先，我们可以提前建立：

```
value -> inorder 中的 index
```

例如：

```
inorder = [9, 3, 15, 20, 7]

indices = {
    9:  0,
    3:  1,
    15: 2,
    20: 3,
    7:  4
}
```

之后：

```
mid = indices[root_val]
```

就是平均 `O(1)` 查询。

其次，我们不再真正切割 `inorder`，而是使用：

```
dfs(left, right)
```

表示：

> 构造 `inorder[left:right+1]` 所对应的子树。

------

## 为什么只需要维护 `inorder` 的边界？

这是这个解法非常值得理解的一点。

我们维护一个：

```
pre_idx
```

它始终表示：

> `preorder` 中下一个应该被创建的节点。

由于前序遍历天然按照：

```
root -> left -> right
```

排列，所以只要我们严格按照：

```
root.left = dfs(...)
root.right = dfs(...)
```

的顺序递归，`pre_idx` 就会自动正确地向前移动。

因此不需要再维护 `preorder` 的左右边界。

------

## 算法步骤

1. 使用哈希表记录 `inorder` 中每个节点的位置。
2. 使用 `pre_idx` 指向 `preorder` 中下一个待处理节点。
3. 定义 `dfs(l, r)`，表示构造中序遍历区间 `[l, r]`。
4. 如果 `l > r`，当前区间为空，返回 `None`。
5. `preorder[pre_idx]` 是当前根节点。
6. `pre_idx += 1`。
7. 使用哈希表找到根节点在 `inorder` 中的位置 `mid`。
8. 递归构造 `[l, mid - 1]`。
9. 递归构造 `[mid + 1, r]`。
10. 返回根节点。

------

## Python 实现

```
class Solution:
    def buildTree(
        self,
        preorder: List[int],
        inorder: List[int]
    ) -> Optional[TreeNode]:

        # value -> 该节点在 inorder 中的位置
        # enumerate(inorder) 会产生 (index, value)
        indices = {
            value: index
            for index, value in enumerate(inorder)
        }

        # preorder 中下一个需要处理的位置
        self.pre_idx = 0

        def dfs(left: int, right: int) -> Optional[TreeNode]:
            # inorder 区间为空
            if left > right:
                return None

            # preorder 按 root -> left -> right 排列，
            # 所以下一个未处理元素就是当前子树的根
            root_val = preorder[self.pre_idx]
            self.pre_idx += 1

            root = TreeNode(root_val)

            # O(1) 找到根节点在 inorder 中的位置
            mid = indices[root_val]

            # 必须先构造左子树
            root.left = dfs(left, mid - 1)

            # 再构造右子树
            root.right = dfs(mid + 1, right)

            return root

        return dfs(0, len(inorder) - 1)
```

------

## Python：`enumerate()`

代码：

```
for index, value in enumerate(inorder):
```

`enumerate()` 可以同时得到：

```
索引 + 元素
```

例如：

```
nums = [10, 20, 30]

for i, value in enumerate(nums):
    print(i, value)
```

得到：

```
0 10
1 20
2 30
```

因此：

```
indices = {
    value: index
    for index, value in enumerate(inorder)
}
```

是一个字典推导式，用于建立：

```
节点值 -> inorder index
```

------

## 复杂度

**时间复杂度：**

```
O(n)
```

原因：

- 建立哈希表：`O(n)`
- 每个节点只创建一次
- 哈希表查询平均 `O(1)`

因此：

```
O(n) + O(n) = O(n)
```

**空间复杂度：**

```
O(n)
```

主要来自：

- 哈希表：`O(n)`
- 递归栈：最坏 `O(n)`

------

# 方法三：双指针 + DFS

## 思路

还可以进一步做到：

- 不切片
- 不使用哈希表
- 时间复杂度仍然为 `O(n)`

我们维护两个指针：

```
pre_idx
in_idx
```

分别表示：

```
pre_idx：preorder 中下一个要创建的节点
in_idx：inorder 中当前处理到的位置
```

然后给递归函数一个 `limit`：

```
dfs(limit)
```

它的含义可以理解为：

> 不断构造当前子树，直到 `inorder[in_idx]` 遇到 `limit` 为止。

------

## `limit` 到底是什么？

这是这个解法最难理解的地方。

假设当前创建了：

```
root = preorder[pre_idx]
```

根据中序遍历：

```
左子树 -> root -> 右子树
```

所以在构造 `root.left` 时：

```
root.left = dfs(root.val)
```

`root.val` 就相当于左子树的**结束标记**。

当：

```
inorder[in_idx] == root.val
```

说明：

```
左子树已经全部构造完毕
```

此时停止构造左子树。

------

## 一个重要修正

原解析中写道：

> nodes less than root appear before it in inorder

这句话是不正确的。

这里完全**不涉及节点值的大小关系**。

二叉树并不一定是二叉搜索树（BST），所以不能认为：

```
左边的节点 < root
右边的节点 > root
```

正确理解应该是：

> 在中序遍历序列中，属于左子树的节点出现在根节点之前。

这是**遍历顺序关系**，不是**数值大小关系**。

------

## Python 实现

```
class Solution:
    def buildTree(
        self,
        preorder: List[int],
        inorder: List[int]
    ) -> Optional[TreeNode]:

        # preorder 中下一个待创建节点
        pre_idx = 0

        # inorder 中当前处理到的位置
        in_idx = 0

        def dfs(limit):
            nonlocal pre_idx, in_idx

            # preorder 已经全部处理完成
            if pre_idx >= len(preorder):
                return None

            # inorder 遇到当前子树的结束标记
            if inorder[in_idx] == limit:
                in_idx += 1
                return None

            # preorder 的下一个元素就是当前根节点
            root = TreeNode(preorder[pre_idx])
            pre_idx += 1

            # 构造左子树。
            # 当 inorder 遇到 root.val 时，
            # 说明左子树构造完成。
            root.left = dfs(root.val)

            # 构造右子树。
            # 右子树继承当前子树原来的结束标记。
            root.right = dfs(limit)

            return root

        # 根节点没有真正的父节点，
        # 因此使用一个不可能出现在树中的值作为 limit
        return dfs(float("inf"))
```

------

## Python：`nonlocal`

这里：

```
pre_idx = 0
in_idx = 0

def dfs(limit):
    nonlocal pre_idx, in_idx
```

`pre_idx` 和 `in_idx`：

- 不是 `dfs()` 内部的局部变量
- 也不是全局变量
- 而是外层 `buildTree()` 函数中的变量

所以需要使用：

```
nonlocal
```

告诉 Python：

> 我想修改外层函数作用域中的变量，而不是创建一个新的局部变量。

例如：

```
def outer():
    x = 0

    def inner():
        nonlocal x
        x += 1
```

如果没有 `nonlocal x`，执行：

```
x += 1
```

会因为 Python 将 `x` 判断为局部变量而出现错误。

------

## Python：`float("inf")`

```
float("inf")
```

表示正无穷：

```
+∞
```

因为题目中的节点值都是普通整数，所以：

```
float("inf")
```

不会与任何节点值相等，可以作为根节点的“永远不会遇到的结束标记”。

更严谨地说，这里需要的并不是“比所有节点都大”，而只是：

> 一个不会与任何节点值相等的 sentinel（哨兵值）。

------

## 复杂度

**时间复杂度：**

```
O(n)
```

每个节点只处理常数次。

**额外空间复杂度：**

```
O(h)
```

其中 `h` 是树高，来自递归调用栈。

- 平衡二叉树：`O(log n)`
- 最坏退化成链表：`O(n)`

不考虑输出树本身，这个方法没有额外的 `O(n)` 哈希表。

------

# 方法四：Morris 风格的 O(1) 额外空间构造

## 思路

这个方法的目标是：

```
时间：O(n)
额外空间：O(1)
```

不使用：

- 哈希表
- 递归调用栈
- 显式栈

核心技巧是：

> 在构造过程中，临时借用节点的 `right` 指针保存“父节点引用”，从而模拟递归调用栈。

处理完对应子树之后，再把临时指针恢复。

这个方法的思想比较巧，但代码可读性明显低于前面的递归方法。代码面试中，除非面试官明确要求 `O(1)` auxiliary space，否则通常不建议把它作为第一解法。

------

## Python 实现

```
class Solution:
    def buildTree(
        self,
        preorder: List[int],
        inorder: List[int]
    ) -> Optional[TreeNode]:

        # Dummy Node：
        # head.right 最终会指向真正的根节点
        head = TreeNode(None)

        curr = head

        # i：preorder 指针
        # j：inorder 指针
        i = 0
        j = 0
        n = len(preorder)

        while i < n and j < n:

            # 创建 preorder 中的下一个节点。
            #
            # curr.right 暂时用于保存父节点引用，
            # 相当于模拟递归调用栈。
            curr.right = TreeNode(
                preorder[i],
                right=curr.right
            )

            curr = curr.right
            i += 1

            # 如果当前节点还没有到达 inorder[j]，
            # 说明仍然需要继续进入左子树。
            while i < n and curr.val != inorder[j]:
                curr.left = TreeNode(
                    preorder[i],
                    right=curr  # 临时保存父节点
                )

                curr = curr.left
                i += 1

            # 当前节点与 inorder[j] 对应，
            # 说明当前左侧路径已经处理完成
            j += 1

            # 沿临时 right 指针向上回溯
            while (
                curr.right
                and j < n
                and curr.right.val == inorder[j]
            ):
                parent = curr.right

                # 删除之前建立的临时父节点链接
                curr.right = None

                # 回到父节点
                curr = parent

                j += 1

        # Dummy Node 的右节点是真正的根
        return head.right
```

------

## Dummy Node 技巧

代码：

```
head = TreeNode(None)
```

创建了一个并不属于最终二叉树的虚拟节点。

这种节点通常称为：

```
Dummy Node
Sentinel Node
虚拟节点 / 哨兵节点
```

它的作用通常是简化边界情况。

最终：

```
return head.right
```

返回真正的根节点即可。

Dummy Node 在链表题中尤其常见，例如：

```
Merge Two Sorted Lists
Remove Nth Node From End
Swap Nodes in Pairs
```

遇到链表题时非常值得考虑。

------

## 复杂度

**时间复杂度：**

```
O(n)
```

每个节点只会被有限次数地访问。

**额外空间复杂度：**

```
O(1)
```

这里指的是不考虑最终返回的二叉树本身。

输出的树本身当然需要：

```
O(n)
```

空间。

------

# 四种方法对比

| 方法           | 时间复杂度 | 额外空间 | 特点                  |
| -------------- | ---------- | -------- | --------------------- |
| DFS + 切片     | `O(n²)`    | 最坏较高 | 最直观，适合理解      |
| Hash Map + DFS | `O(n)`     | `O(n)`   | **最推荐面试使用**    |
| 双指针 + DFS   | `O(n)`     | `O(h)`   | 巧妙，不需要 Hash Map |
| Morris 风格    | `O(n)`     | `O(1)`   | 空间最优，但实现复杂  |

其中 `h` 为树的高度。

对于一般代码面试，推荐掌握顺序：

```
方法一：理解递归逻辑
        ↓
方法二：作为标准面试答案
        ↓
方法三：作为进一步空间优化
        ↓
方法四：了解即可
```

------

# 常见错误

## 1. 数组切片的 Off-by-One Error

假设：

```
mid = inorder.index(root_val)
```

因为根节点左边有：

```
mid
```

个节点，所以 `preorder` 中左子树必须取：

```
preorder[1 : mid + 1]
```

而不是：

```
preorder[1 : mid]
```

即：

```
# ❌ 错误
preorder[1:mid]

# ✅ 正确
preorder[1:mid + 1]
```

原因是 Python 切片右边界不包含在结果中。

------

## 2. 使用全局 `preorder` 指针时先构造右子树

下面是错误的：

```
root.right = dfs(...)
root.left = dfs(...)
```

因为前序遍历的顺序是：

```
root -> left -> right
```

所以必须：

```
root.left = dfs(...)
root.right = dfs(...)
```

否则 `pre_idx` 会提前消费本来属于左子树的节点。

------

## 3. 混淆 `preorder` 和 `inorder` 的作用

牢记一句话：

```
preorder 找 root
inorder 分左右
```

也就是：

```
preorder：
告诉我们“下一个根节点是谁”

inorder：
告诉我们“哪些节点属于左子树，哪些属于右子树”
```

这是整道题最核心的规律。

------

## 4. 把普通二叉树误认为 BST

题目只是说：

```
Binary Tree
```

并没有说：

```
Binary Search Tree
```

因此不能认为：

```
left < root < right
```

这里所有左右子树的判断都来自**遍历顺序**，而不是节点值大小。

------

# 面试时最值得写出的版本

如果面试官没有提出额外的空间限制，优先写：

```
class Solution:
    def buildTree(
        self,
        preorder: List[int],
        inorder: List[int]
    ) -> Optional[TreeNode]:

        # inorder 中：节点值 -> index
        inorder_index = {
            value: i
            for i, value in enumerate(inorder)
        }

        # 指向 preorder 中下一个要创建的节点
        pre_idx = 0

        def dfs(left: int, right: int) -> Optional[TreeNode]:
            nonlocal pre_idx

            # inorder 区间为空
            if left > right:
                return None

            # preorder 的下一个节点就是当前 root
            root_val = preorder[pre_idx]
            pre_idx += 1

            root = TreeNode(root_val)

            # 根据 inorder 找到左右子树分界点
            mid = inorder_index[root_val]

            # preorder 顺序为 root -> left -> right，
            # 因此必须先递归构造左子树
            root.left = dfs(left, mid - 1)
            root.right = dfs(mid + 1, right)

            return root

        return dfs(0, len(inorder) - 1)
```

可以把整个算法压缩成三个关键点来记：

```
1. preorder 决定 root
2. inorder 决定 left / right 的边界
3. Hash Map 将寻找 root 位置从 O(n) 降为 O(1)
```

因此最终：

```
Time:  O(n)
Space: O(n)
```

这是这道题最稳定、最容易解释，也最适合代码面试的解法。