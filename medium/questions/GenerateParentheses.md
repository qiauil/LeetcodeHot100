# 括号生成（Generate Parentheses）

## 题目描述

给定一个整数 `n`，请返回所有能够由 `n` 对括号组成的、**合法且完整的括号字符串**。

合法括号字符串要求：

- 每一个右括号 `)` 都必须有对应的左括号 `(`；
- 任意前缀中，右括号数量都不能超过左括号数量；
- 最终左括号和右括号数量都恰好为 `n`。

结果可以按照任意顺序返回。

---

## 示例 1

```text
输入：
n = 1

输出：
["()"]
```

---

## 示例 2

```text
输入：
n = 3

输出：
[
    "((()))",
    "(()())",
    "(())()",
    "()(())",
    "()()()"
]
```

---

## 约束

- `1 <= n <= 7`

---

# 一、暴力枚举

## 核心思路

长度为 `2n` 的字符串，每一个位置只有两种选择：

```text
(
)
```

因此可以先生成所有可能的长度为 `2n` 的括号字符串。

一共有：

```text
2^(2n)
```

种候选字符串。

然后对每个完整字符串判断它是否为合法括号字符串。

这种方法的特点是：

> 先把所有可能情况都生成出来，再过滤掉非法结果。

它非常直观，但会生成大量明显不合法的字符串。

---

# 如何判断一个括号字符串是否合法？

维护一个变量：

```python
balance
```

表示当前还有多少个左括号没有被匹配。

扫描字符串：

- 遇到 `(`：

```python
balance += 1
```

- 遇到 `)`：

```python
balance -= 1
```

如果过程中：

```python
balance < 0
```

说明某个前缀中右括号数量已经超过左括号数量。

例如：

```text
")("
```

第一个字符就是：

```text
)
```

此时：

```text
balance = -1
```

立即可以判断非法。

最后还需要：

```python
balance == 0
```

表示所有左括号都被正确关闭。

---

## 合法性条件总结

一个括号字符串合法，当且仅当：

```text
1. 扫描过程中 balance 永远不小于 0
2. 扫描结束后 balance == 0
```

---

## 算法步骤

1. 定义 `dfs(s)` 构造字符串。
2. 如果：

```python
len(s) == 2 * n
```

说明当前字符串已经完整。
3. 使用 `valid(s)` 判断它是否为合法括号字符串。
4. 如果合法，加入结果。
5. 如果长度还没到 `2n`：
   - 尝试添加 `(`；
   - 尝试添加 `)`。
6. 返回所有合法结果。

---

## Python 实现

```python
from typing import List


class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        res = []

        def valid(s: str) -> bool:
            balance = 0

            for c in s:
                if c == "(":
                    balance += 1
                else:
                    balance -= 1

                # 某个前缀中右括号过多，直接判定非法
                if balance < 0:
                    return False

            # 最后所有左括号都必须被关闭
            return balance == 0

        def dfs(s: str) -> None:
            # 已经生成长度为 2n 的完整字符串
            if len(s) == 2 * n:
                if valid(s):
                    res.append(s)
                return

            # 暴力尝试两种选择
            dfs(s + "(")
            dfs(s + ")")

        dfs("")

        return res
```

---

## Python 细节说明：`return not balance`

原代码中写：

```python
return not open
```

如果：

```python
open == 0
```

那么：

```python
not 0
```

结果是：

```python
True
```

否则非零整数会被视为真值：

```python
not 3 == False
```

因此：

```python
return not balance
```

在这里等价于：

```python
return balance == 0
```

不过面试中通常更推荐后者，因为语义更加清楚。

---

## 复杂度分析

一共有：

```text
2^(2n) = 4^n
```

个长度为 `2n` 的候选字符串。

每一个完整字符串还需要：

```text
O(n)
```

时间进行合法性验证。

因此：

- **时间复杂度：`O(n * 4^n)`**

如果把所有生成过程和结果字符串都计算在内，空间最坏也可以达到：

- **空间复杂度：`O(n * 4^n)`**

