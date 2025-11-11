# Collections in C# Part II - Arrays, Lists and HashSets

## 1. Introduction and Recap

In [Part I](/Data_Structures_Algorithms/00_Collections_1_Strings.md) of this series, we explored **strings** — the most common and deceptively simple collection in C#. We learned how strings are **immutable sequences of characters**, how to perform core operations like concatenation, searching, and splitting, and how to write efficient string-handling code using tools like `StringBuilder`.

Now, we’re ready to move beyond text and into **general-purpose collections** — the backbone of most data handling in C#.

C# provides several built-in types for storing and managing groups of values, each with its own strengths:

* **Arrays (`T[]`)** – Fixed-size, fast, and memory-efficient.
* **Lists (`List<T>`)** – Dynamic, flexible, and widely used in day-to-day coding.
* **HashSets (`HashSet<T>`)** – Designed for uniqueness and fast membership lookups.

In this post, we’ll explore how these collections work under the hood, how to choose the right one, and what performance tradeoffs to keep in mind.

You’ll learn:

* The **syntax and behavior** of each type
* Common **operations and patterns** (add, remove, iterate, search)
* Practical **examples and performance insights**
* A short **LeetCode-style exercise** to tie it all together

By the end, you’ll have a solid mental model of C#’s most important collections — and be ready to decide *when to use an array, when to reach for a list, and when a hash set is the best tool for the job.*

---

## 2. Arrays (`T[]`)

### 2.1 Basics

An **array** in C# is the simplest form of collection. It represents a **fixed-size, ordered sequence** of elements, all of the same type.

```csharp
// Declaration and initialization
int[] numbers = new int[3];       // [0, 0, 0]
numbers[0] = 10;                  // [10, 0, 0]

// Inline initialization
string[] fruits = { "apple", "banana", "cherry" };

// Access by index
string first = fruits[0];  // "apple"
```

Key characteristics:

* **Fixed size**: Once an array is created, its length cannot change.
* **Zero-based indexing**: The first element is at index `0`.
* **Type safety**: All elements must be of the same type.

### 2.2 Common Operations

Arrays come with useful built-in methods and can also be combined with **loops** or **LINQ**.

#### **Iteration**

```csharp
int[] numbers = { 1, 2, 3, 4 };

// For loop
for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}

// Foreach loop
foreach (int n in numbers)
{
    Console.WriteLine(n);
}

// LINQ
var evens = numbers.Where(n => n % 2 == 0);
```

#### **Searching, Sorting, Reversing**

```csharp
string[] fruits = { "banana", "apple", "cherry" };

// Search
bool hasApple = fruits.Contains("apple");   // true
int index = Array.IndexOf(fruits, "cherry"); // 2

// Sort
Array.Sort(fruits);   // ["apple", "banana", "cherry"]

// Reverse
Array.Reverse(fruits); // ["cherry", "banana", "apple"]
```

#### **Multi-Dimensional Arrays**

C# supports both **rectangular arrays** (matrix-like) and **jagged arrays** (arrays of arrays).

```csharp
// Rectangular array (2x3 matrix)
int[,] matrix = new int[2, 3]
{
    { 1, 2, 3 },
    { 4, 5, 6 }
};

// Jagged array (array of arrays)
int[][] jagged = new int[2][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5 };
```

### 2.3 Performance Notes

* **Access by index**: `O(1)`
* **Search by value**: `O(n)` (linear scan unless sorted + binary search)
* **Sorting**: typically `O(n log n)`
* **Resizing**: requires allocating a new array and copying elements (`O(n)`)
* Insertions/removals in the middle are costly (require shifting elements).
* Lacks higher-level features available in `List<T>` or `HashSet<T>`.

---

## 3. Lists (`List<T>`)

### 3.1 Basics

A **List\<T>** is a dynamic, strongly-typed collection that grows or shrinks automatically as elements are added or removed. Unlike arrays, you don’t need to know the size in advance.

```csharp
// Declaration and initialization
List<string> fruits = new List<string>();
fruits.Add("apple");
fruits.Add("banana");

// Inline initialization
var numbers = new List<int> { 1, 2, 3, 4 };
```

Key characteristics:

* **Dynamic resizing**: Internally, a List uses an array that resizes as needed.
* **Indexed access**: Like arrays, elements are accessed by zero-based index.

### 3.2 Common Operations

#### **Adding, Removing, Inserting**

```csharp
var fruits = new List<string> { "apple", "banana", "cherry" };

fruits.Add("date");         // ["apple", "banana", "cherry", "date"]
fruits.Remove("banana");    // ["apple", "cherry", "date"]
fruits.Insert(1, "pear");   // ["apple", "pear", "cherry", "date"]

int index = fruits.IndexOf("cherry"); // 2
```

