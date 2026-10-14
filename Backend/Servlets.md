## Web Technology - Unit 4: Comprehensive Study Guide

### **Part 1: Web Servers and Servlets**

---
### **1. Tomcat Web Server**

#### **1.1 Introduction to Tomcat**

**Apache Tomcat** is an open-source web server and servlet container developed by the Apache Software Foundation. It implements several Java EE specifications including Java Servlet, JavaServer Pages (JSP), Java EL, and WebSocket.

*(Java Enterprise Edition (Java EE), now officially known as Jakarta EE, is a set of specifications built on top of the Java Standard Edition (Java SE) designed for developing and running large-scale, multi-tiered, and secure network applications)*
#### **Key Features:**
- **Lightweight**: Compared to full Java EE application servers
- **Open Source**: Free to use and modify
- **Cross-platform**: Runs on Windows, Linux, macOS
- **Standards Compliant**: Implements Java Servlet and JSP specifications
- **Embeddable**: Can be embedded in Java applications
#### **Architecture Components:**
```
┌─────────────────────────────────────┐
│         Tomcat Server               │
├─────────────────────────────────────┤
│  ┌───────────────────────────────┐  │
│  │     Catalina (Servlet Engine) │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │     Coyote (HTTP Connector)   │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │     Jasper (JSP Engine)       │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

#### **Directory Structure:**

```
TOMCAT_HOME/
├── bin/          # Startup/shutdown scripts
├── conf/         # Configuration files
│   ├── server.xml
│   ├── web.xml
│   └── context.xml
├── lib/          # JAR files and libraries
├── logs/         # Log files
├── temp/         # Temporary files
├── webapps/      # Web applications deployed here
└── work/         # Compiled JSP files
```

#### **Important Configuration Files:**
**server.xml** - Main configuration file
```xml
<Server port="8005" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
	<Engine name="Catalina" defaultHost="localhost">
    <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
      </Host>
    </Engine>
  </Service>
</Server>
```
**web.xml** - Deployment descriptor for web applications
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="3.1">
    <display-name>My Web Application</display-name>
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.example.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```

### **2. Introduction to Servlets**
#### **2.1 What is a Servlet?**

A **Servlet** is a Java class that extends the capabilities of servers to host applications accessed via a request-response programming model. Servlets are primarily used to process or store data submitted by an HTML form, provide dynamic content, and manage state information.

#### **Key Characteristics:**

- Server-side technology
- Platform-independent
- Uses Java programming language
- Extends javax.servlet.http.HttpServlet class
- Handles HTTP requests and responses
- Thread-safe and scalable

#### **Advantages of Servlets:**

1. **Performance**: Better performance than CGI as servlets are loaded once and handle multiple requests
2. **Portability**: Platform independent (Write Once, Run Anywhere)
3. **Robustness**: JVM managed memory and exception handling
4. **Extensibility**: Can be easily extended using Java inheritance
5. **Security**: Java security features apply

---

### **3. Lifecycle of a Servlet**

The servlet lifecycle consists of **three main phases**:

```
┌──────────────────────────────────────────┐
│    1. INITIALIZATION (init method)       │
│         Called once when loaded          │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│    2. SERVICE (service method)           │
│      Called for each client request      │
│      ┌─────────────────────────┐         │
│      │  doGet()  or  doPost()  │         │
│      └─────────────────────────┘         │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│    3. DESTRUCTION (destroy method)       │
│         Called once before unloading     │
└──────────────────────────────────────────┘
```

#### **Detailed Lifecycle Methods:**

##### **3.1 init() Method**

```java
public void init(ServletConfig config) throws ServletException {
    super.init(config);
    // Initialization code here
    // Called only once when servlet is first loaded
    // Used to initialize resources like database connections
}
```

**Purpose:**
- Initialize servlet configuration
- Load initialization parameters
- Establish database connections
- Initialize resources
##### **3.2 service() Method**
java

