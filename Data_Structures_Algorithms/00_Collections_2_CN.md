# C# 集合（Collections）第二部分 —— 数组、列表与哈希集

## 1. 引言与回顾

在本系列的[第一部分](/Data_Structures_Algorithms/00_Collections_1_Strings.md)中，我们探讨了 **字符串（string）** —— C# 中最常见、但也最容易被低估的集合类型。我们了解了字符串作为**不可变的字符序列**的特性，学习了拼接、搜索、分割等核心操作，以及如何使用 `StringBuilder` 提高字符串处理效率。

现在，我们将从文本转向更通用的 **数据集合** —— 它们是 C# 中数据处理的核心。

C# 提供了多种内置集合类型，用于存储和管理一组数据，每种都有其优势：

* **数组（`T[]`）** —— 固定大小，速度快，内存效率高。
* **列表（`List<T>`）** —— 动态可变，灵活易用，是日常开发的首选。
* **哈希集（`HashSet<T>`）** —— 用于去重和快速查找。

在本文中，我们将深入了解这些集合的内部机制，学习如何选择合适的类型，以及它们在性能上的差异。

你将学到：

* 各种集合的 **语法与行为**
* 常见的 **操作与模式**（添加、删除、遍历、搜索）
* 实际 **示例与性能洞察**
* 一个 **LeetCode 风格练习** 来巩固知识

读完本篇后，你将清楚地知道 —— *什么时候该用数组、什么时候用列表、什么时候哈希集才是最合适的工具。*

---

## 2. 数组（`T[]`）

### 2.1 基础

在 C# 中，**数组**是最基础的集合形式。它表示一个**固定大小、顺序有序**、类型一致的元素序列。

```csharp
// 声明与初始化
int[] numbers = new int[3];       // [0, 0, 0]
numbers[0] = 10;                  // [10, 0, 0]

// 内联初始化
string[] fruits = { "apple", "banana", "cherry" };

// 按索引访问
string first = fruits[0];  // "apple"
```

关键特性：

* **固定大小**：数组创建后长度不可更改。
* **从 0 开始索引**：第一个元素的索引为 `0`。
* **类型安全**：所有元素必须是同一类型。

### 2.2 常见操作

数组拥有若干内置方法，也可结合 **循环** 或 **LINQ** 使用。

#### **遍历**

```csharp
int[] numbers = { 1, 2, 3, 4 };

// for 循环
for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}

// foreach 循环
foreach (int n in numbers)
{
    Console.WriteLine(n);
}

// LINQ
var evens = numbers.Where(n => n % 2 == 0);
```

#### **搜索、排序、反转**

```csharp
string[] fruits = { "banana", "apple", "cherry" };

// 搜索
bool hasApple = fruits.Contains("apple");   // true
int index = Array.IndexOf(fruits, "cherry"); // 2

// 排序
Array.Sort(fruits);   // ["apple", "banana", "cherry"]

// 反转
Array.Reverse(fruits); // ["cherry", "banana", "apple"]
```

#### **多维数组**

C# 支持 **矩形数组**（多维数组）和 **交错数组**（数组的数组）。

```csharp
// 矩形数组（2x3 矩阵）
int[,] matrix = new int[2, 3]
{
    { 1, 2, 3 },
    { 4, 5, 6 }
};

// 交错数组（数组的数组）
int[][] jagged = new int[2][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
```

### 2.3 性能分析

* 按索引访问：`O(1)`
* 按值查找：`O(n)`（线性搜索，若已排序可用二分法）
* 排序：约 `O(n log n)`
* 调整大小：需重新分配数组并复制元素（`O(n)`）
* 插入/删除中间元素代价高（需要移动元素）
* 不具备 `List<T>` 或 `HashSet<T>` 的高级特性

---

## 3. 列表（`List<T>`）

### 3.1 基础

**List<T>** 是一种动态的强类型集合，可根据需要自动增长或缩减。与数组不同，你无需提前知道大小。

```csharp
// 声明与初始化
List<string> fruits = new List<string>();
fruits.Add("apple");
fruits.Add("banana");

// 内联初始化
var numbers = new List<int> { 1, 2, 3, 4 };
```

关键特性：

* **动态扩容**：内部通过数组实现，容量不足时自动扩容。
* **索引访问**：与数组一样，通过下标访问元素。

### 3.2 常见操作

#### **添加、删除、插入**

```csharp
var fruits = new List<string> { "apple", "banana", "cherry" };

fruits.Add("date");         // ["apple", "banana", "cherry", "date"]
fruits.Remove("banana");    // ["apple", "cherry", "date"]
fruits.Insert(1, "pear");   // ["apple", "pear", "cherry", "date"]

int index = fruits.IndexOf("cherry"); // 2
```

#### **遍历与 LINQ 查询**

```csharp
// 遍历
foreach (var fruit in fruits)
{
    Console.WriteLine(fruit);
}

// LINQ 查询
var longNames = fruits.Where(f => f.Length > 5).ToList();
// ["banana", "cherry"]

bool anyCitrus = fruits.Any(f => f.Contains("orange")); // false
string first = fruits.FirstOrDefault(); // "apple"
```

### 3.3 性能分析

