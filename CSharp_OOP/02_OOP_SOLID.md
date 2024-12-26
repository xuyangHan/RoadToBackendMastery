# Step-by-Step Guide to Writing Clean Code: How SOLID Principles Enhance Your C# Object-Oriented Design

In software development, creating maintainable, scalable, and flexible applications is a critical goal. As projects grow in size and complexity, adhering to robust architectural principles can help developers manage code efficiently and reduce technical debt. The **SOLID principles** are designed to address these challenges.

**SOLID** is an acronym for five fundamental design principles of object-oriented programming (OOP): **Single Responsibility Principle (SRP)**, **Open/Closed Principle (OCP)**, **Liskov Substitution Principle (LSP)**, **Interface Segregation Principle (ISP)**, and **Dependency Inversion Principle (DIP)**. These principles, introduced by Robert C. Martin (“Uncle Bob”), aim to guide developers in writing cleaner, modular, and reusable code.

This article is the second in our OOP series and delves deep into each SOLID principle, demonstrating their application with practical C# examples. If you already understand the basics of OOP in C#, this guide will help you further enhance your code design skills. If you'd like to review the fundamentals of OOP in C#, consider starting with [Mastering the Core Concepts of C# OOP: The Key to Understanding Design Patterns](CSharp_OOP/01_OOP_Concepts_EN.md) before proceeding.

Whether you're an experienced developer or new to software design, understanding these principles will empower you to write more robust, maintainable code.

---

## 1. Single Responsibility Principle (SRP)

- **Definition**: A class should have only one reason to change, meaning it should have only one responsibility.
- **Example**: Consider an `Employee` class that manages both employee details and payroll processing. This violates SRP because it has two reasons to change: employee information and payroll logic.

**Non-compliant Example:**

```csharp
public class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }

    // Violates SRP: Handles payroll processing
    public void ProcessPayroll()
    {
        // Payroll logic
    }
}
```

**Compliant Example (Applying SRP):**

```csharp
public class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }
}

public class PayrollProcessor
{
    public void ProcessPayroll(Employee employee)
    {
        // Payroll logic
    }
}
```

In this example, the `Employee` class now only handles employee details, while the `PayrollProcessor` handles payroll processing, adhering to the SRP principle.

---

## 2. Open/Closed Principle (OCP)

- **Definition**: Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification.
- **Example**: When adding new functionality, extend the class rather than modifying the existing one.

**Non-compliant Example:**

```csharp
public class Rectangle
{
    public int Width { get; set; }
    public int Height { get; set; }

    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class AreaCalculator
{
    public int CalculateArea(Rectangle rectangle)
    {
        return rectangle.CalculateArea();
    }

    // Adding support for new shapes would require modifying AreaCalculator
}
```

**Compliant Example (Applying OCP):**

```csharp
public interface IShape
{
    int CalculateArea();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class Circle : IShape
{
    public int Radius { get; set; }

    public int CalculateArea()
    {
        return (int)(Math.PI * Radius * Radius);
    }
}

public class AreaCalculator
{
    public int CalculateArea(IShape shape)
    {
        return shape.CalculateArea(); // Unchanged
    }
}
```

Now, the `AreaCalculator` class does not need to be modified when new shapes are added, complying with the Open/Closed Principle.

---

## 3. Liskov Substitution Principle (LSP)

- **Definition**: Subtypes must be substitutable for their base types without altering the program’s correctness.
- **Example**: A `Square` should be able to replace a `Rectangle` without breaking functionality. However, if a square alters rectangle behavior (e.g., independently adjusting width and height), it violates LSP.

**Non-compliant Example:**

```csharp
public class Rectangle
{
    public int Width { get; set; }
    public int Height { get; set; }

    public int GetArea()
    {
        return Width * Height;
    }
}

public class Square : Rectangle
{
    // Overrides Width to enforce equal dimensions
    public new int Width
    {
        set
        {
            base.Width = value;
            base.Height = value; // Forces height to match width
        }
    }

    // Overrides Height to enforce equal dimensions
    public new int Height
    {
        set
        {
            base.Height = value;
            base.Width = value; // Forces width to match height
        }
    }
}
```

Using `Square` as a `Rectangle` breaks expected behavior:

```csharp
Rectangle rect = new Square();
rect.Width = 4;  // Set width
rect.Height = 5; // Set height
Console.WriteLine(rect.GetArea()); // Expected 4 * 5 = 20, but returns 5 * 5 = 25
```

Since `Square` enforces equal width and height, it violates the expected independence of rectangle dimensions.

**Compliant Example:**

To comply with LSP, avoid direct inheritance and use composition to differentiate `Square` and `Rectangle` behavior:

```csharp
public interface IShape
{
    int GetArea();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public int GetArea()
    {
        return Width * Height;
    }
}

public class Square : IShape
{
    public int Side { get; set; }

    public int GetArea()
    {
        return Side * Side;
    }
}
```

This approach ensures `Rectangle` and `Square` are independent and comply with LSP, while clearly distinguishing their behaviors.

---

## 4. Interface Segregation Principle (ISP)

- **Definition**: A class should not be forced to implement interfaces it does not use. Instead, there should be multiple small, specific interfaces rather than a single general-purpose one.
- **Example**: An `IWorker` interface with both `Work` and `Eat` methods is problematic, as some workers may not need the `Eat` method.

**Non-compliant Example:**

```csharp
public interface IWorker
{
    void Work();
    void Eat();
}

public class HumanWorker : IWorker
{
    public void Work() { /* Working logic */ }
    public void Eat() { /* Eating logic */ }
}

public class RobotWorker : IWorker
{
    public void Work() { /* Working logic */ }
    public void Eat() { /* Robots don't eat */ }
}
```

**Compliant Example (Applying ISP):**

```csharp
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
}

public class HumanWorker : IWorkable, IFeedable
{
    public void Work() { /* Working logic */ }
    public void Eat() { /* Eating logic */ }
}

public class RobotWorker : IWorkable
{
    public void Work() { /* Working logic */ }
}
```

In this example, the `RobotWorker` class is no longer forced to implement the `Eat` method, complying with the ISP principle.

---

## 5. Dependency Inversion Principle (DIP)

- **Definition**: High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.
- **Example**: Instead of having high-level classes directly depend on low-level classes, abstract the dependency relationship.

The core idea of **DIP** is: **High-level modules (business logic) should not depend on low-level modules (specific implementations); both should depend on abstractions (interfaces or abstract classes).** This promotes flexible code where specific implementations can be swapped without impacting the main logic.

**Non-compliant Example:**

```csharp
public class FileManager
{
    private readonly FileSaver _fileSaver;

    public FileManager()
    {
        _fileSaver = new FileSaver(); // Tight coupling
    }

    public void SaveFile(string data)
    {
        _fileSaver.Save(data);
    }
}

public class FileSaver
{
    public void Save(string data) { /* Save logic */ }
}
```

- **FileManager** is a high-level module responsible for file management logic.
- **FileSaver** is a low-level module implementing specific file-saving functionality.

The issue is the tight coupling between **FileManager** and **FileSaver**:

1. **FileManager** must directly depend on the concrete `FileSaver` class, making it hard to switch to other storage solutions (e.g., database, cloud storage).
2. Replacing `FileSaver` (e.g., with `DatabaseSaver`) requires modifying the `FileManager` code:

```csharp
   public FileManager()
   {
       _fileSaver = new DatabaseSaver(); // Change the dependency
   }
```

This design reduces flexibility and violates the Open/Closed Principle (OCP).

**Compliant Example:**

To comply with DIP, introduce an abstraction (interface or abstract class) so that the **high-level module** depends only on the abstraction, not the specific implementation.

```csharp
public interface IStorage
{
    void Save(string data); // Abstract storage functionality
}

public class FileManager
{
    private readonly IStorage _storage;

    public FileManager(IStorage storage)
    {
        _storage = storage; // Dependency injection: pass an object implementing IStorage
    }

    public void SaveFile(string data)
    {
        _storage.Save(data); // Call abstract method, unconcerned with implementation
    }
}

public class FileSaver : IStorage
{
    public void Save(string data) { /* File saving logic */ }
}

public class DatabaseSaver : IStorage
{
    public void Save(string data) { /* Database saving logic */ }
}
```

**Key Points:**

1. **Introducing the Interface `IStorage`**:
   - Defines an abstraction for "storage capability," regardless of whether it's file storage, database storage, or another solution.
