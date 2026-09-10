# 字母异位词分组（Group Anagrams）

给定一个字符串数组 `strs`，请将所有互为**字母异位词（anagram）**的字符串分组到同一个子列表中。

返回结果的顺序可以是任意的。

所谓**字母异位词**，是指两个字符串包含完全相同的字符，并且每个字符出现的次数也相同，只是字符排列顺序可能不同。

例如：

- `"eat"` 和 `"tea"` 是字母异位词。
- `"act"` 和 `"cat"` 是字母异位词。
- `"rat"` 和 `"car"` 不是字母异位词。

### 示例 1

```text
输入：
strs = ["act","pots","tops","cat","stop","hat"]

输出：
[["hat"],["act","cat"],["stop","pots","tops"]]
```

输出顺序并不重要，例如下面的结果同样正确：

```text
[["act","cat"],["hat"],["pots","tops","stop"]]
```

### 示例 2

```text
输入：
strs = ["x"]

输出：
[["x"]]
```

### 示例 3

```text
输入：
strs = [""]

输出：
[[""]]
```

### 约束条件

```text
1 <= strs.length <= 10000
0 <= strs[i].length <= 100
strs[i] 只包含小写英文字母
```

------

# 核心思路

这道题真正需要解决的问题是：

> **如何为所有互为字母异位词的字符串找到一个相同的“标识”？**

如果我们能构造这样一个统一的标识，就可以：

```text
标识 -> 所有具有该标识的字符串
```

然后使用哈希表进行分组。

例如：

```text
"act" -> 某个 key
"cat" -> 同一个 key
```

那么它们自然就会被放进同一个列表。

这道题常见的两种 key 构造方式是：

1. **排序后的字符串**
2. **26 个字母的出现次数**

这两种方法本质完全一样：

> 构造一个能够唯一表示“字符串字符组成”的 canonical representation（规范化表示）。

------

# 解法一：排序 + 哈希表

## 思路

如果两个字符串是字母异位词，那么将它们的字符排序以后，结果一定相同。

例如：

```text
"eat" -> "aet"
"tea" -> "aet"
"ate" -> "aet"
```

所以我们可以把：

```python
''.join(sorted(s))
```

作为这个字符串所属字母异位词组的 key。

例如：

```text
"act"  -> "act"
"cat"  -> "act"

"pots" -> "opst"
"tops" -> "opst"
"stop" -> "opst"
```

于是哈希表最终可能是：

```text
{
    "act": ["act", "cat"],
    "opst": ["pots", "tops", "stop"],
    "aht":  ["hat"]
}
```

最后只需要返回哈希表中的所有 value。

------

## 算法步骤

1. 创建一个哈希表 `res`。
2. 遍历 `strs` 中的每个字符串 `s`。
3. 将 `s` 排序。
4. 把排序后的字符串作为 key。
5. 将原始字符串 `s` 加入这个 key 对应的列表。
6. 最后返回所有分组。

------

## Python 实现

```python
from typing import List
from collections import defaultdict


class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        # key: 排序后的字符串
        # value: 所有排序结果相同的原始字符串
        groups = defaultdict(list)

        for s in strs:
            # sorted(s) 会返回一个字符列表
            # 例如：
            # "cat" -> ['a', 'c', 't']
            #
            # ''.join(...) 再将字符列表拼回字符串：
            # ['a', 'c', 't'] -> "act"
            key = ''.join(sorted(s))

            # 所有具有相同 key 的字符串一定互为字母异位词
            groups[key].append(s)

        # groups.values() 返回所有分组
        return list(groups.values())
```

------

# 为什么这个算法正确？

关键性质是：

> 两个字符串互为字母异位词，当且仅当它们排序后的结果完全相同。

例如：

```text
"pots"
"tops"
"stop"
```

排序后都是：

```text
"opst"
```

因此它们一定会进入：

```python
groups["opst"]
```

而如果两个字符串排序后的结果不同，就意味着：

- 至少存在一个不同的字符，或者
- 某个字符出现次数不同，

所以它们不可能是字母异位词。

因此，这个 key 可以正确地区分不同的字母异位词组。

------

# 时间复杂度

设：

- `m` = 字符串数量
- `n` = 最长字符串长度

对于每个字符串，我们需要排序：

```text
O(n log n)
```

一共有 `m` 个字符串，因此总时间复杂度为：

```text
O(m × n log n)
```

### 空间复杂度

排序得到的 key、哈希表以及最终结果需要存储字符串，因此总体空间复杂度可以写作：

```text
O(m × n)
```

严格来说，如果不计算最终返回结果，额外空间会因 Python 排序实现等细节有所不同。不过代码面试中写：

```text
Space: O(mn)
```

通常完全足够。

------

# Python 知识：`defaultdict`

