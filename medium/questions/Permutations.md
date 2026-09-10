# Permutations（全排列）

给定一个由 **互不相同的整数** 组成的数组 `nums`，返回它的所有可能排列。答案可以按 **任意顺序** 返回。

## 示例

### 示例 1

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

### 示例 2

```text
输入：nums = [7]

输出：[[7]]
```

## 约束

- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有整数互不相同

---

# 核心思路

全排列问题的本质是：

> 在每一个位置上，从还没有使用过的数字中选择一个数字，然后继续处理下一个位置。

对于长度为 `n` 的数组：

- 第 1 个位置有 `n` 种选择；
- 第 2 个位置有 `n - 1` 种选择；
- 第 3 个位置有 `n - 2` 种选择；
- ...
- 最终一共有：

\[
n! = n \times (n-1) \times \cdots \times 1
\]

种排列。

由于最终必须返回全部 `n!` 个排列，而每个排列长度为 `n`，因此仅仅构造输出就至少需要：

\[
O(n \cdot n!)
\]

的时间和空间。

这也是为什么对于本题而言，`O(n · n!)` 可以视为最优的渐进时间复杂度。

---

# 解法一：递归 + 插入

## 思路

这种方法不是逐个位置选择数字，而是从一个较小问题的所有排列出发，构造更大问题的排列。

例如要求：

```text
[1, 2, 3]
```

先递归求：

```text
[2, 3]
```

的所有排列：

```text
[2, 3]
[3, 2]
```

然后把 `1` 插入到每个排列的所有可能位置。

对于：

```text
[2, 3]
```

可以得到：

```text
[1, 2, 3]
[2, 1, 3]
[2, 3, 1]
```

对于：

```text
[3, 2]
```

可以得到：

```text
[1, 3, 2]
[3, 1, 2]
[3, 2, 1]
```

因此能够得到全部排列。

## 递归关系

假设已经知道 `nums[1:]` 的所有排列。

对于每一个较小排列，都把 `nums[0]` 插入：

```text
0, 1, 2, ..., len(permutation)
```

这些位置。

这样即可生成当前数组的全部排列。

## 算法步骤

1. 如果 `nums` 为空，返回 `[[]]`。
2. 递归求 `nums[1:]` 的所有排列。
3. 对于每一个较小排列：
   - 将 `nums[0]` 插入所有可能位置；
   - 将新排列加入结果。
4. 返回结果。

## 代码

```python
from typing import List


class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        # 空数组只有一种排列：空排列
        if len(nums) == 0:
            return [[]]

        # 先求除第一个元素之外，其余元素的所有排列
        perms = self.permute(nums[1:])

        res = []

        for p in perms:
            # 对长度为 k 的排列，一共有 k + 1 个插入位置
            for i in range(len(p) + 1):
                # 必须复制一份，避免修改原来的排列
                p_copy = p.copy()

                # 将 nums[0] 插入位置 i
                p_copy.insert(i, nums[0])

                res.append(p_copy)

        return res
```

## Python 细节

### `nums[1:]`

```python
nums[1:]
```

表示列表切片，从索引 `1` 开始复制到结尾。

例如：

```python
nums = [1, 2, 3]
nums[1:]
```

结果为：

```python
[2, 3]
```

需要注意：切片会创建一个新的列表，因此它本身需要 `O(k)` 时间和空间。

### `list.copy()`

```python
p_copy = p.copy()
```

创建列表的浅拷贝。

对于本题中的整数列表，可以理解为：

```python
p_copy = p[:]
```

两者效果基本相同。

### `list.insert(i, value)`

```python
p_copy.insert(i, nums[0])
```

表示把元素插入索引 `i`。

例如：

```python
p = [2, 3]
p.insert(1, 1)
```

得到：

```python
[2, 1, 3]
```

由于 Python 列表底层是动态数组，插入中间位置需要移动后面的元素，因此 `insert()` 最坏为 `O(n)`。

## 复杂度

### 时间复杂度

严格来看，这种“复制 + 插入”的实现会产生额外的列表复制和移动开销，因此时间复杂度可写为：

\[
O(n^2 \cdot n!)
\]

