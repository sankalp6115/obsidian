# From Code to Server: How to deploy your website in a Virtual Machine

Recently, I got an assignment from my College Professor (he is very ambitious), first i will explain the problem statement:

## Question 1

In this question, we are required to host a website created using HTML, CSS and JS on the frontend and Java Servlets on the backend and DerbyDB for database and data persistence. This website will be hosted on host machine.

Hosting of this website will take place using Apache Tomcat Web Server.
Tomcat is chosen because it is a lightweight, open-source Java Servlet  container that natively supports Jakarta EE Servlets.

This website will have 3 features:

Files:

```bash
cd myapp
```

-   <u>index.html</u> 

    ​	Index is the central landing page that links to all three features via navigation hyperlinks. 

-   <u>feature1.html</u>

    ​	Feature 1 has string operations. It contains several string operations like reverse, length,uppercase,lowercase,word count, frequency count etc. 
    It is implemented using **HTML**, **CSS** and **Javascript**. It is a static app, that is directly servable by tomcat without needing any backend logic and connection and handles everything on the client-side.

-   <u>feature2.html</u>

    ​	Feature 2 is a calculator app that can handle calculations with operations such as add,subtract,multiply,divide,trigonometric calculations, exponents etc. 
    It is implemented using **HTML**, **CSS** and **Javascript**. It is a static app, that is directly servable by tomcat without needing any backend logic and connection and handles everything on the client-side.

-   <u>feature3.html</u>

    ​	Feature 3 is a CRUD (Create-Read-Update-Delete) App that can be used to perform CRUD operations on Derby DB (which is a SQL type Relational Database Management System).  It contains a form which user can fill to send data to servlet using **POST** method and then servlet can send/receive data from DerbyDB.
    This app is implemented using **HTML**, **CSS** and **Javascript** on the frontend, and uses **Java** servlets on the backend that contains the connection to **DerbyDB** and DB operations.

Frontend UI Design:

<div style="
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
  grid-auto-flow: dense;
">
  <img src="../../../Library/Application%20Support/typora-user-images/image-20260401020847210.png" style="width: 100%;">
  <img src="../../../Library/Application%20Support/typora-user-images/image-20260401020923747.png" style="width: 100%;">
  <img src="../../../Library/Application%20Support/typora-user-images/image-20260401020958035.png" style="width: 100%;">
  <img src="../../../Library/Application%20Support/typora-user-images/image-20260401021014967.png" style="width: 100%;">
</div>




 ```bash
 cd WEB-INF/classes
 ```

-   <u>servlet.class</u>

    ​	This is compiled from servlet.java that contains the backend logic for feature3 that mainly involves CRUD operations on **DerbyDB**. It uses JakartaEE of Java. 

    Compiling servlet.java using servlet-api.jar found in lib folder inside apache-tomcat directory.

    ```bash
    javac -cp ".:tomcat/11.0.20/libexec/lib/servlet-api.jar" "tomcat/11.0.20/libexec/webapps/myapp/servlet.java
    ```

    or

    ```bash
    javac -cp ".:tomcat/11.0.20/libexec/lib/servlet-api.jar" \
          tomcat/11.0.20/libexec/webapps/myapp/servlet.java
    ```

    

-   <u>DerbyDB</u>

    ​	**Apache Derby** is an open-source, relational database management system (RDBMS) written entirely in Java. The project has a small footprint of about 3.5 megabytes and can be deployed in two primary modes: embedded within a Java application or as a client/server using a network server. We are using it in embedded mode within our application to store and persist client data.

    DB Schema

    ```sql
    CREATE TABLE users (
      id    INT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
      name  VARCHAR(100),
      email VARCHAR(100)
    );
    ```

    >    Instead of a `web.xml` deployment descriptor, we use Jakarta Servlet  annotations directly in the servlet source code: ```java @WebServlet("/user") public class servlet extends HttpServlet { ... } ``` This tells Tomcat to map all requests to `/user` directly to this servlet  class, eliminating the need for a separate XML configuration file.

    

    Install Apache Tomcat Web Server on system.
    After installing Tomcat: 

    We make the below folder structure and drop it in:

    -    /opt/homebrew/opt/tomcat/<version-name>/webapps/<your-app-name>
    -   C:/Program Files/Apache Software Foundation/Tomcat<version-name>/webapps/<your-app-name>

    ### **Folder structure**

    myapp/
    ├─ db/
    ├─ WEB-INF/
    │  ├─ classes/
    │  │  └─ servlet.class
    │  └─ lib/
    │     └─ derby.jar
    ├─ feature1.html
    ├─ feature2.html
    ├─ feature3.html
    └─ index.html

    ## System Design

    -   Client (Browser)
    -   Frontend (HTML/CSS/JS)
    -   Backend (Servlets)
    -   Database (DerbyDB)
    -   Server (Tomcat)

    ## Data Flow

    **Feature 1 & 2 (Client-side):**

    1.  User inputs data
    2.  JavaScript processes it
    3.  Result displayed instantly

    **Feature 3 (CRUD):**

    1.  User submits form (POST request)
    2.  Request reaches servlet
    3.  Servlet processes request
    4.  JDBC connects to DerbyDB
    5.  SQL Query executed
    6.  Response sent back

    

-   #### Now traverse to /opt/homebrew/opt/tomcat/bin

```bash
./startup.sh
```

-   Or hit ```brew services start tomcat``` on Mac (if installed via Homebrew).

-   Or run ```startup.bat``` in Windows.

-   This starts our tomcat server. Now our website is live.

-   Visit localhost:8080/<app-name>

-   We can also access our live website on other devices that are on the same network as our PC by visiting:

    ​	```http://<pc-ip>:8080```

    <img src="../../../Library/Application%20Support/typora-user-images/image-20260330022103359.png" alt="image-20260330022103359" style="zoom:50%;" />

## Question2

In this question, the website we hosted on our local machine, the same website has to be hosted on a Alpine Linux VM.
For this purpose, we use VirtualBox. (On mac laptops, use UTM.)

