# 二叉搜索树中第 K 小的整数（Kth Smallest Integer in BST）

给定一棵二叉搜索树（BST）的根节点 `root` 和一个整数 `k`，返回树中**第 `k` 小的值**。

这里的 `k` 从 **1 开始计数（1-indexed）**。也就是说：

- `k = 1`：返回最小值
- `k = 2`：返回第二小的值
- 以此类推

## 二叉搜索树的性质

二叉搜索树满足：

- 每个节点的**左子树**中的所有节点值都 **小于** 当前节点值。
- 每个节点的**右子树**中的所有节点值都 **大于** 当前节点值。
- 左右子树本身也都是二叉搜索树。

因此，对于任意节点，都有：

```
左子树 < 当前节点 < 右子树
```

这道题最重要的知识点是：

> **BST 的中序遍历（Left → Node → Right）会按照从小到大的顺序访问节点。**

因此，这道题本质上可以转化为：

> **对 BST 进行中序遍历，找到第 `k` 个被访问的节点。**

------

# 1. 暴力解法：遍历 + 排序

## 思路

最直接的方法甚至不需要利用 BST 的性质。

我们可以：

1. 遍历整棵树。
2. 把所有节点的值保存到数组 `arr`。
3. 对 `arr` 排序。
4. 返回 `arr[k - 1]`。

因为 Python 数组下标从 `0` 开始，而题目的 `k` 从 `1` 开始，所以第 `k` 小对应：

```
arr[k - 1]
```

## 代码

```
# 二叉树节点定义
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        arr = []

        def dfs(node):
            # 递归终止条件
            if not node:
                return

            # 当前节点加入数组
            arr.append(node.val)

            # 遍历左右子树
            dfs(node.left)
            dfs(node.right)

        # 收集所有节点
        dfs(root)

        # 从小到大排序
        arr.sort()

        # k 从 1 开始，数组下标从 0 开始
        return arr[k - 1]
```

## 复杂度分析

设树中共有 `n` 个节点。

- **时间复杂度：`O(n log n)`**
  - DFS 遍历：`O(n)`
  - 排序：`O(n log n)`
  - 因此整体为 `O(n log n)`
- **空间复杂度：`O(n)`**
  - `arr` 需要保存全部 `n` 个节点。
  - 递归栈最坏情况下也可能达到 `O(n)`。

## 总结

这个方法非常直观，但没有利用：

```
BST + 中序遍历 = 有序序列
```

因此不是面试中最推荐的方案。

------

# 2. 中序遍历 + 数组

## 思路

BST 最关键的性质之一：

> **中序遍历（Left → Node → Right）得到的节点值天然就是升序的。**

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

因此，我们不需要排序。

只需要：

1. 对 BST 进行中序遍历。
2. 把访问到的节点依次加入数组。
3. 返回 `arr[k - 1]`。

## 代码

```
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        arr = []

        def dfs(node):
            if not node:
                return

            # 1. 左子树
            dfs(node.left)

            # 2. 当前节点
            arr.append(node.val)

            # 3. 右子树
            dfs(node.right)

        dfs(root)

        return arr[k - 1]
```

## 为什么中序遍历是有序的？

BST 满足：

```
左子树所有值 < 当前节点 < 右子树所有值
```

而中序遍历恰好按照：

```
左子树 → 当前节点 → 右子树
```

访问。

递归地应用这个性质，最终得到：

```
最小 → ... → 当前节点 → ... → 最大
```

也就是完整的升序序列。

## 复杂度分析

- **时间复杂度：`O(n)`**
- **空间复杂度：`O(n)`**

这里虽然去掉了排序，将时间复杂度从 `O(n log n)` 优化到了 `O(n)`，但仍然需要保存全部节点。

实际上我们并不需要完整的排序数组。

例如 `k = 3` 时：

```
2 → 3 → 4
          ↑
       找到了
```

访问到第三个节点之后，就已经可以结束了。

这就引出了下一种更好的方法。

------

# 3. 递归中序 DFS + 提前终止

## 思路

仍然利用：

```
中序遍历 = 从小到大访问 BST
```

但是这一次不保存整个数组。

我们可以维护一个计数器：

```
cnt = k
```

每访问一个节点：

```
cnt -= 1
```

当：

```
cnt == 0
```

说明当前节点就是第 `k` 小的节点。

例如：

```
中序遍历：

2    3    4    5    6
↓    ↓    ↓
k=3  k=2  k=1
          ↓
        答案
```

找到以后就可以提前结束遍历。

## 代码

