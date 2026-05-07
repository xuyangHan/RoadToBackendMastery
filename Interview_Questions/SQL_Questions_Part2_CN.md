# 📚 SQL 陷阱与边界场景 —— 第 2 部分：窗口、递归、lateral、数据变更

这是两部分中的**第 2 部分**。**[第 1 部分](SQL_Questions_CN.md)** 已打好基础（`NULL`/谓词、**`IN`/`EXISTS`**、聚合、**`GROUP BY`**、连接扇出、集合操作、排序稳定性）。这一部分聚焦高级面试高频模式，**但不会变成 ORM 调优手册**。

配套参考：**[DBMS Questions](DBMS_Questions_CN.md)**，涵盖规范化、索引与经典面试定义。

---

## 1. 🚀 引言

窗口函数（使用 **`OVER`**）本质上是“**不压缩行数**的聚合”——除非你刻意再嵌套或重塑结果，否则每个输入行都会产生一个输出行。分区内排序、窗口帧定义、以及并列值导致的非确定性，是最常绊倒过程式程序员的点。

CTE（**`WITH`**）可读性与“优化栅栏”传说是两件事——PostgreSQL 通常会把简单 CTE 合并进外层计划，但“递归可读性”与“是否物化”仍是面试常考点。

示例继续以 PostgreSQL 为主；必要时补充 **`APPLY`**（SQL Server 对应 **`LATERAL`**）对照。

---

## 2. 🪟 窗口函数：分区、帧、非确定性

**机制。** 对每一行，计算 **`function(...) OVER (...)`**。这个窗口由以下部分定义：可选 **`PARTITION BY`**（分片）、必要时在分片内 **`ORDER BY`**、以及可选显式 **`FRAME`** 限定。

**排名家族。** **`ROW_NUMBER()`**、**`RANK()`**、**`DENSE_RANK()`**：

* **`ROW_NUMBER()`** 按顺序给唯一序号；若排序键并列且未打破并列，分配顺序是任意的。
* **`RANK`** 并列后会跳号；**`DENSE_RANK`** 不跳号。

面试陷阱：

```sql
SELECT id, amt,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
FROM orders;
```

如果 **`created_at`** 存在并列，且无次级键，**`rn`** 在并列内的分配会出现**非确定性**。修复：**`ORDER BY created_at DESC, id DESC`** 以保证复现性。

**`ROWS` vs `RANGE` 帧。**

* **`ROWS BETWEEN … PRECEDING AND CURRENT ROW`** 按物理行偏移计数（通常符合“最近 k 行滚动和”的直觉）。
* **`RANGE`** 按当前行排序键的**逻辑值距离**取范围——当排序键有并列时，窗口可能被放大/缩小，结果与直觉不一致（尤其是 **`SUM`** 这类在 peer 组上聚合的场景）。

**聚合 vs 窗口。**

* **`SUM(x) OVER (PARTITION …)`** 且无 `ORDER` 时：对整个分区求值并在每行重复显示。
* **`COUNT(*) OVER ()`** 统计窗口可见行数；与 **`COUNT(*)` + GROUP BY** 语义不同。

**窗口中的 distinct：** 在较早 PostgreSQL 版本里，**`COUNT(DISTINCT x) OVER (...)`** 长期不可用；直到 **PostgreSQL 17** 才补齐。旧题里常默认不可用，需要 `array_agg(DISTINCT …)`、二次嵌套或改写。

**与 `GROUP BY` 的心智差异：** **`GROUP BY`** 会把多行压成一行；窗口函数保留行数，做的是“每行上的滚动/分区计算”。

常见陷阱：

* 在同层 `SELECT` 里写 **`WHERE ROW_NUMBER(...) = 1`** 过滤窗口别名——`WHERE` 看不到该别名。应先包一层子查询/CTE 再过滤（如下）。

标准写法：

```sql
SELECT * FROM (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
  FROM emp
) t
WHERE rn = 1;
```

* 排名函数在 `OVER` 里忘记写 **`ORDER BY`**（分区内顺序将不确定）。

*[存储层的连接算法（nested loop/hash）见 **[DBMS Questions](DBMS_Questions_CN.md)**；窗口函数属于逻辑层易错点。]*

---

## 3. 🎯 `DISTINCT ON`、每组取一、去重惯用法

**PostgreSQL 独有 `DISTINCT ON (keys)`**：按 **`keys`** 去重后，保留每组**第一行**；“第一”由 **`ORDER BY`** 决定：

