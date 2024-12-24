# 从C#触发Python代码：实用指南

在软件开发的世界中，利用多种编程语言可以显著提升生产力和功能性。结合C#和Python的优势就是一个强大的例子，它允许开发者利用C#在应用开发中的强大功能，同时充分发挥Python的简洁性和广泛的库生态系统。本文将探讨从C#触发Python代码的各种方法，重点介绍每种方法的优点和应用场景。

---

## 使用Process运行Python脚本

从C#执行Python代码最简单的方法之一是创建一个Python脚本文件，并使用`System.Diagnostics.Process`类运行它。这种方法简单直接，当你需要利用Python的广泛库来完成数据可视化、机器学习或其他计算任务时，它可以非常强大。

**详细示例 - 生成Bokeh可视化图:**

这个详细示例演示了如何使用Bokeh将Python可视化集成到C#应用程序中。通过创建一个包含添加不同类型元素到图表的方法的类，用户可以看到这种方法的实际应用和好处。

```csharp
using System;
using System.Diagnostics;
using System.IO;
using System.Text;

public class BokehPlot
{
    private StringBuilder script;
    private string filePath;

    public BokehPlot(string filePath)
    {
        this.filePath = filePath;
        script = new StringBuilder();
        script.AppendLine("from bokeh.plotting import figure, output_file, show");
        script.AppendLine($"output_file('{filePath}')");
        script.AppendLine("p = figure(title='Interactive Bokeh Plot')");
    }

    public void AddPoints((double, double)[] points, string layerName, string color)
    {
        script.AppendLine($"p.scatter(x={string.Join(",", points.Select(p => p.Item1))}, y={string.Join(",", points.Select(p => p.Item2))}, legend_label='{layerName}', fill_color='{color}', size=8)");
    }

    public void AddLine((double, double)[] points, string layerName, string color)
    {
        script.AppendLine($"p.line(x={string.Join(",", points.Select(p => p.Item1))}, y={string.Join(",", points.Select(p => p.Item2))}, legend_label='{layerName}', line_color='{color}', line_width=2)");
    }

    public void AddPolygon((double, double)[] points, string layerName, string color)
    {
        script.AppendLine($"p.patch(x={string.Join(",", points.Select(p => p.Item1))}, y={string.Join(",", points.Select(p => p.Item2))}, legend_label='{layerName}', fill_color='{color}', fill_alpha=0.6)");
    }

    public void AddAxis(string axisType, double[] positions, string layerName)
    {
        script.AppendLine($"p.{axisType}_axis.axis_label = '{layerName}'");
        script.AppendLine($"p.{axisType}_axis.ticker = {string.Join(",", positions)}");
    }

    public void Show()
    {
        script.AppendLine("show(p)");
        string scriptPath = "bokeh_script.py";
        File.WriteAllText(scriptPath, script.ToString());

        ProcessStartInfo start = new ProcessStartInfo
        {
            FileName = "python",
            Arguments = scriptPath,
            UseShellExecute = false,
            RedirectStandardOutput = true,
            RedirectStandardError = true,
            CreateNoWindow = true
        };

        using (Process process = Process.Start(start))
        {
            using (StreamReader reader = process.StandardOutput)
            {
                string result = reader.ReadToEnd();
                Console.WriteLine(result);
            }
        }
    }
}

// 示例用法
class Program
{
    static void Main()
    {
        BokehPlot bokeh = new BokehPlot(filePath: "C:/myplots/myplot");

        var pointsA = new (double, double)[] { (1, 2), (2, 4), (3, 6) };
        var pointsB = new (double, double)[] { (1, 3), (2, 6), (3, 9) };
        var line = new (double, double)[] { (0, 0), (4, 8) };
        var polygon = new (double, double)[] { (1, 1), (1, 4), (4, 4), (4, 1) };

        bokeh.AddPoints(pointsA, layerName: "points-A", color: "red");
        bokeh.AddPoints(pointsB, layerName: "points-B", color: "blue");
        bokeh.AddLine(line, layerName: "line", color: "green");
        bokeh.AddPolygon(polygon, layerName: "polygon", color: "yellow");

        bokeh.AddAxis("x", new double[] { 0, 1, 2, 3, 4 }, "X Axis");
        bokeh.AddAxis("y", new double[] { 0, 2, 4, 6, 8 }, "Y Axis");

        bokeh.Show();
    }
}
```

**优点：**

- 实现简单。只需要确保系统中安装了Python和所需的包。
- 适用于小型脚本和任务。

**缺点：**

- 对执行环境的控制有限。
- 不适合实时或高度交互的应用程序。

---

## Python.NET (pythonnet)

Python.NET (pythonnet) 是一个强大的工具，允许C#程序直接调用Python代码并使用Python库。它桥接了C#和Python环境，使两种语言之间的交互变得无缝。这种方法特别适用于需要利用Python庞大库生态系统或执行更适合Python的任务的情况。

**详细示例 - 从C#调用Python方法:**

这个详细示例演示了如何使用Python.NET从C#应用程序中调用Python方法。通过创建一个简单的Python类和方法，然后从C#中导入和调用它，用户可以看到这种方法的实际应用和好处。

```python
# pythonExample.py

class Calculator:
    def AddInPython(self, a, b):
        return a + b
```

