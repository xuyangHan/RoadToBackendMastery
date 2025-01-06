# 构建一个电子书阅读器：C# 面向对象(OOP)设计模拟面试

## 1. 引言

欢迎来到我们 C# 面向对象编程（OOP）博客系列的最后一篇！在本系列中，我们通过实际案例探索了 OOP 的基础知识、高级概念以及设计模式。现在是时候将所有内容结合起来，应用于一个真实场景中。

在这篇文章中，我们将解决一个常见的面向对象设计面试问题：设计一个电子书阅读器应用程序。这是一个绝佳的机会，展示如何运用 OOP 原则高效地解决现实世界中的问题。从识别系统需求到用 C# 实现解决方案，我们将逐步引导你完成整个过程。

无论你是准备技术面试，还是想加深对 OOP 的理解，这篇文章都将为你提供宝贵的见解和实用技巧。让我们开始了解问题描述并设计解决方案吧！

---

## 2. 问题描述

你被要求设计一个电子书阅读器应用程序。该应用程序应允许用户管理其数字图书馆、阅读书籍并跟踪其阅读进度。以下是具体需求：

### 功能需求

1. **图书馆管理：**
   - 用户可以将书籍添加到个人图书馆中。
   - 用户可以从图书馆中移除书籍。

2. **阅读功能：**
   - 用户可以从图书馆中选择一本书作为当前阅读书籍。
   - 应用程序应每次显示一页文字内容。
   - 用户可以在页面之间导航（上一页/下一页）。
   - 应保存阅读进度（例如，记住最后阅读的页面）。

### 假设条件

- 初始实现专注于单用户场景（不考虑多用户）。
- 电子书内容为纯文本，无复杂格式（如字体大小等）。

---

## 3. 系统设计

### 高级架构

系统将包含以下三个主要组件：

1. **Library（图书馆管理器）：** 负责管理用户图书馆中的书籍集合。
2. **BookReader（书籍阅读器）：** 处理阅读功能，包括页面导航和书签。
3. **Book（书籍）：** 表示单个电子书及其内容。

### 关键类及其职责

1. **Library（图书馆）：**

   - 管理用户的书籍集合。
   - 方法：
     - `AddBook(Book book)`: 将一本书添加到图书馆中。
     - `RemoveBook(string title)`: 根据标题从图书馆中移除一本书。
     - `GetBook(int bookId)`: 根据 bookId 获取一本书。

2. **BookReader（书籍阅读器）：**

   - 处理当前的阅读会话。
   - 方法：
     - `OpenBook(Book book)`: 设置当前阅读的书籍。
     - `NextPage()`: 跳转到下一页。
     - `PreviousPage()`: 跳转到上一页。
     - `GetCurrentPage()`: 获取当前页面的内容。
     - `BookmarkPage()`: 保存当前页面为书签。

3. **Book（书籍）：**

   - 表示一本电子书。
   - 属性：
     - `Title`: 书籍标题。
     - `BookId`: 书籍 ID。
     - `Author`: 书籍作者。
     - `Content`: 书籍的纯文本内容，按页分割。
     - `Bookmark`: 上次阅读的页码。
   - 方法：
     - `GetPage(int pageNumber)`: 获取指定页码的内容。

### 数据流

1. 用户通过 **Library** 添加或移除书籍。  
2. 当用户选择一本书时，该书会被传递给 **BookReader** 以进行阅读。  
3. **BookReader** 从 **Book** 类中获取内容，并负责管理页面导航和书签功能。  

**示例流程：**

1. 用户通过 `Library.AddBook()` 添加一本新书到图书馆。  
2. 用户选择一本书进行阅读，触发 `BookReader.OpenBook()`。  
3. **BookReader** 使用 `Book.GetPage(1)` 显示第一页内容。  
4. 用户通过 `BookReader.NextPage()` 导航到下一页。  
5. 用户通过 `BookReader.BookmarkPage()` 保存当前页面为书签。  

---

## 4. 用 C# 实现

下面是电子书阅读器应用程序核心功能的分步实现：

### 第一步：定义 `Book` 类

```csharp
public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public List<string> Content { get; set; } // 按页分割的内容
    public int Bookmark { get; set; } // 最后阅读的页码

    public Book(int bookId, string title, string author, List<string> content)
    {
        BookId = bookId;
        Title = title;
        Author = author;
        Content = content;
        Bookmark = 0; // 从第一页开始
    }

    public string GetPage(int pageNumber)
    {
        if (pageNumber < 0 || pageNumber >= Content.Count)
        {
            return "无效的页码。";
        }
        return Content[pageNumber];
    }
}
```

### 第二步：定义 `Library` 类

