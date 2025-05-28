# 精通 C# 中的依赖注入：从基础到 Redis 缓存实战

## 🧩 引言：为什么依赖注入如此重要

依赖注入（Dependency Injection，简称 DI）不仅仅是个流行词——它是构建**模块化、可测试、易维护软件**的基础设计模式。通过反转依赖的创建与传递方式，DI 使类摆脱对具体实现的耦合。

这在真实世界的应用中有什么意义？

* 减少逻辑与实例的重复。
* 允许轻松替换实现（例如，从内存缓存切换到 Redis）。
* 测试轻松 —— 无需启动整个系统即可单元测试。

本文将带你一步步了解 C# 中的 DI：从“老旧糟糕”的方式开始，转向干净灵活的设计，并以 Redis 缓存的实战场景收尾。

---

## 🚫 天真的实现方式：没有依赖注入

假设我们要开发一个用户注册功能，并发送欢迎邮件。一种典型但紧耦合的实现可能如下：

```csharp
public class EmailService
{
    public void Send(string to, string message)
    {
        Console.WriteLine($"发送邮件到 {to}：{message}");
    }
}

public class UserController
{
    private EmailService _emailService = new EmailService(); // 强耦合

    public void RegisterUser(string email)
    {
        // 注册用户逻辑...
        _emailService.Send(email, "欢迎！");
    }
}
```

### 🔍 这有什么问题？

* **强耦合**：`UserController` 自己创建了 `EmailService` 的实例。
* **重复**：每个需要邮件功能的类都会新建实例。
* **难以测试**：想测试 `UserController` 而不真正发送邮件？很难 stub 或 mock。
* **缺乏灵活性**：想换成使用第三方邮件 API？得到处重构。

---

## ✅ 使用依赖注入的重构方案

我们通过 **构造函数注入**（最常见的 DI 技术）来改善设计：

```csharp
public class EmailService
{
    public void Send(string to, string message)
    {
        Console.WriteLine($"发送邮件到 {to}：{message}");
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
        _emailService.Send(email, "欢迎！");
    }
}
```

我们现在让外部系统（通常是 DI 容器）提供 `EmailService` 实例。优势包括：

* **可复用性**：共享的 `EmailService` 实例可供多个控制器使用。
* **可测试性**：可轻松注入 mock 邮件服务进行测试。
* **灵活性**：无需改动控制器即可替换服务实现。

在 ASP.NET Core 中，我们可以在 `Startup.cs` 注册服务：

```csharp
services.AddSingleton<EmailService>();
services.AddTransient<UserController>();
```

每个 `UserController` 都会获取一个新的实例，但它们共用同一个 `EmailService` 实例（因为它是 singleton）。

接下来我们会进一步探讨，看看如何使用 DI 来轻松切换本地缓存与 Redis 缓存，而无需修改核心业务逻辑。

---

## ⚙️ 实战案例：在 Redis 与本地缓存之间切换

在真实应用中，缓存对性能和扩展性至关重要。无论是用户会话、产品数据还是 API 结果，缓存都能极大地减少加载时间和系统负载。

问题是：
**如何设计系统，才能轻松在本地缓存（适用于开发或简单场景）与 Redis（适用于生产环境）之间切换？**

答案就是：**依赖注入 + 接口编程**。

---

### 🧱 第一步：定义缓存接口

我们先定义一个抽象接口，封装缓存逻辑：

```csharp
public interface ICacheService
{
    Task<string> GetAsync(string key);
    Task SetAsync(string key, string value);
}
```

---

### 🚀 第二步：实现本地内存缓存

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

### 🌐 第三步：实现 Redis 缓存

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

### ⚙️ 第四步：根据环境注册服务

使用 DI，我们可以根据环境**动态选择缓存实现**：

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

现在，任何控制器或服务只需请求 `ICacheService`，它就会**自动获取正确的实现**。

---

### 🧪 加分项：在控制器中使用

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
            return $"来自缓存：{cached}";

        // 模拟数据库查询
        string productData = $"Product_{productId}_Details";
        await _cache.SetAsync(productId, productData);

        return $"来自数据库：{productData}";
    }
}
```

> 无需改动 `ProductController` 的任何代码，只需调整 DI 注册逻辑，就能在 Redis 与本地缓存之间自由切换。

---

## 🧠 为什么 Redis + DI 如此强大？

* Redis 是**分布式的内存型键值数据库**，非常适合需要跨服务或容器共享数据的场景。
* DI 让应用逻辑**与缓存实现解耦**。
* 你可以在运行时切换缓存实现，而不需要改动业务代码。
* 使用 DI 管理 `IConnectionMultiplexer` 连接，可以确保资源利用最优化（使用单例模式复用连接）。

---

## ✅ 总结

依赖注入不仅仅是组织代码的工具 —— 它是构建**可扩展、灵活、可测试**系统架构的基石。

通过使用 DI：

* 避免了不必要的实例创建。
* 能轻松进行单元测试和模拟依赖。
* 可以灵活切换实现（如 Redis 和本地缓存），无需改动业务逻辑。

当 DI 与 Redis 结合使用时，你的系统就能实现**水平扩展、智能缓存、干净的模块边界** —— 面对高并发也能从容应对。
