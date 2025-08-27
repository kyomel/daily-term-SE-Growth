day - 1

## HTTP Push and Pull

### Definition:

- HTTP Pull: Traditional HTTP model where the client initiates requests to the server to retrieve data. The server responds only when requested.
- HTTP Push: Server-initiated communication where the server sends data to the client without the client explicitly requesting it at that moment.

### Example:

- HTTP Pull:

```
REST API calls
GET /api/users/123
```

- HTTP Push:
  Server-Sent Events (SSE)
  WebSocket
  Push Notifications
  HTTP/2 Server Push

---

## Law of Demeter (LoD)

### Definition:

The Law of Demeter (also known as the Principle of Least Knowledge) is a design guideline that states an object should only communicate with its immediate friends and not with strangers. It promotes loose coupling by limiting the knowledge an object has about other objects.

\*\*Core Rule:"Don't talk to strangers"

### Example:

```
// GOOD: Following LoD - direct communication only
public class OrderProcessor {
    public void processOrder(Customer customer) {
        // Let customer handle its own business
        String city = customer.getCityName();
        double tax = customer.getTaxRate();
    }
}

public class Customer {
    private Address address;

    // Customer knows how to get its city name
    public String getCityName() {
        return address.getCityName();
    }

    // Customer knows how to get tax rate
    public double getTaxRate() {
        return address.getTaxRate();
    }
}
```

---

day - 2

## Space Complexity

### Definition:

Space complexity is the amount of memory an algorithm needs to run as a function of the size of its input.

### Example:

If an algorithm uses a fixed amount of memory regardless of input size, it has O(1) space complexity.
If extra space grows with input, for example, storing an array of size n, it has O(n) space complexity.

Summary:
Space complexity measures an algorithm’s memory usage as input size increases.

---

## Time Complexity

### Definition:

Time complexity measures how the running time of an algorithm increases as the size of the input grows.

### Example:

If an algorithm examines every element in a list of size n, , its time complexity is O(n).
If it uses nested loops through the list, its time complexity is O(n²).

Summary:
Time complexity describes how the execution time of an algorithm grows with input size.

---

day - 5

## The 3 Vs(Data Engineering)

### Definition:

The 3 Vs are the fundamental characteristics that define Big Data challenges in data engineering:

- Volume - The amount/size of data
- Velocity - The speed of data generation and processing
- Variety - The different types and formats of data

### Example:

- Volume

```
• Netflix: 15+ billion hours of content watched monthly
• Facebook: 4+ petabytes of data generated daily
• Walmart: 2.5 petabytes of customer transaction data per hour
```

- Velocity

```
•• Twitter: 500+ million tweets per day (real-time processing)
• Stock Trading: Millions of transactions per second
• IoT Sensors: Continuous data streams from smart devices
```

- Variety

```
• Structured: SQL databases, CSV files
• Semi-structured: JSON, XML, logs
• Unstructured: Images, videos, emails, social media posts
```

Real-World Scenario
E-commerce Platform (Amazon):

- Volume: Billions of product searches and purchases
- Velocity: Real-time recommendation updates during browsing
- Variety: User profiles, product catalogs, reviews, images, clickstreams

---

## The REDCAMEL Approach for Designing APIs

### Definition:

REDCAMEL is an approach for designing APIs and services, consisting of these steps:

RE: Requirements — Identify functional and non-functional requirements.
DC: Design Considerations — Make architectural and technology decisions.
AM: API Model — Define the API structure and data formats.
E: Evaluating Non-functional Requirements — Assess performance, scalability, security, etc.
L: Latency Budget — Allocate acceptable response times for the system.
This structured approach helps systematically design APIs aligned with service needs.

Breakdown of REDCAMEL:
RE (Requirements): Gather and analyze both functional (what the system should do) and non-functional requirements (performance, scalability, security). This step sets the foundation for all design decisions.

DC (Design Considerations): Based on requirements, decide on architectural styles (e.g., REST, gRPC), data formats (JSON, Protobuf), protocols, and their versions to best meet the needs.

AM (API Model): Define the API endpoints, request/response structures, data schemas, and interaction patterns that the service will expose.