#### **Iteration & LINQ Queries**

```csharp
// Iteration
foreach (var fruit in fruits)
{
    Console.WriteLine(fruit);
}

// LINQ
var longNames = fruits.Where(f => f.Length > 5).ToList();
// ["banana", "cherry"]

bool anyCitrus = fruits.Any(f => f.Contains("orange")); // false
string first = fruits.FirstOrDefault(); // "apple"
```

### 3.3 Performance Notes

🔧 **Performance Notes**

* Access by index: **O(1)**
* Add at end (amortized): **O(1)**
* Insert/remove in middle: **O(n)** (due to shifting elements).
* Search by value (linear): **O(n)**
* Use when you need dynamic sizing and the convenience of built-in collection methods.

---

## 4. HashSets (`HashSet<T>`)

### 4.1 Basics

A **HashSet\<T>** is a collection of **unique elements** with no guaranteed order. Internally, it uses a hash table for fast lookups.

```csharp
// Declaration and initialization
var set = new HashSet<int> { 1, 2, 3 };

// Adding duplicates has no effect
set.Add(3); // still {1, 2, 3}
```

Key characteristics:

* **No duplicates**: Each element must be unique.
* **Unordered**: The order of elements is not preserved.
* **Hash-based**: Operations rely on efficient hashing.

### 4.2 Common Operations

#### **Add, Remove, Contains**

```csharp
var set = new HashSet<string>();
set.Add("apple");
set.Add("banana");
set.Remove("apple");

bool hasBanana = set.Contains("banana"); // true
```

#### **Set Operations**

HashSet shines when working with mathematical set logic:

```csharp
var setA = new HashSet<int> { 1, 2, 3 };
var setB = new HashSet<int> { 3, 4, 5 };

// Union
setA.UnionWith(setB);     // {1, 2, 3, 4, 5}

// Intersection
setA.IntersectWith(setB); // {3}

// Difference
setA.ExceptWith(setB);    // {1, 2}
```

### 4.3 Performance Notes

* Add/Remove/Contains: **O(1)** on average (but `O(n)` in worst-case with poor hashing).
* Union/Intersect/Except: proportional to size of the sets.
* Use when you need to enforce **uniqueness**.

---

## 6. Practice in LeetCode Examples

So far, we’ve learned the strengths of different collection types — **arrays**, **lists**, and **hash sets** — and when each shines.
Now, let’s put that knowledge into practice.

When solving algorithm problems (like those on LeetCode), the key is not just to “make it work,” but to **make it efficient**.
Choosing the right collection can reduce your algorithm from **O(n²)** to **O(n)** — often the difference between “Time Limit Exceeded” and an accepted solution.

Let’s look at two classic examples.

### **Example 1 – Two Sum**

**Goal:** Find two numbers that add up to a target.

#### ❌ Naive Approach (O(n²))

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

This works, but it checks every possible pair — that’s **O(n²)** comparisons.
As the array grows, the runtime explodes.

#### ✅ Optimized Approach (O(n))

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

By trading a bit of memory for a `HashSet`, we reduce nested loops to constant-time lookups — a classic example of how **Hash-based collections** drastically improve performance.

### **Example 2 – Replace Words (Optimized Lookup with HashSet)**

#### ❌ Inefficient Approach (O(n × m))

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

Each word is checked against *every* root — this double loop quickly becomes slow when the dictionary grows.

#### ✅ Optimized with HashSet (O(n × k))

```csharp
public string ReplaceWords(IList<string> dictionary, string sentence)
    {
        // Convert dictionary to HashSet for O(1) lookups
        HashSet<string> dictSet = new HashSet<string>(dictionary);

        // Split the sentence into words
        string[] words = sentence.Split(' ');

        // Replace each word with its shortest root if it exists
        for (int i = 0; i < words.Length; i++)
        {
            words[i] = ShortestRoot(words[i], dictSet);
        }

        // Join the words back into a sentence
        return string.Join(" ", words);
    }
```

Using a `HashSet` turns root lookups into **constant-time operations**, cutting down unnecessary comparisons dramatically.

---

## **7. Conclusion**

Choosing the right collection in C# depends on balancing **performance, memory, and problem requirements**:

* **Arrays** give you raw speed and predictable indexing but are rigid in size.
* **List\<T>** offers flexibility with dynamic resizing, making it the default choice for most general-purpose tasks.
* **HashSet\<T>** shines when you care about uniqueness or fast lookups without needing order.

When solving problems (like those on LeetCode), always consider the **time complexity of operations**—for example, `Contains` in a `HashSet` is O(1) on average, while in a `List` it’s O(n).

👉 **Key takeaway**: don’t just reach for the most familiar collection—pick the one that fits the problem’s needs.
