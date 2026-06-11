## Web Technology Lab File

### Practical 1

**Design a HTML page to display your CV.**

```html
<!doctype html>
<html>
  <head>
    <title>Sankalp - CV</title>
  </head>
  <body>
    <h1>Sankalp Omar</h1>
    <p>Student | Developer</p>
    <hr />

    <h2>Profile</h2>
    <p>
      Motivated student with a strong interest in software development,
      particularly in web technologies and Python-based applications.
    </p>

    <h2>Skills</h2>
    <ul>
      <li>JavaScript (Vanilla)</li>
      <li>HTML</li>
      <li>Python</li>
      <li>Basic Networking</li>
      <li>Linux (Alpine, Termux)</li>
    </ul>

    <h2>Projects</h2>
    <ul>
      <li>To-do list app</li>
      <li>Calculator app</li>
      <li>Personal Portfolio Website</li>
      <li>Word Counter app</li>
    </ul>

    <h2>Education</h2>
    <p>Currently pursuing undergraduate studies in Computer Science, HBTU.</p>

    <h2>Interests</h2>
    <ul>
      <li>Full Stack Development</li>
      <li>System Design</li>
      <li>AI / ML</li>
    </ul>

    <h2>Contact</h2>
    <p>Email: sankalpomar6115@gmail.com</p>
  </body>
</html>
```

![cv-page-00001](../College/WebTech/Practicals/Outputs/cv-page-00001.jpg)





### Practical 2

Design a HTML form to reserve a railway ticket.

```html
<!doctype html>
<html>
  <head>
    <title>Railway Ticket Reservation</title>
  </head>
  <body>
    <h1>Railway Ticket Reservation Form</h1>

    <form>
      <fieldset>
        <legend>Passenger Details</legend>

        <label>Full Name:</label><br />
        <input type="text" name="fullname" required /><br /><br />

        <label>Age:</label><br />
        <input type="number" name="age" required /><br /><br />

        <label>Gender:</label><br />
        <input type="radio" name="gender" value="Male" /> Male
        <input type="radio" name="gender" value="Female" /> Female
        <input type="radio" name="gender" value="Other" /> Other<br /><br />
      </fieldset>

      <fieldset>
        <legend>Journey Details</legend>

        <label>From:</label><br />
        <input type="text" name="from" required /><br /><br />

        <label>To:</label><br />
        <input type="text" name="to" required /><br /><br />

        <label>Date of Journey:</label><br />
        <input type="date" name="date" required /><br /><br />

        <label>Class:</label><br />
        <select name="class">
          <option>Sleeper</option>
          <option>AC 3 Tier</option>
          <option>AC 2 Tier</option>
          <option>First Class</option></select
        ><br /><br />
      </fieldset>

      <fieldset>
        <legend>Contact Details</legend>

        <label>Mobile Number:</label><br />
        <input type="tel" name="mobile" required /><br /><br />

        <label>Email:</label><br />
        <input type="email" name="email" /><br /><br />
      </fieldset>

      <input type="submit" value="Reserve Ticket" />
      <input type="reset" value="Clear" />
    </form>
  </body>
</html>
```

![railway-ticket-page-00001](../College/WebTech/Practicals/Outputs/railway-ticket-page-00001.jpg)





### Practical 3

Write a Java Script program that finds the greatest common divisor of two numbers.

```html
<!doctype html>
<html>
  <head>
    <title>GCD Calculator</title>
  </head>
  <body>
    <script>
      function gcd(a, b) {
        while (b !== 0) {
          let temp = b;
          b = a % b;
          a = temp;
        }
        return a;
      }

      let num1 = 88;
      let num2 = 18;

      document.write(
        "GCD of " + num1 + " and " + num2 + " is " + gcd(num1, num2),
      );
    </script>
  </body>
</html>
```

![gcd-page-00001](../College/WebTech/Practicals/Outputs/gcd-page-00001.jpg)





### Practical 4

 In the form mentioned in problem 2 to reserve a railway ticket add the following validations using java Script. o From city and to city are two different cities. o Age of passengers should not be greater than 150. o Name of the passenger should be a string of a maximum length



```html
<!doctype html>
<html>
  <head>
    <title>Railway Ticket Reservation</title>
    <script>
      function validateForm() {
        let name = document.forms["ticketForm"]["fullname"].value.trim();
        let age = parseInt(document.forms["ticketForm"]["age"].value);
        let from = document.forms["ticketForm"]["from"].value
          .trim()
          .toLowerCase();
        let to = document.forms["ticketForm"]["to"].value.trim().toLowerCase();
        if (name.length > 50 || !/^[a-zA-Z\s]+$/.test(name)) {
          alert("Name must be a string with maximum length of 50 characters.");
          return false;
        }
        if (age > 150) {
          alert("Age cannot be greater than 150.");
          return false;
        }
        if (from === to) {
          alert("From city and To city must be different.");
          return false;
        }

        return true;
      }
    </script>
  </head>
  <body>
    <h1>Railway Ticket Reservation Form</h1>

    <form name="ticketForm" onsubmit="return validateForm();">
      <fieldset>
        <legend>Passenger Details</legend>

        <label>Full Name:</label><br />
        <input type="text" name="fullname" required /><br /><br />

        <label>Age:</label><br />
        <input type="number" name="age" required /><br /><br />

        <label>Gender:</label><br />
        <input type="radio" name="gender" value="Male" /> Male
        <input type="radio" name="gender" value="Female" /> Female
        <input type="radio" name="gender" value="Other" /> Other<br /><br />
      </fieldset>

      <fieldset>
        <legend>Journey Details</legend>

        <label>From:</label><br />
        <input type="text" name="from" required /><br /><br />

        <label>To:</label><br />
        <input type="text" name="to" required /><br /><br />

        <label>Date of Journey:</label><br />
        <input type="date" name="date" required /><br /><br />

        <label>Class:</label><br />
        <select name="class">
          <option>Sleeper</option>
          <option>AC 3 Tier</option>
          <option>AC 2 Tier</option>
          <option>First Class</option></select
        ><br /><br />
      </fieldset>

      <fieldset>
        <legend>Contact Details</legend>

        <label>Mobile Number:</label><br />
        <input type="tel" name="mobile" required /><br /><br />

        <label>Email:</label><br />
        <input type="email" name="email" /><br /><br />
      </fieldset>

      <input type="submit" value="Reserve Ticket" />
      <input type="reset" value="Clear" />
    </form>
  </body>
</html>

```

