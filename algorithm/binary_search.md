# Binary Search Explained: Closed, Half-Open, and Open Intervals

Binary search is often introduced as a simple algorithm:

1. Look at the middle element.
2. Decide whether to search the left half or the right half.
3. Repeat until the answer is found.

The idea sounds easy, but binary search is famous for producing subtle bugs.

Should the loop condition be `left < right` or `left <= right`?

Should we write `right = mid`, `right = mid - 1`, or `left = mid + 1`?

Why do some implementations initialize `right = len(nums) - 1`, while others use `right = len(nums)`?

The answer is that these details are not arbitrary. They are determined by one design choice:

**How do we define the current search interval?**

In this article, we will use the classic `lower_bound` problem to understand three common binary-search styles:

- Closed interval: `[left, right]`
- Left-closed, right-open interval: `[left, right)`
- Open interval: `(left, right)`

Once the interval definition is clear, the loop condition and pointer updates follow naturally.

------

## 1. The Problem: Find the First Element Greater Than or Equal to the Target

Suppose we are given a sorted array and a target value.

We want to return the index of the target if it exists. Otherwise, we return the position where it should be inserted to preserve sorted order.

For example:

```python
nums = [1, 3, 5, 7, 9]
target = 6
```

The answer is:

```text
3
```

because `7` is the first element greater than or equal to `6`.

More generally, the problem is:

> Find the smallest index `i` such that `nums[i] >= target`.

This operation is commonly called `lower_bound`.

Instead of thinking about "finding the target," it is often more useful to transform the array into a Boolean condition:

```python
nums[i] >= target
```

For example:

```text
nums   = [1, 3, 5, 5, 5, 8, 10]
target = 5

condition:
         F  F  T  T  T  T   T
```

The problem now becomes:

> Find the first `True`.

This viewpoint makes binary search much easier to reason about.

------

## 2. The Real Foundation of Binary Search: Loop Invariants

All three implementations in this article maintain the same conceptual structure:

```text
definitely < target | unknown region | definitely >= target
```

The algorithm repeatedly picks a middle index from the unknown region and classifies it.

If:

```python
nums[mid] < target
```

then `mid` belongs to the left region.

If:

```python
nums[mid] >= target
```

then `mid` belongs to the right region.

The only difference between the three implementations is how `left` and `right` are positioned relative to the unknown region.

That difference determines:

- initialization,
- loop condition,
- pointer updates,
- termination condition.

------

# 3. Closed Interval: `[left, right]`

A closed-interval implementation looks like this:

```python
def lower_bound(nums, target):
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = (left + right) // 2

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return left
```

The key definition is:

> `[left, right]` is the current unknown search region.

Everything outside that interval has already been classified.

Conceptually:

```text
[0 ... left-1]    [left ... right]    [right+1 ... n-1]
   < target           unknown             >= target
```

A useful loop invariant is:

```text
nums[left - 1] < target
nums[right + 1] >= target
```

when those indices exist.

------

## 4. Why Initialize `left = 0` and `right = n - 1`?

Initially, every array element is unknown.

If the array has length `n`, its valid indices are:

```text
0, 1, 2, ..., n - 1
```

Therefore the entire unknown region is:

```text
[0, n - 1]
```

So we initialize:

```python
left = 0
right = len(nums) - 1
```

------

## 5. Why Is the Loop Condition `left <= right`?

The search region is a closed interval:

```text
[left, right]
```

A closed interval still contains an element when:

```text
left == right
```

For example:

```text
[3, 3]
```

contains exactly one index: `3`.

Therefore the region is non-empty whenever:

```python
left <= right
```

The loop stops only when:

```text
left > right
```

At that point, the unknown region is empty.

------

## 6. Why Does `nums[mid] < target` Give `left = mid + 1`?

Suppose:

```python
nums[mid] < target
```

Because the array is sorted, every index up to `mid` also contains a value smaller than the target.

Therefore none of these indices can be the answer:

```text
[left, mid]
```

The remaining unknown region becomes:

```text
[mid + 1, right]
```

So we write:

```python
left = mid + 1
```