```sql
SELECT DISTINCT ON (customer_id)
  customer_id, id AS latest_order_id, created_at
FROM orders
ORDER BY customer_id, created_at DESC;
```

**陷阱：** **`ORDER BY`** 必须以 **`DISTINCT ON`** 的表达式开头，才能得到可解释且确定的选择行为（详见 PostgreSQL 文档）。

**可移植替代。** 可用 **`LATERAL`**（见 **§5**）做“相关排序后取一行”，或 **`ROW_NUMBER` + 外层过滤**（见 **§2**）。

“每组最新一条”这道经典题，若你能同时说出 **`DISTINCT ON`**、**`LATERAL` + `FETCH FIRST 1 ROW`**、**`ROW_NUMBER`** 三种方案及其可移植性取舍，通常就已是高级答法。

常见陷阱：

* 混淆 **`DISTINCT`** 与 **`DISTINCT ON`**：前者按整行去重；后者在同键 peer 里选代表行。

---

## 4. ♻️ 公共表表达式（`WITH`）与递归

**可读性。** CTE 链式写法便于表达“阶段 1 → 阶段 2”的数据流水。引擎可能会把 CTE 内联进外层，也可能保持独立。**面试要点：** 不要背“CTE 一定物化”。在 PostgreSQL 中，可通过 **`WITH ... AS MATERIALIZED (...)`** 显式要求物化。

**递归 CTE 结构（`UNION`/`UNION ALL` 的关键差异）：**

```sql
WITH RECURSIVE tree(id, parent_id, depth) AS (
  SELECT id, parent_id, 0 FROM nodes WHERE parent_id IS NULL
  UNION ALL
  SELECT n.id, n.parent_id, t.depth + 1
  FROM nodes n
  JOIN tree t ON n.parent_id = t.id
)
SELECT * FROM tree;
```

**陷阱。**

* **`UNION`**（而非 **`UNION ALL`**）会在每轮迭代去重。对唯一路径树可能可行，但在有环图中容易引发错误直觉。图遍历常需 **`UNION ALL`**，并配合环检测（PostgreSQL 14+ 的 **`SEARCH`** / **`CYCLE`**，或手工 `visited` 集）。
* 缺少基线条件或连接条件不当，会导致**不终止**或资源爆炸（应具备“最大深度护栏”意识）。
* 递归成员对 CTE 的前向引用必须安全。

**面试回答线索：** 当图可能有环时，优先给出能**口头证明终止性**的叙述，而不只是语法拼接。

常见陷阱：

* 没说清去重语义，却期待递归自动给出正确路径计数。
* 在需要的方言（如 PostgreSQL）里忘写 **`RECURSIVE`** 关键字。

---

## 5. 📎 `LATERAL` 连接（SQL Server 对应 `APPLY`）

**`LATERAL`** 会为**每一行外层输入**执行一次子查询，并允许子查询引用外层列——可理解为 `FROM` 子句里的相关子查询。

```sql
SELECT c.*, recent.*
FROM customers c
JOIN LATERAL (
  SELECT o.id, o.total
  FROM orders o
  WHERE o.customer_id = c.id
  ORDER BY o.created_at DESC
  FETCH FIRST 1 ROW ONLY   -- 类似 TOP 1
) AS recent ON true;
```

**`LEFT JOIN LATERAL … ON true`** 的行为类似 **`OUTER APPLY`**：当 lateral 子查询无行时，外层行仍保留，lateral 列为 **`NULL`**。

**SQL Server：** 通常写 **`OUTER APPLY`** / **`CROSS APPLY`**，一般不需要 **`ON true`**。

当你要实现“每个父记录取前 1 条子记录”，并希望直接使用 `LIMIT`/`FETCH` 语义时，此模式比到处铺 **`ROW_NUMBER()`** 更自然。

*[`SELECT` 中的相关标量子查询可看作更“懒”的近亲；优化器可能做类似融合。]*

常见陷阱：

* 用了 `INNER` lateral 而内层为空，导致外层行被丢（和普通内连接一致）。若想可选匹配，应使用 `LEFT` lateral。

---

## 6. ✏️ 数据变更、upsert 与并发备注

PostgreSQL 的 **`UPDATE … FROM`** 可以直接关联辅助结果集，适合简洁批量更新（SQL Server 也支持类似“连接更新”，但语法不同）：

```sql
UPDATE target t
SET value = v.new_value
FROM staging v
WHERE t.id = v.id AND t.ver = v.expected_ver;
```

