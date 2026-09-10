# 1. 两数相加（Add Two Numbers）[Easy]

给定两个**非空**链表 `l1` 和 `l2`，它们分别表示两个非负整数。

数字按照**逆序**存储。例如，数字 `321` 在链表中表示为：

```text
1 -> 2 -> 3
```

链表中的每个节点只存储一位数字。

可以假设这两个数字都不会包含前导零，除非数字本身就是 `0`。

请返回两个数字之和，并同样使用链表表示结果。

## 约束

- `1 <= l1.length, l2.length <= 100`
- `0 <= Node.val <= 9`

## 破局点

逆序存储使遍历方向与竖式加法一致；同时遍历两条链表并维护进位，循环条件要包含最后的 `carry`。

```python
# 单链表节点定义
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def addTwoNumbers(
        self,
        l1: Optional[ListNode],
        l2: Optional[ListNode]
    ) -> Optional[ListNode]:

        # dummy 是哑节点，不属于最终答案的一部分
        dummy = ListNode()

        # cur 始终指向结果链表的最后一个节点
        cur = dummy

        # 保存上一位产生的进位
        carry = 0

        # 只要：
        # 1. l1 还有节点
        # 2. l2 还有节点
        # 3. 或者仍然存在进位
        # 就需要继续处理
        while l1 or l2 or carry:

            # 某个链表结束后，对应数字视为 0
            v1 = l1.val if l1 else 0
            v2 = l2.val if l2 else 0

            # 当前位总和
            total = v1 + v2 + carry

            # 更新进位
            carry = total // 10

            # 当前结果数字
            digit = total % 10

            # 把当前数字加入结果链表
            cur.next = ListNode(digit)

            # cur 移动到新创建的节点
            cur = cur.next

            # 输入链表指针向后移动
            if l1:
                l1 = l1.next

            if l2:
                l2 = l2.next

        # dummy 本身不是答案，因此返回 dummy.next
        return dummy.next
```

# 2. 盛最多水的容器（Container With Most Water）[Easy]

给定一个整数数组 `heights`，其中：

```text
heights[i]
```

表示第 `i` 根竖线的高度。

你可以选择任意两根竖线，与 `x` 轴共同组成一个容器。

请返回这个容器能够盛放的**最大水量**。

## 约束

- `2 <= heights.length <= 100000`
- `0 <= heights[i] <= 10000`

## 破局点

面积由短板决定。宽度缩小时，移动长板不可能改善当前短板，所以每次只移动较短的一侧。

```python
from typing import List


class Solution:
    def maxArea(self, heights: List[int]) -> int:
        # 左右指针分别从数组两端开始
        l = 0
        r = len(heights) - 1

        # 记录最大容器面积
        res = 0

        while l < r:
            # 容器高度由较短的一边决定
            height = min(heights[l], heights[r])

            # 两根竖线之间的距离
            width = r - l

            # 当前容器面积
            area = height * width

            # 更新最大值
            res = max(res, area)

            # 移动较短的一侧
            #
            # 原因：
            # 宽度下一步一定会减少，
            # 只有尝试找到更高的短板，才有机会获得更大的面积。
            if heights[l] <= heights[r]:
                l += 1
            else:
                r -= 1

        return res
```

# 3. 复制带随机指针的链表（Copy Linked List with Random Pointer）

给定一个长度为 `n` 的链表头节点 `head`。

与普通单链表不同，每个节点除了包含 `next` 指针之外，还包含一个额外的 `random` 指针：

- `next` 指向链表中的下一个节点；
- `random` 可以指向链表中的**任意节点**，也可以为 `None`。

请创建该链表的一个**深拷贝（Deep Copy）**。

新的链表必须包含恰好 `n` 个**全新的节点**。对于每一个复制后的节点：

- `val` 与原节点相同；
- `next` 指向原节点 `next` 所对应的**复制节点**；
- `random` 指向原节点 `random` 所对应的**复制节点**。

特别注意：

> 新链表中的任何指针都不能指向原链表中的节点。

返回复制后链表的头节点。

题目示例通常使用 `[val, random_index]` 表示一个节点，其中：

- `val`：节点的值；
- `random_index`：该节点的 `random` 指针指向的节点下标；
- 如果 `random = None`，则对应 `random_index = null`。

## 破局点

把复制节点插在原节点后，使 `original.next` 就是其副本；于是复制节点的随机指针可由 `original.random.next` 得到，最后拆分并恢复原链表。

```python
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':

        if head is None:
            return None

        # ==================================================
        # Step 1：在每个原节点后插入它的复制节点
        #
        # A -> B -> C
        #
        # 变成：
        #
        # A -> A' -> B -> B' -> C -> C'
        # ==================================================

        l1 = head

        while l1 is not None:
            l2 = Node(l1.val)

            # 新节点先连接原来的下一个节点
            l2.next = l1.next

            # 原节点再连接复制节点
            l1.next = l2

            # 跳到下一个原节点
            l1 = l2.next

        # 复制链表的头一定是 head 后面的节点
        newHead = head.next

        # ==================================================
        # Step 2：设置复制节点的 random
        # ==================================================

        l1 = head

        while l1 is not None:

            if l1.random is not None:
                # l1.next 是 l1 的复制节点
                #
                # l1.random.next 是
                # l1.random 对应的复制节点
                l1.next.random = l1.random.next

            # 跳过复制节点，到下一个原节点
            l1 = l1.next.next

        # ==================================================
        # Step 3：拆开两个链表
        #
        # A -> A' -> B -> B'
        #
        # 恢复：
        # A -> B
        #
        # 得到：
        # A' -> B'
        # ==================================================

        l1 = head

        while l1 is not None:
            l2 = l1.next

            # 恢复原链表
            l1.next = l2.next

            # 连接复制链表
            if l2.next is not None:
                l2.next = l2.next.next

            # 前往下一个原节点
            l1 = l1.next

        return newHead
```

# 4. 二叉树的层序遍历（Binary Tree Level Order Traversal）[Easy]

给定一棵二叉树的根节点 `root`，返回该二叉树的**层序遍历**结果。

返回结果是一个二维列表，其中每个子列表包含树中**同一层的所有节点值**，并且节点按照**从左到右**的顺序排列。

例如，对于下面的二叉树：

```text
        3
       / \
      9   20
         /  \
        15   7
```

层序遍历结果为：

```text
[
    [3],
    [9, 20],
    [15, 7]
]
```

## 破局点

BFS 天然逐层遍历；进入每轮时固定 `level_size = len(q)`，只处理当前层，新加入的节点留到下一轮。

```python
from collections import deque


class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        # 空树
        if not root:
            return []

        res = []

        # deque 用作 BFS 队列
        q = deque([root])

        while q:
            # 当前队列中的节点数量
            # 就是当前这一层的节点数量
            level_size = len(q)

            # 保存当前层节点
            level = []

            # 只处理当前这一层
            for _ in range(level_size):
                # 从队列左侧取出节点
                node = q.popleft()

                level.append(node.val)

                # 将下一层节点加入队列
                if node.left:
                    q.append(node.left)

                if node.right:
                    q.append(node.right)

            # 当前层处理完毕
            res.append(level)

        return res
```

# 5. Binary Tree Right Side View（二叉树的右视图）

给定一棵二叉树的根节点 `root`，返回从二叉树**右侧观察时能够看到的节点值**，结果按照**从上到下**的顺序排列。

例如：

```text
        1
       / \
      2   3
       \   \
        5   4
```

从右侧观察，可以看到：

```text
1 → 3 → 4
```

因此返回：

```text
[1, 3, 4]
```

## 破局点

按“根、右、左”DFS；每个深度第一次访问的节点就是右视图节点，用 `depth == len(res)` 判断首次到达。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right


class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        # res[i] 保存深度 i 能够从右侧看到的节点
        res = []

        def dfs(node, depth):
            # 到达空节点，结束递归
            if not node:
                return

            # 如果 depth == len(res)，说明这是第一次访问这一层
            # 因为我们优先访问右子树，所以当前节点就是这一层最右侧的节点
            if depth == len(res):
                res.append(node.val)

            # 一定要先访问右子树
            dfs(node.right, depth + 1)

            # 然后再访问左子树
            dfs(node.left, depth + 1)

        dfs(root, 0)

        return res
```

# 6. 从前序遍历与中序遍历构造二叉树 [Difficult]

给定两个整数数组 `preorder` 和 `inorder`：

- `preorder`：某棵二叉树的**前序遍历**
- `inorder`：同一棵二叉树的**中序遍历**
- 两个数组长度相同，并且所有节点值**互不相同**

请根据前序遍历和中序遍历重新构造这棵二叉树，并返回其根节点。

## 破局点

前序遍历依次给出根节点，中序遍历划分左右子树。用哈希表定位根节点，并用全局前序指针按“根、左、右”推进。

```python
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

# 7. 课程表（Course Schedule）

给定一个数组 `prerequisites`，其中：

```text
prerequisites[i] = [a, b]
```

表示如果想学习课程 `a`，必须先完成课程 `b`。

例如，`[0, 1]` 表示必须先学习课程 `1`，然后才能学习课程 `0`。

总共有 `numCourses` 门课程，编号从 `0` 到 `numCourses - 1`。

如果可以完成所有课程，返回 `True`；否则返回 `False`。

## 示例 1

```text
输入：numCourses = 2, prerequisites = [[0, 1]]
输出：True
```

解释：可以先学习课程 `1`，再学习课程 `0`。

## 示例 2

```text
输入：numCourses = 2, prerequisites = [[0, 1], [1, 0]]
输出：False
```

解释：

- 学习课程 `0` 之前必须先学习课程 `1`
- 学习课程 `1` 之前又必须先学习课程 `0`

两门课程互相依赖，形成了环，因此无法完成所有课程。

## 数据范围

- `1 <= numCourses <= 1000`
- `0 <= prerequisites.length <= 1000`
- `prerequisites[i].length == 2`
- `0 <= a, b < numCourses`
- 所有先修课程关系都是唯一的

## 破局点

问题等价于判断有向图是否有环。Kahn 算法不断删除入度为 0 的节点；若最终处理数不足 `numCourses`，剩余节点必在环中。

```python
from collections import deque
from typing import List


class Solution:
    def canFinish(
        self,
        numCourses: int,
        prerequisites: List[List[int]]
    ) -> bool:
        # indegree[i] 表示课程 i 需要先完成的课程数量
        indegree = [0] * numCourses

        # graph[i] 表示完成课程 i 后可以继续学习的课程
        graph = [[] for _ in range(numCourses)]

        for course, prerequisite in prerequisites:
            # prerequisite -> course
            graph[prerequisite].append(course)
            indegree[course] += 1

        # 将所有没有先修课程的课程加入队列
        queue = deque()

        for course in range(numCourses):
            if indegree[course] == 0:
                queue.append(course)

        finished_courses = 0

        while queue:
            # popleft() 可以在 O(1) 时间内取出队首元素
            course = queue.popleft()
            finished_courses += 1

            # 完成当前课程后，更新后续课程的入度
            for next_course in graph[course]:
                indegree[next_course] -= 1

                # 所有先修课程均已完成
                if indegree[next_course] == 0:
                    queue.append(next_course)

        return finished_courses == numCourses
```

# 8. 组合总和（Combination Sum）

给定一个由不同整数组成的数组 `nums` 和一个目标整数 `target`，请返回所有和等于 `target` 的唯一组合。

`nums` 中的同一个数字可以被无限次选取。如果两个组合中每个数字出现的次数都相同，则认为它们是同一个组合。

结果可以按任意顺序返回，每个组合中的数字也可以按任意顺序排列。

> 以下算法要求 `nums` 中的元素均为正整数。若包含 `0` 或负数，由于每个数字可以无限次使用，搜索过程可能无法终止。

## 破局点

排序后从 `start` 向后枚举以避免排列重复；选择 `nums[j]` 后仍从 `j` 递归以允许复用，超过目标即可剪枝。

