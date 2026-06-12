### **What are Cookies?**

Cookies are **small pieces of data stored in the client browser**. They are used to maintain state across HTTP requests.
Since HTTP is **stateless**, cookies help in remembering:
- Login status
- User preferences
- Session identifiers
## **Types of Cookies**

### 1. **Session Cookies**

- Stored temporarily in browser memory
- Deleted when browser closes

### 2. **Persistent Cookies**

- Stored on disk
- Have expiry time

### 3. **Secure Cookies**

- Sent only over HTTPS

### 4. **HttpOnly Cookies**

- Not accessible via JavaScript (prevents XSS theft)

## **How Cookies Work**

1. Server sends cookie:

```java
Set-Cookie: user=Sankalp
```

2. Browser stores it
3. Browser sends back:

```java
Cookie: user=Sankalp
```

## **Servlet Cookie Implementation**

### **Creating Cookie**

```java
Cookie cookie = new Cookie("username", "Sankalp");
cookie.setMaxAge(60*60); // 1 hour (written in seconds)
response.addCookie(cookie);
```

### **Reading Cookie**

```java
Cookie[] cookies = request.getCookies();
if (cookies != null) {    
	for (Cookie c : cookies) {        
		if (c.getName().equals("username")) {
			out.println("Welcome " + c.getValue());        
			}    
		}}
```

## **Advantages of Cookies**
- Lightweight
- Easy to implement
- Works across requests

## **Limitations**
- Limited size (~4KB)
- Stored on client → less secure
- Can be disabled by user

# Session Tracking
Session tracking is a technique to **maintain user state across multiple requests**.
## **Why Needed?**

HTTP is stateless:

- Each request is independent
- No memory of previous interaction
## **Session Tracking Techniques**

### 1. **Cookies (Most Common)**
- Session ID stored in cookie
### 2. **URL Rewriting**

```java
response.encodeURL("home.jsp");
```

Example:

```java
home.jsp;jsessionid=1234
```

### 3. **Hidden Form Fields**

```html
<input type="hidden" name="sessionId" value="1234">
```

### 4. **HTTPSession**
### **Creating Session**

```java
HttpSession session = request.getSession();
```

### **Storing Data**

```java
session.setAttribute("user", "Sankalp");
```

### **Retrieving Data**

```java
String user = (String) session.getAttribute("user");
```

### **Invalidating Session**

```java
session.invalidate();
```

## **Session Lifecycle**

- Created → `getSession()`
- Active → user interacting
- Timeout → inactive for defined time
- Destroyed → logout or invalidate()

## **Session Timeout Configuration**

```
<session-config>  
	<session-timeout>30</session-timeout>
</session-config>
```

(30 minutes)

## **Cookies vs Session**

| Feature  | Cookies | Session         |
| -------- | ------- | --------------- |
| Storage  | Client  | Server          |
| Security | Low     | High            |
| Size     | Limited | Large           |
| Speed    | Faster  | Slightly slower |
# Security Issues in Web Applications
## **1. Session Hijacking**

### **What Happens**

- Attacker steals session ID
- Gains access as legitimate user

### **How**

- Sniffing (HTTP traffic)
- XSS stealing cookies

### **Prevention**

- Use HTTPS
- Use HttpOnly cookies
- Regenerate session IDs
## **2. Cross-Site Scripting (XSS)**

### **Example Attack**

```
<script>alert('Hacked')</script>
```

Injected into input fields.

### **Types**

- Stored XSS
- Reflected XSS

### **Prevention**

- Input validation
- Output encoding
- Use frameworks

---

## **3. SQL Injection**

### **Example**

```
SELECT * FROM users WHERE name = 'admin' OR '1'='1';
```

### **Cause**

- Directly using user input in SQL

### **Prevention**

```
PreparedStatement ps = conn.prepareStatement(    "SELECT * FROM users WHERE name=?");ps.setString(1, username);
```

---

## **4. Cross-Site Request Forgery (CSRF)**

### **Concept**

- User unknowingly performs actions on another site

### **Example**

- Clicking malicious link triggers bank transfer

### **Prevention**

- CSRF tokens
- SameSite cookies

---

## **5. Insecure Cookies**

### Problems:

- Accessible via JavaScript
- Sent over HTTP

### Fix:

```
cookie.setHttpOnly(true);cookie.setSecure(true);
```

---

## **6. Weak Session Management**

### Issues:

- Predictable session IDs
- Long session lifetime

### Fix:

- Use random session IDs
- Set timeout
- Invalidate after logout

---

## **7. HTTPS and Encryption**

### Importance:

- Encrypts data between client and server

### Without HTTPS:

- Data can be intercepted

---

## **8. Input Validation**

Always validate:

- Forms
- URL parameters
- Headers

## **Best Practices Summary**

- Always use HTTPS
- Use HttpSession instead of custom tracking
- Set cookies as Secure + HttpOnly
- Validate all user inputs
- Use PreparedStatements
- Implement session timeout
- Avoid business logic in JSP