# 岛屿数量（Number of Islands）

给定一个二维网格 `grid`，其中：

- `'1'` 表示陆地
- `'0'` 表示水域

请计算并返回网格中 **岛屿的数量**。

一个岛屿由若干个在 **水平或垂直方向相邻** 的陆地单元格组成。也就是说，只考虑 **上、下、左、右四个方向**，不考虑对角线连接。

可以假设网格外围全部是水。

------

## 核心思路

这道题本质上是在二维网格中求 **连通分量（Connected Components）的数量**。

每一个岛屿就是一个由 `'1'` 组成的独立连通分量。

常见的解决方法有：

1. **DFS（深度优先搜索）**
2. **BFS（广度优先搜索）**
3. **并查集（Disjoint Set Union / Union-Find）**

面试中通常优先掌握 DFS 和 BFS。它们的思路最直接，也最容易写正确。

------

# 1. 深度优先搜索 DFS

## 思路

可以把整个 `grid` 看成一张地图。

当我们从左到右、从上到下遍历网格时，如果遇到一个 `'1'`，说明发现了一个此前还没有访问过的新岛屿。

此时：

1. 岛屿数量 `+1`
2. 从这个位置开始 DFS
3. 找到与它相连的所有陆地
4. 将这些陆地全部改成 `'0'`

这个过程可以理解为：

> 每发现一个岛屿，就用 DFS 把整个岛屿“淹没”。

这样之后再遍历到这些位置时，它们已经变成 `'0'`，因此不会被重复统计。

------

## 算法步骤

1. 遍历整个二维网格。
2. 如果当前单元格为 `'1'`：
   - 说明发现了一个新的岛屿。
   - `islands += 1`
   - 从当前单元格执行 DFS。
3. DFS 中：
   - 如果当前位置越界，直接返回。
   - 如果当前位置已经是 `'0'`，直接返回。
   - 将当前位置设为 `'0'`，表示已经访问。
   - 递归搜索上、下、左、右四个方向。
4. 遍历完成后返回 `islands`。

------

## 代码

```
from typing import List


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        # 四个搜索方向：
        # 下、上、右、左
        directions = [
            (1, 0),
            (-1, 0),
            (0, 1),
            (0, -1)
        ]

        ROWS, COLS = len(grid), len(grid[0])
        islands = 0

        def dfs(r: int, c: int) -> None:
            # 1. 越界
            # 2. 当前不是陆地
            # 都不需要继续搜索
            if (
                r < 0
                or c < 0
                or r >= ROWS
                or c >= COLS
                or grid[r][c] == "0"
            ):
                return

            # 标记当前陆地已经访问过
            # 直接修改原 grid，因此不需要额外 visited 集合
            grid[r][c] = "0"

            # 搜索四个相邻方向
            for dr, dc in directions:
                dfs(r + dr, c + dc)

        # 遍历整个网格
        for r in range(ROWS):
            for c in range(COLS):
                if grid[r][c] == "1":
                    # 每遇到一个还没有访问过的陆地，
                    # 就意味着发现了一个新的岛屿
                    islands += 1

                    # 把整个岛屿全部标记为已访问
                    dfs(r, c)

        return islands
```

------

## 为什么每次 DFS 只会计算一个岛屿？

假设当前第一次发现：

```
1 1 0
1 0 0
0 0 1
```

从左上角的 `'1'` 开始 DFS：

```
1 1
1
```

这三个格子彼此相连，所以 DFS 会把它们全部访问并改成 `'0'`。

之后继续遍历：

```
0 0 0
0 0 0
0 0 1
```

最后右下角的 `'1'` 会触发第二次 DFS。

因此：

> **DFS 被启动了多少次，就有多少个岛屿。**

这也是这道题最关键的理解。

------

## 时间复杂度

设：

- `m` = 行数
- `n` = 列数

每个格子最多被访问一次，因此：

\[ O(mn) \]

### 空间复杂度

最坏情况下，整个网格都是陆地：

```
1 1 1
1 1 1
1 1 1
```

DFS 的递归调用栈最坏可能达到：

\[ O(mn) \]

因此：

- **时间复杂度：`O(mn)`**
- **空间复杂度：`O(mn)`**