E (Evaluating Non-functional Requirements): Evaluate how well the design meets criteria like latency, throughput, fault tolerance, and security to ensure quality.

L (Latency Budget): Allocate time limits for each component or operation in the system to meet overall latency goals, ensuring responsiveness.

This approach ensures a comprehensive and systematic design of APIs tailored to service requirements.

### Example:

```
# E-commerce API following REDCAMEL principles

# Base URL with versioning
https://api.ecommerce.com/v1

# Endpoints
GET    /users                    # List users (paginated)
POST   /users                    # Create user
GET    /users/{id}               # Get user details
PUT    /users/{id}               # Update user
DELETE /users/{id}               # Delete user
GET    /users/{id}/orders        # Get user's orders
POST   /users/{id}/orders        # Create order for user

# Response format
{
  "data": { /* actual data */ },
  "meta": {
    "timestamp": "2025-08-05T14:23:44Z",
    "version": "1.0.0",
    "request_id": "req_123456"
  },
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
```

---

day - 6

## Event-Based Concurrency

### Definition:

Event-Based Concurrency is a programming model where tasks run concurrently by reacting to events (such as user actions, messages, or I/O), allowing efficient management of multiple tasks without multiple threads.

### Example:

Node.js HTTP Server (Non-blocking, Event-Based):

```
const http = require("http");

http.createServer((req, res) => {
  res.end("Hello, Event-Based Concurrency!");
}).listen(3000);

console.log("Server running on port 3000");
```

---

day - 7

## TLB Basic Algorithm

### Definition:

The TLB (Translation Lookaside Buffer) Basic Algorithm handles virtual-to-physical memory address translation by first checking the TLB (a fast, small cache). If the page number is found (TLB hit), the frame number is retrieved quickly; if not (TLB miss), the address is resolved via page tables, and the translation is then updated in the TLB for future accesses.

### Example:

```
VPN = (VirtualAddress & VPN_MASK) >> SHIFT
(Success, TlbEntry) = TLB_Lookup(VPN)
if (Success == True)   // TLB Hit
    if (CanAccess(TlbEntry.ProtectBits) == True)
        Offset   = VirtualAddress & OFFSET_MASK
        PhysAddr = (TlbEntry.PFN << SHIFT) | Offset
        Register = AccessMemory(PhysAddr)
    else
        RaiseException(PROTECTION_FAULT)
else                  // TLB Miss
    PTEAddr = PTBR + (VPN * sizeof(PTE))
    PTE = AccessMemory(PTEAddr)
    if (PTE.Valid == False)
        RaiseException(SEGMENTATION_FAULT)
    else if (CanAccess(PTE.ProtectBits) == False)
        RaiseException(PROTECTION_FAULT)
    else
        TLB_Insert(VPN, PTE.PFN, PTE.ProtectBits)
        RetryInstruction()
```

The algorithm the hardware follows works like this: first, extract the virtual page number (VPN) from the virtual address (Line 1 in the code snippet above), and check if the TLB holds the translation for this VPN (Line 2). If it does, we have a TLB hit, which means the TLB holds the translation. Success! We can now extract the page frame number (PFN) from the relevant TLB entry, concatenate that onto the offset from the original virtual address, and form the desired physical address (PA), and access memory (Lines 5–7), assuming protection checks do not fail (Line 4).

---

## Linus Torvald

### Profile:

Linus Torvalds is a software engineer from Finland best known as the creator of the Linux kernel. He was born on December 28, 1969, in Helsinki, Finland. Linus began developing the Linux kernel in 1991 as a personal project while he was a student at the University of Helsinki. His project eventually grew into a widely used open-source operating system, found on servers, mobile devices (such as Android), and supercomputers around the world.

Besides Linux, Torvalds is also known for developing Git, a version control system that has become the industry standard for software development.

Summary:

Name: Linus Benedict Torvalds
Born: December 28, 1969, Helsinki, Finland
Main Contributions: Creator of the Linux kernel, developer of Git
Impact: Open source community, global computer technology development

---

day - 8

## Architecture Meta-Frame

### Definition:

An Architecture Meta-Frame is a high-level conceptual framework that provides standardized patterns, principles, and guidelines for designing and organizing software architectures. It serves as a "framework for frameworks" that helps architects make consistent decisions about system structure, component relationships, and design patterns across different projects.

Architecture refers to the fundamental structure and organization of a system, defining its components, their relationships, and how they interact to fulfill requirements.

Architecture is the design framework of software systems shaped by architecture drivers, guiding the selection of architectural styles to meet significant system considerations.

### Example:

- TOGAF (The Open Group Architecture Framework)

```
Enterprise Architecture Meta-Frame:
  - Business Architecture
  - Data Architecture
  - Application Architecture
  - Technology Architecture
```

- Clean Architecture Meta-Frame

```
// Layered dependency structure
interface CleanArchitectureMetaFrame {
  entities: CoreBusinessLogic;
  useCases: ApplicationLogic;
  adapters: InterfaceAdapters;
  frameworks: ExternalFrameworks;
}
```

- Microservices Meta-Frame

```
Service Design Patterns:
  - Domain-Driven Design boundaries
  - API Gateway pattern
  - Circuit Breaker pattern
  - Event Sourcing pattern
  - CQRS pattern
```

- Cloud-Native Meta-Frame

```
- 12-Factor App principles
- Container orchestration patterns
- Service mesh architecture
- Observability patterns
```

---

day - 11

## Variable Shadowing

### Definition:

Variable shadowing is a technique in which a variable declared within a certain scope has the same name as a variable declared in an outer scope. This is also known as masking. This outer variable is said to be shadowed by the inner variable, while the inner variable is said to mask the outer variable.

### Example:

```rust
fn main() {
  let outer_variable = 112;
  { // start of code block
        let inner_variable = 213;
        println!("block variable: {}", inner_variable);
        let outer_variable = 117;
        println!("block variable outer: {}", outer_variable);
  } // end of code block
    println!("outer variable: {}", outer_variable);
}
```

---

day - 12

## The Three-Schema Architecture

### Definition:

The Three-Schema Architecture is a database design framework that separates database systems into three distinct levels of abstraction to achieve data independence and system flexibility.

| Schema Level      | Purpose                    | Audience                 |
| ----------------- | -------------------------- | ------------------------ |
| Internal Schema   | Physical storage structure | Database administrators  |
| Conceptual Schema | Logical database structure | Database designers       |
| External Schema   | User-specific views        | End users & applications |

### Example:

- - University Database System

Internal Schema:

```
-- Storage details, indexing, file organization
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
) ENGINE=InnoDB
PARTITION BY HASH(student_id) PARTITIONS 4;

-- B-tree index on student_id
CREATE INDEX idx_student_name ON STUDENT(name) USING BTREE;
```

Conceptual Schema:

```
ENTITIES:
- Student (student_id, name, email, enrollment_date)
- Course (course_id, title, credits, department)
- Enrollment (student_id, course_id, grade, semester)

RELATIONSHIPS:
- Student ←→ Enrollment ←→ Course (Many-to-Many)
```

External Schema:

```
-- Student Portal View
CREATE VIEW StudentPortal AS
SELECT s.name, c.title, e.grade
FROM Student s
JOIN Enrollment e ON s.student_id = e.student_id
JOIN Course c ON e.course_id = c.course_id;

-- Faculty View
CREATE VIEW FacultyView AS
SELECT c.title, COUNT(e.student_id) as enrolled_count
FROM Course c
LEFT JOIN Enrollment e ON c.course_id = e.course_id
GROUP BY c.course_id, c.title;
```

---

day - 13

## The "No Mapping" Strategy

### Definition:

The "No Mapping" Strategy in hexagonal architecture means using the same domain model objects directly across different layers (domain, application, and infrastructure) without creating separate DTOs or mapping between them. This reduces complexity and boilerplate code but may couple the domain model more tightly to external systems.

### Example:

Suppose you have a User entity in your domain:

```
type User struct {
    ID    string
    Name  string
    Email string
}
```

With the "No Mapping" strategy, you’d:

- Use this User struct directly in your application service.
- Pass it directly to your database repository implementation.
- Return it directly in your API response.

