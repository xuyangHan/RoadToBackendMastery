# **Strings and Collections in C#**

## **1. Introduction**

When we write software, we rarely deal with just a single piece of data. Most of the time, we need to store, organize, and manipulate **groups of data** — whether that’s a list of users, a set of unique IDs, or even just the characters inside a sentence.

This is where **strings and collections** come in.

* A **string** is one of the most commonly used types in C#. It represents text, but under the hood, it is also a **collection of characters**. That makes it a natural bridge to understanding how collections work.
* **Collections** like arrays, lists, and hash sets are the building blocks for storing and working with data in memory. Each has its strengths and tradeoffs:

  * **Arrays**: fixed-size, fast access.
  * **Lists**: dynamic size, flexible but with some performance costs.
  * **HashSets**: enforce uniqueness and provide very fast lookups.

Throughout this post, we’ll start with strings, then explore arrays, lists, and hash sets. For each, we’ll look at **basic usage, common operations, performance characteristics, and when to use them**. By the end, you’ll have a practical overview of these core concepts — enough to write cleaner, more efficient C# code and make informed choices about which collection type fits your needs.

---

## **2. Strings**

### **2.1 Basics**

A [**string**](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/) in C# represents a sequence of characters. You can declare and initialize it in several ways:

```csharp
string greeting = "Hello, world!";
string empty = string.Empty;  // equivalent to ""
char firstChar = greeting[0]; // 'H'
```

A few key properties of strings in C#:

* **Reference type** – Even though they look like primitive values, strings are objects in .NET.
* **Immutable** – Once a string is created, it cannot be changed. Any operation that seems to “modify” a string actually creates a new one in memory.
* **Indexable** – You can access individual characters using zero-based indexing (`string[index]`).

For example:

```csharp
string word = "cat";
char c = word[1]; // 'a'
```

### **2.2 Common Operations**

Strings come with a rich set of built-in operations:

#### **Concatenation**

```csharp
string s1 = "Hello";
string s2 = "World";
string result = s1 + " " + s2;          // "Hello World"
string concat = string.Concat(s1, s2);  // "HelloWorld"
```

For repeated concatenation inside loops, prefer `StringBuilder` (more on that later).

#### **Replace / Remove / Substring**

```csharp
string text = "I like cats";
string replaced = text.Replace("cats", "dogs"); // "I like dogs"
string removed = text.Remove(2, 5);             // "Ikes"
string sub = text.Substring(2, 4);              // "like"
```

#### **Split / Join**

```csharp
string csv = "apple,banana,cherry";
string[] parts = csv.Split(',');   // ["apple", "banana", "cherry"]

string joined = string.Join(" | ", parts);
// "apple | banana | cherry"
```

#### **Searching**

```csharp
string sentence = "C# strings are powerful.";

bool contains = sentence.Contains("strings");    // true
int index = sentence.IndexOf("are");             // 10
bool starts = sentence.StartsWith("C#");         // true
bool ends = sentence.EndsWith("powerful.");      // true
```

### **2.3 Performance Considerations**

Because **strings are immutable**, operations that appear to modify them actually allocate **new strings in memory**:

```csharp
string s = "a";
for (int i = 0; i < 1000; i++)
{
    s += "x"; // creates a new string on every iteration
}
```

The above code is inefficient because each concatenation allocates a new string, resulting in O(n²) behavior overall.

To handle repeated modifications efficiently, use **`StringBuilder`**:

```csharp
var sb = new StringBuilder("a");
for (int i = 0; i < 1000; i++)
{
    sb.Append("x");
}
string result = sb.ToString();
```

This avoids repeated allocations and is much faster for large-scale concatenation.

**Time complexity (approximate):**

* Indexing (`string[i]`): **O(1)**
* Searching (`IndexOf`, `Contains`): **O(n)**
* Concatenation (`+` with short strings): **O(n)** but can degrade if repeated many times
* `StringBuilder.Append`: amortized **O(1)** per operation

👉 **Key takeaway:** Strings are simple to use and safe thanks to immutability, but for heavy modifications, prefer `StringBuilder`.

---

## **3. Arrays**

### **3.1 Basics**

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

---

### **3.2 Common Operations**

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

---

### **3.3 Pros, Cons, and Performance**

✅ **Pros**

* Very fast **index-based access** (`O(1)`).
* Compact memory layout.
* Good for scenarios where size is **known ahead of time**.

⚠️ **Cons**

* **Fixed size** — cannot grow or shrink dynamically.
* Insertions/removals in the middle are costly (require shifting elements).
* Lacks higher-level features available in `List<T>` or `HashSet<T>`.

🔧 **Performance Notes**

* **Access by index**: `O(1)`
* **Search by value**: `O(n)` (linear scan unless sorted + binary search)
* **Sorting**: typically `O(n log n)`
* **Resizing**: requires allocating a new array and copying elements (`O(n)`)

👉 **When to use arrays**:

* You know the exact number of elements in advance.
* You need **fast random access**.
* You want minimal overhead compared to higher-level collections.

---

## **4. List\<T>**

### **4.1 Basics**

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

---

### **4.2 Common Operations**

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

---

### **4.3 Pros, Cons, and Performance**

✅ **Pros**

* Fast **index-based access** (`O(1)`).
* Automatically grows as needed.
* Rich API with built-in methods and LINQ support.

⚠️ **Cons**

* Insert/remove in the **middle** is slower (`O(n)`, due to shifting elements).
* More memory overhead than a fixed array.

