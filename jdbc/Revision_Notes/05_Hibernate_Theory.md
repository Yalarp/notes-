# 📘 Hibernate ORM - Complete Theory
## Understanding Object-Relational Mapping & Persistence

---

## 🔄 1. The Object-Relational Impedance Mismatch

### 1.1 Two Different Worlds

**Object-Oriented World (Java)**:
- Works with **Objects** (encapsulation, inheritance, polymorphism)
- Data + Behavior together
- Relations via **references** (object references)
- Identity by **equality** (==, equals())

**Relational World (Database)**:
- Works with **Tables** (rows and columns)
- Only data, no behavior
- Relations via **foreign keys**
- Identity by **primary key**

### 1.2 The Mismatch Problems

| Problem | OO Approach | Relational Approach |
|---------|-------------|---------------------|
| **Granularity** | Many small classes | Few normalized tables |
| **Inheritance** | Natural (extends) | No direct support |
| **Identity** | Object reference | Primary key |
| **Association** | One-way reference | Foreign key (bidirectional) |
| **Navigation** | Object.getRelated() | JOIN queries |

**Example of the problem**:
```java
// Object-Oriented: Simple navigation
Employee emp = getEmployee(1);
String deptName = emp.getDepartment().getName();
String managerName = emp.getDepartment().getManager().getName();
```

In JDBC, this requires multiple queries or complex JOINs!

### 1.3 How ORM Solves This

ORM (Object-Relational Mapping) provides automatic translation:

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORM MAPPING                                   │
│                                                                  │
│   JAVA WORLD                         DATABASE WORLD              │
│   ┌────────────────┐                 ┌────────────────┐         │
│   │ Class Employee │  ◀═══════════▶  │ Table employee │         │
│   │ - id: int      │      ORM        │ - emp_id INT   │         │
│   │ - name: String │    (Hibernate)  │ - emp_name VARCHAR│      │
│   │ - salary: double│                │ - salary DECIMAL│        │
│   └────────────────┘                 └────────────────┘         │
│                                                                  │
│   Employee e = new Employee();        INSERT INTO employee      │
│   e.setName("John");          ═══▶   (emp_name, salary)        │
│   session.persist(e);                 VALUES ('John', ...);     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏗️ 2. Hibernate Architecture

### 2.1 Core Components

```
┌─────────────────────────────────────────────────────────────────┐
│                  HIBERNATE ARCHITECTURE                          │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    APPLICATION LAYER                        │ │
│  │    (Your Java code - POJOs, DAOs, Services)                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                  HIBERNATE CORE                             │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │ │
│  │  │Configuration │─▶│SessionFactory│─▶│    Session       │  │ │
│  │  │              │  │              │  │                  │  │ │
│  │  │ Reads config │  │Heavy, once   │  │Lightweight       │  │ │
│  │  │ Builds factory│ │Thread-safe   │  │Not thread-safe   │  │ │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘  │ │
│  │                                            │                │ │
│  │                                            ▼                │ │
│  │                    ┌────────────────────────────────────┐   │ │
│  │                    │        Transaction                 │   │ │
│  │                    │    Control commit/rollback         │   │ │
│  │                    └────────────────────────────────────┘   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    DATABASE                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Configuration

**Purpose**: Tell Hibernate about database and mappings.

```java
Configuration cfg = new Configuration();
cfg.configure("hibernate.cfg.xml");
```

**hibernate.cfg.xml contents**:
```xml
<hibernate-configuration>
    <session-factory>
        <!-- Database connection -->
        <property name="connection.url">jdbc:mysql://localhost/mydb</property>
        <property name="connection.username">root</property>
        <property name="connection.password">password</property>
        <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        
        <!-- Hibernate dialect -->
        <property name="dialect">org.hibernate.dialect.MySQL8Dialect</property>
        
        <!-- Schema generation -->
        <property name="hbm2ddl.auto">update</property>
        
        <!-- Show SQL in console -->
        <property name="show_sql">true</property>
        
        <!-- Entity mappings -->
        <mapping class="com.example.Employee"/>
    </session-factory>
