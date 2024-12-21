# 精通 C# 中的结构型模式(structural patterns)：全面指南

在这篇博客中，我们将探索 **结构型设计模式 (structural design patterns)** 及其在 C# 中的实现。这些模式涉及如何组合对象和类以形成更大的结构，使我们能够构建灵活且高效的系统。在之前讨论了创建型模式之后，我们现在深入研究能够帮助组织代码、管理关系并更有效地维护系统的模式。

---

## **1. 适配器(Adapter)模式**

**目的**：适配器模式将一个接口转换为客户端期望的另一个接口。当整合具有不兼容接口的系统时，它特别有用。

### **何时使用适配器模式**

- 当你需要使用一个不兼容接口的类时。
- 让两个系统无缝协作（例如，将遗留系统与新接口集成）。

### **适配器模式示例**

假设我们有一个遗留支付系统需要与一个新的支付 API 进行工作。我们可以创建一个适配器，将遗留系统的调用转换为新 API，使集成平稳进行，而无需修改遗留代码。

```csharp
// 遗留支付系统
public class OldPaymentSystem
{
    public void ProcessOldPayment() => Console.WriteLine("在遗留系统中处理支付。");
}

// 新支付API接口
public interface IPaymentProcessor
{
    void ProcessPayment();
}

// 旧系统的适配器
public class PaymentAdapter : IPaymentProcessor
{
    private readonly OldPaymentSystem _oldPaymentSystem;

    public PaymentAdapter(OldPaymentSystem oldPaymentSystem)
    {
        _oldPaymentSystem = oldPaymentSystem;
    }

    public void ProcessPayment()
    {
        _oldPaymentSystem.ProcessOldPayment();
    }
}
```

在业务逻辑层（BLL）中，`IPaymentProcessor`接口可以被注入，使适配器适用于现有的业务流程。以下是我们在 BLL 中使用`PaymentAdapter`的示例：

```csharp
public class PaymentService
{
    private readonly IPaymentProcessor _paymentProcessor;

    // 注入 IPaymentProcessor，它可以是 PaymentAdapter
    public PaymentService(IPaymentProcessor paymentProcessor)
    {
        _paymentProcessor = paymentProcessor;
    }

    public void ProcessTransaction()
    {
        Console.WriteLine("在业务逻辑层中启动支付处理...");
        _paymentProcessor.ProcessPayment();  // 适配器或新系统都可以在这里互换使用
        Console.WriteLine("支付处理完成。");
    }
}

// 在 BLL 中使用适配器
class Program
{
    static void Main()
    {
        OldPaymentSystem oldSystem = new OldPaymentSystem();
        IPaymentProcessor paymentProcessor = new PaymentAdapter(oldSystem); // 使用适配器替代新系统
        PaymentService paymentService = new PaymentService(paymentProcessor);
        
        paymentService.ProcessTransaction();
    }
}
```

### **适配器模式为什么很有用**

- **无缝集成**：允许旧系统与新的业务逻辑协作，而无需修改原有代码。
- **代码灵活性**：将来，`PaymentService`可以轻松将`PaymentAdapter`替换为实现了`IPaymentProcessor`的新支付处理器。

---

## **2. 桥接(Bridge)模式**

**目的**：桥接模式将抽象与其实现分离，使两者可以独立变化。当对象和其功能都需要扩展且彼此互不影响时，此模式尤为有用。

### **何时使用桥接模式**

- 当你希望将抽象与实现解耦时。
- 为了避免因多种变体产生庞大的类层次结构。

### **桥接模式示例**

假设你有一个几何图形类 `Shape`，它有两个子类：`Circle`（圆形）和 `Square`（正方形）。你希望在类层次结构中加入颜色属性，如红色和蓝色。此时，如果继续使用继承方法，就需要创建四个类的组合，例如 `BlueCircle`（蓝色圆形）和 `RedSquare`（红色正方形）。

使用**桥接模式**，可以在不影响彼此的情况下，独立扩展图形（例如 `Circle` 和 `Square`）或颜色（例如 `Red` 和 `Blue`）。

