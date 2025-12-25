# Day 7: Hibernate & ORM

## 1. ORM (Object Relational Mapping)
*   **Problem with JDBC**: Developer writes SQL manually, handles connections, and maps ResultSet to Objects line-by-line.
*   **Solution (ORM)**: Framework that automatically maps Java Objects (Entities) to Database Tables.
*   **Hibernate**: A popular ORM framework implementation.

---

## 2. Hibernate Architecture
### Core Objects
1.  **Configuration**:
    *   Loads `hibernate.cfg.xml`.
    *   Builds the `SessionFactory`.
2.  **SessionFactory**:
    *   **Heavyweight** object. Created **once per application**.
    *   Thread-safe.
    *   Factory for `Session` objects.
3.  **Session**:
    *   **Lightweight** object. Created **per request** or unit of work.
    *   **Not thread-safe**.
    *   Represents a connection to the DB. Used for CRUD operations (`save`, `get`, `delete`).
4.  **Transaction**:
    *   Required for data modification (Insert, Update, Delete).
    *   `session.beginTransaction()`, `tx.commit()`, `tx.rollback()`.

---

## 3. Configuration (`hibernate.cfg.xml`)
Root tag: `<hibernate-configuration>`.
Key Properties:
*   `hibernate.connection.driver_class`: JDBC Driver.
*   `hibernate.connection.url`: Database URL.
*   `hibernate.connection.username` / `password`: Credentials.
*   `hibernate.dialect`: SQL Dialect (e.g., `org.hibernate.dialect.MySQLDialect`). Translates HQL to database-specific SQL.
*   `hibernate.hbm2ddl.auto`: Auto-generation of tables (`update`, `create`, `create-drop`, `validate`).
*   `show_sql`: Prints generated SQL to console.
*   `<mapping class="com.mypack.Employee"/>`: Registers Entity classes.

---

## 4. Entity Class (Annotations)
A POJO representing a table.
*   `@Entity`: Marks class as a Hibernate Entity (Mandatory).
*   `@Table(name="emp")`: Maps to specific table name (Optional, defaults to class name).
*   `@Id`: Marks primary key (Mandatory).
*   `@GeneratedValue(strategy = GenerationType.IDENTITY)`: Auto-increment (Identity column).
*   `@Column(name="ename")`: Maps field to column (Optional).

```java
@Entity
@Table(name = "Employee")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    
    @Column(name="emp_name")
    private String name;
    
    // Getters and Setters
}
```

---

## 5. Basic Usage Flow
```java
// 1. Configure
Configuration cfg = new Configuration().configure();

// 2. Build Factory
SessionFactory factory = cfg.buildSessionFactory();

// 3. Open Session
Session session = factory.openSession();

// 4. Begin Transaction
Transaction tx = session.beginTransaction();

// 5. Operation
Employee e1 = new Employee();
e1.setName("Scott");
session.persist(e1); // Saves to DB

// 6. Commit
tx.commit();

// 7. Close
session.close();
factory.close();
```

---

## 6. Design Patterns: Flyweight
*   **Goal**: Minimize memory use by sharing data with other similar objects.
*   **Concept**: Use existing objects instead of creating new ones if they match.
*   **Examples in Java**:
    *   **String Constant Pool**: Reuses String literals.
    *   **Wrapper Classes**: `Integer.valueOf(10)` returns cached object for small values (-128 to 127).
