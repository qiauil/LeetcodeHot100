# 数据流中的中位数（Find Median From Data Stream）

中位数（Median）是一个有序整数序列中位于中间位置的值。

- 如果序列长度为奇数，中位数就是正中间的元素。
- 如果序列长度为偶数，不存在唯一的中间元素，因此中位数定义为中间两个元素的平均值。

例如：

```text
[1, 2, 3]
中位数 = 2

[1, 2, 3, 4]
中位数 = (2 + 3) / 2 = 2.5
```

这道题的关键在于：数据会不断加入，而我们需要随时快速查询当前所有数据的中位数。

------

# 1. 排序法（Sorting）

## 思路

最简单直接的方法是：

1. 使用一个数组保存所有已经加入的数据。
2. 每次调用 `findMedian()` 时，对整个数组排序。
3. 根据数组长度是奇数还是偶数，返回中间元素或者中间两个元素的平均值。

排序之后，所有元素都按照从小到大的顺序排列，因此中位数的位置非常容易确定。

假设排序后的数组长度为 `n`：

- 如果 `n` 是奇数，中位数为：

```python
data[n // 2]
```

- 如果 `n` 是偶数，中位数为：

```python
(data[n // 2 - 1] + data[n // 2]) / 2
```

这种方法非常容易理解和实现，但缺点也很明显：**每次查询中位数都需要重新排序整个数组**。

------

## 算法步骤

### 初始化

创建一个空数组：

```python
data = []
```

用于保存所有加入的数据。

### `addNum(num)`

直接将新元素加入数组：

```python
data.append(num)
```

### `findMedian()`

1. 对数组进行排序。
2. 获取数组长度 `n`。
3. 如果 `n` 为奇数，返回正中间元素。
4. 如果 `n` 为偶数，返回中间两个元素的平均值。

------

## Python 实现

```python
class MedianFinder:

    def __init__(self):
        # 保存数据流中所有已经加入的数字
        self.data = []

    def addNum(self, num: int) -> None:
        # 直接将新数字加入数组
        self.data.append(num)

    def findMedian(self) -> float:
        # 每次查询中位数时重新排序
        self.data.sort()

        n = len(self.data)

        # 如果元素个数为奇数
        if n % 2 == 1:
            return self.data[n // 2]

        # 如果元素个数为偶数
        left = self.data[n // 2 - 1]
        right = self.data[n // 2]

        return (left + right) / 2.0
```

也可以将奇偶判断写得更紧凑：

```python
class MedianFinder:

    def __init__(self):
        self.data = []

    def addNum(self, num: int) -> None:
        self.data.append(num)

    def findMedian(self) -> float:
        self.data.sort()
        n = len(self.data)

        return (
            self.data[n // 2]
            if n & 1
            else (self.data[n // 2 - 1] + self.data[n // 2]) / 2.0
        )
```

------

## Python 语法补充：`n & 1`

代码：

```python
n & 1
```

使用的是**按位与运算（bitwise AND）**。

整数在二进制中：

- 偶数最后一位一定是 `0`
- 奇数最后一位一定是 `1`

因此：

```python
n & 1
```

可以用来判断 `n` 是否为奇数：

```python
if n & 1:
    # n 是奇数
```

它等价于：

```python
if n % 2 == 1:
```

在面试中，两种写法都可以。通常 `n % 2` 可读性更高。

------

## 时间与空间复杂度

假设当前已经有 `n` 个元素。

### `addNum`

直接向数组末尾添加元素：

```text
时间复杂度：O(1)
```

### `findMedian`

需要对 `n` 个元素排序：

```text
时间复杂度：O(n log n)
```

### 空间复杂度

需要保存所有数据：

```text
O(n)
```

Python 的 `list.sort()` 本身还可能使用额外的排序辅助空间，但在这类题目的标准复杂度分析中，主要的数据结构空间仍记为：

```text
O(n)
```

------

## 这种方法的问题

假设数据流已经包含：

```text
1,000,000
```

个数字。

如果我们频繁调用：

```python
findMedian()
```

那么每一次查询都需要重新排序百万级别的数据，非常浪费。

实际上，我们根本不需要知道完整的排序结果。

我们真正需要的信息只有：

> **中间位置附近的元素。**

这就引出了更加高效的“双堆（Two Heaps）”方法。

------

# 2. 双堆法（Two Heaps）

## 核心思路

