day - 1

## Delta Encoding

### Definition:
Delta Encoding is a data compression technique that stores only the differences (deltas) between consecutive values instead of storing the absolute values. This is highly effective when data changes gradually or has patterns, as differences are typically smaller numbers that require less storage space.

**Key Properties:**
- Space efficient: Differences are usually smaller than original values
- Pattern exploitation: Works best with sequential or time-series data
- Simple concept: Store first value + sequence of changes
- Reversible: Can reconstruct original data perfectly

**How It Works:**
- Store the first value as-is (base value)
- For each subsequent value, store the difference from the previous value
- To decode: start with base value and apply deltas sequentially

### Example:
Stock Prices
```
Original prices: [$100.50, $100.75, $100.60, $101.10, $100.90]
Delta encoded:   [$100.50, +$0.25, -$0.15, +$0.50, -$0.20]

Benefits:
- Smaller numbers to store/transmit
- Easier to spot trends (mostly small changes)
- Better compression when combined with other techniques
```

---

day - 2

### Chubby

### Definition:
Chubby is a distributed lock service developed by Google that provides coarse-grained locking and reliable small-file storage for loosely-coupled distributed systems. It's designed to help coordinate activities across thousands of machines in a cluster, primarily for leader election, configuration storage, and naming services.

**Key Properties:**
- Coarse-grained locking: Held for hours/days, not seconds
- High availability: Designed for 99.95% uptime
- Small file storage: Acts like a simple file system for config data
- Advisory locks: Applications cooperate voluntarily
- Paxos-based: Uses Paxos consensus algorithm for consistency

**Architecture:**
- Chubby cell: Typically 5 replicas (servers)
- Master replica: One elected leader handles all requests
- Client library: Applications use simple API
- Lock delay: Prevents split-brain scenarios

### Example:
Web service with multiple servers needs to elect a leader for background tasks
```
Initial state:
┌─────────────────────────────────┐
│         Chubby Cell             │
│  [Replica1] [Replica2*]         │  * = Master
│  [Replica3] [Replica4]          │
│  [Replica5]                     │
└─────────────────────────────────┘

Application Servers:
[WebServer1] [WebServer2] [WebServer3]
```

---

day - 3

## ID Tokens (OpenID Connect)

### Definition:
ID Tokens are JSON Web Tokens (JWTs) used in OpenID Connect (OIDC) to provide identity information about an authenticated user. Unlike access tokens (which grant API access), ID tokens contain claims about who the user is—their identity, profile information, and authentication context.

**Key Properties:**
- JWT format: Base64-encoded, cryptographically signed
- Identity proof: Confirms user authentication occurred
- Claims-based: Contains user attributes (name, email, etc.)
- Short-lived: Typically expires in 1 hour
- Client-consumed: Meant for the application, not APIs

### Example:
User logs into a web app using Google OAuth
- User Authentication Flow
```
1. User clicks "Login with Google"
2. App redirects to Google's authorization server
3. User enters credentials on Google
4. Google redirects back with authorization code
5. App exchanges code for tokens (including ID token)
```

---

day - 6

## Sticky Session Architecture

### Definition:
Sticky Session Architecture (also called Session Affinity) is a load balancing technique that routes all requests from a specific user to the same server for the duration of their session. This ensures that session data stored in server memory remains accessible throughout the user's interaction with the application.

**Key Properties:**
- User-server binding: Each user "sticks" to one server for the duration of their session
- Session persistence: Session data stays in server memory
- Load balancer routing: Routes based on user identifier (cookie, IP, etc.)
- Stateful servers: Each server maintains user-specific state
- Simple implementation: No need for shared session storage

**How It Works:**
- User makes first request → Load balancer assigns them to a server
- Load balancer creates "affinity" (remembers the assignment)
- All subsequent requests from that user go to the same server
- Server keeps session data in local memory/cache

### Example: 
E-commerce website with shopping cart functionality
```
Initial Setup:
┌─────────────────┐
│  Load Balancer  │
│   (with sticky  │
│   session map)  │
└─────────────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌───▼───┐ ┌─────────┐
│Server A│ │Server B│ │Server C │
│        │ │        │ │         │
└────────┘ └────────┘ └─────────┘

Step 1: User's first request
User (session_id: abc123) → Load Balancer → Server A
Load Balancer remembers: abc123 → Server A

Step 2: User adds iPhone to cart
Request → Server A (stores cart: [iPhone])
Server A's memory: {abc123: {cart: [iPhone]}}

Step 3: User adds iPad to cart
Request (session_id: abc123) → Load Balancer → Server A (same server!)
Server A's memory: {abc123: {cart: [iPhone, iPad]}}

Step 4: User proceeds to checkout
Request (session_id: abc123) → Load Balancer → Server A
Server A has complete cart history: [iPhone, iPad]
```

---
