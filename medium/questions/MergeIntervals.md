# 合并区间（Merge Intervals）

给定一个区间数组 `intervals`，其中 `intervals[i] = [start_i, end_i]`。

请合并所有互相重叠的区间，并返回合并后的、彼此不重叠的区间数组。返回结果的顺序可以任意。

> 两个区间如果存在至少一个公共点，就视为重叠。  
> 例如：
>
> - `[1, 2]` 和 `[3, 4]` 不重叠；
> - `[1, 2]` 和 `[2, 3]` 重叠，因为它们在点 `2` 相交。

## 示例 1

```text
输入：intervals = [[1,3],[1,5],[6,7]]

输出：[[1,5],[6,7]]
```

解释：

- `[1,3]` 与 `[1,5]` 重叠，因此可以合并为 `[1,5]`；
- `[6,7]` 与 `[1,5]` 不重叠，因此单独保留。

## 示例 2

```text
输入：intervals = [[1,2],[2,3]]

输出：[[1,3]]
```

虽然第一个区间在 `2` 结束，第二个区间在 `2` 开始，但二者共享点 `2`，因此属于重叠区间。

## 约束条件

- `1 <= intervals.length <= 1000`
- `intervals[i].length == 2`
- `0 <= start <= end <= 1000`

---

# 解法一：排序 + 贪心合并

这是这道题最经典、最推荐掌握的解法。

## 核心思路

如果区间是无序的，那么当前区间可能与前面很多区间发生重叠，处理起来比较麻烦。

但如果我们先按照区间的起点从小到大排序：

```text
[1, 4], [2, 5], [3, 7], [9, 10]
```

那么当我们处理当前区间时，只需要判断它是否与结果数组中**最后一个已经合并好的区间**重叠。

原因是：

- 前面的区间起点都不会比当前区间更大；
- 之前能够合并的区间已经全部合并到了 `output[-1]`；
- 因此当前区间是否需要继续合并，只取决于它是否与最后一个合并区间相交。

这是一种典型的贪心思想：

> 每次都尽可能扩展当前正在维护的合并区间；  
> 只有确定当前区间与它不再重叠时，才开启一个新的区间。

## 判断是否重叠

假设：

```text
最后一个合并区间：[last_start, last_end]
当前区间：[start, end]
```

如果：

```python
start <= last_end
```

说明两个区间重叠。

注意这里必须使用 `<=`，而不是 `<`。

例如：

```text
[1, 3]
[3, 5]
```

它们共享点 `3`，所以应当合并成：

```text
[1, 5]
```

## 合并方法

重叠后，新区间的结束位置应该是：

```python
max(last_end, end)
```

而不能直接赋值为 `end`。

例如：

```text
[1, 10]
[2, 5]
```

第二个区间虽然起点更靠后，但它完全被第一个区间包含。

合并结果仍然应该是：

```text
[1, 10]
```

## 算法步骤

1. 按照每个区间的起点进行升序排序。

2. 将第一个区间放入结果数组 `output`。

3. 依次遍历排序后的区间。

4. 取出结果数组中最后一个区间的结束位置 `last_end`。

5. 如果：

   ```python
   start <= last_end
   ```

   说明当前区间与最后一个区间重叠，将结束位置更新为：

   ```python
   max(last_end, end)
   ```

6. 否则，将当前区间作为一个新的独立区间加入结果数组。

7. 遍历结束后返回结果。

## Python 实现

```python
from typing import List


class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        # 按照区间起点从小到大排序
        intervals.sort(key=lambda pair: pair[0])

        # 至少存在一个区间，因此可以直接放入第一个区间
        output = [intervals[0]]

        # 依次处理每个区间
        for start, end in intervals:
            # 当前结果中最后一个合并区间的结束位置
            last_end = output[-1][1]

            if start <= last_end:
                # 当前区间与最后一个区间重叠
                # 扩展最后一个区间的结束位置
                output[-1][1] = max(last_end, end)
            else:
                # 没有重叠，需要开启一个新的合并区间
                output.append([start, end])

        return output
```

