# Longest Valid Parentheses（最长有效括号）

给定一个只包含字符：

```text
'('
')'
```

的字符串 `s`，返回其中 **最长有效括号子串** 的长度。

所谓有效括号字符串（well-formed parentheses），要求每一个左括号 `(` 都能够和一个右括号 `)` 正确匹配，并且括号的嵌套关系合法。

例如：

```text
"()"
"(())"
"()()"
"(()())"
```

都是有效括号字符串。

而：

```text
"(()"
")("
"())("
```

都不是完整的有效括号字符串。

------

## 示例

### 示例 1

```text
Input:
s = "(()"

Output:
2
```

因为最长有效括号子串是：

```text
"()"
```

长度为：

```text
2
```

------

### 示例 2

```text
Input:
s = ")()())"

Output:
4
```

最长有效括号子串是：

```text
"()()"
```

长度为：

```text
4
```

------

### 示例 3

```text
Input:
s = ""

Output:
0
```

------

# 一、问题的核心

这道题最容易混淆的地方在于：

> 题目要求的是最长有效括号 **substring（子串）**，而不是 subsequence（子序列）。

Substring 必须是连续的。

例如：

```text
s = ")()())"
```

虽然整个字符串中存在多个可以匹配的括号，但我们要求的是某一段连续区间：

```text
"()()"
```

长度为 `4`。

------

# 解法一：Stack（栈）

## 思路

这是这道题最经典、最直观的解法之一。

括号匹配问题通常会让我们想到栈，因为：

```text
(
```

可以暂时存入栈中，等待之后的：

```text
)
```

与它匹配。

但是，这道题不仅需要判断括号是否合法，还需要计算：

```text
最长连续有效区间的长度
```

因此，我们不能只存储括号字符，而应该存储：

```text
括号对应的下标 index
```

这样，当找到一个有效区间时，就可以直接计算长度。

------

## 核心技巧：栈中保存“最后一个无法匹配的位置”

初始化：

```python
stack = [-1]
```

这里的：

```text
-1
```

不是字符串中的真实下标。

它是一个：

```text
boundary / base index
```

也就是计算有效区间长度时的“边界”。

例如：

```text
s = "()"
```

索引：

```text
0 1
( )
```

遇到 `(`：

```text
stack = [-1, 0]
```

遇到 `)`：

先弹出：

```text
0
```

此时：

```text
stack = [-1]
```

那么当前有效括号长度：

```text
i - stack[-1]
=
1 - (-1)
=
2
```

刚好得到：

```text
"()"
```

的长度。

------

## 算法步骤

遍历字符串中的每个位置 `i`。

### 情况 1：遇到 `(`

将它的下标压入栈：

```python
stack.append(i)
```

------

### 情况 2：遇到 `)`

先弹出栈顶：

```python
stack.pop()
```

因为当前 `)` 尝试和之前的一个 `(` 匹配。

接下来有两种情况。

#### 情况 A：栈不为空

说明当前 `)` 成功匹配。

当前有效括号区间的长度：

```python
i - stack[-1]
```

为什么？

因为：

```text
stack[-1]
```

表示当前有效区间之前最近的一个“边界”。

所以真正的有效区间是：

```text
stack[-1] + 1 ... i
```

长度就是：

```text
i - stack[-1]
```

------

#### 情况 B：栈为空

说明当前出现了一个无法匹配的：

```text
)
```

例如：

```text
")"
```

或者：

```text
"())"
```

这个 `)` 会成为新的边界。

因此：

```python
stack.append(i)
```

------

