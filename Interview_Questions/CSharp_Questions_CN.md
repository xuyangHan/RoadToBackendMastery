# 📚 C# 陷阱与边界情况 — 第 1 部分：从基础到异步

本文是**两部分系列**中的**第 1 部分**，涵盖**第 1–7 节**（类型到 **`async`/`Task`**）。**[第 2 部分](CSharp_Questions_Part2_CN.md)** 将继续讲解面向对象、泛型、资源释放、设计模式以及速查内容——这样每一篇都可以更聚焦，而不是一次性塞进太多内容。

## 1. 🚀 引言

C# 表面上看起来很直观，但很多面试题会用几行看似简单的代码，考出完全出乎意料的结果。这种“反直觉”的地方，往往来自语言在背后替你做的事情——默认值、类型规则，以及那些你从未显式写出来的行为。

这篇文章整理了一些常见的「陷阱」：它们在日常开发中不一定经常手写，但却很容易被问到、也很容易出错。重点不是死记每一个边界情况，而是理解语言**为什么**会这样设计，这样你在面对新问题时也能自己推导出来。

有两个核心概念会反复出现：

* **编译期 vs 运行期。** 有些行为在程序运行**之前**就已经确定了，比如编译器如何选择重载、推断类型；而另一些则只有在代码真正**执行**时才会体现出来，比如 `null`、虚方法调用、延迟执行的 LINQ，以及 `async` 的续体。把这两者混在一起，是很多“坑”的来源。

* **隐藏的默认值。** 字段、泛型、枚举、集合等都有各自的默认规则，而你往往不会显式写出来。一旦题目依赖这些默认值，如果你没意识到语言已经“帮你填好了”，答案就会看起来不对。

---

## 2. 🧱 数据类型与默认值（基础）

大多数关于 C# 的「刁钻」题，归根结底还是在考**类型**：什么会被复制、什么可以是 `null`，以及当你没有显式赋值时，语言默认帮你做了什么。本节把这些基础集中梳理一下。

**值类型 vs 引用类型**是最核心的一层划分。值类型（如 `int`、`bool`、`DateTime`、结构体）在赋值或传参时通常会**复制一份数据**，每个变量都有自己独立的值。引用类型（类、接口、委托、数组）则保存的是对象的**引用**；变量之间复制的是“指向同一个对象的地址”，而不是对象本身。面试里很常见的一类问题，就是考你是否清楚在某一行赋值之后，是变成了共享**同一个对象**，还是变成了两个**独立副本**。

**默认值与 `default`。** 每种类型都有对应的默认值：数值类型是 0，`bool` 是 `false`，引用类型是 `null`，结构体则是“所有字段都为各自的默认值”。你可以显式写 `default(T)`，或者在类型已知时直接使用 `default` 字面量。这里有一个很容易被忽略的点：**字段**和**局部变量**的行为是不一样的。字段在未赋值时会自动初始化为默认值，而局部变量必须先被**明确赋值**后才能使用，否则会直接编译报错。


```csharp
int a = default(int);   // 0
int b = default;        // 从左端推断类型时同上
string s = default;     // 引用类型为 null

class Example
{
    int count;            // 0
    bool flag;            // false
    decimal amount;       // 0m
    double ratio;         // 0d
    DateTime when;        // DateTime.MinValue (0001-01-01, 00:00:00)
    string label;         // 不可为 null 的 `string`（NRT）：运行期默认与其它引用字段一样（null），直到赋值；编译器可能警告（CS8618）
    string? name;         // 可空 `string?`：默认 null；类型含义是「可能为 null」
    int? optional;        // null — 无值
    Guid id;              // Guid.Empty
    char ch;              // '\0' (U+0000)

    struct Pair { public int A; public string? B; }
    Pair pair;            // A = 0, B = null

    void M()
    {
        Console.WriteLine($"count    = {count}");       // 输出: count    = 0
        Console.WriteLine($"flag     = {flag}");       // 输出: flag     = False
        Console.WriteLine($"amount   = {amount}");     // 输出: amount   = 0
        Console.WriteLine($"ratio    = {ratio}");      // 输出: ratio    = 0
        Console.WriteLine($"when     = {when:O}");     // 输出: when     = 0001-01-01T00:00:00.0000000
        Console.WriteLine($"label    = {(label is null ? "null" : label)}"); // 输出: label    = null
        Console.WriteLine($"name     = {(name is null ? "null" : name)}");     // 输出: name     = null
        Console.WriteLine($"optional = {(optional.HasValue ? optional.Value.ToString() : "null")}"); // 输出: optional = null
        Console.WriteLine($"id       = {id}");         // 输出: id       = 00000000-0000-0000-0000-000000000000
        Console.WriteLine($"ch       = {(ch == '\0' ? "\\0 (nul)" : ch.ToString())}"); // 输出: ch       = \0 (nul)
        Console.WriteLine($"pair     = (A={pair.A}, B={(pair.B is null ? "null" : pair.B)})"); // 输出: pair     = (A=0, B=null)

        int local;
        // Console.WriteLine(local); // 错误：local 未明确赋值
        local = default;
        Console.WriteLine($"local (after assign) = {local}"); // 输出: local (after assign) = 0
    }
}
```