实际情况下 DFS 递归深度通常不会始终达到 `mn`，但分析最坏情况时需要按照 `O(mn)` 计算。

------

# 2. 广度优先搜索 BFS

## 思路

BFS 与 DFS 的核心逻辑完全相同：

> 每发现一块新的陆地，就把整个岛屿全部访问一遍。

区别只在于搜索方式：

- DFS 使用 **递归 / 栈**
- BFS 使用 **队列**

发现一个 `'1'` 后：

1. 岛屿数量 `+1`
2. 将这个位置加入队列
3. 不断从队列中取出位置
4. 检查其上、下、左、右
5. 如果相邻位置也是陆地，就加入队列继续搜索

直到队列为空，说明当前岛屿已经全部访问完毕。

------

## 算法步骤

1. 遍历整个网格。
2. 遇到 `'1'`：
   - `islands += 1`
   - 从当前位置执行 BFS。
3. BFS：
   - 将起点加入队列。
   - 立即将它标记为 `'0'`。
   - 当队列非空时：
     - 取出队首元素。
     - 搜索四个方向。
     - 如果相邻格子是陆地：
       - 将其标记为 `'0'`
       - 加入队列。
4. 最终返回岛屿数量。

------

## 代码

```
from typing import List
from collections import deque


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        directions = [
            (1, 0),
            (-1, 0),
            (0, 1),
            (0, -1)
        ]

        ROWS, COLS = len(grid), len(grid[0])
        islands = 0

        def bfs(r: int, c: int) -> None:
            # deque 是 Python 中实现 BFS 队列的常用数据结构
            q = deque()

            # 起点入队
            q.append((r, c))

            # 入队时就立刻标记为 visited
            grid[r][c] = "0"

            while q:
                # 从队列左侧取出最早加入的元素
                row, col = q.popleft()

                # 搜索四个方向
                for dr, dc in directions:
                    nr = row + dr
                    nc = col + dc

                    # 越界则跳过
                    if (
                        nr < 0
                        or nc < 0
                        or nr >= ROWS
                        or nc >= COLS
                    ):
                        continue

                    # 不是陆地则跳过
                    if grid[nr][nc] == "0":
                        continue

                    # 标记为已访问
                    grid[nr][nc] = "0"

                    # 加入 BFS 队列
                    q.append((nr, nc))

        for r in range(ROWS):
            for c in range(COLS):
                if grid[r][c] == "1":
                    islands += 1
                    bfs(r, c)

        return islands
```

------

## `collections.deque` 说明

BFS 中最重要的数据结构是 **队列（Queue）**。

Python 中通常使用：

```
from collections import deque
```

创建队列：

```
q = deque()
```

加入元素：

```
q.append(x)
```

从队首取元素：

```
q.popleft()
```

为什么不推荐普通 `list`？

例如：

```
q = []
q.append(x)
q.pop(0)
```

虽然逻辑上也能实现队列，但：

```
q.pop(0)
```

的时间复杂度是：

\[ O(n) \]

因为删除第一个元素以后，后面的所有元素都需要向前移动。

而：

```
deque.popleft()
```

是：

\[ O(1) \]

因此，Python 中写 BFS 时通常应该优先使用 `deque`。

------

## 一个非常重要的 BFS 细节：什么时候标记 visited？

推荐：

```
grid[nr][nc] = "0"
q.append((nr, nc))
```

即：

> **加入队列时立即标记 visited。**

不要等到 `popleft()` 时才标记。

例如：

```
A B
C D
```

如果 `D` 同时可以从 `B` 和 `C` 到达，而 `D` 入队时没有立即标记，那么：

```
B -> D 入队
C -> D 再次入队
```

同一个节点可能进入队列多次。

因此 BFS 中常见原则是：

> **enqueue 时标记 visited，而不是 dequeue 时。**

------

## 时间复杂度

每个格子最多被访问一次：

\[ O(mn) \]

## 空间复杂度

最坏情况下 BFS 队列可能保存大量陆地：

\[ O(mn) \]

因此：

- **时间复杂度：`O(mn)`**
- **空间复杂度：`O(mn)`**

------

# 3. DFS 与 BFS 对比

