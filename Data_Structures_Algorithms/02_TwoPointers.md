# Optimizing Code Efficiency in C# Using Two-Pointer and Sliding Window Techniques: LeetCode Examples Explained

In the world of algorithm design, **efficiency** is paramount. Among various techniques to optimize code performance, the **two-pointer** technique stands out, especially when working with arrays and lists.

This article explores how to use **two-pointer**, **fast-slow pointer**, and **sliding window** techniques to improve code efficiency in C#. We will also include relevant LeetCode examples to demonstrate the implementation of these techniques and their impact on runtime performance.

---

## What is the Two-Pointer Technique?

The two-pointer technique involves using two pointers to traverse a data structure simultaneously or at different speeds. It is particularly effective for problems involving searching, sorting, or pairing elements.

### Types of Two-Pointer Techniques

- **Start and End Pointers**: These pointers are placed at the beginning and end of a list and move towards each other. This method is especially useful for problems like finding pairs of elements in a sorted array.
- **Fast and Slow Pointers**: One pointer moves faster than the other. This technique is often used to detect cycles in linked lists.
- **Sliding Window**: This technique uses two pointers to create a window that expands or contracts, allowing efficient evaluation of subarrays.

---

## Advantages of the Two-Pointer Technique

- **Improves Time Complexity**: Many problems traditionally solved using nested loops (O(n²)) can be optimized to linear time (O(n)) with the two-pointer technique.
- **Reduces Space Complexity**: This technique often uses constant space, requiring only a few extra variables for pointer manipulation.
- **Simple and Intuitive**: Algorithms implemented with two-pointer techniques often result in cleaner and easier-to-understand code.

---

## When to Use the Two-Pointer Technique

### 1. **Start and End Pointers (Left Pointer and Right Pointer)**

- This technique is particularly useful when traversing or processing an array or list from both ends, gradually narrowing the range.
- **Best Use Cases:**
  - Working with **sorted** arrays/lists. This technique simplifies finding pairs of elements or specific values meeting certain conditions (e.g., sum, product).
  - Handling **symmetry-based** operations (e.g., palindrome checks).
  - Optimizing operations that require shrinking the search range.
