day - 1

## Dark Deployments

### Definition:

Dark Deployments (also called Dark Launches) is a deployment strategy where new features or code changes are released to production but remain hidden from regular users. The new functionality runs silently in the background, allowing teams to test performance, stability, and behavior in a real production environment without any user impact.

**Key characteristics:**

- Code deployed but invisible to users
- Tests with real production traffic
- Zero user-facing risk during testing
- Enables gradual rollout when ready
- Validates performance before full release

### Example:

Scenario: E-commerce site wants to deploy a new search algorithm

```
Traditional Deployment (Risky):

Monday: Deploy new search algorithm
↓
Users immediately see different results
↓
Bug discovered: "laptop" search returns "shoes"
↓
Customer complaints flood support
↓
Emergency rollback required
Dark Deployment (Safe):

Monday: Deploy new search algorithm (hidden)
↓
Users still see OLD search results
↓
NEW algorithm runs silently in background
↓
Compare results, measure performance
↓
Fix issues without any user impact
↓
Friday: Confident? Make it visible to users
```

---

day - 2

## Tromboning

### Definition:

Tromboning is a network inefficiency where traffic takes an unnecessarily long, looping path to reach its destination - going far away only to come back to a nearby location. The name comes from the shape of a trombone slide, where air travels out and back. This wastes bandwidth, increases latency, and adds unnecessary load on network devices.

**Key characteristics:**

- Traffic travels longer distance than necessary
- Increases network latency significantly
- Wastes bandwidth and resources
- Common in data centers, VPNs, and cloud environments
- Often caused by poor network design or security requirements

### Example:

Office Communication Problem
Scenario: Two employees in the same building want to share a file

```
❌ Tromboning (Inefficient):

Alice (Floor 1)                    Bob (Floor 2)
     │                                  ▲
     │                                  │
     ▼                                  │
┌─────────────────────────────────────────────────────┐
│                 LOCAL NETWORK                       │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │   CORPORATE     │
            │   HEADQUARTERS  │    ← 500 miles away!
            │   (Firewall)    │
            └─────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│                 LOCAL NETWORK                       │
└─────────────────────────────────────────────────────┘
                     │
                     ▼
              Bob receives file
Path Traveled: 1000+ miles round trip
Latency: 200ms
Result: Slow file transfer between neighbors!

✅ Optimal Path (No Tromboning):

Alice (Floor 1)
     │
     │  Direct local path
     │
     ▼
Bob (Floor 2)
Path Traveled: 50 feet
Latency: 2ms
Result: Fast, efficient transfer!
```

---

day - 3

## Incremental Data Ingestion

### Definition:

Incremental Data Ingestion is a data loading strategy where only new, modified, or deleted data is captured and transferred since the last ingestion cycle, rather than reloading the entire dataset every time. This approach significantly reduces processing time, resource consumption, and system load by focusing only on changes.

**Key characteristics:**

- Processes only changed data (delta)
- Requires change tracking mechanism
- Dramatically faster than full loads
- Reduces system resource usage
- Keeps data warehouse near real-time

### Example:

E-Commerce Orders Database
Scenario: Daily sync of 10 million orders to data warehouse

```
FULL INGESTION:
┌─────────────┐         ┌─────────────┐
│   SOURCE    │  ALL    │    DATA     │
│  DATABASE   │ ══════► │  WAREHOUSE  │
│ 10 Million  │ Records │             │
└─────────────┘         └─────────────┘
Time: 5 hours | Resources: High | Network: Heavy


INCREMENTAL INGESTION:
┌─────────────┐         ┌─────────────┐
│   SOURCE    │ Changes │    DATA     │
│  DATABASE   │ ──────► │  WAREHOUSE  │
│ 10 Million  │  Only   │             │
└─────────────┘ (5,000) └─────────────┘
Time: 2 mins | Resources: Low | Network: Light
Real-World Pipeline Example:
┌─────────────────────────────────────────────────────┐
│           INCREMENTAL INGESTION PIPELINE            │
└─────────────────────────────────────────────────────┘

Step 1: Check Last Sync Point
┌─────────────────────┐
│ Last Sync: 2024-01-01 12:00:00                     │
│ High Watermark ID: 1000000                          │
└─────────────────────┘
            │
            ▼
Step 2: Extract Delta
┌─────────────────────┐
│ Query: SELECT * FROM orders                         │
│ WHERE modified_at > '2024-01-01 12:00:00'          │
│ Result: 5,000 records                              │
└─────────────────────┘
            │
            ▼
Step 3: Transform
┌─────────────────────┐
│ Clean data                                          │
│ Apply business rules                                │
│ Format for warehouse                                │
└─────────────────────┘
            │
            ▼
Step 4: Load (Merge)
┌─────────────────────┐
│ INSERT new records                                  │
│ UPDATE modified records                             │
│ DELETE removed records                              │
└─────────────────────┘
            │
            ▼
Step 5: Update Checkpoint
┌─────────────────────┐
│ New Sync: 2024-01-02 12:00:00                      │
│ New High Watermark: 1005000                         │
└─────────────────────┘
```

