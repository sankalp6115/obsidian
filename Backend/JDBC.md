#### **1.1 What is JDBC?**

**JDBC (Java Database Connectivity)** is a Java API that enables Java applications to interact with databases. It provides methods to query and update data in a database and is oriented towards relational databases.

#### **Key Features:**
- **Database Independence**: Write once, run with any database
- **Standard API**: Consistent interface for different databases
- **SQL Support**: Execute SQL queries and updates
- **Transaction Management**: Control commit and rollback
- **Prepared Statements**: Prevent SQL injection attacks

#### **JDBC Architecture:**

```
┌────────────────────────────────────────────┐
│         Java Application                   │
│                                            │
│  ┌────────────────────────────────────┐    │
│  │       JDBC API                     │    │
│  │  (java.sql.* / javax.sql.*)        │    │
│  └─────────────┬──────────────────────┘    │
└────────────────┼───────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│         JDBC Driver Manager                 │
└─────────────────┬───────────────────────────┘
                  │
        ┌─────────┴─────────┬─────────────┐
        │                   │             │
        ▼                   ▼             ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ MySQL Driver │   │Oracle Driver │   │ SQL Server   │
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                   │
       ▼                  ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   MySQL DB   │   │  Oracle DB   │   │ SQL Server   │
└──────────────┘   └──────────────┘   └──────────────┘
```

#### **1.2 JDBC Driver Types**

1. **Type 1 - JDBC-ODBC Bridge Driver**
    - Uses ODBC driver to connect
    - Platform dependent 
    - Slow performance
2. **Type 2 - Native API Driver (Partly Java Driver)**
    - Uses database-specific native libraries
    - Better performance than Type 1
    - Platform dependent
3. **Type 3 - Network Protocol Driver (Middleware Driver)**
    - Uses middleware server
    - Platform independent
    - Database independent
4. **Type 4 - Thin Driver (Pure Java Driver)**
    - **Most commonly used**
    - Pure Java implementation
    - Platform independent
    - Best performance

### **2. Core JDBC Components**

#### **2.1 Key Interfaces and Classes**

```java
// Core JDBC Interfaces
java.sql.Driver             // Database driver
java.sql.Connection         // Database connection
java.sql.Statement          // Execute SQL statements
java.sql.PreparedStatement  // Precompiled SQL statements
java.sql.CallableStatement  // Stored procedures
java.sql.ResultSet          // Query results
java.sql.ResultSetMetaData  // Result set metadata
java.sql.DatabaseMetaData   // Database metadata
```

### **3. JDBC Programming Steps**
#### **Complete JDBC Workflow:**

```
1. Load Driver Class
        ↓
2. Establish Connection
        ↓
3. Create Statement
        ↓
4. Execute Query/Update
        ↓
5. Process ResultSet
        ↓
6. Close Resources
```

#### **3.1 Step-by-Step Implementation**

##### **Step 1: Load the JDBC Driver**

```java
// Method 1: Using Class.forName() (older approach)
Class.forName("com.mysql.cj.jdbc.Driver");

// Method 2: Automatic (JDBC 4.0+) - Preferred
// Driver is loaded automatically from classpath
```

**Common JDBC Drivers:**

```java
// MySQL
"com.mysql.cj.jdbc.Driver"

// Oracle
"oracle.jdbc.driver.OracleDriver"

// SQL Server
"com.microsoft.sqlserver.jdbc.SQLServerDriver"

// PostgreSQL
"org.postgresql.Driver"

// H2 Database
"org.h2.Driver"
```

##### **Step 2: Establish Connection**

```java
import java.sql.*;

public class DatabaseConnection {
    
    // Database credentials
    private static final String DB_URL = "jdbc:mysql://localhost:3306/mydb";
    private static final String USER = "root";
    private static final String PASS = "password";
    
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(DB_URL, USER, PASS);
    }
}
```

**Connection URL Formats:**

