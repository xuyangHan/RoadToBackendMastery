# Mastering Structural Patterns in C#: A Comprehensive Guide

In this blog, we will explore **structural design patterns** and their implementation in C#. These patterns deal with how objects and classes are combined to form larger structures, enabling us to build flexible and efficient systems. After discussing creational patterns earlier, we now delve into patterns that help organize code, manage relationships, and maintain systems more effectively.

---

## **1. Adapter Pattern**

**Purpose**: The Adapter Pattern converts an interface into another interface that a client expects. It is especially useful when integrating systems with incompatible interfaces.

### **When to Use the Adapter Pattern**

- When you need to use a class with an incompatible interface.
- To enable seamless collaboration between two systems (e.g., integrating a legacy system with a new interface).

### **Adapter Pattern Example**

Suppose we have a legacy payment system that needs to work with a new payment API. We can create an adapter to translate calls from the legacy system into the new API, ensuring smooth integration without modifying the legacy code.

```csharp
// Legacy payment system
public class OldPaymentSystem
{
    public void ProcessOldPayment() => Console.WriteLine("Processing payment in the legacy system.");
}

// New payment API interface
public interface IPaymentProcessor
{
    void ProcessPayment();
}

// Adapter for the old system
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

In the business logic layer (BLL), the `IPaymentProcessor` interface can be injected, making the adapter suitable for existing workflows. Here's an example of how to use `PaymentAdapter` in the BLL:

```csharp
public class PaymentService
{
    private readonly IPaymentProcessor _paymentProcessor;

    // Inject IPaymentProcessor, which can be a PaymentAdapter
    public PaymentService(IPaymentProcessor paymentProcessor)
    {
        _paymentProcessor = paymentProcessor;
    }

    public void ProcessTransaction()
    {
        Console.WriteLine("Starting payment processing in the business logic layer...");
        _paymentProcessor.ProcessPayment();  // Adapter or new system can be used interchangeably here
        Console.WriteLine("Payment processing completed.");
    }
}

// Using the adapter in BLL
class Program
{
    static void Main()
    {
        OldPaymentSystem oldSystem = new OldPaymentSystem();
        IPaymentProcessor paymentProcessor = new PaymentAdapter(oldSystem); // Using adapter for the old system
        PaymentService paymentService = new PaymentService(paymentProcessor);
        
        paymentService.ProcessTransaction();
    }
}
```

### **Why the Adapter Pattern is Useful**

- **Seamless Integration**: Enables the old system to work with new business logic without modifying the original code.
- **Code Flexibility**: In the future, `PaymentService` can easily replace `PaymentAdapter` with a new payment processor implementing `IPaymentProcessor`.

---

## **2. Bridge Pattern**

**Purpose**: The Bridge Pattern separates an abstraction from its implementation so that the two can vary independently. This is especially useful when both the object and its functionality need to be extended without impacting each other.

### **When to Use the Bridge Pattern**

- When you want to decouple an abstraction from its implementation.
- To avoid a large class hierarchy caused by multiple variations.

### **Bridge Pattern Example**

Suppose you have a `Shape` class with two subclasses: `Circle` and `Square`. You want to add a `Color` property, such as red and blue. If you continue using inheritance, you will need to create combinations like `BlueCircle` and `RedSquare`.

Using the **Bridge Pattern**, you can independently extend shapes (e.g., `Circle`, `Square`) and colors (e.g., `Red`, `Blue`) without affecting each other.

```csharp
// Color interface
public interface IColor
{
    string ApplyColor();  // Interface to define how to apply color
}

// Concrete implementations of IColor
public class Red : IColor
{
    public string ApplyColor() => "Applying red color.";
}

public class Blue : IColor
{
    public string ApplyColor() => "Applying blue color.";
}

// Abstract Shape class (abstraction layer)
public abstract class Shape
{
    protected IColor color;  // Bridge by holding a reference to IColor

    public Shape(IColor color)
    {
        this.color = color;
    }

    public abstract void Draw();  // Abstract method to draw the shape
}

// Concrete Shape classes (refined abstraction layer)
public class Circle : Shape
{
    public Circle(IColor color) : base(color) { }