## 代码细节说明

### `list.sort()`

```python
intervals.sort(...)
```

`list.sort()` 会直接修改原列表，而不是创建一个新列表。

这里使用：

```python
key=lambda pair: pair[0]
```

表示按照每个区间的第一个元素，也就是起点排序。

例如：

```python
intervals = [[6, 8], [1, 3], [2, 5]]

intervals.sort(key=lambda pair: pair[0])
```

排序后：

```python
[[1, 3], [2, 5], [6, 8]]
```

如果不希望修改原数组，也可以使用：

```python
intervals = sorted(intervals, key=lambda pair: pair[0])
```

### `lambda`

这里的：

```python
lambda pair: pair[0]
```

等价于：

```python
def get_start(pair):
    return pair[0]
```

`lambda` 适合表示这种非常简单、只使用一次的小函数。

### `output[-1]`

Python 中：

```python
output[-1]
```

表示列表中的最后一个元素。

因此：

```python
output[-1][1]
```

表示：

> 结果数组中最后一个区间的结束位置。

## 复杂度分析

设区间数量为 `n`。

### 时间复杂度

排序需要：

```text
O(n log n)
```

之后线性遍历所有区间需要：

```text
O(n)
```

因此总时间复杂度为：

```text
O(n log n)
```

### 空间复杂度

如果暂时不计算返回结果所占空间：

- Python 排序内部通常会使用额外空间；
- 具体辅助空间取决于排序实现。

从算法分析角度，经常记作：

```text
O(n)
```

另外，结果数组在最坏情况下需要保存所有区间，因此输出空间最多为：

```text
O(n)
```

## 为什么这是首选解法

这个方法通常是面试中最值得优先给出的方案，因为它：

- 思路直观；
- 代码短；
- 时间复杂度优秀；
- 不依赖题目中数值范围较小这一特殊条件；
- 很容易推广到其他区间类问题。

---

# 解法二：扫描线（Sweep Line）

## 核心思路

扫描线算法不直接考虑“两个区间怎么合并”，而是把每个区间转换成数轴上的两个事件：

对于：

```text
[start, end]
```

可以理解为：

- 在 `start` 处，一个区间开始；
- 在 `end` 处，一个区间结束。

我们从左到右扫描所有边界，同时维护：

```python
have
```

表示当前有多少个区间处于“打开”状态。

当：

```text
have: 0 -> 正数
```

说明我们进入了一段被区间覆盖的区域，一个新的合并区间开始。

当：

```text
have: 正数 -> 0
```

说明当前所有已经开始的区间都结束了，因此一个合并区间结束。

## 一个例子

假设：

```text
[1, 4]
[2, 5]
```

对应事件：

```text
位置 1：+1
位置 2：+1
位置 4：-1
位置 5：-1
```

扫描过程：

```text
位置 1：have = 1
位置 2：have = 2
位置 4：have = 1
位置 5：have = 0
```

因此整个 `[1, 5]` 中始终至少有一个区间处于活动状态，最终得到：

```text
[1, 5]
```

## 为什么同一点开始和结束不会出错

例如：

```text
[1, 2]
[2, 3]
```

在位置 `2`：

- 第一个区间结束：`-1`
- 第二个区间开始：`+1`

累计变化为：

```text
0
```

因此活动区间数量不会在 `2` 处降到 `0`，两个区间自然会被合并。

这正好符合题目中：

```text
[1,2] 和 [2,3] 属于重叠区间
```

的定义。

## 算法步骤

1. 创建哈希表 `mp`，记录每个边界位置对活动区间数量的影响。

2. 对于每个区间 `[start, end]`：

   ```python
   mp[start] += 1
   mp[end] -= 1
   ```

3. 对所有出现过的边界位置进行排序。

4. 使用 `have` 记录当前活动区间数。

5. 如果当前没有正在构造的合并区间，则当前位置作为新的起点。

6. 更新：

   ```python
   have += mp[i]
   ```

7. 当 `have == 0` 时，说明当前覆盖区域结束：

   - 将当前位置作为合并区间终点；
   - 保存结果；
   - 开始寻找下一个合并区间。

