# 路径总和 III（Path Sum III）

给定一棵二叉树的根节点 `root` 和一个整数 `targetSum`。

请返回二叉树中**节点值之和等于 `targetSum` 的路径数量**。

需要注意：

- 路径**不要求从根节点开始**
- 路径**不要求在叶子节点结束**
- 路径必须是**连续的**
- 路径方向必须始终**从父节点向子节点**
- 同一条路径不能“拐回去”访问父节点

例如：

```
        10
       /  \
      5   -3
     / \    \
    3   2    11
   / \   \
  3  -2   1
```

如果：

```
targetSum = 8
```

合法路径包括：

```
5 -> 3
5 -> 2 -> 1
-3 -> 11
```

因此答案为：

```
3
```

------

# 一、核心难点

这道题和普通的 `Path Sum` 最大的区别是：

> **路径不一定从根节点开始。**

如果题目要求路径必须从 `root` 开始，那么只需要维护一个当前路径和：

```
current_sum
```

不断向左右子树 DFS 即可。

但现在路径可以从任意节点开始，例如：

```
        10
       /
      5
     /
    3
```

如果：

```
targetSum = 8
```

合法路径是：

```
5 -> 3
```

而不是：

```
10 -> 5 -> 3
```

所以我们需要想办法判断：

> 当前节点结束的所有向下路径中，有多少条路径的节点和恰好等于 `targetSum`？

这也是这道题的核心。

------

# 方法一：双重 DFS / 枚举每个起点

## 思路

最直接的想法是：

> 既然路径可以从任意节点开始，那就把**每一个节点都当作路径起点**尝试一次。

我们可以设计两个 DFS：

```
dfs(node)
    枚举每一个可能的起点

countPath(node, remaining)
    假设路径必须从 node 开始，
    计算向下有多少条路径满足目标
```

例如：

```
        10
       /  \
      5   -3
```

我们依次尝试：

```
从 10 开始的路径
从 5 开始的路径
从 -3 开始的路径
...
```

对于每个起点，再向下搜索。

------

## 一个容易犯错的地方

假设当前：

```
remaining == node.val
```

说明：

```
从路径起点到当前 node
```

已经构成了一条合法路径。

但是**不能立刻停止搜索**。

因为节点值可能包含：

```
0
负数
正数
```

例如：

```
targetSum = 8

8
 \
  0
```

这里实际上存在两条合法路径：

```
8
8 -> 0
```

所以即使当前已经满足 `targetSum`，仍然需要继续搜索下面的节点。

------

## 算法步骤

对于每个节点 `node`：

1. 把 `node` 当作路径起点。
2. 从 `node` 开始向下 DFS。
3. 每经过一个节点，就从剩余目标 `remaining` 中减去当前节点值。
4. 如果当前节点值恰好等于 `remaining`，说明找到一条合法路径。
5. 继续搜索左右子树。
6. 然后把 `node.left` 和 `node.right` 分别作为新的路径起点。

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
    def pathSum(
        self,
        root: Optional[TreeNode],
        targetSum: int
    ) -> int:

        # 计算：
        # 如果路径必须从 node 开始，
        # 有多少条向下路径的和等于 remaining
        def countPath(node, remaining):
            if not node:
                return 0

            count = 0

            # 当前节点正好让路径和达到目标
            if node.val == remaining:
                count += 1

            # 注意：
            # 即使这里已经找到了一条合法路径，
            # 也不能停止，因为后面可能存在 0 或负数，
            # 继续向下仍然可能产生新的合法路径。
            count += countPath(
                node.left,
                remaining - node.val
            )

            count += countPath(
                node.right,
                remaining - node.val
            )

            return count

        if not root:
            return 0

        # 1. 以当前 root 为路径起点
        result = countPath(root, targetSum)

        # 2. 路径也可能从左子树中的任意节点开始
        result += self.pathSum(root.left, targetSum)

        # 3. 路径也可能从右子树中的任意节点开始
        result += self.pathSum(root.right, targetSum)

        return result
