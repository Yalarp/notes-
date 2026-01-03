# JUnit Fundamentals and Annotations

## Table of Contents
1. [Introduction to Unit Testing](#introduction-to-unit-testing)
2. [What is JUnit?](#what-is-junit)
3. [JUnit 4 vs JUnit 5](#junit-4-vs-junit-5)
4. [Setting Up JUnit Project](#setting-up-junit-project)
5. [JUnit Annotations](#junit-annotations)
6. [Assert Methods](#assert-methods)
7. [Demo 1: Basic Test Case](#demo-1-basic-test-case)
8. [Demo 2: JUnitCore with Log4j](#demo-2-junitcore-with-log4j)
9. [Execution Flow](#execution-flow)
10. [Interview Questions](#interview-questions)

---

## Introduction to Unit Testing

### What is Unit Testing?

**Definition:** A unit test is a way of testing a unit - **the smallest piece of code that can be logically isolated in the system**.

In Java, a "unit" typically refers to:
- A single method
- A class
- A small group of closely related classes

### Why Unit Testing?

| Benefit | Description |
|---------|-------------|
| **Early Bug Detection** | Find bugs before integration |
| **Code Quality** | Forces modular, testable code design |
| **Documentation** | Tests serve as living documentation |
| **Refactoring Safety** | Confident code changes with test coverage |
| **Regression Prevention** | Ensure changes don't break existing functionality |

### Testing Pyramid
```
           /\
          /  \
         /E2E \         ← Few, Slow, Expensive
        /------\
       /  Integ \       ← Integration Tests  
      /----------\
     /    UNIT    \     ← Many, Fast, Cheap
    /--------------\
```

---

## What is JUnit?

**Definition:** JUnit is a **unit testing framework for Java** that provides annotations and assertions for writing and running tests.

### Key Features
- Open-source testing framework
- Provides annotations for test lifecycle
- Rich set of assertion methods
- Integration with build tools (Maven, Gradle)
- IDE support (Eclipse, IntelliJ IDEA)

### JUnit Versions
| Version | Release | Key Features |
|---------|---------|--------------|
| JUnit 4 | 2006 | Annotations, Assert class |
| JUnit 5 | 2017 | Modular architecture, new annotations |

---

## JUnit 4 vs JUnit 5

### Comparison Table

| Aspect | JUnit 4 | JUnit 5 |
|--------|---------|---------|
| **Package** | `org.junit` | `org.junit.jupiter.api` |
| **Architecture** | Monolithic | Modular (Platform + Jupiter + Vintage) |
| **Annotations** | `@Before`, `@After` | `@BeforeEach`, `@AfterEach` |
| **All Tests** | `@BeforeClass`, `@AfterClass` | `@BeforeAll`, `@AfterAll` |
| **Ignore Test** | `@Ignore` | `@Disabled` |
| **Expected Exception** | `@Test(expected=...)` | `assertThrows()` |
| **Assert class** | `org.junit.Assert` | `org.junit.jupiter.api.Assertions` |

### Annotation Mapping

| JUnit 4 | JUnit 5 | Purpose |
|---------|---------|---------|
| `@Test` | `@Test` | Marks a test method |
| `@Before` | `@BeforeEach` | Runs before each test |
| `@After` | `@AfterEach` | Runs after each test |
| `@BeforeClass` | `@BeforeAll` | Runs once before all tests |
| `@AfterClass` | `@AfterAll` | Runs once after all tests |
| `@Ignore` | `@Disabled` | Skips the test |
| `@RunWith` | `@ExtendWith` | Custom runner/extension |

---

## Setting Up JUnit Project

### Step-by-Step Project Setup

#### Step 1: Create Java Project
```
File → New → Java Project
    Project Name: JUnitDemo
    → Finish
```

#### Step 2: Create Package and Class
```
Right-click project → New → Package
    Name: mypack

Right-click mypack → New → Class
    Name: Sample
```

#### Step 3: Convert to Maven Project
```
Right-click project → Configure → Convert to Maven Project
```

#### Step 4: Add JUnit 4 Dependency

Add to `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

#### Step 5: Create Test Class
```
Right-click mypack → New → JUnit Test Case
    
    ✓ New JUnit Jupiter test  (for JUnit 5)
    OR
    ✓ New JUnit 4 test        (for JUnit 4)
    
    Name: SampleTest
    Class under test: mypack.Sample
    
    → Next
    
    Select methods to test
    → Finish
```

---

## JUnit Annotations

### Core Annotations Reference

| Annotation | JUnit Version | Description |
|------------|---------------|-------------|
| `@Test` | 4 & 5 | Marks method as test case |
| `@BeforeEach` | 5 | Runs before each test method |
| `@AfterEach` | 5 | Runs after each test method |
| `@BeforeAll` | 5 | Runs once before all tests (must be static) |
| `@AfterAll` | 5 | Runs once after all tests (must be static) |
| `@Disabled` | 5 | Disables a test method/class |
| `@DisplayName` | 5 | Custom display name for test |
| `@Nested` | 5 | Nested test class |
| `@Order` | 5 | Specifies test execution order |
| `@TestMethodOrder` | 5 | Configures test method ordering |

### Lifecycle Annotations Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TEST CLASS LIFECYCLE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  @BeforeAll (static) ─── Runs ONCE before all tests                 │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  @BeforeEach ─── Runs before EACH test                        │   │
│  │       │                                                        │   │
│  │       ▼                                                        │   │
│  │  @Test ─── testMethod1()                                       │   │
│  │       │                                                        │   │
│  │       ▼                                                        │   │
│  │  @AfterEach ─── Runs after EACH test                          │   │
│  └──────────────────────────────────────────────────────────────┘   │
│       │                                                              │
│       ▼ (repeat for each test method)                               │
│                                                                      │
│  @AfterAll (static) ─── Runs ONCE after all tests                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Assert Methods

### JUnit Assert Class

**Package:** `org.junit.Assert` (JUnit 4) or `org.junit.jupiter.api.Assertions` (JUnit 5)

### Common Assert Methods

| Method | Description | Example |
|--------|-------------|---------|
| `assertEquals(expected, actual)` | Checks if two values are equal | `assertEquals(4, 2+2)` |
| `assertTrue(condition)` | Checks if condition is true | `assertTrue(x > 0)` |
| `assertFalse(condition)` | Checks if condition is false | `assertFalse(list.isEmpty())` |
| `assertNull(object)` | Checks if object is null | `assertNull(result)` |
| `assertNotNull(object)` | Checks if object is not null | `assertNotNull(response)` |
| `assertSame(obj1, obj2)` | Checks same object reference | `assertSame(singleton, getInstance())` |
| `assertNotSame(obj1, obj2)` | Checks different references | `assertNotSame(obj1, obj2)` |
| `assertArrayEquals(arr1, arr2)` | Compares arrays element by element | `assertArrayEquals(expected, actual)` |
| `fail()` | Forces test to fail | `fail("Should not reach here")` |

### AssertJ Library (Alternative)

**Package:** `org.assertj.core.api.Assertions`

```java
import static org.assertj.core.api.Assertions.assertThat;

// Fluent assertion style
assertThat(actualResult).isEqualTo(expectedResult);
assertThat(result).isTrue();
assertThat(list.size()).isGreaterThan(0);
```

---

## Demo 1: Basic Test Case

### Project Structure
```
mypack/
├── Sample.java          (Class under test)
└── SampleTest.java      (Test class)
```

### Sample.java (Class Under Test)

```java
package mypack;

public class Sample 
{
    public int sqr(int k)
    {
        return k*k;
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 1 | `package mypack;` | Package declaration |
| 3 | `public class Sample` | Class to be tested |
| 5-8 | `public int sqr(int k)` | Method that squares a number |
| 7 | `return k*k;` | Returns square of input |

### SampleTest.java (Test Class)

```java
package mypack;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import java.util.*;

class SampleTest {

    private Scanner sc = new Scanner(System.in);
    private Sample s = new Sample();
    
    @Test
    public void testSqr() {
        System.out.println("Enter input");
        int input = sc.nextInt();
        System.out.println("Enter expected Result");
        int result = sc.nextInt();
        assertEquals(result, s.sqr(input));
    }
    
    @BeforeEach
    void setUp() throws Exception {
        System.out.println("Before Testing");
    }

    @AfterEach
    void tearDown() throws Exception {
        System.out.println("After Testing");
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3 | `import static org.junit.jupiter.api.Assertions.*;` | Static import for assertions |
| 5-7 | Import statements | JUnit 5 annotations |
| 10 | `class SampleTest` | Test class (no public required in JUnit 5) |
| 12 | `private Scanner sc = new Scanner(System.in);` | Scanner for user input |
| 13 | `private Sample s = new Sample();` | Instance of class under test |
| 15-22 | `@Test public void testSqr()` | Test method for sqr() |
| 21 | `assertEquals(result, s.sqr(input))` | Verify expected equals actual |
| 24-27 | `@BeforeEach setUp()` | Runs before each test |
| 29-32 | `@AfterEach tearDown()` | Runs after each test |

### Running the Test
```
Right-click on SampleTest → Run As → JUnit Test
```

### Expected Console Output (if input=5, expected=25)
```
Before Testing
Enter input
5
Enter expected Result
25
After Testing
```

### Test Result Indicators
| Color | Meaning |
|-------|---------|
| 🟢 Green Bar | All tests passed |
| 🔴 Red Bar | One or more tests failed |

---

## Demo 2: JUnitCore with Log4j

### Purpose
Demonstrate programmatic test execution using `JUnitCore` class with Log4j logging.

### What is JUnitCore?

> JUnitCore is an inbuilt class in JUnit package and it is based on **facade design pattern**. JUnitCore class is used to run only specified test classes.

`org.junit.runner.JUnitCore` class is responsible for executing tests. The `runClasses()` method enables running one or more test classes which yield a `Result` object (`org.junit.runner.Result`).

### Maven Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

### Project Structure
```
mypack/
├── Sample.java
├── SampleTest.java
├── SampleTestRunner.java
└── log4j.properties (in src/main/resources)
```

### Sample.java

```java
package mypack;

public class Sample 
{
    public int sqr(int k)
    {
        return k*k;
    }
}
```

### SampleTest.java (JUnit 4 Style)

```java
package mypack;

import static org.junit.Assert.*;

import java.util.Scanner;

import org.junit.Test;

public class SampleTest 
{
    private Scanner sc = new Scanner(System.in);
    private Sample s = new Sample();
    
    @Test
    public void testSqr() {
        System.out.println("Enter input");
        int input = sc.nextInt();
        System.out.println("Enter expected Result");
        int result = sc.nextInt();
        assertEquals(result, s.sqr(input));
    }
}
```

### SampleTestRunner.java

```java
package mypack;

import org.apache.log4j.Logger;
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure;

public class SampleTestRunner
{
    private static Logger logger = Logger.getLogger(SampleTestRunner.class);
    
    public static void main(String[] args)
    {
        // Run the test class and get result
        Result result = JUnitCore.runClasses(SampleTest.class);
        
        // Log all failures
        for (Failure failure : result.getFailures()) {
            logger.error(failure.toString());
        }
        
        // Log success message if all tests passed
        if(result.wasSuccessful())
            logger.info("it was successful");
    }	
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3 | `import org.apache.log4j.Logger;` | Log4j logging framework |
| 4 | `import org.junit.runner.JUnitCore;` | JUnit runner class |
| 5 | `import org.junit.runner.Result;` | Test result container |
| 6 | `import org.junit.runner.notification.Failure;` | Failure information |
| 10 | `Logger.getLogger(SampleTestRunner.class)` | Create logger for this class |
| 15 | `JUnitCore.runClasses(SampleTest.class)` | Execute tests in SampleTest |
| 18-20 | `for (Failure failure : ...)` | Iterate through failures |
| 19 | `logger.error(failure.toString())` | Log each failure as error |
| 23-24 | `if(result.wasSuccessful())` | Check if all tests passed |

### log4j.properties

```properties
# Root logger option
log4j.rootLogger=INFO, stdout, file

# Console appender
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n

# File appender
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=logs/app.log
log4j.appender.file.MaxFileSize=5MB
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

**Location:** `src/main/resources/log4j.properties`

### Two Ways to Run Test Cases

| Method | How to Execute |
|--------|----------------|
| **1. IDE** | Right-click on `SampleTest` → Run As → JUnit Test |
| **2. Programmatic** | Run `SampleTestRunner.main()` which calls `JUnitCore.runClasses(SampleTest.class)` |

### Running the Test
```
Right-click on SampleTestRunner → Run As → Java Application
```

---

## Execution Flow

### JUnitCore Execution Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    JUNITCORE EXECUTION FLOW                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. main() starts in SampleTestRunner                                │
│       │                                                              │
│       ▼                                                              │
│  2. JUnitCore.runClasses(SampleTest.class)                          │
│       │                                                              │
│       ├─ Creates JUnitCore instance                                  │
│       ├─ Creates Request for SampleTest                              │
│       ├─ Runs all @Test methods                                      │
│       │                                                              │
│       ▼                                                              │
│  3. For each @Test method:                                           │
│       ├─ Execute @BeforeEach (if present)                            │
│       ├─ Execute test method                                         │
│       ├─ Capture pass/fail result                                    │
│       └─ Execute @AfterEach (if present)                             │
│       │                                                              │
│       ▼                                                              │
│  4. Result object returned                                           │
│       ├─ getFailures() → List of failures                            │
│       └─ wasSuccessful() → true/false                                │
│       │                                                              │
│       ▼                                                              │
│  5. Log results using Log4j                                          │
│       ├─ logger.error() for failures                                 │
│       └─ logger.info() for success                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Q1: What is the difference between @Before and @BeforeEach?
**Answer:**
- `@Before` is JUnit 4 annotation
- `@BeforeEach` is JUnit 5 annotation
- Both serve the same purpose: run before each test method

### Q2: What is the purpose of JUnitCore class?
**Answer:** JUnitCore is a facade class that provides programmatic way to run JUnit tests. Its `runClasses()` method executes tests and returns a Result object containing success/failure information.

### Q3: Why use assertEquals instead of == for comparison?
**Answer:**
- `assertEquals()` uses `.equals()` method for comparison
- Provides meaningful error messages on failure
- Works correctly with objects (not just primitives)
- Part of standard testing practice

### Q4: What happens if @BeforeEach throws an exception?
**Answer:** The test method will not execute, and the test will be marked as failed. The @AfterEach method will still be called for cleanup.

### Q5: Can we run multiple test classes together?
**Answer:** Yes, using either:
1. `JUnitCore.runClasses(Test1.class, Test2.class)`
2. Test Suites with `@Suite.SuiteClasses`

---

## Summary

| Concept | Description |
|---------|-------------|
| **Unit Testing** | Testing smallest isolated code units |
| **JUnit** | Java testing framework |
| **@Test** | Marks method as test case |
| **@BeforeEach** | Setup before each test |
| **@AfterEach** | Cleanup after each test |
| **assertEquals()** | Asserts two values are equal |
| **JUnitCore** | Programmatic test runner |
| **Result** | Contains test execution results |

---

**Previous:** [01_Internationalization_I18N_Complete_Guide.md](./01_Internationalization_I18N_Complete_Guide.md)  
**Next:** [03_JUnit_Exception_Testing_and_Suites.md](./03_JUnit_Exception_Testing_and_Suites.md)