```java
protected void service(HttpServletRequest request, 
                      HttpServletResponse response) 
                      throws ServletException, IOException {
    // Called for each client request
    // Determines request type and calls appropriate method
    String method = request.getMethod();
    if (method.equals("GET")) {
        doGet(request, response);
    } else if (method.equals("POST")) {
        doPost(request, response);
    }
}
```

**Purpose:**

- Handle client requests
- Route to appropriate handler methods
- Called multiple times (once per request)

##### **3.3 destroy() Method**
```java
public void destroy() {
    // Cleanup code here
    // Called once before servlet is unloaded
    // Close database connections, save state, etc.
}
```

**Purpose:**
- Release resources
- Close connections
- Save persistent state
- Perform cleanup operations

### **4. JSDK (Java Servlet Development Kit)**

JSDK provides the necessary libraries and tools to develop Java servlets. It includes:
- Servlet API classes and interfaces
- Development tools
- Documentation
- Sample servlets
#### **Core Components:**

1. **javax.servlet package** - Core servlet interfaces
2. **javax.servlet.http package** - HTTP-specific servlet support
3. **Servlet container** - Runtime environment

---
### **5. The Servlet API**
#### **5.1 Core Interfaces and Classes**

```
Servlet API Hierarchy:
═══════════════════════
Servlet (interface)
    ↑
    │ implements
    │
GenericServlet (abstract class)
    ↑
    │ extends
    │
HttpServlet (abstract class)
    ↑
    │ extends
    │
YourServlet (concrete class)
```

#### **5.2 Key Interfaces:**

##### **Servlet Interface**
```java
public interface Servlet {
    void init(ServletConfig config) throws ServletException;
    void service(ServletRequest req, ServletResponse res) 
         throws ServletException, IOException;
    void destroy();
    ServletConfig getServletConfig();
    String getServletInfo();
}
```

##### **ServletConfig Interface**
```java
public interface ServletConfig {
    String getServletName();
    ServletContext getServletContext();
    String getInitParameter(String name);
    Enumeration<String> getInitParameterNames();
}
```

##### **ServletContext Interface**
```java
public interface ServletContext {
    Object getAttribute(String name);
    void setAttribute(String name, Object object);
    void removeAttribute(String name);
    String getInitParameter(String name);
    void log(String message);
}
```

---
### **6. The javax.servlet Package**

#### **6.1 Important Classes and Interfaces:**
**GenericServlet** - Protocol-independent servlet base class
```java
import javax.servlet.*;
import java.io.*;

public class MyGenericServlet extends GenericServlet {
    @Override
    public void service(ServletRequest request, 
                       ServletResponse response) 
                       throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>Generic Servlet Example</h1>");
        out.println("</body></html>");
    }
}
```

#### **6.2 ServletRequest Interface**

**Methods:**
- `String getParameter(String name)` - Get single parameter
- `String[] getParameterValues(String name)` - Get multiple values
- `Enumeration getParameterNames()` - Get all parameter names
- `Map getParameterMap()` - Get parameter map
- `String getRemoteAddr()` - Get client IP
- `int getContentLength()` - Get content length

#### **6.3 ServletResponse Interface**

**Methods:**

- `void setContentType(String type)` - Set response content type
- `PrintWriter getWriter()` - Get character output stream
- `ServletOutputStream getOutputStream()` - Get binary output stream
- `void setCharacterEncoding(String charset)` - Set encoding

---

### **7. Reading Servlet Parameters**

#### **7.1 Request Parameters**

Request parameters are sent from client to server via:

- **URL query string** (GET method)
- **Request body** (POST method)
- **HTML forms**

#### **Example 1: Simple Parameter Reading**

**HTML Form:**

html