```

------

## 如何理解两个 DFS？

这段代码最值得理解的是两个递归函数的职责完全不同。

### `pathSum(node)`

回答：

> 在以 `node` 为根的整棵子树中，一共有多少条合法路径？

因此它会枚举：

```
node
node.left
node.right
...
```

作为路径起点。

------

### `countPath(node, remaining)`

回答：

> 如果路径**必须从 node 开始**，向下走能找到多少条合法路径？

因此这里不能跳过某个节点。

只能：

```
node
 ↓
left / right
 ↓
...
```

------

## 复杂度

设树中共有 `n` 个节点。

最坏情况下，例如树退化成链表：

```
1
 \
  2
   \
    3
     \
      4
```

第一个节点向下搜索 `n` 个节点；

第二个节点搜索 `n - 1` 个；

第三个节点搜索 `n - 2` 个；

因此：

```
n + (n - 1) + (n - 2) + ... + 1
= O(n²)
```

所以：

```
时间复杂度：O(n²)
```

对于比较平衡的树，实际表现会更好，但最坏情况仍然是 `O(n²)`。

递归栈空间：

```
O(h)
```

其中 `h` 是树的高度：

```
平衡树：O(log n)
最坏情况：O(n)
```

------

# 方法二：前缀和 + Hash Map【推荐】

这是这道题最重要、也最推荐在面试中使用的方法。

时间复杂度可以从：

```
O(n²)
```

优化到：

```
O(n)
```

------

# 前缀和的核心思想

先暂时忘掉二叉树，考虑一个数组：

```
[10, 5, 3]
```

从起点开始计算前缀和：

```
节点：        10    5    3
前缀和：      10   15   18
```

假设我们想知道：

```
5 -> 3
```

这段路径的和。

可以计算：

```
18 - 10 = 8
```

也就是说：

> **一段连续区间的和 = 当前前缀和 - 之前某个前缀和**

数组中的经典公式是：

```
sum(i...j)
=
prefix[j] - prefix[i - 1]
```

二叉树中其实完全一样。

只不过数组中的“连续区间”变成了：

> 从某个祖先节点向下到当前节点的一段连续路径。

------

# 从公式推导到这道题

假设从根节点走到当前节点的路径为：

```
root -> ... -> A -> ... -> current
```

当前从根到 `current` 的路径和为：

```
currentSum
```

假设在之前某个位置，前缀和为：

```
prefixSum
```

那么它们之间这段路径的和就是：

```
currentSum - prefixSum
```

我们希望这段路径满足：

```
currentSum - prefixSum = targetSum
```

移项：

```
prefixSum = currentSum - targetSum
```

所以问题就变成：

> 在到达当前节点之前，我们曾经遇到过多少次前缀和 `currentSum - targetSum`？

如果出现了 `k` 次，那么就说明：

```
有 k 条路径
```

以当前节点作为终点，并且路径和等于 `targetSum`。

------

# 一个具体例子

仍然考虑：

```
        10
       /
      5
     /
    3
```

并且：

```
targetSum = 8
```

走到节点 `3` 时：

```
currentSum = 10 + 5 + 3 = 18
```

我们寻找：

```
currentSum - targetSum
= 18 - 8
= 10
```

之前确实出现过前缀和：

```
10
```

它对应：

```
root -> 10
```

那么去掉这一部分：

```
18 - 10 = 8
```

剩下的路径就是：

```
5 -> 3
```

所以找到一条合法路径。

------

# 为什么需要 Hash Map？

我们需要不断询问：

```
之前出现过多少次前缀和 X？
```

因此可以使用字典：

```
prefix_count = {}
```

记录：

```
prefix_sum -> 出现次数
```

例如：

```
prefix_count = {
    0: 1,
    10: 1,
    15: 1
}
```

查询：

```
prefix_count.get(currentSum - targetSum, 0)
```

平均只需要：

```
O(1)
```

时间。

------

# 为什么保存的是“出现次数”，而不是 True / False？

因为同一个前缀和可能在当前路径中出现多次。

例如某些节点值包含：

```
0
正数
负数
```

可能出现：

```
prefixSum = 10
...
prefixSum = 10
```

如果当前：

```
currentSum - targetSum = 10
```

那么这两个不同的位置分别对应一条不同路径。

因此 Hash Map 必须记录：

```
前缀和 -> 出现次数
```

而不能只记录：

```
前缀和是否出现
```

------

# 最关键的初始化：`{0: 1}`

我们初始化：

```
prefix_count = {0: 1}
```

这一行非常重要。

为什么？

假设：

```
        5
       /
      3
