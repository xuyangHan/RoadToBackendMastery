# Writing Clean and Consistent C# Code: A .NET Coding Style Guide

For C# developers working within the .NET ecosystem, maintaining a consistent coding style is essential for building readable, maintainable, and scalable applications. Following established **coding conventions** improves code clarity, reduces errors, and makes collaboration across teams smoother. In this article, we’ll explore key C# and .NET coding standards, inspired by Microsoft’s official guidelines, focusing on naming conventions, formatting, and best practices.

---

## **Why Coding Standards Matter in C#**

In the world of software development, code is written not just for machines but for humans as well.

Adopting consistent coding standards can:

- **Enhance readability**: Well-structured code is easier to read and understand, especially in collaborative environments.
- **Reduce errors**: Consistent formatting and naming practices help identify mistakes early.
- **Improve collaboration**: Shared standards ensure everyone on the team is on the same page across multiple projects.
- **Facilitate maintenance**: Properly formatted code is easier to debug and maintain.

---

## **Naming Conventions in C#**

Naming conventions are critical for making code self-explanatory. Here are some best practices for naming variables, methods, classes, and other elements in C#.

### **PascalCase vs camelCase**

- **PascalCase**: Used for public members, class names, method names, public properties, enums, and filenames. Each word in the name starts with an uppercase letter.  
  - Examples: `CustomerService`, `GetCustomerData()`.
- **camelCase**: Used for local variables and parameters. The first word starts with a lowercase letter. For private, protected, internal, and protected internal fields and properties, prefix with an underscore.  
  - Examples: `customerCount`, `isLoggedIn`, `_customerCount`.

### **Descriptive Names**

- Ensure names are **descriptive** and clearly convey the purpose of a variable or method. For example, avoid using `temp` or `var1`; instead, choose names like `customerList` or `totalRevenue`.

### **Naming Classes and Interfaces**

- Class names should always be **nouns** or **noun phrases** (e.g., `FileManager`, `DataProcessor`).
- Interface names should describe behavior and start with an uppercase `I`, followed by a noun or phrase (e.g., `IRepository`, `ILogger`).

For detailed guidelines on naming, refer to Microsoft’s official [naming conventions](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines).

---

## **Code Formatting Guidelines**

Maintaining consistent code layout and formatting aids both readability and debugging. Here are some essential formatting rules:

### **Indentation**

- Use **two spaces** for each level of indentation—avoid using tabs to ensure consistency across environments.
  - Example:

```csharp
  public void CalculateTotal()
  {
      if (isProcessed)
      {
          total += price;
      }
  }
```

### **Braces**

- Always use braces `{}` in `if`, `else`, `for`, `while`, and other statements, even if the block contains only a single statement. This prevents potential errors when adding additional lines later.
  - Example:

```csharp
  if (isValid)
  {
      ProcessData();
  }
```

### **Method Declaration Format**

- Keep method signatures clear and aligned:
  - Place the return type, method name, and opening parenthesis on a single line.
  - Example:

```csharp
  public string GetCustomerNameById(int customerId)
  {
      // Method body
  }
```

### **Complete Example**

