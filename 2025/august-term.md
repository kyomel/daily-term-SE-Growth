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