---

day - 4

## Point of Presence (PoP)

### Definition:

Point of Presence (PoP) is a physical location where network infrastructure is deployed to provide local access points for users. It contains servers, routers, switches, and other equipment that bring content and services closer to end users, reducing latency and improving performance by minimizing the distance data must travel.

**Key characteristics:**

- Geographically distributed access points
- Contains networking and server equipment
- Reduces distance between users and content
- Improves speed and reduces latency
- Used by CDNs, ISPs, and cloud providers

### Example:

Video Streaming Service
Scenario: Netflix serving users worldwide

```
With PoPs (Distributed):

User in Tokyo                    Local PoP
   Japan                          Tokyo
     │                              │
     │         50 miles             │
     └──────────────────────────────┘

Latency: 5ms
Video buffering: None 😊
Experience: Excellent
How PoPs Work:
                    ┌─────────────────┐
                    │   ORIGIN        │
                    │   DATA CENTER   │
                    │   (California)  │
                    └────────┬────────┘
                             │
         Content distributed to PoPs worldwide
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   PoP Tokyo   │   │  PoP London   │   │ PoP São Paulo │
│   🇯🇵 Japan    │   │   🇬🇧 UK       │   │  🇧🇷 Brazil    │
│               │   │               │   │               │
│ ├─ Servers    │   │ ├─ Servers    │   │ ├─ Servers    │
│ ├─ Routers    │   │ ├─ Routers    │   │ ├─ Routers    │
│ └─ Cache      │   │ └─ Cache      │   │ └─ Cache      │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        ▼                   ▼                   ▼
   Japanese Users      European Users     Brazilian Users
```

---

day - 5

## Incremental Static Regeneration (ISR)

### Definition:

Incremental Static Regeneration (ISR) is a web development strategy that allows static pages to be updated after the site has been built, without requiring a complete rebuild. It combines the speed benefits of static sites with the freshness of dynamic content by regenerating individual pages in the background based on a specified time interval or on-demand triggers.

**Key characteristics:**

- Updates static pages without full site rebuild
- Serves cached pages instantly while regenerating
- Balances performance with content freshness
- Reduces build times for large sites
- Popularized by Next.js framework

### Example:

News Website with 10,000 Articles
Scenario: News site needs fast pages but updated content

```
Incremental Static Regeneration (ISR):

Article updated at 2:00 PM
         │
         ▼
Only that ONE page regenerates
         │
         ▼
┌─────────────────────────────────────────────────────┐
│  REGENERATING 1 PAGE...                             │
│                                                     │
│  ████████████████████████████████████  100%        │
│                                                     │
│  Time elapsed: 2 seconds                            │
└─────────────────────────────────────────────────────┘

Total time to see update: Seconds! 😊
```

---

day - 8

## Profile-First System

### Definition:

Profile-First System is an architecture approach where user profiles serve as the central foundation for all system functionality, personalization, and decision-making. Instead of building features and then adding user data, the system is designed from the ground up around rich user profiles that drive every interaction, recommendation, and experience.

**Key characteristics:**

- User profile is the core entity
- All features reference profile data
- Personalization built into every layer
- Data collection prioritizes profile enrichment
- Decisions driven by profile attributes

### Example:

Music Streaming Platform
Scenario: Building a music streaming service