面试官喜欢与默认值和类型挂钩的点：

* **`string`。** 它是引用类型，但在许多 API 中表现得像值：不可变，且 `==` 按**字符内容**比较，而非引用同一性。这会让只背过「引用类型用引用相等」的人踩坑。

* **`DateTime`。** 普通 `DateTime` 是值类型，**绝不**为 null。若需要「没有日期」，用 `DateTime?`（`Nullable<DateTime>`）。除非使用可空形式，否则将 `DateTime` 与 `null` 比较是编译期错误。

* **`Nullable<T>`（`int?` 等）。** 包装值类型以表示「无值」与正常值并存。进阶时要了解它如何提升运算符，以及 `HasValue` / `Value` 的用法。

* **装箱与拆箱。** 把值类型放进 `object` 或非泛型接口槽位会在堆上分配装箱副本。这既耗内存，也可能改变相等性行为（例如比较装箱值）。热路径与隐蔽 bug 都会出现。

    ```csharp
    int x = 5;
    object boxed = x;        // 装箱：int 复制到堆上的 object
    int y = (int)boxed;      // 拆箱：object 转回 int

    Console.WriteLine(x == y);        // True：比较值（都是 int）
    Console.WriteLine(boxed == (object)y); // False：不同装箱对象（引用不同）


    // 即便数值相等，装箱对象也是不同实例！

    void Log(object value)
    {
        Console.WriteLine(value);
    }

    Log(42); // ⚠️ 此处发生装箱
    ```

* **`decimal` 与 `double`。** `double` 是二进制浮点：快，但像 `0.1` 这样的分数并不总能精确表示，因此 `0.1 + 0.2` 是经典惊讶。`decimal` 是十进制浮点，通常是金额与精确十进制运算的选择，但有一定性能代价。

* **`const` 与 `readonly`。**  
  - `const` 必须是**编译期**常量（字面量及简单组合）。  
  - `readonly` 字段只能赋值一次——在实例构造函数中，静态字段则在静态构造函数中。  

```csharp
public class Example
{
    public const int ConstValue = 10; // 编译时用值替换，必须是常量
    public readonly int ReadonlyField; // 只能在构造函数中赋值

    public int NormalField; // 可随时修改

    public Example(int val)
    {
        ReadonlyField = val; // 允许：在构造函数中设置
        // ConstValue = 20;   // ❌ 错误：不能对 const 赋值
    }
}
```

**对照表**

| 特性       | 何时设定？                     | 初始化后能否修改？ | 用法示例                    |
|------------|-------------------------------|--------------------|-----------------------------|
| `const`    | 声明时（编译期）               | ❌ 否              | `public const int X = 1;`   |
| `readonly` | 构造函数中（运行期）           | ❌ 否              | `public readonly int Y;`    |

* **枚举。** 默认数值为 `0`，即便你从未为 0 定义名称。从整数强制转换可能得到**未命名**的值；

```csharp
enum Color { Red = 1, Green = 2, Blue = 4 }
Color c1 = (Color)0;         // 0 合法，但未命名
Color c2 = (Color)7;         // 7 不是命名值，但运行期仍合法
```

**可空引用类型（NRT）**是在引用类型之上的**编译期**一层：`string` 表示「不应为 null」，`string?` 表示「可能为 null」，在你忽略时编译器会警告。`!` 运算符只是让编译器别警告；**不会**加入运行期检查。**第 5 节**会更深入 null 处理与 NRT 在日常代码中的位置。

常见陷阱：

* **`null` 与空字符串。** `null` 表示「没有字符串」；`""` 是长度为 0 的真实字符串。在 NRT 下，将 `null` 赋给不可为 null 的 `string` 会产生警告（若将警告视为错误则直接报错）。

* **`DateTime` 与 null。** 场景需要「缺失」时用 `DateTime?` 或对 `default(DateTime)` 做检查，而不是在非可空 `DateTime` 上使用 `null`。

* **创建后的数组。** 引用类型的新数组元素填 `null`；值类型的新数组元素填该类型的默认比特。

* **局部变量与字段。** 指望局部像字段一样「默认初始化」会导致编译错误；指望字段像明确赋值的局部一样行为则可能导致微妙的零或 null bug。

* **装箱比较。** 一侧装箱一侧不装箱时，相等性与同一性都可能出人意料；在重要场景应使用清晰类型与未装箱比较。

---

## 3. ➕ 运算符与表达式

表达式看起来很小，但编译器很忙：它会挑选重载、提升类型，并应用你从未写下的规则。面试题常利用这种落差——尤其是当 `+`、`==` 或 `switch` 在英语里「像」数学或相等，但在 C# 里含义更严格时。

