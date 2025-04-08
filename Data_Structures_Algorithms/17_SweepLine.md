# Mastering the Sweep Line Algorithm: From LeetCode to Real-World

The Sweep Line algorithm is one of those hidden gems that shows up in coding interviews, but also quietly powers a variety of real-world applications — from calendar scheduling to geometric computation. In this post, we’ll demystify how it works, implement it in C# using `SortedDictionary`, and explore where it really shines in practice.

---

## 🟢 1. Introduction to Sweep Line

Imagine you're standing at the left edge of a timeline or a 2D space, dragging a vertical line from left to right. Every time the line touches something important — like the start or end of an event — you "process" that event. That’s the essence of the Sweep Line technique.

Instead of scanning all elements blindly, we **preprocess the data into sorted events** (like interval starts/ends, or points in 2D space), and then use an efficient data structure to **track the current state** as the line moves.

### 🧠 How it Works

- **Step 1: Convert** intervals, rectangles, or objects into a set of events (start and end points).
- **Step 2: Sort** all events by their position (often X or time).
- **Step 3: Sweep** through the events in order, updating a dynamic structure (like a count, heap, or set).
- **Step 4: Process** logic as needed — overlaps, intersections, free time, etc.

### 📈 Visual Example

Let’s say we want to find the **maximum number of overlapping meetings**:

Given Meetings:

``` plaintext
Meeting A: [1, 5)
Meeting B: [2, 6)
Meeting C: [4, 7)
```

``` plaintext
Time:     1   2   3   4   5   6   7
          |---|---|---|---|---|---|
A:        [===============)
B:            [===============)
C:                    [===========)
```

Events:

``` plaintext
+1 at 1 (A starts)
+1 at 2 (B starts)
+1 at 4 (C starts)
-1 at 5 (A ends)
-1 at 6 (B ends)
-1 at 7 (C ends)
```

As we sweep:

- At time 1: active = 1  
- At time 2: active = 2  
- At time 4: active = 3 ✅ max
- At time 5: active = 2  
- ...

---

## ⚙️ 2. Operations on Sweep Line Using `SortedDictionary` (C#)

In C#, `SortedDictionary<int, int>` is a great way to track event counts in a sweep line. It keeps keys sorted and allows O(log n) insertions and lookups.

### 🧪 Example: Find Maximum Overlaps

```csharp
public int MaxOverlap(int[][] intervals) {
    var timeline = new SortedDictionary<int, int>();

    foreach (var interval in intervals) {
        int start = interval[0], end = interval[1];
        timeline[start] = timeline.GetValueOrDefault(start, 0) + 1;
        timeline[end] = timeline.GetValueOrDefault(end, 0) - 1;
    }

    int active = 0, max = 0;
    foreach (var kvp in timeline) {
        active += kvp.Value;
        max = Math.Max(max, active);
    }

    return max;
}
```

### 🧭 Explanation

- We mark `+1` at each start and `-1` at each end.
- Then we sweep through the sorted timeline, adjusting the active count.
- The peak value of `active` is our answer.

This technique works well for:

- Counting overlapping events
- Resource tracking
- Managing concurrent tasks

---

## 🌍 3. Real-World Applications

The Sweep Line technique isn't just for toy problems — it's used in real-world systems across industries:

### 📅 Calendar & Scheduling

- **Meeting conflict detection**: Quickly find time conflicts in multiple calendars.
- **Free slot detection**: Use sweep line to find gaps between meetings.

### 📊 System Logs & Event Tracking

- **Concurrent users**: Count how many sessions were active at any moment.
- **Peak load**: Track maximum memory/CPU usage over time.

### 🗺️ Computational Geometry

- **Line segment intersections**: Used in map rendering and GIS tools.
- **Polygon union/overlap detection**: Helpful for game physics and CAD systems.
- **Constructing Voronoi diagrams**.

### 🎮 Game Development

- **Collision detection**: In 2D games, check which objects overlap on X-axis before checking more expensive Y-axis.
- **Rendering order**: Z-ordering based on sweep line depth.

If you’ve ever booked a meeting on Google Calendar or played a 2D game, there’s a good chance sweep line logic was working behind the scenes.

---

## 🧩 4. Leetcode Questions & Solutions

Once you're familiar with the sweep line technique, you'll start to notice it behind many interval-based problems. This section walks through some Leetcode questions that are particularly well-suited for sweep line. We’ll start with a classic and accessible one:

### 📌 1854. Maximum Population Year [🔗](https://leetcode.com/problems/maximum-population-year/)

> Given people's birth and death years, return the **earliest year** with the **maximum population** (a person is alive from `birth` to `death - 1`).

### 🧹 How Sweep Line Applies

This problem is a textbook example:

- Every person adds a `+1` to the population at their **birth year**
- They subtract `-1` at the **death year** (exclusive)
- You process these **timeline events** in order and maintain a running total

It’s the same principle as tracking active intervals, or overlapping meetings — just in a historical context.

### ✍️ C# Solution (Sweep Line Style)

```csharp
public class Solution {
    public int MaximumPopulation(int[][] logs) {
        int[] years = new int[101]; // from 1950 to 2050

        foreach (var log in logs) {
            years[log[0] - 1950]++;   // birth year
            years[log[1] - 1950]--;   // death year (exclusive)
        }

        int maxPop = 0, curr = 0, result = 1950;

        for (int i = 0; i < 101; i++) {
            curr += years[i];
            if (curr > maxPop) {
                maxPop = curr;
                result = 1950 + i;
            }
        }

        return result;
    }
}
```

### 📚 More Leetcode Questions

| ID    | Title                                                | Note                                 |
|-------|------------------------------------------------------|--------------------------------------|
| 56    | [Merge Intervals](https://leetcode.com/problems/merge-intervals/)         | Greedy + sort (basic sweep-like idea) |
| 1288  | [Remove Covered Intervals](https://leetcode.com/problems/remove-covered-intervals/) | Sort + interval elimination          |
| 986   | [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/) | Two-pointer, related idea            |
| 253   | [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)       | Sweep line with heap (advanced)      |
| 759   | [Employee Free Time](https://leetcode.com/problems/employee-free-time/)   | Merge + sweep                        |

---

## 🏁 5. Conclusion

The **sweep line algorithm** is a powerful and versatile technique, especially when dealing with interval-based problems. By transforming intervals into **events** and processing them in order, you can efficiently handle a wide variety of real-world challenges, such as scheduling, population analysis, and computational geometry.

### ✨ Key Takeaways

- **Event-based thinking** is central to the sweep line algorithm. By converting continuous intervals into discrete events (start or end points), we can track the state of an evolving system, like overlapping intervals or populations.
- **Efficiency**: Sweep line problems often transform brute-force solutions into much faster algorithms, typically reducing time complexity from quadratic to linear with sorting and event processing.
- **Common Patterns**: Many LeetCode problems, from **maximum population year** to **interval merging**, use a sweep line approach, making it a valuable pattern to recognize and apply.

### 🔄 When to Use Sweep Line

- When dealing with **intervals** or **overlapping events**.
- When you need to process large datasets of **start/end times** (e.g., meetings, schedules, population changes).
- When you’re looking for an efficient way to handle **dynamic changes** over a continuous range of time or space.

### 🚀 Wrapping Up

Understanding and mastering sweep line techniques will significantly enhance your problem-solving toolkit, making you more efficient at solving **real-world scheduling problems** and **algorithmic challenges**. With its ability to turn complex overlapping problems into simple event-processing tasks, it’s a must-have in any programmer’s arsenal.
