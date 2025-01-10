# Roadmap to Backend Programming Master: API Basics Every Backend Developer Should Know

Welcome to the fifth article in our series aimed at providing a comprehensive roadmap to mastering backend programming. In this post, we'll dive into APIs, a core component of backend development. Whether you're building a simple web application or a complex distributed system, understanding APIs is critical. This guide will cover fundamental API concepts, demonstrate how to set up a project in .NET, and explore common authentication methods. We'll also discuss best practices for API versioning and documentation. Let's get started!

---

## **Introduction to APIs**

API (Application Programming Interface) enables communication between different software applications. In web development, APIs are typically used to connect the frontend of a web application to its backend or to integrate different services.

| **API Type** | **Advantages**                 | **Disadvantages**            |
|--------------|--------------------------------|------------------------------|
| **REST**     | Simple, widely adopted, clear structure | Can be verbose for complex data |
| **JSON API** | Efficient payload, structured responses | Complexity in adhering to standards |
| **SOAP**     | Secure, transactional, formal structure | Verbose, slower |
| **gRPC**     | Fast, supports streaming, lightweight | Requires setup, less human-readable |
| **GraphQL**  | Flexible, efficient queries   | Complex to implement, potential security issues |

### **REST**

REST (Representational State Transfer) is the most common API architectural style characterized by stateless communication and a predefined set of HTTP methods (`GET`, `POST`, `PUT`, `DELETE`, etc.). REST APIs are popular for their simplicity, scalability, and use of standard web protocols.

- **Advantages**: Easy to understand, widely adopted, straightforward to implement.
- **Disadvantages**: Can be cumbersome when handling complex queries.

### **JSON API**

JSON API is a standard for structuring API responses using the JSON format. It aims to reduce the amount of data transferred over the network while maintaining clarity. JSON API enforces strict guidelines on how clients should request and modify data.

- **Advantages**: Reduces payload size, avoids over-fetching or under-fetching data.
- **Disadvantages**: Slightly more complex than basic REST.

**Request**:

```plaintext
GET /api/products/123
```

**Response**:

```json
{
  "data": {
    "type": "products",
    "id": "123",
    "attributes": {
      "name": "Wireless Mouse",
      "price": 29.99,
      "inStock": true
    }
  }
}
```

The JSON API standard defines a specific data representation format, providing clarity and structure, especially useful for frontend applications dealing with complex data relationships.

### **SOAP**

SOAP (Simple Object Access Protocol) is a protocol-based API that encodes messages in XML. It is known for its strict standards and focus on security and transaction integrity.

- **Advantages**: Strong security, suitable for complex operations.
- **Disadvantages**: Verbose, slower, harder to implement than REST.

SOAP uses XML as its request and response format, making it verbose but highly structured:

**Request** (XML format):

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:prod="http://example.com/products">
   <soapenv:Header/>
   <soapenv:Body>
      <prod:GetProduct>
         <prod:id>123</prod:id>
      </prod:GetProduct>
   </soapenv:Body>
</soapenv:Envelope>
```

**Response** (XML format):

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
   <soapenv:Body>
      <GetProductResponse>
         <Product>
            <id>123</id>
            <name>Wireless Mouse</name>
            <price>29.99</price>
            <inStock>true</inStock>
         </Product>
      </GetProductResponse>
   </soapenv:Body>
</soapenv:Envelope>
```

SOAP's strict structure and support for built-in security and transaction handling make it ideal for environments where data integrity is critical.

### **gRPC**

gRPC is a modern, high-performance framework that uses Protocol Buffers (a binary serialization format) instead of JSON. It aims to improve speed and efficiency, especially in microservices environments.

- **Advantages**: Very fast, lightweight payload, supports streaming.
- **Disadvantages**: Less human-readable, requires more setup.

gRPC uses **Protocol Buffers** to define data models. The following example demonstrates gRPC's setup, highlighting its difference from JSON:

**ProductService.proto**:

```protobuf
syntax = "proto3";

service ProductService {
  rpc GetProduct(ProductRequest) returns (ProductResponse);
}

message ProductRequest {
  int32 id = 1;
}

message ProductResponse {
  int32 id = 1;
  string name = 2;
  double price = 3;
  bool inStock = 4;
}
```

**Binary Data Exchange**: Unlike JSON APIs, gRPC transmits data in a compact binary format, reducing size and improving speed. Here's an example of a gRPC response at the binary level:

```plaintext
\0x08\x7B\x12\x0EWireless Mouse\x19\x00\x00\x3E\x40\x20\x01
```