```python
from typing import List


class Solution:
    def combinationSum(
        self, nums: List[int], target: int
    ) -> List[List[int]]:
        result: List[List[int]] = []

        # 排序后可以在数字过大时提前结束枚举
        nums.sort()

        def dfs(
            start: int,
            current: List[int],
            total: int
        ) -> None:
            # 找到一个合法组合
            if total == target:
                result.append(current.copy())
                return

            # 只从 start 向后枚举，避免出现排列顺序不同的重复组合
            for j in range(start, len(nums)):
                next_total = total + nums[j]

                # nums 已排序，后续数字只会更大
                if next_total > target:
                    break

                # 选择 nums[j]
                current.append(nums[j])

                # 继续从 j 开始，使 nums[j] 可以被重复选择
                dfs(j, current, next_total)

                # 撤销选择
                current.pop()

        dfs(0, [], 0)
        return result
```

# 9. 零钱兑换（Coin Change）

给定一个整数数组 `coins`，其中每个元素表示一种硬币的面额；再给定一个整数 `amount`，表示目标金额。

请返回凑出目标金额所需要的最少硬币数量。如果无法恰好凑出该金额，则返回 `-1`。

每种硬币都可以使用无限次。

## 示例

```text
输入：coins = [1, 2, 5], amount = 11
输出：3
解释：11 = 5 + 5 + 1
输入：coins = [2], amount = 3
输出：-1
输入：coins = [1], amount = 0
输出：0
```

## 破局点

令 `dp[a]` 为凑出金额 `a` 的最少硬币数，则 `dp[a] = min(dp[a - coin] + 1)`；用 `amount + 1` 表示不可达。

```python
from typing import List


class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        # dp[a] 表示凑出金额 a 所需要的最少硬币数量。
        #
        # amount + 1 是一个不可能成为有效答案的值，
        # 因此用它表示“当前金额还无法凑出”。
        dp = [amount + 1] * (amount + 1)

        # 凑出金额 0 不需要任何硬币。
        dp[0] = 0

        # 从小金额开始计算，逐步得到更大金额的答案。
        for current_amount in range(1, amount + 1):
            for coin in coins:
                # 只有剩余金额非负时，才能选择这枚硬币。
                if current_amount >= coin:
                    dp[current_amount] = min(
                        dp[current_amount],
                        dp[current_amount - coin] + 1
                    )

        # 如果状态没有被更新，说明无法凑出目标金额。
        return -1 if dp[amount] == amount + 1 else dp[amount]
```

# 10. 每日温度（Daily Temperatures）

给定一个整数数组 `temperatures`，其中 `temperatures[i]` 表示第 `i` 天的温度。

请返回一个数组 `result`，其中 `result[i]` 表示：从第 `i` 天开始，需要等待多少天才能遇到温度更高的一天。

如果未来不存在温度更高的日子，则令 `result[i] = 0`。

例如：

```text
输入：temperatures = [73, 74, 75, 71, 69, 72, 76, 73]
输出：             [1,  1,  4,  2,  1,  1,  0,  0]
```

## 破局点

维护存放“尚未找到更暖日期”的下标单调栈；当前温度更高时持续弹栈，下标差就是等待天数。

```python
from typing import List


class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        res = [0] * n

        # 栈中保存日期下标，而不是温度本身
        stack = []

        for i, current_temperature in enumerate(temperatures):
            # 当前温度可以为栈顶日期提供答案
            while (
                stack
                and current_temperature > temperatures[stack[-1]]
            ):
                previous_day = stack.pop()
                res[previous_day] = i - previous_day

            # 当前日期等待未来某个更暖和的日子
            stack.append(i)

        return res
```

# 11. 字符串解码（Decode String）

给定一个经过编码的字符串 `s`，返回它解码后的字符串。

编码规则为 `k[encoded_string]`，方括号中的 `encoded_string` 需要重复恰好 `k` 次，其中 `k` 保证是正整数。

可以假设输入字符串始终有效：

- 不包含多余空格；
- 方括号一定正确匹配；
- 不会出现 `3a`、`2[4]`、`a[a]` 或 `a[2]` 等非法输入；
- 解码后的字符串长度不超过 `100,000`。

```text
输入：s = "3[a2[c]]"
输出："accaccacc"
```

## 破局点

遇到 `[` 时用两个栈保存外层字符串和重复次数；遇到 `]` 时弹栈并完成当前层拼接。数字要按十进制累积，以支持多位数。

```python
class Solution:
    def decodeString(self, s: str) -> str:
        # 保存进入每层括号之前的字符串
        string_stack = []

        # 保存每层括号对应的重复次数
        count_stack = []

        # 当前括号层已经构造出的字符串
        cur = ""

        # 当前正在解析的重复次数
        k = 0

        for c in s:
            if c.isdigit():
                # 累积多位数，例如 12：
                # 先得到 1，再得到 1 * 10 + 2 = 12
                k = k * 10 + int(c)

            elif c == "[":
                # 保存进入括号前的状态
                string_stack.append(cur)
                count_stack.append(k)

                # 开始构造括号内部的字符串
                cur = ""
                k = 0

            elif c == "]":
                # 当前括号内部的解码结果
                inner = cur

                # 恢复外层字符串和当前层的重复次数
                previous = string_stack.pop()
                count = count_stack.pop()

                # 将当前层结果拼接回外层
                cur = previous + inner * count

            else:
                # 普通字母直接加入当前字符串
                cur += c

        return cur
```

# 12. 编辑距离（Edit Distance）

给定两个仅由小写英文字母组成的字符串 `word1` 和 `word2`。

你可以对 `word1` 执行任意次数的以下三种操作：

- 在任意位置插入一个字符；
- 删除任意位置的一个字符；
- 替换任意位置的一个字符。

返回将 `word1` 转换成 `word2` 所需的最少操作次数。

## 破局点

令 `dp[i][j]` 表示把 `word1[i:]` 变成 `word2[j:]` 的最少操作数。字符相同则走右下角；不同则在删除、插入、替换三个后继状态中取最小值并加一。

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # 多出的一行和一列用于表示某个字符串已经处理完
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        # word1 已处理完：需要插入 word2 剩余字符
        for j in range(n + 1):
            dp[m][j] = n - j

        # word2 已处理完：需要删除 word1 剩余字符
        for i in range(m + 1):
            dp[i][n] = m - i

        # 从右下角向左上角填表
        for i in range(m - 1, -1, -1):
            for j in range(n - 1, -1, -1):
                if word1[i] == word2[j]:
                    # 当前字符相同，不需要操作
                    dp[i][j] = dp[i + 1][j + 1]
                else:
                    delete_cost = dp[i + 1][j]
                    insert_cost = dp[i][j + 1]
                    replace_cost = dp[i + 1][j + 1]

                    dp[i][j] = 1 + min(
                        delete_cost,
                        insert_cost,
                        replace_cost,
                    )

        return dp[0][0]
```

# 13. 轮转数组（Rotate Array）

给定一个整数数组 `nums`，将数组向右轮转 `k` 步。题目要求直接修改原数组，不返回新数组。

```text
输入：nums = [1,2,3,4,5,6,7], k = 3
输出：[5,6,7,1,2,3,4]
```

## 约束

- `1 <= nums.length <= 10^5`
- `-(2^31) <= nums[i] <= 2^31 - 1`
- `0 <= k <= 10^5`

## 破局点

先用 `k %= n` 去掉整圈的无效轮转。若把数组分成前缀 `A` 和长度为 `k` 的后缀 `B`，目标就是把 `A + B` 变成 `B + A`：反转整个数组后，再分别反转前 `k` 项和剩余项即可原地完成。

```python
from typing import List


class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        n = len(nums)
        k %= n

        def reverse(left: int, right: int) -> None:
            # 原地反转 nums[left:right + 1]
            while left < right:
                nums[left], nums[right] = nums[right], nums[left]
                left += 1
                right -= 1

        # A + B -> reverse(B) + reverse(A)
        reverse(0, n - 1)

        # reverse(B) -> B
        reverse(0, k - 1)

        # reverse(A) -> A
        reverse(k, n - 1)
```

# 14. Find All Anagrams in a String（找到字符串中所有字母异位词）

给定两个字符串 `s` 和 `p`，返回 `s` 中所有是 `p` 的字母异位词子串的起始索引。答案可以按任意顺序返回。

如果两个字符串包含完全相同的字符，且每个字符出现次数完全相同，只是排列顺序不同，那么它们互为字母异位词。

```text
输入：s = "cbaebabacd", p = "abc"
输出：[0, 6]

输入：s = "abab", p = "ab"
输出：[0, 1, 2]
```

## 约束

- `1 <= s.length, p.length <= 3 * 10^4`
- `s` 和 `p` 只包含小写英文字母。

## 破局点

异位词必须与 `p` 等长且字符频率相同，因此维护一个长度为 `len(p)` 的固定滑动窗口，只更新进入和离开的字符频率。

```python
from typing import List


class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        n, m = len(s), len(p)

        if m > n:
            return []

        p_count = [0] * 26
        window = [0] * 26

        for ch in p:
            p_count[ord(ch) - ord('a')] += 1

        res = []

        for right, ch in enumerate(s):
            # 当前字符进入窗口
            window[ord(ch) - ord('a')] += 1

            # 窗口长度超过 m 时，移除最左边字符
            if right >= m:
                old = s[right - m]
                window[ord(old) - ord('a')] -= 1

            # 窗口长度达到 m 后判断
            if right >= m - 1 and window == p_count:
                res.append(right - m + 1)

        return res
```

# 15. 在排序数组中查找元素的第一个和最后一个位置

给定一个按非递减顺序排列的整数数组 `nums` 和目标值 `target`，找出 `target` 在数组中的第一个位置和最后一个位置。

如果数组中不存在 `target`，返回 `[-1, -1]`。算法的时间复杂度必须为 `O(log n)`。

```text
输入：nums = [5, 7, 7, 8, 8, 10], target = 8
输出：[3, 4]

输入：nums = [5, 7, 7, 8, 8, 10], target = 6
输出：[-1, -1]

输入：nums = [], target = 0
输出：[-1, -1]
```

## 破局点

做两次二分搜索：找到 `target` 后不立即返回，找左边界时继续向左，找右边界时继续向右。

```python
from typing import List


class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        def find_boundary(find_left: bool) -> int:
            """
            find_left 为 True：寻找 target 的最左位置
            find_left 为 False：寻找 target 的最右位置
            """
            left, right = 0, len(nums) - 1
            result = -1

            while left <= right:
                mid = left + (right - left) // 2

                if nums[mid] < target:
                    # 当前值太小，target 只能在右边
                    left = mid + 1

                elif nums[mid] > target:
                    # 当前值太大，target 只能在左边
                    right = mid - 1

                else:
                    # 找到了 target，先记录当前位置
                    result = mid

                    if find_left:
                        # 继续搜索左侧，寻找更靠左的 target
                        right = mid - 1
                    else:
                        # 继续搜索右侧，寻找更靠右的 target
                        left = mid + 1

            return result

        first = find_boundary(True)
        last = find_boundary(False)

        return [first, last]
```

# 16. 寻找旋转排序数组中的最小值

给定一个长度为 `n` 的数组，它原本按升序排列，之后被旋转了 `1` 到 `n` 次。

例如，`nums = [1, 2, 3, 4, 5, 6]` 旋转 4 次后可变为 `[3, 4, 5, 6, 1, 2]`；旋转 6 次后仍为原数组。

假设 `nums` 中所有元素互不相同，请返回数组中的最小元素，并设计时间复杂度为 `O(log n)` 的算法。

## 破局点

将 `nums[mid]` 与 `nums[right]` 比较：若中点更小，最小值在 `[left, mid]`；若中点更大，最小值在 `[mid + 1, right]`。

```python
from typing import List


class Solution:
    def findMin(self, nums: List[int]) -> int:
        left, right = 0, len(nums) - 1

        # 循环结束条件是 left == right
        while left < right:
            # 这种写法可以避免某些语言中的整数溢出问题
            mid = left + (right - left) // 2

            if nums[mid] < nums[right]:
                # 最小值可能位于 mid，
                # 因此不能排除 mid
                right = mid
            else:
                # 因为元素互不相同，这里实际上是 nums[mid] > nums[right]
                # 最小值一定在 mid 的右边
                left = mid + 1

        # left 和 right 最终指向同一个位置，即最小元素的位置
        return nums[left]