相比标准回溯的 `O(n · n!)`，它并不是最优实现。

### 空间复杂度

最终结果本身需要：

\[
O(n \cdot n!)
\]

递归深度为：

\[
O(n)
\]

因此总空间复杂度主要由输出决定：

\[
O(n \cdot n!)
\]

---

# 解法二：迭代 + 插入

## 思路

这种方法与上一种递归方法完全相同，只是不再递归，而是从空排列开始逐步构造。

初始：

```text
[[]]
```

处理 `1`：

```text
[[1]]
```

处理 `2`：

```text
[[2, 1], [1, 2]]
```

处理 `3`：

```text
[
    [3, 2, 1],
    [2, 3, 1],
    [2, 1, 3],
    [3, 1, 2],
    [1, 3, 2],
    [1, 2, 3]
]
```

每加入一个数字，就把它插入当前所有排列的所有位置。

## 算法步骤

1. 初始化：

```python
perms = [[]]
```

2. 遍历 `nums` 中的每一个数字 `num`。
3. 对当前已有的每一个排列：
   - 把 `num` 插入所有位置；
   - 得到下一轮排列。
4. 用新排列替换旧排列。
5. 返回最终结果。

## 代码

```python
from typing import List


class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        # 初始状态：空数组只有一个空排列
        perms = [[]]

        for num in nums:
            new_perms = []

            for p in perms:
                # num 可以插入到 0 ~ len(p) 共 len(p) + 1 个位置
                for i in range(len(p) + 1):
                    p_copy = p.copy()
                    p_copy.insert(i, num)
                    new_perms.append(p_copy)

            # 当前数字处理完后，进入下一轮
            perms = new_perms

        return perms
```

## 复杂度

由于同样频繁执行列表复制和 `insert()`，时间复杂度可以写为：

\[
O(n^2 \cdot n!)
\]

空间复杂度主要是最终答案：

\[
O(n \cdot n!)
\]

---

# 解法三：标准回溯

## 思路

这是全排列问题最经典、也最值得在面试中掌握的方法。

维护三个状态：

- `perm`：当前正在构造的排列；
- `used[i]`：`nums[i]` 是否已经使用；
- `res`：所有完整排列。

每一层递归决定：

> 当前这个位置放哪个还没有使用的数字？

例如：

```text
nums = [1, 2, 3]
```

决策树大致为：

```text
                  []
          /        |        \
        [1]       [2]       [3]
       /   \     /   \     /   \
   [1,2] [1,3] ...         ...
      |      |
 [1,2,3] [1,3,2]
```

走到一个完整排列之后，就将其保存。

然后撤销刚才的选择，继续尝试其他数字。

这就是：

> 选择 → 递归 → 撤销选择

也就是回溯算法最核心的模板。

## 算法步骤

1. 创建：
   - `perm = []`
   - `used = [False] * len(nums)`
   - `res = []`
2. 如果 `len(perm) == len(nums)`：
   - 当前已经是一个完整排列；
   - 保存一个副本。
3. 遍历所有下标 `i`：
   - 如果 `used[i]` 为 `False`：
     - 选择 `nums[i]`；
     - 标记为已使用；
     - 递归；
     - 删除刚刚加入的数字；
     - 恢复未使用状态。
4. 返回结果。

## 代码

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

---

# 如何理解回溯

很多回溯题都可以抽象成下面的模板：

```python
def backtrack(state):
    if 满足结束条件:
        保存答案
        return

    for choice in 所有可能选择:
        if choice 不合法:
            continue

        做选择
        backtrack(新的状态)
        撤销选择
```

本题中：

```text
state  = 当前排列
choice = 选择一个还没有使用过的 nums[i]
```

所以代码核心实际上就是：

```python
perm.append(nums[i])
used[i] = True

backtrack(...)

perm.pop()
used[i] = False
```

这四行代码非常值得熟悉。

---

# 为什么保存答案时一定要复制？

错误写法：

```python
self.res.append(perm)
```

正确写法：

```python
self.res.append(perm.copy())
```

或者：

```python
self.res.append(perm[:])
```

