# From Code to Server: How I Deployed a Full Website on a Virtual Machine

*What it actually takes to go from "it works on my laptop" to "it works in the real world."*

------

The brief looked simple enough: **build a web application with string operations, a math calculator, and database CRUD functionality, then deploy it.**

I nodded. Sure. I've written HTML. I've done Java. How hard could it be?

Embarrassingly hard, as it turned out - though not where I expected. The actual features weren't the problem. The real nightmare was everything *around* them: servers, virtual machines, network configs and the slow, humbling process of learning how computers actually talk to each other.

This is that story - honest, messy, and hopefully useful to whoever's about to make the same mistakes I did.

------

## What I Was Building

The app had three features, each one teaching me something different.

### Feature 1: String Operations

Take a string, do things to it — reverse it, check for palindromes, count vowels, uppercase it. Classic stuff.

The whole thing ran client-side. JavaScript handles it in the browser, result appears instantly, no server involved. Nothing leaves the user's machine.

This was my first real lesson: *not everything needs a backend.* If the logic is simple and the data stays local, dragging a server into it just adds complexity for no reason.

### Feature 2: Mathematical Calculator

Addition, subtraction, multiplication, division, trig functions. Also client-side — `Math.sin()` works perfectly well in a browser without asking a server for help.

At this point I had two features that just... worked. JavaScript ran, page responded, life was good. I was feeling pretty confident.

Then came Feature 3.

### Feature 3: CRUD with a Database

Create, Read, Update, Delete. The moment you need data to *persist* — to survive a page refresh, to be there tomorrow on someone else's device — JavaScript alone can't save you. You need a server. You need a database. You need a backend.

I went with **Apache Derby** in embedded mode, which means the database runs inside the same Java process as the app rather than as a separate server. No MySQL setup, no connection strings pointing at remote machines. For a learning project where you're already drowning in new concepts, that convenience matters.

## Serving the Application: Apache Tomcat

With the code written, I needed something to actually *run* it and make it reachable from a browser. That's where **Apache Tomcat** comes in.

Tomcat does three things at once: listens for HTTP requests on port 8080, manages Java "servlets" (the classes that handle specific URLs), and bridges your frontend and backend together. When your browser asks for an HTML file, Tomcat sends it. When your browser submits a form, Tomcat hands it off to the right servlet.

The file structure it expects is fairly rigid:

```
tomcat/
 └── webapps/
      └── myapp/
           ├── index.html
           ├── feature1.html
           ├── feature2.html
           ├── feature3.html
           └── WEB-INF/
                ├── classes/       ← compiled .class files
                └── lib/           ← .jar files (Derby's driver lives here)
```

Run the startup script and your app appears at `http://localhost:8080/myapp`. The first time that URL actually works, it hits differently — something you *wrote* is being served through a real HTTP server. It feels more real than opening a file in a browser. A small thing that somehow felt significant.

### What Actually Happens When You Click Submit

For string operations and the calculator, the flow is trivially simple: browser asks for the HTML, Tomcat sends it, JavaScript runs, done.

For CRUD, it's a different story:

1.  User fills a form and clicks Submit
2.  Browser sends an HTTP POST to something like `/myapp/users`
3.  Tomcat figures out which servlet handles that URL
4.  Servlet runs, talks to Derby via JDBC
5.  Derby executes the SQL
6.  Servlet builds a response
7.  Tomcat sends it back
8.  Browser shows the result

Every single one of those steps can fail independently. I know, because most of them did.

------

## Moving to a Virtual Machine

Working on your own machine is comfortable in a deceptive way. Java is installed. The paths are right. The right version of everything is there because *you* put it there.

Real deployment strips all that away. The server you deploy to was configured by someone else, probably runs an unfamiliar Linux distribution, and is missing half the tools you assumed would be there. A VM is how you practice for that.

I used **Alpine Linux** — deliberately minimal. It ships with almost nothing. No Java, no Tomcat, no development tools. You install everything explicitly, which forces you to actually understand what your application depends on rather than benefiting from whatever happened to be pre-installed on your laptop.

### Why the Discomfort Is the Point

The first time I logged into the Alpine VM, I was immediately lost. File system looked wrong. Commands I relied on weren't there. I spent an embarrassing amount of time figuring out how to exit a text editor.

But that discomfort is the education. It strips away assumptions and makes you think deliberately about what you're actually doing.