**模式匹配**是语言在**运行期**检查值的类型并一步绑定名称的地方。你会看到带类型检查的 `is`。**经典 `switch`** 是语句级，除非 `break` 或 `return` 否则会贯穿。对**枚举**，若未覆盖每个命名值，编译器**不会**警告——从整数转换仍可能在运行期产生无效枚举值，因此无法保证「穷尽」处理。**`when` 守卫**可在每个 `case` 上增加额外条件，而无需嵌套巨大 `if` 链。

```csharp
// 仅类型检查的 `is`（C# 8 之前）
object obj = "hello";
if (obj is string s)   // 类型检查并绑定到 s
{
    if (s.Length > 0)  // 长度单独检查
        Console.WriteLine(s);
}

// 带 `when` 守卫的经典 switch 语句（C# 8 之前）
static string Describe(object x)
{
    switch (x)
    {
        case int n when n < 0:
            return "negative";
        case int n:
            return $"int {n}";
        case null:
            return "null";
        default:
            return "other";
    }
}

// 枚举示例（C# 8 之前）
enum Level { Off = 0, On = 1 }

Level rogue = (Level)99; // 从 int 转换，可能不是命名成员

string label;
switch (rogue)
{
    case Level.Off:
        label = "off";
        break;
    case Level.On:
        label = "on";
        break;
    default:
        label = "not a named member"; // 捕获任意无效枚举
        break;
}
```

**`checked` 与 `unchecked`。** 整数运算默认在 *unchecked* 上下文中进行（普通 `int` 代码的默认，除非项目启用了 checked 算术）。在 *checked* 上下文中，溢出会抛出 `OverflowException`，而不是静默回绕。整个项目的设置（`<CheckForOverflowUnderflow>` 或旧式 `/checked`）会改变你对白板上片段的预期。

```csharp
int max = int.MaxValue;

// 默认（unchecked）：溢出时回绕（无异常）
int wrapped = max + 1;  // 回绕到 int.MinValue（溢出但不抛异常）
Console.WriteLine(wrapped); // 输出: -2147483648

// Checked：溢出时抛出
try
{
    int result = checked(max + 1); // 抛出 OverflowException
}
catch (OverflowException)
{
    Console.WriteLine("Overflow detected!");
}

// 也可用 'unchecked' 强制不抛异常：
int noError = unchecked(max + 1); // 始终回绕，从不抛出
```

常见陷阱：

* **`char` 与 `+`。** `'a' + 'b' + 'c'` 会提升为 `int`（Unicode 码点）再相加：97 + 98 + 99 = **294**，不是字符串。只有表达式链中出现 `string` 时，`+` 才会像往常那样对后续部分做字符串拼接。

* **字符串与数字。** `"1" + 2` 是字符串拼接（`"12"`）。`1 + 2 + "3"` 从左到右：先得到 `3`，再得到 `"33"`。若本意是算术，应用括号或显式解析。

* **`==` 与 `.Equals()`。** 对许多类型，`==` 被重载或提升（可空类型）。**`x.Equals(y)` 在 `x` 为 null 时会抛**；**`Equals(x, y)`**（`object` 上的静态方法）或 **`string.Equals(a, b)`** 能安全处理 null。在选用形式前要知道哪一侧可能为 null。

* **引用相等与值相等。** 两个字符相同的不同字符串，`==` 的引用相等可能成立也可能不成立；`string` 的 `==` 按值比较。对你自己的类型，行为取决于是否重写 `Equals`/`GetHashCode` 以及如何使用 `==`（未重载则为引用相等）。

```csharp
// --- Null 安全：哪一侧允许为 null？ ---
string? a = null;
string b = "hi";

// a.Equals(b);                 // NullReferenceException — 在 null 上调用实例方法
bool ok1 = b.Equals(a);        // false；`b` 非 null，调用安全
bool ok2 = object.Equals(a, b); // false；静态辅助，任一侧可为 null
bool ok3 = string.Equals(a, b); // false；字符串同理
bool ok4 = a == b;             // false；string 的 `==` 允许左侧为 null

// --- 文本相同、对象不同：值相等，引用不等 ---
string s1 = new string(new[] { 'a', 'b', 'c' });
string s2 = new string(new[] { 'a', 'b', 'c' });
bool sameChars = s1 == s2;                 // true — string 按内容重载 `==`
bool sameInstance = ReferenceEquals(s1, s2); // false — 两个堆对象

// 类型为 `object`：`==` 仅为引用相等（不应用 string 重载）
object o1 = s1;
object o2 = s2;
bool objectOpEquals = o1 == o2;            // false — 引用不同
bool objectEquals = o1.Equals(o2);       // true — 运行期类型仍是 `string`

// 无 `==` 重载的另一种引用类型：形状相同，标识不同
var list1 = new List<int> { 1, 2, 3 };
var list2 = new List<int> { 1, 2, 3 };
bool listsSameRef = list1 == list2;      // false — 两个 `List<>` 实例
bool listsSameElements = Enumerable.SequenceEqual(list1, list2); // true — 序列相同
// 对你自己的 `class`，除非重载，否则 `==` 也是引用相等
```

