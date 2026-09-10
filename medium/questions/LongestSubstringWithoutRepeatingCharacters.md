# 无重复字符的最长子串（Longest Substring Without Repeating Characters）

给定一个字符串 `s`，请找出其中**不包含重复字符的最长子串长度**。

这里的**子串（substring）\**指的是字符串中一段\**连续的字符序列**。

------

## 示例 1

```text
Input: s = "zxyzxyz"

Output: 3
```

最长的不重复子串可以是：

```text
"zxy"
"xyz"
"yzx"
```

长度都是：

```text
3
```

------

## 示例 2

```text
Input: s = "xxxx"

Output: 1
```

因为任何长度大于 `1` 的子串都会包含重复的 `x`。

------

## 约束

- `0 <= s.length <= 50,000`
- `s` 可能包含可打印 ASCII 字符

------

# 1. 暴力解法

## 思路

最直接的方法是：

> 枚举每一个可能的起点，然后不断向右扩展，直到遇到重复字符。

对于每个起点 `i`：

1. 创建一个集合 `char_set`
2. 从 `i` 开始向右遍历
3. 如果当前字符没有出现过：
   - 加入集合
4. 如果当前字符已经出现过：
   - 当前子串无法继续扩展
   - 结束本轮
5. 更新最长长度

例如：

```text
s = "abcabc"
```

从索引 `0` 开始：

```text
a
ab
abc
```

继续遇到：

```text
a
```

因为 `a` 已经出现，所以停止。

然后再从索引 `1` 开始重新计算：

```text
b
bc
bca
```

这会产生大量重复工作。

------

## 算法步骤

1. 初始化：

```python
res = 0
```

1. 枚举每个起点 `i`
2. 为当前起点创建空集合：

```python
char_set = set()
```

1. 从 `i` 开始向右扩展：

```python
for j in range(i, len(s)):
```

1. 如果：

```python
s[j] in char_set
```

说明遇到重复字符，停止扩展。

1. 否则将字符加入集合。
2. 用集合长度更新答案。

------

## 代码

原题代码中 `lass Solution:` 少了一个 `c`，正确写法如下：

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # 记录最长的不重复子串长度
        res = 0

        # 枚举每一个可能的起点
        for i in range(len(s)):
            # 保存当前子串中已经出现的字符
            char_set = set()

            # 从 i 开始向右扩展
            for j in range(i, len(s)):
                # 如果字符重复，当前子串不能继续扩展
                if s[j] in char_set:
                    break

                # 当前字符第一次出现
                char_set.add(s[j])

            # 更新最长长度
            res = max(res, len(char_set))

        return res
```

------

## 复杂度分析

设：

```text
n = 字符串长度
m = 字符集大小 / 最多可能出现的不同字符数量
```

### 时间复杂度

最坏情况下：

```text
O(n²)
```

例如：

```text
s = "abcdef..."
```

没有重复字符时：

```text
从 0 开始检查 n 个字符
从 1 开始检查 n-1 个字符
从 2 开始检查 n-2 个字符
...
```

总次数为：

```text
n + (n - 1) + ... + 1
```

因此：

```text
O(n²)
```

原答案写成 `O(n * m)` 也可以作为更细致的描述，因为单次向右扩展最长不会超过不同字符数量 `m`。

不过在一般算法分析中，通常直接写：

```text
O(n²)
```

更加直观。

### 空间复杂度

```text
O(m)
```

因为集合中最多保存 `m` 个不同字符。

如果只考虑 ASCII 字符集，由于字符种类上限是常数，也可以把空间看作 `O(1)`；但一般面试中通常仍然写成：

```text
O(m)
```

更通用。

------

# 2. 滑动窗口 + Set

## 核心思路

暴力解法最大的问题是：

> 一旦遇到重复字符，就把整个窗口丢掉，然后从下一个位置重新开始。

实际上没有必要。

假设当前窗口是：

```text
[a b c]
```

接下来又遇到：

```text
a
```

变成：

```text
a b c a
```

这时不需要从头重新计算，只需要不断移动左边界，直到旧的 `a` 被移出窗口。

于是：

```text
a b c a
^
l
```

先移除左边的 `a`：

```text
b c a
```

窗口重新变成无重复字符。

这就是典型的**滑动窗口（Sliding Window）**。

------

# 什么是滑动窗口？

滑动窗口通常使用两个指针：

```text
l = left
r = right
```

表示一个连续区间：

```text
s[l : r + 1]
```

例如：

```text
s = "abcdef"

      l     r
      ↓     ↓
      c d e
