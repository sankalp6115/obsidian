Go (usually called **Go**) is a compiled systems programming language created at **Google** in 2009 by **Robert Griesemer**, **Rob Pike**, and **Ken Thompson** (the same Ken Thompson who helped create Unix).

So they built Go with these priorities:

- very **fast compilation**
- built-in **concurrency**
- **simple syntax**
- **static typing**
- **single binary deployment**

Some famous tools written in Go:

- **Docker**
- **Kubernetes**
- **Terraform**
- **Prometheus**
- **Grafana**

Typical Go use cases:

**1. Backend APIs**

- REST services
- microservices
- authentication services

**2. Cloud infrastructure**

- distributed systems
- container runtimes
- orchestration tools

**3. Dev tools**

- CLI tools
- compilers
- build systems

**4. Networking software**

- proxies
- gateways
- load balancers


Why Companies Like Go
### 1. Concurrency (Goroutines)

Go has a lightweight thread model called **goroutines**.

A goroutine is basically a tiny thread that costs almost nothing.

### 2. Extremely Simple Language

The entire language specification is small. You can realistically learn most of it in a few days.
Examples:

- no inheritance
- no complicated generics (until recently)
- no macros
- very small syntax surface

This simplicity is deliberate. The creators wanted codebases where every engineer can read every file.

### 3. Single Binary Deployment

When you compile Go:

```
go build
```

You get **one executable file**.

No runtime dependencies. No JVM. No Python interpreter.

You can literally copy the binary to a server and run it.


## Downsides

The language also has tradeoffs.

**1. Minimalistic to a fault**

Some developers think it is _too simple_.  
Certain abstractions are intentionally missing.

**2. Error handling style**

Instead of exceptions:

```
if err != nil {    return err}
```

You see this pattern everywhere.

**3. Less expressive than some languages**

Compared with languages like **Rust** or **C++**, Go sacrifices power for clarity.



Golang: The language itself was written in Go.
It refers to how a programming language is **implemented**.

Every programming language needs a **compiler or interpreter**. That program reads source code and turns it into machine code the computer can run.

When someone says:

> “The language itself was written in Go”

they mean **the Go compiler is written in Go**.

The main compiler is **Go Compiler (cmd/compile)**, and it is part of the **Go Toolchain**.

### The Bootstrapping Idea

Here is the interesting part.

You cannot initially write a compiler in the language that does not exist yet. So languages usually go through this process:

1. **First compiler written in another language**
2. That compiler can compile programs in the new language
3. Eventually the compiler is rewritten in the new language itself

This process is called **bootstrapping**.

Go followed this path:

1. Early Go compiler was written in **C**
2. Once Go became stable, the compiler was **rewritten in Go**
3. Now Go compiles Go

So when you run:

```
go build program.go
```

You are running a Go program that compiles another Go program.

Examples of languages that compile themselves:

- **Go**
- **Rust**
- **C**
- **Java**

## Projects
https://chatgpt.com/c/69b83de7-d978-8322-8986-1a9fb85cb1bb?mweb_fallback=1
https://chatgpt.com/share/6a3fed57-cfe4-83ee-846f-c00b286b17fc