```csharp
// 颜色接口
public interface IColor
{
    string ApplyColor();  // 定义如何应用颜色的接口
}

// IColor 的具体实现
public class Red : IColor
{
    public string ApplyColor() => "应用红色。";
}

public class Blue : IColor
{
    public string ApplyColor() => "应用蓝色。";
}

// 抽象图形类（抽象层）
public abstract class Shape
{
    protected IColor color;  // 通过持有 IColor 的引用建立桥接

    public Shape(IColor color)
    {
        this.color = color;  // 通过持有 IColor 引用建立桥接
    }

    public abstract void Draw();  // 抽象方法，用于绘制图形
}

// 具体图形类（细化抽象层）
public class Circle : Shape
{
    public Circle(IColor color) : base(color) { }

    public override void Draw() => Console.WriteLine($"绘制圆形。{color.ApplyColor()}");  // 使用颜色实现绘制
}

public class Square : Shape
{
    public Square(IColor color) : base(color) { }

    public override void Draw() => Console.WriteLine($"绘制正方形。{color.ApplyColor()}");  // 使用颜色实现绘制
}
```

在业务逻辑层（BLL）中，抽象类 `Shape` 可以被注入，并与各种 `IColor` 实现一起使用。以下是如何在 BLL 中使用桥接模式的示例：

```csharp
public class DrawingService
{
    private readonly Shape _shape;

    // 注入带有颜色的 Shape
    public DrawingService(Shape shape)
    {
        _shape = shape;
    }

    public void RenderShape()
    {
        Console.WriteLine("在业务逻辑层渲染形状...");
        _shape.Draw();  // Shape 类使用桥接来应用颜色并绘制自身
    }
}

// 在 BLL 中使用
class Program
{
    static void Main()
    {
        IColor red = new Red();
        Shape circle = new Circle(red);  // 桥接：红色圆形
        DrawingService drawingService = new DrawingService(circle);
        drawingService.RenderShape();  // 输出：绘制圆形。应用红色。

        // 同时改变图形和颜色
        IColor blue = new Blue();
        Shape square = new Square(blue);  // 桥接：蓝色正方形
        drawingService = new DrawingService(square);
        drawingService.RenderShape();  // 输出：绘制正方形。应用蓝色。
    }
}
```

### **桥接模式为什么很有用**

- **灵活扩展**：可以在不修改核心代码的情况下，独立扩展图形和颜色。
- **解耦**：在业务逻辑层（BLL）中的 `DrawingService` 并不知道具体的图形或颜色实现，促进了解耦和可扩展性。`Shape` 类不需要了解 `IColor` 的具体实现细节（如 `Red` 或 `Blue`）。桥接通过在 `Shape` 中持有对 `IColor` 的引用，使得图形的抽象层与具体的颜色实现解耦。

---

## **3. 组合(Composite)模式**

**目的**：组合模式允许你以一致的方式处理单个对象和对象组。当处理类似文件系统、组织结构或任务处理系统等层次结构时，该模式特别有用，因为它可以统一处理单个任务和任务组。

### **何时使用组合模式**

- 当你的系统涉及整体部分层次结构时，例如任务和子任务，这样可以统一处理单个元素和组合元素。
- 为了通过使用相同的接口简化复杂结构的处理，不论是简单对象还是组合对象。
- 通过一致地将行为委托给组件，减少条件逻辑的使用。

### **组合模式示例：业务逻辑层中的处理与子处理系统**

在这个例子中，我们有一个 **Processing** 类用于记录操作日志。每个 **Processing** 对象可以包含多个子处理，它们都通过 **CompositeProcessing** 类以统一的方式处理。这有助于使用相同的接口管理单个任务和组合任务，从而清理业务逻辑层中的复杂逻辑。

```csharp
// 组件接口
public interface IProcessing
{
    void Log();  // 日志记录接口
}

// 叶子：单个处理类
public class Processing : IProcessing
{
    private string _name;

    public Processing(string name)
    {
        _name = name;
    }

    public void Log()
    {
        Console.WriteLine($"记录处理日志：{_name}");
    }
}

// 组合类：具有子处理的组合处理
public class CompositeProcessing : IProcessing
{
    private List<IProcessing> _subProcessings = new List<IProcessing>();
    private string _compositeProcessingName;

    public CompositeProcessing(string compositeProcessingName)
    {
        _compositeProcessingName = compositeProcessingName;
    }

    public void AddSubProcessing(IProcessing subProcessing)
    {
        _subProcessings.Add(subProcessing);
    }

    public void RemoveSubProcessing(IProcessing subProcessing)
    {
        _subProcessings.Remove(subProcessing);
    }

    public void Log()
    {
        Console.WriteLine($"记录组合处理日志：{_compositeProcessingName}");
        foreach (var subProcessing in _subProcessings)
        {
            subProcessing.Log();  // 统一调用子处理的日志记录方法
        }
    }
}
```