```html
<!DOCTYPE html>
<html>
<head>
    <title>User Registration</title>
</head>
<body>
    <h2>Registration Form</h2>
    <form action="RegisterServlet" method="post">
        <label>Name:</label>
        <input type="text" name="username" required><br><br>
        
        <label>Email:</label>
        <input type="email" name="email" required><br><br>
        
        <label>Age:</label>
        <input type="number" name="age" min="18" max="100"><br><br>
        
        <label>Gender:</label>
        <input type="radio" name="gender" value="male"> Male
        <input type="radio" name="gender" value="female"> Female<br><br>
        
        <label>Hobbies:</label>
        <input type="checkbox" name="hobbies" value="reading"> Reading
        <input type="checkbox" name="hobbies" value="sports"> Sports
        <input type="checkbox" name="hobbies" value="music"> Music<br><br>
        
        <input type="submit" value="Register">
    </form>
</body>
</html>
```

**Servlet Implementation:**

java

```java
import java.io.*;
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import jakarta.servlet.annotation.WebServlet;

@WebServlet("/register")
public class RegisterServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request,
                          HttpServletResponse response)
                          throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        String username = request.getParameter("username");
        String email = request.getParameter("email");
        String age = request.getParameter("age");
        String gender = request.getParameter("gender");
        String[] hobbies = request.getParameterValues("hobbies");

        out.println("<!DOCTYPE html>");
        out.println("<html><head><title>Registration Success</title></head>");
        out.println("<body>");
        out.println("<h2>Registration Successful!</h2>");
        out.println("<p><strong>Name:</strong> " + username + "</p>");
        out.println("<p><strong>Email:</strong> " + email + "</p>");
        out.println("<p><strong>Age:</strong> " + age + "</p>");
        out.println("<p><strong>Gender:</strong> " + gender + "</p>");

        out.println("<p><strong>Hobbies:</strong> ");
        if (hobbies != null && hobbies.length > 0) {
            for (int i = 0; i < hobbies.length; i++) {
                out.print(hobbies[i]);
                if (i < hobbies.length - 1) {
                    out.print(", ");
                }
            }
        } 
        else {out.print("None selected");}
        out.println("</p>");

        out.println("</body></html>");
        out.close();
    }
}
```

#### **Example 2: Advanced Parameter Handling**
```java
import java.io.*;
import java.util.*;
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import jakarta.servlet.annotation.WebServlet;
@WebServlet("/params")
public class AdvancedParameterServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request,
                         HttpServletResponse response)
                         throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();

        out.println("<html><body>");
        out.println("<h2>All Request Parameters</h2>");
        out.println("<table border='1'>");
        out.println("<tr><th>Parameter Name</th><th>Value(s)</th></tr>");
        Enumeration<String> paramNames = request.getParameterNames();
        while(paramNames.hasMoreElements()){
            String paramName = paramNames.nextElement();
            String[] paramValues = request.getParameterValues(paramName);
            out.println("<tr>");
            out.println("<td>" + paramName + "</td>");
            out.println("<td>");

            if (paramValues != null) {
                for (String value : paramValues) {
                    out.println(value + "<br>");
                }
            }
            out.println("</td>");
            out.println("</tr>");
        }
        out.println("</table>");
        out.println("<h3>Parameter Map:</h3>");
        Map<String, String[]> paramMap = request.getParameterMap();
        for (Map.Entry<String, String[]> entry : paramMap.entrySet()) {
            out.println("<p><strong>" + entry.getKey() + ":</strong> "
                    + Arrays.toString(entry.getValue()) + "</p>");
        }
        out.println("</body></html>");
        out.close();
    }
}
```

---

### **8. Reading Initialization Parameters**

#### **8.1 What are Initialization Parameters?**

Initialization parameters are configuration values defined in **web.xml** that are available to servlets during their lifetime. They are loaded once when the servlet is initialized.
#### **Types:**
1. **Servlet-specific parameters** - Available to one servlet
2. **Context parameters** - Available to all servlets in the application
#### **8.2 Servlet Initialization Parameters**

