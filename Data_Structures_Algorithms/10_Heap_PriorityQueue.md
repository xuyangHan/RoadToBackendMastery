# The Power of Heaps and Priority Queues in C#: From Basics to LeetCode Mastery

## 1. Definition of Heaps and Priority Queues

**Heap Definition:**  
A **heap** is a special tree-based data structure that satisfies the **heap property**. There are two types of heaps:

- **Min-Heap:** Each node's value is less than or equal to the values of its children. The root node contains the smallest element.
- **Max-Heap:** Each node's value is greater than or equal to the values of its children. The root node contains the largest element.

**Priority Queue Definition:**  
A **priority queue** is an abstract data type similar to a regular queue, but each element has a priority. Elements are dequeued based on priority rather than insertion order. A heap is the most common implementation because it efficiently maintains the order.

**Key Properties:**

- **Insertion:** Adding an element while maintaining the heap property.
- **Deletion:** Removing the root node (highest or lowest priority) and re-adjusting the heap.
- **Heap Property:** For any node at index `i`:
  - Parent node index: `(i - 1) / 2`
  - Left child index: `2 * i + 1`
  - Right child index: `2 * i + 2`

* * *

## 2. When and Why to Use Heaps/Priority Queues

**Advantages:**

- **Efficient Access:** Provides O(1) time complexity to access the minimum or maximum element.
- **Insertion and Deletion:** Both operations have a time complexity of O(log n), which is faster than maintaining a sorted list or array.
- **Memory Efficient:** No additional space is required apart from array storage.

**Comparison:**

- **Sorted List:**
  - **Insertion:** O(n) because elements must be placed in the correct position.
  - **Access Min/Max:** O(1), as the first or last element is directly accessible.

- **Heap:**
  - **Insertion:** O(log n), as every insertion maintains the heap property.
  - **Access Min/Max:** O(1), as the root node is always accessible.

**Use Cases:**

- **Dijkstra's Algorithm:** Finding the shortest path in a graph.
- **Task Scheduling:** Managing tasks with priorities in CPU scheduling.
- **Median Finding:** Quickly finding the median in a dynamic data stream.

* * *

## 3. Implementing Heaps in Csharp

**Built-in Min-Heap and Max-Heap Support:** In .NET 6 and later, the `PriorityQueue<TElement, TPriority>` class provides out-of-the-box priority queue support. By default, it is a **min-heap**, but it can be configured as a **max-heap** by providing a custom comparer.

### **1. Min-Heap Example (Default Behavior)**

In a min-heap, the element with the lowest priority will be dequeued first:

```csharp
var minHeap = new PriorityQueue<string, int>();

minHeap.Enqueue("Task 1", 3); // Priority 3
minHeap.Enqueue("Task 2", 1); // Priority 1
minHeap.Enqueue("Task 3", 2); // Priority 2

while (minHeap.Count > 0)
{
    Console.WriteLine(minHeap.Dequeue());
}
// Output: Task 2, Task 3, Task 1 (in ascending priority order)
```

### **2. Max-Heap Example (Custom Comparer)**

In a max-heap, the element with the highest priority will be dequeued first. Use a custom comparer to reverse the default behavior:

```csharp
var maxHeap = new PriorityQueue<string, int>(Comparer<int>.Create((x, y) => y.CompareTo(x)));

maxHeap.Enqueue("Task 1", 3); // Priority 3
maxHeap.Enqueue("Task 2", 1); // Priority 1
maxHeap.Enqueue("Task 3", 2); // Priority 2

while (maxHeap.Count > 0)
{
    Console.WriteLine(maxHeap.Dequeue());
}
// Output: Task 1, Task 3, Task 2 (in descending priority order)
```

### **Explanation:**

- **Min-Heap:** The default behavior of `PriorityQueue` dequeues elements with **the lowest priority** first.
- **Max-Heap:** By providing a custom comparer (reversing the comparison `y.CompareTo(x)`), we can dequeue elements with **the highest priority** first.

This setup allows you to easily switch between min-heaps and max-heaps based on the problem's requirements.

* * *

## 4. LeetCode Problems and Solutions

### **Problem 703: Kth Largest Element in a Stream (Easy)**

**Description:**  
Design a class to find the **Kth largest element** in a data stream. The class should always maintain the Kth largest element.

**Why Use a Heap?**  
Using a min-heap of size `k` is ideal, as the heap's root always holds the Kth largest value.

**Solution Approach:**

- Maintain a min-heap of size `k`.
- When a new element is added:
  - If the heap has fewer than `k` elements, insert the new element.
  - If the heap is full and the new element is larger than the root, insert the new element and adjust the heap.

**C# Implementation:**

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