## Python 代码

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        # 栈中保存括号的下标
        #
        # -1 作为初始边界，
        # 方便计算从字符串开头开始的有效括号长度
        # 重要特性： 除了栈底元素之外，栈里的其他元素一定都是未匹配的左括号下标
        stack = [-1]

        max_len = 0

        for i, ch in enumerate(s):

            if ch == '(':
                # 左括号暂时无法匹配，
                # 将它的下标放入栈中
                stack.append(i)

            else:
                # 当前右括号尝试匹配最近的左括号
                stack.pop()

                if not stack:
                    # 没有左括号可以匹配当前 ')'
                    #
                    # 当前下标成为新的无效边界
                    stack.append(i)
                    # 结合下面的else, 可以发现只有stack为空的时候才会压入“）”，因此“）”肯定位于栈底

                else:
                    # 当前存在合法括号区间
                    #
                    # stack[-1] 是这个合法区间之前的边界
                    current_len = i - stack[-1]

                    max_len = max(max_len, current_len)
                    # 注意，这时并没有把"）"压入栈

        return max_len
```

------

## 示例分析

假设：

```text
s = ")()())"
```

索引：

```text
index:  0 1 2 3 4 5
char:   ) ( ) ( ) )
```

初始化：

```text
stack = [-1]
max_len = 0
```

------

### i = 0，字符 `)`

弹出：

```text
-1
```

栈为空。

说明当前 `)` 无法匹配。

把 `0` 作为新的边界：

```text
stack = [0]
```

------

### i = 1，字符 `(`

压栈：

```text
stack = [0, 1]
```

------

### i = 2，字符 `)`

弹出：

```text
1
```

得到：

```text
stack = [0]
```

当前有效区间：

```text
i - stack[-1]
=
2 - 0
=
2
```

对应：

```text
"()"
```

所以：

```text
max_len = 2
```

------

### i = 3，字符 `(`

```text
stack = [0, 3]
```

------

### i = 4，字符 `)`

弹出：

```text
3
```

得到：

```text
stack = [0]
```

当前有效长度：

```text
4 - 0
=
4
```

对应：

```text
"()()"
```

所以：

```text
max_len = 4
```

------

### i = 5，字符 `)`

弹出：

```text
0
```

栈为空。

说明这个 `)` 没办法匹配。

把 `5` 设为新边界：

```text
stack = [5]
```

最终：

```text
max_len = 4
```

------

## 复杂度分析

每个字符最多：

```text
入栈一次
出栈一次
```

所以：

```text
时间复杂度：O(n)
```

最坏情况下，例如：

```text
"((((((("
```

所有左括号都会进入栈：

```text
空间复杂度：O(n)
```

------

## Python 补充：`enumerate`

这里：

```python
for i, ch in enumerate(s):
```

`enumerate()` 可以同时得到：

```text
index
value
```

例如：

```python
s = "()"

for i, ch in enumerate(s):
    print(i, ch)
```

结果：

```text
0 (
1 )
```

因此特别适合这道题，因为我们既需要：

```text
括号字符
```

又需要：

```text
括号所在的下标
```

------

# 解法二：Dynamic Programming（动态规划）

## 思路

这道题也可以使用 DP。

定义：

```text
dp[i]
```

表示：

> 以 `s[i]` 结尾的最长有效括号子串长度。

这里一定要注意：

```text
dp[i]
```

不是：

> 前 `i` 个字符中最长的答案。

而是：

> 必须以位置 `i` 结尾的最长有效括号。

这是理解 DP 解法的关键。

------

## 为什么只有 `)` 才可能产生答案？

任何有效括号字符串一定以：

```text
)
```

结束。

例如：

```text
()
(())
()()
```

所以：

```text
如果 s[i] == '('
那么 dp[i] = 0
```

我们只需要处理：

```text
s[i] == ')'
```

的情况。

------

# 情况一：`...()`

如果：

```text
s[i] == ')'
s[i - 1] == '('
```

那么当前位置形成：

```text
()
```

例如：

```text
... ( )
    ↑ ↑
   i-1 i
