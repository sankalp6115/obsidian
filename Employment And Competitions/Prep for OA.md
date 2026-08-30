## Keys

- **Candidate key**: minimal set of attributes that uniquely identifies a row. A table can have multiple candidate keys.
- **Primary key**: one candidate key chosen to be _the_ identifier. Implies NOT NULL + UNIQUE automatically.
- **Composite key**: a primary/candidate key made of 2+ columns together (neither column alone is unique). Common gotcha: a composite key's _individual_ columns can have duplicate values — only the _combination_ must be unique.
- **Foreign key**: references a primary/unique key in another (or same) table. Can be NULL (unless also NOT NULL constrained). Enforces referential integrity.
- **Super key** (often forgotten): any set of attributes that uniquely identifies a row — candidate keys are the _minimal_ super keys. Trick question bait: "Is every super key a candidate key?" → No. "Is every candidate key a super key?" → Yes.

**Gotcha:** Self-referencing foreign key (e.g., `employees.manager_id → employees.employee_id`) — know this pattern, it's a classic OA question (org chart / hierarchy).

## Normalization

Goal: reduce redundancy, avoid update/insert/delete anomalies.

|Form|Rule|Fixes|
|---|---|---|
|1NF|Atomic values only, no repeating groups/arrays in a column|Multi-valued columns|
|2NF|1NF + no **partial dependency** (non-key attr depends on only _part_ of a composite PK)|Only relevant when PK is composite|
|3NF|2NF + no **transitive dependency** (non-key attr depends on another non-key attr, not directly on PK)|A → B → C chains|
|BCNF (bonus)|Every determinant must be a candidate key|Stricter version of 3NF|

**Classic OA trap:** "Table has a single-column PK — is it automatically in 2NF?" → **Yes**, because partial dependency only applies to composite keys. If PK isn't composite, 1NF → automatically satisfies 2NF.

**Example to keep in your head:**

```
Table: StudentCourse(StudentID, CourseID, StudentName, CourseName, Grade)
PK = (StudentID, CourseID)
```

- `StudentName` depends only on `StudentID` → partial dependency → violates 2NF.
- `CourseName` depends only on `CourseID` → also partial dependency.
- Fix: split into `Student(StudentID, StudentName)`, `Course(CourseID, CourseName)`, `Enrollment(StudentID, CourseID, Grade)`.

**Transitive dependency example (3NF violation):**

```
Table: Employee(EmpID, DeptID, DeptName)
```

`DeptName` depends on `DeptID`, which depends on `EmpID` → transitive → violates 3NF. Fix: separate `Department(DeptID, DeptName)`.

## ACID

- **Atomicity**: all-or-nothing (transaction fully completes or fully rolls back).
- **Consistency**: DB moves from one valid state to another (constraints, triggers, cascades hold).
- **Isolation**: concurrent transactions don't interfere — governed by isolation levels.
- **Durability**: once committed, survives crashes (write-ahead logging, etc.)

**Isolation levels (frequently asked, know the anomaly each one prevents):**

|Level|Dirty Read|Non-repeatable Read|Phantom Read|
|---|---|---|---|
|Read Uncommitted|✗ possible|✗ possible|✗ possible|
|Read Committed|✓ prevented|✗ possible|✗ possible|
|Repeatable Read|✓ prevented|✓ prevented|✗ possible|
|Serializable|✓ prevented|✓ prevented|✓ prevented|

Quick definitions:

- **Dirty read**: reading uncommitted data from another transaction.
- **Non-repeatable read**: re-reading the same row gives a different value because another transaction updated + committed it in between.
- **Phantom read**: re-running the same query returns a _different set of rows_ (new row inserted by another transaction matching your WHERE clause).

## Transactions

```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT; -- or ROLLBACK on failure
```

Know **SAVEPOINT** conceptually: lets you roll back part of a transaction without rolling back everything.

## Indexing

- Speeds up reads (WHERE, JOIN, ORDER BY) at the cost of slower writes (INSERT/UPDATE/DELETE need to update the index too) and extra storage.
- **Clustered index**: determines physical row order on disk — a table can have only **one** (usually the PK).
- **Non-clustered index**: separate structure with pointers back to actual rows — a table can have **many**.
- Underlying structure is usually a **B+ Tree** (know this term, it comes up).
- **Gotcha:** indexing every column isn't free — it slows down writes and can even confuse the query optimizer. Also, a composite index on `(A, B)` speeds up queries filtering on `A` or `(A,B)`, but generally **not** on `B` alone (leftmost prefix rule) — this is a favorite trick question.

## Joins