```
Profile-First System:

Build Profile Foundation First:
┌─────────────────────────────────────────────────────┐
│  1. Design rich user profile schema                │
│  2. Build profile data collection                  │
│  3. Every feature connects to profile              │
│  4. Personalization is default behavior            │
└─────────────────────────────────────────────────────┘

Result: Unique experience for every user
User gets: Personalized everything from day one

Profile-First Architecture:
┌─────────────────────────────────────────────────────┐
│              USER PROFILE (Central Hub)            │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Identity:                                          │
│  ├─ Name: Sarah                                    │
│  ├─ Age: 28                                        │
│  └─ Location: Austin, TX                           │
│                                                     │
│  Music Preferences:                                 │
│  ├─ Genres: [Indie, Electronic, Jazz]             │
│  ├─ Tempo: Prefers upbeat (120+ BPM)              │
│  ├─ Era: 2010s - Present                          │
│  └─ Mood patterns: Energetic mornings, Calm nights│
│                                                     │
│  Listening Behavior:                                │
│  ├─ Peak hours: 7-9 AM, 6-10 PM                   │
│  ├─ Average session: 45 minutes                   │
│  ├─ Skip rate: 15%                                │
│  └─ Repeat behavior: High for favorites           │
│                                                     │
│  Social:                                            │
│  ├─ Friends: [12 connections]                     │
│  ├─ Shared playlists: 5                           │
│  └─ Influence score: Medium                       │
│                                                     │
└─────────────────────────────────────────────────────┘
                         │
        All features connect to profile
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    ▼                    ▼                    ▼
┌─────────┐        ┌─────────┐        ┌─────────┐
│ Search  │        │  Home   │        │ Player  │
│         │        │  Feed   │        │         │
│ Results │        │ Content │        │ Queue   │
│ ranked  │        │ curated │        │ built   │
│ by      │        │ for     │        │ around  │
│ profile │        │ profile │        │ profile │
└─────────┘        └─────────┘        └─────────┘
```

---

day - 9

## TypedArrays

### Definition:

TypedArrays are array-like objects in JavaScript that provide a mechanism for reading and writing raw binary data in memory buffers. Unlike regular arrays that can hold mixed data types, TypedArrays store fixed data types (integers, floats) in contiguous memory blocks, enabling faster performance and efficient memory usage for numerical operations.

**Key characteristics:**

- Fixed data type per array (Int8, Float32, etc.)
- Contiguous memory allocation
- Significantly faster for numerical operations
- Direct binary data manipulation
- Essential for WebGL, audio processing, file handling

### Example:

TypedArray
Scenario: Storing image pixel values (0-255)

```
// TypedArray - optimized for bytes (0-255)
const pixels = new Uint8Array([255, 128, 64, 255, 0, 128, 200, 100]);

// Benefits:
// - Each element uses exactly 1 byte
// - Type-safe
// - Much faster operations

pixels[0] = 300;      // Automatically becomes 44 (overflow handling)
pixels[1] = -1;       // Automatically becomes 255 (underflow handling)
console.log(pixels);  // Uint8Array [44, 255, 64, 255, 0, 128, 200, 100]

TypedArray Types Available:
┌─────────────────────────────────────────────────────┐
│              TYPED ARRAY TYPES                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  INTEGER TYPES:                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │ Int8Array      │ -128 to 127        │ 1 byte│   │
│  │ Uint8Array     │ 0 to 255           │ 1 byte│   │
│  │ Int16Array     │ -32768 to 32767    │ 2 bytes   │
│  │ Uint16Array    │ 0 to 65535         │ 2 bytes   │
│  │ Int32Array     │ -2B to 2B          │ 4 bytes   │
│  │ Uint32Array    │ 0 to 4B            │ 4 bytes   │
│  │ BigInt64Array  │ Very large         │ 8 bytes   │
│  │ BigUint64Array │ Very large         │ 8 bytes   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  FLOATING POINT TYPES:                              │
│  ┌─────────────────────────────────────────────┐   │
│  │ Float32Array   │ 32-bit float       │ 4 bytes   │
│  │ Float64Array   │ 64-bit float       │ 8 bytes   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  SPECIAL:                                           │
│  ┌─────────────────────────────────────────────┐   │
│  │ Uint8ClampedArray │ 0-255, clamped  │ 1 byte│   │
│  │                   │ (no overflow)   │       │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
Memory Comparison:
┌─────────────────────────────────────────────────────┐
│          MEMORY USAGE COMPARISON                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Storing 1000 numbers (0-255):                      │
│                                                     │
│  Regular Array:                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │ [255, 128, 64, ...]                         │   │
│  │                                             │   │
│  │ Memory: ~8,000+ bytes                       │   │
│  │ (Each number stored as 64-bit float)       │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Uint8Array:                                        │
│  ┌─────────────────────────────────────────────┐   │
│  │ Uint8Array([255, 128, 64, ...])             │   │
│  │                                             │   │
│  │ Memory: 1,000 bytes exactly                 │   │
│  │ (Each number stored as 8-bit integer)      │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Savings: 87.5% less memory!                       │
│                                                     │
└─────────────────────────────────────────────────────┘
How TypedArrays Work:
┌─────────────────────────────────────────────────────┐
│              TYPED ARRAY ARCHITECTURE               │
└─────────────────────────────────────────────────────┘

Step 1: ArrayBuffer (Raw Memory)
┌─────────────────────────────────────────────────────┐
│  ArrayBuffer - Raw binary data container            │
│                                                     │
│  const buffer = new ArrayBuffer(16);  // 16 bytes  │
│                                                     │
│  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐│
│  │00│00│00│00│00│00│00│00│00│00│00│00│00│00│00│00││
│  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘│
│   0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15  │
│                                                     │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
Step 2: TypedArray View (Interpret the bytes)
┌─────────────────────────────────────────────────────┐
│  Same buffer, different interpretations:            │
│                                                     │
│  As Uint8Array (16 elements, 1 byte each):          │
│  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐│
│  │ 0│ 1│ 2│ 3│ 4│ 5│ 6│ 7│ 8│ 9│10│11│12│13│14│15││
│  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘│
│                                                     │
│  As Int32Array (4 elements, 4 bytes each):          │
│  ┌────────────┬────────────┬────────────┬──────────┐│
│  │     0      │     1      │     2      │    3     ││
│  └────────────┴────────────┴────────────┴──────────┘│
│                                                     │
│  As Float64Array (2 elements, 8 bytes each):        │
│  ┌────────────────────────┬────────────────────────┐│
│  │          0             │          1             ││
│  └────────────────────────┴────────────────────────┘│
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

day - 10

## Cache Stampede

### Definition:

Cache Stampede (also called Thundering Herd or Dog-pile Effect) is a phenomenon where multiple requests simultaneously attempt to regenerate the same cached data when it expires or becomes unavailable. Instead of one request rebuilding the cache while others wait, all requests hit the backend simultaneously, potentially overwhelming the database or origin server and causing system failure.

**Key characteristics:**

- Occurs when popular cache entries expire
- Multiple simultaneous backend requests
- Can cascade into system-wide failures
- More severe with high-traffic systems
- Preventable with proper caching strategies

### Example:

Popular Product Page
Scenario: E-commerce site with cached product data

```
Proper Cache Handling (The Solution):