```

# 17. 寻找重复数（Find the Duplicate Number）

给定一个整数数组 `nums`，其中包含 `n + 1` 个整数，并且每个整数都位于 `[1, n]` 范围内。

数组中恰好有一个整数重复出现，其他整数最多出现一次。请返回这个重复的整数。

要求不能修改原数组，并且额外空间必须为 `O(1)`。

## 破局点

把下标看作节点、`nums[i]` 看作下一节点，数组就形成了带环链表；重复数正是环入口。先用快慢指针相遇，再从起点和相遇点同步前进寻找入口。

```python
class Solution:
    def findDuplicate(self, nums: List[int]) -> int:
        slow = fast = 0

        # Phase 1: 找到环内相遇点
        while True:
            slow = nums[slow]
            fast = nums[nums[fast]]

            if slow == fast:
                break

        # Phase 2: 找到环入口
        finder = 0

        while finder != slow:
            finder = nums[finder]
            slow = nums[slow]

        return slow
```

# 18. Flatten Binary Tree to Linked List（二叉树展开为链表）

给定二叉树的根节点 `root`，请将这棵二叉树原地展开为一个单链表。

展开之后：

- 仍然使用原来的 `TreeNode` 节点；
- 每个节点的 `left` 都必须为 `None`；
- 每个节点的 `right` 指向链表中的下一个节点；
- 节点顺序必须与原二叉树的先序遍历顺序完全一致。

```text
原二叉树：
        1
       / \
      2   5
     / \   \
    3   4   6

展开后：1 -> 2 -> 3 -> 4 -> 5 -> 6
```

## 破局点

若当前节点有左子树，就找到左子树的最右节点，让它连接原右子树，再把整棵左子树移到右边并清空 `left`。

```python
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

# 19. 括号生成（Generate Parentheses）

给定一个整数 `n`，请返回所有能够由 `n` 对括号组成的、合法且完整的括号字符串。结果可以按照任意顺序返回。

合法括号字符串要求：

- 每一个右括号 `)` 都必须有对应的左括号 `(`；
- 任意前缀中，右括号数量都不能超过左括号数量；
- 最终左括号和右括号数量都恰好为 `n`。

```text
输入：n = 1
输出：["()"]

输入：n = 3
输出：["((()))", "(()())", "(())()", "()(())", "()()()"]
```

## 约束

- `1 <= n <= 7`

## 破局点

回溯时只生成合法前缀：左括号不足 `n` 时可加 `(`；右括号数量少于左括号时才可加 `)`。

```python
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        res = []
        path = []

        def backtrack(openN: int, closeN: int) -> None:
            if openN == closeN == n:
                res.append("".join(path))
                return

            if openN < n:
                path.append("(")
                backtrack(openN + 1, closeN)
                path.pop()

            if closeN < openN:
                path.append(")")
                backtrack(openN, closeN + 1)
                path.pop()

        backtrack(0, 0)

        return res
```

# 20. 字母异位词分组（Group Anagrams）

给定一个字符串数组 `strs`，请将所有互为字母异位词的字符串分组到同一个子列表中。返回结果的顺序可以任意。

字母异位词包含完全相同的字符，并且每个字符出现次数相同，只是排列顺序可能不同。

```text
输入：strs = ["act", "pots", "tops", "cat", "stop", "hat"]
输出：[["hat"], ["act", "cat"], ["stop", "pots", "tops"]]

输入：strs = ["x"]
输出：[["x"]]

输入：strs = [""]
输出：[[""]]
```

## 约束

- `1 <= strs.length <= 10000`
- `0 <= strs[i].length <= 100`
- `strs[i]` 只包含小写英文字母。

## 破局点

同组字符串需要共享一个哈希键。因为字符集固定为 26 个小写字母，可用字符频率元组作为唯一标识并用哈希表分组。

```python
from typing import List
from collections import defaultdict


class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        groups = defaultdict(list)

        for s in strs:
            # 统计 a-z 每个字符出现的次数
            count = [0] * 26

            for char in s:
                count[ord(char) - ord('a')] += 1

            # tuple 是不可变对象，可以作为哈希表 key
            groups[tuple(count)].append(s)

        return list(groups.values())
```

# 21. 打家劫舍（House Robber）

给定一个整数数组 `nums`，其中 `nums[i]` 表示第 `i` 间房屋中存放的金额。

所有房屋沿直线排列，因此第 `i` 间房屋与第 `i - 1` 间、第 `i + 1` 间房屋相邻。

你计划偷窃这些房屋，但不能同时偷窃两间相邻的房屋，否则安全系统会自动报警。

请返回在不触发警报的情况下，能够偷窃到的最大金额。

**破局点：** 每间房只需比较“不偷：上一状态”和“偷：上上状态 + 当前金额”。状态只依赖前两项，因此用两个变量完成 `O(n)` 时间、`O(1)` 空间的 DP。

```python
from typing import List


class Solution:
    def rob(self, nums: List[int]) -> int:
        # prev2：相当于 dp[i - 2]
        # prev1：相当于 dp[i - 1]
        prev2 = 0
        prev1 = 0

        for money in nums:
            # 不偷当前房屋：prev1
            # 偷当前房屋：prev2 + money
            current = max(prev1, prev2 + money)

            # 状态向前移动
            prev2 = prev1
            prev1 = current

        return prev1
```

# 22. 实现 Trie（前缀树）

**前缀树（Prefix Tree）**，也称为 **Trie（字典树）**，是一种树形数据结构，用于高效地存储和查询一组字符串。

实现 `PrefixTree` 类：

- `PrefixTree()`：初始化前缀树。
- `insert(word)`：将字符串 `word` 插入前缀树。
- `search(word)`：如果 `word` **完整存在**于前缀树中，即之前被插入过，则返回 `True`；否则返回 `False`。
- `startsWith(prefix)`：如果之前插入过的某个字符串以 `prefix` 为前缀，则返回 `True`；否则返回 `False`。

**破局点：** 相同前缀共享路径；节点除子节点外还要保存 `end`，用来区分“路径存在”和“完整单词存在”。

```python
class PrefixTreeNode:
    def __init__(self):
        # children[i] 保存字符 chr(ord('a') + i) 对应的子节点
        # 题目假设字符串只包含 a-z，因此固定使用长度为 26 的数组
        self.children = [None] * 26

        # 当前节点是否代表某个完整单词的结尾
        self.end = False


class PrefixTree:
    def __init__(self):
        # Trie 使用一个不代表任何字符的虚拟根节点
        self.root = PrefixTreeNode()

    def insert(self, word: str) -> None:
        curr = self.root

        # 从左到右处理 word 中的每个字符
        for c in word:
            # 将 'a' ~ 'z' 转换为 0 ~ 25
            i = ord(c) - ord("a")

            # 如果对应的子节点不存在，就创建它
            if curr.children[i] is None:
                curr.children[i] = PrefixTreeNode()

            # 移动到下一层
            curr = curr.children[i]

        # 整个 word 遍历结束后，
        # 当前节点就是这个完整单词的结尾
        curr.end = True

    def search(self, word: str) -> bool:
        curr = self.root

        for c in word:
            i = ord(c) - ord("a")

            # 如果某个字符对应的路径不存在，
            # 那么 word 一定没有被插入
            if curr.children[i] is None:
                return False

            curr = curr.children[i]

        # 路径存在并不意味着完整单词存在
        # 必须检查当前节点是不是某个单词的结尾
        return curr.end

    def startsWith(self, prefix: str) -> bool:
        curr = self.root

        for c in prefix:
            i = ord(c) - ord("a")

            # 前缀中的某个字符不存在，则此前缀不存在
            if curr.children[i] is None:
                return False

            curr = curr.children[i]

        # 只要整个 prefix 的路径存在即可，
        # 不要求当前节点是完整单词的结尾
        return True
```

# 23. Jump Game（跳跃游戏）

给定一个整数数组 `nums`，其中 `nums[i]` 表示你在索引 `i` 处**最多可以向前跳多少步**。

如果从索引 `0` 出发能够到达最后一个索引，返回 `True`；否则返回 `False`。

**破局点：** 从右向左维护最靠左的“好位置” `goal`；若 `i + nums[i] >= goal`，则 `i` 也能到终点。最终检查 `goal` 是否移到 `0`。

```python
from typing import List

class Solution:
    def canJump(self, nums: List[int]) -> bool:
        # goal 表示：
        # 当前最靠左的、确定能够到达最终终点的位置
        goal = len(nums) - 1

        # 从倒数第二个位置开始向左遍历
        for i in range(len(nums) - 2, -1, -1):

            # 如果从当前位置可以到达 goal
            # 那么当前位置本身也成为一个“好位置”
            if i + nums[i] >= goal:
                goal = i

        # 如果 goal 最终移动到了 0，
        # 说明从起点可以到达终点
        return goal == 0
```

# 24. Jump Game II（跳跃游戏 II）

给定一个整数数组 `nums`，其中 `nums[i]` 表示从索引 `i` 出发，**向右最多可以跳跃的距离**。

当位于索引 `i` 时，可以跳到任意索引 `i + j`，其中 `j <= nums[i]` 且 `i + j < len(nums)`。初始位置为索引 `0`。

请返回到达数组最后一个位置所需要的**最少跳跃次数**。题目保证一定存在一种方案可以到达最后一个位置。

**破局点：** 把当前一次跳跃能覆盖的区间视为 BFS 一层。扫描整层并记录下一层最远边界；到达当前边界时，跳数加一并更新边界。

```python
from typing import List

class Solution:
    def jump(self, nums: List[int]) -> int:
        jumps = 0

        # 当前 jumps 次跳跃能够覆盖到的最远边界
        current_end = 0

        # 在下一次跳跃之后能够到达的最远位置
        farthest = 0

        # 不需要处理最后一个位置，
        # 因为到达最后一个位置之后不需要继续跳
        for i in range(len(nums) - 1):

            # 不断扩大“下一跳”的最远覆盖范围
            farthest = max(farthest, i + nums[i])

            # 已经扫描完当前这一跳能够覆盖的所有位置
            if i == current_end:

                # 必须进行下一次跳跃
                jumps += 1

                # 更新新的覆盖边界
                current_end = farthest

        return jumps
```

# 25. 数组中的第 K 个最大元素

给定一个未排序的整数数组 `nums` 和一个整数 `k`，请返回数组中第 `k` 大的元素。

这里的“第 `k` 大”指数组按从小到大排序后所处的位置，并不是第 `k` 个不同的元素。因此，重复元素也会被正常计数。

进阶：你能否在不对整个数组排序的情况下解决这个问题？

**破局点：** 第 `k` 大对应升序下标 `len(nums) - k`。Quickselect 每次分区后只继续搜索目标所在的一侧，期望 `O(n)`；随机 pivot 可降低连续极端分区的概率。

```python
import random
from typing import List


class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 转换为升序数组中的目标下标
        target = len(nums) - k
        left = 0
        right = len(nums) - 1

        while left <= right:
            # 随机选择 pivot，降低连续出现最坏分区的概率
            pivot_index = random.randint(left, right)

            # 将 pivot 移到最右侧，方便统一执行分区
            nums[pivot_index], nums[right] = (
                nums[right],
                nums[pivot_index],
            )
            pivot = nums[right]

            insert = left

            # 将所有 <= pivot 的元素移动到左侧
            for i in range(left, right):
                if nums[i] <= pivot:
                    nums[insert], nums[i] = nums[i], nums[insert]
                    insert += 1

            # 将 pivot 放到最终位置
            nums[insert], nums[right] = nums[right], nums[insert]

            if insert == target:
                return nums[insert]
            elif insert > target:
                right = insert - 1
            else:
                left = insert + 1

        # 在题目保证输入合法时，不会执行到这里
        raise ValueError("k 超出有效范围")
```

# 26. 二叉搜索树中第 K 小的整数

给定一棵二叉搜索树（BST）的根节点 `root` 和一个整数 `k`，返回树中**第 `k` 小的值**。

这里的 `k` 从 **1 开始计数（1-indexed）**。也就是说，`k = 1` 返回最小值，`k = 2` 返回第二小的值，以此类推。

**破局点：** BST 的中序遍历天然升序。用栈模拟 `Left → Node → Right`，访问到第 `k` 个节点立即返回，无需遍历整棵树。

