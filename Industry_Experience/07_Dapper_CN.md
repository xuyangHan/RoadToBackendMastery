# 高效数据库连接：在 .NET 中使用 Dapper 的最佳实践

## **介绍**

在 .NET 开发领域，高效的数据库连接对于构建可扩展的应用程序至关重要。虽然有各种各样的 ORM（对象关系映射器）可用，但 Dapper 以其简单性、性能和灵活性脱颖而出。 在本文中，我们将探讨 Dapper 如何简化 .NET 框架内的数据库交互。无论您是一位经验丰富的开发人员，希望优化数据库访问，还是一位对最新工具好奇的新手，本指南都将逐步引导您完成这个过程。

---

## **步骤**

1. **安装相关包**:
    - 在 Visual Studio 中打开您的 .NET 项目。
    - 在解决方案资源管理器中右键单击您的项目，然后选择 "管理 NuGet 包"。
    - 搜索 "Dapper" 并安装最新版本。
    - 如果您使用的是 MySQL，请同时安装 MySQL 连接器的 "MySql.Data" 包。

2. **为 Dapper 创建空间**:
    - 在您的项目中创建一个名为 "Repositories" 的新文件夹或命名空间，以组织与数据库相关的代码。这是您将使用 Dapper 与数据库交互的类和方法所在之处。
    - 我们来看一个使用 Dapper 与 MySQL 数据库交互的存储库的设置示例。以下是一个类的示例结构：

    ```csharp
    using Dapper;
    using MySql.Data.MySqlClient;
    using System;
    using System.Collections.Generic;
    using System.Configuration;

    namespace MyRepository
    {
        //http://stackoverflow.com/questions/23023534/managing-connection-with-non-buffered-queries-in-dapper
        public class DbQuery
        {
            protected readonly string _connectionString;
        
            public readonly MySqlQueries MySql;

            public DbQuery(string database)
            {
                _connectionString = ConfigurationManager.ConnectionStrings[database]?.ConnectionString;

                if (_connectionString == null)
                {
                    _connectionString = DapperConstants.GetConnectionString(database);
                }

                MySql = new MySqlQueries(this);
            }

            public DbQuery() { }

            internal MySqlConnection GetOpenConnection()
            {
                var connection = new MySqlConnection(_connectionString);
                connection.Open();
                return connection;
            }

            internal DynamicParameters ToDynamicParameters(Dictionary<string, object> param)
            {
                DynamicParameters dynamicParams = new DynamicParameters();
                foreach (var keyVal in param)
                {
                    dynamicParams.Add(keyVal.Key, keyVal.Value);
                }

                return dynamicParams;
            }

            public interface IQueries
            {
                int Execute(string sql, object param, int? commandTimeout);
                IEnumerable<T> Query<T>(string sql, object param, int? commandTimeout);
                T QueryFirstOrDefault<T>(string sql, object param, int? commandTimeout);
            }

            public class MySqlQueries : IQueries
            {
                private DbQuery Db;
                public MySqlQueries(DbQuery dBQuery)
                {
                    Db = dBQuery;
                }

                /// <summary>
                /// Execute parameterized SQL.
                /// </summary>
                /// <returns>The number of rows affected.</returns>
                public int Execute(string sql, object param = null, int? commandTimeout = null)
                {
                    object p = param is Dictionary<string, object> dictionary ? Db.ToDynamicParameters(dictionary) : param;

                    return RunInMySQLDbConnection(a=> a.Execute(sql, p, commandTimeout: commandTimeout));
                }

                /// <summary>
                /// Executes a query, returning the data typed as T.
                /// </summary>
                public IEnumerable<T> Query<T>(string sql, object param = null, int? commandTimeout = null)
                {
                    object p = param is Dictionary<string, object> dictionary ? Db.ToDynamicParameters(dictionary) : param;

                    return RunInMySQLDbConnection(a => a.Query<T>(sql, p, commandTimeout: commandTimeout));
                }

                /// <summary>
                /// Executes a query, returning the data typed as T.
                /// </summary>
                public T QuerySingle<T>(string sql, object param = null, int? commandTimeout = null)
                {
                    object p = param is Dictionary<string, object> dictionary ? Db.ToDynamicParameters(dictionary) : param;

                    return RunInMySQLDbConnection(a => a.QuerySingle<T>(sql, p, commandTimeout: commandTimeout));
                }

                /// <summary>
                /// Executes a single-row query, returning the data typed as type.
                /// </summary>            
                public T QueryFirstOrDefault<T>(string sql, object param = null, int? commandTimeout = null)
                {
                    object p = param is Dictionary<string, object> dictionary ? Db.ToDynamicParameters(dictionary) : param;

                    return RunInMySQLDbConnection(a => a.QueryFirstOrDefault<T>(sql, p, commandTimeout: commandTimeout));
                }

                /// <summary>
                /// Perform a multi-mapping query with 2 input types. This returns a single type,
                /// combined from the raw types via map.
                /// </summary>
                public IEnumerable<TReturn> Query<TFirst, TSecond, TReturn>(string sql, Func<TFirst, TSecond, TReturn> map, object param = null, string splitOn = "Id")
                {
                    object p = param is Dictionary<string, object> dictionary ? Db.ToDynamicParameters(dictionary) : param;

                    return RunInMySQLDbConnection(a => a.Query(sql, map, p, splitOn: splitOn));
                }

                /// <summary>
                /// Append all SQL queries into a single string with all of its parameters.
                /// The result will be split based on the supplied Types in the supplied list response.
                /// </summary>            
                public List<IEnumerable<dynamic>> QueryMultiple(string sql, object param, IEnumerable<Type> types)
                {
                    List<IEnumerable<dynamic>> returnObjs = new List<IEnumerable<dynamic>>();

                    object p = param is Dictionary<string, object> dictionary ? Db.ToDynamicParameters(dictionary) : param;

                    using (var connection = Db.GetOpenConnection())
                    {
                        using (var multi = connection.QueryMultiple(sql, p))
                        {
                            foreach (Type t in types)
                            {
                                IEnumerable<dynamic> obj = multi.Read(t);
                                returnObjs.Add(obj);
                            }
                        }
                    }

                    return returnObjs;
                }

                private T RunInMySQLDbConnection<T>(Func<MySqlConnection, T> getData, MySqlConnection connOverride = null)
                {
                    if (connOverride != null)
                    {
                        return getData(connOverride);
                    }
                    else
                    {
                        using (var connection = new MySqlConnection(Db._connectionString))
                        {
                            connection.Open();
                            return getData(connection);
                        }
                    }
                }

                /// <summary>
                /// Dummy type for excluding from multi-map
                /// </summary>
                private class DontMap { }
            }
        }
    }
    ```