🔧 **性能要点**

* 按索引访问：**O(1)**
* 尾部添加（均摊）：**O(1)**
* 中间插入/删除：**O(n)**（因需移动元素）
* 按值搜索：**O(n)**
* 适合需要动态大小、方便操作的通用场景

---

## 4. 哈希集（`HashSet<T>`）

### 4.1 基础

**HashSet<T>** 是一个只包含**唯一元素**、且不保证顺序的集合。其内部通过哈希表实现高效查找。

```csharp
// 声明与初始化
var set = new HashSet<int> { 1, 2, 3 };

// 添加重复元素无效
set.Add(3); // 仍然是 {1, 2, 3}
```

关键特性：

* **不允许重复**：元素唯一。
* **无序**：不保证插入顺序。
* **基于哈希表**：操作依赖哈希函数效率。

### 4.2 常见操作

#### **添加、删除、判断包含**

```csharp
var set = new HashSet<string>();
set.Add("apple");
set.Add("banana");
set.Remove("apple");

bool hasBanana = set.Contains("banana"); // true
```

#### **集合运算**

哈希集在数学集合操作中非常高效：

```csharp
var setA = new HashSet<int> { 1, 2, 3 };
var setB = new HashSet<int> { 3, 4, 5 };

// 并集
setA.UnionWith(setB);     // {1, 2, 3, 4, 5}

// 交集
setA.IntersectWith(setB); // {3}

// 差集
setA.ExceptWith(setB);    // {1, 2}
```

### 4.3 性能分析

* 添加/删除/查找：平均 **O(1)**（哈希冲突时最坏为 O(n)）
* 并集/交集/差集：与集合大小成正比
* 适合需要**唯一性**与**快速查找**的场景

---

## 6. LeetCode 实战练习

到目前为止，我们已了解数组、列表和哈希集的优劣及应用场景。
接下来通过算法题来看看如何选择合适的集合以提升性能。

在解决算法题（例如 LeetCode）时，关键不仅是“能实现”，更是**高效实现**。
选择正确的集合类型，往往能让算法从 **O(n²)** 优化到 **O(n)** —— 这可能就是“超时未通过”和“成功提交”的区别。

### **示例 1 – Two Sum（两数之和）**

**目标：** 找出数组中两数之和等于目标值的下标。

#### ❌ 朴素解法（O(n²)）

```csharp
public int[] TwoSum(int[] nums, int target)
{
    for (int i = 0; i < nums.Length; i++)
    {
        for (int j = i + 1; j < nums.Length; j++)
        {
            if (nums[i] + nums[j] == target)
                return new int[] { i, j };
        }
    }
    return Array.Empty<int>();
}
```

此方法检查每一对数，复杂度为 **O(n²)**，当数组较大时性能极差。

#### ✅ 优化解法（O(n)）

```csharp
public int[] TwoSum(int[] nums, int target)
{
    HashSet<int> seen = new HashSet<int>();
    for (int i = 0; i < nums.Length; i++)
    {
        int complement = target - nums[i];
        if (seen.Contains(complement))
            return new int[] { Array.IndexOf(nums, complement), i };
        seen.Add(nums[i]);
    }
    return Array.Empty<int>();
}
```

通过使用 `HashSet` 进行常数时间查找，避免了嵌套循环。
这是 **哈希集合优化性能** 的典型案例。

---

### **示例 2 – Replace Words（优化查找）**

#### ❌ 低效解法（O(n × m)）

```csharp
public string ReplaceWords(IList<string> dictionary, string sentence)
{
    string[] words = sentence.Split(' ');
    for (int i = 0; i < words.Length; i++)
    {
        foreach (var root in dictionary)
        {
            if (words[i].StartsWith(root))
            {
                words[i] = root;
                break;
            }
        }
    }
    return string.Join(" ", words);
}
```

每个单词都要与所有前缀对比，双重循环使效率急剧下降。

#### ✅ 使用 HashSet 优化（O(n × k)）

```csharp
public string ReplaceWords(IList<string> dictionary, string sentence)
{
    // 将字典转为 HashSet，实现 O(1) 查找
    HashSet<string> dictSet = new HashSet<string>(dictionary);

    // 分割句子
    string[] words = sentence.Split(' ');

    // 逐个替换
    for (int i = 0; i < words.Length; i++)
    {
        words[i] = ShortestRoot(words[i], dictSet);
    }

    // 拼回句子
    return string.Join(" ", words);
}
```

借助 `HashSet`，前缀查找由线性变为常数时间，大幅减少无效比较。

---

## **7. 总结**

在 C# 中选择合适的集合类型，需要在 **性能、内存与需求** 之间权衡：

* **数组（Array）**：提供最高访问速度，但大小固定。
* **列表（List<T>）**：灵活可扩展，是通用开发首选。
* **哈希集（HashSet<T>）**：在需要唯一性或快速查找时最强大。

在算法题或工程实践中，务必考虑操作的**时间复杂度**。
例如：`HashSet.Contains` 平均 O(1)，而 `List.Contains` 是 O(n)。

👉 **关键结论：** 不要只用你最熟悉的集合 —— 而要用**最适合问题场景的集合**。
