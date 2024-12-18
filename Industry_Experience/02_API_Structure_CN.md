## 从零到一：在 .NET 中构建高效模块化 RESTful API 的实战指南

### 介绍

在现代软件开发中，创建可扩展和可维护的应用程序至关重要。这就是 RESTful API 和模块化架构的原则发挥作用的地方。在本文中，我们将探讨如何在.NET 中设置 RESTful API，同时确保模块化项目结构。通过本指南，您将掌握创建具有干净、可维护代码的 API 知识。

### 设置项目

首先，让我们开始创建项目结构。我们的项目将包括以下层次：

- **控制器（Controllers）**：处理传入的 HTTP 请求。
- **ExampleProject**：封装核心业务逻辑，包括存储库、模型、模块等。

#### 项目结构

这是我们项目结构的高级概览：

```
MyApiProject/
|-- Controllers/
|   |-- ExampleController.cs
|-- ExampleProject/
|   |-- Repositories/
|   |   |-- ExampleQueries.cs
|   |-- Models/
|   |   |-- ExampleModels.cs
|   |-- Modules/
|   |   |-- ExampleModules.cs
```

### 创建 RESTful API

#### 设置控制器

控制器负责处理 HTTP 请求并返回响应。让我们创建一个简单的控制器：

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using System.Web.Http;
using System.Web.Http.Cors;

namespace ExampleWeb.Controllers
{
    [EnableCors(origins: "*", headers: "*", methods: "*")]
    public class ExampleController : ApiController
    {
        private MediaTypeHeaderValue JSON = new MediaTypeHeaderValue("application/json");
		private ExampleBLL exampleBLL = new ExampleBLL();

		public ExampleController()
		{
		}

        [HttpGet]
        [Route("api/Example/GetData")]
        public HttpResponseMessage GetData([FromUri] GetDataRequest request)
		{
			try
			{
				if (request == null)
				{
					return Request.CreateResponse(HttpStatusCode.BadRequest, "Request cannot be null. ", JSON);
				}

				List<ExampleModel> data = exampleBLL.GetData(request);

				return Request.CreateResponse(HttpStatusCode.OK, data, JSON);
			}
			catch (Exception e)
			{
				string error = e.Message + Environment.NewLine + e.StackTrace;
				return Request.CreateResponse(HttpStatusCode.InternalServerError, "Failed to Get Data.", JSON);
			}
		}

        [HttpPost]
		[Route("api/Example/PostData")]
		public HttpResponseMessage PostData(PostDataRequest request)
		{
			try
			{
				if (request == null)
				{
					return Request.CreateResponse(HttpStatusCode.BadRequest, "Request cannot be null. ", JSON);
				}

				PostDataResponse response = exampleBLL.PostData(request);

				return Request.CreateResponse(HttpStatusCode.OK, response, JSON);
			}
			catch (Exception e)
			{
				string error = e.Message + Environment.NewLine + e.StackTrace;
				return Request.CreateResponse(HttpStatusCode.InternalServerError, "Failed to Post Data.", JSON);
			}
		}
    }
}
```

### 设置 ExampleProject 层

创建表示数据库表的数据模型：

```csharp
namespace ExampleModels
{
    public class ExampleModel
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
    }
}
```

创建一个查询类来使用 Dapper 处理数据库操作：

**ExampleQueries.cs**

```csharp
using Repository; // 检查之前的文章了解如何设置Dapper
using ExampleModels;
using System;
using System.Collections.Generic;
using System.Linq;

namespace ExampleProject.Repositories
{
    public class ExampleQueries : DbQuery
    {
        public ExampleQueries() : base(DapperConstants.DB_Example)
		{
		}

		public Data GetData(){
			// sql.. param.. Query..
		}

		public Data PostData(){
			// sql.. param.. Execute..
		}
    }
}
```

模块处理核心业务操作，并使用存储库与数据库交互：

**ExampleBLL.cs**

```csharp
using Repository;
using System;
using System.Collections.Generic;
using System.Linq;
using ExampleProject.Models;
using ExampleProject.Repositories;

namespace ExampleProject.ExampleModules
{
    public class ExampleBLL
    {
        public ExampleBLL() { }

		private readonly ExampleQueries exampleQueries = new ExampleQueries();

		public List<ExampleModel> GetData(GetDataRequest request){
			// 使用exampleQueries做一些操作..
		}


		public PostDataResponse PostData(PostDataRequest request){
			// 使用exampleQueries做一些操作..
		}
    }
}
```

### 最佳实践

#### 控制器最佳实践

- **精简控制器**：保持控制器专注于处理 HTTP 请求和响应。将业务逻辑委托给服务。
- **HTTP 状态码**：返回适当的 HTTP 状态码（例如，200 OK，201 Created，404 Not Found）。
- **错误处理**：实现全局异常处理中间件以优雅地管理错误。

#### 服务和存储库最佳实践

- **关注点分离**：确保服务处理业务逻辑，存储库管理数据访问。
- **单一职责原则**：每个模块、服务和存储库应只有一个责任。

#### 通用最佳实践

- **单元测试和集成测试**：编写测试以确保代码按预期运行。
- **配置管理**：使用配置文件和环境变量存储敏感数据和设置。
- **日志记录和监控**：实现日志记录以进行调试和监控。可以使用 Serilog 等工具。
- **API 文档**：使用 Swagger/OpenAPI 记录您的 API 端点。

### 结论

在本文中，我们介绍了在.NET 中构建具有模块化架构的 RESTful API 的基础知识。我们讨论了如何设置控制器，实现 DapperLayer，定义模型，以及创建模块和服务。通过遵循最佳实践，您可以确保 API 可扩展、可维护且易于扩展。

请记住，随着您对这些技术的深入了解，可以进一步探索增强和优化。编程愉快！