为了在数据不断加入的情况下快速找到中位数，可以将所有数字分成两个部分：

```text
较小的一半        较大的一半
small             large

[ ... ]           [ ... ]
```

分别使用两个堆维护。

### `small`

保存**较小的一半数字**。

我们希望能够快速获得：

> 较小的一半里面最大的数字。

因此需要使用：

```text
最大堆（Max Heap）
```

### `large`

保存**较大的一半数字**。

我们希望能够快速获得：

> 较大的一半里面最小的数字。

因此需要使用：

```text
最小堆（Min Heap）
```

最终结构类似：

```text
small                     large
最大堆                    最小堆

较小的一半                 较大的一半

      3                    4
     / \                  / \
    1   2                6   5

small 最大值 = 3
large 最小值 = 4
```

如果总元素数量为偶数：

```text
median = (3 + 4) / 2
```

------

# 3. 双堆需要维护的两个不变量

双堆方法能够正确工作的关键，是始终维护下面两个条件。

## 条件一：两个堆的大小最多相差 1

必须满足：

```text
|len(small) - len(large)| <= 1
```

例如下面这些情况是合法的：

```text
small = 3 个
large = 3 个
```

或者：

```text
small = 4 个
large = 3 个
```

或者：

```text
small = 3 个
large = 4 个
```

但是：

```text
small = 5 个
large = 3 个
```

是不合法的，需要重新平衡。

------

## 条件二：`small` 中的所有数字 ≤ `large` 中的所有数字

也就是说：

```text
max(small) <= min(large)
```

由于：

```text
small 是最大堆
large 是最小堆
```

因此实际上只需要比较：

```text
small 的堆顶
large 的堆顶
```

只要始终维护这个性质：

```text
small_top <= large_top
```

两个堆就分别代表排好序后的左半部分和右半部分。

------

# 4. 为什么这样就能快速找到中位数？

假设共有奇数个元素：

```text
1 2 3 | 4 5
```

如果左边比右边多一个：

```text
small = [1, 2, 3]
large = [4, 5]
```

那么中位数就是：

```text
max(small) = 3
```

如果：

```text
1 2 | 3 4 5
```

右边多一个：

```text
small = [1, 2]
large = [3, 4, 5]
```

中位数就是：

```text
min(large) = 3
```

如果元素数量为偶数：

```text
1 2 | 3 4
```

那么：

```text
small = [1, 2]
large = [3, 4]
```

中位数为：

```text
(max(small) + min(large)) / 2
```

因此我们完全不需要对所有数据排序。

------

# 5. Python 中的 `heapq`

Python 标准库提供：

```python
import heapq
```

`heapq` 实现的是：

```text
最小堆（Min Heap）
```

例如：

```python
import heapq

heap = []

heapq.heappush(heap, 5)
heapq.heappush(heap, 2)
heapq.heappush(heap, 8)

print(heap[0])
```

输出：

```text
2
```

也就是说：

```python
heap[0]
```

始终是堆中的最小值。

------

## 常用函数

### `heapq.heappush`

向堆中插入一个元素：

```python
heapq.heappush(heap, num)
```

时间复杂度：

```text
O(log n)
```

------

### `heapq.heappop`

删除并返回堆顶最小元素：

```python
value = heapq.heappop(heap)
```

时间复杂度：

```text
O(log n)
```

------

### 查看堆顶

直接访问：

```python
heap[0]
```

时间复杂度：

```text
O(1)
```

注意：

```python
heap
```

内部并不是一个完整排序后的数组。

它只保证：

```text
heap[0]
```

一定是最小值。

------

# 6. Python 如何实现最大堆？

Python 的 `heapq` 默认只提供最小堆。

为了模拟最大堆，可以将所有数字取负。

例如原本：

```text
1, 3, 5
```

存入：

```text
-1, -3, -5
```

最小的负数：

```text
-5
```

对应原来的最大数：

```text
5
```

因此：

```python
heapq.heappush(small, -num)
```

表示将 `num` 放入最大堆。

获取最大值时：

```python
-small[0]
```

删除最大值时：

```python
value = -heapq.heappop(small)
```

这是 Python 堆相关题目中非常常见的技巧。

------

# 7. 双堆算法

## 初始化

创建两个堆：

```python
small = []
large = []
```

其中：

```text
small：较小的一半，模拟最大堆
large：较大的一半，普通最小堆
```

------

## `addNum(num)`

### 第一步：决定放入哪个堆