```

并且：

```
targetSum = 8
```

走到节点 `3`：

```
currentSum = 8
```

我们查找：

```
currentSum - targetSum
= 8 - 8
= 0
```

如果：

```
prefix_count[0] = 1
```

那么说明找到一条合法路径：

```
5 -> 3
```

也就是：

> 从真正的根节点开始的路径，也应该被统计。

所以可以把：

```
prefix_count = {0: 1}
```

理解为：

> 在访问任何节点之前，我们已经存在一个“空路径”，它的前缀和为 0。

这是前缀和问题中非常常见的初始化技巧。

------

# 为什么必须 Backtracking？

这是整个最优解中最容易出错的地方。

考虑：

```
        10
       /  \
      5   -3
```

DFS 访问左子树时，会把：

```
10
15
...
```

这些前缀和放进 Hash Map。

但是当我们离开左子树、进入右子树时：

> 左子树中的前缀和不能继续存在。

因为：

```
左子树节点 -> 右子树节点
```

并不是合法的向下路径。

例如：

```
      10
     /  \
    5   -3
```

我们不能形成：

```
5 -> 10 -> -3
```

因为这条路径先从子节点走回父节点，然后再向下。

题目明确要求：

```
只能从父节点向子节点
```

因此 DFS 离开一个节点时必须：

```
prefix_count[current_sum] -= 1
```

这一步叫：

```
Backtracking
回溯
```

它保证 Hash Map 中始终只保存：

> **当前 DFS 路径上的前缀和。**

------

# 算法步骤

维护：

```
prefix_count
```

其中：

```
prefix_count[x]
```

表示：

> 当前从根节点走到当前节点的 DFS 路径中，前缀和 `x` 出现过多少次。

初始化：

```
prefix_count = {0: 1}
```

然后 DFS：

1. 将当前节点加入路径：

```
current_sum += node.val
```

1. 查找：

```
current_sum - targetSum
```

之前出现过多少次：

```
count += prefix_count.get(
    current_sum - targetSum,
    0
)
```

1. 将当前 `current_sum` 加入 Hash Map：

```
prefix_count[current_sum] += 1
```

1. DFS 左子树。
2. DFS 右子树。
3. 离开当前节点之前进行回溯：

```
prefix_count[current_sum] -= 1
```

------

# Python 实现

```
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def pathSum(
        self,
        root: Optional[TreeNode],
        targetSum: int
    ) -> int:

        # prefix_count[prefix_sum]
        # 表示当前 DFS 路径上，该前缀和出现了多少次
        #
        # 0: 1 表示在还没有访问任何节点之前，
        # 存在一个前缀和为 0 的“空前缀”
        prefix_count = {0: 1}

        def dfs(node, current_sum):
            if not node:
                return 0

            # 当前从 root 到 node 的路径和
            current_sum += node.val

            # 如果之前存在 prefix：
            #
            # current_sum - prefix = targetSum
            #
            # 那么：
            #
            # prefix = current_sum - targetSum
            #
            # 每出现一次这样的 prefix，
            # 就对应一条以当前 node 为终点的合法路径
            count = prefix_count.get(
                current_sum - targetSum,
                0
            )

            # 把当前前缀和加入当前 DFS 路径
            prefix_count[current_sum] = (
                prefix_count.get(current_sum, 0) + 1
            )

            # 继续向左右子树搜索
            count += dfs(node.left, current_sum)
            count += dfs(node.right, current_sum)

            # Backtracking：
            # 离开当前节点时，删除当前节点贡献的前缀和。
            #
            # 防止左子树中的前缀和影响右子树。
            prefix_count[current_sum] -= 1

            return count

        return dfs(root, 0)
```

------

# Python：`dict.get()`

这里经常使用：

```
prefix_count.get(key, 0)
```

Python 字典的：

```
dict.get(key, default)
```

表示：

> 如果 `key` 存在，返回对应值；否则返回 `default`。

例如：

```
count = {
    10: 2,
    20: 1
}

count.get(10, 0)
# 2