**Why Bridged Network?** 
By default, VirtualBox uses NAT networking, which means the VM gets its  own private IP that is only accessible from the host machine itself.  Bridged Adapter mode makes the VM appear as a separate device on the  same physical network - it gets its own IP from the router/hotspot,  making it reachable from phones, tablets, and other laptops on the same network.

**Why UTM on Apple Silicon Macs?**
VirtualBox does not support ARM architecture, which is what Apple Silicon use. UTM uses QEMU under the hood and supports both ARM and x86 emulation, making it the correct choice for Mac laptops in our group.

Download alpine linux ISO from internet minding the architecture of CPU of our system.
After downloading Alpine, create new VM (keep in mind to change network configuration to Bridged Network for proper working), setup Alpine:

-   Enter user as root.

-   ```setup-alpine```

-   remove "#" from second line of repositories to allow softwares from being downloaded.

-   ```apk update```

-   ```apk upgrade```

-   ```apk add openjdk21```

-   ```apk add tomcat```

    ```bash
    apk add openssh
    rc-service sshd start
    rc-update add sshd default
    ```

    Important: Traverse to /etc/apk

    >   ```bash
    >   nano repositories
    >   ```
    >
    >   Alpine Linux by default only enables the main package repository. The  community repository (second line in `/etc/apk/repositories`) contains  packages like Tomcat and nginx that we need. Uncommenting it by removing  the `#` gives `apk` access to these packages.

-   SSH into host machine terminal.

-   ```ssh root@<vm-ip>```

>    SSH is installed so we can control the Alpine VM directly from our  host machine's terminal, and use `scp` to transfer our app files  without needing to use the VM's console window.

-   Copy folder of our app into Linux.

    -   ```scp -r <source-of-host-machine-folder> root@<vm-ip>:<destination-on-virtual-machine>``` 

    ## Folder Structure

    myapp/
    ├── WEB-INF/
    │   ├── classes/
    │   │   └── servlet.class
    ├── feature1.html
    ├── feature2.html
    ├── feature3.html
    └── index.html

-   Paste app folder in ```/opt/apache-tomcat-<version>/webapps```

-   Travel to ```/opt/apache-tomcat-<version>/bin```

-   ```./startup.sh``` to start tomcat.

-   Visit ```http://<vm-ip>:8080/<app-name>``` in browser.

-   Our website is running on Alpine VM and completely accessible on other devices.

### Where to place the files and why?

**Why `webapps/`?** Tomcat's `webapps/` directory is its deployment directory - any folder placed here is automatically detected, loaded, and served by Tomcat as a web application. Each subfolder becomes a context path, so our app becomes accessible at `http://<vm-ip>:8080/myapp/`. This is Tomcat's standard and expected deployment location.

**Why `WEB-INF/classes/` for the servlet?** `WEB-INF/` is a protected directory - its contents are never served directly to the client. Tomcat looks specifically inside `WEB-INF/classes/` for compiled servlet `.class` files. Placing `servlet.class` here makes it discoverable by Tomcat's class loader while keeping it inaccessible from the browser directly.

**Why `WEB-INF/lib/` for derby.jar?** Tomcat's class loader automatically includes any `.jar` files found in `WEB-INF/lib/` in the application's classpath at runtime. Placing `derby.jar` here ensures the servlet can find and use Derby's JDBC driver without any manual classpath configuration.

**Why are HTML files at the root of `myapp/`?** HTML, CSS and JS files are static resources. Tomcat serves them directly from the application root, so `feature1.html` becomes accessible at `http://<vm-ip>:8080/myapp/feature1.html` without any routing or servlet involvement.

### Code Explanation

## Frontend Code Explanation

**Feature 1 and Feature 2** are entirely client-side. When the browser requests `feature1.html` or `feature2.html`, Tomcat simply reads the file off disk and sends it to the browser - no Java code runs. All logic (string operations, math calculations) executes inside the browser via JavaScript. The server is only involved in delivering the file, not in processing anything.

**Feature 3 frontend (`feature3.html`)** contains an HTML form that collects user input and sends it as an HTTP POST request to `/myapp/user`. JavaScript handles dynamic updates to the displayed table after each operation.

## Backend Code Explanation

The backend is a single Java servlet mapped to the `/user` endpoint using the `@WebServlet("/user")` annotation. It extends `HttpServlet` and handles two HTTP methods:

**`doPost()`** - handles Create and Update operations:

-   Reads form parameters (`name`, `email`, `course`) from the request.
-   Uses JDBC to execute the appropriate `INSERT` or `UPDATE` SQL on DerbyDB.
-   Redirects back to the list view after completion.

Derby runs in **embedded mode** - it is not a separate process or server. It starts automatically when the servlet's JDBC connection is first opened, and the database files are stored as a folder on the VM's filesystem at the path specified in the connection URL.

## Question3

In this question, we are asked to design a split system, that uses two different VMs, both of Alpine linux.
In one of the machines, feature1 (String Operations) and feature2 (Calculator) will be run and **nginx** will be used for reverse proxy the Java server. 
The other VM will have tomcat, and servlet running on it, and feature 3 (CRUD on Derby) running on it.

We are using **Single computer, Dual Virtual Machine Architecture**.
We will make an Alpine VM,  install nginx on it.

-   ```apk add nginx```
-   Put ```feature1.html```,```feature2.html```,```feature3.html```,```index.html``` in VMs ```/var/www/localhost/htdocs``` directly.
-   Configure nginx.conf (configuration file of nginx) (```/etc/nginx/http.d/default.conf```) and (```/etc/nginx/nginx.conf```).

The nginx configuration on VM-1 serves two purposes: 

1.   **Static file serving** - directly serves `feature1.html`, `feature2.html`,    `feature3.html` and `index.html` from `/var/www/localhost/htdocs`
2.   **Reverse Proxy** - forwards any request hitting `/myapp/user` to Tomcat running on VM-2.

```nginx
# /etc/nginx/http.d/default.conf
server {
    listen 80;
    # Serve static frontend files
    location / {
        root /var/www/localhost/htdocs;
        index index.html;
    }
    # Reverse proxy - CRUD requests forwarded to Tomcat on VM-2
    location /myapp/user {
        proxy_pass http://<VM-IP>:8080/myapp/user;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Create another VM, with similar configuration.
This VM will contain codes for servlet, and DerbyDB.

-   Folder Structure:

```txt
/opt/tomcat/webapps/<App-name>/
├── WEB-INF/
   └── classes/
           └── servlet.class