    public override void Draw() => Console.WriteLine($"Drawing a circle. {color.ApplyColor()}");
}

public class Square : Shape
{
    public Square(IColor color) : base(color) { }

    public override void Draw() => Console.WriteLine($"Drawing a square. {color.ApplyColor()}");
}
```

In the business logic layer (BLL), the abstract `Shape` class can be injected and used with various `IColor` implementations. Here's an example:

```csharp
public class DrawingService
{
    private readonly Shape _shape;

    // Inject a Shape with color
    public DrawingService(Shape shape)
    {
        _shape = shape;
    }

    public void RenderShape()
    {
        Console.WriteLine("Rendering shape in the business logic layer...");
        _shape.Draw();  // Shape uses the bridge to apply color and draw itself
    }
}

// Using the Bridge Pattern in BLL
class Program
{
    static void Main()
    {
        IColor red = new Red();
        Shape circle = new Circle(red);  // Bridge: Red circle
        DrawingService drawingService = new DrawingService(circle);
        drawingService.RenderShape();  // Output: Drawing a circle. Applying red color.

        // Change both shape and color
        IColor blue = new Blue();
        Shape square = new Square(blue);  // Bridge: Blue square
        drawingService = new DrawingService(square);
        drawingService.RenderShape();  // Output: Drawing a square. Applying blue color.
    }
}
```

### **Why the Bridge Pattern is Useful**

- **Flexible Extensions**: You can independently extend shapes and colors without modifying core code.
- **Decoupling**: The `DrawingService` in the business logic layer (BLL) does not know the specific implementations of shapes or colors, promoting decoupling and scalability. The `Shape` class does not need to know the details of `IColor` implementations (like `Red` or `Blue`). The bridge works by holding a reference to `IColor`, decoupling the abstraction from the implementation.

---

## **3. Composite Pattern**

**Purpose**: The Composite Pattern allows you to treat individual objects and groups of objects uniformly. This is particularly useful in handling hierarchical structures such as file systems, organizational charts, or task processing systems because it simplifies handling individual tasks and groups of tasks under a unified approach.

### **When to Use Composite Pattern**

- When your system involves a whole-part hierarchy, such as tasks and subtasks, enabling uniform handling of individual elements and composite elements.
- To simplify complex structure handling by using the same interface, regardless of whether it is a simple object or a composite object.
- To reduce conditional logic by delegating behavior consistently to components.

### **Composite Pattern Example: A Processing System with Sub-processes**

In this example, we have a **Processing** class for logging operations. Each **Processing** object can contain multiple sub-processes, which are managed uniformly using the **CompositeProcessing** class. This approach helps manage individual tasks and composite tasks through a common interface, cleaning up complex logic in the business logic layer.

```csharp
// Component Interface
public interface IProcessing
{
    void Log();  // Logging interface
}

// Leaf: A single processing task
public class Processing : IProcessing
{
    private string _name;

    public Processing(string name)
    {
        _name = name;
    }

    public void Log()
    {
        Console.WriteLine($"Logging processing task: {_name}");
    }
}