Cache Status: Product #123 data EXPIRED at 10:00:00

10:00:01 - 1000 users request Product #123 simultaneously
┌─────────────────────────────────────────────────────┐
│                                                     │
│  User 1: "Cache miss! I'll rebuild. Set LOCK."     │
│  User 2: "Cache locked. Wait or use stale data."   │
│  User 3: "Cache locked. Wait or use stale data."   │
│  ...                                               │
│  User 1000: "Cache locked. Wait or use stale data."│
│                                                     │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│              DATABASE SERVER                        │
│                                                     │
│  ✅ ONLY 1 QUERY                                   │
│                                                     │
│  CPU: 5% █░░░░░░░░░░░░░░░░░░░                      │
│  Connections: Normal                                │
│  Response time: 50ms                                │
│  Status: HEALTHY                                    │
│                                                     │
└─────────────────────────────────────────────────────┘

Result: Website stays UP! 😊
Visual Timeline:
┌─────────────────────────────────────────────────────┐
│              CACHE STAMPEDE TIMELINE                │
└─────────────────────────────────────────────────────┘

WITHOUT PROTECTION:
─────────────────────────────────────────────────────────
Time:     10:00:00    10:00:01              10:00:05
          │           │                      │
Cache:    EXPIRES     │                      │
          │           │                      │
Requests: │     ──────┼──────                │
          │     1000 simultaneous            │
          │     cache misses                 │
          │           │                      │
Database: │     💥 OVERLOADED 💥            CRASHED
─────────────────────────────────────────────────────────


WITH PROTECTION (Lock/Mutex):
─────────────────────────────────────────────────────────
Time:     10:00:00    10:00:01    10:00:02    10:00:03
          │           │           │           │
Cache:    EXPIRES     │           REBUILT     │
          │           │           │           │
Request 1:│     ──────┼───────────┼           │
          │     Gets lock        Updates     │
          │     Queries DB       cache       │
          │           │           │           │
Requests: │     999 requests     Get fresh   │
2-1000:   │     wait/stale       data        │
          │           │           │           │
Database: │     1 query only ✅              HEALTHY
─────────────────────────────────────────────────────────
```

---

day - 11

## Principle of Least Privilege(PoLP)

### Definition:

Principle of Least Privilege (PoLP) is a security concept where users, applications, and systems are granted only the minimum permissions necessary to perform their specific tasks - nothing more. By limiting access rights, organizations reduce the attack surface, minimize potential damage from breaches, and prevent accidental or intentional misuse of resources.

**Key characteristics:**

- Grant minimum required access only
- Permissions based on job function
- Temporary elevated access when needed
- Regular access reviews and revocation
- Applies to users, applications, and services

### Example:

Company Employee Access
Scenario: Different employees need different system access

```
With PoLP (Minimum Required Access):