递归深度本身为：

```text
O(n)
```

---

## 暴力法的问题

大量字符串其实很早就已经确定非法。

例如：

```text
")..."
```

第一个字符就是右括号，这个分支永远不可能成为合法答案。

但暴力法仍然会继续把它补到长度：

```text
2n
```

然后才验证。

因此更好的方法是：

> 在构造过程中就禁止非法选择。

这正是回溯剪枝的思路。

---

# 二、回溯（Backtracking）

## 核心思路

与暴力法不同，回溯法不会生成所有字符串再判断。

它只构造：

```text
仍然有可能成为合法答案的字符串
```

维护两个变量：

```python
openN
closedN
```

分别表示当前已经使用了多少个：

```text
(
)
```

合法括号字符串始终需要满足两个重要条件。

---

# 条件 1：左括号不能超过 `n`

总共只有：

```text
n
```

个左括号。

所以只有在：

```python
openN < n
```

时，才能继续添加：

```text
(
```

---

# 条件 2：右括号数量不能超过左括号数量

只有当前已经存在尚未匹配的左括号时，才能添加右括号。

也就是：

```python
closedN < openN
```

才能添加：

```text
)
```

例如：

```text
当前字符串 = "(("
```

此时：

```text
openN = 2
closedN = 0
```

可以添加右括号。

但如果当前：

```text
"()"
```

那么：

```text
openN = 1
closedN = 1
```

不能继续添加：

```text
)
```

否则会得到：

```text
"())"
```

这已经不合法。

---

# 为什么 `closedN < openN` 是核心条件？

括号合法性的本质就是：

> 任意前缀中，右括号数量都不能超过左括号数量。

即：

```text
closedN <= openN
```

而添加一个新的右括号之前，需要确保添加后仍满足这个关系。

因此添加前必须有：

```text
closedN < openN
```

---

# 决策树理解

假设：

```text
n = 3
```

初始：

```text
open = 0
close = 0
```

第一个字符只能是：

```text
(
```

因为：

```text
close < open
```

当前并不成立，所以不能先放 `)`。

随着搜索继续，每一步只允许合法选择。

因此回溯树中不会出现：

```text
")("
"())"
"))("
```

这类一定非法的前缀。

这就是“剪枝”。

---

## 算法步骤

1. 使用列表 `stack` 保存当前括号路径。
2. 使用：
   - `openN`：当前左括号数量；
   - `closedN`：当前右括号数量。
3. 如果：

```python
openN == closedN == n
```

说明已经得到一个完整合法答案。
4. 如果：

```python
openN < n
```

可以添加 `(`。
5. 如果：

```python
closedN < openN
```

可以添加 `)`。
6. 每次递归结束后撤销当前选择。
7. 返回全部结果。

---

## Python 实现

```python
from typing import List


class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        res = []

        # 当前正在构造的括号序列
        stack = []

        def backtrack(openN: int, closedN: int) -> None:
            # 左右括号都已经使用 n 个
            # 当前字符串一定完整且合法
            if openN == closedN == n:
                res.append("".join(stack))
                return

            # 只要左括号还没有用完，就可以添加 '('
            if openN < n:
                stack.append("(")

                backtrack(openN + 1, closedN)

                # 撤销选择
                stack.pop()

            # 只有存在尚未匹配的左括号时，
            # 才能添加 ')'
            if closedN < openN:
                stack.append(")")

                backtrack(openN, closedN + 1)

                # 撤销选择
                stack.pop()

        backtrack(0, 0)

        return res
```

---

# 为什么这里是真正典型的“回溯”？

核心代码结构：

```python
stack.append("(")

backtrack(...)

stack.pop()
```

对应标准回溯模板：

```text
做选择
→ 递归搜索
→ 撤销选择
```

因为：

```python
stack
```

是一个可变列表。

不同递归分支共享同一个 `stack` 对象。

因此当前分支结束后必须：

```python
stack.pop()
```

恢复到进入当前分支之前的状态。

否则后续其他分支会受到污染。

---

# Python 函数说明：`"".join(stack)`