```java
// MySQL
"jdbc:mysql://hostname:port/database_name"
"jdbc:mysql://localhost:3306/studentdb"

// Oracle
"jdbc:oracle:thin:@hostname:port:SID"
"jdbc:oracle:thin:@localhost:1521:orcl"

// SQL Server
"jdbc:sqlserver://hostname:port;databaseName=dbname"
"jdbc:sqlserver://localhost:1433;databaseName=mydb"

// PostgreSQL
"jdbc:postgresql://hostname:port/database"
"jdbc:postgresql://localhost:5432/testdb"

// H2 (embedded)
"jdbc:h2:~/test"
"jdbc:h2:mem:testdb"
```

##### **Step 3: Create Statement**

```java
Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);

// Three types of statements:

// 1. Statement - for simple queries
Statement stmt = conn.createStatement();

// 2. PreparedStatement - for parameterized queries (recommended)
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM users WHERE id = ?"
);

// 3. CallableStatement - for stored procedures
CallableStatement cstmt = conn.prepareCall(
    "{call getUserById(?)}"
);
```

##### **Step 4: Execute Query/Update**

```java
// For SELECT queries - returns ResultSet
ResultSet rs = stmt.executeQuery("SELECT * FROM users");

// For INSERT, UPDATE, DELETE - returns row count
int rowsAffected = stmt.executeUpdate(
    "INSERT INTO users (name, email) VALUES ('John', 'john@example.com')"
);

// For any SQL - returns boolean
boolean hasResultSet = stmt.execute("SELECT * FROM users");
```

##### **Step 5: Process ResultSet**

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
    String email = rs.getString("email");
    
    System.out.println(id + ": " + name + " - " + email);
}
```

##### **Step 6: Close Resources**

```java
// Always close in reverse order: ResultSet → Statement → Connection
try {
    if (rs != null) rs.close();
    if (stmt != null) stmt.close();
    if (conn != null) conn.close();
} catch (SQLException e) {
    e.printStackTrace();
}

// Better: Use try-with-resources (Java 7+)
try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery("SELECT * FROM users")) {
    
    while (rs.next()) {
        // Process results
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

---

### **4. Complete JDBC Examples**

#### **4.1 Database Setup**

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS studentdb;
USE studentdb;

-- Create students table
CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INT,
    course VARCHAR(50),
    gpa DECIMAL(3,2),
    enrollment_date DATE,
    active BOOLEAN DEFAULT TRUE
);

-- Insert sample data
INSERT INTO students (name, email, age, course, gpa, enrollment_date) VALUES
('Alice Johnson', 'alice@example.com', 20, 'Computer Science', 3.85, '2023-09-01'),
('Bob Smith', 'bob@example.com', 22, 'Electrical Engineering', 3.60, '2023-09-01'),
('Charlie Brown', 'charlie@example.com', 21, 'Mechanical Engineering', 3.45, '2023-09-01'),
('Diana Prince', 'diana@example.com', 23, 'Computer Science', 3.95, '2023-09-01'),
('Eve Adams', 'eve@example.com', 20, 'Civil Engineering', 3.70, '2023-09-01');

-- Create courses table
CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_code VARCHAR(10) UNIQUE NOT NULL,
    course_name VARCHAR(100) NOT NULL,
    credits INT NOT NULL,
    department VARCHAR(50)
);

INSERT INTO courses (course_code, course_name, credits, department) VALUES
('CS101', 'Introduction to Programming', 4, 'Computer Science'),
('CS201', 'Data Structures', 4, 'Computer Science'),
('EE101', 'Circuit Analysis', 3, 'Electrical Engineering'),
('ME101', 'Engineering Mechanics', 3, 'Mechanical Engineering');
```

#### **4.2 Complete CRUD Operations Example**

```java
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

// Student Model Class
class Student {
    private int id;
    private String name;
    private String email;
    private int age;
    private String course;
    private double gpa;
    private Date enrollmentDate;
    private boolean active;
    
