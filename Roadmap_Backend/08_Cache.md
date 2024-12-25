# Roadmap to Backend Programming Master: Caching Techniques

Welcome to another chapter in the **Backend Programming Mastery** series! **Caching** is one of the most effective ways to enhance backend application performance. By temporarily storing frequently accessed data, caching reduces redundant computations and database queries, thereby shortening response times and easing server load.

In this article, we will delve into server-side caching to boost backend performance, providing a solid foundation for handling high-traffic applications and optimizing server load. We will introduce core caching concepts, focusing on server-side caching strategies like data caching, `Redis`, and `Memcached`. We will also briefly cover `client-side caching` and `CDNs` to give you a comprehensive understanding of caching across the tech stack. However, our emphasis is on helping you master backend caching, ensuring your APIs are fast and stable.

---

## **Server-side Caching**

Server-side caching involves storing data closer to the backend to reduce latency and server load. Let’s explore some key technologies:

### **Data Caching**

Data caching involves temporarily storing frequently accessed data in memory, reducing the need for repeated database queries. This is particularly useful for infrequently changing data, such as reference data or user preferences.

**Example**: Cache a frequently accessed database query for product details in memory with a 30-minute time-to-live (TTL).

To set up data caching in a .NET application and make it accessible to the Business Logic Layer (BLL), we can use `IMemoryCache`, a built-in caching mechanism in .NET Core. Here's a step-by-step guide with code examples and explanations.

#### **1. Configure `IMemoryCache` in the Startup Class**

First, add `IMemoryCache` to the service collection in the `Startup.cs` file.

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMemoryCache(); // Register IMemoryCache
        services.AddTransient<IProductService, ProductService>(); // Register BLL service
    }
}
```

#### **2. Create a BLL Service That Uses Caching**

Suppose we have a `ProductService` class in the BLL to retrieve product details. We will use `IMemoryCache` to temporarily store product data, avoiding repeated database calls.

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

        // Try to get data from the cache
        if (_cache.TryGetValue(cacheKey, out Product product))
        {
            return product;
        }

        // If not in cache, fetch from repository
        product = await _productRepository.GetProductByIdAsync(productId);

        // Store data in the cache with an expiration time
        _cache.Set(cacheKey, product, _cacheDuration);

        return product;
    }
}
```

- **Cache Lookup**: `TryGetValue` checks whether the product is already cached. If available, it directly returns the cached product, saving a database query.
- **Cache Storage**: If the product is not cached, it is retrieved from `_productRepository` and stored in the cache with a unique key (e.g., `"Product_{productId}"`) and a 30-minute TTL.

#### **3. Access Cached Data in the Controller**

In your controller, call `ProductService.GetProductAsync` to fetch product details, leveraging caching automatically.

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

- When `GetProduct` is called, `ProductService.GetProductAsync` first attempts to retrieve product data from the cache.
- If the data exists in the cache, it is returned immediately. Otherwise, it is fetched from the repository, cached, and then returned.

By caching data this way, we minimize database calls and significantly improve performance for frequently requested data.

### **Caching Method Results (Timed Memoization)**

Caching method results involves storing the output of function calls. This reduces the overhead of repeated computations for identical inputs.

- **Example**: If your API frequently computes shipping costs based on the same data, you can cache these results for a specified time.
- **Implementation**: For detailed code examples of timed memoization, refer to [this previous post on caching method results](https://juejin.cn/post/7378328335036661799).

### **Memcached**

Memcached is a high-performance, distributed memory object caching system ideal for caching small data chunks like database query results, session data, or rendered pages. Unlike `IMemoryCache`, `Memcached` is designed for distributed operation, meaning it can scale horizontally across multiple servers. This is beneficial for high-load applications requiring a caching system across multiple machines.

`Memcached` typically runs as a standalone server (or cluster of servers) that multiple application instances can connect to. This setup allows centralized caching of data, enabling different parts of your application or multiple applications to share cached data.

To use Memcached in a .NET application, you typically interact with it through client libraries like `EnyimMemcached` or `StackExchange.Redis` (when using Redis). Methods for storing and retrieving cached data conceptually resemble those of `IMemoryCache` but involve connecting to the Memcached server:

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

        // Try to get the product from Memcached
        var product = await _memcachedClient.GetAsync<Product>(cacheKey);
        if (product != null)
        {
            return product;
        }

        // If not cached, fetch from the database and cache it
        product = await _productRepository.GetProductByIdAsync(productId);
        await _memcachedClient.SetAsync(cacheKey, product, _cacheDuration);

        return product;
    }
}
```

In this example:

- `IMemcachedClient` represents the client interface for interacting with Memcached.
- `SetAsync` and `GetAsync` handle storing and retrieving data from the Memcached server, similar to `IMemoryCache`.

Using Memcached is a more scalable solution for large, distributed applications, while `IMemoryCache` is better suited for smaller, single-server applications. Both are highly effective, and the choice depends on your application architecture and caching needs.

### **Redis**

Redis is an in-memory key-value store widely used for caching due to its advanced data structure support and persistence options.

Redis operates similarly to Memcached as a standalone caching server but offers additional features and flexibility, making it a popular choice for more complex caching scenarios.

1. **Advanced Data Structures**: Redis supports various data types—strings, lists, sets, hashes, and sorted sets—making it highly versatile. This enables storing not just simple values but also structured data like recent activity lists or user sessions directly in the cache.

2. **Persistence Options**: Unlike Memcached, which retains data only in memory, Redis offers optional persistence. It can write data to disk, allowing recovery after server restarts. This makes Redis suitable for scenarios where cached data must survive restarts.

3. **Atomic Operations**: Redis provides built-in atomic operations for its data structures. For example, you can atomically increment counters, add items to a list, or update sorted sets. This is useful for applications requiring reliable concurrent data updates in the cache.

4. **Publish/Subscribe Messaging**: Redis supports publish/subscribe (pub/sub) patterns where messages are broadcast to all subscribers. This is convenient for applications needing real-time notifications or message streaming.

```csharp
using StackExchange.Redis;

