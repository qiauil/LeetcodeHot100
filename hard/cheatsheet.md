# Hard 面试速查表

> 每题只保留面试中最值得掌握的解法。代码默认使用 LeetCode 提供的 `ListNode`、`TreeNode` 等定义。

# 1. 寻找两个正序数组的中位数（Median of Two Sorted Arrays）

给定两个正序数组，要求在 `O(log(m + n))` 时间内求合并后的中位数。

## 破局点

在较短数组上二分切分线，使左右两部分元素数量满足中位数要求；当 `A左最大 <= B右最小` 且 `B左最大 <= A右最小` 时，切分合法。用正负无穷统一处理边界。

```python
class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1

        m, n = len(nums1), len(nums2)
        left, right = 0, m

        while left <= right:
            cut1 = (left + right) // 2
            cut2 = (m + n + 1) // 2 - cut1

            left1 = nums1[cut1 - 1] if cut1 else float('-inf')
            right1 = nums1[cut1] if cut1 < m else float('inf')
            left2 = nums2[cut2 - 1] if cut2 else float('-inf')
            right2 = nums2[cut2] if cut2 < n else float('inf')

            if left1 <= right2 and left2 <= right1:
                if (m + n) % 2:
                    return float(max(left1, left2))
                return (max(left1, left2) + min(right1, right2)) / 2
            if left1 > right2:
                right = cut1 - 1
            else:
                left = cut1 + 1

        raise ValueError('Input arrays must be sorted')
```

复杂度：时间 `O(log(min(m, n)))`，空间 `O(1)`。

# 2. 接雨水（Trapping Rain Water）

给定柱状图高度，计算下雨后能够接住的雨水总量。

## 破局点

某位置水位由左右最高柱的较小值决定。双指针中，较矮一侧的水量已经能被该侧最大值确定，因此处理较矮侧并向内移动。

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        left, right = 0, len(height) - 1
        left_max = right_max = 0
        water = 0

        while left < right:
            if height[left] <= height[right]:
                left_max = max(left_max, height[left])
                water += left_max - height[left]
                left += 1
            else:
                right_max = max(right_max, height[right])
                water += right_max - height[right]
                right -= 1

        return water
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 3. N 皇后（N-Queens）

在 `n × n` 棋盘上放置 `n` 个皇后，使任意两个皇后不同行、不同列、不同对角线，返回所有布局。

## 破局点

按行放置天然消除同行冲突；用集合记录列、主对角线 `row - col`、副对角线 `row + col`。每次选择后递归，返回时撤销选择。

```python
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        result = []
        board = [['.'] * n for _ in range(n)]
        columns = set()
        diagonals = set()
        anti_diagonals = set()

        def backtrack(row):
            if row == n:
                result.append([''.join(line) for line in board])
                return

            for col in range(n):
                diagonal = row - col
                anti_diagonal = row + col
                if (
                    col in columns
                    or diagonal in diagonals
                    or anti_diagonal in anti_diagonals
                ):
                    continue

                board[row][col] = 'Q'
                columns.add(col)
                diagonals.add(diagonal)
                anti_diagonals.add(anti_diagonal)

                backtrack(row + 1)

                board[row][col] = '.'
                columns.remove(col)
                diagonals.remove(diagonal)
                anti_diagonals.remove(anti_diagonal)

        backtrack(0)
        return result
```

复杂度：时间约 `O(n!)`，辅助空间 `O(n)`（不计棋盘与答案）。

# 4. 最小覆盖子串（Minimum Window Substring）

在字符串 `s` 中找到包含字符串 `t` 全部字符（含重复次数）的最短子串。

## 破局点

右指针扩张直到窗口满足需求，左指针再尽量收缩。不要反复比较整个频次表；用 `formed` 记录已有多少种字符达到了所需数量。