如果：

```python
large
```

不为空，并且：

```python
num > large[0]
```

说明当前数字应该属于较大的那一半，因此放入：

```python
large
```

否则放入：

```python
small
```

注意，因为 `small` 是通过负数模拟的最大堆，所以需要：

```python
heapq.heappush(self.small, -num)
```

------

### 第二步：重新平衡

如果：

```text
small 比 large 多至少 2 个元素
```

那么将 `small` 最大的元素移动到 `large`：

```python
value = -heapq.heappop(self.small)
heapq.heappush(self.large, value)
```

反过来，如果：

```text
large 比 small 多至少 2 个元素
```

将 `large` 最小的元素移动到 `small`。

这样可以确保：

```text
|len(small) - len(large)| <= 1
```

------

## `findMedian()`

有三种情况。

### `small` 更多

```python
len(small) > len(large)
```

中位数就是：

```python
-small[0]
```

### `large` 更多

```python
len(large) > len(small)
```

中位数就是：

```python
large[0]
```

### 两个堆一样大

取两个堆顶的平均值：

```python
(-small[0] + large[0]) / 2.0
```

------

# 8. Python 实现

```python
import heapq


class MedianFinder:

    def __init__(self):
        # small 保存较小的一半数字。
        #
        # Python heapq 只有最小堆，因此通过存储负数模拟最大堆。
        # 例如数字 5 会保存为 -5。
        #
        # -small[0] 就是较小一半中的最大值。
        self.small = []

        # large 保存较大的一半数字。
        #
        # 使用普通最小堆。
        # large[0] 就是较大一半中的最小值。
        self.large = []

    def addNum(self, num: int) -> None:

        # -------------------------
        # 1. 将 num 放入正确的堆
        # -------------------------

        # 如果 large 不为空，并且 num 比 large 中最小的数字还大，
        # 那么 num 属于较大的一半。
        if self.large and num > self.large[0]:
            heapq.heappush(self.large, num)

        else:
            # 否则放入较小的一半。
            # small 是最大堆，因此存储 -num。
            heapq.heappush(self.small, -num)

        # -------------------------
        # 2. 平衡两个堆
        # -------------------------

        # small 比 large 多超过一个元素：
        #
        # 将 small 中最大的元素移动到 large。
        if len(self.small) > len(self.large) + 1:

            # small 中存储的是负数，
            # 所以取出后重新变成正数。
            value = -heapq.heappop(self.small)

            heapq.heappush(self.large, value)

        # large 比 small 多超过一个元素：
        #
        # 将 large 中最小的元素移动到 small。
        elif len(self.large) > len(self.small) + 1:

            value = heapq.heappop(self.large)

            # small 是最大堆，因此存负数。
            heapq.heappush(self.small, -value)

    def findMedian(self) -> float:

        # small 元素更多：
        # 中位数就是 small 的最大值。
        if len(self.small) > len(self.large):
            return -self.small[0]

        # large 元素更多：
        # 中位数就是 large 的最小值。
        if len(self.large) > len(self.small):
            return self.large[0]

        # 两个堆大小相同：
        # 中位数为两个堆顶元素的平均值。
        return (-self.small[0] + self.large[0]) / 2.0
```

------

# 9. 插入过程示例

假设依次插入：

```text
1, 2, 3, 4
```

## 插入 1

`large` 为空，因此放入 `small`：

```text
small = [1]
large = []
```

中位数：

```text
1
```

------

## 插入 2

因为：

```text
large 为空
```

按照当前代码，先将 `2` 放入 `small`：

```text
small = [1, 2]
large = []
```

此时：

```text
len(small) = 2
len(large) = 0
```

大小相差超过 1，因此需要重新平衡。

从 `small` 中取出最大值：

```text
2
```

移动到 `large`：

```text
small = [1]
large = [2]
```

中位数：

```text
(1 + 2) / 2 = 1.5
```

------

## 插入 3

因为：

```text
3 > large[0] = 2
```

所以加入 `large`：

```text
small = [1]
large = [2, 3]
```

中位数：

```text
2
```

------

## 插入 4

加入 `large`：

```text
small = [1]
large = [2, 3, 4]
```

两个堆大小相差超过 1。

从 `large` 取出最小值：

```text
2
```

放入 `small`：

```text
small = [1, 2]
large = [3, 4]
```

中位数：

```text
(2 + 3) / 2 = 2.5
```

------

