# Spring Boot Unit Testing Complete Guide

## Table of Contents
1. [Introduction to Spring Boot Testing](#introduction-to-spring-boot-testing)
2. [Project Setup](#project-setup)
3. [Entity Layer: Person](#entity-layer-person)
4. [Repository Layer: PersonRepository](#repository-layer-personrepository)
5. [Service Layer: PersonService](#service-layer-personservice)
6. [Controller Layer: PersonController](#controller-layer-personcontroller)
7. [Testing Repository Layer](#testing-repository-layer)
8. [Testing Service Layer with Mockito](#testing-service-layer-with-mockito)
9. [Complete Test Classes](#complete-test-classes)
10. [Execution Flow](#execution-flow)
11. [Interview Questions](#interview-questions)

---

## Introduction to Spring Boot Testing

### What is Unit Testing in Spring Boot?

**Definition:** Unit testing in Spring Boot means testing **individual layers (Controller, Service, Repository) in isolation**, ensuring each component works correctly before integration.

### Layer Testing Strategy

```
┌─────────────────────────────────────────────────────────────────────┐
│                 SPRING BOOT LAYERS                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  CONTROLLER LAYER                                            │    │
│  │  • @RestController                                           │    │
│  │  • Handles HTTP requests                                     │    │
│  │  • Test: MockMvc, @WebMvcTest                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                           │                                          │
│                           ▼                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  SERVICE LAYER                                               │    │
│  │  • @Service                                                  │    │
│  │  • Business logic                                            │    │
│  │  • Test: @Mock, @ExtendWith(MockitoExtension.class)         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                           │                                          │
│                           ▼                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  REPOSITORY LAYER                                            │    │
│  │  • @Repository / JpaRepository                               │    │
│  │  • Database operations                                       │    │
│  │  • Test: @SpringBootTest, @DataJpaTest                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                           │                                          │
│                           ▼                                          │
│                    ┌──────────────┐                                  │
│                    │   DATABASE   │                                  │
│                    │   (MySQL)    │                                  │
│                    └──────────────┘                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

> In this example we are testing all the layers i.e. Controller, Service and Repository **in isolation**.

---

## Project Setup

### Dependencies (Spring Initializr)
```
✓ Spring Web
✓ Spring Data JPA
✓ MySQL Driver
✓ Spring Boot DevTools
```

### application.properties

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/hiber
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

#### Configuration Explanation

| Property | Description |
|----------|-------------|
| `spring.datasource.url` | JDBC URL for MySQL database "hiber" |
| `spring.datasource.username` | Database username |
| `spring.datasource.password` | Database password |
| `spring.jpa.hibernate.ddl-auto=update` | Auto-create/update tables based on entities |
| `spring.jpa.show-sql=true` | Print SQL queries to console |

### Project Structure
```
src/main/java/
├── com/example/demo/
│   └── JUnitProApplication.java
├── com/example/entities/
│   └── Person.java
├── com/example/repositories/
│   └── PersonRepository.java
├── com/example/services/
│   └── PersonService.java
└── com/example/controllers/
    └── PersonController.java

src/test/java/
└── com/example/demo/
    ├── Calculator.java
    ├── JUnitProApplicationTests.java
    ├── PersonRepositoryTest.java
    └── PersonServiceTest.java
```

---

## Entity Layer: Person

### Person.java

```java
package com.example.entities;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Person 
{
    private int id;
    private String name, city;
    
    @Override
    public String toString() {
        return "Person [id=" + id + ", name=" + name + ", city=" + city + "]";
    }
    
    public String getCity() {
        return city;
    }
    
    public void setCity(String city) {
        this.city = city;
    }
    
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    public int getId() {
        return id;
    }
    
    public void setId(int id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3-6 | Imports | JPA annotations from jakarta.persistence |
| 8 | `@Entity` | Marks class as JPA entity (maps to database table) |
| 11-12 | Fields | Entity fields mapped to table columns |
| 15-17 | `toString()` | String representation for debugging |
| 27-28 | `@Id` | Marks id as primary key |
| 28 | `@GeneratedValue(strategy=IDENTITY)` | Auto-increment by database (MySQL) |

---

## Repository Layer: PersonRepository

### PersonRepository.java

```java
package com.example.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import com.example.entities.Person;

public interface PersonRepository extends JpaRepository<Person, Integer>
{
    @Query("select case when count(s) >0 then true else false end from Person s where s.id=?1")
    boolean isPersonExistsById(Integer id);
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 8 | `extends JpaRepository<Person,Integer>` | Inherits CRUD operations for Person (ID type: Integer) |
| 10 | `@Query(...)` | Custom JPQL query |
| 10 | `count(s) >0 then true else false` | Returns boolean based on count |
| 10 | `?1` | First method parameter placeholder |
| 11 | `isPersonExistsById(Integer id)` | Custom method to check if person exists |

### Inherited Methods from JpaRepository
| Method | Description |
|--------|-------------|
| `save(entity)` | Save or update entity |
| `findById(id)` | Find by primary key |
| `findAll()` | Get all entities |
| `deleteById(id)` | Delete by primary key |
| `count()` | Count all entities |

---

## Service Layer: PersonService

### PersonService.java

```java
package com.example.services;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.entities.Person;
import com.example.repositories.PersonRepository;

@Service
public class PersonService 
{
    @Autowired
    private PersonRepository prepository;
    
    public void addPerson(Person person)
    {
        //prepository.save(person);
    }
    
    public List<Person> getAllPersons()
    {
        return prepository.findAll();
    }

    public PersonService(PersonRepository prepository) {
        super();
        this.prepository = prepository;
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 11 | `@Service` | Marks as Spring service bean |
| 14-15 | `@Autowired PersonRepository` | Injects repository dependency |
| 17-20 | `addPerson(Person)` | Method to add person (commented for testing) |
| 22-25 | `getAllPersons()` | Returns all persons from database |
| 27-30 | Constructor | **Constructor injection** for testability with mocks |

---

## Controller Layer: PersonController

### PersonController.java

```java
package com.example.controllers;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import com.example.entities.Person;
import com.example.services.PersonService;

@RestController
public class PersonController 
{
    @Autowired
    private PersonService pservice;
    
    public PersonController(PersonService pservice) {
        super();
        this.pservice = pservice;
    }

    @PostMapping("/addPerson")
    public void addPerson(@RequestBody Person person)
    {
        //pservice.addPerson(person);
    }
    
    @GetMapping("/persons")
    public ResponseEntity<?> getAllPersons()
    {
        return ResponseEntity.ok(pservice.getAllPersons());
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 13 | `@RestController` | Combines @Controller and @ResponseBody |
| 16-17 | `@Autowired PersonService` | Injects service dependency |
| 19-22 | Constructor | Constructor injection for testing |
| 24-28 | `addPerson()` | POST endpoint to add person |
| 25 | `@RequestBody Person` | Deserializes JSON body to Person object |
| 30-34 | `getAllPersons()` | GET endpoint returning all persons |
| 33 | `ResponseEntity.ok(...)` | Returns 200 OK with body |

---

## Testing Repository Layer

### Key Annotations

| Annotation | Purpose |
|------------|---------|
| `@SpringBootTest` | Loads full Spring context |
| `@Autowired` | Injects real repository bean |
| `@TestMethodOrder` | Controls test execution order |
| `@Order(n)` | Specifies test order (1, 2, 3...) |
| `@BeforeEach` | Setup before each test |

### PersonRepositoryTest.java

```java
package com.example.demo;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.List;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import com.example.entities.Person;
import com.example.repositories.PersonRepository;
import org.junit.jupiter.api.MethodOrderer;

@SpringBootTest
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class PersonRepositoryTest {

    @Autowired
    private PersonRepository repository;
    
    @Test
    @Order(2)
    void testIsPersonExistsById() {
        System.out.println("ispersonexistsbyid method");
        Person p = new Person();
        p.setName("Abc");
        p.setCity("Mumbai");
        repository.save(p);
        
        boolean result = repository.isPersonExistsById(p.getId());
        assertThat(result).isTrue();
    }
    
    @Test
    @Order(1)
    void testgetAllPersons()
    {
        System.out.println("getAllPerson method");
        List<Person> mylist = repository.findAll();
        assertThat(mylist.size()).isGreaterThan(0);
    }
    
    @BeforeEach
    void setUp()
    {
        System.out.println("Setting up");
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3 | `import static org.assertj.core.api.Assertions.assertThat` | AssertJ fluent assertions |
| 17 | `@SpringBootTest` | Loads complete application context |
| 18 | `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)` | Enables @Order annotation |
| 21-22 | `@Autowired PersonRepository` | Injects **real** repository (with DB connection) |
| 25 | `@Order(2)` | This test runs second |
| 26-35 | `testIsPersonExistsById()` | Tests custom query method |
| 28-31 | Create and save Person | Creates entity, saves to real database |
| 33 | `repository.isPersonExistsById(p.getId())` | Tests custom JPQL query |
| 34 | `assertThat(result).isTrue()` | AssertJ assertion |
| 38 | `@Order(1)` | This test runs first |
| 39-44 | `testgetAllPersons()` | Tests findAll() method |
| 43 | `assertThat(mylist.size()).isGreaterThan(0)` | Verifies list has elements |
| 46-50 | `@BeforeEach setUp()` | Runs before each test |

### Why @SpringBootTest for Repository?

The repository test needs **real database connection** because:
- We're testing actual SQL queries
- Custom JPQL queries must be validated
- JpaRepository methods interact with database

---

## Testing Service Layer with Mockito

### Key Annotations

| Annotation | Purpose |
|------------|---------|
| `@SpringBootTest` | Load Spring context |
| `@ExtendWith(MockitoExtension.class)` | Enable Mockito annotations |
| `@Mock` | Create mock object |
| `verify()` | Verify method was called |

### PersonServiceTest.java

```java
package com.example.demo;

import static org.mockito.Mockito.verify;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.boot.test.context.SpringBootTest;

import com.example.entities.Person;
import com.example.repositories.PersonRepository;
import com.example.services.PersonService;

@SpringBootTest
@ExtendWith(MockitoExtension.class)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class PersonServiceTest 
{

    @Mock
    private PersonRepository prepository;
    
    private PersonService pservice;
    
    @BeforeEach
    void setUp()
    {
        pservice = new PersonService(this.prepository);
    }
    
    @Test 
    @Order(2)
    void testGetAllPersons() {
        System.out.println("testgetallpersons");
        pservice.getAllPersons(); 
        verify(prepository).findAll();
    }
     
    @Test
    @Order(1)
    void testaddPerson() {
        System.out.println("testaddperson");
        Person p = new Person();
        pservice.addPerson(p);
        verify(prepository).save(p);
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3 | `import static org.mockito.Mockito.verify` | Import verify() method |
| 19 | `@SpringBootTest` | Load Spring context |
| 20 | `@ExtendWith(MockitoExtension.class)` | Enable Mockito for JUnit 5 |
| 21 | `@TestMethodOrder(...)` | Control test execution order |
| 25-26 | `@Mock PersonRepository` | Creates **mock** repository (not real) |
| 28 | `private PersonService pservice` | Service to test (not a mock) |
| 30-34 | `@BeforeEach setUp()` | Creates service with **mock** repository |
| 33 | `pservice = new PersonService(this.prepository)` | Inject mock into service |
| 38 | `@Order(2)` | Run second |
| 39-42 | `testGetAllPersons()` | Test getAllPersons method |
| 40 | `pservice.getAllPersons()` | Call service method |
| 41 | `verify(prepository).findAll()` | Verify repository's findAll() was called |
| 45 | `@Order(1)` | Run first |
| 46-51 | `testaddPerson()` | Test addPerson method |
| 50 | `verify(prepository).save(p)` | Verify save() was called with Person p |

### Why Use @Mock for Service Testing?

> We are not going to "Autowire" PersonRepository because we **don't want to invoke repository methods**. Instead we will use @Mock for PersonRepository.
>
> `getAllPersons()` method of PersonService invokes repository's findAll() method, but **repository's method already we've tested**. So we need to test **only** `getAllPersons()` of PersonService class. That means we need to use Mockito inside PersonService class.

---

## Complete Test Classes

### Calculator.java (Simple Test Example)

```java
package com.example.demo;

public class Calculator 
{
    public int doSum(int a, int b, int c)
    {
        return a + b + c;
    }
    
    public int doMultiplication(int a, int b)
    {
        return a * b;
    }
}
```

### JUnitProApplicationTests.java

```java
package com.example.demo;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class JUnitProApplicationTests 
{
    private Calculator c = new Calculator();
    
    @Test
    void contextLoads() 
    {
    }
    
    @Test
    void testSum()
    {
        int expectedResult = 22;
        int actualResult = c.doSum(12, 3, 7);
        
        assertThat(actualResult).isEqualTo(expectedResult);
    }
    
    @Test
    void testMultiplication()
    {
        int expectedResult = 20;  // Note: 4*5=20, not 6
        int actualResult = c.doMultiplication(4, 5);
        assertThat(actualResult).isEqualTo(expectedResult);
    }
}
```

---

## Execution Flow

### Repository Test Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│               REPOSITORY TEST EXECUTION FLOW                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. @SpringBootTest loads full application context                   │
│       ├─ Connects to real MySQL database                             │
│       ├─ Creates all Spring beans                                    │
│       └─ Injects real PersonRepository                               │
│                                                                      │
│  2. @TestMethodOrder ensures tests run in order                      │
│                                                                      │
│  3. Test: testgetAllPersons() [Order 1]                              │
│       ├─ @BeforeEach: "Setting up" printed                           │
│       ├─ repository.findAll() → Real SQL query executed              │
│       ├─ assertThat(mylist.size()).isGreaterThan(0)                  │
│       └─ PASS if database has records                                │
│                                                                      │
│  4. Test: testIsPersonExistsById() [Order 2]                         │
│       ├─ @BeforeEach: "Setting up" printed                           │
│       ├─ Create Person, save to database                             │
│       ├─ repository.isPersonExistsById() → Custom JPQL               │
│       ├─ assertThat(result).isTrue()                                 │
│       └─ PASS if person found                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Service Test Flow (With Mock)

```
┌─────────────────────────────────────────────────────────────────────┐
│                  SERVICE TEST EXECUTION FLOW                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. @ExtendWith(MockitoExtension.class) initializes Mockito          │
│                                                                      │
│  2. @Mock PersonRepository → Creates mock (NOT real repository)     │
│                                                                      │
│  3. @BeforeEach setUp()                                              │
│       └─ pservice = new PersonService(mockRepository)                │
│       └─ Service now uses MOCK, not real DB                          │
│                                                                      │
│  4. Test: testaddPerson() [Order 1]                                  │
│       ├─ Create Person object                                        │
│       ├─ pservice.addPerson(p)                                       │
│       │     └─ Calls mockRepository.save(p)                          │
│       │     └─ NO real database operation                            │
│       ├─ verify(prepository).save(p)                                 │
│       │     └─ Checks that save() was called with Person p           │
│       └─ PASS if save() was called                                   │
│                                                                      │
│  5. Test: testGetAllPersons() [Order 2]                              │
│       ├─ pservice.getAllPersons()                                    │
│       │     └─ Calls mockRepository.findAll()                        │
│       │     └─ Returns empty list (default mock behavior)            │
│       ├─ verify(prepository).findAll()                               │
│       │     └─ Checks that findAll() was called                      │
│       └─ PASS if findAll() was called                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Additional Testing Tips

### AssertJ Library

**Purpose:** Fluent assertions library with better readability than JUnit assertions.

**Maven Dependency:**
```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.24.2</version>
    <scope>test</scope>
</dependency>
```

**Usage:**
```java
import static org.assertj.core.api.Assertions.assertThat;

assertThat(actualResult).isEqualTo(expectedResult);
assertThat(result).isTrue();
assertThat(list.size()).isGreaterThan(0);
assertThat(string).startsWith("Hello");
```

### Test Execution Order

**Dependency:**
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
</dependency>
```

**Usage:**
```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class MyTest {
    
    @Test
    @Order(1)
    void firstTest() { }
    
    @Test
    @Order(2)
    void secondTest() { }
}
```

---

## Interview Questions

### Q1: What is the difference between @SpringBootTest and @DataJpaTest?
**Answer:**
- `@SpringBootTest`: Loads full application context, all beans
- `@DataJpaTest`: Loads only JPA-related components (repositories), uses embedded database

### Q2: Why use @Mock instead of @Autowired for service testing?
**Answer:** Because we want to test the service **in isolation**, without actually calling repository methods. We've already tested repository separately, so we mock it to focus only on service logic.

### Q3: What does verify() do in Mockito?
**Answer:** `verify()` checks that a specific method was called on a mock object. It doesn't check return values; it checks **invocation**.

### Q4: Why do we need constructor injection for testing?
**Answer:** Constructor injection allows us to pass mock objects during testing. If we only use field injection with @Autowired, we cannot easily substitute mocks.

### Q5: What happens if @SpringBootTest is not added to repository test?
**Answer:** The test will fail with NullPointerException because the repository reference will be null (Spring context not loaded, dependency injection not performed).

---

## Summary

| Layer | Test Approach | Key Annotations |
|-------|---------------|-----------------|
| **Repository** | Real database | `@SpringBootTest`, `@Autowired` |
| **Service** | Mock dependencies | `@Mock`, `@ExtendWith(MockitoExtension.class)` |
| **Controller** | Mock service | `@WebMvcTest`, `MockMvc` |

| Concept | Description |
|---------|-------------|
| `@SpringBootTest` | Loads full Spring context |
| `@Mock` | Creates mock object |
| `@BeforeEach` | Setup before each test |
| `verify()` | Checks method invocation |
| `assertThat()` | AssertJ fluent assertion |
| `@Order` | Controls test execution order |

---

**Previous:** [04_Mockito_Mocking_Framework.md](./04_Mockito_Mocking_Framework.md)  
**Next:** [06_Spring_Boot_Excel_Upload_to_MySQL.md](./06_Spring_Boot_Excel_Upload_to_MySQL.md)
