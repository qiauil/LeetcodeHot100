# Easy 面试速查表

> 每题只保留面试中最值得掌握的解法。代码默认使用 LeetCode 提供的 `ListNode`、`TreeNode` 等定义。

# 1. 两数之和（Two Sum）

给定整数数组 `nums` 和目标值 `target`，返回和为目标值的两个元素下标。

## 破局点

遍历到 `num` 时，只需询问补数 `target - num` 是否已经出现；哈希表保存“值 → 下标”，先查询再写入可避免重复使用当前元素。

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        seen = {}

        for i, num in enumerate(nums):
            complement = target - num
            if complement in seen:
                return [seen[complement], i]
            seen[num] = i

        return []
```

复杂度：时间 `O(n)`，空间 `O(n)`。

# 2. 买卖股票的最佳时机（Best Time to Buy and Sell Stock）

只能完成一次买入和一次卖出，且买入必须早于卖出，求最大利润。

## 破局点

把当天视为卖出日，最优买入价就是此前见过的最低价格；一边更新最大利润，一边维护历史最低价。

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = prices[0]
        max_profit = 0

        for price in prices:
            max_profit = max(max_profit, price - min_price)
            min_price = min(min_price, price)

        return max_profit
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 3. 有效的括号（Valid Parentheses）

判断只含 `()[]{}` 的字符串能否正确闭合且顺序合法。

## 破局点

右括号只能和最近尚未匹配的左括号配对，天然符合栈的后进先出；遇到右括号时检查栈顶，最后栈必须为空。

```python
class Solution:
    def isValid(self, s: str) -> bool:
        pairs = {')': '(', ']': '[', '}': '{'}
        stack = []

        for char in s:
            if char not in pairs:
                stack.append(char)
            elif not stack or stack.pop() != pairs[char]:
                return False

        return not stack
```

复杂度：时间 `O(n)`，空间 `O(n)`。

# 4. 合并两个有序链表（Merge Two Sorted Lists）

将两个升序链表合并为一个升序链表。

## 破局点

用哑节点统一处理头节点，每次摘下两条链表中较小的节点接到结果尾部；一条耗尽后直接接上另一条。

```python
class Solution:
    def mergeTwoLists(
        self,
        list1: Optional[ListNode],
        list2: Optional[ListNode]
    ) -> Optional[ListNode]:
        dummy = ListNode()
        tail = dummy

        while list1 and list2:
            if list1.val <= list2.val:
                tail.next, list1 = list1, list1.next
            else:
                tail.next, list2 = list2, list2.next
            tail = tail.next

        tail.next = list1 or list2
        return dummy.next
```

复杂度：时间 `O(m + n)`，空间 `O(1)`。

# 5. 搜索插入位置（Search Insert Position）

在升序数组中查找目标值；不存在时返回其保持有序的插入位置。

## 破局点

答案是第一个大于等于 `target` 的位置。用左闭右开区间 `[left, right)` 做二分，循环结束时 `left` 就是答案，也自然允许返回 `len(nums)`。

```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums)

        while left < right:
            mid = left + (right - left) // 2
            if nums[mid] < target:
                left = mid + 1
            else:
                right = mid

        return left
```

复杂度：时间 `O(log n)`，空间 `O(1)`。

# 6. 爬楼梯（Climbing Stairs）

每次可爬 1 或 2 级，求爬到第 `n` 级的不同方法数。

## 破局点

到第 `i` 级的最后一步只可能来自 `i - 1` 或 `i - 2`，所以 `dp[i] = dp[i-1] + dp[i-2]`；只保留前两个状态即可。

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n <= 2:
            return n

        prev2, prev1 = 1, 2
        for _ in range(3, n + 1):
            prev2, prev1 = prev1, prev1 + prev2

        return prev1
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 7. 二叉树的中序遍历（Binary Tree Inorder Traversal）

按“左子树—根节点—右子树”的顺序返回二叉树节点值。

## 破局点

显式栈模拟递归：一路压入左链；无左可走时弹栈访问，再转向右子树。相比 Morris 遍历更易在面试中稳定写对。

```python
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        result = []
        stack = []
        current = root

        while current or stack:
            while current:
                stack.append(current)
                current = current.left

            current = stack.pop()
            result.append(current.val)
            current = current.right

        return result
```

复杂度：时间 `O(n)`，空间 `O(h)`，`h` 为树高。

# 8. 对称二叉树（Symmetric Tree）

判断一棵二叉树是否关于根节点轴对称。

## 破局点

对称比较不是比较两棵同方向子树，而是镜像配对：`left.left` 对 `right.right`，`left.right` 对 `right.left`。

```python
class Solution:
    def isSymmetric(self, root: Optional[TreeNode]) -> bool:
        def mirror(left, right):
            if not left or not right:
                return left is right
            return (
                left.val == right.val
                and mirror(left.left, right.right)
                and mirror(left.right, right.left)
            )

        return mirror(root.left, root.right) if root else True
```

复杂度：时间 `O(n)`，空间 `O(h)`。

# 9. 二叉树的最大深度（Maximum Depth of Binary Tree）

返回二叉树从根到最远叶子节点的节点数。

## 破局点

空树深度为 0；非空树深度等于左右子树最大深度加 1。这正是后序 DFS 的定义式。

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        return 1 + max(self.maxDepth(root.left), self.maxDepth(root.right))
```

复杂度：时间 `O(n)`，空间 `O(h)`。

# 10. 将有序数组转换为二叉搜索树（Convert Sorted Array to BST）

