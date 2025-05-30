# 掌握贪心(Greedy)算法：从 LeetCode 难题到系统架构

## 1. **引言**

贪心算法是算法工具箱中看似简单却极具威力的工具之一。如果你做过一些 LeetCode 的题目或参加过技术面试，很可能遇到过一种“直接用贪心就行了”的解法。

但这到底是什么意思呢？

本质上，**贪心算法**是一步步构建解决方案，每一步都选择**当前看起来最优的选项**。希望通过这种局部最优的选择，最终能构造出全局最优的解。有时候它奏效，有时候……则完全失败。

一个常见的误解是“贪心 = 快且正确”——但贪心并不在于速度。它是一种思维方式：**抓住眼前看起来最好的东西**，并信任问题的结构能让你走到最后。

你可以这样理解：
> 贪心写起来简单，证明起来困难。

这也是它在面试中如此受欢迎的原因之一：它不仅测试你的编码能力，还考察你是否理解问题本质，以及是否能为自己的解法证明。

---

## 2. **实践中的贪心策略**

不同于 BFS、DFS 或动态规划，贪心没有像“使用队列”或“填充 DP 数组”那样的通用模板。它更像是一种思维方式，每个问题可能需要不同的思路——但确实存在一些常见的模式。

### 🔧 常见的贪心模式

- **排序**：重新排列项以简化决策  
- **单遍决策**：只遍历一次并作出决策  
- **贪心指标**：最少移动、最大收益、最早完成等等

来看一个具体例子：

### 💡 LeetCode 2037 – *将每位学生安排到座位的最少移动次数*

> 给你两个数组 `seats` 和 `students`，分别表示座位和学生的位置。一次移动表示学生向左或右移动一单位。求将所有学生就座所需的*最少总移动次数*？

```csharp
public int MinMovesToSeat(int[] seats, int[] students) {
    Array.Sort(seats);
    Array.Sort(students);
    int moves = 0;
    for (int i = 0; i < seats.Length; i++) {
        moves += Math.Abs(seats[i] - students[i]);
    }
    return moves;
}
```

为什么排序有效？因为它**贪心地将最近的空位分配给每位学生**，每一步都尽量减少移动距离。

这个解法既简洁又优雅。但**为什么它可行？**  
因为：

- 每一对学生与座位的匹配是独立的（一个学生的移动不会影响另一个）
- 排序让位置尽可能对齐
- 局部最优的选择*确实*构成了全局最优解

---

### 🔍 贪心适用的信号

#### ✅ 必备条件

- **贪心选择性质**：全局最优解可以通过一系列局部最优选择得到
- **最优子结构**：问题可以被拆解为可以贪心解决的子问题

#### 🔍 如何识别贪心题

- 出现以下字样：
  - “最少移动次数”
  - “最大化收益”
  - “最早截止时间”
- 每个决策可以独立做出（不需要回退或重新考虑）
- 不需要跟踪多种路径或组合

#### ❌ 贪心失效的场景

- 某个贪心选择会影响之后的选项
- 问题需要全面探索（如回溯或动态规划）
- 经典示例：**0/1 背包问题**——不能总是选择价值最高的物品

---

## 3. **LeetCode 上的经典贪心问题**

下面是一些贪心算法表现出色的问题及其原因。

