# 📚 SQL Traps & Edge Cases — Part 1: Foundations

This article is **Part 1** of a two-part series on **language-level** SQL surprises—predicates that look true but behave as unknown, aggregates with `NULL`, join shape, and set operations. **[Part 2](SQL_Questions_Part2.md)** continues with window functions, `DISTINCT ON`, CTEs and recursion, `LATERAL`, and mutation quirks.

---

## 1. Introduction

SQL looks declarative on the page. Many interview questions use five-line snippets whose result depends on rules you rarely spell out when writing normal reports: **`NULL`** as a third truth value in predicates, **`WHERE` vs `HAVING`** evaluation order intuition, **`OUTER JOIN` + filter in `WHERE`**, and aggregates that silently skip `NULL` inputs.

This guide is intentionally **narrow**: it complements the encyclopedic **[DBMS Questions](DBMS_Questions.md)** interview Q&A with **mental models** for those traps. Examples are written for **PostgreSQL** first. Where **SQL Server** or **MySQL** commonly disagree, there is a short callout as you do not need to memorize three dialects if you grasp the invariant behind the discrepancy.

These are two ideas that show up everywhere in SQL: **Predicates** vs **Aggregates**

- Predicates (used in WHERE and JOIN ON) decide whether a row is kept or removed.
A condition that evaluates to TRUE keeps the row, FALSE removes it, and UNKNOWN (like with NULL comparisons) also removes it.
- Aggregate functions work after filtering and are used to compute summary values like totals or averages. Most aggregates ignore NULL values in their calculations. COUNT(*) counts all rows, while COUNT(expression) only counts rows where the expression is not NULL.

---

## 2. `NULL`, three-valued logic, and predicates

**`NULL`** means “no value,” not zero and not empty string. In SQL predicates, comparisons involving `NULL` usually yield **`UNKNOWN`**—neither **`TRUE`** nor **`FALSE`**—until you decide what to do with `UNKNOWN`.

**Filtering rule.** `WHERE`, `JOIN ... ON`, and `HAVING` (after aggregates) keep rows only when the boolean expression evaluates to **`TRUE`**. Rows for which the predicate is **`FALSE`** or **`UNKNOWN`** are dropped—so **`UNKNOWN`** behaves like **`FALSE`** from the caller’s perspective, but it is *not* the same logical value inside compound expressions.

**Common interview tripwire:**  

```sql
-- Wrong way to ask “Is col NULL?”
SELECT * FROM t WHERE col = NULL;  -- UNKNOWN for every row; result is empty.

-- Correct
SELECT * FROM t WHERE col IS NULL;
SELECT * FROM t WHERE col IS NOT NULL;
```

**`=` for row equality.** For two nullable columns `a` and `b`, `a = b` is **`UNKNOWN`** if either operand is **`NULL`** (except in contexts that redefine comparison, such as **`DISTINCT`** or some set operations—which treat **`NULL`** as comparable to **`NULL`** for duplicate detection).

Quick truth tables (interview folklore worth knowing):

| `p`     | `NOT p` |
|---------|---------|
| `TRUE`  | `FALSE` |
| `FALSE` | `TRUE`  |
| `UNKNOWN` | `UNKNOWN` |

| `p` AND `q` when `q` is … | `TRUE` | `FALSE` | `UNKNOWN` |
|---------------------------|--------|---------|-----------|
| `p = TRUE`                | `TRUE` | `FALSE` | `UNKNOWN` |
| `p = FALSE`               | `FALSE`| `FALSE` | `FALSE`   |
| `p = UNKNOWN`             | `UNKNOWN` | `FALSE` | `UNKNOWN` |

Rough memory aid: **`AND`** with **`FALSE`** is **`FALSE`**; **`OR`** with **`TRUE`** is **`TRUE`**; **otherwise **`UNKNOWN`** often propagates.**

**Outer joins.** Preserve rows from one side even when **`ON`** does not match. If the non-preserving side has no matching row, those columns arrive as **`NULL`**. If **`ON`** is written as **`a.col = b.col`** and `b.col` can be **`NULL`**, **`UNKNOWN`** prevents a match—even though **`NULL`** in the preserved row still survives the **`LEFT`** side.

Things interviewers like to stitch together:

* **“Why did my **`NOT`** filter wipe the table?”** Because **`NOT UNKNOWN`** is **`UNKNOWN`**, and **`UNKNOWN`** drops the row—see **`NOT IN`** below.

