# 利用Unix时间简化时区处理：C# 实战经验

## 介绍

处理时间戳和不同的时区可能是软件开发中的一个棘手问题。尤其是当系统不确定给定的日期时间是UTC还是本地时间时，通常会遇到与时间转换相关的问题。在这篇文章中，我将分享我们如何在C#应用程序中使用Unix时间来简化时间管理的经验。

### 问题

在我们的工作中，经常遇到系统对给定的日期时间是UTC还是本地时间感到困惑，从而导致几个小时的时间差异。这在不同时间格式和时区之间进行转换时尤其问题严重。

### 我们的解决方案

为了解决这些问题，我们决定使用Unix时间（以秒或毫秒为单位）来替换数据库中的日期时间或字符串表示。Unix时间是一种定义明确的、与时区无关的时间表示方法，这使得时间处理和转换变得更容易。

## 实现Timestamp类

在我们的C#代码中，我们创建了一个封装Unix时间（以毫秒为单位）的`Timestamp`类。这个类提供了各种方法来转换为其他时间格式或从其他时间格式转换，确保所有时间操作的一致性和正确性。

### Timestamp类定义

以下是我们定义的`Timestamp`类：

```csharp
public class Timestamp
{
    public double UnixTime_ms { get; set; } = 0.0;
    
    public static DateTime UnixEpoch = new DateTime(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc);

    public static Timestamp CurrentTimestamp()
    {
        return FromDateTimeUTC(DateTime.UtcNow);
    }

    public static Timestamp FromUnixTimeSec(double unix_s)
    {
        return new Timestamp()
        {
            UnixTime_ms = unix_s * 1000.0,
        };
    }

    public static Timestamp FromUnixTimeMilliSec(double unix_ms)
    {
        return new Timestamp()
        {
            UnixTime_ms = unix_ms,
        };
    }

    public static Timestamp FromDateTimeUTC(DateTime dateTime)
    {
        if (dateTime.Kind == DateTimeKind.Unspecified)
        {
            dateTime = DateTime.SpecifyKind(dateTime, DateTimeKind.Utc);
        }

        return new Timestamp()
        {
            UnixTime_ms = (dateTime - UnixEpoch).TotalMilliseconds
        };
    }

    public static Timestamp FromTimestampUTC(string timestampUTC)
    {
        bool tryDateTime = DateTime.TryParse(timestampUTC, out DateTime res);
        if (tryDateTime)
        {
            long unixMs = new DateTimeOffset(res.ToUniversalTime()).ToUnixTimeMilliseconds();
            return new Timestamp()
            {
                UnixTime_ms = unixMs,
            };
        }
        else
        {
            return new Timestamp();
        }
    }

    public override bool Equals(object obj)
    {
        return obj is Timestamp timestamp &&
               UnixTime_ms == timestamp.UnixTime_ms;
    }

    public override int GetHashCode()
    {
        return -262018729 + UnixTime_ms.GetHashCode();
    }

    public static bool operator ==(Timestamp left, Timestamp right)
    {
        return EqualityComparer<Timestamp>.Default.Equals(left, right);
    }

    public static bool operator !=(Timestamp left, Timestamp right)
    {
        return !(left == right);
    }

    public static bool operator >(Timestamp left, Timestamp right)
    {
        return left.UnixTime_ms > right.UnixTime_ms;
    }

    public static bool operator <(Timestamp left, Timestamp right)
    {
        return left.UnixTime_ms < right.UnixTime_ms;
    }

    public static bool operator >=(Timestamp left, Timestamp right)
    {
        return left.UnixTime_ms >= right.UnixTime_ms;
    }

    public static bool operator <=(Timestamp left, Timestamp right)
    {
        return left.UnixTime_ms <= right.UnixTime_ms;
    }
}
```

### 转换的扩展方法

我们还创建了扩展方法，用于在不同时间格式之间进行转换：