## Python 实现

```python
from collections import defaultdict
from typing import List


class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        # mp[x] 表示扫描到位置 x 时，
        # 当前活动区间数量需要发生多少变化
        mp = defaultdict(int)

        for start, end in intervals:
            # 一个区间从 start 开始
            mp[start] += 1

            # 一个区间在 end 结束
            mp[end] -= 1

        res = []
        interval = []
        have = 0

        # 从左到右扫描所有边界
        for position in sorted(mp):
            # 如果当前还没有正在构造的区间，
            # 那么该位置就是新合并区间的起点
            if not interval:
                interval.append(position)

            # 更新当前活动区间数量
            have += mp[position]

            # 所有活动区间都已经结束
            if have == 0:
                interval.append(position)
                res.append(interval)

                # 重置，准备构造下一个区间
                interval = []

        return res
```

## Python 类说明：`defaultdict`

代码中使用了：

```python
from collections import defaultdict

mp = defaultdict(int)
```

普通字典在访问不存在的键时：

```python
mp[x]
```

会抛出：

```text
KeyError
```

而：

```python
defaultdict(int)
```

会在键不存在时自动创建默认值。

因为：

```python
int()
```

的默认结果是：

```python
0
```

所以可以直接写：

```python
mp[start] += 1
mp[end] -= 1
```

而不需要提前判断键是否存在。

普通字典版本通常需要写成：

```python
mp[start] = mp.get(start, 0) + 1
mp[end] = mp.get(end, 0) - 1
```

## 复杂度分析

假设总共有 `n` 个区间。

每个区间最多贡献两个边界，所以不同边界数量最多为 `2n`。

排序这些边界需要：

```text
O(n log n)
```

扫描需要：

```text
O(n)
```

因此总时间复杂度：

```text
O(n log n)
```

空间复杂度：

```text
O(n)
```

## 对扫描线方法的理解

扫描线是一类非常重要的思想。

它尤其适合处理：

- 区间覆盖；
- 同时在线人数；
- 同时进行的会议数量；
- 日程冲突；
- 飞机数量统计；
- 矩形面积计算；
- 区间重叠计数。

这道题中，扫描线虽然没有比排序贪心更简单，但非常适合作为扩展思路学习。

---

# 解法三：利用数值范围的数组扫描

这一解法利用了题目中的特殊约束：

```text
0 <= start <= end <= 1000
```

因为区间坐标很小，所以可以使用数组来记录信息，而不一定非要对所有区间进行比较。

这种方法本质上也是一种贪心扫描。

## 核心思路

对于每个可能的起点 `start`，只需要记录：

> 所有从 `start` 开始的区间中，最远能够延伸到哪里。

例如：

```text
[1, 3]
[1, 5]
[1, 4]
```

我们只需要记录：

```text
start = 1 -> farthest end = 5
```

因为其他两个区间已经完全被 `[1,5]` 包含。

然后从左到右扫描所有可能的起点。

维护：

```python
have
```

表示当前正在构造的合并区间最远能够到达的位置。

如果扫描到某个位置 `i`，发现有新区间从这里开始，就更新：

```python
have = max(have, 新区间的结束位置)
```

只要后面还有新的区间在当前覆盖范围内开始，就可能继续扩展 `have`。

当：

```python
i == have
```

说明扫描已经到达当前合并区间最远的位置，可以安全结束这个区间。

## 为什么保存 `end + 1`

代码中使用：

```python
mp[start] = max(mp[start], end + 1)
```

而不是直接保存：

```python
end
```

这是因为数组初始化值为：

```python
0
```

如果某个真实区间是：

```text
[0, 0]
```

它的结束位置也是 `0`。

如果直接存储 `end`，那么：

```python
mp[0] == 0
```

就无法区分：

- “没有区间从这里开始”
- “有一个区间 `[0,0]` 从这里开始”

因此统一保存：

```python
end + 1
```

这样：

```text
0
```

专门表示“没有区间”。

读取时再转换回来：

```python
mp[i] - 1
```