`stack` 是字符列表。

例如：

```python
stack = ["(", "(", ")", ")"]
```

执行：

```python
"".join(stack)
```

得到：

```text
"(())"
```

`join` 是把字符串列表拼接成一个完整字符串的常用方式。

---

# 回溯法为什么比暴力法高效？

暴力法会探索整棵二叉树：

```text
每一位：
(
或
)
```

因此叶子数量：

```text
2^(2n) = 4^n
```

而回溯法会提前剪掉所有不合法分支。

例如以下前缀根本不会产生：

```text
")"
"())"
"()))"
```

所以实际访问的状态少得多。

---

## 复杂度分析

合法括号字符串的数量不是 `4^n`，而是第 `n` 个 **Catalan Number（卡特兰数）**：

```text
C_n = 1 / (n + 1) * binom(2n, n)
```

其渐近规模约为：

```text
C_n = O(4^n / n^(3/2))
```

每个答案长度为：

```text
2n
```

因此生成全部输出至少需要：

```text
O(n * C_n)
```

时间。

所以更精确地说：

- **时间复杂度：`O(n * C_n)`**
- **输出空间复杂度：`O(n * C_n)`**
- **额外递归空间复杂度：`O(n)`**

很多面试资料也会用比较宽松的上界：

```text
O(n * 4^n)
```

来描述。

但如果追求更精确的分析，卡特兰数更合适。

---

# 卡特兰数为什么会出现在这里？

合法括号序列数量是经典的卡特兰数问题。

前几个值为：

```text
n = 1 -> 1
n = 2 -> 2
n = 3 -> 5
n = 4 -> 14
n = 5 -> 42
```

例如：

```text
n = 3
```

恰好有：

```text
5
```

种合法括号组合：

```text
((()))
(()())
(())()
()(())
()()()
```

---

# 三、动态规划（Dynamic Programming）

## 核心思路

合法括号字符串也可以由更小规模的合法括号字符串组合得到。

任意一个合法括号字符串，都可以写成：

```text
( left ) right
```

其中：

- `left` 本身是一个合法括号字符串；
- `right` 也是一个合法括号字符串。

假设当前需要生成：

```text
k
```

对括号。

如果 `left` 使用：

```text
i
```

对括号，那么：

```text
( left )
```

一共使用：

```text
i + 1
```

对括号。

剩余给 `right` 的括号数量就是：

```text
k - i - 1
```

因此：

```text
dp[k]
```

可以由所有：

```text
dp[i]
```

和：

```text
dp[k - i - 1]
```

组合出来。

---

# 状态定义

定义：

```python
dp[k]
```

表示：

> 所有由 `k` 对括号组成的合法括号字符串。

基础情况：

```python
dp[0] = [""]
```

---

# 为什么 `dp[0] = [""]`？

这点非常重要。

零对括号只有一种“合法结构”：

```text
空字符串
```

它不是题目在 `n = 0` 时要求返回的业务答案，而是动态规划组合过程中的**单位元素**。

例如：

```text
k = 1
i = 0
```

公式：

```text
"(" + left + ")" + right
```

其中：

```text
left = ""
right = ""
```

才能构造：

```text
"()"
```

如果：

```python
dp[0] = []
```

就无法构造任何更大的答案。

---

# 状态转移

对于：

```text
k = 1 ... n
```

枚举：

```text
i = 0 ... k - 1
```

然后：

```python
for left in dp[i]:
    for right in dp[k - i - 1]:
        dp[k].append("(" + left + ")" + right)
```

也就是：

```text
( left ) right
```

---

# 一个例子：`n = 3`

已知：

```text
dp[0] = [""]
dp[1] = ["()"]
dp[2] = ["()()", "(())"]
```

构造：

```text
dp[3]
```

---

## 当 `i = 0`

```text
left  使用 0 对
right 使用 2 对
```

得到：

```text
()()()
()(())
```

---

## 当 `i = 1`

```text
left  使用 1 对
right 使用 1 对
```

得到：

```text
(())()
```

---