    // Constructors
    public Student() {}
    
    public Student(String name, String email, int age, String course, double gpa) {
        this.name = name;
        this.email = email;
        this.age = age;
        this.course = course;
        this.gpa = gpa;
    }
    
    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    
    public String getCourse() { return course; }
    public void setCourse(String course) { this.course = course; }
    
    public double getGpa() { return gpa; }
    public void setGpa(double gpa) { this.gpa = gpa; }
    
    public Date getEnrollmentDate() { return enrollmentDate; }
    public void setEnrollmentDate(Date enrollmentDate) { 
        this.enrollmentDate = enrollmentDate; 
    }
    
    public boolean isActive() { return active; }
    public void setActive(boolean active) { this.active = active; }
    
    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                ", course='" + course + '\'' +
                ", gpa=" + gpa +
                ", enrollmentDate=" + enrollmentDate +
                ", active=" + active +
                '}';
    }
}

// Database Configuration Class
class DatabaseConfig {
    private static final String DB_URL = "jdbc:mysql://localhost:3306/studentdb";
    private static final String USER = "root";
    private static final String PASSWORD = "password";
    
    // JDBC Driver (auto-loaded in JDBC 4.0+)
    static {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("MySQL Driver not found", e);
        }
    }
    
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(DB_URL, USER, PASSWORD);
    }
}

// Data Access Object (DAO) for Student
class StudentDAO {
    