两种方法本质上都在做：

> 从一个陆地节点出发，找到整个连通分量。

区别主要在于搜索顺序。

| 方法 | 数据结构        | 搜索方式         | 时间复杂度 | 最坏空间复杂度 |
| ---- | --------------- | ---------------- | ---------- | -------------- |
| DFS  | 递归栈 / 显式栈 | 一条路径深入到底 | `O(mn)`    | `O(mn)`        |
| BFS  | Queue           | 一层一层扩展     | `O(mn)`    | `O(mn)`        |

这道题中，两种方法都很好。

不过 Python 面试中有一个实际问题：

> Python 的递归深度有限。

如果网格特别大，而且整个岛屿形成一条非常长的路径，递归 DFS 有可能触发：

```
RecursionError
```

因此从工程角度来说，**BFS 或迭代 DFS 会更加稳健**。

------

# 4. 并查集 Disjoint Set Union

## 思路

并查集的思考方式与 DFS/BFS 不太一样。

假设一开始把每一个 `'1'` 都看成一个独立岛屿：

```
1 1 0
1 0 1
```

初始可以认为有 4 个岛屿：

```
A B .
C . D
```

但是：

- `A` 和 `B` 相邻
- `A` 和 `C` 相邻

所以：

```
A、B、C
```

实际上属于同一个岛屿。

并查集负责不断执行：

```
union(A, B)
union(A, C)
```

每成功合并两个此前不同的集合：

```
islands -= 1
```

最终剩下的集合数量就是岛屿数量。

------

## 并查集需要理解的两个核心操作

### `find(x)`

找到节点 `x` 所属集合的根节点。

例如：

```
1 -> 2 -> 5
3 -> 5
4 -> 5
```

那么：

```
find(1) == 5
find(3) == 5
find(4) == 5
```

说明它们属于同一个集合。

------

### `union(a, b)`

将 `a` 和 `b` 所属的集合进行合并。

如果：

```
find(a) == find(b)
```

说明它们已经属于同一个集合，不需要再次合并。

------

## 路径压缩 Path Compression

看下面这条父节点链：

```
1 -> 2 -> 3 -> 4 -> 5
```

查询：

```
find(1)
```

需要一路找到 `5`。

路径压缩会在查询之后把它改造成类似：

```
1 ─┐
2 ─┤
3 ─┼-> 5
4 ─┘
```

代码：

```
def find(self, node):
    if self.parent[node] != node:
        self.parent[node] = self.find(self.parent[node])

    return self.parent[node]
```

关键是这一句：

```
self.parent[node] = self.find(self.parent[node])
```

它不仅寻找根节点，同时把路径上的节点直接连接到根节点。

------

## 按集合大小合并 Union by Size

假设：

```
集合 A：100 个节点
集合 B：2 个节点
```

合并时，通常应该让小集合连接到大集合下面。

这样可以避免生成非常深的树。

代码逻辑：

```
if self.size[root_a] >= self.size[root_b]:
    parent[root_b] = root_a
else:
    parent[root_a] = root_b
```

路径压缩 + 按大小合并之后，并查集操作非常接近常数时间。

------

## 代码

下面对原解法稍作整理，变量名按照 Python 常见风格使用小写。