```python
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

# 27. 电话号码的字母组合

给定一个字符串 `digits`，其中只包含数字 `2` 到 `9`。

每个数字都对应手机九宫格键盘上的一组字母：

```text
2 -> abc
3 -> def
4 -> ghi
5 -> jkl
6 -> mno
7 -> pqrs
8 -> tuv
9 -> wxyz
```

每个数字都可以代表它映射到的任意一个字母。请返回 `digits` 能表示的所有可能字母组合，结果可以按任意顺序返回。

约束：`0 <= digits.length <= 4`，`2 <= digits[i] <= 9`。

**破局点：** 每个数字对应决策树的一层，枚举该数字的所有字母并递归到下一层；处理完全部数字时收集路径。

```python
class Solution:
    def letterCombinations(self, digits: str) -> list[str]:
        if not digits:
            return []

        mapping = {
            "2": "abc",
            "3": "def",
            "4": "ghi",
            "5": "jkl",
            "6": "mno",
            "7": "pqrs",
            "8": "tuv",
            "9": "wxyz",
        }

        res = []

        def backtrack(i: int, cur: str) -> None:
            if i == len(digits):
                res.append(cur)
                return

            for c in mapping[digits[i]]:
                backtrack(i + 1, cur + c)

        backtrack(0, "")

        return res
```

# 28. Linked List Cycle II（环形链表 II）

给定一个链表的头节点 `head`，如果链表中存在环，请返回**环开始的第一个节点**；如果链表不存在环，则返回 `None`。

要求：

- **不能修改链表**。
- `pos` 只是评测系统为了描述链表结构而使用的辅助信息，**不会作为参数传入**。

**破局点：** Floyd 算法分两阶段：快慢指针先在环内相遇；再让一个指针回到头节点，两者同速前进，第二次相遇点就是环入口。

```python
from typing import Optional


class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow = head
        fast = head

        # Phase 1:
        # 判断是否存在环，并找到第一次相遇点
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

            if slow == fast:
                # Phase 2:
                # 从 head 和相遇点同时出发
                slow = head

                while slow != fast:
                    slow = slow.next
                    fast = fast.next

                # 第二次相遇的位置就是环入口
                return slow

        # fast 到达 None，说明不存在环
        return None
```

# 29. 最长公共子序列

给定两个字符串 `text1` 和 `text2`，返回它们的最长公共子序列（Longest Common Subsequence，简称 LCS）的长度。如果不存在公共子序列，则返回 `0`。

**子序列**是指：从原序列中删除任意数量的元素（也可以一个都不删除），并且不改变剩余元素的相对顺序，所得到的新序列。

两个字符串的**公共子序列**，是指同时为这两个字符串子序列的字符串。

**破局点：** 令 `dp[i][j]` 表示两个后缀的 LCS 长度。字符相同就同时前进；不同就分别跳过一个字符取较大值。依赖右、下、右下状态，所以从右下向左上填表。

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        m, n = len(text1), len(text2)

        # dp[i][j] 表示 text1[i:] 和 text2[j:] 的 LCS 长度
        # 额外的一行和一列表示空字符串
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        # 从右下角向左上角填表
        for i in range(m - 1, -1, -1):
            for j in range(n - 1, -1, -1):
                if text1[i] == text2[j]:
                    # 当前字符相同，同时移动两个索引
                    dp[i][j] = 1 + dp[i + 1][j + 1]
                else:
                    # 跳过 text1[i] 或 text2[j]
                    dp[i][j] = max(
                        dp[i + 1][j],
                        dp[i][j + 1]
                    )

        return dp[0][0]
```

# 30. 最长连续序列（Longest Consecutive Sequence）

给定一个整数数组 `nums`，返回其中能够组成的**最长连续序列的长度**。

所谓连续序列，是指序列中的后一个元素恰好比前一个元素大 `1`。

需要注意：

- 这些元素**不要求在原数组中相邻**。
- 只要求它们的数值能够构成连续整数。
- 题目要求最终算法的时间复杂度为 **`O(n)`**。

**破局点：** 先用集合去重。只从不存在前驱 `num - 1` 的数字开始向后计数，确保每条连续序列只扫描一次，整体为 `O(n)`。

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        # 用 set 去重，同时支持平均 O(1) 查询
        num_set = set(nums)

        # 保存最长连续序列长度
        longest = 0

        # 遍历 set 而不是原数组，可以避免重复数字带来的重复处理
        for num in num_set:

            # 只有 num - 1 不存在时，
            # num 才是某个连续序列的起点
            if num - 1 not in num_set:

                current = num
                streak = 1

                # 从起点向后寻找连续数字
                while current + 1 in num_set:
                    current += 1
                    streak += 1

                # 更新最长长度
                longest = max(longest, streak)

        return longest
```

# 31. 最长递增子序列（Longest Increasing Subsequence, LIS）

给定一个整数数组 `nums`，返回其中**最长严格递增子序列**的长度。

**子序列（subsequence）\**是指：从原序列中删除任意数量（也可以一个都不删除）的元素，并且\**不改变剩余元素之间的相对顺序**，得到的新序列。

例如：

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]

一个最长递增子序列可以是：
[2, 3, 7, 101]

长度为 4。
```

注意这里要求的是**严格递增**：

```
[1, 2, 2, 3]
```

不能把两个 `2` 同时放进严格递增子序列中。

## 破局点

维护 `dp[i]` 为以第i个数字结尾的递增子序列，状态转移方程为`dp[i]=max(dp[j]+1,dp[i])`·其中j是小于i的所有可能数字。

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        #dp[i] the length of the sequence end with i
        dp = [1]*len(nums)
        for i in range(1,len(nums)):
            for j in range(i):
                if nums[i] > nums[j]:
                    dp[i]=max(dp[j]+1,dp[i])
        return max(dp)
```

# 32. 最长回文子串（Longest Palindromic Substring）

给定一个字符串 `s`，返回 `s` 中最长的**回文子串**。

**回文串（Palindrome）**是指正着读和反着读完全相同的字符串。

如果存在多个长度相同的最长回文子串，返回其中任意一个即可。

## 约束

- `1 <= s.length <= 1000`
- `s` 只包含数字和英文字母

## 破局点

回文串围绕中心对称。枚举每个字符，同时以 `(i, i)` 和 `(i, i + 1)` 为中心向两侧扩展，分别覆盖奇数和偶数长度的回文串。

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        resIdx = 0
        resLen = 0
        n = len(s)

        for i in range(n):
            # -----------------------
            # 情况 1：奇数长度回文串
            # 中心为 s[i]
            # -----------------------
            l, r = i, i

            while l >= 0 and r < n and s[l] == s[r]:
                currLen = r - l + 1

                if currLen > resLen:
                    resIdx = l
                    resLen = currLen

                # 继续向两侧扩展
                l -= 1
                r += 1

            # -----------------------
            # 情况 2：偶数长度回文串
            # 中心位于 i 和 i+1 之间
            # -----------------------
            l, r = i, i + 1

            while l >= 0 and r < n and s[l] == s[r]:
                currLen = r - l + 1

                if currLen > resLen:
                    resIdx = l
                    resLen = currLen

                l -= 1
                r += 1

        return s[resIdx:resIdx + resLen]
```

# 33. 无重复字符的最长子串（Longest Substring Without Repeating Characters）

给定一个字符串 `s`，请找出其中**不包含重复字符的最长子串长度**。

这里的**子串（substring）\**指的是字符串中一段\**连续的字符序列**。

## 示例 1

```text
Input: s = "zxyzxyz"

Output: 3
```

最长的不重复子串可以是：

```text
"zxy"
"xyz"
"yzx"
```

长度都是：

```text
3
```

## 示例 2

```text
Input: s = "xxxx"

Output: 1
```

因为任何长度大于 `1` 的子串都会包含重复的 `x`。

## 约束

- `0 <= s.length <= 50,000`
- `s` 可能包含可打印 ASCII 字符

## 破局点

用哈希表记录字符最后出现的位置。从左向右，遇到重复字符时，将左边界直接跳到旧位置后一位；用 `max` 保证左边界永不后退。

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # 记录：
        # 字符 -> 最近一次出现的索引
        last_seen = {}

        # 当前窗口左边界
        l = 0

        # 最长无重复子串长度
        res = 0

        # r 是当前窗口右边界
        for r in range(len(s)):
            char = s[r]

            # 如果这个字符以前出现过，
            # 左边界至少要移动到其上一次位置的后一位
            #
            # max(...) 非常重要：
            # 防止左指针向后移动
            if char in last_seen:
                l = max(l, last_seen[char] + 1)

            # 更新当前字符最近一次出现的位置
            last_seen[char] = r

            # 当前窗口为 s[l : r + 1]
            res = max(res, r - l + 1)

        return res
```

# 34. 二叉树的最近公共祖先（Lowest Common Ancestor of a Binary Tree）

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

## 破局点

递归返回当前子树中找到的 `p`、`q` 或 LCA。若左右子树都返回非空节点，当前根就是 LCA；否则将非空的一侧继续向上传递。

```python
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

# 35. LRU Cache（最近最少使用缓存）

实现一个 **LRU（Least Recently Used，最近最少使用）缓存**类 `LRUCache`。

需要支持以下操作：

- `LRUCache(int capacity)`：初始化一个容量为 `capacity` 的 LRU 缓存。
- `int get(int key)`：
  - 如果 `key` 存在，返回对应的 `value`；
  - 否则返回 `-1`。
- `void put(int key, int value)`：
  - 如果 `key` 已经存在，更新它对应的 `value`；
  - 如果 `key` 不存在，则插入新的 `(key, value)`；
  - 如果插入后缓存容量超过 `capacity`，删除**最近最少使用**的 key。

只要某个 key 被执行过 `get` 或 `put`，就认为它刚刚被使用过，也就是成为 **Most Recently Used（MRU，最近使用）** 的元素。

要求：

> `get()` 和 `put()` 的平均时间复杂度都必须达到 **O(1)**。

## 破局点

哈希表负责 O(1) 定位节点，双向链表负责 O(1) 删除和移动节点。链表左端保存 LRU、右端保存 MRU，虚拟头尾节点可统一所有边界操作。

```python
class Node:
    def __init__(self, key: int, val: int):
        self.key = key
        self.val = val

        # 双向链表指针
        self.prev = None
        self.next = None


class LRUCache:

    def __init__(self, capacity: int):
        self.capacity = capacity

        # Hash Map:
        # key -> 对应的链表 Node
        self.cache = {}

        # Dummy Nodes
        #
        # left.next  永远指向 LRU
        # right.prev 永远指向 MRU
        self.left = Node(0, 0)
        self.right = Node(0, 0)

        # 初始化：
        # left <-> right
        self.left.next = self.right
        self.right.prev = self.left

    def _remove(self, node: Node) -> None:
        """
        从双向链表中删除 node。
        已知 node 的情况下时间复杂度 O(1)。
        """
        prev_node = node.prev
        next_node = node.next

        prev_node.next = next_node
        next_node.prev = prev_node

    def _insert_mru(self, node: Node) -> None:
        """
        将 node 插入 right 前面，
        使其成为 Most Recently Used。
        """
        prev_node = self.right.prev

        # prev_node <-> node
        prev_node.next = node
        node.prev = prev_node

        # node <-> right
        node.next = self.right
        self.right.prev = node

    def get(self, key: int) -> int:
        # Hash Map 平均 O(1) 查找
        if key not in self.cache:
            return -1

        node = self.cache[key]

        # get 也算一次访问，
        # 所以需要把它移动到 MRU
        self._remove(node)
        self._insert_mru(node)

        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            # key 已存在：
            # 更新 value，并移动到 MRU
            node = self.cache[key]
            node.val = value

            self._remove(node)
            self._insert_mru(node)

        else:
            # key 不存在：
            # 创建新的节点
            node = Node(key, value)

            # 同时加入 Hash Map 和 Linked List
            self.cache[key] = node
            self._insert_mru(node)

            # 如果超过容量，删除 LRU
            if len(self.cache) > self.capacity:
                # left.next 永远是 LRU
                lru = self.left.next

                # 从链表删除
                self._remove(lru)

                # 从 Hash Map 删除
                del self.cache[lru.key]
```

