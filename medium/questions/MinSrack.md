# 最小栈（Min Stack）

设计一个栈，使其支持以下操作：

- `MinStack()`：初始化栈对象。
- `push(val)`：将元素 `val` 压入栈中。
- `pop()`：删除栈顶元素。
- `top()`：获取栈顶元素。
- `getMin()`：获取栈中的最小元素。

要求每个操作的时间复杂度均为 \(O(1)\)。

## 前置知识

在解决这道题之前，建议掌握：

- **栈（Stack）**：遵循后进先出（LIFO，Last In First Out）原则，常见操作包括压栈、出栈和查看栈顶。
- **辅助数据结构**：使用额外的栈或变量，记录主数据结构之外的状态。
- **空间与时间的权衡**：通过使用额外空间换取更快的操作。例如，使用 \(O(n)\) 额外空间，将 `getMin()` 优化至 \(O(1)\)。

------

## 方法一：暴力查找

### 思路

普通栈只保存元素本身，并不会额外记录当前最小值。

因此，每次调用 `getMin()` 时，可以暂时弹出栈中的所有元素，在此过程中计算最小值，然后再将所有元素放回原栈。

这种方法比较直观，但每次查询最小值都需要遍历整个栈，不满足题目要求的 \(O(1)\) 时间复杂度。它更适合作为理解和对比其他优化方案的基础方法。

### 算法步骤

1. `push(val)`：将 `val` 添加到列表末尾。
2. `pop()`：删除列表末尾的元素。
3. `top()`：返回列表末尾的元素。
4. `getMin()`：
   - 创建一个临时栈。
   - 依次弹出原栈中的所有元素，同时记录最小值。
   - 将临时栈中的元素依次弹出并放回原栈，从而恢复原来的元素顺序。
   - 返回找到的最小值。

```
class MinStack:
    def __init__(self):
        # 使用 Python 列表模拟栈
        self.stack = []

    def push(self, val: int) -> None:
        # append() 将元素添加到列表末尾，相当于压栈
        self.stack.append(val)

    def pop(self) -> None:
        # pop() 删除并返回列表末尾的元素
        self.stack.pop()

    def top(self) -> int:
        # -1 表示列表中的最后一个元素，即栈顶元素
        return self.stack[-1]

    def getMin(self) -> int:
        temp_stack = []
        minimum = self.stack[-1]

        # 弹出所有元素，并计算最小值
        while self.stack:
            top_value = self.stack.pop()
            minimum = min(minimum, top_value)
            temp_stack.append(top_value)

        # 将元素重新放回原栈，恢复原来的顺序
        while temp_stack:
            self.stack.append(temp_stack.pop())

        return minimum
```

### 复杂度分析

- `push()`、`pop()`、`top()`：
  - 时间复杂度：\(O(1)\)
- `getMin()`：
  - 时间复杂度：\(O(n)\)
  - 临时空间复杂度：\(O(n)\)
- 栈本身的存储空间：\(O(n)\)

------

## 方法二：使用两个栈

### 思路

为了避免每次查询最小值时遍历整个栈，可以使用一个额外的栈 `min_stack`，记录主栈在每一层对应的最小值。

例如，依次压入：

```
主栈：     3    5    2    2    4
最小值栈： 3    3    2    2    2
```

两个栈的长度始终相同。`min_stack` 的栈顶就是当前主栈中的最小值。

因此：

- 压栈时，同时记录新的最小值。
- 出栈时，同时弹出两个栈的栈顶。
- 查询最小值时，直接返回 `min_stack` 的栈顶。

这是最直观、最容易在面试中正确实现的方案。

### 算法步骤

维护两个栈：

- `stack`：保存所有实际元素。
- `min_stack`：保存主栈每一层对应的最小值。

执行 `push(val)` 时：

1. 将 `val` 压入主栈。
2. 比较 `val` 和此前的最小值。
3. 将新的最小值压入 `min_stack`。

执行 `pop()` 时，同时弹出两个栈的栈顶，保证它们始终同步。

### 代码实现

```
class MinStack:
    def __init__(self):
        # 保存实际元素
        self.stack = []

        # min_stack[i] 表示 stack[0:i+1] 中的最小值
        self.min_stack = []

    def push(self, val: int) -> None:
        self.stack.append(val)

        if not self.min_stack:
            # 第一个元素本身就是当前最小值
            current_min = val
        else:
            # 新的最小值是 val 与原最小值中的较小者
            current_min = min(val, self.min_stack[-1])

        self.min_stack.append(current_min)

    def pop(self) -> None:
        # 两个栈必须同时弹出，保持长度和层级一致
        self.stack.pop()
        self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        # 辅助栈的栈顶始终是当前最小值
        return self.min_stack[-1]
```