/opt/tomcat/lib/
└── derby.jar
```

-   **Now start nginx on VM-1**
-   Start tomcat on VM-2
-   Hit ```http://<vm-ip>/index.html``` on browser.

The website is live and is working from 2 different virtual machines, nginx being the frontend & reverse proxy, and tomcat serving the backend.

### VM-to-VM Communication
 Both VMs are configured with **Bridged Networking** in VirtualBox. This places both VMs on the same physical network as the host machine,  allowing VM-1 to reach VM-2 directly using its local IP address. The  `proxy_pass` directive in nginx uses this IP to forward requests.

>   **Note:** Since we are using a mobile hotspot for network, the IP address of VM-2 may change when the network resets. In that case, update`<VM2-IP>` in this config and reload nginx:
>
>   ```bash
>   nginx -t
>   ```
>
>   ```bash
>   rc-service nginx restart 
>   ```

### How it works end-to-end

```
User Browser
    │
    │  GET /index.html, /feature1.html, /feature2.html, /feature3.html
    ▼
Nginx (VM-1) ──── serves directly from /var/www/localhost/htdocs
    │
    │  POST /myapp/user  (form submission from feature3.html)
    ▼
Nginx reverse proxies ──────────────────────────────►  Tomcat (VM-2):8080
                                                            │
                                                            ▼
                                                       servlet.class
                                                            │
                                                            ▼
                                                         DerbyDB
                                                            │
                                                       SQL Response
                                                            │
HTML Response sent back through Nginx   ◄───────────────────┘
					to User Browser
```

### Data Flow

1. User opens `http://<VM1-IP>/index.html` - served by Nginx directly
2. User navigates to Feature 3 - `feature3.html` is served by Nginx directly
3. User fills the CRUD form and submits - form hits `POST /myapp/user`
4. Nginx intercepts `/myapp/user` and reverse proxies it to Tomcat on VM-2
5. Tomcat receives the request, runs `servlet.class`
6. Servlet performs the required CRUD operation on DerbyDB
7. Response travels back through Nginx to the user's browser

<img src="../../../Library/Application%20Support/typora-user-images/image-20260401010655966.png" alt="image-20260401010655966" style="zoom:50%;" />

### Concept Used

This architecture uses **Reverse Proxying** - Nginx acts as the single point 
of contact for the client. The client has no knowledge of VM-2's existence or 
its IP. Nginx internally delegates backend requests to Tomcat on VM-2 over the 
private network between the two VMs.

This separation of concerns gives us:
- **VM-1 (Nginx):** Handles all client-facing traffic, serves static content 
  at high speed
- **VM-2 (Tomcat):** Handles only application logic and database operations, 
  never directly exposed to the client

---

## Question 4

Since managerial restrictions prohibit the use of Apache Tomcat (or any other  web server) on VM-2, we need an alternative way to handle backend CRUD  operations.

### Architecture Overview

VM-1 (Nginx) remains completely unchanged from Q3. Only VM-2 changes - Tomcat 
is removed and replaced with a custom backend service.

```
User Browser
    │
    ▼
Nginx (VM-1) - static files served directly
    │
    │  POST /myapp/user
    ▼
Custom Backend on VM-2 (no Tomcat)
    │
    ▼
DerbyDB
```

### Solution: Custom Java HTTP Server + DerbyDB

In this solution, we use Java's built-in `com.sun.net.httpserver.HttpServer`  class, which provides a lightweight HTTP server without requiring any external  web server like Tomcat. It listens on port 8081 and handles CRUD requests by  parsing POST body parameters and executing SQL on DerbyDB.Folder Structure on VM-2.

```
/opt/javaserver/
├── Server.java     ← custom HTTP server source
├── Server.class    ← compiled class
└── derby.jar       ← DerbyDB driver
```

#### How to Run on VM-2

```bash
cd /opt/myapp
# Compile
javac -cp .:derby.jar Server.java
# Run (background)
java -cp .:derby.jar Server &
```

Server starts listening on port 8081. Nginx on VM-1 reverse proxies `/myapp/user` to `http://<VM2-IP>:8081/myapp/user` - same config as Q3,  no nginx changes needed.

#### How it Works

```
Incoming HTTP Request (raw TCP)
    │
    ▼
ServerSocket (port 8081)
    │
    ▼
Manual HTTP parsing
(read request line → method, path, headers, body)
    │
    ├── GET  /myapp/user → fetch all records from Derby → return HTML table
    ├── POST /myapp/user → insert record into Derby → redirect
    ├── POST /myapp/user?action=update → update record → redirect  
    └── GET  /myapp/user?action=delete&id=X → delete record → redirect
```

#### Key Concept

This replaces Tomcat's entire HTTP handling with a raw `ServerSocket` - Java's 
built-in TCP socket API. The server manually reads the HTTP request line by 
line, extracts the method, path and body, performs the DB operation, and writes 
a valid HTTP response back. No web server framework is involved at all.

| Aspects          | (Java + Derby)                |
| ---------------- | ----------------------------- |
| Language         | Java                          |
| Database         | Apache Derby (embedded)       |
| HTTP Handling    | Manual `ServerSocket` parsing |
| Setup complexity | Medium (manual HTTP parsing)  |
| Dependencies     | `derby.jar` only              |
| Tomcat used?     | No                            |
| Web server used? | No                            |

**We have also tried using a Go Server with SQLite, where Go provides a built-in lightweight HTTP Server, and SQLite being a lightweight, file-based, open-source RDBMS. It is also SQL based.**

## Question 5

All three systems (Q2, Q3, Q4) were tested using Apache JMeter with the following configuration:

| Parameter       | Value                          |
| --------------- | ------------------------------ |
| Tool            | Apache JMeter                  |
| Request Type    | HTTP POST                      |
| Target Endpoint | `/myapp/user` (CRUD list page) |
| Ramp-up Period  | 30sec                          |
| Error Threshold | Observed per system            |

### Results Summary

