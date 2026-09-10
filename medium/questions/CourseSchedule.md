# 课程表（Course Schedule）

给定一个数组 `prerequisites`，其中：

```
prerequisites[i] = [a, b]
```

表示如果想学习课程 `a`，必须先完成课程 `b`。

例如，`[0, 1]` 表示必须先学习课程 `1`，然后才能学习课程 `0`。

总共有 `numCourses` 门课程，编号从 `0` 到 `numCourses - 1`。

如果可以完成所有课程，返回 `True`；否则返回 `False`。

## 示例 1

```
输入：numCourses = 2, prerequisites = [[0, 1]]
输出：True
```

解释：可以先学习课程 `1`，再学习课程 `0`。

## 示例 2

```
输入：numCourses = 2, prerequisites = [[0, 1], [1, 0]]
输出：False
```

解释：

- 学习课程 `0` 之前必须先学习课程 `1`
- 学习课程 `1` 之前又必须先学习课程 `0`

两门课程互相依赖，形成了环，因此无法完成所有课程。

## 数据范围

- `1 <= numCourses <= 1000`
- `0 <= prerequisites.length <= 1000`
- `prerequisites[i].length == 2`
- `0 <= a, b < numCourses`
- 所有先修课程关系都是唯一的

------

# 核心思想：判断有向图中是否存在环

可以把课程关系建模为一张有向图：

- 每门课程是一个节点
- `[a, b]` 表示课程 `b` 指向课程 `a`
- 这条边表示：完成 `b` 后才能学习 `a`

如果图中存在环，例如：

```
A → B → C → A
```

那么无论从哪门课程开始，都无法满足所有先修条件。

因此，本题本质上是：

> 判断一张有向图中是否存在环。

常见解法有两种：

1. DFS 深度优先搜索检测环
2. BFS 拓扑排序（Kahn 算法）

------

# 解法一：DFS 检测环

## 思路

进行 DFS 时，需要区分三种状态：

- `0`：课程尚未访问
- `1`：课程正在当前 DFS 路径中
- `2`：课程已经处理完成，从该课程开始不存在环

如果在 DFS 过程中再次遇到状态为 `1` 的课程，说明我们沿着当前路径回到了之前的节点，因此存在环。

例如：

```
0 → 1 → 2
    ↑   ↓
    └───┘
```

搜索到课程 `2` 后，又回到了当前搜索路径中的课程 `1`，因此检测到环。

## 算法步骤

1. 根据先修关系构建邻接表。
2. 使用数组 `state` 记录每门课程的访问状态。
3. 对每门尚未访问的课程执行 DFS。
4. DFS 过程中：
   - 遇到状态 `1`，说明存在环。
   - 遇到状态 `2`，说明该课程已经检查完毕，可以直接返回。
5. 如果所有课程都没有检测到环，则可以完成全部课程。

## Python 代码

```
from typing import List


class Solution:
    def canFinish(
        self,
        numCourses: int,
        prerequisites: List[List[int]]
    ) -> bool:
        # graph[course] 保存学习 course 之后可以继续学习的课程
        graph = [[] for _ in range(numCourses)]

        for course, prerequisite in prerequisites:
            # prerequisite -> course
            graph[prerequisite].append(course)

        # state[i] 表示课程 i 的访问状态：
        # 0：尚未访问
        # 1：正在当前 DFS 路径中
        # 2：已经处理完成
        state = [0] * numCourses

        def has_cycle(course: int) -> bool:
            # 当前路径中再次遇到该课程，说明存在环
            if state[course] == 1:
                return True

            # 该课程之前已经处理完成，不需要重复搜索
            if state[course] == 2:
                return False

            # 将课程标记为“正在访问”
            state[course] = 1

            # 检查它指向的后续课程
            for next_course in graph[course]:
                if has_cycle(next_course):
                    return True

            # 当前课程及其后续课程均不存在环
            state[course] = 2
            return False

        # 图可能由多个互不连通的部分组成，因此必须检查所有课程
        for course in range(numCourses):
            if has_cycle(course):
                return False

        return True
```

## 复杂度分析

设：

- `V` 为课程数量，即 `numCourses`
- `E` 为先修关系数量，即 `len(prerequisites)`

每个节点和每条边最多被访问一次：

- 时间复杂度：`O(V + E)`
- 空间复杂度：`O(V + E)`

空间主要用于邻接表、状态数组以及递归调用栈。

## 三种状态为什么重要？

只使用普通的 `visited` 集合无法区分：

- 节点正在当前递归路径中
- 节点已经在之前的搜索中处理完成

而检测有向图中的环，关键是判断一个节点是否出现在当前递归路径中。

例如：

```
0 → 1
 \
  → 2 → 1
```

课程 `1` 会被访问两次，但图中并没有环。第二次遇到已经处理完成的课程 `1`，不应该判断为环。

## Python 递归深度说明

Python 默认递归深度通常接近 `1000`。本题最多有 `1000` 门课程，如果依赖关系形成一条很长的链，DFS 可能接近递归深度限制。

在实际面试中可以说明：

- DFS 思路清晰，但 Python 中要注意递归深度。
- Kahn 拓扑排序使用迭代方式，不存在递归栈溢出问题。

------

# 解法二：拓扑排序（Kahn 算法）

