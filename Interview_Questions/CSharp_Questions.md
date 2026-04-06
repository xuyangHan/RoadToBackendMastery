# 📚 C# Traps & Edge Cases — Part 1: From Basics Through Async

This article is **Part 1** of a two-part series. It covers **Sections 1–7** (types through **`async`/`Task`**). **[Part 2](CSharp_Questions_Part2.md)** continues with OOP, generics, disposal, patterns, and a cheat sheet—so neither post tries to do everything at once.

## 1. 🚀 Introduction

C# often looks straightforward on the page. Many interview questions use small snippets that behave in a way you did not expect. That gap usually comes from rules the language applies for you—defaults, type rules, and timing you never had to spell out in code.

This guide is a review of those “traps”: things that are easy to get wrong even if you rarely write them by hand in day-to-day work. The point is not to memorize every edge case. It is to know *why* the language does what it does so you can reason through new questions calmly.

Two ideas show up again and again:

* **Compile time vs run time.** The compiler accepts some code or chooses overloads and types before your program runs. Other behavior only appears when the code executes (nulls, virtual calls, deferred LINQ, async continuations). Confusing those two moments is a common source of surprises.

* **Hidden defaults.** Fields, generics, enums, and collections all have default values and rules you might never have written explicitly. When a question assumes those defaults, the answer can look “wrong” until you remember what the language filled in for you.

---

## 2. 🧱 Data Types & Defaults (Foundation)

Most “trick” questions about C# still come back to types: what gets copied, what can be null, and what the language assumes when you do not write a value explicitly. This section is that background in one place.

**Value types and reference types** are the first split. Value types (such as `int`, `bool`, `DateTime`, structs) are usually copied when you assign or pass them around; each variable holds its own bits. Reference types (classes, interfaces, delegates, arrays) store a reference to an object; copying the variable copies the reference, not a full duplicate. Interviews often check whether you know *when* assignment shares one object versus two.

**Defaults and `default`.** Every type has a default value: numeric zero, `false` for `bool`, `null` for reference types, and “all fields set to *their* defaults” for structs. You can write `default(T)` or, when the compiler already knows `T`, the `default` literal. Remember the difference between **fields** and **local variables**: fields get default values automatically if you do not assign them; locals must be definitely assigned before use, or the compiler reports an error.

```csharp
int a = default(int);   // 0
int b = default;        // same, when type is inferred from the left-hand side
string s = default;     // null for a reference type

class Example
{
    int count;            // 0
    bool flag;            // false
    decimal amount;       // 0m
    double ratio;         // 0d
    DateTime when;        // DateTime.MinValue (0001-01-01, 00:00:00)
    string label;         // non-nullable `string` (NRT): same runtime default as any reference field (null) until you assign; compiler may warn (CS8618)
    string? name;         // nullable `string?`: default null; meaning is “may be null” in the type system
    int? optional;        // null — no value
    Guid id;              // Guid.Empty
    char ch;              // '\0' (U+0000)

    struct Pair { public int A; public string? B; }
    Pair pair;            // A = 0, B = null

    void M()
    {
        Console.WriteLine($"count    = {count}");       // output: count    = 0
        Console.WriteLine($"flag     = {flag}");       // output: flag     = False
        Console.WriteLine($"amount   = {amount}");     // output: amount   = 0
        Console.WriteLine($"ratio    = {ratio}");      // output: ratio    = 0
        Console.WriteLine($"when     = {when:O}");     // output: when     = 0001-01-01T00:00:00.0000000
        Console.WriteLine($"label    = {(label is null ? "null" : label)}"); // output: label    = null
        Console.WriteLine($"name     = {(name is null ? "null" : name)}");     // output: name     = null
        Console.WriteLine($"optional = {(optional.HasValue ? optional.Value.ToString() : "null")}"); // output: optional = null
        Console.WriteLine($"id       = {id}");         // output: id       = 00000000-0000-0000-0000-000000000000
        Console.WriteLine($"ch       = {(ch == '\0' ? "\\0 (nul)" : ch.ToString())}"); // output: ch       = \0 (nul)
        Console.WriteLine($"pair     = (A={pair.A}, B={(pair.B is null ? "null" : pair.B)})"); // output: pair     = (A=0, B=null)

        int local;
        // Console.WriteLine(local); // error: local not definitely assigned
        local = default;
        Console.WriteLine($"local (after assign) = {local}"); // output: local (after assign) = 0
    }
}
```

Things interviewers like to tie to defaults and types:

* **`string`.** It is a reference type, but it behaves like a value in many APIs: immutable, and `==` compares character content, not reference identity. That trips people who only memorized “reference types use reference equality.”

* **`DateTime`.** A plain `DateTime` is a value type and is *never* null. If you need “no date,” use `DateTime?` (`Nullable<DateTime>`). Comparing `DateTime` to `null` is a compile-time mistake unless you use the nullable form.

* **`Nullable<T>` (`int?`, etc.).** Wraps a value type so it can represent “no value” alongside normal values. Know how it lifts operators and how `HasValue` / `Value` work when you move past basics.

* **Boxing and unboxing.** Putting a value type in an `object` or a non-generic interface slot allocates a boxed copy on the heap. That costs memory and can change how equality behaves (for example, comparing boxed values). Hot paths and subtle bugs both show up here.

    ```csharp
    int x = 5;
    object boxed = x;        // boxing: int copied to object on the heap
    int y = (int)boxed;      // unboxing: object cast back to int

    Console.WriteLine(x == y);        // True: compares values (both ints)
    Console.WriteLine(boxed == (object)y); // False: different boxed objects (references not the same)


    // Even when values are equal, boxed objects are different instances!

    void Log(object value)
    {
        Console.WriteLine(value);
    }

    Log(42); // ⚠️ boxing happens here
    ```

* **`decimal` vs `double`.** `double` is binary floating point: fast, but fractions like `0.1` are not always exact, so `0.1 + 0.2` is a classic surprise. `decimal` is decimal floating point and is the usual choice for money and exact decimal math, at some performance cost.

* **`const` and `readonly`.**  
  - `const` must be a compile-time constant (literals and simple combinations).  
  - `readonly` fields are set once—in an instance constructor or static constructor for static fields.  

```csharp
public class Example
{
    public const int ConstValue = 10; // replaced by value at compile time, must be constant
    public readonly int ReadonlyField; // can only set in constructor

    public int NormalField; // can be changed at any time

    public Example(int val)
    {
        ReadonlyField = val; // Allowed: set in the constructor
        // ConstValue = 20;   // ❌ Error: cannot assign to const
    }
}
```

**Summary Table**

| Feature   | Set When?                       | Modifiable After Init? | Usage Example            |
|-----------|---------------------------------|------------------------|--------------------------|
| `const`   | At declaration (compile time)   | ❌ No                  | `public const int X = 1;`  |
| `readonly`| In constructor (run time)       | ❌ No                  | `public readonly int Y;`   |

* **Enums.** The default numeric value is `0` even if you never defined a name for zero. Casts from integers can produce values that are not named; 

```csharp
enum Color { Red = 1, Green = 2, Blue = 4 }
Color c1 = (Color)0;         // 0 is valid, but not named
Color c2 = (Color)7;         // 7 is not a named value, but is valid at runtime
```

**Nullable reference types (NRT)** are a compile-time layer on top of reference types: `string` means “should not be null,” `string?` means “may be null,” and the compiler warns when you ignore that. The `!` operator only tells the compiler to stop warning; it does not add a runtime check. **Section 5** goes deeper into null handling and how NRT fits day-to-day code.

Common traps:

* **`null` vs empty string.** `null` is “no string”; `""` is a real string with length zero. Under NRT, assigning `null` to a non-nullable `string` is a warning (or error if you treat warnings as errors).

* **`DateTime` and null.** Use `DateTime?` or check against `default(DateTime)` when the scenario needs “missing,” not `null` on a non-nullable `DateTime`.

* **Arrays after creation.** A new array of a reference type fills slots with `null`; a new array of a value type fills slots with that type’s default bits.

* **Locals vs fields.** Expecting a local variable to “default” like a field leads to compile errors; expecting a field to behave like a definite-assignment local leads to subtle zero or null bugs.

* **Boxing in comparisons.** Equality and identity can surprise you when one side is boxed and the other is not; prefer clear types and unboxed comparisons when it matters.

---

## 3. ➕ Operators & Expressions

Expressions look small, but the compiler is busy: it picks overloads, promotes types, and applies rules you never wrote down. Interview questions often exploit that gap—especially when `+`, `==`, or `switch` do something that “feels” like math or equality in English but means something stricter in C#.

**Pattern matching** is where the language inspects a value’s type at run time and binds names in one step. You will see `is` with type checks. **Classic `switch`** statements are statement-based and fall through unless you `break` or `return`. With **enums**, the compiler does **not** warn if you do not cover every named value—casts from integers can still produce invalid enum values at run time, so “exhaustive” handling cannot be guaranteed. **`when` guards** can add extra conditions on each `case` without nesting huge `if` chains.

