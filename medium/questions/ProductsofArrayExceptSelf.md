# 除自身以外数组的乘积（Products of Array Except Self）

给定一个整数数组 `nums`，返回数组 `output`，其中：

```text
output[i] = nums 中除 nums[i] 以外所有元素的乘积
```

题目保证每个结果都可以存储在 **32 位整数**中。

**进阶要求：** 能否在 **O(n)** 时间复杂度内，并且**不使用除法**解决这个问题？

### 示例 1

```text
输入：nums = [1, 2, 4, 6]

输出：[48, 24, 12, 8]
```

解释：

```text
output[0] = 2 × 4 × 6 = 48
output[1] = 1 × 4 × 6 = 24
output[2] = 1 × 2 × 6 = 12
output[3] = 1 × 2 × 4 = 8
```

### 示例 2

```text
输入：nums = [-1, 0, 1, 2, 3]

输出：[0, -6, 0, 0, 0]
```

### 约束条件

- `2 <= nums.length <= 100000`
- `-30 <= nums[i] <= 30`
- 任意前缀或后缀的乘积都保证可以存储在 **32 位整数**中。

------

# 方法一：暴力枚举

## 思路

最直接的方法就是严格按照题意来做：

对于数组中的每一个位置 `i`，遍历整个数组，将除了 `nums[i]` 以外的所有元素相乘。

例如：

```text
nums = [1, 2, 4, 6]

i = 0:
2 × 4 × 6 = 48

i = 1:
1 × 4 × 6 = 24
```

这种方法非常容易想到，但问题在于：

- 一共有 `n` 个位置；
- 每个位置都需要再次遍历 `n` 个元素。

因此会产生大量重复计算。

## 算法步骤

1. 创建长度为 `n` 的结果数组 `res`。
2. 遍历每一个索引 `i`。
3. 对于每个 `i`：
   - 初始化 `prod = 1`；
   - 再遍历整个数组；
   - 如果当前索引 `j == i`，跳过；
   - 否则将 `nums[j]` 乘入 `prod`。
4. 将最终乘积保存到 `res[i]`。
5. 返回 `res`。

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)
        res = [0] * n

        # 对每一个位置分别计算答案
        for i in range(n):
            prod = 1

            # 遍历整个数组
            for j in range(n):
                # 跳过当前位置
                if i == j:
                    continue

                prod *= nums[j]

            res[i] = prod

        return res
```

## 时间与空间复杂度

- 时间复杂度：**O(n²)**
- 额外空间复杂度：**O(1)**
- 如果计算返回结果数组本身，则总空间为 **O(n)**。

这里通常说“额外空间复杂度 O(1)”是因为题目要求我们必须返回一个长度为 `n` 的结果数组，这个输出数组一般不算额外空间。

------

# 方法二：使用除法

## 思路

如果数组中不存在 `0`，事情会非常简单。

假设所有元素的乘积为：

```text
total = nums[0] × nums[1] × ... × nums[n - 1]
```

那么：

```text
res[i] = total / nums[i]
```

例如：

```text
nums = [1, 2, 4, 6]

total = 48

res[0] = 48 / 1 = 48
res[1] = 48 / 2 = 24
res[2] = 48 / 4 = 12
res[3] = 48 / 6 = 8
```

真正需要处理的是 **0**。

### 情况一：没有 0

直接：

```text
res[i] = product / nums[i]
```

### 情况二：恰好有一个 0

例如：

```text
[-1, 0, 1, 2, 3]
```

所有非零数字的乘积：

```text
-1 × 1 × 2 × 3 = -6
```

对于 `0` 所在的位置：

```text
-1 × 1 × 2 × 3 = -6
```

而对于其他位置，由于乘积中一定包含那个 `0`，所以结果全部为：

```text
0
```

因此：

```text
[0, -6, 0, 0, 0]
```

### 情况三：至少有两个 0

例如：

```text
[1, 0, 3, 0]
```

无论排除哪个元素，剩余元素中都至少还存在一个 `0`。

所以结果一定是：

```text
[0, 0, 0, 0]
```

## 算法步骤

第一次遍历数组：

- 计算所有非零元素的乘积 `prod`；
- 统计 `0` 的数量 `zero_cnt`。

然后根据零的数量处理：

- `zero_cnt >= 2`：所有答案都是 `0`；
- `zero_cnt == 1`：
  - `0` 所在位置的答案是 `prod`；
  - 其他位置都是 `0`；
- `zero_cnt == 0`：
  - `res[i] = prod // nums[i]`。

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        prod = 1
        zero_cnt = 0

        # 计算所有非零元素的乘积，并统计 0 的数量
        for num in nums:
            if num != 0:
                prod *= num
            else:
                zero_cnt += 1

        # 如果至少有两个 0，无论排除哪个位置，
        # 剩余元素中都至少有一个 0
        if zero_cnt > 1:
            return [0] * len(nums)

        res = [0] * len(nums)

        for i, num in enumerate(nums):
            if zero_cnt == 1:
                # 只有 0 所在的位置可以得到非零结果
                if num == 0:
                    res[i] = prod
                else:
                    res[i] = 0
            else:
                # 没有 0 时可以直接使用除法
                res[i] = prod // num

        return res
