# Optimizing Code Efficiency in C# Using Stacks and Queues: Practical Applications and LeetCode Problems

When writing efficient code, **Stack** and **Queue** are two versatile and powerful data structures. Understanding their working principles, time complexity, and use cases can significantly improve the way you solve problems in C#. Not only are stacks and queues crucial for algorithm challenges, but they also form the foundation of many real-world systems, such as message processing and undo-redo operations.

In this blog, we will explore how to use stacks and queues effectively in C#, discuss their real-world applications, and review some common **LeetCode problems** where these structures play a key role.

---

## **Practical Applications of Stack and Queue in C#**

### **Using Queues in Real-World Applications**

A **Queue** follows the First-In-First-Out (FIFO) principle, meaning the first element added is processed first. It is often used in scenarios where tasks need to be processed in the order they arrive.

- **Message Processing (e.g., MQTT Messages):**  
  In distributed systems, message brokers like **MQTT** use queues to manage messages. When multiple clients publish messages, they are queued and processed in order, ensuring reliability and sequence. In high-traffic environments, queues ensure that no messages are lost and all are processed. When the queue is empty, the system waits for new messages to arrive.

  ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;

    class MessageProcessor
    {
        private static readonly Queue<string> messageQueue = new Queue<string>();
        private static readonly object lockObject = new object();

        static void Main(string[] args)
        {
            // Start a thread to simulate message processing
            Thread messageProcessorThread = new Thread(ProcessMessages);
            messageProcessorThread.Start();

            // Simulate incoming messages
            EnqueueMessage("Message 1");
            Thread.Sleep(1000); // Simulate delay
            EnqueueMessage("Message 2");
            Thread.Sleep(1000); // Simulate delay
            EnqueueMessage("Message 3");
        }

        // Method to enqueue messages
        public static void EnqueueMessage(string message)
        {
            lock (lockObject)
            {
                messageQueue.Enqueue(message);
                Monitor.Pulse(lockObject); // Notify the processing thread about new messages
            }
        }

        // Method to process messages
        public static void ProcessMessages()
        {
            while (true)
            {
                string message = null;

                lock (lockObject)
                {
                    // Wait for messages in the queue
                    while (messageQueue.Count == 0)
                    {
                        Monitor.Wait(lockObject); // Wait for messages to be enqueued
                    }

                    message = messageQueue.Dequeue();
                }

                // Simulate message processing
                ProcessMessage(message);
            }
        }

        // Simulated message processing
        public static void ProcessMessage(string message)
        {
            Console.WriteLine($"Processing {message}");
            Thread.Sleep(500); // Simulate processing delay
        }
    }
  ```

- **Task Scheduling (Thread Pool and Queues):**  
  In real-world systems, task scheduling is critical, especially when managing multiple tasks concurrently. A thread pool provides a robust solution by reusing a set of worker threads instead of creating and destroying threads on demand. Internally, the thread pool uses a queue to manage tasks, ensuring the system doesn’t overload by creating too many threads.  

  When tasks are submitted to a thread pool, they are enqueued if no threads are available. This queue ensures tasks are processed in a First-In-First-Out manner.

  ```csharp
  using System;
  using System.Threading;

  class ThreadPoolQueueExample
  {
      static void Main()
      {
          // Queue some tasks for the thread pool
          ThreadPool.QueueUserWorkItem(ProcessTask, "Task 1");
          ThreadPool.QueueUserWorkItem(ProcessTask, "Task 2");
          ThreadPool.QueueUserWorkItem(ProcessTask, "Task 3");

          // Allow enough time for all tasks to complete
          Thread.Sleep(3000);
      }

      // Simulated task processing method
      static void ProcessTask(object taskName)
      {
          string task = (string)taskName;
          Console.WriteLine($"{task} is being processed.");
          Thread.Sleep(1000); // Simulate task processing delay
          Console.WriteLine($"{task} has completed.");
      }
  }
  ```

### **Using Stacks in Real-World Applications**

A **Stack** follows the Last-In-First-Out (LIFO) principle, making it ideal for scenarios where the last operation needs to be undone first.

- **Undo/Redo Functionality in Applications:**  
  Applications like text editors or graphic editors use stacks to track user actions. When a user presses undo, the last action is popped from the stack and undone.

  ```csharp
  Stack<string> actionStack = new Stack<string>();
  actionStack.Push("Typed 'Hello'");
  actionStack.Push("Bolded text");
  
  // Undo the last action
  string lastAction = actionStack.Pop(); // Removes "Bolded text"
  UndoAction(lastAction);
  ```

- **Web Browser Navigation:**  
  Every time you navigate to a new page, the browser pushes the current page onto a stack. When you press the back button, the last page is popped from the stack and displayed.

- **Backtracking Algorithms (e.g., Mazes, Puzzles):**  
  Stacks are essential in backtracking algorithms where all possible paths are explored. If a path leads to a dead end, you backtrack to the previous point using the stack.

---

## **Algorithm Problems Solved with Stacks and Queues**

In algorithm challenges, stacks and queues are often used in traversal problems, especially in **Depth-First Search (DFS)** and **Breadth-First Search (BFS).**

- **DFS with Stack:**  
  In DFS, you explore as far as possible along one path before backtracking. A stack helps record the current path. When exploring a node, push it onto the stack, and when done, pop it off.

  ```csharp
  Stack<TreeNode> stack = new Stack<TreeNode>();
  stack.Push(root);
  
  while (stack.Count > 0)
  {
      TreeNode node = stack.Pop();
      Process(node);
      
      // Push the right child first so the left child is processed next (LIFO order)
      if (node.Right != null) stack.Push(node.Right);
      if (node.Left != null) stack.Push(node.Left);
  }
  ```

- **BFS with Queue:**  
  BFS uses a queue to explore nodes level by level, making it suitable for problems where systematic exploration is needed, such as shortest path algorithms.

  ```csharp
  Queue<TreeNode> queue = new Queue<TreeNode>();
  queue.Enqueue(root);
  
  while (queue.Count > 0)
  {
      TreeNode node = queue.Dequeue();
      Process(node);
      
      if (node.Left != null) queue.Enqueue(node.Left);
      if (node.Right != null) queue.Enqueue(node.Right);
  }
  ```

In both DFS and BFS, using stacks or queues makes traversal straightforward and efficient.

---

## **Performance and Efficiency: Stack/Queue vs List/Dictionary**

One of the primary advantages of using **Stack** or **Queue** is their constant time **(O(1))** complexity for adding and removing elements. Unlike a list, where inserting or removing elements at the beginning can be \(O(n)\), stack and queue operations remain efficient.

- **Stack vs List (Push/Pop vs Insert/Remove):**  
  In C#, pushing elements to a stack or queue is faster than inserting them at a specific position in a list. Lists may involve element shifting, adding extra overhead.

- **Dictionary vs Queue in Key-Value Retrieval:**  
  While dictionaries provide \(O(1)\) key-value lookup, they are not suitable for ordered or sequential processing. Queues excel in processing elements in order, maintaining the sequence of operations.

---

## **LeetCode Problems Demonstrating Applications of Stacks and Queues**

For those preparing for coding interviews, here are some **LeetCode** problems that can be efficiently solved using stacks and queues. Each problem comes with detailed examples and explanations to showcase the power of these data structures.

---

### **Stack-Based Problems and Examples**

Stacks are a powerful data structure, especially for algorithms involving nested or sequential operations. Below are two important problems that highlight the utility of stacks.

#### **1. [LeetCode 20: Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)**

This problem asks you to determine if a string containing parentheses is valid. A valid string means every opening parenthesis has a corresponding closing parenthesis in the correct order. Stacks are ideal for this problem because of their **Last-In-First-Out (LIFO)** property, which aligns with how parentheses are closed.

##### **20 Solution**

```csharp
public class Solution {
    public bool IsValid(string s) {
        Stack<char> stack = new Stack<char>();
        Dictionary<char, char> matchingPairs = new Dictionary<char, char> {
            { ')', '(' },
            { '}', '{' },
            { ']', '[' }
        };
        
        foreach (char c in s) {
            if (matchingPairs.ContainsValue(c)) {
                stack.Push(c);
            } else if (matchingPairs.ContainsKey(c)) {
                if (stack.Count == 0 || stack.Pop() != matchingPairs[c]) {
                    return false;
                }
            }
        }
        return stack.Count == 0;
    }
}
```

##### **20 Explanation**

- A stack is used to store opening parentheses.
- When a closing parenthesis is encountered, the program checks if it matches the top of the stack. If not or if the stack is empty, the string is invalid.
- The time complexity is **O(n)**, as the string is traversed once, and stack operations (push/pop) are constant time.

---

#### **2. [LeetCode 84: Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/)**

This problem requires finding the largest rectangle that can be formed in a histogram. A stack helps efficiently calculate the area by keeping track of indices and calculating the area whenever a smaller height is encountered.

##### **84 Solution**

```csharp
public class Solution {
    public int LargestRectangleArea(int[] heights) {
        Stack<int> stack = new Stack<int>();
        int maxArea = 0;
        int n = heights.Length;

        for (int i = 0; i < n; i++) {
            while (stack.Count > 0 && heights[i] < heights[stack.Peek()]) {
                int height = heights[stack.Pop()];
                int width = stack.Count > 0 ? i - stack.Peek() - 1 : i;
                maxArea = Math.Max(maxArea, height * width);
            }
            stack.Push(i);
        }

        while (stack.Count > 0) {
            int height = heights[stack.Pop()];
            int width = stack.Count > 0 ? n - stack.Peek() - 1 : n;
            maxArea = Math.Max(maxArea, height * width);
        }

        return maxArea;
    }
}
```

##### **84 Explanation**

- The stack stores the indices of histogram bars in increasing order of height.
- When encountering a bar shorter than the stack's top, calculate the area of rectangles using the popped heights.
- This solution has a time complexity of **O(n)** because each bar is pushed and popped from the stack at most once.

---

### **Queue-Based Problems and Examples**

Queues are another critical data structure, often used for problems involving sequential processing. The **First-In-First-Out (FIFO)** property makes them suitable for breadth-first search (BFS) and related problems.

#### **1. [LeetCode 207: Course Schedule](https://leetcode.com/problems/course-schedule/)**

This problem asks if it's possible to complete all courses given their prerequisites. It’s a classic **topological sorting** problem that can be solved efficiently using a queue.

##### **207 Solution**

```csharp
public class Solution {
    public bool CanFinish(int numCourses, int[][] prerequisites) {
        List<int>[] graph = new List<int>[numCourses];
        int[] inDegree = new int[numCourses];
        Queue<int> queue = new Queue<int>();
        
        for (int i = 0; i < numCourses; i++) {
            graph[i] = new List<int>();
        }

        // Build the graph and calculate in-degrees
        foreach (var pair in prerequisites) {
            graph[pair[1]].Add(pair[0]);
            inDegree[pair[0]]++;
        }

        // Enqueue courses with no prerequisites (in-degree 0)
        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) {
                queue.Enqueue(i);
            }
        }

        int completedCourses = 0;

        while (queue.Count > 0) {
            int course = queue.Dequeue();
            completedCourses++;
            
            foreach (int nextCourse in graph[course]) {
                inDegree[nextCourse]--;
                if (inDegree[nextCourse] == 0) {
                    queue.Enqueue(nextCourse);
                }
            }
        }

        return completedCourses == numCourses;
    }
}
```

##### **207 Explanation**

- A queue is used to perform topological sorting on the graph of courses.
- Courses with no prerequisites are processed first. For every course completed, the in-degrees of its dependent courses are reduced.
- If all courses can be processed, return `true`. The time complexity is **O(V + E)**, where `V` is the number of courses and `E` is the number of prerequisites.

---

### **Additional LeetCode Problems to Practice Stacks and Queues**

#### **Stack-Related Problems**

1. [LeetCode 32: Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/)
2. [LeetCode 155: Min Stack](https://leetcode.com/problems/min-stack/)
3. [LeetCode 316: Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/)
4. [LeetCode 394: Decode String](https://leetcode.com/problems/decode-string/)

#### **Queue-Related Problems**

1. [LeetCode 933: Number of Recent Calls](https://leetcode.com/problems/number-of-recent-calls/)
2. [LeetCode 279: Perfect Squares](https://leetcode.com/problems/perfect-squares/)
3. [LeetCode 542: 01 Matrix](https://leetcode.com/problems/01-matrix/)

These problems are excellent for deepening your understanding of how stacks and queues can be applied in real-world scenarios and coding challenges.

---

## **Conclusion**

Stacks and queues are indispensable tools in C# programming, providing efficient solutions for both real-world systems and algorithmic challenges. Whether processing MQTT messages, implementing undo functionality, or traversing graphs with BFS and DFS, these data structures offer significant advantages.

By practicing stack- and queue-related problems on LeetCode, you can not only become a better programmer but also prepare thoroughly for coding interviews. Keep exploring and mastering the versatility of stacks and queues!