### **组合模式在业务逻辑层中的示例**

以下是如何在业务逻辑层（BLL）中使用 **组合(Composite)模式** 来简化日志记录逻辑的示例，其中的处理任务可能具有多个子任务，且二者统一记录日志：

```csharp
class Program
{
    static void Main(string[] args)
    {
        // 单个处理（叶子节点）
        IProcessing process1 = new Processing("处理1");
        IProcessing process2 = new Processing("处理2");

        // 组合处理（组合节点）
        CompositeProcessing mainProcess = new CompositeProcessing("主处理");

        // 将单个处理添加到组合处理中
        mainProcess.AddSubProcessing(process1);
        mainProcess.AddSubProcessing(process2);

        // 子组合处理
        CompositeProcessing subProcessGroup = new CompositeProcessing("子处理组");
        subProcessGroup.AddSubProcessing(new Processing("子处理1"));
        subProcessGroup.AddSubProcessing(new Processing("子处理2"));

        // 将子组合添加到主处理中
        mainProcess.AddSubProcessing(subProcessGroup);

        // 记录所有处理的日志
        mainProcess.Log();
    }
}
```

### **组合模式为什么在业务逻辑层中有用**

- **统一处理**：组合模式允许我们以相同的方式处理单个处理任务和处理组任务。这减少了复杂的条件逻辑（如 `if` 语句判断某对象是单个任务还是组合任务）的需求。
  
- **代码简洁模块化**：通过将单个处理和组合处理抽象到公共接口（`IProcessing`），业务逻辑层能够以统一的方式处理它们，从而使代码更加简洁、模块化且易于维护。

- **可扩展性**：该模式允许轻松扩展。你可以添加更多的处理类型（如新的子处理）而无需更改现有的结构，使其灵活且可扩展，适合较大的系统。

- **减少冗余**：如果不使用组合模式，你需要为处理单个任务和组合任务编写不同的逻辑。使用组合模式后，逻辑统一化，简化了维护和进化。

通过在层次结构（如任务管理系统）中使用组合模式，可以简化业务逻辑层中的代码，使其更易于扩展、测试和维护。

---

## **4. 装饰者(Decorator)模式**

**目的**：装饰者模式允许你动态地为对象添加职责，而不需要修改其结构。这对于添加诸如日志记录、安全性或缓存等功能特别有用。

### **何时使用装饰者模式**

- 当你需要动态地为对象添加功能时。
- 为避免为每个新功能创建子类时，可以使用装饰者模式进行功能扩展。

### **装饰者模式示例**

让我们通过添加加密和日志记录功能来增强一个基础的数据流。

```csharp
// 组件接口
public interface IDataStream
{
    void Write(string data);  // 写入数据接口
}

// 具体组件
public class FileStream : IDataStream
{
    public void Write(string data) => Console.WriteLine($"将数据写入文件: {data}");
}

// 装饰者
public class StreamDecorator : IDataStream
{
    protected IDataStream _stream;

    public StreamDecorator(IDataStream stream) => _stream = stream;

    public virtual void Write(string data) => _stream.Write(data);
}

// 具体装饰者：加密数据流
public class EncryptedStream : StreamDecorator
{
    public EncryptedStream(IDataStream stream) : base(stream) { }

    public override void Write(string data)
    {
        var encryptedData = Encrypt(data);
        base.Write(encryptedData);  // 使用加密后的数据进行写入
    }

    private string Encrypt(string data) => $"加密({data})";
}

// 具体装饰者：日志记录数据流
public class LoggedStream : StreamDecorator
{
    public LoggedStream(IDataStream stream) : base(stream) { }

    public override void Write(string data)
    {
        Log(data);  // 先记录日志
        base.Write(data);  // 再执行写入操作
    }

    private void Log(string data) => Console.WriteLine($"日志记录: {data}");
}
```

### **装饰者模式为什么有用**

- 可以在不修改现有代码的情况下为对象增强新的行为。
- 提供了逐步增加功能的灵活性。

---

## **5. 外观(Facade)模式**

**目的**：外观模式为复杂系统提供了一个简化的接口。它隐藏了子系统的复杂性，使得与整个系统的交互变得更容易。

### **何时使用外观模式**

- 当你想简化与复杂系统的交互时。
- 为一组接口提供统一的API时。

### **外观模式示例**

以下是一个多媒体系统的外观，它控制视频、音频和字幕。

