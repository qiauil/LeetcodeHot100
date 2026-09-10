# 单词拆分（Word Break）

给定字符串 `s` 和字符串字典 `wordDict`。如果可以使用字典中的单词，将 `s` 拆分成一个由若干单词组成的序列，则返回 `True`；否则返回 `False`。

字典中的单词可以重复使用任意次数，并且可以假设 `wordDict` 中的单词互不相同。

例如：

```
s = "leetcode"
wordDict = ["leet", "code"]

输出：True
解释："leetcode" 可以拆分为 "leet" + "code"
```

> 以下代码中的 `List` 来自 `typing` 模块：
>
> ```
> from typing import List
> ```

------

## 一、递归：逐个尝试字典中的单词

### 核心思路

定义：

```
dfs(i) = 字符串后缀 s[i:] 能否由字典中的单词组成
```

站在位置 `i`，依次尝试字典中的每个单词：

- 如果单词 `word` 与 `s` 从位置 `i` 开始的内容匹配；
- 就递归判断剩余部分 `s[i + len(word):]`；
- 只要有一种选择最终到达字符串末尾，就说明拆分成功。

递归树中的每条路径，都对应一种可能的单词拆分方式。

### 算法步骤

1. 定义递归函数 `dfs(i)`。
2. 如果 `i == len(s)`，说明整个字符串已经匹配完毕，返回 `True`。
3. 遍历 `wordDict` 中的每个单词：
   - 检查该单词能否放在位置 `i`；
   - 如果能够匹配，递归处理匹配后的剩余部分。
4. 如果任何一种选择成功，返回 `True`。
5. 如果所有选择均失败，返回 `False`。

### 代码

```
from typing import List


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        n = len(s)

        def dfs(i: int) -> bool:
            # 已经成功匹配完整个字符串
            if i == n:
                return True

            # 尝试将每个字典单词放在当前位置
            for word in wordDict:
                next_i = i + len(word)

                # 单词不能超过字符串边界，并且内容必须匹配
                if next_i <= n and s[i:next_i] == word:
                    if dfs(next_i):
                        return True

            # 所有单词都无法形成有效拆分
            return False

        return dfs(0)
```

### 复杂度分析

设：

- `n` 为字符串 `s` 的长度；
- `m` 为字典中的单词数；
- `t` 为字典中最长单词的长度。

在最坏情况下，递归会枚举大量拆分方案，因此：

- 时间复杂度：指数级，宽松上界可写为 `O(t · m^n)`；
- 空间复杂度：`O(n)`，主要来自递归调用栈。

这里的时间复杂度上界比较宽松，因为真正的递归深度还取决于字典中最短单词的长度。

### 存在的问题

不同的拆分路径可能重复到达同一个位置。例如，`dfs(5)` 可能被计算很多次。因此，普通递归通常会超时，它更适合用于理解问题结构。

------

## 二、递归：使用哈希集合枚举切分位置

### 核心思路

上一种方法在每个位置尝试所有字典单词。另一种方式是：

- 固定起点 `i`；
- 枚举所有可能的终点 `j`；
- 判断子串 `s[i:j + 1]` 是否在字典中。

首先将 `wordDict` 转换成集合 `word_set`，这样成员查询的平均时间复杂度为 `O(1)`。

### 代码

```
from typing import List


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        word_set = set(wordDict)
        n = len(s)

        def dfs(i: int) -> bool:
            # 空后缀可以被成功拆分
            if i == n:
                return True

            # 枚举当前单词的结束位置
            for j in range(i, n):
                current_word = s[i:j + 1]

                # 当前子串是单词时，继续处理剩余部分
                if current_word in word_set and dfs(j + 1):
                    return True

            return False

        return dfs(0)
```

### 复杂度分析

不使用记忆化时，算法仍然需要枚举指数级的拆分组合：

- 时间复杂度：通常记作指数级，可粗略表示为 `O(n · 2^n)`；
- 空间复杂度：
  - 递归栈为 `O(n)`；
  - 哈希集合保存字典需要 `O(m · t)`。

需要注意：Python 的字符串切片会创建新字符串，切片和计算其哈希值并不严格是 `O(1)`。因此，如果把字符串操作成本也计算进去，实际最坏复杂度可能更高。

------

## 三、自顶向下动态规划：递归加记忆化