```

这两个字符本身贡献：

```text
2
```

并且它们前面可能已经存在一段有效括号。

所以：

```text
dp[i]
=
dp[i - 2]
+
2
```

如果：

```text
i < 2
```

那么前面没有位置 `i - 2`，直接认为前面的有效长度为 `0`。

------

## 示例

```text
s = "()()"
```

对于最后一个 `)`：

```text
i = 3
```

因为：

```text
s[2] == '('
```

所以：

```text
dp[3]
=
dp[1] + 2
=
2 + 2
=
4
```

从而把：

```text
"()"
```

和前面的：

```text
"()"
```

连接起来。

------

# 情况二：`...))`

这一种情况更重要。

如果：

```text
s[i] == ')'
s[i - 1] == ')'
```

那么当前 `)` 不可能直接和 `i - 1` 匹配。

因为：

```text
i - 1
```

本身已经是某段有效括号的结尾。

因此，我们需要：

> 跳过 `i - 1` 结尾的整个有效括号区间，再看看它前面是否存在一个 `(`。

------

## 如何找到可能匹配的 `(`？

已知：

```text
dp[i - 1]
```

表示以 `i - 1` 结尾的有效括号长度。

因此这一段有效括号的起始位置是：

```text
i - dp[i - 1]
```

那么它前面一个位置就是：

```text
i - dp[i - 1] - 1
```

这就是当前：

```text
s[i] = ')'
```

可能匹配的左括号位置。

令：

```text
match_index
=
i - dp[i - 1] - 1
```

如果：

```text
match_index >= 0
```

并且：

```text
s[match_index] == '('
```

说明可以形成一个更长的有效括号。

------

## 此时的递推关系

首先：

```text
dp[i - 1]
```

是内部已经匹配好的部分。

再加上：

```text
(
)
```

这一对：

```text
+ 2
```

此外，在这个新匹配到的左括号之前，可能还有一段有效括号：

```text
dp[match_index - 1]
```

所以：

```text
dp[i]
=
dp[i - 1]
+
2
+
dp[match_index - 1]
```

当然，如果：

```text
match_index == 0
```

那么前面没有元素，最后这一项记为 `0`。

------

# 一个重要示例：`()(())`

考虑：

```text
s = "()(())"
```

索引：

```text
index: 0 1 2 3 4 5
char:  ( ) ( ( ) )
```

在：

```text
i = 5
```

当前位置：

```text
s[5] = ')'
```

前一个字符：

```text
s[4] = ')'
```

而：

```text
dp[4] = 2
```

对应：

```text
s[3:5] = "()"
```

于是：

```text
match_index
=
5 - 2 - 1
=
2
```

发现：

```text
s[2] == '('
```

所以位置：

```text
2 和 5
```

可以匹配。

于是：

```text
dp[5]
=
dp[4]
+
2
+
dp[1]
```

即：

```text
2 + 2 + 2
=
6
```

最终得到：

```text
"()(())"
```

长度：

```text
6
```

------

## Python 代码

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        n = len(s)

        # dp[i]：
        # 以 s[i] 结尾的最长有效括号长度
        dp = [0] * n

        max_len = 0

        # 有效括号至少需要两个字符，
        # 因此从 index 1 开始
        for i in range(1, n):

            # 只有 ')' 才可能成为有效括号的结尾
            if s[i] == ')':

                # Case 1:
                #
                # ...()
                if s[i - 1] == '(':
                    dp[i] = 2

                    # 如果 () 前面还有有效括号，
                    # 将它们连接起来
                    if i >= 2:
                        dp[i] += dp[i - 2]

                # Case 2:
                #
                # ...))
                else:
                    # 跳过以 i-1 结尾的有效括号，
                    # 找当前 ')' 可能匹配的 '('
                    match_index = i - dp[i - 1] - 1

                    if (
                        match_index >= 0
                        and s[match_index] == '('
                    ):
                        # 当前匹配：
                        #
                        # ( previous_valid )
                        dp[i] = dp[i - 1] + 2

                        # 当前新形成的有效括号前面，
                        # 可能还有另一段连续的有效括号
                        if match_index - 1 >= 0:
                            dp[i] += dp[match_index - 1]

                max_len = max(max_len, dp[i])

        return max_len
```

------

## DP 状态总结

定义：

```text
dp[i]
=
以 s[i] 结尾的最长有效括号长度
```

如果：

```text
s[i] == '('
```

那么：

```text
dp[i] = 0
```

如果：

```text
s[i] == ')'
```

分为两种情况。

### Case 1

```text
...()
```

即：

```text
s[i - 1] == '('
```

那么：

```text
dp[i]
=
dp[i - 2]
+
2
```

------

### Case 2

```text
...))
```

找到：

```text
j = i - dp[i - 1] - 1
```

如果：

```text
j >= 0
and
s[j] == '('
```

那么：

```text
dp[i]
=
dp[i - 1]
+
2
+
dp[j - 1]
```

其中越界部分按照 `0` 处理。

------

## 复杂度分析

我们只遍历字符串一次：

```text
时间复杂度：O(n)
```

使用长度为 `n` 的 DP 数组：

```text
空间复杂度：O(n)
```

------

# 解法三：双向扫描 / Two Pass

## 思路

这是一种非常巧妙的：

```text
O(n) 时间
O(1) 空间
```

解法。

我们只维护两个变量：

```text
left
right
```

分别表示当前扫描区间中：

```text
'(' 的数量
')' 的数量
```

如果：

```text
left == right
```

说明当前这一段括号数量平衡。

于是：

```text
当前有效长度 = 2 * right
```

------

## 为什么只统计数量就够了？

从左往右扫描时，我们维护：

```text
left >= right
```

只要：

```text
right > left
```

说明右括号太多。

例如：

```text
())
```

无论后面发生什么，前面的：

```text
)
```

过多的问题都无法被后面的字符修复。

所以需要：

```text
reset
```

即：

```python
left = 0
right = 0
```

------

# 为什么还需要从右往左扫描？

仅仅从左往右扫描是不够的。

例如：

```text
s = "(()"
```

从左向右：

```text
left  = 2
right = 1
```

最后：

```text
left > right
```

但是其中明明存在：

```text
"()"
```

长度为：

```text
2
```

问题在于：

> 多出来的是左括号。

从左往右扫描时：

```text
left > right
```

并不能立即判断前面的部分完全无效。

因此还必须反方向扫描。

------

## 从右向左扫描

从右向左时逻辑完全对称。

如果：

```text
left == right
```

则当前有效长度：

```text
2 * left
```

如果：

```text
left > right
```

说明左括号过多。

从右向左来看，这已经无法被更左侧的字符修复。

因此：

```text
reset
```

------

## Python 代码

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        max_len = 0

        left = 0
        right = 0

        # -------------------------
        # 第一遍：从左向右
        # -------------------------
        for ch in s:
            if ch == '(':
                left += 1
            else:
                right += 1

            # 左右括号数量相等，
            # 当前区间是一个有效括号区间
            if left == right:
                max_len = max(max_len, 2 * right)

            # 右括号过多，
            # 当前区间不可能继续形成合法括号
            elif right > left:
                left = 0
                right = 0

        # 为第二遍扫描重新初始化
        left = 0
        right = 0

        # -------------------------
        # 第二遍：从右向左
        # -------------------------
        for ch in reversed(s):
            if ch == '(':
                left += 1
            else:
                right += 1

            if left == right:
                max_len = max(max_len, 2 * left)

            # 从右向左扫描时，
            # 如果左括号更多，就无法继续匹配
            elif left > right:
                left = 0
                right = 0

        return max_len
```

------

## Python 补充：`reversed()`

这里：

```python
for ch in reversed(s):
```

表示逆序遍历字符串。

例如：

```python
s = "abc"
```

那么：

```python
list(reversed(s))
```

得到：

```text
['c', 'b', 'a']
```

它不会修改原始字符串。

------

## 复杂度分析

虽然扫描了两遍，但是：

```text
O(n) + O(n)
=
O(2n)
=
O(n)
```

所以：

```text
时间复杂度：O(n)
```

只使用：

```text
left
right
max_len
```

几个变量：

```text
空间复杂度：O(1)
```

------

# 三种方案对比

| 方法                | 时间复杂度 | 空间复杂度 | 特点                       |
| ------------------- | ---------- | ---------- | -------------------------- |
| Stack               | `O(n)`     | `O(n)`     | 最直观，最容易在面试中解释 |
| Dynamic Programming | `O(n)`     | `O(n)`     | 状态转移较复杂，但很经典   |
| 双向扫描            | `O(n)`     | `O(1)`     | 空间最优，但思路更巧妙     |

------

# 面试推荐

如果第一次遇到这道题，最推荐首先掌握：

```text
Stack
```

因为它最符合括号问题的直觉。

面试时可以这样逐步思考：

```text
括号匹配
    ↓
想到 Stack
    ↓
不仅要知道是否匹配，
还需要计算连续区间长度
    ↓
所以 Stack 保存 index，而不是 '('
    ↓
增加一个 boundary = -1
    ↓
使用 i - stack[-1] 计算长度
```

这条推导路线非常自然。

------

# Stack 解法中最重要的细节

## 为什么初始化 `stack = [-1]`？

这是 Stack 解法中最值得理解的一点。

考虑：

```text
s = "()()"
```

如果没有：

```text
-1
```

那么第一个完整的：

```text
()
```

匹配之后，栈就空了。

我们不知道应该用哪个下标来计算：

```text
长度
```

而：

```text
-1
```

相当于在字符串开头之前放置了一个：

```text
virtual boundary
```

所以：

```text
index: -1 0 1
           ( )
```

当处理到：

```text
i = 1
```

长度：

```text
1 - (-1)
=
2
```

当处理到：

```text
i = 3
```

长度：

```text
3 - (-1)
=
4
```

因此：

> 栈底保存的并不一定是左括号，而是当前合法区间之前的最近边界。

这是这个算法真正的核心。

------

# 一个容易误解的地方

Stack 中保存的元素有两种含义。

有时候它表示：

```text
尚未匹配的 '(' 的下标
```

有时候栈底元素表示：

```text
最近一个无法匹配的 ')' 的下标
```

例如：

```text
s = ")()()"
```

第一位：

```text
index = 0
char = ')'
```

无法匹配。

于是：

```text
stack = [0]
```

这里：

```text
0
```

不是一个等待匹配的左括号。

它代表：

```text
当前有效括号区间不能跨过的位置
```

因此后面的：

```text
()()
```

长度可以通过：

```text
4 - 0
=
4
```

直接计算。

------

# DP 解法最值得记住的技巧

DP 的最大难点不是：

```text
...()
```

而是：

```text
...))
```

遇到这种情况时，不要只观察：

```text
s[i - 1]
```

而应该：

> 跳过前一个位置已经形成的整个合法括号区间。

关键公式：

```text
match_index
=
i - dp[i - 1] - 1
```

可以把它理解为：

```text
当前 )
   ↑
   i