# 36. Maximum Product Subarray（乘积最大子数组）

给定一个整数数组 `nums`，请找出数组中**乘积最大的连续非空子数组**，并返回该乘积。

**子数组（subarray）** 是数组中一段连续、非空的元素序列。

可以假设最终答案能够存储在 **32 位整数**范围内。

> 注意：如果子数组只有一个元素，那么它的乘积就是这个元素本身。

## 破局点

负数会让最大值和最小值互换角色，因此同时维护以当前位置结尾的最大、最小乘积。每轮都必须基于更新前的两个状态计算。

```python
from typing import List


class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        # 全局最大乘积
        res = nums[0]

        # curMax：以上一个位置结尾的最大乘积
        # curMin：以上一个位置结尾的最小乘积
        curMax = 1
        curMin = 1

        for num in nums:
            # 保存更新前的状态
            prevMax = curMax
            prevMin = curMin

            # 当前位置可以：
            # 1. 自己重新开始一个子数组
            # 2. 接在之前最大乘积之后
            # 3. 接在之前最小乘积之后
            curMax = max(
                num,
                num * prevMax,
                num * prevMin
            )

            curMin = min(
                num,
                num * prevMax,
                num * prevMin
            )

            # 更新全局答案
            res = max(res, curMax)

        return res
```

# 37. 最大子数组和（Maximum Subarray）

给定一个整数数组 `nums`，请找出一个具有最大和的 **连续子数组（subarray）**，并返回这个最大和。

**子数组** 是数组中一段连续且非空的元素序列。

## 示例 1

```text
输入：nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

输出：6
```

解释：

```text
[4, -1, 2, 1]
```

的元素和为：

```text
4 + (-1) + 2 + 1 = 6
```

这是所有连续子数组中的最大和。

## 示例 2

```text
输入：nums = [1]

输出：1
```

## 示例 3

```text
输入：nums = [5, 4, -1, 7, 8]

输出：23
```

解释：

```text
[5, 4, -1, 7, 8]
```

整个数组的和为：

```text
23
```

## 破局点

令 `cur_sum` 表示以当前位置结尾的最大子数组和。每到一个元素，只需比较“从当前元素重新开始”和“接在此前子数组后面”，并同步更新全局最大值。

```python
from typing import List


class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        # 以当前位置结尾的最大子数组和
        cur_sum = nums[0]

        # 全局最大子数组和
        res = nums[0]

        for i in range(1, len(nums)):
            num = nums[i]

            # 当前只有两种选择：
            # 1. 从当前元素重新开始
            # 2. 延续之前的子数组
            cur_sum = max(
                num,
                cur_sum + num
            )

            # 更新全局答案
            res = max(
                res,
                cur_sum
            )

        return res
```

# 38. 合并区间（Merge Intervals）

给定一个区间数组 `intervals`，其中 `intervals[i] = [start_i, end_i]`。

请合并所有互相重叠的区间，并返回合并后的、彼此不重叠的区间数组。返回结果的顺序可以任意。

> 两个区间如果存在至少一个公共点，就视为重叠。  
> 例如：
>
> - `[1, 2]` 和 `[3, 4]` 不重叠；
> - `[1, 2]` 和 `[2, 3]` 重叠，因为它们在点 `2` 相交。

## 示例 1

```text
输入：intervals = [[1,3],[1,5],[6,7]]

输出：[[1,5],[6,7]]
```

解释：

- `[1,3]` 与 `[1,5]` 重叠，因此可以合并为 `[1,5]`；
- `[6,7]` 与 `[1,5]` 不重叠，因此单独保留。

## 示例 2

```text
输入：intervals = [[1,2],[2,3]]

输出：[[1,3]]
```

虽然第一个区间在 `2` 结束，第二个区间在 `2` 开始，但二者共享点 `2`，因此属于重叠区间。

## 约束条件

- `1 <= intervals.length <= 1000`
- `intervals[i].length == 2`
- `0 <= start <= end <= 1000`

## 破局点

先按起点排序，使可能重叠的区间相邻。遍历时只需与结果中的最后一个区间比较：起点大于其终点就新增，否则扩展最后区间的终点。

```python
from typing import List


class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        # 1. 按起点排序
        intervals.sort(key=lambda interval: interval[0])

        merged = []

        for start, end in intervals:
            # merged 为空，或者当前区间与最后一个区间不重叠
            if not merged or start > merged[-1][1]:
                merged.append([start, end])
            else:
                # 存在重叠，扩展最后一个区间的终点
                merged[-1][1] = max(merged[-1][1], end)

        return merged
```

# 39. 最小路径和（Minimum Path Sum）

给定一个包含非负整数的 `m × n` 网格 `grid`，请找出一条从左上角到右下角的路径，使得路径上所有数字之和最小。

每次只能向右或向下移动一步。

## 破局点

从右下向左上计算，当前位置的最小路径和等于当前值加上下方、右方结果的较小值。一维 `dp` 中，更新前的 `dp[c]` 是下方，`dp[c + 1]` 是右方，因此内层必须从右向左遍历。

```python
from typing import List


class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:
        rows, cols = len(grid), len(grid[0])

        # dp[c] 在更新前表示下方格子的结果
        # dp[c + 1] 表示右侧格子的结果
        dp = [float("inf")] * (cols + 1)

        # 为右下角设置虚拟入口
        dp[cols - 1] = 0

        for r in range(rows - 1, -1, -1):
            # 必须从右向左遍历
            for c in range(cols - 1, -1, -1):
                dp[c] = grid[r][c] + min(
                    dp[c],      # 来自下方
                    dp[c + 1],  # 来自右侧
                )

        return dp[0]
```

# 40. 最小栈（Min Stack）

设计一个栈，使其支持以下操作：

- `MinStack()`：初始化栈对象。
- `push(val)`：将元素 `val` 压入栈中。
- `pop()`：删除栈顶元素。
- `top()`：获取栈顶元素。
- `getMin()`：获取栈中的最小元素。

要求每个操作的时间复杂度均为 \(O(1)\)。

## 破局点

主栈保存实际元素，辅助栈在每一层保存“截至该层的最小值”。两栈同步压入和弹出，因此辅助栈顶始终是当前最小值。

```python
class MinStack:
    def __init__(self):
        # 保存实际元素
        self.stack = []

        # min_stack[i] 表示 stack[0:i+1] 中的最小值
        self.min_stack = []

    def push(self, val: int) -> None:
        self.stack.append(val)

        if not self.min_stack:
            # 第一个元素本身就是当前最小值
            current_min = val
        else:
            # 新的最小值是 val 与原最小值中的较小者
            current_min = min(val, self.min_stack[-1])

        self.min_stack.append(current_min)

    def pop(self) -> None:
        # 两个栈必须同时弹出，保持长度和层级一致
        self.stack.pop()
        self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        # 辅助栈的栈顶始终是当前最小值
        return self.min_stack[-1]
```

# 41. 下一个排列（Next Permutation）

整数数组的一个**排列（Permutation）**，是指将数组中的所有元素按照某种顺序重新排列得到的序列。

数组的**下一个排列（Next Permutation）**，指的是按照**字典序（Lexicographical Order）**排列后，紧接在当前排列之后的那个排列。如果当前排列已经是字典序最大的排列，则需要将数组重新排列成字典序最小的排列，也就是升序排列。

例如：

```text
[1, 2, 3] → [1, 3, 2]
[2, 3, 1] → [3, 1, 2]
[3, 2, 1] → [1, 2, 3]
```

给定一个整数数组 `nums`，请找出 `nums` 的下一个排列。

要求：

- 必须**原地（in-place）修改** `nums`
- 只能使用 **O(1)** 的额外空间

**破局点：** 从右向左找到第一个升序转折点 `pivot`；用右侧刚好更大的数替换它，再反转原本降序的后缀，使增幅最小。若不存在转折点，直接反转整个数组。

```python
class Solution:
    def nextPermutation(self, nums: list[int]) -> None:
        """
        原地将 nums 修改为它的下一个字典序排列。
        """

        n = len(nums)

        # --------------------------------------------------
        # Step 1:
        # 从右向左寻找第一个 nums[i] < nums[i + 1] 的位置。
        # nums[i] 就是 pivot。
        # --------------------------------------------------
        i = n - 2

        while i >= 0 and nums[i] >= nums[i + 1]:
            i -= 1

        # --------------------------------------------------
        # Step 2:
        # 如果找到了 pivot，
        # 从右向左寻找第一个严格大于 nums[i] 的元素。
        # --------------------------------------------------
        if i >= 0:
            j = n - 1

            while nums[j] <= nums[i]:
                j -= 1

            # 用后缀中“刚好比 pivot 大”的元素替换 pivot。
            nums[i], nums[j] = nums[j], nums[i]

        # --------------------------------------------------
        # Step 3:
        # 将 pivot 后面的降序部分反转成升序，
        # 从而得到尽可能小的后缀。
        #
        # 如果 i == -1，则这里相当于反转整个数组。
        # --------------------------------------------------
        left = i + 1
        right = n - 1

        while left < right:
            nums[left], nums[right] = nums[right], nums[left]

            left += 1
            right -= 1
```

# 42. 岛屿数量（Number of Islands）

给定一个二维网格 `grid`，其中：

- `'1'` 表示陆地
- `'0'` 表示水域

请计算并返回网格中**岛屿的数量**。

一个岛屿由若干个在**水平或垂直方向相邻**的陆地单元格组成。也就是说，只考虑上、下、左、右四个方向，不考虑对角线连接。

可以假设网格外围全部是水。

**破局点：** 每遇到一块未访问的陆地，岛屿数加一，并用 DFS 将与它四向相连的整座岛屿“淹没”；之后不会重复计数。

```python
from typing import List


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        # 四个搜索方向：
        # 下、上、右、左
        directions = [
            (1, 0),
            (-1, 0),
            (0, 1),
            (0, -1)
        ]

        ROWS, COLS = len(grid), len(grid[0])
        islands = 0

        def dfs(r: int, c: int) -> None:
            # 1. 越界
            # 2. 当前不是陆地
            # 都不需要继续搜索
            if (
                r < 0
                or c < 0
                or r >= ROWS
                or c >= COLS
                or grid[r][c] == "0"
            ):
                return

            # 标记当前陆地已经访问过
            # 直接修改原 grid，因此不需要额外 visited 集合
            grid[r][c] = "0"

            # 搜索四个相邻方向
            for dr, dc in directions:
                dfs(r + dr, c + dc)

        # 遍历整个网格
        for r in range(ROWS):
            for c in range(COLS):
                if grid[r][c] == "1":
                    # 每遇到一个还没有访问过的陆地，
                    # 就意味着发现了一个新的岛屿
                    islands += 1

                    # 把整个岛屿全部标记为已访问
                    dfs(r, c)

        return islands
```

# 43. 分割回文串（Palindrome Partitioning）

给定一个字符串 `s`，将它分割成若干个子串，使得分割后的每个子串都是回文串。

返回所有可能的回文串分割方案，答案可以按任意顺序排列。

回文串是指正着读和倒着读都相同的字符串，例如 `"a"`、`"aba"`、`"abba"`。

```text
输入：s = "aab"

输出：
[
    ["a", "a", "b"],
    ["aa", "b"]
]
```

**破局点：** 从 `start` 开始枚举下一段的终点；只有当前子串是回文串时才选择它，然后递归处理剩余字符串，返回后撤销选择。

```python
from typing import List


class Solution:
    def partition(self, s: str) -> List[List[str]]:
        res: List[List[str]] = []
        part: List[str] = []

        def dfs(start: int) -> None:
            # start 到达字符串末尾，得到一个完整分割方案
            if start == len(s):
                # 必须保存副本，因为 part 后续还会被修改
                res.append(part.copy())
                return

            # 枚举当前子串的结束位置
            for end in range(start, len(s)):
                if self.is_palindrome(s, start, end):
                    # 做出选择
                    part.append(s[start:end + 1])

                    # 递归处理剩余字符串
                    dfs(end + 1)

                    # 撤销选择
                    part.pop()

        dfs(0)
        return res

    def is_palindrome(self, s: str, left: int, right: int) -> bool:
        """判断闭区间 s[left:right+1] 是否为回文串。"""
        while left < right:
            if s[left] != s[right]:
                return False

            left += 1
            right -= 1

        return True
```

