# **Preparing for a C# Code Review Interview: Practical Examples & Tips**

## 📌 How to Approach These Interviews

I recently went through an interview that had a really interesting format: instead of whiteboard puzzles or trick questions, they gave me some code and asked me to review it — just like I would in a real pull request from a junior developer.

Honestly, I think it’s a great way to interview. It shows whether you’ve actually built and maintained real software: can you spot bugs, suggest improvements, and explain your reasoning without being condescending? It’s also becoming a more common interview style, since it tests both your technical skills and how you communicate.

In this post, I’ll share a bunch of **common code issues** that often come up in these review-style interviews, along with examples of how to explain them and propose fixes. Think of it as a prep guide to help you feel confident if you get this kind of question.

---

## 🔹 Part 1: Design & Maintainability Issues

In many of these interviews, the setup is pretty realistic. Instead of giving you an isolated snippet, they’ll often share a small project with you — maybe through something like Visual Studio Code Live Share. It’s usually a simple .NET web app or API with just enough code to resemble a real-world repo.

Before you dive into individual files, take a step back. Look at the **overall structure**:

* How are the layers separated (controllers, services, repositories)?
* Are dependencies injected, or are classes hardcoding what they need?
* Do classes follow single responsibility, or are they mixing concerns?

This section highlights common **design and maintainability pitfalls** you might see in that kind of setup. Think of it as the “bigger picture” review before you zoom in on specific bugs.

### 1. Layered Architecture

```csharp
public class UserController
{
    private readonly IUserService _service;
    public UserController(IUserService service) => _service = service;

    public async Task<IActionResult> GetUser(int id)
    {
        var response = await _service.GetUser(id);
        return Ok(response);
    }
}
```

In a review-style interview, one of the first things to check is whether the codebase respects this separation. A healthy project usually has:

* **Controllers** → **only deal with HTTP requests/responses**, not business rules.
* **Services** → contain business logic (rules, validation, calculations). Business logic layer (BLL) should separated that can be tested independently.
* **Repositories** → focusing purely on data access, so they can be swapped (SQL, NoSQL, mock repo, etc.).
* **Models and modules** → well-organized by feature or layer.
* A structure that supports scaling into microservices, instead of dumping everything into one giant project.

📂 **Example file structure:**

```plaintext
/src
  /Controllers
    UserController.cs
    OrderController.cs
  /Services
    UserService.cs
    OrderService.cs
  /Repositories
    IUserRepository.cs
    UserRepository.cs
  /Models
    User.cs
    Order.cs
```

This way, if the company later decides to split into microservices, or replace the persistence layer, the impact is minimal — layers are already decoupled.

### 2. Tight Coupling vs Dependency Injection

```csharp
public class OrderService
{
    private readonly SqlOrderRepository _repo = new SqlOrderRepository();
    public void PlaceOrder(Order order) => _repo.Save(order);
}
```

⚠️ **What’s wrong here?**

* `OrderService` is **tightly coupled** to `SqlOrderRepository`.
* If you later want to switch to a different data store (NoSQL, in-memory, mock for testing), you’ll have to **change the `OrderService` code itself**.
* This makes the system harder to test, maintain, and extend.

✅ Fix with **Dependency Injection**:

```csharp
public class OrderService
{
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) => _repo = repo;

    public void PlaceOrder(Order order) => _repo.Save(order);
}
```

* With DI, you can mock dependencies easily:

```csharp
var mockRepo = new Mock<IOrderRepository>();
mockRepo.Setup(r => r.Save(It.IsAny<Order>()));

var service = new OrderService(mockRepo.Object);
service.PlaceOrder(new Order { Id = 1, Amount = 100 });
```

Now:

* `OrderService` depends on an **abstraction (`IOrderRepository`)** rather than a concrete class.
* You can easily swap implementations (SQL, MongoDB, mock repository).
* Unit testing becomes much simpler — you just inject a fake repo.
* This follows the **Dependency Inversion Principle (DIP)**, one of the SOLID design principles.

👉 I actually wrote a [**previous blog post**](../CSharp_OOP/08_Dependency_Injection.md) diving deeper into **Dependency Injection in .NET**. If you want more detail on how to wire it up with the built-in DI container, that’s a good next read.

### 3. Too Many Responsibilities (God Class / Separation of Concerns)

A common issue is when one class tries to do everything — creating users, sending emails, logging, processing payments, generating reports, etc.
This violates **SRP (Single Responsibility Principle)** and makes the code hard to test, extend, or maintain.

