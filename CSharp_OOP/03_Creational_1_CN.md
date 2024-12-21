# 掌握 C# 中的创建型(Creational) 模式：单例模式(Singleton)与工厂方法(Factory Method)

在开发复杂的 C# 应用程序时，管理对象的创建可能会变得棘手。创建型设计模式提供了结构化的解决方案，通过控制和管理对象的创建过程，使代码更具灵活性、可维护性和可扩展性。在这个两部分的系列文章中，我们将深入探讨创建型模式，首先从两个基础模式开始：**单例模式(Singleton)** 和 **工厂方法(Factory Method)**。

在深入了解创建型设计模式之前，建议您先掌握 C# 中一些关键的面向对象编程 (OOP) 概念。在我们之前的文章中，我们讨论了接口(interface)、抽象类(abstract class)、继承(inheritance)和多态(polymorphism)等基础知识，这些对于理解本篇文章中的示例至关重要。如果您在理解本文中的代码或概念时遇到困难，强烈建议您回顾之前的基础指南。

---

## **对象创建的常见问题**

在大型系统中，直接使用构造函数创建对象可能会导致以下问题：

- **紧耦合**：代码依赖于具体的类，降低了灵活性。
- **缺乏灵活性**：更改对象的创建方式需要修改大量代码。
- **代码重复**：创建类似对象的代码可能会被多次重复，导致重复代码的出现。

创建型模式通过将对象创建逻辑与应用程序的核心逻辑分离，解决了这些问题，从而提升了代码的灵活性和可重用性。

---

## **创建型模式(Creational Patterns)概述**

创建型模式通过抽象对象创建的过程，解耦客户端代码与实际类实例化的逻辑。以下是一些常见的创建型模式：

- **单例模式(Singleton)**：确保一个类只有一个实例，并提供全局访问点。
- **工厂方法(Factory Method)**：定义一个创建对象的接口，但允许子类决定实例化的具体类型。
- **抽象工厂(Abstract Factory)**：提供一个接口，用于创建一系列相关或依赖的对象。
- **建造者模式(Builder)**：按步骤构建一个复杂的对象。
- **原型模式(Prototype)**：通过复制现有对象（原型）创建新对象。

在本文中，我们将重点介绍**单例模式(Singleton)** 和 **工厂方法(Factory Method)**。

---

## **单例模式(Singleton)**

### **什么是单例模式？**

**单例模式**确保一个类只有一个实例，并提供一个全局访问点来访问该实例。它对于管理共享资源非常有用，比如日志服务、配置设置或数据库连接。

### **它解决了什么问题？**

在许多应用程序中，确保某个类只有一个实例是很重要的。例如，多个数据库连接或日志服务的实例可能会导致不必要的资源消耗或状态不一致。单例模式通过提供一种机制，确保类只有一个实例并且可以全局访问，解决了这些问题。

### **如何使用泛型在 C# 中实现单例模式**

使用泛型 `Singleton<T>` 类可以简化单例模式的实现。通过继承 `Singleton<T>`，任何类都可以轻松成为单例，而不需要为每个类重复编写单例逻辑。

以下是使用泛型实现单例模式的示例：

```csharp
public abstract class Singleton<T> where T : class
{
    private static T _instance = null;
    private static readonly object _lock = new object();

    // 返回类的单一实例，必要时创建它
    public static T Instance
    {
        get
        {
            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = Activator.CreateInstance(typeof(T), true) as T;
                }
                return _instance;
            }
        }
    }
}
```

现在，你可以通过继承 `Singleton<T>` 来轻松创建单例类：

**示例：作为单例的 LoggingService**:

```csharp
public class LoggingService : Singleton<LoggingService>
{
    // 私有构造函数，防止直接实例化
    private LoggingService() { }

    public void Log(string message)
    {
        Console.WriteLine("日志条目: " + message);
    }
}

// 使用
LoggingService.Instance.Log("这是一条日志。");
```

### **何时使用单例模式**

- **资源管理**：当你需要控制对共享资源（如日志、配置或数据库连接）的访问，并确保只有一个实例存在时，可以使用单例模式。
- **全局状态**：用于管理应用程序范围内的设置或服务，在多个实例会导致不一致时尤为适用。
- **缓存**：用于缓存全局可访问的数据，确保数据不被重复实例化。

### **Singleton 常见陷阱**

- **过度使用**：如果不当使用，单例模式可能引入隐藏的依赖性，使代码更难测试。
- **线程安全**：在多线程环境中，如果处理不当，单例模式可能导致竞态条件。

### **泛型单例方法的优势**

- **可重用性**：单例逻辑封装在 `Singleton<T>` 类中，可以在不同类之间复用。
- **减少样板代码**：不需要为每个遵循单例模式的类编写重复的单例逻辑。
- **灵活性**：通过使用泛型，可以为任何类类型强制实现单例行为，而无需额外的代码。

### **单例模式 vs. 静态类(Static Class)**

虽然**单例模式**和**静态类**都确保对某些资源的单一访问，但它们的用途不同：

- **单例模式**是基于实例的模式，控制对象创建，允许延迟初始化。它可以实现接口、继承其他类，并在对象行为和资源管理方面提供更多灵活性。这使得单例模式非常适合管理共享资源，如数据库连接或日志。
  
