# Flatten Binary Tree to Linked List（二叉树展开为链表）

给定二叉树的根节点 `root`，请将这棵二叉树**原地展开（in-place）**为一个单链表。

展开之后：

- 仍然使用原来的 `TreeNode` 节点。
- 每个节点的 `left` 都必须为 `None`。
- 每个节点的 `right` 指向链表中的下一个节点。
- 链表中的节点顺序必须与原二叉树的**先序遍历（Preorder Traversal）**顺序完全一致。

例如：

```
原二叉树：

        1
       / \
      2   5
     / \   \
    3   4   6
```

先序遍历顺序：

```
1 → 2 → 3 → 4 → 5 → 6
```

展开之后：

```
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

所有节点：

```
node.left = None
```

最终相当于：

```
1 → 2 → 3 → 4 → 5 → 6
```

------

# 核心思路

题目明确要求最终链表按照：

```
Preorder Traversal
```

也就是：

```
Root → Left → Right
```

排列。

因此这道题本质上是在做：

> **先序遍历 + 修改节点指针。**

但这里有一个非常重要的问题：

当我们修改：

```
node.right
```

时，有可能把原来的右子树覆盖掉。

因此无论使用哪种方法，都需要特别注意：

> **在修改树结构之前，不要丢失原来的左右子树。**

这道题有多种解法。比较值得掌握的是：

1. **DFS + `prev` 指针**
2. **迭代 / 原地修改指针**

其中第二种方法可以做到 `O(1)` 额外空间，非常适合作为这道题的进阶解法。

------

# 方法一：DFS + `prev` 指针

## 思路

最直观的想法是：

1. 按照先序遍历得到节点。
2. 将上一个节点的 `right` 指向当前节点。
3. 将上一个节点的 `left` 设置为 `None`。

例如先序遍历：

```
1 → 2 → 3 → 4 → 5 → 6
```

当访问 `2` 时：

```
prev = 1
current = 2
```

于是：

```
prev.left = None
prev.right = current
```

得到：

```
1 → 2
```

然后：

```
prev = 2
current = 3
```

继续连接：

```
1 → 2 → 3
```

最终就可以构造出链表。

------

## 一个隐藏的问题

但是不能简单地写：

```
prev.right = node
dfs(node.left)
dfs(node.right)
```

因为我们正在**修改原来的树结构**。

例如：

```
        1
       / \
      2   5
```

如果某一步修改了：

```
node.right
```

那么原来的右子树 `5` 可能就找不到了。

因此需要：

> **先保存原来的左右子节点，再修改指针。**

------

## 算法步骤

1. 创建 `prev`，表示先序遍历中上一个访问的节点。

2. DFS 当前节点。

3. 在修改指针之前，保存：

   ```
   left = node.left
   right = node.right
   ```

4. 如果存在 `prev`：

   - `prev.left = None`
   - `prev.right = node`

5. 更新：

   ```
   prev = node
   ```

6. 按照先序遍历顺序继续：

   - DFS 原来的左子树。
   - DFS 原来的右子树。

7. 整棵树处理完成后，树本身就已经变成链表。

------

## Python 实现

```
class Solution:
    def flatten(self, root: Optional[TreeNode]) -> None:
        """
        Do not return anything, modify root in-place instead.
        """

        # prev 表示先序遍历中上一个访问的节点
        prev = None

        def dfs(node):
            nonlocal prev

            if not node:
                return

            # 非常重要：
            # 修改指针之前，先保存原来的左右子树
            left = node.left
            right = node.right

            # 将上一个节点连接到当前节点
            if prev:
                prev.left = None
                prev.right = node

            # 当前节点成为新的 prev
            prev = node

            # 先序遍历：Root -> Left -> Right
            # 注意这里必须使用之前保存的 left 和 right
            dfs(left)
            dfs(right)

        dfs(root)
```

------

# Python：`nonlocal`

这里使用了：

```
nonlocal prev
```

这是 Python 中一个很值得掌握的关键字。

我们有：

```
prev = None

def dfs(node):
    nonlocal prev
```

`prev` 定义在外层函数：

```
flatten()
```

中，而我们希望在内部函数：

```
dfs()
```

中修改它。

如果直接写：

```
prev = node
```

Python 会默认认为 `prev` 是 `dfs()` 内部的**局部变量**。

因此：

```
nonlocal prev
```

是在告诉 Python：

> `prev` 不是当前 `dfs()` 的局部变量，请使用外层 `flatten()` 函数中的那个 `prev`。

这是递归 DFS 中维护共享状态时很常见的写法。

------

# 时间复杂度

每个节点只访问一次，因此：

```
O(n)
```

------

# 空间复杂度

递归调用栈最大深度等于树高 `h`：

```
O(h)
```

最坏情况下，树退化成链表：

```
O(n)
```

对于平衡二叉树：

```
O(log n)
```

------

# 方法二：原地修改指针 —— O(1) 额外空间

这个方法更加巧妙，也是这道题非常值得理解的一种解法。

## 核心观察

假设当前节点是：

```
        1
       / \
      2   5
     / \
    3   4
