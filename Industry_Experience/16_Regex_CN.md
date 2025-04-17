# 使用正则表达式解决问题：从 LeetCode 到生产环境

## 🧠 引言：即使在 AI 时代，为什么正则表达式仍然重要

正则表达式（regex）是一种紧凑且强大的方式，用于定义文本中的模式。它可以帮助开发者精准地匹配、提取和处理字符串。如果你曾写过邮箱验证器、分析日志，或者在文件中搜索特定格式的数据，那你大概率已经接触过正则了。

### 那么，什么是正则表达式？

本质上，正则表达式就是一种字符串模式。例如：

- `\d+` 匹配一个或多个数字。
- `^Hello` 匹配以 "Hello" 开头的字符串。
- `[a-zA-Z0-9]{6,}` 匹配长度至少为 6 的字母和数字组合。

正则表达式就像一种**模式小语言**，可广泛用于多种编程环境 —— 包括 C#、JavaScript、Python、MySQL 甚至 shell 脚本。

### 正则强大在哪里？

正则表达式的真正魔力在于：**一次编写，多处匹配**。一个模式就能匹配大量不同的文本场景。相比多行循环和条件判断，一条简洁有力的规则就能解决问题。

### 但……现在不是都靠工具了吗？

确实。如今像 ChatGPT、regex101.com、各种 IDE 插件等工具都能快速帮你生成正则模式。尤其当正则只是“顺手一用”而不是你的核心任务时，这些工具非常有帮助。

但**盲目复制**正则是有风险的。你可能会：

- 忽略了某些工具没考虑到的边界情况
- 使用了过于严格或太宽松的模式
- 引入性能问题（如灾难性回溯）

### 懂一点正则 = 拥有主动权

哪怕只是掌握一些基础正则知识，也能帮你：

- 调试行为不符合预期的模式
- 针对特定场景定制工具生成的正则
- 避免过度复杂或有 bug 的表达式
- 在 code review 时更清楚地与团队沟通

### 正则适用的场景 ✨

正则表达式最适合用于：

- **验证** 输入（如邮箱、手机号、用户名等）
- **提取** 半结构化文本中的结构化数据（如日志）
- **筛选** 大数据集中的特定模式（如 SQL 查询）
- **替换** 字符串中的模式内容（如清洗文本）

用得好时，正则可以是你最被低估的省时利器。

---

## 🔧 正则实战：C# 和 MySQL 中的用法

在进入 LeetCode 题目之前，我们先来看看正则在两个实际环境中的应用：C# 和 MySQL。

### 📚 你应该掌握的正则基础概念

- `\d`: 数字 (0–9)
- `\w`: 单词字符（字母、数字、下划线）
- `.`: 任意字符（换行符除外）
- `*`: 匹配 0 次或多次
- `+`: 匹配 1 次或多次
- `?`: 匹配 0 次或 1 次
- `^`: 匹配字符串开头
- `$`: 匹配字符串结尾
- `[]`: 字符集
- `()`: 分组
- `{n,m}`: 数量范围（最少 n 次，最多 m 次）

你不需要全部记住 —— 只要掌握这些基本知识，就足以理解、修改工具生成的表达式。

---

### 🔵 C# 中的使用示例

C# 在 `System.Text.RegularExpressions` 命名空间中内置了对正则的支持。

#### 1. 邮箱验证器

```csharp
string email = "user@example.com";
bool isValid = Regex.IsMatch(email, @"^[\w\.-]+@[\w\.-]+\.\w{2,}$");
Console.WriteLine(isValid); // true
```

#### 2. 电话号码解析（美式格式）

```csharp
string phone = "(123) 456-7890";
Match match = Regex.Match(phone, @"\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}");
Console.WriteLine(match.Success); // true
```

#### 3. 日志行过滤器（查找以 ERROR 开头的行）

```csharp
string log = "ERROR: Something failed\nINFO: All good\nERROR: Another failure";
foreach (Match m in Regex.Matches(log, @"^ERROR:.*", RegexOptions.Multiline))
{
    Console.WriteLine(m.Value);
}
// 输出：
// ERROR: Something failed
// ERROR: Another failure
```

---

### 🟡 MySQL 中的使用示例

MySQL 通过 `REGEXP` 关键字支持正则（或其同义词 `RLIKE`），可以用于 `WHERE` 子句中。

#### 1. 查找用户名以大写字母开头的用户

```sql
SELECT * FROM users
WHERE username REGEXP '^[A-Z]';
```

#### 2. 查找来自特定域名的邮箱地址

```sql
SELECT * FROM users
WHERE email REGEXP '@example\\.com$';
```

#### 3. 提取包含错误代码的日志（如 "ERR123"、"ERR500"）

```sql
SELECT * FROM logs
WHERE message REGEXP 'ERR[0-9]{3}';
```

