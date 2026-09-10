# Minimum Window Substring（最小覆盖子串）

给定两个字符串 `s` 和 `t`，请返回 `s` 中最短的一个**子串**，使得这个子串包含 `t` 中的所有字符，包括重复出现的字符。

如果不存在这样的子串，返回空字符串 `""`。

可以假设正确答案始终唯一。

## 示例 1

```text
输入：
s = "OUZODYXAZV"
t = "XYZ"

输出：
"YXAZ"
```

解释：

`"YXAZ"` 是包含 `t` 中 `"X"`、`"Y"` 和 `"Z"` 的最短子串。

------

## 示例 2

```text
输入：
s = "xyz"
t = "xyz"

输出：
"xyz"
```

------

## 示例 3

```text
输入：
s = "x"
t = "xy"

输出：
""
```

------

## 约束条件

- `1 <= s.length <= 100,000`
- `1 <= t.length <= 100,000`
- `s` 和 `t` 只包含大小写英文字母

------

# 前置知识

在解决这道题之前，建议熟悉以下概念：

### 1. 哈希表 Hash Map

用来记录字符出现的次数。

例如：

```python
t = "AABC"
```

可以统计为：

```python
{
    "A": 2,
    "B": 1,
    "C": 1
}
```

这里特别重要的一点是：

> 这道题不仅要求字符“出现”，还要求字符的**出现次数足够**。

所以 `"ABC"` 并不能覆盖 `"AABC"`，因为少了一个 `"A"`。

------

### 2. 滑动窗口 Sliding Window

滑动窗口通常适用于：

> 在数组或字符串中寻找满足某种条件的连续区间。

这道题中，我们维护一个窗口：

```text
s[l : r + 1]
```

其中：

- `l` 是窗口左边界
- `r` 是窗口右边界

核心操作只有两个：

```text
右指针 r 向右移动
→ 扩大窗口
→ 尝试让窗口满足条件

左指针 l 向右移动
→ 缩小窗口
→ 尝试找到更短答案
```

------

### 3. 双指针 Two Pointers

滑动窗口本质上是双指针的一种应用。

```text
      l
      ↓
s = A D O B E C O D E B A N C
              ↑
              r
```

两个指针都只会向右移动，不会回退。

这正是滑动窗口能够达到 `O(n)` 级别效率的原因之一。

------

# 解法一：暴力枚举

## 思路

最直接的思路是：

> 枚举 `s` 中的每一个可能的子串，然后检查这个子串是否包含 `t` 中所有字符以及对应数量。

假设我们固定一个起点 `i`：

```text
s = ABCDE
    ↑
    i
```

然后不断向右扩展终点 `j`：

```text
A
AB
ABC
ABCD
ABCDE
```

对于每一个 `[i, j]`，统计其中字符数量，再和 `t` 的字符频率进行比较。

如果当前子串已经覆盖了 `t`，就尝试更新最短答案。

------

## 算法步骤

1. 如果 `t` 为空，直接返回 `""`。
2. 使用哈希表 `countT` 统计 `t` 中每个字符需要出现多少次。
3. 初始化：
   - `res = [-1, -1]`：记录当前最优窗口的左右边界。
   - `resLen = infinity`：记录当前最短长度。
4. 枚举每一个起始位置 `i`。
5. 从 `i` 开始向右枚举结束位置 `j`：
   - 将 `s[j]` 加入当前字符统计 `countS`。
   - 遍历 `countT`，判断当前子串是否已经包含全部目标字符。
6. 如果当前窗口合法，并且比历史最优结果更短，则更新答案。
7. 最后返回最短子串；如果没有找到，则返回 `""`。

------

