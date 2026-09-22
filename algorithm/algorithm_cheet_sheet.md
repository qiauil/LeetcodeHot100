## Backtracking

### A General Backtracking Template

Most backtracking algorithms follow this pattern:

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

### Example

[Permutations](https://leetcode.cn/problems/permutations/)

Given an array `nums` of distinct integers, return all the possible permutations. You can return the answer in **any order**.

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

## Binary Search

```python
def binary_search(nums, target):
    left = 0
    right = len(nums)

    while left < right:
        mid = (left + right) // 2

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left
```

### Example

[Search Insert Position](https://leetcode.cn/problems/search-insert-position/)

Given a sorted array of distinct integers and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

You must write an algorithm with `O(log n)` runtime complexity.

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

## Binary Tree

### DFS

* pre-order

```python
def inorder_traversal(node: TreeNode):
    if node is None:
        return
    print(node.val)
    inorder_traversal(node.left)
    inorder_traversal(node.right)
```

* in-order

```python
def inorder_traversal(node: TreeNode):
    if node is None:
        return
    inorder_traversal(node.left)
    print(node.val)
    inorder_traversal(node.right)
```

* post-order

```python
def postorder_traversal(node: TreeNode):
    if node is None:
        return
    postorder_traversal(node.left)
    postorder_traversal(node.right)
    print(node.val)
```

### Iterative DFS

* pre-order

```python
def preorder_iterative(root):
    if root is None:
        return []
    result = []
    stack = [root]
    while stack:
        current = stack.pop()
        result.append(current.val)
        # Important: push right first, then left
        if current.right:
            stack.append(current.right)
        if current.left:
            stack.append(current.left)
    return result
```

* In-order

```python
def inorder_iterative(root):
    result = []
    stack = []
    current = root
    while current or stack:
        # 1. Keep moving left
        while current:
            stack.append(current)
            current = current.left
        # 2. Once the leftmost point is reached, visit the stack's top node
        current = stack.pop()
        result.append(current.val)
        # 3. Move to the right subtree
        current = current.right
    return result
```

* post-order

```python
def inorder_iterative(root):
    result = []
    stack = []
    current = root
    while current or stack:
        # 1. Keep moving left
        while current:
            stack.append(current)
            current = current.left
        # 2. Once the leftmost point is reached, visit the stack's top node
        current = stack.pop()
        result.append(current.val)
        # 3. Move to the right subtree
        current = current.right
    return result
```

### BFS

```python
from collections import deque

class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []

        res = []
        q = deque([root])

        while q:
            # in each iteration, we will process all nodes at the current level
            # Get the number of nodes at the current level
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

