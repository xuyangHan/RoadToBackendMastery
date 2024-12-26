# Mastering DFS and BFS in C#: Techniques, Implementation, and LeetCode Examples

In this blog, we will delve into two fundamental graph traversal algorithms: Depth-First Search (DFS) and Breadth-First Search (BFS). These algorithms are key tools for solving many programming problems, particularly those involving traversal or searching in data structures like trees and graphs. DFS and BFS each have unique ways of exploring nodes and edges, making them suitable for different types of problems, from finding the shortest path in a maze to computing connected components in a network.

We’ll cover the core concepts behind DFS and BFS, how they work, and when to prioritize one over the other. Additionally, we will implement these algorithms in C# and explore their performance in practical applications like geometry and pathfinding. We’ll also examine some common LeetCode problems to demonstrate how DFS and BFS effectively solve challenges like matrix traversal, finding islands in grids, and solving puzzles. Understanding how these algorithms work and when to use them will enhance your problem-solving skills and code efficiency.

---

## What are DFS and BFS?

**Depth-First Search (DFS)** dives as deep as possible down one branch of a tree or graph before backtracking. It explores one branch first, then tries other branches, making it suitable for problems requiring exhaustive path exploration.

**Breadth-First Search (BFS)** explores all nodes at the current depth level before moving to the next level. It’s ideal for problems requiring the shortest path between two nodes.

### DFS vs. BFS

| DFS                       | BFS                      |
| ------------------------- | ------------------------ |
| Uses a stack (implicit or explicit) | Uses a queue                |
| Suitable for exhaustive search or puzzle-solving | Suitable for shortest-path problems |
| May use less memory for narrow trees/graphs | May use more memory for wide graphs |

---

## How Do They Work?

### Depth-First Search (DFS)

DFS starts from a node, explores as far as possible along each branch, and then backtracks. There are two common implementations of DFS: recursive and iterative (using an explicit stack).

#### Steps for DFS

1. Start from a root or any node.
2. Visit the node.
3. Recursively or iteratively visit all unvisited neighbors.
4. Backtrack if there are no remaining neighbors.

### Breadth-First Search (BFS)

BFS starts from a node, explores all neighbors at the current level, and then moves to the next level. A queue is typically used to track the next nodes to visit.

#### Steps for BFS

1. Start from a root or any node.
2. Visit the node and add all its neighbors to a queue.
3. Dequeue a node, visit it, and repeat for all its neighbors.

---

## When to Use DFS and BFS?

- **Use DFS** when you need to:
  - Explore all possible paths, such as solving a maze or backtracking problems.
  - Solve tree traversal problems like post-order or in-order traversal.
  - Handle problems requiring deep exploration before moving to other paths (e.g., finding connected components in a graph).

- **Use BFS** when you need to:
  - Find the shortest path in an unweighted graph (e.g., shortest path problems like **Word Ladder**).
  - Perform level-order traversal of a tree or graph, making it suitable for problems like **zigzag level-order traversal**.

---

## Implementing DFS and BFS in CSharp

### DFS in CSharp

#### Recursive DFS

```csharp
// Recursive DFS
void DFSRecursive(City city, HashSet<City> visited)
{
    if (city == null || visited.Contains(city)) return;
    
    visited.Add(city);
    Console.WriteLine(city.Name);
    
    foreach (var neighboringCity in city.NeighboringCities)
    {
        DFSRecursive(neighboringCity, visited);
    }
}
```

#### Iterative DFS

```csharp
// Iterative DFS (using stack)
void DFSIterative(City startCity)
{
    var visited = new HashSet<City>();
    var stack = new Stack<City>();
    
    stack.Push(startCity);
    
    while (stack.Count > 0)
    {
        var city = stack.Pop();
        
        if (!visited.Contains(city))
        {
            visited.Add(city);
            Console.WriteLine(city.Name);
            
            foreach (var neighboringCity in city.NeighboringCities)
            {
                stack.Push(neighboringCity);
            }
        }
    }
}
```

---

### BFS in CSharp

```csharp
void BFS(City startCity)
{
    var visited = new HashSet<City>();
    var queue = new Queue<City>();
    
    queue.Enqueue(startCity);
    
    while (queue.Count > 0)
    {
        var city = queue.Dequeue();
        
        if (!visited.Contains(city))
        {
            visited.Add(city);
            Console.WriteLine(city.Name);
            
            foreach (var neighboringCity in city.NeighboringCities)
            {
                queue.Enqueue(neighboringCity);
            }
        }
    }
}
```

In this example, **City** is a class representing a city, with properties like `Name` and a list of `NeighboringCities`.

---

## LeetCode Examples

### DFS Example: LeetCode 79 - Word Search

**Problem**: Given a 2D board and a word, determine if the word exists in the grid. The word can be constructed from letters of sequentially adjacent cells (horizontally or vertically).

**DFS Solution**: We can use DFS to search all possible paths in the grid.