**web.xml Configuration:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="https://jakarta.ee/xml/ns/jakartaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee 
                             https://jakarta.ee/xml/ns/jakartaee/web-app_6_0.xsd"
         version="6.0">
    <servlet>
        <servlet-name>RegisterServlet</servlet-name>
        <servlet-class>RegisterServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>RegisterServlet</servlet-name>
        <url-pattern>/register</url-pattern>
    </servlet-mapping>
</web-app>
```

**Servlet Implementation:**

java

```java
import javax.servlet.*;
import javax.servlet.http.*;
import java.io.*;
import java.util.*;

public class ConfigServlet extends HttpServlet {
    
    private String adminEmail;
    private int maxUploadSize;
    private String databaseDriver;
    
    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        
        // Reading servlet-specific init parameters
        adminEmail = config.getInitParameter("adminEmail");
        maxUploadSize = Integer.parseInt(
        config.getInitParameter("maxUploadSize"));
        databaseDriver = config.getInitParameter("databaseDriver");
        
        // Log initialization
        log("ConfigServlet initialized with:");
        log("Admin Email: " + adminEmail);
        log("Max Upload Size: " + maxUploadSize + " bytes");
        log("Database Driver: " + databaseDriver);
    }
    
    @Override
    protected void doGet(HttpServletRequest request, 
                        HttpServletResponse response) 
                        throws ServletException, IOException {
        
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        
        out.println("<!DOCTYPE html>");
        out.println("<html><head><title>Configuration Info</title></head>");
        out.println("<body>");
        
        // Display servlet-specific parameters
        out.println("<h2>Servlet Initialization Parameters</h2>");
        out.println("<p><strong>Admin Email:</strong> " + adminEmail + "</p>");
        out.println("<p><strong>Max Upload Size:</strong> " 
                   + maxUploadSize + " bytes</p>");
        out.println("<p><strong>Database Driver:</strong> " 
                   + databaseDriver + "</p>");
        
        // Reading context parameters
        ServletContext context = getServletContext();
        String appName = context.getInitParameter("applicationName");
        String supportEmail = context.getInitParameter("supportEmail");
        
        out.println("<h2>Context Parameters</h2>");
        out.println("<p><strong>Application Name:</strong> " 
                   + appName + "</p>");
        out.println("<p><strong>Support Email:</strong> " 
                   + supportEmail + "</p>");
        
        // List all init parameters
        out.println("<h3>All Servlet Init Parameters:</h3>");
        out.println("<ul>");
        Enumeration<String> initParams = getInitParameterNames();
        while (initParams.hasMoreElements()) {
            String paramName = initParams.nextElement();
            String paramValue = getInitParameter(paramName);
            out.println("<li><strong>" + paramName + ":</strong> " 
                       + paramValue + "</li>");
        }
        out.println("</ul>");
        
        out.println("</body></html>");
    }
}
```

#### **8.3 Using Annotations (Servlet 3.0+)**

java

```java
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.*;

@WebServlet("/annotationConfig")
public class AnnotationConfigServlet extends HttpServlet {
    
    private String adminEmail;
    private int maxConnections;
    private int timeout;
    
    @Override
    public void init() throws ServletException {
        adminEmail = getInitParameter("adminEmail");
        maxConnections = Integer.parseInt(getInitParameter("maxConnections"));
        timeout = Integer.parseInt(getInitParameter("timeout"));
        
        log("Servlet initialized with annotations");
    }
    
