# 锁定 Swagger：在 .NET 中实现基础认证与 API 密钥安全

## 1. 引言与回顾

在我的上一篇文章 **[第一部分：从 .NET Core 项目自动生成 Swagger 文档](06_Swagger_API_Doc_CN.md)** 中，我们介绍了集成 Swashbuckle 的基础知识，将 C# 控制器转换为精美且具交互性的文档页面。这对于本地开发来说是革命性的，让你只需在 `localhost` 点击一下即可测试接口。

然而，当你的项目离开本地环境并部署到测试或生产服务器时，一个新的挑战随之而来：**“Localhost” 的保护气泡破裂了。**

### 开放地图的风险

当你部署 API 时，仅仅将 Swagger 移至公共 URL（例如 `https://api.yourdomain.com/swagger`）就像是在银行正门上贴了一张详细的保险库蓝图。

向公共互联网暴露原始 API 架构（Schema）会带来重大风险：

* **攻击面映射：** 黑客无需猜测你的接口；Swagger 会准确告诉他们敏感数据的存放位置。
* **信息泄露：** 你的内部数据模型、参数类型和逻辑流程将成为公开知识。
* **资源耗尽：** 如果没有保护，任何人（或任何机器人）都可以使用 “Try it out” 功能向你的数据库发送垃圾请求。

### 超越开发模式

在默认的 .NET 模板中，你会注意到 Swagger 被包裹在 `if (app.Environment.IsDevelopment())` 块中。虽然这通过完全隐藏文档保证了生产服务器的安全，但当你需要与内部其他部门、前端团队或 QA 测试人员共享这些文档时，这并不方便。

在本文中，我们将弥补这一差距。我们将演示如何在保持生产环境 Swagger 可用的同时，添加**两层核心安全防护**：

1. **用户凭证 (Basic Auth)：** 这是一个“前门”锁，要求输入用户名和密码才能查看文档页面。
2. **API 密钥 (API Keys)：** 这是一个“门禁卡”系统，确保只有获得授权的用户才能真正从 Swagger 界面内执行 API 调用。

让我们深入了解如何在不牺牲实用性的情况下保护你的文档。

---

## 2. 文档门户管控（“前门”）

### 目标

保护已部署 Swagger 实例的第一步是确保文档网站本身不是公开的。我们要防止未经授权的用户、爬虫和机器人看到我们的 API 定义。

### 解决方案：HTTP 基础认证 (Basic Authentication)

为了实现这一点，我们为 HTTP 基础认证实现一个**自定义中间件**。当用户访问 `/swagger` URL 时，服务器将拦截请求，并在渲染任何内容之前向浏览器发起挑战，要求输入用户名和密码。

### 实现：`SwaggerBasicAuthMiddleware`

以下是中间件逻辑。它检查请求路径是否以 `/swagger` 开头，查找 `Authorization` 请求头，并根据你的配置验证凭据。

```csharp
public class SwaggerBasicAuthMiddleware
{
    private readonly RequestDelegate _next;
    public SwaggerBasicAuthMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        // 仅针对 Swagger 端点
        if (context.Request.Path.StartsWithSegments("/swagger"))
        {
            string authHeader = context.Request.Headers["Authorization"];
            if (authHeader != null && authHeader.StartsWith("Basic "))
            {
                var header = System.Net.Http.Headers.AuthenticationHeaderValue.Parse(authHeader);
                var credentials = System.Text.Encoding.UTF8.GetString(Convert.FromBase64String(header.Parameter)).Split(':');
                var user = credentials[0];
                var password = credentials[1];

                // 根据你的设置进行验证（建议从环境变量中读取）
                if (user == "admin" && password == "YourStrongPassword") 
                {
                    await _next(context);
                    return;
                }
            }

            // 如果未经授权，向浏览器发送挑战头
            context.Response.Headers["WWW-Authenticate"] = "Basic";
            context.Response.StatusCode = 401;
        }
        else
        {
            await _next(context);
        }
    }
}

```

### 理解 `WWW-Authenticate` 响应头