### 核心思路

在递归过程中，同一个状态 `dfs(i)` 可能被重复计算。

但是：

```
s[i:] 能否被拆分
```

只取决于 `i`，其结果不会发生变化。因此，可以用 `memo` 保存已经计算过的状态：

```
memo[i] = s[i:] 是否能够被成功拆分
```

这样，每个起始位置最多只会被完整计算一次。

### 算法步骤

1. 使用字典 `memo` 保存每个位置的计算结果。
2. 设置基础状态 `memo[len(s)] = True`。
3. 计算 `dfs(i)` 时：
   - 如果 `i` 已经在 `memo` 中，直接返回结果；
   - 否则依次尝试所有字典单词。
4. 找到有效拆分时记录 `memo[i] = True`。
5. 所有单词均失败时记录 `memo[i] = False`。

### 代码

```
from typing import List


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        n = len(s)

        # 空后缀一定可以被拆分
        memo = {n: True}

        def dfs(i: int) -> bool:
            # 直接复用已经计算过的结果
            if i in memo:
                return memo[i]

            for word in wordDict:
                next_i = i + len(word)

                if next_i <= n and s[i:next_i] == word:
                    if dfs(next_i):
                        memo[i] = True
                        return True

            # 记录失败状态，避免以后重复搜索
            memo[i] = False
            return False

        return dfs(0)
```

### 复杂度分析

- 状态数量：`n + 1`；
- 每个状态最多尝试 `m` 个单词；
- 每次字符串比较最多需要 `O(t)`。

因此：

- 时间复杂度：`O(n · m · t)`；
- 空间复杂度：`O(n)`，包括记忆化结果和递归栈。

这是代码面试中比较自然且容易讲清楚的解法。

------

## 四、自顶向下动态规划：哈希集合与最大长度剪枝

### 核心思路

使用哈希集合枚举子串时，没有必要让子串长度超过字典中最长单词的长度。

假设最长单词长度为 `t`，从位置 `i` 开始时，只需要检查：

```
s[i:i+1], s[i:i+2], ..., s[i:i+t]
```

同时使用记忆化，使每个位置只被求解一次。

### 代码

```
from typing import List


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        word_set = set(wordDict)
        max_word_len = max(map(len, wordDict), default=0)
        n = len(s)

        memo = {}

        def dfs(i: int) -> bool:
            if i == n:
                return True

            if i in memo:
                return memo[i]

            # 结束位置不能超过字符串末尾，
            # 当前子串的长度也不能超过最长单词长度
            end_limit = min(n, i + max_word_len)

            for end in range(i + 1, end_limit + 1):
                current_word = s[i:end]

                if current_word in word_set and dfs(end):
                    memo[i] = True
                    return True

            memo[i] = False
            return False

        return dfs(0)
```

### 复杂度分析

如果暂时将集合查询看成平均 `O(1)`，每个位置最多尝试 `t` 个子串。

不过，Python 创建和计算子串 `s[i:end]` 的哈希值需要与子串长度相关的时间，因此：

- 时间复杂度：`O(n · t² + m · t)`；
- 空间复杂度：`O(n + m · t)`。

其中：

- `O(n)` 用于记忆化和递归栈；
- `O(m · t)` 用于存储字典集合。

与逐个尝试所有字典单词相比，这种方式在最长单词较短时通常很有效。

------

## 五、自底向上动态规划

### 状态定义

定义：

```
dp[i] = 后缀 s[i:] 是否能够被字典单词拆分
```

基础状态为：

```
dp[n] = True
```

因为空字符串可以看作已经完成拆分。

如果单词 `word` 能够匹配 `s` 从位置 `i` 开始的内容，并且：

```
dp[i + len(word)] == True
```

那么就可以得到：

```
dp[i] = True
```

由于 `dp[i]` 依赖右侧位置的状态，所以需要从右向左计算。

### 代码

```
from typing import List


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        n = len(s)

        # dp[i] 表示 s[i:] 是否可以被拆分
        dp = [False] * (n + 1)

        # 空后缀可以被成功拆分
        dp[n] = True

        # 从右向左计算每个状态
        for i in range(n - 1, -1, -1):
            for word in wordDict:
                next_i = i + len(word)

                # 当前单词能够匹配，并且剩余后缀可以拆分
                if (
                    next_i <= n
                    and s[i:next_i] == word
                    and dp[next_i]
                ):
                    dp[i] = True
                    break

        return dp[0]
```

