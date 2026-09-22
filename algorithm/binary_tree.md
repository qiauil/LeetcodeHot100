Binary tree is an important data structure. It is used in many applications, such as expression parsing, searching, and sorting. In most cases, the traversal of a binary tree is the core step in solving problems related to binary trees. In this notebook, we will discuss four common types of binary tree traversal metods.

We will use the following binary tree as an example:

```text
        1
       / \
      2   3
     / \   \
    4   5   6
```


```python
# initialize binary tree with a list of values
from typing import List
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

root = TreeNode(1)

root.left = TreeNode(2)
root.right = TreeNode(3)

root.left.left = TreeNode(4)
root.left.right = TreeNode(5)

root.right.right = TreeNode(6)
```

## Depth-First Traversal

Depth-first traversal is a type of traversal that explores as far as possible along each branch before **backtracking**. A most common implementation of depth-first traversal will introduce recursion:


```python
def dfs(node: TreeNode):
    if node is None:
        return
    dfs(node.left)
    dfs(node.right)
```

The above code goes through a binary tree in depth-first order, visiting the left subtree first, then the right subtree. The function `dfs` takes a `TreeNode` as input and recursively calls itself on the left and right children of the node until it reaches a leaf node (where both left and right children are `None`). The **backtracking** occurs after the `dfs(node.right)` call, when the function returns to the previous level of recursion, i.e., going back to the parent node to explore the right subtree after finishing the left subtree.

With the example binary tree, the above code works as follows:
1. `dfs(1)` is called, not None, so it calls `dfs(node.left)`, which is `dfs(2)`.
2. `dfs(2)` is called, not None, so it calls `dfs(node.left)`, which is `dfs(4)`.
3. `dfs(4)` is called, not None, so it calls `dfs(node.left)` for the left child.
4. `dfs(None)` is called, it returns immediately.
5. Back to `dfs(4)`, it now calls `dfs(node.right)` for the right child.
6. `dfs(None)` is called, it returns immediately.
7. **Back to `dfs(2)`**, it now calls `dfs(node.right)`.
8. `dfs(5)` is called, not None, so it calls `dfs(node.left)` for the left child.
9. `dfs(None)` is called, it returns immediately.
10. **Back to `dfs(5)`**, it now calls `dfs(node.right)`.
11. `dfs(None)` is called, it returns immediately.
12. **Back to `dfs(2)`**, it has finished exploring both subtrees, so it returns.
13. **Back to `dfs(1)`**, it now calls `dfs(node.right)` for the right subtree.
14. `dfs(3)` is called, not None, so it calls `dfs(node.left)` for the left child.
15. `dfs(None)` is called, it returns immediately.
16. **Back to `dfs(3)`**, it now calls `dfs(node.right)` for the right child.
17. `dfs(None)` is called, it returns immediately.
18. **Back to `dfs(1)`**, it has finished exploring both subtrees, so it returns.

There are three common types of depth-first traversal: **pre-order**, **in-order**, and **post-order**. The difference between them is the order in which the nodes are visited. Implementations of these thre types of depth-first traversal can be modified from the above code.

### Pre-order Traversal
Pre-order traversal visits the root node first, then the left subtree, and finally the right subtree. In pre-order traversal, the above example binary tree will be traversed in the following order: [1, 2, 4, 5, 3, 6]. The implementation of pre-order traversal is as follows:


```python
def preorder_traversal(node: TreeNode):
    if node is None:
        return
    print(node.val)
    preorder_traversal(node.left)
    preorder_traversal(node.right)

preorder_traversal(root)
```

    1
    2
    4
    5
    3
    6


### In-order Traversal
In-order traversal visits the left subtree first, then the root node, and finally the right subtree. In in-order traversal, the above example binary tree will be traversed in the following order: [4, 2, 5, 1, 3, 6]. The implementation of in-order traversal is as follows:


```python
def inorder_traversal(node: TreeNode):
    if node is None:
        return
    inorder_traversal(node.left)
    print(node.val)
    inorder_traversal(node.right)
inorder_traversal(root)
```

    4
    2
    5
    1
    3
    6


### Post-order Traversal
Post-order traversal visits the left subtree first, then the right subtree, and finally the root node. In post-order traversal, the above example binary tree will be traversed in the following order: [4, 5, 2, 6, 3, 1]. The implementation of post-order traversal is as follows:


```python
def postorder_traversal(node: TreeNode):
    if node is None:
        return
    postorder_traversal(node.left)
    postorder_traversal(node.right)
    print(node.val)
postorder_traversal(root)
```

    4
    5
    2
    6
    3
    1