* **事件与委托字段。** 从外部看，**`event`** 不是普通字段：其它类型只能 `+=`/`-=` 订阅，不能像裸委托那样赋值或调用。只有声明类型可以引发它。在另一个类里把事件当普通委托变量用会按设计失败。**[第 2 部分](CSharp_Questions_Part2_CN.md)**（第 11 节）会更详细讨论委托、多播与 `EventHandler` 模式。

---

## 4. 🔁 集合与 LINQ

集合存放数据；**LINQ** 描述如何**遍历**和**筛选**这些数据。常见陷阱是忘记许多 LINQ 操作返回**惰性**的 `IEnumerable` 管道：在**有人拉取**序列之前（例如 `foreach`、`ToList()` 或其它消费运算符）什么都不会执行。在此之前，「查询」是配方，不是快照。这又回到**运行期**：在枚举之前，源或查询内部使用的变量都可能改变，枚举两次可能让整个管道跑**两遍**。

例如：

```csharp
int b = 3;
var query = list.Where(a => a > b); // 定义查询，但不执行

b = 1; // b 在定义查询之后、枚举之前改变
var results = query.ToList(); // 过滤时用的是 b == 1，不是 3！
```

此例中，过滤器使用的 `b` 不是在创建查询时「捕获」的，而是在**枚举时**捕获的。这是延迟执行的经典演示；若你期望查询是定义时数据与变量的快照，会感到意外。

**会抛异常的运算符**编码了严格预期。**`First()`** 要求至少一个元素；**`FirstOrDefault()`** 在序列为空时返回 `default(T)`——根据「没有结果」是常态还是异常来选。**`Single()`** 坚持**恰好一个**元素（零个或多个都会抛）。**`SingleOrDefault()`** 允许**零或一**（空时返回 `default(T)`，两个及以上匹配则抛）。因此当序列本就不保证唯一时，`Single*` 是差选择。**`Last()`**、**`ElementAt(index)`** 等常有 `*OrDefault` 变体——同理：位置或唯一性规则失败时是抛还是返回 default。

```csharp
using System.Linq;

var empty = Array.Empty<string>();
// empty.First();              // InvalidOperationException — 序列为空
var a = empty.FirstOrDefault(); // null；对 `int` 序列，空 → `0`（易与真实零混淆）

var one = new[] { "only" };
var b = one.Single();           // "only"
// one.SingleOrDefault();      // 同上 — 仍恰好一个元素

var none = Array.Empty<string>();
// none.Single();              // InvalidOperationException — 零个元素
var c = none.SingleOrDefault(); // null — 允许零个元素

var two = new[] { "x", "y" };
// two.Single();               // InvalidOperationException — 多于一个
// two.SingleOrDefault();      // 同上 — 仍多于一个

var nums = new[] { 10, 20, 30 };
// nums.ElementAt(10);        // ArgumentOutOfRangeException
var d = nums.ElementAtOrDefault(10); // 0 — `int` 的 default，对值类型并非「缺失」语义

// nums.Last();               // 30
// Array.Empty<int>().Last(); // InvalidOperationException
```

**LINQ 内部的相等性**遵循类型的规则，而非你希望它表示的含义。**`Distinct()`** 使用元素类型的默认相等（对引用类型，通常指若重写则使用 **`Equals`/`GetHashCode`**，否则为引用同一性）。两个不同对象实例即便「业务字段」相同仍可能算不同。若需要值语义，应重写相等性、用投影（`Select`）成可比较的东西，或传入 **`IEqualityComparer<T>`**。

```csharp
class Person
{
    public string Name;
    public int Age;

    // public override bool Equals(object obj)
    //    => obj is Person p && p.Name == Name && p.Age == Age;

    // public override int GetHashCode() => HashCode.Combine(Name, Age);
}

var list = new List<Person>
{
    new Person { Name = "Alice", Age = 25 },
    new Person { Name = "Alice", Age = 25 }
};

var distinct = list.Distinct();
Console.WriteLine(distinct.Count()); // 2 ❌
```

**字典**将键映射到值。**`Add(key, value)`** 在键已存在时抛异常；**索引器** `dict[key] = value` 在键存在时**替换**值——面试中容易混淆。**`TryGetValue`** 一次查找并告知是否找到键；调用两次索引器或 `ContainsKey` 再加索引器多做工作，且若另一线程修改字典仍可能竞态（并发场景应换用其它类型）。**Null 键**取决于字典：**`Dictionary<TKey,TValue>`** 在 `TKey` 为不可空引用类型时不允许 null 键；可空键时 `null` 可能被允许，也可能是刻意的「缺键」味道——要清楚你的契约。

