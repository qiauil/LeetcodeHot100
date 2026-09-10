# 分割回文串（Palindrome Partitioning）

给定一个字符串 `s`，将它分割成若干个子串，使得分割后的每个子串都是回文串。

返回所有可能的回文串分割方案，答案可以按任意顺序排列。

回文串是指正着读和倒着读都相同的字符串，例如 `"a"`、`"aba"`、`"abba"`。

## 示例

```
输入：s = "aab"

输出：
[
    ["a", "a", "b"],
    ["aa", "b"]
]
```

------

## 解法一：双下标回溯

### 思路

可以把分割字符串理解成在字符之间放置“切割线”。

使用两个下标：

- `j`：当前子串的起始位置。
- `i`：当前尝试的子串的结束位置。

当前正在检查的子串是 `s[j:i+1]`。

每一步有两种可能：

1. 如果 `s[j:i+1]` 是回文串，就在 `i` 后面切一刀：
   - 将该子串加入当前方案。
   - 从 `i + 1` 开始寻找下一个子串。
2. 暂时不切割，让 `i` 向右移动，尝试更长的子串。

选择一个回文子串后，需要在递归返回时撤销选择，这就是回溯。

### 算法步骤

1. 使用 `part` 保存当前分割方案。
2. 使用 `res` 保存全部合法方案。
3. 定义 `dfs(j, i)`：
   - `j` 是当前子串的起点。
   - `i` 是当前子串正在尝试的终点。
4. 如果 `i == len(s)`：
   - 当 `j == i` 时，说明字符串恰好被完整分割，将 `part` 的副本加入结果。
   - 结束当前递归。
5. 如果 `s[j:i+1]` 是回文串：
   - 选择它。
   - 从 `i + 1` 开始递归。
   - 撤销选择。
6. 将 `i` 向右移动，尝试更长的子串。

### 代码

```
from typing import List


class Solution:
    def partition(self, s: str) -> List[List[str]]:
        res: List[List[str]] = []
        part: List[str] = []

        def dfs(start: int, end: int) -> None:
            # end 已经越过字符串末尾
            if end >= len(s):
                # start == end 表示字符串恰好被完整分割
                if start == end:
                    res.append(part.copy())
                return

            # 如果 s[start:end+1] 是回文串，可以在 end 后切割
            if self.is_palindrome(s, start, end):
                part.append(s[start:end + 1])

                # 从下一个位置开始构造新的子串
                dfs(end + 1, end + 1)

                # 回溯：撤销刚才选择的子串
                part.pop()

            # 暂时不切割，继续扩大当前子串
            dfs(start, end + 1)

        dfs(0, 0)
        return res

    def is_palindrome(self, s: str, left: int, right: int) -> bool:
        """使用双指针判断 s[left:right+1] 是否为回文串。"""
        while left < right:
            if s[left] != s[right]:
                return False

            left += 1
            right -= 1

        return True
```

### 复杂度分析

设字符串长度为 `n`。

- 时间复杂度：`O(n · 2^n)`
  - 最多存在 `2^(n-1)` 种切割方式。
  - 回文判断和构造答案都可能需要 `O(n)` 时间。
- 辅助空间复杂度：`O(n)`
  - 主要来自递归调用栈和当前分割方案。
- 结果空间复杂度：最坏为 `O(n · 2^n)`。

这个写法能够正确解决问题，但两个递归下标的含义稍微复杂。面试中通常更推荐下面的标准回溯写法。

------

## 解法二：标准回溯

### 思路

从左到右构造分割方案。

假设当前需要处理的位置是 `start`，我们枚举下一段子串的结束位置 `end`：

```
s[start:end+1]
```

如果该子串是回文串，就可以把它作为当前分割方案的下一部分，然后继续处理从 `end + 1` 开始的剩余字符串。

例如，对于 `"aab"`：

```
从下标 0 开始：

选择 "a"
    选择 "a"
        选择 "b"
            得到 ["a", "a", "b"]

选择 "aa"
    选择 "b"
        得到 ["aa", "b"]
```

