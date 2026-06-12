JSP is a **server-side technology** used to create dynamic web pages using: HTML+Java Code.
JSP is internally converted into a **Servlet** by the server (e.g., Apache Tomcat).
## **JSP Lifecycle**
1. Translation (JSP → Servlet)
2. Compilation
3. Class loading
4. Initialization (`jspInit()`)
5. Request handling (`_jspService()`)
6. Destruction (`jspDestroy()`)
# **2. Anatomy of a JSP Page**

### **Basic Structure**

```jsp
<%@ page language="java" contentType="text/html" %>
<html>
<head>    
	<title>JSP Page</title>
</head>
<body>
<%-- JSP Comment --%>
<%String name = "Sankalp";%>
<h1>Hello <%= name %></h1>
</body>
</html>
```

## **Components**

### 1. **Directives**
Directives provide **global instructions** to the JSP container — they affect the whole page during translation into a servlet.

```
<%@ page import="java.util.*" %>
```
### 2. **Scriptlets**
Allow embedding **Java code** inside JSP.
```
<%int x = 10;%>
```
### 3. **Expressions**
Used to **directly output values** to the browser.
```jsp
<%= x %>
```

![[Pasted image 20260512171456.png]]![[Pasted image 20260512173212.png]]

---

### 4. **Comments**

```
<%-- hidden comment --%>
```


## JSP Application Design using MVC
![JavaBeans Communicating with Events (Intro)](https://images.openai.com/static-rsc-4/IAST1-KFQ--U_U4Ni6VzfrzIT-UGIlr5ZbYOv84aMF0x7FnMOPNUt5dNoUoet67F5CLFFz-V4YK4ryr0hiNIKw-wsHQnMVjAeIDr_qV0WWdQcpdOExjGov8_Ju5bn5gZMSvRAKyV2o5_rNUoxFuV3HK-inZGnSHzlZYsSMyw_mNd6hPn9frTmn4jgC2oltGO?purpose=fullsize)
### **MVC Components**
![MVC - Glossary | MDN](https://developer.mozilla.org/en-US/docs/Glossary/MVC/model-view-controller-light-blue.png)

|Component|Role|
|---|---|
|Model|Business logic (JavaBeans)|
|View|JSP|
|Controller|Servlet|
## **Flow**

1. Client sends request
2. Servlet (Controller) processes
3. Calls Model (JavaBeans)
4. Forwards to JSP (View)
5. JSP displays result
## **Example**

### Servlet (Controller)

```java
String name = request.getParameter("name");
request.setAttribute("user",name);
request.getRequestDispatcher("welcome.jsp").forward(request, response);
```

### JSP (View)

```
<h1>Welcome <%= request.getAttribute("user") %></h1>
```


# **4. Generating Dynamic Content in JSP**

### Example

```
<%
int a = 5;    
int b = 10;
%>
<p>Sum: <%= a + b %></p>
```
### Loop Example

```
<%for(int i = 1; i <= 5; i++) {%>    
		<p>Number: <%= i %></p><%
	}
	%>
```

# **5. JSP Scripting Elements**

## **Types**

### 1. Scriptlet

```
<% int x = 5; %>
```

### 2. Expression

```
<%= x %>
```

### 3. Declaration

```
<%! int x = 10; %>
```

## **Difference**

| Element     | Scope                 |
| ----------- | --------------------- |
| Scriptlet   | Inside service method |
| Declaration | Class level           |
| Expression  | Output                |
# **6. Implicit JSP Objects**

JSP provides **predefined objects** (no need to create manually).
## **Important Objects**

| Object      | Type                | Use           |
| ----------- | ------------------- | ------------- |
| request     | HttpServletRequest  | Client data   |
| response    | HttpServletResponse | Send response |
| session     | HttpSession         | User session  |
| application | ServletContext      | Global data   |
| out         | JspWriter           | Output        |
| config      | ServletConfig       | Config        |
| pageContext | PageContext         | Page scope    |
## **Example**

```
<%String user = (String) session.getAttribute("user");%><h1>Welcome <%= user %></h1>
```

# **7. Conditional Processing in JSP**

### **If Condition**

```
<%if(session.getAttribute("user") != null) {%>    <h1>Logged In</h1><%} else {%>    <h1>Please Login</h1><%}%>
```

# **8. Sharing Session and Application Data**

---

## **Session Scope (User-specific)**

```
<%session.setAttribute("username", "Sankalp");%>
```

Retrieve:

```
<%= session.getAttribute("username") %>
```

---

## **Application Scope (Global)**

```
<%application.setAttribute("siteName", "MyWebsite");%>
```

Retrieve:

```
<%= application.getAttribute("siteName") %>
```
## **Difference**

|Scope|Lifetime|Visibility|
|---|---|---|
|Session|User session|Single user|
|Application|App lifecycle|All users|

---

# **9. Memory Usage Considerations**

This is **important for exams**.

---

## **Problems**

- Too much data in session → memory overflow
- Large objects → performance issues
- Multiple users → high RAM usage

---

## **Best Practices**

### 1. **Minimize Session Data**

Bad:

```
session.setAttribute("largeObject", hugeData);
```

Good:

- Store only IDs
- Fetch data when needed

---

### 2. **Use Application Scope Carefully**

- Avoid storing user-specific data

---

### 3. **Avoid Business Logic in JSP**

- Move logic to Servlets / JavaBeans

---

### 4. **Use MVC Architecture**

- Keeps JSP lightweight

---

### 5. **Session Timeout**

- Prevent memory leaks

---

### 6. **Invalidate Sessions**

```
session.invalidate();
```

---

# **Key Exam Points (High Value)**

- JSP is converted into Servlet
- MVC = Servlet + JSP + JavaBeans
- Implicit objects simplify coding
- Session vs Application scope difference
- Scriptlets are discouraged in modern development
- Memory management is critical for scalability