```csharp
// 外观类：MultimediaFacade
public class MultimediaFacade
{
    private readonly VideoPlayer _videoPlayer;
    private readonly SubtitleService _subtitleService;

    // 外观构造函数初始化所有子系统
    public MultimediaFacade()
    {
        _videoPlayer = new VideoPlayer();
        _subtitleService = new SubtitleService();
    }

    // 简化的接口：播放带有字幕的视频
    public void PlayVideoWithSubtitles(string videoFile, string subtitleFile)
    {
        _videoPlayer.PlayVideo(videoFile);  // 播放视频
        _subtitleService.LoadSubtitles(subtitleFile);  // 加载字幕
        Console.WriteLine("正在播放带字幕的视频...");
    }
}
```

现在，我们看看如何在业务逻辑层中使用 **MultimediaFacade** 简化与多媒体子系统的交互。

```csharp
// 业务逻辑层类：MultimediaService
public class MultimediaService
{
    private readonly MultimediaFacade _multimediaFacade;

    // 构造函数初始化 MultimediaFacade
    public MultimediaService()
    {
        _multimediaFacade = new MultimediaFacade();
    }

    // 播放带字幕的电影
    public void PlayMovieWithSubtitles(string videoFile, string subtitleFile)
    {
        Console.WriteLine("业务逻辑层中开始播放带字幕的电影...");
        _multimediaFacade.PlayVideoWithSubtitles(videoFile, subtitleFile);
    }
}

// 应用程序中的使用
class Program
{
    static void Main()
    {
        MultimediaService multimediaService = new MultimediaService();

        // 使用业务逻辑层播放带字幕的电影
        multimediaService.PlayMovieWithSubtitles("movie.mp4", "movie_subtitles.srt");
    }
}
```

### **外观模式为什么有用**

- **统一接口**：`MultimediaFacade` 提供了一个简单的接口，用于与复杂的多媒体子系统（音频、视频和字幕）交互。业务逻辑层（BLL）不需要与每个组件单独交互，只需处理外观即可。
- **关注点分离**：多媒体子系统的复杂性被封装在外观内。业务逻辑层（BLL）只需要处理高级操作（如播放带字幕的视频或播放音频文件）。
- **易于维护**：如果多媒体子系统发生变化（例如，视频或音频有了新功能），这些变化被隔离在外观中，不会直接影响到业务逻辑层（BLL）。

---

## **6. 享元(Flyweight)模式**

**目的**：享元模式用于通过在多个对象之间共享对象状态的公共部分来最小化内存使用。享元模式允许您在多个对象之间共享相同的数据，而只单独存储唯一属性，而不是为具有许多相似属性的对象创建单独的实例。

### **享元模式使用场景**

- 当您的系统涉及大量相似对象时。
- 通过共享在这些对象中相同的数据来优化内存使用。

### **享元模式 - 理解问题**

想象一下，您正在开发一个包含大量树木的游戏。每棵树都有相同的属性，例如其类型（例如，橡树、松树）、颜色和纹理，但其唯一属性（例如，世界中的位置）各不相同。

如果为每棵树创建一个单独的对象，包括其所有属性（名称、颜色、纹理、位置），将会使用大量内存，因为您对每棵树重复存储相同的数据（名称、颜色、纹理）。

享元模式通过在多棵树之间**共享**内在属性（如名称、颜色和纹理），而只单独存储唯一属性（如位置）来解决此问题。

### **享元模式示例：带有树木的游戏**

让我们用一个例子，展示如何在游戏中显示大量树木，每棵树共享相同的结构（名称、颜色和纹理），但具有独特的位置。

#### **逐步示例**

1. **共享状态（内在状态）**：这是所有对象共有的数据，例如树的名称、颜色和纹理。
2. **唯一状态（外在状态）**：这是每个对象变化的数据，例如树在世界中的位置（例如，坐标 `x`、`y`）。

#### **享元类（共享树类型）**

`TreeType` 类表示树的 **共享状态**，例如其名称、颜色和纹理。相同类型的多棵树将重用此对象。

```csharp
// 享元
public class TreeType
{
    private string _name;  // 内在状态（共享）
    private string _color; // 内在状态（共享）
    private string _texture; // 内在状态（共享）

    public TreeType(string name, string color, string texture)
    {
        _name = name;
        _color = color;
        _texture = texture;
    }

    public void Display(int x, int y)
    {
        Console.WriteLine($"在 ({x}, {y}) 显示 {_name} 树");
    }
}
```

