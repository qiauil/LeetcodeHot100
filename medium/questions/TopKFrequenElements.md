# Top K Frequent Elements — 前 K 个高频元素

## 题目说明

  给定一个整数数组 `nums` 和一个整数 `k`，返回数组中出现频率最高的 `k` 个元素。

  测试数据保证答案总是唯一的。

  返回结果可以是任意顺序。

### 示例 1

  ```text
  输入: nums = [1,2,2,3,3,3], k = 2
  
  输出: [2,3]
  ```

  解释：

  - `1` 出现 1 次
  - `2` 出现 2 次
  - `3` 出现 3 次

  因此出现频率最高的两个元素是 `3` 和 `2`。

### 示例 2

  ```text
  输入: nums = [7,7], k = 1
  
  输出: [7]
  ```

### 约束条件

  ```text
  1 <= nums.length <= 10^4
  -1000 <= nums[i] <= 1000
  1 <= k <= nums 中不同元素的数量
  ```

------

## 解法一：排序 Sorting

### 核心思路

  想找出出现频率最高的 `k` 个元素，第一步自然是先统计每个数字出现了多少次。

  例如：

  ```text
  nums = [1,2,2,3,3,3]
  ```

  统计之后得到：

  ```text
  1 -> 1 次
  2 -> 2 次
  3 -> 3 次
  ```

  然后我们把每个元素和它的出现次数组合起来：

  ```text
  [1, 1]
  [2, 2]
  [3, 3]
  ```

  其中格式是：

  ```text
  [frequency, number]
  ```

  接着根据出现频率进行排序。

  排序之后：

  ```text
  [
      [1, 1],
      [2, 2],
      [3, 3]
  ]
  ```

  因为频率最高的元素会出现在数组尾部，所以只需要从尾部取出 `k` 个元素即可。

  整体流程可以记成：

  ```text
  统计频率
     ↓
  按照频率排序
     ↓
  取最大的 k 个
  ```

------

### 算法步骤

  1. 使用 Hash Map 统计每个数字出现的次数。
  2. 将每个数字转换成 `[出现次数, 数字]`。
  3. 对这些 pair 进行排序。
  4. 从排序后的数组末尾开始取元素。
  5. 一共取 `k` 个。
  6. 返回结果。

------

### 带中文注释的代码

  ```python
  class Solution:
      def topKFrequent(self, nums: List[int], k: int) -> List[int]:
          # count[num] 表示 num 出现的次数
          count = {}
  
          # 统计每个数字出现的频率
          for num in nums:
              count[num] = 1 + count.get(num, 0)
  
          # arr 中保存 [频率, 数字]
          arr = []
  
          for num, cnt in count.items():
              arr.append([cnt, num])
  
          # Python list 默认按照第一个元素排序
          # 因此这里首先按照频率从小到大排序
          arr.sort()
  
          res = []
  
          # 数组末尾是频率最大的元素
          # 每次取出一个，直到得到 k 个答案
          while len(res) < k:
              freq, num = arr.pop()
              res.append(num)
  
          return res
  ```

  这里有一个 Python 细节值得注意。

  ```python
  arr.sort()
  ```

  如果 `arr` 中的元素是：

  ```python
  [freq, num]
  ```

  Python 会先比较 `freq`。

  如果频率一样，才会继续比较 `num`。

  不过题目已经保证最终答案唯一，因此不需要特别处理频率相同导致的歧义。

------

### 复杂度分析

  假设：

  ```text
  n = nums 的长度
  m = 不同数字的数量
  ```

  显然：

  ```text
  m <= n
  ```

  统计频率需要：

  ```text
  O(n)
  ```

  排序 `m` 个不同元素需要：

  ```text
  O(m log m)
  ```

  最坏情况下每个数字都不同：

  ```text
  m = n
  ```

  因此整体时间复杂度为：

  ```text
  O(n log n)
  ```

  空间复杂度为：

  ```text
  O(n)
  ```

  主要用于 Hash Map 和排序数组。