| Metric               | Q2 (Single VM, Tomcat) | Q3 (Nginx + Tomcat, 2 VMs) | Q4 (Custom Java Server) |
| :------------------- | ---------------------- | -------------------------- | ----------------------- |
| Total Samples        | 9,000                  | 9,000                      | 9,000                   |
| Avg Response Time    | 871 ms                 | 716 ms                     | 1,590 ms                |
| Median Response Time | 750 ms                 | 716 ms                     | 580 ms                  |
| 90th Percentile      | 1,578 ms               | 843 ms                     | 2,496 ms                |
| 95th Percentile      | 1,777 ms               | 934 ms                     | 4,459 ms                |
| 99th Percentile      | 2,672 ms               | 7,523 ms                   | 30,272 ms               |
| Min Response Time    | 6 ms                   | 6 ms                       | 10 ms                   |
| Max Response Time    | 14,665 ms              | 14,665 ms                  | 64,096 ms               |
| Throughput           | 0.285 req/sec          | 119.27 req/sec             | 83.83 req/sec           |
| Error Rate           | 0.000%                 | 0.000%                     | 0.733%                  |

### Response Time Graphs

**Image 1 - Q2 (Single VM, Tomcat)** Peak response time reached ~3,800 ms around 21:38:20, forming a clear bell curve over a ~2 minute window as load ramped up and then subsided. Start and end times show fast responses (~100 ms) when concurrency is low.

| Samples | Avg (ms) | Median (ms) | 90th % (ms) | 95th % (ms) | 99th % (ms) | Min (ms) | Max (ms) | Throughput (req/s) | Error % |
| ------- | -------- | ----------- | ----------- | ----------- | ----------- | -------- | -------- | ------------------ | ------- |
| 9,000   | 871      | 750         | 1,578       | 1,777       | 2,672       | 6        | 14,665   | 0.285              | 0.000%  |



**Image 2 - Q3 (Nginx + Tomcat split across 2 VMs)** Peak response time of ~1,220 ms reached around 00:27:20–00:27:30. Significantly flatter and lower curve compared to Q2. System handles load more gracefully with a gentler rise.

| Samples | Avg (ms) | Median (ms) | 90th % (ms) | 95th % (ms) | 99th % (ms) | Min (ms) | Max (ms) | Throughput (req/s) | Error % |
| ------- | -------- | ----------- | ----------- | ----------- | ----------- | -------- | -------- | ------------------ | ------- |
| 9,000   | 716      | 716         | 843         | 934         | 7,523       | 6        | 14,665   | 119.27             | 0.000%  |



**Image 3 - Q4 (Custom Server)** Curve shape is nearly identical to Q3, peaking at the same ~1,220 ms. However the raw aggregate data tells a different story - Q4's 99th percentile and max response time are dramatically higher, indicating occasional very slow responses that don't show up in the averaged graph.

| Samples | Avg (ms) | Median (ms) | 90th % (ms) | 95th % (ms) | 99th % (ms) | Min (ms) | Max (ms) | Throughput (req/s) | Error % |
| ------- | -------- | ----------- | ----------- | ----------- | ----------- | -------- | -------- | ------------------ | ------- |
| 9,000   | 1,590    | 580         | 2,496       | 4,459       | 30,272      | 10       | 64,096   | 83.83              | 0.733%  |



<img src="./Testing/Q2.png" alt="Q2" style="zoom:50%;" />

Fig: NGINX and Tomcat (Single Virtual Machine)

<img src="./Testing/Q3.jpeg" alt="Q3" style="zoom:50%;" />

Fig: NGINX and Tomcat (Dual Virtual Machine)

<img src="./Testing/Q4.jpeg" alt="Q4" style="zoom:50%;" />

Fig: Custom Java Server and Go Server

###  Analysis

**Q2 vs Q3 - Split architecture is faster on average**

Q3's average response time (716 ms) is about 18% lower than Q2 (871 ms), and its throughput is dramatically higher - 119 req/sec vs 0.285 req/sec. This is because in Q3, Nginx handles static file serving extremely efficiently without involving Java at all, freeing Tomcat on VM-2 to focus entirely on servlet execution. In Q2, Tomcat has to handle everything - static files, servlet logic, and DB operations - on a single VM, creating a bottleneck under load.

**Q3 vs Q4 - Custom server is less reliable**

While Q4's median response time (580 ms) is actually lower than Q3 (716 ms), meaning it handles light load quickly, its tail latencies are far worse. The 99th percentile for Q4 is 30,272 ms compared to Q3's 7,523 ms, and Q4 has a 0.733% error rate while Q3 has zero errors. Our custom Java `ServerSocket` server has no thread pooling, no request queue management, and no keep-alive handling that Tomcat provides out of the box. Under high concurrency, it begins dropping or delaying requests.

**Why Q4 median is deceptively low**

The median of 580 ms suggests that most requests complete quickly, but the high standard deviation (5,379 ms) and extreme max (64,096 ms) indicate the server handles easy requests well but struggles severely when concurrent load piles up. Tomcat's internal thread pool prevents this scenario - it queues excess requests rather than letting them time out.

### Key Findings

1.  **Nginx + Tomcat split (Q3) is the best performing architecture** - lower average latency, highest throughput, and zero errors.
2.  **Single VM Tomcat (Q2) is stable but slower** - handles all requests correctly with no errors, but response times climb faster under load since one machine handles everything.
3.  **Custom server (Q4) is the weakest under high concurrency** - while it works correctly for small loads (low median), it produces errors and extreme tail latencies at scale because it lacks Tomcat's thread management and request queuing infrastructure.
4.  **The bell-curve shape** visible in all three response time graphs is characteristic of JMeter's ramp-up behaviour - few users at the start, peak concurrency in the middle, and cooldown at the end. This is normal and expected.
5.  **Real-world implication** - For a production deployment, Q3's architecture (reverse proxy + dedicated app server) is the standard industry pattern precisely because of the performance advantage demonstrated here.

## Codes

