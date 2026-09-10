# 电话号码的字母组合（Letter Combinations of a Phone Number）

## 题目描述

给定一个字符串 `digits`，其中只包含数字 `2` 到 `9`。

每个数字都对应手机九宫格键盘上的一组字母：

```text
2 -> abc
3 -> def
4 -> ghi
5 -> jkl
6 -> mno
7 -> pqrs
8 -> tuv
9 -> wxyz
```

每个数字都可以代表它映射到的任意一个字母。

请返回 `digits` 能表示的所有可能字母组合。结果可以按任意顺序返回。

### 约束

- `0 <= digits.length <= 4`
- `2 <= digits[i] <= 9`

---

# 一、回溯（Backtracking）

## 核心思路

这道题本质上是在做：

> 对每一个数字，从它对应的字母集合中选择一个字母，并枚举所有可能的选择结果。

例如：

```text
digits = "23"
```

对应：

```text
2 -> abc
3 -> def
```

所有组合为：

```text
ad ae af
bd be bf
cd ce cf
```

整个过程可以看成一棵决策树：

```text
              ""
        /      |      \
       a       b       c
     / | \   / | \   / | \
    ad ae af bd be bf cd ce cf
```

- 每一层代表一个数字；
- 每一个分支代表当前数字可以选择的一个字母；
- 当处理完所有数字时，就得到一个完整组合。

因此这是一个非常典型的回溯问题。

---

## 回溯中的三个关键元素

### 1. 当前状态

使用：

```python
curStr
```

表示当前已经构造出的字符串。

例如：

```text
"a"
"bd"
```

### 2. 当前处理的位置

使用：

```python
i
```

表示当前正在处理：

```python
digits[i]
```

### 3. 终止条件

当：

```python
i == len(digits)
```

说明每个数字都已经选择了一个字母。

此时：

```python
res.append(curStr)
```

把当前组合加入结果即可。

---

## 算法步骤

1. 如果 `digits` 为空，直接返回 `[]`。
2. 建立数字到字母的映射。
3. 定义递归函数 `backtrack(i, curStr)`。
4. 如果 `i == len(digits)`：
   - 当前字符串已经完整；
   - 将它加入结果并返回。
5. 否则遍历 `digits[i]` 对应的所有字母。
6. 对每个字母 `c`：
   - 把它加入当前字符串；
   - 递归处理下一个数字。
7. 从 `backtrack(0, "")` 开始搜索。
8. 返回所有结果。

---

## Python 实现

```python
from typing import List


class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []

        digitToChar = {
            "2": "abc",
            "3": "def",
            "4": "ghi",
            "5": "jkl",
            "6": "mno",
            "7": "pqrs",
            "8": "tuv",
            "9": "wxyz",
        }

        res = []

        def backtrack(i: int, curStr: str) -> None:
            # 已经为每一个数字选择了一个字母
            if i == len(digits):
                res.append(curStr)
                return

            # 枚举当前数字对应的所有字母
            for c in digitToChar[digits[i]]:
                # 选择当前字符，然后递归处理下一个数字
                backtrack(i + 1, curStr + c)

        backtrack(0, "")

        return res
```

---

## 为什么这里没有显式“撤销选择”？

很多回溯代码会写成：

```python
path.append(x)
backtrack(...)
path.pop()
```

也就是：

```text
做选择 -> 递归 -> 撤销选择
```

但这里使用：

```python
curStr + c
```

每次都会创建一个新的字符串。

例如：

```python
curStr = "ab"
newStr = curStr + "c"
```

此时：

```text
curStr = "ab"
newStr = "abc"
```

原来的 `curStr` 并没有被修改。

因此这里不需要显式执行 `pop()` 来撤销选择。

---

## Python 特性说明：字符串不可变

Python 中的字符串是不可变对象（immutable）。

因此：

```python
curStr + c
```

不会修改原字符串，而是返回一个新的字符串。

这让本题的回溯代码可以写得非常简洁。

---

## 复杂度分析

设：

```text
n = len(digits)
```

数字 `7` 和 `9` 各自最多对应 4 个字符，因此最终组合数量最多为：

```text
4^n
```

每个结果字符串长度为 `n`。

因此：

- **时间复杂度：`O(n * 4^n)`**
- **输出空间复杂度：`O(n * 4^n)`**
- **额外递归空间复杂度：`O(n)`**

递归深度最多就是数字数量 `n`。

> 有时会把搜索树节点数量粗略写成 `O(4^n)`，但如果把最终字符串的构造和复制成本也计算进去，更完整的写法是 `O(n * 4^n)`。

---