2. **FileManager Depends Only on `IStorage`**:
   - `FileManager` focuses only on its business logic and relies on an abstract `IStorage` interface, not specific implementations like `FileSaver` or `DatabaseSaver`.
3. **Dependency Injection**:
   - The specific implementation of `IStorage` is injected into `FileManager` via its constructor. This pattern, called **dependency injection**, allows external control over which implementation is used.

**Benefits of DIP:**

1. **Flexibility**:
   - Easily switch between implementations (e.g., from file storage to database storage) without modifying `FileManager`.
2. **Extensibility**:
   - To add a new storage solution (e.g., cloud storage), implement `IStorage` and inject it into `FileManager`—no need to change existing code.
3. **Single Responsibility and Independence**:
   - High-level modules handle business logic, and low-level modules focus on implementation details. Both are decoupled via abstractions.
4. **Testability**:
   - Mock implementations of `IStorage` can be injected into `FileManager` for unit testing, enabling isolated testing of the high-level module.

---

## 6. Law of Demeter (LoD)

- **Definition**: A module should not know the inner workings of other modules. It should interact only with its direct collaborators.
- **Example**: Avoid chained calls or accessing nested objects through multiple layers.

The **Law of Demeter** emphasizes **the principle of least knowledge**: **an object should only communicate with its immediate collaborators and should not expose its internal structure.** In other words, interactions between classes should occur through explicit interfaces rather than direct access to their internal details.

**Non-compliant Example:**

```csharp
public class Person
{
    public Address Address { get; set; }
}

public class Address
{
    public string City { get; set; }
}

public class Program
{
    public void PrintCity(Person person)
    {
        Console.WriteLine(person.Address.City); // Violates the Law of Demeter
    }
}
```

1. **High Coupling**:
   - The `Program` class depends not only on `Person` but also indirectly on `Address`. If the `Address` structure changes (e.g., renaming `City` to `Location`), the `Program` code must be updated, increasing maintenance costs.
2. **Breaking Encapsulation**:
   - The `Program` class accesses the internal structure of `Person` (`Address`), exposing implementation details to external classes.
3. **Fragile Code**:
   - If the `Address` property is `null`, a `NullReferenceException` is thrown because `Program` directly accesses `person.Address.City`.

**Compliant Example (Applying LoD):**

```csharp
public class Person
{
    public string GetCity() => Address.City;
    public Address Address { get; set; }
}

public class Address
{
    public string City { get; set; }
}

public class Program
{
    public void PrintCity(Person person)
    {
        Console.WriteLine(person.GetCity()); // Compliant with the Law of Demeter
    }
}
```

Here, `Program` no longer directly accesses `Address`. Instead, it relies on `Person`'s method, complying with the Law of Demeter.

1. **Reduced Coupling**:
   - `Program` now depends only on `Person`, not on `Address`. Changes in `Address` structure are localized to `Person`.
2. **Improved Encapsulation**:
   - `Person` hides the details of `Address`, exposing only the necessary information (`GetCity()` method).
3. **Robust Code**:
   - Null checks can be handled inside the `GetCity()` method:

    ```csharp
    public string GetCity() => Address?.City ?? "Unknown City";
    ```

    This prevents `Program` from handling null references directly, improving reliability.

4. **Easier Testing**:
   - To test `Program`, only `Person` needs to be mocked. Testing is simpler and more focused.

---

### **Summary**

Mastering the **SOLID Principles** is essential for building software that stands the test of time. Applying these fundamental design principles enables you to create maintainable, flexible, and scalable systems. Here's a quick recap of what we covered:

- **SRP** ensures classes have a single responsibility, improving clarity and focus.
- **OCP** encourages extending functionality without modifying existing code, reducing the risk of introducing bugs.
- **LSP** emphasizes substitutability, ensuring derived classes seamlessly replace their base classes.
- **ISP** promotes small, specific interfaces, minimizing unnecessary dependencies.
- **DIP** decouples high-level and low-level modules, enhancing flexibility and testability.

By integrating these principles into your development process, you can write cleaner, more modular code that is easier to maintain, extend, and collaborate on. Remember, good software design is not just about writing code that works—it’s about writing code that is easy to understand and maintain.

Start applying the SOLID principles to your projects today and enjoy the benefits of cleaner, more organized code! 🚀