## Iterative depth-First Traversal
Sometimes, we may want to implement depth-first traversal without recursion. In this case, we can use a stack to keep track of the nodes to be visited. The implementation of depth-first traversal with a stack is slight difficult than the recursive implementation. Below are the implementations of pre-order, in-order, and post-order traversal with a stack.
### 7.1 Iterative Preorder

Because a stack is **last in, first out (LIFO)**, push the right child before the left child so that the left subtree is visited first.


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
preorder_iterative(root)
```




    [1, 2, 4, 5, 3, 6]



One way to understand the execution is:

```text
stack = [1]

Pop 1
result = [1]
stack = [3, 2]

Pop 2
result = [1, 2]
stack = [3, 5, 4]

Pop 4
result = [1, 2, 4]
...
```

---

### 7.2 Iterative Inorder: Left → Root → Right

The key to iterative inorder traversal is:

> **Keep moving left and push every node you pass onto the stack.**

When you cannot move farther left, pop and visit the top node, then move to its right subtree.


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


print(inorder_iterative(root))
```

    [4, 2, 5, 1, 3, 6]


At the beginning, for example:

```text
current = 1

Push 1
↓
Push 2
↓
Push 4
↓
None
```

* At this point, `stack = [1, 2, 4]`. There is no remaining left child, so pop `4`: `result = [4]`, `stack = [1, 2]`.
* Current is now `4.right`, which is `None`. Pop `2`: `result = [4, 2]`, `stack = [1]`. 
* Current is now `2.right`, which is `5`. Push `5`: `stack = [1, 5]`. There is no remaining left child, so pop `5`: `result = [4, 2, 5]`, `stack = [1]`. 
* Current is now `5.right`, which is `None`. Pop `1`: `result = [4, 2, 5, 1]`, `stack = []`. 
* Current is now `1.right`, which is `3`. Push `3`: `stack = [3]`. There is no remaining left child, so pop `3`: `result = [4, 2, 5, 1, 3]`, `stack = []`. 
* Current is now `3.right`, which is `6`. Push `6`: `stack = [6]`. There is no remaining left child, so pop `6`: `result = [4, 2, 5, 1, 3, 6]`, and the stack is empty. The traversal is complete.

The process can be remembered as:

```text
Move all the way left → push nodes → when blocked, pop → visit → move right → repeat
```

---

### 7.3 Iterative Postorder: Left → Right → Root

There are several iterative postorder implementations. One intuitive approach first constructs:

```text
Root → Right → Left
```

, which is similar to preorder traversal. Reversing that result produces:

```text
Left → Right → Root
```


```python
def postorder_iterative(root):
    if root is None:
        return []
    result = []
    stack = [root]
    while stack:
        node = stack.pop()
        result.append(node.val)
        # Push left first, then right
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)
    return result[::-1]


print(postorder_iterative(root))
```

    [4, 5, 2, 6, 3, 1]


A useful mnemonic is that preorder is `Root → Left → Right`, so nodes are pushed right first and then left. For postorder, first construct `Root → Right → Left` by pushing left first and then right. Finally, `result[::-1]` gives `Left → Right → Root`.

## 5. Level-Order Traversal

Level-order traversal differs from the first three: it visits the tree level by level, **from top to bottom and from left to right**.

```text
        1        ← Level 1
       / \
      2   3      ← Level 2
     / \   \
    4   5   6    ← Level 3
```

The result is:

```text
[1, 2, 3, 4, 5, 6]
```

It is usually implemented with a **queue**:


```python
from collections import deque

def level_order(root):
    if root is None:
        return []
    result = []
    queue = deque([root])
    while queue:
        node = queue.popleft()
        result.append(node.val)
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
    return result


print(level_order(root))
```

    [1, 2, 3, 4, 5, 6]


The central idea is:

```text
Queue: [1]

Remove 1; add 2 and 3
[2, 3]

Remove 2; add 4 and 5
[3, 4, 5]

Remove 3; add 6
[4, 5, 6]

...
```

It is therefore fundamentally **BFS (breadth-first search)**.

In practice, there is another common implementation of level-order traversal that can distinguish the levels of the tree. Below is a simple implementation:

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

---

## 6. Comparing the Four Traversals

For the tree:

```text
        1
       / \
      2   3
     / \   \
    4   5   6
```

The results can be summarized as follows:

| Traversal | Order | Output |
| --- | --- | --- |
| Preorder | Root → Left → Right | `[1, 2, 4, 5, 3, 6]` |
| Inorder | Left → Root → Right | `[4, 2, 5, 1, 3, 6]` |
| Postorder | Left → Right → Root | `[4, 5, 2, 6, 3, 1]` |
| Level order | Top to bottom, left to right | `[1, 2, 3, 4, 5, 6]` |

