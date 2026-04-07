# 📚 C# 陷阱与边界情况 — 第 2 部分：面向对象、设计模式

这是**两部分系列**中的**第 2 部分**。如果还没看过，建议先读 **[第 1 部分](CSharp_Questions_CN.md)**（**第 1–7 节**）：类型与默认值、运算符、LINQ、null、异常，以及 **`async`/`Task`**。

## 1. 🚀 引言（续）

**第 1 部分**主要打基础：值类型 vs 引用类型、**`default`**、模式匹配、延迟执行、**`?.`** / NRT、**`throw;`** vs **`throw ex;`**，以及“用同步方式阻塞异步”的常见坑。

这一部分从**第 8 节**继续，重点转向：**对象模型**（继承、多态、`static`）、**泛型与变型**、**委托与事件**（也会呼应**第 1 部分**第 3 节里提到的 **`event` vs 字段**）、**`using` / 资源释放**（对应**第 1 部分**第 6 节的异常处理），以及可选的 **SQL**、**真实场景中的设计模式**，最后再给一份方便复习的**速查表**。

整体思路和前一篇一致：关注**正确思维模型**，而不是罗列语法点。章节编号依然保持 **8–17**，这样你在两篇之间来回查也不会乱。

---

## 8. 🧬 面向对象与继承

面试里的继承题，很少真的让你画 UML。它们更关心三件事：**到底执行了什么、执行顺序是什么、以及哪个类型在起作用**——是变量声明时的类型，还是 `new` 出来的实际类型。

本质上的坑和**第 1 部分**是一样的：**编译期 vs 运行期**。编译器“看到”的，和程序运行时“真正发生的”，往往不是一回事。

**静态类型 vs 运行期类型。** 比如：

```csharp
A a = new B();
```

这里变量 `a` 的编译期类型是 **`A`**，但堆上的对象实际是 **`B`**。这个区别会影响很多行为：

* **成员访问**：通过 `a` 能访问哪些成员，取决于编译期类型 `A`
* **虚方法调用（`virtual` / `override`）**：由运行期类型 `B` 决定
* **`new` 关键字隐藏成员**：这里通常是编译期类型说了算（也是经典面试坑）

面试很喜欢拿这一行做文章，因为你只要把一个关键字从 `override` 改成 `new`，输出结果就可能完全变掉。

**构造函数执行顺序。** 创建一个派生类对象时，初始化顺序是固定的，但很多人直觉会写错：

* 先执行**最派生类的字段初始化**
* 然后进入 `base(...)` 调用链
* 每进入一层：先初始化该类的字段，再执行该类的构造函数体
* 一路往上直到 `object`
* 最后才回到最派生类，执行它的构造函数体

可以这样理解：**在子类构造函数真正开始执行之前，父类已经“完全准备好了”**。

另外要区分：这里说的是**实例构造函数**（面试常考），而**静态构造函数**是另一套规则（每个类型只会在首次使用时执行一次）。


```csharp
class Base
{
    public Base() => Console.WriteLine("Base ctor");
}

class Derived : Base
{
    string log = Init(); // 字段初始值设定项在构造函数体之前运行（见输出顺序）

    static string Init()
    {
        Console.WriteLine("Derived field init");
        return "x";
    }

    public Derived() => Console.WriteLine("Derived ctor");
}

// new Derived() 打印：
// Derived field init
// Base ctor
// Derived ctor
```

**基构造函数调用。** 派生类的构造函数**必须**最终调用某个基类构造函数——要么隐式调用（如果基类有无参构造函数），要么显式写 **`: base(...)`** 或 **`: this(...)`**（后者会先委托给同类的另一个构造函数，但最终仍要走到基类）。如果基类没有可访问的无参构造函数，而你又省略了 **`base(args)`**，派生类就**编译不过**。这是面试中常见的「为什么报错？」陷阱。

**`virtual` / `override` 与 `new`（隐藏）。** 基类里的 **`virtual`** 成员在运行时创建一个可替换的**槽**。派生类用 **`override`** 提供新的实现——无论变量声明为基类还是派生类，运行时都会用派生实现。**`new`** 则**不参与这个槽**，它会在派生类型中引入一个“同名的新方法”，只影响编译期查找。换句话说：如果通过基类引用调用隐藏的方法，调用的仍是基类实现，除非你显式**转换到派生类型**。