### 回溯模板

整个过程可以概括为：

```
枚举选择
    如果选择合法：
        做出选择
        递归处理剩余部分
        撤销选择
```

### 算法步骤

1. 定义 `dfs(start)`，表示从 `start` 开始分割剩余字符串。
2. 如果 `start == len(s)`：
   - 说明整个字符串已经分割完成。
   - 将当前方案的副本加入结果。
3. 枚举结束位置 `end`。
4. 如果 `s[start:end+1]` 是回文串：
   - 将它加入当前方案。
   - 递归处理 `end + 1` 后面的字符串。
   - 删除刚加入的子串，继续尝试其他选择。

### 代码

```
from typing import List


class Solution:
    def partition(self, s: str) -> List[List[str]]:
        res: List[List[str]] = []
        part: List[str] = []

        def dfs(start: int) -> None:
            # start 到达字符串末尾，得到一个完整分割方案
            if start == len(s):
                # 必须保存副本，因为 part 后续还会被修改
                res.append(part.copy())
                return

            # 枚举当前子串的结束位置
            for end in range(start, len(s)):
                if self.is_palindrome(s, start, end):
                    # 做出选择
                    part.append(s[start:end + 1])

                    # 递归处理剩余字符串
                    dfs(end + 1)

                    # 撤销选择
                    part.pop()

        dfs(0)
        return res

    def is_palindrome(self, s: str, left: int, right: int) -> bool:
        """判断闭区间 s[left:right+1] 是否为回文串。"""
        while left < right:
            if s[left] != s[right]:
                return False

            left += 1
            right -= 1

        return True
```

### 复杂度分析

- 时间复杂度：`O(n · 2^n)`
- 辅助空间复杂度：`O(n)`
- 结果空间复杂度：最坏为 `O(n · 2^n)`

### Python 语法说明

#### `range(start, len(s))`

生成从 `start` 到 `len(s) - 1` 的整数，用来枚举子串的结束位置。

#### `s[start:end + 1]`

Python 切片采用左闭右开区间：

```
s[start:end + 1]
```

表示取出下标区间 `[start, end]` 的字符。

#### `part.copy()`

创建 `part` 的浅拷贝。

这里的元素都是不可变字符串，因此浅拷贝已经足够。如果直接写：

```
res.append(part)
```

保存的只是同一个列表对象的引用。之后的 `append()` 和 `pop()` 会修改该列表，导致结果错误。

#### `part.pop()`

删除列表的最后一个元素，用于撤销刚才做出的选择，这是回溯算法的关键操作。

------

## 解法三：动态规划预处理 + 回溯

### 思路

标准回溯会多次判断同一个子串是不是回文串。

例如，双指针判断一次回文串需要 `O(n)` 时间。为了避免重复检查，可以先使用动态规划预处理所有子串。

定义：

```
dp[i][j] = s[i:j+1] 是否为回文串
```

判断一个子串是否为回文串时，有如下状态转移：

```
dp[i][j] =
    s[i] == s[j]
    并且
    （子串长度不超过 2，或者 dp[i+1][j-1] 为 True）
```

也就是：

```
dp[i][j] = (
    s[i] == s[j]
    and (j - i <= 1 or dp[i + 1][j - 1])
)
```

预处理完成后，回溯中只需要查询 `dp[start][end]`，每次回文判断都变成 `O(1)`。

### 为什么要按子串长度递增计算

计算 `dp[i][j]` 时，可能依赖内部子串：

```
dp[i + 1][j - 1]
```

内部子串的长度比当前子串短，因此应当先计算短子串，再计算长子串。

### 代码

