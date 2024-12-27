# From Zero to One: A Practical Guide to Building Efficient Modular RESTful APIs in .NET

## Introduction

In modern software development, creating scalable and maintainable applications is crucial. This is where the principles of RESTful APIs and modular architecture come into play. In this article, we will explore how to set up a RESTful API in .NET while ensuring a modular project structure. By following this guide, you will gain the knowledge to create APIs with clean, maintainable code.

---

## Setting Up the Project

Let’s start by creating the project structure. Our project will include the following layers:

- **Controllers**: Handle incoming HTTP requests.
- **ExampleProject**: Encapsulates core business logic, including repositories, models, modules, etc.

### Project Structure

Here’s an overview of our project structure:

```plaintext
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

---

## Creating the RESTful API

### Setting Up the Controller

The controller is responsible for handling HTTP requests and returning responses. Let’s create a simple controller:

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
                    return Request.CreateResponse(HttpStatusCode.BadRequest, "Request cannot be null.", JSON);
                }

                List<ExampleModel> data = exampleBLL.GetData(request);

                return Request.CreateResponse(HttpStatusCode.OK, data, JSON);
            }
            catch (Exception e)
            {
                string error = e.Message + Environment.NewLine + e.StackTrace;
                return Request.CreateResponse(HttpStatusCode.InternalServerError, "Failed to get data.", JSON);
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
                    return Request.CreateResponse(HttpStatusCode.BadRequest, "Request cannot be null.", JSON);
                }

                PostDataResponse response = exampleBLL.PostData(request);

                return Request.CreateResponse(HttpStatusCode.OK, response, JSON);
            }
            catch (Exception e)
            {
                string error = e.Message + Environment.NewLine + e.StackTrace;
                return Request.CreateResponse(HttpStatusCode.InternalServerError, "Failed to post data.", JSON);
            }
        }
    }
}
```

### Setting Up the ExampleProject Layer

Create a model to represent data tables in the database:

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

Create a query class to handle database operations using Dapper:

**ExampleQueries.cs**:

```csharp
using Repository; // Refer to previous articles on how to set up Dapper.
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
            // SQL, parameters, Query...
        }

        public Data PostData(){
            // SQL, parameters, Execute...
        }
    }
}
```

The module handles core business operations and interacts with the database using the repository:

**ExampleBLL.cs**:

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
            // Perform operations using exampleQueries...
        }

        public PostDataResponse PostData(PostDataRequest request){
            // Perform operations using exampleQueries...
        }
    }
}
```

---

## Best Practices

### Controller Best Practices

- **Keep Controllers Lightweight**: Focus controllers on handling HTTP requests and responses. Delegate business logic to services.
- **HTTP Status Codes**: Return appropriate HTTP status codes (e.g., 200 OK, 201 Created, 404 Not Found).
- **Error Handling**: Implement global exception handling middleware to gracefully manage errors.

### Service and Repository Best Practices

- **Separation of Concerns**: Ensure services handle business logic and repositories manage data access.
- **Single Responsibility Principle**: Each module, service, and repository should have a single responsibility.

### General Best Practices

- **Unit and Integration Testing**: Write tests to ensure code works as expected.
- **Configuration Management**: Use configuration files and environment variables to store sensitive data and settings.
- **Logging and Monitoring**: Implement logging for debugging and monitoring. Tools like Serilog can be helpful.
- **API Documentation**: Use Swagger/OpenAPI to document your API endpoints.

---

## Conclusion

In this article, we covered the basics of building a modular RESTful API in .NET. We discussed setting up controllers, implementing a Dapper layer, defining models, and creating modules and services. By following best practices, you can ensure your API is scalable, maintainable, and easy to extend.

As you dive deeper into these techniques, explore further enhancements and optimizations. Happy coding!
