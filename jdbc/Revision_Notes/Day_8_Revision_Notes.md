# Day 8: HQL, Caching & Associations

## 1. HQL (Hibernate Query Language)
*   **Concept**: Object-oriented query language. Works on **Entity Classes and Fields**, not Tables and Columns.
*   **Database Independent**: Hibernate translates HQL to database-specific SQL (using Dialect).

### usage
```java
// 1. Create Query
Query<Employee> q = session.createQuery("from Employee where salary > :minsal", Employee.class);

// 2. Set Parameters
q.setParameter("minsal", 50000);

// 3. Execute
List<Employee> list = q.list();
```
*   **Select Clause**: `select name from Employee` returns list of Strings.
*   **Update/Delete**: `q.executeUpdate()`.

---

## 2. Caching
Mechanism to improve performance by reducing database hits.

### First Level Cache (Session Scope)
*   **Scope**: Associated with the `Session` object.
*   **Default**: Enabled by default. Cannot be disabled.
*   **Behavior**: If you request the same object (by ID) twice in the **same session**, Hibernate returns the cached object without firing a second SQL query.
*   **Clear**: `session.clear()` or `session.evict(obj)`.

### Second Level Cache (SessionFactory Scope)
*   **Scope**: Associated with `SessionFactory`. Shared across **all sessions**.
*   **Default**: Disabled. Needs 3rd party provider (EhCache, Infinispan).
*   **Configuration**:
    ```xml
    <property name="hibernate.cache.use_second_level_cache">true</property>
    <property name="hibernate.cache.region.factory_class">org.hibernate.cache.jcache.JCacheRegionFactory</property>
    ```
*   **Usage**: Annotate Entity with `@Cacheable` and `@Cache(usage=CacheConcurrencyStrategy.READ_ONLY)`.

---

## 3. Association Mapping
Mapping relationships between database tables to Java objects.

### Types
1.  **One-to-One** (`@OneToOne`): e.g., Employee <-> Passport.
2.  **One-to-Many** (`@OneToMany`): e.g., Department -> Employees.
3.  **Many-to-One** (`@ManyToOne`): e.g., Employee -> Department.
4.  **Many-to-Many** (`@ManyToMany`): e.g., Student <-> Course.

### Key Annotations
*   `@JoinColumn(name="dept_id")`: Specifies the foreign key column.
*   `@OneToMany(mappedBy="dept")`:
    *   **mappedBy** tells Hibernate that this side is the **inverse** (passive) side.
    *   The other side (Employee) owns the relationship (contains the Foreign Key).
    *   Prevents creating an extra mapping table.
*   `cascade = CascadeType.ALL`:
    *   If you save Parent, Child is automatically saved.
    *   If you delete Parent, Child is automatically deleted.

### Example (One-to-Many)
```java
@Entity
public class Dept {
    @Id
    private int deptId;
    
    @OneToMany(mappedBy = "dept", cascade = CascadeType.ALL)
    private List<Employee> emps = new ArrayList<>();
}

@Entity
public class Employee {
    @Id
    private int empId;
    
    @ManyToOne
    @JoinColumn(name = "deptId")
    private Dept dept;
}
```
