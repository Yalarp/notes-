# JPQL and Custom Queries - Complete Guide

## Table of Contents
1. [What is JPQL?](#what-is-jpql)
2. [JPQL vs SQL](#jpql-vs-sql)
3. [Basic JPQL Syntax](#basic-jpql-syntax)
4. [@Query Annotation](#query-annotation)
5. [Named Parameters](#named-parameters)
6. [JPQL Joins](#jpql-joins)
7. [Native SQL Queries](#native-sql-queries)
8. [Update and Delete Queries](#update-and-delete-queries)
9. [Summary](#summary)

---

## What is JPQL?

**JPQL (Java Persistence Query Language)** is an object-oriented query language for querying JPA entities. It's similar to SQL but queries entities instead of tables.

> [!IMPORTANT]
> JPQL queries **entities and their properties**, not database tables and columns.

---

## JPQL vs SQL

| Aspect | JPQL | SQL |
|--------|------|-----|
| **Queries** | Entity classes | Database tables |
| **Properties** | Entity fields | Column names |
| **Case** | Case-sensitive (entities) | Usually case-insensitive |
| **Joins** | Based on relationships | Based on foreign keys |

### Example Comparison

```java
// SQL
SELECT * FROM students WHERE age > 18

// JPQL
SELECT s FROM Student s WHERE s.age > 18
```

---

## Basic JPQL Syntax

### SELECT Queries

```java
// Get all students
SELECT s FROM Student s

// Get specific fields
SELECT s.name, s.age FROM Student s

// With WHERE clause
SELECT s FROM Student s WHERE s.age > 18

// With ORDER BY
SELECT s FROM Student s ORDER BY s.name ASC

// With LIKE
SELECT s FROM Student s WHERE s.name LIKE '%John%'

// With IN
SELECT s FROM Student s WHERE s.age IN (18, 19, 20)

// With BETWEEN
SELECT s FROM Student s WHERE s.age BETWEEN 18 AND 25
```

### Aggregate Functions

```java
// Count
SELECT COUNT(s) FROM Student s

// Average
SELECT AVG(s.age) FROM Student s

// Max/Min
SELECT MAX(s.age), MIN(s.age) FROM Student s

// Sum
SELECT SUM(s.marks) FROM Student s

// Group By
SELECT s.department, COUNT(s) FROM Student s GROUP BY s.department

// Having
SELECT s.department, AVG(s.age) FROM Student s 
GROUP BY s.department 
HAVING AVG(s.age) > 20
```

---

## @Query Annotation

### Basic Usage

```java
@Repository
public interface StudentRepository extends JpaRepository<Student, Integer> {
    
    // Custom JPQL query
    @Query("SELECT s FROM Student s WHERE s.age > :age")
    List<Student> findStudentsOlderThan(@Param("age") int age);
    
    // Query with multiple conditions
    @Query("SELECT s FROM Student s WHERE s.name = :name AND s.age > :age")
    List<Student> findByNameAndMinAge(
        @Param("name") String name, 
        @Param("age") int age
    );
    
    // Ordering
    @Query("SELECT s FROM Student s ORDER BY s.age DESC")
    List<Student> findAllSortedByAge();
    
    // Distinct
    @Query("SELECT DISTINCT s.department FROM Student s")
    List<String> findAllDepartments();
}
```

---

## Named Parameters

### Using @Param

```java
// Named parameters with @Param
@Query("SELECT s FROM Student s WHERE s.name = :studentName")
List<Student> findByName(@Param("studentName") String name);

// Multiple parameters
@Query("SELECT s FROM Student s WHERE s.age BETWEEN :minAge AND :maxAge")
List<Student> findByAgeRange(
    @Param("minAge") int min, 
    @Param("maxAge") int max
);
```

### Positional Parameters

```java
// Positional parameters (1-indexed)
@Query("SELECT s FROM Student s WHERE s.name = ?1 AND s.age = ?2")
List<Student> findByNameAndAge(String name, int age);
```

### Named Parameters vs Positional

| Named | Positional |
|-------|------------|
| More readable | Less readable |
| Order doesn't matter | Order matters |
| `:paramName` | `?1`, `?2` |
| Recommended | Legacy approach |

---

## JPQL Joins

### Implicit Join (Dot Notation)

```java
// Access related entity through relationship
@Query("SELECT b FROM Book b WHERE b.author.name = :authorName")
List<Book> findByAuthorName(@Param("authorName") String name);
```

### Explicit JOIN

```java
// INNER JOIN
@Query("SELECT b FROM Book b JOIN b.author a WHERE a.country = :country")
List<Book> findByAuthorCountry(@Param("country") String country);
```

### LEFT JOIN

```java
// LEFT JOIN (includes books without reviews)
@Query("SELECT b FROM Book b LEFT JOIN b.reviews r WHERE r IS NULL")
List<Book> findBooksWithoutReviews();
```

### JOIN FETCH (Eager Loading)

```java
// Fetch related entities in single query (avoids N+1)
@Query("SELECT b FROM Book b JOIN FETCH b.author")
List<Book> findAllWithAuthors();

// Multiple JOIN FETCH
@Query("SELECT b FROM Book b JOIN FETCH b.author JOIN FETCH b.categories")
List<Book> findAllWithAuthorsAndCategories();
```

### JOIN vs JOIN FETCH

| JOIN | JOIN FETCH |
|------|-----------|
| Filters based on joined entity | Loads joined entity data |
| Doesn't load related data | Loads related data (eager) |
| May cause N+1 later | Prevents N+1 problem |

---

## Native SQL Queries

### When to Use Native SQL

- Database-specific features
- Complex queries hard to express in JPQL
- Performance optimization
- Legacy SQL migration

### Usage

```java
@Query(value = "SELECT * FROM students WHERE age > ?1", nativeQuery = true)
List<Student> findOlderStudentsNative(int age);

@Query(value = "SELECT * FROM students ORDER BY age DESC LIMIT 10", nativeQuery = true)
List<Student> findTop10ByAge();

// With named parameter (some drivers)
@Query(value = "SELECT * FROM students WHERE name = :name", nativeQuery = true)
List<Student> findByNameNative(@Param("name") String name);
```

### JPQL vs Native

| JPQL | Native SQL |
|------|------------|
| Entity classes | Table names |
| Database portable | Database specific |
| Type safe | Raw SQL |
| Recommended | Use when needed |

---

## Update and Delete Queries

### @Modifying Annotation

Update and delete queries require `@Modifying` and `@Transactional`:

```java
@Modifying
@Transactional
@Query("UPDATE Student s SET s.age = :age WHERE s.id = :id")
int updateAge(@Param("id") int id, @Param("age") int age);

@Modifying
@Transactional
@Query("DELETE FROM Student s WHERE s.age < :age")
int deleteByAgeLessThan(@Param("age") int age);

@Modifying
@Transactional
@Query("UPDATE Student s SET s.active = true WHERE s.department = :dept")
int activateByDepartment(@Param("dept") String department);
```

### Return Value

- Update/Delete queries return `int` (affected rows count)
- Can also return `void`

### clearAutomatically

```java
@Modifying(clearAutomatically = true)  // Clear persistence context after
@Transactional
@Query("UPDATE Student s SET s.age = :age WHERE s.id = :id")
int updateAge(@Param("id") int id, @Param("age") int age);
```

---

## Summary

### Quick Reference

| Feature | Syntax | Example |
|---------|--------|---------|
| Select | `SELECT e FROM Entity e` | `SELECT s FROM Student s` |
| Where | `WHERE e.field = :param` | `WHERE s.age > :age` |
| Join | `JOIN e.relation r` | `JOIN b.author a` |
| Fetch | `JOIN FETCH e.relation` | `JOIN FETCH b.author` |
| Native | `nativeQuery = true` | `SELECT * FROM students` |
| Update | `@Modifying` | `UPDATE Entity SET ...` |

### Key Points

1. **JPQL** queries entities, not tables
2. Use **@Query** for custom queries
3. Use **@Param** for named parameters
4. Use **JOIN FETCH** to avoid N+1
5. Use **@Modifying** for UPDATE/DELETE
6. Use **nativeQuery** for database-specific SQL

---

## Practice Questions

1. What is JPQL and how is it different from SQL?
2. How do you use named parameters in @Query?
3. What is the difference between JOIN and JOIN FETCH?
4. When would you use native SQL instead of JPQL?
5. What annotations are needed for UPDATE queries?
6. Write a JPQL query to find students older than 18 in a specific department.

---

**End of Note 16: JPQL and Custom Queries**

*Previous: [15_JPA_Relationships.md](file:///c:/Users/2706p/Desktop/mcq/notes/15_JPA_Relationships.md)*  
*Next: [17_Executable_JAR.md](file:///c:/Users/2706p/Desktop/mcq/notes/17_Executable_JAR.md)*