```
from typing import List


class DSU:
    def __init__(self, n: int):
        # 初始状态：
        # 每个节点的父节点都是自己
        self.parent = list(range(n))

        # 每个集合初始只有一个节点
        self.size = [1] * n

    def find(self, node: int) -> int:
        # 路径压缩
        if self.parent[node] != node:
            self.parent[node] = self.find(self.parent[node])

        return self.parent[node]

    def union(self, u: int, v: int) -> bool:
        root_u = self.find(u)
        root_v = self.find(v)

        # 已经属于同一个集合
        if root_u == root_v:
            return False

        # 按集合大小合并：
        # 小集合挂到大集合下面
        if self.size[root_u] >= self.size[root_v]:
            self.parent[root_v] = root_u
            self.size[root_u] += self.size[root_v]
        else:
            self.parent[root_u] = root_v
            self.size[root_v] += self.size[root_u]

        # 表示两个此前独立的集合成功合并
        return True


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        ROWS, COLS = len(grid), len(grid[0])

        # 一共有 ROWS * COLS 个可能的节点
        dsu = DSU(ROWS * COLS)

        def index(r: int, c: int) -> int:
            """
            将二维坐标 (r, c)
            转换成一维节点编号。
            """
            return r * COLS + c

        directions = [
            (1, 0),
            (-1, 0),
            (0, 1),
            (0, -1)
        ]

        islands = 0

        for r in range(ROWS):
            for c in range(COLS):

                if grid[r][c] == "0":
                    continue

                # 暂时把当前陆地看成一个独立岛屿
                islands += 1

                # 尝试与周围陆地合并
                for dr, dc in directions:
                    nr = r + dr
                    nc = c + dc

                    if (
                        nr < 0
                        or nc < 0
                        or nr >= ROWS
                        or nc >= COLS
                    ):
                        continue

                    if grid[nr][nc] == "0":
                        continue

                    # 如果成功合并两个不同集合，
                    # 岛屿数量减少 1
                    if dsu.union(
                        index(r, c),
                        index(nr, nc)
                    ):
                        islands -= 1

        return islands
```

------

## 二维坐标如何映射成一维？

并查集通常使用整数编号：

```
0, 1, 2, 3, ...
```

但是网格中的位置是：

```
(r, c)
```

所以需要：

```
index = r * COLS + c
```

例如：

```
grid:

(0,0) (0,1) (0,2)
(1,0) (1,1) (1,2)
```

如果：

```
COLS = 3
```

那么：

```
(0,0) -> 0
(0,1) -> 1
(0,2) -> 2

(1,0) -> 3
(1,1) -> 4
(1,2) -> 5
```

这种二维坐标转一维索引的技巧在很多矩阵题中都很常见：

```
index = row * number_of_columns + column
```

值得记住。

------

## 并查集复杂度

假设网格有：

\[ mn \]

个节点。

每个格子最多检查 4 个邻居，因此一共进行 `O(mn)` 次 `find / union` 操作。

使用：

- Path Compression
- Union by Size

后，单次并查集操作的摊还复杂度为：

\[ O(\alpha(mn)) \]

其中：

\[ \alpha \]

是反阿克曼函数（Inverse Ackermann Function）。

它增长极其缓慢，在现实规模的数据中基本可以视为一个非常小的常数。

因此更严格地说：

\[ O(mn\alpha(mn)) \]

通常也可以简化认为：

\[ O(mn) \]

空间复杂度：

\[ O(mn) \]

因为需要保存：

```
parent
size
```

两个数组。

------

# 5. 并查集代码可以进一步优化

原代码对每块陆地都检查：

```
上
下
左
右
```

这实际上会重复处理邻接关系。

例如：

```
A B
```

处理 `A` 时：

```
A -> B
```

处理 `B` 时又会：

```
B -> A
```

虽然并查集能够识别已经合并，因此不会产生错误，但是存在一些重复操作。

遍历网格时，可以只检查：

```
右
下
```

因为：

- 左边已经处理过
- 上边已经处理过

这样代码会更加高效一些。

```
directions = [
    (1, 0),   # 下
    (0, 1),   # 右
]
```

不过面试中使用四方向版本同样是正确的，而且通常更加直观。

------

# 6. 常见错误

## 错误一：没有标记 visited

DFS / BFS 最大的常见错误之一就是：

> 搜索过一个陆地以后，没有标记它已经访问。

错误示例：

```
def dfs(r, c):
    if grid[r][c] == "1":
        dfs(r + 1, c)
        dfs(r - 1, c)
```

假设两个相邻格子：

```
A B
```

那么可能出现：

```
A -> B
B -> A
A -> B
B -> A
...
```

造成无限递归。

正确做法：

```
def dfs(r, c):
    if grid[r][c] == "1":
        # 先标记 visited
        grid[r][c] = "0"

        dfs(r + 1, c)
```

原则是：

> **访问节点以后，尽早标记 visited。**

------

# 7. 错误二：把每个 `'1'` 都计算成一个岛屿

错误：

```
for r in range(ROWS):
    for c in range(COLS):
        if grid[r][c] == "1":
            islands += 1
```

例如：

```
1 1
1 1
```