The `+1` is important.

Since `mid` belongs to the current closed interval and has just been proven not to be the answer, it must be removed from the unknown region.

If we wrote:

```python
left = mid
```

the interval might fail to shrink and the algorithm could enter an infinite loop.

------

## 7. Why Does `nums[mid] >= target` Give `right = mid - 1`?

Suppose:

```python
nums[mid] >= target
```

Then `mid` is a valid candidate, but it may not be the first valid candidate.

For example:

```text
nums = [1, 3, 5, 5, 5, 8]
target = 5
```

If `mid == 3`, then `nums[mid] == 5`, but the correct answer is index `2`.

So we still need to search to the left.

More importantly, once we know:

```python
nums[mid] >= target
```

`mid` no longer belongs to the unknown region. It can be classified as part of the right side.

Therefore the unknown interval changes from:

```text
[left, right]
```

to:

```text
[left, mid - 1]
```

which gives:

```python
right = mid - 1
```

------

## 8. Why Does the Closed-Interval Version Return `left`?

When the loop terminates:

```text
left > right
```

In practice, the pointers have crossed:

```text
left == right + 1
```

From the invariant:

```text
nums[left - 1] < target
nums[right + 1] >= target
```

and because:

```text
right + 1 == left
```

we obtain:

```text
nums[left - 1] < target
nums[left] >= target
```

Therefore `left` is exactly the first position whose value is greater than or equal to the target.

So:

```python
return left
```

------

# 9. Left-Closed, Right-Open Interval: `[left, right)`

The second style is:

```python
def lower_bound(nums, target):
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

This time the unknown region is:

```text
[left, right)
```

The left boundary is included, while the right boundary is excluded.

For example:

```text
[2, 5)
```

contains indices:

```text
2, 3, 4
```

but not `5`.

------

## 10. Why Initialize `right = len(nums)`?

Suppose the array length is `5`.

Its valid indices are:

```text
0, 1, 2, 3, 4
```

The entire array can be represented naturally as:

```text
[0, 5)
```

Therefore:

```python
left = 0
right = len(nums)
```

Here, `right` does not have to be a valid array index.

It is a boundary.

That distinction is important.

------

## 11. Why Is the Loop Condition `left < right`?

The unknown interval is:

```text
[left, right)
```

This interval becomes empty when:

```text
left == right
```

For example:

```text
[4, 4)
```

contains no elements.

Therefore the interval is non-empty exactly when:

```python
left < right
```

------

## 12. Why Does the Left Boundary Still Use `mid + 1`?

If:

```python
nums[mid] < target
```

then `mid` definitely belongs to the left side.

Since `mid` is inside the unknown interval, it must be removed from it.

The new unknown interval is:

```text
[mid + 1, right)
```

so:

```python
left = mid + 1
```

This is the same logic as in the closed-interval version.

------

## 13. Why Does the Right Boundary Use `right = mid`?

This is one of the most important differences.

Suppose:

```python
nums[mid] >= target
```

Then `mid` belongs to the right side.

The invariant for this version can be viewed as:

```text
[0, left)      definitely < target
[left, right)  unknown
[right, n)     definitely >= target
```

Once `mid` has been classified as `>= target`, we can move it into the right region.

The new unknown region becomes:

```text
[left, mid)
```

Therefore:

```python
right = mid
```

Notice that `mid` does not remain in the unknown region.

It becomes the new right boundary, and the right boundary is excluded from `[left, right)`.

This is why we do not need `mid - 1`.

------

## 14. Why Can We Return Either `left` or `right`?

The loop ends when:

```text
left == right
```

At this point, the unknown interval is:

```text
[left, left)
```

which is empty.

The invariant tells us:

```text
everything before left is < target
everything from right onward is >= target
```

Since:

```text
left == right
```

that position is exactly the boundary between the two regions.

Therefore both are equivalent:

```python
return left
```

or:

```python
return right
```

------

# 15. Open Interval: `(left, right)`

The third implementation is:

```python
def lower_bound(nums, target):
    left, right = -1, len(nums)

    while left + 1 < right:
        mid = (left + right) // 2

        if nums[mid] < target:
            left = mid
        else:
            right = mid

    return right