```csharp
using System;                                       // `using` statements should be at the top, outside of the namespace.

namespace MyNamespace {                             // Namespaces use PascalCase.
                                                    // Indent within the namespace.
  public interface IMyInterface {                   // Interface names start with 'I'.
    public int Calculate(float value, float exp);   // Methods use PascalCase, with a space after commas.
  }

  public enum MyEnum {                              // Enums use PascalCase.
    Yes,                                            // Enum constants use PascalCase.
    No,
  }

  public class MyClass {                            // Classes use PascalCase.
    public int Foo = 0;                             // Public member variables use PascalCase.
    public bool NoCounting = false;                 // Encourage the use of field initializers.
    private class Results {
      public int NumNegativeResults = 0;
      public int NumPositiveResults = 0;
    }
    private Results _results;                       // Private member variables use _camelCase.
    public static int NumTimesCalled = 0;
    private const int _bar = 100;                   // `const` does not affect naming conventions.
    private int[] _someTable = {                    // Use 2 spaces for collection initializers.
      2, 3, 4,                                      // Indent with spaces.
    };

    public MyClass() {
      _results = new Results {
        NumNegativeResults = 1,                     // Use 2-space indentation for object initializers.
        NumPositiveResults = 1,                     // Maintain consistency.
      };
    }

    public int CalculateValue(int mulNumber) {      // No newline before the opening brace.
      var resultValue = Foo * mulNumber;            // Local variables use camelCase.
      NumTimesCalled++;
      Foo += _bar;

      if (!NoCounting) {                            // No space after unary operators, space after `if`.
        if (resultValue < 0) {                      // Always use braces, spaces around comparison operators.
          _results.NumNegativeResults++;
        } else if (resultValue > 0) {               // No newline between braces and `else`.
          _results.NumPositiveResults++;
        }
      }

      return resultValue;
    }

    public void ExpressionBodies() {
      // Keep simple lambdas on a single line if possible, without parentheses or braces.
      Func<int, int> increment = x => x + 1;

      // Align the closing brace with the character following the opening brace.
      Func<int, int, long> difference1 = (x, y) => {
        long diff = (long)x - y;
        return diff >= 0 ? diff : -diff;
      };

      // For multiline definitions, indent the entire body.
      Func<int, int, long> difference2 =
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          };

      // Inline lambda arguments should follow these rules. For lambda-containing arguments, add a newline before the parameters.
      CallWithDelegate(
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          });
    }

    void DoNothing() {}                             // Empty blocks can remain compact.

    // When possible, align wrapped parameters with the first parameter.
    void AVeryLongFunctionNameThatCausesLineWrappingProblems(int longArgumentName,
                                                             int p1, int p2) {}

    // If alignment with the first parameter is impractical, wrap all parameters on new lines with a 4-space indent.
    void AnotherLongFunctionNameThatCausesLineWrappingProblems(
        int longArgumentName, int longArgumentName2, int longArgumentName3) {}

    void CallingLongFunctionName() {
      int veryLongArgumentName = 1234;
      int shortArg = 1;
      // Align wrapped parameters with the first parameter if possible.
      AnotherLongFunctionNameThatCausesLineWrappingProblems(shortArg, shortArg,
                                                            veryLongArgumentName);
      // Use a 4-space indent for all parameters if alignment is impractical or unreadable.
      AnotherLongFunctionNameThatCausesLineWrappingProblems(
          veryLongArgumentName, veryLongArgumentName, veryLongArgumentName);
    }
  }
}
```

---

## **C# Coding Standards**

### **Avoid Long Methods**

- Keep methods short and focused. Ideally, a method should perform one task or be responsible for a single module of functionality.
- Prefer single-line LINQ calls and imperative code over heavily chained LINQ expressions. Mixing imperative code with highly chained LINQ can be difficult to read. Avoid using `Container.ForEach(...)` for more than one statement.

### **Use `var` Carefully**

- Variables and fields that can be declared as `const` should always use `const`. If `const` is not applicable, `readonly` is a suitable alternative.
- Use `var` for brevity when the type is obvious.
  - Example:

```csharp
  var customerList = new List<Customer>();  // Clear usage
```

- Avoid using `var` for primitive types or when the type is not immediately apparent to the reader.
  - Example:

```csharp
  var listOfItems = GetList(); // Discouraged
```

### **Leverage Expression-Bodied Members**

- To keep code concise, use **expression-bodied members** for simple getters or methods returning values.
  - Example:

```csharp
  public string FullName => $"{FirstName} {LastName}";
```

---

## **Code Comments and Documentation**

While writing self-explanatory code is crucial, **comments** are still necessary in some cases to explain complex logic or design decisions.

### **XML Documentation Comments**

- Use **XML documentation comments** to provide detailed descriptions for public members.
  - Example:

```csharp
  /// <summary>
  /// Retrieves customer details by their unique ID.
  /// </summary>
  public Customer GetCustomerById(int customerId)
  {
      // Implementation here
  }
```

### **Inline Comments**

- Avoid inline comments that simply repeat the code. Instead, focus on explaining **why** something is done, rather than **what** is done.

---

## **Enforcing Coding Standards**

To maintain consistency in your project, leverage tools that automatically check and enforce coding standards:

- **EditorConfig**: A file to enforce style rules across different IDEs, including Visual Studio.
- **StyleCop**: Analyzes code for style violations and reports any issues.
- **Roslyn Analyzers**: Help enforce best practices in C# codebases by analyzing code during builds.

---

## **Conclusion**

Following consistent coding standards in C# improves clarity and enables a codebase that is easier to maintain and collaborate on. Whether you’re working in a small team or contributing to a large open-source project, adhering to these practices ensures your code is accessible to all stakeholders.

For more detailed information, refer to Microsoft’s [official coding conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) and Google’s [style guide](https://google.github.io/styleguide/csharp-style.html#c-coding-guidelines).