## Python 实现

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        if t == "":
            return ""

        # countT[c] 表示目标字符串 t 中字符 c 需要出现多少次
        countT = {}
        for c in t:
            countT[c] = 1 + countT.get(c, 0)

        # res 保存当前最优答案的左右边界
        # resLen 保存最短长度
        res = [-1, -1]
        resLen = float("infinity")

        # 枚举所有子串的左边界
        for i in range(len(s)):
            countS = {}

            # 枚举所有子串的右边界
            for j in range(i, len(s)):
                # 将 s[j] 加入当前子串
                countS[s[j]] = 1 + countS.get(s[j], 0)

                # 检查当前子串是否覆盖 t
                valid = True

                for c in countT:
                    # 如果某个字符数量不足，则当前窗口无效
                    if countS.get(c, 0) < countT[c]:
                        valid = False
                        break

                # 当前窗口合法，并且更短
                if valid and (j - i + 1) < resLen:
                    resLen = j - i + 1
                    res = [i, j]

        l, r = res

        if resLen == float("infinity"):
            return ""

        # Python 切片右边界不包含，因此要使用 r + 1
        return s[l:r + 1]
```

------

## Python 语法说明：`dict.get()`

代码中有：

```python
countT.get(c, 0)
```

Python 字典的：

```python
dictionary.get(key, default)
```

表示：

> 如果 `key` 存在，返回对应的 value；否则返回 `default`。

例如：

```python
count = {}

count.get("A", 0)
# 0
```

因此：

```python
countT[c] = 1 + countT.get(c, 0)
```

就是常见的字符计数写法。

等价于：

```python
if c not in countT:
    countT[c] = 0

countT[c] += 1
```

------

## Python 语法说明：`float("infinity")`

```python
resLen = float("infinity")
```

表示正无穷。

这里用于初始化“当前最短长度”：

```text
resLen = ∞
```

这样任何真实的字符串长度都会比它小。

也经常写成：

```python
float("inf")
```

两者效果相同。

------

## 时间复杂度

设：

- `n = len(s)`
- `m = len(t)`
- `u = t` 中不同字符的数量

首先构造 `countT`：

```text
O(m)
```

`s` 一共有大约：

```text
O(n²)
```

个子串。

对于每个子串，我们最多检查 `u` 个目标字符：

```text
O(u)
```

因此总时间复杂度为：

$O(m+n^2u)$

通常简化理解为：

$O(n^2u)$

------

## 空间复杂度

需要两个字符计数哈希表：

```text
countT
countS
```

因此空间复杂度可以记为：

$O(k)$

其中 `k` 是 `s` 和 `t` 中不同字符的总数量。

------

## 暴力解法的问题

由于：

```text
n <= 100,000
```

那么 `O(n²)` 的算法基本不可接受。

例如：

```text
n = 100,000

n² = 10,000,000,000
```

因此这道题真正需要掌握的是下面的 **Sliding Window**。

------

# 解法二：滑动窗口

## 核心思路

我们真正需要寻找的是：

> `s` 中最短的、能够覆盖 `t` 的连续窗口。

可以观察到一个非常重要的性质：

### 当当前窗口不合法时

我们缺少一些字符，所以应该：

> 移动右指针 `r`，扩大窗口。

------

### 当当前窗口已经合法时

继续扩大窗口通常只会让答案变长。

所以应该：

> 移动左指针 `l`，尽可能缩小窗口。

于是整个算法形成：

```text
不合法
→ 扩张右边界

合法
→ 收缩左边界
→ 不断记录更短答案