So your repository might look like:

```
func (r *UserRepository) FindByID(id string) (*User, error) {
    // Directly query the DB and scan into domain struct
}
```

And your handler:

```
func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    user, _ := h.repo.FindByID("123")
    json.NewEncoder(w).Encode(user) // no mapping to a DTO
}
```

If you’d like, I can also make you a short pros & cons table so you see when to use and avoid this strategy.

---

day - 14

## Monotonically Increasing

### Definition:

A sequence or function is monotonically increasing if it never decreases
as you move from left to right (or from one term to the next).
In other words, each value is greater than or equal to the previous value.

Mathematically, for a sequence $a_1, a_2, a_3, \ldots, a_n$,
it's monotonically increasing if $a_1 \le a_2 \le a_3 \le \ldots \le a_n$.

### Example:

Your daily step count over a week:

- Monday: 5,000 steps
- Tuesday: 5,000 steps
- Wednesday: 6,200 steps
- Thursday: 6,200 steps
- Friday: 7,800 steps
- Saturday: 8,500 steps
- Sunday: 8,500 steps

This sequence is monotonically increasing because each day's count is $\ge$ the previous day's count.

---

## Twitter Snowflake

### Definition:

A Twitter Snowflake is a unique 64-bit ID system used by Twitter (now X) to generate billions of unique identifiers for tweets, users, and other objects. It's called "Snowflake" because like real snowflakes, each ID is guaranteed to be unique.

**How It Works:**

The 64-bit ID is split into parts:

- Timestamp (41 bits): When it was created
- Machine ID (10 bits): Which server generated it
- Sequence (12 bits): Counter for IDs created in the same millisecond

### Example:

A Twitter Snowflake ID might look like: 1234567890123456789

When you see a tweet URL like:

```
https://twitter.com/user/status/1234567890123456789
```

That long number (1234567890123456789) is the Snowflake ID for that specific tweet.

**Why It's Useful**

- Roughly sortable by time: Newer tweets have higher IDs
- Globally unique: No two tweets will ever have the same ID
- High performance: Can generate millions of IDs per second across multiple servers

**Real-World Impact**
This system allows Twitter to handle billions of tweets while ensuring every single one has a unique identifier, making it possible to link, share, and reference any tweet ever posted!

---

day - 15

## The Halting Problem

### Definition:

The Halting Problem is a fundamental question in computer science: "Given any computer program and its input, can we write another program that determines whether the first program will eventually stop running (halt) or run forever (infinite loop)?"

The answer is NO - it's mathematically impossible to create such a universal "halt detector."

### Example:

- Program A:

```
count = 1
while count <= 10:
    print(count)
    count += 1
```

This clearly halts after printing 1-10.

- Program B:

```
while True:
    print("Hello")
```

This obviously runs forever.

- Program C:

```
n = 3
while n != 1:
    if n % 2 == 0:
        n = n // 2
    else:
        n = 3 * n + 1
```

This is the famous Collatz conjecture - mathematicians still don't know if it halts for all starting numbers.

**Why It Matters**

- Software testing: We can't automatically detect all infinite loops
- Theoretical limits: Some problems are fundamentally unsolvable
- AI safety: We can't predict if an AI system will behave as expected in all cases

---

day - 18

## Leaderless Replication

### Definition:

Leaderless Replication is a database strategy where there's no single "master" server controlling writes. Instead, any server (replica) can accept both read and write operations directly from clients. All replicas are equal - no boss, no followers.

**How It Works**

- Any replica can handle writes
- Changes are propagated to other replicas asynchronously
- Uses techniques like quorum voting to ensure consistency
- Conflict resolution handles simultaneous writes

### Example:

Imagine a shared grocery list app used by your family:

\*\*Traditional (Leader-based):

- Only Mom's phone can add items
- Everyone else must ask Mom to update the list
- If Mom's phone dies, no one can add groceries

\*\*Leaderless:

- Anyone can add items from their phone
- Dad adds "milk" at 2 PM
- Sister adds "bread" at 2:01 PM
- Both changes sync to everyone's phones
- If there's a conflict (both add "apples"), the system resolves it automatically