这是数组编码中很常见的小技巧。

## 算法步骤

1. 找到所有区间中最大的起点：

   ```python
   max_val
   ```

2. 创建数组：

   ```python
   mp = [0] * (max_val + 1)
   ```

3. 对于每个区间 `[start, end]`：

   ```python
   mp[start] = max(mp[start], end + 1)
   ```

4. 从左到右扫描 `mp`。

5. 如果某个位置存在区间起点：

   - 如果当前还没有合并区间，则记下起点；
   - 更新当前能够到达的最远终点 `have`。

6. 当扫描位置 `i` 等于 `have`：

   - 当前区间结束；
   - 加入答案；
   - 重置状态。

7. 扫描结束后，如果还有未关闭的区间，将它加入答案。

## Python 实现

```python
from typing import List


class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        # 找到最大的区间起点
        max_val = max(interval[0] for interval in intervals)

        # mp[start] 保存：
        # 所有从 start 开始的区间中，最大的 end + 1
        #
        # 使用 end + 1 是为了让 0 可以表示：
        # “没有区间从这个位置开始”
        mp = [0] * (max_val + 1)

        for start, end in intervals:
            mp[start] = max(mp[start], end + 1)

        res = []

        # 当前正在合并的区间最远能够到达的位置
        have = -1

        # 当前合并区间的起点
        interval_start = -1

        for i in range(len(mp)):
            if mp[i] != 0:
                # 如果目前没有正在构造的区间，
                # 那么 i 就是新区间的起点
                if interval_start == -1:
                    interval_start = i

                # 更新当前区间能够覆盖到的最远终点
                have = max(have, mp[i] - 1)

            # 已经扫描到了当前能够覆盖到的最远位置
            if have == i:
                res.append([interval_start, have])

                # 重置状态
                have = -1
                interval_start = -1

        # 有些区间的终点可能大于最大起点，
        # 此时扫描数组结束时该区间仍然没有关闭
        if interval_start != -1:
            res.append([interval_start, have])

        return res
```

## Python 函数说明：`max()` + 生成器表达式

这里使用：

```python
max(interval[0] for interval in intervals)
```

其中：

```python
interval[0] for interval in intervals
```

是一个生成器表达式，会依次产生每个区间的起点。

例如：

```python
intervals = [[1, 3], [6, 8], [4, 9]]
```

相当于对：

```text
1, 6, 4
```

调用：

```python
max(...)
```

最终得到：

```python
6
```

也可以写成：

```python
max_val = max([interval[0] for interval in intervals])
```

但这里并不需要真正创建整个临时列表，所以生成器表达式通常更加自然。

## 复杂度分析

设：

- `n` 为区间数量；
- `m` 为所有区间中最大的起点值。

建立 `mp` 需要：

```text
O(n)
```

扫描数组需要：

```text
O(m)
```

因此总时间复杂度：

```text
O(n + m)
```

空间复杂度：

```text
O(m)
```

原解析中将空间复杂度写作 `O(n)` 并不够准确。

因为这里真正决定数组 `mp` 大小的是：

```text
最大起点值 m
```

所以辅助空间应记为：

```text
O(m)
```

## 这种方法的局限性

这个方案只有在坐标范围较小时才有优势。

当前题目中：

```text
start <= 1000
```

所以数组很小。

但如果题目改成：

```text
0 <= start <= end <= 10^9
```

那么就绝对不能创建长度接近 `10^9` 的数组。

因此，这种方法依赖数值范围，不如排序方案通用。

---

# 三种方法对比

| 方法        |   时间复杂度 |                        额外空间 | 推荐程度 | 特点                             |
| ----------- | -----------: | ------------------------------: | -------- | -------------------------------- |
| 排序 + 贪心 | `O(n log n)` | 与排序实现有关，结果最多 `O(n)` | ★★★★★    | 最经典、最通用、最适合面试       |
| 扫描线      | `O(n log n)` |                          `O(n)` | ★★★★☆    | 思想通用，适合扩展到区间计数问题 |
| 数组扫描    |   `O(n + m)` |                          `O(m)` | ★★★☆☆    | 利用坐标范围较小的特殊条件       |

