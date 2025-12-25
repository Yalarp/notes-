# Day 6: MVC & Custom Tags

## 1. MVC Architecture (Model View Controller)
A design pattern to separate concerns in web applications.

### Components
1.  **Controller (Servlet)**:
    *   Receives the request.
    *   Extracts data (Parameters).
    *   Invokes Business Logic (Model).
    *   Stores result in Scope (usually **Request Attribute**).
    *   Forwards to View (JSP).
2.  **Model (POJO/Bean)**:
    *   Contains business logic and data.
    *   Reusable and independent of Servlet API (mostly).
3.  **View (JSP)**:
    *   Presentation layer.
    *   Retrieves data from attributes (EL/JSTL) and displays it.

### MVC Flow Example
1.  **Form**: User enters "BE" in `form.html`.
2.  **Controller**: `CareerServ` gets parameter `qualification`.
3.  **Model**: `CareerExpert.getAdvice("BE")` returns "DAC Course".
4.  **Controller**: `request.setAttribute("advice", "DAC Course");`
5.  **Controller**: `request.getRequestDispatcher("result.jsp").forward(req, res);`
6.  **View**: `result.jsp` prints `${advice}`.

---

## 2. User Defined Tags (Custom Tags)
Create custom JSP tags to encapsulate logic and keep JSP clean (Scriptlet-free).

### Components
1.  **Tag Handler Class**: Java class that functions as the tag.
    *   Extends `SimpleTagSupport` (JSP 2.0+).
    *   Override `doTag()`.
2.  **TLD (Tag Library Descriptor)**: XML file describing the tag.
    *   Location: `WEB-INF/tlds/mytags.tld`.
3.  **JSP**: Uses the tag.

### Steps to Create
1.  **Create Handler**:
    ```java
    public class MyTag extends SimpleTagSupport {
        public void doTag() throws JspException, IOException {
            JspWriter out = getJspContext().getOut();
            out.print("Hello from Custom Tag!");
        }
    }
    ```
2.  **Create TLD (`WEB-INF/hello.tld`)**:
    ```xml
    <taglib>
        <tlib-version>1.0</tlib-version>
        <short-name>m</short-name>
        <uri>http://mypack/mytags</uri>
        <tag>
            <name>hello</name>
            <tag-class>mypack.MyTag</tag-class>
            <body-content>empty</body-content>
        </tag>
    </taglib>
    ```
3.  **Use in JSP**:
    ```jsp
    <%@ taglib uri="http://mypack/mytags" prefix="m" %>
    <m:hello />
    ```

### Tag Attributes
To pass data to tag: `<m:greet name="Scott" />`.
1.  **Java**: Add field `name` + Setter method in Handler class.
2.  **TLD**: Add `<attribute>` section.
    ```xml
    <attribute>
        <name>name</name>
        <required>true</required>
    </attribute>
    ```
