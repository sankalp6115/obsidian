# FastAPI Week 7: JWT & OAuth2 - Concept-Focused Security Guide

**Duration:** 14 hours  
**Learning Style:** Concepts first, protocols over implementation  
**Prerequisites:** Weeks 1-6 completed  
**Goal:** Master authentication & authorization patterns, understand JWT and OAuth2 deeply

---

## Table of Contents

1. [Authentication vs Authorization - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#authentication-vs-authorization---15-hours)
2. [HTTP Authentication Fundamentals - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#http-authentication-fundamentals---2-hours)
3. [JWT (JSON Web Tokens) - 3.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#jwt-json-web-tokens---35-hours)
4. [OAuth2 Protocol - 4 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#oauth2-protocol---4-hours)
5. [Security Best Practices - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#security-best-practices---2-hours)
6. [Real-World Decision Making - 1.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#real-world-decision-making---15-hours)

---

# AUTHENTICATION VS AUTHORIZATION - 1.5 HOURS

## Part 1: Fundamental Definitions

### Formal Definition: Authentication

**Authentication (Authn):** The process of verifying that someone is who they claim to be.

```
Question authentication answers:
"Are you really Alice?"

Verification methods:
- Something you know (password)
- Something you have (phone, hardware token)
- Something you are (fingerprint, face)
- Somewhere you are (IP address, GPS)

Result: Identity confirmed or denied
```

### Formal Definition: Authorization

**Authorization (Authz):** The process of determining what an authenticated user is allowed to do.

```
Question authorization answers:
"What can Alice do?"

After proving you're Alice:
- Can Alice view posts?
- Can Alice delete posts?
- Can Alice create users?
- Can Alice access admin panel?

Result: Permissions granted or denied
```

### The Critical Distinction

**Real-world analogy:**

```
Airport security checkpoint:

AUTHENTICATION:
- "May I see your ID?"
- Guard checks passport
- Confirms: "Yes, you're John Smith"

AUTHORIZATION:
- Guard checks ticket
- Sees destination: "New York"
- Allows through: "Gate 5"
- Denies through: "Gate 12" (different flight)

Both needed: Proven identity + Confirmed permissions
```

### Why The Distinction Matters

**Common mistake:** Confusing the two

```
WRONG: "User is authenticated, so they can access anything"
RIGHT: "User is authenticated. Now check their permissions."

Consequence of confusion:
- Authentication succeeds → user logged in
- No authorization check → user can do anything
- Security vulnerability!
```

---

## Part 2: Authentication Flow Basics

### The Basic Authentication Challenge

**Scenario:**

```
Client: "Hi server, I'm Alice"
Server: "How do I know you're really Alice?"
Client: "Here's my password"
Server: "Correct! You're Alice"
```

**Problem:** How does this work over HTTP?

### HTTP Statelessness Challenge

**Core issue:**

```
HTTP is stateless: No memory between requests

Request 1: Client sends password
Request 2: Client makes API call
Server: "Who are you?" (forgot!)

Solution needed: 
"Prove your identity with EVERY request"

Methods:
1. Send credentials every time (password every request - BAD)
2. Send token after login (token proves identity - GOOD)
```

### Mental Model: The Token Concept

**Abstraction:**

```
Before authentication:
- You: Stranger to server
- Server trusts: Nothing about you
- Access: None

Authentication succeeds:
- You get: Token (proof of identity)
- Token represents: "Server verified this person"
- Token encodes: Identity, issue time, expiration

With each request:
- You send: Token
- Server checks: Is token valid and fresh?
- Access: Granted (if token valid)

Logout:
- You discard: Token
- Token now worthless: Server won't accept it
```

---

# HTTP AUTHENTICATION FUNDAMENTALS - 2 HOURS

## Part 1: Basic Authentication (HTTP Basic Auth)

### Formal Definition: HTTP Basic Authentication

**HTTP Basic Auth:** Client sends username:password (base64 encoded) in Authorization header with every request.

### How It Works Conceptually

```
Client requests: GET /protected-resource

Client encodes: "alice:mypassword" → base64 → "YWxpY2U6bXlwYXNzd29yZA=="

Client sends header:
Authorization: Basic YWxpY2U6bXlwYXNzd29yZA==

Server:
1. Decodes base64 → "alice:mypassword"
2. Checks if alice exists
3. Checks if password matches
4. Grants access

Next request: Client sends SAME header again
```

### Basic Auth Trade-offs

**Advantages:**

- ✓ Simple to understand
- ✓ Built into HTTP spec
- ✓ All browsers support it
- ✓ No extra infrastructure

**Disadvantages:**

- ✗ Password sent every request (more exposure)
- ✗ Server must verify password every time (slow)
- ✗ Base64 is not encryption (base64 decoding = plaintext password)
- ✗ No logout mechanism
- ✗ Must use HTTPS or passwords exposed

### When to Use Basic Auth

```
✓ Internal APIs (trusted networks)
✓ Development/testing
✓ APIs with minimal security needs
✓ Simple integrations

✗ Public APIs
✗ High-security applications
✗ Applications with many users
✗ When security is priority
```

---

## Part 2: Bearer Tokens (Generic Pattern)

### Formal Definition: Bearer Token

**Bearer Token:** Opaque token sent in Authorization header that proves user identity without resending credentials.

### The Bearer Pattern

```
Authorization: Bearer <token-here>

"Bearer" means: "I'm presenting a token that grants access"

Server doesn't need to:
- Verify password
- Store password
- Validate credentials again

Server only needs to:
- Verify token is legitimate
- Check if token expired
```

### Bearer Token Advantages

**Conceptual benefit:**

```
Basic Auth approach:
Request 1: Alice sends password → server checks → access granted
Request 2: Alice sends password → server checks → access granted
Request 3: Alice sends password → server checks → access granted
...
Request 100: Alice sends password → server checks → access granted

Cost: 100 password verifications

Bearer Token approach:
Initial: Alice sends password → server creates token
Request 1: Alice sends token → server verifies token → access
Request 2: Alice sends token → server verifies token → access
Request 100: Alice sends token → server verifies token → access

Cost: 1 password check + 100 faster token verifications
```

**Security benefit:**

```
Token advantages:
- Can expire (credentials don't)
- Can be revoked (password change required before)
- Can be limited in scope (password is all-or-nothing)
- Can include metadata (who issued, what access)
```

---

## Part 3: Session-Based vs Token-Based Authentication

### Session-Based Authentication

**How it works:**

```
Client login:
1. Send username:password
2. Server verifies
3. Server creates session (stores in memory/database)
4. Server sends back session ID cookie

Client subsequent requests:
1. Browser automatically sends cookie
2. Server looks up session
3. Confirms user identity
4. Grants access

Logout:
1. Server deletes session
2. Cookie no longer valid
```

**Conceptual model:**

```
Session = Server-side memory of user login
Cookie = Token that references that memory

Server must remember every logged-in user
```

### Token-Based Authentication

**How it works:**

```
Client login:
1. Send username:password
2. Server verifies
3. Server creates token (stateless, self-contained)
4. Server sends token to client

Client subsequent requests:
1. Client sends token in header
2. Server validates token signature
3. Server extracts user info from token
4. Grants access

Logout:
1. Client discards token
2. Token still valid technically (but client won't use it)
```

**Conceptual model:**

```
Token = Self-contained proof of identity
No server-side memory needed
Server only validates, doesn't store
```

### Session vs Token Comparison

**Session-Based:**

```
Pro:
- Can revoke immediately (server deletes)
- Can change permissions instantly
- Can logout and invalidate everywhere
- Server has control

Con:
- Server must store every session
- Doesn't scale to millions of users
- Can't work across multiple servers easily
- Requires shared session storage
```

**Token-Based:**

```
Pro:
- Stateless (server doesn't store anything)
- Scales to millions of users
- Works across multiple servers
- Fits distributed systems
- Easier for mobile/API clients

Con:
- Can't revoke immediately (token valid until expiry)
- Must wait for expiration to take effect
- Permissions changes not instant
- More complex security model
```

### Decision: When to Use Each

```
Use Session-Based:
- Web applications
- Traditional server-rendered apps
- Need immediate revocation
- Single server
- Small user base

Use Token-Based:
- APIs
- Mobile apps
- Microservices
- Multiple servers
- Distributed systems
- Large user base
```

---

# JWT (JSON WEB TOKENS) - 3.5 HOURS

## Part 1: JWT Structure and Concept

### Formal Definition: JWT

**JWT (JSON Web Token):** A standard format for creating tokens that can be verified without accessing a database. Token is cryptographically signed to prove authenticity.

### JWT's Core Innovation

**The problem it solves:**

```
Before JWT:
Token = Random string
Server receives token → Must lookup in database
Database query for every request!

With JWT:
Token = Encoded data + signature
Server receives token → Verify signature mathematically
No database lookup needed!

Result: Stateless authentication possible
```

### JWT Structure

**Visual breakdown:**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFsaWNlIn0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Part 1: Header (algorithm info)
Part 2: Payload (data)
Part 3: Signature (proof it wasn't tampered with)

Three parts separated by dots
```

### JWT Payload (The Data)

**What goes in the token:**

```
{
  "sub": "user_123",           // subject (who)
  "name": "Alice",             // username
  "email": "alice@example.com",// email
  "iat": 1516239022,           // issued at (when)
  "exp": 1516242622,           // expiration (when it dies)
  "role": "admin"              // permissions
}
```

**Conceptual understanding:**

```
Token IS the identity proof
Token CONTAINS the user info
No database lookup needed
Server reads user info directly from token
```

### Security: The Signature

**Why signature matters:**

```
Without signature:
- Token = plaintext data
- Attacker can modify: Change role from "user" to "admin"
- Server trusts modified token
- SECURITY BROKEN

With signature:
- Token = encoded data + cryptographic signature
- Server has SECRET KEY
- Attacker modifies token → signature breaks
- Server detects tampering
- Rejects token
- SECURE
```

**How signature works (conceptually):**

```
Server has secret key: "super-secret-123"
Server creates token data: "sub=alice,role=user"
Server signs it: HMAC-SHA256(data, secret) = "xyz789"
Token sent: "sub=alice,role=user.xyz789"

Later, attacker intercepts:
Modifies token: "sub=alice,role=admin.xyz789"
Server verifies: HMAC-SHA256("sub=alice,role=admin", secret)
Result: "abc123" ≠ "xyz789"
Server rejects: Signature doesn't match!
```

---

## Part 2: JWT Claims and Lifecycle

### Standard JWT Claims (What's in the token)

**Registered Claims (standard ones):**

```
iss (issuer):    Who created token - "https://auth.example.com"
sub (subject):   Who is token for - "user_123"
aud (audience):  Who should use it - "api.example.com"
exp (expiration):When it expires - Unix timestamp
iat (issued at): When created - Unix timestamp
nbf (not before):When it becomes valid - Unix timestamp
jti (JWT ID):    Unique identifier - Prevent replay
```

**Custom Claims:**

```
Add whatever you need:
- role: "admin"
- permissions: ["read", "write", "delete"]
- department: "engineering"
- api_key_name: "production_key"

Design principle:
Include what you need to make authorization decision
Don't include secrets or sensitive data
```

### JWT Lifecycle

**Timeline:**

```
t=0:     User logs in
         Server creates JWT with exp=now+1hour
         Server sends JWT to client

t=5m:    Client makes request with JWT
         Server verifies signature
         Server checks: is exp > now?
         Yes → request granted

t=59m:   Client makes request with JWT
         Server checks: is exp > now?
         Yes, still valid → request granted

t=61m:   Client makes request with JWT
         Server checks: is exp > now?
         No, expired! → request rejected
         Client must login again

t=∞:     Even if attacker steals JWT, it's worthless
         (unlike password which never expires)
```

### Access Tokens vs Refresh Tokens

**Problem JWT solves:**

```
Make token short-lived (security):
- If stolen, attacker has limited time
- Reduce damage from compromise

But users hate re-authenticating often:
- Every 15 minutes to login again? Annoying

Solution: Two-token pattern
```

**Two-Token Pattern:**

```
Access Token:
- Short lived (15 minutes)
- Included in API requests
- Quick expiration limits damage

Refresh Token:
- Long lived (7 days)
- Stored securely (usually httpOnly cookie)
- Used to get new access token
- Never included in API calls

Flow:
User login → Get both tokens

Make API request:
- Use access token
- Expires after 15 min
- User continues using app

Access token expires:
- Client uses refresh token to get new access token
- All automatic, user doesn't notice

User logout:
- Delete refresh token
- Access tokens still valid until expiration (acceptable)
- New tokens can't be issued
```

**Trade-off explained:**

```
Single token (simple):
- One token controls everything
- Must be short-lived (security)
- Users must re-auth frequently (poor UX)

Two tokens (complex):
- Access token short-lived (security)
- Refresh token long-lived (UX)
- Can revoke refresh token instantly
- Access tokens still valid (acceptable tradeoff)
```

---

## Part 3: JWT Security Principles

### Signature Algorithm Choice

**Concept: HMAC vs RSA**

```
HMAC (symmetric):
- Same secret signs AND verifies
- Faster
- Can't be verified by third parties
- Good for: Internal APIs

RSA (asymmetric):
- Private key signs
- Public key verifies
- Can share public key widely
- Good for: Third-party verification, OAuth2
```

### Secret Key Management

**Critical concept:** Protect your secret

```
Token security = Secret security

If secret compromised:
- Attacker can forge valid tokens
- Impersonate any user
- Access any resource
- TOTAL COMPROMISE

Therefore:
- Never commit secret to git
- Use environment variables
- Rotate regularly
- Different secrets per environment
```

### Common JWT Security Mistakes

**Mistake 1: Unencrypted tokens**

```
JWT tokens are BASE64 ENCODED, not encrypted
- Anyone can read the payload
- Don't put secrets in JWT!

Payload visible to:
- User (token in browser)
- Network observer (if HTTP)
- Anyone with token

Solution:
- Only put non-sensitive data in JWT
- Use HTTPS to hide token
```

**Mistake 2: No signature verification**

```
WRONG: Trust token without verification
- Server accepts modified tokens
- Attacker changes role from user to admin
- Security broken

RIGHT: Always verify signature
- Ensure token wasn't modified
- Ensure server issued it
- Reject unsigned tokens
```

**Mistake 3: Ignoring expiration**

```
WRONG: Create token with no expiration
- Stolen token valid forever
- Can't limit damage

RIGHT: Set reasonable expiration
- Balance security (short) vs UX (not too short)
- 15-60 minutes typical for access tokens
```

---

# OAUTH2 PROTOCOL - 4 HOURS

## Part 1: OAuth2 Purpose and Philosophy

### Formal Definition: OAuth2

**OAuth2:** Authorization protocol that allows users to grant third-party applications access to their resources without sharing their password.

### The Problem OAuth2 Solves

**Scenario before OAuth2:**

```
You: "I want to use Spotify app"
Spotify app: "Give me your password"
You: "Here's my password"
Spotify app: "Now I have your password!"

Problems:
- You share password with app
- App has full access (can't limit)
- Can't revoke without changing password
- Password changes affect all apps
- App could be malicious with your password
```

**After OAuth2:**

```
You: "I want to use Spotify app"
Spotify app: "Redirect to Spotify to authorize"
(Browser opens Spotify.com)
Spotify: "Spotify app wants permission to read your playlists. OK?"
You: "Yes"
Spotify: "Here's a token for Spotify app"
Spotify app: "Got token, can read playlists"

Benefits:
- You never give app your password
- Spotify controls what app can do
- Can revoke any time
- All apps have different access
- Password never shared
```

### OAuth2 Is Not Authentication

**Critical distinction:**

```
OAuth2 IS authorization:
- Proves: "What can app do?"
- Answers: "Access calendar, read emails"

OAuth2 is NOT authentication:
- Doesn't prove: "Who is the user?"
- Doesn't answer: "Are you Alice?"

Many systems layer authentication on top of OAuth2
(OpenID Connect does this)
```

### Use Case: Why It Matters

```
Real-world flows:

Sign up with Google:
1. Redirect to Google
2. User logs in to Google
3. Google asks: "App wants your email, OK?"
4. App gets email from Google
5. App creates account

Sign up with Facebook:
1. Similar flow
2. Facebook handles authentication
3. App just receives authorized data

Result:
- User manages one password (Google/Facebook)
- App never sees password
- App only gets what user authorized
```

---

## Part 2: OAuth2 Roles and Grant Types

### The Four OAuth2 Roles

**Role 1: Resource Owner**

```
The person who owns the data
Example: You own your Gmail inbox
You decide who can access it
```

**Role 2: Client Application**

```
The app that wants access to data
Example: Calendar app wants to read Gmail
Must ask Resource Owner for permission
```

**Role 3: Authorization Server**

```
The service that manages permissions
Example: Google (your email provider)
Asks: "Is it OK?"
Issues tokens to prove you said yes
```

**Role 4: Resource Server**

```
The service that stores actual data
Example: Gmail servers
Only releases data if token valid
Often same as Authorization Server
```

### OAuth2 Grant Types (Different Flows)

**Grant Type 1: Authorization Code Flow (Most Common)**

```
Best for: Web applications, mobile apps
Most secure flow

Sequence:
1. User clicks "Sign in with Google"
2. Redirected to Google login
3. Google asks: "Calendar app wants your calendar, OK?"
4. User clicks Yes
5. Google sends back code (temporary)
6. App exchanges code for token (behind scenes)
7. App gets token
8. App uses token to access calendar

Why this is secure:
- Code is temporary (expires in seconds)
- Code only works with app's secret
- Token never exposed to browser
```

**Grant Type 2: Implicit Flow (Legacy, Don't Use)**

```
Best for: Nothing (deprecated)
Skipped because less secure than Authorization Code Flow
```

**Grant Type 3: Client Credentials Flow**

```
Best for: Server-to-server, not users
Scenario: Your backend needs to access another service

Sequence:
1. Server has: client_id + client_secret
2. Server sends: "I'm app X with secret Y"
3. Authorization server: "Verified, here's token"
4. Server uses token to access API

No user involved
Service-to-service authentication
```

**Grant Type 4: Resource Owner Password Flow (Avoid)**

```
Best for: Legacy apps, testing only
User gives password to app (defeats OAuth purpose)
Skip this in new applications
```

---

## Part 3: OAuth2 in Practice

### OAuth2 + OpenID Connect (OIDC)

**The Gap:**

```
OAuth2 alone:
- Q: "Can I access user's calendar?"
- A: "Yes" (authorization)

Missing:
- Q: "Who is the user?"
- A: ??? (authentication)

Solution: OpenID Connect (OIDC)
- Built on top of OAuth2
- Adds authentication layer
- Adds identity information
- Widely used now
```

### Real-World Flows

**Flow 1: Sign in with Provider (Web App)**

```
User goes to your website
Clicks: "Sign in with Google"
↓
Redirected to Google
↓
Google: "Enter your Google credentials"
User: "Enters password"
↓
Google: "Sign in with Google app wants your profile, OK?"
User: "OK"
↓
Redirected back to your site
Your site: "Got token from Google"
Your site: "Got user info from Google"
Your site: "Created account for user"
User: "Logged in!"
```

**Flow 2: Third-party App Access (Desktop App)**

```
User opens Spotify desktop app
Clicks: "Connect to Apple Music account"
↓
Browser opens Apple Music login
User: "Logs in to Apple Music"
↓
Apple Music: "Spotify wants to read your playlists, OK?"
User: "OK"
↓
Browser shows: "You can close this window"
Spotify app: "Got token from Apple"
Spotify app: "Can now read playlists"
User: "App working with real data"
```

### Scope: Limiting Permissions

**Concept: Granular Access Control**

```
Google OAuth flow:
Asks for: "Scope: email, calendar.read, calendar.write"

User sees:
✓ Access your email
✓ Read your calendar
✓ Modify your calendar

User can:
- Grant all
- Grant some
- Deny all

App cannot ask for more than requested scope
Cannot downgrade to less permissions requested
```

---

## Part 4: JWT vs OAuth2

### Are They Competitors?

**No, different purposes:**

```
JWT: Format for tokens
- How to structure a token
- How to sign it
- How to verify it

OAuth2: Protocol for authorization
- How to grant permissions
- How to request access
- How roles interact

They often work together:
OAuth2 flow generates JWT token
App uses JWT to make requests
```

### Common Pattern: OAuth2 + JWT

```
Authorization Server (OAuth2):
- Authenticates user
- User grants permissions
- Generates JWT token
- Sends JWT to client

Client:
- Stores JWT
- Includes in API requests
- Sends: Authorization: Bearer <JWT>

API Server:
- Receives JWT
- Verifies JWT signature (no database needed)
- Reads user info from JWT
- Checks permissions
- Grants access
```

---

# SECURITY BEST PRACTICES - 2 HOURS

## Part 1: Storage and Transport

### Storing Tokens Safely

**For Web Apps (Browser):**

```
Option 1: localStorage
- Easy to access
- Vulnerable to XSS attacks
- Attacker's JavaScript can read it
- RISK: Not recommended for sensitive apps

Option 2: Session Cookie (httpOnly)
- Cannot be read by JavaScript
- Protected from XSS attacks
- Sent automatically with requests
- RECOMMENDED: More secure

Option 3: Memory
- Lost on page refresh
- Better UX with localStorage but with security
- Moderate approach

Recommendation: httpOnly cookie for access token
```

**For Mobile Apps:**

```
Option 1: Keychain (iOS) / Keystore (Android)
- Secure storage provided by OS
- Encrypted
- RECOMMENDED: Use it

Option 2: SharedPreferences / UserDefaults
- Not encrypted
- Accessible to other apps
- RISK: Don't use for tokens

Recommendation: Use OS-provided secure storage
```

### Transport Security

**Critical rule:**

```
ALWAYS use HTTPS
- HTTP exposes tokens to network interception
- Attacker on network can read token
- Token compromised = Full access

No exceptions
No "It's internal network"
No "Testing purposes"

HTTPS is prerequisite
```

---

## Part 2: Token Lifecycle Security

### Preventing Token Replay Attacks

**Attack scenario:**

```
1. Attacker intercepts token (even over HTTPS, if SSL broken)
2. Attacker uses stolen token
3. Server accepts it
4. Attacker gains access

Solution: Token expiration
- Token valid for 15 minutes
- After 15 minutes, even if stolen, it's worthless
- User must refresh (gets new token)
```

### Token Revocation Strategy

**Challenge:**

```
Tokens are stateless
Server doesn't know about them
Can't "revoke" a token directly

Solutions:

1. Wait for expiration (passive)
- Token naturally expires
- No immediate revocation

2. Blacklist (active)
- Maintain list of revoked tokens
- Check blacklist on each request
- Defeats stateless advantage

3. Short expiration + refresh (practical)
- Access token expires quickly (15 min)
- Refresh token can be revoked
- Can't get new access token after refresh revoked
- Existing access tokens still valid (acceptable)

Recommendation: Short expiration + refresh token revocation
```

### Logout Implementation

**Concept:**

```
Access Token:
- Expires naturally
- Still valid until expiration
- Can't be revoked
- Acceptable because short-lived

Refresh Token:
- Must be invalidated on logout
- Delete from storage
- Mark as revoked on server
- User can't get new access token

Result:
- Immediate logout effect (can't get new tokens)
- Existing access token valid until expiration
- Usually acceptable (expires in 15 min anyway)
```

---

## Part 3: Common Security Threats

### Threat 1: Weak Secret Keys

**Risk:**

```
Attacker guesses secret key
Can forge valid tokens
Can impersonate any user
TOTAL COMPROMISE
```

**Mitigation:**

```
- Use cryptographically strong random generation
- Long enough (32+ bytes for HMAC-SHA256)
- Unique per environment
- Rotated regularly
- Never in version control
```

### Threat 2: Signature Algorithm Attack

**Risk:**

```
Attacker tricks server into accepting unsigned token
Or using wrong algorithm
```

**Mitigation:**

```
- Specify algorithm in server config (not in token header)
- Don't trust algorithm from token
- Server enforces which algorithm to use
- Reject unexpected algorithms
```

### Threat 3: Expired Token Ignored

**Risk:**

```
Server accepts expired token
Attacker uses old token
```

**Mitigation:**

```
- Always check expiration
- Reject expired tokens
- No exceptions
- Even if token otherwise valid
```

### Threat 4: Data in Token

**Risk:**

```
Sensitive data in token
- Password hint
- Internal user IDs
- Credit card numbers
- API keys

Base64 decoded → plaintext visible to anyone with token
```

**Mitigation:**

```
- Only non-sensitive data in token
- User ID, username, role: OK
- Passwords, API keys, credit cards: NOT OK
- Treat token like public data
```

---

# REAL-WORLD DECISION MAKING - 1.5 HOURS

## Part 1: When to Use What

### Decision Framework: Authentication Method

```
Question 1: What type of application?
├─ Web app → Cookies (session-based or JWT)
├─ Mobile app → JWT (no cookies)
├─ API (internal) → JWT or Basic Auth
└─ API (third-party) → OAuth2 + JWT

Question 2: Need third-party integration?
├─ Yes → OAuth2 (users sign in with Google, GitHub, etc)
└─ No → JWT or session-based

Question 3: How many users?
├─ <1000 → Sessions OK, simpler
├─ 1000-100k → JWT, can distribute
└─ >100k → JWT, must scale

Question 4: Need immediate logout?
├─ Yes → Sessions (instant revocation)
└─ No → JWT (acceptable slight delay)

Result: Combination answers define strategy
```

### Decision: JWT Implementation Details

```
Question 1: Access token lifetime?
├─ Publicly accessible data → 1 hour
├─ User-specific data → 15-30 minutes
├─ Sensitive operations → 5-15 minutes

Question 2: Refresh token needed?
├─ Mobile/SPA → Yes (auto-refresh)
├─ Server-rendered → Maybe (depends on UX)
└─ API-only → No (client logs in again)

Question 3: Single or Multiple tokens?
├─ Need to scope access (e.g., specific API key access)
   → Different tokens with different scopes
├─ Simple case → Single token

Question 4: Where to store refresh token?
├─ Web app → httpOnly cookie (secure)
├─ Mobile → OS keychain (secure)
└─ SPA → localStorage (convenient, less secure)
```

---

## Part 2: Integration Patterns

### Pattern 1: Session-Based (Traditional Web)

```
Use when:
- Server-rendered pages
- Traditional web app
- Users expect "logout works immediately"
- Few servers or shared session storage

Architecture:
User login → Server creates session → Server sends cookie
Cookie sent automatically with every request
Server looks up session → grants access
Logout → Server deletes session

Pros: Simple, instant revocation
Cons: Doesn't scale, requires session storage
```

### Pattern 2: JWT-Based (Modern API)

```
Use when:
- REST API
- Multiple servers
- Mobile app
- Microservices

Architecture:
User login → Server creates JWT → Client stores JWT
Client sends: Authorization: Bearer <JWT>
Server validates JWT signature (no DB lookup)
Logout → Client deletes JWT (token still valid, acceptable)

Pros: Scalable, stateless, simple
Cons: Can't revoke immediately, more complex
```

### Pattern 3: OAuth2 + JWT (Social Login)

```
Use when:
- Want social login (Google, GitHub, etc)
- Third-party data access needed
- Users trust established providers

Architecture:
User clicks "Sign in with Google"
Redirect to Google OAuth flow
User logs in to Google
User grants permission
Google sends authorization code
Your server exchanges code for JWT from Google
Your server creates session/JWT for your app

Pros: Security delegated to provider, popular
Cons: Dependency on third party, more complex
```

---

## Part 3: Real-World Scenarios

### Scenario 1: Microservices Architecture

```
Setup:
- API Gateway (handles auth)
- Multiple microservices
- Users across globe

Best approach:
- Use JWT (stateless)
- Central auth service creates JWT
- All microservices validate JWT signature
- No need for shared session storage
- Each service independent

Reasoning:
- Can't use sessions (no shared storage)
- JWT allows each service to validate independently
- Signature verification needs only secret key (no DB)
- Scales easily
```

### Scenario 2: Mobile App with High Security

```
Setup:
- iOS/Android app
- Sensitive data (financial, health)
- Wants frictionless UX

Best approach:
- Access token (15 min, expires quickly)
- Refresh token (7 days, stored securely)
- Use OS keychain
- Background refresh when expired

Reasoning:
- Access token stolen has limited window (15 min)
- Refresh token can be revoked on logout
- UX: Seamless refresh, user doesn't notice
- Security: Balanced (steal access token = 15 min access)
```

### Scenario 3: Internal Company API

```
Setup:
- Employees only
- Trusted network
- Simple requirements
- Small team

Approach:
- API key authentication (simplest)
- Or JWT (slightly more complex)
- Or Basic Auth (acceptable internally)
- No OAuth2 needed

Reasoning:
- Internal network is trusted
- Complexity of OAuth2 not justified
- Simplicity valued
- Can start simple, add complexity later
```

### Scenario 4: Third-Party Developers Integration

```
Setup:
- Want external developers to build apps
- Using your data
- Don't want them knowing users' passwords

Approach:
- OAuth2 with your service as provider
- Each app gets code
- Users authorize app to access their data
- Apps get tokens
- Apps use tokens to access API

Reasoning:
- Developers get standardized way
- Users control what each app accesses
- Can revoke per-app anytime
- Users' passwords never shared
```

---

# WEEK 7 CHECKPOINT PROJECT

## Project: Blog API - Full Authentication & Authorization

### Objective

Add enterprise-grade authentication and authorization to your Week 6 Blog API:

1. Implement user authentication strategy
2. Design and implement authorization system
3. Integrate secure token handling
4. Implement role-based access control
5. Write tests for security scenarios

### Part 1: Authentication Strategy Decision

**Task:** Design your authentication system

**Steps:**

1. Decide: JWT or sessions or both?
    
    - What is your application type?
    - How many users expected?
    - What are revocation requirements?
    - Write decision document
2. Token design (if JWT):
    
    - What claims needed?
    - What expiration times?
    - Access + refresh or single token?
    - Storage strategy for your client?
3. Integration points:
    
    - Where does authentication happen?
    - Middleware? Dependency? Both?
    - How do tokens flow through API?

**Deliverable:** Authentication Design Document

```
- Chosen method (session/JWT/both)
- Why chosen (trade-offs analysis)
- Token structure (if applicable)
- Storage strategy
- Integration points
```

### Part 2: Authorization Architecture

**Task:** Design access control

**Analysis:**

- What roles exist? (admin, editor, user, anonymous)
- What permissions per role?
- Can permissions be overridden?
- How to handle ownership? (user can edit own posts)
- How to handle delegation? (admin can grant permissions)

**Implementation questions:**

- Check authorization: In middleware? In endpoint? In database?
- How to express permissions clearly?
- How to reuse authorization checks?

**Deliverable:** Authorization Design Document

```
- Role hierarchy
- Permissions per role
- Permission checks (ownership, delegation, etc)
- Where checks happen
- How to test them
```

### Part 3: Token Security Audit

**Task:** Audit your authentication for security

**Checklist:**

- [ ] Tokens signed and verified
- [ ] Signature algorithm not from token header
- [ ] Expiration always checked
- [ ] No sensitive data in token
- [ ] Secret key secure (not in code, in env)
- [ ] HTTPS enforced (conceptually)
- [ ] Token storage appropriate for client type
- [ ] Logout invalidates refresh tokens

**Findings Document:**

- What security issues found?
- How to fix each?
- Mitigation strategy?

### Part 4: Role-Based Access Control (RBAC)

**Task:** Implement role-based permissions

**Example scenarios:**

- GET /posts/ → All authenticated users
- POST /posts/ → Only editors and admins
- PUT /posts/{id} → Only author, editors, admins
- DELETE /posts/{id} → Only author (if not published), admins
- DELETE /users/{id} → Only admins

**Implementation:**

- How to check user role?
- How to check ownership?
- How to handle multiple permission checks?
- How to make this code DRY?

### Part 5: Security Testing

**Task:** Test authentication/authorization thoroughly

**Test categories:**

- Unit tests: Authorization logic
- Integration tests: Token flow
- Security tests: Breach scenarios

**Test scenarios:**

- [ ] Valid token grants access
- [ ] Expired token rejected
- [ ] Modified token rejected
- [ ] No token rejected
- [ ] User can't change role in token
- [ ] User can't access other user's data
- [ ] Non-admin can't access admin endpoints
- [ ] Logout invalidates refresh token
- [ ] Can't use old refresh token

### Part 6: Documentation

**Task:** Document for other developers

**Document:**

- How to log in (API flow)
- How to use tokens in requests
- What tokens look like
- What happens when token expires
- How to refresh
- How to logout
- Error messages and what they mean
- Security considerations

### Deliverables Summary

1. **Authentication Strategy Document**
2. **Authorization Architecture Document**
3. **Security Audit Report**
4. **Implementation with:**
    - Authentication endpoints (login, refresh, logout)
    - Token generation and validation
    - Authorization checks
    - Security tests
5. **API Documentation** (updated for auth)

### Evaluation Checklist

**Conceptual Understanding:**

- [ ] Can explain when to use JWT vs sessions
- [ ] Can identify security vulnerabilities
- [ ] Can design appropriate role hierarchy
- [ ] Can explain OAuth2 purpose (even if not implementing)
- [ ] Can describe token lifecycle

**Implementation:**

- [ ] Authentication works end-to-end
- [ ] Authorization enforced correctly
- [ ] Tokens validated and verified
- [ ] Tests cover security scenarios
- [ ] Documentation clear for others

---

# KEY CONCEPTS SUMMARY

## Authentication vs Authorization

- Authentication: Proves identity
- Authorization: Determines permissions
- Both needed for security

## HTTP Authentication Methods

- Basic Auth: Credentials every request (simple, insecure)
- Bearer Tokens: Token once (better)
- Sessions: Server-side memory (simple, doesn't scale)
- Tokens: Stateless (scalable)

## JWT (JSON Web Tokens)

- Format for tokens with signature
- Three parts: Header.Payload.Signature
- Signature proves token untampered
- Can be verified without database
- Token contains user info (don't put secrets)

## OAuth2

- Authorization protocol (not authentication)
- Allow third-party access without passwords
- Multiple flows for different scenarios
- Requires Authorization Server and Resource Server

## Security Best Practices

- Always use HTTPS
- Protect secret keys
- Always verify signature
- Always check expiration
- Don't put sensitive data in token
- Use appropriate storage for client type
- Short access token + revocable refresh token

---

# NEXT STEPS AFTER WEEK 7

You've now mastered core FastAPI concepts and security fundamentals. You're ready to build production applications with:

- Authentication methods appropriate to use cases
- Secure token handling
- Proper authorization patterns
- Protection against common attacks

**Upcoming weeks:**

- Week 8: Advanced authorization and security patterns
- Week 9: WebSockets and real-time features
- Week 10: Background tasks and async patterns
- Week 11: Testing and monitoring
- Week 12: Deployment and production operations

**At this point, you can build serious applications.**

---

# RESOURCES FOR DEEPER LEARNING

**Documentation:**

- JWT.io (see token structure visually)
- OAuth2 RFC 6749 (official spec)
- OpenID Connect spec (authentication layer on OAuth2)

**Conceptual Reading:**

- "OAuth2 in Action" (book)
- OWASP Authentication Cheat Sheet
- OWASP Authorization Cheat Sheet

**Tools:**

- jwt.io/debugger (see token structure)
- Postman (test with tokens)
- Browser developer tools (watch token in requests)

---

**End of Week 7 Content**

You've progressed from "how do I build an API" to "how do I secure my API properly." Security is not an afterthought—it's a design decision made from the start. 🔒

**Next week:** Advanced authorization patterns, or did you want to deepen Week 7 concepts first?