count.get(30, 0)
# 0
```

如果直接写：

```
count[30]
```

而 `30` 不存在，就会产生：

```
KeyError
```

所以统计频率时：

```
dictionary.get(key, 0)
```

非常常见。

------

# 前缀和代码到底在维护什么？

这部分值得单独记住。

假设当前 DFS 正在访问：

```
        10
       /
      5
     /
    3
```

当访问 `3` 时，当前路径是：

```
10 -> 5 -> 3
```

那么 Hash Map 中大致保存：

```
0  -> 1
10 -> 1
15 -> 1
18 -> 1
```

它表示的不是：

> 整棵树曾经出现过什么前缀和。

而是：

> **当前从根到 `3` 的这条 DFS 路径上出现过什么前缀和。**

一旦 DFS 从 `3` 返回：

```
prefix_count[18] -= 1
```

从 `5` 返回：

```
prefix_count[15] -= 1
```

这样进入别的分支时，就不会错误地使用左边路径上的节点。

------

# 为什么不能先加入当前前缀和，再检查答案？

正确顺序是：

```
# 1. 先找答案
count += prefix_count.get(
    current_sum - targetSum,
    0
)

# 2. 再加入当前 prefix
prefix_count[current_sum] += 1
```

而不是反过来。

特别是：

```
targetSum = 0
```

时，如果先把当前 `current_sum` 加进去，那么：

```
current_sum - targetSum
= current_sum
```

当前节点自己的前缀和就会匹配自己。

这相当于统计了一条：

```
长度为 0 的空路径
```

但题目要求路径必须至少包含一个节点。

所以：

> **先查询，再加入当前前缀和。**

------

# 方法二复杂度

对于每个节点：

```
Hash Map 查询：平均 O(1)
Hash Map 更新：平均 O(1)
```

每个节点只访问一次，因此：

```
时间复杂度：O(n)
```

空间方面：

Hash Map 中只需要保存当前 DFS 路径上的前缀和，所以严格来说：

```
O(h)
```

递归栈也是：

```
O(h)
```

其中 `h` 为树的高度。

因此：

```
额外空间复杂度：O(h)
```

最坏情况下树退化成链表：

```
O(n)
```

所以面试中通常也会写：

```
Space: O(n)
```

作为最坏复杂度。

------

# 两种方法对比

| 方法                  | 时间复杂度 | 空间复杂度           | 特点             |
| --------------------- | ---------- | -------------------- | ---------------- |
| 枚举起点 + DFS        | `O(n²)`    | `O(h)`               | 直观，容易想到   |
| Prefix Sum + Hash Map | `O(n)`     | `O(h)` / 最坏 `O(n)` | **推荐面试使用** |

这道题非常适合按照下面的优化路线讲：

```
因为路径可以从任意节点开始
        ↓
枚举每一个节点作为起点
        ↓
每个起点再向下 DFS
        ↓
最坏 O(n²)
        ↓
发现本质上是在询问“祖先到当前节点之间的路径和”
        ↓
转化为前缀和问题
        ↓
currentSum - previousSum = targetSum
        ↓
寻找 previousSum = currentSum - targetSum
        ↓
用 Hash Map O(1) 查询
        ↓
最终 O(n)
```

------

# 常见错误

## 1. 认为路径必须从根节点开始

错误思路：

```
current_sum += node.val

if current_sum == targetSum:
    result += 1
```

这样只能找到：

```
root -> ... -> node
```

这种路径。

但题目允许：

```
某个中间节点 -> ... -> node
```

所以必须能够“减掉前面的某段路径”。

这正是：

```
currentSum - previousSum
```

的作用。

------

## 2. 忘记 `{0: 1}`

错误：

```
prefix_count = {}
```

正确：

```
prefix_count = {0: 1}
```

否则所有：

```
从根节点开始
并且和恰好等于 targetSum
```

的路径都需要额外特殊处理。

`{0: 1}` 可以统一这些情况。

------

## 3. 忘记 Backtracking

错误：

```
prefix_count[current_sum] += 1

count += dfs(node.left, current_sum)
count += dfs(node.right, current_sum)