面试官在集合与 LINQ 上常探的点：

* **抛与不抛的查询。** 「无匹配」何时是错误（`First`） versus 正常结果（`FirstOrDefault`）？唯一性何时是真实不变量（`Single`） versus 在真实数据上会被打破的假设？

* **延迟执行。** 链式 `Where`/`Select` 构成管道；lambda 中的副作用在**每次**枚举时都会执行。**`ToArray()` / `ToList()`** 在那一刻**一次性**复制结果——之后列表不会「跟随」源上的后续变化，除非重建。

* **多次枚举。** 保存 `IEnumerable<T>` 并枚举两次可能意味着**两遍完整遍历**（以及昂贵逻辑或副作用的**两轮**）。若需要稳定快照，应物化（materialize）。

常见陷阱：

* **`foreach` 与修改。** 迭代时改变集合结构（增删）常抛 **`InvalidOperationException`**（「集合已修改」）。应用副本、谨慎地用索引 `for`，或为并发修改设计的类型。

* **会「移动」的结果。** 若在**可变**列表上构建查询并在列表**改变之后**枚举，看到的是**当前**内容——而非写查询时的内容。需要固定画面时应尽早物化。

* **空与 null。** **null** 集合变量与**空**序列不同；若不先防护，LINQ 扩展方法在 `null` 上会抛。

---

## 5. ⚠️ Null 处理

**`null`** 对引用类型表示「没有对象」，**`Nullable<T>`**（`int?` 等）对值类型表示「可能没有值」。面试题常把这些想法与集合、字符串混在一起，因此本节目标与其它地方相同：分清**运行期**发生什么，**编译器**只能警告什么。

**`NullReferenceException`** 是在 **`null`** 引用上使用成员时得到的——例如你以为引用指向对象，调用方法、读属性或索引，实际上它是 `null`。

```csharp
string s = null;
Console.WriteLine(s.Length); // 抛出 NullReferenceException
```

解决办法不是「捕获所有 NRE」；而是**检测**、使用 **null 条件**访问，或在构造时通过构造函数、工厂、API **保证**不变量（合同说不能返回 null 时就不返回）。

**null 条件运算符 `?.`** 在某一步为 **`null`** 时停止链。其余表达式跳过，当最终结果为引用类型或可空值类型时，整个表达式求值为 **`null`**。**`?.` 链式**（`a?.b?.c`）是深层图避免嵌套 `if` 噪音的简写——记住**短路**：若前一步为 **`null`**，后面不会执行。

```csharp
class Node { public Node? Next; public string? Label; }

Node? head = null;

string? name = head?.Next?.Label; // 求值为 null，不抛异常
```

若不用 null 条件，需要嵌套 `if` 才能避免 `NullReferenceException`。

**`??`（null 合并）** 在左侧为 **`null`** 时取右侧。**`??=`** 仅当左侧为 **`null`** 时才赋右侧——适合字段与局部上的惰性默认值。两者都不会从世界上消除 **`null`**；只在代码该时刻选一个后备。

```csharp
string? name = null;
string display = name ?? "Unknown";  // name 为 null 则为 "Unknown"

string? label = null;
label ??= "Default Label";           // label 现为 "Default Label"
```

**可空引用类型（NRT）** 在引用类型上加 **`?`**（`string?`），并在你可能解引用 **`null`** 时警告。这**不是**运行期特性：若关闭检查、使用旧库或对编译器撒谎，**`null`** 仍可能出现在类型说「不应为 null」的地方。**null 宽容后缀 `!`** 表示「我断言此处非 null」——**消除警告**；**不**插入检查。若你错了，运行期仍会得到 **`NullReferenceException`。**第 2 节**已将 **`string`** / **`string?`** 与字段默认值纳入此图景。

**心智模型（与第 2 节同一故事）。** 对**值类型**，「缺失」是 **`T?`** / **`Nullable<T>`** 与 **`HasValue`**。对**引用类型**，在设计排除之前 **`null`** 始终是可能的运行期值；NRT 用来**尽早发现错误**，而不是从 CLR 抹去 **`null`**。

面试官在 null 处理上常探的点：

* **异常实际抛在何处。** 一长串调用：哪次解引用是 **`null`**？能否用 **`?.`** 重写，或拆成局部让下一位读者（和调试器）看清？

* **「无」与「空」。** **`null`**、**空字符串**、**空集合**回答不同问题；API 应说明用哪一种，调用方不应混用。

* **NRT 纪律。** 何时 **`!`** 合理（在你真正信任的守卫之后）versus 掩盖警告？返回类型何时应为 **`string?`** 以迫使调用方思考？

常见陷阱：

* **`list == null` 与空列表。** **`Count == 0`** 不能说明变量非 null；先检查 **`null`**，或用 **null 条件**（`list?.Count`）并同时处理「无列表」与「无项」。