| 问题 | 策略 | 贪心为何可行 | 链接 |
|------|------|----------------|------|
| **将每位学生安排到座位的最少移动次数** | 排序 + 匹配 | 排序对齐位置，最小化总移动距离 | [LeetCode 2037](https://leetcode.com/problems/minimum-number-of-moves-to-seat-everyone/) |
| **分割平衡字符串** | 基于计数的贪心 | 每当 L = R，出现一个平衡子串 | [LeetCode 1221](https://leetcode.com/problems/split-a-string-in-balanced-strings/) |
| **活动选择问题** | 最早结束优先 | 选择最早结束的任务可以留出更多空间 | [LeetCode 435 变体](https://leetcode.com/problems/non-overlapping-intervals/) |
| **跳跃游戏 II** | 单遍最大可达 | 每次都跳到当前窗口内能到的最远处 | [LeetCode 45](https://leetcode.com/problems/jump-game-ii/) |

### ✨ 分析几个例子

#### ✅ **分割平衡字符串**

```csharp
public int BalancedStringSplit(string s) {
    int result = 0, balance = 0;
    foreach (var c in s) {
        balance += (c == 'R') ? 1 : -1;
        if (balance == 0) result++;
    }
    return result;
}
```

只要字符串当前变得平衡（L == R），就立刻分割。这种即时决策就是贪心：无需预判，只要条件达成就立即操作。

#### ✅ **跳跃游戏 II**

```csharp
public int Jump(int[] nums) {
    int jumps = 0, end = 0, farthest = 0;
    for (int i = 0; i < nums.Length - 1; i++) {
        farthest = Math.Max(farthest, i + nums[i]);
        if (i == end) {
            jumps++;
            end = farthest;
        }
    }
    return jumps;
}
```

你每步扩展“最远可达位置”，只有在“必须跳”时才跳。这是贪心策略：**永远跳到当前可达的最远处**，从而保证跳跃次数最少。

---

## 4. **贪心 vs 动态规划：比较分析**

乍看之下，贪心与动态规划（DP）非常相似——两者都从左到右构建解。

但**核心差异**在于如何决策：

- **贪心**：每一步都做出*局部最优选择*——相信它最终会导向全局最优。
- **DP**：每一步都*综合考虑过去的所有决策*，选择当前最优的组合。

简言之：

> 贪心只回看**一步**，DP 回看**所有步**。

---

### 🔁 举个具体例子：[LeetCode 518 – 硬币兑换 II](https://leetcode.com/problems/coin-change-ii/)

在这个经典的 DP 问题中，贪心策略完全失败。最优解需要尝试所有硬币组合。

#### ✅ 动态规划解法

```csharp
public int Change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;

    foreach (int coin in coins) {
        for (int i = coin; i <= amount; i++) {
            dp[i] += dp[i - coin];
        }
    }
    return dp[amount];
}
```

这里，`dp[i]` 存储组成金额 `i` 的方法数。它*积累子问题的最优结果*，不像贪心那样假设“大硬币一定更好”。

---

### 🧠 总结对比表

| 特征 | 贪心 | 动态规划 |
|------|------|-----------|
| 决策范围 | 当前一步 | 所有可能的子问题 |
| 速度 | 通常更快 | 通常更慢，需用内存 |
| 模板 | 无通用模板 | 有明确定义的表或递归 |
| 正确性保证 | 仅在满足贪心条件时 | 建模正确则始终正确 |
| 适用场景 | 简单、一遍逻辑 | 存在重叠子问题 |

---

### 🔍 该选谁？

- **使用贪心的时机**：
  - 你能证明贪心选择性质
  - 子问题彼此独立
  - 题目出现 *“最大化收益”*、*“最小步数”*、*“最早完成”* 等表述

- **使用动态规划的时机**：
  - 决策依赖之前的选择
  - 需要尝试多个组合
  - 问题涉及*计数*、*划分*、或*子集*

---

## 5. **现实世界中的贪心实践**

虽然贪心算法在面试中大放异彩，但它在真实系统中同样扮演着重要角色，特别是在不允许回溯的场景中，贪心策略往往自然地浮现出来。

以下是一些现实世界中使用贪心逻辑的场景：

### 🛣 网络设计：最小生成树

- **Kruskal** 和 **Prim** 算法使用贪心策略连接所有节点，并最小化总边权——广泛用于网络路由、电路设计和交通基础设施。
- 📌 示例：构建成本最低的光纤网络连接各城市。

### 💰 资源分配与调度

- 贪心常用于 **任务调度**、**CPU 任务优先级分配**、**活动安排**，目标是最大化吞吐量或最小化空闲时间。
- 📌 示例：为员工安排值班，最大化覆盖时间并最小化重复。

### 🎬 基于区间的计划

- **活动选择问题**是经典贪心问题：选择最早结束的活动以空出更多时间——应用于日历应用、预订系统等。
- 📌 示例：自动安排会议，选择最多不冲突的时间段。

### 🧾 文件压缩

- **哈夫曼编码**是一个贪心算法，根据字符频率构造前缀码——广泛用于 ZIP 和 PNG 等无损压缩格式。

### 🚗 实时调度与打车匹配

- 像 Uber、Lyft 这样的应用使用**贪心启发式算法**快速匹配司机与乘客，目标是最小等待时间与最短距离。
- 📌 示例：指派离乘客最近的司机，而非寻找全局最优匹配（代价太高）。

### 📦 库存管理 / 缓存策略

- 在内存缓存或商品补货系统中，贪心策略（如**最近最少使用 (LRU)**）在简洁与效率之间取得良好平衡。
- 📌 示例：服务器缓存淘汰最久未访问的内容来为新内容腾空间。

---

## 6. **结语**

贪心算法的魅力在于它的简单直接：每一步都选择当前最优解，信任这样一路走下去最终也能得到全局最优。

有时候——确实能行得通。

但掌握贪心，并不是靠记住一堆套路，而是要**培养一种直觉**：判断什么时候“局部最优”可以安全地导向“全局最优”。

以下是一些关键要点：

- ✅ **贪心很优雅**，当它适用时，算法又快又简洁。  
- ❌ **贪心不是万能的**——当你拿不准时，记得模拟一下，或者退一步用动态规划。  
- 🎯 **你的目标**不是强行把每道题都往贪心上套，而是识别它的前提是否成立。

所以下次遇到一道题目，它在问你 *最小值*、*最大值* 或 *最快路径* —— 停一下，问自己一句：  
> “我能否放心地每一步都选当前最优，然后最终仍能赢下整局？”

如果答案是肯定的——那么，恭喜你找到了一个贪心解法。