这里有：

```
4 个陆地格子
```

但只有：

```
1 个岛屿
```

正确理解应该是：

> 只有在发现一个 **尚未访问过的新连通分量** 时才执行 `islands += 1`。

所以：

```
if grid[r][c] == "1":
    islands += 1
    dfs(r, c)
```

DFS 会负责把当前整个岛屿全部标记掉。

------

# 8. 错误三：边界判断顺序错误

错误：

```
if grid[nr][nc] == "1" and 0 <= nr < ROWS:
```

这里的问题是：

Python 会先执行：

```
grid[nr][nc]
```

如果 `nr` 或 `nc` 已经越界，程序会直接报：

```
IndexError
```

正确写法：

```
if (
    0 <= nr < ROWS
    and 0 <= nc < COLS
    and grid[nr][nc] == "1"
):
```

Python 的 `and` 使用 **短路求值（Short-Circuit Evaluation）**。

如果：

```
0 <= nr < ROWS
```

已经是 `False`，后面的条件就不会继续执行。

因此不会访问非法索引。

------

# 9. Python 的链式比较

Python 支持：

```
0 <= nr < ROWS
```

它相当于：

```
0 <= nr and nr < ROWS
```

所以通常可以把边界条件写成：

```
if (
    0 <= nr < ROWS
    and 0 <= nc < COLS
    and grid[nr][nc] == "1"
):
```

相比：

```
nr >= 0 and nr < ROWS
```

更加符合 Python 的常见写法。

------

# 10. 错误四：错误地把对角线当作连接

例如：

```
1 0
0 1
```

题目规定只有：

```
上
下
左
右
```

可以连接。

因此这里是：

```
2 个岛屿
```

而不是：

```
1 个岛屿
```

所以只能使用：

```
directions = [
    (1, 0),
    (-1, 0),
    (0, 1),
    (0, -1)
]
```

不能添加：

```
(1, 1)
(1, -1)
(-1, 1)
(-1, -1)
```

------

# 11. 一个容易忽略的问题：这些方法修改了输入 `grid`

DFS 和 BFS 解法中都有：

```
grid[r][c] = "0"
```

这意味着：

> 函数执行结束以后，原来的 `grid` 会被修改。

这在 LeetCode 这道题里通常没有问题，并且可以避免创建额外的：

```
visited
```

集合。

但是实际面试中可以主动说明：

> “这里我直接修改输入网格来标记 visited。如果题目要求不能修改输入，我会使用一个 `visited` 集合。”

例如：

```
visited = set()
```

然后：

```
visited.add((r, c))
```

判断：

```
if (r, c) in visited:
    return
```

这样不会修改原始 `grid`。

------

# 12. 面试中推荐的解法

对于这道题，通常最推荐：

### 第一选择：DFS

优点：

- 思路最直接
- 代码短
- 很容易解释
- 非常典型的 Grid DFS

核心模板：

```
for 每个格子:
    if 是未访问陆地:
        answer += 1
        dfs()
```

------

### 第二选择：BFS

如果担心 Python 递归深度，可以选择 BFS：

```
deque
```

通常更加稳健。

------

### 并查集

并查集并不是这道题最简单的方法。

但它非常适合展示：

- 连通分量思想
- Union-Find
- Path Compression
- Union by Size / Rank

并且如果题目进一步变成：

> 不断动态增加陆地，每次增加之后都询问当前有多少岛屿

例如经典变体 **Number of Islands II**，那么并查集通常会变得非常有优势。

------

# 13. 面试记忆模板

这道题可以记成一个非常通用的 **Grid Connected Components 模板**：

```
for r in range(ROWS):
    for c in range(COLS):

        if 当前节点满足条件并且尚未访问:

            # 找到一个新的连通分量
            answer += 1

            # DFS / BFS
            # 把整个连通分量全部访问
```

很多题都可以套用这个结构，例如：

- Number of Islands
- Max Area of Island
- Flood Fill
- Surrounded Regions
- Rotting Oranges
- Pacific Atlantic Water Flow
- Walls and Gates

其中最值得记住的一句话是：

> **外层循环负责找到新的连通分量，DFS/BFS 负责把整个连通分量遍历完。**