// Composite: A processing task with sub-processes
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
        Console.WriteLine($"Logging composite processing: {_compositeProcessingName}");
        foreach (var subProcessing in _subProcessings)
        {
            subProcessing.Log();  // Delegates logging to sub-processes
        }
    }
}
```

### **Using Composite Pattern in the Business Logic Layer**

Here is an example of how the **Composite Pattern** simplifies logging logic in the business logic layer, where tasks may include multiple subtasks, and both are logged uniformly:

```csharp
class Program
{
    static void Main(string[] args)
    {
        // Individual processing tasks (leaf nodes)
        IProcessing process1 = new Processing("Task 1");
        IProcessing process2 = new Processing("Task 2");

        // Composite processing task (composite node)
        CompositeProcessing mainProcess = new CompositeProcessing("Main Task");

        // Add individual tasks to the composite task
        mainProcess.AddSubProcessing(process1);
        mainProcess.AddSubProcessing(process2);

        // Sub-composite processing task
        CompositeProcessing subProcessGroup = new CompositeProcessing("Sub-task Group");
        subProcessGroup.AddSubProcessing(new Processing("Sub-task 1"));
        subProcessGroup.AddSubProcessing(new Processing("Sub-task 2"));

        // Add the sub-composite to the main task
        mainProcess.AddSubProcessing(subProcessGroup);

        // Log all tasks
        mainProcess.Log();
    }
}
```

### **Why Composite Pattern is Useful in the Business Logic Layer**

- **Uniform Handling**: The Composite Pattern allows you to handle individual tasks and task groups in the same way, reducing the need for complex conditional logic (e.g., `if` statements checking if an object is a single task or a group task).
  
- **Code Modularity and Cleanliness**: By abstracting single tasks and composite tasks into a common interface (`IProcessing`), the business logic layer can handle them uniformly, resulting in cleaner, more modular, and easier-to-maintain code.

- **Scalability**: This pattern allows for easy extension. You can add more processing types (e.g., new subtasks) without modifying the existing structure, making it flexible and scalable for larger systems.

- **Reducing Redundancy**: Without the Composite Pattern, separate logic would be needed for handling single and group tasks. The Composite Pattern unifies this logic, simplifying maintenance and evolution.

By using the Composite Pattern in hierarchical structures (e.g., task management systems), you can simplify code in the business logic layer, making it more extensible, testable, and maintainable.

---

## **4. Decorator Pattern**

**Purpose**: The Decorator Pattern allows you to dynamically add responsibilities to an object without modifying its structure. This is particularly useful for adding features like logging, security, or caching.

### **When to Use Decorator Pattern**

- When you need to dynamically add functionality to objects.
- To avoid creating subclasses for every new feature by extending functionality using decorators.

### **Decorator Pattern Example**

Let’s enhance a basic data stream with encryption and logging capabilities.

```csharp
// Component Interface
public interface IDataStream
{
    void Write(string data);  // Data write interface
}

// Concrete Component
public class FileStream : IDataStream
{
    public void Write(string data) => Console.WriteLine($"Writing data to file: {data}");
}

// Decorator
public class StreamDecorator : IDataStream
{
    protected IDataStream _stream;

    public StreamDecorator(IDataStream stream) => _stream = stream;

    public virtual void Write(string data) => _stream.Write(data);
}

// Concrete Decorator: Encrypted Stream
public class EncryptedStream : StreamDecorator
{
    public EncryptedStream(IDataStream stream) : base(stream) { }

    public override void Write(string data)
    {
        var encryptedData = Encrypt(data);
        base.Write(encryptedData);  // Writes encrypted data
    }

    private string Encrypt(string data) => $"Encrypted({data})";
}

// Concrete Decorator: Logged Stream
public class LoggedStream : StreamDecorator
{
    public LoggedStream(IDataStream stream) : base(stream) { }

    public override void Write(string data)
    {
        Log(data);  // Log data before writing
        base.Write(data);
    }

    private void Log(string data) => Console.WriteLine($"Logging data: {data}");
}
```

### **Why Decorator Pattern is Useful**

- Allows you to add new functionality dynamically without modifying existing code.
- Provides flexibility to stack multiple decorators for cumulative functionality (e.g., logging and encrypting data).
- Encourages composition over inheritance, avoiding the complexity of extensive subclassing.

---

## **5. Facade Pattern**

**Purpose**: The Facade pattern provides a simplified interface to a complex subsystem. It hides the subsystem's complexity, making it easier to interact with the entire system.

### **When to Use the Facade Pattern**

- When you want to simplify interaction with a complex system.
- When you need a unified API for a group of interfaces.

### **Facade Pattern Example**

Below is an example of a multimedia system facade that controls video playback, audio, and subtitles.

```csharp
// Facade class: MultimediaFacade
public class MultimediaFacade
{
    private readonly VideoPlayer _videoPlayer;
    private readonly SubtitleService _subtitleService;

    // Facade constructor initializes all subsystems
    public MultimediaFacade()
    {
        _videoPlayer = new VideoPlayer();
        _subtitleService = new SubtitleService();
    }

