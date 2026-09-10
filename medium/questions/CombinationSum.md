# 组合总和（Combination Sum）

给定一个由不同整数组成的数组 `nums` 和一个目标整数 `target`，请返回所有和等于 `target` 的唯一组合。

`nums` 中的同一个数字可以被无限次选取。如果两个组合中每个数字出现的次数都相同，则认为它们是同一个组合。

结果可以按任意顺序返回，每个组合中的数字也可以按任意顺序排列。

> 以下算法要求 `nums` 中的元素均为正整数。若包含 `0` 或负数，由于每个数字可以无限次使用，搜索过程可能无法终止。

------

## 方法一：选择或跳过的回溯

### 核心思路

从左到右处理数组中的每个数字。对于当前数字 `nums[i]`，有两种选择：

1. 选择当前数字：因为数字可以重复使用，所以递归时仍然停留在索引 `i`。
2. 跳过当前数字：递归到索引 `i + 1`，之后不再使用当前数字。

这种搜索顺序能够避免重复组合。例如，它只会生成 `[2, 3]`，不会再生成顺序不同但含义相同的 `[3, 2]`。

当当前总和：

- 等于 `target`：记录当前组合；
- 大于 `target`：停止搜索；
- 尚未达到目标但已用完所有数字：停止搜索。

### 算法步骤

1. 定义递归函数 `dfs(i, current, total)`：
   - `i` 表示当前处理的数字下标；
   - `current` 表示正在构造的组合；
   - `total` 表示当前组合的元素总和。
2. 如果 `total == target`，将当前组合的副本加入结果。
3. 如果 `total > target` 或 `i` 越界，结束当前搜索分支。
4. 将 `nums[i]` 加入组合，并从下标 `i` 继续递归。
5. 撤销刚才的选择。
6. 跳过 `nums[i]`，从下标 `i + 1` 继续递归。

### 代码

```
from typing import List


class Solution:
    def combinationSum(
        self, nums: List[int], target: int
    ) -> List[List[int]]:
        result: List[List[int]] = []

        def dfs(i: int, current: List[int], total: int) -> None:
            # 找到一个合法组合
            if total == target:
                # 必须保存副本，否则后续回溯会修改原列表
                result.append(current.copy())
                return

            # 数字已经用完，或者当前总和已经超过目标
            if i >= len(nums) or total > target:
                return

            # 选择 nums[i]
            current.append(nums[i])

            # 仍然从 i 开始，因为 nums[i] 可以重复使用
            dfs(i, current, total + nums[i])

            # 撤销选择，恢复递归前的状态
            current.pop()

            # 跳过 nums[i]，以后不再选择这个数字
            dfs(i + 1, current, total)

        dfs(0, [], 0)
        return result
```

### 复杂度分析

令：

- `n = len(nums)`；
- `a = min(nums)`；
- `L = target // a`，即一个组合可能达到的最大长度；
- `S` 为最终答案中组合的数量。

递归最多连续选择 `L` 个数字，同时还需要处理最多 `n` 次“跳过数字”的操作。

- 时间复杂度：可用较宽松的上界表示为
  `O(2^(n + L) + S × L)`
  其中 `S × L` 是复制所有答案所需要的时间。
- 辅助空间复杂度：`O(n + L)`，用于递归调用栈和当前组合。
- 结果空间复杂度：`O(S × L)`。

实际运行时间通常远小于这个宽松上界，因为总和超过 `target` 时会立即剪枝。

------

## 方法二：排序、枚举与剪枝

### 核心思路

先将 `nums` 排序，然后在每一层递归中枚举可以选择的数字。

假设当前递归从索引 `start` 开始：

- 只枚举 `start` 及其之后的数字，从而避免产生不同顺序的重复组合；
- 选择 `nums[j]` 后，下一层仍从 `j` 开始，因为该数字可以重复使用；
- 如果加入 `nums[j]` 后总和超过 `target`，由于数组已经排序，后面的数字只会更大，因此可以立即结束当前循环。

与方法一相比，这种写法通常更直观，并且排序后能够更早剪枝。

### 算法步骤

1. 对 `nums` 进行升序排序。
2. 定义递归函数 `dfs(start, current, total)`。
3. 如果 `total == target`，保存当前组合的副本。
4. 从 `start` 开始枚举候选数字：
   - 如果加入当前数字后超过目标值，结束循环；
   - 将当前数字加入组合；
   - 从当前下标继续递归，允许重复使用该数字；
   - 回溯，移除刚加入的数字。
5. 从 `dfs(0, [], 0)` 开始搜索。

### 代码

