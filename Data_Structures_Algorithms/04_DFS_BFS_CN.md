## 在C#中掌握DFS和BFS：技术、实现与LeetCode示例

在这篇博文中，我们将探讨两个基本的图遍历算法：深度优先搜索（DFS）和广度优先搜索（BFS）。这些算法是解决许多编程问题的关键工具，特别是那些涉及遍历或搜索数据结构（如树和图）的问题。DFS 和 BFS 都有其独特的探索节点和边的方式，使它们适用于不同类型的问题，从迷宫中寻找最短路径到计算网络中的连通组件。

我们将介绍DFS和BFS背后的核心概念，它们如何工作，以及在哪些场景下应优先选择其中一个。此外，我们还会在C#中实现这些算法，并深入探讨其在几何和路径规划等实际应用中的表现。我们还会研究一些常见的LeetCode问题，展示DFS和BFS如何有效地解决诸如遍历矩阵、寻找网格中的岛屿和解决谜题等挑战。理解这些算法的工作原理以及何时使用它们，将有助于提升你的问题解决能力和代码效率。

## 什么是DFS和BFS？

**深度优先搜索 (DFS)** 在一条树或图的分支上尽可能深入，直到无法继续再回溯。它先深入一个分支，然后再尝试其他分支，适用于需要探索所有可能路径的问题。

**广度优先搜索 (BFS)** 在进入下一层之前，先探索当前层的所有节点。它适用于需要在两个节点之间找到最短路径的问题。

### DFS vs BFS

| DFS            | BFS               |
| -------------- | ----------------- |
| 使用栈（隐式或显式）     | 使用队列              |
| 适合穷尽搜索或解决谜题    | 适合最短路径问题          |
| 在树遍历中可能消耗更少的内存 | 在宽度较大的图中可能会占用更多内存 |

## 它们是如何工作的？

![BFS-and-DFS-Algorithms.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/a2c82cd063214b5ab65d34f687120e8d~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5LiA5Y-q5ouJ5Y-k:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDM1MjMwMjA3NTg3NDM5MyJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1735225760&x-orig-sign=x4gAbs6tnX0tezhZ6Po%2FKFAIUFQ%3D)

### 深度优先搜索 (DFS)

DFS 从一个节点开始，沿着每个分支尽可能深入，然后回溯。实现DFS有两种常见方法：递归和迭代（使用显式栈）。

以下是DFS的工作步骤：

1. 从根节点或任意节点开始。
2. 访问该节点。
3. 递归或迭代地访问所有未访问的邻居。
4. 如果没有剩余的邻居，则回溯。

### 广度优先搜索 (BFS)

BFS 从一个节点开始，先探索所有当前层的邻居，然后再进入下一层。这通常通过队列来跟踪要访问的下一个节点。

BFS的工作步骤如下：

1. 从根节点或任意节点开始。
2. 访问该节点并将其所有邻居添加到队列中。
3. 出列一个节点，访问它，并对其所有邻居重复此操作。

## 何时使用DFS和BFS？

- **使用DFS** 当你需要：
  - 探索所有可能的路径，例如解决迷宫问题或回溯问题。
  - 解决涉及树遍历的问题，如后序遍历或中序遍历。
  - 处理那些你希望深入一个路径并穷尽它，然后再移动到下一个路径的问题（例如在图中寻找连通组件）。

- **使用BFS** 当你需要：
  - 在无权图中找到最短路径，例如在最短路径问题中，如 **单词接龙**。
  - 层次遍历树或图，使其非常适合处理像 **之字形层次遍历** 这样的问题。

***

## 在C#中实现DFS和BFS的常用方法

### 在C#中实现DFS

```csharp
// 递归DFS
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

```csharp
// 迭代DFS（使用栈）
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

### 在C#中实现BFS

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

在这个例子中，**City** 是表示一个城市的类，具有属性 `Name` 和一个 `NeighboringCities` 的列表。

***

## LeetCode 示例

### DFS 示例：LeetCode 79 - 单词搜索

**问题**：给定一个二维网格和一个单词，判断该单词是否存在于网格中，单词可以通过水平或垂直相邻的单元格组成。