**<u>index.html</u>**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>The United Tools</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&family=Playfair+Display:ital,wght@0,700;1,400&display=swap" rel="stylesheet">
<!-- AI Code start -->
</head>
<body>
  <header>
    <div class="site-title">The <em>United</em> Tools</div>
    <div class="site-tagline">three tools, one place</div>
  </header>
  <nav>
    <a href="index.html" class="active">Home</a>
    <a href="feature1.html">Stringify</a>
    <a href="feature2.html">Mathematize</a>
    <a href="feature3.html">Cruise in CRUD</a>
  </nav>
  <main>
    <div class="home-grid">
      <a href="feature1.html" class="tool-card glass">
        <span class="tool-card-icon">✦</span>
        <div class="tool-card-name">Stringify</div>
        <div class="tool-card-desc">Reverse, palindrome, anagram, word count, character frequency - all your string operations in one spot.</div>
        <span class="tool-card-arrow">Open tool →</span>
      </a>
      <a href="feature2.html" class="tool-card glass">
        <span class="tool-card-icon">◈</span>
        <div class="tool-card-name">Mathematize</div>
        <div class="tool-card-desc">Scientific calculator with sin, cos, tan, π, e, powers and full keyboard support including typed function names.</div>
        <span class="tool-card-arrow">Open tool →</span>
      </a>
      <a href="feature3.html" class="tool-card glass">
        <span class="tool-card-icon">⬡</span>
        <div class="tool-card-name">Cruise in CRUD</div>
        <div class="tool-card-desc">Create, read, update and delete records in your browser - a simple local data manager with live feedback.</div>
        <span class="tool-card-arrow">Open tool →</span>
      </a>
    </div>
  </main>
</body>
</html>
```

**<u>Feature1.html</u>**

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Stringify - The United Tools</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&family=Playfair+Display:ital,wght@0,700;1,400&display=swap" rel="stylesheet">
 <!-- AI Code start -->
</head>
<body>
  <header>
    <div class="site-title">The <em>United</em> Tools</div>
    <div class="site-tagline">three tools, one place</div>
  </header>
  <nav>
    <a href="index.html">Home</a>
    <a href="feature1.html" class="active">Stringify</a>
    <a href="feature2.html">Mathematize</a>
    <a href="feature3.html">Cruise in CRUD</a>
  </nav>
  <main>
    <div class="card glass">
      <div class="card-title">Stringify</div>
      <div class="field-group">
        <div>
          <label>String 1</label>
          <input type="text" id="str1" placeholder="Type your string here…">
        </div>
        <div>
          <label>String 2 <span style="font-size:10px;color:var(--text-dim);text-transform:none;letter-spacing:0">(for concat / compare / anagram)</span></label>
          <input type="text" id="str2" placeholder="Optional second string…">
        </div>
        <div>
          <label>Operation</label>
          <select id="option">
            <option value="length">Length</option>
            <option value="reverse">Reverse</option>
            <option value="palindrome">Check Palindrome</option>
            <option value="uppercase">Uppercase</option>
            <option value="lowercase">Lowercase</option>
            <option value="concatenate">Concatenate (str1 + str2)</option>
            <option value="initials">Get Initials</option>
            <option value="wordcount">Word Count</option>
            <option value="trim">Trim Whitespace</option>
            <option value="replace">Replace Vowels with *</option>
            <option value="charfreq">Character Frequency</option>
            <option value="compare">Compare Strings</option>
            <option value="contains">String 1 contains String 2?</option>
            <option value="anagram">Check Anagram</option>
          </select>
        </div>
        <div class="output-box" id="output">
          <span class="output-placeholder">Result will appear here…</span>
        </div>
        <button class="btn-run" id="btn">Run Operation</button>
      </div>
    </div>
  </main>
<script>
const input1 = document.getElementById('str1');
const input2 = document.getElementById('str2');
const output = document.getElementById('output');
const btn    = document.getElementById('btn');
const select = document.getElementById('option');
btn.addEventListener('click', () => {
  const s1 = input1.value;
  const s2 = input2.value;
  let result = '';
  switch (select.value) {
    case 'length':
      result = `Length of "${s1}" = ${s1.length}`;
      break;
    case 'reverse':
      result = `Reverse: ${s1.split('').reverse().join('')}`;
      break;
    case 'palindrome': {
      const rev = s1.split('').reverse().join('');
      result = `"${s1}" is ${s1.toLowerCase() === rev.toLowerCase() ? 'a Palindrome' : 'NOT a Palindrome'}`;
      break;
    }
    case 'uppercase':
      result = s1.toUpperCase();
      break;
    case 'lowercase':
      result = s1.toLowerCase();
      break;
    case 'concatenate':
      result = `"${s1}" + "${s2}" = "${s1 + s2}"`;
      break;
    case 'initials': {
      const initials = s1.trim().split(/\s+/).map(w => w[0]?.toUpperCase() || '').join('.');
      result = `Initials: ${initials}`;
      break;
    }
    case 'wordcount': {
      const words = s1.trim() === '' ? 0 : s1.trim().split(/\s+/).length;
      result = `Word count: ${words}`;
      break;
    }
    case 'trim':
      result = `Trimmed: "${s1.trim()}"`;
      break;
    case 'replace':
      result = `After replacing vowels: ${s1.replace(/[aeiouAEIOU]/g, '*')}`;
      break;
    case 'charfreq': {
      const freq = {};
      for (const ch of s1) freq[ch] = (freq[ch] || 0) + 1;
      result = Object.entries(freq).map(([k,v]) => `'${k}': ${v}`).join('  ');
      break;
    }
    case 'compare':
      result = s1 === s2 ? 'Strings are EQUAL' : 'Strings are NOT equal';
      break;
    case 'contains':
      result = s1.includes(s2) ? `"${s1}" CONTAINS "${s2}"` : `"${s1}" does NOT contain "${s2}"`;
      break;
    case 'anagram': {
      const sort = s => s.toLowerCase().replace(/\s/g,'').split('').sort().join('');
      result = sort(s1) === sort(s2) ? `"${s1}" and "${s2}" ARE anagrams` : 'NOT anagrams';
      break;
    }
    default:
      result = 'Unknown operation';
  }
  output.innerHTML = result;
  output.classList.add('has-result');
});
[input1, input2].forEach(el => el.addEventListener('keydown', e => {
  if (e.key === 'Enter') btn.click();
}));
</script>
</body>
</html>
```