重新变得不合法
→ 再扩张右边界
```

这就是典型的滑动窗口模式。

------

# 一个关键设计：`have` 和 `need`

这是这道题最值得理解的部分。

假设：

```python
t = "AABC"
```

那么：

```python
countT = {
    "A": 2,
    "B": 1,
    "C": 1
}
```

这里：

```python
need = len(countT)
```

所以：

```text
need = 3
```

注意：

> `need` 不是 `len(t)`。

因为我们不是单独追踪每一个字符，而是追踪：

> 有多少种字符已经满足所需要的数量。

------

例如当前窗口的字符数量为：

```python
window = {
    "A": 2,
    "B": 1,
    "C": 0
}
```

那么：

```text
A 已满足
B 已满足
C 未满足
```

因此：

```text
have = 2
need = 3
```

此时：

```python
have != need
```

窗口不合法。

------

如果之后加入一个 `"C"`：

```python
window["C"] = 1
```

那么：

```text
A 已满足
B 已满足
C 已满足
```

于是：

```python
have = 3
need = 3
```

此时：

```python
have == need
```

意味着：

> 当前窗口已经完全覆盖了 `t`。

------

# 为什么要用“不同字符种类”而不是总字符数量？

例如：

```python
t = "AAB"
```

需要：

```text
A × 2
B × 1
```

如果我们简单统计“已经找到多少个目标字符”，处理重复字符时会非常麻烦。

而使用：

```text
have = 已经满足要求的字符种类数量
need = 总共需要满足的字符种类数量
```

逻辑会非常清晰。

------

# 算法步骤

### 第一步：统计 `t`

构造：

```python
countT
```

记录每个字符所需数量。

------

### 第二步：初始化滑动窗口

```python
window = {}
```

用于记录当前窗口中的字符数量。

同时：

```python
have = 0
need = len(countT)
```

以及：

```python
l = 0
```

------

### 第三步：不断移动右指针

对于每个：

```python
r
```

把：

```python
s[r]
```

加入当前窗口。

------

### 第四步：更新 `have`

只有当某个目标字符的数量：

```python
window[c] == countT[c]
```

时，才说明：

> 这个字符刚刚从“不满足”变成了“满足”。

因此：

```python
have += 1
```

------

### 第五步：当前窗口合法时不断缩小

只要：

```python
have == need
```

就说明窗口有效。

于是：

1. 尝试更新答案。
2. 删除左侧字符 `s[l]`。
3. `l += 1`。
4. 继续判断窗口是否仍然合法。

------

### 第六步：删除字符后更新 `have`

如果删除 `s[l]` 后：

```python
window[s[l]] < countT[s[l]]
```

说明这个字符已经不够用了。

因此：

```python
have -= 1
```

当前窗口重新变成非法状态，停止收缩，继续让右指针向右寻找新的字符。

------

# Python 实现

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        if t == "":
            return ""

        # countT:
        # 记录目标字符串 t 中，每个字符需要出现多少次
        #
        # window:
        # 记录当前滑动窗口中，每个字符实际出现多少次
        countT = {}
        window = {}

        for c in t:
            countT[c] = 1 + countT.get(c, 0)

        # need:
        # 一共有多少种目标字符需要满足
        #
        # have:
        # 当前已经有多少种字符达到了目标数量
        have = 0
        need = len(countT)

        # 当前最优答案
        res = [-1, -1]
        resLen = float("infinity")

        # 左指针
        l = 0

        # r 是右指针，不断向右扩大窗口
        for r in range(len(s)):
            c = s[r]

            # 将当前字符加入窗口
            window[c] = 1 + window.get(c, 0)

            # 如果 c 是目标字符，并且它的数量
            # 刚好达到目标要求，则多满足了一种字符
            if c in countT and window[c] == countT[c]:
                have += 1

            # 如果所有目标字符都已经满足，
            # 尝试不断缩小左边界
            while have == need:

                # 先记录当前合法窗口
                if (r - l + 1) < resLen:
                    res = [l, r]
                    resLen = r - l + 1

                # 准备移除窗口最左侧字符
                left_char = s[l]
                window[left_char] -= 1

                # 如果删除后，该字符数量低于目标要求，
                # 那么窗口不再合法
                if (
                    left_char in countT
                    and window[left_char] < countT[left_char]
                ):
                    have -= 1

                # 收缩窗口
                l += 1

        l, r = res

        if resLen == float("infinity"):
            return ""

        return s[l:r + 1]
```

------

# 滑动窗口运行示例

考虑：

```text
s = "OUZODYXAZV"
t = "XYZ"
```

目标字符为：

```python
countT = {
    "X": 1,
    "Y": 1,
    "Z": 1
}
```

所以：

```text
need = 3
```

------

右指针不断扩张：

```text
O
OU
OUZ
OUZO
OUZOD
OUZODY
OUZODYX
```

此时已经：

```text
X ✓
Y ✓
Z ✓
```

因此：