```csharp
public class Library
{
    private List<Book> Library = new List<Book>();

    public void AddBook(Book book)
    {
        Library.Add(book);
    }

    public void RemoveBook(int bookId)
    {
        Library.RemoveAll(b => b.BookId == bookId);
    }

    public Book GetBook(int bookId)
    {
        return Library.FirstOrDefault(b => b.BookId == bookId);
    }
}
```

### 第三步：定义 `BookReader` 类

```csharp
public class BookReader
{
    private Book ActiveBook;
    private int CurrentPage;

    public void OpenBook(Book book)
    {
        ActiveBook = book;
        CurrentPage = book.Bookmark;
    }

    public string GetCurrentPage()
    {
        return ActiveBook?.GetPage(CurrentPage) ?? "当前没有打开的书籍。";
    }

    public void NextPage()
    {
        if (ActiveBook != null && CurrentPage < ActiveBook.Content.Count - 1)
        {
            CurrentPage++;
        }
    }

    public void PreviousPage()
    {
        if (ActiveBook != null && CurrentPage > 0)
        {
            CurrentPage--;
        }
    }

    public void BookmarkPage()
    {
        if (ActiveBook != null)
        {
            ActiveBook.Bookmark = CurrentPage;
        }
    }
}
```

### 代码解释

1. **Book 类**：
   - 表示一本电子书，包含 id、标题、作者、内容和书签等属性。
   - 使用 `GetPage()` 方法获取特定页的内容。

2. **Library 类**：
   - 管理书籍集合，包含添加、移除和通过 bookId 获取书籍的方法。

3. **BookReader 类**：
   - 管理当前的阅读会话，包括页面导航和书签功能。
   - 与 `Book` 类交互以获取和显示内容。

---

## 5. 数据库设计

为了支持应用程序的功能，系统需要两种不同的存储解决方案：

### **1. 元数据存储（关系型数据库）**

存储关于图书馆、书籍和用户阅读进度的信息。

#### **表设计**

**Library 表（图书馆表）：**

- `LibraryId`（主键）：每个图书馆的唯一标识符。
- `BookIds`（JSON 格式）：包含图书馆中书籍 `BookId` 的 JSON 数组。
- `UserId`：可选字段，为未来的多用户支持预留。

| LibraryId | BookIds              | UserId |
|-----------|----------------------|--------|
| 1         | [101, 102, 103]     | null   |
| 2         | [104, 105]          | null   |

**Book 表（书籍表）：**

- `BookId`（主键）：每本书的唯一标识符。
- `Title`：书籍标题。
- `Author`：书籍作者。
- `Bookmark`：最后阅读的页码（阅读进度）。

| BookId | Title          | Author         | Bookmark |
|--------|----------------|----------------|----------|
| 101    | "C# 基础"     | "John Smith"  | 5        |
| 102    | "OOP 设计"    | "Jane Doe"    | 12       |
| 103    | "系统设计"     | "Alex White"  | 0        |

### **2. 内容存储（文件系统或对象存储）**

存储书籍的实际内容。每本书的内容按页分割并保存在文件系统或对象存储（如 AWS S3、Azure Blob Storage）中。

#### 文件命名规则

- 文件使用 `BookId` 命名以便于检索。
- 示例：`101.txt`，`102.txt`

**目录结构：**

```plaintext
/content_storage/
    101.txt
    102.txt
    103.txt
```

### **工作流程**

1. 当用户将一本书添加到图书馆时，该书的 `BookId` 会被追加到 `Library` 表中的 `BookIds` JSON 数组中。
2. 当用户打开一本书时，系统从 `Book` 表中检索其元数据，并从文件系统中获取其内容文件。
3. 阅读进度（如书签）会更新到 `Book` 表中。

为了与数据库交互，我们需要定义 **数据访问对象（DAO）** 类来处理数据的保存和查询操作。DAO 层将包含将关系数据映射为对象以及对象映射回关系数据的方法。

---

## 6. 总结

### 关键要点

- 理解并应用面向对象的原则对设计可扩展和可维护的系统至关重要。
- 使用单例模式（Singleton）、工厂模式（Factory）和观察者模式（Observer）等设计模式可以显著提升应用程序的设计质量。
- 数据库设计和 DAO 类在高效管理持久化数据方面起着关键作用。

### 应对类似面试问题的技巧

- 将问题分解为较小、可管理的部分。
- 专注于需求，确保您的解决方案满足需求。
- 使用 UML 图或伪代码清晰地传达您的思路。
- 强调设计模式的使用并解释您的设计决策。

通过这种方法，您将能够更好地应对技术面试中的面向对象设计问题。