# 44. 分割等和子集（Partition Equal Subset Sum）

给定一个只包含正整数的数组 `nums`。

请判断是否可以将数组划分为两个子集 `subset1` 和 `subset2`，使得：

```text
sum(subset1) == sum(subset2)
```

如果可以，返回 `True`；否则返回 `False`。

**破局点：** 总和为奇数时必然无解；否则问题等价于能否选出和为 `total // 2` 的子集。定义：`dp[i][j]`表示使用前 `i` 个数字，能否组成和 `j`。状态转移方程为
```python
dp[i][j] = (
    dp[i - 1][j]
    or
    dp[i - 1][j - num]
)
```

```python
from typing import List


class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False

        target = total // 2
        n = len(nums)

        # dp[i][j]:
        # 使用前 i 个数字，能否凑出和 j
        dp = [
            [False] * (target + 1)
            for _ in range(n + 1)
        ]

        # 和为 0 永远可以通过“什么都不选”实现
        for i in range(n + 1):
            dp[i][0] = True

        for i in range(1, n + 1):
            num = nums[i - 1]

            for j in range(1, target + 1):
                # 不选择当前数字
                dp[i][j] = dp[i - 1][j]

                # 如果当前数字可以放入
                if num <= j:
                    # 或者选择当前数字
                    dp[i][j] = (
                        dp[i][j]
                        or dp[i - 1][j - num]
                    )

        return dp[n][target]
```

# 45. 划分字母区间（Partition Labels）

给定一个只包含小写英文字母的字符串 `s`。

将字符串划分成**尽可能多**的子字符串，并保证：

> 同一个字母最多只能出现在其中一个子字符串中。

返回一个整数列表，表示各个子字符串的长度。

**破局点：** 预先记录每个字符最后出现的位置。扫描时不断扩大当前分区的最远边界；当索引到达该边界，分区内所有字符都不会在后面出现，应立即切分。

```python
from typing import List


class Solution:
    def partitionLabels(self, s: str) -> List[int]:
        # last_index[c]：字符 c 最后一次出现的位置
        last_index = {}

        # 第一次遍历：
        # 相同字符的索引会不断被覆盖，
        # 最终留下的就是它最后一次出现的位置
        for i, c in enumerate(s):
            last_index[c] = i

        res = []
        size = 0  # 当前分区长度
        end = 0   # 当前分区必须到达的最远位置

        # 第二次遍历
        for i, c in enumerate(s):
            size += 1

            # 当前字符如果在更远的位置还会出现，
            # 当前分区必须扩展到那里
            end = max(end, last_index[c])

            # 已到达当前分区中所有字符的最远最后位置
            if i == end:
                res.append(size)
                size = 0

        return res
```

# 46. 路径总和 III（Path Sum III）

给定  一棵二叉树的根节点 `root` 和一个整数 `targetSum`。

请返回二叉树中**节点值之和等于 `targetSum` 的路径数量**。

需要注意：

- 路径**不要求从根节点开始**
- 路径**不要求在叶子节点结束**
- 路径必须是**连续的**
- 路径方向必须始终**从父节点向子节点**
- 同一条路径不能“拐回去”访问父节点

例如：

```text
        10
       /  \
      5   -3
     / \    \
    3   2    11
   / \   \
  3  -2   1

targetSum = 8
```

合法路径为 `5 -> 3`、`5 -> 2 -> 1`、`-3 -> 11`，因此答案为 `3`。

**破局点：** DFS 维护当前根路径上的前缀和计数。以当前节点结尾的合法路径数，就是此前前缀和 `current_sum - targetSum` 的出现次数；离开节点时必须回溯计数，避免不同分支互相污染。

```python
from typing import Optional


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

# 47. 完全平方数（Perfect Squares）

给定一个整数 `n`，返回**和为 `n` 的完全平方数的最少数量**。

**完全平方数（Perfect Square）**是某个整数的平方，例如 `1, 4, 9, 16, 25, ...`。

```text
n = 12
12 = 4 + 4 + 4
答案为 3

n = 13
13 = 9 + 4
答案为 2
```

**破局点：** 定义 `dp[target]` 为组成 `target` 所需的最少完全平方数个数。枚举最后选择的平方数 `square`，状态转移为 `dp[target] = min(dp[target], 1 + dp[target - square])`。

```python
class Solution:
    def numSquares(self, n: int) -> int:
        # dp[target] 表示：
        # 和为 target 所需要的最少完全平方数数量
        #
        # 最坏情况下全部使用 1²，
        # 所以初始化为 n
        dp = [n] * (n + 1)

        # 组成 0 不需要任何完全平方数
        dp[0] = 0

        # 从小到大计算每一个 target
        for target in range(1, n + 1):

            s = 1

            # 只枚举不超过 target 的完全平方数
            while s * s <= target:
                square = s * s

                dp[target] = min(
                    dp[target],
                    1 + dp[target - square]
                )

                s += 1

        return dp[n]
```

# 48. 全排列（Permutations）

给定一个由**互不相同的整数**组成的数组 `nums`，返回它的所有可能排列。答案可以按任意顺序返回。

```text
输入：nums = [1, 2, 3]

输出：
[
    [1, 2, 3],
    [1, 3, 2],
    [2, 1, 3],
    [2, 3, 1],
    [3, 1, 2],
    [3, 2, 1]
]
```

约束：

- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有整数互不相同

**破局点：** 每层递归决定当前位置放哪个尚未使用的数字；完成一个排列后保存副本，再按“选择、递归、撤销选择”的模板继续搜索。

```python
from typing import List


class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        self.res = []

        # used[i] 表示 nums[i] 是否已经出现在当前排列中
        used = [False] * len(nums)

        self.backtrack(nums, [], used)

        return self.res

    def backtrack(
        self,
        nums: List[int],
        perm: List[int],
        used: List[bool]
    ) -> None:

        # 当前排列长度达到 n，说明找到一个完整排列
        if len(perm) == len(nums):
            # 必须保存副本，不能直接 append(perm)
            self.res.append(perm.copy())
            return

        for i in range(len(nums)):
            # 当前数字已经使用过，跳过
            if used[i]:
                continue

            # 1. 做选择
            perm.append(nums[i])
            used[i] = True

            # 2. 递归搜索下一层
            self.backtrack(nums, perm, used)

            # 3. 撤销选择（回溯）
            perm.pop()
            used[i] = False
```

# 49. 除自身以外数组的乘积（Products of Array Except Self）

给定一个整数数组 `nums`，返回数组 `output`，其中：

```text
output[i] = nums 中除 nums[i] 以外所有元素的乘积
```

题目保证每个结果都可以存储在 **32 位整数**中。

**进阶要求：** 能否在 **O(n)** 时间复杂度内，并且**不使用除法**解决这个问题？

```text
输入：nums = [1, 2, 4, 6]
输出：[48, 24, 12, 8]

输入：nums = [-1, 0, 1, 2, 3]
输出：[0, -6, 0, 0, 0]
```

约束条件：

- `2 <= nums.length <= 100000`
- `-30 <= nums[i] <= 30`
- 任意前缀或后缀的乘积都保证可以存储在 **32 位整数**中。

**破局点：** `answer[i] = 左侧乘积 × 右侧乘积`。第一遍把左侧乘积写入答案，第二遍用一个变量累积右侧乘积并乘入答案，即可不使用除法且将额外空间降为 `O(1)`（不计输出数组）。

```python
from typing import List


class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)

        # res 最开始用于保存“左侧所有元素的乘积”
        res = [1] * n

        # -------------------------
        # 第一遍：从左向右
        # -------------------------

        # prefix 表示当前位置左边所有元素的乘积
        prefix = 1

        for i in range(n):
            # 此时 prefix 还没有乘 nums[i]
            # 所以它恰好表示 nums[i] 左侧所有元素的乘积
            res[i] = prefix

            # 更新 prefix，供下一个位置使用
            prefix *= nums[i]

        # -------------------------
        # 第二遍：从右向左
        # -------------------------

        # postfix 表示当前位置右边所有元素的乘积
        postfix = 1

        for i in range(n - 1, -1, -1):
            # res[i] 当前保存左侧乘积
            # 再乘右侧乘积，就是最终答案
            res[i] *= postfix

            # 更新 postfix，供左边的位置使用
            postfix *= nums[i]

        return res
```

# 50. 删除链表的倒数第 N 个节点（Remove Nth Node From End of List）

给定一个单链表的头节点 `head` 和一个整数 `n`，删除链表中**倒数第 `n` 个节点**，并返回删除后的链表头节点。

```text
输入：head = [1, 2, 3, 4], n = 2
输出：[1, 2, 4]

输入：head = [5], n = 1
输出：[]

输入：head = [1, 2], n = 2
输出：[2]
```

约束：

- 链表节点总数记为 `sz`
- `1 <= sz <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= sz`

**破局点：** 先让快指针领先 `n` 个节点，再同步移动；快指针到达末尾时，慢指针恰好位于待删节点之前。虚拟头节点统一处理删除原头节点的情况。

```python
from typing import Optional


# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next


class Solution:
    def removeNthFromEnd(
        self,
        head: Optional[ListNode],
        n: int
    ) -> Optional[ListNode]:

        # 虚拟头节点，统一处理“删除原头节点”的情况
        dummy = ListNode(0, head)

        left = dummy
        right = head

        # 先让 right 向前移动 n 步
        for _ in range(n):
            right = right.next

        # 两个指针同步向前移动
        #
        # 当 right 到达 None 时，
        # left 正好停在待删除节点的前一个节点
        while right:
            left = left.next
            right = right.next

        # 删除目标节点
        left.next = left.next.next

        # dummy.next 始终代表删除后的真正头节点
        return dummy.next
```

# 51. Spiral Matrix（螺旋矩阵）

## 题干

给定一个 `m × n` 的整数矩阵 `matrix`，请按照**顺时针螺旋顺序（Spiral Order）**返回矩阵中的所有元素。

例如：

```text
matrix =
[
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
]
```

遍历顺序为：

```text
1 → 2 → 3
        ↓
4 → 5   6
↑       ↓
7 ← 8 ← 9
```

因此返回：

```python
[1, 2, 3, 6, 9, 8, 7, 4, 5]
```

## 破局点

维护 `top / bottom / left / right` 四个边界，每走完一条边就收缩对应边界；走完上边和右边后必须再次检查边界，避免单行或单列被重复访问。时间 `O(mn)`，除结果外空间 `O(1)`。

```python
from typing import List


class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        res = []

        left, right = 0, len(matrix[0])
        top, bottom = 0, len(matrix)

        while left < right and top < bottom:

            # 上：左 -> 右
            for col in range(left, right):
                res.append(matrix[top][col])
            top += 1

            # 右：上 -> 下
            for row in range(top, bottom):
                res.append(matrix[row][right - 1])
            right -= 1

            # 防止单行 / 单列情况下重复访问
            if left >= right or top >= bottom:
                break

            # 下：右 -> 左
            for col in range(right - 1, left - 1, -1):
                res.append(matrix[bottom - 1][col])
            bottom -= 1

            # 左：下 -> 上
            for row in range(bottom - 1, top - 1, -1):
                res.append(matrix[row][left])
            left += 1

        return res
```

# 52. 排序链表（Sort List）

## 题干

给你链表的头节点 `head`，请将其按**升序**排列，并返回排序后的链表。

例如：

```text
输入：

4 -> 2 -> 1 -> 3

输出：

1 -> 2 -> 3 -> 4
```

## 破局点

链表适合归并排序：快慢指针找中点并断链，递归排好左右两半，再原地合并。时间 `O(n log n)`，递归栈空间 `O(log n)`。

```python
class Solution:
    def sortList(self, head: Optional[ListNode]) -> Optional[ListNode]:

        # 0 / 1 个节点已经有序
        if head is None or head.next is None:
            return head

        # ---------- 1. 找中点 ----------
        slow = head
        fast = head.next

        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        # ---------- 2. 从中间断开 ----------
        right = slow.next
        slow.next = None

        left = head

        # ---------- 3. 分别排序 ----------
        left = self.sortList(left)
        right = self.sortList(right)

        # ---------- 4. 合并 ----------
        dummy = ListNode()
        tail = dummy

        while left and right:
            if left.val <= right.val:
                tail.next = left
                left = left.next
            else:
                tail.next = right
                right = right.next

            tail = tail.next

        # 接上剩余节点
        tail.next = left if left else right

        return dummy.next