gRPC is highly efficient, using a binary data format for communication. It supports advanced features like data streaming, making it ideal for performance-critical applications.

### **GraphQL**

GraphQL is a query language for APIs that allows clients to request exactly the data they need. It offers greater flexibility compared to REST, enabling clients to define the structure of the response.

- **Advantages**: High flexibility, efficient data retrieval, minimizes over-fetching.
- **Disadvantages**: More complex to implement, security challenges, steep learning curve.

GraphQL lets clients precisely define the data they need. Unlike other APIs that return fixed structures, GraphQL queries are more flexible:

**Request**:

```graphql
query {
  product(id: "123") {
    id
    name
    price
    inStock
  }
}
```

**Response**:

```json
{
  "data": {
    "product": {
      "id": "123",
      "name": "Wireless Mouse",
      "price": 29.99,
      "inStock": true
    }
  }
}
```

GraphQL provides flexibility, allowing clients to request only the data they need without over-fetching. Responses are returned in JSON format, making it both familiar and efficient for web developers.

---

## **Setting Up an API in .NET**

In a previous blog, I detailed a step-by-step guide to setting up an API in .NET. If you haven't read it, you can [check it out here](https://example.com). This section summarizes the process and explains why understanding API setup is crucial.

Building APIs in .NET offers hands-on experience in creating a structured, secure, and scalable web service. Mastering API setup helps you:

- **Structure Data Access**: Learn to organize data flow between clients and servers efficiently and securely.
- **Implement Security**: Understand how to protect endpoints with authentication and authorization methods like JWT or OAuth.
- **Follow API Standards**: Proper setup ensures adherence to best practices and standards (e.g., RESTful design), making your service more consistent and user-friendly.
- **Handle Errors**: Create meaningful error responses to help clients understand problems effectively.
- **Optimize Performance**: Learn to enhance database queries, caching, and load balancing for optimal performance.

By mastering API setup, you'll be better equipped to tackle real-world backend tasks and create reliable services that adapt and scale with your projects. For the complete guide and code examples, refer to my previous blog!

---

## **API Authentication Basics**

Authentication is crucial for protecting APIs from unauthorized access. Below is a quick overview of common authentication methods:

### **JWT (JSON Web Token)**

JWT is a compact and self-contained token format widely used in RESTful API authentication. It allows information to be securely transmitted between parties as a JSON object. This token is typically generated upon user login and subsequently used to authenticate API requests.

- **Use Cases**: Ideal for stateless authentication in modern web applications, especially when protecting API endpoints without maintaining server-side sessions.

#### **Implementation Example in .NET**

To illustrate how JWT authentication works, let's explore a specific example using a `.NET` API.

##### **Step 1: Generate JWT Upon User Login**

After a successful user login, a JWT is generated. Here's a simplified example of generating a JWT in a `.NET` project:

```csharp
public string GenerateJwtToken(string userId)
{
    var tokenHandler = new JwtSecurityTokenHandler();
    var key = Encoding.ASCII.GetBytes("YourSecretKey"); // Replace with your actual key

    var tokenDescriptor = new SecurityTokenDescriptor
    {
        Subject = new ClaimsIdentity(new Claim[]
        {
            new Claim(ClaimTypes.NameIdentifier, userId)
        }),
        Expires = DateTime.UtcNow.AddHours(1),
        SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature)
    };

    var token = tokenHandler.CreateToken(tokenDescriptor);
    return tokenHandler.WriteToken(token);
}
```

In this example:

- The JWT includes a claim for the user ID (`userId`).
- The token is set to expire in 1 hour.
- The token is signed using a secret key to ensure its integrity.

##### **Step 2: Secure API Endpoints**

To protect API endpoints (e.g., `GetUserInfo`) using JWT authentication, follow these steps:

1. **Configure JWT in `Startup.cs`**:

   Add JWT authentication configuration in the `ConfigureServices` method:

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
           .AddJwtBearer(options =>
           {
               options.TokenValidationParameters = new TokenValidationParameters
               {
                   ValidateIssuer = true,
                   ValidateAudience = true,
                   ValidateLifetime = true,
                   ValidateIssuerSigningKey = true,
                   ValidIssuer = "yourdomain.com",
                   ValidAudience = "yourdomain.com",
                   IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("YourSecretKey"))
               };
           });

       services.AddControllers();
   }
   ```

   Ensure you replace `"YourSecretKey"` with a secure key for your application.

2. **Protect the `GetUserInfo` Endpoint**:

   Use the `[Authorize]` attribute to enforce authentication:

   ```csharp
   [Authorize]
   [ApiController]
   [Route("[controller]")]
   public class UserController : ControllerBase
   {
       [HttpGet("GetUserInfo")]
       public IActionResult GetUserInfo(string userId)
       {
           // Safely retrieve user information
           var userInfo = GetUserDetails(userId);
           return Ok(userInfo);
       }
   }
   ```

##### **Step 3: Use JWT in Client Requests**

Clients (e.g., a webpage) should send the JWT in the `Authorization` header for each request:

```javascript
// Example using JavaScript Fetch API
fetch('https://yourapi.com/User/GetUserInfo?userId=123', {
    method: 'GET',
    headers: {
        'Authorization': 'Bearer YOUR_JWT_TOKEN_HERE'
    }
})
.then(response => response.json())
.then(data => {
    console.log(data); // Handle user information response
})
.catch(error => {
    console.error('Error:', error);
});
```

In this example:

- The client sends the JWT token in the `Authorization` header as `Bearer YOUR_JWT_TOKEN_HERE`.
- The server validates the token before granting access to the requested data.

By implementing JWT authentication, you create a stateless and secure method of user authentication, perfect for modern web applications.

### **OAuth**

OAuth allows your application to delegate authentication to trusted third-party providers (e.g., Google), offering a secure way to manage user credentials without handling passwords directly. Here's how it works in our `.NET` project example:

1. **User Logs in with Google**:
   - The user clicks "Login with Google" on your webpage.
   - The user is redirected to Google's login page to enter credentials.

2. **Obtain Authorization Code**:
   - Google requests user consent to share information with your application.
   - After consent, Google redirects the user back to your API with an authorization code.

3. **Exchange Authorization Code for Access Token**:
   - Your server exchanges the authorization code for an access token.
   - This token is used to authenticate subsequent API requests.

4. **Access Protected API**:
   - Use the token to authenticate users and securely access the `GetUserInfo` endpoint.

#### **Implementation in .NET**

1. **OAuth Setup**: Configure Google authentication in your `.NET` API:

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddAuthentication(options =>
       {
           options.DefaultAuthenticateScheme = CookieAuthenticationDefaults.AuthenticationScheme;
           options.DefaultChallengeScheme = GoogleDefaults.AuthenticationScheme;
       })
       .AddCookie()
       .AddGoogle(options =>
       {
           options.ClientId = "YOUR_GOOGLE_CLIENT_ID";
           options.ClientSecret = "YOUR_GOOGLE_CLIENT_SECRET";
           options.CallbackPath = new PathString("/signin-google");
       });

       services.AddControllers();
   }
   ```

2. **Initiate Login**: Create an endpoint to start the OAuth flow:

   ```csharp
   [ApiController]
   [Route("[controller]")]
   public class AuthController : ControllerBase
   {
       [HttpGet("Login")]
       public IActionResult Login()
       {
           return Challenge(new AuthenticationProperties { RedirectUri = "/Auth/Callback" }, "Google");
       }

       [HttpGet("Callback")]
       public IActionResult Callback()
       {
           // Redirect after successful login
           return Redirect("/User/GetUserInfo");
       }
   }
   ```

3. **Protect `GetUserInfo` API**: Restrict access to authenticated users with the `[Authorize]` attribute:

   ```csharp
   [Authorize]
   [ApiController]
   [Route("[controller]")]
   public class UserController : ControllerBase
   {
       [HttpGet("GetUserInfo")]
       public IActionResult GetUserInfo()
       {
           var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
           var userInfo = GetUserDetails(userId); // Safely fetch user info
           return Ok(userInfo);
       }
   }
   ```

Using OAuth:

- Your application relies on trusted providers (e.g., Google) for managing user authentication.
- User credentials are never exposed to your server.
- OAuth tokens are used to securely access protected resources.

### **Basic vs. Token-Based vs. Cookie-Based Authentication**

**Basic Authentication** uses a simple HTTP header to send a username and password in Base64 encoding.

- **When to Use**: Suitable for simple or internal APIs without complex security requirements.
- **Drawbacks**: Exposes credentials in every request; must be used over HTTPS for security.

Understanding the differences between **Token-Based** and **Cookie-Based** authentication is essential to choosing the right method for your API. Below is a quick comparison with code examples highlighting their differences:

#### **Key Differences**