```
from typing import List


class Solution:
    def partition(self, s: str) -> List[List[str]]:
        n = len(s)

        # dp[i][j] 表示 s[i:j+1] 是否为回文串
        dp = [[False] * n for _ in range(n)]

        # 按照子串长度从短到长计算
        for length in range(1, n + 1):
            for left in range(n - length + 1):
                right = left + length - 1

                # 首尾字符相同，并且：
                # 1. 长度为 1 或 2；或者
                # 2. 内部子串也是回文串
                if s[left] == s[right]:
                    if length <= 2 or dp[left + 1][right - 1]:
                        dp[left][right] = True

        res: List[List[str]] = []
        part: List[str] = []

        def dfs(start: int) -> None:
            if start == n:
                res.append(part.copy())
                return

            for end in range(start, n):
                # O(1) 查询当前子串是否为回文串
                if dp[start][end]:
                    part.append(s[start:end + 1])
                    dfs(end + 1)
                    part.pop()

        dfs(0)
        return res
```

### 复杂度分析

- DP 预处理时间：`O(n²)`
- 回溯及构造答案：`O(n · 2^n)`
- 总时间复杂度：`O(n² + n · 2^n)`，通常简写为 `O(n · 2^n)`
- 辅助空间复杂度：`O(n²)`
  - DP 表占用 `O(n²)`。
  - 递归栈和当前方案占用 `O(n)`。
- 结果空间复杂度：最坏为 `O(n · 2^n)`。

### Python 二维列表说明

下面是创建二维布尔列表的常用写法：

```
dp = [[False] * n for _ in range(n)]
```

不要写成：

```
dp = [[False] * n] * n
```

后一种写法会让每一行都引用同一个内部列表。修改其中一行时，其他行也会一起改变。

------

## 解法四：动态规划预处理 + 返回式递归

### 思路

前三种写法通过共享的 `part` 和 `res` 保存当前状态。

另一种方式是让每次递归直接返回答案：

```
从位置 i 开始的全部分割方案
=
每一个合法的回文前缀
+
剩余字符串的全部分割方案
```

定义：

```
dfs(i)
```

返回从下标 `i` 开始的所有回文分割方案。

如果选择 `s[i:j+1]` 作为第一段，那么剩余答案由 `dfs(j + 1)` 提供。将当前子串添加到每一个后续方案的前面即可。

### 为什么递归终点返回 `[[]]`

当 `i == n` 时，没有剩余字符串需要分割。

此时需要返回：

```
[[]]
```

它表示“存在一种合法的空分割方案”。

这样，上一层才能执行：

```
[current_substring] + []
```

从而形成一个完整答案。

如果返回空列表 `[]`，上一层就没有任何方案可以组合，最终将得不到结果。

### 代码

```
from typing import List


class Solution:
    def partition(self, s: str) -> List[List[str]]:
        n = len(s)

        # 预处理所有回文子串
        dp = [[False] * n for _ in range(n)]

        for length in range(1, n + 1):
            for left in range(n - length + 1):
                right = left + length - 1

                dp[left][right] = (
                    s[left] == s[right]
                    and (
                        length <= 2
                        or dp[left + 1][right - 1]
                    )
                )

        def dfs(start: int) -> List[List[str]]:
            # 一种合法的空方案，用于和上一层进行组合
            if start == n:
                return [[]]

            result: List[List[str]] = []

            for end in range(start, n):
                if not dp[start][end]:
                    continue

                current = s[start:end + 1]

                # 获取剩余字符串的全部分割方案
                suffix_partitions = dfs(end + 1)

                # 将当前回文串添加到每个后续方案之前
                for suffix in suffix_partitions:
                    result.append([current] + suffix)

            return result

        return dfs(0)
```

### 复杂度分析

- DP 预处理时间：`O(n²)`
- 构造全部答案：最坏为 `O(n · 2^n)`
- 总时间复杂度：`O(n² + n · 2^n)`
- 辅助空间复杂度：`O(n² + n)`
- 结果空间复杂度：最坏为 `O(n · 2^n)`

这段代码没有对 `dfs(start)` 的结果进行缓存，因此某些相同的后缀可能被重复求解。可以使用 `functools.cache` 进一步减少重复递归。

