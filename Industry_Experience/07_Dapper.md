# Efficient Database Connections: Best Practices for Using Dapper in .NET

## **Introduction**

In .NET development, efficient database connections are crucial for building scalable applications. While there are various ORMs (Object-Relational Mappers) available, Dapper stands out due to its simplicity, performance, and flexibility. In this article, we will explore how Dapper simplifies database interaction within the .NET framework. Whether you're an experienced developer looking to optimize database access, or a beginner curious about the latest tools, this guide will walk you through the process step by step.

---

## **Steps**

1. **Install Necessary Packages**:
    - Open your .NET project in Visual Studio.
    - Right-click your project in the Solution Explorer and select "Manage NuGet Packages".
    - Search for "Dapper" and install the latest version.
    - If you're using MySQL, install the "MySql.Data" package for MySQL Connector as well.

2. **Create a Namespace for Dapper**:
    - Create a new folder or namespace in your project called "Repositories" to organize your database-related code. This will be where you create classes and methods to interact with the database using Dapper.
    - Here's an example setup for a repository that interacts with a MySQL database using Dapper. Below is a sample class structure:

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

3. **Configure and Set Up Credentials**:
    - Update the database connection string using your project’s configuration files (such as `app.json` or `web.config`).

    Example:

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

4. **Create Queries and Import Dapper in Your Project**:
    - At the top of your C# code file where you will write database queries, import the necessary namespaces.
    - In your query classes, inherit from the `DbQuery` class. This will allow you to use the database connection methods and utilities provided by `DbQuery`.
    - You can now write SQL queries in your query methods to interact with the database. Dapper provides methods to execute these queries and map the results to objects.

    Example:

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

## **Conclusion**

In this guide, we explored how to use Dapper to simplify database interactions within the .NET framework, offering an efficient and flexible solution for building applications. By leveraging Dapper’s lightweight and high-performance ORM features, developers can streamline database connections, improve application performance, and accelerate development cycles. From setting up Dapper in .NET projects to easily performing CRUD operations, we’ve covered the basic steps to kick-start efficient database connections. Whether you’re using MySQL, SQL Server, PostgreSQL, or other databases supported by Dapper, the principles remain the same, providing a consistent and intuitive experience across different data sources. As you continue exploring Dapper and database connections in .NET, remember to keep experimenting, learning, and refining your skills. Stay curious, stay innovative, and embrace the power of Dapper to unlock new possibilities in your development projects. With Dapper’s support, you can tackle database connection challenges with confidence and efficiency. Happy coding!