Setting up the environment meant creating the VM in VirtualBox, going through Alpine's minimal installer, enabling package repositories, and installing Java, Tomcat, and SSH. Once SSH was running, I could control the VM from my regular terminal on the host machine, which made everything significantly less painful. The `scp` command alone — being able to copy files directly from my laptop into the VM — saved a huge amount of time.

### The Networking Problem That Stopped Everything

Here's something tutorials often gloss over: by default, VirtualBox uses **NAT networking**. In NAT mode, the VM can reach the internet just fine — it can download packages, ping servers — but the traffic is one-directional. Other devices on your network cannot reach the VM. It's invisible from the outside.

I discovered this the hard way when I tried to access the app from my phone for a demo. Nothing. The VM was running, Tomcat was running, but the app was completely unreachable.

The fix was switching to **Bridged networking**, where the VM behaves like a real, independent device on your local network, gets its own IP from your router, and can be reached by anything else on the network.

After that change, I opened the app on my phone over WiFi. Simple thing, but it felt like breaking through a wall.

------

## Splitting Across Two Machines

The next phase was the one that made real web infrastructure suddenly make sense: splitting the system across two VMs.

**VM 1** runs **Nginx** — a fast, efficient web server, particularly good with static content. It serves HTML, CSS, and JS directly, and acts as a **reverse proxy** that forwards other requests to the backend.

**VM 2** runs **Apache Tomcat** — all the Java servlet logic and database work happens here.

### What a Reverse Proxy Actually Does

This concept confused me for a while. Here's how it actually works:

The user talks to Nginx on VM 1. They don't know VM 2 exists. They don't know Tomcat exists. From their perspective, there's one server.

Nginx looks at each request and makes a call: is this a static file? Serve it straight from disk. Is this a request for `/myapp/users`? Forward it to Tomcat on VM 2, wait for the response, send it back.

The full journey of one request:

```
User's Browser
    ↓  HTTP Request
   Nginx (VM 1)
    ↓  Proxied Request
  Tomcat (VM 2)
    ↓  JDBC Query
   Derby Database
    ↑  Query Result
  Tomcat (VM 2)
    ↑  HTTP Response
   Nginx (VM 1)
    ↑  Forwarded Response
User's Browser
```

### Why Bother With Two Machines?

It sounds like unnecessary complexity. Two machines, more networking, more things to break. But there are real reasons:

**Performance.** Nginx handles static files extremely efficiently — thousands of concurrent requests without flinching. Keeping static content on Nginx means Tomcat only sees requests that actually need backend work.

**Isolation.** If the database has a meltdown, the web server can still serve the homepage. Things fail independently rather than all at once.

**Scalability.** If Tomcat becomes a bottleneck, you can add more Tomcat instances and have Nginx distribute requests across them. The frontend doesn't change at all.

**Security.** VM 2 doesn't need to face the internet. Only VM 1 does. The backend lives privately, which reduces how much of your system is exposed.

------

## Removing Tomcat: Constraint-Based Design

Then came the constraint that forced the most interesting thinking: **you cannot use Apache Tomcat on the backend.**

My first reaction was annoyance. Why break something that's working? But constraints like this are useful precisely because they force you to understand what a tool was actually doing for you.

Tomcat was doing a lot. It was listening on a port, parsing HTTP requests, routing them to the right handler, managing database connections, serializing responses. Without it, all of that becomes your problem.

### Option 1: Java's Built-in HTTP Server

Java has a built-in HTTP server tucked away in `com.sun.net.httpserver`. It's minimal — not for production traffic — but it works:

java

```java
HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
server.createContext("/myapp/users", new UserHandler());
server.start();
```

You get an `HttpExchange` object, you write your response to it. Parsing request bodies, routing GET vs POST, setting headers — all your responsibility now. Tedious, but I understood HTTP far better by the end of it than I ever would have from reading about it.

### Option 2: Go with SQLite

The second approach used **Go** — a language Google built for exactly this kind of systems work — with **SQLite** as the database.

Go's HTTP handling is genuinely pleasant:

go

```go
http.HandleFunc("/myapp/users", handleUsers)
http.ListenAndServe(":8080", nil)
```

Go also handles concurrency without ceremony — every request automatically runs in its own goroutine, so the server handles multiple requests simultaneously without you writing any threading code.

