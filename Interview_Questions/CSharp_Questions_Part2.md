# 📚 C# Traps & Edge Cases — Part 2: OOP, Patterns & Cheat Sheet

This is **Part 2** of a two-part series. If you have not read it yet, start with **[Part 1](CSharp_Questions.md)** (Sections **1–7**): types and defaults, operators, LINQ, nulls, exceptions, and **`async`/`Task`**.

## 1. 🚀 Introduction (continued)

**Part 1** built the day-to-day foundation: value vs reference types, **`default`**, pattern matching, deferred queries, **`?.`** / NRT, **`throw;`** vs **`throw ex;`**, and sync-over-async traps. Here we continue with **Section 8** onward—the **object model** (inheritance, polymorphism, `static`), **generics and variance**, **delegates and events** (including the **`event`** vs field story from **Part 1**, Section 3), **`using` / disposal** (paired with exception handling in **Part 1**, Section 6), plus optional **SQL**, **real-world patterns**, and a **cheat sheet** for a quick review.

Same through-line as before: **correct mental models**, not a syntax checklist. Section numbers **8–17** match the original outline so you can jump between posts without renumbering.

---

## 8. 🧬 OOP & Inheritance

Inheritance questions in interviews rarely ask you to draw a UML diagram. They ask what **runs**, in **what order**, and which **type**—the one on the variable or the one behind **`new`**—actually decides behavior. The traps are the same story as **Part 1**: **compile time** (what the compiler sees on the reference) versus **run time** (what object really lives on the heap).

**Static type vs run-time type.** When you write `A a = new B();`, the **variable** `a` has compile-time type **`A`**. The **object** on the heap is a **`B`**. That split matters for **member lookup** (what names are legal on `a` without a cast), for **`virtual`/`override`** dispatch (run-time type wins), and for **`new` hiding** (compile-time type often wins for the hidden member—see below). Interview snippets love this line because one word change (`override` vs `new`) flips the printed output.

**Constructor order.** Creating a derived instance runs initialization in a fixed order: **most-derived field initializers** run first, then the implicit **`base(...)`** chain runs. When control enters the **base** constructor, **that type’s field initializers** run, then the **base constructor body**, and only afterward does the **derived constructor body** continue. Repeat the idea down the chain until **`object`**. Mental model: base fields and invariants are ready before derived code runs. **Static** constructors are separate (per type, on first use); **instance** construction is what whiteboard questions usually mean.

```csharp
class Base
{
    public Base() => Console.WriteLine("Base ctor");
}

class Derived : Base
{
    string log = Init(); // field initializer runs before ctor bodies (see order in output)

    static string Init()
    {
        Console.WriteLine("Derived field init");
        return "x";
    }

    public Derived() => Console.WriteLine("Derived ctor");
}

// new Derived() prints:
// Derived field init
// Base ctor
// Derived ctor
```

**Chaining to a base constructor.** A derived constructor **must** eventually call some base constructor—either implicitly (parameterless base `ctor` if it exists) or explicitly with **`: base(...)`** or **`: this(...)`** (the latter delegates to another constructor on the **same** type first, which still must reach a base). If the base has no accessible parameterless constructor and you omit **`base(args)`**, the derived class does not compile. That is a common “why won’t this build?” interview pin.

**`virtual` / `override` vs `new` (hide).** A **`virtual`** member in a base type establishes a **slot** that **run-time dispatch** can replace. A derived **`override`** supplies the implementation the runtime uses when the object is really derived—even if the variable is typed as the base. **`new`** on a derived member **does not participate** in that slot. It introduces a **different** method that **hides** the base name for **compile-time** lookup when the expression is typed as the derived type. Through a **base-typed** reference, the **hidden** base implementation is still what you get for the hidden member unless you **cast** to the derived type.

Quick reference — **`new` vs `override`** (same call site shape, different rule):

| Mechanism | Through `Base b = new Derived();` calling the instance method |
|-----------|----------------------------------------------------------------|
| **`override`** | Runs **`Derived`** implementation (polymorphism) |
| **`new` (hide) in `Derived`** | Runs **`Base`** implementation when invoked on **`b`** typed as **`Base`** |

