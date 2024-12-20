# 在 C# 中堆和优先队列的强大功能：从基础到 LeetCode 精通

## 1. 堆与优先队列的定义

**堆的定义：**  
**堆**是一种特殊的基于树的数据结构，满足**堆属性**。堆分为两种类型：

- **最小堆（Min-Heap）：** 每个节点的值小于或等于其子节点的值。根节点包含最小的元素。
- **最大堆（Max-Heap）：** 每个节点的值大于或等于其子节点的值。根节点包含最大的元素。

**优先队列的定义：**  
**优先队列**是一种抽象数据类型，类似于普通队列，但每个元素都有一个优先级。元素按优先级而非插入顺序出队。堆是最常见的实现方式，因为它在维护顺序方面非常高效。

**关键属性：**

- **插入：** 添加元素并维护堆属性。
- **删除：** 移除根节点（最高或最低优先级）并重新调整堆。
- **堆属性：** 对于索引为 `i` 的任意节点：
  - 父节点索引：`(i - 1) / 2`
  - 左子节点索引：`2 * i + 1`
  - 右子节点索引：`2 * i + 2`

* * *

## 2. 何时以及为何使用堆/优先队列

**优点：**

- **高效访问：** 提供 O(1) 时间访问最小或最大元素。
- **插入与删除：** 这两种操作的时间复杂度都是 O(log n)，比维护一个已排序的列表或数组更快。
- **内存高效：** 除了数组存储外，不需要额外的空间。

**比较：**

- **已排序列表：**
  - **插入：** O(n)，因为需要将元素放在正确位置。
  - **访问最小/最大值：** O(1)，可以直接访问头部或尾部。

- **堆：**
  - **插入：** O(log n)，每次插入都会维护堆属性。
  - **访问最小/最大值：** O(1)，无论堆的大小如何，都可以直接访问根节点。

**使用场景：**

- **Dijkstra 算法：** 在图中寻找最短路径。
- **任务调度：** 在 CPU 调度中管理带优先级的任务。
- **中位数查找：** 快速找出动态数据流中的中位数。

* * *

## 3. 在 C# 中实现堆

**内置的最小堆和最大堆支持**: 在 .NET 6 及更高版本中，`PriorityQueue<TElement, TPriority>` 类提供了开箱即用的优先队列。默认情况下，它是一个**最小堆**，但可以通过自定义比较器配置为**最大堆**。

### **1. 最小堆示例（默认行为）**

在最小堆中，优先级最低的元素会先被出队：

```csharp
var minHeap = new PriorityQueue<string, int>();

minHeap.Enqueue("任务 1", 3); // 优先级 3
minHeap.Enqueue("任务 2", 1); // 优先级 1
minHeap.Enqueue("任务 3", 2); // 优先级 2

while (minHeap.Count > 0)
{
    Console.WriteLine(minHeap.Dequeue());
}
// 输出: 任务 2, 任务 3, 任务 1（按优先级升序）
```

### **2. 最大堆示例（自定义比较器）**

在最大堆中，优先级最高的元素会先被出队。使用自定义比较器来反转默认行为：

```csharp
var maxHeap = new PriorityQueue<string, int>(Comparer<int>.Create((x, y) => y.CompareTo(x)));

maxHeap.Enqueue("任务 1", 3); // 优先级 3
maxHeap.Enqueue("任务 2", 1); // 优先级 1
maxHeap.Enqueue("任务 3", 2); // 优先级 2

while (maxHeap.Count > 0)
{
    Console.WriteLine(maxHeap.Dequeue());
}
// 输出: 任务 1, 任务 3, 任务 2（按优先级降序）
```

### **解释：**

- **最小堆:** 默认的 `PriorityQueue` 行为是先出队**最低优先级**的元素。
- **最大堆:** 通过提供一个反转比较的自定义比较器（`y.CompareTo(x)`），可以实现先出队**最高优先级**的元素。