**速查：`new` vs `override`（方法签名相同，但规则不同）**

| 机制                       | `Base b = new Derived();` 调用实例方法时    |
| ------------------------ | ------------------------------------ |
| **`override`**           | 调用 **Derived** 的实现（多态）               |
| **Derived 中的 `new`（隐藏）** | 通过类型为 **Base** 的引用调用时执行 **Base** 的实现 |


```csharp
class BaseV
{
    public virtual void M() => Console.WriteLine("BaseV.M virtual");
}

class DerivedOverride : BaseV
{
    public override void M() => Console.WriteLine("DerivedOverride.M");
}

class BaseH
{
    public void M() => Console.WriteLine("BaseH.M"); // 非 virtual — 派生中 `new` 隐藏此名
}

class DerivedHide : BaseH
{
    public new void M() => Console.WriteLine("DerivedHide.M");
}

static class DispatchDemo
{
    public static void Run()
    {
        BaseV bv = new DerivedOverride();
        bv.M(); // 输出: DerivedOverride.M

        BaseH bh = new DerivedHide();
        bh.M(); // 输出: BaseH.M — `new` 未替换虚分派（本无虚槽）

        ((DerivedHide)bh).M(); // 输出: DerivedHide.M — 编译期类型为 DerivedHide
    }
}
```

若你可能想用 **`override`** 却用了 **`new`**，编译器会警告（CS0108/CS0114 族——措辞因场景而异）。把该警告当提示：**`new`** 几乎总是刻意的**隐藏**，不是多态的替代品。

**抽象类与接口。** **抽象类**是单继承锚点：可把**字段**、**protected** 状态与**共享**实现放在一处。C# 只允许**一个**基类，因此抽象类「占用」该槽。**接口**是**契约**：一个类型可实现**多个**接口。

```csharp
abstract class Animal
{
    protected string Name;
    public Animal(string name) { Name = name; }
    public abstract void Speak(); // 可有字段与共享实现
}

interface IFly
{
    void Fly(); // 仅是契约 — 无字段
}

class Bird : Animal, IFly
{
    public Bird(string name) : base(name) { }
    public override void Speak() => Console.WriteLine($"{Name} says tweet!");
    public void Fly() => Console.WriteLine($"{Name} flies!");
}
```

**听起来相似的访问修饰符。** 多数面试停留在 **`public` / `private` / `protected` / `internal`**。组合形式容易绊人：

* **`protected internal`** 表示**任一**门通过即可：**同一程序集**内的代码**或****派生**类型中的代码（即便在另一程序集）可见该成员。
* **`private protected`**（C# 7.2+）**更严**：仅**派生**且**同一程序集**的类型可见。它**不是**「private *或* protected」；是**交集**，不是并集。

**`sealed`。** 类上的 **`sealed`** 禁止再派生——框架类型有时为保持不变量与可预测行为而密封。 **`override` 方法上的 `sealed`** 阻止链上进一步重写。面试里若把 **JIT 去虚化**当作*主要*理由，比谈**设计**（密封表明「此特化已完成」并保护假设）要弱。

面试官在 OOP 与继承上常探的点：

* **构造函数顺序**与**字段初始值设定项**、**`: base` / `: this`**——先打印什么，为什么？
* **`Base b = new Derived()`** — 对 **`virtual`**、**`override`**、**`new`**，哪个**重载**或**方法**会运行？
* **何时适合 `new`** versus 用 **`virtual`/`override`** 修正基类设计。
* **抽象类 vs 接口** 作为新抽象——状态、默认行为、**DIM** 版本控制、可测试性。
* **`protected internal` 与 `private protected`** — 可见性的并集 vs 交集。

常见陷阱：

