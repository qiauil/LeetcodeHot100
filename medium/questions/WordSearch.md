# 单词搜索（Word Search）

给定一个二维字符网格 `board` 和一个字符串 `word`。如果能够在网格中找到该单词，返回 `True`；否则返回 `False`。

单词必须由网格中水平或垂直相邻的单元格依次组成，即每一步只能向上、下、左、右移动。在同一条搜索路径中，每个单元格最多只能使用一次。

------

## 核心思路：深度优先搜索与回溯

可以把网格中的每个单元格都当作单词的起点，然后使用深度优先搜索（DFS）依次匹配单词中的字符。

对于当前位置 `(r, c)` 和待匹配字符下标 `i`：

1. 判断当前位置是否可以匹配 `word[i]`。
2. 如果可以，则暂时将该单元格标记为“已访问”。
3. 继续向上、下、左、右搜索 `word[i + 1]`。
4. 搜索结束后撤销访问标记，使该单元格可以被其他路径使用。

“选择当前位置 → 继续搜索 → 撤销选择”的过程，就是典型的回溯。

------

## 解法一：回溯 + 哈希集合

### 思路

使用一个集合 `path`，保存当前搜索路径中已经使用过的坐标。

Python 的 `set` 是哈希集合：

- `path.add((r, c))`：记录当前位置已经被使用。
- `(r, c) in path`：判断当前位置是否已经被使用。
- `path.remove((r, c))`：回溯时移除当前位置。

集合中存储的 `(r, c)` 是一个元组。元组是不可变对象，因此可以作为哈希集合中的元素。

### 算法步骤

1. 遍历网格中的每个单元格，将其作为搜索起点。
2. 定义 `dfs(r, c, i)`，表示从 `(r, c)` 开始，能否匹配 `word[i:]`。
3. 如果 `i == len(word)`，说明所有字符均已匹配，返回 `True`。
4. 如果越界、字符不匹配或当前位置已被使用，返回 `False`。
5. 将当前位置加入 `path`。
6. 向四个方向递归搜索下一个字符。
7. 将当前位置从 `path` 中移除，完成回溯。

```
from typing import List


class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        rows, cols = len(board), len(board[0])

        # 保存当前搜索路径中已经使用过的坐标
        path = set()

        def dfs(r: int, c: int, i: int) -> bool:
            # 已经成功匹配完整个单词
            if i == len(word):
                return True

            # 越界、字符不匹配，或者当前位置已被使用
            if (
                r < 0
                or c < 0
                or r >= rows
                or c >= cols
                or board[r][c] != word[i]
                or (r, c) in path
            ):
                return False

            # 做出选择：将当前位置加入当前路径
            path.add((r, c))

            # 尝试向四个方向匹配下一个字符
            found = (
                dfs(r + 1, c, i + 1)
                or dfs(r - 1, c, i + 1)
                or dfs(r, c + 1, i + 1)
                or dfs(r, c - 1, i + 1)
            )

            # 撤销选择，使当前位置可以被其他路径使用
            path.remove((r, c))

            return found

        # 尝试将每个单元格作为单词的起点
        for r in range(rows):
            for c in range(cols):
                if board[r][c] == word[0] and dfs(r, c, 0):
                    return True

        return False
```

### 复杂度分析

设：

- `R`、`C` 分别为网格的行数和列数；
- `L` 为单词长度；
- `M = R × C` 为网格单元格总数。
- 时间复杂度：`O(R × C × 4^L)`
  更精确地说，可以写成 `O(R × C × 3^(L-1))`。因为第一次最多有 4 个方向，之后不能立即回到上一个单元格，所以通常最多只剩 3 个方向。
- 空间复杂度：`O(L)`
  递归调用栈和 `path` 最多保存 `L` 个位置。

------

## 解法二：回溯 + 访问数组

### 思路

创建一个与 `board` 大小相同的二维布尔数组 `visited`：

- `visited[r][c] == True`：当前路径已经使用了该单元格。
- `visited[r][c] == False`：当前路径尚未使用该单元格。

与哈希集合相比，访问数组不需要保存坐标元组，查询也很直接；但无论当前单词有多短，它都会额外分配一个与整个网格一样大的数组。

### 算法步骤

1. 创建大小为 `R × C` 的 `visited` 数组。
2. 从每个可能的起点开始执行 DFS。
3. 匹配当前位置后，将 `visited[r][c]` 设置为 `True`。
4. 搜索四个相邻位置。
5. 搜索完成后，将 `visited[r][c]` 恢复为 `False`。

```
from typing import List


class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        rows, cols = len(board), len(board[0])

        # visited[r][c] 表示该单元格是否已在当前路径中
        visited = [[False] * cols for _ in range(rows)]

        def dfs(r: int, c: int, i: int) -> bool:
            # 所有字符均已成功匹配
            if i == len(word):
                return True

            # 检查边界、字符和访问状态
            if (
                r < 0
                or c < 0
                or r >= rows
                or c >= cols
                or board[r][c] != word[i]
                or visited[r][c]
            ):
                return False

            # 标记当前位置
            visited[r][c] = True

            found = (
                dfs(r + 1, c, i + 1)
                or dfs(r - 1, c, i + 1)
                or dfs(r, c + 1, i + 1)
                or dfs(r, c - 1, i + 1)
            )

            # 回溯：撤销访问标记
            visited[r][c] = False

            return found

        for r in range(rows):
            for c in range(cols):
                if board[r][c] == word[0] and dfs(r, c, 0):
                    return True

        return False
```

### 复杂度分析