# 10. 为什么插入后重新平衡不会破坏顺序？

这是理解双堆方法非常重要的一点。

我们维护：

```text
small 中所有数字 <= large 中所有数字
```

如果 `small` 太大，我们移动：

```text
small 中最大的元素
```

到 `large`。

这个元素本来就是 `small` 中最大的，因此移动后不会使剩余的 `small` 出现过大的元素。

类似地，如果 `large` 太大，我们移动：

```text
large 中最小的元素
```

到 `small`。

这个元素本来就是 `large` 中最小的，因此它是最适合移动到左边的元素。

所以：

> **重新平衡时一定移动“靠近中间”的元素。**

这也是双堆方法最核心的思想之一。

------

# 11. 时间与空间复杂度

假设当前数据流中有 `n` 个元素。

## `addNum`

插入堆：

```text
O(log n)
```

重新平衡时，最多进行一次：

```text
heappop + heappush
```

每个操作都是：

```text
O(log n)
```

因此：

```text
addNum 时间复杂度：O(log n)
```

------

## `findMedian`

只访问：

```python
small[0]
```

以及：

```python
large[0]
```

不需要修改堆。

因此：

```text
findMedian 时间复杂度：O(1)
```

------

## 空间复杂度

所有数字最终都会存储在两个堆中：

```text
O(n)
```

------

## 如果函数被调用多次

假设：

- 一共有 `n` 个数字被插入。
- `addNum()` 总共调用 `m₁` 次。
- `findMedian()` 总共调用 `m₂` 次。

那么双堆方案总体大约为：

```text
O(m₁ log n + m₂)
```

而在通常的 LeetCode / 面试复杂度分析中，更常见的写法是直接写单次操作复杂度：

```text
addNum:      O(log n)
findMedian:  O(1)
Space:       O(n)
```

------

# 12. 排序法与双堆法对比

| 方法           | `addNum` | `findMedian` | 空间复杂度 |
| -------------- | -------- | ------------ | ---------- |
| 每次查询时排序 | O(1)     | O(n log n)   | O(n)       |
| 双堆           | O(log n) | O(1)         | O(n)       |

如果题目强调：

```text
数据不断到来
并且需要不断查询中位数
```

那么双堆通常就是标准答案。

------

# 13. 常见错误

## 错误一：两个堆的类型使用错误

正确结构应该是：

```text
small → 最大堆
large → 最小堆
```

因为我们需要：

```text
small 最大值
large 最小值
```

这两个值正好位于整个有序序列的中间。

如果使用两个最小堆，就无法在 O(1) 时间获得较小一半中的最大值。

在 Python 中，由于 `heapq` 默认只有最小堆，因此通常通过：

```python
-num
```

模拟最大堆。

------

## 错误二：忘记平衡两个堆

必须始终保证：

```text
|len(small) - len(large)| <= 1
```

例如：

```text
small = 5 个
large = 2 个
```

此时即使两个堆内部的数据划分正确，也不能直接通过堆顶确定中位数。

因此每次：

```python
addNum()
```

之后都要检查两个堆的大小。

------

## 错误三：偶数个元素时只返回一个堆顶

例如：

```text
1 2 | 3 4
```

真正的中位数是：

```text
(2 + 3) / 2 = 2.5
```

而不是：

```text
2
```

也不是：

```text
3
```

因此两个堆大小相同时必须返回：

```python
(-self.small[0] + self.large[0]) / 2.0
```

------

## 错误四：使用整数除法

在某些语言中：

```text
(2 + 3) / 2
```

如果两边都是整数，有可能得到：

```text
2
```

而不是：

```text
2.5
```

Python 3 中：

```python
/
```

本身就是浮点除法，因此没有这个问题。

但如果使用：

```python
//
```

则会进行向下取整：

```python
5 // 2
# 2
```

所以计算中位数时不要使用：

```python
//
```

------

## 错误五：整数溢出

在 Python 中普通整数可以自动扩展，因此通常不需要担心整数溢出。

但是在 Java、C++ 等固定宽度整数类型的语言中，如果两个数字非常大：

```text
a + b
```

可能发生整数溢出。

例如：

```text
a = INT_MAX
b = INT_MAX
```

直接：

```cpp
(a + b) / 2.0
```

可能在执行加法时就已经溢出。

可以先转换为更大的类型：

```cpp
((long long)a + b) / 2.0
```

或者使用类似：

```text
a + (b - a) / 2.0
```

