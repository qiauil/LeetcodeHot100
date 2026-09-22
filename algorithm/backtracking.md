## What Is Backtracking?

Backtracking is a **depth-first search (DFS) technique that systematically enumerates possible solutions**. Its basic process can be summarized as follows:

> **Backtracking = make a choice → explore that choice → undo it when the path fails or has been fully explored → try the next choice.**

Imagine walking through a maze. At each fork, you have several possible directions:

```text
       Start
         |
     ┌───┴───┐
   Left     Right
   /  \      /  \
  A    B    C    D
```

Your strategy might be:

1. Take the left path.
2. Choose another path at the next fork.
3. If you reach a dead end:
   - go back;
   - try a different path.
4. Continue until you find the exit.

The act of **going back** is the essence of backtracking.

## A General Backtracking Template

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

When designing a backtracking solution, focus on three questions:

1. **What is the current path?** It represents the choices already made.
2. **What choices are currently available?** These are the valid next steps from the current state.
3. **When should the search stop?** The current path may form a complete solution, or it may be impossible to extend into one.

## Backtracking and Depth-First Search

> **Backtracking is usually implemented with DFS.**

DFS describes the order in which a search proceeds:

```text
Go as deep as possible
          ↓
Reach the end of a branch
          ↓
Return to the previous state
```

Backtracking places additional emphasis on state changes:

```text
Make a choice
      ↓
Search recursively
      ↓
Restore the previous state
```

For example:

```python
path.append(x)
dfs()
path.pop()
```

The call to `path.pop()` is the characteristic backtracking step: it restores the state so that the next choice can be explored independently.

In short:

```text
DFS + state restoration = backtracking
```

## Classic Backtracking Problems

### 1. Permutations

Given:

```text
nums = [1, 2, 3]
```

we want to generate every possible ordering:

```text
[1, 2, 3]
[1, 3, 2]
[2, 1, 3]
[2, 3, 1]
[3, 1, 2]
[3, 2, 1]
```

The search process can be represented as a decision tree:

```text
                       []
             /          |          \
           [1]         [2]         [3]
          /   \        /  \        /  \
      [1,2] [1,3]  [2,1] [2,3] [3,1] [3,2]
        |      |      |      |      |      |
    [1,2,3][1,3,2][2,1,3][2,3,1][3,1,2][3,2,1]
```

Backtracking traverses this **decision tree**. At each level, it chooses one unused number; after exploring that branch, it marks the number as available again.

```python
def permute(nums):
    result = []
    path = []
    used = [False] * len(nums)

    def backtrack():
        if len(path) == len(nums):
            result.append(path.copy())
            return

        for i in range(len(nums)):
            if used[i]:
                continue

            # Make a choice
            used[i] = True
            path.append(nums[i])
            backtrack()
            path.pop()
            used[i] = False

    backtrack()
    return result


print(permute([1, 2, 3]))
```

Output:

```text
[[1, 2, 3], [1, 3, 2], [2, 1, 3],
 [2, 3, 1], [3, 1, 2], [3, 2, 1]]
```

### 2. Combinations

Suppose:

```text
nums = [1, 2, 3, 4]
k = 2
```

We want all combinations of length two:

```text
[1, 2]
[1, 3]
[1, 4]
[2, 3]
[2, 4]
[3, 4]
```

Unlike permutations, combinations do not care about order: `[1, 2]` and `[2, 1]` represent the same choice. A `start` index prevents the algorithm from revisiting earlier elements.

```python
def combine(nums, k):
    result = []
    path = []

    def backtrack(start):
        if len(path) == k:
            result.append(path.copy())
            return

        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1)
            path.pop()

    backtrack(0)
    return result


print(combine([1, 2, 3, 4], 2))
```

Output:

```text
[[1, 2], [1, 3], [1, 4], [2, 3], [2, 4], [3, 4]]
```

### 3. Subsets

Given `[1, 2, 3]`, the complete set of subsets is:

```text
[]
[1]
[2]
[3]
[1, 2]
[1, 3]
[2, 3]
[1, 2, 3]
```

The search structure resembles the combinations problem, but there is one important difference: **every node in the decision tree is a valid result**, not only paths of a particular length.

```python
def subsets(nums):
    result = []
    path = []

    def backtrack(start):
        result.append(path.copy())

        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1)
            path.pop()

    backtrack(0)
    return result


print(subsets([1, 2, 3]))
```

Output:

```text
[[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]
```

## Comparing Permutations, Combinations, and Subsets

All three problems can be understood as traversals of a decision tree. The main difference is how each algorithm chooses the next branch and decides when to record an answer.

| Problem | Available choices | When to record a result |
|---|---|---|
| Permutations | Any element not yet used | When the path contains every element |
| Combinations | Elements at or after `start` | When the path reaches length `k` |
| Subsets | Elements at or after `start` | At every node in the tree |

For permutations, every level may choose any unused number:

```text
         []
      /   |   \
     1    2    3
```

For combinations, once `1` is selected, only `2`, `3`, and later elements remain available. After selecting `2`, only `3` and later elements remain. The `start` index enforces this rule and avoids duplicate combinations.

Subsets use a similar search structure:

```text
[]
├── [1]
│   ├── [1, 2]
│   │   └── [1, 2, 3]
│   └── [1, 3]
├── [2]
│   └── [2, 3]
└── [3]
```

Here, every node—including the empty root—is part of the answer.

## Pruning the Search Tree

Suppose a search tree looks like this:

```text
                 root
            /      |      \
           A       B       C
          /|\     /|\     /|\
```

If we know in advance that branch `B` cannot produce a valid answer, there is no reason to explore any part of its subtree. Skipping such branches is called **pruning**.

Pruning often appears in code as:

```python
for choice in choices:
    if cannot_produce_a_solution(choice):
        continue
```

or:

```python
if current_state_is_invalid:
    return
```

Pruning can dramatically reduce the number of states an algorithm explores.

### Example: Generate Parentheses

For `n = 3`, all valid strings are:

```text
((()))
(()())
(())()
()(())
()()()
```

A brute-force approach could generate every six-character string made from `(` and `)`, then test each one. Backtracking avoids invalid strings during the search by enforcing two rules:

```text
number of opening parentheses <= n
number of closing parentheses <= number of opening parentheses
```

The second rule prevents invalid prefixes such as `)(`. Once a prefix is invalid, none of its descendants can form a valid result, so the entire branch is pruned.

```python
def generate_parentheses(n):
    result = []
    path = []

    def backtrack(open_count, close_count):
        if open_count == close_count == n:
            result.append("".join(path))
            return

        if open_count < n:
            path.append("(")
            backtrack(open_count + 1, close_count)
            path.pop()

        if close_count < open_count:
            path.append(")")
            backtrack(open_count, close_count + 1)
            path.pop()

    backtrack(0, 0)
    return result


print(generate_parentheses(3))
```

Output:

```text
['((()))', '(()())', '(())()', '()(())', '()()()']
```

## Final Takeaway

Backtracking becomes much easier to reason about when you view it as a traversal of a decision tree. At every node:

1. choose one available option;
2. recursively explore the resulting state;
3. undo the choice;
4. prune the branch as soon as it cannot lead to a valid solution.

Once you can identify the **path**, the **available choices**, and the **stopping condition**, many seemingly different problems reduce to the same reusable pattern.