```csharp
bool Exist(char[,] board, string word)
{
    for (int i = 0; i < board.GetLength(0); i++)
    {
        for (int j = 0; j < board.GetLength(1); j++)
        {
            if (DFS(board, word, i, j, 0))
                return true;
        }
    }
    return false;
}

bool DFS(char[,] board, string word, int i, int j, int index)
{
    if (index == word.Length) return true;
    if (i < 0 || j < 0 || i >= board.GetLength(0) || j >= board.GetLength(1) || board[i, j] != word[index])
        return false;
    
    char temp = board[i, j];
    board[i, j] = '#';  // Mark the cell as visited
    
    bool found = DFS(board, word, i + 1, j, index + 1) ||
                 DFS(board, word, i - 1, j, index + 1) ||
                 DFS(board, word, i, j + 1, index + 1) ||
                 DFS(board, word, i, j - 1, index + 1);
    
    board[i, j] = temp;  // Unmark the cell
    return found;
}
```

### BFS Example: LeetCode 127 - Word Ladder

**Problem**: Given two words (start and end word) and a dictionary, find the shortest transformation sequence from the start word to the end word, changing only one letter at a time.

**BFS Solution**: Since we need to find the shortest path, BFS is the optimal solution.

```csharp
int LadderLength(string beginWord, string endWord, IList<string> wordList)
{
    var wordSet = new HashSet<string>(wordList);
    if (!wordSet.Contains(endWord)) return 0;

    var queue = new Queue<(string word, int length)>();
    queue.Enqueue((beginWord, 1));

    while (queue.Count > 0)
    {
        var (word, length) = queue.Dequeue();

        for (int i = 0; i < word.Length; i++)
        {
            var charArray = word.ToCharArray();
            for (char c = 'a'; c <= 'z'; c++)
            {
                charArray[i] = c;
                var newWord = new string(charArray);

                if (newWord == endWord) return length + 1;
                if (wordSet.Contains(newWord))
                {
                    queue.Enqueue((newWord, length + 1));
                    wordSet.Remove(newWord);
                }
            }
        }
    }
    return 0;
}
```

### More Examples

Here are some additional LeetCode problems that are great for practicing DFS and BFS:

### DFS Related LeetCode Problems

1. **[79. Word Search](https://leetcode.cn/problems/word-search/)**: Explore all possible paths in a 2D grid.
2. **[130. Surrounded Regions](https://leetcode.cn/problems/surrounded-regions/)**: Use DFS to flip surrounded regions in a 2D grid.
3. **[200. Number of Islands](https://leetcode.cn/problems/number-of-islands/)**: Use DFS to compute connected components (islands).
4. **[417. Pacific Atlantic Water Flow](https://leetcode.cn/problems/pacific-atlantic-water-flow/)**: Use DFS to find cells that can flow into both the Pacific and Atlantic oceans.
5. **[22. Generate Parentheses](https://leetcode.cn/problems/generate-parentheses/)**: Use DFS to generate all valid parentheses combinations.

### BFS Related LeetCode Problems

1. **[127. Word Ladder](https://leetcode.cn/problems/word-ladder/)**: Use BFS to find the shortest transformation sequence.
2. **[994. Rotting Oranges](https://leetcode.cn/problems/rotting-oranges/)**: Use BFS to spread rot in a grid.
3. **[542. 01 Matrix](https://leetcode.cn/problems/01-matrix/)**: Use BFS to find the shortest distance to 0 in a matrix.
4. **[752. Open the Lock](https://leetcode.cn/problems/open-the-lock/)**: Use BFS to find the shortest sequence of operations to unlock the lock.
5. **[133. Clone Graph](https://leetcode.cn/problems/clone-graph/)**: Use BFS (or DFS) to clone a graph.
6. **[207. Course Schedule](https://leetcode.cn/problems/course-schedule/)**: Use BFS (topological sorting) to determine if all courses can be completed.
7. **[210. Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/)**: Use BFS and topological sorting to find the order to finish all courses.

### Problems Using Both DFS and BFS

1. **[490. The Maze](https://leetcode.cn/problems/the-maze/)**: Can be solved using both DFS and BFS with different strategies.
2. **[433. Minimum Genetic Mutation](https://leetcode.cn/problems/minimum-genetic-mutation/)**: Use BFS to find the shortest mutation sequence or DFS to explore all paths.
3. **[787. Cheapest Flights Within K Stops](https://leetcode.cn/problems/cheapest-flights-within-k-stops/)**: BFS is typically used for the shortest path, but DFS can also work.

---

## Practical Applications in Geometry

DFS and BFS are not only useful for coding problems but also have real-world applications in geometry. Here are some examples:

1. **Maze Solving**: DFS can be used to explore all possible paths, while BFS is used to find the shortest path.
2. **Connected Component Analysis**: In image processing, DFS can help identify connected components in binary images, such as finding spots or contours.
3. **Flood Fill Algorithm**: In graphics programs, DFS and BFS can be used to fill an area with a specific color.

## Conclusion

DFS and BFS are powerful algorithms commonly used for graph and tree traversal in C#. DFS is suitable for exhaustive search and backtracking, while BFS excels in finding the shortest path. Both algorithms are crucial for solving LeetCode challenges as well as practical applications in geometry.

Understanding when to use which technique, and how to implement them efficiently, can greatly enhance your problem-solving abilities.
