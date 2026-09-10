# 有效括号（Valid Parentheses）

## 题目描述

给定一个字符串 `s`，其中只包含以下字符：

```text
'(', ')', '{', '}', '[', ']'
```

当且仅当满足以下所有条件时，字符串 `s` 才是**有效括号字符串**：

1. 每一个左括号都必须由**相同类型**的右括号闭合。
2. 左括号必须按照**正确的嵌套顺序**闭合。
3. 每一个右括号都必须存在一个与之对应的、类型相同的左括号。

如果 `s` 是有效字符串，返回 `True`；否则返回 `False`。

例如：

```text
s = "()"
=> True

s = "()[]{}"
=> True

s = "(]"
=> False

s = "([{}])"
=> True

s = "([)]"
=> False
```

------

# 解法一：暴力消除

## 思路

有效的括号字符串一定可以不断找到以下三种相邻的匹配括号：

```text
()
{}
[]
```

因此可以不断把这些已经匹配好的括号对删除。

例如：

```text
s = "({[]})"

第一次删除：
"({[]})"
   ↓
"({})"

第二次删除：
"({})"
 ↓
"()"

第三次删除：
"()"
 ↓
""
```

最终字符串为空，因此原字符串有效。

再比如：

```text
s = "([)]"
```

字符串中不存在任何：

```text
()
{}
[]
```

因此无法继续删除，最终字符串仍然是：

```text
([)]
```

说明括号无法正确匹配。

这个方法本质上是不断消除最内层已经匹配完成的括号。

------

## 算法步骤

1. 只要字符串中仍然存在 `"()"`、`"{}"` 或 `"[]"`：
   - 将它们全部删除。
2. 当再也无法删除匹配括号时：
   - 如果字符串为空，返回 `True`。
   - 否则返回 `False`。

------

## Python 实现

```python
class Solution:
    def isValid(self, s: str) -> bool:
        # 只要还存在相邻且匹配的括号，就继续删除
        while '()' in s or '{}' in s or '[]' in s:
            s = s.replace('()', '')
            s = s.replace('{}', '')
            s = s.replace('[]', '')

        # 如果所有括号最终都能被消除，说明字符串有效
        return s == ''
```

------

## Python 函数说明：`str.replace()`

这里使用了：

```python
s.replace(old, new)
```

它会返回一个新的字符串，将 `s` 中所有的 `old` 替换为 `new`。

例如：

```python
s = "(())"
s = s.replace("()", "")

print(s)
```

结果：

```text
()
```

需要注意，Python 的字符串 `str` 是**不可变对象（immutable）**。

因此：

```python
s.replace("()", "")
```

不会直接修改 `s`，必须写成：

```python
s = s.replace("()", "")
```

------

## 时间复杂度

### 时间复杂度：`O(n²)`

一次：

```python
'()' in s
```

或者：

```python
s.replace(...)
```

通常都需要扫描整个字符串，因此一次操作可能需要 `O(n)`。

最坏情况下，每一轮可能只消除一小部分括号，而总共可能进行 `O(n)` 轮，因此：

```text
O(n) × O(n) = O(n²)
```

所以时间复杂度为：

```text
O(n²)
```

### 空间复杂度：`O(n)`

由于 Python 字符串不可变，`replace()` 会创建新的字符串，因此最坏情况下需要：

```text
O(n)
```

额外空间。

------

# 解法二：栈 Stack

这是本题最经典、也是面试中最推荐的解法。

## 核心思路

括号匹配具有非常明显的：

> **后进先出（Last In, First Out，LIFO）**

特征。

例如：

```text
([{}])
```

左括号出现顺序：

```text
(
[
{
```

但是关闭顺序是：

```text
}
]
)
```

也就是说：

```text
最后打开的括号，必须最先关闭。
```

这正好符合**栈（Stack）**的数据结构。

可以把它想象成叠盘子：

```text
push (
push [
push {
```

此时栈：

```text
顶部
{
[
(
底部
```

遇到：

```text
}
```

必须和栈顶 `{` 匹配，然后弹出：

```text
pop {
```

