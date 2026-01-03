# Mockito Mocking Framework

## Table of Contents
1. [Introduction to Mocking](#introduction-to-mocking)
2. [What is Mockito?](#what-is-mockito)
3. [Setting Up Mockito](#setting-up-mockito)
4. [Core Mockito Concepts](#core-mockito-concepts)
5. [Complete Example: Authentication System](#complete-example-authentication-system)
6. [Execution Flow](#execution-flow)
7. [Interview Questions](#interview-questions)

---

## Introduction to Mocking

### What is Mocking?

**Definition:** Mocking is a testing technique where **real objects are replaced with fake objects** that simulate the behavior of real objects in controlled ways.

### Why Should We Mock?

| Reason | Description |
|--------|-------------|
| **Isolation** | Test a class without its dependencies |
| **Speed** | Avoid slow operations (database, network) |
| **Control** | Define exact behavior of dependencies |
| **Reliability** | No external service failures affect tests |
| **Focus** | Test only the code under test, not dependencies |

### Real Object vs Mock Object

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REAL OBJECT vs MOCK OBJECT                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  REAL OBJECT                      MOCK OBJECT                        │
│  ────────────                     ───────────                        │
│  ┌─────────────────┐              ┌─────────────────┐               │
│  │ DatabaseService │              │ Mock<Database>  │               │
│  ├─────────────────┤              ├─────────────────┤               │
│  │ - Connects to DB│              │ - No real DB    │               │
│  │ - Runs queries  │              │ - Returns fake  │               │
│  │ - Slow/External │              │   data you set  │               │
│  └─────────────────┘              └─────────────────┘               │
│                                                                      │
│  Use When:                        Use When:                          │
│  - Integration testing            - Unit testing                     │
│  - End-to-end testing             - Testing in isolation             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## What is Mockito?

**Definition:** Mockito is a **popular open-source mocking framework for Java** that allows creating mock objects for unit testing.

### Key Features
| Feature | Description |
|---------|-------------|
| **Easy Syntax** | Fluent API for creating and configuring mocks |
| **Verification** | Verify method calls on mock objects |
| **Argument Matching** | Flexible matchers for method arguments |
| **Annotations** | `@Mock`, `@InjectMocks`, `@Spy` |
| **Integration** | Works with JUnit 4, JUnit 5, TestNG |

---

## Setting Up Mockito

### Maven Dependency

```xml
<dependencies>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-junit-jupiter</artifactId>
        <version>4.5.1</version>
    </dependency>
</dependencies>
```

### Build Path Configuration
```
Project → Build Path → Add Library → JUnit 5
```

---

## Core Mockito Concepts

### 1. Creating Mocks

#### Using Mockito.mock()

```java
// Create a mock of an interface
AuthenticatorInterface authenticatorMock = Mockito.mock(AuthenticatorInterface.class);
```

**What happens:**
> `mock()` method of Mockito will:
> - a) Generate Mock of AuthenticatorInterfaceImpl
> - b) Instantiate this mock
>
> That means we can say that "authenticatorMock" reference refers to the mock object (mock implementation of AuthenticatorInterface)

#### Using @Mock Annotation

```java
@ExtendWith(MockitoExtension.class)
class MyTest {
    
    @Mock
    private AuthenticatorInterface authenticatorMock;
    
    // Mock is automatically created and injected
}
```

### 2. Stubbing with when().thenReturn()

**Purpose:** Define what the mock should return when a method is called.

```java
// When authenticateUser is called with these arguments, return true
when(authenticatorMock.authenticateUser("admin", "password123"))
    .thenReturn(true);

// Now calling the method returns the stubbed value
boolean result = authenticatorMock.authenticateUser("admin", "password123");
// result = true
```

### 3. Verification with verify()

**Purpose:** Check that a method was called with specific arguments.

```java
// Call the method
service.getAllPersons();

// Verify that the repository's findAll() was called
verify(repository).findAll();
```

### Core Methods Summary

| Method | Purpose | Example |
|--------|---------|---------|
| `mock(Class)` | Create mock object | `mock(Service.class)` |
| `when(mock.method()).thenReturn(value)` | Define return value | `when(mock.get()).thenReturn("data")` |
| `verify(mock).method()` | Verify method was called | `verify(mock).save(entity)` |
| `verify(mock, times(n))` | Verify call count | `verify(mock, times(2)).get()` |
| `doThrow().when(mock)` | Throw exception | `doThrow(ex).when(mock).delete()` |

---

## Complete Example: Authentication System

### Project Structure
```
mypack/
├── AuthenticatorInterface.java      (Interface)
├── AuthenticatorInterfaceImpl.java  (Real implementation)
├── AuthenticatorApplication.java    (Application using interface)
└── AuthenticatorApplicationTest.java (Test with mock)
```

### AuthenticatorInterface.java

```java
package mypack;

public interface AuthenticatorInterface
{
    public boolean authenticateUser(String username, String password);
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3 | `public interface AuthenticatorInterface` | Interface defining authentication contract |
| 5 | `boolean authenticateUser(...)` | Method to authenticate user with username/password |

### AuthenticatorInterfaceImpl.java

```java
package mypack;

public class AuthenticatorInterfaceImpl implements AuthenticatorInterface {

    @Override
    public boolean authenticateUser(String username, String password) {
        System.out.println("inside authenticateUser method");
        if(username.equalsIgnoreCase("scott") && password.equals("tiger"))
            return true;
        else
            return false;
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 3 | `implements AuthenticatorInterface` | Provides concrete implementation |
| 6-12 | `authenticateUser(...)` | Real authentication logic |
| 7 | `System.out.println(...)` | Logs that method was called (for debugging) |
| 8 | `username.equalsIgnoreCase("scott")` | Case-insensitive username check |
| 8 | `password.equals("tiger")` | Case-sensitive password check |
| 9-11 | `if...return...else` | Returns true if credentials match |

### AuthenticatorApplication.java

```java
package mypack;

public class AuthenticatorApplication {
     
    private AuthenticatorInterface authenticator;
 
    /**
     * AuthenticatorApplication constructor.
     *
     * @param authenticator Authenticator interface implementation.
     */
    public AuthenticatorApplication(AuthenticatorInterface authenticator) {
        this.authenticator = authenticator;
    }
 
    /**
     * Tries to authenticate an user with the received user name and password, with the received
     * AuthenticatorInterface interface implementation in the constructor.
     *
     * @param username The user name to authenticate.
     * @param password The password to authenticate the user.
     * @return True if the user has been authenticated; false if it has not.
     */
    public boolean authenticate(String username, String password) {
        boolean authenticated;
         
        authenticated = this.authenticator.authenticateUser(username, password);
         
        return authenticated;
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 5 | `private AuthenticatorInterface authenticator;` | Dependency (interface, not implementation) |
| 12-14 | Constructor | **Constructor Injection** - receives dependency |
| 13 | `this.authenticator = authenticator;` | Store the injected dependency |
| 24-30 | `authenticate(...)` | Business method that uses the dependency |
| 27 | `this.authenticator.authenticateUser(...)` | Delegates to the interface |

**Design Pattern:** This class uses **Dependency Injection** through constructor, making it testable with mocks.

### AuthenticatorApplicationTest.java

```java
package mypack;

import static org.junit.Assert.assertFalse;
import org.junit.Test;
import org.mockito.Mockito;
import static org.mockito.Mockito.when;

public class AuthenticatorApplicationTest {
     
    @Test
    public void testAuthenticate() {
        AuthenticatorInterface authenticatorMock;
        AuthenticatorApplication authenticator;
        String username = "JavaCodeGeeks";
        String password = "unsafePassword";
         
        // Step 1: Create mock of the interface
        authenticatorMock = Mockito.mock(AuthenticatorInterface.class);
        System.out.println(authenticatorMock.getClass().getName());
        
        // Step 2: Inject mock into application
        authenticator = new AuthenticatorApplication(authenticatorMock);
         
        // Step 3a: Stub the mock to return false
        /*when(authenticatorMock.authenticateUser(username, password)).thenReturn(true);
        boolean actual = authenticator.authenticate(username, password);
        System.out.println("actual is\t"+actual);*/
        
            //or
            
        // Step 3b: Stub the mock to return false
        when(authenticatorMock.authenticateUser(username, password)).thenReturn(false);
       
        // Step 4: Call the method under test
        boolean actual = authenticator.authenticate(username, password);
        System.out.println("actual is\t"+actual);
     
        // Step 5: Now let's pass real implementation of AuthenticatorInterface
        authenticator = new AuthenticatorApplication(new AuthenticatorInterfaceImpl());
        boolean b = authenticator.authenticate("scott","tiger");
        System.out.println(b);
    }
}
```

#### Line-by-Line Explanation

| Line | Code | Explanation |
|------|------|-------------|
| 5 | `import org.mockito.Mockito;` | Main Mockito class for mock creation |
| 6 | `import static org.mockito.Mockito.when;` | Static import for when() method |
| 14-15 | Test data | Username and password for testing |
| 18 | `Mockito.mock(AuthenticatorInterface.class)` | Creates mock object |
| 19 | `authenticatorMock.getClass().getName()` | Shows mock class name (proxy class) |
| 22 | `new AuthenticatorApplication(authenticatorMock)` | Inject mock into application |
| 32 | `when(...).thenReturn(false)` | Stub mock to return false |
| 35 | `authenticator.authenticate(...)` | Call method under test |
| 39 | `new AuthenticatorInterfaceImpl()` | Create real implementation |
| 40 | `authenticate("scott","tiger")` | Test with real implementation |

### Test Execution Steps

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MOCK TEST EXECUTION                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PART 1: Testing with Mock                                           │
│  ─────────────────────────                                           │
│                                                                      │
│  Step 1: Create Mock                                                 │
│     authenticatorMock = Mockito.mock(AuthenticatorInterface.class)   │
│         └─ Creates proxy object implementing AuthenticatorInterface  │
│         └─ Mock class: mypack.AuthenticatorInterface$MockitoMock     │
│                                                                      │
│  Step 2: Inject Mock                                                 │
│     authenticator = new AuthenticatorApplication(authenticatorMock)  │
│         └─ Application now uses mock, not real authenticator         │
│                                                                      │
│  Step 3: Stub Behavior                                               │
│     when(authenticatorMock.authenticateUser("JavaCodeGeeks",         │
│          "unsafePassword")).thenReturn(false);                       │
│         └─ Tell mock what to return when called with these args      │
│                                                                      │
│  Step 4: Call Method Under Test                                      │
│     authenticator.authenticate("JavaCodeGeeks", "unsafePassword")    │
│         └─ authenticate() calls authenticatorMock.authenticateUser() │
│         └─ Mock returns stubbed value: false                         │
│         └─ Result: actual = false                                    │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PART 2: Testing with Real Implementation                            │
│  ─────────────────────────────────────────                           │
│                                                                      │
│  Step 5: Create Real Implementation                                  │
│     authenticator = new AuthenticatorApplication(                    │
│                         new AuthenticatorInterfaceImpl())            │
│         └─ Now using REAL authenticator, not mock                    │
│                                                                      │
│  Step 6: Call Method                                                 │
│     authenticator.authenticate("scott", "tiger")                     │
│         └─ Real authenticateUser() is called                         │
│         └─ Prints: "inside authenticateUser method"                  │
│         └─ Checks: scott/tiger matches                               │
│         └─ Returns: true                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Expected Console Output

```
mypack.AuthenticatorInterface$MockitoMock$123456789
actual is	false
inside authenticateUser method
true
```

### Key Differences: Mock vs Real

| Aspect | Mock | Real Implementation |
|--------|------|---------------------|
| **Object Type** | Proxy/Mock class | AuthenticatorInterfaceImpl |
| **Behavior** | Returns stubbed values | Executes real logic |
| **S.o.p output** | NO output | "inside authenticateUser method" |
| **Return value** | Controlled by when().thenReturn() | Based on actual logic |
| **DB/Network** | None | Would connect if implemented |

---

## Execution Flow

### Complete Mockito Test Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MOCKITO TEST LIFECYCLE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. JUnit starts test class                                          │
│       │                                                              │
│       ▼                                                              │
│  2. @ExtendWith(MockitoExtension.class) initializes Mockito          │
│       │                                                              │
│       ▼                                                              │
│  3. @Mock fields are populated with mock objects                     │
│       │                                                              │
│       ▼                                                              │
│  4. @BeforeEach method runs                                          │
│       └─ Inject mocks into class under test                         │
│       │                                                              │
│       ▼                                                              │
│  5. @Test method runs                                                │
│       ├─ Step A: Arrange (setup, stub mocks)                         │
│       ├─ Step B: Act (call method under test)                        │
│       └─ Step C: Assert (verify behavior)                            │
│       │                                                              │
│       ▼                                                              │
│  6. verify() checks method invocations                               │
│       │                                                              │
│       ▼                                                              │
│  7. Test completes (pass/fail)                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Q1: What is the difference between mock and spy?
**Answer:**
- **Mock:** Completely fake object, all methods return default values unless stubbed
- **Spy:** Partial mock, real methods are called unless specifically stubbed

### Q2: When would you use when().thenReturn() vs doReturn().when()?
**Answer:**
- `when().thenReturn()`: Standard way, method is actually called during stubbing
- `doReturn().when()`: Used for void methods or when actual method call should be avoided during stubbing

### Q3: What is the purpose of verify()?
**Answer:** `verify()` checks that a method was called on a mock object with specific arguments and optionally a specific number of times.

### Q4: Can we mock a class instead of an interface?
**Answer:** Yes, Mockito can mock classes (not just interfaces). However, it cannot mock final classes or methods by default (unless using mockito-inline).

### Q5: What does @InjectMocks do?
**Answer:** `@InjectMocks` creates an instance of the class and injects all `@Mock` fields into it (via constructor, setter, or field injection).

---

## Summary

| Concept | Description |
|---------|-------------|
| **Mocking** | Replacing real objects with fake controlled objects |
| **Mockito** | Java mocking framework |
| `mock(Class)` | Creates mock object |
| `when().thenReturn()` | Defines mock behavior |
| `verify()` | Checks method was called |
| **Dependency Injection** | Design pattern enabling testability |
| **Constructor Injection** | Pass dependencies via constructor |

---

**Previous:** [03_JUnit_Exception_Testing_and_Suites.md](./03_JUnit_Exception_Testing_and_Suites.md)  
**Next:** [05_Spring_Boot_Unit_Testing_Complete.md](./05_Spring_Boot_Unit_Testing_Complete.md)