* **`string.IsNullOrEmpty` 与 `IsNullOrWhiteSpace`。** 当业务规则需要时，一次调用处理 **`null`**、**`""`** 与（空白）**`"   "`**——不要自己重实现而漏掉边界。

* **以为 NRT 在运行期保证安全。** 警告只与标注和你分析过的路径一样好；互操作、序列化或反射仍可能产生 **`null`**。

---

## 6. 🔥 异常处理

异常用于**异常**控制流：你不希望出现在主路径上的失败。在面试与生产环境中，反复出现的主题是：是否**保留信息**（尤其是**堆栈跟踪**）、是否**捕获了正确的类型**、以及 **`finally`** 与**取消**是否仍按读者预期工作。

**`throw;` 与 `throw ex;`。**  
在 **`catch`** 内，裸 **`throw;`** 重抛**同一**异常对象并**保留原始堆栈**——若不是在转换错误，这通常是你想要的。**`throw ex;`**（或粗心的 **`throw new Exception(..., ex)`**）常会**重置**堆栈到当前帧，使生产日志难读。仅当你**增加上下文**并用 **`InnerException`**（或等价物）**链接**时，才用**新**异常类型包装，以免丢失旧堆栈。

```csharp
try
{
    throw new InvalidOperationException("Original error.");
}
catch (Exception ex)
{
    // 重抛同一异常；保留堆栈：
    throw;
    // 若写成：
    // throw ex; 
    // ...堆栈将从本行开始，而不是上面的原始 throw
}

// 或：新异常但带内部异常以保留上下文：
try
{
    throw new InvalidOperationException("Original error.");
}
catch (Exception ex)
{
    throw new ApplicationException("Extra context here.", ex); // 链接，堆栈保留在 'ex' 中
}
```

---

**`catch` 顺序与 `catch when`。**  
运行期按**自上而下**匹配 **`catch`**；**第一个兼容类型**获胜。把**更具体**的类型放在 **`catch (Exception)`** 之前，否则通用处理会吞掉你本想区别对待的情况。**`catch (Exception ex) when (condition)`** 在**不改变**捕获类型的情况下过滤——仍捕获 **`Exception`**，但仅当 **`condition`** 为真。这避免了只想处理子集时别扭的 **`catch`** / 重抛模式。

```csharp
try
{
    // 可能抛出任意异常的代码
}
catch (ArgumentNullException ex) // 具体类型在前
{
    Console.WriteLine("Caught ArgumentNullException: " + ex.Message);
}
catch (Exception ex) // 更通用的处理
{
    Console.WriteLine("Caught some other exception: " + ex.Message);
}

// 使用 'catch when' 过滤器
try
{
    // 可能抛出的代码
}
catch (Exception ex) when (ex.Message.Contains("transient")) // 仅当消息匹配
{
    Console.WriteLine("Transient error, try again maybe.");
}
```

---

**`finally` 与控制流。**  
**`finally`** 在离开关联 **`try`** 时**会执行**——无论是**正常结束**、**`return`**、**`break`**、**`continue`**，还是**抛出**异常（除非进程先崩溃）。若 **`try`** 含 **`return`**，**`finally`** 仍在方法**实际返回**给调用方**之前**执行；**`finally`** 中的副作用仍可能在选定返回值**之后**运行——在争论测验输出前要先懂语言规则。

```csharp
int TestFinally()
{
    try
    {
        return 42; // 'finally' 仍会执行
    }
    finally
    {
        Console.WriteLine("Finally block runs, even after return.");
    }
} // Console 在方法返回前输出

void TestBreakContinue()
{
    for (int i = 0; i < 3; i++)
    {
        try
        {
            if (i == 1) break;
            if (i == 2) continue;
        }
        finally
        {
            Console.WriteLine($"Finally runs for i={i}");
        }
    }
}
// 输出会包含每次循环迭代的 'finally'，即便发生 'break' 或 'continue'
```

---

**设计习惯。**  
用 **`InnerException`** 包装可携带原始原因；裸 **`throw;`** 表示「本层无可补充」。许多系统在**边界**（HTTP 中间件、顶层处理）**只记一次日志**，而不是**到处 catch**。**空 `catch`** 与只**吞掉**的 **`catch (Exception)`** 会隐藏 bug——包括取消与 **`OutOfMemoryException`**。**`ExceptionDispatchInfo.Capture(ex).Throw()`**（及相关模式）可在必须先存异常、稍后再抛时**按原始堆栈重抛**——例如跨过某些 **`await`** 边界——而不会像 **`throw ex;`** 那样重写堆栈。