```
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

这里使用 `break` 比 `return` 更能准确表达“结束当前循环”的意图。在这段代码中两者结果相同，因为循环之后没有其他逻辑。

### 复杂度分析

令 `L = target // min(nums)`，即组合的最大可能长度。

- 排序时间复杂度：`O(n log n)`。
- 回溯时间复杂度：可使用宽松上界 `O(n^L)` 表示。
- 复制答案的时间：`O(S × L)`。
- 辅助空间复杂度：`O(L)`，主要来自递归栈和当前组合。
- 结果空间复杂度：`O(S × L)`。

因此总时间复杂度可以写为：

```
O(n log n + n^L + S × L)
```

回溯算法的实际复杂度与输入数字、目标值以及合法组合数量密切相关，因此通常使用搜索树规模或输出相关复杂度进行描述。

------

## 为什么不会产生重复组合？

关键限制是：每次递归只允许从当前下标向后选择数字。

例如：

```
nums = [2, 3, 6, 7]
```

如果当前已经选择了 `3`，后续只能继续选择 `3、6、7`，不能回头选择 `2`。

因此算法可能生成：

```
[2, 2, 3]
```

但不会再生成：

```
[2, 3, 2]
[3, 2, 2]
```

这几个组合本质相同，只会保留其中一种规范顺序。

------

## 常见错误

### 1. 选择一个数字后错误地移动到下一下标

因为每个数字可以无限次使用，所以选择 `nums[i]` 后应该继续停留在 `i`。

```
# 错误：选择后移动到 i + 1，导致当前数字无法重复使用
current.append(nums[i])
dfs(i + 1, current, total + nums[i])
```

正确写法：

```
# 正确：继续从 i 开始，允许重复选择 nums[i]
current.append(nums[i])
dfs(i, current, total + nums[i])
```

在枚举形式的回溯中也是同样的道理：

```
# j 而不是 j + 1
dfs(j, current, total + nums[j])
```

如果题目规定每个数字只能使用一次，才应该递归到 `j + 1`。

### 2. 没有限制下一次搜索的起始下标

如果每次都从索引 `0` 开始枚举，可能同时生成 `[2, 3]` 和 `[3, 2]`。

```
# 容易生成不同顺序的重复组合
for j in range(len(nums)):
    ...
```

应该从当前起始位置向后枚举：

```
# 每个组合按照固定的下标顺序生成
for j in range(start, len(nums)):
    ...
```

### 3. 保存当前列表时没有复制

下面的写法保存的是同一个列表对象：

```
# 错误
result.append(current)
```

回溯过程中 `current` 会不断被修改，已经保存的答案也会随之改变。

正确写法：

```
result.append(current.copy())
```

也可以写成：

```
result.append(current[:])
```

### 4. 忘记回溯

加入一个数字并完成递归后，必须删除刚才加入的数字：

```
current.append(nums[j])
dfs(j, current, total + nums[j])
current.pop()
```

`list.pop()` 会删除并返回列表的最后一个元素。这里使用它来恢复递归前的状态。

### 5. 未在总和超过目标时剪枝

如果当前总和已经超过目标值，继续递归无法得到合法结果。

未排序时可以这样剪枝：

```
if total > target:
    return
```

排序后可以进行更强的剪枝：

```
for j in range(start, len(nums)):
    if total + nums[j] > target:
        break
```

因为数组已经升序排列，当前数字过大时，后面的数字也一定过大。

------

## Python 相关说明

### `List[int]`

`List[int]` 是类型注解，表示“元素均为整数的列表”。

```
from typing import List

nums: List[int] = [2, 3, 6, 7]
```

在 Python 3.9 及以上版本中，也可以直接写：

```
nums: list[int] = [2, 3, 6, 7]
```

类型注解通常不会影响程序运行，但可以提高代码可读性，并帮助编辑器和类型检查工具发现错误。

### `list.append(value)`

将一个元素添加到列表末尾：

```
current.append(nums[j])
```

时间复杂度通常为均摊 `O(1)`。

### `list.pop()`

删除并返回列表的最后一个元素：

```
current.pop()
```

从末尾删除元素的时间复杂度为 `O(1)`，因此很适合用于回溯。

### `list.copy()`

创建列表的浅拷贝：

```
result.append(current.copy())
```

复制长度为 `k` 的列表需要 `O(k)` 时间和空间。

------

## 总结

这道题的回溯设计有三个关键点：

1. 选择当前数字后停留在当前下标，从而允许重复使用。
2. 下一次只从当前下标向后搜索，从而避免重复组合。
3. 当总和超过目标值时立即停止搜索；排序后还可以提前结束整个枚举循环。

实际面试中更推荐使用第二种“排序 + 枚举 + 剪枝”的写法，因为结构更简洁，去重逻辑也更直观。