```csharp
// `is` type check only (pre-C# 8)
object obj = "hello";
if (obj is string s)   // type check + bind to s
{
    if (s.Length > 0)  // separate check for length
        Console.WriteLine(s);
}

// Classic switch statement with `when` guards (pre-C# 8)
static string Describe(object x)
{
    switch (x)
    {
        case int n when n < 0:
            return "negative";
        case int n:
            return $"int {n}";
        case null:
            return "null";
        default:
            return "other";
    }
}

// Enum example (pre-C# 8)
enum Level { Off = 0, On = 1 }

Level rogue = (Level)99; // cast from int, may not be named

string label;
switch (rogue)
{
    case Level.Off:
        label = "off";
        break;
    case Level.On:
        label = "on";
        break;
    default:
        label = "not a named member"; // catch any invalid enum
        break;
}
```

**`checked` and `unchecked`.** Integer arithmetic normally wraps in an *unchecked* context (the default for plain `int` code unless the project sets checked arithmetic). In a *checked* context, overflow throws `OverflowException` instead of silently wrapping. Whole-project settings (`<CheckForOverflowUnderflow>` or legacy `/checked`) can flip what you expect from a snippet on a whiteboard.

```csharp
int max = int.MaxValue;

// Default (unchecked): wraps around on overflow (no error)
int wrapped = max + 1;  // wraps to int.MinValue (overflow, but no exception)
Console.WriteLine(wrapped); // Output: -2147483648

// Checked: throws on overflow
try
{
    int result = checked(max + 1); // throws OverflowException
}
catch (OverflowException)
{
    Console.WriteLine("Overflow detected!");
}

// You can also force no exception with 'unchecked':
int noError = unchecked(max + 1); // always wraps, never throws
```

Common traps:

* **`char` and `+`.** `'a' + 'b' + 'c'` promotes to `int` (Unicode code points) and adds: 97 + 98 + 99 = **294**, not a string. Only when a `string` appears in the chain does `+` become concatenation for the rest of the expression as usual.

* **Strings vs numbers.** `"1" + 2` is string concatenation (`"12"`). `1 + 2 + "3"` evaluates left to right: first `3`, then `"33"`. The fix for “I meant math” is parentheses or explicit parsing.

* **`==` vs `.Equals()`.** For many types, `==` is overloaded or lifted (nullable types). **`x.Equals(y)` throws if `x` is null**; **`Equals(x, y)`** (static on `object`) or **`string.Equals(a, b)`** handles nulls safely. Know which side can be null before you pick a form.

* **Reference equality vs value equality.** Two different strings with the same characters may or may not share reference equality; `string` compares value for `==`. For your own types, behavior depends on whether you override `Equals`/`GetHashCode` and how you use `==` (reference equality unless overloaded).

```csharp
// --- Null-safety: who is allowed to be null? ---
string? a = null;
string b = "hi";

// a.Equals(b);                 // NullReferenceException — instance method on null
bool ok1 = b.Equals(a);        // false; `b` is not null, so the call is safe
bool ok2 = object.Equals(a, b); // false; static helper, either side may be null
bool ok3 = string.Equals(a, b); // false; same idea for strings
bool ok4 = a == b;             // false; string `==` allows null on the left

// --- Same text, different objects: value equal, reference not equal ---
string s1 = new string(new[] { 'a', 'b', 'c' });
string s2 = new string(new[] { 'a', 'b', 'c' });
bool sameChars = s1 == s2;                 // true — string overloads `==` by content
bool sameInstance = ReferenceEquals(s1, s2); // false — two heap objects

// Typed as `object`: `==` is reference equality only (no string overload applies)
object o1 = s1;
object o2 = s2;
bool objectOpEquals = o1 == o2;            // false — different references
bool objectEquals = o1.Equals(o2);       // true — run-time type is still `string`

// Another reference type with no `==` overload: same “shape,” different identity
var list1 = new List<int> { 1, 2, 3 };
var list2 = new List<int> { 1, 2, 3 };
bool listsSameRef = list1 == list2;      // false — two `List<>` instances
bool listsSameElements = Enumerable.SequenceEqual(list1, list2); // true — same sequence
// For your own `class`, `==` is reference equality too unless you overload it.
```

* **Events vs delegate fields.** An **`event`** is not a normal field from the outside: other types can only `+=` and `-=` subscribers, not assign or invoke it like a bare delegate. Only the declaring type can raise it. Treating an event like a regular delegate variable in another class fails by design. **[Part 2](CSharp_Questions_Part2.md)** (Section 11) picks up delegates, multicast, and the `EventHandler` pattern in more detail.

