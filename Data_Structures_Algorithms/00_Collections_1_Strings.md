# Collections in C# Part I - Strings

## 1. Introduction

When we write software, we rarely work with just a single piece of data. Most of the time, we need to store, organize, and manipulate **groups of related values** — a list of users, a set of unique IDs, or even the individual characters inside a sentence.

This is where **collections** come in.

In C#, a *collection* is any object that can hold multiple elements, such as arrays, lists, or sets. Collections form the foundation of most programs — they power everything from database results and API responses to in-memory caches and text processing.

Interestingly, even though we usually think of a **string** as “just text,” it’s technically a **collection of characters**. You can index it, iterate over it, and transform it much like you would with other collection types. This makes it the perfect place to start learning how collections behave in C#.

In this two-part series, we’ll explore how to work effectively with C# collections:

* **Part I (this post):** focuses on **strings** — their structure, common operations, and performance considerations.
* **Part II:** dives into **arrays, lists, and hash sets**, examining their differences, when to use each, and how they perform under the hood.

By the end of this series, you’ll have a solid grasp of how to handle data efficiently in memory — writing cleaner, faster, and more expressive C# code.

---

## 2. String Basics

A [**string**](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/) in C# represents an immutable sequence of characters. You can declare and initialize strings in several ways:

```csharp
string greeting = "Hello, world!";
string empty = string.Empty;  // equivalent to ""
char firstChar = greeting[0]; // 'H'
```

A few key characteristics of strings in C#:

* **Reference type** – Although they behave like primitive values, strings are actually objects in .NET (`System.String`).
* **Immutable** – Once a string is created, it cannot be changed. Any operation that appears to “modify” a string (like concatenation or replace) actually returns a **new string instance**.
* **Indexable and enumerable** – You can access individual characters using zero-based indexing (`string[index]`) or iterate over them using a `foreach` loop.

Example:

```csharp
string word = "cat";
char c = word[1]; // 'a'

foreach (char ch in word)
{
    Console.WriteLine(ch);
}
```

---

## 3. Core String Operations

Strings in C# come with a rich set of built-in methods for manipulating text. Below are the most commonly used ones, grouped by category.

### 3.1 Concatenation

You can join strings using the `+` operator or static methods like `string.Concat`.

```csharp
string s1 = "Hello";
string s2 = "World";

// Using '+' operator
string result = s1 + " " + s2;          // "Hello World"

// Using string.Concat
string concat = string.Concat(s1, s2);  // "HelloWorld"
```

> 💡 **Tip:** For combining a few strings, `+` is fine.
> But if you’re building strings repeatedly inside loops, use `StringBuilder` for better performance (since strings are immutable and each `+` creates a new one).

---

### 3.2 Replace / Remove / Substring

These methods let you extract or modify parts of a string.

```csharp
string text = "I like cats";

// Replace substring, replaces all occurrences of the specified substring
string replaced = text.Replace("cats", "dogs"); // "I like dogs"

// Remove substring (starting at index 2, remove 5 chars)
string removed = text.Remove(2, 5);             // "Ikes"

// Extract substring (starting at index 2, take 4 chars)
string sub = text.Substring(2, 4);              // "like"
```

> 🧠 **Note:** Since strings are immutable, all these methods return **new string instances** — the original `text` remains unchanged.

---

### 3.3 Split / Join

Splitting and joining are common when handling structured text, such as CSV or user input.

```csharp
string csv = "apple,banana,cherry";

// Split by delimiter → array of strings
string[] parts = csv.Split(',');   // ["apple", "banana", "cherry"]

// Join array back into a single string
string joined = string.Join(" | ", parts);  
// "apple | banana | cherry"
```

> 🧩 **Usage tip:** `Split()` and `Join()` are often used together when transforming text — for example, parsing CSV data or formatting display strings.

---

### 3.4 Searching

C# provides several methods to find or check for text inside strings.

```csharp
string sentence = "C# strings are powerful.";

// Check for substring presence
bool contains = sentence.Contains("strings");    // true

// Find zero-based index of substring (returns -1 if not found)
int index = sentence.IndexOf("are");             // 11

// Check prefixes or suffixes
bool starts = sentence.StartsWith("C#");         // true
bool ends = sentence.EndsWith("powerful.");      // true
```

> 🔍 **Tip:** These methods are case-sensitive by default.
> Use overloads like `Contains("STRINGS", StringComparison.OrdinalIgnoreCase)` for case-insensitive checks.

---

### 3.5 Formatting and Interpolation

C# provides multiple ways to embed values into strings. The two most common approaches are **composite formatting** and **string interpolation**, and you can also use **verbatim strings** for multi-line text or literal backslashes.

#### Composite Formatting

Composite formatting uses placeholders like `{0}`, `{1}` which are replaced by arguments passed to `Console.WriteLine` or `string.Format`.

```csharp
var pw = (firstName: "Phillis", lastName: "Wheatley", born: 1753);

// Using composite formatting
Console.WriteLine("{0} {1} was born in {2}.", pw.firstName, pw.lastName, pw.born);
// Output: "Phillis Wheatley was born in 1753."
```

> 🧠 **Note:** Works in older C# code, but readability decreases with many placeholders.


#### String Interpolation (`$`)

String interpolation is a modern, readable alternative. Prefix a string with `$` and embed expressions inside `{}`.