## 当 `i = 2`

```text
left  使用 2 对
right 使用 0 对
```

得到：

```text
(()())
((()))
```

最终共：

```text
5
```

个结果。

---

## Python 实现

原始代码可以稍微调整循环起点，使含义更加清楚：

```python
from typing import List


class Solution:
    def generateParenthesis(self, n: int) -> List[str]:
        # dp[k]：所有由 k 对括号组成的合法字符串
        dp = [[] for _ in range(n + 1)]

        # 组合过程的基础情况
        dp[0] = [""]

        # 从 1 对括号开始逐步构造
        for k in range(1, n + 1):
            # left 使用 i 对括号
            # right 使用 k - i - 1 对括号
            for i in range(k):
                for left in dp[i]:
                    for right in dp[k - i - 1]:
                        dp[k].append(
                            "(" + left + ")" + right
                        )

        return dp[n]
```

---

# 为什么这个公式不会漏掉答案？

每一个非空合法括号字符串的第一个字符一定是：

```text
(
```

它一定存在一个与之匹配的：

```text
)
```

那么这个匹配的右括号就把字符串分成：

```text
( left ) right
```

其中：

```text
left
```

和：

```text
right
```

都必须分别合法。

所以所有合法括号字符串都一定能被这种结构表示。

---

# 为什么不会重复？

对于一个确定的合法括号字符串：

```text
( left ) right
```

第一个左括号所匹配的右括号位置是唯一的。

因此：

- `left` 使用多少对括号唯一；
- `right` 使用多少对括号也唯一。

所以这种分解是唯一的，不会因为不同拆分产生重复答案。

---

## 复杂度分析

最终答案数量是：

```text
C_n
```

每个答案长度是：

```text
2n
```

因此至少需要：

```text
O(n * C_n)
```

时间和输出空间。

DP 还会保存：

```text
dp[0], dp[1], ..., dp[n]
```

所有较小规模的结果。

因此整体空间也与累计输出规模相关。

常见近似写法：

- **时间复杂度：`O(n * C_n)`**
- **空间复杂度：`O(n * C_n)`**

也可以使用较宽松上界：

```text
O(n * 4^n)
```

---

# 回溯 vs 动态规划

两者都会生成所有答案，但思路不同。

## 回溯

是：

```text
从空字符串开始
→ 一步一步做合法选择
→ 构造完整答案
```

属于：

```text
自顶向下搜索
```

---

## 动态规划

是：

```text
先知道小规模答案
→ 再组合成更大规模答案
```

属于：

```text
自底向上构造
```

对于这道题，回溯代码更简单、更自然，因此通常是面试首选。

---

# 常见错误

## 1. 添加右括号的条件写错

错误：

```python
if closedN < n:
    ...
```

这个条件只保证：

```text
右括号总数没有超过 n
```

但无法保证当前前缀合法。

例如：

```text
openN = 1
closedN = 1
```

如果只检查：

```text
closedN < n
```

仍然允许添加右括号，得到：

```text
())
```

这是非法的。

正确条件是：

```python
if closedN < openN:
```

因为只有存在尚未匹配的左括号，才能添加右括号。

---

# 2. 忘记撤销选择

如果使用：

```python
stack
```

这样的可变列表：

```python
stack.append("(")
backtrack(...)
```

递归返回以后必须：

```python
stack.pop()
```

否则当前分支加入的字符会残留到下一条分支。

标准模板：

```python
stack.append(choice)
backtrack(...)
stack.pop()
```

---

# 3. 基础条件只看字符串长度，但分支条件本身写错

在正确剪枝的前提下：

```python
len(stack) == 2 * n
```

和：

```python
openN == closedN == n
```

最终是等价的。

但更推荐写：

```python
if openN == closedN == n:
```

因为它直接表达：

```text
n 个左括号
+
n 个右括号
```

都已经使用完毕。

同时，如果分支条件存在 bug，这种写法也更容易暴露问题。

---

# 4. 把 `closedN <= openN` 用作添加右括号条件

错误：