**DFS 解决方案**：我们可以使用 DFS 来搜索网格中的所有可能路径。

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
    board[i, j] = '#';  // 标记该单元格已访问
    
    bool found = DFS(board, word, i + 1, j, index + 1) ||
                 DFS(board, word, i - 1, j, index + 1) ||
                 DFS(board, word, i, j + 1, index + 1) ||
                 DFS(board, word, i, j - 1, index + 1);
    
    board[i, j] = temp;  // 取消标记该单元格
    return found;
}
```

### BFS 示例：LeetCode 127 - 单词接龙

**问题**：给定两个单词（起始和结束单词）以及一个字典，找出从起始单词到结束单词的最短转换序列，每次只能改变一个字母。

**BFS 解决方案**：由于我们需要找到最短路径，BFS 是最优解。

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

### 更多示例

以下是一些适合练习 DFS 和 BFS 的 LeetCode 问题列表：

### DFS 相关 LeetCode 问题

1. **[79. 单词搜索](https://leetcode.cn/problems/word-search/)**: 在二维网格中探索所有可能路径。
2. **[130. 被围绕的区域](https://leetcode.cn/problems/surrounded-regions/)**: 使用 DFS 翻转二维网格中被围绕的区域。
3. **[200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)**: 使用 DFS 计算连通分量（岛屿）。
4. **[417. 太平洋大西洋水流问题](https://leetcode.cn/problems/pacific-atlantic-water-flow/)**: 使用 DFS 找到可以流入太平洋和大西洋的单元格。
5. **[22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)**: 使用 DFS 生成所有有效的括号组合。

### BFS 相关 LeetCode 问题

1. **[127. 单词接龙](https://leetcode.cn/problems/word-ladder/)**: 使用 BFS 寻找最短转换序列。
2. **[994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)**: 使用 BFS 在网格中传播腐烂。
3. **[542. 01 矩阵](https://leetcode.cn/problems/01-matrix/)**: 使用 BFS 找到矩阵中到 0 的最短距离。
4. **[752. 打开转盘锁](https://leetcode.cn/problems/open-the-lock/)**: 使用 BFS 寻找打开锁的最短操作序列。
5. **[133. 克隆图](https://leetcode.cn/problems/clone-graph/)**: 使用 BFS（或 DFS）克隆图。
6. **[207. 课程表](https://leetcode.cn/problems/course-schedule/)**: 使用 BFS（拓扑排序）判断是否可以完成所有课程。
7. **[210. 课程表 II](https://leetcode.cn/problems/course-schedule-ii/)**: 使用 BFS 和拓扑排序找到完成所有课程的顺序。

### 同时使用 DFS 和 BFS 的问题

1. **[490. 迷宫](https://leetcode.cn/problems/the-maze/)**: 可以通过 DFS 和 BFS 分别采用不同的策略来解决。
2. **[433. 最小基因变化](https://leetcode.cn/problems/minimum-genetic-mutation/)**: 使用 BFS 寻找最短突变序列或使用 DFS 探索所有路径。
3. **[787. K 站中转内最便宜的航班](https://leetcode.cn/problems/cheapest-flights-within-k-stops/)**: BFS 通常用于寻找最短路线，但 DFS 也可以工作。

***

## 几何中的实际应用

DFS 和 BFS 不仅限于编码问题，它们在几何领域也有实际的应用。以下是一些例子：

1. **迷宫解法**：DFS 可以用来探索所有可能的路径，而 BFS 则用于找到最短路径。
2. **连通分量分析**：在图像处理领域，DFS 可以帮助识别二值图像中的连通分量，例如查找斑点或轮廓。
3. **泛洪填充算法**：在图形程序中，DFS 和 BFS 都可以用于用特定颜色填充区域。

## 结论

DFS 和 BFS 是两个强大的算法，常用于 C# 中图和树的遍历。DFS 适合穷举搜索和回溯，而 BFS 则擅长寻找最短路径。这两种算法在解决 LeetCode 挑战以及几何的实际应用中都至关重要。

理解何时使用哪种技术，以及如何高效实现它们，可以显著提升你的问题解决能力。