---

## 4. 🔁 Collections & LINQ

Collections hold data; **LINQ** describes *how* to walk and filter that data. A common trap is forgetting that many LINQ operations return **lazy** `IEnumerable` pipelines: nothing runs until something **pulls** the sequence (for example `foreach`, `ToList()`, or another consuming operator). Until then, the “query” is a recipe, not a snapshot. That ties back to **run time**: the source or even variables used inside your query can change before you enumerate, and enumerating twice can run the whole pipeline twice.

For example:

```csharp
int b = 3;
var query = list.Where(a => a > b); // defines a query, but does NOT run it

b = 1; // b changes AFTER query is defined, but BEFORE enumeration
var results = query.ToList(); // filters with b == 1, not 3!
```

In this example, the value of `b` used in the filter is not "captured" at the time you create the query, but rather at the time you enumerate it. This is a classic demonstration of deferred execution and can lead to surprises if you expect the query to be a snapshot of data and variables "at definition."

**Operators that throw** encode strict expectations. **`First()`** demands at least one element; **`FirstOrDefault()`** returns `default(T)` when the sequence is empty—pick the one that matches whether “none” is normal or exceptional. **`Single()`** insists on **exactly one** element (throws if there are zero or more than one). **`SingleOrDefault()`** allows **zero or one** (returns `default(T)` when empty, throws when two or more match). That makes `Single*` a poor choice when the sequence was never meant to be unique. **`Last()`**, **`ElementAt(index)`**, and similar APIs often have `*OrDefault` variants—same idea: throw versus default when the position or uniqueness rule fails.

```csharp
using System.Linq;

var empty = Array.Empty<string>();
// empty.First();              // InvalidOperationException — sequence is empty
var a = empty.FirstOrDefault(); // null; for `int` sequences, empty → `0` (easy to confuse with a real zero)

var one = new[] { "only" };
var b = one.Single();           // "only"
// one.SingleOrDefault();      // same — still exactly one element

var none = Array.Empty<string>();
// none.Single();              // InvalidOperationException — zero elements
var c = none.SingleOrDefault(); // null — zero elements allowed

var two = new[] { "x", "y" };
// two.Single();               // InvalidOperationException — more than one
// two.SingleOrDefault();      // same — still more than one

var nums = new[] { 10, 20, 30 };
// nums.ElementAt(10);        // ArgumentOutOfRangeException
var d = nums.ElementAtOrDefault(10); // 0 — default for `int`, not “missing” semantics for value types

// nums.Last();               // 30
// Array.Empty<int>().Last(); // InvalidOperationException
```

**Equality inside LINQ** follows the type’s rules, not what you wish it meant. **`Distinct()`** uses the element type’s default equality (for reference types, that usually means **`Equals`/`GetHashCode`** if overridden; otherwise reference identity). Two different object instances with the same “business” fields can still count as distinct. If you need value semantics, you override equality, use a projection (`Select`) into something comparable, or pass an **`IEqualityComparer<T>`**.

```csharp
class Person
{
    public string Name;
    public int Age;

    // public override bool Equals(object obj)
    //    => obj is Person p && p.Name == Name && p.Age == Age;

    // public override int GetHashCode() => HashCode.Combine(Name, Age);
}

var list = new List<Person>
{
    new Person { Name = "Alice", Age = 25 },
    new Person { Name = "Alice", Age = 25 }
};

var distinct = list.Distinct();
Console.WriteLine(distinct.Count()); // 2 ❌
```

**Dictionaries** map keys to values. **`Add(key, value)`** throws if the key already exists; the **indexer** `dict[key] = value` **replaces** the value when the key is present—easy to mix up in interviews. **`TryGetValue`** fetches in one lookup and tells you whether the key was found; calling the indexer twice or using `ContainsKey` plus indexer does extra work and can still race if another thread mutates the dictionary (for concurrent scenarios, different types apply). **Null keys** depend on the dictionary: **`Dictionary<TKey,TValue>`** does not allow a null key when `TKey` is a non-nullable reference type; with nullable keys, `null` may be allowed or may be a deliberate “missing key” smell—know your contract.

Things interviewers like to probe in collections and LINQ:

* **Throwing vs non-throwing queries.** When is “no match” an error (`First`) versus a normal outcome (`FirstOrDefault`)? When is uniqueness a real invariant (`Single`) versus an assumption that breaks on real data?

