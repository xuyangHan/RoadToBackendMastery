# Using C# Dictionary to Improve Code Performance: LeetCode Example Analysis

When optimizing C# code, selecting the right data structure is critical. `Dictionary` is one of the most powerful and commonly used structures. This article explores how to leverage `Dictionary` effectively, its advantages, and demonstrates its performance benefits through a LeetCode example, especially useful during coding interviews.

## What is a Dictionary?

In C#, a `Dictionary` is a collection of key-value pairs. Each key in the dictionary is unique and maps to a specific value. Internally, `Dictionary` uses a hash table, which gives it an average time complexity of **O(1)** for lookups, insertions, and deletions. This is in sharp contrast to lists or arrays, where these operations may require **O(n)** time, especially for larger datasets.

---

## Why Choose Dictionary?

The primary advantage of a `Dictionary` lies in its performance for key-based lookups. Here are some key scenarios where `Dictionary` offers significant benefits and why you should consider using it:

1. **Fast Lookups**:  
   If you need to frequently retrieve values based on unique keys, such as looking up a user by ID, `Dictionary` is ideal. It offers an average **O(1)** lookup time, much faster than searching through a list.

2. **Efficient Insertions and Deletions**:  
   When adding or removing elements frequently, `Dictionary` outperforms lists, which may require shifting elements or finding indexes. `Dictionary` allows efficient insertions and deletions without re-indexing.

3. **Key-Value Mapping**:  
   If your data naturally fits into a key-value pair structure (e.g., mapping usernames to email addresses or product IDs to product details), `Dictionary` is the go-to data structure. It enables efficient storage and retrieval using unique keys.

4. **Avoiding Duplicates**:  
   Since `Dictionary` keys must be unique, it is extremely useful for scenarios where duplicate keys should be prevented. For example, ensuring user IDs or product codes are unique is straightforward with `Dictionary`.

5. **Dynamic and Flexible Data**:  
   For dynamic datasets where the size of elements is not predetermined, `Dictionary` is invaluable. Unlike arrays, it does not require specifying an initial size and can grow dynamically as data is added.

6. **Sparse Data Sets**:  
   If your dataset is large but most values are unused or undefined (like sparse matrices), `Dictionary` is more efficient than arrays or lists, which allocate memory for every possible value.

7. **Constant-Time Membership Checking**:  
   If you frequently need to check whether a particular element exists (e.g., checking if a specific word has already been seen), `Dictionary` allows constant-time membership checks, unlike lists that require scanning the entire list.

### When **Not** to Use Dictionary

- **Small Datasets**: For very small datasets, the overhead of hashing and maintaining a `Dictionary` might not offer significant performance improvements compared to simpler structures like lists.
- **Ordered Data**: If your operations require processing elements in a specific order (e.g., processing a list of tasks sequentially), `Dictionary` is less suitable as it does not guarantee order.
- **Memory Overhead**: In memory-sensitive applications, the additional overhead of storing keys and their hash codes may make `Dictionary` less desirable than simpler structures.

By considering these factors, you can determine when using a `Dictionary` will provide optimal performance and flexibility for your application.

---

## Comparing Dictionary and List

Let’s illustrate the efficiency of `Dictionary` over `List` by solving a popular coding problem: **Two Sum**.

### Problem: Two Sum (LeetCode #1)

> Given an array of integers, return the indices of two numbers such that they add up to a specific target.

We’ll solve this problem using both `List` and `Dictionary` and compare their performance.

#### Solution Using List (O(n²) Complexity)

```csharp
public int[] TwoSumList(int[] nums, int target) {
    for (int i = 0; i < nums.Length; i++) {
        for (int j = i + 1; j < nums.Length; j++) {
            if (nums[i] + nums[j] == target) {
                return new int[] { i, j };
            }
        }
    }
    return null;
}
```

In this solution, we iterate through the array twice to check every pair of numbers. The time complexity is **O(n²)**, which can be very slow for large arrays.

#### Solution Using Dictionary (O(n) Complexity)

```csharp
public int[] TwoSumDictionary(int[] nums, int target) {
    Dictionary<int, int> map = new Dictionary<int, int>();
    for (int i = 0; i < nums.Length; i++) {
        int complement = target - nums[i];
        if (map.ContainsKey(complement)) {
            return new int[] { map[complement], i };
        }
        map[nums[i]] = i;
    }
    return null;
}
```

Here, we use a `Dictionary` to store each number’s complement and its index. This allows us to find the solution in **O(n)** time, as we only traverse the array once.

### Performance Comparison

- **List Solution**: O(n²)
- **Dictionary Solution**: O(n)

By using a `Dictionary`, we significantly reduce runtime, making the solution much more scalable for large datasets. This illustrates why understanding when and how to use `Dictionary` is crucial for coding interviews and performance-critical applications.

---

## Common LeetCode Problems Using Dictionary

If you are preparing for interviews, you might encounter problems where `Dictionary` is an ideal solution. Below are some key LeetCode problems that rely on `Dictionary` for efficient solutions:

### 1. **Contains Duplicate** (LeetCode [#217](https://leetcode.com/problems/contains-duplicate/))

#### 1.1 Problem Description

> Given an integer array `nums`, return `true` if any value appears at least twice in the array, and `false` if every element is distinct.

#### 1.2 Solution Using Dictionary

