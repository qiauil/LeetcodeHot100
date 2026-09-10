# N 皇后（N Queens）

## 题目描述

**N 皇后问题**要求在一个 `n x n` 的国际象棋棋盘上放置 `n` 个皇后，并且任意两个皇后之间都不能互相攻击。

国际象棋中的皇后可以攻击：

- 同一行
- 同一列
- 两条对角线方向

给定一个整数 `n`，返回 **N 皇后问题的所有不同解法**。

每个解法由一个棋盘布局组成：

- `"Q"` 表示皇后
- `"."` 表示空位置

答案可以按 **任意顺序** 返回。

### 示例 1

```text
输入：n = 4

输出：
[
    [".Q..", "...Q", "Q...", "..Q."],
    ["..Q.", "Q...", "...Q", ".Q.."]
]
```

解释：`4` 皇后问题一共有两种不同的合法布局。

### 示例 2

```text
输入：n = 1

输出：
[["Q"]]
```

### 约束

```text
1 <= n <= 8
```

------

# 核心思路：回溯

N 皇后是非常典型的 **回溯（Backtracking）+ 剪枝（Pruning）** 问题。

因为每一行最终一定只放一个皇后，所以我们可以按照：

```text
第 0 行
↓
第 1 行
↓
第 2 行
↓
...
↓
第 n - 1 行
```

逐行放置皇后。

在第 `r` 行，我们尝试把皇后放在每一个列 `c`：

```text
(r, 0)
(r, 1)
(r, 2)
...
(r, n - 1)
```

如果当前位置不会和前面已经放置的皇后冲突，就：

1. 放置皇后
2. 递归处理下一行
3. 递归返回后撤销当前选择
4. 尝试下一个位置

这个“**做选择 → 递归 → 撤销选择**”的过程，就是回溯算法最核心的模式。

------

# 对角线为什么可以用 `row + col` 和 `row - col`？

这是这道题最重要的技巧之一。

假设棋盘坐标为：

```text
(row, col)
```

对于 `/` 方向的对角线：

```text
(0, 2)
(1, 1)
(2, 0)
```

可以发现：

```text
row + col = 2
```

所以：

> 同一条 `/` 对角线上的格子具有相同的 `row + col`。

而对于 `\` 方向：

```text
(0, 0)
(1, 1)
(2, 2)
```

可以发现：

```text
row - col = 0
```

所以：

> 同一条 `\` 对角线上的格子具有相同的 `row - col`。

因此判断 `(r, c)` 是否和已有皇后冲突，只需要检查：

```text
c
r + c
r - c
```

是否已经被占用。

------

# 方法一：基础回溯 + 扫描棋盘

## 思路

最直接的方法是不额外保存攻击状态。

每次准备在 `(r, c)` 放置皇后时，直接检查棋盘：

- 当前列的上方
- 左上方对角线
- 右上方对角线

为什么只检查上方？

因为我们是 **从上到下逐行放置皇后**。当前行下面还没有放置任何皇后，所以根本不需要检查。

另外，因为每一行只会放一个皇后，所以也不需要检查同行。

## 算法步骤

1. 创建一个 `n x n` 棋盘，全部初始化为 `"."`。
2. 从第 `0` 行开始执行回溯。
3. 对当前第 `r` 行，枚举所有列 `c`。
4. 调用 `isSafe(r, c)` 检查当前位置：
   - 上方同列是否有皇后
   - 左上对角线是否有皇后
   - 右上对角线是否有皇后
5. 如果安全：
   - 放置 `"Q"`
   - 递归处理第 `r + 1` 行
   - 撤销当前皇后
6. 当 `r == n` 时，说明所有行都已经成功放置皇后，保存当前答案。

## 代码

```python
from typing import List