```

对应：

```python
s[l:r + 1]
```

滑动窗口的基本思想是：

- `r` 向右移动：扩大窗口
- 如果窗口违反条件：
  - 移动 `l`
  - 缩小窗口
- 始终维护窗口满足某种性质

本题需要维护的性质就是：

> 窗口内所有字符都不重复。

------

## 窗口中的数据结构

我们用：

```python
char_set = set()
```

保存当前窗口中的字符。

因此可以快速判断：

```python
s[r] in char_set
```

平均时间复杂度为：

```text
O(1)
```

------

## 算法步骤

初始化：

```python
char_set = set()
l = 0
res = 0
```

然后让右指针 `r` 从左向右扫描字符串。

对于每个字符 `s[r]`：

### 如果没有重复

直接加入集合：

```python
char_set.add(s[r])
```

然后更新：

```python
res = max(res, r - l + 1)
```

### 如果出现重复

不断移动左指针：

```python
while s[r] in char_set:
```

每次：

```python
char_set.remove(s[l])
l += 1
```

直到当前字符不再重复。

然后再把 `s[r]` 加入集合。

------

## 代码

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # 当前滑动窗口中的字符
        char_set = set()

        # 左边界
        l = 0

        # 最长无重复子串长度
        res = 0

        # r 是窗口右边界
        for r in range(len(s)):

            # 如果 s[r] 已经存在于窗口中，
            # 就不断缩小左边界，直到重复字符被移除
            while s[r] in char_set:
                char_set.remove(s[l])
                l += 1

            # 此时 s[r] 已经不会造成重复
            char_set.add(s[r])

            # 当前窗口是 s[l : r + 1]
            res = max(res, r - l + 1)

        return res
```

------

# 示例过程

假设：

```text
s = "abcabcbb"
```

开始：

```text
l = 0
char_set = {}
```

### `r = 0`

字符：

```text
a
```

加入集合：

```text
{a}
```

窗口：

```text
"a"
```

长度：

```text
1
```

------

### `r = 1`

字符：

```text
b
```

集合：

```text
{a, b}
```

窗口：

```text
"ab"
```

长度：

```text
2
```

------

### `r = 2`

字符：

```text
c
```

集合：

```text
{a, b, c}
```

窗口：

```text
"abc"
```

长度：

```text
3
```

------

### `r = 3`

又遇到：

```text
a
```

但是：

```text
a in char_set
```

所以开始缩小窗口。

删除 `s[l]`：

```text
删除 a
l = 1
```

现在集合：

```text
{b, c}
```

再加入新的 `a`：

```text
{a, b, c}
```

窗口：

```text
"bca"
```

长度仍然是：

```text
3
```

------

# 为什么滑动窗口是 O(n)？

代码中虽然有：

```python
for ...
    while ...
```

但它并不是 `O(n²)`。

原因是每个字符最多经历两次操作：

1. 被右指针加入窗口一次
2. 被左指针移出窗口一次

`r` 只向右移动：

```text
0 -> 1 -> 2 -> ... -> n-1
```

`l` 也只向右移动：

```text
0 -> 1 -> 2 -> ... -> n-1
```

两个指针都不会向后退。

所以总操作次数最多是常数倍的 `n`：

```text
O(n)
```

------

## 复杂度

### 时间复杂度

```text
O(n)
```

### 空间复杂度

```text
O(m)
```

其中 `m` 是字符串中可能出现的不同字符数量。

------

# 3. 最优滑动窗口：Hash Map + 直接跳跃

前面的滑动窗口已经是：

```text
O(n)
```

但还可以进一步优化实现方式。

虽然渐进时间复杂度不会变得更好，但可以避免：

```python
while ...
```

逐个删除字符。

------

## 核心思路

前面的 Set 解法遇到重复字符时，会这样移动：

```text
l -> l+1 -> l+2 -> l+3 -> ...
```

直到旧的重复字符被移出去。

但如果我们知道：

> 这个重复字符上一次出现在哪里

那么可以直接把 `l` 跳过去。

因此我们使用一个哈希表：

```python
last_seen = {}
```

保存：

```text
字符 -> 最近一次出现的位置
```

例如：

```text
s = "abcba"
```

扫描到：

```text
a b c b
      ↑
```

新的 `b` 出现在索引 `3`。

之前的 `b` 出现在：

```text
index = 1
```

那么为了删除旧的 `b`，左边界至少应该移动到：

