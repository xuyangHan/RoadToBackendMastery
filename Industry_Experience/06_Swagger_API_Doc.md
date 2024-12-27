# Simplified API Documentation: Automatic Generation with Swagger and Manual Creation Methods

Clearly, API documentation is crucial in various scenarios: whether for internal team reference, public documentation, or just to record available APIs. Swagger is an excellent tool that serves as a template to automatically generate documentation as long as the required format is followed.

In this blog post, I will guide you step by step through two methods:

1. **Automatically generate Swagger documentation from an existing .NET Core project.**
2. **Manually create Swagger files and use MkDocs to generate a static documentation site.**

***

## Part 1: Automatically Generate Swagger Documentation from a .NET Core Project

Assuming you already have a .NET Core Web API project, we can easily integrate Swagger to automatically generate API documentation.

### Step 1: Add Swagger NuGet Package

Use the NuGet Package Manager or the following command to install the `Swashbuckle.AspNetCore` package:

```bash
dotnet add package Swashbuckle.AspNetCore
```

Once the package is installed, Swagger will automatically configure the `Program.cs` file. Since .NET 6+ uses a simplified `Program.cs`, you don't need to manually add a `Startup.cs` file.

### Step 2: Verify Configuration

After installation, you will notice that the necessary Swagger configuration has been added automatically to the `Program.cs` file, including:

```csharp
// For more information on configuring Swagger/OpenAPI, visit https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

This configuration enables Swagger generation and the UI. Although `app.UseSwaggerUI()` is left empty, Swagger UI will still be available at the default URL: `http://localhost:<port>/swagger`.

### Step 3: View the Automatically Generated Documentation

Run your project and navigate to `http://localhost:<port>/swagger` to view your automatically generated Swagger documentation.

![swagger_doc_1.jpeg](../assets/images/swagger_doc_1.jpeg)

***

## Part 2: Manually Create Swagger YAML/JSON Files and Use MkDocs to Generate a Static Site

For better control over your API documentation, you can manually create Swagger files and use MkDocs to generate a static site.

### Step 1: Project Structure and File Layout

Create a directory structure to organize your Swagger documentation and MkDocs configuration. The basic structure should look like this:

``` plaintext
    swagger-server/
    │
    ├── docs/
    │   └── api-doc.yml
    │   └── index.md
    │
    ├── mkdocs.yml
    └── Pipfile
```

* `docs/api-doc.yml`: This is where you store your Swagger specification.
* `mkdocs.yml`: The MkDocs configuration file.
* `Pipfile`: Used to add dependencies needed for the virtual environment.

### Step 2: Set Up the Virtual Environment

It’s recommended to set up a virtual environment to manage your dependencies locally. We will use **Pipenv** to easily manage the virtual environment and dependencies.

Assuming you already have Python installed (preferably version 3.12), if you don't have Pipenv installed, you can install it via pip:

```bash
pip install pipenv
```

Next, create a `Pipfile` to manage your project's dependencies. Here’s the content for our project’s `Pipfile`:

```Pipfile
    [[source]]
    url = "https://pypi.org/simple"
    verify_ssl = true
    name = "pypi"

    [packages]
    mkdocs = "*"
    mkdocs-render-swagger-plugin = "*"
    mkdocs-swagger-ui = "*"

    [dev-packages]

    [requires]
    python_version = "3.12.6"
```

With the `Pipfile` in place, run the following command to install dependencies:

```bash
pipenv install
```

This will create a virtual environment and install all the necessary packages specified in the `Pipfile`.

### Step 3: Create Swagger YAML File

In the `docs/` folder, create `api-doc.yml` to define your API documentation. Here’s an example:

```yaml
openapi: 3.0.0
info:
  title: My API
  description: A sample API for demonstration purposes
  version: 1.0.0
paths:
  /users:
    get:
      summary: Get all users
      responses:
        '200':
          description: A JSON array of user objects
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: integer
                    name:
                      type: string
  /users/{id}:
    get:
      summary: Get a user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User details
        '404':
          description: User not found
```

### Step 4: Configure MkDocs

In the root directory, create the `mkdocs.yml` file to configure MkDocs:

```yaml
site_name: My API Documentation
theme: readthedocs

nav:
- API reference:
  - MyApp: index.md

plugins:
  - swagger-ui-tag
```

This configuration will load the Swagger specification you created and apply the `swagger-ui-tag` plugin to display it in a user-friendly format.

`index.md`:

``` xml
<swagger-ui src="api-doc.yml"/>
```

### Step 5: Build and Serve the Documentation

Once everything is set up, you can build and serve the documentation locally:

```bash
mkdocs build // Create site folder
mkdocs serve // Start a local server
```

Access `http://127.0.0.1:8000/` in your browser to view the Swagger-generated API documentation hosted via MkDocs.

![swagger_doc_2.jpeg](../assets/images/swagger_doc_2.jpeg)

### Step 6: Host Your Documentation

To host your documentation, you can deploy the static site generated by MkDocs to any platform that supports static site hosting, such as:

* **GitHub Pages**
* **Netlify**
* **AWS S3**

***

## Conclusion

In this article, we introduced two methods for creating API documentation with Swagger:

1. **Automatically generated documentation** through a .NET Core project for quick and easy API recording.
2. **Manually creating Swagger YAML/JSON files** and using MkDocs to generate a static documentation site, giving you full control over the look and style of the documentation.

Both methods are powerful, and you can choose the one that best fits your project needs. If you have any questions or suggestions, feel free to leave a comment below.

Happy documenting!