**<u>Feature2.html</u>**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mathematize - The United Tools</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&family=DM+Mono:wght@300;400;500&family=Playfair+Display:ital,wght@0,700;1,400&display=swap" rel="stylesheet">
  <!-- AI Code start -->
</head>
<body>

  <header>
    <div class="site-title">The <em>United</em> Tools</div>
    <div class="site-tagline">three tools, one place</div>
  </header>

  <nav>
    <a href="index.html">Home</a>
    <a href="feature1.html">Stringify</a>
    <a href="feature2.html" class="active">Mathematize</a>
    <a href="feature3.html">Cruise in CRUD</a>
  </nav>

  <main>
    <div class="calc">
      <div class="display">
        <div class="display-history" id="history"></div>
        <div class="display-expr"   id="expr"></div>
        <div class="display-main"   id="main">0<span class="cursor"></span></div>
      </div>
      <div class="kb-hint">keyboard enabled - type <span>sin cos tan pi</span> directly</div>
      <div class="buttons" id="buttons">
        <button class="btn fn"    data-action="sin">sin</button>
        <button class="btn fn"    data-action="cos">cos</button>
        <button class="btn fn"    data-action="tan">tan</button>
        <button class="btn clear" data-action="clear">AC</button>

        <button class="btn fn"    data-action="pow">xⁿ</button>
        <button class="btn const" data-action="pi">π</button>
        <button class="btn const" data-action="e">e</button>
        <button class="btn op"    data-action="del">⌫</button>

        <button class="btn paren" data-action="(">(</button>
        <button class="btn paren" data-action=")">)</button>
        <button class="btn op"    data-action="%">%</button>
        <button class="btn op"    data-action="/">÷</button>

        <button class="btn"    data-action="7">7</button>
        <button class="btn"    data-action="8">8</button>
        <button class="btn"    data-action="9">9</button>
        <button class="btn op" data-action="*">×</button>

        <button class="btn"    data-action="4">4</button>
        <button class="btn"    data-action="5">5</button>
        <button class="btn"    data-action="6">6</button>
        <button class="btn op" data-action="-">−</button>

        <button class="btn"    data-action="1">1</button>
        <button class="btn"    data-action="2">2</button>
        <button class="btn"    data-action="3">3</button>
        <button class="btn op" data-action="+">+</button>

        <button class="btn zero" data-action="0">0</button>
        <button class="btn"      data-action=".">.</button>
        <button class="btn eq"   data-action="=">=</button>
      </div>
      <div class="typed-expr" id="typed-expr" style="display:none">
        <span class="label">input›</span><span class="val" id="typed-val"></span>
      </div>
    </div>
  </main>

<script>
let expr = '', result = null, justEvaled = false, rawInput = '';

const mainEl   = document.getElementById('main');
const exprEl   = document.getElementById('expr');
const histEl   = document.getElementById('history');
const typedBox = document.getElementById('typed-expr');
const typedVal = document.getElementById('typed-val');

function render(mode) {
  const cursor = '<span class="cursor"></span>';
  mainEl.className = 'display-main' + (mode === 'result' ? ' result' : mode === 'error' ? ' error' : '');
  if (mode === 'result') {
    mainEl.innerHTML = formatNum(result) + cursor;
    exprEl.textContent = expr;
  } else if (mode === 'error') {
    mainEl.innerHTML = 'Error' + cursor;
    exprEl.textContent = expr;
  } else {
    mainEl.innerHTML = (expr || '0') + cursor;
    exprEl.textContent = '';
  }
}

function formatNum(n) {
  if (typeof n !== 'number') return String(n);
  if (!isFinite(n)) return n > 0 ? '∞' : '-∞';
  if (Number.isNaN(n)) return 'NaN';
  return parseFloat(n.toPrecision(10)).toString();
}