class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        res = []

        # 使用二维字符数组表示棋盘。
        # 之所以不用字符串，是因为 Python 字符串不可修改。
        board = [["."] * n for _ in range(n)]

        def backtrack(r: int) -> None:
            # 成功放完 n 行，得到一个完整解
            if r == n:
                # board 中的每一行目前是 List[str]，
                # 题目要求每一行是字符串，因此使用 join 转换。
                solution = ["".join(row) for row in board]
                res.append(solution)
                return

            # 尝试在第 r 行的每一列放置皇后
            for c in range(n):
                if self.isSafe(r, c, board):
                    # 做选择
                    board[r][c] = "Q"

                    # 递归处理下一行
                    backtrack(r + 1)

                    # 撤销选择，恢复现场
                    board[r][c] = "."

        backtrack(0)

        return res

    def isSafe(self, r: int, c: int, board: List[List[str]]) -> bool:
        # 1. 检查当前列上方
        row = r - 1
        while row >= 0:
            if board[row][c] == "Q":
                return False
            row -= 1

        # 2. 检查左上方对角线
        row, col = r - 1, c - 1

        while row >= 0 and col >= 0:
            if board[row][col] == "Q":
                return False

            row -= 1
            col -= 1

        # 3. 检查右上方对角线
        row, col = r - 1, c + 1

        while row >= 0 and col < len(board):
            if board[row][col] == "Q":
                return False

            row -= 1
            col += 1

        return True
```

## Python 细节：`"".join(row)`

例如：

```python
row = [".", "Q", ".", "."]

"".join(row)
```

结果为：

```text
".Q.."
```

`join()` 是 Python 字符串的方法，用于把一个字符串序列连接起来。

语法：

```python
separator.join(iterable)
```

例如：

```python
"-".join(["a", "b", "c"])
```

得到：

```text
"a-b-c"
```

这里分隔符使用空字符串 `""`，所以所有字符直接连接。

## 复杂度

通常将搜索复杂度粗略写作：

```text
O(n!)
```

因为每一行选择一列，而已经使用过的列不能再次使用，搜索树规模大致受到排列数量 `n!` 的限制。

但是这个版本每次判断一个位置是否安全还需要扫描最多 `O(n)` 个格子，因此相比后面的集合版本具有额外的检查开销。

另外，每发现一个答案时，将棋盘转换为字符串需要：

```text
O(n²)
```

空间复杂度：

```text
O(n²)
```

主要用于保存棋盘。

递归栈深度为：

```text
O(n)
```

------

# 方法二：回溯 + Hash Set

这是面试中最推荐掌握的写法之一。

相比方法一，它不再每次扫描棋盘判断冲突，而是直接记录：

```text
哪些列已经被使用
哪些 / 对角线已经被使用
哪些 \ 对角线已经被使用
```

这样就可以在平均 **O(1)** 时间判断当前位置是否合法。

## 思路

使用三个集合：

```python
cols
pos_diag
neg_diag
```

分别保存：

```text
cols      → col
pos_diag  → row + col
neg_diag  → row - col
```

对于位置：

```python
(r, c)
```

如果满足下面任意条件：

```python
c in cols
r + c in pos_diag
r - c in neg_diag
```

就说明当前位置会受到已有皇后的攻击。

------

## 算法步骤

1. 创建三个集合：
   - `cols`
   - `pos_diag`
   - `neg_diag`
2. 创建空棋盘。
3. 从第 `0` 行开始回溯。
4. 枚举当前行的所有列。
5. 如果列或者两个对角线已经被占用，直接跳过。
6. 否则：
   - 将状态加入集合
   - 放置皇后
   - 递归下一行
7. 返回后：
   - 从集合删除对应状态
   - 删除棋盘上的皇后
8. 当成功走到 `r == n` 时保存答案。

## 代码

```python
from typing import List


class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        # 已经放置皇后的列
        cols = set()

        # / 对角线：row + col 相同
        pos_diag = set()

        # \ 对角线：row - col 相同
        neg_diag = set()

        res = []

        board = [["."] * n for _ in range(n)]

        def backtrack(r: int) -> None:
            # 所有行都已经成功放置皇后
            if r == n:
                res.append(["".join(row) for row in board])
                return

            # 尝试当前行的每一列
            for c in range(n):

                # 判断列和两条对角线是否冲突
                if (
                    c in cols
                    or (r + c) in pos_diag
                    or (r - c) in neg_diag
                ):
                    continue

                # ----------------
                # 做选择
                # ----------------

                cols.add(c)
                pos_diag.add(r + c)
                neg_diag.add(r - c)

                board[r][c] = "Q"

                # 进入下一层决策
                backtrack(r + 1)

                # ----------------
                # 撤销选择
                # ----------------

                cols.remove(c)
                pos_diag.remove(r + c)
                neg_diag.remove(r - c)

                board[r][c] = "."

        backtrack(0)

        return res
