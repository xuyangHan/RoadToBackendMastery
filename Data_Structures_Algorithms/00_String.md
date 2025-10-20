# **Strings in C#**

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