```
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        # cnt 表示距离第 k 小还剩多少个节点
        cnt = k

        # 保存最终答案
        res = root.val

        def dfs(node):
            nonlocal cnt, res

            if not node:
                return

            # 1. 先遍历左子树
            dfs(node.left)

            # 如果答案已经找到，不再继续
            if cnt == 0:
                return

            # 2. 访问当前节点
            cnt -= 1

            # 当前节点就是第 k 小
            if cnt == 0:
                res = node.val
                return

            # 3. 最后遍历右子树
            dfs(node.right)

        dfs(root)

        return res
```

## `nonlocal` 是什么？

这里使用了：

```
nonlocal cnt, res
```

这是一个值得注意的 Python 语法。

`cnt` 和 `res` 定义在外层函数：

```
def kthSmallest(...):
    cnt = k
    res = root.val

    def dfs(node):
        ...
```

但是我们希望在内部的 `dfs()` 中修改它们：

```
cnt -= 1
res = node.val
```

如果没有 `nonlocal`，Python 会把赋值后的 `cnt` 和 `res` 当作 `dfs()` 自己的局部变量。

使用：

```
nonlocal cnt, res
```

就是告诉 Python：

> `cnt` 和 `res` 不是 `dfs()` 的局部变量，请使用外层函数中的变量。

它与 `global` 不同：

```
global    → 使用全局作用域中的变量
nonlocal  → 使用最近一层外部函数作用域中的变量
```

在递归 DFS 题目中，`nonlocal` 经常用于维护：

- 计数器
- 最大值 / 最小值
- 最终答案
- 某种全局状态

## 复杂度分析

设树高为 `h`。

- **时间复杂度：`O(h + k)`**
  - 首先最多需要沿树向下走 `h` 层。
  - 然后按照中序顺序访问节点直到第 `k` 个。
  - 最坏情况下仍然是 `O(n)`。
- **空间复杂度：`O(h)`**
  - 主要来自递归调用栈。
  - 平衡 BST：`O(log n)`
  - 极度倾斜的 BST：`O(n)`

## 面试理解

相比上一种方案：

```
中序遍历 + 数组
        ↓
保存全部节点 O(n)
```

这里变成：

```
中序遍历 + Counter
        ↓
访问到第 k 个就停止
```

这是非常自然的一步优化。

------

# 4. 迭代中序 DFS（推荐掌握）

## 思路

递归中序遍历：

```
dfs(node.left)
visit(node)
dfs(node.right)
```

实际上可以使用一个**栈 `stack`** 手动模拟。

基本流程：

```
不断向左走
   ↓
把节点压入 stack
   ↓
走到最左边
   ↓
pop 一个节点
   ↓
这个节点就是当前最小的未访问节点
   ↓
进入它的右子树
   ↓
重复
```

例如：

```
        5
       /
      3
     /
    2
```

首先不断向左：

```
stack = [5, 3, 2]
```

然后：

```
stack.pop()
```

得到 `2`。

由于 `2` 是整棵 BST 最左边的节点，因此它就是最小值。

之后继续按照这个过程访问：

```
2 → 3 → ... → 5 → ...
```

这实际上就是中序遍历。

## 代码

```
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        stack = []
        curr = root

        # 只要还有节点需要处理，就继续
        while stack or curr:

            # 一直向左走
            # 将路径上的节点全部压入栈
            while curr:
                stack.append(curr)
                curr = curr.left

            # 当前栈顶就是下一个最小节点
            curr = stack.pop()

            # 访问一个节点
            k -= 1

            # 第 k 个访问的节点就是答案
            if k == 0:
                return curr.val

            # 当前节点处理完后，进入右子树
            curr = curr.right
```

## 为什么要写 `while stack or curr`？

这里有两个状态：

```
curr
```

表示当前正在处理的节点；

```
stack
```

表示之前暂存、还没有正式访问的节点。

因此即使：

```
curr is None
```

也不代表遍历结束。

例如：

```
stack = [5, 3, 2]
curr = None
```

虽然 `curr` 已经为空，但是 `stack` 中还有节点等待处理。

所以条件必须是：

```
while stack or curr:
```

即：

> 当前还有节点，或者栈中还有待处理节点，就继续。

## `stack.append()` 和 `stack.pop()`

Python 的 `list` 可以直接作为栈使用。

压栈：

```
stack.append(node)
```

弹出栈顶：

```
node = stack.pop()
```

例如：

```
stack = []

stack.append(5)
stack.append(3)
stack.append(2)

print(stack)
# [5, 3, 2]

x = stack.pop()
# x = 2

print(stack)
# [5, 3]
```