</hibernate-configuration>
```

### 2.3 What is Dialect?

**Dialect** tells Hibernate which database-specific SQL to generate.

**Why needed?**
Each database has slightly different SQL syntax:
- MySQL: `LIMIT 10`
- Oracle: `ROWNUM <= 10`
- PostgreSQL: `LIMIT 10`
- SQL Server: `TOP 10`

Dialect handles these differences transparently!

### 2.4 SessionFactory

**Purpose**: Factory to create Session objects.

**Key characteristics**:
- **Heavy object**: Expensive to create
- **Thread-safe**: Can be shared across threads
- **Immutable**: Once created, configuration can't change
- **One per database**: Create once at application startup

```java
SessionFactory factory = cfg.buildSessionFactory();
// Use this single instance throughout application!
```

### 2.5 Session

**Purpose**: Main interface to interact with database.

**Key characteristics**:
- **Lightweight**: Cheap to create
- **NOT thread-safe**: One per thread/request
- **Short-lived**: Open, use, close
- **Wrapper around Connection**: Plus caching and more

```java
Session session = factory.openSession();
// Perform operations
session.close();
```

---

## 📊 3. Object States in Hibernate

### 3.1 Three States of an Object

```
┌────────────────────────────────────────────────────────────────┐
│                    OBJECT STATE DIAGRAM                         │
│                                                                 │
│                    ┌─────────────┐                             │
│                    │  TRANSIENT  │                             │
│                    │             │                             │
│                    │ new Object()│                             │
│                    │ No ID, not  │                             │
│                    │ in session  │                             │
│                    └──────┬──────┘                             │
│                           │                                    │
│                           │ persist()/save()                   │
│                           ▼                                    │
│                    ┌─────────────┐                             │
│                    │  PERSISTENT │                             │
│                    │             │                             │
│                    │ Has ID      │                             │
│                    │ In session  │                             │
│                    │ Auto-sync   │                             │
│                    │ with DB     │                             │
│                    └──────┬──────┘                             │
│                           │                                    │
│                           │ session.close()/evict()            │
│                           ▼                                    │
│                    ┌─────────────┐                             │
│                    │  DETACHED   │                             │
│                    │             │                             │
│                    │ Has ID      │◀─── merge()/update()        │
│                    │ Not in      │     re-attaches             │
│                    │ session     │                             │
│                    │ Changes NOT │                             │
│                    │ auto-synced │                             │
│                    └─────────────┘                             │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 3.2 State Definitions

**Transient**:
- Created using `new` keyword
- NO identifier (id is null or default)
- NOT associated with any Session
- NOT in database
- Garbage collected if no references

**Persistent**:
- Has database identifier
- Associated with active Session
- Exists in Session's first-level cache
- Changes **automatically synchronized** with database on commit

**Detached**:
- Was persistent, but Session is closed
- Has identifier (was saved before)
- Changes NOT automatically synced
- Must be re-attached using `merge()` or `update()`

### 3.3 State Transitions Code

```java
// TRANSIENT - just created, no ID
Employee e = new Employee();
e.setName("John");
// State: TRANSIENT

// PERSISTENT - saved to session
Session session = factory.openSession();
Transaction tx = session.beginTransaction();
session.persist(e);
// State: PERSISTENT (ID assigned by Hibernate)

tx.commit();
session.close();
// State: DETACHED (session closed)

// Re-attach detached object
Session session2 = factory.openSession();
Transaction tx2 = session2.beginTransaction();
e.setName("John Updated");
session2.merge(e);
// State: PERSISTENT again
tx2.commit();
```

---

## 🔧 4. Session Methods - Detailed Understanding

### 4.1 persist() - Save New Object

```java
Employee e = new Employee();
e.setName("John");
session.persist(e);  // SQL INSERT generated (not executed yet)
tx.commit();         // SQL actually executed here
```

- Makes transient object persistent
- INSERT scheduled for commit
- Returns void
- ID assigned based on strategy

### 4.2 get() vs load()

**get()** - Eager loading:
```java
Employee e = session.get(Employee.class, 1);
// SELECT immediately executed
// Returns null if not found
```

**load()** - Lazy loading:
```java
Employee e = session.load(Employee.class, 1);
// Returns PROXY, no SQL yet!
e.getName();  // NOW SQL executes
// Throws ObjectNotFoundException if not found
```

| Feature | get() | load() |
|---------|-------|--------|
| SQL execution | Immediately | When property accessed |
| If not found | Returns null | Throws exception |
| Returns | Actual object | Proxy object |
| Use when | Need data now | Might not need data |

### 4.3 update() vs merge()

**update()** - Re-attach detached object:
```java
session.update(detachedEmployee);
// Throws exception if object with same ID already in session!
```

**merge()** - Safer re-attachment:
```java
Employee managed = session.merge(detachedEmployee);
// Copies state from detached to new persistent object
// Original detached object remains detached!
```

| Feature | update() | merge() |
|---------|----------|---------|
| Original object | Becomes persistent | Stays detached |
| Duplicate ID in session | Exception | Handles gracefully |
| Return value | void | Merged object |
| Safe to use | Less | More |

### 4.4 refresh() - Reload from Database

```java
Employee e = session.get(Employee.class, 1);
e.setName("Modified");  // In-memory change

// Undo changes - reload from DB
session.refresh(e);
// e.getName() returns original DB value
```

**Use cases**:
- Discard local changes
- Reload after database trigger modified data
- Sync with external changes

### 4.5 delete() - Remove Object

```java
Employee e = session.get(Employee.class, 1);
session.delete(e);
// Object becomes transient (removed from DB on commit)
```

---

## ✨ 5. Automatic Dirty Checking

### 5.1 The Concept

**Dirty checking** = Hibernate automatically detects changes to persistent objects.

```java
Employee e = session.get(Employee.class, 1);  // Load from DB
// e is PERSISTENT, in session's cache

e.setSalary(50000);  // Just change the object!
// NO explicit update() call needed!

tx.commit();
// Hibernate compares current state with original
// Detects change → generates UPDATE SQL automatically!
```

### 5.2 How It Works Internally