* 以为 **`new`** 像 **`override`** 一样产生多态行为；并不会；对基类型引用，它**破坏**该成员的统一处理。
* 忘记从**派生**类型初始值设定项序列的角度看，**字段初始值设定项**在**基构造函数体**之前运行（见上文示例输出）。
* 混淆 **`protected internal`**（**或**）与 **`private protected`**（**且**——同程序集**且**派生）。
* 仅谈「速度」而密封——未提 **API** 与**不变量**理由——面试官常想听工程判断，而非微优化传言。

---

## 9. 🧠 多态深入

**第 8 节**已讲清 **`virtual`/`override`** 与 **`new`** 及 **`Base b = new Derived()`** 分派。要带走的话很短：

> **编译期（引用）类型**决定哪些成员*可见*以及**非虚**重载如何解析；**运行期类型**决定 **`virtual`/`override`** 行为。

本节叠加上面试常考：**安全得到正确类型**、**接口作为系统外缘形状**、以及多态在玩具片段之外**为何**有价值。

**转型：直接转型 vs `as` vs `is`。** **直接转型** `(T)obj` 告诉运行期「此对象真是 **`T`**。」若不是，得到 **`InvalidCastException`**。**`as`** 运算符 **`obj as T`** 在引用与 **`T`** 不兼容（或值已为 **`null`**）时返回 **`null`**，不会因该次失败转换而抛——自然与 **`null`** 检查配对。现代 C# 中 **`is`** 常为**模式**：**`if (obj is T t)`** 仅在模式匹配时赋值 **`t`**；配合可空引用类型，编译器常在该分支收窄 **`null`**。单独的 **`is T`** 是无赋值的布尔测试。

```csharp
object o = "hello";

string ok = (string)o;           // "hello" — 若 `o` 不是 string 则抛 InvalidCastException
string? viaAs = o as string;     // 同一引用或 null — 不匹配时不抛
// int bad = (int)o;             // InvalidCastException — 不是装箱的 int

if (o is string s)               // 模式：块内 `s` 为 string
    Console.WriteLine(s.Length);
```

**细节：** **`as`** 仅适用于**引用类型**、**可空值类型**与**接口**——不适用于像 **`int`** 这样的非可空值类型，因为「转换失败」没有 **`null`** 哨兵。值类型用 **`is`**、**模式**，或 **`Nullable<T>`** 模式。

**基于接口的设计。** 依赖 **`IEnumerable`**、**`IDisposable`**、你自己的 **`IOrderRepository`** 等——而不是到处写具体 **`SqlOrderRepository`**。实现变化时**调用点**仍稳定，**替换**显而易见：满足契约的任何东西都可放在变量后。**第 8 节**对比了**抽象类**（状态 + 一个基）与**接口**（多契约、现代 C# 的 **DIM**）。面试里「面向接口编程」不是教条——是**控制耦合**。

**显式接口实现（EIMI）。** 当类实现的两个接口用**相同签名**（同名同形）定义成员，或你希望成员**仅**能通过接口类型调用时，用 **`void IFoo.M()`** 风格语法实现。类表面上**没有**名为 **`M`** 的 `public` 方法，除非你另加公共包装。持有 **`ConcreteType`** 的调用者可能完全看不到 **`M`**；持有 **`IFoo`** 的可以。这是刻意的**封装**与**名称消歧**，不是编译器 bug。

```csharp
interface IReadable { void Load(); }
interface IWritable { void Load(); } // 同名，不同契约 — 人为但面试常见形状

class Doc : IReadable, IWritable
{
    void IReadable.Load() => Console.WriteLine("read path");
    void IWritable.Load() => Console.WriteLine("write path");
}

static class EimiDemo
{
    public static void Run()
    {
        var doc = new Doc();
        // doc.Load(); // 无法编译 — 无公共 Load

        IReadable r = doc;
        r.Load(); // 输出: read path

        IWritable w = doc;
        w.Load(); // 输出: write path
    }
}
```

**在真实系统中为何重要。** **单元测试**用 **fakes** 与 **stubs** 换**接口**实现，无需继承生产中的密封类型。**插件**与**模块化**宿主在**稳定接口**或**抽象**契约后加载实现。**ORM** 与**代理**常给你**子类**或**运行时生成**的类型，仍**像**你的 **`DbContext`** 实体或 **`IRepository`** 一样行为——你的代码留在**抽象**上，那些技巧保持**不可见**。面试收获：多态不是「OOP 词汇」；它是**在何处允许变化**而无需重写调用方。