**Real-World Examples**

- Amazon DynamoDB: Global database with multiple regions
- Cassandra: NoSQL database used by Netflix, Uber
- Riak: Distributed key-value store

Benefits

- High availability: No single point of failure
- Better performance: Writes can happen anywhere
- Global scale: Users write to nearest replica

---

day - 19

## The PACELC Theorem

### Definition:

The PACELC Theorem is an extension of the famous CAP theorem that describes trade-offs in distributed systems. It states:

"In case of network Partition, choose between Availability and Consistency; Else (when network is fine), choose between Latency and Consistency."

P = Partition (network failure)
A = Availability
C = Consistency
E = Else (normal operation)
L = Latency
C = Consistency

### Example:

Think of a multi-location bank system with branches in New York and London:

During Network Partition (PA or PC):
Network cable between cities gets cut:

Choose Availability (PA):

Both branches stay open and accept transactions
Risk: Someone might withdraw money in both cities, overdrawing account
Choose Consistency (PC):

System shuts down to prevent conflicts
Risk: Customers can't access their money
During Normal Operation (EL or EC):
Network is working fine:

Choose Latency (EL):

Show account balance immediately from local cache
Risk: Balance might be slightly outdated (eventual consistency)
Choose Consistency (EC):

Always check with all locations before showing balance
Risk: Takes longer to display account information

---

day - 20

## Object-Oriented Analysis and Design (OOAD)

## Definition:

Object-Oriented Analysis and Design (OOAD) is a software development approach that models real-world problems using objects (things) and their interactions. It breaks down complex systems into manageable, reusable components that mirror how we naturally think about the world.

Analysis = Understanding the problem and requirements
Design = Creating a solution using objects and their relationships

Core Principles

- Encapsulation: Bundle data and methods together
- Inheritance: Create new classes based on existing ones
- Polymorphism: Same interface, different behaviors
- Abstraction: Hide complex details, show only essentials

## Example:

Let's design a Library Management System

**Analysis (What do we need?)**

- Track books, members, and borrowing
- Members can borrow and return books
- System tracks due dates and fines

**Design (How do we solve it?)**

- Use classes for Book, Member, and Borrowing
- Implement methods for borrowing and returning
- Use inheritance for shared behavior
- Use encapsulation to hide implementation details

```
class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
        self.is_available = True

    def borrow(self):
        self.is_available = False

    def return_book(self):
        self.is_available = True

class Member:
    def __init__(self, name, member_id):
        self.name = name
        self.member_id = member_id
        self.borrowed_books = []

    def borrow_book(self, book):
        if book.is_available:
            book.borrow()
            self.borrowed_books.append(book)

class Library:
    def __init__(self):
        self.books = []
        self.members = []

    def add_book(self, book):
        self.books.append(book)
```

---

day - 21

## Adaptive Replacement Cache (ARC)

### Definition:

Adaptive Replacement Cache (ARC) is a smart caching algorithm that automatically balances between keeping recently used items and frequently used items. It "learns" from access patterns and adjusts itself to minimize cache misses without any manual tuning.

**Key Features**

- Self-tuning: Automatically adjusts between recency and frequency
- Ghost lists: Remembers recently evicted items to make better decisions
- Adaptive: Changes behavior based on workload patterns

### Example:

- Music Streaming App
  Imagine your music app cache that holds 6 songs:

```
Initial State - Playing your usual favorites:

Recent (T1): [Song A, Song B]
Frequent (T2): [Song C, Song D, Song E, Song F]
```

Scenario 1: You discover new music and play many new songs

```
Access pattern: New songs G, H, I, J...

ARC learns: "User is exploring new music, prioritize recent songs"
Adapts to:
Recent (T1): [Song G, Song H, Song I, Song J]  ← Grows
Frequent (T2): [Song E, Song F]                ← Shrinks
```

Scenario 2: You go back to your old favorites

```
Access pattern: Songs C, D, E, F repeatedly...

ARC learns: "User prefers familiar songs, prioritize frequent ones"
Adapts to:
Recent (T1): [Song G]                          ← Shrinks
Frequent (T2): [Song C, Song D, Song E, Song F, Song H]  ← Grows
```

