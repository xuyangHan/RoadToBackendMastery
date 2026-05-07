# 📚 SQL Traps & Edge Cases — Part 2: Windows, recursion, lateral, mutations

This is **Part 2** of two. **[Part 1](SQL_Questions.md)** builds the foundation (`NULL`/predicates, **`IN`/`EXISTS`**, aggregates, **`GROUP BY`**, join fan-out, set ops, sort stability). Here we tackle patterns that dominate “senior” SQL screens **without turning into an ORM tuning manual**.

Companion reference: **[DBMS Questions](DBMS_Questions.md)** for normalization, indexing, and classic interview definitions.

---

## 1. 🚀 Introduction

Window functions (using **`OVER`**) behave like aggregates **without collapsing groups**—each input row yields an output row unless you deliberately nest or reshape the result elsewhere. Ordering inside the partition, framing, and nondeterministic ties remain the traps that procedural programmers stumble on most.

CTE (**`WITH`**) readability is orthogonal to **optimization fence** folklore—PostgreSQL tends to merge simple CTEs into the outer plan, yet **readable** recursion and **materialization** expectations stay interview topics.

PostgreSQL-forward examples persist; **`APPLY`** (SQL Server ↔ **`LATERAL`**) noted where it helps.

---

## 2. 🪟 Window functions: partitioning, framing, nondeterminism

**Mechanism.** For each row, evaluate **`function(...) OVER (...)`** using a *window*: rows grouped by **`PARTITION BY`** (optional slice), ordered within the slice by **`ORDER BY`** where required, optionally constrained by an explicit **`FRAME`**.

**Ranking family.** **`ROW_NUMBER()`**, **`RANK()`**, **`DENSE_RANK()`**:

* **`ROW_NUMBER()`** assigns unique ints in order; ties break arbitrarily unless sort key ties are impossible.
* **`RANK`** leaves gaps after ties; **`DENSE_RANK`** does not gap.

Interview trap:

```sql
SELECT id, amt,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
FROM orders;
```

If **`created_at` ties**, **`rn`** assigns among ties **without a secondary key** → **vendor-specific tie-break**. Fix: **`ORDER BY created_at DESC, id DESC`** for reproducible demos.

**`ROWS` vs `RANGE` frames.**

* **`ROWS BETWEEN … PRECEDING AND CURRENT ROW`** counts physical row offsets (usually what you visualize as “moving sum of last **`k`** rows”).
* **`RANGE`** aligns by **logical value distance from current row’s sort key**—duplicates tied on the **`ORDER BY`** keys can inflate/deflate windows vs naive mental models (especially aggregates like **`SUM` over peers** tied on order key).

**Aggregates versus windows.**

* **`SUM(x) OVER (PARTITION …)` without `ORDER`** → aggregate over **entire partition** for each row (same value repeated).
* **`COUNT(*) OVER ()`** counts rows visible in partition—different from **`COUNT(*)` grouped**.

**Distinct windows:** **`COUNT(DISTINCT x) OVER (...)`** was historically unavailable in PostgreSQL until **PostgreSQL 17**. Older interview puzzles may disallow it outright; **`array_agg(DISTINCT …)`**, double nesting, or rewrites remain fallbacks.

**Mindset versus `GROUP BY`:** **`GROUP BY`** collapses many rows into one row per group; window functions preserve row count—“rolling” calculations **per row**.

Common traps:

* Trying to filter on a window alias **`WHERE ROW_NUMBER(...) = 1` in the same `SELECT`**—that alias is invisible to **`WHERE`**. Wrap in a derived table or CTE, then filter (**below**).

Canonical pattern:

```sql
SELECT * FROM (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
  FROM emp
) t
WHERE rn = 1;
```

* Forgetting **`ORDER BY` inside `OVER`** for ranking functions (**nondeterministic** partition ordering).

*[Join algorithms at storage level—nested loop vs hash—sit in **[DBMS Questions](DBMS_Questions.md)**; windows are logical layer traps.]*

---

## 3. 🎯 `DISTINCT ON`, top-one-per-group, dedupe idioms

**PostgreSQL-exclusive `DISTINCT ON (keys)`**: pick **first row per distinct **`keys`** combo after **`ORDER BY`** dictates which row counts as **first**:

```sql
SELECT DISTINCT ON (customer_id)
  customer_id, id AS latest_order_id, created_at
FROM orders
ORDER BY customer_id, created_at DESC;
```

**Trap:** **`ORDER BY` must begin with the same expressions as **`DISTINCT ON`** so Postgres can pick deterministically (see PostgreSQL documentation).

**Portable alternative.** Correlated “order and take one” via **`LATERAL`** (**§5**) or **`ROW_NUMBER`** with an outer **`WHERE`** (**§2**).

The “latest row per key” cliché lands when you can name **three** implementations—**`DISTINCT ON`**, **`LATERAL` + `FETCH FIRST 1 ROW`**, **`ROW_NUMBER`**—and their portability trade-offs.

Common traps:

* **`DISTINCT` vs `DISTINCT ON`**: former dedupes entire row shape; **`ON`** picks representative row among peers.

---

## 4. ♻️ Common table expressions (`WITH`) & recursion

**Readability.** CTE chains document pipeline stages (stage 1 → stage 2). Engines may merge CTE bodies into outer queries or keep them separate—inspect **`EXPLAIN`** on production-shaped queries. **Interview takeaway:** do not memorize “CTEs always materialize.” In PostgreSQL, **`WITH ... AS MATERIALIZED (...)`** *opts into* storing the CTE result when you need that fence.

**Recursive CTE anatomy (`UNION`/`UNION ALL` flip):**

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

**Traps.**

* **`UNION`** (versus **`UNION ALL`**) eliminates duplicate rows across iterations—fine for pure trees with unique paths, misleading for cyclic graphs. **Graph walks** often need **`UNION ALL`**, cycle detection (PostgreSQL 14+ **`SEARCH`** / **`CYCLE`**, or a manual **`visited`** set), or a different traversal narrative—know whether the interview graph is guaranteed acyclic.
* Missing **base case**/bad join → **nontermination**/resource blowup (**max depth guard** mentally).
* Recursive member must reference CTE forward **safely**.

**Interview line:** Prefer **`EXISTS`/`JOIN` iterative expansion** narratives that prove **termination** verbally when cycles exist.

Common traps:

* Expecting recursion to **`DISTINCT`** path counts without dedupe semantics clarity.
* Forgetting **`RECURSIVE` keyword where required** (**PostgreSQL**).

---

## 5. 📎 `LATERAL` joins (and `APPLY` on SQL Server)

**`LATERAL`** executes the subquery **for each outer row** and may reference outer columns—a correlated **`FROM`**-clause subquery in portable SQL terms.

```sql
SELECT c.*, recent.*
FROM customers c
JOIN LATERAL (
  SELECT o.id, o.total
  FROM orders o
  WHERE o.customer_id = c.id
  ORDER BY o.created_at DESC
  FETCH FIRST 1 ROW ONLY   -- TOP 1 analog
) AS recent ON true;
```

**`LEFT JOIN LATERAL … ON true`** behaves like **`OUTER APPLY`**: when the lateral subquery returns no rows, outer columns survive and lateral-selected columns arrive as **`NULL`**.

**SQL Server:** **`OUTER APPLY`** / **`CROSS APPLY`** are the customary spellings—you typically omit **`ON true`**.

Use this pattern when **“top per parent”** coupling needs a **`LIMIT`/ `FETCH`** without reaching for **`ROW_NUMBER()`** everywhere.

*[Correlated scalar subqueries in **`SELECT`** are lazy cousins—optimizer may fuse similarly.]*

Common traps:

* **`INNER`** lateral with empty inner → drops outer rows (like **`INNER`** join)—choose **`LEFT` lateral for optional match.

---

## 6. ✏️ Mutations, upsert, concurrency footnotes

PostgreSQL **`UPDATE … FROM`** joins auxiliary row sets inline for succinct bulk patches (SQL Server expresses the same join-with-update shape with different wording):

```sql
UPDATE target t
SET value = v.new_value
FROM staging v
WHERE t.id = v.id AND t.ver = v.expected_ver;
```

**Version column** guarded updates echo optimistic concurrency (**0 rows updated ⇒ lost race**)—interview concurrency story without ORM fluff.