```

------

## Python 细节：`set`

Python 的 `set` 是 **哈希集合**。

例如：

```python
cols = set()

cols.add(2)
cols.add(5)
```

现在：

```python
2 in cols
```

返回：

```python
True
```

常用操作：

```python
s.add(x)
```

向集合加入元素。

```python
s.remove(x)
```

删除元素。

```python
x in s
```

判断元素是否存在。

这些操作平均时间复杂度都是：

```text
O(1)
```

因此特别适合解决：

> “某个状态是否已经出现？”

这一类问题。

------

# 为什么集合方法比扫描棋盘更好？

方法一判断：

```text
(r, c) 能不能放？
```

需要在棋盘上向三个方向扫描。

而集合版本相当于提前把信息进行了总结：

```text
列 2 是否已经被攻击？
→ 直接查询 cols

/ 对角线 5 是否已经被攻击？
→ 直接查询 pos_diag

\ 对角线 -2 是否已经被攻击？
→ 直接查询 neg_diag
```

因此：

```text
扫描棋盘：
O(n)

Hash Set：
平均 O(1)
```

这属于非常典型的：

> **空间换时间**

------

# 方法三：回溯 + Visited Array

## 思路

这种方法和 Hash Set 本质完全相同，只是把集合改成了布尔数组。

因为状态范围是确定的：

```text
列：
0 ~ n-1

row + col：
0 ~ 2n-2

row - col：
-(n-1) ~ n-1
```

所以可以使用数组直接表示某个状态是否已经被占用。

------

## 对角线数组长度为什么约为 `2n`？

对于：

```python
r + c
```

最小值：

```text
0
```

最大值：

```text
(n - 1) + (n - 1)
= 2n - 2
```

因此理论上需要：

```text
2n - 1
```

个位置。

代码中写：

```python
[False] * (2 * n)
```

只是稍微多分配一个位置，实现更简单，也完全没有问题。

------

## 为什么 `r - c` 要加 `n`？

因为：

```python
r - c
```

可能是负数。

例如：

```text
r = 0
c = 3

r - c = -3
```

虽然 Python 支持负索引，但这里如果直接使用：

```python
neg_diag[-3]
```

它表示的是“从数组结尾倒数第三个元素”，而不是我们想要的数学索引。

因此要进行偏移：

```python
r - c + n
```

把所有值移动到非负范围。

------

## 代码

```python
from typing import List


class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        # 第 c 列是否已经存在皇后
        cols = [False] * n

        # / 对角线：r + c
        pos_diag = [False] * (2 * n)

        # \ 对角线：r - c + n
        neg_diag = [False] * (2 * n)

        res = []

        board = [["."] * n for _ in range(n)]

        def backtrack(r: int) -> None:
            if r == n:
                res.append(["".join(row) for row in board])
                return

            for c in range(n):
                pos_index = r + c
                neg_index = r - c + n

                # 当前位置存在冲突
                if (
                    cols[c]
                    or pos_diag[pos_index]
                    or neg_diag[neg_index]
                ):
                    continue

                # 做选择
                cols[c] = True
                pos_diag[pos_index] = True
                neg_diag[neg_index] = True

                board[r][c] = "Q"

                backtrack(r + 1)

                # 撤销选择
                cols[c] = False
                pos_diag[pos_index] = False
                neg_diag[neg_index] = False

                board[r][c] = "."

        backtrack(0)

        return res
