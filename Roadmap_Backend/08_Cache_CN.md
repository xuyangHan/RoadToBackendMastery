# 从小白到大神：后端开发者必学之缓存技术（Cache）

欢迎来到**后端编程大师之路**系列的又一篇章！**缓存**是提升后端应用性能的最有效方法之一。通过临时存储频繁访问的数据，可以减少重复计算和数据库查询，从而缩短响应时间并减轻服务器负载。

在本文中，我们将深入探讨服务器端缓存，以增强后端性能，为处理高流量应用和优化服务器负载提供坚实的基础。我们将向您介绍核心的缓存概念，涵盖服务器端缓存策略，如数据缓存、`Redis` 和 `Memcached`。我们还会简要介绍`客户端缓存`和 `CDN`，使您全面了解缓存在整个技术栈中的作用，但我们的重点是帮助您掌握后端缓存的专业知识，让您的 API 既快速又稳定。

---

## **服务器端（Server-side）缓存**

服务器端缓存涉及将数据存储在离后端更近的位置，以减少延迟和服务器负载。让我们探讨一些关键技术：

### **数据缓存**

数据缓存涉及在内存中临时存储频繁访问的数据，从而减少重复数据库查询的需求。这对于不经常变化的数据（如参考数据或用户偏好）特别有用。

**示例**：缓存一个经常访问的产品详细信息的数据库查询，将其存储在内存缓存中，设置 30 分钟的生存时间（TTL）。

要在 .NET 应用程序中设置数据缓存并使其可供业务逻辑层（BLL）访问，我们可以使用 `IMemoryCache`，这是 .NET Core 中的内置缓存机制。以下是一个逐步指南，包括代码示例和解释。

#### **1. 在 Startup 类中配置 `IMemoryCache`**

首先，您需要在 `Startup.cs` 文件中将 `IMemoryCache` 添加到服务集合中。

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMemoryCache(); // 注册 IMemoryCache
        services.AddTransient<IProductService, ProductService>(); // 注册 BLL 服务
    }
}
```

#### **2. 创建使用缓存的 BLL 服务**

假设我们在 BLL 中有一个 `ProductService` 类用于检索产品详细信息。我们将使用 `IMemoryCache` 临时存储产品数据，以避免重复的数据库调用。

```csharp
public interface IProductService
{
    Task<Product> GetProductAsync(int productId);
}

public class ProductService : IProductService
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _productRepository;
    private readonly TimeSpan _cacheDuration = TimeSpan.FromMinutes(30); // TTL

    public ProductService(IMemoryCache cache, IProductRepository productRepository)
    {
        _cache = cache;
        _productRepository = productRepository;
    }

    public async Task<Product> GetProductAsync(int productId)
    {
        string cacheKey = $"Product_{productId}";

        // 尝试从缓存中获取数据
        if (_cache.TryGetValue(cacheKey, out Product product))
        {
            return product;
        }

        // 如果不在缓存中，从存储库获取数据
        product = await _productRepository.GetProductByIdAsync(productId);

        // 将数据存储在缓存中，并设置过期时间
        _cache.Set(cacheKey, product, _cacheDuration);

        return product;
    }
}
```

- **缓存查找**：`TryGetValue` 检查产品是否已缓存。如果可用，则直接返回缓存的产品，从而节省数据库查询。
- **缓存存储**：如果产品未被缓存，则方法从 `_productRepository` 中检索它，并使用唯一键（例如，`"Product_{productId}"`）和 30 分钟的 TTL 将其存储在缓存中。

#### **3. 在控制器中访问缓存数据**

在您的控制器中，您可以调用 `ProductService.GetProductAsync` 来获取产品详细信息，自动利用缓存。

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _productService.GetProductAsync(id);
        if (product == null)
        {
            return NotFound();
        }
        return Ok(product);
    }
}
```

- 当调用 `GetProduct` 方法时，`ProductService.GetProductAsync` 首先尝试从缓存中检索产品数据。
- 如果数据存在于缓存中，它会立即返回。否则，它会从存储库中获取、缓存，然后返回。

通过这种方式缓存数据，我们最小化数据库调用，显著提高频繁请求数据的性能。

### **缓存方法结果（定时记忆化）**

缓存方法结果涉及存储函数调用的输出。这减少了相同输入产生相同结果时的计算开销。