这里使用：

```python
from collections import defaultdict

groups = defaultdict(list)
```

`defaultdict` 是 Python `dict` 的一个子类。

普通字典中，如果 key 不存在：

```python
d = {}

d["abc"].append("abc")
```

会直接报：

```text
KeyError
```

因此通常需要：

```python
if key not in d:
    d[key] = []

d[key].append(s)
```

而使用：

```python
defaultdict(list)
```

以后，当访问一个不存在的 key 时，它会自动创建：

```python
[]
```

所以可以直接：

```python
groups[key].append(s)
```

这在“分组”类问题中非常常见。

例如：

```python
groups = defaultdict(list)

groups["fruit"].append("apple")
groups["fruit"].append("banana")
groups["animal"].append("cat")
```

最终：

```python
{
    "fruit": ["apple", "banana"],
    "animal": ["cat"]
}
```

### 面试记忆

看到这种模式：

```text
key -> 一组元素
```

通常可以第一时间考虑：

```python
defaultdict(list)
```

------

# Python 知识：`sorted()`

`sorted()` 是 Python 内置排序函数。

例如：

```python
sorted("cat")
```

返回的是：

```python
['a', 'c', 't']
```

注意，它返回的是：

```python
list
```

而不是字符串。

所以这里需要：

```python
''.join(sorted(s))
```

把字符重新拼接成字符串。

例如：

```python
''.join(['a', 'c', 't'])
```

得到：

```text
"act"
```

------

# 解法二：字符频率 + 哈希表

这个方法比排序更高效，也是这道题很经典的优化方案。

## 核心思路

字母异位词有一个非常重要的性质：

> 每个字符出现的次数一定完全相同。

例如：

```text
"eat"
```

包含：

```text
a: 1
e: 1
t: 1
```

而：

```text
"tea"
```

同样包含：

```text
a: 1
e: 1
t: 1
```

所以我们不必排序。

由于题目明确规定字符串只包含：

```text
a - z
```

一共只有 26 个小写英文字母。

因此，可以创建一个长度为 26 的数组：

```python
count = [0] * 26
```

分别统计：

```text
a b c d ... z
```

出现的次数。

------

# 一个具体例子

假设字符串是：

```text
"abbc"
```

那么：

```text
a -> 1
b -> 2
c -> 1
其他 -> 0
```

对应的数组大致是：

```text
[1, 2, 1, 0, 0, ..., 0]
```

而：

```text
"bcab"
```

也会得到完全一样的数组：

```text
[1, 2, 1, 0, 0, ..., 0]
```

因此它们属于同一个字母异位词组。

------

# 算法步骤

对于每个字符串：

1. 创建长度为 `26` 的计数数组。
2. 遍历字符串中的每个字符。
3. 计算该字符对应 `0 ~ 25` 中的哪个位置。
4. 增加对应位置的计数。
5. 将计数数组转换成 tuple。
6. 使用这个 tuple 作为哈希表 key。
7. 将当前字符串加入对应分组。

------

# Python 实现

```python
from typing import List
from collections import defaultdict


class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        # key:
        # 26 个字母对应的出现次数
        #
        # value:
        # 具有相同字符频率的所有字符串
        groups = defaultdict(list)

        for s in strs:
            # count[0] 表示 'a' 出现次数
            # count[1] 表示 'b' 出现次数
            # ...
            # count[25] 表示 'z' 出现次数
            count = [0] * 26

            for char in s:
                # 将字符转换为 0 ~ 25 的索引
                index = ord(char) - ord('a')
                count[index] += 1

            # list 不能作为 dictionary 的 key，
            # 所以需要转换为不可变的 tuple
            key = tuple(count)

            groups[key].append(s)

        return list(groups.values())
```

------

# `ord()` 是什么？

`ord()` 是 Python 内置函数，用于获得一个字符对应的 Unicode code point。

例如：

```python
ord('a')
```

得到：

```text
97
```

而：

```python
ord('b')
```

得到：

```text
98
```

因此：

```python
ord('b') - ord('a')
```

就是：

```text
98 - 97 = 1
```

于是可以实现：

```text
a -> 0
b -> 1
c -> 2
...
z -> 25
```

所以：

```python
count[ord(char) - ord('a')] += 1
```

就是在统计对应字母出现的次数。

可以把它理解成：

```text
字符 -> 数组下标
```

------

# 为什么需要 `tuple(count)`？

这是这份代码里一个非常值得理解的 Python 细节。

我们可能最开始想直接：

```python
groups[count].append(s)
```

但是这样会报错：

```text
TypeError: unhashable type: 'list'
```

原因是 Python 字典的 key 必须是**可哈希（hashable）**的对象。

通常来说，字典 key 必须是不可变的，例如：