* **Deferred execution.** Chained `Where`/`Select` builds a pipeline; side effects in lambdas run **each time** you enumerate. **`ToArray()` / `ToList()`** copy results **once** at that moment—after that, the list does not “follow” later changes to the original source unless you rebuild.

* **Multiple enumeration.** Storing `IEnumerable<T>` and enumerating it twice can mean **two full passes** (and **two rounds** of any expensive logic or side effects). If you need a stable snapshot, materialize.

Common traps:

* **`foreach` and mutation.** Changing a collection’s structure (adding/removing) while iterating often throws **`InvalidOperationException`** (“collection was modified”). Use a copy, a `for` loop over indices with care, or types designed for concurrent modification.

* **Results that “move.”** If you build a query over a **mutable** list and enumerate **after** the list changes, you see the **current** contents—not the contents at the time you wrote the query. Materialize early when you need a fixed picture.

* **Empty vs null.** A **null** collection variable is not the same as an **empty** sequence; LINQ extension methods throw on `null` unless you guard first.

---

## 5. ⚠️ Null Handling

**`null`** means “no object” for reference types, and **`Nullable<T>`** (`int?`, and so on) means “maybe no value” for value types. Interview questions often mix those ideas with collections and strings, so the goal here is the same as elsewhere: know what runs at **run time** versus what the **compiler** can only warn about.

**`NullReferenceException`** is what you get when you use a member on a reference that is **`null`**—for example, if you try to call a method, read a property, or index as if the reference pointed at an object, but in reality it is `null`.

```csharp
string s = null;
Console.WriteLine(s.Length); // Throws NullReferenceException
```

The fix is not “catch every NRE”; it is to **test**, use **null-conditional** access, or **guarantee** invariants at construction time (constructors, factories, APIs that never return null when the contract says they must not).

**The null-conditional operator `?.`** stops the chain as soon as one step is **`null`**. The rest of the expression is skipped, and the whole expression evaluates to **`null`** when the final result type is a reference type or nullable value type. **`?.` chained** (`a?.b?.c`) is one short pattern for deep graphs without nested `if` noise—just remember **short-circuit** behavior: later parts never run if an earlier part was **`null`**.

```csharp
class Node { public Node? Next; public string? Label; }

Node? head = null;

string? name = head?.Next?.Label; // Evaluates to null, no exception is thrown.
```

If you wrote this without the null-conditional operator, you'd need nested `if` checks to avoid a `NullReferenceException`.

**`??` (null-coalescing)** picks the right-hand side when the left is **`null`**. **`??=`** assigns the right-hand side **only if** the left is **`null`**—handy for lazy defaults on fields and locals. Neither operator removes **`null`** from the world; they only choose a fallback at that moment in the code.

```csharp
string? name = null;
string display = name ?? "Unknown";  // "Unknown" if name is null

string? label = null;
label ??= "Default Label";           // label is now "Default Label"
```

**Nullable reference types (NRT)** add **`?`** on reference types (`string?`) and warnings when you might dereference **`null`**. That is **not** a runtime feature: **`null` can still appear** where the type says it should not, if you disable checks, use legacy libraries, or lie to the compiler. The **null-forgiving postfix `!`** means “I assert this is not null here”—it **silences a warning**; it does **not** insert a check. If you were wrong, you still get **`NullReferenceException`** at run time. **Section 2** already tied **`string`** / **`string?`** and field defaults to this picture.

**Mental model (same story as Section 2).** For **value types**, “missing” is **`T?`** / **`Nullable<T>`** with **`HasValue`**. For **reference types**, **`null`** is always a possible run-time value until your design rules it out; NRT is there to **catch mistakes early**, not to erase **`null`** from the CLR.

Things interviewers like to probe in null handling:

* **Where the exception actually throws.** A long chain of calls: which dereference was **null**? Can you rewrite with **`?.`** or split into locals so the next reader (and debugger) can see it?

* **“No” vs “empty.”** **`null`**, **empty string**, and **empty collection** answer different questions; APIs should say which they use, and callers should not treat them as interchangeable.

* **NRT discipline.** When is **`!`** justified (after a guard you truly trust) versus papering over warnings? When should the return type be **`string?`** so callers are forced to think?

Common traps:

* **`list == null` vs empty list.** **`Count == 0`** does not tell you the variable is non-null; check **`null`** first, or use **null-conditional** (`list?.Count`) and handle both “no list” and “no items.”

* **`string.IsNullOrEmpty`** and **`IsNullOrWhiteSpace`.** Treat **`null`**, **`""`**, and (for whitespace) **`"   "`** in one call when that matches your business rule—do not reimplement with easy-to-miss edge cases.