* **`COALESCE` vs `CASE`.** **`COALESCE(a,b)`** is roughly “first non-null” and is shorthand for **`CASE`**; **`NULLIF(a,b)`** maps equal pairs to **`NULL`** (useful before divisions).

Common traps:

* Writing **`WHERE col != NULL`** and expecting **`IS NOT NULL`** behavior.
* Assuming **`JOIN ... ON a.x = b.x`** behaves like **`IS NOT DISTINCT FROM`** (`a.x` and `b.x` both **`NULL`** do not satisfy plain **`=`**).

In **PostgreSQL**, **`IS DISTINCT FROM`** and **`IS NOT DISTINCT FROM`** are explicit two-valued comparisons that treat **`NULL`** as comparable to **`NULL`** when you need **`=` semantics without **`UNKNOWN`** surprises.

---

## 3. `IN`, `NOT IN`, `EXISTS`, `NOT EXISTS`

**`EXISTS(subquery)`** is **`TRUE`** if the subquery returns at least one row; **`NOT EXISTS`** is **`TRUE`** if it returns no rows. It does **not** compare scalar values row-by-row via equality in the **`NULL`**-fragile sense—interview-safe for “matching presence.”

**`IN (list)`** expands to **`= ANY(...)`**: `expr IN (v1,v2,...,NULL)` behaves like **`(expr = v1) OR … OR (expr = vNK)`**. If **`expr`** equals some **`vi`**, the result can be **`TRUE`**. If **`expr`** differs from every non-null **`vi`** **and** **`NULL`** is in the list, at least one disjunct may be **`UNKNOWN`**, yielding **`UNKNOWN`** unless some other disjunct is **`TRUE`**.

The famous foot-gun:

```sql
-- Suppose subquery yields (2), (3), (NULL).
SELECT ...
FROM outer_t o
WHERE o.id NOT IN (SELECT nullable_col FROM inner_t WHERE ...);

-- Equivalent idea: FOR ALL ... NOT (... = inner)
-- Any inner row with NULL yields UNKNOWN for THAT comparison;
-- unless every comparison is decisively TRUE or FALSE, NOT IN resolves to UNKNOWN (no rows pass).
```

**`NOT EXISTS (SELECT … WHERE correlated_match)`** is almost always clearer for “has no counterpart” when **`NULL`** can appear:

```sql
SELECT o.*
FROM outer_t o
WHERE NOT EXISTS (
  SELECT 1
  FROM inner_t i
  WHERE i.foreign_key = o.id AND /* ... */
);
```

Interview punchline:

* Prefer **`NOT EXISTS`** over **`NOT IN (SELECT … nullable …)`** when **`NULL`** is possible.

**`IN (subquery)`** can still duplicate work if written naïvely—but optimizers typically rewrite **`EXISTS`**. Prefer clarity over micro-mythology.

Things interviewers ask:

* **Why does `NOT IN` return no rows unexpectedly?**
* **`EXISTS SELECT 1` vs `SELECT *`**: convention only; **`EXISTS`** short-circuits logically.

*[Correlated subqueries](DBMS_Questions.md) basics live in **[DBMS Questions](DBMS_Questions.md)**; this section stays on predicate truth value.*

Common traps:

* **`NOT IN`** with nullable subquery expressions.
* Assuming **`WHERE x NOT IN (...)`** is exactly the logical negation of **`WHERE x IN (...)`** when **`NULL`** is involved.

---

## 4. 🔢 Aggregates, `FILTER`, `NULL`, and distinct

**Skipping `NULL`.** **`SUM`**, **`AVG`**, **`MIN`**, **`MAX`** aggregate only non-null values. **`COUNT(expr)`** counts rows where **`expr`** is **not** **`NULL`**. **`COUNT(*)`** counts rows in the group—even when every projected column might be **`NULL`** (anything already removed by **`WHERE`** never reaches **`GROUP BY`**).

Interview contrast:

```sql
SELECT AVG(x)::numeric         -- denominators ignore NULL xs
FROM (VALUES (NULL::int), (2), (4)) AS t(x);  -- result: 3 (avg of non-null rows only)

SELECT COUNT(*) FROM (VALUES (NULL::int), (2), (4)) AS t(x);  -- 3
SELECT COUNT(x) FROM (VALUES (NULL::int), (2), (4)) AS t(x);  -- 2 (NULL not counted as a value-of-x)
SELECT COUNT(*) FILTER (WHERE x IS NOT DISTINCT FROM NULL)
FROM (VALUES (NULL::int), (2), (4)) AS t(x);  -- 1 — only the NULL-valued x
```