跳过前面的合法区间：
<------ dp[i-1] ------>

再向左走一步，
就是当前 ')' 希望匹配的 '('
```

所以对于 DP 方案，可以记住：

```text
看到 ...))
→ 跳过前面的有效括号
→ 寻找更前面的 '('
```

------

# 双向扫描为什么必须扫描两遍？

可以用两个反例直接记忆。

------

## 只从左向右的问题

```text
"(()"
```

左括号过多：

```text
left > right
```

但其中存在：

```text
"()"
```

所以从左往右无法处理：

```text
多余的左括号
```

------

## 从右往左补充

逆向扫描：

```text
"(()"
```

实际上看成：

```text
")(("
```

从右向左统计时，就能够发现：

```text
left == right
```

对应：

```text
"()"
```

长度：

```text
2
```

因此：

```text
从左向右：
解决右括号过多

从右向左：
解决左括号过多
```

两个方向组合以后，才能覆盖所有情况。

------

# 常见错误

## 1. Stack 只保存 `'('`

例如：

```python
stack.append('(')
```

这种方法适合：

```text
判断字符串是否合法
```

但不适合直接解决：

```text
最长有效括号长度
```

因为你无法知道有效区间的：

```text
起点位置
```

所以这道题应该保存：

```python
stack.append(i)
```

也就是：

```text
index
```

------

## 2. 忘记 Stack 的初始 `-1`

错误：

```python
stack = []
```

会导致处理：

```text
"()"
```

或者：

```text
"()()"
```

这种从索引 `0` 开始的有效括号时，长度计算非常麻烦。

推荐：

```python
stack = [-1]
```

把：

```text
-1
```

作为虚拟边界。

------

## 3. 遇到无法匹配的 `)` 后不更新边界

例如：

```text
s = ")()()"
```

最前面的：

```text
)
```

意味着后面的合法字符串：

```text
"()()"
```

不能跨过它。

所以当 Stack 弹空时：

```python
if not stack:
    stack.append(i)