```text
have == need
```

窗口合法。

现在开始不断移动 `l`：

```text
OUZODYX
 UZODYX
  ZODYX
```

再继续缩小就会把 `"Z"` 删除，因此窗口会失效。

之后继续扩大右边界。

最终能够找到：

```text
YXAZ
```

长度为：

```text
4
```

这就是最短覆盖子串。

------

# 为什么右指针和左指针都不会回退？

这是理解时间复杂度的关键。

右指针：

```python
for r in range(len(s)):
```

从：

```text
0 → n - 1
```

只走一遍。

左指针：

```python
l += 1
```

也始终只向右移动。

虽然它位于一个 `while` 循环中，但它整个算法最多也只能从：

```text
0 → n - 1
```

走一次。

所以实际上：

```text
右指针最多移动 n 次
左指针最多移动 n 次
```

总工作量：

$O(n+n)=O(n)$

这也是滑动窗口中非常常见的复杂度分析方法。

------

# 时间复杂度

构造 `countT`：

$O(m)$

右指针遍历 `s`：

$O(n)$

左指针整个过程中最多移动 `n` 次：

$O(n)$

因此：

$O(m+2n)$

忽略常数：

$\boxed{O(n+m)}$

------

# 空间复杂度

主要使用：

```python
countT
window
```

两个哈希表。

因此空间复杂度为：

$\boxed{O(k)}$

其中 `k` 是相关字符串中不同字符的数量。

由于题目明确规定只有大小写英文字母，因此理论上最多只有：

```text
52
```

种不同字符。

所以在这道具体题目的字符集限制下，也可以认为额外空间实际上是：

$O(1)$

但面试时一般写：

$O(k)$

更加通用。

------

# 为什么条件必须是 `window[c] == countT[c]`？

这一点非常容易写错。

假设：

```python
t = "AA"
```

因此：

```python
countT["A"] = 2
```

窗口第一次遇到 `"A"`：

```text
window["A"] = 1
```

还没有满足要求。

第二次遇到 `"A"`：

```text
window["A"] = 2
```

此时刚刚满足：

```python
have += 1
```

如果之后再遇到 `"A"`：

```text
window["A"] = 3
```

虽然 `"A"` 还是满足要求，但我们不能再次：

```python
have += 1
```

否则就重复计数了。

所以必须使用：

```python
window[c] == countT[c]
```

而不是：

```python
window[c] >= countT[c]
```

------

# 为什么删除字符时使用 `<`？

同样假设：

```python
countT["A"] = 2
```

当前窗口：

```text
window["A"] = 3
```

删除一个 `"A"`：

```text
window["A"] = 2
```

仍然满足要求，所以：

```python
have
```

不能减少。

但是如果再删除一个：

```text
window["A"] = 1
```

现在才真正不满足要求。

因此判断条件为：

```python
window[left_char] < countT[left_char]
```

此时：

```python
have -= 1
```

------

# 高频易错点

## 1. 忽略 `t` 中的重复字符

例如：

```text
t = "AAB"
```

不能只判断：

```text
窗口里有没有 A 和 B
```

而应该判断：

```text
A >= 2
B >= 1
```

因此一定要记录字符频率。

------

## 2. `need` 写成 `len(t)`

错误：

```python
need = len(t)
```

正确：

```python
need = len(countT)
```

因为 `have` 统计的是：

> 已经满足要求的字符**种类数量**。

例如：

```text
t = "AABC"
```

虽然：

```text
len(t) = 4
```

但是：

```text
need = 3
```

对应：

```text
A
B
C
```

三种字符。

------

## 3. 每遇到一个目标字符都执行 `have += 1`

错误写法类似：

```python
if c in countT:
    have += 1
```

这样无法正确处理重复字符。

正确逻辑是：

```python
if c in countT and window[c] == countT[c]:
    have += 1
```

只有：

> 某种字符第一次达到所要求的数量

才增加 `have`。

------

## 4. 收缩窗口时忘记更新 `have`

删除左侧字符以后必须检查：