这种设置允许您根据问题的需求在最小堆和最大堆之间轻松切换。

* * *

## 4. LeetCode 题目与解决方案

### **问题 703: 数据流中的第 K 大元素（简单）**

**描述:**  
设计一个类来找到数据流中的**第 K 大元素**。类应始终维护第 K 大元素。

**为什么使用堆？**  
使用大小为 `k` 的最小堆最为理想，因为堆顶始终保持第 K 大值。

**解决方案思路：**

- 维护一个大小为 `k` 的最小堆。
- 当新元素被添加时：
  - 如果堆中元素少于 `k` 个，直接插入。
  - 如果堆已满，且新元素大于堆顶元素，则插入新元素并调整堆。

**C# 实现：**

```csharp
public class KthLargest
{
    private PriorityQueue<int, int> minHeap;
    private int k;

    public KthLargest(int k, int[] nums)
    {
        this.k = k;
        minHeap = new PriorityQueue<int, int>();

        foreach (int num in nums)
            Add(num);
    }

    public int Add(int val)
    {
        if (minHeap.Count < k)
            minHeap.Enqueue(val, val);
        else if (val > minHeap.Peek())
        {
            minHeap.Dequeue();
            minHeap.Enqueue(val, val);
        }

        return minHeap.Peek();
    }
}
```

**替代方案:**  
使用排序列表插入的时间复杂度为 O(n log n)，而堆方法的时间复杂度为 O(log k)。

* * *

### **问题 347: 前 K 个高频元素（中等）**

**描述:**  
给定一个非空整数数组，返回**前 K 个出现频率最高的元素**。

**为什么使用堆？**  
最小堆可以高效地跟踪前 `k` 个高频元素，无需对整个数组排序。

**解决方案思路：**

- 使用字典统计频率。
- 使用最小堆存储前 `k` 个基于频率的元素。

**C# 实现：**

```csharp
public int[] TopKFrequent(int[] nums, int k)
{
    var frequencyMap = new Dictionary<int, int>();
    foreach (var num in nums)
        frequencyMap[num] = frequencyMap.GetValueOrDefault(num, 0) + 1;

    var minHeap = new PriorityQueue<int, int>();

    foreach (var entry in frequencyMap)
    {
        minHeap.Enqueue(entry.Key, entry.Value);
        if (minHeap.Count > k)
            minHeap.Dequeue();
    }

    return minHeap.UnorderedItems.Select(x => x.Element).ToArray();
}
```

**替代方案:**  
对频率映射排序的时间复杂度为 O(n log n)，而堆方法的时间复杂度为 O(n log k)。

* * *

### **问题 215: 数组中的第 K 大元素（中等）**

**描述:**  
在未排序的数组中找到第 `k` 大的元素。

**为什么使用堆？**  
大小为 `k` 的最小堆可以高效地维护第 `k` 大元素。

**解决方案思路：**

- 用前 `k` 个元素构建一个最小堆。
- 遍历剩余的数组元素，维护堆的大小。

**C# 实现：**

```csharp
public int FindKthLargest(int[] nums, int k)
{
    var minHeap = new PriorityQueue<int, int>();

    foreach (var num in nums)
    {
        minHeap.Enqueue(num, num);
        if (minHeap.Count > k)
            minHeap.Dequeue();
    }

    return minHeap.Peek();
}
```

**替代方案:**  
对数组排序的时间复杂度为 O(n log n)，而堆方法的时间复杂度为 O(n log k)。

* * *

### **问题 295: 数据流的中位数（困难）**

**描述:**  
设计一个数据结构，支持添加整数并找出当前数据流的**中位数**。

**为什么使用两个堆？**

- 使用**最大堆**存储左半部分（较小的元素），使用**最小堆**存储右半部分（较大的元素）。
- 平衡这两个堆可以高效地提取中位数。

**C# 实现：**