注意：MySQL 的正则引擎比 C# 或 Python 更简化 —— 它不支持所有高级特性（如前瞻/后顾），但用来过滤文本数据已经非常实用。

---

## 💡 Regex遇上LeetCode：优雅解决问题

### 🧬 问题 [3475]：DNA 模式识别

你有一个名为 `Samples` 的表，包含 `sample_id`、`dna_sequence` 和 `species` 三列。你的任务是识别 DNA 序列中的模式，例如是否以特定起始密码子（如 `ATG`）开头，是否以终止密码子（如 `TAA`、`TAG`、`TGA`）结尾，或者是否包含特定子串（如 `ATAT`、`GGG`）。

#### 👴 传统 SQL 解法（使用 `LIKE`）

```sql
SELECT 
    sample_id, 
    dna_sequence, 
    species,

    CASE
        WHEN dna_sequence LIKE 'ATG%' THEN 1
        ELSE 0
    END AS has_start,

    CASE 
        WHEN dna_sequence LIKE '%TAA' THEN 1
        WHEN dna_sequence LIKE '%TAG' THEN 1
        WHEN dna_sequence LIKE '%TGA' THEN 1
        ELSE 0
    END AS has_stop,

    CASE 
        WHEN dna_sequence LIKE '%ATAT%' THEN 1
        ELSE 0
    END AS has_atat,

    CASE 
        WHEN dna_sequence LIKE '%GGG%' THEN 1
        ELSE 0
    END AS has_ggg

FROM Samples
ORDER BY sample_id;
```

这个写法虽然能用，但显得冗长又重复。你忍不住觉得应该有更优雅的方式……

#### ✨ 使用正则表达式的 SQL 一行解法

```sql
SELECT sample_id,
       dna_sequence,
       species,
       dna_sequence REGEXP '^ATG'           AS has_start,
       dna_sequence REGEXP 'TAA$|TAG$|TGA$' AS has_stop,
       dna_sequence REGEXP 'ATAT'           AS has_atat,
       dna_sequence REGEXP 'GGG'            AS has_ggg
  FROM Samples
 ORDER BY sample_id;
```

#### 🧠 正则表达式拆解

- `^ATG`: `^` 表示字符串开头，因此匹配以 `ATG` 开始的序列。
- `TAA$|TAG$|TGA$`: `|` 是 OR 操作符，`$` 表示结尾 — 检查是否以某个终止密码子结尾。
- `ATAT`: 简单子串匹配，只要出现就行。
- `GGG`: 同上，匹配连续三个 G。

#### 💡 关键点

仅用几条正则规则，就将多个基于 `LIKE` 的条件逻辑压缩成了简洁的表达式。这样不仅代码更易读，也更方便将来扩展或调整逻辑。

---

## ⚖️ 平衡：善用工具，但理解输出

### 🤖 现代开发者的现实

说实话，大多数人并不会每天都从零写 regex。毕竟我们现在有：

- 🔍 [regex101.com](https://regex101.com/)
- 💬 ChatGPT 等 AI 工具生成正则
- 🧠 AI 驱动的 IDE（如 Copilot、IntelliCode）

几秒钟内你就能搞定一个正则表达式。

### 🛠️ 但理解仍然重要

出问题的场景不少见：

- ❌ **复制粘贴的正则悄悄失效** — 差一点就匹配不上。
- 🔍 **没人能读懂那个正则** — 包括一周后的你自己。
- 🐌 **性能隐患** — 贪婪模式引发灾难性回溯（catastrophic backtracking）。

### 🧭 负责任地使用正则的小贴士

1. **掌握基础语法**：
   - `\d` = 数字
   - `\w` = 单词字符
   - `.` = 任意字符
   - `^` 和 `$` = 开头/结尾锚点
   - `*`, `+`, `?` = 数量修饰符
   - `[]` = 字符集合
   - `()` = 分组和捕获

2. **上线前一定要测试你的模式**。

3. **复杂模式一定要写注释**，尤其是生产代码：

   ```csharp
   // 匹配美国电话号码： (123) 456-7890 或 123-456-7890
   var pattern = @"\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}";
   ```

4. **维护一个正则备忘录或片段仓库**。总比第30次去搜“email 正则”来得快。

---

## ✅ 结语

正则表达式的意义不在于炫技，而是快速、清晰地解决问题。

不管是在面试、脚本还是线上查询中，正则都能帮你节省代码量、提升效率。你不需要成为正则大师，但就像 Git 或 SQL 一样，**具备正则“读写能力”** 会让你更高效、更可靠。

在 AI 工具越来越聪明的时代，理解 regex 本质，是你查错、补坑、防止误用的关键。

所以，请放心大胆地用 ChatGPT 去生成那个表达式，但请一定——读懂它、理解它、调整它。  
这，才是真正的专业。