```sql
-- INNER JOIN: only matching rows
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;

-- LEFT JOIN: all left rows, NULLs for unmatched right
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;

-- RIGHT JOIN: all right rows, NULLs for unmatched left
-- FULL OUTER JOIN: all rows from both, NULLs where no match (not supported in MySQL directly — emulate with UNION of LEFT + RIGHT)

-- SELF JOIN: table joined with itself
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.employee_id;

-- CROSS JOIN: cartesian product (every row × every row)
SELECT a.x, b.y FROM table_a a CROSS JOIN table_b b;
```

**Gotcha commonly tested:** find employees with NO department (unmatched rows):

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;
```

This "LEFT JOIN + IS NULL" pattern is extremely common in OAs — memorize it.

## GROUP BY / HAVING / ORDER BY

**Execution order matters** (this is asked a lot as "what's wrong with this query"):

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

- `WHERE` filters rows **before** grouping (can't use aggregate functions in WHERE).
- `HAVING` filters groups **after** aggregation (used with aggregate functions).

```sql
-- Departments with more than 5 employees earning above 50000
SELECT dept_id, COUNT(*) AS emp_count, AVG(salary) AS avg_sal
FROM employees
WHERE salary > 50000
GROUP BY dept_id
HAVING COUNT(*) > 5
ORDER BY avg_sal DESC;
```

**Classic gotcha:** `SELECT dept_id, name, COUNT(*) FROM employees GROUP BY dept_id;` — this is often _invalid_ (or gives unpredictable results in lenient engines) because `name` isn't in GROUP BY and isn't aggregated. Every non-aggregated column in SELECT must appear in GROUP BY.

## Subqueries

```sql
-- Scalar subquery
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- IN subquery
SELECT name FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE location = 'NY');

-- Correlated subquery (re-runs per outer row — know this term!)
SELECT name, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary) FROM employees e2 WHERE e2.dept_id = e1.dept_id
);

-- EXISTS (often more efficient than IN for large subquery results)
SELECT name FROM employees e
WHERE EXISTS (SELECT 1 FROM projects p WHERE p.emp_id = e.emp_id);
```

**Gotcha:** `IN` vs `NOT IN` with NULLs — if the subquery returns even one NULL, `NOT IN` silently returns **zero rows** (a well-known SQL trap). `NOT EXISTS` doesn't have this issue — prefer it when NULLs might be present.

**Nth highest salary** (very commonly asked, memorize a version):

```sql
SELECT DISTINCT salary
FROM employees e1
WHERE N - 1 = (
    SELECT COUNT(DISTINCT salary) FROM employees e2 WHERE e2.salary > e1.salary
);

