# Day 2: Servlets & Maven

## 1. Web Application Basics
*   **Static Content**: Same content for all users (HTML). Request -> Response.
*   **Dynamic Content**: Content generated based on request (Servlet/JSP). Request -> Processing -> Response.
*   **CGI (Common Gateway Interface)**: Old way. Creates a **new process** for every request (Heavy).
*   **Servlet**: Java way. Uses **threads** from a thread pool (Lightweight, efficient).

## 2. Servlet Architecture
**Servlet** is a Java class that runs on a server and handles HTTP requests.
### Hierarchy
1.  `jakarta.servlet.Servlet` (Interface) - Top level contract.
2.  `jakarta.servlet.GenericServlet` (Abstract Class) - Protocol independent (HTTP, FTP, etc.).
3.  `jakarta.servlet.http.HttpServlet` (Abstract Class) - HTTP specific. **Always extend this**.

### Lifecycle Methods
1.  **`init(ServletConfig)`**: Called **once** when servlet is loaded.
    *   Used for initialization (DB connection, loading properties).
2.  **`service(req, res)`**: Called **per request** in a dedicated thread.
    *   Dispatches to `doGet`, `doPost`, etc.
3.  **`destroy()`**: Called **once** when server shuts down or app is undeployed.
    *   Used for cleanup (closing DB connection).

## 3. Creating a Servlet
### Workflow
1.  Extend `HttpServlet`.
2.  Override `doGet()` or `doPost()`.
3.  Map URL using `@WebServlet("/url")` or `web.xml`.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    // 1. Init (Optional)
    public void init(ServletConfig config) {
        System.out.println("Servlet Initialized");
    }

    // 2. Handle GET Request
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<h1>Hello from Servlet</h1>");
    }
}
```

## 4. HttpServletRequest & Response
*   **HttpServletRequest**: Represents the incoming request.
    *   `getParameter("name")`: Get form data/query params.
    *   `getParameterValues("checkboxName")`: Get multiple values.
*   **HttpServletResponse**: Represents the outgoing response.
    *   `setContentType("text/html")`: Set MIME type.
    *   `getWriter()`: Get character stream (`PrintWriter`) for text/html.
    *   `getOutputStream()`: Get binary stream (`ServletOutputStream`) for images/files.

## 5. Deployment Structure
*   **Root Folder**: HTML, CSS, JS, JSP files.
*   **WEB-INF**: Private folder (not accessible directly).
    *   `web.xml`: Deployment Descriptor.
    *   `classes`: Compiled `.class` files.
    *   `lib`: External JARs (e.g., `mysql-connector.jar`).

## 6. Database in Servlets
**Best Practice**:
1.  **DB Connection**: Create in `init()` or use a Connection Pool.
2.  **Logic**: Execute queries in `doGet/doPost`.
3.  **Properties File**: Store credentials in `db.properties` and load using `ResourceBundle`.

### Example: Using ResourceBundle for DB Config
```java
// Logic to get connection using properties
ResourceBundle rb = ResourceBundle.getBundle("mypack.db"); 
String url = rb.getString("url");
String user = rb.getString("user");
// ... create connection
```

## 7. Maven
**Maven** is a build automation and dependency management tool.
*   **pom.xml**: Project Object Model. Configuration file.
*   **Dependencies**: JARs needed by the project (defined in `pom.xml`).
    *   Maven downloads them from **Central Repo** to **Local Repo** (`.m2`).
*   **Build Lifecycle**: `clean` -> `compile` -> `test` -> `package` (creates WAR) -> `install`.

## 8. Design Patterns (Brief)
*   **Factory Pattern**: Creating objects without specifying the exact class of object that will be created.
*   **Singleton Pattern**: Ensuring a class has only one instance (e.g., Database Connection).

## 9. Key Differences
### doGet vs doPost
| doGet | doPost |
| :--- | :--- |
| Data sent in **URL header**. | Data sent in **Request Body**. |
| Limited size. | Unlimited size. |
| Visible (Less secure). | Hidden (More secure). |
| Idempotent (Safe to repeat). | Not idempotent. |

### Web Server vs App Server
*   **Web Server** (e.g., Apache HTTP): Serves static content.
*   **App Server** (e.g., JBoss, WebLogic): Serves dynamic content, full EE support (EJB, JMS).
*   **Tomcat**: Technically a **Servlet Container** (Web Container).

## 10. Summary Steps for Servlet App
1.  Create Dynamic Web Project (or Maven Project).
2.  Add dependencies (MySQL driver) to `WEB-INF/lib` or `pom.xml`.
3.  Create HTML form (`action="myservlet" method="post"`).
4.  Create Servlet class (`@WebServlet("/myservlet")`).
5.  Handle logic in `doPost`.
6.  Run on Server.