```

## Python 补充：`enumerate`

这里使用了：

```python
for i, num in enumerate(nums):
```

`enumerate()` 可以同时获得：

- 元素索引 `i`
- 元素本身 `num`

例如：

```python
nums = [10, 20, 30]

for i, num in enumerate(nums):
    print(i, num)
```

输出：

```text
0 10
1 20
2 30
```

相比：

```python
for i in range(len(nums)):
    num = nums[i]
```

通常更加简洁、符合 Python 的写法。

## 时间与空间复杂度

- 时间复杂度：**O(n)**
- 额外空间复杂度：**O(1)**
- 输出数组占用 **O(n)**。

不过，这个方法**不满足题目的进阶要求**，因为使用了除法。

------

# 方法三：前缀乘积 + 后缀乘积

这是这道题最重要的核心思路。

## 核心思想

对于任意位置 `i`，我们真正需要的是：

```text
左边所有元素的乘积 × 右边所有元素的乘积
```

也就是：

```text
res[i]
= nums[0] × ... × nums[i-1]
  ×
  nums[i+1] × ... × nums[n-1]
```

因此可以将问题拆成两个部分：

- `pref[i]`：索引 `i` **左侧所有元素**的乘积；
- `suff[i]`：索引 `i` **右侧所有元素**的乘积。

最终：

```text
res[i] = pref[i] × suff[i]
```

------

## 前缀乘积

定义：

```text
pref[i] = nums[i] 左边所有元素的乘积
```

注意：

> **不包含 `nums[i]` 本身。**

例如：

```text
nums = [1, 2, 4, 6]
```

则：

```text
pref[0] = 1
pref[1] = 1
pref[2] = 1 × 2 = 2
pref[3] = 1 × 2 × 4 = 8
```

所以：

```text
pref = [1, 1, 2, 8]
```

为什么：

```text
pref[0] = 1
```

因为索引 `0` 左侧没有任何元素。

在乘法中，空集合通常使用乘法单位元：

```text
1
```

因为：

```text
x × 1 = x
```

------

## 后缀乘积

类似地：

```text
suff[i] = nums[i] 右边所有元素的乘积
```

对于：

```text
nums = [1, 2, 4, 6]
```

得到：

```text
suff[3] = 1
suff[2] = 6
suff[1] = 4 × 6 = 24
suff[0] = 2 × 4 × 6 = 48
```

因此：

```text
suff = [48, 24, 6, 1]
```

------

## 合并结果

现在：

```text
res[i] = pref[i] × suff[i]
```

所以：

```text
i = 0: 1 × 48 = 48
i = 1: 1 × 24 = 24
i = 2: 2 × 6  = 12
i = 3: 8 × 1  = 8
```

得到：

```text
[48, 24, 12, 8]
```

## 算法步骤

1. 创建：

   - `pref`
   - `suff`
   - `res`

2. 设置：

   ```python
   pref[0] = 1
   suff[n - 1] = 1
   ```

3. 从左向右计算前缀乘积。

4. 从右向左计算后缀乘积。

5. 对每个位置：

   ```python
   res[i] = pref[i] * suff[i]
   ```

6. 返回结果。

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)

        res = [0] * n
        pref = [0] * n
        suff = [0] * n

        # 第一个位置左边没有元素
        pref[0] = 1

        # 最后一个位置右边没有元素
        suff[n - 1] = 1

        # pref[i] 表示 i 左边所有元素的乘积
        for i in range(1, n):
            pref[i] = pref[i - 1] * nums[i - 1]

        # suff[i] 表示 i 右边所有元素的乘积
        for i in range(n - 2, -1, -1):
            suff[i] = suff[i + 1] * nums[i + 1]

        # 左边乘积 × 右边乘积
        for i in range(n):
            res[i] = pref[i] * suff[i]

        return res
```

## Python 补充：逆序 `range`

代码：

```python
range(n - 2, -1, -1)
```