```python
from collections import Counter


class Solution:
    def minWindow(self, s: str, t: str) -> str:
        if not t or len(t) > len(s):
            return ''

        need = Counter(t)
        window = {}
        required = len(need)
        formed = 0
        left = 0
        best_length = float('inf')
        best_start = 0

        for right, char in enumerate(s):
            window[char] = window.get(char, 0) + 1
            if char in need and window[char] == need[char]:
                formed += 1

            while formed == required:
                if right - left + 1 < best_length:
                    best_length = right - left + 1
                    best_start = left

                left_char = s[left]
                window[left_char] -= 1
                if left_char in need and window[left_char] < need[left_char]:
                    formed -= 1
                left += 1

        if best_length == float('inf'):
            return ''
        return s[best_start:best_start + best_length]
```

复杂度：时间 `O(|s| + |t|)`，空间 `O(|Σ|)`。

# 5. K 个一组翻转链表（Reverse Nodes in k-Group）

每 `k` 个节点为一组翻转链表，不足 `k` 个的末尾节点保持原顺序。

## 破局点

每轮先找到本组第 `k` 个节点，找不到就结束；保存下一组起点后，把 `group_next` 作为反转的初始前驱，可让本组尾部在反转时直接接好后续链表。

```python
class Solution:
    def reverseKGroup(
        self,
        head: Optional[ListNode],
        k: int
    ) -> Optional[ListNode]:
        dummy = ListNode(0, head)
        group_prev = dummy

        def get_kth(node, steps):
            while node and steps:
                node = node.next
                steps -= 1
            return node

        while True:
            kth = get_kth(group_prev, k)
            if not kth:
                break

            group_next = kth.next
            previous = group_next
            current = group_prev.next

            while current is not group_next:
                next_node = current.next
                current.next = previous
                previous = current
                current = next_node

            old_group_start = group_prev.next
            group_prev.next = kth
            group_prev = old_group_start

        return dummy.next
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 6. 最长有效括号（Longest Valid Parentheses）

给定只含 `(` 和 `)` 的字符串，返回最长格式正确且连续的括号子串长度。

## 破局点

栈保存尚未匹配的位置，并以 `-1` 作为上一段无效边界。遇到右括号先弹栈；栈空说明当前右括号成为新边界，否则当前有效长度是 `i - stack[-1]`。

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        stack = [-1]
        longest = 0

        for i, char in enumerate(s):
            if char == '(':
                stack.append(i)
            else:
                stack.pop()
                if not stack:
                    stack.append(i)
                else:
                    longest = max(longest, i - stack[-1])

        return longest
```

复杂度：时间 `O(n)`，空间 `O(n)`。

# 7. 合并 K 个升序链表（Merge k Sorted Lists）

合并 `k` 个升序链表并返回一条升序链表。

## 破局点

最小堆始终保存每条链表当前最小的头节点；弹出全局最小节点后，只把它的后继加入堆。堆元组加入唯一序号，避免值相同时 Python 比较 `ListNode`。

```python
import heapq
from itertools import count


class Solution:
    def mergeKLists(
        self,
        lists: List[Optional[ListNode]]
    ) -> Optional[ListNode]:
        heap = []
        unique = count()

        for node in lists:
            if node:
                heapq.heappush(heap, (node.val, next(unique), node))

        dummy = ListNode()
        tail = dummy

        while heap:
            _, _, node = heapq.heappop(heap)
            tail.next = node
            tail = tail.next
            if node.next:
                heapq.heappush(
                    heap,
                    (node.next.val, next(unique), node.next)
                )

        return dummy.next
```

复杂度：设总节点数为 `N`，时间 `O(N log k)`，空间 `O(k)`。

# 8. 柱状图中最大的矩形（Largest Rectangle in Histogram）

给定柱状图高度，求其中可形成的最大矩形面积。

## 破局点

单调递增栈保存“某高度最早可延伸到的下标”。遇到更矮柱时，弹出的高度已确定右边界；当前柱可继承被弹元素的最早起点。扫描结束后再结算仍在栈中的柱。

