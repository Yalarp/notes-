# Day 4: Attributes, JSP, and Deployment

## 1. Attributes vs. Parameters
| Feature | Parameters | Attributes |
| :--- | :--- | :--- |
| **Type** | String (`String`) | Object (`Object`) |
| **Origin** | Client (HTML Form) or Config (`web.xml`) | Server (Set programmatically) |
| **Modifiable** | Read-only (mostly) | Read/Write (`setAttribute`) |
| **Method** | `getParameter()` | `getAttribute()` |
| **Scope** | Request, Init, Context | Request, Session, Context |

### Attribute Scopes
1.  **Request Attribute** (`request.setAttribute("key", obj)`):
    *   Available for the duration of one request (including `forward`). Use to pass data from Servlet -> JSP.
2.  **Session Attribute** (`session.setAttribute("key", obj)`):
    *   Available across multiple requests of the same user.
3.  **Context Attribute** (`context.setAttribute("key", obj)`):
    *   Available to all users/servlets. Global data.

---

## 2. Load-on-Startup
By default, Servlets are loaded lazily (on first request).
To load a servlet when the server starts (e.g., for initialization tasks):
*   **Web.xml**:
    ```xml
    <servlet>
        <servlet-name>MyServlet</servlet-name>
        <servlet-class>com.demo.MyServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    ```
*   **Value**: Positive integer. Lower number = Higher priority.

---

## 3. Error Handling
Instead of showing generic stack traces to users, configure a common error page.
*   **Web.xml**:
    ```xml
    <error-page>
        <exception-type>java.lang.Exception</exception-type>
        <location>/error.jsp</location>
    </error-page>
    <error-page>
        <error-code>404</error-code>
        <location>/404.jsp</location>
    </error-page>
    ```
*   **JSP setup**:
    *   `error.jsp`: `<%@ page isErrorPage="true" %>` (Has access to `exception` object).
    *   Normal JSP: `<%@ page errorPage="error.jsp" %>` (Optional override).

---

## 4. JSP (JavaServer Pages)
JSP is a server-side technology that allows mixing HTML with Java.
*   **Execution**: JSP -> Translated to Java Servlet -> Compiled to `.class` -> Loaded & Executed.
*   **Lifecycle**:
    *   `jspInit()` (Overrideable)
    *   `_jspService()` (The main logic, **Final** - Cannot override)
    *   `jspDestroy()` (Overrideable)

### JSP Implicit Objects (9 Objects)
Available automatically in scriptlets/expressions:
1.  **out** (`JspWriter`): Printing to client.
2.  **request** (`HttpServletRequest`)
3.  **response** (`HttpServletResponse`)
4.  **session** (`HttpSession`)
5.  **application** (`ServletContext`)
6.  **config** (`ServletConfig`)
7.  **pageContext** (`PageContext`): Access to all scopes.
8.  **page** (`Object`): The servlet instance itself (`this`).
9.  **exception** (`Throwable`): Available only in error pages.

### JSP Elements
1.  **Scriptlet** `<% java code %>`: Executes Java logic.
2.  **Expression** `<%= variable %>`: Prints value (no semicolon).
3.  **Declaration** `<%! method/field %>`: Defines instance variables/methods.
4.  **Directive** `<%@ directive ... %>`:
    *   `page`: `<%@ page import="java.util.*" contentType="text/html" %>`
    *   `include`: `<%@ include file="header.jsp" %>` (Static include at translation time).
    *   `taglib`: `<%@ taglib uri="..." prefix="c" %>` (JSTL).

---

## 5. Deployment (WAR)
*   **WAR (Web Archive)**: A standard format to package web apps.
*   **Structure**:
    *   Root (HTML, JSP, CSS)
    *   `WEB-INF/web.xml`
    *   `WEB-INF/classes`
    *   `WEB-INF/lib`
*   **Deployment**: Export as `.war` and place in Tomcat's `webapps` folder.

## 6. Thread Safety
*   **Servlet/JSP**: Single Instance, Multiple Threads.
*   **Rule**: Do not use instance variables for request-specific data (not thread-safe). Use local variables in `doGet/doPost`.
