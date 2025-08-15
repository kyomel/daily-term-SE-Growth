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