```python
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        stack = []  # (start_index, height)
        largest = 0

        for i, height in enumerate(heights):
            start = i
            while stack and stack[-1][1] > height:
                index, previous_height = stack.pop()
                largest = max(largest, previous_height * (i - index))
                start = index
            stack.append((start, height))

        n = len(heights)
        for index, height in stack:
            largest = max(largest, height * (n - index))

        return largest
```

复杂度：时间 `O(n)`，空间 `O(n)`。

# 9. 数据流的中位数（Find Median from Data Stream）

设计一个结构，支持不断加入整数并随时返回当前所有数字的中位数。

## 破局点

用最大堆维护较小的一半、最小堆维护较大的一半；保证两堆大小差不超过 1，且小半边所有值不大于大半边。Python 用负数模拟最大堆。

```python
import heapq


class MedianFinder:
    def __init__(self):
        self.small = []  # 最大堆，实际保存负数
        self.large = []  # 最小堆

    def addNum(self, num: int) -> None:
        heapq.heappush(self.small, -num)
        heapq.heappush(self.large, -heapq.heappop(self.small))

        if len(self.large) > len(self.small):
            heapq.heappush(self.small, -heapq.heappop(self.large))

    def findMedian(self) -> float:
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2
```

复杂度：`addNum` 时间 `O(log n)`，`findMedian` 时间 `O(1)`，空间 `O(n)`。

# 10. 滑动窗口最大值（Sliding Window Maximum）

长度为 `k` 的窗口从数组左侧滑到右侧，返回每个窗口中的最大值。

## 破局点

单调递减队列保存下标：队首永远是窗口最大值；新元素入队前删掉队尾所有不大于它的元素，并及时移除已经离开窗口的队首。

```python
from collections import deque


class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        queue = deque()
        result = []

        for right, num in enumerate(nums):
            while queue and queue[0] <= right - k:
                queue.popleft()

            while queue and nums[queue[-1]] <= num:
                queue.pop()
            queue.append(right)

            if right >= k - 1:
                result.append(nums[queue[0]])

        return result
```

复杂度：时间 `O(n)`，空间 `O(k)`。

# 11. 缺失的第一个正数（First Missing Positive）

在未排序整数数组中找出没有出现的最小正整数，要求 `O(n)` 时间和 `O(1)` 额外空间。

## 破局点

答案一定在 `[1, n + 1]`。把值 `x` 原地交换到下标 `x - 1`；归位后，第一个满足 `nums[i] != i + 1` 的位置就是答案。交换条件必须排除重复值以防死循环。

```python
class Solution:
    def firstMissingPositive(self, nums: List[int]) -> int:
        n = len(nums)

        for i in range(n):
            while (
                1 <= nums[i] <= n
                and nums[nums[i] - 1] != nums[i]
            ):
                target = nums[i] - 1
                nums[i], nums[target] = nums[target], nums[i]

        for i, num in enumerate(nums):
            if num != i + 1:
                return i + 1

        return n + 1
```

复杂度：时间 `O(n)`，空间 `O(1)`。

# 12. 二叉树中的最大路径和（Binary Tree Maximum Path Sum）

求二叉树中任意非空路径的最大节点值之和；路径中相邻节点必须有边相连，且节点不能重复。

## 破局点

向父节点只能贡献一条单边路径，因此 DFS 返回 `node.val + max(left_gain, right_gain)`；但以当前节点为最高点的完整路径可以同时取左右增益。负增益应截断为 0。

```python
class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        best = float('-inf')

        def gain(node):
            nonlocal best
            if not node:
                return 0

            left_gain = max(gain(node.left), 0)
            right_gain = max(gain(node.right), 0)

            best = max(best, node.val + left_gain + right_gain)
            return node.val + max(left_gain, right_gain)

        gain(root)
        return best
```

复杂度：时间 `O(n)`，空间 `O(h)`，`h` 为树高。