- **示例**：如果您的 API 基于相同的数据频繁计算运费，则可以缓存这些结果一段指定的时间。
- **实现**：有关使用定时记忆化技术的详细代码示例，请参考[之前关于缓存方法结果的帖子](https://juejin.cn/post/7378328335036661799)。

### **Memcached**

Memcached 是一个高性能的分布式内存对象缓存系统，理想用于缓存小块数据，如数据库查询结果、会话数据或渲染页面。与 `IMemoryCache` 不同，`Memcached` 旨在分布式操作，意味着您可以横向扩展到多个服务器。这对于需要跨多个机器的缓存系统的高负载应用程序来说是非常有益的。

`Memcached` 通常作为独立服务器（或服务器集群）运行，多个应用程序实例可以连接到它。此设置允许您集中缓存数据，使您应用程序的不同部分或多个应用程序能够共享缓存数据。

要在 .NET 应用程序中使用 Memcached，您通常通过客户端库与之交互，如 `EnyimMemcached` 或 `StackExchange.Redis`（在与 Redis 一起使用时）。存储和检索缓存数据的方法在概念上与 `IMemoryCache` 相似，但涉及连接到 Memcached 服务器：

```csharp
public class ProductService
{
    private readonly IMemcachedClient _memcachedClient;
    private readonly IProductRepository _productRepository;
    private readonly TimeSpan _cacheDuration = TimeSpan.FromMinutes(30);

    public ProductService(IMemcachedClient memcachedClient, IProductRepository productRepository)
    {
        _memcachedClient = memcachedClient;
        _productRepository = productRepository;
    }

    public async Task<Product> GetProductAsync(int productId)
    {
        string cacheKey = $"Product_{productId}";

        // 尝试从 Memcached 获取产品
        var product = await _memcachedClient.GetAsync<Product>(cacheKey);
        if (product != null)
        {
            return product;
        }

        // 如果未缓存，从数据库检索并缓存
        product = await _productRepository.GetProductByIdAsync(productId);
        await _memcachedClient.SetAsync(cacheKey, product, _cacheDuration);

        return product;
    }
}
```

在此示例中：

- `IMemcachedClient` 表示与 Memcached 交互的客户端接口。
- `SetAsync` 和 `GetAsync` 方法处理从 Memcached 服务器存储和检索数据，与 `IMemoryCache` 类似。

使用 Memcached 是一个更具可扩展性的解决方案，适用于大型分布式应用程序，而 `IMemoryCache` 更适合用于较小的单服务器应用程序。两者都非常有效，选择取决于您应用程序的架构和缓存需求。

### **Redis**

Redis 是一个内存中的键值存储，由于其高级数据结构支持和持久性选项，广泛用于缓存。

`Redis` 的工作方式与 `Memcached` 类似，作为独立的缓存服务器，但它具有额外的功能和灵活性，使其成为更复杂的缓存场景中的热门选择。

1. **高级数据结构**：Redis 支持多种数据类型——字符串、列表、集合、哈希和有序集合，使其高度灵活。这使您能够直接在缓存中存储不仅仅是简单值，还可以存储更结构化的数据，如最近活动的列表或用户会话。

2. **持久性选项**：与仅在内存中保留数据的 Memcached 不同，Redis 提供可选的持久性。它可以将数据写入磁盘，从而在服务器重启时恢复数据。这使得 Redis 适合于需要缓存但不能在重启时丢失所有数据的情况。

3. **原子操作**：Redis 为其数据结构提供内置的原子操作。例如，您可以原子性地递增计数器、向列表中添加项目或更新有序集合。这对于需要在缓存中可靠地进行并发数据更新的应用程序非常有用。

4. **发布/订阅消息**：Redis 支持发布/订阅（pub/sub）模式，消息会广播给所有订阅者。这对于需要实时通知或消息流的应用程序非常方便。

```csharp
using StackExchange.Redis;

// 连接到 Redis 服务器
var redis = ConnectionMultiplexer.Connect("localhost:6379");
var db = redis.GetDatabase();

// 缓存一个频繁查询的用户会话，设置 TTL
string userId = "user:12345";
string sessionData = "user_session_data_here";
db.StringSet(userId, sessionData, TimeSpan.FromMinutes(30));

// 检索缓存的会话数据
var cachedSession = db.StringGet(userId);
Console.WriteLine(cachedSession); // 输出: user_session_data_here
```

#### **何时选择 Redis 而非 Memcached**

- 需要 **数据持久性**。
- 需要在缓存中使用 **复杂数据结构** 或 **原子操作**。
- 需要 **实时通知**，此时 Redis 的发布/订阅功能非常有用。
- **性能可扩展性** 是优先考虑的，因为 Redis 提供强大的扩展选项。

Redis 的高级功能使其适合更复杂、数据驱动的应用程序，而 Memcached 仍然是简单数据缓存的理想选择。

---

## **客户端（Client-side）缓存**

客户端缓存侧重于在用户设备上存储数据，以减少服务器负载并增强用户体验。

### **浏览器缓存**

现代浏览器将图像、CSS、JavaScript 和 API 响应等资源本地缓存，以减少加载时间和带宽使用。

- **实现**：使用 HTTP 头控制缓存行为：
  - **Cache-Control**：指定资源应缓存多久。
  - **ETag**：检查文件是否已更改。
  - **Expires**：为缓存设置过期日期。

- **示例**：设置 `Cache-Control: max-age=3600` 将使资源缓存一小时。

### **LocalStorage/SessionStorage**

HTML5 提供了 `LocalStorage` 和 `SessionStorage` 用于在客户端存储数据。`LocalStorage` 在会话之间持久存在，而 `SessionStorage` 在会话结束时被清除。

- **使用场景**：用于存储需要持久化的非敏感数据（例如，用户偏好或向导中的表单输入）。

---

## **内容分发网络 (CDN)**

CDN 是一个全球分布的服务器网络，负责将静态内容（如图像、样式表、脚本）缓存到离最终用户更近的地方。这可以减少延迟并提高 Web 应用程序的加载速度。

### **CDN 如何工作**

CDN 在靠近用户的边缘服务器上缓存内容。当用户请求资源时，内容会从最近的 CDN 节点交付，而不是从源服务器获取，从而降低延迟和带宽成本。

- **边缘缓存与源缓存**：
  - **边缘缓存**：在 CDN 的边缘服务器上缓存内容。
  - **源缓存**：直接在后台服务器上缓存内容。

### **使用 CDN 的好处**

- **改善加载时间**：资源从离用户更近的服务器交付。
- **降低带宽成本**：将流量从源服务器卸载到 CDN。
- **增强可用性**：CDN 提供冗余；如果一个节点失败，另一个节点可以提供内容。

后端开发人员通常不会直接处理 CDN 缓存，因为这通常由 DevOps 或基础设施团队管理。然而，理解 CDN 缓存是很有价值的，因为后端 API 往往需要为与 CDN 层的兼容性进行优化，尤其是在内容交付速度至关重要的大型应用程序中。

特别是，后端程序员应该了解缓存头、缓存失效和 TTL（生存时间）设置如何与 CDN 配合使用，因为这些配置可能会影响数据如何提供给客户端，以及后端被调用的频率。掌握这些知识可以使后端开发人员能够有效地与 DevOps 团队合作，并优化 API 以适应 CDN 缓存。因此，虽然后端开发人员在日常工作中不会管理 CDN，但对这一概念的深入理解是有益的。

---

## **缓存策略**

缓存不仅仅是存储数据——有效的缓存需要为你的用例选择合适的策略：

- **旁路缓存 (Cache-aside)**：应用程序代码在访问数据库之前检查缓存。如果数据不在缓存中，则从数据库中获取数据，然后将其缓存以供将来使用。

- **写入缓存 (Write-through)**：数据同时写入缓存和数据库。这确保缓存始终与数据库同步。

- **写回缓存 (Write-back)**：数据最初写入缓存，然后异步地持久化到数据库，从而提高写入性能。

- **读缓存 (Read-through)**：缓存负责获取数据，如果数据不在缓存中，从而简化应用程序代码。

### **缓存失效**

缓存面临的最大挑战之一是知道何时使陈旧的数据失效。以下是一些常见策略：

- **生存时间 (TTL)**：在设定时间后自动过期数据。
- **手动失效**：当数据不再有效时，显式地将其从缓存中删除。
- **陈旧数据保留 (Stale-While-Revalidate)**：在后台请求获取新数据时，提供陈旧数据。

---

## **结论**

作为掌握后端开发路线图的一部分，缓存是一项基础技能，可以极大地增强应用程序的性能和可扩展性。在本文中，我们探讨了从服务器端缓存到客户端缓存及 CDN 使用的基本缓存技术，以及 `Memcached` 和 `Redis` 等流行工具。有效的缓存需要一种战略性的方法，平衡缓存层、理解缓存过期以及知道何时使数据失效。掌握这些原则将使你能够构建响应迅速、高效的后端系统，轻松处理更高的负载和复杂性。

请继续关注本系列的下一篇文章，我们将深入探讨高级后端策略和优化，以继续提升你的开发技能！