// Connect to the Redis server
var redis = ConnectionMultiplexer.Connect("localhost:6379");
var db = redis.GetDatabase();

// Cache a frequently queried user session with TTL
string userId = "user:12345";
string sessionData = "user_session_data_here";
db.StringSet(userId, sessionData, TimeSpan.FromMinutes(30));

// Retrieve the cached session data
var cachedSession = db.StringGet(userId);
Console.WriteLine(cachedSession); // Output: user_session_data_here
```

#### **When to Choose Redis Over Memcached**

- When **data persistence** is required.
- When **complex data structures** or **atomic operations** are needed in the cache.
- When **real-time notifications** are required, leveraging Redis’s pub/sub capabilities.
- When **scalability** is a priority, as Redis offers robust scaling options.

Redis’s advanced features make it suitable for more complex, data-driven applications, while Memcached remains ideal for simple data caching.

---

## **Client-side Caching**

Client-side caching focuses on storing data on the user’s device to reduce server load and enhance user experience.

### **Browser Caching**

Modern browsers cache resources such as images, CSS, JavaScript, and API responses locally to reduce load times and bandwidth usage.

- **Implementation**: Use HTTP headers to control caching behavior:
  - **Cache-Control**: Specify how long resources should be cached.
  - **ETag**: Check whether a file has changed.
  - **Expires**: Set an expiration date for cached resources.

- **Example**: Setting `Cache-Control: max-age=3600` will cache a resource for one hour.

### **LocalStorage/SessionStorage**

HTML5 provides `LocalStorage` and `SessionStorage` for client-side data storage. `LocalStorage` persists between sessions, while `SessionStorage` is cleared at the end of the session.

- **Use Case**: Store non-sensitive data that needs to persist, such as user preferences or form inputs in a wizard.

---

## **Content Delivery Network (CDN)**

A CDN is a globally distributed network of servers designed to cache static content (such as images, stylesheets, and scripts) closer to end users. This reduces latency and improves the loading speed of web applications.

### **How CDN Works**

CDNs cache content on edge servers located near users. When a user requests a resource, it is delivered from the nearest CDN node instead of the origin server, reducing latency and bandwidth costs.

- **Edge Cache vs. Origin Cache**:
  - **Edge Cache**: Caches content on CDN edge servers.
  - **Origin Cache**: Caches content directly on backend servers.

### **Benefits of Using a CDN**

- **Improved Load Times**: Resources are delivered from servers closer to users.
- **Reduced Bandwidth Costs**: Offloads traffic from origin servers to the CDN.
- **Enhanced Availability**: CDNs provide redundancy; if one node fails, another node can serve the content.

Backend developers typically do not manage CDN caching directly, as this is often handled by DevOps or infrastructure teams. However, understanding CDN caching is valuable since backend APIs often need to be optimized for compatibility with CDN layers, especially in large applications where content delivery speed is critical.

Specifically, backend developers should be familiar with cache headers, cache invalidation, and TTL (Time-To-Live) configurations to understand how data is delivered to clients and how often the backend is called. This knowledge allows backend developers to collaborate effectively with DevOps teams and optimize APIs for CDN caching. While backend developers might not manage CDNs daily, a deep understanding of this concept is highly beneficial.

---

## **Caching Strategies**

Caching is not just about storing data—effective caching requires selecting the right strategy for your use case:

- **Cache-aside**: Application code checks the cache before accessing the database. If the data is not in the cache, it retrieves it from the database and caches it for future use.
- **Write-through**: Data is written to both the cache and the database simultaneously, ensuring the cache is always in sync with the database.
- **Write-back**: Data is initially written to the cache and asynchronously persisted to the database, improving write performance.
- **Read-through**: The cache is responsible for fetching data if it is not already cached, simplifying application code.

### **Cache Invalidation**

One of the biggest challenges in caching is knowing when to invalidate stale data. Common strategies include:

- **Time-To-Live (TTL)**: Automatically expires data after a set time.
- **Manual Invalidation**: Explicitly removes data from the cache when it is no longer valid.
- **Stale-While-Revalidate**: Serves stale data while asynchronously fetching fresh data in the background.

---

## **Conclusion**

Caching is a fundamental skill in mastering the backend development roadmap, greatly enhancing application performance and scalability. In this article, we explored essential caching techniques, from server-side caching to client-side caching and CDN usage, as well as popular tools like `Memcached` and `Redis`. Effective caching requires a strategic approach, balancing cache layers, understanding cache expiration, and knowing when to invalidate data. Mastering these principles will enable you to build highly responsive and efficient backend systems capable of handling greater loads and complexity.

Stay tuned for the next article in this series, where we will delve into advanced backend strategies and optimizations to further elevate your development skills!