面试官在多态上常探的点：

* **哪种转型会抛**，哪种返回 **`null`**，何时 **`is` 模式**比 **`as` + 检查**更清晰。
* **EIMI**：谁能调用该方法，如何**消歧**两个接口。
* **设计**：新接缝用接口还是抽象基——**测试替身**与**演进**是合理追问。

常见陷阱：

* 用 **`as`** 却忘记 **`null`**——**`as`** 静默失败，下一行 **`NullReferenceException`**。
* 以为**直接转型**失败会给 **`null`**；只有 **`as`**（对支持的 **`T`**）会。
* **EIMI** 在公共类型上「找不到」——然后疑惑 **`new Doc().M()`** 为何不编译。

---

## 10. 🧪 静态与实例

**实例**成员属于**对象**：每个实例有自己的**字段**，**`this`** 指向该对象。  
**静态**成员属于**类型**：一个 **`static` 字段**被**所有**实例共享（以及根本没有实例的代码）。  
**`static`** 方法里没有 **`this`**，因为没有**接收者对象**——只有**类型**。

```csharp
class Counter
{
    public int InstanceCount = 0;        // 每个对象自有
    public static int StaticCount = 0;   // 所有对象共享

    public Counter()
    {
        InstanceCount++; // 只影响本对象
        StaticCount++;   // 影响类型共享计数
    }

    public void IncrementInstance() { InstanceCount++; }
    public static void IncrementStatic() { StaticCount++; /* 无 this! */ }
}

var c1 = new Counter();
var c2 = new Counter();
Console.WriteLine(c1.InstanceCount); // 1
Console.WriteLine(c2.InstanceCount); // 1
Console.WriteLine(Counter.StaticCount); // 2
Counter.IncrementStatic(); // 静态方法：无需实例
// c1.IncrementStatic(); // 合法但易误导；优先 Counter.IncrementStatic() 
```

---

**何时用 `static`：**  
对**无每对象状态**的**纯辅助**（如 **`Math.Max`**、字符串工具）、**单例/共享缓存**（谨慎！）、**不变量**（**`const`**/**`static readonly`**）使用 **static**。

```csharp
public static class MathHelpers
{
    public static int Double(int n) => n * 2; // 纯、无状态辅助
}

public class SingletonDemo
{
    public static readonly SingletonDemo Instance = new SingletonDemo(); // 「单例」
    private SingletonDemo() { /* ... */ }
}
```

**运算符重载**与**扩展方法**必须是 static：
```csharp
public class Money
{
    public decimal Value;
    public Money(decimal value) { Value = value; }
    
    // 运算符重载必须是 static
    public static Money operator +(Money a, Money b) => new Money(a.Value + b.Value);
}

// 扩展方法必须在 static class 内且本身为 static
public static class StringExtensions
{
    public static bool IsCapitalized(this string s) => !string.IsNullOrEmpty(s) && char.IsUpper(s[0]);
}
```

---

**`static` class：**  
**`static` class** 等价于 **`abstract`** 且 **`sealed`**：  
**不能**实例化（`new`），也**不能**继承：

```csharp
static class GlobalHelpers
{
    public static void SayHello() => Console.WriteLine("Hello!");
}

// var obj = new GlobalHelpers();     // 编译错误：不能创建实例
// class MyHelpers : GlobalHelpers {} // 编译错误：不能从 static 派生
```

**`static` 构造函数**每类型在**首次使用前**运行**一次**，**线程安全**：

```csharp
class Logger
{
    public static readonly Logger Instance;

    static Logger() // 恰好在首次使用 Logger 之前运行一次
    {
        Instance = new Logger();
        // 此处抛错是进程级致命错误！
    }

    private Logger() { }
}
```

---

**限制与测试痛点：**  
- **`static` 方法不能访问实例字段/方法（无 `this`）**  
- **重度 static 可变状态**使**测试与并行执行**棘手  
- **不能重写 static 方法：** 不能通过子类或接口做多态  
- **依赖注入（DI）** 偏好**实例**服务：
    - **静态服务定位器**是反模式（隐藏依赖、破坏测试）

```csharp
class Trouble
{
    public static int GlobalState = 0;
    public static void SetState(int x) => GlobalState = x;
}