的形式降低溢出的风险。

------

## 错误六：插入元素时破坏两个堆的顺序关系

除了大小平衡之外，还必须维护：

```text
max(small) <= min(large)
```

也就是：

```text
small 中所有数字 <= large 中所有数字
```

不能简单地随便选择一个堆插入元素，然后只调整两个堆的大小。

因为即使大小正确：

```text
small = [1, 100]
large = [2, 3]
```

仍然是错误的划分。

`100` 应该位于 `large`，而不是 `small`。

------

# 14. 一种更常见的双堆写法

还有一种非常常见、而且在面试中比较容易证明正确性的写法：

> 始终让 `small` 的元素数量等于 `large`，或者比 `large` 多一个。

也就是说始终维护：

```text
len(small) == len(large)
```

或者：

```text
len(small) == len(large) + 1
```

这样一来：

- 奇数个元素时，中位数永远是 `small` 堆顶。
- 偶数个元素时，中位数是两个堆顶的平均值。

实现如下：

```python
import heapq


class MedianFinder:

    def __init__(self):
        # small：较小的一半，最大堆
        self.small = []

        # large：较大的一半，最小堆
        self.large = []

    def addNum(self, num: int) -> None:
        # 第一步：
        # 先将 num 放进 small。
        #
        # Python 没有原生最大堆，所以存储负数。
        heapq.heappush(self.small, -num)

        # 第二步：
        # 将 small 中最大的数字移动到 large。
        #
        # 这样可以保证：
        # max(small) <= min(large)
        largest_in_small = -heapq.heappop(self.small)
        heapq.heappush(self.large, largest_in_small)

        # 第三步：
        # 始终保证 small 的元素数量 >= large。
        #
        # 如果 large 比 small 更多，
        # 就把 large 中最小的元素移回 small。
        if len(self.large) > len(self.small):
            smallest_in_large = heapq.heappop(self.large)
            heapq.heappush(self.small, -smallest_in_large)

    def findMedian(self) -> float:
        # 奇数个元素：
        #
        # small 一定比 large 多一个元素。
        if len(self.small) > len(self.large):
            return -self.small[0]

        # 偶数个元素：
        #
        # 两个堆大小相同。
        return (-self.small[0] + self.large[0]) / 2.0
```

这个版本的复杂度仍然是：

```text
addNum:      O(log n)
findMedian:  O(1)
Space:       O(n)
```

它的优势是中位数判断稍微更加统一：

```text
奇数 → small 堆顶
偶数 → 两个堆顶平均值
```

------

# 15. 面试时如何解释双堆思路

可以按照下面这个逻辑快速向面试官解释：

```text
我并不需要维护所有元素的完整排序，
因为中位数只和序列中间位置有关。

所以我把所有数字分成两半：

small 保存较小的一半，
large 保存较大的一半。

small 使用最大堆，因此可以 O(1) 得到左半部分最大值；
large 使用最小堆，因此可以 O(1) 得到右半部分最小值。

我维护两个 invariant：

1. small 中所有元素 <= large 中所有元素；
2. 两个堆的大小最多相差 1。

这样，如果总元素个数是奇数，
中位数就是较大堆的堆顶；

如果是偶数，
中位数就是两个堆顶的平均值。

插入一个数字只需要堆操作，因此是 O(log n)，
查询中位数只访问堆顶，因此是 O(1)。
```

------

# 16. 核心总结

这道题最重要的不是记住代码，而是理解为什么需要两个堆。

完整排序：

```text
1 2 3 4 5 6 7 8
```

中位数其实只关心：

```text
        ↓ ↓
1 2 3 [4 5] 6 7 8
```

所以没有必要维护：

```text
所有元素之间的完整顺序
```

只需要维护：

```text
左边最大的数字
右边最小的数字
```

这正是：

```text
最大堆 + 最小堆
```

最擅长解决的问题。

可以把双堆法记成下面这张结构图：

```text
              Median
                 ↓

       small            large
      最大堆             最小堆

   较小的一半           较大的一半

        ↓                  ↓

   max(small)         min(large)

        └────── 中间两个候选值 ──────┘
```

因此最值得记住的三个结论是：

```text
1. small = 较小的一半 + 最大堆
2. large = 较大的一半 + 最小堆
3. 两个堆大小最多相差 1
```

以及复杂度：

```text
addNum      O(log n)
findMedian  O(1)
Space       O(n)
```
