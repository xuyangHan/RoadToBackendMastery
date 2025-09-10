# **C# 代码审查面试准备：实用示例与技巧**

## 📌 如何应对这种面试

我最近参加了一场挺有意思的面试，不是传统的白板题或者脑筋急转弯，而是直接给你一段代码，让你去审查——就像在真实的 pull request 里帮初级开发者 review 一样。

老实说，我觉得这种面试方式特别好。它能真正考察你有没有**实际写过和维护过代码**：你能不能发现 bug、给出改进建议，并且讲得让人听得懂，而不是摆出一副“我比你厉害”的架势？这种形式也越来越常见，因为它既考技术，又考沟通能力。

我将分享一些**常见代码问题**，以及如何解释和提出修复方案。可以把它当作准备指南，让你在面对这种问题时更有信心。

---

## 🔹 第 1 部分：设计与可维护性问题

在很多这类面试中，环境设置比较真实。通常不会只给你一个孤立的代码片段，而是通过 Visual Studio Code Live Share 之类的方式，给你一个小项目。通常是简单的 .NET Web 应用或 API，代码量足以模拟真实仓库。

在深入单个文件之前，先**从整体结构观察**：

* 层次是否清晰（Controllers、Services、Repositories）？
* 依赖是否注入，还是类中硬编码？
* 类是否遵循单一职责，还是混合了多个功能？

本节列出了常见的**设计和可维护性问题**，可以作为“整体视角”的审查，先了解大局，再聚焦具体 bug。

### 1. 分层架构

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

在面试中，首先要看代码是否遵循分层原则。一个健康的项目通常具有：

* **Controllers** → **仅处理 HTTP 请求/响应**，不包含业务逻辑。
* **Services** → 包含业务逻辑（规则、验证、计算），可独立测试。
* **Repositories** → 只负责数据访问，可替换（SQL、NoSQL、Mock 仓库等）。
* **Models / Modules** → 按功能或层组织良好。
* 架构支持未来拆分为微服务，而不是把所有东西堆在一个大项目中。

📂 **示例文件结构：**

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

这样，如果公司未来拆分微服务或替换持久化层，影响最小——层之间已经解耦。

### 2. 紧耦合 vs 依赖注入

```csharp
public class OrderService
{
    private readonly SqlOrderRepository _repo = new SqlOrderRepository();
    public void PlaceOrder(Order order) => _repo.Save(order);
}
```

⚠️ **问题：**

* `OrderService` 与 `SqlOrderRepository` **紧密耦合**。
* 如果以后想换数据存储（NoSQL、内存、测试 Mock），必须 **修改 OrderService 代码**。
* 系统难以测试、维护和扩展。

✅ **用依赖注入修复：**

```csharp
public class OrderService
{
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) => _repo = repo;

    public void PlaceOrder(Order order) => _repo.Save(order);
}
```

* 利用 DI 可以轻松 Mock 依赖：

```csharp
var mockRepo = new Mock<IOrderRepository>();
mockRepo.Setup(r => r.Save(It.IsAny<Order>()));

var service = new OrderService(mockRepo.Object);
service.PlaceOrder(new Order { Id = 1, Amount = 100 });
```

现在：

* `OrderService` 依赖 **抽象 (`IOrderRepository`)** 而非具体类。
* 可以轻松替换实现（SQL、MongoDB、Mock）。
* 单元测试更简单——只需注入假仓库。
* 遵循 **依赖倒置原则 (DIP)**，属于 SOLID 原则之一。

👉 我之前写过一篇 [**文章**](../CSharp_OOP/08_Dependency_Injection_CN.md) 深入讲解 **.NET 中的依赖注入**，包括内置 DI 容器的用法，非常适合进一步阅读。

### 3. 责任过多 (God Class / 关注点分离)

常见问题是一个类尝试做所有事情——创建用户、发送邮件、记录日志、处理支付、生成报表等。
这违反 **单一职责原则 (SRP)**，代码难以测试、扩展和维护。

❌ 示例：

```csharp
public class UserManager
{
    public void CreateUser(string name) { /*...*/ }
    public void SendWelcomeEmail(string email) { /*...*/ }
    public void LogUserCreation(string message) { /*...*/ }
}
```

或者：

```csharp
public class OrderProcessor
{
    public void ProcessPayment(Order order) { /*...*/ }
    public void GenerateMonthlyReport(List<Order> orders) { /*...*/ }
}
```

✅ 修复：拆分成独立的、聚焦的类：

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

👉 每个类 **职责单一**，设计更模块化、可测试且可维护。

### 4. 错误处理与日志最佳实践

遇到问题时，需要同时具备：

* **健壮的错误处理**（不要悄悄吞掉异常）。
* **规范日志**（方便后续调试）。

❌ 不良示例：

```csharp
try 
{ 
    // ...
} 
catch 
{ 
    // 忽略异常，调试噩梦
}
```

```csharp
Console.WriteLine("Something failed"); // 无结构、无上下文
```

✅ 更好的方式（基于之前分层架构示例）：

```csharp
public class UserController : ControllerBase
{
    private readonly IUserService _service;
    private readonly ILogger<UserController> _logger;

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
            return StatusCode(500, "Internal server error");
        }
    }
}
```

✨ 改进点：

* 使用 **`ILogger<T>`** 替代 `Console.WriteLine`。
* 对 **未找到用户** 情况做了处理。
* 包裹服务调用 **try/catch**，记录异常并返回 500，而不是崩溃。