原因是 `perm` 在整个回溯过程中始终是同一个列表对象。

例如找到：

```text
[1, 2, 3]
```

之后，程序还会继续执行：

```python
perm.pop()
```

如果结果列表保存的是 `perm` 本身，那么之前存进去的内容也会跟着变化。

因此必须保存当前状态的一个副本。

---

# 复杂度

## 时间复杂度

总共有：

\[
n!
\]

个排列。

每个完整排列复制到结果中需要：

\[
O(n)
\]

因此：

\[
O(n \cdot n!)
\]

这是生成所有排列时的渐进最优复杂度。

## 空间复杂度

如果包括返回结果：

\[
O(n \cdot n!)
\]

如果只计算算法额外使用的辅助空间：

- `perm`：`O(n)`
- `used`：`O(n)`
- 递归调用栈：`O(n)`

因此额外空间为：

\[
O(n)
\]

---

# 解法四：回溯 + 位掩码 Bitmask

## 思路

这个方法和标准回溯完全一样。

唯一的区别是：

标准版本使用：

```python
used = [False] * n
```

记录哪些数字被使用。

Bitmask 版本使用一个整数：

```python
mask
```

把整数的每一位当作一个布尔值。

例如有 4 个元素：

```text
index:  3 2 1 0
mask:   0 1 0 1
```

二进制：

```text
0101
```

表示：

- 第 `0` 个元素已经使用；
- 第 `1` 个元素没有使用；
- 第 `2` 个元素已经使用；
- 第 `3` 个元素没有使用。

---

# 位运算说明

这是这一解法中最重要的 Python / 位运算知识。

## `1 << i`

```python
1 << i
```

表示把二进制 `1` 左移 `i` 位。

例如：

```text
1 << 0 = 0001
1 << 1 = 0010
1 << 2 = 0100
1 << 3 = 1000
```

因此：

```python
1 << i
```

可以用来得到一个只有第 `i` 位为 `1` 的整数。

---

## 检查第 i 位是否已经被使用

```python
mask & (1 << i)
```

如果结果不为 `0`，说明第 `i` 位已经是 `1`，也就是已经使用。

因此：

```python
if not (mask & (1 << i)):
```

表示：

> 如果第 `i` 个元素还没有被使用。

---

## 把第 i 位设置为 1

```python
mask | (1 << i)
```

例如：

```text
mask       = 0101
1 << 1     = 0010
----------------
OR          0111
```

于是第 `1` 位被设置为了 `1`。

---

# 代码

```python
from typing import List


class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        self.res = []

        # mask = 0 表示当前还没有使用任何元素
        self.backtrack(nums, [], 0)

        return self.res

    def backtrack(
        self,
        nums: List[int],
        perm: List[int],
        mask: int
    ) -> None:

        if len(perm) == len(nums):
            self.res.append(perm.copy())
            return

        for i in range(len(nums)):
            # 如果 mask 的第 i 位已经是 1，说明 nums[i] 已使用
            if mask & (1 << i):
                continue

            # 选择 nums[i]
            perm.append(nums[i])

            # 使用 OR 将第 i 位设置为 1，然后进入下一层
            self.backtrack(
                nums,
                perm,
                mask | (1 << i)
            )

            # 回溯
            perm.pop()
```

## 为什么这里不需要恢复 `mask`？

标准回溯中要写：

```python
used[i] = True
...
used[i] = False
```

但这里不需要对 `mask` 做类似的撤销。

原因是 Python 中整数是不可变对象。

递归时：

```python
mask | (1 << i)
```

会计算出一个新的整数并传给下一层。

当前这一层原来的 `mask` 不会被修改。

因此返回以后，原来的 `mask` 自然还是原来的状态。

---

# 复杂度

时间复杂度：

\[
O(n \cdot n!)
\]

如果包括输出：

\[
O(n \cdot n!)
\]

如果只计算辅助空间：

- 当前排列：`O(n)`
- 递归栈：`O(n)`
- `mask`：`O(1)`（在本题规模下可以视为常数）

因此辅助空间：

\[
O(n)
\]

---

# 解法五：原地交换回溯

## 思路

这是非常经典的另一种全排列写法。