```text
1 + 1 = 2
```

所以直接：

```python
l = 2
```

不需要逐个删除。

------

# 为什么是 `last_seen[char] + 1`？

假设：

```text
s = "abca"
```

旧的 `a`：

```text
index = 0
```

新的 `a`：

```text
index = 3
```

如果我们想让窗口中不再包含旧 `a`，左边界必须移动到旧 `a` 的后一位：

```text
旧 a
↓
a b c a
  ↑
  新的 l
```

也就是：

```python
last_seen["a"] + 1
```

------

# 为什么必须使用 `max`？

这是这道题最容易出错的地方之一。

正确代码：

```python
l = max(l, last_seen[s[r]] + 1)
```

不能简单写：

```python
l = last_seen[s[r]] + 1
```

原因是：

> 某个字符之前的出现位置可能已经在当前窗口左侧。

这种情况下，它已经不属于当前窗口，因此不应该让 `l` 往回移动。

------

## 一个经典反例：`"abba"`

考虑：

```text
s = "abba"
```

索引：

```text
0 1 2 3
a b b a
```

### `r = 0`

```text
a
```

窗口：

```text
"a"
l = 0
```

记录：

```python
last_seen["a"] = 0
```

------

### `r = 1`

```text
b
```

窗口：

```text
"ab"
```

记录：

```python
last_seen["b"] = 1
```

------

### `r = 2`

又遇到：

```text
b
```

之前：

```text
last_seen["b"] = 1
```

所以：

```python
l = 1 + 1 = 2
```

窗口现在是：

```text
"b"
```

------

### `r = 3`

遇到：

```text
a
```

之前：

```text
last_seen["a"] = 0
```

但注意：

```text
l = 2
```

旧的 `a` 在索引 `0`，已经不属于当前窗口。

如果错误写：

```python
l = last_seen["a"] + 1
```

那么：

```text
l = 1
```

左指针竟然从：

```text
2
```

退回到了：

```text
1
```

这是错误的。

所以必须写：

```python
l = max(l, last_seen[s[r]] + 1)
```

这样：

```text
max(2, 1) = 2
```

左指针不会后退。

------

# 算法步骤

1. 创建字典：

```python
last_seen = {}
```

用于记录每个字符最近一次出现的位置。

1. 初始化：

```python
l = 0
res = 0
```

1. 使用 `r` 遍历字符串
2. 如果：

```python
s[r] in last_seen
```

说明这个字符曾经出现过。

更新：

```python
l = max(l, last_seen[s[r]] + 1)
```

1. 更新当前字符的最近位置：

```python
last_seen[s[r]] = r
```

1. 当前窗口长度：

```python
r - l + 1
```

1. 更新答案。

------

# 最优代码

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # 记录：
        # 字符 -> 最近一次出现的索引
        last_seen = {}

        # 当前窗口左边界
        l = 0

        # 最长无重复子串长度
        res = 0

        # r 是当前窗口右边界
        for r in range(len(s)):
            char = s[r]

            # 如果这个字符以前出现过，
            # 左边界至少要移动到其上一次位置的后一位
            #
            # max(...) 非常重要：
            # 防止左指针向后移动
            if char in last_seen:
                l = max(l, last_seen[char] + 1)

            # 更新当前字符最近一次出现的位置
            last_seen[char] = r

            # 当前窗口为 s[l : r + 1]
            res = max(res, r - l + 1)

        return res
```

------

# Python 字典 `dict`

这里使用的：

```python
last_seen = {}
```

是 Python 的字典 `dict`。

它本质上也是一种哈希表。

例如：

```python
last_seen = {
    "a": 3,
    "b": 5,
    "c": 7
}
```

表示：

```text
a 最近出现在索引 3
b 最近出现在索引 5
c 最近出现在索引 7
```

可以通过：

```python
last_seen["a"]
```

得到：

```text
3
```

字典的以下操作平均都是：

```text
O(1)
```

包括：

```python
char in last_seen
last_seen[char]
last_seen[char] = index
```

因此非常适合记录字符最近一次出现的位置。

------

# Set 解法 vs Hash Map 解法

两个滑动窗口解法的区别主要在于：

### Set

记录：

```text
“当前窗口中有哪些字符”
```

遇到重复字符时：

```python
while s[r] in char_set:
    ...
