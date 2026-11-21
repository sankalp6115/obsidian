# FastAPI Week 8: Advanced Security & Authorization - Concept-Focused Guide

**Duration:** 13 hours **Learning Style:** Concepts and threat models first, code lookups via documentation **Prerequisites:** Weeks 1-7 completed **Goal:** Understand how authorization models differ, how attackers think about APIs, and how to design defenses in depth

---

## Table of Contents

1. [Authorization Models - 3 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#authorization-models---3-hours)
2. [Ownership, IDOR and Object-Level Authorization - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#ownership-idor-and-object-level-authorization---2-hours)
3. [Password Storage and Credential Handling - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#password-storage-and-credential-handling---2-hours)
4. [Threat Modeling for APIs - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#threat-modeling-for-apis---25-hours)
5. [Defense in Depth - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#defense-in-depth---2-hours)
6. [Secrets, Audit and Incident Thinking - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#secrets-audit-and-incident-thinking---15-hours)

---

# AUTHORIZATION MODELS - 3 HOURS

## Part 1: Why a Model Matters

### Formal Definition: Access Control Model

**Access Control Model:** A formal way of describing which subjects (users, services) may perform which actions (read, write, delete) on which objects (posts, users, files), under which conditions.

Without a model, authorization becomes a collection of `if` statements scattered across endpoints. Each new endpoint is a new chance to forget a check. A model gives you one vocabulary and one place to reason about correctness.

### The Four Classic Models

**1. Discretionary Access Control (DAC)**

```
The owner of a resource decides who else can access it.
Example: A user shares a Google Doc with a colleague.

Strength: Flexible, matches how people think about sharing.
Weakness: Permissions spread unpredictably; hard to audit globally.
```

**2. Mandatory Access Control (MAC)**

```
A central policy classifies subjects and objects (e.g., confidentiality levels).
Owners cannot override the policy.
Example: Government classification levels.

Strength: Strict, predictable.
Weakness: Rigid; rarely a fit for typical web APIs.
```

**3. Role-Based Access Control (RBAC)**

```
Permissions attach to roles. Users receive roles.
User → Role → Permissions

Example: editor can create and update posts; admin can also delete users.

Strength: Easy to reason about and audit; matches organizational structure.
Weakness: Role explosion. "Editor-in-finance-region-EU-but-not-HR" becomes too many roles.
```

**4. Attribute-Based Access Control (ABAC)**

```
Decisions depend on attributes of the subject, object, action and environment.
Rule form: allow if subject.department == object.department and time is within business hours

Strength: Expressive; handles fine-grained conditions without role explosion.
Weakness: Policies become hard to test and audit; logic spreads through code.
```

**Relationship-Based Access Control (ReBAC)** is a practical extension: access follows relationships in a graph ("user is a member of team that owns project"). Systems such as Google's Zanzibar popularized it.

### Choosing a Model: The Concept

```
Start with RBAC for coarse permissions (who is an admin, who is an editor).
Add ownership rules for object-level permissions (only the author edits a post).
Move toward ABAC or ReBAC only when conditions multiply and role count explodes.

Rule of thumb: Use the simplest model whose policies you can explain in one paragraph.
```

### Least Privilege and Default Deny

Two principles underlie every model:

- **Least privilege:** A subject receives only the permissions needed for its task.
- **Default deny:** If no rule explicitly allows an action, it is refused.

Default deny is the most important of the two. A forgotten endpoint should fail closed, not open.

---

## Part 2: Scopes vs Roles

### Concept: Two Ways to Express Permission in Tokens

```
Role claim:    "role": "editor"
               Meaning: this user is an editor. Server decides what editors can do.

Scope claim:   "scope": "posts:read posts:write"
               Meaning: this token may read and write posts, nothing else.
```

**Roles** describe the identity of the user. **Scopes** describe what a particular token is allowed to do, and they can be narrower than the user's full rights.

Scopes matter most when:

- A third-party app should get limited access (OAuth2 from Week 7)
- A token is issued for one purpose (e.g., a token for uploading images only)
- You want to reduce damage if a token leaks

### Concept: Permission Checks Happen on the Server

```
Client-side checks (hiding a button) are UX, not security.
Any attacker can call the API directly, bypassing the UI.

Rule: Every permission decision must be enforced by the server,
on every request, for every object.
```

---

# OWNERSHIP, IDOR AND OBJECT-LEVEL AUTHORIZATION - 2 HOURS

## Part 1: The Most Common Real-World Vulnerability

### Formal Definition: Insecure Direct Object Reference (IDOR)

**IDOR:** A vulnerability where an application exposes a reference to an internal object (such as a database ID) and does not verify that the requesting user is allowed to access that specific object.

```
Request:  GET /posts/42
Server:   Returns post 42 to anyone who asks.
Attacker: Tries GET /posts/43, /posts/44, ...
Result:   Reads drafts belonging to other authors.
```

IDOR is consistently ranked among the most common API flaws (it appears as "Broken Object Level Authorization" at the top of the OWASP API Security Top 10). The bug is almost never "no login required." The bug is "logged in, but not checked against this object."

### The Two Questions Every Endpoint Must Answer

```
1. Is this user allowed to perform this ACTION at all?   (function-level)
2. Is this user allowed to act on THIS OBJECT?            (object-level)

Example: DELETE /posts/42
- Function-level: Does the user have the "delete post" permission?
- Object-level:  Is the user the author of post 42, or an admin?

Passing (1) without (2) is the classic bug.
```

## Part 2: Ownership Patterns

### Pattern: Ownership Field

```
Each object records its owner.
Authorization compares the requester with the owner.

Posts: owner = author_id
Check:  requester.id == post.author_id OR requester is admin
```

### Pattern: Ownership Through Relationships

```
Comment belongs to a post; post belongs to an author.
Who may edit a comment?
- Its author
- The author of the post it belongs to (moderation)
- Admins

Questions to ask: How deep does ownership go? Does it inherit?
```

### Pattern: Return "Not Found" Instead of "Forbidden"

**Concept:** When a user has no access to an object, returning 403 Forbidden reveals that the object exists. Returning 404 Not Found hides its existence.

```
Trade-off:
- 403: Clearer for legitimate users; leaks existence.
- 404: Hides existence; slightly confusing for legitimate users.

Use 404 for private resources where existence itself is sensitive.
```

### Pattern: Filter at Query Time, Not After Fetch

```
WRONG:  Load all posts, then remove those the user cannot see.
        Risks: leaks through timing, pagination counts, and bugs in the filter.

RIGHT:  Build the query so the database only returns visible rows.
        Pagination and totals then remain correct.
```

## Part 3: Mass Assignment

### Formal Definition: Mass Assignment

**Mass assignment:** A vulnerability where a client sends extra fields in a request body that the server blindly copies into the object, such as `"role": "admin"` in a profile update.

```
Client sends: {"username": "alice", "role": "admin"}
Server copies everything into the User object.
Result: Alice promotes herself.
```

**Principle:** Accept only the fields a given operation is allowed to change. Separate input schemas for create, update, and admin operations is the standard defense. This connects directly to the schema-separation decision from Week 6.

---

# PASSWORD STORAGE AND CREDENTIAL HANDLING - 2 HOURS

## Part 1: Why Passwords Are Not Stored Plainly

### Formal Definition: Password Hashing

**Password hashing:** Transforming a password with a one-way function so that the stored value cannot be reversed to the original password, while a correct password can still be verified.

```
Stored: a one-way digest of the password, never the password itself.
Login:  hash the submitted password the same way, compare digests.

If the database leaks, attackers get digests, not passwords.
```

### Why Fast Hashes Are Wrong

```
SHA-256 is designed to be fast.
Modern GPUs compute billions of SHA-256 hashes per second.
An attacker can test enormous password lists against a leaked database.

Password hashing functions are designed to be deliberately slow and memory-hard.
```

### The Three Properties of a Good Password Hash

```
1. Salt:        A unique random value per password, so identical passwords
                produce different digests and precomputed tables (rainbow tables)
                become useless.
2. Work factor: A tunable cost that makes each guess expensive.
                The cost should be raised as hardware improves.
3. Algorithm:   Use a vetted, purpose-built function such as Argon2id
                (current recommendation), scrypt, or bcrypt.

Never invent your own hashing scheme.
Salts and work factors are handled by the library; your job is choosing it.
```

## Part 2: Credential Attacks

### Attack: Credential Stuffing

```
Attackers reuse username/password pairs leaked from other sites.
Defense concepts:
- Rate limiting per account and per IP
- Breach-password checks at registration and password change
- Multi-factor authentication
```

### Attack: Brute Force and Password Spraying

```
Brute force: many guesses against one account.
Spraying:    one common password tried against many accounts,
             staying under per-account lockout thresholds.

Defenses are layered: per-IP limits, per-account limits, global anomaly detection.
```

### Concept: Account Enumeration

```
Login says "wrong password" for existing users and "no such user" otherwise.
Attackers learn which emails have accounts.

Principle: Return the same message and similar timing for both cases.
Registration and password reset face the same issue.
```

### Concept: Rate Limiting Is a Policy, Not a Feature

```
Questions to decide:
- Limit what? (requests, failed logins, password resets)
- Keyed by what? (IP, account, token, API key)
- Window and burst? (fixed window, sliding window, token bucket)
- Behavior on limit? (429 with Retry-After)

Caveat: IP-based limits fail behind shared NAT or proxies.
Consider the proxy configuration before trusting client IP headers.
```

---

# THREAT MODELING FOR APIS - 2.5 HOURS

## Part 1: Thinking Like an Attacker

### Formal Definition: Threat Model

**Threat model:** A structured analysis of what you are protecting, who might attack it, how, and what you will do about each risk.

### The Four Questions (Shostack-style)

```
1. What are we building?           (draw the data flows)
2. What can go wrong?              (enumerate threats)
3. What are we going to do about it? (mitigations)
4. Did we do a good enough job?    (review, test)
```

### Applying STRIDE to an API

```
S - Spoofing:              Pretending to be another user      (weak auth, stolen tokens)
T - Tampering:             Modifying data in transit or storage (unsigned tokens, mass assignment)
R - Repudiation:           Denying an action happened          (missing audit logs)
I - Information disclosure: Leaking data                       (IDOR, verbose errors, enumeration)
D - Denial of service:     Exhausting resources                (no rate limits, unbounded queries)
E - Elevation of privilege: Gaining higher rights              (broken function-level checks)

Walk each endpoint through all six letters.
```

## Part 2: OWASP API Security Top 10 (Conceptual Map)

```
Broken Object Level Authorization  → IDOR (covered above)
Broken Authentication              → weak credentials, token mistakes
Broken Object Property Level Auth  → mass assignment, over-exposed fields
Unrestricted Resource Consumption  → rate limits, pagination caps, payload size
Broken Function Level Authorization → admin endpoints reachable by users
Unrestricted Access to Sensitive Business Flows → abusing signup, coupons
Server-Side Request Forgery (SSRF) → server fetching attacker-chosen URLs
Security Misconfiguration          → debug mode, open CORS, default secrets
Improper Inventory Management      → forgotten old API versions still live
Unsafe Consumption of APIs         → trusting third-party API responses blindly
```

Read the current OWASP list directly for exact wording; the categories above are the conceptual core.

## Part 3: Injection in an ORM World

### Concept: Why SQL Injection Is Mostly Solved, But Not Entirely

```
Parameterized queries (which ORMs use by default) separate SQL code from data.
Injection happens when data is concatenated into SQL text.

Still risky:
- Raw SQL strings built with f-strings
- Dynamic column or table names from user input (parameters cannot bind identifiers)
- ORDER BY clauses built from request fields
```

**Principle:** Treat any user input that controls structure (columns, sort order, table names) as hostile, and map it through an allowlist.

### Concept: Validation vs Sanitization vs Encoding

```
Validation:   Reject input that does not match the expected shape (Pydantic, Week 2-3).
Sanitization: Remove or transform dangerous content. Brittle; avoid as a primary defense.
Encoding:     Escape data for the context where it lands (HTML, SQL, shell, URL).

Primary defenses: validate at the boundary, parameterize at the database,
encode at output. Each context needs its own encoding.
```

## Part 4: Server-Side Request Forgery

```
Scenario: An endpoint accepts a URL to fetch a profile image.
Attacker supplies: http://169.254.169.254/ (cloud metadata service)
Server fetches it and returns internal credentials.

Defenses (conceptual):
- Allowlist destinations rather than blocklisting bad ones
- Resolve DNS and check the resulting IP is not private or link-local
- Restrict outbound network access from the application
```

---

# DEFENSE IN DEPTH - 2 HOURS

## Part 1: The Principle

### Formal Definition: Defense in Depth

**Defense in depth:** Layering independent security controls so that the failure of one does not compromise the system.

```
Layer 1: Network (firewall, only expose needed ports)
Layer 2: Transport (HTTPS everywhere, HSTS from Week 4)
Layer 3: Edge (rate limits, request size limits)
Layer 4: Application (authentication, authorization, validation)
Layer 5: Data (encryption at rest, least-privilege database users)
Layer 6: Monitoring (audit logs, alerts)

An attacker must defeat several layers, not one.
```

## Part 2: Trust Boundaries

### Concept: Where Trust Changes

```
Trust boundary: a line where data moves from less-trusted to more-trusted territory.

Examples:
- Internet → your API (all input untrusted)
- API → database (validated input, but database credentials are sensitive)
- API → third-party service (their responses are untrusted input too)
- Internal service → internal service (often over-trusted; still worth authenticating)

Rule: Validate and authorize at every boundary, not only at the outermost edge.
```

## Part 3: Error Handling as a Security Surface

```
Verbose errors leak:
- Stack traces (library versions, file paths)
- SQL fragments (schema structure)
- Internal IDs and configuration

Principle: Detailed errors go to logs. Clients receive generic messages with a
correlation ID that support can match to the log entry.
```

Connect to Week 3: custom exception handlers are where this policy is enforced.

## Part 4: CORS and CSRF, Revisited

### Concept: Two Different Browser Protections

```
CORS (Week 4): Controls whether JavaScript on another origin may READ responses.
CSRF:          A malicious site causes the victim's browser to SEND a request
               using ambient credentials (cookies).

Why it matters:
- Cookie-authenticated APIs are CSRF-exposed. SameSite cookies and CSRF tokens mitigate.
- Bearer-token APIs (Week 7) are not automatically sent by browsers, so CSRF is much less of a concern,
  but tokens stored in JavaScript-readable storage become an XSS concern.

Choosing a storage model is choosing which attack you are exposed to.
```

## Part 5: Security Headers and Content Policies

```
Headers from Week 4 (X-Content-Type-Options, X-Frame-Options, HSTS, CSP)
form one layer. Their job is to limit what a browser will do with your responses,
even if something upstream fails.

Conceptual principle: the API returns JSON, so strict CSP and framing controls
matter most where the API serves HTML (docs pages, admin panels).
```

---

# SECRETS, AUDIT AND INCIDENT THINKING - 1.5 HOURS

## Part 1: Secrets Management

### Formal Definition: Secret

**Secret:** Any value whose disclosure would let an unauthorized party act as your system: signing keys, database passwords, API keys, encryption keys.

### Concept: Secret Lifecycle

```
Create:   Generate with a cryptographically secure source, never by hand.
Store:    Environment or a dedicated secret manager. Never in source control.
Use:      Load at startup; minimize where in code it is visible.
Rotate:   Plan for it. A rotation process you have never run will fail when needed.
Revoke:   Know how to invalidate a compromised secret quickly.
```

### Concept: Key Rotation with Tokens

```
JWT signing keys can be rotated with a key identifier (kid) in the token header.
Old and new keys are accepted during a transition window, then the old key is retired.

Principle: rotation must not require logging every user out, if you can avoid it.
```

## Part 2: Audit Logging

### Formal Definition: Audit Log

**Audit log:** An append-only record of security-relevant actions: who did what, to which object, when, and from where.

```
Log these:
- Login success and failure (without the password)
- Permission changes and role grants
- Deletions and privileged operations
- Token issuance and revocation

Do not log:
- Passwords, tokens, or full secrets
- Full request bodies containing personal data without a reason
```

### Concept: Logs Are Also a Target

```
Attackers erase logs once inside.
Protect logs: restrict write access, ship them to a separate system,
and make them tamper-evident where the stakes justify it.
```

## Part 3: Incident Thinking

### Concept: Assume Breach

```
Ask the planning questions before an incident:
- If a token leaks, how do we invalidate it, and how fast?
- If a database dump leaks, what can an attacker actually do with it?
- If an admin account is compromised, what is the blast radius?
- Who gets paged, and what is the first action?

Good designs make each answer small: short token lifetimes, hashed passwords,
least-privilege roles, and audit trails that show what happened.
```

---

# KEY CONCEPTS SUMMARY

## Authorization Models

- DAC, MAC, RBAC, ABAC, ReBAC: different trade-offs between flexibility and auditability
- Start with RBAC plus ownership; add complexity only when needed
- Default deny and least privilege are the foundations

## Object-Level Security

- IDOR: authenticated is not the same as authorized for this object
- Filter in queries, not after fetching
- Guard against mass assignment with input schemas per operation

## Credentials

- Password hashing with a salted, slow, vetted algorithm (Argon2id preferred)
- Defend against credential stuffing, spraying, and enumeration together
- Rate limiting is a policy with several dimensions

## Threat Modeling

- Draw data flows, enumerate threats (STRIDE), mitigate, review
- Map OWASP API Top 10 categories onto each endpoint
- Injection risk lives in raw SQL, dynamic identifiers, and ordering inputs

## Defense in Depth

- Multiple independent layers, validation at every trust boundary
- Generic errors to clients, detailed errors to logs
- Browser protections (CORS, CSRF, headers) address different threats

## Operations

- Secrets have a lifecycle, including rotation and revocation
- Audit logs record security-relevant actions, without secrets
- Plan for breach: make the blast radius of each failure small

---

# WEEK 8 CHECKPOINT PROJECT

## Project: Harden the Blog API

### Objective

Take your Week 7 Blog API with authentication and produce a hardened version with a written security analysis.

### Part 1: Authorization Model Document

Write a one-page policy covering:

- Roles and what each role can do
- Which operations are ownership-based
- Which operations are scope-restricted
- Default-deny behavior for unlisted endpoints

### Part 2: IDOR and Mass Assignment Review

For every endpoint that takes an ID or a request body:

- State the function-level check
- State the object-level check
- List the fields a client may set, per operation
- Note any endpoint where you are unsure, and why

### Part 3: Credential Hardening Plan

Describe:

- Your password hashing choice and why it fits
- How login responds to unknown users versus wrong passwords
- Your rate limits: what is keyed by what, and the response on limit

### Part 4: Threat Model

Draw the data flows for the Blog API, then walk each endpoint through STRIDE. Output a table: threat, affected endpoint, mitigation, residual risk.

### Part 5: Error and Logging Policy

- What clients see on a 500 error
- What is written to logs and what is excluded
- How a support engineer links a client report to a log entry

### Part 6: Secrets and Rotation Runbook

Write steps for:

- Rotating the JWT signing key without mass logout
- Revoking a compromised refresh token family
- Responding to a leaked database password

### Evaluation Checklist

**Conceptual understanding:**

- [ ] Can explain RBAC vs ABAC and when each fits
- [ ] Can describe IDOR and give an example in your own API
- [ ] Can explain why fast hashes are wrong for passwords
- [ ] Can apply STRIDE to one endpoint
- [ ] Can distinguish CORS from CSRF

**Implementation:**

- [ ] Object-level checks exist on all mutating endpoints
- [ ] Input schemas prevent privilege fields from being set by clients
- [ ] Login does not reveal which accounts exist
- [ ] Rate limits applied to login and password reset
- [ ] Errors do not leak stack traces

---

# RESOURCES FOR DEEPER LEARNING

**Standards and references:**

- OWASP API Security Top 10 (read the current edition)
- OWASP Authentication, Authorization, and Password Storage Cheat Sheets
- NIST SP 800-63B (digital identity guidelines)
- RFC 9700 (OAuth 2.0 security best current practice)

**Concepts to read about:**

- Google Zanzibar paper (ReBAC at scale)
- "Threat Modeling: Designing for Security" by Adam Shostack
- "Web Application Security" by Andrew Hoffman

**Tools for exploration:**

- OWASP ZAP (test your own API for common flaws)
- Burp Suite Community (inspect and tamper with requests)
- Your own API, tested with a second user account, to check IDOR by hand

---

**End of Week 8 Content**

Security is mostly a habit of asking "who is allowed to do this, to this specific thing?" at every boundary. Once that question is automatic, most vulnerabilities stop appearing in your code.

**Next week:** WebSockets and real-time features, or do you want to revisit any Week 8 concept first?