```csharp
public bool ContainsDuplicate(int[] nums)
{
    Dictionary<int, int> nums_dict = new Dictionary<int, int>();
    foreach (int num in nums)
    {
        if (nums_dict.ContainsKey(num))
        {
            return true;  // Duplicate element found
        }
        else
        {
            nums_dict.Add(num, 0);  // Add the number to the dictionary
        }
    }

    return false;  // No duplicate elements found
}
```

#### 1.3 Explanation

In this problem, we need to check if there are any duplicate values in the array. Using a `Dictionary` allows us to store each encountered number and check in constant time (average **O(1)**) whether a number has already been seen.

### 2. **Longest Harmonious Subsequence** (LeetCode [#594](https://leetcode.com/problems/longest-harmonious-subsequence/))

#### 2.1 Problem Description

> We define a harmonious array as an array where the difference between its maximum and minimum values is exactly 1.  
> Given an integer array `nums`, return the length of its longest harmonious subsequence among all possible subsequences.

#### 2.2 Solution Using Dictionary

```csharp
public int FindLHS(int[] nums)
{
    Dictionary<int, int> nums_cnts = new Dictionary<int, int>();

    // Count the occurrences of each number in the array
    foreach (int num in nums)
    {
        if (nums_cnts.ContainsKey(num))
        {
            nums_cnts[num] += 1;
        }
        else
        {
            nums_cnts.Add(num, 1);
        }
    }

    int LHS = 0;

    // Traverse the dictionary to find harmonious subsequences
    foreach (var kv in nums_cnts)
    {
        int num = kv.Key;

        // Check if num + 1 exists in the dictionary
        if (nums_cnts.ContainsKey(num + 1))
        {
            int LHS_temp = kv.Value + nums_cnts[num + 1];
            LHS = Math.Max(LHS, LHS_temp);
        }
    }

    return LHS;
}
```

#### 2.3 Explanation

In this problem, we need to find the length of the longest harmonious subsequence where the difference between the maximum and minimum values is exactly 1. Using `Dictionary` helps us efficiently count the frequency of each element and use this information to identify harmonious subsequences.

### 3. **Isomorphic Strings** (LeetCode [#205](https://leetcode.com/problems/isomorphic-strings/))

#### 3.1 Problem Description

> Given two strings `s` and `t`, determine if they are isomorphic.  
> Two strings are isomorphic if the characters in `s` can be replaced to get `t`.  
> No two characters may map to the same character, but a character may map to itself.

#### 3.2 Solution Using Dictionary

```csharp
public bool IsomorphicStrings(string s, string t)
{
    if (s.Length != t.Length)
    {
        return false;
    }

    Dictionary<char, int> s_dict = new Dictionary<char, int>();
    Dictionary<char, int> t_dict = new Dictionary<char, int>();

    for (int i = 0; i < s.Length; i++)
    {
        char char_in_s = s[i];
        char char_in_t = t[i];

        if (s_dict.ContainsKey(char_in_s) && t_dict.ContainsKey(char_in_t))
        {
            if (s_dict[char_in_s] != t_dict[char_in_t])
            {
                return false;  // Mismatch found
            }
        }
        else if (!s_dict.ContainsKey(char_in_s) && !t_dict.ContainsKey(char_in_t))
        {
            s_dict.Add(char_in_s, i);
            t_dict.Add(char_in_t, i);
        }
        else
        {
            return false;  // One character has been seen, but the other hasn't
        }
    }

    return true;
}
```

#### 3.3 Explanation

This problem requires checking if two strings are "isomorphic," meaning there is a one-to-one mapping between characters in `s` and `t`. Using `Dictionary` allows us to efficiently establish this mapping and ensure no two characters map to the same character.

### 4. **Valid Anagram** (LeetCode [#242](https://leetcode.com/problems/valid-anagram/))

#### 4.1 Problem Description

> Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, and `false` otherwise.  
> An **anagram** is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

#### 4.2 Solution Using Dictionary

```csharp
public bool IsValidAnagram(string s, string t)
{
    if (s.Length != t.Length)
    {
        return false;
    }

    Dictionary<char, int> s_dict = new Dictionary<char, int>();

    foreach (char char_in_s in s)
    {
        if (s_dict.ContainsKey(char_in_s))
        {
            s_dict[char_in_s] += 1;
        }
        else
        {
            s_dict.Add(char_in_s, 1);
        }
    }

    foreach (char char_in_t in t)
    {
        if (s_dict.ContainsKey(char_in_t))
        {
            if (s_dict[char_in_t] == 0)
            {
                return false;
            }
            s_dict[char_in_t] -= 1;
        }
        else
        {
            return false;
        }
    }

    foreach (var kv in s_dict)
    {
        if (kv.Value != 0)
        {
            return false;
        }
    }

    return true;
}
```

#### 4.3 Explanation

This problem asks whether one string is an anagram of another. Using a `Dictionary` to count character frequencies allows us to easily verify whether the two strings contain the same characters in the same quantities.

---

## Conclusion

`Dictionary` is an efficient data structure, ideal for problems that require frequent lookups, insertions, or deletions. This is particularly important in interviews, where optimizing time complexity can mean the difference between success and failure.

By practicing problems that leverage `Dictionary`, you will gain a deeper understanding of C# and improve your interview performance. Try solving the listed LeetCode problems above, and you will see how powerful this data structure can be!