#### **享元工厂（管理共享对象）**

此工厂确保每种 `TreeType`（享元）仅创建一个实例。如果已经存在相同名称、颜色和纹理的树，则重用现有的。

```csharp
// 享元工厂
public class TreeFactory
{
    private Dictionary<string, TreeType> _treeTypes = new();

    public TreeType GetTreeType(string name, string color, string texture)
    {
        string key = $"{name}-{color}-{texture}";
        if (!_treeTypes.ContainsKey(key))
        {
            _treeTypes[key] = new TreeType(name, color, texture);
        }
        return _treeTypes[key];
    }
}
```

#### **Context 类（唯一位置数据）**

`Tree` 类表示实际的树对象，但仅存储其 **唯一状态**，例如其位置（`x`、`y`）。共同状态（树类型、颜色、纹理）存储在共享的 `TreeType` 对象中。

```csharp
// Context 类
public class Tree
{
    private int _x; // 外在状态（唯一）
    private int _y; // 外在状态（唯一）
    private TreeType _type; // 内在状态（共享）

    public Tree(int x, int y, TreeType type)
    {
        _x = x;
        _y = y;
        _type = type;
    }

    public void Display()
    {
        _type.Display(_x, _y); // 使用共享 TreeType 和唯一位置
    }
}
```

#### **在 BLL 中的使用**

现在，您可以使用享元模式在游戏中有效地创建和管理大量树木。

```csharp
class Program
{
    static void Main(string[] args)
    {
        TreeFactory treeFactory = new TreeFactory();

        // 创建共享树类型（享元）
        TreeType oakType = treeFactory.GetTreeType("Oak", "Green", "OakTexture");
        TreeType pineType = treeFactory.GetTreeType("Pine", "DarkGreen", "PineTexture");

        // 创建具有唯一位置但共享树类型的树
        Tree tree1 = new Tree(10, 20, oakType);
        Tree tree2 = new Tree(15, 25, oakType); // 重用 oakType
        Tree tree3 = new Tree(5, 30, pineType);
        
        // 显示树木
        tree1.Display();
        tree2.Display();
        tree3.Display();
    }
}
```

### **享元模式的用途**

1. **内存优化**：通过在多个树木之间共享内在状态（树类型、颜色、纹理），我们大大减少了内存使用。系统仅存储每种树类型的一个实例，并在必要时重用它，而每个树对象仅存储其唯一位置数据。
2. **性能**：在处理成千上万或数百万个相似对象时，享元模式可以通过最小化对象创建和内存消耗来显著提高性能。
3. **简化对象管理**：通过将共享状态与唯一状态分离，您可以避免管理大量完全唯一的对象，这简化了对象的创建并提高了可维护性。

---

## **7. 代理(Proxy)模式**

**目的**：  
代理模式为对象提供一个中介或占位符，以控制对该对象的访问。这在以下情况下非常有用：

- 当您不想立即创建对象，因为在资源方面代价高昂（例如，需要大量内存或处理）。
- 当您需要添加额外的功能，例如日志记录、安全检查或延迟初始化，而不修改底层对象本身。
- 当您想通过添加抽象层来管理客户端与实际对象之间的交互。

### **代理模式的类型**

1. **虚拟代理**：延迟创建资源密集型对象，直到需要时（懒加载）。
2. **保护代理**：控制对敏感对象的访问，通常添加安全检查。
3. **远程代理**：管理与位于不同地址空间（例如，不同服务器）的对象的交互。
4. **缓存代理**：添加缓存功能以避免冗余工作。

### **代理模式 - 使用场景**

- **昂贵对象创建**：如果对象消耗资源且并不总是需要，则代理可以推迟其创建，直到需要为止。
- **访问控制**：如果某些用户或系统不应完全访问某些对象，则代理可以强制实施安全规则。
- **日志记录/监控**：当您希望记录对对象的访问或跟踪与对象的交互，而无需修改其核心逻辑时。

### **虚拟代理在业务逻辑层（BLL）中的详细示例**

想象一下，您有一个 `ExpensiveObject`，它的创建需要很长时间或使用大量内存。您不想在绝对需要之前创建这个对象。在这种情况下，您可以使用 **虚拟代理** 来延迟 `ExpensiveObject` 的创建，直到它实际上被使用的时刻。

#### **ExpensiveObject 和 IExpensiveObject 接口**

首先，让我们定义 `ExpensiveObject` 类和它所实现的 `IExpensiveObject` 接口：