接下来 `]` 必须匹配 `[``，`)`必须匹配`(`。

最终栈为空，说明所有括号都成功匹配。

------

# 算法步骤

建立一个栈：

```python
stack = []
```

同时建立一个：

```text
右括号 -> 左括号
```

的映射：

```python
close_to_open = {
    ")": "(",
    "]": "[",
    "}": "{"
}
```

然后遍历字符串中的每一个字符 `c`。

### 情况一：`c` 是左括号

直接压入栈：

```python
stack.append(c)
```

例如：

```text
(
[
{
```

会依次进入栈。

### 情况二：`c` 是右括号

例如：

```text
)
]
}
```

需要检查两个条件：

```python
stack
```

表示栈不能为空。

以及：

```python
stack[-1] == close_to_open[c]
```

表示栈顶的左括号必须和当前右括号匹配。

如果匹配：

```python
stack.pop()
```

否则直接：

```python
return False
```

### 遍历结束

最后必须保证：

```python
stack
```

为空。

如果还有左括号留在栈中，说明它们没有被关闭。

------

# Python 实现

```python
class Solution:
    def isValid(self, s: str) -> bool:
        # 使用 list 模拟栈
        stack = []

        # 建立“右括号 -> 对应左括号”的映射
        close_to_open = {
            ")": "(",
            "]": "[",
            "}": "{"
        }

        for c in s:
            # 如果 c 是右括号
            if c in close_to_open:

                # 栈不能为空，并且栈顶必须是对应的左括号
                if stack and stack[-1] == close_to_open[c]:
                    stack.pop()
                else:
                    return False

            else:
                # 题目保证只有括号字符，
                # 因此不是右括号时就一定是左括号
                stack.append(c)

        # 所有括号都正确匹配时，栈最终必须为空
        return not stack
```

原答案中的：

```python
return True if not stack else False
```

可以简化成：

```python
return not stack
```

因为：

```python
not stack
```

本身就已经是布尔值。

------

# 代码执行示例

假设：

```text
s = "([{}])"
```

遍历过程如下：

| 当前字符 | 操作    | stack             |
| -------- | ------- | ----------------- |
| `(`      | push    | `['(']`           |
| `[`      | push    | `['(', '[']`      |
| `{`      | push    | `['(', '[', '{']` |
| `}`      | pop `{` | `['(', '[']`      |
| `]`      | pop `[` | `['(']`           |
| `)`      | pop `(` | `[]`              |

最终：

```python
stack == []
```

所以返回：

```python
True
```

------

# 为什么必须使用栈？

考虑：

```text
([)]
```

如果只统计每种括号的数量：

```text
( 和 ) 各有一个
[ 和 ] 各有一个
```

从数量上看完全匹配。

但嵌套顺序实际上是错误的。

处理到：

```text
([)
```

时，栈为：

```text
[
(
```

此时遇到了：

```text
)
```

它应该匹配：

```text
(
```

但当前栈顶却是：

```text
[
```

因此可以立即判断：

```python
False
```

所以这道题的关键并不是：

> 左右括号数量是否相同

而是：

> **最近打开的括号，是否被当前右括号正确关闭。**

这就是使用栈的根本原因。

------

# Python 中如何使用列表实现栈

Python 并没有要求必须使用专门的 `Stack` 类。

通常直接使用：

```python
list
```

即可实现栈。

## 入栈：`append()`

```python
stack.append(x)
```

例如：

```python
stack = []

stack.append("(")
stack.append("[")

print(stack)
```

得到：

```python
['(', '[']
```

------

## 查看栈顶：`stack[-1]`

```python
stack[-1]
```

表示列表的最后一个元素，也就是栈顶。

例如：

```python
stack = ["(", "[", "{"]

print(stack[-1])
```

结果：

```text
{
```

注意：

如果：

```python
stack = []
```

此时直接执行：

```python
stack[-1]
```

会产生：

```text
IndexError
```

所以必须先检查：

```python
if stack:
```

------

## 出栈：`pop()`

```python
stack.pop()
```

会删除并返回列表最后一个元素。

例如：

```python
stack = ["(", "[", "{"]

x = stack.pop()

print(x)
print(stack)
```

结果：

```text
{
['(', '[']
```

因此：

```python
append()
```

和：

```python
pop()
```

组合起来就可以实现标准的 LIFO 栈。

------

# Python 字典在本题中的作用

代码：

```python
close_to_open = {
    ")": "(",
    "]": "[",
    "}": "{"
}
```

使用的是 Python 的：

```python
dict
```

即哈希表 / 字典。

我们希望在遇到一个右括号时快速查出它对应的左括号。

例如：

```python
close_to_open[")"]
```

得到：

```text
(
```

而：

```python
close_to_open["]"]
```

得到：

```text
[
```

字典查找平均时间复杂度为：

```text
O(1)
```

因此不会影响整个算法的线性时间复杂度。

------

# 时间与空间复杂度

设字符串长度为 `n`。

## 时间复杂度：`O(n)`

字符串中的每个字符只会被遍历一次。

每个左括号：

```python
stack.append()
```

最多入栈一次。

每个右括号对应的：

```python
stack.pop()
```

最多执行一次。

而这些操作的平均时间复杂度都是：

```text
O(1)
```

因此总时间复杂度：

```text
O(n)
```

------

## 空间复杂度：`O(n)`

最坏情况下：

```text
(((((((((
```

所有字符都是左括号。

这些字符都会进入栈，因此栈最多保存 `n` 个元素。

所以空间复杂度：

```text
O(n)
```

------

# 常见错误

## 1. 没有先检查栈是否为空

错误写法：

```python
if stack[-1] == close_to_open[c]:
    stack.pop()
```

例如输入：

```text
")"
```

此时：

```python
stack = []
```

直接访问：

```python
stack[-1]
```

会抛出：

```text
IndexError
```

正确写法：

```python
if stack and stack[-1] == close_to_open[c]:
    stack.pop()
else:
    return False
```

这里利用了 Python 的**短路求值（short-circuit evaluation）**。

对于：

```python
A and B
```

如果 `A` 已经是 `False`，Python 就不会继续计算 `B`。

因此：

```python
stack and stack[-1] == close_to_open[c]
```

当 `stack` 为空时，不会执行：

```python
stack[-1]
```

从而避免错误。

------

# 2. 忘记最后检查栈是否为空

例如：

```text
s = "(()"
```

处理过程并不会遇到错误的右括号。

最终栈中却仍然有：

```text
(
```

因此只判断遍历过程中有没有错误是不够的。

最后必须：

```python
return not stack
```

------

# 3. 括号映射方向写反

推荐：

```python
close_to_open = {
    ")": "(",
    "]": "[",
    "}": "{"
}
```

即：

```text
右括号 -> 左括号
```

因为我们真正需要进行匹配检查的时候，是**遇到右括号时**。

于是可以直接：

```python
if c in close_to_open:
```

判断当前字符是否为右括号。

再通过：

```python
close_to_open[c]
```

找到它对应的左括号。

这种方向能够让代码非常自然。

------

# 4. 错误地使用队列 Queue

括号匹配需要：

```text
最后进入的左括号
↓
最先被匹配
```

即：

```text
LIFO
Last In, First Out
```

所以需要：

```text
Stack
```

而队列是：

```text
FIFO
First In, First Out
```

因此不适合解决这道题。

例如：

```text
([])
```

左括号进入顺序：

```text
(
[
```

但是第一个应该被关闭的是：

```text
[
```

而不是：

```text
(
```

因此必须检查**最近出现的左括号**。

------

# 5. 只检查括号数量

下面这种思路是不够的：

```python
s.count("(") == s.count(")")
```

因为：

```text
([)]
```

所有括号数量都正确，但嵌套顺序错误。

所以这道题不仅需要匹配：

```text
数量
```

还需要匹配：

```text
类型 + 顺序
```

栈正好能够同时解决这两个问题。

------

# 面试中推荐的最终版本

```python
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []

        # 右括号 -> 对应左括号
        close_to_open = {
            ")": "(",
            "]": "[",
            "}": "{"
        }

        for c in s:
            if c in close_to_open:
                # 右括号出现时：
                # 1. 栈不能为空
                # 2. 栈顶左括号必须和当前右括号匹配
                if not stack or stack[-1] != close_to_open[c]:
                    return False

                stack.pop()

            else:
                # 左括号入栈
                stack.append(c)

        # 如果仍有左括号没有被匹配，则无效
        return not stack
```

这个版本将失败条件直接写出来：

```python
if not stack or stack[-1] != close_to_open[c]:
    return False
```

通常在代码面试中比较容易解释。

------

# 面试总结

这道题最重要的识别点是：

```text
括号匹配
→ 存在嵌套关系
→ 最近打开的括号必须最先关闭
→ LIFO
→ Stack
```

可以记住一个非常常见的判断模式：

```python
for c in s:
    if c 是左括号:
        stack.append(c)
    else:
        if stack为空 or stack顶部不匹配:
            return False
        stack.pop()

return stack为空
```

对于括号匹配、HTML/XML 标签匹配、表达式解析等存在**嵌套结构**的问题，栈通常都是首先应该考虑的数据结构。