```

------

# Hash Set 和 Visited Array 怎么选择？

两者算法思想完全相同。

| 方法          | 优点               | 缺点                   |
| ------------- | ------------------ | ---------------------- |
| Hash Set      | 代码直观，容易理解 | 有哈希表常数开销       |
| Boolean Array | 常数开销更小       | 对角线索引需要额外计算 |

面试中一般：

> **Hash Set 写法最容易解释，也最不容易写错。**

如果面试官进一步询问优化，可以再说明：

```text
Hash Set
↓
Boolean Array
↓
Bit Mask
```

它们本质上都在优化同一件事：

> 如何快速记录并查询列和对角线是否已经被占用。

------

# 方法四：回溯 + Bit Mask

## 思路

这是更加底层、高效的状态表示方式。

之前使用：

```python
cols = set()
```

或者：

```python
cols = [False] * n
```

现在可以直接用一个整数的二进制位表示。

例如：

```text
cols = 00101000
```

可以理解成：

```text
第 3 列已经使用
第 5 列已经使用
```

一个整数本身就可以保存很多个 Boolean 状态。

因此：

- 列使用一个整数
- `/` 对角线使用一个整数
- `\` 对角线使用一个整数

------

# Bit Mask 基础

## `1 << c`

```python
1 << c
```

表示把二进制的 `1` 向左移动 `c` 位。

例如：

```python
1 << 0
0001
1 << 1
0010
1 << 3
1000
```

因此：

```python
1 << c
```

可以生成：

> “第 `c` 位对应的掩码”。

------

## 使用 `&` 判断某一位是否已经存在

例如：

```python
cols & (1 << c)
```

如果结果不是 `0`，说明 `cols` 的第 `c` 位已经是 `1`。

也就是：

```text
第 c 列已经存在皇后
```

------

## 使用 XOR `^` 修改状态

XOR 的规则：

```text
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0
```

因此：

```python
mask ^= bit
```

会把某一位翻转：

```text
0 → 1
1 → 0
```

回溯算法中可以利用这个特点：

第一次：

```python
mask ^= bit
```

表示加入状态。

递归返回以后再次：

```python
mask ^= bit
```

就恢复原来的状态。

不过在普通工程代码里，通常也可以写得更加语义明确：

```python
mask |= bit
```

添加位；

回溯时：

```python
mask &= ~bit
```

清除位。

原解使用 XOR 是因为可以非常方便地“开关”同一个 bit。

------

## 代码

```python
from typing import List


class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        # 每一位代表对应列是否已经被使用
        cols = 0

        # 每一位代表对应的 / 对角线是否被使用
        pos_diag = 0

        # 每一位代表对应的 \ 对角线是否被使用
        neg_diag = 0

        res = []

        board = [["."] * n for _ in range(n)]

        def backtrack(r: int) -> None:
            # 因为下面需要修改外层函数中的三个整数，
            # 所以必须声明 nonlocal。
            nonlocal cols, pos_diag, neg_diag

            if r == n:
                res.append(["".join(row) for row in board])
                return

            for c in range(n):
                col_bit = 1 << c
                pos_bit = 1 << (r + c)
                neg_bit = 1 << (r - c + n)

                # 使用位与 & 判断相应 bit 是否已经为 1
                if (
                    (cols & col_bit)
                    or (pos_diag & pos_bit)
                    or (neg_diag & neg_bit)
                ):
                    continue

                # 做选择：把相应 bit 从 0 翻转为 1
                cols ^= col_bit
                pos_diag ^= pos_bit
                neg_diag ^= neg_bit

                board[r][c] = "Q"

                backtrack(r + 1)

                # 撤销选择：再次 XOR，相应 bit 从 1 恢复成 0
                cols ^= col_bit
                pos_diag ^= pos_bit
                neg_diag ^= neg_bit

                board[r][c] = "."

        backtrack(0)

        return res
```

------

# Python 细节：`nonlocal`

这里有：

```python
cols = 0

def backtrack():
    nonlocal cols
```

为什么需要 `nonlocal`？

因为整数是不可变对象。

当内部函数执行：

```python
cols ^= 1
```

本质上等价于给 `cols` **重新赋值**。

Python 默认会把被重新赋值的变量视为当前函数的局部变量。

因此，如果希望修改外层函数：

```python
solveNQueens()
```

中的 `cols`，必须声明：

```python
nonlocal cols
```

例如：

```python
def outer():
    x = 10

    def inner():
        nonlocal x
        x += 1

    inner()

    print(x)