## 思路

`indegree[i]` 表示课程 `i` 还有多少门先修课程没有完成，也就是节点 `i` 的入度。

如果一门课程的入度为 `0`，说明它没有尚未完成的先修课程，可以立即学习。

每完成一门课程，就删除它指向其他课程的边，并减少这些课程的入度。

不断重复这个过程：

- 如果最终完成了全部课程，说明图中没有环。
- 如果队列为空时仍有课程未完成，说明剩余课程之间存在环。

## 算法步骤

1. 建立从先修课程指向后续课程的邻接表。
2. 统计每门课程的入度。
3. 将所有入度为 `0` 的课程加入队列。
4. 不断从队列中取出课程：
   - 将完成课程数量加一。
   - 将其后续课程的入度减一。
   - 如果某门后续课程的入度变成 `0`，将其加入队列。
5. 判断最终完成的课程数量是否等于 `numCourses`。

## Python 代码

```
from collections import deque
from typing import List


class Solution:
    def canFinish(
        self,
        numCourses: int,
        prerequisites: List[List[int]]
    ) -> bool:
        # indegree[i] 表示课程 i 需要先完成的课程数量
        indegree = [0] * numCourses

        # graph[i] 表示完成课程 i 后可以继续学习的课程
        graph = [[] for _ in range(numCourses)]

        for course, prerequisite in prerequisites:
            # prerequisite -> course
            graph[prerequisite].append(course)
            indegree[course] += 1

        # 将所有没有先修课程的课程加入队列
        queue = deque()

        for course in range(numCourses):
            if indegree[course] == 0:
                queue.append(course)

        finished_courses = 0

        while queue:
            # popleft() 可以在 O(1) 时间内取出队首元素
            course = queue.popleft()
            finished_courses += 1

            # 完成当前课程后，更新后续课程的入度
            for next_course in graph[course]:
                indegree[next_course] -= 1

                # 所有先修课程均已完成
                if indegree[next_course] == 0:
                    queue.append(next_course)

        return finished_courses == numCourses
```

## 复杂度分析

每门课程最多进入和离开队列一次，每条边也只会被处理一次：

- 时间复杂度：`O(V + E)`
- 空间复杂度：`O(V + E)`

------

# `deque` 的额外说明

`deque` 是 Python `collections` 模块提供的双端队列：

```
from collections import deque
```

常用操作包括：

```
queue = deque()

queue.append(value)      # 从右侧加入元素，O(1)
queue.appendleft(value)  # 从左侧加入元素，O(1)

queue.pop()              # 从右侧删除元素，O(1)
queue.popleft()          # 从左侧删除元素，O(1)
```

拓扑排序需要频繁删除队首元素，因此应该使用：

```
queue.popleft()
```

不建议使用普通列表的：

```
queue.pop(0)
```

因为 `pop(0)` 删除第一个元素后，需要移动后面的所有元素，时间复杂度为 `O(n)`。

------

# 两种方法对比

| 方法          | 核心思想                  | 判断环的方式             | 优点                                   |
| ------------- | ------------------------- | ------------------------ | -------------------------------------- |
| DFS           | 沿依赖关系进行深度搜索    | 遇到当前递归路径中的节点 | 代码直观，适合直接检测环               |
| Kahn 拓扑排序 | 不断处理入度为 `0` 的节点 | 最终无法处理全部节点     | 无递归深度问题，还可以生成课程学习顺序 |

对于 Python，这道题通常更推荐 Kahn 拓扑排序，因为它完全使用迭代方式，更加稳健。

------

# 常见错误

## 1. DFS 只使用一个全局 `visited` 集合

下面的写法无法区分节点是“正在访问”还是“已经处理完成”：

```
visited = set()


def dfs(node):
    if node in visited:
        # 错误：重复访问不一定表示存在环
        return False

    visited.add(node)

    for neighbor in graph[node]:
        if not dfs(neighbor):
            return False

    return True
```

正确做法是使用三种状态，或者分别维护：

- `visiting`：当前 DFS 路径中的节点
- `visited`：已经完全处理结束的节点

## 2. 忘记检查非连通部分

课程图不一定全部连通。

错误写法：

```
return not has_cycle(0)
```

如果环出现在不包含课程 `0` 的部分，这种写法就无法检测到。

正确写法：

```
for course in range(numCourses):
    if has_cycle(course):
        return False

return True
```

## 3. 邻接表方向与入度含义不一致

`[a, b]` 表示先学习 `b`，再学习 `a`，因此拓扑排序通常建立：

```
b → a
```

对应代码：

```
graph[b].append(a)
indegree[a] += 1
```

即：

```
for course, prerequisite in prerequisites:
    graph[prerequisite].append(course)
    indegree[course] += 1
```

如果把边反向建立，虽然“图中是否存在环”的判断结果仍然不会改变，但 `indegree` 就不再表示课程的先修课数量，代码含义与算法解释会不一致。

## 4. 将无环误认为只有一个合法顺序

一张无环图可能有多个合法的课程顺序。

例如：

```
0 → 2
1 → 2
```

下面两个顺序都合法：

```
0, 1, 2
1, 0, 2
```

本题只要求判断是否可以完成所有课程，不要求返回具体的课程顺序。