这样，Controller 更加 **可投入生产**，而不仅仅是“演示用”。

---

## 🔹 第 2 部分：常见代码审查问题（正确性与性能）

在理解了**整体项目结构和设计选择**之后，下一步是深入**具体代码本身**。面试官通常希望你能“嗅出”潜在问题——正确性 bug、隐藏的性能陷阱，或未来可能带来麻烦的设计捷径。

在真实的 pull request 中，有经验的开发者会指出这些问题，尤其是给初级同事。为了帮助你准备，我整理了一些**最常见的陷阱**，并展示如何识别和修复它们。

### 1. 空值与异常安全

```csharp
public string GetUserName(User user)
{
    return user.Name.ToUpper(); // 可能抛出 NullReferenceException
}
```

⚠️ **问题原因：**
在真实项目中，`NullReferenceException` 是 **最常见的运行时错误之一**。当对象或属性意外为 `null` 时访问，就会抛出此异常。通过代码审查或写 null-safe 代码可以提前避免大量 bug 和生产问题。

✅ 修复：

```csharp
public string? GetUserName(User? user)
{
    if (string.IsNullOrEmpty(user?.Name)) return null;
    return user.Name.ToUpperInvariant();
}
```

* 使用 null 条件操作符 `?.` 安全访问属性。
* 输入缺失时返回 `null`，避免运行时崩溃。

### 2. Async/Await 使用不当

```csharp
public async Task<string> GetDataAsync()
{
    var result = GetDataFromDbAsync().Result; // 死锁风险
    return result;
}
```

✅ 修复：

```csharp
public async Task<string> GetDataAsync()
{
    var result = await GetDataFromDbAsync();
    return result;
}
```

### 3. 魔法数字 / 硬编码

```csharp
public decimal CalculateDiscount(decimal amount)
{
    return amount * 0.07m;  // 7% 折扣
}
```

⚠️ **问题：**

* `0.07` 是**魔法数字**——不清楚含义且难以维护。
* 折扣率变更时需要**修改代码并重新部署**，容易出错。
* 调试和理解业务逻辑更困难。

✅ 修复（使用常量或参数）：

```csharp
private const decimal DefaultDiscountRate = 0.07m;

public decimal CalculateDiscount(decimal amount, decimal? discountRate = null)
{
    return amount * (discountRate ?? DefaultDiscountRate);
}
```

💡 **更进一步：**

* 将参数存储在 **配置文件 (`appsettings.json`)**、**数据库** 或 **功能开关系统** 中。
* 这样可以 **无需重部署即可修改数值**，数字也更易理解。

### 4. 并发问题

```csharp
public class Counter
{
    private int count = 0;
    public void Increment()
    {
        count++; // 竞态条件
    }
}
```

⚠️ **问题：**

* 多线程同时调用 `Increment()` 会引发**竞态条件**，`count` 可能无法正确增加。

✅ 基本锁机制修复：

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

💡 **注意：**

* 该类混合了内部状态和行为，单元测试不便。
* 更好的方式是 **尽量减少共享可变状态**，或者返回**新结果**而非修改内部字段。
* 如果必须可变，可使用 **线程安全原语（如 `Interlocked.Increment`）**。

```csharp
public static class Counter
{
    public static int Increment(int currentCount)
    {
        return currentCount + 1; // 无共享可变状态
    }
}
```

### 5. 遍历集合时修改

```csharp
foreach (var user in users)
{
    if (!user.IsActive)
        users.Remove(user); // InvalidOperationException
}
```

✅ 修复：

```csharp
users = users.Where(u => u.IsActive).ToList();
```

### 6. 字符串性能陷阱

```csharp
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString(); // O(n²)，因为字符串不可变
}
```

✅ 修复：

```csharp
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);
}
string result = sb.ToString();
```

### 7. 低效循环 / 算法

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

✅ 修复：

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

### 8. LINQ 重复枚举

```csharp
var query = users.Where(u => u.IsActive);
if (query.Count() > 0)
{
    foreach (var user in query) // 再次执行查询
    {
        Console.WriteLine(user.Name);
    }
}
```

✅ 修复：

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

### 9. 资源管理 / IDisposable

```csharp
public void SaveData(string filePath, string data)
{
    var writer = new StreamWriter(filePath); // 未释放
    writer.WriteLine(data);
}
```

✅ 修复：

```csharp
public void SaveData(string filePath, string data)
{
    using var writer = new StreamWriter(filePath);
    writer.WriteLine(data);
}
```

---

## 🔹 第 3 部分：如何在代码审查面试中脱颖而出

代码审查面试越来越常见，是公司**考察技术能力和实际判断力**的有效方式。目标不仅是找出 bug，还要展示你能像有经验的开发者一样思考：发现问题、理解权衡，并提出可维护的解决方案。

准备时：

1. **从正确性开始**——发现明显 bug 或运行时风险。
2. **关注可读性和性能**——指出低效或难理解的代码。
3. **评估设计与可维护性**——考虑关注点分离、依赖注入和可测试性。
4. **清晰解释思路**——说明问题原因、影响及修复效果。
5. **提供具体、最小示例**——小而独立的代码片段让建议易于理解。

通过按这些原则结构化审查，并将问题分组为 **正确性与性能**、**设计与可维护性**、**最佳实践**，面试官能清楚看到你的**思考过程**。

✅ 这种方法显示你不仅是写代码的人，更是关心**质量、可维护性和实际影响**的开发者。