| Aspect                        | Cookie-Based Authentication                      | Token-Based Authentication                   |
|-------------------------------|--------------------------------------------------|-----------------------------------------------|
| **Storage**                   | Session stored on the server, cookies on client | Tokens stored on client (local/session storage) |
| **State Management**          | Server maintains session state                  | Stateless; server doesn't store session state |
| **Client Handling**           | Cookies are automatically managed by the browser| Tokens must be manually included in requests   |
| **Security Considerations**   | Vulnerable to CSRF without CSRF token; use HTTPS| Vulnerable to XSS if improperly stored; expiry reduces risk |
| **Use Cases**                 | Suitable for traditional web apps with server-rendered pages | Suitable for modern SPAs, mobile apps, scalable APIs |

##### **Cookie-Based Example**

In this example, the server creates a session upon user login and sends a cookie to the client. The browser automatically includes the cookie in each request.

```csharp
[HttpPost("login")]
public IActionResult Login([FromBody] LoginRequest request)
{
    if (ValidateCredentials(request))
    {
        // Create session and set cookie
        var cookieOptions = new CookieOptions
        {
            HttpOnly = true,
            Secure = true,
            Expires = DateTime.Now.AddHours(1)
        };
        Response.Cookies.Append("SessionId", CreateSession(request.UserId), cookieOptions);
        return Ok();
    }
    return Unauthorized();
}

[Authorize]
[HttpGet("GetUserInfo")]
public IActionResult GetUserInfo()
{
    var userId = GetUserIdFromSession(Request.Cookies["SessionId"]);
    var userInfo = GetUserDetails(userId);
    return Ok(userInfo);
}
```

##### **Token-Based Example**

Here, the server generates a JWT upon user login. The client stores the token and includes it in the `Authorization` header for each API request.

```csharp
[HttpPost("login")]
public IActionResult Login([FromBody] LoginRequest request)
{
    if (ValidateCredentials(request))
    {
        var token = GenerateJwtToken(request.UserId);
        return Ok(new { Token = token });
    }
    return Unauthorized();
}

[Authorize]
[HttpGet("GetUserInfo")]
public IActionResult GetUserInfo()
{
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var userInfo = GetUserDetails(userId);
    return Ok(userInfo);
}
```

By understanding these differences, you can decide whether Cookie-Based or Token-Based authentication is more suitable for your API, depending on use cases and security requirements.

---

## **API Versioning and Documentation**

Managing API versions is critical to maintaining compatibility when introducing changes. Here are some common strategies:

### **API Versioning Strategies**

1. **URL Versioning**: Include the version number in the URL:

   ```plaintext
   /api/v1/products
   ```

2. **Query Parameter Versioning**: Include the version as a query parameter:

   ```plaintext
   /api/products?version=1
   ```

3. **Header Versioning**: Specify the version in the `Accept` header:

   ```plaintext
   Accept: application/vnd.myapi.v1+json
   ```

### **API Documentation**

Proper documentation is essential for making your API easy to understand and integrate. Swagger is a powerful tool that helps automate the creation and maintenance of API documentation.

- **Using Swagger**:
  - **Add Swagger Middleware**: Integrate Swagger into your .NET Core project with a few lines of configuration to generate documentation automatically.
  - **Use Annotations**: Add simple comments and attributes to your API endpoints to make the documentation clear and informative.
  - **Visual Interface**: Access the Swagger UI to view a user-friendly, visual representation of your API. The Swagger interface also allows direct testing of API endpoints, simplifying development and debugging.

Proper API versioning and documentation enhance the developer experience, provide structured guidance for consuming your service, and ensure compatibility as the API evolves.

For a detailed guide on setting up Swagger for .NET Core, including both automatic and manual configuration options, make sure to check out [my previous article](../Industry_Experience/06_Swagger_API_Doc.md).

---

## **Conclusion**

In this article, we covered the basics of APIs, from understanding what an API is and comparing different types to setting up APIs in .NET and implementing authentication. We also highlighted the importance of API versioning and documentation, with tools like Swagger simplifying the process of creating clear and maintainable API documentation. These foundational concepts set the stage for building robust and scalable web services, equipping you with the tools needed to create effective APIs.

In the next article, we will delve into advanced topics in API design. We will explore input validation, scalability considerations, handling concurrency, performance best practices, and more. These advanced concepts will help you level up your API skills, ensuring the APIs you build are not only functional but also secure, efficient, and scalable.

Stay tuned as we continue this journey into mastering backend programming, step by step!