```python
if closedN <= openN:
    stack.append(")")
```

假设：

```text
closedN == openN
```

当前所有左括号都已经匹配。

此时再添加 `)` 就会立即使：

```text
closedN > openN
```

形成非法前缀。

因此必须严格是：

```python
closedN < openN
```

---

# 5. 左括号条件写成 `openN <= n`

错误：

```python
if openN <= n:
```

当：

```text
openN == n
```

时还会再添加一个左括号，使左括号数量达到：

```text
n + 1
```

所以正确条件是：

```python
openN < n
```

---

# 6. 混淆 `dp[0] = [""]` 和空输入答案

动态规划中：

```python
dp[0] = [""]
```

是为了让组合公式成立。

它表示：

```text
“零对括号存在一种空结构”
```

这是 DP 的数学基础状态。

而本题约束：

```text
n >= 1
```

所以不会真正要求返回 `dp[0]`。

---

# 解法对比

| 方法 | 时间复杂度 | 额外空间 / 输出规模 | 面试推荐程度 |
|---|---:|---:|---|
| 暴力枚举 | `O(n * 4^n)` | 很大 | 不推荐 |
| 回溯 | `O(n * C_n)` | `O(n)` 递归栈，不含输出 | **最推荐** |
| 动态规划 | `O(n * C_n)` | 保存各规模结果 | 推荐理解 |

其中：

```text
C_n
```

是第 `n` 个卡特兰数。

---

# 面试最推荐的写法

```python
class Solution:
    def generateParenthesis(self, n: int) -> list[str]:
        res = []
        path = []

        def backtrack(openN: int, closeN: int) -> None:
            if openN == closeN == n:
                res.append("".join(path))
                return

            if openN < n:
                path.append("(")
                backtrack(openN + 1, closeN)
                path.pop()

            if closeN < openN:
                path.append(")")
                backtrack(openN, closeN + 1)
                path.pop()

        backtrack(0, 0)

        return res
```

核心只需要记住两个条件：

```python
if openN < n:
    # 可以放 '('
```

以及：

```python
if closeN < openN:
    # 可以放 ')'
```

---

# 面试时如何快速解释正确性？

可以这样理解：

```text
左括号：
只要还没有使用完 n 个，就可以继续放。

右括号：
只有当前存在尚未匹配的左括号，才能放。
```

因此整个搜索过程中始终满足：

```text
0 <= closeN <= openN <= n
```

当：

```text
openN == closeN == n
```

时，自然得到一个合法完整答案。

---

# 进一步总结：回溯和暴力 DFS 的区别

两种方法看起来都使用 DFS，但核心区别在于：

## 暴力 DFS

```text
先生成
再验证
```

每一步都尝试：

```text
(
)
```

即使当前前缀已经确定不可能合法，也继续搜索。

---

## 回溯

```text
边生成
边保证合法
```

只在满足条件时进入下一层。

这就是：

```text
剪枝
```

因此回溯通常可以理解为：

> DFS + 合法性约束 + 撤销选择

---

# 与上一类“电话号码字母组合”回溯题的区别

电话号码字母组合中：

```text
每一个候选字符都天然合法
```

所以每一层只需要遍历候选项。

而本题中：

```text
并不是每个 '(' 或 ')' 都可以随时选择
```

需要额外使用：

```python
openN < n
closedN < openN
```

来剪掉非法分支。

因此这道题非常适合理解：

> 回溯不仅是枚举，还可以在搜索过程中利用约束进行剪枝。

---

# 总结

这道题最重要的三个知识点是：

```text
1. 合法括号的前缀中：
   右括号数量永远不能超过左括号数量

2. 回溯时只允许：
   openN < n 时添加 '('
   closeN < openN 时添加 ')'

3. 生成结果数量属于卡特兰数：
   C_n
```

面试中通常直接使用：

```text
Backtracking
```

就是最自然、最清晰的解法。

可以把整个算法记成一句话：

> **左括号没用完就可以放；右括号只有在左括号更多时才能放；两者都达到 n 时得到一个答案。**