- **Example LeetCode Problems:**
  - Two Sum II (LeetCode #167)
  - 3Sum (LeetCode #15)

### 2. **Fast and Slow Pointers**

- This pattern is well-suited for linked list problems or cycle detection algorithms. The idea is to have one pointer move faster (usually twice as fast) than the other to identify conditions when the pointers meet.
- **Best Use Cases:**
  - Detecting cycles in linked lists.
  - Efficiently finding the middle of a linked list.
  - Optimizing problems involving slow-growing sequences or long traversals.
- **Example LeetCode Problems:**
  - Linked List Cycle (LeetCode #141)
  - Find the Duplicate Number (LeetCode #287)

### 3. **Sliding Window**

- This technique is used for problems involving sequences (strings, arrays) that require checking a continuous subset of elements. By dynamically adjusting the window size, it efficiently tracks certain properties (e.g., length, sum).
- **Best Use Cases:**
  - Finding the longest, shortest, or a subset with specific characteristics in **subarrays** or **substrings**.
  - Calculating distinct elements, sums, or other properties within a window.
- **Example LeetCode Problems:**
  - Longest Substring Without Repeating Characters (LeetCode #3)
  - Minimum Size Subarray Sum (LeetCode #209)

---

## Common LeetCode Problems Using the Two-Pointer Technique

### 1. **Two Sum II - Input Array Is Sorted (LeetCode #167)** - Start and End Pointers

   **Problem:**  
   Given a sorted integer array and a target sum, find two numbers such that their sum equals the target.

   **Solution Using Two Pointers (Left Pointer and Right Pointer):**

   ```csharp
   public int[] TwoSum(int[] numbers, int target)
   {
       int left = 0, right = numbers.Length - 1;

       while (left < right)
       {
           int sum = numbers[left] + numbers[right];

           if (sum == target)
           {
               return new int[] { left + 1, right + 1 }; // Return 1-based indices
           }
           else if (sum < target)
           {
               left++; // Move left pointer to increase the sum
           }
           else
           {
               right--; // Move right pointer to decrease the sum
           }
       }
       return new int[] { -1, -1 };
   }
   ```

   **How It Works:**

- Two pointers (`left` starts from the beginning, `right` starts from the end) help narrow the search range based on the current sum.
- If the current sum is less than the target, move the left pointer. If it is greater, move the right pointer.

   **Comparison to Brute Force:**

- Brute force checks all possible pairs, resulting in O(n²) time complexity. The two-pointer technique reduces it to O(n).

---

### 2. **3Sum (LeetCode #15)** - Start and End Pointers

   **Problem:**  
   Find all unique triplets in an array such that their sum equals zero.

   **Solution Using Two Pointers (Left Pointer and Right Pointer):**

   ```csharp
   public IList<IList<int>> ThreeSum(int[] nums)
   {
       Array.Sort(nums); // Sort the array first
       List<IList<int>> result = new List<IList<int>>();

       for (int i = 0; i < nums.Length - 2; i++)
       {
           if (i > 0 && nums[i] == nums[i - 1]) continue; // Skip duplicates

           int left = i + 1, right = nums.Length - 1;

           while (left < right)
           {
               int sum = nums[i] + nums[left] + nums[right];

               if (sum == 0)
               {
                   result.Add(new List<int> { nums[i], nums[left], nums[right] });
                   left++;
                   right--;

                   // Skip duplicates
                   while (left < right && nums[left] == nums[left - 1]) left++;
                   while (left < right && nums[right] == nums[right + 1]) right--;
               }
               else if (sum < 0)
               {
                   left++; // Move left pointer to increase the sum
               }
               else
               {
                   right--; // Move right pointer to decrease the sum
               }
           }
       }
       return result;
   }
   ```

   **How It Works:**

- After sorting the array, for each number `nums[i]`, use two pointers (`left` and `right`) to find two other numbers whose sum with `nums[i]` equals zero.
- Sorting allows efficient adjustment of the pointers to optimize the search.

   **Comparison to Brute Force:**

- Without two pointers, all combinations must be checked with three nested loops, resulting in O(n³). Two pointers reduce the complexity to O(n²).

---

### 3. **Linked List Cycle (LeetCode #141)** - Fast and Slow Pointers

   **Problem:**  
   Determine if a given linked list contains a cycle. If it does, the fast and slow pointers will eventually meet inside the cycle.

   **Solution Using Fast and Slow Pointers:**

   ```csharp
   public bool HasCycle(ListNode head)
   {
       if (head == null) return false; // Edge case: empty list
       ListNode slow = head, fast = head;

       while (fast != null && fast.next != null)
       {
           slow = slow.next;        // Slow pointer moves one step
           fast = fast.next.next;   // Fast pointer moves two steps

           if (slow == fast) return true; // Cycle detected
       }

       return false; // No cycle detected
   }
   ```

   **How It Works:**

- The fast pointer moves two steps at a time, while the slow pointer moves one step. If a cycle exists, the fast pointer will eventually catch up to the slow pointer within the cycle.
- If the fast pointer reaches the end of the list (`fast == null` or `fast.next == null`), no cycle exists.

   **Comparison to Brute Force:**

- **Naive Approach:** Without two pointers, visited nodes must be marked (e.g., using a hash set) to detect revisits, increasing space complexity to O(n).
- **Fast and Slow Pointers:** This method achieves O(n) time complexity and O(1) space complexity, making it more space-efficient.

---

### 4. Longest Substring Without Repeating Characters (LeetCode #3) - Sliding Window

**Problem:**  
Given a string, find the length of the longest substring without repeating characters.

**Solution using Sliding Window:**

```csharp
public int LengthOfLongestSubstring(string s) {
    HashSet<char> set = new HashSet<char>();  // To store unique characters in the current window
    int left = 0, maxLength = 0;              // Left pointer and the maximum length

    // Right pointer iterates through the string
    for (int right = 0; right < s.Length; right++) {
        // If the current character already exists in the set, shrink the window
        while (set.Contains(s[right])) {
            set.Remove(s[left]);
            left++;  // Move the left pointer forward
        }

        set.Add(s[right]);  // Add the current character to the set
        maxLength = Math.Max(maxLength, right - left + 1);  // Update the max length
    }

    return maxLength;
}
```

**How It Works:**

- The sliding window technique dynamically adjusts the window size as it traverses the string.  
- The `left` pointer marks the start of the current window, and the `right` pointer marks the current character being considered.  
- If a duplicate character is found, the `left` pointer moves forward to shrink the window until the duplicate is removed, ensuring the substring remains unique.  

**Comparison to Non-Sliding Window:**

- **Brute Force Approach:** Checks all possible substrings, resulting in \(O(n^2)\) complexity as it involves a nested loop.  
- **Sliding Window Approach:** Both `left` and `right` pointers move forward only once, yielding \(O(n)\) complexity. Each character is added and removed from the set at most once, making it highly efficient.

---

### Additional Problems with Similar Techniques

Here are more LeetCode problems categorized by technique, showcasing a variety of use cases for two-pointer and sliding window strategies:

#### **Two Pointers**

1. [Two Sum II - Input Array is Sorted (LeetCode #167)](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)  
2. [3Sum (LeetCode #15)](https://leetcode.com/problems/3sum/)  
3. [Container With Most Water (LeetCode #11)](https://leetcode.com/problems/container-with-most-water/)  
4. [Trapping Rain Water (LeetCode #42)](https://leetcode.com/problems/trapping-rain-water/)  
5. [Remove Duplicates from Sorted Array (LeetCode #26)](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)  
6. [Valid Palindrome II (LeetCode #680)](https://leetcode.com/problems/valid-palindrome-ii/)  

#### **Fast-Slow Pointers**

1. [Linked List Cycle (LeetCode #141)](https://leetcode.com/problems/linked-list-cycle/)  
2. [Linked List Cycle II (LeetCode #142)](https://leetcode.com/problems/linked-list-cycle-ii/)  
3. [Find the Duplicate Number (LeetCode #287)](https://leetcode.com/problems/find-the-duplicate-number/)  
4. [Middle of the Linked List (LeetCode #876)](https://leetcode.com/problems/middle-of-the-linked-list/)  
5. [Palindrome Linked List (LeetCode #234)](https://leetcode.com/problems/palindrome-linked-list/)  

#### **Sliding Window**

1. [Longest Substring Without Repeating Characters (LeetCode #3)](https://leetcode.com/problems/longest-substring-without-repeating-characters/)  
2. [Minimum Size Subarray Sum (LeetCode #209)](https://leetcode.com/problems/minimum-size-subarray-sum/)  
3. [Permutation in String (LeetCode #567)](https://leetcode.com/problems/permutation-in-string/)  
4. [Subarrays with K Different Integers (LeetCode #992)](https://leetcode.com/problems/subarrays-with-k-different-integers/)  
5. [Longest Repeating Character Replacement (LeetCode #424)](https://leetcode.com/problems/longest-repeating-character-replacement/)  
6. [Max Consecutive Ones III (LeetCode #1004)](https://leetcode.com/problems/maximum-consecutive-ones-iii/)  

---

## Conclusion

The **sliding window**, **two-pointer**, and **fast-slow pointer** strategies are powerful techniques for optimizing code. By mastering when and how to use these methods, you can significantly improve the efficiency and clarity of your algorithms.

Practicing the listed problems will help deepen your understanding and prepare you for coding interviews where these techniques are frequently used.