这一行代码触发了“魔法”：

`context.Response.Headers["WWW-Authenticate"] = "Basic";`

当浏览器收到 `401 Unauthorized` 状态码以及这个特定的响应头时，它会自动触发其**原生登录弹窗**。这是一种无需构建自定义登录页面或 UI 即可添加安全性的轻量级方法。

### 配置：注册中间件

在 .NET 中间件管道中，顺序决定一切。为了保护 Swagger，你必须在 `Program.cs` 调用 Swagger 服务**之前**注册你的自定义中间件。

如果你将其放置在 `app.UseSwagger()` 之后，文档将在安全检查运行之前就已经加载了！

```csharp
var app = builder.Build();

// 1. 首先注册基础认证以管控页面
app.UseMiddleware<SwaggerBasicAuthMiddleware>();

// 2. 然后启用 Swagger
app.UseSwagger();
app.UseSwaggerUI();

app.UseHttpsRedirection();
// ... 管道的其余部分

```

通过添加这一层，你已经成功地将 API “蓝图”从公众视野中隐藏了起来。

---

## 3. 授权 API 调用（“门禁卡”）

### 目标

现在“前门”已经锁上了，我们面临第二个挑战：允许授权用户实际测试接口。由于我们的生产环境 API 本身可能受到其安全逻辑的保护，我们需要一种方法，让 Swagger 在通过 UI 发出的每个请求中都能携带一张“门禁卡”（API 密钥）。

### 步骤 A：Swagger UI 设置

为了实现这一点，我们需要在 `Program.cs` 的 `AddSwaggerGen` 配置中告诉 Swagger 我们的 API 预期会接收一个安全令牌。

我们使用两个特定的方法：

1. **`AddSecurityDefinition`**：定义安全性如何工作（例如：Header 中的 API Key）。
2. **`AddSecurityRequirement`**：确保端点上出现“锁”图标，并确保在我们点击 “Execute” 时真正发送该密钥。

```csharp
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

    // 定义 API Key 安全方案
    c.AddSecurityDefinition("apiKey", new OpenApiSecurityScheme
    {
        Description = "请输入 API Key 以解锁端点。",
        Name = "api_key",             // Header 或查询参数的名称
        In = ParameterLocation.Header, // 我们将其放在 Header 中
        Type = SecuritySchemeType.ApiKey,
        Scheme = "ApiKeyScheme"
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "apiKey" }
            },
            Array.Empty<string>()
        }
    });
});

```

#### **Header vs. Query：应该使用哪种？**

* **`ParameterLocation.Header`**：现代 API 的标准做法。密钥作为 HTTP Header 发送（例如 `api_key: your_key_here`）。它更整洁，且不会出现在服务器日志或浏览器历史记录中。
* **`ParameterLocation.Query`**：密钥附加在 URL 后面（例如 `?api_key=your_key_here`）。这便于快速调试，但安全性较低，因为密钥在地址栏中清晰可见。

---

### 智能中间件：`SwaggerApiKeyMiddleware`

仅仅在 Swagger 页面上添加按钮是不够的；服务器需要真正去*验证*那个密钥。我们使用自定义中间件来拦截这些调用。

### **专业技巧：来源引用（Referer）逻辑**

一个常见的问题是，我们只希望这个特定的 API Key 检查应用于**使用 Swagger 页面的人员**。我们不希望破坏 Postman 用户或移动应用的访问，因为它们可能使用完全不同的认证系统（如 JWT）。

通过检查 **`Referer`** 请求头，我们可以检测请求是否源自 Swagger UI。