```

输出：

```text
11
```

这里的 `nonlocal` 表示：

> `x` 不是 `inner()` 的局部变量，而是来自上一层函数作用域。

------

# 四种方法之间的关系

这四种方法的 **搜索逻辑完全相同**。

真正变化的只是：

> 如何判断当前位置是否与之前的皇后发生冲突。

可以理解成下面的演进：

```text
方法一
每次扫描棋盘
     ↓
方法二
Hash Set 保存攻击状态
     ↓
方法三
Boolean Array 保存攻击状态
     ↓
方法四
Bit Mask 保存攻击状态
```

对应查询方式：

```text
扫描棋盘：

    当前列有没有皇后？
    → 去棋盘上找


Hash Set：

    c in cols


Boolean Array：

    cols[c]


Bit Mask：

    cols & (1 << c)
```

------

# 面试推荐写法

如果这是普通算法面试，我更推荐掌握 **Hash Set 版本**。

原因是它兼顾：

```text
代码简洁
+
逻辑清晰
+
冲突判断 O(1)
+
不容易出现 Bit Mask 的位运算错误
```

真正应该在面试中讲清楚的核心是：

```text
每行只放一个皇后
↓
因此不需要检查同行

从上往下放置
↓
因此只需要考虑之前的行

列冲突
↓
col 相同

/ 对角线冲突
↓
row + col 相同

\ 对角线冲突
↓
row - col 相同

递归以后恢复状态
↓
Backtracking
```

------

# 一个更简洁的推荐版本

实际上我们甚至不一定需要维护整个二维棋盘。

因为每一行只会放一个皇后，所以只需要记录：

```python
queens[r] = c
```

表示：

> 第 `r` 行的皇后位于第 `c` 列。

等找到完整答案以后，再生成字符串棋盘。

这样可以让搜索状态更加紧凑。

```python
from typing import List


class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        cols = set()
        pos_diag = set()
        neg_diag = set()

        # queens[r] = c
        # 表示第 r 行的皇后位于第 c 列
        queens = [-1] * n

        res = []

        def backtrack(r: int) -> None:
            if r == n:
                board = []

                for c in queens:
                    row = "." * c + "Q" + "." * (n - c - 1)
                    board.append(row)

                res.append(board)
                return

            for c in range(n):
                if (
                    c in cols
                    or r + c in pos_diag
                    or r - c in neg_diag
                ):
                    continue

                # 做选择
                queens[r] = c
                cols.add(c)
                pos_diag.add(r + c)
                neg_diag.add(r - c)

                backtrack(r + 1)

                # 撤销选择
                queens[r] = -1
                cols.remove(c)
                pos_diag.remove(r + c)
                neg_diag.remove(r - c)

        backtrack(0)

        return res
```

这种写法的一个优点是：

> 搜索阶段只保存真正重要的信息——“每一行的皇后在哪一列”。

二维棋盘主要是输出格式所需要的，并不是搜索算法本身必须维护的状态。

------

# 时间与空间复杂度

## 时间复杂度

N 皇后的精确时间复杂度比较难写成简单闭式。

面试中通常可以写：

```text
O(n!)
```

作为回溯搜索规模的粗略上界。

原因是：

第一行最多有：

```text
n
```

种选择。

第二行由于不能重复列，最多：

```text
n - 1
```

种选择。

然后是：

```text
n - 2
```

……

因此搜索空间可以粗略理解为：

```text
n × (n - 1) × (n - 2) × ...
≈ n!
```

实际上对角线约束会进一步大量剪枝，所以真正搜索的节点数量通常小于简单的 `n!` 排列搜索。

如果考虑生成每个最终棋盘需要 `O(n²)`，则输出所有答案本身也具有不可忽略的成本。

------

## 空间复杂度

如果维护完整棋盘：

```text
O(n²)
```

另外：

```text
递归调用栈：O(n)

