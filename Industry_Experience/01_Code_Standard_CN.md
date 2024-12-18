## 编写干净且一致的 C# 代码：.NET 代码规范指南

作为在 .NET 生态系统中工作的 C# 开发者，保持一致的代码风格对于开发可读、可维护和可扩展的应用程序至关重要。遵循既定的 **代码规范** 可以提高代码的清晰度，减少错误，并使跨团队协作变得更加轻松。在这篇文章中，我们将探讨关键的 C# 和 .NET 代码规范，借鉴微软的官方指南，重点关注命名规范、格式化和最佳实践。

---

#### **为什么 C# 中的代码规范很重要**

在软件开发的世界里，代码不仅是为机器编写的——它同样是为人类编写的。

所以，一致的代码规范应该：

- **增强可读性**：遵循规范的代码更容易阅读和理解，尤其是在协作环境中。
- **减少错误**：一致的格式和命名实践可以帮助早期识别错误。
- **改善协作**：随着团队在多个项目中工作，共享标准确保每个人都在同一页面上。
- **促进维护**：格式正确的代码在调试和维护时更容易处理。

---

#### **C# 中的命名规范**

命名规范对使代码自解释至关重要。以下是为 C# 中的变量、方法、类和其他元素命名的一些最佳实践。

##### **PascalCase vs camelCase**

- **PascalCase**：用于公共成员、类名、方法名、公共属性、枚举（Enum）和文件名。名称中的每个单词首字母均为大写。
  - 示例：`CustomerService`、`GetCustomerData()`。
- **camelCase**：用于局部变量和参数。第一个单词以小写字母开头。对于私有、受保护、内部和受保护的内部字段和属性，在开头添加一个下划线。
  - 示例：`customerCount`、`isLoggedIn`、`_customerCount`。

##### **描述性命名**

- 确保名称 **具有描述性**，清楚地传达变量或方法的目的。例如，避免使用 `temp` 或 `var1`，而选择像 `customerList` 或 `totalRevenue` 这样的名称。

##### **命名类和接口**

- 类名应始终为 **名词** 或 **名词短语**（例如，`FileManager`、`DataProcessor`）。
- 接口名称应描述行为，并以大写字母 `I` 开头，后接名词或短语（例如，`IRepository`、`ILogger`）。

有关命名的详细指南，可以参考微软的官方 [命名规范](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines) 。

---

#### **代码格式化规范**

保持一致的代码布局和格式不仅有助于可读性，还有助于调试。以下是一些关键的格式化规则：

##### **缩进**

- 使用 **两个空格** 表示每个缩进级别——避免使用制表符（Tab），以保持跨环境的一致性。
  - 示例：
  ```csharp
  public void CalculateTotal()
  {
      if (isProcessed)
      {
          total += price;
      }
  }
  ```

##### **大括号**

- 在 `if`、`else`、`for`、`while` 和其他语句中始终使用大括号 `{}`，即使块中只有一条语句。这可以防止后续添加额外行时可能出现的错误。
  - 示例：
  ```csharp
  if (isValid)
  {
      ProcessData();
  }
  ```

##### **方法声明格式**

- 保持方法签名清晰且对齐：
  - 返回类型、方法名称和开括号应在长签名的单独行上。
  - 示例：
  ```csharp
  public string GetCustomerNameById(int customerId)
  {
      // 方法主体
  }
  ```

##### **完整示例：**