// 测试示例：若一个测试改了 Trouble.GlobalState，另一测试可能得到意外结果
```

> **最佳实践：** 对真正想要全局的无状态辅助或进程级配置用 `static`。对服务、逻辑以及希望测试/复用/模拟的东西，优先*实例* + *接口*。


面试官在 `static` 上常探的点：

* **`static`** 方法为何没有 **`this`**；**共享**状态对**线程**与**测试**意味着什么。
* **`static` class** 与仅有 **`static`** 成员的普通类——**继承**与**构造**差异。
* **`static`** 何时合适 versus **实例** + **接口** 以换取**可测试性**。

常见陷阱：

* 图方便用 **`static`** **可变** **全局**状态——**不稳定**测试与**隐藏**耦合。
* 把 **`static`** 方法当**多态**；**override** 适用于**实例** **`virtual`** 成员，不适用于 **`static`**（派生类型中的 **`static`** 按名称**隐藏**基 **`static`** 成员——又是**编译期**故事，不是**虚**分派）。

---

## 11. 🧩 泛型、委托、事件与扩展方法

本节把四个「语言机制」话题捆在一起——出现频率低于 **`async`** 或 **null**，但仍会作为追问。记住**头条规则**；除非面试官深挖，否则不必证明每个变型角落。

**泛型与 `where` 约束。** **`where T : class`**（仅引用类型）、**`where T : struct`**（非可空值类型）、**`where T : new()`**（无参构造函数——便于 **`new T()`**）、**`where T : SomeBase`**、**`where T : ISome`**，以及新版中的组合（**`class?`**、**`struct?`**、**`notnull`** 等）告诉编译器 **`T`** 能是什么，使重载与 API 保持安全。无约束时，除非**转型**，否则只有 **`object`** 级操作。

**协变（`out`）与逆变（`in`）。** 这些关键字用于**接口与委托上的泛型类型参数**，而非任意 **`List<T>`**。**`out T`** 表示「**T** 只出现在**输出**位置」——当 **`Dog : Animal`** 时，调用方可把 **`IEnumerable<Dog>`** 当作 **`IEnumerable<Animal>`**（**只读**动物序列）。**`in T`** 表示「**T** 只出现在**输入**位置」——可把 **`IComparer<Animal>`** 传给需要 **`IComparer<Dog>`** 的地方（**更宽**的比较器，**更窄**的类型）。**`IList<T>`** 是**不变**的：既能**读**又能**写** **`T`**，若允许 **`IList<Dog>`** 当作 **`IList<Animal>`**，别人就能往你仍认为是**仅狗**的列表里**插入 `Cat`**——类型系统拒绝那样做。

```csharp
class Animal { }
class Dog : Animal { }

IEnumerable<Dog> dogs = Array.Empty<Dog>();
IEnumerable<Animal> animals = dogs; // OK — IEnumerable<out T> 协变

// IList<Dog> dogList = new List<Dog>();
// IList<Animal> broken = dogList; // 编译错误 — IList<T> 不变
```

**委托与多播。** **委托**是带类型的方法指针；**`+=`** 增加另一个目标（**多播**）。对 **`void`** 返回，调用按顺序执行；若某一目标**抛错**，该同步调用中**后续目标不会**执行。对非 **`void`** 多播委托，**返回**值来自列表中**最后一个**目标——在 trivia 题里易被忘记。

**事件与委托字段。** **`event`** 将**外部**代码限制为只能 **`+=`** / **`-=`**；只有**声明类型**应**引发**（调用）它。这与 **[第 1 部分](CSharp_Questions_CN.md)** **第 3 节**是同一故事——把 **`event`** 当作**能力**，不是**共享字段**。常见表面是 **`EventHandler` / `EventArgs`**（或 **`EventHandler<TEventArgs>`**）：**`object? sender`**，**`EventArgs`**（或派生类型）承载数据。

**扩展方法。**  
**扩展方法**是定义在 **`static` class** 里的 **`static`** 方法，**第一个**参数带 `this T target` 标记。这样你可以用实例式语法调用——`target.Method(...)`——尽管方法实为 static：编译器在底层把 `target.Method(args)` 重写为 `Extensions.Method(target, args)`。

**注意：** 扩展方法不能访问 `T` 的 private 成员；只能使用 `T` 公开暴露的东西。  
要使用它们，必须**导入命名空间**（`using YourNamespace;`）。若存在相同签名的实例方法，实例方法*优先*——扩展是后备。

若「我的扩展被忽略了」，检查：  
- 是否忘了 `using`？  
- class 或 method 是否为 public？  
- static class 是否在作用域内？  
- 是否有真实实例方法同名？

**示例：**

```csharp
namespace Extensions
{
    public static class StringExtensions
    {
        // string 的扩展方法：统计词数
        public static int WordCount(this string s)
        {
            if (string.IsNullOrWhiteSpace(s)) return 0;
            return s.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
        }
    }
}