    // Simplified interface: Play video with subtitles
    public void PlayVideoWithSubtitles(string videoFile, string subtitleFile)
    {
        _videoPlayer.PlayVideo(videoFile);  // Play video
        _subtitleService.LoadSubtitles(subtitleFile);  // Load subtitles
        Console.WriteLine("Playing video with subtitles...");
    }
}
```

Now, let’s see how **MultimediaFacade** can be used in the business logic layer (BLL) to simplify interactions with the multimedia subsystem.

```csharp
// Business Logic Layer class: MultimediaService
public class MultimediaService
{
    private readonly MultimediaFacade _multimediaFacade;

    // Constructor initializes MultimediaFacade
    public MultimediaService()
    {
        _multimediaFacade = new MultimediaFacade();
    }

    // Play a movie with subtitles
    public void PlayMovieWithSubtitles(string videoFile, string subtitleFile)
    {
        Console.WriteLine("Starting movie playback with subtitles in the business logic layer...");
        _multimediaFacade.PlayVideoWithSubtitles(videoFile, subtitleFile);
    }
}

// Application usage
class Program
{
    static void Main()
    {
        MultimediaService multimediaService = new MultimediaService();

        // Use the business logic layer to play a movie with subtitles
        multimediaService.PlayMovieWithSubtitles("movie.mp4", "movie_subtitles.srt");
    }
}
```

### **Why the Facade Pattern Is Useful**

- **Unified Interface**: `MultimediaFacade` provides a simple interface to interact with complex multimedia subsystems (audio, video, and subtitles). The business logic layer (BLL) doesn’t need to interact with each component individually, only with the facade.
- **Separation of Concerns**: The complexity of the multimedia subsystem is encapsulated within the facade. The business logic layer (BLL) deals only with high-level operations (e.g., playing a video with subtitles or audio files).
- **Ease of Maintenance**: If the multimedia subsystem changes (e.g., new video or audio functionality), those changes are isolated within the facade and do not directly impact the business logic layer (BLL).

---

## **6. Flyweight Pattern**

**Purpose**: The Flyweight pattern minimizes memory usage by sharing the common parts of object state among multiple objects. It allows you to share the same data across multiple objects while storing only unique attributes separately, rather than creating individual instances with many similar attributes.

### **When to Use the Flyweight Pattern**

- When your system involves a large number of similar objects.
- To optimize memory usage by sharing identical data across these objects.

### **Understanding the Flyweight Pattern**

Imagine you’re developing a game with many trees. Each tree has identical attributes like type (e.g., oak, pine), color, and texture, but unique attributes like position in the world.

Creating a separate object for each tree, including all its attributes (name, color, texture, position), would consume a lot of memory, as the same data (name, color, texture) would be stored redundantly for every tree.

The Flyweight pattern solves this problem by **sharing** intrinsic attributes (like name, color, and texture) across multiple trees while storing only unique attributes (like position) for each individual tree.

### **Flyweight Pattern Example: Trees in a Game**

Let’s illustrate how to display many trees in a game, where each tree shares the same structure (name, color, and texture) but has unique positions.

#### **Step-by-Step Example**

1. **Shared State (Intrinsic State)**: Common data across all objects, such as the tree’s name, color, and texture.
2. **Unique State (Extrinsic State)**: Data that varies between objects, such as a tree’s position (`x`, `y`).

#### **Flyweight Class (Shared Tree Type)**

The `TreeType` class represents the **shared state**, such as the tree’s name, color, and texture. Multiple trees of the same type will reuse this object.

```csharp
// Flyweight
public class TreeType
{
    private string _name;  // Intrinsic state (shared)
    private string _color; // Intrinsic state (shared)
    private string _texture; // Intrinsic state (shared)

    public TreeType(string name, string color, string texture)
    {
        _name = name;
        _color = color;
        _texture = texture;
    }

    public void Display(int x, int y)
    {
        Console.WriteLine($"Displaying {_name} tree at ({x}, {y})");
    }
}
```

#### **Flyweight Factory (Manages Shared Objects)**

This factory ensures that each `TreeType` (Flyweight) is created only once. If a tree type with the same name, color, and texture already exists, it reuses the existing one.

```csharp
// Flyweight Factory
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