**`INSERT … ON CONFLICT … DO UPDATE`** (PostgreSQL upsert shorthand):

```sql
INSERT INTO counters(key, tally)
VALUES ('x', 1)
ON CONFLICT (key)
DO UPDATE SET tally = counters.tally + EXCLUDED.tally;
```

**Caveats (high level).**

* Default **`READ COMMITTED`** still permits **serialization anomalies** in larger transactions— **`ON CONFLICT`** is not a substitute for declaring the isolation level your business narrative actually requires.
* **SQL Server `MERGE`** has had subtle correctness issues historically; many teams stick to **`UPDATE` then `INSERT`** patterns unless **`MERGE`** is justified and reviewed.

**MySQL analogy:** **`ON DUPLICATE KEY UPDATE`**.

**DEFAULT versus explicit NULL:** Omitting a column picks up its **`DEFAULT`**, when defined. Supplying **`NULL`** in **`VALUES (...)`** is a deliberate **`NULL`** for that column—even if a **`DEFAULT`** exists—unless generated-column restrictions (`GENERATED ALWAYS`, **`OVERRIDE`**, dialect-specific wording) disallow it—check docs for your engine.

Common traps:

* **PostgreSQL `DELETE`** has no **`LIMIT`**—scope deletes carefully (**`WHERE`** on keys, **`USING`** a constrained subselect, or **`WITH … DELETE`** chaining).
* **`RETURNING`** (PostgreSQL) returns projected rows straight from **`INSERT`**/**`UPDATE`**/**`DELETE`**; SQL Server’s closest analog is **`OUTPUT`** (different rules—read that chapter before mixing patterns).

---

## 7. 📋 Cheat sheet — last-minute invariant list

Truth & presence

* Prefer **`NOT EXISTS`** over **`NOT IN (SELECT nullable …)`**.
* Predicate survival needs **`TRUE`**, not merely “non-false”—**`UNKNOWN` drops rows.
* **`NULL`-safe equality when needed**: **`IS NOT DISTINCT FROM`**.

Aggregation & grouping

* **`COUNT(*)`** counts rows per group; **`COUNT(col)`** skips rows where **`col`** evaluates to **`NULL`.
* **`AVG` denominator** excludes **`NULL`** inputs; **`AVG` over empty/populated-but-all-null quirks** ⇒ **`NULL` aggregate output possible.
* **`GROUP BY`:** every bare **`SELECT`** column must be grouped or aggregated (strict engines).

Joins & shape

* **`LEFT JOIN`** + **`WHERE`** on nullable RHS columns can negate outer preservation—move predicates to **`ON`** when that is not intended.
* Multiplicity inflates duplicates before **`DISTINCT`**.

Sets & sorting

* **`UNION`** costs duplicate elimination versus **`UNION ALL`.
* Any **`LIMIT`/`FETCH`/`TOP`** “top‑N” needs deterministic **`ORDER BY`**.

Windows & advanced

* Window results filter via **outer query** (**subquery**/CTE)—not **`WHERE` over raw window alias**.
* Tie-break ranking with extra **`ORDER`** keys.
* **`ROWS` framing vs `RANGE` framing`: peer ties matter.
* PostgreSQL **`DISTINCT ON`** requires **`ORDER BY`** to align with **`DISTINCT ON`** keys first.
* Recursive CTE termination & cycle strategy—verbal readiness.

Mutation & portability

* Know your engine’s **`upsert`** story (**`ON CONFLICT`/`MERGE`/duplicate key`).
* Serialization still depends on isolation level—even correct upsert predicates can compose into larger anomalies.

---

## 8. 🏁 Conclusion

**Part 1** taught predicate logic and aggregates; **Part 2** layered windows, deterministic ranking, Postgres **`DISTINCT ON`/`LATERAL`**, recursion cautions, and mutation realities.

If you skim one thing hours before an interview loop: rework one **`NOT IN` → `NOT EXISTS`** example mentally, verbalize **`OUTER JOIN`** filter relocation, sketch a **`ROW_NUMBER()`** dedupe-last pattern in a nested query, and name **`RANGE`** framing tie ambiguity—those four anecdotes cover disproportionate trivia density.

Happy querying.