┌─────────────────────────────────────────────────────┐
│              GOOD: LEAST PRIVILEGE                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  New Intern "Alex" receives:                        │
│  ┌─────────────────────────────────────────────┐   │
│  │  ✓ Blog content editor access               │   │
│  │  ✗ No database access                       │   │
│  │  ✗ No admin panel access                    │   │
│  │  ✗ No production server access              │   │
│  │  ✗ No financial records access              │   │
│  │  ✗ No customer data access                  │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Alex's actual job: Update blog posts ✅           │
│                                                     │
│  BENEFITS:                                          │
│  ✅ Can't accidentally break critical systems      │
│  ✅ No access to sensitive data                    │
│  ✅ If account hacked → minimal damage             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

day - 12

## Commercial ERP

### Definition:

Commercial ERP (Enterprise Resource Planning) is a pre-built, vendor-developed software suite that integrates and manages all core business processes - including finance, HR, manufacturing, supply chain, sales, and customer relations - into a single unified system. Organizations purchase licenses or subscriptions from commercial vendors like SAP, Oracle, or Microsoft rather than building custom solutions.

**Key characteristics:**

- Pre-built by software vendors
- Integrates multiple business functions
- Centralized database for all departments
- Licensed or subscription-based pricing
- Vendor-provided support and updates
- Industry-specific configurations available

### Example:

Manufacturing Company Operations
Scenario: Furniture company managing daily operations

```
With Commercial ERP (Unified System):

┌─────────────────────────────────────────────────────┐
│              UNIFIED ERP SYSTEM                     │
│                  (e.g., SAP)                        │
├─────────────────────────────────────────────────────┤
│                                                     │
│        ┌─────────────────────────┐                 │
│        │   CENTRAL DATABASE      │                 │
│        │   (Single Source of     │                 │
│        │    Truth)               │                 │
│        └───────────┬─────────────┘                 │
│                    │                               │
│    ┌───────────────┼───────────────┐               │
│    │       │       │       │       │               │
│    ▼       ▼       ▼       ▼       ▼               │
│ ┌──────┬──────┬──────┬──────┬──────┬──────┐       │
│ │Sales │Inven-│Finance│ HR  │Produc│Purch-│       │
│ │      │tory  │       │     │tion  │asing │       │
│ └──────┴──────┴──────┴──────┴──────┴──────┘       │
│                                                     │
│  ALL CONNECTED - Real-time data sharing!           │
│                                                     │
└─────────────────────────────────────────────────────┘

BENEFITS:
✅ Sales sees real-time inventory
✅ Finance auto-updated with every transaction
✅ Production gets orders instantly
✅ HR tracks labor costs automatically
✅ Reports generated in seconds
How ERP Connects Everything:
┌─────────────────────────────────────────────────────┐
│         CUSTOMER ORDER FLOW IN ERP                  │
└─────────────────────────────────────────────────────┘

Step 1: Customer places order
┌─────────────────────────────────────────────────────┐
│  SALES MODULE                                       │
│  ├─ Order #1234: 50 wooden chairs                  │
│  ├─ Customer: ABC Furniture Store                  │
│  └─ Delivery date: March 15                        │
│                                                     │
│  → Automatically triggers...                       │
└─────────────────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  INVENTORY  │  │ PRODUCTION  │  │  FINANCE    │
│             │  │             │  │             │
│ Check stock:│  │ Schedule:   │  │ Create:     │
│ Wood: 200ft │  │ Cut wood    │  │ Invoice     │
│ Screws: 500 │  │ Assembly    │  │ $2,500      │
│ Fabric: 30m │  │ Finishing   │  │             │
│             │  │ Due: Mar 12 │  │ Update:     │
│ Low stock   │  │             │  │ Revenue     │
│ alert! →    │  │             │  │ forecast    │
└──────┬──────┘  └─────────────┘  └─────────────┘
       │
       ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ PURCHASING  │  │     HR      │  │  SHIPPING   │
│             │  │             │  │             │
│ Auto-create:│  │ Assign:     │  │ Schedule:   │
│ PO for wood │  │ 3 workers   │  │ Pickup      │
│ PO for fabric  │ 8 hours each│  │ Mar 15      │
│             │  │             │  │             │
│ Send to     │  │ Track labor │  │ Generate    │
│ supplier    │  │ costs       │  │ shipping    │
└─────────────┘  └─────────────┘  └─────────────┘

ALL FROM ONE ORDER ENTRY!
```

---