    // CREATE - Insert a new student
    public boolean insertStudent(Student student) {
        String sql = "INSERT INTO students (name, email, age, course, gpa, enrollment_date) " +
                     "VALUES (?, ?, ?, ?, ?, ?)";
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql, 
                     Statement.RETURN_GENERATED_KEYS)) {
            
            pstmt.setString(1, student.getName());
            pstmt.setString(2, student.getEmail());
            pstmt.setInt(3, student.getAge());
            pstmt.setString(4, student.getCourse());
            pstmt.setDouble(5, student.getGpa());
            pstmt.setDate(6, new Date(System.currentTimeMillis()));
            
            int rowsAffected = pstmt.executeUpdate();
            
            if (rowsAffected > 0) {
                // Get auto-generated ID
                try (ResultSet generatedKeys = pstmt.getGeneratedKeys()) {
                    if (generatedKeys.next()) {
                        student.setId(generatedKeys.getInt(1));
                    }
                }
                return true;
            }
            
        } catch (SQLException e) {
            System.err.println("Error inserting student: " + e.getMessage());
            e.printStackTrace();
        }
        
        return false;
    }
    
    // READ - Get student by ID
    public Student getStudentById(int id) {
        String sql = "SELECT * FROM students WHERE id = ?";
        Student student = null;
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    student = extractStudentFromResultSet(rs);
                }
            }
            
        } catch (SQLException e) {
            System.err.println("Error getting student: " + e.getMessage());
            e.printStackTrace();
        }
        
        return student;
    }
    
    // READ - Get all students
    public List<Student> getAllStudents() {
        String sql = "SELECT * FROM students ORDER BY name";
        List<Student> students = new ArrayList<>();
        
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            while (rs.next()) {
                students.add(extractStudentFromResultSet(rs));
            }
            
        } catch (SQLException e) {
            System.err.println("Error getting all students: " + e.getMessage());
            e.printStackTrace();
        }
        
        return students;
    }
    
    // READ - Search students by course
    public List<Student> getStudentsByCourse(String course) {
        String sql = "SELECT * FROM students WHERE course = ? ORDER BY gpa DESC";
        List<Student> students = new ArrayList<>();
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, course);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    students.add(extractStudentFromResultSet(rs));
                }
            }
            
        } catch (SQLException e) {
            System.err.println("Error searching students: " + e.getMessage());
            e.printStackTrace();
        }
        
        return students;
    }
    
    // UPDATE - Update student information
    public boolean updateStudent(Student student) {
        String sql = "UPDATE students SET name = ?, email = ?, age = ?, " +
                     "course = ?, gpa = ?, active = ? WHERE id = ?";
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, student.getName());
            pstmt.setString(2, student.getEmail());
            pstmt.setInt(3, student.getAge());
            pstmt.setString(4, student.getCourse());
            pstmt.setDouble(5, student.getGpa());
            pstmt.setBoolean(6, student.isActive());
            pstmt.setInt(7, student.getId());
            
            int rowsAffected = pstmt.executeUpdate();
            return rowsAffected > 0;
            
        } catch (SQLException e) {
            System.err.println("Error updating student: " + e.getMessage());
            e.printStackTrace();
        }
        
        return false;
    }
    
    // DELETE - Delete student by ID
    public boolean deleteStudent(int id) {
        String sql = "DELETE FROM students WHERE id = ?";
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            
            int rowsAffected = pstmt.executeUpdate();
            return rowsAffected > 0;
            
        } catch (SQLException e) {
            System.err.println("Error deleting student: " + e.getMessage());
            e.printStackTrace();
        }
        
        return false;
    }
    
    // Helper method to extract Student from ResultSet
    private Student extractStudentFromResultSet(ResultSet rs) throws SQLException {
        Student student = new Student();
        student.setId(rs.getInt("id"));
        student.setName(rs.getString("name"));
        student.setEmail(rs.getString("email"));
        student.setAge(rs.getInt("age"));
        student.setCourse(rs.getString("course"));
        student.setGpa(rs.getDouble("gpa"));
        student.setEnrollmentDate(rs.getDate("enrollment_date"));
        student.setActive(rs.getBoolean("active"));
        return student;
    }
    
    // Advanced: Get students with GPA above threshold
    public List<Student> getStudentsWithHighGPA(double minGpa) {
        String sql = "SELECT * FROM students WHERE gpa >= ? ORDER BY gpa DESC";
        List<Student> students = new ArrayList<>();
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setDouble(1, minGpa);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    students.add(extractStudentFromResultSet(rs));
                }
            }
            
        } catch (SQLException e) {
            System.err.println("Error getting high GPA students: " + e.getMessage());
            e.printStackTrace();
        }
        
        return students;
    }
    
    // Get total student count
    public int getStudentCount() {
        String sql = "SELECT COUNT(*) FROM students";
        
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            if (rs.next()) {
                return rs.getInt(1);
            }
            
        } catch (SQLException e) {
            System.err.println("Error getting student count: " + e.getMessage());
            e.printStackTrace();
        }
        
        return 0;
    }
}

// Main Test Class
public class JDBCCompleteExample {
    