```csharp
using System;                                       // `using` 应位于顶部，命名空间之外。

namespace MyNamespace {                             // 命名空间使用 PascalCase。
                                                    // 在命名空间后进行缩进。
  public interface IMyInterface {                   // 接口名称以 'I' 开头
    public int Calculate(float value, float exp);   // 方法使用 PascalCase，逗号后有空格。
  }

  public enum MyEnum {                              // 枚举使用 PascalCase。
    Yes,                                            // 枚举常量使用 PascalCase。
    No,
  }

  public class MyClass {                            // 类使用 PascalCase。
    public int Foo = 0;                             // 公共成员变量使用 PascalCase。
    public bool NoCounting = false;                 // 鼓励使用字段初始化器。
    private class Results {
      public int NumNegativeResults = 0;
      public int NumPositiveResults = 0;
    }
    private Results _results;                       // 私有成员变量使用 _camelCase。
    public static int NumTimesCalled = 0;
    private const int _bar = 100;                   // const 不影响命名规范。
    private int[] _someTable = {                    // 容器初始化器使用 2
      2, 3, 4,                                      // 空格缩进。
    }

    public MyClass() {
      _results = new Results {
        NumNegativeResults = 1,                     // 对象初始化器使用 2 空格
        NumPositiveResults = 1,                     // 缩进。
      };
    }

    public int CalculateValue(int mulNumber) {      // 开括号前没有换行。
      var resultValue = Foo * mulNumber;            // 局部变量使用 camelCase。
      NumTimesCalled++;
      Foo += _bar;

      if (!NoCounting) {                            // 一元运算符后无空格，'if' 后有空格。
        if (resultValue < 0) {                      // 即使可选也使用大括号，比较运算符两侧有空格。
          _results.NumNegativeResults++;
        } else if (resultValue > 0) {               // 大括号与 else 之间没有换行。
          _results.NumPositiveResults++;
        }
      }

      return resultValue;
    }

    public void ExpressionBodies() {
      // 对于简单的 lambda，尽量放在一行，如果可能，不需要括号或大括号。
      Func<int, int> increment = x => x + 1;

      // 结束大括号与包含开括号的行的第一个字符对齐。
      Func<int, int, long> difference1 = (x, y) => {
        long diff = (long)x - y;
        return diff >= 0 ? diff : -diff;
      };

      // 如果在续行中定义，整个主体应缩进。
      Func<int, int, long> difference2 =
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          };

      // 内联 lambda 参数也遵循这些规则。如果包含 lambda，建议在参数组之前添加换行。
      CallWithDelegate(
          (x, y) => {
            long diff = (long)x - y;
            return diff >= 0 ? diff : -diff;
          });
    }

    void DoNothing() {}                             // 空块可以简洁。

    // 如果可能，按照第一个参数对齐换行包装参数。
    void AVeryLongFunctionNameThatCausesLineWrappingProblems(int longArgumentName,
                                                             int p1, int p2) {}

    // 如果对齐参数行与第一个参数不匹配，或难以阅读，使用 4 空格缩进在新行中包装所有参数。
    void AnotherLongFunctionNameThatCausesLineWrappingProblems(
        int longArgumentName, int longArgumentName2, int longArgumentName3) {}

    void CallingLongFunctionName() {
      int veryLongArgumentName = 1234;
      int shortArg = 1;
      // 如果可能，按照第一个参数对齐换行包装参数。
      AnotherLongFunctionNameThatCausesLineWrappingProblems(shortArg, shortArg,
                                                            veryLongArgumentName);
      // 如果对齐参数行与第一个参数不匹配，或难以阅读，使用 4 空格缩进在新行中包装所有参数。
      AnotherLongFunctionNameThatCausesLineWrappingProblems(
          veryLongArgumentName, veryLongArgumentName, veryLongArgumentName);
    }
  }
}
```

---

#### **C# 编码规范**

##### **避免长方法**

- 保持方法简短且专注。理想情况下，一个方法应执行一个任务或负责一个功能模块。
- 通常，优先选择单行 LINQ 调用和命令式代码，而不是长链的 LINQ。将命令式代码与高度链式的 LINQ 混合往往难以阅读。避免使用 `Container.ForEach(...)` 处理超过一条语句的内容。

##### **谨慎使用 `var`**

- 可以声明为 `const` 的变量和字段应始终使用 `const`。如果不能使用 `const`，则 `readonly` 是一个合适的替代方案。
- 当类型显而易见时，使用 `var` 有时可以使代码更简洁。
  - 示例：
  ```csharp
  var customerList = new List<Customer>();  // 清晰的用法
  ```
- 避免对基本类型使用 `var`，或者当用户显然需要知道类型时。
  - 示例：
  ```csharp
  var listOfItems = GetList(); // 不鼓励使用
  ```

##### **使用表达式主体成员**

- 为了保持代码简洁，利用 **表达式主体成员** 来简化仅有 getter 的属性或返回值的方法。
  - 示例：
  ```csharp
  public string FullName => $"{FirstName} {LastName}";
  ```

---

#### **代码注释和文档**

虽然编写自解释的代码至关重要，但在某些情况下，**注释**仍然是必要的，以解释复杂的逻辑或设计选择。

##### **XML 文档注释**

- 对于公共成员，使用 **XML 文档注释** 提供 API 的详细描述。
  - 示例：
  ```csharp
  /// <summary>
  /// 根据唯一 ID 检索客户详细信息。
  /// </summary>
  public Customer GetCustomerById(int customerId)
  {
      // 此处代码
  }
  ```

##### **行内注释**

- 避免简单重复代码的行内注释。相反，应关注解释 **为什么** 要这样做，而不是 **做了什么**。

---

#### **强制代码规范的工具**

为了在项目中保持一致性，可以使用自动检查和强制执行代码规范的工具：

- **EditorConfig**：一个在不同 IDE（包括 Visual Studio）中强制实施样式规则的文件。
- **StyleCop**：分析代码以检查代码样式违规，并报告任何不合规的问题。
- **Roslyn 分析器**：通过在构建过程中分析代码来帮助在 C# 代码库中强制执行最佳实践。

---

#### **结论**

通过遵循 C# 和 .NET 中一致的代码规范，您不仅能编写更清晰的代码，还能为一个更易于维护和协作的代码库做出贡献。无论您是在小团队中工作还是为大型开源项目做贡献，遵循这些实践可确保您的代码对所有相关人员更加可访问。

有关更深入的信息，请查看微软的 [官方代码规范文档](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) 和谷歌的 [样式指南](https://google.github.io/styleguide/csharp-style.html#c-coding-guidelines)。