### 复杂度分析

- 时间复杂度：`O(n · m · t)`；
- 空间复杂度：`O(n)`。

### 与记忆化递归的关系

自顶向下和自底向上实际上使用了相同的状态：

```
dfs(i)  <=>  dp[i]
```

区别在于：

- 自顶向下只计算递归实际访问到的状态；
- 自底向上按照固定顺序计算所有状态；
- 自底向上没有递归深度限制，通常更适合作为面试中的最终解法。

------

## 六、自底向上动态规划：哈希集合

还可以定义前缀状态：

```
dp[i] = 前 i 个字符 s[:i] 是否可以被成功拆分
```

这是另一种非常常见、通常也更直观的写法。

基础状态：

```
dp[0] = True
```

如果存在某个位置 `j`，满足：

```
dp[j] == True
并且
s[j:i] 在字典中
```

则：

```
dp[i] = True
```

### 代码

```
from typing import List


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        word_set = set(wordDict)
        max_word_len = max(map(len, wordDict), default=0)
        n = len(s)

        # dp[i] 表示前 i 个字符 s[:i] 是否可以被拆分
        dp = [False] * (n + 1)
        dp[0] = True

        for end in range(1, n + 1):
            # 单词长度不需要超过字典中的最长单词
            start_limit = max(0, end - max_word_len)

            for start in range(end - 1, start_limit - 1, -1):
                if dp[start] and s[start:end] in word_set:
                    dp[end] = True
                    break

        return dp[n]
```

### 复杂度分析

考虑 Python 字符串切片的成本：

- 时间复杂度：`O(n · t² + m · t)`；
- 空间复杂度：`O(n + m · t)`。

这是一种很适合作为面试主答案的实现：状态清晰，不使用递归，并且利用最长单词长度进行了剪枝。

------

## 七、动态规划与 Trie（前缀树）

### Trie 是什么

Trie，也称为前缀树，是一种专门用于存储字符串集合的树形结构。

Trie 中：

- 每条边代表一个字符；
- 从根节点到某个节点的路径代表一个字符串前缀；
- `is_word = True` 表示从根节点到当前节点构成了一个完整单词。

例如，插入 `"cat"` 和 `"car"` 后，它们可以共享前缀 `"ca"`：

```
root
 └── c
      └── a
           ├── t  (完整单词)
           └── r  (完整单词)
```

### 核心思路

从每个位置 `i` 开始，沿着 Trie 和字符串同时向右移动：

- 如果当前字符不在 Trie 的子节点中，可以立即停止；
- 如果到达一个完整单词，并且该单词后面的状态为 `True`，则当前位置也可以拆分。

相比于反复调用 `trie.search(s, i, j)`，直接从位置 `i` 向右遍历 Trie 更高效，因为它不会为每个结束位置重新从 Trie 根节点开始搜索。

### 优化后的代码

```
from typing import Dict, List


class TrieNode:
    def __init__(self):
        # children[字符] = 对应的下一个 Trie 节点
        self.children: Dict[str, "TrieNode"] = {}

        # 当前节点是否代表一个完整单词的结尾
        self.is_word = False


class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        """将一个单词插入 Trie。"""
        node = self.root

        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()

            node = node.children[char]

        # 标记完整单词的结尾
        node.is_word = True


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        trie = Trie()

        for word in wordDict:
            trie.insert(word)

        n = len(s)

        # dp[i] 表示 s[i:] 是否可以被拆分
        dp = [False] * (n + 1)
        dp[n] = True

        # 从右向左计算
        for i in range(n - 1, -1, -1):
            node = trie.root

            # 从 s[i] 开始沿着 Trie 向右匹配
            for j in range(i, n):
                char = s[j]

                # 当前前缀不属于任何字典单词，可以立即停止
                if char not in node.children:
                    break

                node = node.children[char]

                # s[i:j+1] 是完整单词，并且剩余部分可以拆分
                if node.is_word and dp[j + 1]:
                    dp[i] = True
                    break

        return dp[0]
```

### 复杂度分析

设所有字典单词的字符总数为：

```
L = sum(len(word) for word in wordDict)
```

每个位置最多沿 Trie 匹配 `t` 个字符：