将严格升序数组转换为一棵高度平衡的二叉搜索树。

## 破局点

中点作为根能让左右节点数尽量平衡；递归用中点左侧建左子树、中点右侧建右子树。

```python
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        def build(left, right):
            if left > right:
                return None

            mid = (left + right) // 2
            root = TreeNode(nums[mid])
            root.left = build(left, mid - 1)
            root.right = build(mid + 1, right)
            return root

        return build(0, len(nums) - 1)
```

复杂度：时间 `O(n)`，空间 `O(log n)`（递归栈，不计输出树）。

# 11. 杨辉三角（Pascal's Triangle）

给定行数 `numRows`，返回杨辉三角的前 `numRows` 行。

## 破局点

每行两端恒为 1，中间位置由上一行相邻两数相加得到；直接逐行构造最直观，也最不易出现组合数除法问题。

```python
class Solution:
    def generate(self, numRows: int) -> List[List[int]]:
        triangle = []

        for row_index in range(numRows):
            row = [1] * (row_index + 1)
            for j in range(1, row_index):
                row[j] = triangle[-1][j - 1] + triangle[-1][j]
            triangle.append(row)

        return triangle
```

复杂度：时间 `O(numRows²)`；除输出外额外空间 `O(1)`。

# 12. 只出现一次的数字（Single Number）

数组中除一个元素只出现一次外，其余元素都恰好出现两次，找出单独的元素。

## 破局点

异或满足 `x ^ x = 0`、`x ^ 0 = x`，且交换、结合律成立；把全部元素异或，成对元素自动抵消。

```python
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        result = 0
        for num in nums:
            result ^= num
        return result
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 13. 环形链表（Linked List Cycle）

判断链表中是否存在某个节点可通过不断沿 `next` 再次到达。

## 破局点

慢指针每次一步、快指针每次两步；有环时快指针必在环内追上慢指针，无环时快指针先到链尾。

```python
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        slow = fast = head

        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                return True

        return False
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 14. 相交链表（Intersection of Two Linked Lists）

给定两个无环单链表，返回它们相交的起始节点；不相交则返回 `None`。

## 破局点

两个指针分别走 `A + B` 和 `B + A`，换道后总路程相同，长度差被自动抵消；相交时在交点相遇，否则同时到达 `None`。

```python
class Solution:
    def getIntersectionNode(
        self,
        headA: ListNode,
        headB: ListNode
    ) -> Optional[ListNode]:
        p1, p2 = headA, headB

        while p1 is not p2:
            p1 = p1.next if p1 else headB
            p2 = p2.next if p2 else headA

        return p1
```

复杂度：时间 `O(m + n)`，空间 `O(1)`。

# 15. 多数元素（Majority Element）

找出数组中出现次数超过 `⌊n / 2⌋` 的元素；题目保证它存在。

## 破局点

Boyer–Moore 投票把多数元素和其他元素两两抵消；多数元素数量占优，最终候选者一定是答案。

```python
class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        candidate = None
        count = 0

        for num in nums:
            if count == 0:
                candidate = num
            count += 1 if num == candidate else -1

        return candidate
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 16. 反转链表（Reverse Linked List）

反转一个单链表并返回新的头节点。

## 破局点

每次改写 `current.next` 前必须先保存后继节点，否则会丢失未处理部分；`previous` 始终代表已经反转好的链表头。

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        previous = None
        current = head

        while current:
            next_node = current.next
            current.next = previous
            previous = current
            current = next_node

        return previous
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 17. 翻转二叉树（Invert Binary Tree）

交换二叉树中每个节点的左右子树，返回根节点。

## 破局点

局部动作只有“交换左右孩子”；再递归处理交换后的两棵子树即可。每个节点恰好处理一次。

```python
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root:
            return None

        root.left, root.right = root.right, root.left
        self.invertTree(root.left)
        self.invertTree(root.right)
        return root
```

复杂度：时间 `O(n)`，空间 `O(h)`。

# 18. 回文链表（Palindrome Linked List）

判断单链表的节点值序列是否为回文。

## 破局点

快慢指针找中点，原地反转后半段，再从两端逐项比较；若希望无副作用，比较后把后半段再次反转恢复。

```python
class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        previous = None
        while slow:
            next_node = slow.next
            slow.next = previous
            previous = slow
            slow = next_node

        left, right = head, previous
        while right:
            if left.val != right.val:
                return False
            left = left.next
            right = right.next

        return True
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 19. 移动零（Move Zeroes）

在保持非零元素相对顺序的前提下，将数组中的所有零移到末尾，且必须原地操作。

## 破局点

`write` 指向下一个非零元素应放的位置；扫描到非零值就与 `write` 位置交换。交换写法能同时清理当前位置，又保持稳定顺序。

```python
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        write = 0

        for read in range(len(nums)):
            if nums[read] != 0:
                nums[write], nums[read] = nums[read], nums[write]
                write += 1
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 20. 二叉树的直径（Diameter of Binary Tree）

返回二叉树中任意两节点之间最长路径的边数，该路径不一定经过根节点。

## 破局点

DFS 向父节点返回“单边最大高度”，同时用 `左高 + 右高` 更新经过当前节点的直径。不要把节点数与边数混淆。

```python
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        diameter = 0

        def height(node):
            nonlocal diameter
            if not node:
                return 0

            left_height = height(node.left)
            right_height = height(node.right)
            diameter = max(diameter, left_height + right_height)
            return 1 + max(left_height, right_height)

        height(root)
        return diameter
```

复杂度：时间 `O(n)`，空间 `O(h)`。
