# 🗄️ ADO.NET Fundamentals - Complete Guide

## 📚 Table of Contents
- [Overview](#-overview)
- [Connection](#-connection)
- [Command](#-command)
- [DataReader](#-datareader)
- [Parameters and SQL Injection](#-parameters-and-sql-injection)
- [Transactions](#-transactions)
- [Interview Questions](#-interview-questions)

---

## 🎯 Overview

**ADO.NET** is the data access technology for .NET applications to interact with databases.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ADO.NET ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   .NET Application                                                  │
│         │                                                           │
│         ▼                                                           │
│   ┌─────────────────────────────────────────────────────────┐      │
│   │ ADO.NET Components                                       │      │
│   │  • Connection (SqlConnection)                            │      │
│   │  • Command (SqlCommand)                                  │      │
│   │  • DataReader (SqlDataReader) - Forward-only reading    │      │
│   │  • DataAdapter + DataSet - Disconnected data            │      │
│   │  • Parameter (SqlParameter)                              │      │
│   │  • Transaction                                           │      │
│   └─────────────────────────────────────────────────────────┘      │
│         │                                                           │
│         ▼                                                           │
│   Database (SQL Server, MySQL, Oracle, etc.)                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Namespaces

```csharp
// SQL Server
using System.Data.SqlClient;  // .NET Framework
using Microsoft.Data.SqlClient;  // .NET Core/.NET 5+

// MySQL
using MySql.Data.MySqlClient;

// Common interfaces
using System.Data;
```

---

## 🔷 Connection

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// SQLCONNECTION - DATABASE CONNECTION
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Data.SqlClient;

class ConnectionDemo
{
    // Connection string components:
    // Server: Database server name or IP
    // Database: Database name
    // Integrated Security: Windows Auth (true) or SQL Auth (false)
    // User Id/Password: For SQL Server Authentication
    
    private static string _connectionString = 
        @"Server=localhost\SQLEXPRESS;Database=TestDB;Integrated Security=true;";
    
    // Alternative with SQL Authentication
    private static string _sqlAuth = 
        @"Server=192.168.1.100;Database=TestDB;User Id=sa;Password=myPassword;";
    
    static void Main()
    {
        // ─────────────────────────────────────────────────────────────────
        // METHOD 1: Using statement (recommended)
        // ─────────────────────────────────────────────────────────────────
        
        using (SqlConnection connection = new SqlConnection(_connectionString))
        // Line 1: using ensures connection is closed and disposed
        {
            connection.Open();
            // Line 2: Open connection to database
            
            Console.WriteLine($"State: {connection.State}");
            // Output: State: Open
            
            // Perform database operations...
            
        }  // Connection automatically closed here
        
        // ─────────────────────────────────────────────────────────────────
        // METHOD 2: C# 8+ using declaration
        // ─────────────────────────────────────────────────────────────────
        
        using SqlConnection conn = new SqlConnection(_connectionString);
        conn.Open();
        // Connection disposed at end of scope
        
        // ─────────────────────────────────────────────────────────────────
        // CONNECTION PROPERTIES
        // ─────────────────────────────────────────────────────────────────
        
        using (SqlConnection c = new SqlConnection(_connectionString))
        {
            Console.WriteLine($"State: {c.State}");           // Closed
            c.Open();
            Console.WriteLine($"State: {c.State}");           // Open
            Console.WriteLine($"Server: {c.DataSource}");     
            Console.WriteLine($"Database: {c.Database}");     
            Console.WriteLine($"Timeout: {c.ConnectionTimeout}s");
        }
    }
}
```

### Connection String from Configuration

```csharp
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=MyDB;Integrated Security=true;"
  }
}

// Reading in .NET Core
using Microsoft.Extensions.Configuration;

IConfiguration config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .Build();

string connStr = config.GetConnectionString("DefaultConnection");
```

---

## 🔶 Command

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// SQLCOMMAND - EXECUTE SQL QUERIES
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Data.SqlClient;

class CommandDemo
{
    private static string _connStr = @"Server=.\SQLEXPRESS;Database=TestDB;Integrated Security=true;";
    
    static void Main()
    {
        using (SqlConnection conn = new SqlConnection(_connStr))
        {
            conn.Open();
            
            // ─────────────────────────────────────────────────────────────
            // ExecuteNonQuery - INSERT, UPDATE, DELETE (returns rows affected)
            // ─────────────────────────────────────────────────────────────
            
            string insertSql = "INSERT INTO Employees (Name, Salary) VALUES ('John', 50000)";
            
            using (SqlCommand cmd = new SqlCommand(insertSql, conn))
            // Line 1: Create command with SQL and connection
            {
                int rowsAffected = cmd.ExecuteNonQuery();
                // Line 2: ExecuteNonQuery returns number of rows affected
                Console.WriteLine($"Inserted {rowsAffected} row(s)");
            }
            
            // ─────────────────────────────────────────────────────────────
            // ExecuteScalar - Returns single value (first column of first row)
            // ─────────────────────────────────────────────────────────────
            
            string countSql = "SELECT COUNT(*) FROM Employees";
            
            using (SqlCommand cmd = new SqlCommand(countSql, conn))
            {
                object result = cmd.ExecuteScalar();
                // Line 3: Returns first column of first row
                int count = Convert.ToInt32(result);
                Console.WriteLine($"Total employees: {count}");
            }
            
            // Get identity value after insert
            string insertWithIdentity = @"
                INSERT INTO Employees (Name) VALUES ('Jane');
                SELECT SCOPE_IDENTITY();";
            
            using (SqlCommand cmd = new SqlCommand(insertWithIdentity, conn))
            {
                int newId = Convert.ToInt32(cmd.ExecuteScalar());
                Console.WriteLine($"New employee ID: {newId}");
            }
            
            // ─────────────────────────────────────────────────────────────
            // ExecuteReader - Returns multiple rows (see DataReader section)
            // ─────────────────────────────────────────────────────────────
        }
    }
}
```

---

## 🔵 DataReader

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// SQLDATAREADER - READ QUERY RESULTS
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Data.SqlClient;

class DataReaderDemo
{
    static void ReadEmployees()
    {
        string connStr = @"Server=.\SQLEXPRESS;Database=TestDB;Integrated Security=true;";
        
        using (SqlConnection conn = new SqlConnection(connStr))
        {
            conn.Open();
            
            string sql = "SELECT Id, Name, Salary, HireDate FROM Employees";
            
            using (SqlCommand cmd = new SqlCommand(sql, conn))
            using (SqlDataReader reader = cmd.ExecuteReader())
            // Line 1: ExecuteReader returns DataReader for result set
            {
                // ─────────────────────────────────────────────────────────
                // READING ROWS
                // ─────────────────────────────────────────────────────────
                
                while (reader.Read())
                // Line 2: Read() advances to next row, returns false at end
                {
                    // Method 1: By column index
                    int id = reader.GetInt32(0);
                    // Line 3: GetInt32(0) = first column as int
                    
                    string name = reader.GetString(1);
                    // GetString(1) = second column as string
                    
                    decimal salary = reader.GetDecimal(2);
                    
                    // Method 2: By column name (slower but clearer)
                    DateTime hireDate = reader.GetDateTime(reader.GetOrdinal("HireDate"));
                    // GetOrdinal returns column index by name
                    
                    // Method 3: Generic indexer (returns object)
                    object value = reader["Name"];
                    
                    Console.WriteLine($"{id}: {name} - ${salary:N0} - {hireDate:d}");
                }
            }
            
            // ─────────────────────────────────────────────────────────────
            // HANDLING NULL VALUES
            // ─────────────────────────────────────────────────────────────
            
            string sqlWithNull = "SELECT Name, MiddleName FROM Employees";
            using (SqlCommand cmd = new SqlCommand(sqlWithNull, conn))
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    string name = reader.GetString(0);
                    
                    // Check for DBNull before reading
                    string middleName = reader.IsDBNull(1) ? null : reader.GetString(1);
                    // Line 4: IsDBNull checks if column is NULL
                    
                    // Alternative with null coalescing
                    string middle = reader["MiddleName"] as string;
                }
            }
        }
    }
    
    // ─────────────────────────────────────────────────────────────────────
    // MAPPING TO OBJECTS
    // ─────────────────────────────────────────────────────────────────────
    
    static List<Employee> GetAllEmployees()
    {
        var employees = new List<Employee>();
        
        using (var conn = new SqlConnection(_connStr))
        using (var cmd = new SqlCommand("SELECT * FROM Employees", conn))
        {
            conn.Open();
            using (var reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    employees.Add(new Employee
                    {
                        Id = reader.GetInt32(reader.GetOrdinal("Id")),
                        Name = reader.GetString(reader.GetOrdinal("Name")),
                        Salary = reader.GetDecimal(reader.GetOrdinal("Salary"))
                    });
                }
            }
        }
        
        return employees;
    }
    
    private static string _connStr = "";
}

class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Salary { get; set; }
}
```

---

## 🟢 Parameters and SQL Injection

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// PARAMETERIZED QUERIES - PREVENT SQL INJECTION
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Data.SqlClient;

class ParameterDemo
{
    // ❌ DANGEROUS: SQL Injection vulnerability
    static void UnsafeQuery(string userName)
    {
        // If userName = "'; DROP TABLE Users; --"
        string sql = $"SELECT * FROM Users WHERE Name = '{userName}'";
        // Results in: SELECT * FROM Users WHERE Name = ''; DROP TABLE Users; --'
        // TABLE DELETED!
    }
    
    // ✅ SAFE: Parameterized query
    static void SafeQuery(string userName)
    {
        using (var conn = new SqlConnection(_connStr))
        using (var cmd = new SqlCommand())
        {
            cmd.Connection = conn;
            cmd.CommandText = "SELECT * FROM Users WHERE Name = @Name";
            // Line 1: @Name is parameter placeholder
            
            // Method 1: AddWithValue (simple but type inference issues)
            cmd.Parameters.AddWithValue("@Name", userName);
            // Line 2: Value is treated as literal, not SQL
            
            // Method 2: Add with explicit type (recommended)
            cmd.Parameters.Add("@Name", SqlDbType.NVarChar, 50).Value = userName;
            // Line 3: Explicit type and size
            
            conn.Open();
            using (var reader = cmd.ExecuteReader())
            {
                // Read results...
            }
        }
    }
    
    // ─────────────────────────────────────────────────────────────────────
    // MULTIPLE PARAMETERS
    // ─────────────────────────────────────────────────────────────────────
    
    static void InsertEmployee(string name, decimal salary, int deptId)
    {
        using (var conn = new SqlConnection(_connStr))
        using (var cmd = new SqlCommand())
        {
            cmd.Connection = conn;
            cmd.CommandText = @"
                INSERT INTO Employees (Name, Salary, DepartmentId)
                VALUES (@Name, @Salary, @DeptId)";
            
            cmd.Parameters.Add("@Name", SqlDbType.NVarChar, 100).Value = name;
            cmd.Parameters.Add("@Salary", SqlDbType.Decimal).Value = salary;
            cmd.Parameters.Add("@DeptId", SqlDbType.Int).Value = deptId;
            
            conn.Open();
            cmd.ExecuteNonQuery();
        }
    }
    
    // ─────────────────────────────────────────────────────────────────────
    // OUTPUT PARAMETERS
    // ─────────────────────────────────────────────────────────────────────
    
    static int InsertAndGetId(string name)
    {
        using (var conn = new SqlConnection(_connStr))
        using (var cmd = new SqlCommand())
        {
            cmd.Connection = conn;
            cmd.CommandText = @"
                INSERT INTO Employees (Name) VALUES (@Name);
                SET @NewId = SCOPE_IDENTITY();";
            
            cmd.Parameters.Add("@Name", SqlDbType.NVarChar).Value = name;
            
            var outputParam = cmd.Parameters.Add("@NewId", SqlDbType.Int);
            outputParam.Direction = ParameterDirection.Output;
            // Line 4: Output parameter receives value from SQL
            
            conn.Open();
            cmd.ExecuteNonQuery();
            
            return (int)outputParam.Value;
        }
    }
    
    private static string _connStr = "";
}
```

---

## 🟣 Transactions

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// TRANSACTIONS - ALL OR NOTHING
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Data.SqlClient;

class TransactionDemo
{
    static void TransferMoney(int fromAccount, int toAccount, decimal amount)
    {
        using (var conn = new SqlConnection(_connStr))
        {
            conn.Open();
            
            // Begin transaction
            SqlTransaction transaction = conn.BeginTransaction();
            // Line 1: Start transaction - groups operations
            
            try
            {
                // Debit from source account
                using (var cmd = new SqlCommand())
                {
                    cmd.Connection = conn;
                    cmd.Transaction = transaction;
                    // Line 2: Associate command with transaction
                    
                    cmd.CommandText = "UPDATE Accounts SET Balance = Balance - @Amount WHERE Id = @Id";
                    cmd.Parameters.AddWithValue("@Amount", amount);
                    cmd.Parameters.AddWithValue("@Id", fromAccount);
                    cmd.ExecuteNonQuery();
                }
                
                // Credit to destination account
                using (var cmd = new SqlCommand())
                {
                    cmd.Connection = conn;
                    cmd.Transaction = transaction;
                    
                    cmd.CommandText = "UPDATE Accounts SET Balance = Balance + @Amount WHERE Id = @Id";
                    cmd.Parameters.AddWithValue("@Amount", amount);
                    cmd.Parameters.AddWithValue("@Id", toAccount);
                    cmd.ExecuteNonQuery();
                }
                
                // All succeeded - commit
                transaction.Commit();
                // Line 3: Commit makes changes permanent
                Console.WriteLine("Transfer successful!");
            }
            catch (Exception ex)
            {
                // Something failed - rollback everything
                transaction.Rollback();
                // Line 4: Rollback undoes ALL changes in transaction
                Console.WriteLine($"Transfer failed: {ex.Message}");
                throw;
            }
        }
    }
    
    private static string _connStr = "";
}
```

---

## 🎤 Interview Questions

**Q1: What is ADO.NET?**
> Data access technology in .NET for connecting to databases, executing commands, and retrieving results.

**Q2: Difference between ExecuteReader, ExecuteNonQuery, ExecuteScalar?**
> ExecuteReader returns DataReader for SELECT queries. ExecuteNonQuery returns rows affected for INSERT/UPDATE/DELETE. ExecuteScalar returns single value.

**Q3: How to prevent SQL Injection?**
> Use parameterized queries with `@ParameterName` placeholders and `SqlParameter` objects.

**Q4: What is a transaction?**
> Groups multiple operations into atomic unit - all succeed or all fail together.

**Q5: Connected vs Disconnected architecture?**
> Connected: DataReader keeps connection open. Disconnected: DataAdapter fills DataSet, connection closed.

---

## 📝 Summary

| Component | Purpose | Key Method |
|-----------|---------|------------|
| `SqlConnection` | Database connection | `Open()`, `Close()` |
| `SqlCommand` | Execute SQL | `ExecuteReader/NonQuery/Scalar` |
| `SqlDataReader` | Read results | `Read()`, `GetXxx()` |
| `SqlParameter` | Prevent injection | `Add()`, `AddWithValue()` |
| `SqlTransaction` | Atomicity | `Commit()`, `Rollback()` |
