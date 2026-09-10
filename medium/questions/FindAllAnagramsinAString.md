# Find All Anagrams in a String（找到字符串中所有字母异位词）

给定两个字符串 `s` 和 `p`，返回 `s` 中所有是 `p` 的 **字母异位词（Anagram）** 的子串的起始索引。答案可以按 **任意顺序** 返回。

## 什么是 Anagram？

如果两个字符串包含完全相同的字符，并且每个字符出现的次数完全相同，只是排列顺序不同，那么它们互为字母异位词。

例如：

```text
"abc"
"bac"
"cba"
```

互为 Anagram。

---

## 示例 1

```text
输入：s = "cbaebabacd", p = "abc"
输出：[0, 6]
```

解释：

- 从索引 `0` 开始的子串 `"cba"` 是 `"abc"` 的 Anagram。
- 从索引 `6` 开始的子串 `"bac"` 也是 `"abc"` 的 Anagram。

## 示例 2

```text
输入：s = "abab", p = "ab"
输出：[0, 1, 2]
```

因为长度为 `2` 的三个子串：

```text
"ab"
"ba"
"ab"
```

都和 `"ab"` 有相同的字符频率。

---

## 约束

- `1 <= s.length, p.length <= 3 * 10^4`
- `s` 和 `p` 只包含小写英文字母

---

# 核心观察

如果 `s` 中某个子串是 `p` 的 Anagram，那么它必须满足：

1. 长度等于 `len(p)`；
2. 每个字符的出现次数和 `p` 完全相同。

因此问题可以转化为：

> 在 `s` 中检查所有长度为 `len(p)` 的窗口，并判断窗口内字符频率是否等于 `p` 的字符频率。

这是一个典型的 **固定长度滑动窗口** 问题。

---

# 解法一：滑动窗口 + Counter

## 思路

假设：

```python
m = len(p)
```

我们维护 `s` 中一个长度始终为 `m` 的窗口。

每向右移动一步：

- 一个新字符进入窗口；
- 一个旧字符离开窗口；
- 更新字符频率；
- 判断当前窗口是否和 `p` 的字符频率相同。

这样就不需要对每个子串重新统计字符。

## 代码

```python
from typing import List
from collections import Counter


class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        n = len(s)
        m = len(p)

        # p 比 s 长，不可能存在合法窗口
        if m > n:
            return []

        res = []

        # p 的字符频率
        p_count = Counter(p)

        # 初始化第一个长度为 m 的窗口
        window_count = Counter(s[:m])

        if window_count == p_count:
            res.append(0)

        # 从第二个窗口开始向右滑动
        for right in range(m, n):
            # 新字符进入窗口
            window_count[s[right]] += 1

            # 左侧旧字符离开窗口
            left_char = s[right - m]
            window_count[left_char] -= 1

            # 为了保持 Counter 干净，计数为 0 时删除
            if window_count[left_char] == 0:
                del window_count[left_char]

            # 当前窗口的起始位置
            left = right - m + 1

            if window_count == p_count:
                res.append(left)

        return res
```

## Python：`collections.Counter`

`Counter` 是 Python 标准库 `collections` 中用于统计元素频率的类。

```python
from collections import Counter

Counter("abca")
```

得到：

```python
Counter({'a': 2, 'b': 1, 'c': 1})
```

因此：

```python
Counter("abc") == Counter("bca")
```

结果为：

```python
True
```

这使得它非常适合判断两个字符串是否互为 Anagram。

## 复杂度

设：

```text
n = len(s)
```

由于题目只包含 26 个小写字母，所以比较两个频率表最多检查 26 个字符，可以视为常数时间。

时间复杂度：

\[
O(n)
\]

空间复杂度：

\[
O(1)
\]

这里的 `O(1)` 建立在字符集固定为 26 个小写英文字母的前提上。

---

# 解法二：滑动窗口 + 长度为 26 的数组

这是面试中更推荐的版本。

因为题目明确说明只包含：

```text
'a' ~ 'z'
```

总共 26 个字符，所以可以直接使用长度为 26 的数组保存频率。

## 字符映射

我们希望：

```text
'a' -> 0
'b' -> 1
'c' -> 2
...
'z' -> 25
```

Python 中可以使用：