与前两个回溯方法不同，它不需要：

```python
used
```

也不需要：

```python
mask
```

而是直接在原数组上交换元素。

假设正在处理位置：

```python
idx
```

可以把数组理解成两部分：

```text
[ 已确定的部分 | 尚未确定的部分 ]
 0 ... idx-1     idx ... n-1
```

例如：

```text
nums = [1, 2, 3]
idx = 1
```

表示：

```text
[1 | 2, 3]
```

`nums[0]` 已经确定。

接下来只需要决定：

```text
nums[1]
```

应该放 `2` 还是 `3`。

做法就是让：

```python
i = idx ... n - 1
```

分别与：

```python
nums[idx]
```

交换。

---

# 交换过程示例

对于：

```text
[1, 2, 3]
```

开始：

```text
idx = 0
```

选择 `1`：

```text
swap(0, 0)

[1, 2, 3]
```

进入下一层：

```text
idx = 1
```

再选择：

```text
swap(1, 1)
→ [1, 2, 3]
```

得到一个排列。

之后回退到：

```text
idx = 1
```

再选择：

```text
swap(1, 2)
→ [1, 3, 2]
```

于是得到另一个排列。

---

# 代码

```python
from typing import List


class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        self.res = []

        self.backtrack(nums, 0)

        return self.res

    def backtrack(self, nums: List[int], idx: int) -> None:
        # idx == len(nums)：
        # 所有位置都已经确定
        if idx == len(nums):
            self.res.append(nums.copy())
            return

        # 尝试把后面的每一个数字放到 idx 位置
        for i in range(idx, len(nums)):
            # 做选择：
            # 把 nums[i] 放到当前 idx 位置
            nums[idx], nums[i] = nums[i], nums[idx]

            # 继续确定下一个位置
            self.backtrack(nums, idx + 1)

            # 撤销选择：
            # 恢复数组原来的状态
            nums[idx], nums[i] = nums[i], nums[idx]
```

---

# Python 多变量交换

Python 可以直接写：

```python
a, b = b, a
```

所以交换列表两个位置时：

```python
nums[idx], nums[i] = nums[i], nums[idx]
```

不需要显式创建临时变量。

等价于其他语言中常见的：

```text
temp = nums[idx]
nums[idx] = nums[i]
nums[i] = temp
```

---

# 为什么交换之后必须再交换回来？

考虑：

```python
nums[idx], nums[i] = nums[i], nums[idx]
```

递归结束之后，当前数组已经被修改。

如果不恢复，下一次循环就不是从原来的状态开始，整个搜索树会被破坏。

因此回溯过程始终要求：

```text
做选择
↓
递归
↓
撤销选择
```

这里：

```python
nums[idx], nums[i] = nums[i], nums[idx]
```

既是“做选择”，也是之后的“撤销选择”。

---

# 复杂度

时间复杂度：

\[
O(n \cdot n!)
\]

虽然交换本身是 `O(1)`，但每找到一个完整排列，仍然必须执行：

```python
nums.copy()
```

复制长度为 `n` 的结果。

因此整体仍然是：

\[
O(n \cdot n!)
\]

如果包括输出空间：

\[
O(n \cdot n!)
\]

如果只计算辅助空间：

- 递归调用栈深度：`O(n)`
- 没有 `used` 数组

因此：

\[
O(n)
\]

这里有时会被称为“原地回溯”，因为搜索过程中直接复用 `nums`。

需要注意：

> “原地”并不意味着整体空间复杂度是 `O(1)`。

递归栈仍然需要 `O(n)` 空间，而且返回的结果仍然占据 `O(n · n!)` 空间。

---

# 五种方法对比

| 方法 | 核心思想 | 时间复杂度 | 辅助空间 | 面试推荐度 |
|---|---|---:|---:|---|
| 递归 + 插入 | 先求小问题排列，再插入新元素 | `O(n² · n!)` | `O(n)` + 输出 | 一般 |
| 迭代 + 插入 | 逐步把新元素插入已有排列 | `O(n² · n!)` | 中间结果较多 | 一般 |
| 标准回溯 | `used` 数组记录已使用元素 | `O(n · n!)` | `O(n)` | **非常推荐** |
| 回溯 + Bitmask | 用整数位代替 `used` | `O(n · n!)` | `O(n)` | 推荐 |
| 原地交换回溯 | 交换元素确定当前位置 | `O(n · n!)` | `O(n)` | **非常推荐** |