------

## 解法二：最小堆 Min Heap

  这个解法在 Top K 问题中非常重要，因为它代表了一类非常常见的面试套路：

  > **当题目要求“最大的 K 个”或“最小的 K 个”时，可以考虑维护一个大小为 K 的 Heap。**

------

### 核心思路

  仍然先统计频率：

  ```text
  1 -> 1
  2 -> 2
  3 -> 3
  ```

  但这次我们不把所有元素排序。

  我们只想维护：

  ```text
  当前频率最大的 k 个元素
  ```

  这里使用一个 **Min Heap（最小堆）**。

  最小堆有一个非常重要的性质：

  ```text
  heap[0]
  ```

  永远是堆中最小的元素。

  因此我们可以让堆最多只保存 `k` 个元素。

  每加入一个新的元素：

  ```python
  heapq.heappush(heap, (frequency, num))
  ```

  如果堆的大小超过 `k`：

  ```python
  if len(heap) > k:
      heapq.heappop(heap)
  ```

  把频率最小的那个移除。

  这样最终留下来的，自然就是频率最大的 `k` 个元素。

------

### 一个具体例子

  假设：

  ```text
  nums = [1,2,2,3,3,3]
  k = 2
  ```

  统计之后：

  ```text
  1 -> 1
  2 -> 2
  3 -> 3
  ```

  初始化：

  ```text
  heap = []
  ```

  加入 `1`：

  ```text
  heap = [(1, 1)]
  ```

  加入 `2`：

  ```text
  heap = [
      (1, 1),
      (2, 2)
  ]
  ```

  目前大小为 2，不超过 `k`。

  加入 `3`：

  ```text
  heap = [
      (1, 1),
      (2, 2),
      (3, 3)
  ]
  ```

  现在：

  ```text
  len(heap) = 3 > k
  ```

  所以删除最小的频率：

  ```text
  (1, 1)
  ```

  最终：

  ```text
  heap = [
      (2, 2),
      (3, 3)
  ]
  ```

  剩下的就是答案：

  ```text
  [2, 3]
  ```

------

### 为什么找“最大的 K 个”反而使用 Min Heap？

  这是非常值得理解的地方。

  直觉上可能会觉得：

  > 我要找最大的，应该用 Max Heap。

  但实际上，如果我们想一直维护“当前最大的 K 个”，真正需要知道的是：

  > **这 K 个元素里面，谁最小？**

  因为当新的候选元素进来时，如果它比当前 Top K 中最小的那个更重要，我们就把最小的淘汰掉。

  所以我们需要快速找到：

  ```text
  Top K 中最小的元素
  ```

  这正是 Min Heap 最擅长的事情。

  可以把它理解为一个只有 `k` 个座位的排行榜：

  ```text
            Top K
      ┌─────────────┐
      │ frequency 8 │
      │ frequency 7 │
      │ frequency 5 │ ← 最弱的选手
      └─────────────┘
  ```

  如果来了一个：

  ```text
  frequency = 6
  ```

  那么 `5` 就会被淘汰。

  Min Heap 可以让我们快速找到这个 `5`。

------

### 带中文注释的代码

  ```python
  import heapq
  
  class Solution:
      def topKFrequent(self, nums: List[int], k: int) -> List[int]:
          # count[num] = num 出现的次数
          count = {}
  
          # 第一步：统计频率
          for num in nums:
              count[num] = 1 + count.get(num, 0)
  
          # Python 的 heapq 默认实现的是 Min Heap
          heap = []
  
          # 遍历所有不同的数字
          for num in count:
              # 将 (频率, 数字) 放入最小堆
              heapq.heappush(heap, (count[num], num))
  
              # 堆中最多只保留 k 个元素
              # 如果超过 k 个，就删除频率最小的元素
              if len(heap) > k:
                  heapq.heappop(heap)
  
          res = []
  
          # 此时 heap 中剩下的就是频率最高的 k 个数字
          for _ in range(k):
              freq, num = heapq.heappop(heap)
              res.append(num)
  
          return res
  ```

