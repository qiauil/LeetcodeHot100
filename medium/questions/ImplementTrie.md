# 实现 Trie（前缀树）

## 题目描述

**前缀树（Prefix Tree）**，也称为 **Trie（字典树）**，是一种树形数据结构，用于高效地存储和查询一组字符串。

Trie 特别适合处理与**字符串前缀**有关的问题，常见应用包括：

- 搜索框的自动补全（Autocomplete）
- 拼写检查（Spell Checker）
- 字典中的单词查询
- 前缀匹配
- IP 路由中的最长前缀匹配等

实现 `PrefixTree` 类：

- `PrefixTree()`：初始化前缀树。
- `insert(word)`：将字符串 `word` 插入前缀树。
- `search(word)`：如果 `word` **完整存在**于前缀树中，即之前被插入过，则返回 `True`；否则返回 `False`。
- `startsWith(prefix)`：如果之前插入过的某个字符串以 `prefix` 为前缀，则返回 `True`；否则返回 `False`。

------

## 核心思路

Trie 的核心思想是：

> **具有相同前缀的字符串，共享相同的路径。**

例如依次插入：

```
apple
app
apt
```

Trie 的结构可以理解为：

```
root
 └── a
      └── p
           ├── p
           │    └── l
           │         └── e
           └── t
```

其中：

- `app` 和 `apple` 共享前缀 `"app"`
- `app`、`apple` 和 `apt` 共享前缀 `"ap"`

不过，仅仅保存字符路径还不够。

例如插入：

```
apple
```

之后：

```
search("app")
```

应该返回 `False`，因为虽然 `"app"` 是 `"apple"` 的前缀，但 `"app"` 本身并没有被插入。

因此，每个 Trie 节点还需要一个额外的标记，例如：

```
end = True
```

表示：

> **是否有一个完整的单词在当前节点结束。**

------

# 数据结构设计

首先定义 Trie 中的节点：

```
class PrefixTreeNode:
    def __init__(self):
        # children[i] 表示当前节点是否存在对应的下一个字符
        # 0 -> 'a'
        # 1 -> 'b'
        # ...
        # 25 -> 'z'
        self.children = [None] * 26

        # 标记是否有一个完整单词在当前节点结束
        self.end = False
```

这里假设输入字符串只包含小写英文字母 `a-z`。

每个节点维护一个长度为 `26` 的数组：

```
self.children = [None] * 26
```

例如：

```
children[0]  -> 'a'
children[1]  -> 'b'
children[2]  -> 'c'
...
children[25] -> 'z'
```

如果：

```
children[2] is not None
```

说明当前节点之后存在字符 `'c'`。

------

## Python 中的 `ord()` 函数

代码中有一个比较重要的 Python 内置函数：

```
ord(c)
```

`ord()` 用于返回一个字符对应的 Unicode 编码值。

例如：

```
ord("a")  # 97
ord("b")  # 98
ord("c")  # 99
```

因此：

```
ord(c) - ord("a")
```

可以把 `'a' ~ 'z'` 映射到数组下标 `0 ~ 25`：

```
'a' -> 97 - 97 = 0
'b' -> 98 - 97 = 1
'c' -> 99 - 97 = 2
...
'z' -> 122 - 97 = 25
```

所以我们可以通过：

```
i = ord(c) - ord("a")
```

快速找到字符 `c` 对应的子节点。

------

# 完整代码

```
class PrefixTreeNode:
    def __init__(self):
        # children[i] 保存字符 chr(ord('a') + i) 对应的子节点
        # 题目假设字符串只包含 a-z，因此固定使用长度为 26 的数组
        self.children = [None] * 26

        # 当前节点是否代表某个完整单词的结尾
        self.end = False


class PrefixTree:
    def __init__(self):
        # Trie 使用一个不代表任何字符的虚拟根节点
        self.root = PrefixTreeNode()

    def insert(self, word: str) -> None:
        curr = self.root

        # 从左到右处理 word 中的每个字符
        for c in word:
            # 将 'a' ~ 'z' 转换为 0 ~ 25
            i = ord(c) - ord("a")

            # 如果对应的子节点不存在，就创建它
            if curr.children[i] is None:
                curr.children[i] = PrefixTreeNode()

            # 移动到下一层
            curr = curr.children[i]

        # 整个 word 遍历结束后，
        # 当前节点就是这个完整单词的结尾
        curr.end = True

    def search(self, word: str) -> bool:
        curr = self.root

        for c in word:
            i = ord(c) - ord("a")

            # 如果某个字符对应的路径不存在，
            # 那么 word 一定没有被插入
            if curr.children[i] is None:
                return False

            curr = curr.children[i]

        # 路径存在并不意味着完整单词存在
        # 必须检查当前节点是不是某个单词的结尾
        return curr.end

    def startsWith(self, prefix: str) -> bool:
        curr = self.root

        for c in prefix:
            i = ord(c) - ord("a")

            # 前缀中的某个字符不存在，则此前缀不存在
            if curr.children[i] is None:
                return False

            curr = curr.children[i]

        # 只要整个 prefix 的路径存在即可，
        # 不要求当前节点是完整单词的结尾
        return True
```

# `insert()` 详解

假设：

```
trie.insert("cat")
```

最开始：

```
curr = root
```

处理 `'c'`：

```
i = ord("c") - ord("a")   # 2
```

检查：

```
curr.children[2]
```

如果不存在，就创建：

```
curr.children[2] = PrefixTreeNode()
```