---

# 面试中推荐掌握的两个版本

如果面试时间有限，优先熟练掌握下面两个版本即可。

## 版本一：`used` 数组

优点：

- 最直观；
- 与组合、子集、DFS 等其他回溯题思路一致；
- 很容易扩展到更加复杂的条件。

核心模板：

```python
for i in range(len(nums)):
    if used[i]:
        continue

    perm.append(nums[i])
    used[i] = True

    backtrack(...)

    perm.pop()
    used[i] = False
```

## 版本二：交换回溯

优点：

- 不需要额外的 `used` 数组；
- 代码简洁；
- 很适合纯全排列问题。

核心模板：

```python
for i in range(idx, len(nums)):
    nums[idx], nums[i] = nums[i], nums[idx]

    backtrack(nums, idx + 1)

    nums[idx], nums[i] = nums[i], nums[idx]
```

---

# 常见错误

## 1. 保存引用而不是副本

错误：

```python
res.append(perm)
```

正确：

```python
res.append(perm.copy())
```

或者：

```python
res.append(perm[:])
```

因为回溯过程中会不断修改 `perm`。

---

## 2. 忘记回溯

例如：

```python
perm.append(nums[i])
used[i] = True

backtrack(...)
```

如果后面忘记：

```python
perm.pop()
used[i] = False
```

那么下一条搜索路径会继承上一条路径的状态，从而漏掉大量排列。

---

## 3. 用 `nums[i] in perm` 判断是否已经使用

可以写：

```python
if nums[i] in perm:
    continue
```

但一般不推荐。

原因是：

```python
nums[i] in perm
```

需要线性扫描当前列表，单次判断为：

\[
O(n)
\]

于是会引入额外开销。

更推荐：

```python
used[i]
```

因为布尔数组查询是：

\[
O(1)
\]

或者使用 Bitmask：

```python
mask & (1 << i)
```

也是常数时间判断。

---

## 4. 混淆“输出空间”和“辅助空间”

全排列一定会产生：

\[
n!
\]

个结果。

每个结果长度为：

\[
n
\]

所以返回结果本身必然需要：

\[
O(n \cdot n!)
\]

空间。

因此分析空间复杂度时最好明确区分：

### 包括输出

```text
O(n · n!)
```

### 不包括输出，只看辅助空间

标准回溯：

```text
O(n)
```

交换回溯：

```text
O(n)
```

---

# 一个重要的复杂度结论

对于“返回所有排列”这种问题，不能只写：

\[
O(n!)
\]

因为每个排列包含 `n` 个元素。

为了真正构造并保存一个排列，需要至少：

\[
O(n)
\]

时间。

共有：

\[
n!
\]

个排列。

所以最优渐进时间复杂度是：

\[
\boxed{O(n \cdot n!)}
\]

同理，返回结果本身的空间复杂度也是：

\[
\boxed{O(n \cdot n!)}
\]

---

# 面试总结

全排列是非常典型的回溯题。

最重要的是理解：

```text
当前位置有哪些选择？
        ↓
选择一个还没有使用过的数字
        ↓
递归处理下一个位置
        ↓
撤销选择
```

对应标准代码结构：

```python
for choice in choices:
    if choice 已经使用:
        continue

    做选择
    backtrack()
    撤销选择
```

对于本题，最推荐记住：

```python
perm.append(nums[i])
used[i] = True

backtrack(...)

perm.pop()
used[i] = False
```

如果已经熟悉回溯，还可以进一步掌握“原地交换”的写法：

```python
nums[idx], nums[i] = nums[i], nums[idx]

backtrack(nums, idx + 1)

nums[idx], nums[i] = nums[i], nums[idx]
```

这两种写法都能够以：

\[
O(n \cdot n!)
\]

的最优时间复杂度生成全部排列。