------

### Min Heap 的复杂度

  仍然设：

  ```text
  n = nums 长度
  m = 不同元素数量
  ```

  统计频率：

  ```text
  O(n)
  ```

  接下来遍历 `m` 个不同元素。

  每次执行：

  ```python
  heappush()
  heappop()
  ```

  因为堆的大小始终最多为 `k`，每次 Heap 操作的复杂度是：

  ```text
  O(log k)
  ```

  因此：

  ```text
  O(m log k)
  ```

  最坏 `m = n`：

  ```text
  O(n log k)
  ```

  最后从 Heap 中取出 `k` 个元素：

  ```text
  O(k log k)
  ```

  所以整体通常写作：

  ```text
  O(n log k)
  ```

  空间复杂度：

  ```text
  O(n + k)
  ```

  其中：

  ```text
  O(n)
  ```

  用于 frequency map，

  ```text
  O(k)
  ```

  用于 heap。

  由于 `k <= n`，有时也可以简化写成：

  ```text
  O(n)
  ```

------

## 阶段性对比：Sorting vs Min Heap

  | 方法     | 时间复杂度   | 空间复杂度 | 思路         |
  | -------- | ------------ | ---------- | ------------ |
  | Sorting  | `O(n log n)` | `O(n)`     | 全部排序     |
  | Min Heap | `O(n log k)` | `O(n + k)` | 只维护 Top K |

  如果：

  ```text
  k << n
  ```

  比如：

  ```text
  n = 1,000,000
  k = 10
  ```

  Min Heap 的优势就会比较明显。

  因为：

  ```text
  Sorting:
  O(n log n)
  
  Heap:
  O(n log 10)
  ```

  而 `log 10` 是一个很小的数。

------

## Top K 通用面试模式

  这道题其实属于非常经典的：

  **Hash Map + Heap**

  模式。

  通常看到以下关键词：

  ```text
  Top K
  K largest
  K smallest
  K most frequent
  K closest
  ```

  都应该立刻想到 Heap。

  一个很实用的记忆方式是：

  ```text
  Top K 最大
  → 维护大小为 K 的 Min Heap
  
  Top K 最小
  → 维护大小为 K 的 Max Heap
  ```

  原因都是：

  > Heap 顶部放的是“当前 Top K 中最应该被淘汰的那个元素”。

## 解法三：Bucket Sort（桶排序）

### 1. 核心思路

前两个解法都是：

```text
Hash Map 统计频率
        ↓
按照频率找 Top K
```

区别只在第二步：

- Sorting：把所有频率排序
- Min Heap：维护大小为 `k` 的堆
- Bucket Sort：**直接用“频率”作为数组下标**

关键观察是：

> 如果 `nums` 的长度为 `n`，那么任何一个数字出现的次数一定在 `1 ~ n` 之间。

例如：

```python
nums = [1, 2, 2, 3, 3, 3]
```

这里：

```text
n = 6
```

任何数字的出现频率不可能超过 `6`。

因此我们可以创建一个大小为：

```python
len(nums) + 1
```

的数组：

```python
freq = [[] for _ in range(len(nums) + 1)]
```

数组下标代表：

> **出现次数**

而对应位置存储：

> **出现这么多次的数字**

------

### 2. 用例子理解 Bucket

对于：

```python
nums = [1, 2, 2, 3, 3, 3]
```

首先统计频率：

```text
1 -> 1 次
2 -> 2 次
3 -> 3 次
```

然后建立桶：

```text
bucket index
    ↓

0 → []
1 → [1]
2 → [2]
3 → [3]
4 → []
5 → []
6 → []
```

也就是说：

```python
freq[1] = [1]
freq[2] = [2]
freq[3] = [3]
```

这里非常重要：

```python
freq[i]
```

表示：

