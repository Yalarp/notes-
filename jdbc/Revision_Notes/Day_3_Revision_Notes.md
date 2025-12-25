# Day 3: Parameters, Dispatching, and Session Management

## 1. Types of Parameters
Data can be passed to Servlets in three scopes:

| Type | Scope | Defined In | Access Method | Object Used |
| :--- | :--- | :--- | :--- | :--- |
| **Request Parameter** | Single Request | HTML Form / URL | `getParameter(name)` | `HttpServletRequest` |
| **Init Parameter** | Single Servlet | `web.xml` / `@WebServlet` | `getInitParameter(name)` | `ServletConfig` |
| **Context Parameter** | Entire Web App | `web.xml` | `getInitParameter(name)` | `ServletContext` |

### Configuration Example (web.xml)
```xml
<web-app>
    <!-- Context Param (Global) -->
    <context-param>
        <param-name>adminEmail</param-name>
        <param-value>admin@example.com</param-value>
    </context-param>

    <servlet>
        <servlet-name>MyServlet</servlet-name>
        <servlet-class>com.demo.MyServlet</servlet-class>
        <!-- Init Param (Servlet Specific) -->
        <init-param>
            <param-name>maxUploadSize</param-name>
            <param-value>10MB</param-value>
        </init-param>
    </servlet>
</web-app>
```

---

## 2. ServletConfig vs ServletContext
*   **ServletConfig**:
    *   Created **one per Servlet**.
    *   Contains init parameters for that specific servlet.
    *   To get: `getServletConfig()` or `this.getServletConfig()`.
*   **ServletContext**:
    *   Created **one per Web Application**.
    *   Shared by all servlets. Used for global configuration (DB URL, etc.).
    *   To get: `getServletContext()`.
    *   **Note**: `init(ServletConfig config)` is the lifecycle method where config is set. `init()` is a convenience method.

---

## 3. Communication between Web Components
What if one servlet can't finish the request? It hands over control to another.

### A. Send Redirect (`response.sendRedirect("url")`)
*   **Mechanism**: Server tells browser to make a **new request** to the new URL.
*   **URL**: Changes in browser address bar.
*   **Scope**: Can redirect to external sites (e.g., google.com).
*   **Speed**: Slower (2 round trips).
*   **Data**: Request attributes are **lost** (because it's a new request).

### B. Request Dispatcher (Forward & Include)
Use `RequestDispatcher rd = request.getRequestDispatcher("url");`

1.  **Forward (`rd.forward(req, res)`)**:
    *   **Mechanism**: Server transfers control internally. Browser is unaware.
    *   **URL**: Does **not** change.
    *   **Scope**: Internal to web app only.
    *   **Speed**: Faster (1 round trip).
    *   **Data**: Request attributes are **preserved**.
2.  **Include (`rd.include(req, res)`)**:
    *   Includes the response of the second component into the first.
    *   Control returns to the first servlet after execution.

---

## 4. Session Management (Making HTTP Stateful)
HTTP is **stateless**. To maintain user state (shopping cart, login status), we use:

### A. Hidden Form Fields
*   `<input type="hidden" name="user" value="scott">`
*   Data travels in form body. Only works with form submissions.

### B. Cookies (Client-side)
*   Small text data stored in browser.
*   **Send**: `response.addCookie(new Cookie("key", "value"))`
*   **Get**: `request.getCookies()` (returns array)
*   **Disable**: User can disable cookies in browser.

### C. HttpSession (Server-side)
*   Best for sensitive/complex data (Java Objects).
*   **Create/Get**: `HttpSession session = request.getSession();`
    *   `getSession(true)` (default): Create new if not exists.
    *   `getSession(false)`: Return null if not exists.
*   **Store Data**: `session.setAttribute("user", userObj);`
*   **Retrieve Data**: `session.getAttribute("user");`
*   **Destroy**: `session.invalidate();` (Logout).

### D. URL Rewriting
*   Used when **Cookies are disabled**.
*   Appends session ID to URL: `url;jsessionid=123...`
*   Usage: `response.encodeURL("nextPage")`.
*   Drawback: URLs look messy.

---

## 5. Design Patterns
*   **Abstract Factory**: "Factory of Factories". Provides an interface for creating families of related or dependent objects without specifying their concrete classes.
