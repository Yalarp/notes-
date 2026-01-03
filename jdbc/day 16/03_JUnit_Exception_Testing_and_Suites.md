# JUnit Exception Testing and Test Suites

## Table of Contents
1. [Exception Testing](#exception-testing)
2. [Demo 3: Exception Testing Scenarios](#demo-3-exception-testing-scenarios)
3. [Test Suites](#test-suites)
4. [Demo 4: Complete Suite Test](#demo-4-complete-suite-test)
5. [Three Ways to Run Tests](#three-ways-to-run-tests)
6. [Execution Flow](#execution-flow)
7. [Interview Questions](#interview-questions)

---

## Exception Testing

### What is Exception Testing?

**Definition:** Exception testing verifies that your code correctly throws expected exceptions under specific conditions.

### Why Test Exceptions?
| Reason | Example |
|--------|---------|
| **Validate Error Handling** | Division by zero should throw ArithmeticException |
| **Verify Boundary Conditions** | Accessing index -1 should throw IndexOutOfBoundsException |
| **Ensure Type Safety** | Invalid cast should throw ClassCastException |

### JUnit 4 Approach: @Test(expected = Exception.class)

```java
@Test(expected = ArithmeticException.class)
public void testDivisionByZero() {
    int result = 10 / 0;  // Should throw ArithmeticException
}
```

**How it works:**
1. JUnit runs the test method
2. If the specified exception is thrown → Test PASSES ✅
3. If NO exception is thrown → Test FAILS ❌
4. If a DIFFERENT exception is thrown → Test FAILS ❌

### JUnit 5 Approach: assertThrows()

```java
@Test
public void testDivisionByZero() {
    assertThrows(ArithmeticException.class, () -> {
        int result = 10 / 0;
    });
}
```

---

## Demo 3: Exception Testing Scenarios

### Project Structure
```
mypack/
├── Target1.java          (Class under test)
└── TestJunit1.java       (Test class with exception tests)
```

### Target1.java (Class Under Test)

```java
package mypack;

public class Target1 {

    private int num;

    public int getNum() {
        return num;
    }
    
    public int devide(int bynum)
    {
        return num/bynum;
    }
    
    public Target1(int num) {
        super();
        this.num = num;
    }
    
    public Object getObject(int i)
    {
        if(i==1)
        {
            return new String();
        }
        else
        {
            return new Object();
        }
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 5 | `private int num;` | Private field to store number |
| 7-9 | `getNum()` | Getter for num field |
| 11-14 | `devide(int bynum)` | Divides num by bynum (can throw ArithmeticException if bynum=0) |
| 16-19 | `Target1(int num)` | Constructor to initialize num |
| 21-30 | `getObject(int i)` | Returns String if i=1, else returns Object. This creates a ClassCastException scenario |

### TestJunit1.java (Exception Test Class)

```java
package mypack;

import org.junit.Test;
import static org.junit.Assert.assertEquals;

import java.util.ArrayList;

public class TestJunit1 {
        
    int val = 100;      
    Target1 t = new Target1(100);

    @Test(expected = ArithmeticException.class)
    public void testPrintMessage()
    {
        System.out.println("in testprintmessage");
        // t.devide(4);    // This would NOT throw exception - test would FAIL
        t.devide(0);       // This throws ArithmeticException - test PASSES
    }
    
    @Test(expected = IndexOutOfBoundsException.class)
    public void testEmptyList() {
        System.out.println("in testEmptyList");
        new ArrayList<>().get(0);    // Accessing index 0 of empty list - throws exception
        // new ArrayList<>();        // Just creating list - NO exception - test would FAIL
    }
    
    @Test(expected = ClassCastException.class)
    public void tryObject()
    {
        String ref = (String)t.getObject(1);   // Returns String, cast succeeds - test FAILS
        //String ref = (String)t.getObject(2); // Returns Object, cast fails - test would PASS
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 10-11 | Field declarations | Initialize Target1 with value 100 |
| 13-19 | `testPrintMessage()` | Tests ArithmeticException on division by zero |
| 13 | `@Test(expected = ArithmeticException.class)` | Test passes only if this exception is thrown |
| 18 | `t.devide(0)` | 100/0 throws ArithmeticException |
| 21-26 | `testEmptyList()` | Tests IndexOutOfBoundsException on empty list access |
| 24 | `new ArrayList<>().get(0)` | Empty list has no element at index 0 |
| 28-33 | `tryObject()` | Tests ClassCastException on invalid cast |
| 31 | `(String)t.getObject(1)` | Returns String, so cast succeeds - test FAILS! |
| 32 | `(String)t.getObject(2)` | Would return Object, cast to String fails |

### Exception Test Cases Analysis

| Test Method | Expected Exception | Code | Result |
|-------------|-------------------|------|--------|
| `testPrintMessage()` | `ArithmeticException` | `t.devide(0)` → 100/0 | ✅ PASS |
| `testEmptyList()` | `IndexOutOfBoundsException` | `new ArrayList<>().get(0)` | ✅ PASS |
| `tryObject()` | `ClassCastException` | `(String)t.getObject(1)` returns String | ❌ FAIL (no exception) |

### How to Run
```
Right-click on "TestJunit1.java"
    → Run As
        → JUnit Test
```

### Execution Flow for Exception Testing

```
┌─────────────────────────────────────────────────────────────────────┐
│             EXCEPTION TESTING EXECUTION FLOW                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Test: testPrintMessage()                                            │
│  Expected: ArithmeticException                                       │
│       │                                                              │
│       ▼                                                              │
│  1. JUnit starts test                                                │
│  2. t.devide(0) executed                                             │
│       └─ num/bynum = 100/0                                           │
│       └─ ArithmeticException thrown                                  │
│  3. JUnit catches exception                                          │
│  4. Exception matches expected? YES                                  │
│  5. Result: ✅ PASS                                                  │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Test: tryObject()                                                   │
│  Expected: ClassCastException                                        │
│       │                                                              │
│       ▼                                                              │
│  1. JUnit starts test                                                │
│  2. t.getObject(1) executed                                          │
│       └─ i==1 is true                                                │
│       └─ Returns new String()                                        │
│  3. Cast (String)result                                              │
│       └─ String to String - valid cast                               │
│       └─ NO exception thrown                                         │
│  4. Exception expected but not thrown                                │
│  5. Result: ❌ FAIL                                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Test Suites

### What is a Test Suite?

**Definition:** A Test Suite is a collection of test classes that are grouped together to run as a single unit.

### Why Use Test Suites?
| Benefit | Description |
|---------|-------------|
| **Organization** | Group related tests together |
| **Efficiency** | Run multiple test classes at once |
| **CI/CD** | Single entry point for automated testing |
| **Selective Testing** | Run subset of tests by module/feature |

### Creating a Test Suite (JUnit 4)

**Required Annotations:**
```java
@RunWith(Suite.class)
@Suite.SuiteClasses({
   TestClass1.class,
   TestClass2.class,
   TestClass3.class
})
public class TestSuite {
    // Empty class body - just annotations
}
```

---

## Demo 4: Complete Suite Test

### Project Structure
```
mypack/
├── Target.java              (Class under test)
├── TestTarget1.java         (Test class 1)
├── TestTarget2.java         (Test class 2)
├── JUnitTestSuite.java      (Suite class)
├── TestRunner.java          (Programmatic runner)
└── log4j.properties
```

### Target.java (Class Under Test)

```java
package mypack;

public class Target {

    private String message;

    //Constructor
    //@param message to be printed
    public Target(String message){
        this.message = message; 
    }

    // prints the message
    public String printMessage(){
        System.out.println(message);
        return message;
    }   

    // add "Hi!" to the message
    public String salutationMessage(){
        message = "Hi!" + message;
        System.out.println(message);
        return message;
    }   
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 5 | `private String message;` | Private field to store message |
| 9-11 | `Target(String message)` | Constructor to initialize message |
| 14-17 | `printMessage()` | Prints and returns the message |
| 20-24 | `salutationMessage()` | Prepends "Hi!" to message and returns |

### TestTarget1.java

```java
package mypack;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class TestTarget1 {
    String message = "Robert";   
    Target t = new Target(message);
    
    @Test
    public void testPrintMessage() {     
        System.out.println("Inside testPrintMessage()");    
        assertEquals(message, t.printMessage());     
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 7 | `String message = "Robert"` | Expected message value |
| 8 | `Target t = new Target(message)` | Create Target with "Robert" |
| 11-14 | `testPrintMessage()` | Test that printMessage returns correct value |
| 13 | `assertEquals(message, t.printMessage())` | Verify "Robert" equals "Robert" |

### TestTarget2.java

```java
package mypack;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class TestTarget2 {

    String message = "Robert";   
    Target t = new Target(message);
 
    @Test
    public void testSalutationMessage() {
        System.out.println("Inside testSalutationMessage()");
        message = "Hi!" + "Kunal";
        assertEquals(message, t.salutationMessage());
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 8-9 | Field initialization | Same setup as TestTarget1 |
| 12-16 | `testSalutationMessage()` | Test salutationMessage method |
| 14 | `message = "Hi!" + "Kunal"` | Set expected to "Hi!Kunal" |
| 15 | `assertEquals(message, t.salutationMessage())` | But salutationMessage returns "Hi!Robert" |

**Note:** This test will FAIL because:
- Expected: "Hi!Kunal"
- Actual: "Hi!Robert" (from Target's salutationMessage method)

### JUnitTestSuite.java (Test Suite)

```java
package mypack;

import org.junit.runner.RunWith;
import org.junit.runners.Suite;

@RunWith(Suite.class)
@Suite.SuiteClasses({
   TestTarget1.class,
   TestTarget2.class
})
public class JUnitTestSuite {   
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 5 | `@RunWith(Suite.class)` | Use Suite runner instead of default |
| 6-9 | `@Suite.SuiteClasses({...})` | List of test classes to include |
| 7 | `TestTarget1.class` | First test class in suite |
| 8 | `TestTarget2.class` | Second test class in suite |
| 10-11 | `public class JUnitTestSuite` | Empty body - annotations do the work |

### TestRunner.java (Suite Runner)

```java
package mypack;

import org.apache.log4j.Logger;
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure;

public class TestRunner {
    final static Logger logger = Logger.getLogger(TestRunner.class);

    public static void main(String[] args) {
      
        // Run the suite (which includes TestTarget1 and TestTarget2)
        Result result = JUnitCore.runClasses(JUnitTestSuite.class);
        
        // Log all failures
        for (Failure failure : result.getFailures()) {
            logger.error(failure.toString());
        }
        
        // Log success if all tests passed
        if(result.wasSuccessful())
            logger.info("it was successful");
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 9 | `Logger.getLogger(TestRunner.class)` | Create Log4j logger |
| 14 | `JUnitCore.runClasses(JUnitTestSuite.class)` | Run the suite (not individual tests) |
| 17-19 | `for (Failure failure : ...)` | Iterate through all failures |
| 22-23 | `if(result.wasSuccessful())` | Check if all tests in suite passed |

### log4j.properties

```properties
# Root logger option
log4j.rootLogger=INFO, stdout

# Console appender
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

**Location:** `src/main/resources/log4j.properties`

### Maven Dependencies (pom.xml)

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

---

## Three Ways to Run Tests

### Comparison Table

| Method | Command/Action | When to Use |
|--------|----------------|-------------|
| **1. Direct IDE** | Right-click → Run As → JUnit Test | Quick single test execution |
| **2. JUnitCore Single** | `JUnitCore.runClasses(TestClass.class)` | Programmatic single test |
| **3. JUnitCore Suite** | `JUnitCore.runClasses(SuiteClass.class)` | Programmatic multiple tests |

### Detailed Breakdown

#### Method 1: Direct Test Execution
```
Right-click on test class
    → Run As
        → JUnit Test
```

#### Method 2: JUnitCore with Individual Classes
```java
// Run single test class
Result result = JUnitCore.runClasses(SampleTest.class);

// Run multiple test classes
Result result = JUnitCore.runClasses(TestTarget1.class, TestTarget2.class);
```

#### Method 3: JUnitCore with Test Suite
```java
// Define suite class
@RunWith(Suite.class)
@Suite.SuiteClasses({
   TestTarget1.class,
   TestTarget2.class
})
public class JUnitTestSuite {}

// Run the suite
Result result = JUnitCore.runClasses(JUnitTestSuite.class);
```

---

## Execution Flow

### Test Suite Execution Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TEST SUITE EXECUTION FLOW                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. TestRunner.main() starts                                         │
│       │                                                              │
│       ▼                                                              │
│  2. JUnitCore.runClasses(JUnitTestSuite.class)                       │
│       │                                                              │
│       ├─ JUnit reads @RunWith(Suite.class)                           │
│       ├─ JUnit reads @Suite.SuiteClasses                             │
│       └─ Identifies: TestTarget1, TestTarget2                        │
│       │                                                              │
│       ▼                                                              │
│  3. Execute TestTarget1                                              │
│       ├─ testPrintMessage()                                          │
│       ├─ assertEquals("Robert", t.printMessage())                    │
│       └─ Result: ✅ PASS                                             │
│       │                                                              │
│       ▼                                                              │
│  4. Execute TestTarget2                                              │
│       ├─ testSalutationMessage()                                     │
│       ├─ assertEquals("Hi!Kunal", t.salutationMessage())             │
│       ├─ But t.salutationMessage() returns "Hi!Robert"               │
│       └─ Result: ❌ FAIL                                             │
│       │                                                              │
│       ▼                                                              │
│  5. Aggregate Results                                                │
│       ├─ getFailures() → [TestTarget2.testSalutationMessage]         │
│       └─ wasSuccessful() → false                                     │
│       │                                                              │
│       ▼                                                              │
│  6. Log Results                                                      │
│       └─ logger.error("testSalutationMessage: ...")                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Q1: How do you test if a method throws an exception in JUnit 4?
**Answer:**
```java
@Test(expected = SomeException.class)
public void testMethod() {
    // Code that should throw SomeException
}
```

### Q2: What is the difference between @RunWith and @ExtendWith?
**Answer:**
- `@RunWith` is JUnit 4 annotation for custom runners (Suite, Parameterized)
- `@ExtendWith` is JUnit 5 annotation for extensions (MockitoExtension, SpringExtension)

### Q3: What happens if exception test doesn't throw the expected exception?
**Answer:** The test FAILS because JUnit expects the exception but the method completes normally.

### Q4: How to run only specific test classes in a suite?
**Answer:** Modify the `@Suite.SuiteClasses` annotation to include only desired test classes.

### Q5: What is the difference between Result.getFailures() and Result.wasSuccessful()?
**Answer:**
- `getFailures()` returns a List<Failure> of all failed tests
- `wasSuccessful()` returns boolean true only if ALL tests passed

---

## Summary

| Concept | Description |
|---------|-------------|
| **Exception Testing** | Verify code throws expected exceptions |
| `@Test(expected=...)` | JUnit 4 way to expect exceptions |
| **Test Suite** | Group of test classes run together |
| `@RunWith(Suite.class)` | Marks class as test suite |
| `@Suite.SuiteClasses` | Lists test classes in suite |
| **JUnitCore** | Programmatic test runner |
| **Result** | Contains pass/fail information |

---

**Previous:** [02_JUnit_Fundamentals_and_Annotations.md](./02_JUnit_Fundamentals_and_Annotations.md)  
**Next:** [04_Mockito_Mocking_Framework.md](./04_Mockito_Mocking_Framework.md)