> **所有恰好出现 `i` 次的数字。**

然后我们从后往前遍历：

```text
6 → 5 → 4 → 3 → 2 → 1
```

因为：

> 下标越大 → 出现次数越多。

当：

```python
k = 2
```

先遇到：

```text
freq[3] = [3]
```

加入 `3`。

然后：

```text
freq[2] = [2]
```

加入 `2`。

已经收集两个元素，于是直接返回：

```python
[3, 2]
```

题目允许任意顺序，因此 `[3, 2]` 和 `[2, 3]` 都正确。

------

### 3. 完整代码

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        # Step 1:
        # 使用 Hash Map 统计每个数字出现的次数
        count = {}

        for num in nums:
            count[num] = 1 + count.get(num, 0)

        # Step 2:
        # freq[i] 表示：
        # 所有出现次数恰好为 i 的数字
        #
        # 最大频率不可能超过 len(nums)
        # 因此需要 len(nums) + 1 个位置
        freq = [[] for _ in range(len(nums) + 1)]

        # 把数字放入对应频率的 bucket
        for num, cnt in count.items():
            freq[cnt].append(num)

        # Step 3:
        # 从频率最高的位置开始向前找
        res = []

        for i in range(len(freq) - 1, 0, -1):
            # freq[i] 中可能有多个数字
            for num in freq[i]:
                res.append(num)

                # 一旦找到 k 个，就直接返回
                if len(res) == k:
                    return res
```

这是我比较推荐在面试中写的版本。

------

### 4. 为什么是 `len(nums) + 1`？

这一点面试里很容易出现小 bug。

假设：

```python
nums = [7, 7]
```

那么：

```text
len(nums) = 2
```

数字 `7` 的频率是：

```text
2
```

所以我们必须能够访问：

```python
freq[2]
```

如果写成：

```python
freq = [[] for _ in range(len(nums))]
```

那么只有：

```text
freq[0]
freq[1]
```

没有：

```text
freq[2]
```

会发生：

```text
IndexError
```

所以必须写：

```python
len(nums) + 1
```

让合法下标为：

```text
0, 1, 2, ..., n
```

------

### 5. 为什么每个 Bucket 是一个 List？

你可能会想，为什么不是：

```python
freq[cnt] = num
```

而是：

```python
freq[cnt].append(num)
```

因为多个数字可能有相同频率。

例如：

```python
nums = [1, 1, 2, 2, 3]
```

频率为：

```text
1 -> 2 次
2 -> 2 次
3 -> 1 次
```

那么：

```python
freq[2]
```

必须同时保存：

```python
[1, 2]
```

因此每一个 bucket 本身都需要是一个 list：

```text
freq
 │
 ├── freq[0] = []
 ├── freq[1] = [3]
 ├── freq[2] = [1, 2]
 ├── freq[3] = []
 ├── freq[4] = []
 └── freq[5] = []
```

这也是为什么初始化的时候要写：

```python
freq = [[] for _ in range(len(nums) + 1)]
```

------

### 6. Python 知识点：List Comprehension

这里：

```python
freq = [[] for _ in range(len(nums) + 1)]
```

使用的是 Python 非常常见的：

**List Comprehension（列表推导式）**

基本语法：

```python
[expression for variable in iterable]
```

例如：

```python
squares = [x * x for x in range(5)]
```

结果：

```python
[0, 1, 4, 9, 16]
```

这道题：

```python
[[] for _ in range(len(nums) + 1)]
```

可以理解成：

```python
freq = []

for _ in range(len(nums) + 1):
    freq.append([])
```

两个代码完全等价。

------

#### `_` 是什么意思？

这里：

```python
for _ in range(...)
```

`_` 并不是 Python 的特殊语法。

它其实只是一个普通变量名。

但是 Python 程序员习惯使用 `_` 表示：

> “这个变量我根本不会使用。”

例如：

```python
for _ in range(5):
    print("hello")