```

先序遍历要求：

```
1 → 左子树 → 右子树
```

也就是：

```
1 → 2 → 3 → 4 → 5
```

所以从 `1` 出发，我们希望：

```
1.right = 2
```

也就是把整个左子树移动到右边。

但是原来的右子树：

```
5
```

怎么办？

它应该出现在：

```
左子树所有节点之后
```

因此我们需要找到：

> **左子树中先序遍历的最后一个节点。**

------

## 如何找到这个节点？

对于：

```
      2
     / \
    3   4
```

先序遍历：

```
2 → 3 → 4
```

最后一个节点是 `4`。

从结构上看，它就是：

> 当前节点左子树中的**最右节点**。

因此：

```
        1
       / \
      2   5
     / \
    3   4
```

我们找到：

```
predecessor = 4
```

然后执行三个操作：

```
predecessor.right = current.right
current.right = current.left
current.left = None
```

------

# 指针变化过程

原来：

```
        1
       / \
      2   5
     / \
    3   4
```

### 第一步：保存原来的右子树

让左子树最右节点 `4` 指向原来的右子树 `5`：

```
        1
       /
      2
     / \
    3   4
         \
          5
```

对应：

```
predecessor.right = current.right
```

------

### 第二步：把左子树移动到右边

```
current.right = current.left
```

得到：

```
1
 \
  2
 / \
3   4
     \
      5
```

------

### 第三步：清空左指针

```
current.left = None
```

然后继续：

```
current = current.right
```

也就是继续处理节点 `2`。

最终整棵树会变成：

```
1
 \
  2
   \
    3
     \
      4
       \
        5
```

------

# Python 实现

```
class Solution:
    def flatten(self, root: Optional[TreeNode]) -> None:
        """
        Do not return anything, modify root in-place instead.
        """

        current = root

        while current:
            # 如果存在左子树，需要把左子树插入到
            # 当前节点和原右子树之间
            if current.left:

                # 找到左子树中的最右节点
                predecessor = current.left

                while predecessor.right:
                    predecessor = predecessor.right

                # 1. 左子树的最右节点连接原来的右子树
                predecessor.right = current.right

                # 2. 将整个左子树移动到右边
                current.right = current.left

                # 3. 左指针必须清空
                current.left = None

            # 继续处理链表中的下一个节点
            current = current.right
```

------

# 为什么找的是「左子树最右节点」？

这是这个解法最核心的地方。

对于当前节点：

```
        root
       /    \
    left    right
```

先序遍历要求：

```
root → left subtree → right subtree
```

所以我们需要把：

```
right subtree
```

接到：

```
left subtree
```

的**最后面**。

而按照这种不断调整指针的过程，左子树中沿着 `right` 一直走到底的节点，就是需要连接原右子树的位置。

因此：

```
predecessor = current.left

while predecessor.right:
    predecessor = predecessor.right
```

找到连接位置，然后：

```
predecessor.right = current.right
```

把原来的右子树接上去。

可以把整个操作记成：

```
找到左子树最右节点
        ↓
连接原右子树
        ↓
左子树整体搬到右边
        ↓
清空 left
```

------

# 为什么这个方法是 O(n)？

第一眼看到：

```
while current:
```

里面还有：

```
while predecessor.right:
```

可能会误以为时间复杂度是：

```
O(n²)
```

但实际上整体仍然是：

```
O(n)
```

关键原因是：

> 内层循环沿着 `right` 指针寻找 predecessor，而这些边不会被无限重复遍历。

从整棵树来看，每条相关的边只会被有限次数访问，因此所有内层 `while` 累计仍然是 `O(n)`。

所以：

```
时间复杂度：O(n)
```

------

# 空间复杂度

这个方法没有：

- 递归调用栈；
- BFS 队列；
- 额外数组。

只使用：

```
current
predecessor
```

几个指针，因此：

```
O(1)
```

这也是这个方法相比普通 DFS 最大的优势。

------

# 方法三：栈模拟先序遍历

还有一种非常直观的方法：

> 用 Stack 显式模拟先序遍历，然后把访问到的节点连接起来。

因为先序遍历是：

```
Root → Left → Right
```

而栈是：

```
Last In First Out
```

所以我们需要：

> **先把右子节点压栈，再把左子节点压栈。**

这样左子节点就会先出栈。

------

## Python 实现

```
class Solution:
    def flatten(self, root: Optional[TreeNode]) -> None:
        if not root:
            return

        stack = [root]
        prev = None

        while stack:
            node = stack.pop()

            # 将上一个节点连接到当前节点
            if prev:
                prev.left = None
                prev.right = node

            # 先放 right，再放 left
            # 因为 Stack 是 LIFO，所以 left 会先被处理
            if node.right:
                stack.append(node.right)

            if node.left:
                stack.append(node.left)

            prev = node