```python
str
int
tuple
```

而：

```python
list
```

是可变的。

例如：

```python
count = [1, 2, 3]

count[0] = 100
```

它的内容随时可以变化，因此 Python 不允许它作为 dictionary key。

但是：

```python
tuple(count)
```

得到：

```python
(1, 2, 3)
```

tuple 是不可变对象，因此可以作为字典 key。

所以这里：

```python
key = tuple(count)
```

是非常关键的一步。

------

# 为什么频率数组方法正确？

两个字符串互为字母异位词，当且仅当：

```text
a 出现次数相同
b 出现次数相同
c 出现次数相同
...
z 出现次数相同
```

也就是说：

```python
tuple(count1) == tuple(count2)
```

当且仅当两个字符串互为字母异位词。

因此，字符频率数组可以作为每个字母异位词组的唯一标识。

------

# 时间复杂度

设：

- `m` = 字符串数量
- `n` = 最长字符串长度

我们需要访问每个字符串中的每个字符一次。

因此：

```text
时间复杂度：O(m × n)
```

相比排序方法：

```text
O(m × n log n)
```

这里省掉了排序过程中的：

```text
log n
```

所以理论上更优。

------

# 空间复杂度

对于每个字符串，临时创建：

```python
[0] * 26
```

因为 `26` 是常数，所以单个数组只需要：

```text
O(1)
```

额外空间。

如果**不计算最终返回结果**，哈希表中每个 group 的 key 是长度固定为 26 的 tuple，因此辅助空间大约是：

```text
O(m)
```

如果把输出中的所有字符串也算进去，则总空间为：

```text
O(m × n)
```

------

# 两种方法对比

| 方法     | Key             | 时间复杂度    | 优点                 | 缺点                   |
| -------- | --------------- | ------------- | -------------------- | ---------------------- |
| 排序     | 排序后的字符串  | `O(mn log n)` | 简单、直观、容易写对 | 需要排序               |
| 字符计数 | 26 位频率 tuple | `O(mn)`       | 理论上更快           | 实现稍复杂，依赖字符集 |

如果是代码面试，我建议：

**先想到排序方案完全没有问题。**

因为它：

```text
简单
容易解释
容易实现
不容易写错
```

如果面试官继续问：

> Can you do better than sorting?

这时候就可以自然地优化成字符频率方案。

------

# 一个很重要的面试思维：如何设计 Hash Key

这道题很典型地考察：

> 如何把复杂对象转换成一个能够放进 Hash Map 的统一表示。

原始字符串：

```text
"eat"
"tea"
"ate"
```

虽然长得不同，但我们希望把它们映射到同一个 key：

```text
"eat" ─┐
"tea" ─┼──> SAME KEY
"ate" ─┘
```

排序方案构造的是：

```text
"aet"
```

字符计数方案构造的是类似：

```text
(1, 0, 0, 0, 1, ..., 1, ...)
```

这种思想在很多题目里都会出现：

```text
原始对象
   ↓
构造 canonical key
   ↓
HashMap[key].append(object)
```

所以遇到这样的题：

> “把具有某种等价关系的元素分组”

一个很好的思考方向就是：

> **能不能为同一组中的所有元素设计一个相同的 hash key？**

------

# 面试中可以怎么讲

如果面试官让你解释思路，可以比较简洁地说：

> Two strings are anagrams if they contain exactly the same characters with the same frequencies. One simple way to generate a canonical representation is to sort each string. All anagrams will produce the same sorted string, which we can use as a hash map key. Then we group all original strings with the same key together.

如果面试官问能否优化：

> Since the strings only contain 26 lowercase English letters, instead of sorting, we can count the frequency of each character using a fixed-size array of length 26. The frequency tuple uniquely identifies an anagram group, reducing the time complexity from `O(mn log n)` to `O(mn)`.

------

# 最终推荐版本

如果目标是**面试中追求最优复杂度**，我更推荐记住下面这个版本：

```python
from typing import List
from collections import defaultdict


class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        groups = defaultdict(list)

        for s in strs:
            # 统计 a-z 每个字符出现的次数
            count = [0] * 26

            for char in s:
                count[ord(char) - ord('a')] += 1

            # tuple 是不可变对象，可以作为哈希表 key
            groups[tuple(count)].append(s)

        return list(groups.values())
```

**时间复杂度：**

```text
O(mn)
```

**空间复杂度：**

```text
O(m) auxiliary
O(mn) including output
```

这道题最值得记住的不是 `defaultdict` 或 `ord()` 本身，而是这个模式：

> **分组问题 → 找到同组元素共有的特征 → 将该特征作为 Hash Map 的 key。**

而对于 Anagram：

> **排序后的字符串**和**字符频率**都是很自然的 canonical key。