❌ Example:

```csharp
public class UserManager
{
    public void CreateUser(string name) { /*...*/ }
    public void SendWelcomeEmail(string email) { /*...*/ }
    public void LogUserCreation(string message) { /*...*/ }
}
```

or

```csharp
public class OrderProcessor
{
    public void ProcessPayment(Order order) { /*...*/ }
    public void GenerateMonthlyReport(List<Order> orders) { /*...*/ }
}
```

✅ Fix: split responsibilities into separate, focused classes:

```csharp
public class UserService
{
    public void CreateUser(string name) { /*...*/ }
}

public class EmailService
{
    public void SendWelcomeEmail(string email) { /*...*/ }
}

public class Logger
{
    public void Log(string message) { /*...*/ }
}

public class PaymentProcessor
{
    public void ProcessPayment(Order order) { /*...*/ }
}

public class SalesReportGenerator
{
    public decimal CalculateMonthlyTotal(List<Order> orders) => orders.Sum(o => o.Amount);
}
```

👉 This way, each class has a **single purpose**, making the design more modular, testable, and maintainable.

### 4. Error Handling & Logging Best Practices

When something goes wrong, you need both:

* **Robust error handling** (don’t swallow exceptions silently).
* **Proper logging** (so you can debug issues later).

❌ Bad examples:

```csharp
try 
{ 
    // ...
} 
catch 
{ 
    // ignored, debugging nightmare 
}
```

```csharp
Console.WriteLine("Something failed"); // not structured, no context
```

✅ Better approach (revisited Layered Architecture code exmaple):

```csharp
public class UserController : ControllerBase
{
    private readonly IUserService _service;
    private readonly ILogger<UserController> _logger; // add logging

    public UserController(IUserService service, ILogger<UserController> logger)
    {
        _service = service;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetUser(int id)
    {
        try
        {
            var response = await _service.GetUser(id);
            if (response == null)
            {
                _logger.LogWarning("User with Id={UserId} not found", id);
                return NotFound();
            }

            return Ok(response);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "[Error]: failed fetching user with Id={UserId}", id);
            // optional: add email alerts
            return StatusCode(500, "Internal server error");
        }
    }
}
```

✨ Improvements compared to the earlier simple controller:

* Uses **`ILogger<T>`** instead of `Console.WriteLine`.
* Handles **not found case** cleanly.
* Wraps the service call in **try/catch**, logs the exception, and responds with 500 instead of crashing.

This makes the controller production-ready, not just “demo-ready.”

---

## 🔹 Part 2: Common Code Review Issues (Correctness & Performance)

Once you’ve taken a step back to understand the **overall project structure and design choices**, the next stage is to dive into the **details of the code itself**. This is where most interviewers will expect you to “smell” potential issues — correctness bugs, hidden performance traps, or design shortcuts that could cause trouble later.

In real-world pull requests, these are the kinds of things experienced developers point out to junior teammates. To help you prepare, I’ve organized some of the **most common pitfalls** you may encounter during interviews (and in production code) and show how to identify and fix them.

### 1. Null & Exception Safety

```csharp
public string GetUserName(User user)
{
    return user.Name.ToUpper(); // Possible NullReferenceException
}
```

⚠️ **Why this matters:**
In real-world code, `NullReferenceException` is **one of the most common runtime errors**. It happens when an object or property is unexpectedly `null` and someone tries to access it. Catching these early in code review or writing null-safe code can save a lot of bugs and production issues.

✅ Fix:

```csharp
public string? GetUserName(User? user)
{
    if (string.IsNullOrEmpty(user?.Name)) return null;
    return user.Name.ToUpperInvariant();
}
```

* Uses null-conditional operator `?.` to safely access properties.
* Returns `null` when input is missing, avoiding runtime crashes.

### 2. Async/Await Misuse

```csharp
public async Task<string> GetDataAsync()
{
    var result = GetDataFromDbAsync().Result; // Deadlock risk
    return result;
}
```

✅ Fix:

```csharp
public async Task<string> GetDataAsync()
{
    var result = await GetDataFromDbAsync();
    return result;
}
```

### 3. Magic Numbers / Hardcoding

```csharp
public decimal CalculateDiscount(decimal amount)
{
    return amount * 0.07m;  // 7% discount
}
```

⚠️ **Why this is bad:**

* `0.07` is a **magic number** — unclear what it represents and hard to maintain.
* If the discount rate changes, you’d need to **modify code and redeploy**, which is error-prone and slow.
* Makes debugging and understanding the business logic harder.