```
┌────────────────────────────────────────────────────────────────┐
│                  DIRTY CHECKING MECHANISM                       │
│                                                                 │
│   LOAD TIME:                                                   │
│   ┌─────────────────────┐  ┌─────────────────────┐            │
│   │ FIRST-LEVEL CACHE   │  │    SNAPSHOT         │            │
│   │                     │  │                     │            │
│   │ Employee@123        │  │ id=1                │            │
│   │ - id: 1             │  │ name="John"         │            │
│   │ - name: "John"      │  │ salary=40000        │            │
│   │ - salary: 40000     │  │                     │            │
│   └─────────────────────┘  └─────────────────────┘            │
│                                                                │
│   AFTER MODIFICATION (e.setSalary(50000)):                    │
│   ┌─────────────────────┐  ┌─────────────────────┐            │
│   │ FIRST-LEVEL CACHE   │  │    SNAPSHOT         │            │
│   │                     │  │    (unchanged)      │            │
│   │ Employee@123        │  │ id=1                │            │
│   │ - id: 1             │  │ name="John"         │            │
│   │ - name: "John"      │  │ salary=40000        │  Different!│
│   │ - salary: 50000 ◀───│──│───────────────────▶│            │
│   └─────────────────────┘  └─────────────────────┘            │
│                                                                │
│   ON COMMIT: Compare cache with snapshot → Generate UPDATE     │
└────────────────────────────────────────────────────────────────┘
```

### 5.3 Important Rule

**Dirty checking only works for PERSISTENT objects!**

```java
// ❌ Won't work - object is DETACHED
session.close();
e.setSalary(50000);  // No session to track this!
// Opening new session won't auto-update

// ✅ Works - object is PERSISTENT
e.setSalary(50000);
tx.commit();  // Auto UPDATE!
```

---

## 🏷️ 6. Entity Mapping Annotations

### 6.1 Basic Annotations

```java
@Entity  // Marks class as database entity
@Table(name = "employees")  // Table name (optional if same as class)
public class Employee {
    
    @Id  // Primary key field
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    // AUTO_INCREMENT in MySQL
    @Column(name = "emp_id")
    private int id;
    
    @Column(name = "emp_name", nullable = false, length = 100)
    private String name;
    
    @Column(name = "salary", precision = 10, scale = 2)
    private double salary;
    
    // IMPORTANT: Setters REQUIRED even for auto-generated ID!
    // Hibernate uses setter to assign generated ID back to object
}
```

### 6.2 Why Setter is Needed for Auto-Generated ID?

Even with `@GeneratedValue`, setter is mandatory because:

1. Hibernate creates object
2. Hibernate calls `persist()`
3. Database generates ID (AUTO_INCREMENT)
4. Hibernate reads generated ID
5. **Hibernate calls `setId()` to assign it back!**

Without setter → Exception!

---

## 📋 7. Transaction Management

### 7.1 Why Transactions?

Transactions ensure **ACID properties**:
- **Atomicity**: All operations succeed or all fail
- **Consistency**: Database remains valid
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed changes persist

### 7.2 Transaction Pattern

```java
Session session = factory.openSession();
Transaction tx = null;

try {
    tx = session.beginTransaction();
    
    // All database operations here
    session.persist(emp1);
    session.persist(emp2);
    
    // Some might fail
    if (someCondition) {
        throw new RuntimeException("Business rule violated");
    }
    
    tx.commit();  // All succeed together
} catch (Exception e) {
    if (tx != null) {
        tx.rollback();  // All fail together
    }
    throw e;
} finally {
    session.close();
}
```

### 7.3 Commit Triggers SQL

**Important**: SQL statements are NOT executed immediately!

```java
session.persist(e1);  // SQL prepared, not executed
session.persist(e2);  // SQL prepared, not executed
session.persist(e3);  // SQL prepared, not executed

tx.commit();  // NOW all INSERTs execute!
```

This is called **write-behind** - better performance through batching.

---

## 🎯 Key Viva Questions & Answers

1. **What is Hibernate?**
   → Hibernate is an ORM framework that maps Java objects to database tables, eliminating the need for manual SQL.

2. **What is ORM?**
   → ORM (Object-Relational Mapping) is a technique to convert between object-oriented data and relational database data automatically.

3. **What is SessionFactory?**
   → SessionFactory is a thread-safe, immutable factory object created once at startup to produce Session instances.

4. **Difference between get() and load()?**
   → get() executes SQL immediately and returns null if not found; load() returns a proxy and throws exception if not found when accessed.

5. **What are object states in Hibernate?**
   → Transient (new, not in session), Persistent (in session, auto-synced), Detached (was persistent, session closed).

6. **What is Automatic Dirty Checking?**
   → Hibernate automatically detects changes to persistent objects and generates UPDATE SQL on commit without explicit update() call.

7. **Difference between update() and merge()?**
   → update() throws exception if same ID exists in session; merge() copies state safely and returns new managed object.

8. **Why is setter needed for auto-generated ID?**
   → Hibernate uses the setter to assign the database-generated ID back to the Java object after persist.

---

> 📝 **End of Advanced Java Theory Notes**