# 二、迭代法（Iteration）

## 核心思路

除了递归以外，也可以逐层构造答案。

先从：

```python
res = [""]
```

开始。

对于每一个数字：

1. 取出当前已经构造好的所有字符串；
2. 把当前数字对应的每一个字符追加到这些字符串之后；
3. 得到下一层所有组合。

这种方法可以理解为对决策树进行“层序扩展”，和 BFS 的思想类似。

---

## 示例

假设：

```text
digits = "23"
```

初始：

```text
res = [""]
```

处理数字 `2`：

```text
2 -> abc
```

得到：

```text
["a", "b", "c"]
```

再处理数字 `3`：

```text
3 -> def
```

得到：

```text
[
    "ad", "ae", "af",
    "bd", "be", "bf",
    "cd", "ce", "cf"
]
```

---

## 为什么初始化为 `[""]`？

这里的空字符串表示：

```text
“目前还没有处理任何数字”
```

第一次处理数字时，就可以直接执行：

```python
"" + "a"
"" + "b"
"" + "c"
```

从而构造第一层结果。

需要注意：

- **算法内部初始状态**可以是 `[""]`；
- 但如果输入本身为空，题目答案仍然是 `[]`。

---

## 算法步骤

1. 如果 `digits` 为空，返回 `[]`。
2. 初始化 `res = [""]`。
3. 遍历每一个数字 `digit`。
4. 创建临时列表 `tmp`。
5. 对 `res` 中每一个已有组合：
   - 遍历当前数字对应的每一个字母；
   - 生成新字符串并加入 `tmp`。
6. 当前层结束后，让 `res = tmp`。
7. 所有数字处理完成后返回 `res`。

---

## Python 实现

```python
from typing import List


class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []

        digitToChar = {
            "2": "abc",
            "3": "def",
            "4": "ghi",
            "5": "jkl",
            "6": "mno",
            "7": "pqrs",
            "8": "tuv",
            "9": "wxyz",
        }

        # 表示“当前还没有选择任何字符”
        res = [""]

        for digit in digits:
            tmp = []

            # 对当前所有已有组合继续扩展
            for curStr in res:
                for c in digitToChar[digit]:
                    tmp.append(curStr + c)

            # 当前数字处理完成
            res = tmp

        return res
```

---

## 复杂度分析

同样设：

```text
n = len(digits)
```

最终最多有：

```text
4^n
```

个组合，每个组合长度为 `n`。

因此：

- **时间复杂度：`O(n * 4^n)`**
- **输出空间复杂度：`O(n * 4^n)`**

### 迭代法的额外空间

原始解析把迭代法额外空间简单写成 `O(n)` 并不准确。

迭代过程中：

```python
tmp
```

需要保存当前层新生成的大量字符串。

在最后一层，其规模可能已经达到最终输出规模。

所以如果把临时结果也算作辅助空间，最坏情况下可以达到：

```text
O(n * 4^n)
```

---

# 三、更经典的原地回溯模板

本题长度最多只有 4，因此使用：

```python
curStr + c
```

完全足够。

但如果你希望掌握更通用的回溯模板，可以使用一个可变列表 `path`：

```python
path.append(c)
backtrack(...)
path.pop()
```

代码如下：

```python
from typing import List


class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []

        digitToChar = {
            "2": "abc",
            "3": "def",
            "4": "ghi",
            "5": "jkl",
            "6": "mno",
            "7": "pqrs",
            "8": "tuv",
            "9": "wxyz",
        }

        res = []
        path = []

        def backtrack(i: int) -> None:
            if i == len(digits):
                # path 是字符列表，需要拼接成字符串
                res.append("".join(path))
                return

            for c in digitToChar[digits[i]]:
                # 做选择
                path.append(c)

                # 递归处理下一个数字
                backtrack(i + 1)

                # 撤销选择
                path.pop()

        backtrack(0)

        return res
```

这种写法更能体现标准回溯框架：

```text
for 每一个选择:
    做选择
    递归
    撤销选择
```

---

## Python 函数说明：`"".join(path)`

假设：

```python
path = ["a", "d", "g"]
```

执行：

```python
"".join(path)
```

得到：

```text
"adg"
```

`join` 用于把多个字符串按指定分隔符连接起来。

例如：

```python
"-".join(["a", "b", "c"])
```

得到：

```text
"a-b-c"
```

---

# 四、两种回溯写法的区别

### 写法 1：传递新字符串

```python
backtrack(i + 1, curStr + c)
```

优点：

- 代码简单；
- 不需要手动撤销；
- 很适合本题。

### 写法 2：维护 `path`