* **Assuming NRT proves safety at run time.** Warnings are only as good as the annotations and the code paths you analyzed; **`null`** from interop, serialization, or reflection still happens.

---

## 6. 🔥 Exception Handling

Exceptions are for **exceptional** control flow: failures you did not want in the happy path. In interviews—and in production—the recurring theme is whether you **preserve information** (especially the **stack trace**), whether you **catch the right thing**, and whether **`finally`** and **cancellation** still behave the way readers expect.

**`throw;` vs `throw ex;`.**  
Inside a **`catch`**, bare **`throw;`** rethrows the **same** exception object and **keeps the original stack trace**—that is what you usually want when you are not transforming the error. **`throw ex;`** (or **`throw new Exception(..., ex)`** without care) often **rewinds** the stack to the current frame, which makes production logs far harder to read. Wrap with a **new** exception type only when you are adding context and you **chain** with **`InnerException`** (or equivalent) so the old stack is not lost.

```csharp
try
{
    throw new InvalidOperationException("Original error.");
}
catch (Exception ex)
{
    // Rethrows the SAME exception; stack trace is preserved:
    throw;
    // If you do:
    // throw ex; 
    // ...the stack trace will now start from THIS line, not the original throw above.
}

// OR: new exception but WITH inner exception for context:
try
{
    throw new InvalidOperationException("Original error.");
}
catch (Exception ex)
{
    throw new ApplicationException("Extra context here.", ex); // Chains, stack is preserved in 'ex'
}
```

---

**`catch` order and `catch when`.**  
The runtime matches **`catch`** blocks **top to bottom**; the **first compatible type** wins. Put **more specific** types **before** **`catch (Exception)`**, or the general handler will swallow things you meant to handle differently. **`catch (Exception ex) when (condition)`** filters **without** changing which exception type you caught—you still catch **`Exception`**, but only when **`condition`** is true. That avoids awkward **`catch`** / rethrow patterns when you only wanted a subset of cases.

```csharp
try
{
    // Code that might throw any type of exception
}
catch (ArgumentNullException ex) // Specific type FIRST
{
    Console.WriteLine("Caught ArgumentNullException: " + ex.Message);
}
catch (Exception ex) // More general handler
{
    Console.WriteLine("Caught some other exception: " + ex.Message);
}

// Using 'catch when' filter
try
{
    // Code that may throw
}
catch (Exception ex) when (ex.Message.Contains("transient")) // Only when the message matches
{
    Console.WriteLine("Transient error, try again maybe.");
}
```

---

**`finally` and control flow.**  
A **`finally`** block **runs** when control leaves the associated **`try`**—whether that is by **normal completion**, **`return`**, **`break`**, **`continue`**, or a **thrown** exception (unless the process tears down first). If **`try`** contains **`return`**, **`finally`** still runs **before** the method actually returns to the caller; side effects in **`finally`** can still run after the return value is chosen—know the language rules before you argue about quiz output.

```csharp
int TestFinally()
{
    try
    {
        return 42; // 'finally' will STILL RUN
    }
    finally
    {
        Console.WriteLine("Finally block runs, even after return.");
    }
} // Console writes before method returns.

void TestBreakContinue()
{
    for (int i = 0; i < 3; i++)
    {
        try
        {
            if (i == 1) break;
            if (i == 2) continue;
        }
        finally
        {
            Console.WriteLine($"Finally runs for i={i}");
        }
    }
}
// Output will include 'finally' for each loop iteration, even if 'break' or 'continue' occurs.
```

---

**Design habits.**  
Wrapping with **`InnerException`** carries the original cause; a bare **`throw;`** says “this layer has nothing to add.” Many systems **log once at a boundary** (HTTP middleware, top-level handler) instead of **catching everywhere**. **Empty `catch`** blocks and **`catch (Exception)`** that only **swallow** hide bugs—cancellation and **`OutOfMemoryException`** included. **`ExceptionDispatchInfo.Capture(ex).Throw()`** (or related patterns) can **rethrow with the original stack** when you had to store an exception and throw later—for example across some **`await`** boundaries—without rewriting the stack the way **`throw ex;`** does.

