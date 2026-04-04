# 📚 C# Traps & Edge Cases — Part 2: OOP, Patterns & Cheat Sheet

This is **Part 2** of a two-part series. If you have not read it yet, start with **[Part 1](CSharp_Questions.md)** (Sections **1–7**): types and defaults, operators, LINQ, nulls, exceptions, and **`async`/`Task`**.

## 1. 🚀 Introduction (continued)

**Part 1** built the day-to-day foundation: value vs reference types, **`default`**, pattern matching, deferred queries, **`?.`** / NRT, **`throw;`** vs **`throw ex;`**, and sync-over-async traps. Here we continue with **Section 8** onward—the **object model** (inheritance, polymorphism, `static`), **generics and variance**, **delegates and events** (including the **`event`** vs field story from **Part 1**, Section 3), **`using` / disposal** (paired with exception handling in **Part 1**, Section 6), plus optional **SQL**, **real-world patterns**, and a **cheat sheet** for a quick review.

Same through-line as before: **correct mental models**, not a syntax checklist. Section numbers **8–17** match the original outline so you can jump between posts without renumbering.

---

## 8. 🧬 OOP & Inheritance

### Topics:

* constructor order: base class instance ctor runs before derived (field initializers → base ctor → derived ctor)
* `A a = new B()`: variable type `A`, object is `B`

### Virtual dispatch vs hiding:

* **`virtual` / `override`**: runtime type decides which implementation runs (polymorphism)
* **`new` (hide)**: hides base member for compile-time type of the reference; **no polymorphic dispatch** through base-typed variable unless cast

### Quick reference — `new` vs `override`:

| Mechanism | Through `Base b = new Derived();` calling instance method |
|-----------|-----------------------------------------------------------|
| `override` | Runs `Derived` implementation |
| `new` (hide) in `Derived` | Runs `Base` implementation when called on `b` typed as `Base` |

### Abstract vs interface (C# 8+):

* **Abstract class**: single inheritance; shared state and default behavior
* **Interface**: multiple implementation; **default interface methods (DIM)** — default impl on interface; diamond / resolution rules are easy interview fodder (know that C# resolves most conflicts explicitly)

### Access modifiers (trivia that shows up):

* **`protected internal`**: visible in same assembly **or** derived types
* **`private protected`**: visible in derived types **in the same assembly only**

### Other:

* **`sealed`**: class — no further derivation; method — no further override; “performance” is usually not the main reason — clarity and invariants are
* base constructor requirements: derived ctor must chain to base (`: base(...)`)

---

## 9. 🧠 Polymorphism Deep Dive

### Key rule:

> **Compile-time (reference) type** decides which members are *visible* and non-virtual overload resolution; **runtime type** decides **`virtual`/`override`** behavior.

### Include:

* **casting**: `as` / `is` vs direct cast (exceptions)
* **interface-based design**: depend on abstractions; explicit interface implementation (call only through interface type)
* why this matters in real systems (testing, plugin models, ORM proxies)

---

## 10. 🧪 Static vs Instance

### Topics:

* when to use static
* static class limitations

### Gotchas:

* static methods can’t access instance data
* overuse of static hurts testability

---

## 11. 🧩 Generics, Delegates, Events & Extension Methods

### Generics:

* **constraints**: `where T : class`, `struct`, `new()`, base class, interfaces
* **Covariance / contravariance**: `out` (covariant — e.g. `IEnumerable<out T>`), `in` (contravariant — e.g. `IComparer<in T>`); **`IList<T>` is invariant** — why you cannot assign `List<Dog>` to `List<Animal>`

### Delegates:

* **multicast** `+=`: multiple targets; return value and exception propagation rules (later subscribers may not run if earlier throws)

### Events:

* **`EventHandler` / `EventArgs`** pattern; raising from declaring type; external `+=` / `-=` only
* ties to **Part 1**, Section 3 (event vs field)

### Extension methods:

* static methods in static classes; **`this` first parameter**
* **resolution**: needs `using` for namespace; instance methods win over extensions when signatures match; interview trap: “why isn’t my extension picked up?”

---

## 12. 🧰 Resources & Disposal (`using`, `IDisposable`)

### Topics:

* **`using` statement**: syntactic sugar for `try` / `finally` + `Dispose()`
* **`using var`** (C# 8+): dispose at end of enclosing scope — know the scope boundary
* **`IDisposable`**: deterministic cleanup (files, DB connections, handles); still pair with `try`/`catch` at boundaries
* **`IAsyncDisposable`**: `await using` for async cleanup

### Gotchas:

* disposing twice: many types tolerate it; don’t rely on undefined types
* exceptions during dispose: log/suppress policy vs letting it propagate
* link to **Part 1**, Section 6: disposal in `finally` semantics

---

## 13. ✨ Modern C# Snapshot (.NET 6+ / C# 10+)

One place to anchor “current language” without repeating every section:

* **Nullable reference types**: turn warnings on; treat warnings as part of the API contract
* **Records**: immutability-by-convention, `with`, value equality for `record class`
* **Pattern matching + switch expressions**: exhaustiveness habits for enums and discriminated shapes
* **`init` accessors**, file-scoped namespaces, global usings (project hygiene)

---

## 14. 🗄️ SQL & Data Layer Gotchas (optional — full-stack / backend roles)

Useful when the interview blends C# with data access; skip for pure language drills.

### Include:

* `=` vs `IN` vs `LIKE`
* joins vs subqueries
* self join (referral example 🔥)

---

## 15. 🎯 Real-World Patterns

Tie everything together:

* dependency injection
* clean architecture hints
* why these “small things” matter at scale

---

## 16. 🧾 Cheat Sheet Section

Super valuable for future-you:

* default values table (value → default bits / zero; reference → `null`; enum → `0`)
* default for structs with reference fields (C# 10+); `default` literal rules
* operator behaviors (`char` addition, string `+`)
* **`throw;`** preserves stack; **`throw ex;`** does not
* **`async void`**: event handlers only
* **Virtual dispatch** only on `virtual`/`override`; **`new` hides** — no polymorphism through base-typed reference
* **`First`/`Single`**: throw when assumptions fail; prefer `*OrDefault` when absence is normal
* **`ConfigureAwait(false)`**: libraries; **`await`** over `.Result`/`.Wait()` when blocking risks deadlock
* NRT: **`!` does not fix null at runtime**

### Classic one-liners:

> “Value type → zero, Reference type → null”
> “throw; keeps stack trace”
> “Reference type decides access, runtime type decides behavior (for virtuals)”

---

## 17. 🏁 Conclusion

Across **both parts**, the payoff is the same: **think in mental models**—defaults, dispatch rules, when code runs, and what the type system can and cannot prove at compile time.

> “Most bugs don’t come from complex code, but from misunderstood simple behavior.”

If **Part 1** was the engine room and **Part 2** was the blueprint, you now have a single story from **literals** to **architecture**—without having to read it all in one sitting.