**Alternative Solution:**  
Using a sorted list would have a time complexity of O(n log n) for insertion, while the heap approach has a time complexity of O(log k).

* * *

### **Problem 347: Top K Frequent Elements (Medium)**

**Description:**  
Given a non-empty integer array, return the **K most frequent elements**.

**Why Use a Heap?**  
A min-heap can efficiently track the top `k` frequent elements without the need to sort the entire array.

**Solution Approach:**

- Use a dictionary to count frequencies.
- Use a min-heap to store the top `k` elements based on frequency.

**C# Implementation:**

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

**Alternative Solution:**  
Sorting the frequency map would have a time complexity of O(n log n), while the heap solution has a time complexity of O(n log k).

### **Problem 215: Kth Largest Element in an Array (Medium)**

**Description:**  
Find the `k`th largest element in an unsorted array.

**Why Use a Heap?**  
A min-heap of size `k` can efficiently maintain the `k`th largest element.

**Solution Approach:**

- Build a min-heap with the first `k` elements.
- Traverse the remaining array elements, maintaining the size of the heap.

**C# Implementation:**

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

**Alternative Solution:**  
Sorting the array has a time complexity of O(n log n), while the heap solution has a time complexity of O(n log k).

* * *

### **Problem 295: Find Median from Data Stream (Hard)**

**Description:**  
Design a data structure that supports adding integers and finding the **median** of the current data stream.

**Why Use Two Heaps?**

- Use a **max-heap** to store the left half (smaller elements) and a **min-heap** to store the right half (larger elements).
- Balancing these two heaps allows for efficient median extraction.

**C# Implementation:**

```csharp
public class MedianFinder
{
    private PriorityQueue<int, int> maxHeap;  // Left half
    private PriorityQueue<int, int> minHeap;  // Right half

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

        // Balance the heaps
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

**Alternative Solution:**  
Sorting the entire data stream has a time complexity of O(n log n), while the double-heap method has an insertion time complexity of O(log n) and median retrieval time complexity of O(1).

* * *

## 5. Real-world Applications

### **Task Scheduling (CPU/GPU):**

Operating systems and task managers use priority queues to manage process scheduling. Tasks with higher priority are executed before tasks with lower priority, ensuring efficient CPU time allocation.

### **Dijkstra's Shortest Path Algorithm:**

Priority queues are critical in Dijkstra's algorithm, used to determine the shortest path from the source node to all other nodes in a graph. The algorithm repeatedly extracts the next nearest node using a min-heap.

### **Event Simulation Systems:**

In simulation systems (e.g., A* pathfinding algorithm in games or logistics applications), priority queues are used to schedule and manage events based on execution time or importance. For example:

- **A* Algorithm:** Uses a priority queue to explore the most promising path first, based on the total cost (distance traveled + estimated distance to the goal).

### **Industry Use Cases:**

Tech companies use priority queues in various scenarios:

- **Load Balancing:** In distributed systems, tasks are allocated based on the current load or processing capability of servers. Priority queues ensure efficient distribution of tasks.
- **Network Communication:** Routers prioritize data packets based on their importance (e.g., voice data takes priority over background downloads).

* * *

## 6. Tips and Common Pitfalls

### **Handling Duplicate Elements:**

- Priority queues allow duplicate priorities, but this may affect the dequeue order.
- **Tip:** You can add a timestamp or unique ID to differentiate tasks with the same priority.

### **Understanding Heap Operations:**

- **Heapify Up:** Maintains the heap structure when adding a new element.
- **Heapify Down:** Restores the heap structure when removing an element.
- **Tip:** In .NET, `PriorityQueue` automatically handles these operations, but understanding them helps debug custom implementations.

### **Choosing the Right Data Structure:**

- **Min-Heap vs. Max-Heap:** Choose based on whether you frequently need the minimum or maximum value.
- **PriorityQueue vs. SortedList:** Use `PriorityQueue` when you need frequent access to the highest priority element; if you need to traverse the entire list, `SortedList` is more suitable.

### **Performance Considerations:**

- Avoid inserting overly high-priority elements, as this can degrade performance.
- Ensure the comparator logic is consistent to prevent unexpected behavior in custom heaps.

* * *

## 7. Conclusion

Heaps and priority queues are powerful data structures that provide efficient solutions for various problems, from task scheduling to shortest path algorithms. In C#, the `PriorityQueue` class provides built-in support for both min-heaps and max-heaps, simplifying their implementation. By mastering these concepts and applying them to real-world problems, developers can significantly improve the efficiency and performance of their applications.

Whether solving LeetCode problems or designing complex systems, understanding when and how to use priority queues will become an indispensable part of your programming toolkit.