✅ Fix (use constants or parameters):

```csharp
private const decimal DefaultDiscountRate = 0.07m;

public decimal CalculateDiscount(decimal amount, decimal? discountRate = null)
{
    return amount * (discountRate ?? DefaultDiscountRate);
}
```

💡 **Even better:**

* Store such parameters in **configuration files (`appsettings.json`)**, a **database**, or a **feature flag system**, depending on the nature of the parameter.
* This way, you can **change values without redeploying** and the numbers are **self-explanatory**.

### 4. Concurrency Issues

```csharp
public class Counter
{
    private int count = 0;
    public void Increment()
    {
        count++; // Race condition
    }
}
```

⚠️ **Why this is a problem:**

* Multiple threads calling `Increment()` simultaneously can cause **race conditions**, where `count` might not increment correctly.

✅ Basic fix with locking:

```csharp
public class Counter
{
    private int count = 0;
    private readonly object lockObj = new();
    public void Increment()
    {
        lock (lockObj)
        {
            count++;
        }
    }
}
```

💡 **But even this fix isn’t ideal:**

* The class **mixes internal state and behavior**, making it **harder to unit test**.
* A better approach is to **favor simpler, stateless code**, or expose operations in a way that **minimizes shared mutable state**.
* For example, you could have methods that return **new results instead of mutating internal fields**, or use **thread-safe primitives like `Interlocked.Increment`** if mutability is necessary.

```csharp
public static class Counter
{
    public static int Increment(int currentCount)
    {
        // No shared mutable state
        return currentCount + 1;
    }
}
```

### 5. Modifying Collection While Iterating

```csharp
foreach (var user in users)
{
    if (!user.IsActive)
        users.Remove(user); // InvalidOperationException
}
```

✅ Fix:

```csharp
users = users.Where(u => u.IsActive).ToList();
```

### 6. String Performance Trap

```csharp
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString(); // O(n²) due to string immutability
}
```

✅ Fix:

```csharp
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
}
string result = sb.ToString();
```

### 7. Inefficient Loops / Algorithms

```csharp
public bool HasDuplicates(List<int> numbers)
{
    foreach (var n in numbers)
    {
        if (numbers.Count(x => x == n) > 1)
            return true;
    }
    return false;
}
```

✅ Fix:

```csharp
public bool HasDuplicates(List<int> numbers)
{
    var seen = new HashSet<int>();
    foreach (var n in numbers)
    {
        if (!seen.Add(n)) return true;
    }
    return false;
}
```

### 8. LINQ Multiple Enumeration

```csharp
var query = users.Where(u => u.IsActive);
if (query.Count() > 0)
{
    foreach (var user in query) // Executes query again
    {
        Console.WriteLine(user.Name);
    }
}
```

✅ Fix:

```csharp
var activeUsers = users.Where(u => u.IsActive).ToList();
if (activeUsers.Any())
{
    foreach (var user in activeUsers)
    {
        Console.WriteLine(user.Name);
    }
}
```

### 9. Resource Management / IDisposable

```csharp
public void SaveData(string filePath, string data)
{
    var writer = new StreamWriter(filePath); // Not disposed
    writer.WriteLine(data);
}
```

✅ Fix:

```csharp
public void SaveData(string filePath, string data)
{
    using var writer = new StreamWriter(filePath);
    writer.WriteLine(data);
}
```

---

## 🔹 Part 3: How to Stand Out in a Code Review Interview

Code review interviews are becoming a common and effective way for companies to **test both technical skills and practical judgment**. The goal isn’t just to find bugs, but to demonstrate that you can think like an experienced developer: spotting issues, understanding tradeoffs, and suggesting maintainable solutions.

When preparing:

1. **Start with correctness** — catch obvious bugs or runtime risks.
2. **Move to readability and performance** — show that you can spot inefficiencies or confusing code.
3. **Evaluate design & maintainability** — think about separation of concerns, dependency injection, and testability.
4. **Explain your reasoning** clearly — describe why an issue matters, what impact it has, and how your fix improves the code.
5. **Provide concrete, minimal examples** — small, self-contained snippets make your suggestions easy to understand.

By structuring your review around these principles, and grouping issues into **Correctness & Performance**, **Design & Maintainability**, and **Best Practices**, you give interviewers a **clear picture of your thought process**.

✅ Ultimately, this approach shows that you’re not just a coder — you’re a developer who cares about **quality, maintainability, and real-world impact**.