```

# 53. Sort Color

## 题干

给定一个包含 `n` 个元素的数组 `nums`，其中每个元素都是一个整数，用来表示一种颜色：

- `0` 表示红色（Red）
- `1` 表示白色（White）
- `2` 表示蓝色（Blue）

请**原地（in-place）**对数组进行排序，使相同颜色的元素排列在一起，并按照以下顺序排列：

```
红色 0 → 白色 1 → 蓝色 2
```

例如：

```
输入：
nums = [2, 0, 2, 1, 1, 0]

排序后：
nums = [0, 0, 1, 1, 2, 2]
```

要求：

- 必须直接修改 `nums`
- 不需要返回结果
- **不能使用内置排序函数**来完成正式解法

## 破局点

用荷兰国旗三指针维护 `[0, l)` 为 `0`、`[l, i)` 为 `1`、`(r, n)` 为 `2`。与右端交换后，新换来的值尚未检查，所以当前指针不能前进。时间 `O(n)`，空间 `O(1)`。

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        l, r = 0, len(nums) - 1
        i = 0

        def swap(i, j):
            temp = nums[i]
            nums[i] = nums[j]
            nums[j] = temp

        while i <= r:
            if nums[i] == 0:
                swap(l, i)
                l += 1
            elif nums[i] == 2:
                swap(i, r)
                r -= 1
                i -= 1
            i += 1
```

# 54. Set Matrix Zeroes（矩阵置零）

## 题干

给定一个 `m x n` 的整数矩阵 `matrix`，如果某个元素为 `0`，则将该元素所在的**整行和整列**全部设置为 `0`。

要求：必须对原矩阵进行 **原地修改（in-place）**。

## 破局点

把第一行和第一列当作行列标记数组；`matrix[0][0]` 无法同时表达两种状态，因此额外用 `rowZero` 记录第一行。先处理内部区域，最后处理第一列和第一行。时间 `O(mn)`，空间 `O(1)`。

```python
from typing import List


class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        ROWS, COLS = len(matrix), len(matrix[0])

        # 单独记录第一行是否原本包含 0
        rowZero = False

        # 第一遍：
        # 使用第一行和第一列作为 marker
        for r in range(ROWS):
            for c in range(COLS):
                if matrix[r][c] == 0:

                    # 第一行记录：第 c 列需要置零
                    matrix[0][c] = 0

                    if r > 0:
                        # 第一列记录：第 r 行需要置零
                        matrix[r][0] = 0
                    else:
                        # 当前 0 在第一行中
                        rowZero = True

        # 第二遍：
        # 根据第一行和第一列的 marker
        # 更新内部区域
        #
        # 注意：不能从 0 开始，
        # 因为第一行和第一列现在还保存着 marker 信息
        for r in range(1, ROWS):
            for c in range(1, COLS):
                if matrix[r][0] == 0 or matrix[0][c] == 0:
                    matrix[r][c] = 0

        # matrix[0][0] 用来表示：
        # 第一列是否需要置零
        if matrix[0][0] == 0:
            for r in range(ROWS):
                matrix[r][0] = 0

        # rowZero 单独表示：
        # 第一行是否需要置零
        if rowZero:
            for c in range(COLS):
                matrix[0][c] = 0
```

# 55. 搜索旋转排序数组

## 题干

给定一个长度为 `n`、原本按升序排列的数组。该数组经过了 `1` 到 `n` 次旋转。

例如：

```text
原数组：[1, 2, 3, 4, 5, 6]

旋转 1 次：[6, 1, 2, 3, 4, 5]
旋转 3 次：[4, 5, 6, 1, 2, 3]
旋转 6 次：[1, 2, 3, 4, 5, 6]
```

给定旋转后的排序数组 `nums` 和整数 `target`：

- 如果 `target` 存在于 `nums` 中，返回它的下标。
- 如果不存在，返回 `-1`。

可以假设 `nums` 中的所有元素都互不相同。

使用 `O(n)` 时间完成搜索非常简单。请设计一个时间复杂度为 `O(log n)` 的算法。

### 示例

```text
输入：nums = [4, 5, 6, 7, 0, 1, 2], target = 0
输出：4
输入：nums = [4, 5, 6, 7, 0, 1, 2], target = 3
输出：-1
输入：nums = [1], target = 1
输出：0
```

## 破局点

二分后至少一半有序：先判断哪半有序，再判断 `target` 是否落在该有序区间，从而排除另一半。时间 `O(log n)`，空间 `O(1)`。

```python
from typing import List


class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1

        while left <= right:
            mid = left + (right - left) // 2

            # 找到目标值
            if nums[mid] == target:
                return mid

            # 情况一：左半部分 [left, mid] 有序
            if nums[left] <= nums[mid]:
                # target 位于左侧有序区间
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1

            # 情况二：右半部分 [mid, right] 有序
            else:
                # target 位于右侧有序区间
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1

        return -1
```

# 56. Search a 2D Matrix II（搜索二维矩阵 II）

## 题干

给定一个 `m × n` 的整数矩阵 `matrix` 和一个目标值 `target`，请编写一个高效算法判断 `target` 是否存在于矩阵中。

矩阵满足：

- 每一行从左到右**升序排列**
- 每一列从上到下**升序排列**

例如：

```text
matrix = [
    [1,  2,  4,  8],
    [10, 11, 12, 13],
    [14, 20, 30, 40]
]

target = 10
```

输出：

```text
True
```

## 破局点

从右上角开始：当前值太大就左移并排除一列，太小就下移并排除一行。时间 `O(m+n)`，空间 `O(1)`。

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        if not matrix or not matrix[0]:
            return False

        rows = len(matrix)
        cols = len(matrix[0])

        # 从右上角开始搜索
        row = 0
        col = cols - 1

        while row < rows and col >= 0:
            if matrix[row][col] == target:
                return True

            if matrix[row][col] > target:
                # 当前值太大，排除当前列
                col -= 1
            else:
                # 当前值太小，排除当前行
                row += 1

        return False
```

# 57. 搜索二维矩阵

## 题干

给定一个 `m × n` 的整数矩阵 `matrix` 和一个整数 `target`。矩阵满足：

- 每一行都按非递减顺序排列。
- 每一行的第一个元素都大于上一行的最后一个元素。

如果 `target` 存在于矩阵中，返回 `True`；否则返回 `False`。

要求设计时间复杂度为 `O(log(m × n))` 的算法。

## 破局点

矩阵按行展开后整体有序，无需真正展开；把一维下标 `mid` 映射为 `row = mid // cols`、`col = mid % cols` 后直接二分。时间 `O(log(mn))`，空间 `O(1)`。

```python
from typing import List


class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        if not matrix or not matrix[0]:
            return False

        rows, cols = len(matrix), len(matrix[0])

        # 将矩阵视为长度为 rows * cols 的一维有序数组
        left, right = 0, rows * cols - 1

        while left <= right:
            mid = left + (right - left) // 2

            # 将一维下标转换为二维坐标
            row = mid // cols
            col = mid % cols
            value = matrix[row][col]

            if value == target:
                return True
            elif value < target:
                # 目标值在右半部分
                left = mid + 1
            else:
                # 目标值在左半部分
                right = mid - 1

        return False
```

# 58. Rotting Fruit（腐烂的水果）

## 题干

给定一个二维矩阵 `grid`，其中每个单元格可能有以下三种取值：

- `0`：空单元格
- `1`：新鲜水果
- `2`：腐烂水果

每经过 **1 分钟**，如果一个新鲜水果在水平方向或垂直方向上与腐烂水果相邻，那么这个新鲜水果也会腐烂。

返回直到矩阵中 **不存在新鲜水果** 所需要经过的最少分钟数。

如果无论经过多少时间，都无法让所有新鲜水果腐烂，则返回 `-1`。

## 破局点

所有初始腐烂水果是同时扩散的起点，因此一次性入队做多源 BFS；每层代表一分钟，并用 `fresh` 判断是否存在无法到达的新鲜水果。时间、空间均为 `O(mn)`。

```python
from typing import List
from collections import deque


class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        # q 保存当前已经腐烂、并且之后可以继续向外传播的水果
        q = deque()

        # fresh 记录当前还剩多少新鲜水果
        fresh = 0

        # time 表示已经经过多少分钟
        time = 0

        rows = len(grid)
        cols = len(grid[0])

        # 第一次遍历：
        # 1. 统计新鲜水果数量
        # 2. 将所有初始腐烂水果加入队列
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == 1:
                    fresh += 1
                elif grid[r][c] == 2:
                    q.append((r, c))

        # 上、下、左、右四个方向
        directions = [
            (0, 1),   # 右
            (0, -1),  # 左
            (1, 0),   # 下
            (-1, 0),  # 上
        ]

        # 只要还有新鲜水果，并且当前还有腐烂水果可以继续传播
        while fresh > 0 and q:
            # 当前队列中的所有节点属于同一层 BFS，
            # 即它们会在同一分钟向周围传播
            level_size = len(q)

            for _ in range(level_size):
                r, c = q.popleft()

                for dr, dc in directions:
                    nr = r + dr
                    nc = c + dc

                    # 检查是否越界，并判断邻居是不是新鲜水果
                    if (
                        0 <= nr < rows
                        and 0 <= nc < cols
                        and grid[nr][nc] == 1
                    ):
                        # 新鲜水果被感染
                        grid[nr][nc] = 2
                        fresh -= 1

                        # 下一分钟，它还可以继续感染其他水果
                        q.append((nr, nc))

            # 完成一整层 BFS，表示过去了一分钟
            time += 1

        # 如果仍然存在新鲜水果，说明这些水果无法被感染
        return time if fresh == 0 else -1
```

# 59. Rotate Image（旋转图像）

## 题干

给定一个 `n × n` 的整数矩阵 `matrix`，将其**顺时针旋转 90°**。

要求必须进行**原地修改（in-place）**，不能额外创建另一个二维矩阵来完成最终解法。

例如：

```text
原矩阵：

1 2 3
4 5 6
7 8 9

顺时针旋转 90° 后：

7 4 1
8 5 2
9 6 3
```

## 破局点

顺时针旋转 90° 等价于“上下翻转 + 沿主对角线转置”；转置只遍历对角线一侧，避免交换两次。时间 `O(n²)`，空间 `O(1)`。

```python
from typing import List


class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        # 第一步：上下翻转
        # 第一行和最后一行交换，第二行和倒数第二行交换……
        matrix.reverse()

        # 第二步：沿主对角线进行转置
        n = len(matrix)

        for i in range(n):
            # 只遍历主对角线右上方的元素
            # 防止同一对元素被交换两次
            for j in range(i + 1, n):
                matrix[i][j], matrix[j][i] = (
                    matrix[j][i],
                    matrix[i][j],
                )
```

# 60. Subarray Sum Equals K（和为 K 的子数组）

给定一个整数数组 `nums` 和一个整数 `k`，返回数组中 **元素和等于 `k` 的连续非空子数组数量**。

> **Subarray（子数组）** 指数组中一段连续且非空的元素序列。

## 示例 1

```text
输入：
nums = [2, -1, 1, 2]
k = 2

输出：
4
```

满足条件的子数组为 `[2]`、`[2, -1, 1]`、`[-1, 1, 2]`、`[2]`。

## 示例 2

```text
输入：
nums = [4, 4, 4, 4, 4, 4]
k = 4

输出：
6
```

## 约束

- `1 <= nums.length <= 20,000`
- `-1,000 <= nums[i] <= 1,000`
- `-10,000,000 <= k <= 10,000,000`

## 破局点

若当前前缀和为 `cur_sum`，只需统计之前出现过多少次 `cur_sum - k`。哈希表保存前缀和的出现次数；初始化 `{0: 1}`，并且必须先查询、后记录当前前缀和。