```csharp
public class MedianFinder
{
    private PriorityQueue<int, int> maxHeap;  // 左半部分
    private PriorityQueue<int, int> minHeap;  // 右半部分

    public MedianFinder()
    {
        maxHeap = new PriorityQueue<int, int>(Comparer<int>.Create((x, y) => y.CompareTo(x)));
        minHeap = new PriorityQueue<int, int>();
    }

    public void AddNum(int num)
    {
        if (maxHeap.Count == 0 || num <= maxHeap.Peek())
            maxHeap.Enqueue(num, num);
        else
            minHeap.Enqueue(num, num);

        // 平衡堆
        if (maxHeap.Count > minHeap.Count + 1)
            minHeap.Enqueue(maxHeap.Dequeue(), maxHeap.Dequeue());
        else if (minHeap.Count > maxHeap.Count)
            maxHeap.Enqueue(minHeap.Dequeue(), minHeap.Dequeue());
    }

    public double FindMedian()
    {
        if (maxHeap.Count == minHeap.Count)
            return (maxHeap.Peek() + minHeap.Peek()) / 2.0;
        else
            return maxHeap.Peek();
    }
}
```

**替代方案:**  
对整个数据流排序的时间复杂度为 O(n log n)，而双堆方法的插入时间复杂度为 O(log n)，中位数检索时间复杂度为 O(1)。

* * *

## 5. 实际应用场景

### **任务调度（CPU/GPU）：**

操作系统和任务管理器利用优先队列来管理进程调度。具有较高优先级的任务会先于较低优先级的任务执行，从而确保高效分配CPU时间。

### **Dijkstra最短路径算法：**

优先队列在Dijkstra算法中至关重要，用于确定从源节点到图中所有其他节点的最短路径。该算法使用最小堆反复提取下一个最近的节点。

### **事件模拟系统：**

在模拟系统中（例如，游戏中的A*寻路算法或物流应用程序），优先队列用于根据执行时间或重要性调度和管理事件。例如：

- **A*算法：** 使用优先队列首先探索最有希望的路径，基于总成本（已行进距离 + 估算到目标的距离）。

### **行业使用案例：**

科技公司在多个场景中使用优先队列：

- **负载均衡：** 在分布式系统中，任务根据服务器当前的负载或处理能力进行分配。使用优先队列确保高效分布任务。
- **网络通信：** 路由器根据数据包的重要性进行优先处理（例如，语音数据优先于后台下载数据）。

* * *

## 6. 技巧与常见陷阱

### **处理重复元素：**

- 优先队列允许重复优先级，但可能会影响出队顺序。
- **技巧：** 可以添加时间戳或唯一ID来区分具有相同优先级的任务。

### **理解堆操作：**

- **向上堆化（Heapify Up）：** 在添加新元素时维护堆结构。
- **向下堆化（Heapify Down）：** 在移除元素后恢复堆的结构。
- **技巧：** .NET中的`PriorityQueue`内部会自动处理这些操作，但理解它们有助于调试自定义实现。

### **选择合适的数据结构：**

- **最小堆 vs. 最大堆：** 选择取决于是否频繁需要最小值或最大值。
- **PriorityQueue vs. SortedList：** 当只需频繁获取最高优先级元素时，使用`PriorityQueue`；若需要遍历整个列表，`SortedList`更合适。

### **性能考量：**

- 避免插入过高优先级的元素，因为这可能降低性能。
- 确保比较器逻辑一致，以防自定义堆出现意外行为。

* * *

## 7. 总结

堆和优先队列是强大的数据结构，可为各种问题提供高效解决方案，从任务调度到最短路径算法。在C#中，`PriorityQueue`类提供对最小堆和最大堆的内置支持，简化了它们的实现。通过掌握这些概念并将其应用于实际问题，开发人员可以显著提升应用程序的效率和性能。

无论是解决LeetCode问题还是设计复杂系统，理解何时以及如何使用优先队列都将成为编程工具箱中不可或缺的一部分。