```csharp
var jh = (firstName: "Jupiter", lastName: "Hammon", born: 1711);

Console.WriteLine($"{jh.firstName} {jh.lastName} was born in {jh.born}.");
// Output: "Jupiter Hammon was born in 1711."
```

> ✅ **Advantages:** Clearer, type-safe, easier to read than `{0}` placeholders.

#### Verbatim Strings (`@`)

A verbatim string literal (prefixed with `@`) preserves whitespace, line breaks, and backslashes without requiring escapes.

```csharp
string path = @"C:\Users\Public\Documents";
Console.WriteLine(path);
// Output: "C:\Users\Public\Documents"

string multiLine = @"This is line 1
This is line 2
This is line 3";
Console.WriteLine(multiLine);
```

#### Combining `$` and `@`

You can combine both prefixes (`$@"..."`) to get **interpolation + verbatim** — useful for multi-line templates or paths with variables.

```csharp
var user = (firstName: "Alice", lastName: "Smith");
string report = $@"User: {user.firstName} {user.lastName}
Directory: C:\Users\{user.firstName}";

Console.WriteLine(report);
/*
Output:
User: Alice Smith
Directory: C:\Users\Alice
*/
```

> 📝 **Tip:** 
- The order of `$` and `@` doesn’t matter (`$@"..."` or `@$"..."` both work).
- Use interpolation (`$"..."`) whenever possible — it’s clearer, type-safe, and avoids index errors.

---

### 3.6 Converters and `ToString()`

All types in .NET can be converted to strings using `ToString()`, and some conversions (like number → binary string) use `Convert.ToString()`.

```csharp
int number = 123;
Console.WriteLine(number.ToString());     // "123"

DateTime now = DateTime.Now;
Console.WriteLine(now.ToString("yyyy-MM-dd"));  // "2025-11-10"

bool flag = true;
Console.WriteLine(flag.ToString());       // "True"
```

```csharp
int value = 13; // binary: 1101

// Convert integer to binary string, then count number of '1's
int count = Convert.ToString(value, 2)
                 .Count(c => c == '1'); // Output: 3

int num = 42;
Console.WriteLine(Convert.ToString(num, 16)); // "2a" (convert to hexadecimal)
```

> 🧮 **Example use case:** This pattern appears in coding challenges like counting set bits or formatting numeric output.

---

## 4. Performance Considerations

### StringBuilder

Because **strings are immutable**, operations that appear to modify them actually allocate **new strings in memory**

```csharp
string s = "a";
for (int i = 0; i < 1000; i++)
{
    s += "x"; // creates a new string on every iteration
}
```

The above code is inefficient because each concatenation allocates a new string, resulting in **O(n²)** behavior overall.

To handle repeated modifications efficiently, use **`StringBuilder`**:

```csharp
var sb = new StringBuilder("a");
for (int i = 0; i < 1000; i++)
{
    sb.Append("x");
}
string result = sb.ToString();
```

👉 **Key takeaway:** Strings are simple to use and safe thanks to immutability, but for heavy modifications or dynamic content, prefer **`StringBuilder`**. Be aware of interning for memory-sensitive scenarios.

---

### Interning and Memory

* **String interning** is a mechanism where the runtime stores a single copy of each **literal string** in a global pool.

  ```csharp
  string a = "hello";
  string b = "hello";
  Console.WriteLine(object.ReferenceEquals(a, b)); // True
  ```
* **Benefit:** Saves memory for repeated literals.
* **Caution:** Interning does **not apply** to strings created at runtime (e.g., via `new string(...)` or concatenation) unless you explicitly intern them. Overusing interning for dynamically generated strings can actually **increase memory usage**.

---

## 5. Example: LeetCode – Replace Words

Problem link: [Replace Words](https://leetcode.com/problems/replace-words/description/)
Goal: Replace words in a sentence with the **shortest root** from a dictionary if one exists.

### **Efficient C# Solution**

```csharp
public class Solution
{
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

    private string ShortestRoot(string word, HashSet<string> dictSet)
    {
        // Check every prefix of the word
        for (int i = 1; i <= word.Length; i++)
        {
            string root = word.Substring(0, i);
            if (dictSet.Contains(root))
            {
                return root; // Return the shortest matching root
            }
        }

        // No matching root found, return the original word
        return word;
    }
}
```

---

## 6. Summary and What’s Next

In this post, we’ve explored the fundamentals of **strings in C#**, covering both theory and practical usage:

* **String Basics** – How strings are immutable, reference types, and indexable like collections.
* **Core Operations** – Concatenation, `Replace`, `Remove`, `Substring`, `Split`/`Join`, searching, formatting, and conversion.
* **Performance Considerations** – Why repeated concatenation can be costly and how to use `StringBuilder` efficiently; the role of string interning in memory optimization.
* **Practical Example** – A LeetCode-style problem (`Replace Words`) to show how to apply these concepts in a real scenario, comparing straightforward vs efficient string handling.

Strings are everywhere in C# programming, and understanding their behavior and performance implications is key to writing **clean, readable, and efficient code**.

🔹 **Coming up next:** In **Part II**, we’ll move beyond strings and dive into **arrays, lists, and hash sets** — exploring how to store, organize, and manipulate larger collections of data in C#, and when to choose each type.

Stay tuned to learn how the same principles you’ve mastered with strings extend to broader collections, and how to write even more efficient, robust code!