```python
from typing import List


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_count = {0: 1}

        cur_sum = 0
        res = 0

        for num in nums:
            cur_sum += num

            # old_prefix = cur_sum - k
            res += prefix_count.get(cur_sum - k, 0)

            # 当前前缀和供未来位置使用
            prefix_count[cur_sum] = prefix_count.get(cur_sum, 0) + 1

        return res
```

# 61. 子集（Subsets）

给定一个由**互不相同的整数**组成的数组 `nums`，返回该数组所有可能的子集（即幂集）。

结果中不能包含重复的子集，可以按**任意顺序**返回。

```text
输入：nums = [1, 2, 3]

输出：
[
    [],
    [1],
    [2],
    [3],
    [1, 2],
    [1, 3],
    [2, 3],
    [1, 2, 3]
]
```

如果数组长度为 `n`，那么它一共有 \(2^n\) 个不同的子集。

## 破局点

对每个元素只有“选”与“不选”两条分支。递归到底时保存当前子集的副本；探索完“选”的分支后用 `pop()` 撤销选择，再探索“不选”的分支。

```python
from typing import List


class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        # 保存最终生成的所有子集
        res: List[List[int]] = []

        # 当前正在构造的子集
        subset: List[int] = []

        def dfs(i: int) -> None:
            # 所有元素都已经完成选择
            if i == len(nums):
                # 必须保存副本，不能直接保存 subset 本身
                res.append(subset.copy())
                return

            # 选择一：将 nums[i] 加入当前子集
            subset.append(nums[i])
            dfs(i + 1)

            # 回溯：撤销刚才的选择
            subset.pop()

            # 选择二：不将 nums[i] 加入当前子集
            dfs(i + 1)

        dfs(0)
        return res
```

# 62. Swap Nodes in Pairs（两两交换链表中的节点）

给定一个链表的头节点 `head`，请将链表中**每两个相邻节点进行交换**，并返回交换后的链表头节点。

要求：

- **不能修改节点内部的值**
- 必须真正改变节点之间的 `next` 指针关系
- 如果链表节点数为奇数，最后一个节点保持不变

```text
输入：1 → 2 → 3 → 4
输出：2 → 1 → 4 → 3

输入：1 → 2 → 3 → 4 → 5
输出：2 → 1 → 4 → 3 → 5
```

## 破局点

用虚拟头节点统一处理第一组。每次把 `prev → first → second → next` 改成 `prev → second → first → next`，完成三次指针重连后，将 `prev` 移到 `first`。

```python
from typing import Optional


class Solution:
    def swapPairs(
        self,
        head: Optional[ListNode]
    ) -> Optional[ListNode]:

        # 虚拟头节点，方便统一处理第一组节点
        dummy = ListNode(0, head)

        # prev 指向当前待交换节点对的前一个节点
        prev = dummy

        # 至少存在两个节点时才能交换
        while prev.next and prev.next.next:
            first = prev.next
            second = first.next

            # prev -> first -> second -> next
            #
            # 变为：
            #
            # prev -> second -> first -> next

            first.next = second.next
            second.next = first
            prev.next = second

            # first 现在是当前交换组的最后一个节点，
            # 下一轮让它作为 prev
            prev = first

        return dummy.next
```

# 63. 三数之和（3Sum）

给定一个整数数组 `nums`，请返回所有满足 `nums[i] + nums[j] + nums[k] == 0` 的三元组 `[nums[i], nums[j], nums[k]]`，其中下标 `i`、`j`、`k` 两两不同。

最终结果中**不能包含重复的三元组**。返回的三元组顺序以及结果列表中的顺序均不限。

## 示例 1

```text
输入：nums = [-1, 0, 1, 2, -1, -4]
输出：[[-1, -1, 2], [-1, 0, 1]]
```

## 示例 2

```text
输入：nums = [0, 1, 1]
输出：[]
```

## 示例 3

```text
输入：nums = [0, 0, 0]
输出：[[0, 0, 0]]
```

## 约束

- `3 <= nums.length <= 3000`
- `-10^5 <= nums[i] <= 10^5`

## 破局点

排序后固定第一个数，把剩余部分转成双指针 Two Sum：和小就左移，和大就右移。去重有两处：跳过重复的第一个数；找到答案后跳过左右两侧的重复值。

```python
from typing import List


class Solution:
    def threeSum(self, nums: List[int]) -> List[List[int]]:
        res = []

        # 双指针依赖有序数组
        nums.sort()

        for i, a in enumerate(nums):
            # 如果第一个数字已经大于 0，
            # 后面的数字都 >= a > 0，
            # 三个数不可能再加出 0
            if a > 0:
                break

            # 跳过重复的第一个数字，
            # 防止生成相同三元组
            if i > 0 and a == nums[i - 1]:
                continue

            # 在 i 的右侧寻找另外两个数字
            l = i + 1
            r = len(nums) - 1

            while l < r:
                threeSum = a + nums[l] + nums[r]

                if threeSum < 0:
                    # 当前和太小，需要增大
                    l += 1

                elif threeSum > 0:
                    # 当前和太大，需要减小
                    r -= 1

                else:
                    # 找到合法三元组
                    res.append([a, nums[l], nums[r]])

                    # 两边同时向中间移动
                    l += 1
                    r -= 1

                    # 跳过重复的左侧数字
                    #
                    # 条件顺序写成 l < r 在前，
                    # 可以先确认指针仍然有效，再访问 nums[l]
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1

                    # 也可以显式跳过右侧重复数字。
                    # 不是绝对必须，但逻辑更对称、也更易读。
                    while l < r and nums[r] == nums[r + 1]:
                        r -= 1

        return res
```

# 64. Top K Frequent Elements（前 K 个高频元素）

给定一个整数数组 `nums` 和一个整数 `k`，返回数组中出现频率最高的 `k` 个元素。

测试数据保证答案总是唯一的，返回结果可以是任意顺序。

## 示例 1

```text
输入：nums = [1,2,2,3,3,3], k = 2
输出：[2,3]
```

## 示例 2

```text
输入：nums = [7,7], k = 1
输出：[7]
```

## 约束

- `1 <= nums.length <= 10^4`
- `-1000 <= nums[i] <= 1000`
- `1 <= k <= nums` 中不同元素的数量

## 破局点

元素的最高频率不会超过 `n`，因此可让桶下标直接表示频率：`freq[i]` 存放所有恰好出现 `i` 次的元素。倒序扫描桶，取满 `k` 个即返回。

```python
from typing import List


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        # Step 1:
        # 使用 Hash Map 统计每个数字出现的次数
        count = {}

        for num in nums:
            count[num] = 1 + count.get(num, 0)

        # Step 2:
        # freq[i] 表示：
        # 所有出现次数恰好为 i 的数字
        #
        # 最大频率不可能超过 len(nums)
        # 因此需要 len(nums) + 1 个位置
        freq = [[] for _ in range(len(nums) + 1)]

        # 把数字放入对应频率的 bucket
        for num, cnt in count.items():
            freq[cnt].append(num)

        # Step 3:
        # 从频率最高的位置开始向前找
        res = []

        for i in range(len(freq) - 1, 0, -1):
            # freq[i] 中可能有多个数字
            for num in freq[i]:
                res.append(num)

                # 一旦找到 k 个，就直接返回
                if len(res) == k:
                    return res
```

# 65. Unique Paths（不同路径）

有一个 **`m x n`** 的网格。任意时刻，你只能向 **右** 或向 **下** 移动。

给定两个整数 **`m`** 和 **`n`**，返回从网格左上角 **`grid[0][0]`** 到右下角 **`grid[m - 1][n - 1]`** 一共有多少条不同的路径。

可以假设最终答案一定能够存储在 **32 位整数** 中。

## 破局点

每格的路径数等于“右边 + 下边”。用一维数组从右向左更新时，更新前的 `dp[j]` 代表下方，已更新的 `dp[j + 1]` 代表右方，因此状态转移为 `dp[j] += dp[j + 1]`。

```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        # 最后一行全部为 1
        dp = [1] * n

        # 从倒数第二行向上计算
        for _ in range(m - 2, -1, -1):

            # 必须从右向左更新
            for j in range(n - 2, -1, -1):
                # dp[j]：更新前代表下方
                # dp[j + 1]：已经更新，代表右方
                dp[j] += dp[j + 1]

        return dp[0]
```

# 66. 验证二叉搜索树（Valid Binary Search Tree）

给定一棵二叉树的根节点 `root`，判断它是否是一棵**有效的二叉搜索树（Binary Search Tree, BST）**。如果是，返回 `True`；否则返回 `False`。

一棵有效的二叉搜索树需要满足：

- 对于任意节点，其**左子树中的所有节点值**都必须**严格小于**当前节点值。
- 对于任意节点，其**右子树中的所有节点值**都必须**严格大于**当前节点值。
- 左右子树本身也必须是二叉搜索树。

> **关键点：BST 的限制针对的是整棵子树，而不仅仅是直接的左右子节点。**

例如，下面的树不是有效 BST，因为 `15` 位于 `10` 的左子树中，却大于 `10`：

```text
    10
   /
  5
   \
   15
```

## 破局点

每个节点都有一个由所有祖先共同决定的合法开区间 `(lower, upper)`。进入左子树时收紧上界，进入右子树时收紧下界；这能检查整棵子树，而非只比较父子节点。

```python
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

# 67. 单词拆分（Word Break）

给定字符串 `s` 和字符串字典 `wordDict`。如果可以使用字典中的单词，将 `s` 拆分成一个由若干单词组成的序列，则返回 `True`；否则返回 `False`。

字典中的单词可以重复使用任意次数，并且可以假设 `wordDict` 中的单词互不相同。

```text
s = "leetcode"
wordDict = ["leet", "code"]

输出：True
解释："leetcode" 可以拆分为 "leet" + "code"
```

## 破局点

定义 `dp[i]` 表示前 `i` 个字符能否拆分。若 `dp[start]` 为真且 `s[start:end]` 在字典中，则 `dp[end]` 为真；哈希集合加速成员查询，最长单词长度限制枚举范围。

```python
from typing import List


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        word_set = set(wordDict)
        max_word_len = max(map(len, wordDict), default=0)
        n = len(s)

        # dp[i] 表示前 i 个字符 s[:i] 是否可以被拆分
        dp = [False] * (n + 1)
        dp[0] = True

        for end in range(1, n + 1):
            # 单词长度不需要超过字典中的最长单词
            start_limit = max(0, end - max_word_len)

            for start in range(end - 1, start_limit - 1, -1):
                if dp[start] and s[start:end] in word_set:
                    dp[end] = True
                    break

        return dp[n]
```

# 68. 单词搜索（Word Search）

给定一个二维字符网格 `board` 和一个字符串 `word`。如果能够在网格中找到该单词，返回 `True`；否则返回 `False`。

单词必须由网格中水平或垂直相邻的单元格依次组成，即每一步只能向上、下、左、右移动。在同一条搜索路径中，每个单元格最多只能使用一次。

## 破局点

从每个可能的首字符出发做网格 DFS。进入格子后原地标记为已使用，搜索四个方向，最后恢复原字符；这正是“选择—搜索—撤销”的回溯模板。

```python
from typing import List


class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        rows, cols = len(board), len(board[0])

        def dfs(r: int, c: int, i: int) -> bool:
            # 已经匹配完所有字符
            if i == len(word):
                return True

            # 越界或者当前字符不匹配
            # 被标记为 "#" 的位置也会在字符比较时匹配失败
            if (
                r < 0
                or c < 0
                or r >= rows
                or c >= cols
                or board[r][c] != word[i]
            ):
                return False

            # 保存原字符，并将当前位置标记为已使用
            original_char = board[r][c]
            board[r][c] = "#"

            # 搜索四个相邻方向
            found = (
                dfs(r + 1, c, i + 1)
                or dfs(r - 1, c, i + 1)
                or dfs(r, c + 1, i + 1)
                or dfs(r, c - 1, i + 1)
            )

            # 回溯：恢复原字符
            board[r][c] = original_char

            return found

        for r in range(rows):
            for c in range(cols):
                # 只有首字符匹配时才需要启动 DFS
                if board[r][c] == word[0] and dfs(r, c, 0):
                    return True

        return False
```