function evaluate() {
  if (!expr.trim()) return;
  try {
    let js = expr
      .replace(/π/g, 'Math.PI').replace(/\be\b/g, 'Math.E')
      .replace(/sin\(/g, 'Math.sin(').replace(/cos\(/g, 'Math.cos(')
      .replace(/tan\(/g, 'Math.tan(').replace(/\^/g, '**')
      .replace(/÷/g, '/').replace(/×/g, '*').replace(/−/g, '-');
    let open = (js.match(/\(/g)||[]).length, close = (js.match(/\)/g)||[]).length;
    js += ')'.repeat(Math.max(0, open - close));
    let res = Function('"use strict"; return (' + js + ')')();
    histEl.textContent = expr + ' =';
    result = res; justEvaled = true;
    render('result');
  } catch(e) { render('error'); }
}

function append(str) {
  if (justEvaled) {
    const ops = ['+','-','*','/','%','^'];
    expr = ops.includes(str) ? formatNum(result) + str : str;
    justEvaled = false; result = null;
  } else { expr += str; }
  render();
}

function handleAction(action) {
  switch(action) {
    case 'clear': expr=''; result=null; justEvaled=false; rawInput=''; histEl.textContent=''; typedBox.style.display='none'; render(); break;
    case 'del':   if(justEvaled){expr='';justEvaled=false;render();break;} expr=expr.slice(0,-1); render(); break;
    case '=':     evaluate(); break;
    case 'sin':   append('sin('); break;
    case 'cos':   append('cos('); break;
    case 'tan':   append('tan('); break;
    case 'pow':   append('^'); break;
    case 'pi':    append('π'); break;
    case 'e':     append('e'); break;
    default:      append(action);
  }
}

document.getElementById('buttons').addEventListener('click', e => {
  const btn = e.target.closest('[data-action]');
  if (btn) handleAction(btn.dataset.action);
});

const keyMap = { 'Enter':'=', '=':'=', 'Escape':'clear', 'Backspace':'del', 'Delete':'del' };
const allowedChars = /^[0-9+\-*/().^%]$/;

document.addEventListener('keydown', e => {
  if (e.ctrlKey || e.metaKey || e.altKey) return;
  const k = e.key;
  if (keyMap[k]) { e.preventDefault(); handleAction(keyMap[k]); rawInput=''; typedBox.style.display='none'; return; }
  if (allowedChars.test(k)) { e.preventDefault(); handleAction(k); rawInput=''; typedBox.style.display='none'; return; }
  if (/^[a-zA-Z]$/.test(k)) {
    e.preventDefault();
    rawInput += k.toLowerCase();
    typedVal.textContent = rawInput;
    typedBox.style.display = 'block';
    if      (rawInput.endsWith('sin')) { append('sin('); rawInput=''; typedBox.style.display='none'; }
    else if (rawInput.endsWith('cos')) { append('cos('); rawInput=''; typedBox.style.display='none'; }
    else if (rawInput.endsWith('tan')) { append('tan('); rawInput=''; typedBox.style.display='none'; }
    else if (rawInput.endsWith('pi'))  { append('π'); rawInput=''; typedBox.style.display='none'; }
    else if (rawInput === 'e')         { append('e'); rawInput=''; typedBox.style.display='none'; }
    const keywords = ['sin','cos','tan','pi','e'];
    const possible = keywords.some(kw => kw.startsWith(rawInput));
    if (!possible && rawInput.length >= 3) { rawInput=''; typedBox.style.display='none'; }
  }
});

render();
</script>
</body>
</html>
```

**<u>Feature3.html</u>**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Feature 3</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&family=DM+Mono:wght@300;400&family=Playfair+Display:ital,wght@0,700;1,400&display=swap" rel="stylesheet">
<!-- AI Code start -->
</head>
<body>
    <header>
        <div class="site-title">The <em>United</em> Tools</div>
        <div class="site-tagline">three tools, one place</div>
    </header>
    <nav id="navbar">
        <a href="index.html">Home</a>
        <a href="feature1.html">Stringify</a>
        <a href="feature2.html">Mathematize</a>
        <a href="feature3.html" class="active">Cruise in CRUD</a>
    </nav>
    <div class="form-wrapper">
        <form action="/myapp/user" method="post">
            ID: <input name="id"><br>
            Name: <input name="name"><br>
            Email: <input name="email"><br>
            <div class="btn-row">
                <button name="action" value="create">Create</button>
                <button name="action" value="read">Read</button>
                <button name="action" value="update">Update</button>
                <button name="action" value="delete">Delete</button>
            </div>
        </form>
    </div>
</body>
</html>
```

**<u>Servlet.java</u>**

```java
import java.io.*;
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import jakarta.servlet.annotation.WebServlet;
import java.sql.*;

@WebServlet("/user")
public class servlet extends HttpServlet {

    private static final String DB_URL = "jdbc:derby:/opt/homebrew/opt/tomcat/libexec/webapps/myapp/db;create=true";
    @Override
    public void init() throws ServletException {
        try {
            Class.forName("org.apache.derby.jdbc.EmbeddedDriver");
        } catch (ClassNotFoundException e) {
            throw new ServletException("Derby driver not found. Add derby.jar to WEB-INF/lib.", e);
        }
        try (Connection conn = DriverManager.getConnection(DB_URL);
                Statement st = conn.createStatement()) {
            DatabaseMetaData meta = conn.getMetaData();
            try (ResultSet tables = meta.getTables(null, null, "USERS", null)) {
                if (!tables.next()) {
                    st.execute(
                            "CREATE TABLE users (" +
                                    "  id    INT          PRIMARY KEY, " +
                                    "  name  VARCHAR(100) NOT NULL, " +
                                    "  email VARCHAR(150) " +
                                    ")");
                } else {
                    // log("Table 'users' already exists.");
                }
            }

        } catch (SQLException e) {
            throw new ServletException("Failed to initialise database.", e);
        }
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse res)
            throws IOException {
        res.setContentType("text/html;charset=UTF-8");
        PrintWriter out = res.getWriter();
        String action = req.getParameter("action");
        String idStr = req.getParameter("id");
        String name = req.getParameter("name");
        String email = req.getParameter("email");
        if (action == null || action.isBlank()) {
            res.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            out.println("Missing parameter: action");
            return;
        }
        int id = -1;
        if (!"read".equals(action)) {
            if (idStr == null || idStr.isBlank()) {
                res.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                out.println("Missing parameter: id");
                return;
            }
            try {
                id = Integer.parseInt(idStr.trim());
            } catch (NumberFormatException e) {
                res.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                out.println("Invalid id - must be an integer.");
                return;
            }
        }
        if (("create".equals(action) || "update".equals(action))
                && (name == null || name.isBlank())) {
            res.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            out.println("Missing parameter: name");
            return;
        }
        try (Connection conn = DriverManager.getConnection(DB_URL)) {
            conn.setAutoCommit(false);
            try {
                switch (action) {
                    case "create" -> {
                        String sql = "INSERT INTO users (id, name, email) VALUES (?, ?, ?)";
                        try (PreparedStatement ps = conn.prepareStatement(sql)) {
                            ps.setInt(1, id);
                            ps.setString(2, name.trim());
                            ps.setString(3, email != null ? email.trim() : null);
                            ps.executeUpdate();
                        }
                        conn.commit();
                        out.println("User created (id=" + id + ")");
                    }
                    case "read" -> {
                        String sql = (idStr != null && !idStr.isBlank())
                                ? "SELECT * FROM users WHERE id = ?"
                                : "SELECT * FROM users ORDER BY id";

                        try (PreparedStatement ps = conn.prepareStatement(sql)) {
                            if (idStr != null && !idStr.isBlank()) {
                                ps.setInt(1, id);
                            }
                            try (ResultSet rs = ps.executeQuery()) {
                                boolean found = false;
                                while (rs.next()) {
                                    found = true;
                                    out.printf("ID: %d | Name: %s | Email: %s<br>%n",
                                            rs.getInt("id"),
                                            rs.getString("name"),
                                            rs.getString("email"));
                                }
                                if (!found)
                                    out.println("No users found.");
                            }
                        }
                        conn.commit();
                    }
                    case "update" -> {
                        String sql = "UPDATE users SET name = ?, email = ? WHERE id = ?";
                        try (PreparedStatement ps = conn.prepareStatement(sql)) {
                            ps.setString(1, name.trim());
                            ps.setString(2, email != null ? email.trim() : null);
                            ps.setInt(3, id);
                            int rows = ps.executeUpdate();
                            if (rows == 0) {
                                out.println("No user found with id=" + id);
                            } else {
                                out.println("User updated (id=" + id + ")");
                            }
                        }
                        conn.commit();
                    }
                    case "delete" -> {
                        String sql = "DELETE FROM users WHERE id = ?";
                        try (PreparedStatement ps = conn.prepareStatement(sql)) {
                            ps.setInt(1, id);
                            int rows = ps.executeUpdate();
                            if (rows == 0) {
                                out.println("No user found with id=" + id);
                            } else {
                                out.println("User deleted (id=" + id + ")");
                            }
                        }
                        conn.commit();
                    }

                    default -> {
                        res.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                        out.println("Unknown action: " + action);
                        conn.rollback();
                    }
                }
            } catch (SQLException e) {
                // Roll back the transaction on any SQL error
                try {
                    conn.rollback();
                } catch (SQLException rb) {
                    /* ignore rollback error */ }
                handleSQLError(e, out, res);
            }
        } catch (SQLException e) {
            res.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
            out.println("Could not connect to the database.");
        }
    }
    @Override
    public void destroy() {
        try {
            DriverManager.getConnection("jdbc:derby:;shutdown=true");
        } catch (SQLException e) {
            if (!"XJ015".equals(e.getSQLState())) {
            }
        }
    }
    private void handleSQLError(SQLException e, PrintWriter out, HttpServletResponse res)
            throws IOException {
        String state = e.getSQLState();
        if (state == null)
            state = "";

        String msg = switch (state) {
            case "23505" -> "Duplicate ID - a user with that id already exists.";
            case "23503" -> "Foreign key violation.";
            case "42X05" -> "Table 'users' not found. Try restarting the app.";
            case "42821" -> "Data type mismatch in query parameters.";
            default -> "Database error [" + state + "]: " + e.getMessage();
        };

        res.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
        out.println(msg);
        // log("SQLException: " + e.getMessage() + " | SQLState: " + state);
    }
}
```

**<u>javaHTTPServer.java</u>**
(We had to have ChatGPT write this code for us as given the time constraint.)

```java
import com.sun.net.httpserver.*;
import java.io.*;
import java.net.InetSocketAddress;
import java.net.URLDecoder;
import java.sql.*;

public class server {

    public static void main(String[] args) throws Exception {

        HttpServer server = HttpServer.create(new InetSocketAddress(8081), 0);

        server.createContext("/user", exchange -> {

    // Add CORS headers
    exchange.getResponseHeaders().add("Access-Control-Allow-Origin", "*");
    exchange.getResponseHeaders().add("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
    exchange.getResponseHeaders().add("Access-Control-Allow-Headers", "Content-Type");
    exchange.getResponseHeaders().add("Content-Type", "text/html; charset=UTF-8");

    // Handle OPTIONS request (CORS preflight)
    if ("OPTIONS".equals(exchange.getRequestMethod())) {
        exchange.sendResponseHeaders(204, -1);
        return;
    }

    String response = "";

    if ("GET".equals(exchange.getRequestMethod())) {
        response = "Server is running (GET request received)";
    }

    else if ("POST".equals(exchange.getRequestMethod())) {

        InputStream is = exchange.getRequestBody();
        BufferedReader br = new BufferedReader(new InputStreamReader(is));
        
        // Read complete request body
        StringBuilder sb = new StringBuilder();
        String line;
        while ((line = br.readLine()) != null) {
            sb.append(line);
        }
        String data = sb.toString();

        response = handleRequest(data);
    }

    exchange.sendResponseHeaders(200, response.length());
    OutputStream os = exchange.getResponseBody();
    os.write(response.getBytes());
    os.close();
});

        server.start();
        System.out.println("Server running on port 8081...");
    }

    static String handleRequest(String data) {

        String result = "";

        try {
            

            Connection conn = DriverManager.getConnection(
                "jdbc:derby:/root/myDB;");

            // parse form data
            String[] params = data.split("&");
            String action = "", id = "", name = "", email = "";

            for (String p : params) {
                String[] keyValue = p.split("=", 2);
                if (keyValue.length == 2) {
                    String key = keyValue[0];
                    String value = URLDecoder.decode(keyValue[1], "UTF-8");
                    
                    if ("action".equals(key)) action = value;
                    else if ("id".equals(key)) id = value;
                    else if ("name".equals(key)) name = value;
                    else if ("email".equals(key)) email = value;
                }
            }

            // CRUD logic (same as your servlet)

            if ("create".equals(action)) {
                PreparedStatement ps = conn.prepareStatement(
                    "insert into users values (?, ?, ?)");
                ps.setInt(1, Integer.parseInt(id));
                ps.setString(2, name);
                ps.setString(3, email);
                ps.executeUpdate();
                result = "User Inserted";
            }

            else if ("read".equals(action)) {
                Statement st = conn.createStatement();
                ResultSet rs = st.executeQuery("select * from users");

                while (rs.next()) {
                    result += rs.getInt(1) + " "
                           + rs.getString(2) + " "
                           + rs.getString(3) + "<br>";
                }
            }

            else if ("update".equals(action)) {
                PreparedStatement ps = conn.prepareStatement(
                    "update users set name=?, email=? where id=?");
                ps.setString(1, name);
                ps.setString(2, email);
                ps.setInt(3, Integer.parseInt(id));
                ps.executeUpdate();
                result = "User Updated";
            }

            else if ("delete".equals(action)) {
                PreparedStatement ps = conn.prepareStatement(
                    "delete from users where id=?");
                ps.setInt(1, Integer.parseInt(id));
                ps.executeUpdate();
                result = "User Deleted";
            }
				conn.close();} 
				catch (Exception e) {result = e.toString();}
        return result;
    }
}
```



## Use of AI Tools:

AI tools such as ChatGPT and Anthropic Claude were used during the experiment primarily for guidance on problem-solving approaches and for clarifying certain code syntax. The majority of the code was written independently by us, with AI tools assisting mainly in error detection and debugging.

A portion of the Java HTTP server code was generated using AI; however, it did not function as expected. Significant effort was invested on our part to make the code operational. Unfortunately, we do not have access to the AI-generated code used during that stage.