```

我们只想执行五次，并不关心循环变量到底是：

```text
0
1
2
3
4
```

所以写 `_`。

------

### 7. 一个非常重要的 Python 坑

初始化二维 List 时，不建议写：

```python
freq = [[]] * (len(nums) + 1)
```

虽然看起来好像和：

```python
freq = [[] for _ in range(len(nums) + 1)]
```

一样，但实际上完全不同。

例如：

```python
freq = [[]] * 3
```

看起来得到：

```python
[[], [], []]
```

但是三个位置实际上指向的是**同一个 list 对象**。

所以：

```python
freq[0].append(1)
```

结果会变成：

```python
[[1], [1], [1]]
```

这通常不是我们想要的。

而：

```python
freq = [[] for _ in range(3)]
```

会创建三个真正独立的 list。

所以：

```python
freq[0].append(1)
```

得到：

```python
[[1], [], []]
```

这也是算法面试里一个挺常见的 Python 小坑。

------

### 8. Python 知识点：`dict.get()`

我们之前一直用了：

```python
count[num] = 1 + count.get(num, 0)
```

这里值得专门记一下。

字典：

```python
dictionary.get(key, default)
```

含义是：

> 如果 `key` 存在，返回对应 value；否则返回 `default`。

例如：

```python
count = {}

count.get(5, 0)
```

因为 `5` 不存在，所以返回：

```python
0
```

于是第一次遇到 `5`：

```python
count[5] = 1 + 0
```

得到：

```python
count[5] = 1
```

第二次遇到：

```python
count.get(5, 0)
```

得到：

```python
1
```

于是：

```python
count[5] = 2
```

所以：

```python
count[num] = 1 + count.get(num, 0)
```

是一种非常常见的 frequency counting 写法。

------

### 9. Python 知识点：`dict.items()`

这里：

```python
for num, cnt in count.items():
```

`dict.items()` 会同时返回：

```text
key, value
```

例如：

```python
count = {
    1: 1,
    2: 2,
    3: 3
}
```

那么：

```python
count.items()
```

概念上类似：

```python
[
    (1, 1),
    (2, 2),
    (3, 3)
]
```

所以：

```python
for num, cnt in count.items():
```

会依次得到：

```text
num = 1, cnt = 1
num = 2, cnt = 2
num = 3, cnt = 3
```

面试中特别常见。

可以顺便把三个常用方法放在一起记：

```python
for key in dictionary:
    ...

for key in dictionary.keys():
    ...

for value in dictionary.values():
    ...

for key, value in dictionary.items():
    ...
```

其中实际写题时最常用的通常是：

```python
for key in dictionary:
```

和：

```python
for key, value in dictionary.items():
```

------

### 10. Python 知识点：倒序 `range()`

Bucket Sort 最关键的一行之一是：

```python
for i in range(len(freq) - 1, 0, -1):
```

`range()` 完整形式是：

```python
range(start, stop, step)
```

注意：

> `stop` 不包含在结果里。

所以：

```python
range(6, 0, -1)
```

产生：

```text
6, 5, 4, 3, 2, 1
```

不会包含 `0`。

这恰好符合我们的需求，因为：

```python
freq[0]
```

没有意义——不存在“出现 0 次但又存在于数组中的元素”。

这种倒序遍历在面试里非常常见：

```python
for i in range(n - 1, -1, -1):
```

表示：

```text
n-1, n-2, ..., 1, 0
```

建议熟悉。

------

### 11. 完整运行过程

假设：

```python
nums = [1, 2, 2, 3, 3, 3]
k = 2
```

#### Step 1：统计

```python
count = {
    1: 1,
    2: 2,
    3: 3
}
```

#### Step 2：建立 Bucket

初始：

```python
freq = [
    [],  # 0
    [],  # 1
    [],  # 2
    [],  # 3
    [],  # 4
    [],  # 5
    []   # 6
]
```

执行：

```python
for num, cnt in count.items():
    freq[cnt].append(num)