    public static void main(String[] args) {
        StudentDAO studentDAO = new StudentDAO();
        
        System.out.println("=== JDBC CRUD Operations Demo ===\n");
        
        // CREATE - Insert new student
        System.out.println("1. Creating new student...");
        Student newStudent = new Student(
            "Frank Miller", 
            "frank@example.com", 
            22, 
            "Computer Science", 
            3.75
        );
        
        if (studentDAO.insertStudent(newStudent)) {
            System.out.println("✓ Student created successfully with ID: " 
                             + newStudent.getId());
        } else {
            System.out.println("✗ Failed to create student");
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // READ - Get student by ID
        System.out.println("2. Reading student by ID...");
        Student student = studentDAO.getStudentById(1);
        if (student != null) {
            System.out.println("✓ Student found: " + student);
        } else {
            System.out.println("✗ Student not found");
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // READ - Get all students
        System.out.println("3. Reading all students...");
        List<Student> allStudents = studentDAO.getAllStudents();
        System.out.println("Total students: " + allStudents.size());
        for (Student s : allStudents) {
            System.out.printf("ID: %d | Name: %-20s | Course: %-25s | GPA: %.2f%n",
                s.getId(), s.getName(), s.getCourse(), s.getGpa());
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // READ - Search by course
        System.out.println("4. Searching students by course...");
        List<Student> csStudents = studentDAO.getStudentsByCourse("Computer Science");
        System.out.println("Computer Science students: " + csStudents.size());
        for (Student s : csStudents) {
            System.out.println("  - " + s.getName() + " (GPA: " + s.getGpa() + ")");
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // UPDATE - Update student information
        System.out.println("5. Updating student...");
        if (student != null) {
            student.setGpa(3.90);
            student.setCourse("Data Science");
            
            if (studentDAO.updateStudent(student)) {
                System.out.println("✓ Student updated successfully");
                System.out.println("Updated info: " + studentDAO.getStudentById(student.getId()));
            } else {
                System.out.println("✗ Failed to update student");
            }
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Advanced Query - Students with high GPA
        System.out.println("6. Getting students with GPA >= 3.70...");
        List<Student> highGpaStudents = studentDAO.getStudentsWithHighGPA(3.70);
        System.out.println("High GPA students: " + highGpaStudents.size());
        for (Student s : highGpaStudents) {
            System.out.printf("  - %-20s | GPA: %.2f | Course: %s%n",
                s.getName(), s.getGpa(), s.getCourse());
        }
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // Get statistics
        System.out.println("7. Database Statistics:");
        System.out.println("Total students: " + studentDAO.getStudentCount());
        
        System.out.println("\n" + "=".repeat(50) + "\n");
        
        // DELETE - Delete student (commented out for safety)
        /*
        System.out.println("8. Deleting student...");
        if (studentDAO.deleteStudent(newStudent.getId())) {
            System.out.println("✓ Student deleted successfully");
        } else {
            System.out.println("✗ Failed to delete student");
        }
        */
        
        System.out.println("\n=== Demo completed ===");
    }
}
```

#### **4.3 Batch Processing Example**

```java
import java.sql.*;
import java.util.List;

public class BatchProcessingExample {
    
    public static void insertStudentsBatch(List<Student> students) {
        String sql = "INSERT INTO students (name, email, age, course, gpa, enrollment_date) " +
                     "VALUES (?, ?, ?, ?, ?, ?)";
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            // Disable auto-commit for batch processing
            conn.setAutoCommit(false);
            
            for (Student student : students) {
                pstmt.setString(1, student.getName());
                pstmt.setString(2, student.getEmail());
                pstmt.setInt(3, student.getAge());
                pstmt.setString(4, student.getCourse());
                pstmt.setDouble(5, student.getGpa());
                pstmt.setDate(6, new Date(System.currentTimeMillis()));
                
                // Add to batch
                pstmt.addBatch();
            }
            
            // Execute batch
            int[] results = pstmt.executeBatch();
            
            // Commit transaction
            conn.commit();
            
            System.out.println("Batch insert completed: " + results.length + " rows affected");
            
        } catch (SQLException e) {
            System.err.println("Batch processing error: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    public static void main(String[] args) {
        List<Student> students = List.of(
            new Student("Grace Hopper", "grace@example.com", 21, "Computer Science", 3.88),
            new Student("Alan Turing", "alan@example.com", 22, "Mathematics", 3.95),
            new Student("Ada Lovelace", "ada@example.com", 20, "Computer Science", 3.92)
        );
        
        insertStudentsBatch(students);
    }
}
```

### **5. Transaction Management**

#### **5.1 What is a Transaction?**

A **transaction** is a sequence of operations performed as a single logical unit of work. It follows **ACID properties**:

- **A**tomicity: All or nothing
- **C**onsistency: Database remains in valid state
- **I**solation: Concurrent transactions don't interfere
- **D**urability: Committed changes persist

#### **5.2 Transaction Management Example**

```java
import java.sql.*;

public class TransactionExample {
    
    public static boolean transferFunds(int fromStudentId, int toStudentId, 
                                       double amount) {
        Connection conn = null;
        
        try {
            conn = DatabaseConfig.getConnection();
            
            // Start transaction
            conn.setAutoCommit(false);
            
            // Deduct from first student's account
            String deductSql = "UPDATE student_accounts SET balance = balance - ? " +
                              "WHERE student_id = ?";
            try (PreparedStatement deductStmt = conn.prepareStatement(deductSql)) {
                deductStmt.setDouble(1, amount);
                deductStmt.setInt(2, fromStudentId);
                int rows = deductStmt.executeUpdate();
                
                if (rows == 0) {
                    throw new SQLException("Source student not found");
                }
            }
            
            // Add to second student's account
            String addSql = "UPDATE student_accounts SET balance = balance + ? " + "WHERE student_id = ?";
            try (PreparedStatement addStmt = conn.prepareStatement(addSql)) {
                addStmt.setDouble(1, amount);
                addStmt.setInt(2, toStudentId);
                int rows = addStmt.executeUpdate();
                
                if (rows == 0) {
                    throw new SQLException("Destination student not found");
                }
            }
            
            // Commit transaction
            conn.commit();
            System.out.println("✓ Transaction completed successfully");
            return true;
            
        } catch (SQLException e) {
            System.err.println("✗ Transaction failed: " + e.getMessage());
            
            // Rollback on error
            if (conn != null) {
                try {
                    conn.rollback();
                    System.out.println("Transaction rolled back");
                } catch (SQLException ex) {
                    System.err.println("Rollback failed: " + ex.getMessage());
                }
            }
            
            return false;
            
        } finally {
            // Restore auto-commit and close connection
            if (conn != null) {
                try {
                    conn.setAutoCommit(true);
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    public static void main(String[] args) {
        // Transfer 100.0 from student 1 to student 2
        boolean success = transferFunds(1, 2, 100.0);
        
        if (success) {
            System.out.println("Funds transferred successfully");
        } else {
            System.out.println("Fund transfer failed");
        }
    }
}
```


### __1. javax.sql._ Package_*

The `javax.sql` package provides enhanced database connectivity features beyond basic JDBC:

**Key Interfaces:**

- **DataSource**: Factory for database connections (preferred over DriverManager)
- **RowSet**: Enhanced ResultSet with JavaBeans support
- **ConnectionPoolDataSource**: For connection pooling
- **XADataSource**: For distributed transactions

**Advantages over java.sql:**

- Connection pooling support
- Distributed transaction support
- Better performance and scalability

```java
// DataSource example
Context ctx = new InitialContext();
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/MyDB");
Connection conn = ds.getConnection();
```

### **2. Accessing Database from JSP Page**

**Three approaches:**

**A. Direct JDBC in JSP (Not Recommended)**

```jsp
<%@ page import="java.sql.*" %>
<%
    Class.forName("com.mysql.jdbc.Driver");
    Connection conn = DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/mydb", "user", "password");
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery("SELECT * FROM users");
    
    while(rs.next()) {
        out.println(rs.getString("name") + "<br>");
    }
    rs.close();
    stmt.close();
    conn.close();
%>
```

**B. Using JavaBeans (Better)**

```jsp
<jsp:useBean id="dbBean" class="com.example.DatabaseBean" scope="session"/>
<jsp:setProperty name="dbBean" property="*"/>
<%
    List<User> users = dbBean.getUsers();
    for(User user : users) {
        out.println(user.getName());
    }
%>
```

**C. Using JSTL & EL (Best Practice)**

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/sql" prefix="sql" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<sql:query var="result" dataSource="${myDataSource}">
    SELECT * FROM users
</sql:query>

<c:forEach var="row" items="${result.rows}">
    ${row.name}<br>
</c:forEach>
```

### **3. Application-Specific Database Actions**

Common database operations in web applications:

**User Authentication:**

```java
public boolean authenticateUser(String username, String password) {
    String sql = "SELECT * FROM users WHERE username=? AND password=?";
    PreparedStatement pstmt = conn.prepareStatement(sql);
    pstmt.setString(1, username);
    pstmt.setString(2, password);
    ResultSet rs = pstmt.executeQuery();
    return rs.next();
}
```

**CRUD Operations:**

- **Create**: Insert new records
- **Read**: Retrieve and display data
- **Update**: Modify existing records
- **Delete**: Remove records

**Transaction Management:**

```java
try {
    conn.setAutoCommit(false);
    // Multiple SQL operations
    stmt.executeUpdate("UPDATE accounts SET balance=balance-100 WHERE id=1");
    stmt.executeUpdate("UPDATE accounts SET balance=balance+100 WHERE id=2");
    conn.commit();
} catch(Exception e) {
    conn.rollback();
}
```

### **4. Deploying JavaBeans in JSP**

**JavaBean Requirements:**

- Public class with no-arg constructor
- Private properties with public getters/setters
- Serializable (optional but recommended)

**Example Bean:**

```java
public class UserBean implements Serializable {
    private String name;
    private String email;
    
    public UserBean() {}
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

**Using in JSP:**

```jsp
<!-- Instantiate bean -->
<jsp:useBean id="user" class="com.example.UserBean" scope="session"/>

<!-- Set properties -->
<jsp:setProperty name="user" property="name" value="John"/>
<jsp:setProperty name="user" property="*"/> <!-- From request params -->

<!-- Get properties -->
<jsp:getProperty name="user" property="name"/>
```

**Scopes:**

- **page**: Available only in current page
- **request**: Available during request processing
- **session**: Available throughout user session
- **application**: Available to all users

### **5. Introduction to Struts Framework**

**What is Struts?** Apache Struts is an MVC (Model-View-Controller) framework for building Java web applications.

**Architecture Components:**

**1. Model:** Business logic and data (JavaBeans, EJBs) **2. View:** Presentation layer (JSP pages, custom tags) **3. Controller:** ActionServlet coordinates workflow

**Key Components:**

**ActionServlet:**

- Front controller that handles all requests
- Configured in web.xml

**Action Class:**

```java
public class LoginAction extends Action {
    public ActionForward execute(ActionMapping mapping,
                                ActionForm form,
                                HttpServletRequest request,
                                HttpServletResponse response) {
        LoginForm loginForm = (LoginForm) form;
        // Business logic
        if(validate(loginForm)) {
            return mapping.findForward("success");
        }
        return mapping.findForward("failure");
    }
}
```

**ActionForm:**

```java
public class LoginForm extends ActionForm {
    private String username;
    private String password;
    
    // Getters and setters
    
    public ActionErrors validate(ActionMapping mapping,
                                HttpServletRequest request) {
        ActionErrors errors = new ActionErrors();
        if(username == null || username.length() == 0) {
            errors.add("username", new ActionError("error.username.required"));
        }
        return errors;
    }
}
```

**struts-config.xml:**

```xml
<struts-config>
    <form-beans>
        <form-bean name="loginForm" type="com.example.LoginForm"/>
    </form-beans>
    
    <action-mappings>
        <action path="/login"
                type="com.example.LoginAction"
                name="loginForm"
                scope="request">
            <forward name="success" path="/welcome.jsp"/>
            <forward name="failure" path="/login.jsp"/>
        </action>
    </action-mappings>
</struts-config>
```

**Benefits:**

- Separation of concerns (MVC pattern)
- Centralized configuration
- Built-in validation
- Reusable components
- Tag libraries for JSP

---

### **6. Semantic Web**

**Introduction:** The Semantic Web is an extension of the current web where information has well-defined meaning, enabling computers to understand and process data intelligently.

**Growth and Evolution:**

**Timeline:**

- **1989**: Tim Berners-Lee invents WWW (Web 1.0 - read-only)
- **2001**: Semantic Web concept introduced
- **2006**: Web 2.0 (interactive, user-generated content)
- **2010s**: Web 3.0 (Semantic Web + AI)

**Evolution Stages:**

1. **Web 1.0**: Static HTML pages
2. **Web 2.0**: Interactive applications, social media
3. **Web 3.0**: Intelligent, connected data

**Goals and Vision:**

**Primary Goals:**

- Make web content machine-readable
- Enable intelligent data integration
- Facilitate automated reasoning
- Create a global knowledge base

**Vision:**

- Computers understand context and meaning
- Automated data discovery and integration
- Intelligent agents perform complex tasks
- Personalized information delivery

**Need for Semantic Web:**

**Current Web Problems:**

- Data in various formats (HTML, PDF, databases)
- No standard way to represent meaning
- Search engines rely on keywords, not meaning
- Difficult data integration across sources

**Solutions Provided:**

- Standard formats for data exchange (RDF, OWL)
- Formal ontologies define relationships
- Reasoning engines infer new knowledge
- Intelligent querying with SPARQL

**Problems Addressed:**

1. **Information Overload**: Better search and filtering
2. **Data Silos**: Cross-platform integration
3. **Ambiguity**: Context-aware interpretation
4. **Limited Automation**: Intelligent agents
5. **Poor Interoperability**: Standard vocabularies

**Semantic Web Architecture (Layer Cake):**

```
┌─────────────────────────────────┐
│    User Interface & Applications │
├─────────────────────────────────┤
│    Trust & Proof                │
├─────────────────────────────────┤
│    Cryptography                 │
├─────────────────────────────────┤
│    Logic & Rules (SWRL)         │
├─────────────────────────────────┤
│    Ontology (OWL)               │
├─────────────────────────────────┤
│    Taxonomy (RDFS)              │
├─────────────────────────────────┤
│    Data Model (RDF)             │
├─────────────────────────────────┤
│    Syntax (XML)                 │
├─────────────────────────────────┤
│    URI & Unicode                │
└─────────────────────────────────┘
```

**Layer Details:**

**1. URI/IRI:** Unique identifiers for resources **2. XML:** Syntax for structured documents **3. RDF (Resource Description Framework):** Data model (subject-predicate-object triples) **4. RDFS:** Schema language for RDF **5. OWL (Web Ontology Language):** Rich ontology language **6. SPARQL:** Query language for RDF **7. Rules (SWRL):** Semantic Web Rule Language **8. Trust/Proof:** Verification mechanisms

**Applications:**

**1. Healthcare:**

- Electronic health records integration
- Drug interaction checking
- Medical diagnosis support

**2. E-Commerce:**

- Intelligent product search
- Personalized recommendations
- Price comparison across sites

**3. Social Networks:**

- Friend-of-a-friend (FOAF) networks
- Interest-based connections
- Content sharing

**4. Knowledge Management:**

- Corporate knowledge bases
- Research paper linking (linked data)
- Digital libraries

**5. IoT (Internet of Things):**

- Smart home device coordination
- Sensor data integration
- Automated decision-making

**6. Government:**

- Open data initiatives
- Cross-agency information sharing
- Public service discovery

**Technologies:**

**RDF Example:**

```xml
<rdf:Description rdf:about="http://example.org/person/john">
    <foaf:name>John Doe</foaf:name>
    <foaf:age>30</foaf:age>
    <foaf:knows rdf:resource="http://example.org/person/jane"/>
</rdf:Description>
```

**SPARQL Query:**

```sparql
SELECT ?name ?email
WHERE {
    ?person foaf:name ?name .
    ?person foaf:mbox ?email .
    ?person foaf:age ?age .
    FILTER (?age > 25)
}
```

**Challenges:**

- Complexity of implementation
- Lack of widespread adoption
- Scalability concerns
- Privacy and security issues
- Need for standardized ontologies