![railway-with-validation-page-00001](../College/WebTech/Practicals/Outputs/railway-with-validation-page-00001.jpg)





### Practical 5

Write a program for illustrating client/server-side scripting with help of ASP

```asp
<html>
<head>
    <title>Client/Server Side Scripting (ASP)</title>
    <script>
        function validateForm() {
            var name = document.getElementById("name").value;
            if (name === "") {
                alert("Name cannot be empty");
                return false;
            }
            return true;
        }
    </script>
</head>
<body>
<h2>Enter Your Name</h2>
<form method="post" action="form.asp" onsubmit="return validateForm()">
    <input type="text" name="username" id="name">
    <input type="submit" value="Submit">
</form>

<hr>
<%
if Request.ServerVariables("REQUEST_METHOD") = "POST" then
    dim uname
    uname = Request.Form("username")

    if uname <> "" then
        Response.Write("<h3>Hello, " & uname & "</h3>")
    else
        Response.Write("<h3>No name received</h3>")
    end if
end if
%>
</body>
</html>
```

![form-page-00001](../College/WebTech/Practicals/Outputs/form-page-00001.jpg)



### Practical 6

Write a piece of code in XML for creating DTD, which specifies set of rules.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE reservation [
    <!ELEMENT reservation (passenger, journey, contact)>

    <!ELEMENT passenger (name, age, gender)>
    <!ELEMENT name (#PCDATA)>
    <!ELEMENT age (#PCDATA)>
    <!ELEMENT gender (#PCDATA)>

    <!ELEMENT journey (from, to, date, class)>
    <!ELEMENT from (#PCDATA)>
    <!ELEMENT to (#PCDATA)>
    <!ELEMENT date (#PCDATA)>
    <!ELEMENT class (#PCDATA)>

    <!ELEMENT contact (mobile, email)>
    <!ELEMENT mobile (#PCDATA)>
    <!ELEMENT email (#PCDATA)>
]>

<reservation>
    <passenger>
        <name>Sankalp</name>
        <age>21</age>
        <gender>Male</gender>
    </passenger>

    <journey>
        <from>Kanpur</from>
        <to>Delhi</to>
        <date>2026-05-10</date>
        <class>AC 3 Tier</class>
    </journey>

    <contact>
        <mobile>9876543210</mobile>
        <email>sankalp@example.com</email>
    </contact>
</reservation>
```

![xml-dtd-page-00001](../College/WebTech/Practicals/Outputs/xml-dtd-page-00001.jpg)



### Practical 7

Create style sheet in CSS/XSL and display the document in Internet Explorer.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:template match="/">
<html>
<head>
    <title>Reservation Details</title>
</head>
<body>
<h2>Railway Reservation Details</h2>
<table border="1">
    <tr><th colspan="2">Passenger</th></tr>
    <tr>
        <td>Name</td>
        <td><xsl:value-of select="reservation/passenger/name"/></td>
    </tr>
    <tr>
        <td>Age</td>
        <td><xsl:value-of select="reservation/passenger/age"/></td>
    </tr>
    <tr>
        <td>Gender</td>
        <td><xsl:value-of select="reservation/passenger/gender"/></td>
    </tr>

    <tr><th colspan="2">Journey</th></tr>
    <tr>
        <td>From</td>
        <td><xsl:value-of select="reservation/journey/from"/></td>
    </tr>
    <tr>
        <td>To</td>
        <td><xsl:value-of select="reservation/journey/to"/></td>
    </tr>
    <tr>
        <td>Date</td>
        <td><xsl:value-of select="reservation/journey/date"/></td>
    </tr>
    <tr>
        <td>Class</td>
        <td><xsl:value-of select="reservation/journey/class"/></td>
    </tr>

    <tr><th colspan="2">Contact</th></tr>
    <tr>
        <td>Mobile</td>
        <td><xsl:value-of select="reservation/contact/mobile"/></td>
    </tr>
    <tr>
        <td>Email</td>
        <td><xsl:value-of select="reservation/contact/email"/></td>
    </tr>
</table>
</body>
</html>
</xsl:template>
</xsl:stylesheet>
```

![style-page-00001](../College/WebTech/Practicals/Outputs/style-page-00001.jpg)