- **静态类**则是类范围的结构，不能被实例化。它的所有成员都是静态的，且全局可访问。静态类主要用于工具函数或辅助方法，但缺乏对实例化的控制和灵活性，无法像单例那样管理对象生命周期。

简而言之，当你需要更多控制单一实例的生命周期和行为时，选择单例模式；如果只需要简单、可重用的方法，而不需要管理状态，则选择静态类。

---

## **工厂方法(Factory Method)模式**

### **什么是工厂方法模式？**

**工厂方法模式**提供了一种创建对象的方法，而不需要直接指定具体的类。相反，由子类决定实例化哪个类。这样可以根据运行时的条件灵活地创建对象。

### **解决的问题**

当一个系统需要创建对象，但在事先不知道具体要创建哪个类的情况下，使用构造函数直接实例化会将客户端代码与特定实现紧密耦合。工厂方法通过抽象对象创建的过程，使得不同类型的对象能够灵活创建。

### **如何在 C# 中实现工厂方法模式**

下面是工厂方法模式的实现：

```csharp
// 产品接口
public interface IAnimal
{
    string Speak();
}

// 具体产品
public class Dog : IAnimal
{
    public string Speak() => "Woof!";
}

public class Cat : IAnimal
{
    public string Speak() => "Meow!";
}

// 带有工厂方法的创建者类
public abstract class AnimalFactory
{
    public abstract IAnimal CreateAnimal();
}

// 具体创建者类
public class DogFactory : AnimalFactory
{
    public override IAnimal CreateAnimal() => new Dog();
}

public class CatFactory : AnimalFactory
{
    public override IAnimal CreateAnimal() => new Cat();
}
```

核心在于 **`AnimalFactory`** 类定义的 **工厂方法**（`CreateAnimal()`），该方法返回的是接口（**`IAnimal`**）而不是具体类，这为创建不同类型的动物（如 `Dog`、`Cat`）提供了灵活性。工厂模式在业务逻辑层（BLL）中非常有用，因为它可以在不指定具体类的情况下创建对象，符合开闭原则（Open-Closed Principle）。

### 示例：工厂方法如何帮助业务逻辑层（BLL）

假设我们有一个管理宠物的 **PetService** 类。如果不使用工厂模式，代码可能需要了解所有可能的动物类型（如 Dog、Cat 等），导致代码紧密耦合。而通过使用工厂模式，我们可以将宠物创建的逻辑与业务逻辑解耦。

#### 示例：没有使用工厂方法

```csharp
public class PetService
{
    public string GetPetSound(string petType)
    {
        if (petType == "Dog")
        {
            var dog = new Dog();
            return dog.Speak();
        }
        else if (petType == "Cat")
        {
            var cat = new Cat();
            return cat.Speak();
        }

        throw new Exception("Unknown pet type.");
    }
}
```

这种方法使得 **PetService** 类与具体的宠物类紧密耦合，如果需要扩展到新的宠物类型，代码会变得难以维护。

#### 示例：使用工厂方法

使用工厂模式，可以避免 **PetService** 知道具体的类：

```csharp
public class PetService
{
    private readonly AnimalFactory _animalFactory;

    public PetService(AnimalFactory animalFactory)
    {
        _animalFactory = animalFactory;
    }

    public string GetPetSound()
    {
        var animal = _animalFactory.CreateAnimal();
        return animal.Speak();
    }
}
```

现在，你可以通过不同的 `AnimalFactory` 实现类来实例化 `PetService`：

```csharp
var dogService = new PetService(new DogFactory());
Console.WriteLine(dogService.GetPetSound());  // 输出: Woof!

var catService = new PetService(new CatFactory());
Console.WriteLine(catService.GetPetSound());  // 输出: Meow!
```

#### 在业务逻辑层（BLL）中的好处

1. **可扩展性**：添加一种新的动物类型（例如 `Bird`）只需创建一个新的工厂（`BirdFactory`）和具体产品（`Bird`），而无需修改 **PetService** 的代码。
2. **关注点分离**：**PetService** 专注于业务逻辑（获取宠物的声音），而不必担心对象创建的细节。
3. **可测试性**：在单元测试中更容易模拟 **AnimalFactory**，从而更好地控制在测试期间创建的对象类型。

这种使用工厂方法模式的方式为业务逻辑层提供了灵活、可维护和可扩展的解决方案。

### **何时使用工厂方法**

- **将对象创建与业务逻辑解耦**：当系统需要创建对象，但希望将对象创建逻辑与应用程序的其余部分解耦时。
- **当经常添加新类型时**：在新对象类型经常添加的系统中，工厂方法非常有用，因为它避免在引入新类时修改现有代码。

### **工厂方法常见陷阱**

- **复杂性**：如果过度使用，工厂方法可能会通过创建过多的子类引入不必要的复杂性。
- **误用**：有时，直接实例化等更简单的方法就足够了，特别是当逻辑不需要灵活性时。

---

## **结论**

在本文中，我们介绍了两个关键的创建型模式：**单例模式(Singleton)** 和 **工厂方法模式(Factory Method)**。单例模式通过确保只有一个实例来帮助管理共享资源，而工厂方法模式则在对象创建中提供灵活性，而不将代码紧密耦合到具体类上。

在下一部分中，我们将深入探讨 **抽象工厂(Abstract Factory)**、**建造者(Builder)** 和 **原型(Prototype)** 模式，进一步研究更高级的创建型模式。敬请关注！