```python
if (
    left_char in countT
    and window[left_char] < countT[left_char]
):
    have -= 1
```

否则算法会错误地认为窗口始终有效。

------

## 5. 只用 `if have == need`

这里必须使用：

```python
while have == need:
```

而不是：

```python
if have == need:
```

因为当窗口合法以后，我们的目标不是只缩小一次，而是：

> 尽可能一直缩小，直到再缩就不合法。

这也是“寻找最小窗口”的关键。

------

## 6. 先删除左侧字符，再记录答案

正确顺序应该是：

```python
while have == need:
    # 1. 当前窗口合法，所以先更新答案

    # 2. 再删除左侧字符

    # 3. 再移动 l
```

如果先删除字符，可能会错过当前这个合法窗口。

------

## 7. Python 切片的右边界问题

答案记录的是闭区间：

```text
[l, r]
```

即 `l` 和 `r` 对应的位置都属于答案。

但 Python：

```python
s[a:b]
```

表示：

```text
[a, b)
```

右边界 `b` 不包含。

因此必须写：

```python
s[l:r + 1]
```

------

# 面试中最应该记住的模板

这道题可以抽象成下面这个滑动窗口结构：

```python
l = 0

for r in range(len(s)):
    # 1. 加入 s[r]
    add(s[r])

    # 2. 当前窗口合法时不断收缩
    while window_is_valid():

        # 3. 更新答案
        update_answer(l, r)

        # 4. 删除左侧字符
        remove(s[l])

        # 5. 左指针右移
        l += 1
```

也就是：

```text
右边界负责：
找到一个合法窗口

左边界负责：
把合法窗口压缩到尽可能小
```

这是 Minimum Window Substring 最核心的思想。

------

# 推荐面试版本

下面这个版本相对简洁，同时保留了比较清晰的变量含义：

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        if not t:
            return ""

        target = {}
        for c in t:
            target[c] = target.get(c, 0) + 1

        window = {}

        # required:
        # 需要满足多少种字符
        required = len(target)

        # formed:
        # 当前已经满足多少种字符
        formed = 0

        left = 0

        best_left = 0
        best_len = float("inf")

        for right, char in enumerate(s):
            # 扩大窗口
            window[char] = window.get(char, 0) + 1

            # 某一种目标字符刚好满足要求
            if char in target and window[char] == target[char]:
                formed += 1

            # 当前窗口有效，尽可能缩小
            while formed == required:
                current_len = right - left + 1

                if current_len < best_len:
                    best_len = current_len
                    best_left = left

                left_char = s[left]
                window[left_char] -= 1

                # 删除之后，该字符不再满足要求
                if (
                    left_char in target
                    and window[left_char] < target[left_char]
                ):
                    formed -= 1

                left += 1

        if best_len == float("inf"):
            return ""

        return s[best_left:best_left + best_len]
```

这里使用：

```python
for right, char in enumerate(s):
```

等价于：

```python
for right in range(len(s)):
    char = s[right]
```

`enumerate()` 会同时提供：

```text
索引 + 元素
```

例如：

```python
s = "ABC"

for i, c in enumerate(s):
    print(i, c)
```

输出：

```text
0 A
1 B
2 C
```

在 Python 面试代码中很常见。

------

# 解法总结

| 解法     | 核心思想               | 时间复杂度     | 空间复杂度 |
| -------- | ---------------------- | -------------- | ---------- |
| 暴力枚举 | 检查所有可能子串       | `O(m + n²u)`   | `O(k)`     |
| 滑动窗口 | 右边界扩张，左边界收缩 | **`O(n + m)`** | `O(k)`     |

这道题真正需要掌握的不是代码本身，而是下面三个判断：

```text
什么时候扩大窗口？
→ 当前窗口还不满足条件

什么时候缩小窗口？
→ 当前窗口已经满足条件

什么时候窗口有效？
→ have == need
```

以及一个非常关键的不变量：

```text
have
=
当前有多少种目标字符的数量
已经达到 countT 中的要求
```

只要把这个含义维护正确，整个滑动窗口解法就会非常自然。