```python
ord(ch) - ord('a')
```

例如：

```python
ord('c') - ord('a')
```

结果是：

```text
2
```

## 代码

```python
from typing import List


class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        n = len(s)
        m = len(p)

        if m > n:
            return []

        res = []

        # p_count[i] 表示 p 中第 i 个字母的出现次数
        p_count = [0] * 26

        # window_count[i] 表示当前窗口中第 i 个字母的出现次数
        window_count = [0] * 26

        # 统计 p
        for ch in p:
            p_count[ord(ch) - ord('a')] += 1

        # 初始化第一个窗口
        for i in range(m):
            window_count[ord(s[i]) - ord('a')] += 1

        if window_count == p_count:
            res.append(0)

        # 固定长度滑动窗口
        for right in range(m, n):
            # 新字符进入窗口
            window_count[ord(s[right]) - ord('a')] += 1

            # 旧字符离开窗口
            window_count[ord(s[right - m]) - ord('a')] -= 1

            # 当前窗口起点
            left = right - m + 1

            if window_count == p_count:
                res.append(left)

        return res
```

## 为什么推荐这个版本？

相比 `Counter`：

- 不需要哈希表；
- 数据结构固定；
- 访问字符频率是直接数组索引；
- 代码结构清晰；
- 非常容易解释时间复杂度。

因此在面试中，我更推荐这个版本。

## 复杂度

时间复杂度：

\[
O(n)
\]

空间复杂度：

\[
O(1)
\]

因为两个数组长度始终都是 26。

---

# 解法三：滑动窗口 + `need` 计数

还可以进一步维护：

```text
need = 当前窗口还缺多少个字符才能组成 p
```

这样就不需要每次比较两个长度为 26 的数组。

## 思路

先统计 `p` 中每个字符还需要多少个。

当右边加入一个字符时：

- 如果这个字符当前仍然是需要的，`need -= 1`；
- 无论是否多余，都减少该字符的剩余需求数。

当左边移出一个字符时：

- 如果移出的字符不是多余字符，那么窗口会重新缺少它，`need += 1`；
- 恢复该字符的需求数。

当：

```python
need == 0
```

并且窗口长度等于 `len(p)` 时，当前窗口就是 Anagram。

## 代码

```python
from typing import List


class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        if len(p) > len(s):
            return []

        count = [0] * 26

        # count[i]：当前还需要多少个对应字符
        for ch in p:
            count[ord(ch) - ord('a')] += 1

        need = len(p)
        left = 0
        res = []

        for right in range(len(s)):
            right_idx = ord(s[right]) - ord('a')

            # 当前字符仍然是我们需要的
            if count[right_idx] > 0:
                need -= 1

            # 消耗一个该字符
            count[right_idx] -= 1

            # 窗口长度不能超过 len(p)
            if right - left + 1 > len(p):
                left_idx = ord(s[left]) - ord('a')

                # 如果这个字符不是多余字符，
                # 移出后我们会重新缺少一个
                if count[left_idx] >= 0:
                    need += 1

                count[left_idx] += 1
                left += 1

            if need == 0:
                res.append(left)

        return res
```

这个版本同样是：

\[
O(n)
\]

时间复杂度和：

\[
O(1)
\]

空间复杂度。

不过从可读性来说，面试中通常还是“两个频率数组”的版本更容易解释。

---

# 暴力解法

最直接的办法是：

1. 枚举 `s` 中所有长度为 `len(p)` 的子串；
2. 对每个子串排序；
3. 判断是否等于排序后的 `p`。

```python
from typing import List


class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        res = []
        m = len(p)

        target = sorted(p)

        for i in range(len(s) - m + 1):
            if sorted(s[i:i + m]) == target:
                res.append(i)

        return res
```

## 复杂度

假设：

```text
n = len(s)
m = len(p)
```

一共有大约 `n` 个窗口，每个窗口排序需要：

\[
O(m \log m)
\]

因此总时间复杂度：

\[
O(nm \log m)
\]

在：

```text
n, m <= 3 * 10^4
```

时明显不够理想。

---

# 为什么滑动窗口能优化？

假设当前窗口为：

```text
[left ........ right]
```

右移一步：

```text
     [left+1 ........ right+1]
```

只有两个字符发生变化：