### 正确性说明

对于主栈中的每一个位置 `i`，都有：

```
min_stack[i] = min(stack[0], stack[1], ..., stack[i])
```

因此，主栈当前所有元素的范围正好是 `stack[0:i+1]`，对应的最小值一定保存在 `min_stack` 的栈顶。

### 复杂度分析

- `push()`：时间复杂度 \(O(1)\)
- `pop()`：时间复杂度 \(O(1)\)
- `top()`：时间复杂度 \(O(1)\)
- `getMin()`：时间复杂度 \(O(1)\)
- 总空间复杂度：\(O(n)\)

虽然使用了两个栈，但渐进空间复杂度仍然是 \(O(n)\)。

------

## 方法三：一个栈保存差值

### 思路

该方法只使用一个栈，但栈中保存的不是元素本身，而是：

\[ \text{difference} = \text{val} - \text{currentMin} \]

其中 `currentMin` 是压入 `val` 之前的最小值。

另外使用变量 `current_min` 保存当前最小值。

差值的符号具有特殊含义：

- `difference > 0`：新元素大于当前最小值。
- `difference = 0`：新元素等于当前最小值，或者它是栈中的第一个元素。
- `difference < 0`：新元素小于原最小值，因此它成为了新的最小值。

当差值为负数时，它不仅表示当前元素是一个新的最小值，还可以用来恢复此前的最小值。

### 核心推导

假设压栈前的最小值为 `old_min`，压入的新值为 `val`：

\[ difference = val - oldMin \]

如果 `difference < 0`，说明：

\[ val < oldMin \]

因此，压栈后的新最小值为：

\[ currentMin = val \]

根据差值公式：

\[ difference = currentMin - oldMin \]

于是可以得到：

\[ oldMin = currentMin - difference \]

所以，当弹出一个负差值时，可以通过下面的公式恢复此前的最小值：

```
current_min = current_min - difference
```

### 操作分析

#### 压栈

如果栈为空：

- 压入差值 `0`。
- 将 `current_min` 设置为 `val`。

如果栈不为空：

- 压入 `val - current_min`。
- 如果差值小于 `0`，说明 `val` 是新的最小值，此时更新 `current_min = val`。

#### 出栈

- 弹出栈顶差值。
- 如果差值小于 `0`，说明被弹出的元素正是当前最小值。
- 使用差值恢复此前的最小值。

#### 获取栈顶

设栈顶差值为 `difference`：

- 如果 `difference > 0`，真实元素为：

  \[ currentMin + difference \]

- 如果 `difference <= 0`，栈顶元素就是当前最小值 `current_min`。

### 代码实现

```
class MinStack:
    def __init__(self):
        # 栈中保存“元素与压栈前最小值的差”
        self.stack = []

        # 保存当前栈中的最小值
        self.current_min = float("inf")

    def push(self, val: int) -> None:
        if not self.stack:
            # 第一个元素没有此前的最小值，约定保存差值 0
            self.stack.append(0)
            self.current_min = val
            return

        difference = val - self.current_min
        self.stack.append(difference)

        # 差值为负，说明 val 成为了新的最小值
        if difference < 0:
            self.current_min = val

    def pop(self) -> None:
        difference = self.stack.pop()

        if difference < 0:
            # 被弹出的元素是当前最小值
            # 恢复压入该元素之前的最小值
            self.current_min = self.current_min - difference

        if not self.stack:
            # 可选处理：栈空时重置最小值
            self.current_min = float("inf")

    def top(self) -> int:
        difference = self.stack[-1]

        if difference > 0:
            # 真实值 = 压栈时的最小值 + 差值
            # 此时最小值没有发生变化，所以仍是 current_min
            return self.current_min + difference

        # difference <= 0 表示栈顶元素等于当前最小值
        return self.current_min

    def getMin(self) -> int:
        return self.current_min
```

### 示例

依次执行：

```
push(5)
push(2)
push(4)
```

内部状态变化如下：

| 操作      | 保存的差值   | 差值栈       | 当前最小值 |
| --------- | ------------ | ------------ | ---------- |
| `push(5)` | `0`          | `[0]`        | `5`        |
| `push(2)` | `2 - 5 = -3` | `[0, -3]`    | `2`        |
| `push(4)` | `4 - 2 = 2`  | `[0, -3, 2]` | `2`        |

此时：

- `top()`：栈顶差值为 `2`，所以真实值是 `2 + 2 = 4`。
- `getMin()`：返回 `2`。

弹出 `4` 后，当前最小值仍为 `2`。

继续弹出 `2` 时，弹出的差值是 `-3`，因此恢复此前的最小值：

\[ 2 - (-3) = 5 \]

### 复杂度分析