```csharp
public static class TimestampExtensions
{
    private static readonly DateTime UnixEpoch = new DateTime(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc);

    public static double ToUnixS(this Timestamp timestamp) => timestamp.UnixTime_ms / 1000.0;

    public static double ToUnixMins(this Timestamp timestamp) => timestamp.UnixTime_ms / 60000.0;

    public static DateTime ToDateTimeUTC(this Timestamp timestamp) => UnixEpoch.AddMilliseconds(timestamp.UnixTime_ms).ToUniversalTime();

    public static string ToLabel(this Timestamp timestamp) => timestamp.ToDateTimeUTC().ToString("yyyy/MM/dd h:mm:ss tt");

    public static bool IsWithinRange(this Timestamp timestamp, Timestamp startTime, Timestamp endTime)
    {
        if (timestamp == null || startTime == null || endTime == null)
        {
            return false;
        }

        return timestamp.UnixTime_ms >= startTime.UnixTime_ms
            && timestamp.UnixTime_ms <= endTime.UnixTime_ms;   
    }

    // 其他扩展方法...
}
```

## 在主逻辑中使用Timestamp类

在我们的主应用程序逻辑中，我们专门使用`Timestamp`类来表示时间。这种方法帮助我们避免了与时间转换相关的任何问题，因为所有的时间相关数据都以一致的格式存储和处理。

以下是我们在代码中使用`Timestamp`类的示例：

```csharp
public class ExampleService
{
    public void ProcessData()
    {
        // 获取当前时间作为Timestamp
        Timestamp currentTime = Timestamp.CurrentTimestamp();

        // 将Timestamp转换为DateTime以便显示
        DateTime displayTime = currentTime.ToDateTimeUTC();

        // 记录当前时间
        Console.WriteLine($"当前时间: {currentTime.ToLabel()}");
    }
}
```

## 在数据库视图(View)中转换Unix时间

为了方便读懂数据库中Unix时间，可以创建处理这些转换的视图（View）。以下是在SQL Server中的示例：

```sql
CREATE VIEW dbo.ExampleView AS
SELECT
    Id,
    Name,
    Description,
    DATEADD(MILLISECOND, UnixTime_ms, '1970-01-01 00:00:00') AS DateTimeValue
FROM
    dbo.ExampleTable;
```

## 在C#数据访问层（DAO）中映射转换

在查询数据并将其映射到对象时，可以在数据访问层中处理这些转换：

```csharp
public class ExampleRepository
{
    private readonly string connectionString;

    public ExampleRepository(string connectionString)
    {
        this.connectionString = connectionString;
    }

    public ExampleModel GetExampleById(int id)
    {
        using (var connection = new SqlConnection(connectionString))
        {
            string query = "SELECT Id, Name, Description, UnixTime_ms FROM ExampleTable WHERE Id = @Id";
            var exampleData = connection.QuerySingleOrDefault(query, new { Id = id });

            if (exampleData != null)
            {
                return new ExampleModel
                {
                    Id = exampleData.Id,
                    Name = exampleData.Name,
                    Description = exampleData.Description,
                    Timestamp = Timestamp.FromUnixTimeMilliSec(exampleData.UnixTime_ms)
                };
            }

            return null;
        }
    }
}
```

## 这种方法的好处

通过使用Unix时间和专门的`Timestamp`类，我们实现了以下几个好处：

- **一致性**：所有时间数据都以一致的、与时区无关的格式表示。
- **简便性**：封装在`Timestamp`类中的转换方法简化了时间操作。
- **清晰性**：代码中明确显示所有时间数据都以统一的方式处理，从而减少了错误的风险。

## 结论

处理时间戳和时区可能很复杂，但通过使用Unix时间并将其封装在`Timestamp`类中，我们简化了时间管理并避免了许多常见的陷阱。我希望这种方法也能帮助到你的项目。

如果你有任何问题或其他建议，请在评论中分享吧！