- 时间复杂度：`O(R × C × 4^L)`，更精确的上界可写为 `O(R × C × 3^(L-1))`。
- 空间复杂度：`O(R × C + L)`。
  - `visited` 数组占用 `O(R × C)`；
  - 递归调用栈最多占用 `O(L)`。

因此，原答案将这种方法的空间复杂度写成 `O(L)` 并不准确，因为还需要计算整个 `visited` 数组。

------

## 解法三：原地标记的回溯

### 思路

不再创建集合或访问数组，而是直接临时修改 `board`。

进入一个单元格后：

1. 保存它原来的字符。
2. 将其修改为特殊字符，例如 `'#'`，表示当前路径已经使用了该位置。
3. 搜索四个方向。
4. 搜索完成后恢复原字符。

这种方法减少了额外的数据结构，是本题最常用的实现。

### 代码

```
from typing import List


class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        rows, cols = len(board), len(board[0])

        def dfs(r: int, c: int, i: int) -> bool:
            # 已经匹配完所有字符
            if i == len(word):
                return True

            # 越界或者当前字符不匹配
            # 被标记为 "#" 的位置也会在字符比较时匹配失败
            if (
                r < 0
                or c < 0
                or r >= rows
                or c >= cols
                or board[r][c] != word[i]
            ):
                return False

            # 保存原字符，并将当前位置标记为已使用
            original_char = board[r][c]
            board[r][c] = "#"

            # 搜索四个相邻方向
            found = (
                dfs(r + 1, c, i + 1)
                or dfs(r - 1, c, i + 1)
                or dfs(r, c + 1, i + 1)
                or dfs(r, c - 1, i + 1)
            )

            # 回溯：恢复原字符
            board[r][c] = original_char

            return found

        for r in range(rows):
            for c in range(cols):
                # 只有首字符匹配时才需要启动 DFS
                if board[r][c] == word[0] and dfs(r, c, 0):
                    return True

        return False
```

这里使用 `original_char` 保存原字符，比直接写：

```
board[r][c] = word[i]
```

更加清晰、稳妥，也更能体现“恢复现场”的含义。

### 复杂度分析

- 时间复杂度：`O(R × C × 4^L)`，更精确地可以写为 `O(R × C × 3^(L-1))`。
- 辅助空间复杂度：`O(L)`，主要来自递归调用栈。
- 除递归栈以外的额外空间：`O(1)`。

这种方法只是临时修改输入网格，并且会在回溯时恢复，因此函数结束后 `board` 的内容不会发生变化。

------

## Python 语法与函数说明

### `List[List[str]]`

```
board: List[List[str]]
```

这是类型注解，表示 `board` 是一个二维列表，内部元素为字符串。

使用 `List` 前需要导入：

```
from typing import List
```

在 Python 3.9 及以上版本中，也可以直接写：

```
board: list[list[str]]
```

类型注解主要帮助阅读代码和进行静态检查，不会改变程序的运行逻辑。

### `or` 的短路求值

```
found = (
    dfs(...)
    or dfs(...)
    or dfs(...)
    or dfs(...)
)
```

Python 会从左到右计算表达式。一旦某个 `dfs` 返回 `True`，后面的调用就不会继续执行。

这意味着找到一条有效路径后，可以立即停止搜索。

### 二维数组初始化

```
visited = [[False] * cols for _ in range(rows)]
```

这里必须通过列表推导式逐行创建列表。

不建议写成：

```
visited = [[False] * cols] * rows
```

因为这种写法会让所有行引用同一个内部列表。修改其中一行时，其他行也可能一起改变。

------

## 常见错误

### 1. 回溯后没有恢复状态

无论使用集合、访问数组还是原地修改，都必须在递归搜索结束后撤销当前选择：

```
path.remove((r, c))
```

或者：

```
visited[r][c] = False
```

或者：

```
board[r][c] = original_char
```

否则，当前路径的访问状态会污染后续搜索。

### 2. 在检查边界之前访问数组

下面的顺序可能导致下标越界：

```
if board[r][c] != word[i] or r < 0:
    ...
```

必须先确认坐标合法，再访问 `board[r][c]`：

```
if r < 0 or r >= rows or c < 0 or c >= cols:
    return False
```

利用 Python `or` 的短路特性，也可以将它们安全地写在同一个条件中。

### 3. 重复使用同一个单元格

即使相邻单元格的字符匹配，也不能在同一条路径中重复经过某个位置。因此必须维护当前路径的访问状态，而不能只比较字符。

### 4. 找到答案后过早返回，导致状态没有恢复

下面这种写法会在返回前跳过恢复操作：

```
board[r][c] = "#"

if dfs(r + 1, c, i + 1):
    return True

board[r][c] = original_char
```

如果递归成功，函数会直接返回，当前位置无法恢复。

更稳妥的写法是先保存结果，再恢复状态：

```
found = dfs(r + 1, c, i + 1)

board[r][c] = original_char
return found
```

### 5. 对每个单元格都无条件启动 DFS

直接从所有单元格调用 DFS 不会影响正确性，但可以先判断首字符：

```
if board[r][c] == word[0] and dfs(r, c, 0):
    return True
```

这样能够避免大量显然不可能成功的搜索。

------

## 面试总结

这道题是经典的“网格 DFS + 回溯”问题。识别它的关键是：

- 需要尝试多条可能路径；
- 每一步有上、下、左、右四种选择；
- 同一个位置在一条路径中不能重复使用；
- 一条路径失败后，需要恢复状态并尝试其他路径。

三种方法中，原地修改网格通常是面试中的首选方案，因为代码简洁，且除了递归调用栈以外不需要额外的访问结构。