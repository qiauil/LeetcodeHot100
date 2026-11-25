## 回溯

回溯算法模板：

```python
def backtrack(path, choices):
    if end_condition_is_met:
        result.append(path.copy())
        return

    for choice in choices:
        if choice_is_invalid(choice):
            continue

        # 1. Make a choice
        path.append(choice)

        # 2. Explore recursively
        backtrack(path, choices)

        # 3. Undo the choice
        path.pop()
```

例子：[全排列](https://leetcode.cn/problems/permutations/)

给定一个不含重复数字的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。

```
Input: nums = [1,2,3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        res = []
        def backtrack(current,used):
            if len(current)==len(nums):
                res.append(current.copy())
                return
            for i in range(len(nums)):
                if not used[i]:
                    current.append(nums[i])
                    used[i] = True
                    backtrack(current,used)
                    current.pop()
                    used[i]=False
        backtrack([],[False]*len(nums))
        return res
```

## 二分法

二分法模板：

```python
def binary_search(nums, target):
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return left
```

例子：[Search Insert Position](https://leetcode.cn/problems/search-insert-position/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

**Example 1:**

```
Input: nums = [1,3,5,6], target = 5
Output: 2
```

```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        l,r=0,len(nums)-1
        while l<=r:
            mid = (l+r)//2
            if nums[mid] < target:
                l = mid +1
            else:
                r = mid - 1
        return l
```

## 二叉树

### 深度优先搜索（DFS）

* 前序（pre-order ，根-左-右）

```python
def inorder_traversal(node: TreeNode):
    if node is None:
        return
    print(node.val)
    inorder_traversal(node.left)
    inorder_traversal(node.right)
```

* 中序（in-order， 左-根-右）

```python
def inorder_traversal(node: TreeNode):
    if node is None:
        return
    inorder_traversal(node.left)
    print(node.val)
    inorder_traversal(node.right)
```

* 后序(post-order, 左-右-根)

```python
def postorder_traversal(node: TreeNode):
    if node is None:
        return
    postorder_traversal(node.left)
    postorder_traversal(node.right)
    print(node.val)
```

### 迭代式深度优先搜索 （Iterative DFS）

* 前序

```python
def preorder_iterative(root):
    if root is None:
        return []
    result = []
    stack = [root]
    while stack:
        current = stack.pop()
        result.append(current.val)
        # 重点：先右后左，因为pop的时候从尾部开始
        if current.right:
            stack.append(current.right)
        if current.left:
            stack.append(current.left)
    return result
```

* 中序

```python
def inorder_iterative(root):
    result = []
    stack = []
    current = root
    while current or stack:
        # 1. 一路向左
        while current:
            stack.append(current)
            current = current.left
        # 2. 到达左侧尽头，添加结果
        current = stack.pop()
        result.append(current.val)
        # 转向右子树
        current = current.right
    return result
```

* 后序 

  仿照前序，遍历根-右-左，然后列表倒着来，输出左右根

```python
def postorder_iterative(root):
    if root is None:
        return []
    result = []
    stack = [root]
    while stack:
        node = stack.pop()
        result.append(node.val)
        # 先左后右
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)
    return result[::-1]
```

### 广度优先搜索（BFS）：

```python
from collections import deque

class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []

        res = []
        q = deque([root])

        while q:
            # 遍历当前层
            level_size = len(q)
            for i in range(level_size):
                node = q.popleft()
                res.append(node.val)
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
        return res
```