因此：

```
append() → push
pop()    → pop
```

这也是 Python 算法题中实现栈最常见的方式。

## 复杂度分析

更精确地说：

- **时间复杂度：`O(h + k)`**
- **空间复杂度：`O(h)`**

其中 `h` 是树高。

最坏情况下：

```
时间 O(n)
空间 O(n)
```

如果 BST 比较平衡：

```
h = O(log n)
```

那么额外空间为：

```
O(log n)
```

> 原答案将这一方案简单写成 `O(n)` 时间、`O(n)` 空间是最坏情况分析；在面试中写成 `O(h + k)` 时间和 `O(h)` 空间会更加准确。

------

# 5. Morris Traversal：O(1) 额外空间

## 思路

前面的递归和迭代中序遍历都需要额外空间：

```
递归 DFS → recursion stack
迭代 DFS → stack
```

那么能不能：

```
不使用递归
不使用 stack
```

同时完成中序遍历？

可以，这就是 **Morris Traversal（Morris 遍历）**。

它可以做到：

```
时间：O(n)
额外空间：O(1)
```

核心思想是：

> 临时修改树中的指针，让我们遍历完左子树之后能够重新回到当前节点。

------

## 什么是中序前驱？

假设当前节点是：

```
        5
       /
      3
       \
        4
```

对于节点 `5` 来说：

```
左子树 = 3, 4
```

其中最大的节点是：

```
4
```

所以 `4` 是 `5` 的**中序前驱（inorder predecessor）**。

也就是：

> 当前节点左子树中最右边的节点。

寻找方式：

```
pred = curr.left

while pred.right:
    pred = pred.right
```

------

## Morris 的关键操作

正常情况下，我们遍历：

```
        5
       /
      3
       \
        4
```

到了 `4` 之后，如果没有递归栈，我们不知道如何回到 `5`。

所以 Morris 临时建立：

```
4.right = 5
```

变成：

```
        5
       /
      3
       \
        4
         \
          5
```

当然，这只是一个**临时链接（thread）**。

这样访问完 `4` 后，就可以通过：

```
4.right
```

重新回到 `5`。

回来之后，再恢复：

```
4.right = None
```

因此最终树的结构不会发生变化。

------

## 两种情况

### 情况 1：当前节点没有左子树

例如：

```
curr
 ↓
 2
```

那么它就是当前应该访问的节点：

```
k -= 1
```

然后：

```
curr = curr.right
```

------

### 情况 2：当前节点存在左子树

寻找：

```
左子树中最右边的节点
```

即：

```
pred
```

然后存在两种可能。

第一次遇到：

```
pred.right is None
```

建立临时链接：

```
pred.right = curr
```

然后进入左子树：

```
curr = curr.left
```

第二次遇到：

```
pred.right == curr
```

说明：

> 左子树已经遍历完成，现在通过临时链接回到了 `curr`。

于是恢复树：

```
pred.right = None
```

然后正式访问 `curr`。

------

## 代码

```
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        curr = root

        while curr:

            # Case 1:
            # 当前节点没有左子树，可以直接访问
            if not curr.left:
                k -= 1

                if k == 0:
                    return curr.val

                curr = curr.right

            # Case 2:
            # 当前节点存在左子树
            else:
                # 找当前节点的中序前驱：
                # 左子树中最右边的节点
                pred = curr.left

                while pred.right and pred.right != curr:
                    pred = pred.right

                # 第一次来到这里：
                # 建立临时 thread，让前驱指向当前节点
                if not pred.right:
                    pred.right = curr

                    # 继续进入左子树
                    curr = curr.left

                # 第二次来到这里：
                # pred.right == curr
                # 说明左子树已经遍历完毕
                else:
                    # 删除临时链接，恢复原树
                    pred.right = None

                    # 正式访问当前节点
                    k -= 1

                    if k == 0:
                        return curr.val

                    # 进入右子树
                    curr = curr.right

        return -1
```

## 复杂度分析

- **时间复杂度：`O(n)`**
- **额外空间复杂度：`O(1)`**

虽然寻找 `pred` 时存在一个内部 `while`：

```
while pred.right and pred.right != curr:
```

看起来似乎可能产生 `O(n²)`，但实际上不会。

Morris Traversal 中，每条相关的边只会被有限次数访问，因此总时间仍然是：

```
O(n)
```

------

# 一个重要的 Morris Traversal 注意点

上面的代码来自原答案，但这里存在一个值得面试时注意的细节：