🔧 **Performance Notes**

* Access by index: **O(1)**
* Add at end (amortized): **O(1)**
* Insert/remove in middle: **O(n)**
* Search by value (linear): **O(n)**

👉 **When to use List\<T>**:

* You need dynamic sizing.
* You want the convenience of built-in collection methods.
* Random access is important, and middle insertions are rare.

---

## **5. HashSet\<T>**

### **5.1 Basics**

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

---

### **5.2 Common Operations**

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

---

### **5.3 Pros, Cons, and Performance**

✅ **Pros**

* Very fast lookups and uniqueness enforcement (`O(1)` average for add, remove, contains).
* Built-in support for set operations (union, intersect, difference).

⚠️ **Cons**

* No indexing — cannot access by position.
* No ordering (use `SortedSet<T>` if order is required).
* Slight overhead from hashing.

🔧 **Performance Notes**

* Add/Remove/Contains: **O(1)** on average (but `O(n)` in worst-case with poor hashing).
* Union/Intersect/Except: proportional to size of the sets.

👉 **When to use HashSet\<T>**:

* You need to enforce **uniqueness**.
* Fast membership checks are important.
* Order of elements doesn’t matter.

---

## **6. LeetCode Examples**

LeetCode problems are a great way to see how strings and collections appear in real-world coding challenges. Below are a few classic examples and how different data structures make solving them easier.

### **6.1 Strings**

**Problem: Valid Palindrome (LC 125)**
*Check if a string reads the same forward and backward, ignoring case and non-alphanumeric characters.*

```csharp
public bool IsPalindrome(string s)
{
    int left = 0, right = s.Length - 1;
    while (left < right)
    {
        if (!char.IsLetterOrDigit(s[left])) { left++; continue; }
        if (!char.IsLetterOrDigit(s[right])) { right--; continue; }

        if (char.ToLower(s[left]) != char.ToLower(s[right]))
            return false;

        left++;
        right--;
    }
    return true;
}
```

* Uses **indexing** (`s[i]`) to access characters directly.
* Built-in methods like `char.IsLetterOrDigit` and `char.ToLower` make checks easy.
* Two-pointer technique is efficient: `O(n)` time, `O(1)` space.

**Problem: Number of Segments in a String (LC 434)**
*Count how many words (segments) exist in a string, where a segment is defined as a contiguous sequence of non-space characters.*

```csharp
public int CountSegments(string s) {
    if (string.IsNullOrWhiteSpace(s)) {
        return 0;
    }

    return s.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
}
```

* Uses **`Split`** with `StringSplitOptions.RemoveEmptyEntries` to avoid counting empty strings caused by multiple spaces.
* Edge cases like leading, trailing, or multiple spaces between words are handled automatically.
* Runs in `O(n)` time (needs to scan the string once) and `O(n)` space (stores split parts).

### **6.2 Arrays**

**Problem: Two Sum (LC 1)**
*Find two numbers in an array that add up to a target.*

```csharp
public int[] TwoSum(int[] nums, int target)
{
    var map = new Dictionary<int, int>(); // value -> index
    for (int i = 0; i < nums.Length; i++)
    {
        int complement = target - nums[i];
        if (map.ContainsKey(complement))
            return new int[] { map[complement], i };
        map[nums[i]] = i;
    }
    return new int[0];
}
```

* **Indexing** allows quick reference (`nums[i]`).
* Paired with a **Dictionary/HashSet** for faster lookup (instead of O(n²) brute force).

### **6.3 List\<T>**

**Problem: Pascal’s Triangle (LC 118)**
*Generate Pascal’s Triangle up to n rows.*

```csharp
public IList<IList<int>> Generate(int numRows)
{
    var result = new List<IList<int>>();
    for (int i = 0; i < numRows; i++)
    {
        var row = new List<int>(new int[i + 1]);
        row[0] = row[i] = 1;

        for (int j = 1; j < i; j++)
            row[j] = result[i - 1][j - 1] + result[i - 1][j];

        result.Add(row);
    }
    return result;
}
```

* **Dynamic resizing** makes it easy to build rows of varying lengths.
* `List<T>` provides convenient APIs (`Add`, indexing).
* Perfect fit for problems where the result grows dynamically.

### **6.4 HashSet\<T>**

**Problem: Contains Duplicate (LC 217)**
*Check if any element appears more than once in an array.*

```csharp
public bool ContainsDuplicate(int[] nums)
{
    var seen = new HashSet<int>();
    foreach (var num in nums)
    {
        if (!seen.Add(num)) // Add returns false if duplicate
            return true;
    }
    return false;
}
```

👉 These problems highlight how **strings, arrays, lists, and hash sets** map directly to coding challenges. Picking the right structure simplifies the solution, both in code readability and efficiency.

---

## **7. Conclusion**

Choosing the right collection in C# depends on balancing **performance, memory, and problem requirements**:

* **Arrays** give you raw speed and predictable indexing but are rigid in size.
* **List\<T>** offers flexibility with dynamic resizing, making it the default choice for most general-purpose tasks.
* **HashSet\<T>** shines when you care about uniqueness or fast lookups without needing order.

When solving problems (like those on LeetCode), always consider the **time complexity of operations**—for example, `Contains` in a `HashSet` is O(1) on average, while in a `List` it’s O(n).

👉 **Key takeaway**: don’t just reach for the most familiar collection—pick the one that fits the problem’s needs.