-- or simpler, using LIMIT/OFFSET (MySQL/Postgres)
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET N-1;
```

## DELETE vs TRUNCATE vs DROP

||DELETE|TRUNCATE|DROP|
|---|---|---|---|
|Removes|Rows (with WHERE optional)|All rows|Entire table structure + data|
|WHERE clause|✅ Yes|❌ No|❌ No|
|Rollback|✅ Yes (DML, logged row-by-row)|⚠️ Depends on DB (usually can't rollback / auto-commits in MySQL, can in some Postgres versions inside a transaction)|❌ No (DDL)|
|Resets AUTO_INCREMENT|❌ No|✅ Yes|N/A (table gone)|
|Triggers fired|✅ Yes|❌ No|❌ No|
|Speed|Slower (logs each row)|Faster (deallocates pages)|Fastest|
|Category|DML|DDL (in most DBs)|DDL|

**Gotcha:** TRUNCATE is technically DDL in most systems (even though it "feels" like a data operation) — that's why it can't be selectively rolled back the way DELETE can, and why it skips triggers.



Let's go through these the same way — fast, but hitting the intermediate stuff that actually gets asked.

## OSI Model (7 layers)

Memorize top-to-bottom or bottom-to-top with a mnemonic. Bottom-up: **"Please Do Not Throw Sausage Pizza Away"**

|Layer|Name|What it does|Examples|
|---|---|---|---|
|7|Application|User-facing protocols|HTTP, FTP, SMTP, DNS|
|6|Presentation|Encryption, compression, translation (data format)|SSL/TLS, JPEG, ASCII|
|5|Session|Establishes/manages/terminates sessions|APIs, sockets|
|4|Transport|End-to-end delivery, reliability|TCP, UDP|
|3|Network|Routing, logical addressing|IP, ICMP, routers|
|2|Data Link|Physical addressing, frames|Ethernet, MAC address, switches|
|1|Physical|Raw bits over a medium|Cables, radio waves|

**Gotcha commonly asked:** "Which layer does a switch operate at?" → Layer 2 (uses MAC addresses). "Which layer does a router operate at?" → Layer 3 (uses IP addresses). This switch-vs-router-layer question comes up a LOT.

**Another common one:** "Where does a firewall operate?" → Can be Layer 3/4 (packet filtering) or Layer 7 (application firewalls) — mention both if asked.

## TCP/IP Model (4 layers) — simplified OSI

|TCP/IP Layer|Maps to OSI layers|
|---|---|
|Application|Application + Presentation + Session (5,6,7)|
|Transport|Transport (4)|
|Internet|Network (3)|
|Network Access (Link)|Data Link + Physical (1,2)|

**Why asked:** OSI is theoretical/teaching model, TCP/IP is what's actually implemented in real networks. Know this distinction — it's a common one-liner question.

## TCP vs UDP

||TCP|UDP|
|---|---|---|
|Connection|Connection-oriented (handshake first)|Connectionless|
|Reliability|Guaranteed delivery, retransmits lost packets|No guarantee, "fire and forget"|
|Ordering|Packets arrive in order|No ordering guarantee|
|Speed|Slower (overhead of ack, retransmit, flow control)|Faster (minimal overhead)|
|Header size|20 bytes|8 bytes|
|Use cases|Web (HTTP), email, file transfer — anything needing accuracy|Video streaming, gaming, DNS, VoIP — anything needing speed over accuracy|

**Gotcha:** "Why would DNS use UDP if UDP is unreliable?" → DNS queries are small, single request-response, and the overhead of TCP's handshake isn't worth it for a quick lookup — if a packet is lost, the client just retries the whole query, which is still cheaper than TCP overhead. (Note: DNS _does_ use TCP for zone transfers or responses >512 bytes — good bonus fact if asked.)

**Another gotcha:** "Does TCP guarantee delivery or just reliable delivery if the connection stays up?" → TCP guarantees reliable, ordered delivery **as long as the connection doesn't fail catastrophically** — it handles packet loss via retransmission using ACKs and sequence numbers.

## TCP 3-Way Handshake

Establishes a connection before data transfer:

```
Client                          Server
  |------ SYN (seq=x) --------->|     "I want to connect, my starting seq is x"
  |<--- SYN-ACK (seq=y,ack=x+1)-|     "OK, here's my seq y, I acknowledge yours"
  |------ ACK (ack=y+1) ------->|     "Got it, let's talk"
```

- **SYN**: synchronize — client initiates, sends initial sequence number.
- **SYN-ACK**: server acknowledges client's SYN and sends its own SYN.
- **ACK**: client acknowledges server's SYN. Connection established.

**Gotcha:** Why 3 steps and not 2? → Because it needs to be **bidirectional** — both sides need to confirm they can send _and_ receive. A 2-way handshake would let one side assume the connection works without confirming the reverse direction.

**Bonus (sometimes asked):** connection _termination_ uses a **4-way handshake** (FIN, ACK, FIN, ACK) — because when the server gets a FIN, it might still have data to finish sending, so it ACKs the FIN separately from sending its own FIN.

## IP Addresses

- **IPv4**: 32-bit, written as 4 octets (`192.168.1.1`), ~4.3 billion addresses — exhausted, hence IPv6.
- **IPv6**: 128-bit, written in hex groups (`2001:0db8::1`), effectively unlimited addresses.
- **Public vs Private IP**: Private ranges (used inside LANs, not routable on the internet):
    - `10.0.0.0 – 10.255.255.255`
    - `172.16.0.0 – 172.31.255.255`
    - `192.168.0.0 – 192.168.255.255`
- **NAT (Network Address Translation)**: lets multiple devices on a private network share one public IP — good one-liner to know why private IPs even work on the internet.
- **Subnet mask / CIDR notation**: `192.168.1.0/24` means the first 24 bits are the network portion, remaining 8 bits are for hosts (256 addresses, 254 usable). If asked to compute usable hosts: `2^(32-CIDR) - 2` (minus network address and broadcast address).

**Gotcha:** "Why subtract 2?" → One address is reserved for the network identifier (all host bits = 0), one for broadcast (all host bits = 1) — neither can be assigned to an actual device.

## DNS (Domain Name System)

Translates human-readable domain names → IP addresses. Think of it as the internet's phonebook.

**Resolution flow (asked often, know the order):**

```
Browser cache → OS cache → Router cache → ISP's DNS resolver
   → Root DNS server → TLD server (.com, .org) → Authoritative DNS server → IP returned