```csharp
// Bad: empty catch block swallows all exceptions (can hide bugs)
try
{
    DangerousStuff();
}
catch (Exception)
{
    // Catches and ignores EVERY exception (including OutOfMemory, which you probably shouldn't)
}

// Good: wrap with context and preserve cause
try
{
    DangerousStuff();
}
catch (Exception ex)
{
    throw new ApplicationException("Something failed in DangerousStuff.", ex);
}

// Rethrow with preserved stack after storing exception (advanced/async case)
using System.Runtime.ExceptionServices;

ExceptionDispatchInfo captured = null;
try
{
    DangerousStuff();
}
catch (Exception ex)
{
    captured = ExceptionDispatchInfo.Capture(ex); // Stores exception for later
}

// ... later in the code, perhaps after an 'await':
if (captured != null)
    captured.Throw(); // Throws the original, with original stack trace!
```

---

**Cancellation** is normal control flow in modern C#, not always a “bug.” **`OperationCanceledException`** and **`TaskCanceledException`** often mean “this work was stopped on purpose.” Catching **`Exception`** and **ignoring** them breaks **cooperative cancellation** unless you **rethrow** or filter. Prefer **`CancellationToken.ThrowIfCancellationRequested()`** and passing **tokens** through APIs; **Section 7** goes deeper on **async** and tokens.

```csharp
void CancelMe(CancellationToken token)
{
    // Correct: honor token cancellation
    token.ThrowIfCancellationRequested();
    // Do cancelable work...
}

try
{
    var cts = new CancellationTokenSource();
    cts.Cancel();
    CancelMe(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Cancellation was requested and respected.");
}

// BAD: swallowing ALL exceptions breaks cancellation
try
{
    var cts = new CancellationTokenSource();
    cts.Cancel();
    CancelMe(cts.Token);
}
catch (Exception) // Should filter or rethrow for cancellation, not swallow!
{
    // Now cancellation is hidden -- the caller can't tell work was cancelled
}
```

---

**Resources and disposal** belong in the same mental bucket: **`using`** maps to **`try` / `finally` / `Dispose()`** so cleanup runs even when **`try`** throws.

```csharp
using (var stream = new FileStream("myfile.txt", FileMode.Open))
{
    // Work with the file
    // If an exception is thrown here, Dispose() is still called (file closed) before propagating error
}
// Equivalent to:
FileStream stream = null;
try
{
    stream = new FileStream("myfile.txt", FileMode.Open);
    // Work with the file
}
finally
{
    if (stream != null)
        stream.Dispose();
}
```

---

Things interviewers like to probe in exception handling:

* **Stack integrity.**  
  Why does **`throw;`** preserve the stack while **`throw ex;`** usually does not?  
  (See code samples above.)

* **`finally` quiz questions.**
  What runs, and in what order, when **`try`** returns or throws?  
  Can **`finally`** override the return value? (No, but it can throw another exception, which replaces the original return/exception.)

* **Filtering.**  
  **`catch when`** vs catching and rethrowing after an **`if`**—when does each read more clearly?
  ```csharp
  // Filtering with 'catch when'
  try
  {
      PossiblyTransientWork();
  }
  catch (Exception ex) when (IsTransient(ex))
  {
      Console.WriteLine("Handle only transient errors.");
  }

  // Equivalent but noisier:
  try
  {
      PossiblyTransientWork();
  }
  catch (Exception ex)
  {
      if (IsTransient(ex))
          Console.WriteLine("Handle only transient errors.");
      else
          throw;
  }
  ```


Common traps:

* **Broad catch as a default.** **`catch (Exception)`** that logs nothing and continues—especially around **async** or **cancel**—masks real failures.

* **Swallowing cancellation.** Treat **cancel** as a first-class outcome; filter or rethrow unless the layer truly owns the lifetime of the work.

* **Dropping the inner cause.** New wrapper exceptions without an **`InnerException`** link erase the chain debugging tools rely on.

---

## 7. ⚡ Async / Task Pitfalls

**`async` / `await`** splits work across time: an `async` method can **yield** while waiting for I/O or other tasks, without blocking a thread for the entire wait. Understanding traps is all about what the **caller** actually observes: return type, exceptions, ordering, and when you accidentally block a thread the code still needs.

---

**Return types: `async void` vs `async Task`**

*Use `async void` almost only for event handlers.* Callers **cannot `await`** an `async void` method, so errors go to process-wide handlers—not back to the caller. For normal APIs, always return `Task` or `Task<T>`. Exceptions in `async Task` methods are captured on the `Task` and rethrown on `await`.
```csharp
// BAD: async void as a regular API means the caller can't await or catch errors cleanly
public async void DoWorkAsync_Bad()
{
    await Task.Delay(1000);
    throw new Exception("Error is uncatchable by caller!");
}

// GOOD: async Task API lets callers await and catch exceptions
public async Task DoWorkAsync_Good()
{
    await Task.Delay(1000);
    throw new Exception("Error will be caught by awaiter.");
}

// Typical event handler pattern (only here is async void okay)
private async void Button_Click(object sender, EventArgs e)
{
    await DoWorkAsync_Good();
}
```

