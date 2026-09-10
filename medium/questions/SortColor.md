# Sort Color

You are given an array `nums` consisting of `n` elements where each element is an integer representing a color:

- `0` represents red
- `1` represents white
- `2` represents blue

Your task is to **sort the array in-place** such that elements of the same color are grouped together and arranged in the order: red (0), white (1), and then blue (2).

You **must not** use any built-in sorting functions to solve this problem.

## 1. Brute Force

### Intuition

The simplest approach is to use a standard sorting algorithm. Since the array only contains values 0, 1, and 2, sorting will naturally arrange them in the required order. While this works, it does not take advantage of the limited value range.

### Algorithm

1. Use the built-in sort function to sort the array in ascending order.
2. The array is now sorted with all 0s first, then 1s, then 2s.

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        nums.sort()
```

### Time & Space Complexity

- Time complexity: O(nlog⁡n)*O*(*n*log*n*)
- Space complexity: O(1)*O*(1) or O(n)*O*(*n*) depending on the sorting algorithm.

------

## 2. Counting Sort

### Intuition

Since there are only three possible values (0, 1, 2), we can count how many times each appears in a single pass. Then we overwrite the array in a second pass, placing the correct number of 0s, followed by 1s, followed by 2s. This is a classic application of counting sort.

### Algorithm

1. Count the occurrences of `0`, `1`, and `2` in the array.
2. Overwrite the array:
   - Fill the first `count[0]` positions with `0`.
   - Fill the next `count[1]` positions with `1`.
   - Fill the remaining `count[2]` positions with `2`.

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        count = [0] * 3
        for num in nums:
            count[num] += 1

        index = 0
        for i in range(3):
            while count[i]:
                count[i] -= 1
                nums[index] = i
                index += 1
```

### Time & Space Complexity

- Time complexity: O(n)*O*(*n*)
- Space complexity: O(1)*O*(1)

------

## 3. Three Pointers - I

### Intuition

The Dutch National Flag algorithm partitions the array into three sections in a single pass. We maintain pointers for the boundary of 0s (left), the boundary of 2s (right), and the current element being examined. When we see a 0, we swap it to the left section. When we see a 2, we swap it to the right section. 1s naturally end up in the middle.

### Algorithm

1. Initialize three pointers: `l` (boundary for `0`s), `i` (current element), and `r` (boundary for `2`s).
2. While `i <= r`:
   - If `nums[i]` is `0`, swap with `nums[l]`, increment both `l` and `i`.
   - If `nums[i]` is `2`, swap with `nums[r]`, decrement `r` (do not increment `i` since the swapped element needs to be checked).
   - If `nums[i]` is `1`, just increment `i`.

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

### Time & Space Complexity

- Time complexity: O(n)*O*(*n*)
- Space complexity: O(1)*O*(1)

------

## 4. Three Pointers - II

### Intuition

This approach uses insertion boundaries for each color. We track where the next 0, 1, and 2 should be placed. When we encounter a value, we shift the boundaries by overwriting in a cascading manner. For example, when we see a 0, we write 2 at position `two`, then 1 at position `one`, then 0 at position `zero`, and advance all three boundaries.

### Algorithm

1. Initialize three pointers `zero`, `one`, and `two`, all starting at `0`.
2. For each element in the array:
   - If it is `0`: write `2` at `two`, write `1` at `one`, write `0` at `zero`, then increment all three pointers.
   - If it is `1`: write `2` at `two`, write `1` at `one`, then increment `two` and `one`.
   - If it is `2`: write `2` at `two`, then increment `two`.

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        zero = one = two = 0
        for i in range(len(nums)):
            if nums[i] == 0:
                nums[two] = 2
                nums[one] = 1
                nums[zero] = 0
                two += 1
                one += 1
                zero += 1
            elif nums[i] == 1:
                nums[two] = 2
                nums[one] = 1
                two += 1
                one += 1
            else:
                nums[two] = 2
                two += 1
```

### Time & Space Complexity

- Time complexity: O(n)*O*(*n*)
- Space complexity: O(1)*O*(1)

------

## 5. Three Pointers - III

### Intuition

This is a streamlined version of the previous approach. We iterate with pointer `two` and always write 2 at the current position. If the original value was less than 2, we also write 1 at position `one`. If it was less than 1 (i.e., 0), we also write 0 at position `zero`. This cascading write pattern ensures correct placement.

### Algorithm

1. Initialize pointers `zero` and `one` at `0`.
2. Iterate through the array with pointer `two`:
   - Save the current value, then set `nums[two] = 2`.
   - If the saved value was less than `2`, set `nums[one] = 1` and increment `one`.
   - If the saved value was less than `1`, set `nums[zero] = 0` and increment `zero`.

```python
class Solution:
    def sortColors(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        zero = one = 0
        for two in range(len(nums)):
            tmp = nums[two]
            nums[two] = 2
            if tmp < 2:
                nums[one] = 1
                one += 1
            if tmp < 1:
                nums[zero] = 0
                zero += 1
```

### Time & Space Complexity

- Time complexity: O(n)*O*(*n*)
- Space complexity: O(1)*O*(1)

------

## Common Pitfalls

### Incrementing the Current Pointer After Swapping with the Right Boundary

In the Dutch National Flag algorithm, when you swap the current element with the right boundary (for value 2), you must not increment the current pointer. The swapped element from the right has not been examined yet and could be 0, 1, or 2.

### Using Strict Less Than Instead of Less Than or Equal

The loop condition must be `i <= r` not `i < r`. Using strict less than causes the algorithm to miss processing the element at position `r` when `i` equals `r`, potentially leaving an unsorted element.

### Forgetting to Increment the Left Pointer After Swapping Zeros

When swapping a 0 to the left boundary, both the left boundary pointer and the current pointer must advance. The left boundary moves to make room for the next 0, and the current pointer moves because the swapped element (which came from the left) is guaranteed to be 0 or 1, both of which are already correctly positioned.