表示从：

```text
n - 2
```

开始，一直递减到：

```text
0
```

例如：

```python
list(range(3, -1, -1))
```

得到：

```python
[3, 2, 1, 0]
```

需要注意，Python 的 `range(start, stop, step)` **不包含 `stop`**。

因此这里写：

```python
stop = -1
```

才能让索引 `0` 被遍历到。

## 时间与空间复杂度

- 时间复杂度：**O(n)**
- 额外空间复杂度：**O(n)**

这里需要额外的：

```text
pref: O(n)
suff: O(n)
```

所以虽然时间复杂度已经达到最优，但空间仍然可以继续优化。

------

# 方法四：前缀 + 后缀，空间最优解

这是面试中最推荐掌握的解法。

## 核心思路

上一种方法中，我们创建了：

```text
pref
suff
res
```

但仔细观察会发现：

`pref` 数组实际上没有必要单独存在。

因为最终：

```text
res[i] = pref[i] × suff[i]
```

所以我们完全可以先把：

```text
pref[i]
```

直接存进：

```text
res[i]
```

然后从右往左扫描，用一个变量维护后缀乘积，再乘进去。

也就是说：

### 第一次遍历

令：

```text
res[i] = i 左边所有元素的乘积
```

### 第二次遍历

维护：

```text
postfix = i 右边所有元素的乘积
```

然后：

```text
res[i] *= postfix
```

于是：

```text
res[i]
= 左边所有元素乘积
× 右边所有元素乘积
```

正好就是答案。

------

## 第一次遍历：计算左侧乘积

以：

```text
nums = [1, 2, 4, 6]
```

为例。

初始化：

```text
prefix = 1
res = [1, 1, 1, 1]
```

### i = 0

```text
res[0] = 1
prefix = 1 × 1 = 1
```

### i = 1

```text
res[1] = 1
prefix = 1 × 2 = 2
```

### i = 2

```text
res[2] = 2
prefix = 2 × 4 = 8
```

### i = 3

```text
res[3] = 8
prefix = 8 × 6 = 48
```

此时：

```text
res = [1, 1, 2, 8]
```

实际上它就是上一种方法中的：

```text
pref
```

------

## 第二次遍历：加入右侧乘积

从右向左。

初始化：

```text
postfix = 1
```

### i = 3

右边没有元素：

```text
res[3] *= 1
```

得到：

```text
8
```

更新：

```text
postfix *= nums[3]
postfix = 6
```

### i = 2

```text
res[2] *= 6
2 × 6 = 12
```

更新：

```text
postfix = 6 × 4 = 24
```

### i = 1

```text
res[1] *= 24
1 × 24 = 24
```

更新：

```text
postfix = 24 × 2 = 48
```

### i = 0

```text
res[0] *= 48
1 × 48 = 48
```

最终：

```text
res = [48, 24, 12, 8]
```

------

## 代码

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)

        # res 最开始用于保存“左侧所有元素的乘积”
        res = [1] * n

        # -------------------------
        # 第一遍：从左向右
        # -------------------------

        # prefix 表示当前位置左边所有元素的乘积
        prefix = 1

        for i in range(n):
            # 此时 prefix 还没有乘 nums[i]
            # 所以它恰好表示 nums[i] 左侧所有元素的乘积
            res[i] = prefix

            # 更新 prefix，供下一个位置使用
            prefix *= nums[i]

        # -------------------------
        # 第二遍：从右向左
        # -------------------------

        # postfix 表示当前位置右边所有元素的乘积
        postfix = 1

        for i in range(n - 1, -1, -1):
            # res[i] 当前保存左侧乘积
            # 再乘右侧乘积，就是最终答案
            res[i] *= postfix

            # 更新 postfix，供左边的位置使用
            postfix *= nums[i]

        return res
```

## 时间与空间复杂度

- 时间复杂度：**O(n)**
- 额外空间复杂度：**O(1)**
- 输出数组：**O(n)**

这是题目的标准最优解。

------

# 为什么这个方法可以自动处理 0？

这是前缀 / 后缀方法一个很漂亮的地方：

> 完全不需要单独判断 `0`。

例如：

```text
nums = [-1, 0, 1, 2, 3]
```

对于索引 `1`，也就是数字 `0`：

```text
左侧乘积 = -1
右侧乘积 = 1 × 2 × 3 = 6