---

**Forgetting `await`**

If you call an `async` method but forget to `await`, you get a `Task` that may still run (**fire-and-forget**). Exceptions are delayed, and the rest of your method may run out of order.
```csharp
public async Task ExampleAsync()
{
    Task t = DoWorkAsync_Good();  // Forgot 'await'! The work runs in the background.

    // The following line may run BEFORE DoWorkAsync_Good() is done (or even started).
    Console.WriteLine("This might print before work completes!");

    // The exception from DoWorkAsync_Good() may go unobserved unless you later await:
    await t; // Only here will the exception get thrown to your code.
}
```

---

**Sync-over-async and deadlocks**

Calling `.Result` or `.Wait()` on an async method **blocks** the calling thread, risking a deadlock—especially in GUI or ASP.NET SynchronizationContext scenarios. Always prefer awaiting directly.
```csharp
public async Task DeadlockExample()
{
    // This is fine in a console app, but on a UI thread, this can deadlock:
    // Don't do this in WinForms/WPF/ASP.NET!
    Task t = DoWorkAsync_Good();

    // BLOCKS the current thread!
    t.Wait();  // Or: var result = t.Result;
    // If DoWorkAsync_Good tries to do work on this context, it will hang forever.
}

// Fix: use await all the way
public async Task NoDeadlockExample()
{
    await DoWorkAsync_Good(); // Never blocks the thread!
}

// If context resume is not needed, use ConfigureAwait(false) in libraries
public async Task LibraryCode()
{
    await DoWorkAsync_Good().ConfigureAwait(false); // Won't marshal back to capture context
}
```

---

**Cancellation**

Make async cancellation **cooperative**: pass a CancellationToken, use `ThrowIfCancellationRequested`, and honor the token.
```csharp
public async Task WorkWithCancelAsync(CancellationToken token)
{
    for (int i = 0; i < 5; i++)
    {
        token.ThrowIfCancellationRequested(); // Throws if cancelled
        await Task.Delay(500, token);         // Passes token to delay
    }
}

// Usage:
CancellationTokenSource cts = new CancellationTokenSource();
var task = WorkWithCancelAsync(cts.Token);
cts.Cancel(); // Cancel whenever needed
```

---

**`IAsyncEnumerable<T>`** (async streaming)

Use `IAsyncEnumerable<T>` for async streaming: items are produced and consumed over time, with `await foreach`. Execution starts as the consumer iterates.
```csharp
public async IAsyncEnumerable<int> CountAsync()
{
    for (int i = 0; i < 3; i++)
    {
        await Task.Delay(1000); // Pretend async work per item
        yield return i;
    }
}

public async Task Consume()
{
    await foreach (var x in CountAsync())
    {
        Console.WriteLine(x); // Prints 0, then 1, then 2, with delays
    }
}
```
*Deferred execution applies: nothing runs in CountAsync until you enter the `await foreach` loop.*


Things interviewers like to probe in async code:

* **Why `async void` is a bad default.** What can the caller not do? Where do failures go?

* **Deadlock stories.** A **UI** handler blocks on **`.Result`** while the inner work **`await`s** back to the **UI** thread—why can that hang?

* **`ConfigureAwait(false)`** — what problem does it reduce, and when is it irrelevant or the wrong tool?

Common traps:

* **`async void`** in **library** or **business** code “because it compiles.”

* **Blocking** on **`.Result` / `.Wait()`** in **request** or **UI** paths that already sit on a **captured** context.

* **Tokens dropped** after the first layer—every **downstream** method that could run long should accept and **use** the same **`CancellationToken`** when possible.

---

## 🏁 Conclusion (end of Part 1)

You have walked through the traps that dominate **language-level** interviews: **types and defaults**, **operators and equality**, **LINQ**, **nulls**, **exceptions**, and **`async`/`Task`**. If those ideas feel connected—not memorized in isolation—you are ready for a large slice of C# screening questions.

**Continue with [Part 2](CSharp_Questions_Part2.md)** for **inheritance and polymorphism**, **`static`**, **generics, delegates, and events**, **`using` / disposal**, a **modern C#** snapshot, optional **SQL** notes, **real-world patterns**, and a **cheat sheet** for a last pass before you talk to someone.

> “Most bugs don’t come from complex code, but from misunderstood simple behavior”—and that line still applies after Part 1; Part 2 adds the object-model and architecture layers where those bugs show up at scale.