```

得到：

```python
freq = [
    [],     # 0 次
    [1],    # 1 次
    [2],    # 2 次
    [3],    # 3 次
    [],     # 4 次
    [],     # 5 次
    []      # 6 次
]
```

#### Step 3：倒序寻找

```text
i = 6
freq[6] = []

i = 5
freq[5] = []

i = 4
freq[4] = []

i = 3
freq[3] = [3]
res = [3]

i = 2
freq[2] = [2]
res = [3, 2]
```

此时：

```python
len(res) == k
```

所以直接：

```python
return res
```

------

### 12. 时间复杂度

我们仔细分析一下为什么它真的是 `O(n)`。

#### 第一步：统计频率

```python
for num in nums:
```

遍历 `n` 个元素：

```text
O(n)
```

#### 第二步：建立 Bucket

```python
for num, cnt in count.items():
```

假设不同数字数量是 `m`：

```text
O(m)
```

且：

```text
m <= n
```

所以最多：

```text
O(n)
```

#### 第三步：遍历 Bucket

Bucket 长度为：

```text
n + 1
```

最多遍历：

```text
O(n)
```

同时所有 bucket 中的数字总数不会超过不同数字数量 `m`，因此也是：

```text
O(n)
```

所以总时间：

```text
O(n) + O(n) + O(n)
= O(n)
```

最终：

**时间复杂度：`O(n)`**

空间复杂度：

```text
count → O(n)
freq  → O(n)
```

所以：

**空间复杂度：`O(n)`**

------

### 13. 为什么 Bucket Sort 比普通 Sorting 快？

Sorting 的问题是：

```text
我们知道频率之后，还把频率相互比较并排序。
```

这需要：

```text
O(n log n)
```

但 Bucket Sort 利用了额外的信息：

> **频率只能是 1 到 n 之间的整数。**

既然频率本身就可以直接作为数组下标：

```python
freq[count]
```

就完全没有必要比较排序。

可以类比考试成绩。

假设有一百万个人，但分数只能是：

```text
0 ~ 100
```

如果只是想按成绩分类，你没有必要把一百万人全部排序。

直接创建：

```text
bucket[0]
bucket[1]
...
bucket[100]
```

然后把学生扔进对应分数的桶即可。

这就是 Bucket Sort 思维的本质：

> **当排序的 key 范围有限且已知时，可以尝试用空间换时间，直接使用 key 作为下标。**

------

## 三种解法的优化路径

这道题非常适合用来理解算法优化的过程。

### 第一层：Sorting

最自然：

```text
统计
↓
全部排序
↓
Top K
O(n log n)
```

### 第二层：Heap

我们发现：

> 我根本不需要所有元素都有序，只需要 Top K。

所以：

```text
统计
↓
只维护 K 个
O(n log k)
```

### 第三层：Bucket Sort

进一步发现：

> 频率不是任意数字，它最多只有 n。

所以甚至不用比较：

```text
统计
↓
frequency 直接作为 index
↓
从后向前找
O(n)
```

这其实是一个很典型的算法优化思维：

```text
先问：
“我真的需要全部排序吗？”

↓ No

再问：
“我真的需要比较吗？”

↓ No

利用数据范围直接映射
```

------

## 面试记忆点

建议你把这道题最后浓缩成下面三个关键词：

```text
Top K Frequent Elements

1. Hash Map
   → count frequency

2. Bucket
   → bucket[frequency] = numbers

3. Reverse traversal
   → highest frequency first
```

尤其是看到：

> **frequency**

这个词时，可以多想一步：

> “frequency 最大是多少？”

如果答案是：

```text
最多 n
```

那么 **Bucket Sort / frequency array** 往往就是一个值得考虑的方向。

之后你继续发题目时，我也会保持这种格式：除了翻译、算法、复杂度以外，如果出现值得掌握的 Python API，我会单独解释它的**语法、行为、常见面试用法以及容易踩的坑**。