Why ARC is Special

- Traditional LRU: Only considers recent usage → Would miss your favorite songs
- Traditional LFU: Only considers frequency → Would miss new discoveries
- ARC: Automatically finds the perfect balance for YOUR listening habits

---

day - 22

## Gossip Protocol

### Definition:

Gossip Protocol is a communication method where nodes in a distributed system spread information by randomly sharing it with a few neighbors at regular intervals, similar to how rumors spread in social networks. Each node periodically "gossips" with others, ensuring information eventually reaches everyone without needing a central coordinator.

**Key Features**

- Decentralized: No single point of failure
- Eventually consistent: All nodes eventually get the same information
- Fault-tolerant: Works even if some nodes fail
- Scalable: Efficient even with thousands of nodes

### Example:

- Database Cluster
  Traditional Approach (Centralized):

```
Master Server → broadcasts to all nodes
Problem: If master fails, no one gets updates
```

Gossip Protocol Approach:

```
Node B detects A is down
Round 1: Node B → tells Node C, Node D
Round 2: Node C → tells Node E, Node D → tells Node E
Round 3: All nodes know Node A is down
```

---

day - 25

## Memento Design Pattern

### Definition:

Memento Design Pattern is a behavioral design pattern that allows you to save and restore an object's previous state without exposing its internal structure. It's like creating "snapshots" or "save points" that you can return to later, commonly used for undo functionality.

**Key Features**

- Originator: The object whose state you want to save
- Memento: The saved state snapshot
- Caretaker: Manages when to save/restore states

### Example:

- Text Editor with Undo

```
// Memento - stores the state
class TextMemento {
    private String content;

    public TextMemento(String content) {
        this.content = content;
    }

    public String getContent() { return content; }
}

// Originator - object whose state we want to save
class TextEditor {
    private String content = "";

    public void write(String text) {
        content += text;
    }

    public String getContent() { return content; }

    // Save current state
    public TextMemento save() {
        return new TextMemento(content);
    }

    // Restore previous state
    public void restore(TextMemento memento) {
        content = memento.getContent();
    }
}

// Caretaker - manages save points
class EditorHistory {
    private Stack<TextMemento> history = new Stack<>();

    public void backup(TextEditor editor) {
        history.push(editor.save());
    }

    public void undo(TextEditor editor) {
        if (!history.isEmpty()) {
            editor.restore(history.pop());
        }
    }
}
```

[Link-1](https://refactoring.guru/design-patterns/memento)
[Link-2](https://www.geeksforgeeks.org/memento-design-pattern/)

---

day - 26

## Bloom Filters

### Definition:

Bloom Filter is a space-efficient probabilistic data structure that tells you if an element is "definitely not in a set" or "possibly in a set." It can have false positives (says yes when it should say no) but never false negatives (if it says no, the item is definitely not there).

**Key Features**

- Space efficient: Uses much less memory than storing actual data
- Fast lookups: Constant time O(1) operations
- Probabilistic: Can give false positives, but never false negatives
- No deletions: Can't remove items once added

### Example:

- Google Chrome

```
User visits: badsite.com
↓
Check Bloom Filter locally (instant)
↓
Filter says "NO" → Safe to visit ✓
Filter says "YES" → Contact Google servers to double-check
```

---

day - 27

## HSTS (HTTP Strict Transport Security)

### Definition:

HSTS (HTTP Strict Transport Security) is a web security mechanism that forces browsers to only communicate with a website over secure HTTPS connections. Once enabled, it prevents users from accidentally visiting the insecure HTTP version of a site, even if they type "http://" in the address bar.

**Key Features**

- Forces HTTPS: Automatically redirects HTTP to HTTPS
- Prevents downgrade attacks: Stops attackers from forcing HTTP connections
- Browser enforcement: Works at the browser level
- Time-based: Policy expires after a specified period

### Example:

- Online Banking Website

```
User types: http://mybank.com
↓
Browser automatically converts to: https://mybank.com
↓
Secure HTTPS connection established ✓
```

---