```

------

# 为什么 Stack 中是 Right First？

这是非常常见的面试知识点。

我们希望实际访问顺序是：

```
Root → Left → Right
```

但是 Stack 是：

```
LIFO
Last In, First Out
后进先出
```

所以：

```
stack.append(node.right)
stack.append(node.left)
```

之后：

```
stack:

bottom [right, left] top
```

执行：

```
stack.pop()
```

首先拿到：

```
left
```

因此最终访问顺序才会是：

```
Root → Left → Right
```

记住：

```
想先访问 Left
↓
Stack 中反而先 push Right
```

这个规律在很多 DFS 的迭代写法中都会出现。

------

# 三种方法对比

| 方法         | 核心思想                       | 时间复杂度 | 辅助空间 |
| ------------ | ------------------------------ | ---------- | -------- |
| DFS + `prev` | 先序遍历过程中连接前后节点     | `O(n)`     | `O(h)`   |
| 原地指针修改 | 左子树插入当前节点与右子树之间 | `O(n)`     | `O(1)`   |
| Stack        | 显式模拟先序遍历               | `O(n)`     | `O(n)`   |

如果是面试，我建议重点掌握：

```
DFS + prev
        ↓
理解最自然

O(1) 原地修改
        ↓
空间最优，体现对树结构的理解
```

------

# 常见错误

## 1. 修改 `right` 后丢失原来的右子树

这是 DFS 写法最容易出现的问题。

例如：

```
# Wrong
node.right = node.left
node.left = None

dfs(node.right)
```

一旦执行：

```
node.right = node.left
```

原来的：

```
node.right
```

就被覆盖了。

原右子树可能因此丢失。

所以如果使用 DFS 修改节点结构，要么提前保存：

```
left = node.left
right = node.right
```

要么采用其他不会破坏后续遍历的方式。

------

## 2. 忘记把 `left` 设置为 `None`

题目明确要求：

```
所有 left 指针必须为 null
```

所以仅仅：

```
prev.right = node
```

是不够的。

还需要：

```
prev.left = None
```

或者在原地方法中：

```
current.left = None
```

------

## 3. Stack 中先 Push Left

如果写成：

```
stack.append(node.left)
stack.append(node.right)
```

因为 Stack 是 LIFO：

```
Right 会先 pop
```

实际遍历就会变成：

```
Root → Right → Left
```

而不是题目要求的：

```
Root → Left → Right
```

正确顺序：

```
if node.right:
    stack.append(node.right)

if node.left:
    stack.append(node.left)
```

即：

```
先 Push Right
再 Push Left

→ Left 先 Pop
```

------

## 4. 误以为只需要把左子树直接放到右边

下面是不完整的：

```
current.right = current.left
current.left = None
```

因为当前节点可能原本就存在右子树。

例如：

```
        1
       / \
      2   5
```

直接：

```
1.right = 2
```

就会导致节点 `5` 丢失。

所以必须先：

```
找到左子树最后的位置
↓
把原右子树接过去
```

即：

```
predecessor.right = current.right
```

------

# 面试记忆模板

## DFS

核心：

```
Preorder
+
prev
+
修改之前保存左右子树
```

模板：

```
prev = None

def dfs(node):
    nonlocal prev

    if not node:
        return

    left = node.left
    right = node.right

    if prev:
        prev.left = None
        prev.right = node

    prev = node

    dfs(left)
    dfs(right)
```

记忆：

```
保存 Left / Right
      ↓
连接 prev → current
      ↓
Preorder DFS
```

------

## O(1) 原地解法

核心模板：

```
current = root

while current:
    if current.left:
        predecessor = current.left

        while predecessor.right:
            predecessor = predecessor.right

        predecessor.right = current.right
        current.right = current.left
        current.left = None

    current = current.right
```

可以浓缩成四句话：

```
有左子树：

1. 找左子树最右节点
2. 最右节点 → 原右子树
3. 当前 right → 当前 left
4. 当前 left → None
```

或者进一步记成：

```
左子树搬到右边，
原右子树接到左子树最后。
```

------

# 总结

这道题最重要的两个关键词是：

```
Preorder Traversal
+
In-place Pointer Modification
```

题目要求最终顺序：

```
Root → Left → Right
```

因此它本质上就是**把二叉树的先序遍历顺序转化为 `right` 指针形成的链表**。

如果采用 DFS：

```
先保存原树结构
↓
按照 Preorder 遍历
↓
使用 prev 连接相邻节点
```

如果采用 `O(1)` 原地解法：

```
当前节点存在左子树
↓
找到左子树最右节点
↓
它连接原来的右子树
↓
左子树移动到 right
↓
left = None
↓
继续处理 current.right
```

面试中尤其值得记住这个结构转换：

```
Before:

       current
       /     \
    left     right


After:

    current
       \
       left
         \
          ...
            \
         predecessor
              \
              right
```

它保证节点最终严格按照：

```
Root → Left Subtree → Right Subtree
```

排列，也就是题目要求的**先序遍历顺序**。