```
if k == 0:
    return curr.val
```

Morris Traversal 会**临时修改树结构**：

```
pred.right = curr
```

如果找到答案后直接 `return`，有可能此时树中还有之前创建、尚未恢复的临时链接。

因此，如果题目要求：

> **函数执行之后必须保证输入树完全恢复原状**

那么不能简单地找到答案后立即返回，而应该记录答案，并继续 Morris 遍历直到所有临时链接都被清除。

例如可以写成：

```
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        curr = root
        result = None

        while curr:
            if not curr.left:
                # 正常访问节点
                k -= 1

                if k == 0:
                    result = curr.val

                curr = curr.right

            else:
                pred = curr.left

                # 找左子树最右节点
                while pred.right and pred.right != curr:
                    pred = pred.right

                if not pred.right:
                    # 创建临时链接
                    pred.right = curr
                    curr = curr.left

                else:
                    # 删除临时链接，恢复树
                    pred.right = None

                    # 正常访问节点
                    k -= 1

                    if k == 0:
                        result = curr.val

                    curr = curr.right

        return result
```

这样可以确保 Morris Traversal 结束之后：

```
所有临时 thread 都已经被删除
```

代价是找到第 `k` 小之后仍然需要继续遍历，因此不能利用 early stopping。

------

# 常见错误

## 1. 使用前序或后序遍历

BST 中只有：

```
中序遍历：
Left → Node → Right
```

能够直接得到升序序列。

前序：

```
Node → Left → Right
```

后序：

```
Left → Right → Node
```

都不能保证节点按照大小顺序出现。

因此这道题看到：

```
BST + 第 k 小
```

应该迅速想到：

```
Inorder Traversal
```

------

## 2. `k` 的 Off-by-One 错误

题目的 `k` 是 **1-indexed**。

如果使用数组：

```
return arr[k - 1]
```

如果使用计数器：

```
k -= 1

if k == 0:
    return node.val
```

例如：

```
k = 3

第 1 个节点 → k = 2
第 2 个节点 → k = 1
第 3 个节点 → k = 0 → 找到答案
```

------

## 3. 忘记提前终止

如果使用普通递归/迭代中序遍历，一旦找到第 `k` 个节点，就不需要继续遍历。

例如：

```
BST 中有 100000 个节点
k = 3
```

我们实际上只关心：

```
第 1 小
第 2 小
第 3 小
```

因此 early stopping 可以避免大量无意义的遍历。

不过需要注意，**Morris Traversal 是特殊情况**：如果直接提前返回，可能导致临时修改的指针没有被恢复。

------

# 几种方案对比

| 方法             | 时间复杂度              | 额外空间 | 利用 BST 性质 | 可提前结束             |
| ---------------- | ----------------------- | -------- | ------------- | ---------------------- |
| DFS + 排序       | `O(n log n)`            | `O(n)`   | ❌             | ❌                      |
| 中序遍历 + 数组  | `O(n)`                  | `O(n)`   | ✅             | 通常没有               |
| 递归中序 DFS     | `O(h + k)`，最坏 `O(n)` | `O(h)`   | ✅             | ✅                      |
| 迭代中序 DFS     | `O(h + k)`，最坏 `O(n)` | `O(h)`   | ✅             | ✅                      |
| Morris Traversal | `O(n)`                  | `O(1)`   | ✅             | ⚠️ 提前返回需注意恢复树 |

其中 `h` 表示树的高度。

------

# 面试中的推荐解法

这道题最值得掌握的是**迭代中序遍历**：

```
class Solution:
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        stack = []
        curr = root

        while stack or curr:

            # 不断进入左子树
            while curr:
                stack.append(curr)
                curr = curr.left

            # 当前最小的未访问节点
            curr = stack.pop()

            # 访问当前节点
            k -= 1

            if k == 0:
                return curr.val

            # 接下来处理右子树
            curr = curr.right
```

可以把整个算法记成三个动作：

```
1. Go Left
      ↓
2. Pop / Visit
      ↓
3. Go Right
```

也就是：

```
Left → Node → Right
```

这正是中序遍历。

对于 **BST + 第 K 小** 这一类问题，可以建立一个非常直接的思维链：

```
BST
 ↓
中序遍历有序
 ↓
第 k 小 = 中序遍历第 k 个节点
 ↓
不需要完整遍历
 ↓
遍历到第 k 个节点立即返回
```

如果面试官进一步问：

> **“能不能做到 O(1) 额外空间？”**

这时候再引出 **Morris Traversal** 会比较合适。