    @Override
    protected void doGet(HttpServletRequest request, 
                        HttpServletResponse response) 
                        throws ServletException, IOException {
        
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        
        out.println("<html><body>");
        out.println("<h2>Annotation-based Configuration</h2>");
        out.println("<p>Admin Email: " + adminEmail + "</p>");
        out.println("<p>Max Connections: " + maxConnections + "</p>");
        out.println("<p>Timeout: " + timeout + " seconds</p>");
        out.println("</body></html>");
    }
}
```

---

### **9. The javax.servlet.http Package**

#### **9.1 HttpServlet Class**

The most commonly used class for creating web applications.

**Key Methods:**
```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
protected void doPost(HttpServletRequest req, HttpServletResponse resp)
protected void doPut(HttpServletRequest req, HttpServletResponse resp)
protected void doDelete(HttpServletRequest req, HttpServletResponse resp)
protected void doHead(HttpServletRequest req, HttpServletResponse resp)
protected void doOptions(HttpServletRequest req, HttpServletResponse resp)
```

#### **9.2 HttpServletRequest Interface**

**Important Methods:**
```java
// Parameter methods
String getParameter(String name)
String[] getParameterValues(String name)
Map<String, String[]> getParameterMap()

// Header methods
String getHeader(String name)
Enumeration<String> getHeaderNames()
int getIntHeader(String name)
long getDateHeader(String name)

// Session methods
HttpSession getSession()
HttpSession getSession(boolean create)

// Cookie methods
Cookie[] getCookies()

// Request information
String getMethod()              // GET, POST, etc.
String getRequestURI()
String getQueryString()
String getContextPath()
String getServletPath()
String getRemoteAddr()
String getRemoteHost()
int getServerPort()
```

#### **9.3 HttpServletResponse Interface**

**Important Methods:**
```java
// Response status
void setStatus(int sc)
void sendError(int sc)
void sendError(int sc, String msg)
void sendRedirect(String location)

// Content type
void setContentType(String type)
void setCharacterEncoding(String charset)

// Headers
void setHeader(String name, String value)
void addHeader(String name, String value)
void setIntHeader(String name, int value)
void setDateHeader(String name, long date)

// Cookies
void addCookie(Cookie cookie)

// Output
PrintWriter getWriter()
ServletOutputStream getOutputStream()
```

---

### **10. Handling HTTP Requests and Responses**

#### **10.1 GET Request Handler**
```java
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.*;

@WebServlet("/search")
public class SearchServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, 
                        HttpServletResponse response) 
                        throws ServletException, IOException {
        // Set response content type
        response.setContentType("text/html;charset=UTF-8");
        // Get search query parameter
        String query = request.getParameter("q");
        String category = request.getParameter("category");
        PrintWriter out = response.getWriter();
        try {
            out.println("<!DOCTYPE html>");
            out.println("<html>");
            out.println("<head>");
            out.println("<title>Search Results</title>");
            out.println("<style>");
            out.println("body { font-family: Arial, sans-serif; margin: 20px; }");
            out.println(".result { border: 1px solid #ddd; padding: 10px; margin: 10px 0; }");
            out.println("</style>");
            out.println("</head>");
            out.println("<body>");
            
            out.println("<h1>Search Results</h1>");
            
            if (query != null && !query.trim().isEmpty()) {
                out.println("<p>Search query: <strong>" + query + "</strong></p>");
                
                if (category != null) {
                    out.println("<p>Category: <strong>" + category + "</strong></p>");
                }
                
                // Simulate search results
                out.println("<div class='result'>");
                out.println("<h3>Result 1: " + query + " tutorial</h3>");
                out.println("<p>This is a sample search result...</p>");
                out.println("</div>");
                
                out.println("<div class='result'>");
                out.println("<h3>Result 2: Learn " + query + "</h3>");
                out.println("<p>Another sample result...</p>");
                out.println("</div>");
            } else {
                out.println("<p>Please enter a search query.</p>");
            }
            
            // Display request information
            out.println("<hr>");
            out.println("<h3>Request Information:</h3>");
            out.println("<p><strong>Method:</strong> " + request.getMethod() + "</p>");
            out.println("<p><strong>Request URI:</strong> " + request.getRequestURI() + "</p>");
            out.println("<p><strong>Query String:</strong> " + request.getQueryString() + "</p>");
            out.println("<p><strong>Remote Address:</strong> " + request.getRemoteAddr() + "</p>");
            
            out.println("</body>");
            out.println("</html>");
        } finally {
            out.close();
        }
    }
}
```

**HTML Form for Testing:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Search Form</title>
</head>
<body>
    <h2>Search</h2>
    <form action="search" method="get">
        <input type="text" name="q" placeholder="Enter search term" required>
        <select name="category">
            <option value="all">All Categories</option>
            <option value="books">Books</option>
            <option value="articles">Articles</option>
            <option value="videos">Videos</option>
        </select>
        <button type="submit">Search</button>
    </form>
</body>
</html>
```