PostgreSQL **`FILTER (WHERE …)`** is an aggregate modifier: same idea as **`SUM(CASE WHEN … THEN … END)`**, often clearer.

**`COUNT(DISTINCT x)`.** Collapses duplicates; **`NULL`** is conventionally discarded from **distinct counts** (**`COUNT(DISTINCT x)`** does not multiply counts for duplicated **`NULL`** the way naive row counting might imply).

Things interviewers like:

* **Average of empty set:** **`NULL` aggregate result**—not zero.
* **Why two nulls collapsed in **`DISTINCT`**: **`DISTINCT`** row equality treats **`NULL`** as indistinguishable from **`NULL`** for duplicate elimination within the distinct key.

Common traps:

* Expecting **`AVG`** denominator to match **`COUNT(*)`.
* **`MAX`/`MIN` on booleans/strings** behaving “correctly” but surprising humans who only thought about integers.

---

## 5. 📊 `GROUP BY`, `SELECT` list alignment, `HAVING`

**Rule of thumb.** After **`GROUP BY`**, each selected column must either appear in **`GROUP BY`** or be aggregated **functionally dependent** per the dialect’s rule set.

**PostgreSQL** is strict unless every non-grouped **`SELECT`** output is aggregated or provably functionally dependent on **`GROUP BY`** keys (see server docs for exact dependency rules). **SQL Server** and **MySQL (with **`ONLY_FULL_GROUP_BY`**) converge on “no arbitrary column from an arbitrary row of the group”—interviewers still show broken **`SELECT dept, salary FROM emp GROUP BY dept`** snippets.

Conceptual evaluation order (**not literal physical plan**):

1. **`FROM`** / **`JOIN`**
2. **`WHERE`** (filters rows **before** grouping)
3. **`GROUP BY`** (forms groups)
4. **`HAVING`** (filters groups)
5. **`SELECT`** projections (conceptually evaluated here **for result shape**—which is why non-aggregated columns must reconcile with **`GROUP BY`**)
6. **`ORDER BY`**

**`HAVING` without `GROUP BY`:** Treat whole result as one implicit group—you can **`HAVING COUNT(*) > 10`** etc.

Interview trap:

```sql
-- WHERE vs HAVING intuition
SELECT dept, AVG(salary)
FROM emp
WHERE salary IS NOT NULL
GROUP BY dept
HAVING AVG(salary) > 90000;

-- Wrong mental model: "HAVING replaces WHERE"; it applies to groups AFTER aggregation.
```

**`GROUP BY`** with **`NULL`** places all **`NULL`** keys in **one bucket** labeled “unknown group.”

*[Basic “what is **`HAVING`**” Q&A overlaps **[DBMS Questions](DBMS_Questions.md)**—this focuses on correctness and dialect strictness]*

Common traps:

* Filtering on an aggregate expression in **`WHERE`** (compile/logic error—you need **`HAVING`** or subquery).
* Selecting a non-deterministic bare column **`MAX` hidden row** trick (invalid in PostgreSQL strict mode unless pattern is spelled with **`DISTINCT ON`**, **`array_agg`-first-max**, or **`ROW_NUMBER()`—see **[Part 2](SQL_Questions_Part2.md)**).

---

## 6. 🔗 Joins: fan-out and accidental inner joins

**Join fan-out.** If the right-hand side maps one left key to **`N`** rows, the left row appears **`N`** times. Interview “why duplicate rows?” is often “inner join multiplicity,” not buggy **`DISTINCT`** need.

**`LEFT JOIN … WHERE rhs.col = …`** If `rhs.col` is nullable and you filter **`WHERE rhs.id IS NOT NULL`**, you have often **converted** **`LEFT`** into **inner semantics** unless you deliberately want “matching rows where condition holds.” Move match predicates into **`ON`** when you intend to preserve the left spine.

```sql
-- Preserved left rows even without a match after optional join
FROM customer c
LEFT JOIN latest_order lo ON lo.customer_id = c.id AND lo.is_cancelled = false;
-- Vs pushing is_cancelled to WHERE (often drops unmatched customers!)
```

Self-joins (employee/manager graphs) **`NULL`** manager keys **`INNER JOIN`** away—use **`LEFT`** for roots unless you deliberately exclude orphans.