# 忘记删除
```

如果不回溯：

> 左子树产生的前缀和会污染右子树。

从而可能把：

```
左子树某节点 -> 右子树某节点
```

错误地当作合法路径。

正确：

```
prefix_count[current_sum] -= 1
```

------

## 4. 在递归左右子树之前 Backtracking

错误：

```
prefix_count[current_sum] += 1

prefix_count[current_sum] -= 1

count += dfs(node.left, current_sum)
count += dfs(node.right, current_sum)
```

这相当于当前节点还没有被它的子节点使用，就已经从路径中删除了。

正确顺序：

```
进入节点
    ↓
加入 prefix
    ↓
遍历 left
    ↓
遍历 right
    ↓
删除 prefix
    ↓
离开节点
```

也就是典型的：

```
Choose
Explore
Unchoose
```

回溯模式。

------

## 5. 使用全局前缀和集合而不是“当前路径”的前缀和

不能简单记录：

```
整棵树历史上所有 prefix sum
```

必须记录：

```
当前 root -> node 路径上的 prefix sum
```

因为合法路径必须满足祖先关系：

```
ancestor
   ↓
   ↓
descendant
```

而不能跨不同分支。

------

## 6. 认为出现相同前缀和时只需要保存一次

错误：

```
prefix_sums = set()
```

更合理的是：

```
prefix_count = {}
```

因为同一个前缀和可能出现多次，而每一次都可能对应不同的路径起点。

所以必须保存：

```
prefix_sum -> frequency
```

------

# 面试时最值得写出的版本

```
class Solution:
    def pathSum(
        self,
        root: Optional[TreeNode],
        targetSum: int
    ) -> int:

        # prefix_count[x]:
        # 当前 DFS 路径中，前缀和 x 出现的次数
        prefix_count = {0: 1}

        def dfs(node, current_sum):
            if not node:
                return 0

            # root -> 当前节点的路径和
            current_sum += node.val

            # current_sum - previous_sum = targetSum
            #
            # => previous_sum = current_sum - targetSum
            #
            # 每一个这样的 previous_sum
            # 都对应一条以当前节点结尾的合法路径
            result = prefix_count.get(
                current_sum - targetSum,
                0
            )

            # 当前前缀和加入 DFS 路径
            prefix_count[current_sum] = (
                prefix_count.get(current_sum, 0) + 1
            )

            # 继续搜索左右子树
            result += dfs(node.left, current_sum)
            result += dfs(node.right, current_sum)

            # Backtracking：
            # 离开当前节点，撤销当前前缀和
            prefix_count[current_sum] -= 1

            return result

        return dfs(root, 0)
```

可以把这个算法压缩成四句话记忆：

```
1. DFS 时维护 root -> 当前节点的 prefix sum

2. 如果一条路径的和为 target：
   currentSum - previousSum = target

3. 所以查找：
   previousSum = currentSum - target

4. DFS 返回时必须撤销当前 prefix，
   保证 Hash Map 只描述当前路径
```

最终：

```
Time:  O(n)

Space: O(h)
       最坏 O(n)
```

------

# 如何在面试中解释这个解法

可以按照下面的逻辑快速说明：

> 路径必须向下，因此任意合法路径实际上都是当前节点和某个祖先节点之间的一段连续路径。  
>
> 我在 DFS 的过程中维护从根到当前节点的前缀和 `currentSum`。如果之前某个祖先位置的前缀和是 `prefixSum`，那么这两个位置之间的路径和就是 `currentSum - prefixSum`。  
>
> 所以要使路径和等于 `targetSum`，需要满足 `prefixSum = currentSum - targetSum`。我用一个 Hash Map 记录当前 DFS 路径中每个前缀和出现的次数，因此可以 `O(1)` 查询有多少个合法起点。  
>
> 在离开节点时需要回溯删除当前前缀和，因为路径不能跨越左右子树。这样每个节点只访问一次，总时间复杂度是 `O(n)`。

这道题最值得建立的关联是：

```
Path Sum III
    ↓
树上的“连续向下路径”
    ↓
本质类似数组的“连续子数组”
    ↓
Prefix Sum
    ↓
prefix[j] - prefix[i] = target
    ↓
Hash Map 查找历史 prefix
```

以后看到类似：

> **求有多少段连续区间 / 连续路径的和等于某个值**

都可以优先考虑：

```
Prefix Sum + Hash Map
```

这个模式。