// 用法
using Extensions;

class Program
{
    static void Main()
    {
        string msg = "Hello world from extension methods";
        int count = msg.WordCount(); // 调用扩展；打印 5
        Console.WriteLine(count);
    }
}
```

这里 `WordCount()` 并非 `string` 的真实实例方法，但导入 `Extensions` 后，可以像实例方法一样调用 `msg.WordCount()`。

面试官此处有时会触及：

* **为何** **`List<Dog>`** 不是 **`List<Animal>`** 即便 **`Dog : Animal`**（**不变性**与**写**安全）。
* **多播**顺序、**异常**与**返回值**规则。
* **事件**封装 versus 公共 **`Action`** 字段。

常见陷阱：

* 在**可变**集合（**`List<T>`**、**`IList<T>`**）上期待**变型**。
* **引发** **`event`** 时未做 **null** 检查或 **null 条件**（`?.Invoke`），而后备字段可能为空。
* 忘记**扩展**只是**语法糖**——对 **private** 状态无**特殊**访问。

---

## 12. 🧰 资源与释放（`using`、`IDisposable`）

**`IDisposable`** 是**确定性**清理的模式：**文件**、**句柄**、**数据库连接**、**定时器**——凡应在用后尽快**释放**、而非仅等 **GC** 跑的东西。**`using`** 是 **`try` / `finally`** 调用 **`Dispose()`** 的**语法**，以便 **`try`** 抛错时仍清理——与 **[第 1 部分](CSharp_Questions_CN.md)** **第 6 节**的控制流图景相同（**`finally`** 在 **`return`** 前运行）。

**`using` 语句 vs `using var`。** 经典 **`using (var x = …)`** 在块的**闭合大括号**处释放。**`using var x = …;`**（C# 8+）在**封闭作用域**末尾释放——常为**方法**——写起来短，但更易**误读**（「为何还开着？」）。要知道作用域**在何处结束**。

**`IAsyncDisposable` 与 `await using`。** 异步清理（带 **async** flush/close 的**流**）用 **`await using`**，使 **`DisposeAsync()`** 在 **`async`** 方法中正确参与。

**实务坑。** 许多 **BCL** 类型允许 **`Dispose()`** 调用两次（**幂等**）；不要假定**你未编写的**每个类型都如此。若 **`Dispose`** 抛错，你仍须决定**日志**、**抑制**或**传播**——**嵌套 `using`** 若不刻意可能隐藏**次要**故障。

面试官有时会问：

* **`using`** 在脑子里展开成什么（**`finally` + `Dispose`**）。
* **`using var`** 相对方法体在**何处**释放。
* **确定性**清理对**非托管**或**稀缺**资源为何重要。

常见陷阱：

* 短期**昂贵**资源未用 **`using`** / **`try`/`finally`** 导致**泄漏**。
* **方法**作用域上**长寿命**的 **`using var`**——比**紧**块**持有**资源**更久**。

---

## 13. 🗄️ SQL 与数据层陷阱（可选 — 全栈 / 后端岗位）

纯 C# 面试可以跳过这一节；如果涉及 **EF Core**、**Dapper** 或「手写 SQL」的练习，就保留。重点是理解如何写出合理的**谓词**和**返回结果形状**——不是死记每种方言的每个关键字。

**`=` vs `IN` vs `LIKE`**

* **`=`**：用于**单值相等**。当列有索引且表达式可以走索引时，通常性能最好。注意：尽量不要把列包在函数里，否则可能失去索引优化。
* **`IN (...)`**：适合**小且固定的列表**。如果列表很大或是动态生成，数据库通常更推荐使用 **JOIN 临时表**、**表值参数**或批量处理，而不是写一个巨大的 `IN`。
* **`LIKE`**：用于**模式匹配**，`%` 表示任意长度、`_` 表示单个字符。大小写敏感性依赖排序规则（Collation）。前导通配符模式（如 `%foo`）通常会放弃索引扫描，性能较差。

总的来说，面试时要能区分清楚：**“等于某个 id”** 与 **“包含某个子串”** 的场景和性能差异。

**示例：**

假设有 `Users` 表，列 `Id`、`Username`、`Email`。三种写法行为如下：

```sql
-- 相等：找特定 Id 的用户
SELECT * FROM Users WHERE Id = 42;