```csharp
public class SwaggerApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private const string APIKEYNAME = "api_key";

    public SwaggerApiKeyMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        string referer = context.Request.Headers["Referer"].ToString();

        // 仅当请求来自 Swagger UI 时才强制执行此检查
        if (!string.IsNullOrEmpty(referer) && referer.Contains("/swagger"))
        {
            if (!context.Request.Headers.TryGetValue(APIKEYNAME, out var extractedApiKey))
            {
                context.Response.StatusCode = 401;
                await context.Response.WriteAsync("API Key 缺失。请使用 Swagger 中的 'Authorize' 按钮。");
                return;
            }

            var actualKey = Environment.GetEnvironmentVariable("SWAGGER_API_KEY");
            if (actualKey == null || !actualKey.Equals(extractedApiKey))
            {
                context.Response.StatusCode = 401;
                await context.Response.WriteAsync("无效的 API Key。");
                return;
            }
        }

        await _next(context);
    }
}

```

### 为什么说这是“智能”的

该中间件充当了**上下文感知守卫**。如果你从 Postman 调用 API，中间件发现没有“Swagger”来源，就会让开路径，让你标准的 API 认证逻辑接管。但一旦有人尝试在你公开的 Swagger 页面上点击 “Try it out” 按钮，他们就必须输入正确的 API Key。

这在内部团队的易用性与针对外部好奇心的安全性之间取得了完美的平衡。

---

## 4. 整合：中间件管道

在 ASP.NET Core 中，注册中间件的顺序与代码本身一样重要。如果你将安全检查放在端点映射之后，那么“大门”就设在了“房子”后面。

为了确保我们的双层安全机制正常工作，你的 `Program.cs` 管道应遵循以下特定顺序：

### “顺序至上”的拆解

1. **`app.UseMiddleware<SwaggerBasicAuthMiddleware>()`**：从这里开始。这确保了当有人访问 `/swagger` 时，服务器做的第一件事就是要求输入用户名和密码。
2. **`app.UseSwagger()` & `app.UseSwaggerUI()**`：一旦用户通过基础认证，我们才允许 Swagger 引擎生成 JSON 文档并渲染 UI。
3. **`app.UseRouting()`**：这用于识别传入请求想要访问哪个控制器/动作（Action）。
4. **`app.UseMiddleware<SwaggerApiKeyMiddleware>()`**：路由确立后，我们检查 `Referer`。如果请求来自刚刚加载的 Swagger UI，我们强制进行 API Key 检查。
5. **`app.MapControllers()`**：最后，如果通过了所有守卫，请求才被允许到达你实际的 API 逻辑。

### 管道可视化

| 顺序 | 组件 | 目的 |
| --- | --- | --- |
| 1 | **基础认证中间件** | 拦截未经授权的文档“网站”查看。 |
| 2 | **Swagger 中间件** | 提供文档文件服务。 |
| 3 | **API Key 中间件** | 拦截源自 Swagger 的未经授权的接口执行。 |
| 4 | **端点映射** | 执行实际的 API 业务逻辑。 |

---

## 5. 结论

### 深度防御

我们实现的是一种经典的**“深度防御”**策略。我们没有依赖单一的安全措施，而是分层进行了防护：

* **第一层**保护你的知识产权和 API 架构不被索引或扫描。
* **第二层**保护你的数据和服务器资源，防止可能已经获取文档访问权限的未经授权用户滥用。

通过使用 **Referer 逻辑**，我们创建了一个“智能”安全系统，它知道什么时候该严格（针对基于浏览器的 Swagger 用户），什么时候该为 Postman 或移动客户端等其他授权集成工具让路。

### 何时使用此方案 vs. 完整的 Identity Server/JWT

虽然这套设置对于内部工具、测试环境或中小型团队协作非常有效，但在高风险的生产环境中，它并不能替代全尺寸的身份提供商（如 Keycloak、Auth0 或 Microsoft Entra ID）。

* **使用此方法：** 当你需要一种快速、稳健的方式与内部其他部门或信任的合作伙伴共享文档，而又不想背负完整的 OAuth2 设置开销时。
* **迁移到 JWT/OpenID Connect：** 当你的 API 需要细粒度的用户权限（Scopes）、多因素认证，或者正在被公开的第三方开发者调用时。

保护 Swagger 并不意味着要隐藏它。通过几行策略性的中间件代码，你可以将你的文档从安全隐患转变为整个组织的安全、专业的资产。

---