```csharp
// 差：空 catch 吞掉所有异常（会隐藏 bug）
try
{
    DangerousStuff();
}
catch (Exception)
{
    // 捕获并忽略所有异常（包括 OutOfMemory，通常不应如此）
}

// 好：带上下文包装并保留原因
try
{
    DangerousStuff();
}
catch (Exception ex)
{
    throw new ApplicationException("Something failed in DangerousStuff.", ex);
}

// 存储异常后保留堆栈重抛（进阶/async 场景）
using System.Runtime.ExceptionServices;

ExceptionDispatchInfo captured = null;
try
{
    DangerousStuff();
}
catch (Exception ex)
{
    captured = ExceptionDispatchInfo.Capture(ex); // 存储供稍后使用
}

// ... 稍后，可能在 'await' 之后：
if (captured != null)
    captured.Throw(); // 抛出原始异常，保留原始堆栈！
```

---

**取消**在现代 C# 中是正常控制流，不总是「bug」。**`OperationCanceledException`** 与 **`TaskCanceledException`** 常表示「工作被有意停止」。捕获 **`Exception`** 并**忽略**它们会破坏**协作式取消**，除非你**重抛**或过滤。优先使用 **`CancellationToken.ThrowIfCancellationRequested()`** 并在 API 中传递 **token**；**第 7 节**会更深入 **async** 与 token。

```csharp
void CancelMe(CancellationToken token)
{
    // 正确：尊重 token 取消
    token.ThrowIfCancellationRequested();
    // 执行可取消工作...
}

try
{
    var cts = new CancellationTokenSource();
    cts.Cancel();
    CancelMe(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Cancellation was requested and respected.");
}

// 差：吞掉所有异常会破坏取消
try
{
    var cts = new CancellationTokenSource();
    cts.Cancel();
    CancelMe(cts.Token);
}
catch (Exception) // 应对取消过滤或重抛，不应吞掉！
{
    // 取消被隐藏 — 调用方无法得知工作已取消
}
```

---

**资源与释放**属于同一心智桶：**`using`** 对应 **`try` / `finally` / `Dispose()`**，以便 **`try`** 抛错时仍执行清理。

```csharp
using (var stream = new FileStream("myfile.txt", FileMode.Open))
{
    // 使用文件
    // 若此处抛异常，仍在传播错误前调用 Dispose()（关闭文件）
}
// 等价于：
FileStream stream = null;
try
{
    stream = new FileStream("myfile.txt", FileMode.Open);
    // 使用文件
}
finally
{
    if (stream != null)
        stream.Dispose();
}
```

---

面试官在异常处理上常探的点：

* **堆栈完整性。**  
  为何 **`throw;`** 保留堆栈而 **`throw ex;`** 通常不保留？  
  （见上文代码示例。）

* **`finally` 测验题。**
  **`try`** 返回或抛错时，什么按何顺序执行？  
  **`finally`** 能覆盖返回值吗？（不能，但可抛出新异常，取代原返回/异常。）

* **过滤。**  
  **`catch when`** 与捕获后在 **`if`** 里再处理——何时哪种更清晰？
  ```csharp
  // 用 'catch when' 过滤
  try
  {
      PossiblyTransientWork();
  }
  catch (Exception ex) when (IsTransient(ex))
  {
      Console.WriteLine("Handle only transient errors.");
  }

  // 等价但更啰嗦：
  try
  {
      PossiblyTransientWork();
  }
  catch (Exception ex)
  {
      if (IsTransient(ex))
          Console.WriteLine("Handle only transient errors.");
      else
          throw;
  }
  ```


常见陷阱：

* **宽泛 catch 当默认。** 尤其在 **async** 或 **cancel** 周围，**`catch (Exception)`** 什么都不记就继续——会掩盖真实故障。

* **吞掉取消。** 把**取消**当作一等结果；除非该层真正拥有工作生命周期，否则应过滤或重抛。

* **丢掉内部原因。** 无 **`InnerException`** 链接的新包装异常会抹掉调试工具依赖的链。

---

## 7. ⚡ Async / Task 陷阱

**`async` / `await`** 把工作拆到时间轴上：`async` 方法可在等待 I/O 或其它任务时**让出**，而不必在整个等待期间阻塞线程。理解陷阱的关键是**调用方**实际看到什么：返回类型、异常、顺序，以及当你不小心阻塞了代码仍需要的线程时会发生什么。

---

**返回类型：`async void` 与 `async Task`**

*几乎只在事件处理程序中用 `async void`。* 调用方**无法 `await`** `async void` 方法，因此错误进入进程级处理程序——无法回到调用方。对普通 API，应始终返回 `Task` 或 `Task<T>`。`async Task` 方法中的异常会捕获到 `Task` 上，在 `await` 时重抛。
```csharp
// 差：把 async void 当常规 API，调用方无法 await 或干净地 catch
public async void DoWorkAsync_Bad()
{
    await Task.Delay(1000);
    throw new Exception("Error is uncatchable by caller!");
}

// 好：async Task API 让调用方 await 并捕获异常
public async Task DoWorkAsync_Good()
{
    await Task.Delay(1000);
    throw new Exception("Error will be caught by awaiter.");
}

// 典型事件处理模式（仅此处 async void 可接受）
private async void Button_Click(object sender, EventArgs e)
{
    await DoWorkAsync_Good();
}
```