-- IN：Id 在固定集合中
SELECT * FROM Users WHERE Id IN (10, 42, 99);

-- LIKE：用户名以 'alex' 开头
SELECT * FROM Users WHERE Username LIKE 'alex%';

-- LIKE（前导通配符！）：用户名包含 'lex'
SELECT * FROM Users WHERE Username LIKE '%lex%';
```

- 有**单个精确**值时用 `=`。
- **短、固定列表**用 `IN`。
- **模式**匹配用 `LIKE`，但注意前导通配符（`%foo` vs `foo%`）对性能的影响。

**Join vs 子查询。** **Join** 关联**两**组行；**相关子查询**对**每个外层行**运行，若优化器救不了你，容易把自己写成 **O(n²)**。**非相关**子查询在 **`IN`** / **`EXISTS`** 里常**可以**——**`EXISTS`** 对 **「是否存在匹配行？」** 往往最清晰。说「join 总是更快」的面试答案是**错的**；真实工作要**度量**与**读计划**，白板上优先**清晰**的 **join** 图与 **`EXISTS`**，除非问题很平凡，否则少用**嵌套** **相关** **select**。

**自连接。** 同一张表取**两次**用**不同别名**——经典 **「员工与经理」** 或 **推荐**图（**`User.ReferredById` → `User.Id`**）。把 **`Users child`** 与 **`Users referrer`** 在 **`child.ReferredById = referrer.Id`** 上 **join**。陷阱：**`NULL`** **外键**（**无**推荐人）会从**内连接**中**掉出**——**孤儿** / **根**行仍应出现时用 **`LEFT JOIN`**。

面试官有时会问：

* **`IN`** 何时可接受 versus **join** 到 id **列表**。
* **存在性**测试用 **`EXISTS` vs `IN`**（以及 **`NOT IN`** 中的 **NULL** 行为——SQL 里常见的**坑**）。
* **为何** **`LIKE '%x'`** 可能是**性能**红旗。

常见陷阱：

* 子查询可能产生 **`NULL`** 时的 **`NOT IN (subquery)`**——逻辑常出人意料；**`NOT EXISTS`** 通常**心智上更安全**。
* 需求是**精确**匹配还是**模式**时，混淆 **`=`** 与 **`LIKE`**。


---

## 14. 🏁 结论

贯穿**两部分**，核心收获是一致的：建立**正确的思维模型**——理解默认值、分派规则、代码何时运行，以及类型系统在编译期能证明什么、不能证明什么。

> 「大多数 bug 并非来自复杂代码，而是源于对简单行为的误解。」

**第 1 部分**像是基础设施：类型、LINQ、null、异常与 **`async`/`Task`**。
**第 2 部分**在此之上扩展：对象模型（继承、多态、**`static`**）、泛型与事件、资源释放、SQL 笔记、架构要点。

每一部分都可以独立阅读，但合起来，你会从字面量与运算符的基础，一路理解到如何写出可维护的 **.NET** 后端代码——无需一次读完，也能逐步消化。