```

- **Root server**: knows where TLD servers are.
- **TLD server**: knows where authoritative servers for `.com`/`.org`/etc are.
- **Authoritative server**: has the actual DNS record for the specific domain.

**Common record types to know:**

- `A` — maps domain → IPv4 address
- `AAAA` — maps domain → IPv6 address
- `CNAME` — alias, points one domain to another domain
- `MX` — mail server records
- `NS` — nameserver records

## HTTP vs HTTPS

- **HTTP**: plaintext, port 80. Anyone on the network path can read/intercept data.
- **HTTPS**: HTTP + **TLS/SSL encryption**, port 443. Data is encrypted in transit.

**Gotcha — TLS handshake basics (sometimes asked at a high level):**

1. Client says hello (supported ciphers).
2. Server responds with its **certificate** (contains its public key).
3. Client verifies certificate against a trusted Certificate Authority (CA).
4. Symmetric session key gets negotiated (via asymmetric encryption initially — public/private key pair — then switches to symmetric encryption for actual data because it's faster).

**One-liner worth knowing:** HTTPS uses **asymmetric encryption to establish the connection**, then **symmetric encryption for the actual data transfer** (asymmetric is secure but slow, symmetric is fast — best of both).

## HTTP Methods

|Method|Purpose|Idempotent?|Has body?|
|---|---|---|---|
|GET|Retrieve data|✅ Yes|❌ No|
|POST|Create new resource|❌ No|✅ Yes|
|PUT|Update/replace resource entirely|✅ Yes|✅ Yes|
|PATCH|Partially update resource|❌ No (technically, though often treated as idempotent)|✅ Yes|
|DELETE|Remove resource|✅ Yes|Usually no|

**"Idempotent" is the word interviewers love to test** — it means calling it multiple times has the **same effect as calling it once**.

- `DELETE /user/5` called twice → user 5 still deleted either way → idempotent.
- `POST /user` called twice → creates **two** users → NOT idempotent.
- `PUT /user/5 {name: "Alex"}` called 10 times → same end result → idempotent.

**Gotcha:** "Is GET idempotent?" → Yes, and also **safe** (doesn't modify server state at all) — safe is a stronger guarantee than idempotent.

## Status Codes

|Range|Category|Common ones|
|---|---|---|
|1xx|Informational|100 Continue|
|2xx|Success|200 OK, 201 Created, 204 No Content|
|3xx|Redirection|301 Moved Permanently, 302 Found (temp), 304 Not Modified|
|4xx|Client error|400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict|
|5xx|Server error|500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable|

**Gotcha frequently asked:** difference between **401 vs 403**:

- **401 Unauthorized** → you're not authenticated (no/invalid credentials — "who are you?")
- **403 Forbidden** → you're authenticated but don't have permission ("I know who you are, you're just not allowed")

**Another one:** **301 vs 302**:

- **301** = permanent redirect (browsers/search engines cache this, update bookmarks)
- **302** = temporary redirect (don't cache, might change back)

## Cookies vs Sessions

- **Cookie**: small key-value data stored **client-side** (in the browser), sent with every request to that domain. Has attributes like expiry, `HttpOnly` (can't be accessed via JS, protects against XSS), `Secure` (only sent over HTTPS).
- **Session**: server-side storage of user state, identified by a **session ID** — typically that session ID itself is stored in a cookie on the client, but the actual data lives on the server.

**Gotcha (very commonly asked):** "Why not just store everything in cookies?"

- Cookies are limited in size (~4KB), sent with **every** request (adds overhead), and visible/editable by the client (security risk if sensitive data is stored there unencrypted).
- Sessions keep sensitive data server-side; the client only holds an opaque ID that means nothing without server-side lookup.

**Stateless vs stateful — this ties in and gets asked:** HTTP is inherently **stateless** (server doesn't remember you between requests). Cookies/sessions/tokens are the _workaround_ to simulate state on top of a stateless protocol.

**Bonus if asked — JWT vs sessions:** JWT (JSON Web Token) is a stateless alternative — the token itself carries the user's data (signed, not encrypted by default), so the server doesn't need to store session state at all. Trade-off: harder to revoke a JWT before it expires (no central session store to just delete from).

---

**Quick recap:**

- OSI = 7 layers (theory), TCP/IP = 4 layers (practice).
- TCP = reliable/ordered/slower, UDP = fast/no guarantee.
- 3-way handshake = SYN, SYN-ACK, ACK.
- HTTPS = HTTP + TLS (asymmetric to start, symmetric for actual data).
- Idempotent = same result no matter how many times you call it (GET/PUT/DELETE yes, POST no).
- 401 = not authenticated, 403 = authenticated but not allowed.
- Cookies = client-side, Sessions = server-side (with client holding just an ID).

Want a rapid-fire quiz on this, or move to the next topic (OS, OOP, DSA)?It looks like your message didn't come through — no new question landed on my end. Let me know which topic you want next (OS, OOP, DSA, or a rapid-fire quiz on DBMS/CN) and I'll jump right in.