然后移动：

```
curr = curr.children[2]
```

接下来用同样的方法处理 `'a'` 和 `'t'`。

最终得到：

```
root -> c -> a -> t
```

处理完 `"cat"` 后：

```
curr.end = True
```

因此：

```
root
 └── c
      └── a
           └── t (end=True)
```

如果之后再插入：

```
trie.insert("car")
```

那么 `"ca"` 的节点可以直接复用：

```
root
 └── c
      └── a
           ├── t (end=True)
           └── r (end=True)
```

这就是 Trie 能够高效存储大量具有公共前缀字符串的重要原因。

------

# `search()` 详解

假设 Trie 中已经插入：

```
trie.insert("apple")
```

此时：

```
trie.search("apple")
```

会依次寻找：

```
a -> p -> p -> l -> e
```

所有节点都存在，并且最后：

```
e.end == True
```

因此返回：

```
True
```

但是：

```
trie.search("app")
```

虽然：

```
a -> p -> p
```

这条路径存在，但如果 `"app"` 没有单独被插入：

```
p.end == False
```

所以返回：

```
False
```

这也是 Trie 中 `end` 标记存在的关键原因。

------

# `startsWith()` 详解

`startsWith()` 与 `search()` 非常相似。

最大的区别是：

```
search()
```

要求：

```
路径存在 + 最后节点 end == True
```

而：

```
startsWith()
```

只要求：

```
路径存在
```

例如已经插入：

```
trie.insert("apple")
```

那么：

```
trie.startsWith("app")   # True
trie.startsWith("ap")    # True
trie.startsWith("apple") # True
trie.startsWith("apt")   # False
```

因此可以把两个操作总结成：

| 操作                 | 路径必须存在 | 最后节点 `end=True` |
| -------------------- | ------------ | ------------------- |
| `search(word)`       | 是           | 是                  |
| `startsWith(prefix)` | 是           | 否                  |

这是这道题最重要的区别之一。

------

# 复杂度分析

设字符串长度为 `L`。

### `insert(word)`

需要遍历整个字符串：

```
时间复杂度：O(L)
```

### `search(word)`

最多遍历整个字符串：

```
时间复杂度：O(L)
```

### `startsWith(prefix)`

最多遍历整个前缀：

```
时间复杂度：O(L)
```

如果总共插入的所有字符串字符数为 `N`，那么 Trie 最坏情况下需要创建 `O(N)` 个节点。

由于当前实现中每个节点都有：

```
[None] * 26
```

所以每个节点固定维护 26 个子节点位置。

严格来说空间可以写成：

```
O(26N) = O(N)
```

但这个实现的常数空间开销相对较大。

------

# 为什么使用 Trie，而不是 `set`？

如果需求仅仅是：

```
search("apple")
```

Python 的：

```
set()
```

其实通常已经非常高效。

例如：

```
words = set()

words.add("apple")

"apple" in words
```

但是 Trie 的优势在于**前缀查询**。

如果要判断：

```
startsWith("app")
```

Trie 只需要沿着：

```
a -> p -> p
```

走三步即可，时间复杂度与前缀长度有关。

更重要的是，在实际应用中找到 `"app"` 对应的 Trie 节点之后，还可以继续向下搜索：

```
app
├── apple
├── application
├── apply
└── appreciate
```

因此 Trie 非常适合实现**自动补全、前缀搜索和词典系统**。

------

# 另一种 Python 实现：使用字典

Python 中还可以使用 `dict` 保存子节点：

```
class TrieNode:
    def __init__(self):
        self.children = {}
        self.end = False


class PrefixTree:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        curr = self.root

        for c in word:
            # 如果字符 c 对应的节点不存在，则创建
            if c not in curr.children:
                curr.children[c] = TrieNode()

            curr = curr.children[c]

        curr.end = True

    def search(self, word: str) -> bool:
        curr = self.root

        for c in word:
            if c not in curr.children:
                return False

            curr = curr.children[c]

        return curr.end

    def startsWith(self, prefix: str) -> bool:
        curr = self.root

        for c in prefix:
            if c not in curr.children:
                return False

            curr = curr.children[c]

        return True
```

这种写法的一个优点是，不需要：

```
ord(c) - ord("a")
```

而是直接：

```
curr.children[c]
```

同时它只保存**实际存在的子节点**，对于字符分支比较稀疏的 Trie，通常更加节省空间，而且更容易扩展到大写字母、数字甚至其他字符。

不过，在面试中如果题目明确说明只有 `a-z`，使用长度为 `26` 的数组是非常经典的实现方式，也更容易体现 Trie 的底层结构。

------

# 面试要点总结

这道题最值得记住的是 Trie 的三个核心设计：

**1. 每个节点代表一个字符路径**

```
root -> c -> a -> t
```

表示 `"cat"`。

**2. 公共前缀共享节点**

```
car
cat
```

共享：

```
c -> a
```

因此结构为：

```
root
 └── c
      └── a
           ├── r
           └── t
```

**3. 必须使用 `end` 区分“完整单词”和“前缀”**

这是本题最容易被面试官追问的地方。

假设只插入：

```
insert("apple")
```

那么：

```
search("app")       # False
startsWith("app")   # True
```

因为 `"app"` 对应的**路径确实存在**，但 `"app"` 并不是之前插入的**完整单词**。

可以把 Trie 的查询逻辑浓缩成一句话：

> **`search()` = 找到路径 + 检查终止标记；`startsWith()` = 只需要找到路径。**