列集合：O(n)

对角线集合：O(n)
```

因此除去最终答案，整体主要由棋盘决定：

```text
O(n²)
```

如果采用只保存：

```python
queens[r] = c
```

的写法，则搜索过程的额外空间可以降低到：

```text
O(n)
```

不包括最终输出答案。

------

# 常见错误

## 1. 只检查列，没有检查对角线

皇后不仅可以纵向攻击，还可以沿两条对角线攻击。

因此仅仅判断：

```python
c in cols
```

是不够的。

还必须判断：

```python
r + c
r - c
```

是否冲突。

------

# 2. 把两种对角线公式记反

可以通过几个简单坐标理解，而不要完全死记。

对于：

```text
\
```

例如：

```text
(0, 0)
(1, 1)
(2, 2)
```

有：

```text
row - col = 0
```

因此：

```text
\ → row - col
```

对于：

```text
/
```

例如：

```text
(0, 2)
(1, 1)
(2, 0)
```

有：

```text
row + col = 2
```

因此：

```text
/ → row + col
```

------

# 3. 使用数组时忘记处理负数索引

`row - col` 可能小于 `0`。

例如：

```text
0 - 3 = -3
```

因此如果使用 Boolean Array，应该进行偏移：

```python
row - col + n
```

这样才能确保索引是非负数。

------

# 4. 忘记恢复状态

这是所有回溯题最经典的错误。

正确结构通常是：

```python
# 做选择
state.add(...)
board[r][c] = "Q"

backtrack(...)

# 撤销选择
state.remove(...)
board[r][c] = "."
```

可以记成一个非常重要的模板：

```text
Choose
Explore
Unchoose
```

或者：

```text
做选择
↓
递归
↓
撤销选择
```

如果忘记最后一步，前一个分支留下来的状态会污染后续分支。

------

# 5. `continue` 的位置写错

例如：

```python
if conflict:
    continue
```

表示：

> 当前列不合法，直接尝试下一列。

而不是：

```python
return
```

如果错误写成：

```python
if conflict:
    return
```

会直接结束当前整层递归，导致后面的其他列都没有机会尝试，因此可能漏掉大量合法答案。

------

# 6. 保存答案时直接保存可变棋盘对象

错误思路类似：

```python
res.append(board)
```

问题在于：

```python
board
```

在之后的回溯中还会不断变化。

结果列表里的不同“答案”可能实际上全部指向同一个对象。

正确方式是创建当前状态的副本，例如：

```python
res.append(["".join(row) for row in board])
```

这里不仅完成了复制，也同时把：

```python
List[List[str]]
```

转换为了题目要求的：

```python
List[str]
```

------

# 7. 重复检查当前行或者棋盘下方

由于算法保证：

```text
每一行只放一个皇后
```

因此无需检查同行。

由于我们从：

```text
上 → 下
```

逐行放置，因此当前位置下面还没有任何皇后。

对于基础扫描版本，只需要检查：

```text
↑
↖
↗
```

而不需要检查：

```text
↓
↙
↘
```

这也是利用搜索顺序简化约束检查的典型技巧。

------

# 回溯模板总结

N 皇后非常适合理解一个通用回溯模板：

```python
def backtrack(state):
    # 找到一个完整答案
    if is_complete(state):
        save_answer()
        return

    for choice in choices:
        # 剪枝：非法选择直接跳过
        if not valid(choice):
            continue

        # 做选择
        make_choice(choice)

        # 进入下一层
        backtrack(next_state)

        # 撤销选择
        undo_choice(choice)
```

N 皇后对应：

```text
state
→ 当前正在处理哪一行

choices
→ 当前行可以选择哪些列

valid
→ 列以及两条对角线是否冲突

make_choice
→ 放置皇后 + 记录攻击状态

next_state
→ 下一行

undo_choice
→ 删除皇后 + 恢复攻击状态
```

这套思想之后还会大量出现在：

```text
Sudoku Solver
Combination Sum
Permutations
Subsets
Word Search
Palindrome Partitioning
```

等经典面试题中。