3. **配置和凭据设置**:
    - 使用您项目的配置文件（例如 `app.json` 或 `web.config`）更新数据库连接字符串。

    例子:

    ```XML
    <connectionStrings>           
    <add name="MyDatabase" connectionString="Server=myServerAddress;Database=myDataBase;User=myUsername;Password=myPassword;" providerName="MySql.Data.MySqlClient" />        
    </connectionStrings>
    ```

    ```csharp
    using System.Configuration;

    namespace MyRepository
    {
        public static class DapperConstants
        {
        public const string DB_MYSQL = "MyDatabase";
        }
    }

    ```

4. **在项目中创建查询并导入 Dapper**:
    - 在您将编写数据库查询的 C# 代码文件的顶部，导入必要的命名空间。
    - 在您的查询类中，从 `DbQuery` 类继承并进行基类化。这将使您可以使用 `DbQuery` 提供的数据库连接方法和实用程序。
    - 您现在可以在查询方法中编写 SQL 查询，以与数据库交互。Dapper 提供了执行这些查询并将结果映射到对象的方法。

    例子:

    ```csharp
    using MyRepository;
    using System.Collections.Generic;
    using System.Linq;

    namespace RawQueries
    {
        public class MyQueries : DbQuery
        {
            public MyQueries() : base(DapperConstants.DB_MYSQL) 
            {
            }

            public int SaveData(SampleLog log)
            {
                string sql = @"INSERT INTO Logs
                                (JobId, InputFileSize_Kb, ResponseCode, ErrorMessage, ProcessedAt_UnixSec, CompletionTime_Sec)
                                VALUES(@JobId, @InputFileSize_Kb, @ResponseCode, @ErrorMessage, @ProcessedAt_UnixSec, @CompletionTime_Sec);";

                return MySql.Execute(sql, log);
            }

            public List<SampleLog> GetLogs()
            {
                string sql = @"SELECT * FROM Logs
                                ORDER BY ProcessedAt_UnixSec DESC
                                LIMIT 50;";

                return MySql.Query<SampleLog>(sql).ToList();
            }
        }
    }
    ```

---

## **结论**

在本指南中，我们探讨了如何使用 Dapper 在 .NET 框架中简化数据库交互，为构建应用程序提供了高效和灵活的解决方案。通过利用 Dapper 轻量级和高性能的 ORM 功能，开发人员可以简化数据库连接，提高应用程序性能，并加速开发周期。 从在 .NET 项目中设置 Dapper 到轻松执行 CRUD 操作，我们介绍了启动高效数据库连接的基本步骤。无论您是使用 MySQL、SQL Server、PostgreSQL 还是其他 Dapper 支持的数据库，原则都是相同的，为不同数据源提供一致和直观的体验。 在您继续探索 Dapper 和 .NET 中的数据库连接时，请记住继续尝试、学习和完善您的技能。保持好奇心，保持创新，并拥抱 Dapper 的力量，解锁您开发项目中的新可能性。 有了 Dapper 的支持，您可以自信并高效地解决数据库连接的挑战。祝编码愉快！