```python
path.append(c)
backtrack(i + 1)
path.pop()
```

优点：

- 更符合通用回溯模板；
- 对排列、组合、子集等问题更通用；
- 更容易迁移到其他题目。

如果你正在系统准备回溯类面试题，建议重点理解第二种形式。

---

# 常见错误

## 1. 空输入返回错误

错误：

```python
if not digits:
    return [""]
```

正确：

```python
if not digits:
    return []
```

题目要求没有数字时返回空结果。

---

## 2. 数字 `7` 的映射写错

正确的是：

```text
7 -> pqrs
```

原始代码写成了：

```text
qprs
```

虽然字符集合相同，而且题目允许任意顺序返回答案，但标准电话键盘映射应写成：

```python
"7": "pqrs"
```

---

## 3. 忘记 `7` 和 `9` 各有四个字符

大多数数字对应 3 个字符：

```text
2 -> abc
3 -> def
4 -> ghi
5 -> jkl
6 -> mno
8 -> tuv
```

但：

```text
7 -> pqrs
9 -> wxyz
```

分别有 4 个字符。

这也是为什么最坏情况下组合数量使用：

```text
4^n
```

来估计。

---

## 4. 递归索引越界

应该先判断：

```python
if i == len(digits):
    ...
    return
```

然后再访问：

```python
digits[i]
```

否则当 `i == len(digits)` 时，访问 `digits[i]` 会触发 `IndexError`。

---

## 5. 使用数组映射时索引转换错误

除了字典，也可以使用数组：

```python
digitToChar = [
    "",      # 0
    "",      # 1
    "abc",   # 2
    "def",   # 3
    "ghi",   # 4
    "jkl",   # 5
    "mno",   # 6
    "pqrs",  # 7
    "tuv",   # 8
    "wxyz",  # 9
]
```

如果当前：

```python
digit = "7"
```

Python 中可以使用：

```python
digitToChar[int(digit)]
```

因为：

```python
int("7") == 7
```

### Python 与 Java/C++ 的区别

Java/C++ 中经常看到：

```text
digit - '0'
```

但 Python 中不能写：

```python
digit - "0"
```

因为字符串不支持减法。

Python 一般直接使用：

```python
int(digit)
```

---

## 6. 把组合问题误写成排列问题

本题中数字的位置顺序不能改变。

对于：

```text
digits = "23"
```

必须：

```text
先从 abc 中选一个字符
再从 def 中选一个字符
```

而不是任意交换数字对应的层级。

所以这是一棵“每层候选集合固定”的决策树，不是对全部字符做排列。

---

# 解法对比

| 方法 | 时间复杂度 | 额外空间 | 推荐程度 |
|---|---:|---:|---|
| 回溯 | `O(n * 4^n)` | `O(n)` 递归栈，不含输出 | **最推荐** |
| 迭代 | `O(n * 4^n)` | 中间结果最坏达到输出规模 | 推荐 |

由于题目要求返回**所有组合**，最终答案本身就可能有 `4^n` 个，因此指数级输出无法避免。

---

# 面试中推荐记忆模板

```python
class Solution:
    def letterCombinations(self, digits: str) -> list[str]:
        if not digits:
            return []

        mapping = {
            "2": "abc",
            "3": "def",
            "4": "ghi",
            "5": "jkl",
            "6": "mno",
            "7": "pqrs",
            "8": "tuv",
            "9": "wxyz",
        }

        res = []

        def backtrack(i: int, cur: str) -> None:
            if i == len(digits):
                res.append(cur)
                return

            for c in mapping[digits[i]]:
                backtrack(i + 1, cur + c)

        backtrack(0, "")

        return res
```

核心逻辑可以记成：

```text
当前处理 digits[i]
→ 枚举它对应的每一个字母
→ 选择一个字母
→ 递归处理下一个数字
→ 处理完所有数字后加入答案
```

---

# 总结

这道题真正考察的是识别下面这种问题结构：

```text
有多个位置需要依次做选择
+
每个位置有若干候选项
+
需要列出所有完整方案
```

这通常就是：

```text
回溯 / DFS
```

本题决策树中：

```text
层数 = digits 的长度
```

每一层的分支数量为：

```text
3 或 4
```

最大组合数量：

```text
4^n
```

最值得掌握的思想是：

```text
1. 每一层代表一个数字
2. 每条分支代表选择一个字母
3. 走到第 n 层得到一个完整答案
4. 回溯负责遍历整棵决策树
```

掌握这个模型之后，`Subsets`、`Permutations`、`Combination Sum`、`Generate Parentheses` 等大量回溯题都会更容易理解。