- 构建 Trie：`O(L)`；
- 动态规划：`O(n · t)`；
- 总时间复杂度：`O(L + n · t)`；
- 空间复杂度：`O(L + n)`。

Trie 解法的优势是避免了大量字符串切片和重复前缀比较。不过，它的代码更长，并且需要额外的数据结构。在普通面试中，通常先给出记忆化递归或一维动态规划即可；如果面试官继续要求优化字符串匹配，再讨论 Trie。

------

# 常见错误

## 1. DP 数组少创建了一个元素

`dp[n]` 表示处理完全部字符后的状态，因此数组长度必须为 `n + 1`。

```
# 错误：数组只有下标 0 到 n - 1
dp = [False] * len(s)
dp[len(s)] = True  # IndexError

# 正确
dp = [False] * (len(s) + 1)
dp[len(s)] = True
```

------

## 2. 忘记设置空字符串的基础状态

无论使用前缀还是后缀定义，都需要为“没有剩余字符”设置成功状态：

```
# 后缀定义：s[n:] 是空字符串
dp[n] = True
```

或者：

```
# 前缀定义：前 0 个字符可以被拆分
dp[0] = True
```

如果遗漏这一状态，即使最后一个单词匹配成功，也无法完成状态转移。

------

## 3. 没有检查单词是否超出字符串范围

```
for word in wordDict:
    next_i = i + len(word)

    if next_i <= len(s) and s[i:next_i] == word:
        ...
```

严格来说，Python 的字符串切片即使超过右边界也不会抛出异常，例如：

```
"abc"[1:100]  # 结果为 "bc"
```

因此，原问题中所谓“导致越界错误”并不适用于 Python。不过，显式检查边界可以：

- 避免不必要的切片；
- 更清晰地表达“单词必须完整放入剩余字符串”；
- 方便迁移到 C++、Java 等可能需要严格处理边界的语言。

------

## 4. 使用列表进行大量成员查询

```
# wordDict 是列表，查询一次需要 O(m)
if s[i:j + 1] in wordDict:
    ...
```

更合适的方式是使用集合：

```
word_set = set(wordDict)

# 平均 O(1) 的哈希表查询
if s[i:j + 1] in word_set:
    ...
```

但需要注意，构造子串和计算字符串哈希仍然需要时间。

------

## 5. 忘记缓存失败状态

下面这种写法只缓存成功状态，会让失败的子问题被反复计算：

```
if dfs(next_i):
    memo[i] = True
    return True

return False  # 没有缓存失败结果
```

正确做法是同时保存 `False`：

```
if dfs(next_i):
    memo[i] = True
    return True

memo[i] = False
return False
```

在这类决策问题中，失败状态通常比成功状态更多，因此缓存失败结果非常重要。

------

## 6. 混淆 DP 状态的含义

常见的两种定义都正确，但状态转移方向不同：

| 状态定义                            | 基础状态       | 遍历方向 | 最终答案 |
| ----------------------------------- | -------------- | -------- | -------- |
| `dp[i]` 表示后缀 `s[i:]` 是否可拆分 | `dp[n] = True` | 从右向左 | `dp[0]`  |
| `dp[i]` 表示前缀 `s[:i]` 是否可拆分 | `dp[0] = True` | 从左向右 | `dp[n]`  |

实现前应该先明确状态定义，否则很容易出现下标和遍历方向错误。

------

# 解法总结

这道题的本质是：

```
在字符串中寻找一条由合法单词边界组成、从位置 0 到位置 n 的路径。
```

也可以将每个字符串下标看作图中的一个节点：

- 如果 `s[i:j]` 是字典单词，就存在一条从 `i` 到 `j` 的有向边；
- 问题转化为：能否从节点 `0` 到达节点 `n`。

几种解法的演进关系如下：

```
暴力递归
    ↓ 缓存重复状态
记忆化递归
    ↓ 消除递归调用
自底向上动态规划
    ↓ 使用最大单词长度剪枝
DP + 哈希集合
    ↓ 优化前缀匹配
DP + Trie
```

面试时建议优先掌握：

1. 记忆化递归：最容易从暴力搜索自然推导出来；
2. 一维自底向上 DP：代码稳定，没有递归深度问题；
3. Trie：作为进一步优化字符串匹配的扩展方案。