*[Self-join and **`=` vs **`LIKE` notes also appear briefly in **[C# Part 2](CSharp_Questions_Part2.md)**, §13—for full-stack skimming consistency.]*

Common traps:

* **`OUTER`** join then **`WHERE` on preserved side nullable foreign key**.
* Cartesian product from missing **`JOIN`** condition (**`FROM a, b`**) in whiteboard scribbles.

---

## 7. 🔀 `UNION`, `INTERSECT`, `EXCEPT`

**`UNION`** removes duplicates; **`UNION ALL`** concatenates multiset of rows (**faster**, preserves duplicates). Prefer **`UNION ALL`** unless you truly need uniqueness—interview pitfalls often hinge on forgetting distinctiveness cost and duplicate elimination **`NULL`** rules.

Set operations unify **compatible column counts and types**. Implicit casts can surprise (string vs numeric in some engines)—know your literal types in puzzles.

**`INTERSECT` / `EXCEPT`:** Duplicate elimination behaves like **`DISTINCT`** semantics for **`NULL`** within each operand’s row representation—in practice: duplicate **`NULL`** keys treated as duplicates for **`DISTINCT`**.

PostgreSQL aligns with ISO SQL fairly closely here; minor engine differences lie in coercion order—when in doubt on an interview artifact, verbalize alignment of types first.

Common traps:

* Expecting **`UNION`** to **`ORDER`** each operand separately before merging without an outer **`ORDER BY`** (**order is only guaranteed on outer query scope**).

---

## 8. 📍 `ORDER BY`, `NULL`s, and nondeterministic sorts

PostgreSQL **`ORDER BY col`** places **`NULL`s last by **`ASC`** and first by **`DESC`** by **default**, unless you **`NULLS FIRST` / **`NULLS LAST`**. Other engines invert defaults—in **SQL Server** **`NULL`** **`ORDER`** treatment historically differed (**`FIRST`/`LAST`** not standard there); **MySQL** has engine/version quirks.** Always ask “where do **`NULL`**s sort?”**

**Stable vs deterministic.** Without a **tie-break**, equal keys can appear in arbitrary order (**read plans** may vary). **`ORDER BY amount DESC`** with duplicate amounts lacks a total order—in interviews, add **`..., id`** for repeatable output.

Things interviewers test:

* **Pagination correctness** (**`OFFSET`/`LIMIT`** or **`FETCH`**) with unstable sorts—different pages can overlap or reorder under concurrent inserts.

*[Pattern matching (**`LIKE`**) pitfalls sit next to **`=`/`IN`** narrative in **[C# Part 2](CSharp_Questions_Part2.md)** §13—you still owe yourself “leading **`%`** = hard for indexes” intuition.]*

Common traps:

* Assuming **`LIMIT n`** chooses “the **`n`** best” without deterministic ordering.
* Forgetting **`NULLS FIRST`**/`LAST` portability.

---

## 9. 🌍 Dialect delta cheat (PostgreSQL-centric)

Examples in this series use **PostgreSQL** unless noted.

| Idea | PostgreSQL | SQL Server (typical) | MySQL (typical) |
|------|-------------|----------------------|----------------|
| Slice rows after sort | **`LIMIT`** / **`OFFSET`** or **`FETCH FIRST`** | **`TOP (n)`** or **`FETCH`/`OFFSET`** newer | **`LIMIT`** |
| Pattern match case-insensitive | **`ILIKE`**, **`~*`** (regex), or collation | collation / **`ILIKE`**-like equivalents via **`COLLATION`** | collation / **`LIKE BINARY`** distinctions |
| “First row per group” idiom | **`DISTINCT ON`**, **`LATERAL`**, windows | **`OUTER APPLY`**, windows | historically loose **`GROUP BY`**, windows |
| Upsert-ish | **`ON CONFLICT`** | **`MERGE`** (**caution**) | **`ON DUPLICATE KEY UPDATE`** |

**`FETCH FIRST n ROWS ONLY`** is standard spelling; Postgres accepts it and **`LIMIT`**.

Strings: **`'`** escapes double quotes as literals; Postgres identifiers **`"`quote odd names`** differently from strings—doubling pitfalls show up when pasting snippets.

---

## 10. 🏁 Closing Part 1

You now have invariant rules for predicates (`TRUE` survives, **`UNKNOWN`/`FALSE`** drop rows), aggregates vs **`COUNT(*)`**, strict grouping laws, **`OUTER`** join filtering, **`NOT IN`** vs **`NOT EXISTS`**, and sort stability.

Continue to **[SQL Traps & Edge Cases — Part 2: Windows, recursion, lateral, mutations](SQL_Questions_Part2.md)** for ranking, **`DISTINCT ON`**, CTE nuances, **`LATERAL`**, and practical mutation footguns—and a short cheat sheet for last-minute revision.
