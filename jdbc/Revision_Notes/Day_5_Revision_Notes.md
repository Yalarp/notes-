# Day 5: JSP Actions, EL, JSTL & Design Patterns

## 1. JSP Standard Actions
Tags to perform common tasks without Java code (Scriptlets).
*   `<jsp:include page="file.jsp" />`: Includes content at **runtime**.
*   `<jsp:forward page="file.jsp" />`: Forwards request.
*   `<jsp:useBean id="user" class="com.User" scope="request" />`: Creates/Locates a Java Bean.
*   `<jsp:setProperty name="user" property="*" />`: Sets bean properties from request parameters.
*   `<jsp:getProperty name="user" property="name" />`: Prints bean property.

---

## 2. Expression Language (EL)
Simplifies accessing data (Attributes, Params, Beans) in JSP.
*   **Syntax**: `${expression}`
*   **Null-Safe**: Prints nothing instead of `NullPointerException`.
*   **Scope Search Order**: Page -> Request -> Session -> Application.

### Accessing Data
*   **Bean Property**: `${user.name}` (Calls `user.getName()`)
*   **Map/List**: `${map['key']}` or `${list[0]}`
*   **Parameters**: `${param.username}` (Equivalent to `request.getParameter("username")`)

### Implicit Objects in EL
1.  `pageScope`, `requestScope`, `sessionScope`, `applicationScope` (Maps for attributes).
2.  `param`, `paramValues` (Request parameters).
3.  `header`, `headerValues` (HTTP Headers).
4.  `cookie` (Cookies).
5.  `initParam` (Context init params).
6.  `pageContext` (Access to request, session, etc.).

---

## 3. JSTL (JSP Standard Tag Library)
A set of custom tags for common tasks (Loops, If-Else, Formatting).
*   **Setup**: Add `jstl.jar` to `WEB-INF/lib`.
*   **Taglib Directive**:
    *   **Tomcat 9**: `<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>`
    *   **Tomcat 10**: `<%@ taglib uri="jakarta.tags.core" prefix="c" %>`

### Common Tags (`Core`)
1.  **Output**: `<c:out value="${user.name}" default="Guest" />`
2.  **Set Var**: `<c:set var="age" value="25" scope="session" />`
3.  **If**:
    ```xml
    <c:if test="${user.age > 18}">
        <p>Adult</p>
    </c:if>
    ```
4.  **Loop (ForEach)**:
    ```xml
    <c:forEach var="item" items="${myList}">
        <p>${item}</p>
    </c:forEach>
    ```

---

## 4. Design Patterns (Structural & Structural/Creational)
*   **Builder Pattern**:
    *   Separates construction of a complex object from its representation.
    *   Useful when an object has many parameters (some optional).
    *   Example: `new Cake.Builder().sugar(1).butter(0.5).build();`
*   **Prototype Pattern**:
    *   Creates new objects by copying an existing object (Cloning).
    *   Useful when object creation is expensive.
*   **Bridge Pattern**:
    *   Decouples abstraction from implementation so they vary independently.
    *   Example: `Vehicle` (Abstract) has `Engine` (Interface). `Car` uses `PetrolEngine`.
*   **Proxy Pattern**:
    *   Provides a placeholder/surrogate for another object to control access.
    *   **RMI (Remote Method Invocation)** uses Proxy (Stub on client, Skeleton on server) to call methods on remote objects.

## 5. Business Logic Access
*   **Best Practice**: Separate Business Logic from Presentation (Servlet/JSP).
*   **Approach**:
    1.  Create **POJO** (Plain Old Java Object) for logic.
    2.  Package as JAR.
    3.  Add to `WEB-INF/lib`.
    4.  Call logical methods from Servlet: `mypack.Logic.process()`.