```

This version uses the cleanest invariant of the three.

The meaning of the boundaries is:

```text
left  is known to satisfy nums[left] < target
right is known to satisfy nums[right] >= target
```

The unknown region is strictly between them:

```text
(left, right)
```

Conceptually:

```text
< target | left | unknown region | right | >= target
```

------

## 16. Why Initialize `left = -1` and `right = n`?

At the beginning, we do not know any real array element that definitely belongs to the left or right region.

So we introduce two conceptual sentinels:

```text
left = -1
right = n
```

You can imagine:

```text
nums[-1] = -infinity
nums[n]  = +infinity
```

These values are never actually accessed.

They are only a mathematical model.

This immediately gives us the invariant:

```text
nums[left] < target
nums[right] >= target
```

conceptually.

The open interval:

```text
(-1, n)
```

contains all real array indices:

```text
0, 1, ..., n - 1
```

------

## 17. Why Is the Loop Condition `left + 1 < right`?

The unknown region is:

```text
(left, right)
```

An open interval contains at least one integer index only when there is space between the boundaries.

For example:

```text
left = 2
right = 4
```

Then:

```text
(2, 4)
```

contains index `3`.

But if:

```text
left = 2
right = 3
```

then:

```text
(2, 3)
```

contains no integer index at all.

Therefore the loop continues while:

```python
left + 1 < right
```

and terminates when:

```text
left + 1 == right
```

------

## 18. Why Are There No `+1` or `-1` Updates?

This is the elegant part of the open-interval formulation.

Suppose:

```python
nums[mid] < target
```

Then `mid` has been classified as belonging to the left side.

Since `left` itself is outside the unknown interval, we can simply make `mid` the new left boundary:

```python
left = mid
```

The new unknown region becomes:

```text
(mid, right)
```

Similarly, if:

```python
nums[mid] >= target
```

then `mid` belongs to the right side.

So we write:

```python
right = mid
```

and the unknown interval becomes:

```text
(left, mid)
```

No `+1` or `-1` is necessary because neither boundary belongs to the unknown interval.

------

# 19. Comparing the Three Styles

The three implementations can be summarized as follows:

| Style     | Unknown Region  | Initial State               | Continue While     | `< target`       | `>= target`       | Result  |
| --------- | --------------- | --------------------------- | ------------------ | ---------------- | ----------------- | ------- |
| Closed    | `[left, right]` | `left = 0`, `right = n - 1` | `left <= right`    | `left = mid + 1` | `right = mid - 1` | `left`  |
| Half-open | `[left, right)` | `left = 0`, `right = n`     | `left < right`     | `left = mid + 1` | `right = mid`     | `left`  |
| Open      | `(left, right)` | `left = -1`, `right = n`    | `left + 1 < right` | `left = mid`     | `right = mid`     | `right` |

The important point is that none of these update rules should be memorized in isolation.

They are consequences of the interval definition.

------

# 20. Where Do the `+1` and `-1` Come From?

A useful general rule is:

> Whether you need `+1` or `-1` depends on whether the boundary itself belongs to the unknown region.

For a closed interval:

```text
[left, right]
```

both boundaries belong to the unknown region.

After classifying `mid`, it must be excluded:

```python
left = mid + 1
right = mid - 1
```

For a left-closed, right-open interval:

```text
[left, right)
```

`left` belongs to the unknown region, while `right` does not.

Therefore:

```python
left = mid + 1
right = mid
```

For an open interval:

```text
(left, right)
```

neither boundary belongs to the unknown region.

Therefore the classified `mid` can directly become the new boundary:

```python
left = mid
right = mid
```

This is the underlying reason behind the different templates.

------

# 21. Running the Same Example Through All Three Versions

Consider:

```python
nums = [1, 3, 5, 7, 9]
target = 6
```

The answer is index `3`, because:

```text
nums[2] = 5 < 6
nums[3] = 7 >= 6
```

### Closed interval

Start:

```text
[0, 4]
```

`mid = 2`, and:

```text
nums[2] = 5 < 6
```

so:

```text
left = 3
```

Now:

```text
[3, 4]
```

`mid = 3`, and:

```text
nums[3] = 7 >= 6
```

so:

```text
right = 2
```

Now:

```text
left = 3
right = 2
```

The interval is empty.

Return:

```text
3
```

### Half-open interval

Start:

```text
[0, 5)
```

`mid = 2`, so:

```text
nums[2] = 5 < 6
```

and:

```text
left = 3
```

Now:

```text
[3, 5)
```

`mid = 4`, and:

```text
nums[4] = 9 >= 6
```

so:

```text
right = 4
```

Now:

```text
[3, 4)
```

`mid = 3`, and:

```text
nums[3] = 7 >= 6
```

so:

```text
right = 3
```

Now:

```text
[3, 3)
```

The interval is empty.

Return:

```text
3
```

### Open interval

Start:

```text
(-1, 5)
```

`mid = 2`, and:

```text
nums[2] = 5 < 6
```

so:

```text
left = 2
```

Now:

```text
(2, 5)
```

`mid = 3`, and:

```text
nums[3] = 7 >= 6
```

so:

```text
right = 3
```

Now:

```text
(2, 3)
```

There are no unknown indices left.

Return:

```text
3
```

All three approaches arrive at exactly the same result.

------

# 22. Binary Search Is Really About Finding a Boundary

A major improvement in understanding binary search comes from changing the mental model.

Instead of thinking:

> "I am looking for the target."

think:

> "I am looking for the boundary between two monotonic regions."

For `lower_bound`, the regions are:

```text
nums[i] < target       nums[i] >= target
False False False ...  True True True ...
```

We are looking for the first `True`.

Once binary search is understood this way, many variations become straightforward:

- first element `>= target`
- first element `> target`
- last element `< target`
- last element `<= target`
- first occurrence of a value
- last occurrence of a value
- minimum feasible answer
- maximum feasible answer

This is also why binary search works far beyond arrays.

Whenever a Boolean condition changes monotonically from one state to another, there may be a binary-search solution.

------

# 23. Why Is Binary Search `O(log n)`?

At every iteration, binary search reduces the unknown region by roughly half.

Starting with `n` candidates:

```text
n
n / 2
n / 4
n / 8
...
```

After `k` iterations, approximately:

```text
n / 2^k
```

candidates remain.

The process stops when that quantity becomes approximately `1`:

```text
n / 2^k ≈ 1
```

which gives:

```text
2^k ≈ n
```

and therefore:

```text
k ≈ log2(n)
```

So the time complexity is:

```text
O(log n)
```

while the extra space complexity is:

```text
O(1)
```

for the iterative versions shown here.

------

# 24. A Better Way to Write Binary Search

Rather than memorizing a template, use the following reasoning process.

First, define the boundary you are looking for.

For example:

```text
first index where nums[i] >= target
```

Then define the unknown region.

Choose one of:

```text
[left, right]
[left, right)
(left, right)
```

Next, state the loop invariant explicitly.

For example:

```text
everything to the left is < target
everything to the right is >= target
```

Then derive the loop condition from when the unknown interval is non-empty.

Finally, classify `mid` and move it out of the unknown region.

If you follow this process, the correct `+1`, `-1`, and loop condition usually become obvious.

------

# 25. Final Takeaway

The most important lesson about binary search is not a particular code template.

It is this:

> Binary search is about maintaining a precisely defined unknown interval and shrinking it while preserving a loop invariant.

The three common styles differ only in where their boundaries are placed:

```text
Closed:
[left, right]

Half-open:
[left, right)

Open:
(left, right)
```

That single decision explains almost everything:

```text
Closed interval:
mid is inside on both sides
→ use mid + 1 or mid - 1

Half-open interval:
right boundary is already outside the unknown region
→ use mid + 1 or mid

Open interval:
both boundaries are outside the unknown region
→ use mid or mid
```

Once you understand the relationship between **interval semantics**, **loop invariants**, and **boundary updates**, binary search stops being a collection of mysterious templates.

It becomes something you can derive from first principles.
