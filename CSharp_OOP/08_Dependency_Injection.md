# Mastering Dependency Injection in C#: From Basics to Redis-Powered Caching

## 🧩 Introduction: Why Dependency Injection Matters

Dependency Injection (DI) is more than just a buzzword — it's a fundamental design pattern that helps build **modular, testable, and maintainable software**. By inverting the control of how dependencies are created and delivered, DI decouples classes from their concrete implementations.

Why does this matter in real-world applications?

* It reduces duplication of logic and instances.
* It allows easy swapping of implementations (e.g., from an in-memory cache to Redis).
* It makes unit testing a breeze — no need to spin up the entire system.

In this post, we’ll walk through how DI works in C#, starting with the "bad old way," then move into a clean, flexible design using DI. We’ll finish with a real-world use case: caching with Redis.

---

## 🚫 The Naive Approach: Without Dependency Injection

Let’s say we’re building a simple user registration feature, and we need to send welcome emails. A typical, tightly-coupled implementation might look like this:

```csharp
public class EmailService
{
    public void Send(string to, string message)
    {
        Console.WriteLine($"Sending email to {to}: {message}");
    }
}

public class UserController
{
    private EmailService _emailService = new EmailService(); // tightly coupled

    public void RegisterUser(string email)
    {
        // Register the user...
        _emailService.Send(email, "Welcome!");
    }
}
```

### 🔍 What's the problem here?

* **Tight coupling**: `UserController` directly controls the creation of `EmailService`.
* **Duplication**: Every controller or service that needs email functionality creates its own instance.
* **Hard to test**: Want to test `UserController` without sending real emails? Good luck stubbing or mocking this setup.
* **Inflexibility**: What if we want to swap `EmailService` with an external API provider? You’re stuck refactoring everywhere.

---

## ✅ Refactored with Dependency Injection

Let’s clean things up using **constructor injection**, the most common DI technique in C#.

```csharp
public class EmailService
{
    public void Send(string to, string message)
    {
        Console.WriteLine($"Sending email to {to}: {message}");
    }
}

public class UserController
{
    private readonly EmailService _emailService;

    public UserController(EmailService emailService)
    {
        _emailService = emailService;
    }

    public void RegisterUser(string email)
    {
        _emailService.Send(email, "Welcome!");
    }
}
```

We now let an external system (usually a DI container) provide the `EmailService`. This allows for:

* **Reusability**: One shared `EmailService` can be reused across controllers.
* **Testability**: Easily inject a mock email service during testing.
* **Flexibility**: Swap the service implementation at runtime without touching `UserController`.

In an ASP.NET Core app, registering dependencies in `Startup.cs` looks like this:

```csharp
services.AddSingleton<EmailService>();
services.AddTransient<UserController>();
```

Every `UserController` gets a new instance, but they all share the same `EmailService` instance (because it's registered as a singleton).

In the next section, we’ll take this concept further by exploring how DI gives you the **flexibility to switch between local memory and distributed Redis caching**, all without changing your core logic.

---

## ⚙️ Real-World Use Case: Caching with Redis vs. Local Memory

In real-world applications, caching is essential for performance and scalability. Whether it's user sessions, product data, or API results — caching can dramatically reduce load times and system strain.

But here's the challenge:
**How do you design your system so you can easily switch between a local memory cache (good for dev or simple apps) and a distributed cache like Redis (needed in production)?**

The answer? **Dependency Injection + Interfaces.**

---

### 🧱 Step 1: Define a Cache Interface

We start by abstracting the caching logic with an interface:

```csharp
public interface ICacheService
{
    Task<string> GetAsync(string key);
    Task SetAsync(string key, string value);
}
```

---

### 🚀 Step 2: Implement Local Memory Cache

```csharp
public class InMemoryCacheService : ICacheService
{
    private readonly Dictionary<string, string> _cache = new();

    public Task<string> GetAsync(string key)
    {
        _cache.TryGetValue(key, out var value);
        return Task.FromResult(value);
    }

    public Task SetAsync(string key, string value)
    {
        _cache[key] = value;
        return Task.CompletedTask;
    }
}
```

---

### 🌐 Step 3: Implement Redis Cache

```csharp
public class RedisCacheService : ICacheService
{
    private readonly IDatabase _redis;

    public RedisCacheService(IConnectionMultiplexer redis)
    {
        _redis = redis.GetDatabase();
    }

    public async Task<string> GetAsync(string key)
    {
        return await _redis.StringGetAsync(key);
    }

    public async Task SetAsync(string key, string value)
    {
        await _redis.StringSetAsync(key, value, TimeSpan.FromMinutes(30));
    }
}
```

---

### ⚙️ Step 4: Register Based on Environment

With DI, you can **dynamically choose the cache implementation** based on environment:

```csharp
if (env.IsDevelopment())
{
    services.AddSingleton<ICacheService, InMemoryCacheService>();
}
else
{
    services.AddSingleton<IConnectionMultiplexer>(
        ConnectionMultiplexer.Connect("localhost:6379"));
    services.AddSingleton<ICacheService, RedisCacheService>();
}
```

Now any controller or service can request `ICacheService`, and it will **automatically get the right implementation** based on the runtime environment.

---

### 🧪 Bonus: Using It in a Controller

```csharp
public class ProductController
{
    private readonly ICacheService _cache;

    public ProductController(ICacheService cache)
    {
        _cache = cache;
    }

    public async Task<string> GetProduct(string productId)
    {
        var cached = await _cache.GetAsync(productId);
        if (cached != null)
            return $"From cache: {cached}";

        // Simulate database fetch
        string productData = $"Product_{productId}_Details";
        await _cache.SetAsync(productId, productData);

        return $"From DB: {productData}";
    }
}
```

> With zero changes to `ProductController`, you can now switch between local and Redis cache simply by changing your DI registration.

---

## 🧠 Why Redis + DI is So Powerful

* Redis is a **distributed, in-memory key-value store** — perfect for applications that need to share data across multiple servers or containers.
* Using DI ensures **your application logic remains agnostic** to the caching mechanism.
* You can swap out cache implementations at runtime without modifying any core logic or business code.
* DI helps manage Redis connections (via `IConnectionMultiplexer`) as singletons, ensuring efficient resource usage.

---

## ✅ Conclusion

Dependency Injection is more than just a code organization tool — it’s a cornerstone of **scalable, flexible, and testable** application architecture.

By using DI:

* You avoid unnecessary instance duplication.
* You enable seamless testing and mocking.
* You gain the flexibility to switch between implementations (e.g., Redis vs. memory cache) without rewriting your logic.

And when paired with a system like Redis, DI allows you to **scale horizontally, cache intelligently, and maintain clean boundaries** in your application code.
