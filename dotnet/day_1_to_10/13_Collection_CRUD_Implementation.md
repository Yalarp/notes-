# 🛠️ Collection CRUD Implementation - Service Pattern

## 📚 Table of Contents
- [Overview](#-overview)
- [Interface Definition](#-interface-definition)
- [Model Class](#-model-class)
- [Service Implementation](#-service-implementation)
- [Usage Examples](#-usage-examples)
- [Best Practices](#-best-practices)

---

## 🎯 Overview

This guide demonstrates implementing CRUD (Create, Read, Update, Delete) operations on collections using the **Service Pattern** with interfaces.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERVICE PATTERN ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Program.cs            IEmpService           EmployeeService       │
│   ┌──────────┐         ┌──────────┐          ┌──────────────┐      │
│   │  Client  │ ──────▶ │ Interface│ ◀─────── │ Implementation│      │
│   │  Code    │         │ Contract │          │ (List<T>)    │      │
│   └──────────┘         └──────────┘          └──────────────┘      │
│                                                                     │
│   Benefits:                                                         │
│   • Loose coupling - client depends on interface, not impl         │
│   • Easy testing - can mock the interface                          │
│   • Swappable - switch to database without changing client         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Interface Definition

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// IEmpService.cs - SERVICE CONTRACT
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;

namespace CA_collection
{
    // Line 1: Interface defines the contract for employee operations
    //         'I' prefix is C# naming convention for interfaces
    internal interface IEmpService
    {
        // Line 2: Get single employee by ID
        //         Returns null if not found
        Employee GetEmployee(int Id);
        
        // Line 3: Get employees by name (may return multiple)
        //         Returns IEnumerable for flexibility
        IEnumerable<Employee> GetEmployee(string name);
        
        // Line 4: Get all employees
        IEnumerable<Employee> GetAllEmployee();
        
        // Line 5: Create new employee
        //         Returns created employee with assigned ID
        Employee Add(Employee employee);
        
        // Line 6: Delete employee by ID
        //         Returns deleted employee (or null if not found)
        Employee Delete(int Id);
    }
}

// WHY INTERFACE?
// 1. Contract: Guarantees what methods must exist
// 2. Abstraction: Client doesn't know implementation details
// 3. Testability: Can create mock implementations
// 4. Flexibility: Swap List<T> for database later
```

---

## 🔶 Model Class

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// Employee.cs - DATA MODEL
// ═══════════════════════════════════════════════════════════════════════════

using System;

namespace CA_collection
{
    // Line 1: Enum for department - type-safe values
    public enum Dept
    {
        HR,       // Value = 0
        Payroll,  // Value = 1
        IT        // Value = 2
    }
    
    // Line 2: Internal = accessible only within same assembly
    internal class Employee
    {
        // Line 3: Auto-implemented properties
        public string Name { get; set; }
        public int Id { get; set; }
        public string Email { get; set; }
        
        // Line 4: Nullable enum - department can be null (unassigned)
        public Dept? Department { get; set; }
        
        // Line 5: Override ToString for easy display
        //         Called automatically in Console.WriteLine
        public override string ToString()
        {
            return String.Format("{0} {1} {2} {3}", Id, Name, Email, Department);
            // Output: "1 John john@example.com IT"
        }
    }
}
```

---

## 🔵 Service Implementation

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// EmployeeService.cs - CRUD IMPLEMENTATION
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;
using System.Linq;

namespace CA_collection
{
    // Line 1: Class implements the interface
    //         Must provide all methods defined in interface
    internal class EmployeeService : IEmpService
    {
        // Line 2: Static list - shared across all instances
        //         In real app, this would be database connection
        private static List<Employee> _employeeList;

        // Line 3: Constructor initializes with sample data
        public EmployeeService()
        {
            _employeeList = new List<Employee>
            {
                new Employee { Id = 1, Name = "Mary", Department = Dept.HR, 
                               Email = "mary@CDACtech.com" },
                new Employee { Id = 2, Name = "John", Department = Dept.IT, 
                               Email = "john@CDACtech.com" },
                new Employee { Id = 3, Name = "Sam", Department = Dept.IT, 
                               Email = "sam@CDACtech.com" }
            };
        }
        
        // ═══════════════════════════════════════════════════════════════════
        // CREATE - Add new employee
        // ═══════════════════════════════════════════════════════════════════
        
        public Employee Add(Employee employee)
        {
            // Line 4: Generate new ID = max existing ID + 1
            employee.Id = _employeeList.Max(e => e.Id) + 1;
            // Max() finds highest ID value using lambda
            
            // Line 5: Add to collection
            _employeeList.Add(employee);
            
            // Line 6: Return employee with generated ID
            return employee;
        }
        
        // ═══════════════════════════════════════════════════════════════════
        // READ - Get all employees
        // ═══════════════════════════════════════════════════════════════════
        
        public IEnumerable<Employee> GetAllEmployee()
        {
            // Line 7: Return entire list
            //         IEnumerable allows iteration without exposing internal list
            return _employeeList;
        }
        
        // ═══════════════════════════════════════════════════════════════════
        // READ - Get single employee by ID
        // ═══════════════════════════════════════════════════════════════════
        
        public Employee GetEmployee(int Id)
        {
            // Line 8: FirstOrDefault returns first match or null
            return _employeeList.FirstOrDefault(e => e.Id == Id);
            // Lambda: e => e.Id == Id
            //   e = each employee in list
            //   e.Id == Id = condition to match
            //   Returns first employee where ID matches, or null
        }
        
        // ═══════════════════════════════════════════════════════════════════
        // READ - Get employees by name
        // ═══════════════════════════════════════════════════════════════════
        
        public IEnumerable<Employee> GetEmployee(string name)
        {
            // Line 9: FindAll returns ALL matches as new list
            return _employeeList.FindAll(e => e.Name == name);
            // Returns List<Employee> of all employees with matching name
        }
        
        // ═══════════════════════════════════════════════════════════════════
        // DELETE - Remove employee by ID
        // ═══════════════════════════════════════════════════════════════════
        
        public Employee Delete(int Id)
        {
            // Line 10: Find employee to delete
            Employee employee = _employeeList.FirstOrDefault(e => e.Id == Id);
            
            // Line 11: Only remove if found
            if (employee != null)
            {
                _employeeList.Remove(employee);
                // Remove() removes first occurrence of object
            }
            
            // Line 12: Return deleted employee (or null if not found)
            return employee;
        }
    }
}
```

---

## 🟢 Usage Examples

```csharp
// ═══════════════════════════════════════════════════════════════════════════
// Program.cs - CLIENT CODE
// ═══════════════════════════════════════════════════════════════════════════

using System;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.Json.Serialization;

namespace CA_collection
{
    class Program
    {
        static void Main(string[] args)
        {
            // Line 1: Program to INTERFACE, not implementation
            //         IEmpService can be any implementation
            IEmpService empService = new EmployeeService();
            
            // ═══════════════════════════════════════════════════════════════
            // CREATE OPERATIONS
            // ═══════════════════════════════════════════════════════════════
            
            // Line 2: Add new employee
            Employee e1 = empService.Add(new Employee 
            { 
                Name = "Sonam", 
                Department = Dept.IT, 
                Email = "som@CDACtech.com" 
            });
            Console.WriteLine($"{e1} Added in collection");
            // Output: 4 Sonam som@CDACtech.com IT Added in collection
            
            // Line 3: Add another employee
            e1 = empService.Add(new Employee 
            { 
                Name = "Sonam",  // Same name, different department
                Department = Dept.HR, 
                Email = "sonam@CDACtech.com" 
            });
            Console.WriteLine($"{e1} Added in collection");
            // Output: 5 Sonam sonam@CDACtech.com HR Added in collection
            
            // ═══════════════════════════════════════════════════════════════
            // DELETE OPERATION
            // ═══════════════════════════════════════════════════════════════
            
            // Line 4: Delete employee with ID 3
            e1 = empService.Delete(3);
            Console.WriteLine($"{e1} Deleted from collection");
            // Output: 3 Sam sam@CDACtech.com IT Deleted from collection
            
            // ═══════════════════════════════════════════════════════════════
            // READ - Single employee by ID
            // ═══════════════════════════════════════════════════════════════
            
            // Line 5: Get employee with ID 1
            e1 = empService.GetEmployee(1);
            Console.WriteLine($"Record of {e1.Id} ==> {e1}");
            // Output: Record of 1 ==> 1 Mary mary@CDACtech.com HR
            
            // ═══════════════════════════════════════════════════════════════
            // READ - Multiple employees by name
            // ═══════════════════════════════════════════════════════════════
            
            // Line 6: Get all employees named "Sonam"
            IEnumerable<Employee> list = empService.GetEmployee("Sonam");
            Console.WriteLine("Records with name Sonam:");
            
            foreach (Employee emp in list)
            {
                Console.WriteLine(emp);
            }
            // Output:
            // 4 Sonam som@CDACtech.com IT
            // 5 Sonam sonam@CDACtech.com HR
            
            // ═══════════════════════════════════════════════════════════════
            // READ ALL - With JSON Serialization
            // ═══════════════════════════════════════════════════════════════
            
            // Line 7: Get all employees
            list = empService.GetAllEmployee();
            
            // Line 8: Configure JSON serialization options
            var options = new JsonSerializerOptions
            {
                WriteIndented = true,  // Pretty print
                Converters = { new JsonStringEnumConverter(JsonNamingPolicy.CamelCase) }
                // Serialize enum as string: "it" instead of 2
            };
            
            // Line 9: Display as JSON
            foreach (Employee emp in list)
            {
                Console.WriteLine(JsonSerializer.Serialize(emp, options));
            }
            /* Output:
            {
              "Name": "Mary",
              "Id": 1,
              "Email": "mary@CDACtech.com",
              "Department": "hr"
            }
            ...
            */
        }
    }
}
```

### Execution Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ADD OPERATION FLOW                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Client Code                    EmployeeService                     │
│  ┌─────────────┐               ┌─────────────────────┐             │
│  │ empService  │               │ Add(Employee)       │             │
│  │   .Add(...)─┼──────────────▶│ 1. Find max ID      │             │
│  └─────────────┘               │ 2. Assign ID + 1    │             │
│        │                       │ 3. Add to List      │             │
│        │                       │ 4. Return employee  │             │
│        │                       └─────────┬───────────┘             │
│        │                                 │                          │
│        ◀─────────────────────────────────┘                          │
│        │                                                            │
│  ┌─────▼─────┐                                                      │
│  │ Employee  │                                                      │
│  │ with new  │                                                      │
│  │ ID = 4    │                                                      │
│  └───────────┘                                                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✅ Best Practices

### 1. Program to Interface
```csharp
// ✅ GOOD: Depends on interface
IEmpService service = new EmployeeService();

// ❌ BAD: Depends on concrete class
EmployeeService service = new EmployeeService();
```

### 2. Return IEnumerable for Collections
```csharp
// ✅ Returns IEnumerable - flexible, read-only view
public IEnumerable<Employee> GetAll() => _list;

// ❌ Returns List - exposes internal collection
public List<Employee> GetAll() => _list;  // Client can modify!
```

### 3. Validate Input
```csharp
public Employee Add(Employee employee)
{
    if (employee == null) throw new ArgumentNullException(nameof(employee));
    if (string.IsNullOrEmpty(employee.Name)) 
        throw new ArgumentException("Name required");
    // ... proceed
}
```

---

## 🎤 Interview Questions

**Q1: Why use interface IEmpService?**
> Allows loose coupling - client depends on contract not implementation. Easy to swap implementations or mock for testing.

**Q2: Why return IEnumerable instead of List?**
> IEnumerable provides read-only iteration. Returning List exposes internal collection that callers could modify.

**Q3: What does FirstOrDefault do?**
> Returns first matching element, or default value (null for reference types) if no match found.

**Q4: How is ID generated in Add method?**
> Uses `_list.Max(e => e.Id) + 1` - finds maximum existing ID and adds 1.

---

## 📝 Summary

| Operation | Method | LINQ Used |
|-----------|--------|-----------|
| Create | `Add()` | `Max()` |
| Read One | `GetEmployee(id)` | `FirstOrDefault()` |
| Read Many | `GetEmployee(name)` | `FindAll()` |
| Read All | `GetAllEmployee()` | Direct return |
| Delete | `Delete(id)` | `FirstOrDefault()`, `Remove()` |