```csharp
class BaseV
{
    public virtual void M() => Console.WriteLine("BaseV.M virtual");
}

class DerivedOverride : BaseV
{
    public override void M() => Console.WriteLine("DerivedOverride.M");
}

class BaseH
{
    public void M() => Console.WriteLine("BaseH.M"); // not virtual — `new` in derived hides this name
}

class DerivedHide : BaseH
{
    public new void M() => Console.WriteLine("DerivedHide.M");
}

static class DispatchDemo
{
    public static void Run()
    {
        BaseV bv = new DerivedOverride();
        bv.M(); // output: DerivedOverride.M

        BaseH bh = new DerivedHide();
        bh.M(); // output: BaseH.M — `new` did not replace virtual dispatch (there was no virtual slot)

        ((DerivedHide)bh).M(); // output: DerivedHide.M — compile-time type is DerivedHide
    }
}
```

The compiler warns when you probably meant **`override`** but used **`new`** (CS0108/CS0114 family—wording varies by scenario). Treat that warning as a hint: **`new`** is almost always intentional **hiding**, not a substitute for polymorphism.

**Abstract classes vs interfaces (C# 8+).** An **abstract class** is a single inheritance anchor: you can put **fields**, **protected** state, and **shared** implementation in one place. C# allows only **one** base class, so abstract classes “cost” that slot. An **interface** is a **contract**: a type can implement **many** interfaces. Since **C# 8**, interfaces may declare **default interface methods (DIM)**—a body on an interface member implementing types inherit unless they **override** (explicit implementation and diamond scenarios are interview favorites). C# does not do C++-style implicit multiple-inheritance diamonds for state; when two interfaces give conflicting defaults, the **implementing class** usually must **resolve** explicitly (which default wins or an explicit implementation). You do not need every edge case memorized—you need the story: **defaults exist**, **conflicts are resolved in the type system**, and **explicit implementation** is the escape hatch when names collide.

**Access modifiers that sound alike.** Most interviews stick to **`public` / `private` / `protected` / `internal`**. The combined forms trip people up:

* **`protected internal`** means **either** gate passes: code in the **same assembly** **or** code in a **derived** type (even in another assembly) can see the member.
* **`private protected`** (C# 7.2+) is **stricter**: only types that are **derived** **and** in the **same assembly** can see the member. It is **not** “private *or* protected”; it is the **intersection**, not the union.

**`sealed`.** **`sealed`** on a **class** forbids further derivation—framework types sometimes seal to keep invariants and predictable behavior. **`sealed`** on an **`override`** method stops further overriding down the chain. Interview answers that cite **JIT devirtualization** as the *main* reason are weaker than answers about **design**: sealing documents “this specialization is complete” and protects assumptions.

Things interviewers like to probe in OOP and inheritance:

* **Constructor ordering** with **field initializers** and **`: base` / `: this`**—what prints first, and why?
* **`Base b = new Derived()`** — which **overload** or **method** runs for **`virtual`**, **`override`**, and **`new`**?
* **When `new` is appropriate** versus fixing the base design with **`virtual`/`override`**.
* **Abstract class vs interface** for a new abstraction—state, default behavior, versioning with **DIM**, testability.
* **`protected internal` vs `private protected`** — union vs intersection of visibility.

Common traps:

* Assuming **`new`** creates polymorphic behavior like **`override`**. It does not; it **breaks** uniform treatment through a base-typed reference for that member.
* Forgetting that **field initializers** run **before** base constructor bodies from the perspective of the **derived** type’s initializer sequence (see the sample output above).
* Mixing up **`protected internal`** (**OR**) with **`private protected`** (**AND**—same assembly **and** derived).
* **Sealing** for “speed” alone without mentioning **API** and **invariant** reasons—interviewers often listen for engineering judgment, not micro-optimization folklore.

---

## 9. 🧠 Polymorphism Deep Dive

**Section 8** already nailed **`virtual`/`override`** versus **`new`** and **`Base b = new Derived()`** dispatch. The rule to carry forward is short:

> **Compile-time (reference) type** decides which members are *visible* and how **non-virtual** overload resolution works; **run-time type** decides **`virtual`/`override`** behavior.

This section adds what interviews stack on top: **getting to the right type safely**, **interfaces as the outer shape of a system**, and **why** polymorphism pays off outside toy snippets.

**Casting: direct cast vs `as` vs `is`.** A **direct cast** `(T)obj` tells the runtime “this object really is a **`T`**.” If it is not, you get **`InvalidCastException`**. The **`as`** operator **`obj as T`** returns **`null`** when the reference is not compatible with **`T`** (or when the value is already **`null`**), and never throws for that failed conversion—so it pairs naturally with a **`null`** check. **`is`** in modern C# is usually **pattern**-based: **`if (obj is T t)`** assigns **`t`** only when the pattern matches; with nullable reference types, the compiler often narrows **`null`** for you in that branch. **`is T`** alone is a boolean test with no assignment.

```csharp
object o = "hello";

string ok = (string)o;           // "hello" — throws InvalidCastException if `o` is not a string
string? viaAs = o as string;     // same reference or null — no throw on mismatch
// int bad = (int)o;             // InvalidCastException — not a boxed int

if (o is string s)               // pattern: `s` is string inside the block
    Console.WriteLine(s.Length);
```

**Nuance:** **`as`** only works with **reference types**, **nullable value types**, and **interfaces**—not non-nullable value types like **`int`**, because there is no **`null`** sentinel for “conversion failed.” For value types, use **`is`**, try **`pattern`**, or **`Nullable<T>`** patterns.

**Interface-based design.** Depend on **`IEnumerable`**, **`IDisposable`**, your own **`IOrderRepository`**, and so on—not concrete **`SqlOrderRepository`** everywhere. That keeps **call sites** stable when implementations change and makes **substitution** obvious: anything that satisfies the contract can sit behind the variable. **Section 8** contrasted **abstract classes** (state + one base) with **interfaces** (many contracts, **DIM** in modern C#). In interviews, “program to an interface” is not dogma—it is **control of coupling**.

**Explicit interface implementation (EIMI).** When a class implements two interfaces that both define a member with the **same signature** (same name and shape), or when you want a member **only** callable through the interface type, implement with **`void IFoo.M()`**-style syntax. There is **no** `public` method named **`M`** on the class surface unless you add a separate public wrapper. Callers holding **`ConcreteType`** may not see **`M`** at all; callers holding **`IFoo`** do. That is deliberate **encapsulation** and **name disambiguation**, not a compiler bug.

```csharp
interface IReadable { void Load(); }
interface IWritable { void Load(); } // same name, different contract — contrived but common shape in interviews

class Doc : IReadable, IWritable
{
    void IReadable.Load() => Console.WriteLine("read path");
    void IWritable.Load() => Console.WriteLine("write path");
}

static class EimiDemo
{
    public static void Run()
    {
        var doc = new Doc();
        // doc.Load(); // does not compile — no public Load

        IReadable r = doc;
        r.Load(); // output: read path

        IWritable w = doc;
        w.Load(); // output: write path
    }
}
```

**Why this matters in real systems.** **Unit tests** swap **fakes** and **stubs** for **interfaces** without inheriting production sealed types. **Plugin** and **modular** hosts load implementations behind a **stable interface** or **abstract** contract. **ORMs** and **proxies** often give you **subclasses** or **runtime-generated** types that still **quack** like your **`DbContext`** entities or **`IRepository`**—your code stays on the **abstraction** so those tricks stay **invisible**. The interview payoff: polymorphism is not “OOP vocabulary”; it is **where you allow variation** without rewriting callers.

Things interviewers like to probe in polymorphism:

* **Which cast throws**, which returns **`null`**, and when **`is` patterns** are clearer than **`as` + check**.
* **EIMI**: who can call the method, and how you **disambiguate** two interfaces.
* **Design**: interface vs abstract base for a new seam—**test doubles** and **evolution** are fair follow-ups.

Common traps:

* Using **`as`** and forgetting **`null`**—**`as`** failed silently, then **`NullReferenceException`** on the next line.
* Assuming a **direct cast** gives **`null`** on failure; only **`as`** does (for supported **`T`**).
* **EIMI** “missing” on the public type—then wondering why **`new Doc().M()`** does not compile.

---

## 10. 🧪 Static vs Instance

**Instance** members belong to an **object**: each instance has its own **fields**, and **`this`** refers to that object. **Static** members belong to the **type**: one **`static` field** is shared by **all** instances (and by code that has no instance at all). There is no **`this`** in a **`static`** method because there is no **receiver object**—only the **type**.

**When `static` fits.** **Pure** helpers with **no** per-object state (**`Math.Max`**, string utilities). **Singleton-ish** factories or **shared caches**—with care for **thread safety** and **testability**. **`const`** / **`static readonly`** invariants (**`TimeSpan.Zero`**). **Extension method** containers must be **`static`** classes. **Operator overloads** and **implicit conversions** are **`static`** by language rule. The interview-friendly line: reach for **`static`** when the behavior is **a function of the arguments** (and perhaps shared global state you truly mean to share), not **a function of “this object’s” identity**.

**`static` class** (C#) is **`abstract`** and **`sealed`**: you **cannot** **`new`** it or **inherit** from it. It is only a **namespace-like** home for **`static`** members. **`static` constructors** run **once per type**, before the first use of that type, in a **thread-safe** (lazy) way—good for **one-time** initialization of **`static`** state; easy to get wrong if the initializer **throws** or **blocks**.

**Limitations and test pain.** **`static`** methods **cannot** use **instance** fields or **instance** methods unless they are given an **instance** as a parameter. Heavy **`static`** **singletons** and **`static`** **mutable** state make **parallel tests** **order-dependent** and **mocks** awkward—you **cannot** **override** a **`static`** method on a **test double** the way you **virtual**/**interface**-back an instance API. **Dependency injection** prefers **instance** services registered in a **composition root**; **`static`** **service locators** are a common **anti-pattern** in modern answers **unless** the scope is truly **process-wide** and **immutable**.

Things interviewers like to probe with `static`:

* **Why** there is no **`this`** in a **`static`** method; what **shared** state implies for **threads** and **tests**.
* **`static` class** vs a normal class with only **`static`** members—**inheritance** and **construction** differences.
* **When** **`static`** is appropriate vs **instance** + **interface** for **testability**.

Common traps:

* **`static`** **mutable** **global** state “for convenience”—**flaky** tests and **hidden** coupling.
* Treating **`static`** methods as **polymorphic**; **overrides** apply to **instance** **`virtual`** members, not **`static`** ( **`static`** in a derived type **hides** the base **`static`** member by name—another **compile-time** story, not **virtual** dispatch).

---

## 11. 🧩 Generics, Delegates, Events & Extension Methods

This section bundles four “language machinery” topics that show up less often than **`async`** or **nulls**, but still appear as follow-ups. Keep the **headline rules**; skip proving every variance corner case unless the interviewer goes there.

**Generics and `where` constraints.** **`where T : class`** (reference type only), **`where T : struct`** (non-nullable value type), **`where T : new()`** (parameterless constructor—handy for **`new T()`**), **`where T : SomeBase`**, **`where T : ISome`**, and combinations (**`class?`**, **`struct?`**, **`notnull`**, and so on in newer C#) tell the compiler what **`T`** can be so overloads and APIs stay safe. Without constraints, you only get **`object`**-level operations unless you **cast**.

**Covariance (`out`) and contravariance (`in`).** These keywords apply to **generic type parameters on interfaces and delegates**, not arbitrary **`List<T>`**. **`out T`** means “**T** only appears in **output** positions”—callers may treat **`IEnumerable<Dog>`** as **`IEnumerable<Animal>`** when **`Dog : Animal`** (**read-only** sequence of animals). **`in T`** means “**T** only appears in **input** positions”—you can pass an **`IComparer<Animal>`** where an **`IComparer<Dog>`** is expected (**wider** comparer, **narrower** type). **`IList<T>`** is **invariant**: you can **both read and write** **`T`**, so allowing **`IList<Dog>`** as **`IList<Animal>`** would let someone **insert a `Cat`** into a list you still think is only **dogs**—the type system rejects that.

```csharp
class Animal { }
class Dog : Animal { }

IEnumerable<Dog> dogs = Array.Empty<Dog>();
IEnumerable<Animal> animals = dogs; // OK — IEnumerable<out T> is covariant

// IList<Dog> dogList = new List<Dog>();
// IList<Animal> broken = dogList; // compile error — IList<T> is invariant
```

**Delegates and multicast.** A **delegate** is a typed method pointer; **`+=`** adds another target (**multicast**). For **`void`** returns, invocations run in order; if one target **throws**, **later targets do not run** in that synchronous invoke. For non-**`void`** multicast delegates, the **returned** value is from the **last** target in the list—easy to forget in trivia questions.

**Events vs delegate fields.** An **`event`** restricts **external** code to **`+=`** / **`-=`** only; only the **declaring type** should **raise** (invoke) it. That is the same story as **[Part 1](CSharp_Questions.md)**, **Section 3**—treat **`event`** as a **capability**, not a **shared field**. The usual surface is **`EventHandler` / `EventArgs`** (or **`EventHandler<TEventArgs>`**): **`object? sender`**, **`EventArgs`** (or a derived type) for payload.

**Extension methods.** A **`static`** method in a **`static`** class whose **first** parameter is marked **`this T target`** “attaches” syntax **`target.Method(...)`**. The compiler rewrites it to **`Extensions.Method(target, ...)`**—there is no real instance method; **private** members of **`T`** stay **invisible**. **Import** the **namespace** (**`using`**) or the extension is not in scope. **Instance** methods on the type **win** when signatures match; extensions are **lower priority**. If “my extension is ignored,” check **`using`**, **visibility**, **wrong** **`static`** class, or a **nearer** instance overload.

Things interviewers sometimes touch here:

* **Why** **`List<Dog>`** is not **`List<Animal>`** even when **`Dog : Animal`** (**invariance** and **write** safety).
* **Multicast** ordering, **exceptions**, and **return value** rules.
* **Event** encapsulation vs a **public** **`Action`** field.

Common traps:

* Expecting **variance** on **mutable** collections (**`List<T>`**, **`IList<T>`**).
* **Raising** an **`event`** without a **null** check or **null-conditional** (`?.Invoke`) when the backing field can be empty.
* Forgetting that **extensions** are **syntactic sugar**—no **special** access to **private** state.

---

## 12. 🧰 Resources & Disposal (`using`, `IDisposable`)

**`IDisposable`** is the pattern for **deterministic** cleanup: **files**, **handles**, **database connections**, **timers**—anything that should **release** soon after use, not only when the **GC** eventually runs. **`using`** is **syntax** for **`try` / `finally`** that calls **`Dispose()`** so cleanup runs even when **`try`** throws—same **control-flow** picture as **[Part 1](CSharp_Questions.md)**, **Section 6** (**`finally`** runs before **`return`**).

**`using` statement vs `using var`.** Classic **`using (var x = …)`** disposes at the **closing brace** of the block. **`using var x = …;`** (C# 8+) disposes at the **end of the enclosing scope**—often the **method**—which is shorter to write but easier to **misread** (“why is this still open?”). Know **where** the scope **ends**.

**`IAsyncDisposable` and `await using`.** Async cleanup (**streams** with **async** flush/close) uses **`await using`** so **`DisposeAsync()`** participates properly in **`async`** methods.

**Practical gotchas.** Many **BCL** types allow **`Dispose()`** twice (**idempotent**); do not assume **every** type you did not author does. If **`Dispose`** throws, you still must decide **logging**, **suppression**, or **propagation**—**nested `using`** can hide **secondary** failures if you are not deliberate.

Things interviewers sometimes ask:

* **What** **`using`** expands to mentally (**`finally` + `Dispose`**).
* **Where** **`using var`** disposes relative to the method body.
* **Why** **deterministic** cleanup matters for **unmanaged** or **scarce** resources.

Common traps:

* **Leaking** without **`using`** / **`try`/`finally`** on **short-lived** **expensive** resources.
* **Long-lived** **`using var`** at **method** scope—**holds** the resource **longer** than a **tight** block would.

---

## 13. ✨ Modern C# Snapshot (.NET 6+ / C# 10+)

A **compact** “what does **current** C# look like?” anchor without repeating **Part 1** in full.

**Nullable reference types (NRT).** Turn warnings on in **real** projects; treat them as part of the **API contract** (**`string`** vs **`string?`**, **`!`** only **silences** the compiler—**Part 1**, **Sections 2** and **5**). **.NET 6+** templates often enable NRT by default.

**`record` / `record class`.** **Value-based** equality and **`with`** for **non-destructive** copies are the headline; **immutability** is **by convention** unless you enforce it (**`init`**, read-only state). **`record struct`** exists for **value**-type records—know the name, not every edge case.

**Pattern matching and `switch` expressions.** **`switch` expressions** and **property patterns** are **normal** modern style; **exhaustiveness** for **enums** still collides with **numeric** values that are not **named** members (**Part 1**, **Section 3**).

**Project hygiene (C# 10+).** **File-scoped namespaces** (`namespace X;`) drop one brace level; **`global using`** cuts boilerplate **when the team agrees** on **imports**. **`init`** setters pair well with **object initializers** and **immutable**-looking **DTOs**.

Things interviewers sometimes ask:

* **What** a **`record`** buys you vs a **`class`** for **DTOs** and **equality**.
* **Whether** **`!`** fixes **`null`** at **run time** (it does **not**).

Common traps:

* **`record`** **inheritance** and **equality** surprises—**defaults** changed the **story** over versions; check **docs** for your **SDK** when it matters.

---

## 14. 🗄️ SQL & Data Layer Gotchas (optional — full-stack / backend roles)

Skip this block for **pure C#** screens; keep it when the loop includes **EF Core**, **Dapper**, or “write the SQL” exercises. The goal is sane **predicates** and **shape**—not memorizing every dialect keyword.

**`=` vs `IN` vs `LIKE`.** **`=`** is **equality** on a **single** value (often **index-friendly** when the column is indexed and the expression is **sargable**—avoid wrapping the **column** in functions unless you must). **`IN (...)`** is **membership** in a **small**, **fixed** list; for **many** values or **dynamic** lists, platforms usually prefer a **join** to a **temp table** / **table-valued parameter** / **batch** pattern rather than a **giant** **`IN`**. **`LIKE`** is **pattern** match: **`%`** and **`_`** wildcards, **case** rules depend on **collation**; leading-wildcard patterns (**`%foo`**) often **forfeit** simple **index seeks**. Know the difference between **“equals this id”** and **“contains this substring.”**

**Example:**

Suppose you have a `Users` table with columns `Id`, `Username`, and `Email`. Here’s how the three approaches behave:

```sql
-- EQUALS: Find the user with a specific Id
SELECT * FROM Users WHERE Id = 42;

-- IN: Find users whose Id is in a fixed set
SELECT * FROM Users WHERE Id IN (10, 42, 99);

-- LIKE: Find users whose username starts with 'alex'
SELECT * FROM Users WHERE Username LIKE 'alex%';

-- LIKE (wildcard in front!): Find usernames containing 'lex'
SELECT * FROM Users WHERE Username LIKE '%lex%';
```

- Use `=` when you have a **single, exact** value.
- Use `IN` for a **short, fixed list**.
- Use `LIKE` for **pattern** matches, but watch performance for patterns starting with a wildcard (`%foo` vs `foo%`).

**Joins vs subqueries.** A **join** relates rows in **two** sets; a **correlated subquery** runs **per outer row** and is easy to write yourself into **O(n²)** behavior if the optimizer does not save you. **Non-correlated** subqueries in **`IN`** / **`EXISTS`** can be **fine**—**`EXISTS`** is often the clearest for **“is there any matching row?”** Interview answers that say “joins are always faster” are **wrong**; **measure** and **read plans** in real jobs, but for **whiteboards**, prefer **clear** **join** graphs and **`EXISTS`** over **nested** **correlated** **selects** unless the problem is trivial.

**Self-join.** A **self-join** is the same table **twice** with **different aliases**—classic **“employee and manager”** or **referral** graphs (**`User.ReferredById` → `User.Id`**). You **join** **`Users child`** to **`Users referrer`** on **`child.ReferredById = referrer.Id`**. Traps: **`NULL`** **foreign keys** (**no** referrer) drop out of **inner joins**—use **`LEFT JOIN`** when **orphan** / **root** rows should **still** appear.

Things interviewers sometimes ask:

* **When** **`IN`** is acceptable vs a **join** to a **list** of ids.
* **`EXISTS` vs `IN`** for **presence** tests (and **NULL** behavior in **`NOT IN`**—easy **foot-gun** in SQL generally).
* **Why** **`LIKE '%x'`** can be a **performance** red flag.

Common traps:

* **`NOT IN (subquery)`** when the subquery can yield **`NULL`**—the logic often surprises people; **`NOT EXISTS`** is usually **safer** mentally.
* Confusing **`=`** with **`LIKE`** when the requirement is **exact** match vs **pattern**.


---

## 15. 🏁 Conclusion

Across **both parts**, the payoff is the same: **think in mental models**—defaults, dispatch rules, when code runs, and what the type system can and cannot prove at compile time.

> “Most bugs don’t come from complex code, but from misunderstood simple behavior.”

**Part 1** is the engine room: types, LINQ, nulls, exceptions, and **`async`/`Task`**. **Part 2** adds the object model (inheritance, polymorphism, **`static`**), generics and events, disposal, a modern C# snapshot, optional SQL notes, architecture touchpoints, and the cheat sheet in **Section 16** for a fast review. Either part stands on its own; together they run from literals and operators to the kinds of seams and habits that keep **.NET** backends maintainable—without reading everything in a single sitting.
