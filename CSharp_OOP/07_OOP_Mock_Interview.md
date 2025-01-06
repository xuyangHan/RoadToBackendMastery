# Building an eBook Reader: Mock Interview in C# Object-Oriented Design

## 1. Introduction

Welcome to the final installment of our C# Object-Oriented Programming (OOP) blog series! Throughout this series, we have explored OOP fundamentals, advanced concepts, and design patterns through practical examples. Now, it’s time to bring everything together in a real-world scenario.

In this post, we will tackle a common object-oriented design interview question: designing an eBook reader application. This case study is a perfect opportunity to demonstrate how OOP principles can be applied to solve real-world problems effectively. From identifying system requirements to implementing a solution in C#, we’ll guide you step-by-step through the process.

Whether you’re preparing for a technical interview or seeking to deepen your understanding of OOP, this post will provide valuable insights and practical techniques. Let’s dive into the problem description and start designing!

---

## 2. Problem Description

You have been asked to design an eBook reader application. This application should allow users to manage their digital library, read books, and track their reading progress. Below are the requirements:

### Functional Requirements

1. **Library Management:**
   - Users can add books to their personal library.
   - Users can remove books from their library.

2. **Reading Functionality:**
   - Users can select a book from their library as the active book.
   - The application should display one page of text at a time.
   - Users can navigate between pages (next/previous).
   - The reading progress should persist (e.g., remember the last page read).

### Assumptions

- The initial implementation will focus on a single-user scenario (no multi-user).
- The eBook content will be plain text without complex formatting like font size etc.

---

## 3. System Design

### High-Level Architecture

The system will consist of three main components:

1. **Library:** Responsible for managing the collection of books in the user’s library.
2. **BookReader:** Handles the reading functionality, including navigation and bookmarking.
3. **Book:** Represents the individual eBook and its content.

### Key Classes and Responsibilities

1. **Library:**

   - Manages the user’s collection of books.
   - Methods:
     - `AddBook(Book book)`: Adds a book to the library.
     - `RemoveBook(string title)`: Removes a book by title.
     - `GetBook(int bookId)`: Retrieves a book by bookId.

2. **BookReader:**

   - Handles the active reading session.
   - Methods:
     - `OpenBook(Book book)`: Sets the active book.
     - `NextPage()`: Moves to the next page.
     - `PreviousPage()`: Moves to the previous page.
     - `GetCurrentPage()`: Retrieves the current page content.
     - `BookmarkPage()`: Saves the current page as a bookmark.

3. **Book:**

   - Represents an eBook.
   - Properties:
     - `Title`: Title of the book.
     - `BookId`: Id of the book.
     - `Author`: Author of the book.
     - `Content`: Plain text content of the book split into pages.
     - `Bookmark`: The last-read page number.
   - Methods:
     - `GetPage(int pageNumber)`: Retrieves the content of a specific page.

### Data Flow

1. The user interacts with the **Library** to add or remove books.
2. When a book is selected, it is passed to the **BookReader** for reading.
3. The **BookReader** fetches the content from the **Book** class and manages navigation and bookmarking.

**Example Flow:**

1. The user adds a new book to their library using `Library.AddBook()`.
2. The user selects a book to read, triggering `BookReader.OpenBook()`.
3. The **BookReader** displays the first page using `Book.GetPage(1)`.
4. The user navigates to the next page using `BookReader.NextPage()`.
5. The user bookmarks the current page using `BookReader.BookmarkPage()`.

---

## 4. Implementation in CSharp

Below is the step-by-step implementation of the core features of the eBook reader application:

### Step 1: Define the `Book` Class

```csharp
public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public List<string> Content { get; set; } // Content split into pages
    public int Bookmark { get; set; } // Last-read page

    public Book(int bookId, string title, string author, List<string> content)
    {
        BookId = bookId;
        Title = title;
        Author = author;
        Content = content;
        Bookmark = 0; // Start at the first page
    }

    public string GetPage(int pageNumber)
    {
        if (pageNumber < 0 || pageNumber >= Content.Count)
        {
            return "Invalid page number.";
        }
        return Content[pageNumber];
    }
}
```

### Step 2: Define the `Library` Class

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

### Step 3: Define the `BookReader` Class

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
        return ActiveBook?.GetPage(CurrentPage) ?? "No book is currently open.";
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

### Explanation of the Code

1. **Book Class**:
   - Represents an eBook with properties for the id, title, author, content, and bookmark.
   - Handles fetching a specific page using `GetPage()`.

2. **Library Class**:
   - Manages a collection of books with methods to add, remove, and retrieve books by bookId.

3. **BookReader Class**:
   - Manages the active reading session, including navigation and bookmarking.
   - Interacts with the `Book` class to fetch and display content.

---

## 5. Database Setup

To support the application's functionality, the system requires two separate storage solutions:

### **1. Metadata Storage (Relational Database)**

Stores information about libraries, books, and user progress.

#### **Tables Design**

**Library Table:**

- `LibraryId` (Primary Key): Unique identifier for each library.
- `BookIds` (JSON format): A JSON array containing the `BookId`s of books in the library.
- `UserId`: Optional field for future multi-user support.

| LibraryId | BookIds              | UserId |
|-----------|----------------------|--------|
| 1         | [101, 102, 103]     | null   |
| 2         | [104, 105]          | null   |

**Book Table:**

- `BookId` (Primary Key): Unique identifier for each book.
- `Title`: Title of the book.
- `Author`: Author of the book.
- `Bookmark`: Last-read page number (progress).

| BookId | Title          | Author         | Bookmark |
|--------|----------------|----------------|----------|
| 101    | "C# Basics"   | "John Smith"  | 5        |
| 102    | "OOP Design"  | "Jane Doe"    | 12       |
| 103    | "System Design"| "Alex White" | 0        |

### **2. Content Storage (File System or Object Storage)**

Stores the actual content of the books. Each book's content is divided into pages and saved in the file system or an object storage solution (e.g., AWS S3, Azure Blob Storage).

#### File Naming Convention

- Files are named using the `BookId` for easy retrieval.
- Example: `101.txt`, `102.txt`

**Directory Structure:**

``` plaintext
/content_storage/
    101.txt
    102.txt
    103.txt
```

### **How It Works**

1. When a user adds a book to their library, the `BookId` is appended to the `BookIds` JSON array in the `Library` table.
2. When a user opens a book, the system retrieves its metadata from the `Book` table and fetches the content file from the file system.
3. Reading progress (e.g., bookmarks) is updated in the `Book` table.

To interact with the database, we need to define **Data Access Object (DAO)** classes to handle operations like saving and querying data. The DAO layer will include methods to map relational data into objects and vice versa.

---

## 6. Conclusion

### Key Takeaways

- Understanding and implementing object-oriented principles is critical in designing scalable and maintainable systems.
- Design patterns like Singleton, Factory, and Observer can significantly enhance the design of your application.
- Database design and DAO classes play a pivotal role in managing persistent data efficiently.

### Tips for Tackling Similar Interview Questions

- Break down the problem into smaller, manageable parts.
- Focus on the requirements and ensure your solution meets them.
- Use UML diagrams or pseudocode to communicate your thought process clearly.
- Highlight the use of design patterns and justify your design decisions.

With this approach, you'll be well-prepared to tackle object-oriented design questions in technical interviews.