其中：

- `n`：区间数量；
- `m`：最大起点坐标。

---

# 常见错误

## 1. 使用严格小于判断重叠

错误：

```python
if start < last_end:
```

正确：

```python
if start <= last_end:
```

因为：

```text
[1, 3]
[3, 5]
```

共享点 `3`，按照题意属于重叠区间。

最终应合并为：

```text
[1, 5]
```

---

## 2. 合并时直接把终点改成当前区间终点

错误：

```python
output[-1][1] = end
```

正确：

```python
output[-1][1] = max(last_end, end)
```

例如：

```text
[1, 10]
[2, 5]
```

如果直接更新为 `5`，会错误地把原来的区间缩短。

正确结果应该保持：

```text
[1, 10]
```

---

## 3. 排序解法中忘记先排序

排序贪心的正确性依赖于：

> 区间按照起点从小到大处理。

如果没有排序，相互重叠的区间不一定相邻。

例如：

```text
[[1, 3], [8, 10], [2, 6]]
```

其中：

```text
[1,3]
```

实际上应当与：

```text
[2,6]
```

合并。

但是它们在原数组中并不相邻。

排序后：

```text
[[1,3], [2,6], [8,10]]
```

就可以自然地依次完成合并。

---

## 4. 忽略“包含关系”

区间重叠不只有这种情况：

```text
[1,4]
[3,6]
```

也可能是完全包含：

```text
[1,10]
[2,5]
```

因此不能简单认为新的区间一定会扩展结束位置。

使用：

```python
max(last_end, end)
```

可以统一处理：

- 部分重叠；
- 完全包含；
- 相同区间。

---

# 面试时推荐的讲解方式

如果面试中遇到这道题，可以按照下面的逻辑快速说明：

> 首先按照区间起点排序。这样处理当前区间时，只需要与已经合并结果中的最后一个区间比较。
>
> 如果当前区间的起点小于等于最后一个区间的终点，说明两者重叠，将最后区间的终点更新为二者终点的最大值。
>
> 如果当前区间起点大于最后区间终点，说明它们完全不相交，因此把当前区间直接加入结果。
>
> 排序复杂度为 `O(n log n)`，之后只需要一次线性扫描，所以总时间复杂度为 `O(n log n)`。

---

# 进一步理解：为什么排序之后只比较最后一个区间就够了？

这是这道题最关键的正确性理解之一。

假设排序后我们已经处理了：

```text
[1, 3]
[2, 5]
[4, 8]
```

经过合并，结果只剩：

```text
[1, 8]
```

现在处理新区间：

```text
[7, 10]
```

我们无需重新检查：

```text
[1,3]
[2,5]
[4,8]
```

只需要检查：

```text
[1,8]
```

因为前面所有互相连接的覆盖范围已经被压缩成了一个完整区间。

如果当前区间连：

```text
output[-1]
```

都无法重叠，那么由于起点已经排序，它也不可能再与更早结束的区间重叠。

因此：

```python
output[-1]
```

始终包含了我们判断当前区间是否应该继续合并所需要的全部信息。

---

# 推荐最终记忆版本

面试中最建议熟练掌握下面这一版：

```python
from typing import List


class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        # 1. 按起点排序
        intervals.sort(key=lambda interval: interval[0])

        merged = []

        for start, end in intervals:
            # merged 为空，或者当前区间与最后一个区间不重叠
            if not merged or start > merged[-1][1]:
                merged.append([start, end])
            else:
                # 存在重叠，扩展最后一个区间的终点
                merged[-1][1] = max(merged[-1][1], end)

        return merged
```

这一版本相比前面的写法稍微更加紧凑，同时仍然很容易解释。

关键逻辑只有两种情况：

```python
if not merged or start > merged[-1][1]:
```

说明不能合并，开启新区间。

否则：

```python
merged[-1][1] = max(merged[-1][1], end)
```

说明发生重叠，扩展当前区间。

如果只准备一种解法，优先记住这一版。