#### **Context Class (Unique Position Data)**

The `Tree` class represents an actual tree object, but only stores its **unique state**, such as position (`x`, `y`). The shared state (tree type, color, texture) is stored in a shared `TreeType` object.

```csharp
// Context Class
public class Tree
{
    private int _x; // Extrinsic state (unique)
    private int _y; // Extrinsic state (unique)
    private TreeType _type; // Intrinsic state (shared)

    public Tree(int x, int y, TreeType type)
    {
        _x = x;
        _y = y;
        _type = type;
    }

    public void Display()
    {
        _type.Display(_x, _y); // Use shared TreeType and unique position
    }
}
```

#### **Usage in BLL**

Now, you can efficiently create and manage a large number of trees in the game using the Flyweight pattern.

```csharp
class Program
{
    static void Main(string[] args)
    {
        TreeFactory treeFactory = new TreeFactory();

        // Create shared tree types (Flyweights)
        TreeType oakType = treeFactory.GetTreeType("Oak", "Green", "OakTexture");
        TreeType pineType = treeFactory.GetTreeType("Pine", "DarkGreen", "PineTexture");

        // Create trees with unique positions but shared tree types
        Tree tree1 = new Tree(10, 20, oakType);
        Tree tree2 = new Tree(15, 25, oakType); // Reuse oakType
        Tree tree3 = new Tree(5, 30, pineType);

        // Display trees
        tree1.Display();
        tree2.Display();
        tree3.Display();
    }
}
```

### **Why the Flyweight Pattern Is Useful**

1. **Memory Optimization**: By sharing intrinsic state (tree type, color, texture) among multiple trees, memory usage is significantly reduced. The system only stores one instance of each tree type, while each tree object stores only its unique position data.
2. **Performance**: When dealing with thousands or millions of similar objects, the Flyweight pattern improves performance by minimizing object creation and memory usage.
3. **Simplified Object Management**: Separating shared and unique states avoids managing numerous entirely unique objects, simplifying creation and improving maintainability.

---

## **7. Proxy Pattern**

**Purpose**:  
The proxy pattern provides an intermediary or placeholder for an object to control access to it. This is useful in the following situations:

- When you don't want to create an object immediately due to its high resource cost (e.g., requiring a lot of memory or processing power).
- When you need to add extra functionality, such as logging, security checks, or lazy initialization, without modifying the underlying object itself.
- When you want to manage interactions between the client and the actual object by adding an abstraction layer.

### **Types of Proxy Pattern**

1. **Virtual Proxy**: Delays the creation of resource-intensive objects until they are needed (lazy loading).
2. **Protection Proxy**: Controls access to sensitive objects, often adding security checks.
3. **Remote Proxy**: Manages interactions with objects located in different address spaces (e.g., on different servers).
4. **Cache Proxy**: Adds caching functionality to avoid redundant work.

### **Proxy Pattern - Use Cases**

- **Expensive Object Creation**: If an object consumes resources and isn't always needed, a proxy can delay its creation until required.
- **Access Control**: If some users or systems shouldn't have full access to certain objects, a proxy can enforce security rules.
- **Logging/Monitoring**: When you want to log access to or monitor interactions with an object without modifying its core logic.

### **Virtual Proxy Detailed Example in Business Logic Layer (BLL)**

Imagine you have an `ExpensiveObject`, which takes a long time to create or uses a lot of memory. You don't want to create this object until it's absolutely necessary. In this case, you can use a **Virtual Proxy** to delay the creation of the `ExpensiveObject` until it's actually needed.

#### **ExpensiveObject and IExpensiveObject Interface**

First, let's define the `ExpensiveObject` class and the `IExpensiveObject` interface it implements:

```csharp
// Interface for expensive operation
public interface IExpensiveObject
{
    void Process();
}

// A resource-intensive class that requires time to initialize
public class ExpensiveObject : IExpensiveObject
{
    public ExpensiveObject()
    {
        // Simulate the creation process of an expensive object
        Console.WriteLine("ExpensiveObject: Initializing resource-intensive object...");
        System.Threading.Thread.Sleep(2000);  // Simulate delay
    }

    public void Process()
    {
        Console.WriteLine("ExpensiveObject: Processing data...");
    }
}
```

#### **Proxy: ExpensiveObjectProxy**

Now, the **Proxy** will act as an intermediary. It will delay the creation of `ExpensiveObject` until the `Process()` method is called for the first time:

```csharp
// Proxy class that controls access to the expensive object
public class ExpensiveObjectProxy : IExpensiveObject
{
    private ExpensiveObject _expensiveObject;

    // Proxy delays the creation of the object until needed
    public void Process()
    {
        if (_expensiveObject == null)
        {
            Console.WriteLine("Proxy: Creating ExpensiveObject for the first time...");
            _expensiveObject = new ExpensiveObject();
        }
        _expensiveObject.Process();
    }
}
```

In the proxy class, the real `ExpensiveObject` is created only when the `Process()` method is called for the first time. This saves resources if the object isn't needed immediately.

#### **Using Proxy in Business Logic Layer (BLL)**

Now, let's demonstrate how the proxy pattern can be used in the BLL class to manage the creation of `ExpensiveObject`. The BLL layer doesn't need to know whether it's interacting with the real object or the proxy object — it just uses the `IExpensiveObject` interface.

```csharp
// BLL class interacting with the proxy
public class ExpensiveOperationService
{
    private readonly IExpensiveObject _expensiveObjectProxy;

    public ExpensiveOperationService(IExpensiveObject expensiveObjectProxy)
    {
        _expensiveObjectProxy = expensiveObjectProxy;
    }

    // Method in BLL to perform the operation
    public void PerformOperation()
    {
        Console.WriteLine("BLL: Requesting expensive operation...");
        _expensiveObjectProxy.Process();  // Proxy decides when to create the real object
    }
}

// Usage in the application
class Program
{
    static void Main()
    {
        // Use proxy instead of directly creating ExpensiveObject
        IExpensiveObject proxy = new ExpensiveObjectProxy();
        ExpensiveOperationService service = new ExpensiveOperationService(proxy);

        Console.WriteLine("Application: Starting operation...");

        // First request to perform the operation
        service.PerformOperation();  // Proxy creates the real object here

        Console.WriteLine("Application: Performing another operation...");

        // Second request - ExpensiveObject is already created
        service.PerformOperation();  // No need to recreate the object
    }
}
```

#### **Workflow Explanation**

1. When the first call to `service.PerformOperation()` is made, the proxy checks whether the `ExpensiveObject` has been created.
2. If it hasn't, it initializes the `ExpensiveObject` inside the proxy (simulating an expensive operation, such as loading a large dataset, connecting to a remote service, etc.).
3. On subsequent calls, the already created `ExpensiveObject` is reused without reinitialization.

### **Why Use Proxy?**

- **Lazy Loading**: The expensive object is created only when it's truly needed. This can save significant resources, especially if the object may not always be used.
- **Separation of Concerns**: The BLL doesn't need to manage the object creation logic. It interacts with the object via the `IExpensiveObject` interface and doesn't care whether it's dealing with the real object or the proxy.
- **Efficiency**: If multiple operations are requested, the object is created once and reused, improving performance.

The **Proxy Pattern** is useful for controlling access to an object, especially when you want to delay its creation (as in the case of virtual proxies). It performs excellently when dealing with expensive operations, security concerns, or even network calls. In our example, the proxy delays the creation of `ExpensiveObject`, while the BLL interacts with it through an interface, unaware of the underlying complexity.

---

## **Conclusion**

In this blog post, we introduced several structural design patterns and their implementations in C#. Each pattern serves a unique purpose in building efficient code, and knowing when to use them can significantly improve system design. Whether simplifying code with the Facade pattern, optimizing memory with the Flyweight pattern, or enhancing objects with the Decorator pattern, these patterns are important tools for software developers.

Stay tuned for our next blog post on **Behavioral Design Patterns**!