```

需要不断移动左边界。

### Hash Map

记录：

```text
“每个字符最近一次在哪里出现”
```

遇到重复字符时：

```python
l = max(l, last_seen[s[r]] + 1)
```

可以直接跳到正确的位置。

------

## 对比

| 方法                      | 时间复杂度 | 空间复杂度 | 左指针移动方式   |
| ------------------------- | ---------- | ---------- | ---------------- |
| 暴力                      | `O(n²)`    | `O(m)`     | 每个起点重新开始 |
| Sliding Window + Set      | `O(n)`     | `O(m)`     | 逐个向右移动     |
| Sliding Window + Hash Map | `O(n)`     | `O(m)`     | 直接跳到目标位置 |

Set 和 Hash Map 两种滑动窗口的渐进复杂度都是：

```text
O(n)
```

因此第三种并不是在 Big-O 意义上比第二种更快，而是：

> 它能够通过记录上一次出现的位置，让左指针一步跳到正确位置，实现更直接。

------

# 常见错误

## 1. 移动左指针时忘记 `max`

错误：

```python
l = last_seen[s[r]] + 1
```

正确：

```python
l = max(l, last_seen[s[r]] + 1)
```

因为左指针：

> 只能向右移动，绝不能向左移动。

典型反例：

```text
"abba"
```

------

## 2. 忘记更新最近出现的位置

无论当前字符是不是重复，都需要执行：

```python
last_seen[s[r]] = r
```

例如：

```text
s = "abca...a"
```

如果没有及时更新第一次重复后的 `a` 的位置，那么之后再遇到 `a` 时，拿到的仍然是过期索引，就会导致窗口计算错误。

因此逻辑应该始终是：

```python
if char in last_seen:
    l = max(l, last_seen[char] + 1)

last_seen[char] = r
```

------

## 3. 窗口长度忘记 `+1`

当前窗口：

```text
[l, r]
```

左右两端都是包含在窗口中的。

因此长度是：

```python
r - l + 1
```

而不是：

```python
r - l
```

例如：

```text
l = 2
r = 2
```

窗口中明明有一个字符。

正确：

```text
2 - 2 + 1 = 1
```

错误：

```text
2 - 2 = 0
```

------

## 4. 混淆 Substring 和 Subsequence

题目要求的是：

```text
substring
```

即**子串**。

必须连续。

例如：

```text
s = "abcdaef"
```

不能因为：

```text
b c d e f
```

这些字符能从原字符串中挑出来，就认为它们组成一个合法答案。

因为它们不一定连续。

本题维护的永远是连续窗口：

```python
s[l:r + 1]
```

这也是为什么滑动窗口特别适合本题。

------

# 滑动窗口的通用思考方式

这道题也是面试中非常经典的滑动窗口模板。

通常当题目要求：

> 找一个满足某种条件的最长/最短**连续子数组或子串**

就可以考虑滑动窗口。

通用结构大致为：

```python
l = 0

for r in range(len(data)):
    # 将 data[r] 加入窗口

    while 窗口不满足条件:
        # 移除 data[l]
        l += 1

    # 此时窗口重新满足条件
    # 更新答案
```

本题中的“窗口合法条件”就是：

```text
窗口中不存在重复字符
```

所以 Set 版本正好对应这个经典模板：

```python
for r in range(len(s)):
    while s[r] in char_set:
        char_set.remove(s[l])
        l += 1

    char_set.add(s[r])
    res = max(res, r - l + 1)
```

------

# 面试中的核心总结

这道题最重要的思想是：

> **维护一个始终不存在重复字符的滑动窗口。**

Set 版本的核心逻辑：

```python
while s[r] in char_set:
    char_set.remove(s[l])
    l += 1
```

Hash Map 优化版本则记录每个字符：

```text
最近一次出现的位置
```

从而可以直接：

```python
l = max(l, last_seen[s[r]] + 1)
```

跳过重复字符。

如果在面试中使用 Hash Map 解法，可以这样解释：

> “我使用左右两个指针维护一个不包含重复字符的窗口，同时用哈希表记录每个字符最近一次出现的位置。当右指针遇到重复字符时，我将左指针移动到这个字符上一次出现位置的后一位。由于旧的出现位置可能已经在当前窗口之外，所以使用 `max(left, last_seen[char] + 1)`，保证左指针不会向后移动。每个字符只被右指针访问一次，因此时间复杂度是 `O(n)`，空间复杂度是 `O(m)`。”

最值得记住的三行代码是：

```python
if char in last_seen:
    l = max(l, last_seen[char] + 1)

last_seen[char] = r
res = max(res, r - l + 1)
```