```csharp
using System;
using Python.Runtime;

public class PythonNetExample
{
    public void ExecutePythonCode()
    {
        // 初始化Python引擎
        PythonEngine.Initialize();

        using (Py.GIL())
        {
            // 导入Python文件
            dynamic calculatorModule = Py.Import("pythonExample");

            // 创建Calculator类的实例
            dynamic calculator = calculatorModule.Calculator();

            // 调用AddInPython方法
            int result = calculator.AddInPython(5, 10);
            Console.WriteLine($"在Python中加法的结果: {result}");
        }

        // 关闭Python引擎
        PythonEngine.Shutdown();
    }
}

// 示例用法
class Program
{
    static void Main()
    {
        PythonNetExample example = new PythonNetExample();
        example.ExecutePythonCode();
    }
}
```

**环境设置：**

- 确保系统中安装了Python和Python.NET。
- 通过Visual Studio的NuGet包管理器或使用.NET CLI安装Python.NET：

  ```bash
  dotnet add package Python.Runtime
  ```

**项目设置：**

- 确保Python文件`pythonExample.py`在C#项目可以访问的目录中，或修改C#代码中的`sys.path`以包含包含Python文件的目录。

```csharp
using (Py.GIL())
{
    dynamic sys = Py.Import("sys");
    sys.path.append("/path/to/your/python/scripts");
    dynamic calculatorModule = Py.Import("pythonExample");
    // 其余代码
}
```

**优点：**

- 与Python代码的直接集成。
- 访问Python库和函数。

**缺点：**

- 需要Python运行时的支持。
- 可能会导致调试复杂。

---

## IronPython

IronPython是运行在.NET框架上的Python实现，允许与C#直接集成。它允许你执行Python代码、使用Python库，并从C#应用程序中与Python对象交互。这种方法特别适用于希望在不离开.NET生态系统的情况下利用Python功能的开发者。

**详细示例 - 从C#调用Python方法:**

在此示例中，我们将演示如何使用IronPython从C#调用Python方法。我们将创建一个包含方法`AddInPython(int, int)`的Python类，然后从C#应用程序中调用这个方法。

```python
# pythonExample.py

class Calculator:
    def AddInPython(self, a, b):
        return a + b
```

```csharp
using System;
using IronPython.Hosting;
using Microsoft.Scripting.Hosting;

public class IronPythonExample
{
    public void ExecutePythonCode()
    {
        // 创建IronPython脚本引擎
        ScriptEngine engine = Python.CreateEngine();

        // 将包含Python文件的目录添加到搜索路径
        ICollection<string> searchPaths = engine.GetSearchPaths();
        searchPaths.Add("path_to_your_python_file_directory");
        engine.SetSearchPaths(searchPaths);

        // 导入Python文件
        dynamic calculatorModule = engine.ImportModule("pythonExample");

        // 创建Calculator类的实例
        dynamic calculator = calculatorModule.Calculator();

        // 调用AddInPython方法
        int result = calculator.AddInPython(5, 10);
        Console.WriteLine($"在Python中加法的结果: {result}");
    }
}

// 示例用法
class Program
{
    static void Main()
    {
        IronPythonExample example = new IronPythonExample();
        example.ExecutePythonCode();
    }
}
```

**环境设置：**

- 确保`pythonExample.py`文件保存在指定目录中。
- 通过Visual Studio的NuGet包管理器或使用.NET CLI安装IronPython。

**优点：**

- 与.NET应用程序的无缝集成。
- 不需要单独安装Python。

**缺点：**

- 仅限于IronPython支持的功能。
- 不如其他Python实现活跃维护。

---

## Python服务器（例如: FastAPI）

托管Python服务器允许更复杂的交互和可扩展性。你可以使用像FastAPI这样的框架创建一个服务器，并通过HTTP请求从C#进行通信。

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/add")
def add(a: int, b: int):
    return {"result": a + b}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
```

```csharp
using System.Net.Http;
using System.Threading.Tasks;

public class FastApiExample
{
    private static readonly HttpClient client = new HttpClient();

    public async Task<int> AddNumbers(int a, int b)
    {
        HttpResponseMessage response = await client.GetAsync($"http://127.0.0.1:8000/add?a={a}&b={b}");
        response.EnsureSuccessStatusCode();
        string responseBody = await response.Content.ReadAsStringAsync();
        // 假设响应格式为: {"result": result}
        dynamic result = Newtonsoft.Json.JsonConvert.DeserializeObject(responseBody);
        return result.result;
    }
}

// 示例用法
FastApiExample example = new FastApiExample();
int result = await example.AddNumbers(3, 5);
Console.WriteLine($"结果: {result}");
```

**优点：**

- 可扩展，适用于复杂的交互。
- 支持异步和分布式处理。

**缺点：**

- 需要额外的服务器设置和维护。
- 由于网络通信可能导致延迟。

---

## 结论

在本文中，我们探讨了从C#触发Python代码的几种方法：

1. 使用Process运行Python脚本。
2. 使用Python.NET进行直接集成。
3. 在.NET中利用IronPython。
4. 使用像FastAPI这样的框架托管Python服务器。

每种方法都有其优缺点，选择哪种方法取决于你项目的具体需求和约束条件。无论你需要一种简单的方式来执行Python脚本、直接与Python代码集成，还是一种留在.NET生态系统中的解决方案，总有一种方法适合你的需求。

通过利用C#和Python的优势，你可以创建强大、灵活且高效的应用程序，充分发挥每种语言的最佳特性。