```

一定要把当前位置设为新边界。

------

## 4. DP 中把 `dp[i]` 理解成全局最大值

这道题的 DP 定义是：

```text
dp[i]
=
以 i 结尾的最长有效括号长度
```

而不是：

```text
前 i 个位置中的最长有效括号长度
```

例如：

```text
s = "()("
```

可以有：

```text
dp = [0, 2, 0]
```

虽然整个字符串目前的最大答案是：

```text
2
```

但是：

```text
dp[2] = 0
```

因为：

```text
s[2] = '('
```

不存在以它结尾的有效括号。

所以还需要单独维护：

```python
max_len
```

------

## 5. 双向扫描只做一个方向

只从左往右无法处理：

```text
"(()"
```

只从右往左也会存在对应的对称问题。

因此必须：

```text
left → right
+
right → left
```

扫描两遍。

------

# 最推荐代码：Stack

如果面试官没有额外要求：

```text
O(1) space
```

那么 Stack 通常是最推荐首先写出的方案。

原因是：

```text
逻辑直观
代码较短
O(n) 时间
容易证明正确性
```

代码如下：

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        # 保存当前无法跨越的边界，
        # 以及尚未匹配的左括号下标
        stack = [-1]

        max_len = 0

        for i, ch in enumerate(s):

            if ch == '(':
                stack.append(i)

            else:
                # 尝试让当前 ')' 匹配栈顶的 '('
                stack.pop()

                if not stack:
                    # 当前 ')' 无法匹配，
                    # 它成为新的边界
                    stack.append(i)

                else:
                    # stack[-1] 是当前有效区间左侧的边界
                    max_len = max(
                        max_len,
                        i - stack[-1]
                    )

        return max_len
```