- `push()`：时间复杂度 \(O(1)\)
- `pop()`：时间复杂度 \(O(1)\)
- `top()`：时间复杂度 \(O(1)\)
- `getMin()`：时间复杂度 \(O(1)\)
- 总空间复杂度：\(O(n)\)

该方法虽然只使用一个栈，但仍然需要为每个元素保存一个差值，因此渐进空间复杂度依然是 \(O(n)\)。相比双栈方案，它主要减少了常数级的额外空间。

------

## Python 相关说明

### 使用列表模拟栈

Python 没有必须专门使用的内置栈类型，通常直接使用 `list`：

```
stack = []

stack.append(value)  # 压栈，平均时间复杂度 O(1)
value = stack.pop()  # 出栈，时间复杂度 O(1)
value = stack[-1]    # 查看栈顶，时间复杂度 O(1)
```

应当避免使用下面的操作模拟栈的出栈：

```
stack.pop(0)
```

删除列表开头的元素需要移动后面的所有元素，时间复杂度为 \(O(n)\)。

### `float("inf")`

```
minimum = float("inf")
```

它表示正无穷，可以作为最小值变量的初始值。任意有限整数通常都小于正无穷。

不过在这道题中，只要保证栈非空时才调用 `getMin()`，也可以使用 `None` 表示当前不存在最小值。

### Python 中的整数溢出

在 Java、C++ 等使用固定宽度整数的语言中，差值：

```
val - current_min
```

可能发生整数溢出。例如，一个值接近整数最大值，另一个值接近整数最小值时，它们的差可能超出 `int` 的取值范围。

这些语言通常需要使用更大的整数类型，例如 Java 的 `long` 或 C++ 的 `long long`。

Python 的 `int` 支持任意精度整数，会根据数值大小自动扩展，因此通常不需要担心这种整数溢出。

------

## 常见错误

### 1. 两个栈没有保持同步

在双栈方案中，本实现为主栈的每个元素都在 `min_stack` 中保存一个对应的最小值。因此，每次压栈和出栈都必须同时操作两个栈。

错误示例：

```
def pop(self):
    self.stack.pop()
    # 忘记弹出 self.min_stack
```

这样会导致两个栈的层级错位，使 `getMin()` 返回错误结果。

### 2. 只在出现更小值时更新辅助栈，却没有处理重复最小值

另一种节省空间的写法是：只在新元素小于或等于当前最小值时，才将其压入 `min_stack`。

这里必须使用 `<=`，而不能只使用 `<`。例如：

```
push(2)
push(2)
pop()
```

弹出一个 `2` 后，栈中仍然存在另一个 `2`。如果辅助栈没有记录重复的最小值，就可能错误地丢失当前最小值。

正确实现如下：

```
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []

    def push(self, val: int) -> None:
        self.stack.append(val)

        # 必须包含相等的情况，以记录重复最小值
        if not self.min_stack or val <= self.min_stack[-1]:
            self.min_stack.append(val)

    def pop(self) -> None:
        value = self.stack.pop()

        # 只有弹出的元素等于当前最小值时，
        # 才同步弹出辅助栈
        if value == self.min_stack[-1]:
            self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]
```

该变体在最好的情况下可以减少辅助栈中的元素数量，但最坏空间复杂度仍为 \(O(n)\)。

### 3. 差值编码公式使用错误

对于单栈方案，需要记住两条核心公式：

```
保存的差值 = 新元素 - 原最小值
恢复原最小值 = 当前最小值 - 保存的差值
```

当保存的差值为负数时，才表示最小值发生过变化。

### 4. 空栈操作

通常题目会保证不会对空栈调用 `pop()`、`top()` 或 `getMin()`。

如果在实际项目中实现该数据结构，则应主动处理空栈情况，例如抛出异常：

```
if not self.stack:
    raise IndexError("MinStack is empty")
```

------

## 方法对比

| 方法           | `push`   | `pop`    | `top`    | `getMin` | 空间复杂度 | 特点                             |
| -------------- | -------- | -------- | -------- | -------- | ---------- | -------------------------------- |
| 暴力查找       | \(O(1)\) | \(O(1)\) | \(O(1)\) | \(O(n)\) | \(O(n)\)   | 简单，但不满足题目要求           |
| 两个栈         | \(O(1)\) | \(O(1)\) | \(O(1)\) | \(O(1)\) | \(O(n)\)   | 最直观、最容易正确实现           |
| 一个栈保存差值 | \(O(1)\) | \(O(1)\) | \(O(1)\) | \(O(1)\) | \(O(n)\)   | 常数空间更小，但推导和实现更复杂 |

面试中通常优先推荐**双栈方案**：逻辑清楚、容易证明，也不容易出错。差值编码方案可以作为进一步的空间优化思路。