SQLite is Derby's more battle-tested cousin. It's in mobile apps, web browsers, aerospace software. For a small application, it's more than enough.

The Go + SQLite implementation ended up cleaner and noticeably faster. Go's standard library is just better suited to this kind of work than Java's built-in server.

------

## Performance Testing with JMeter

Apache JMeter simulates many users hitting a server at once. You set a target, tell it to fire 200 requests per second, and watch how the server holds up — response times, failure rates, how badly things degrade under pressure.

I tested three setups:

**Single VM with Tomcat** — Everything on one machine. Held up fine under moderate load. Under heavier load, response times climbed as Tomcat started queuing requests it couldn't handle fast enough.

**Dual VM (Nginx + Tomcat)** — Best overall. Separating static file serving from backend processing meant the two components weren't competing for the same machine's resources. Nginx blew through static requests, Tomcat focused entirely on database work.

**Custom Server (Java or Go)** — The custom Java server struggled under heavy load — handling concurrency properly requires more careful tuning than the built-in server makes obvious. Go performed better, mostly because goroutines are lighter than Java threads.

The conclusion: architecture matters more than code-level optimization. The same logic, organized differently, produced genuinely different results. That's not an abstract principle — I watched it happen in the numbers.

------

## The Problems That Ate My Time

A quick honest account of what actually went wrong, because tutorials never include this part.

**A commented-out repository.** At some point `apk add` started silently doing nothing. After hours of confusion I found the problem: a `#` at the start of a line in `/etc/apk/repositories` was disabling the package source. One character. Multiple hours.

**Derby's classpath drama.** Derby ships multiple `.jar` files for different purposes. Using the wrong one produces errors that don't tell you what's actually wrong. Getting the classpath right meant reading the documentation carefully and a fair amount of trial and error.

**The changing IP address.** After setting up bridged networking, I hardcoded the VM's IP into Nginx's config. A few days later the proxy stopped working — the VM had gotten a new IP from the DHCP server. In production you use static IPs or hostnames. Lesson learned the annoying way.

**Port conflicts.** Tomcat wouldn't start because something else was already on port 8080. Finding what it was (`netstat -tulpn | grep 8080`), deciding whether to stop it or change Tomcat's port, then updating Nginx — each step needed a new command I didn't know yet.

**Servlet 404s with no useful information.** When Tomcat can't find a servlet, it returns 404. But that could mean the URL is wrong, the class didn't compile, the class is in the wrong directory, or `web.xml` has a typo. Distinguishing between them required checking every layer methodically.

------

## What I Actually Learned

Looking back, the code is almost the least interesting part of this project. What I actually learned was about systems:

**Local and production are different environments.** Software that works on your laptop doesn't automatically work on a server. Dependencies, paths, permissions, network config — all of it can differ, all of it can break you.

**Servers are just programs listening on ports.** The mystical "server" is a process sitting on a machine waiting for connections. Understanding that made everything else less mysterious.

**Network access has to be explicitly configured.** Firewalls, NAT, port bindings — none of this is permissive by default, for good reason. Getting something accessible from the network requires intention, not just running a process.

**Architecture affects performance more than optimization.** Separation of concerns, reducing resource contention, handling static files efficiently — these architectural choices moved the needle more than any amount of function-level tuning.

**Debugging distributed systems means thinking in layers.** When something breaks across multiple machines, the problem could be anywhere — the browser, the proxy, the app server, the database, or any network hop between them. You have to work through each layer systematically, not guess.

**Constraints produce understanding.** Removing Tomcat was frustrating. Rebuilding what it did taught me more about web servers than any amount of reading would have.

------

## Final Thoughts

This started as a web development assignment and ended as a systems engineering education.

There's a comfortable illusion in early programming courses that writing code is the hard part, and deployment is just administrative overhead. This project broke that illusion completely.

The real world doesn't care if your function logic is correct when the server can't find the class file. It doesn't care if your SQL is efficient when the database connection times out. It doesn't care if your JavaScript is elegant when the VM has no external IP.

Software development, at the level that actually matters, is about understanding how pieces connect, how they communicate, and how they fail. And that only comes from sitting with real machines and real errors until something finally clicks.

The best moment in this whole project wasn't when the CRUD operations worked. It was when I pulled up the app on my phone — over the network, through a proxy, on a virtual machine I had built from scratch — and it loaded. That felt like crossing a line.

From writing code to building systems. That's what this was actually about.