带版本列的更新能表达乐观并发控制（**更新 0 行 => 有竞争冲突**），是面试中常见且不依赖 ORM 的并发叙事。

**`INSERT … ON CONFLICT … DO UPDATE`**（PostgreSQL upsert 简写）：

```sql
INSERT INTO counters(key, tally)
VALUES ('x', 1)
ON CONFLICT (key)
DO UPDATE SET tally = counters.tally + EXCLUDED.tally;
```

**注意点（高层）。**

* 默认 **`READ COMMITTED`** 在复杂事务中仍可能出现序列化异常；**`ON CONFLICT`** 不是隔离级别的替代品。
* **SQL Server 的 `MERGE`** 历史上出现过一些微妙正确性问题；很多团队仍偏向“先 `UPDATE` 再 `INSERT`”的显式模式，除非 `MERGE` 经过充分评审。

**MySQL 对照：** **`ON DUPLICATE KEY UPDATE`**。

**DEFAULT vs 显式 NULL：** 省略列会使用其 **`DEFAULT`**（如果定义了）。在 `VALUES (...)` 里显式给 **`NULL`**，就是明确写入空值——即使该列有默认值。除非遇到生成列/覆盖策略（`GENERATED ALWAYS`、`OVERRIDE`、方言特定限制）不允许，具体以引擎文档为准。

常见陷阱：

* **PostgreSQL `DELETE`** 没有 **`LIMIT`**——应通过主键条件、受限子查询（`USING`）、或 `WITH … DELETE` 谨慎收敛删除范围。
* **`RETURNING`**（PostgreSQL）可直接返回 `INSERT` / `UPDATE` / `DELETE` 影响行；SQL Server 类似能力是 **`OUTPUT`**，但规则不同，不宜混用心智模型。

---

## 7. 📋 速记清单 —— 面试前不变量

真值与存在性

* 子查询列可空时，优先 **`NOT EXISTS`** 而非 **`NOT IN (SELECT nullable …)`**。
* 谓词要“存活”必须是 **`TRUE`**；**`UNKNOWN`** 也会被过滤。
* 需要 `NULL` 安全相等时，用 **`IS NOT DISTINCT FROM`**。

聚合与分组

* **`COUNT(*)`** 按组计行；**`COUNT(col)`** 跳过 `col` 为 **`NULL`** 的行。
* **`AVG`** 分母不含 **`NULL`**；空输入/全空输入都可能得到 **`NULL`** 聚合输出。
* **`GROUP BY`** 下，裸 `SELECT` 列必须被分组或聚合（严格引擎）。

连接与结果形状

* **`LEFT JOIN`** 后在 `WHERE` 过滤右侧可空列，可能破坏外连接保留语义——不想这样就把条件移到 **`ON`**。
* 在 `DISTINCT` 之前，连接基数会先放大行数。

集合与排序

* **`UNION`** 有去重成本；**`UNION ALL`** 无去重。
* 任何 **`LIMIT`/`FETCH`/`TOP`** 的 top-N 都要配确定性的 **`ORDER BY`**。

窗口与高级主题

* 窗口结果要在**外层查询**过滤（子查询/CTE），不是同层 `WHERE` 直接用窗口别名。
* 排名遇并列必须补充排序键打破并列。
* **`ROWS` vs `RANGE`**：并列值（peers）会改变窗口范围。
* PostgreSQL **`DISTINCT ON`** 要求 **`ORDER BY`** 先对齐 **`DISTINCT ON`** 键。
* 递归 CTE 必须有终止策略与环处理方案。

数据变更与可移植性

* 熟悉所用引擎的 upsert 语义（**`ON CONFLICT`** / **`MERGE`** / duplicate key）。
* 是否序列化安全最终仍取决于隔离级别；仅有 upsert 语句并不足够。

---

## 8. 🏁 结语

**第 1 部分**建立了谓词逻辑与聚合基础；**第 2 部分**补上窗口函数、可确定排名、PostgreSQL 的 **`DISTINCT ON` / `LATERAL`**、递归注意点，以及数据变更现实问题。

如果面试前只复盘一件事：请至少在脑中完整走一遍 **`NOT IN` → `NOT EXISTS`**、解释一次 **`OUTER JOIN`** 过滤条件迁移、手写一个嵌套 **`ROW_NUMBER()`** 去重取最新、并说清 **`RANGE`** 帧在并列值下的歧义。仅这四个故事，就覆盖了非常高密度的 SQL 面试陷阱。

祝查询顺利。