答案 = -1 × 6 = -6
```

而对于索引 `0`：

```text
右侧包含 0
```

所以结果自然变成 `0`。

对于索引 `2`：

```text
左侧包含 0
```

结果也自然为 `0`。

因此整个结果：

```text
[0, -6, 0, 0, 0]
```

如果有两个 `0`，任何一个位置的左侧或右侧都必然包含至少一个 `0`，于是结果自然全部为 `0`。

这也是为什么 **前缀 / 后缀方法比除法方法更加通用和优雅**。

------

# 面试中最重要的思维过程

如果面试官要求：

```text
O(n) 时间
不使用除法
```

一个很自然的推导过程是：

首先观察：

```text
answer[i]
=
nums[i] 左边所有元素的乘积
×
nums[i] 右边所有元素的乘积
```

于是把问题拆成：

```text
Prefix × Suffix
```

进一步发现：

```text
Prefix 可以直接存入结果数组
Suffix 不需要数组，只需要一个滚动变量
```

最终得到：

```text
两个线性扫描
+ 两个滚动变量
```

因此达到：

```text
时间：O(n)
额外空间：O(1)
```

可以把整道题浓缩成下面这个公式：

```text
answer[i]
=
product(nums[0:i])
×
product(nums[i+1:n])
```

------

# 常见错误

## 1. Prefix 包含了当前元素

正确的定义是：

```text
pref[i] = i 左边所有元素的乘积
```

不是：

```text
pref[i] = nums[0] × ... × nums[i]
```

因此更新顺序非常重要。

正确：

```python
res[i] = prefix
prefix *= nums[i]
```

错误：

```python
prefix *= nums[i]
res[i] = prefix
```

如果先乘 `nums[i]`，那么 `res[i]` 就会包含当前元素，从而违反题目要求。

------

## 2. Suffix 同样不能包含当前元素

正确顺序：

```python
res[i] *= postfix
postfix *= nums[i]
```

而不是：

```python
postfix *= nums[i]
res[i] *= postfix
```

核心规律其实非常简单：

> **先使用当前累计值，再把当前元素加入累计值。**

无论 Prefix 还是 Suffix 都一样。

------

## 3. 逆序 `range` 写错

正确写法：

```python
for i in range(n - 1, -1, -1):
```

表示：

```text
n-1, n-2, ..., 1, 0
```

常见错误：

```python
range(n - 1, 0, -1)
```

这个写法不会处理索引 `0`。

------

## 4. 使用除法时没有考虑 0

简单写：

```python
total // nums[i]
```

遇到：

```text
nums[i] = 0
```

就会产生除零问题。

而且：

```text
一个 0
```

和：

```text
两个及以上 0
```

还需要不同逻辑。

因此如果题目明确要求**不能使用除法**，通常直接考虑 Prefix / Suffix 会更加合适。

------

## 5. 关于整数溢出

在 Python 中，整数 `int` 会自动扩展，因此一般不需要担心普通整数溢出。

例如：

```python
x = 10 ** 100
```

Python 仍然可以正常存储。

但是在 Java / C++ 等固定整数类型语言中，需要注意：

```text
int
```

通常是 32 位整数。

可能需要根据题目约束使用：

```text
Java: long
C++: long long
```

不过这道题已经明确保证相关乘积能够存储在规定整数范围内，因此按照题目要求实现即可。

------

# 四种方法对比

| 方法                      | 时间复杂度 | 额外空间 | 使用除法 | 推荐程度   |
| ------------------------- | ---------- | -------- | -------- | ---------- |
| 暴力枚举                  | O(n²)      | O(1)     | 否       | 仅用于理解 |
| 总乘积 + 除法             | O(n)       | O(1)     | 是       | 一般       |
| Prefix + Suffix 数组      | O(n)       | O(n)     | 否       | 很好       |
| Prefix + Postfix 滚动变量 | **O(n)**   | **O(1)** | 否       | **最推荐** |

------

# 面试推荐写法

如果需要在代码面试中直接给出最终方案，建议优先写下面这个版本：

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        n = len(nums)
        res = [1] * n

        # 第一遍：
        # res[i] 保存 nums[i] 左边所有元素的乘积
        prefix = 1
        for i in range(n):
            res[i] = prefix
            prefix *= nums[i]

        # 第二遍：
        # postfix 保存 nums[i] 右边所有元素的乘积
        postfix = 1
        for i in range(n - 1, -1, -1):
            res[i] *= postfix
            postfix *= nums[i]

        return res
```

可以记忆成：

```text
左 → 右：先写 prefix，再更新 prefix

右 → 左：先乘 postfix，再更新 postfix
```

或者更简洁：

> **先用，再更新。**

这是避免 Prefix / Suffix 边界错误最好用的记忆方式。