```csharp
// 昂贵操作的接口
public interface IExpensiveObject
{
    void Process();
}

// 一个资源密集型的类，初始化需要时间
public class ExpensiveObject : IExpensiveObject
{
    public ExpensiveObject()
    {
        // 模拟昂贵对象的创建过程
        Console.WriteLine("ExpensiveObject: 正在初始化资源密集型对象...");
        System.Threading.Thread.Sleep(2000);  // 模拟延迟
    }

    public void Process()
    {
        Console.WriteLine("ExpensiveObject: 正在处理数据...");
    }
}
```

#### **代理：ExpensiveObjectProxy**

现在，**代理** 将充当中介。它将延迟 `ExpensiveObject` 的创建，直到首次调用 `Process()` 方法：

```csharp
// 控制对昂贵对象访问的代理类
public class ExpensiveObjectProxy : IExpensiveObject
{
    private ExpensiveObject _expensiveObject;

    // 代理推迟对象创建，直到需要
    public void Process()
    {
        if (_expensiveObject == null)
        {
            Console.WriteLine("Proxy: 正在首次创建 ExpensiveObject...");
            _expensiveObject = new ExpensiveObject();
        }
        _expensiveObject.Process();
    }
}
```

在代理类中，真正的 `ExpensiveObject` 仅在首次调用 `Process()` 方法时创建。这节省了资源，如果对象不需要立即使用。

#### **在业务逻辑层（BLL）中使用代理**

现在，让我们展示如何在 BLL 类中使用代理模式来管理 `ExpensiveObject` 的创建。BLL 层不需要知道它是在与真实对象还是代理对象交互——它仅使用 `IExpensiveObject` 接口。

```csharp
// 与代理交互的业务逻辑层类
public class ExpensiveOperationService
{
    private readonly IExpensiveObject _expensiveObjectProxy;

    public ExpensiveOperationService(IExpensiveObject expensiveObjectProxy)
    {
        _expensiveObjectProxy = expensiveObjectProxy;
    }

    // BLL 中处理数据的方法
    public void PerformOperation()
    {
        Console.WriteLine("BLL: 正在请求昂贵操作...");
        _expensiveObjectProxy.Process();  // 代理决定何时创建实际对象
    }
}

// 应用中的使用
class Program
{
    static void Main()
    {
        // 使用代理而不是直接创建 ExpensiveObject
        IExpensiveObject proxy = new ExpensiveObjectProxy();
        ExpensiveOperationService service = new ExpensiveOperationService(proxy);

        Console.WriteLine("应用程序: 正在启动操作...");

        // 第一次请求执行操作
        service.PerformOperation();  // 代理在这里创建真实对象

        Console.WriteLine("应用程序: 正在执行另一个操作...");

        // 第二次请求 - ExpensiveObject 已经创建
        service.PerformOperation();  // 不需要重新创建对象
    }
}
```

#### **流程说明**

1. 当第一次调用 `service.PerformOperation()` 时，代理检查 `ExpensiveObject` 是否已经创建。
2. 如果没有，它在代理内初始化 `ExpensiveObject`（这一步模拟一个昂贵的操作，比如加载大量数据集、连接到远程服务等）。
3. 在后续调用中，已经创建的 `ExpensiveObject` 被重用，而无需重新初始化。

### **为什么使用代理？**

- **懒加载**：昂贵对象仅在真正需要时创建。这可以节省大量资源，特别是当对象可能并不总是被使用时。
- **关注点分离**：BLL 不需要管理对象创建逻辑。它通过 `IExpensiveObject` 接口与之交互，而不关心它是在处理真实对象还是代理。
- **效率**：如果您请求多个操作，对象仅创建一次并被重用，从而提高性能。

**代理模式** 对于控制对对象的访问非常有用，尤其是当您希望延迟其创建时（如虚拟代理的情况）。当处理昂贵操作、安全问题甚至网络调用时，这种模式表现出色。在我们的示例中，代理在需要时延迟创建 `ExpensiveObject`，而业务逻辑层（BLL）通过接口与之交互，无需了解底层复杂性。

---

## **结论**

在本博客中，我们介绍了几种结构设计模式及其在 C# 中的实现。每种模式在高效构造代码中都有独特的目的，知道何时使用它们可以显著改善系统的设计。无论是通过外观模式简化代码，还是通过享元模式优化内存，或通过装饰器增强对象，这些模式都是软件开发者的重要工具。

请继续关注我们下一个博客中的 **行为设计模式**！