------

## 解法五：记忆化递归

这是对解法四的小幅改进。

因为 `dfs(start)` 的结果只取决于 `start`，所以可以缓存已经计算过的结果。

### 代码

```
from functools import cache
from typing import List, Tuple


class Solution:
    def partition(self, s: str) -> List[List[str]]:
        n = len(s)

        # 预处理所有回文子串
        dp = [[False] * n for _ in range(n)]

        for left in range(n - 1, -1, -1):
            for right in range(left, n):
                dp[left][right] = (
                    s[left] == s[right]
                    and (
                        right - left <= 1
                        or dp[left + 1][right - 1]
                    )
                )

        @cache
        def dfs(start: int) -> Tuple[Tuple[str, ...], ...]:
            if start == n:
                return ((),)

            result = []

            for end in range(start, n):
                if dp[start][end]:
                    current = s[start:end + 1]

                    for suffix in dfs(end + 1):
                        result.append((current,) + suffix)

            # tuple 是不可变对象，适合用作缓存结果
            return tuple(result)

        # 按照题目要求，将 tuple 转换回 List[List[str]]
        return [list(partition) for partition in dfs(0)]
```

### `@cache` 说明

`cache` 来自 Python 标准库：

```
from functools import cache
```

它会根据函数参数缓存返回值。再次使用相同参数调用函数时，可以直接返回之前的计算结果。

缓存的数据使用元组 `tuple`，因为元组是不可变对象，更适合作为记忆化递归中的共享结果。

不过，本题必须输出所有分割方案，答案本身的规模在最坏情况下就是指数级，因此记忆化无法消除输出所需的指数级时间和空间。

------

## 常见错误

### 1. 将 `part` 本身加入结果

错误写法：

```
res.append(part)
```

`part` 在后续回溯中还会被不断修改，结果列表中的所有元素可能指向同一个列表对象。

正确写法：

```
res.append(part.copy())
```

也可以写成：

```
res.append(part[:])
```

------

### 2. 忘记撤销选择

错误写法：

```
part.append(s[start:end + 1])
dfs(end + 1)
```

正确写法：

```
part.append(s[start:end + 1])
dfs(end + 1)
part.pop()
```

如果没有执行 `pop()`，其他递归分支会包含之前分支遗留下来的子串。

------

### 3. 递归终止条件错误

标准回溯的正确终止条件是：

```
if start == len(s):
```

它表示整个字符串恰好被分割完成。

写成：

```
if start > len(s):
```

会漏掉合法答案，因为递归通常只会恰好到达 `len(s)`，不会超过它。

------

### 4. 混淆切片的开闭区间

如果 `end` 是子串最后一个字符的下标，应当写：

```
s[start:end + 1]
```

而不是：

```
s[start:end]
```

因为 Python 切片不包含右端点。

------

### 5. 回文判断的边界条件错误

双指针判断应使用：

```
while left < right:
```

当 `left == right` 时，中间字符不需要比较。单个字符天然是回文串。

------

### 6. DP 表的填充顺序错误

状态 `dp[left][right]` 依赖：

```
dp[left + 1][right - 1]
```

因此必须保证内部的短子串先被计算。

可以：

- 按子串长度从短到长遍历。
- 或让 `left` 从右向左遍历。

------

## 面试建议

一般优先写“解法二：标准回溯”，因为它：

- 逻辑最直接。
- 回溯模板清晰。
- 容易解释和实现。
- 不需要额外的二维数组。

如果面试官继续追问如何优化重复的回文判断，再给出“动态规划预处理 + 回溯”。

本题的核心关系可以总结为：

```
从 start 开始枚举下一段的结束位置 end
    如果 s[start:end+1] 是回文串：
        选择该子串
        递归分割剩余部分
        撤销选择
```

由于题目要求返回所有答案，而合法分割方案的数量本身可能达到指数级，因此无论如何优化，都无法避免最坏情况下的指数级输出复杂度。