复杂度：

```text
时间复杂度：O(n)
空间复杂度：O(n)
```

------

# 如果要求 O(1) Space

可以使用双向扫描：

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        max_len = 0

        left = right = 0

        # 左 -> 右
        for ch in s:
            if ch == '(':
                left += 1
            else:
                right += 1

            if left == right:
                max_len = max(max_len, 2 * right)
            elif right > left:
                left = right = 0

        left = right = 0

        # 右 -> 左
        for ch in reversed(s):
            if ch == '(':
                left += 1
            else:
                right += 1

            if left == right:
                max_len = max(max_len, 2 * left)
            elif left > right:
                left = right = 0

        return max_len
```

复杂度：

```text
时间复杂度：O(n)
空间复杂度：O(1)
```

------

# 一句话记忆

这道题最值得记住三个思路：

```text
Stack：
保存 index，而不是括号字符；
-1 / 无法匹配的 ')' 作为边界；
长度 = i - stack[-1]
DP：
dp[i] = 以 i 结尾的最长合法括号；
遇到 ...)) 时，
跳过 dp[i-1] 再寻找匹配的 '('
Two Pass：
左 → 右解决右括号过多；
右 → 左解决左括号过多；
做到 O(n) 时间、O(1) 空间。
```

如果面试时只能优先掌握一种解法，建议优先记住：

```text
Stack + index + boundary
```

如果面试官继续要求优化空间，再给出：

```text
Two Pass
```