---

**忘记 `await`**

若调用 `async` 方法却忘记 `await`，会得到一个可能仍在运行的 `Task`（**即发即弃**）。异常被推迟，方法的其余部分可能乱序执行。
```csharp
public async Task ExampleAsync()
{
    Task t = DoWorkAsync_Good();  // 忘了 'await'！工作在后台跑

    // 下面这行可能在 DoWorkAsync_Good() 完成（甚至开始）之前就执行
    Console.WriteLine("This might print before work completes!");

    // DoWorkAsync_Good() 的异常可能未被观察，除非稍后 await：
    await t; // 只有在这里异常才会抛到你的代码
}
```

---

**同步阻塞异步与死锁**

对 async 方法调用 `.Result` 或 `.Wait()` 会**阻塞**调用线程，有死锁风险——尤其在 GUI 或 ASP.NET SynchronizationContext 场景。应始终优先直接 await。
```csharp
public async Task DeadlockExample()
{
    // 在控制台应用里可能没事，但在 UI 线程上可能死锁：
    // 不要在 WinForms/WPF/ASP.NET 里这样做！
    Task t = DoWorkAsync_Good();

    // 阻塞当前线程！
    t.Wait();  // 或：var result = t.Result;
    // 若 DoWorkAsync_Good 试图在此上下文上工作，会永远挂起
}

// 修复：一路 await
public async Task NoDeadlockExample()
{
    await DoWorkAsync_Good(); // 从不阻塞线程！
}

// 若不需要回到捕获的上下文，库代码用 ConfigureAwait(false)
public async Task LibraryCode()
{
    await DoWorkAsync_Good().ConfigureAwait(false); // 不会封送回捕获的上下文
}
```

---

**取消**

让 async 取消**协作式**进行：传入 CancellationToken，使用 `ThrowIfCancellationRequested`，并遵守 token。
```csharp
public async Task WorkWithCancelAsync(CancellationToken token)
{
    for (int i = 0; i < 5; i++)
    {
        token.ThrowIfCancellationRequested(); // 若已取消则抛出
        await Task.Delay(500, token);         // 将 token 传给 Delay
    }
}

// 用法：
CancellationTokenSource cts = new CancellationTokenSource();
var task = WorkWithCancelAsync(cts.Token);
cts.Cancel(); // 需要时取消
```

---

**`IAsyncEnumerable<T>`**（异步流）

对异步流使用 `IAsyncEnumerable<T>`：项随时间产生与消费，用 `await foreach`。消费者在迭代时才开始执行。
```csharp
public async IAsyncEnumerable<int> CountAsync()
{
    for (int i = 0; i < 3; i++)
    {
        await Task.Delay(1000); // 假装每项有异步工作
        yield return i;
    }
}

public async Task Consume()
{
    await foreach (var x in CountAsync())
    {
        Console.WriteLine(x); // 打印 0、1、2，中间有延迟
    }
}
```
*延迟执行同样适用：进入 `await foreach` 循环前 CountAsync 不会运行。*


面试官在 async 代码上常探的点：

* **为何默认不要用 `async void`。** 调用方做不到什么？失败去了哪里？

* **死锁故事。** **UI** 处理程序在 **`.Result`** 上阻塞，而内部工作 **`await`** 回 **UI** 线程——为何会挂起？

* **`ConfigureAwait(false)`** — 它减轻什么问题，何时无关紧要或用错工具？

常见陷阱：

* 在**库**或**业务**代码里用 **`async void`**，只因「能编译」。

* 在已坐在**捕获的**上下文上的**请求**或 **UI** 路径上，对 **`.Result` / `.Wait()`** **阻塞**。

* **Token** 在第一层之后就丢了——凡可能长时间运行的**下游**方法在可能的情况下都应接受并**使用**同一 **`CancellationToken`**。

---

## 🏁 结论（第 1 部分结尾）

你已经走过**语言级**面试里占主导的陷阱：**类型与默认值**、**运算符与相等性**、**LINQ**、**null**、**异常**，以及 **`async`/`Task`**。若这些概念在你脑中连成一体——而非孤立背诵——你就为很大一部分 C# 筛选题做好了准备。

**继续阅读 [第 2 部分](CSharp_Questions_Part2_CN.md)**，了解**继承与多态**、**`static`**、**泛型、委托与事件**、**`using` / 释放资源**、**现代 C#** 概览、可选 **SQL** 笔记、**真实世界模式**，以及面谈前最后一遍的**速查表**。

> 「大多数 bug 并非来自复杂代码，而是来自对简单行为的误解」——这句话在第 1 部分之后仍然成立；第 2 部分会加上对象模型与架构层，那些 bug 会在规模上显现。