1. `s[left]` 离开；
2. `s[right + 1]` 进入。

所以不需要重新统计整个新窗口。

只要：

```python
新字符频率 += 1
旧字符频率 -= 1
```

就可以完成窗口状态更新。

这就是滑动窗口优化的核心。

---

# 一个常见的固定长度滑动窗口模板

```python
# 初始化第一个窗口
for i in range(k):
    add(s[i])

check_window()

# 向右移动
for right in range(k, len(s)):
    add(s[right])
    remove(s[right - k])

    left = right - k + 1

    check_window()
```

本题中：

```python
k = len(p)
```

---

# 更紧凑的推荐写法

```python
from typing import List


class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        n, m = len(s), len(p)

        if m > n:
            return []

        p_count = [0] * 26
        window = [0] * 26

        for ch in p:
            p_count[ord(ch) - ord('a')] += 1

        res = []

        for right, ch in enumerate(s):
            # 当前字符进入窗口
            window[ord(ch) - ord('a')] += 1

            # 窗口长度超过 m 时，移除最左边字符
            if right >= m:
                old = s[right - m]
                window[ord(old) - ord('a')] -= 1

            # 窗口长度达到 m 后判断
            if right >= m - 1 and window == p_count:
                res.append(right - m + 1)

        return res
```

---

# 常见错误

## 1. 忘记 Anagram 的长度必须完全相同

例如：

```text
p = "ab"
```

`"aab"` 包含 `a` 和 `b`，但不是 `"ab"` 的 Anagram，因为长度和字符频率不相同。

因此窗口长度必须固定为：

```python
len(p)
```

---

## 2. 每个窗口重新统计字符

例如：

```python
for i in range(...):
    count = Counter(s[i:i + m])
```

这样每次都会重新扫描整个窗口。

滑动窗口只需要更新：

```text
一个进入字符
一个离开字符
```

即可。

---

## 3. 忘记删除离开窗口的字符

如果只不断执行：

```python
window_count[s[right]] += 1
```

却没有：

```python
window_count[s[right - m]] -= 1
```

窗口就会越来越大，不再表示固定长度子串。

---

## 4. 混淆 `right - m` 和窗口起点

当 `right` 表示刚进入窗口的新字符位置：

离开窗口的是：

```python
s[right - m]
```

新窗口起点是：

```python
right - m + 1
```

例如：

```text
m = 3
right = 3
```

旧窗口：

```text
index 0, 1, 2
```

加入：

```text
index 3
```

所以离开的是：

```text
index 0
```

即：

```python
right - m
```

新窗口起点则是：

```text
index 1
```

即：

```python
right - m + 1
```

---

# 方法对比

| 方法 | 时间复杂度 | 空间复杂度 | 推荐度 |
|---|---:|---:|---|
| 每个子串排序 | `O(nm log m)` | `O(m)` | 不推荐 |
| 滑动窗口 + Counter | `O(n)` | `O(1)` | 推荐 |
| 滑动窗口 + 26 数组 | `O(n)` | `O(1)` | **最推荐** |
| 滑动窗口 + `need` | `O(n)` | `O(1)` | 进阶 |

---

# 与 Permutation in String 的关系

这道题与经典题：

```text
Permutation in String
```

本质上几乎完全相同。

区别是：

### Permutation in String

只需要判断：

```text
是否存在一个 Anagram？
```

找到后可以直接：

```python
return True
```

### Find All Anagrams in a String

需要返回：

```text
所有 Anagram 的起点
```

所以每次找到后：

```python
res.append(left)
```

然后继续滑动窗口。

---

# 面试总结

这道题最重要的识别过程是：

```text
寻找 p 的 Anagram
        ↓
字符频率必须相同
        ↓
Anagram 长度必须等于 len(p)
        ↓
只需要检查 s 中所有固定长度窗口
        ↓
相邻窗口只有一个字符进入、一个字符离开
        ↓
使用滑动窗口维护字符频率
```

推荐在面试中使用：

> **固定长度滑动窗口 + 长度为 26 的频率数组**

最终时间复杂度：

\[
oxed{O(n)}
\]

空间复杂度：

\[
oxed{O(1)}
\]

其中 `O(1)` 是因为字符集固定为 26 个小写英文字母。