#### **10.2 POST Request Handler**
```java
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.*;
import java.util.*;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    // Simulated user database
    private static final Map<String, String> users = new HashMap<>();
    
    static {
        users.put("admin", "admin123");
        users.put("user1", "password1");
        users.put("user2", "password2");
    }
    
    @Override
    protected void doPost(HttpServletRequest request, 
                         HttpServletResponse response) 
                         throws ServletException, IOException {
        
        response.setContentType("text/html");
        
        // Get form parameters
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String remember = request.getParameter("remember");
        PrintWriter out = response.getWriter();
        try {
            out.println("<!DOCTYPE html>");
            out.println("<html>");
            out.println("<head><title>Login Result</title>");
            out.println("<style>");
            out.println(".success { color: green; background: #e8f5e9; padding: 15px; border-radius: 5px; }");
            out.println(".error { color: red; background: #ffebee; padding: 15px; border-radius: 5px; }");
            out.println("</style>");
            out.println("</head>");
            out.println("<body>");
            
            // Validate credentials
            if (username != null && password != null) {
                String storedPassword = users.get(username);
                
                if (storedPassword != null && storedPassword.equals(password)) {
                    // Successful login
                    out.println("<div class='success'>");
                    out.println("<h2>Login Successful!</h2>");
                    out.println("<p>Welcome, " + username + "!</p>");
                    
                    if ("on".equals(remember)) {
                        out.println("<p>Remember me option selected.</p>");
                    }
                    
                    out.println("</div>");
                    
                    // Display user info
                    out.println("<h3>Session Information:</h3>");
                    out.println("<p>Username: " + username + "</p>");
                    out.println("<p>Login time: " + new Date() + "</p>");
                    out.println("<p>Client IP: " + request.getRemoteAddr() + "</p>");
                    
                } else {
                    // Failed login
                    out.println("<div class='error'>");
                    out.println("<h2>Login Failed!</h2>");
                    out.println("<p>Invalid username or password.</p>");
                    out.println("<a href='login.html'>Try Again</a>");
                    out.println("</div>");
                    
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                }
            } else {
                out.println("<div class='error'>");
                out.println("<h2>Error</h2>");
                out.println("<p>Username and password are required.</p>");
                out.println("</div>");
                
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            }
            
            out.println("</body>");
            out.println("</html>");
        } finally {
            out.close();
        }
    }
    
    @Override
    protected void doGet(HttpServletRequest request, 
                        HttpServletResponse response) 
                        throws ServletException, IOException {
        // Redirect GET requests to login page
        response.sendRedirect("login.html");
    }
}
```

**login.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h2>Login Form</h2>
    <form action="login" method="post">
        <div class="form-group">
            <label for="username">Username:</label>
            <input type="text" id="username" name="username" required>
        </div>
        
        <div class="form-group">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" required>
        </div>
        
        <div class="checkbox-group">
            <input type="checkbox" id="remember" name="remember">
            <label for="remember" style="display: inline;">Remember Me</label>
        </div>
        
        <button type="submit">Login</button>
    </form>
    
    <p style="margin-top: 20px; font-size: 14px; color: #666;">
        Test credentials: admin/admin123 or user1/password1
    </p>
</body>
</html>
```