day - 1

## Principle of Least Surprise

### Definition:

Principle of Least Surprise (also called Principle of Least Astonishment) is a design principle stating that a system should behave in a way that users expect, minimizing confusion and unexpected outcomes.

In simple terms: Things should work the way people naturally think they work.

**Why It Matters**  
When software violates this principle, users:

- Make mistakes more often
- Feel frustrated
- Take longer to learn the system
- Lose trust in the interface

### Examples:

Good (Follows POLS)

```
# Expected behavior - delete() actually deletes
def delete(item):
    item.clear()  # Actually removes the data
    return True   # Confirms deletion

user = {"name": "Alice"}
delete(user)
# User expects: user is deleted ✓
# Reality: user is deleted ✓
```

---

day - 2

## Sieve Algorithm or Sieve of Eratosthenes

### Definition:

The Sieve Algorithm (specifically Sieve of Eratosthenes) is an ancient, efficient method for finding all prime numbers up to a given limit. It works by systematically "crossing out" multiples of each prime, leaving only primes unmarked.

Key idea: Instead of testing each number individually, eliminate all non-primes in batches.

**Why It Matters**

- Fast: Much faster than checking each number individually
- Simple: Easy to understand and implement
- Efficient: Great for finding many primes at once (up to millions)

**How It Works (Simple Steps)**

- List all numbers from 2 to n
- Start with the first unmarked number (it's prime)
- Cross out all its multiples (they're composite)
- Repeat with the next unmarked number
- Continue until done

### Example:

Find all primes up to 30:

```
Step 1: List numbers 2-30
2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30

Step 2: 2 is prime, cross out multiples of 2
2  3  ✗  5  ✗  7  ✗  9  ✗  11 ✗  13 ✗  15 ✗  17 ✗  19 ✗  21 ✗  23 ✗  25 ✗  27 ✗  29 ✗

Step 3: 3 is prime, cross out multiples of 3
2  3  ✗  5  ✗  7  ✗  ✗  ✗  11 ✗  13 ✗  ✗  ✗  17 ✗  19 ✗  ✗  ✗  23 ✗  25 ✗  ✗  ✗  29 ✗

Step 4: 5 is prime, cross out multiples of 5
2  3  ✗  5  ✗  7  ✗  ✗  ✗  11 ✗  13 ✗  ✗  ✗  17 ✗  19 ✗  ✗  ✗  23 ✗  ✗  ✗  ✗  ✗  29 ✗

Step 5: Continue with 7, 11, 13...

Final Result (primes): 2, 3, 5, 7, 11, 13, 17, 19, 23, 29
```

---

day - 5

## AutoML Frameworks

### Definition:

AutoML (Automated Machine Learning) Frameworks are software tools that automatically handle the complex, time-consuming parts of building machine learning models. They automate tasks like data preprocessing, feature engineering, algorithm selection, and hyperparameter tuning.

Key idea: You provide the data, AutoML does the hard work, and you get a trained model.

**Why It Matters**

Traditional ML requires:

- Deep expertise in algorithms
- Manual feature engineering
- Trial-and-error for best parameters
- Hours/days of experimentation

AutoML provides:

- Automated model selection
- Works for non-experts
- Faster time to deployment
- Often better results than manual tuning

### Example:

```
from autosklearn.classification import AutoSklearnClassifier

# Just 3 lines!
automl = AutoSklearnClassifier(time_left_for_this_task=3600)
automl.fit(X_train, y_train)
predictions = automl.predict(X_test)

# AutoML automatically tried 100+ model configurations!
```

Popular AutoML Frameworks
| Framework | Description | Best For |
|-----------------|-----------------------------------|------------------------------|
| Auto-sklearn | Built on scikit-learn | Tabular data, Python users |
| H2O AutoML | Enterprise-grade AutoML | Business applications |
| Google AutoML | Cloud-based, no-code | Beginners, quick prototypes |
| TPOT | Uses genetic algorithms | Research, optimization |
| AutoKeras | Deep learning focused | Image/text data |
| PyCaret | Low-code ML library | Fast experimentation |

---

day - 6

## Spot Instances

### Definition:

Spot Instances are heavily discounted cloud computing resources (virtual machines) that use spare capacity from cloud providers. They can save you up to 90% compared to regular On-Demand pricing, but come with a trade-off: the cloud provider can reclaim them at any time with short notice (30 seconds to 2 minutes).

Key idea: Cheap cloud servers that can be interrupted when the provider needs the capacity back.

**Why It Matters**
💰 Huge savings: 60-90% cheaper than regular pricing
⚡ Same performance: Identical to standard instances
⚠️ Interruptible: Can be terminated with minimal warning
🎯 Perfect for flexible workloads: Not for critical applications

**How It Works**

Cloud Provider's Perspective:

1. Has 1000 servers in data center
2. Only 600 are being used by regular customers
3. Offers remaining 400 at steep discount as "Spot Instances"
4. If regular customers need them → reclaim from Spot users

Your Perspective:

1. Rent cheap server (90% off!)
2. Run your workload
3. Might get interrupted (2-minute warning)
4. Save tons of money! 💵

### Example:

Scenario: Video Rendering Farm

```
Option 1: On-Demand Instances

Cost: $1.00/hour per server
Workload: Render 100 videos (10 hours)
Servers needed: 10
Total cost: $1.00 × 10 servers × 10 hours = $100
Reliability: ✅ Guaranteed

Option 2: Spot Instances

Cost: $0.10/hour per server (90% discount!)
Workload: Render 100 videos (10 hours)
Servers needed: 10
Total cost: $0.10 × 10 servers × 10 hours = $10
Reliability: ⚠️ Might be interrupted
Savings: $90 (90% saved!) 🎉

Result: If 2 out of 10 servers get interrupted and you restart them, you still save ~$80!
```

---

day - 7

## Telemetry Engineering

### Definition:

Telemetry Engineering is the practice of designing, implementing, and maintaining systems that automatically collect, transmit, and analyze data from remote sources (applications, servers, devices, users) to monitor performance, detect issues, and gain insights.

Key idea: Building the infrastructure that tells you what's happening in your systems, even when you can't see them directly.

**Why It Matters**
**Without telemetry engineering:**

❌ Blind to production issues
❌ Learn about problems from angry customers
❌ Guess why systems fail
❌ Can't optimize performance

**With telemetry engineering:**

✅ Real-time visibility into systems
✅ Detect issues before users notice
✅ Understand root causes quickly
✅ Data-driven optimization

The Four Pillars of Telemetry (MELT)
| Type | What It Captures | Example |
|------|-----------------|---------|
| Metrics | Numerical measurements over time | CPU: 45%, Response time: 120ms |
| Events | Discrete occurrences | User logged in, Payment failed |
| Logs | Text records of activity | "ERROR: Database connection timeout" |
| Traces | Request journey through system | API call → DB query → Cache lookup |

### Example:

Scenario: E-commerce Website

```
[System automatically detects]
Alert: 🚨 Payment API response time: 5000ms (normal: 100ms)
       Database connections: 450/500 (90% capacity)
       Error rate: 15% (normal: 0.1%)

Engineer: *checks dashboard*
Engineer: *sees database connection pool exhausted*
Engineer: *scales database in 5 minutes*
Revenue lost: $500 💰
```

---

day - 8

## Bring Your Own Language Model (BYOLM)

### Definition:

Bring Your Own Language Model (BYOLM) is a framework or platform feature that allows users to integrate their own custom-trained or third-party language models into a system, rather than being limited to pre-built models provided by the platform.

Key idea: Use your preferred AI model instead of being locked into a single provider's model.

**Why It Matters**

**Without BYOLM (Vendor Lock-in):**

❌ Stuck with one provider's models (e.g., only GPT-4)
❌ Can't use specialized models for your industry
❌ Limited control over costs and performance
❌ Dependent on vendor's pricing and availability

**With BYOLM (Flexibility):**

✅ Choose the best model for each task
✅ Use domain-specific models (medical, legal, etc.)
✅ Control costs and data privacy
✅ Switch models without rewriting code

### Example:

Scenario: Healthcare Chatbot
Company: MediCare AI builds patient support chatbots

Problem: Different use cases need different models

Solution: BYOLM platform allows mixing models

---

day - 9

## The Leaky Bucket Algorithm

### Definition:

The Leaky Bucket Algorithm is a rate-limiting technique that controls the flow of data or requests by maintaining a constant output rate, regardless of how bursty the input is. Incoming requests fill a "bucket" with finite capacity, and requests "leak out" at a fixed rate.

Key idea: Smooth out irregular bursts of traffic into a steady, predictable flow.

**Why It Matters**

**Without Rate Limiting:**

❌ Server overload from traffic spikes
❌ System crashes under heavy load
❌ Poor experience for all users
❌ DDoS vulnerabilities

**With Leaky Bucket:**

✅ Steady, predictable traffic flow
✅ Protection from bursts
✅ Fair resource allocation
✅ System stability maintained

### Example:

Imagine a literal bucket with a small hole at the bottom:

```
1. Bucket has capacity: 10 requests
2. Leak rate: 2 requests/second (constant)

Timeline:

t=0s  → 5 requests arrive
        Bucket: [■■■■■□□□□□] (5/10)
        Leak 2 → Process 2 requests

t=1s  → 8 requests arrive
        Bucket: [■■■■■■■■■■] (10/10) FULL!
        3 requests REJECTED (overflow)
        Leak 2 → Process 2 requests

t=2s  → 1 request arrives
        Bucket: [■■■■■■■□□□] (7/10)
        Leak 2 → Process 2 requests

t=3s  → 0 requests arrive
        Bucket: [■■■■■□□□□□] (5/10)
        Leak 2 → Process 2 requests

t=4s  → 0 requests arrive
        Bucket: [■■■□□□□□□□] (3/10)
        Leak 2 → Process 2 requests

t=5s  → 0 requests arrive
        Bucket: [■□□□□□□□□□] (1/10)
        Leak 1 → Process 1 request (only 1 left)

t=6s  → Bucket empty!
        Bucket: [□□□□□□□□□□] (0/10)
```

---

day - 12

## Dynamic Application Security Testing (DAST)

### Definition:

Dynamic Application Security Testing (DAST) is a security testing method that analyzes a running application from the outside (like a hacker would) to find vulnerabilities. It tests the application in its operational state by sending malicious inputs and observing how it responds, without access to the source code.

Key idea: "Black-box" testing that attacks your app while it's running to find security holes before real hackers do.

**Why It Matters**

**Without DAST:**

❌ Security flaws discovered after deployment
❌ Only find issues hackers exploit first
❌ Expensive breaches and data leaks
❌ Compliance violations

**With DAST:**

✅ Find runtime vulnerabilities early
✅ Test real-world attack scenarios
✅ No source code access needed
✅ Works on any technology stack

### Example:
Testing a Login Form
```
Normal User:
┌─────────────────────────┐
│  Login Form             │
├─────────────────────────┤
│ Username: alice         │
│ Password: ••••••        │
│         [Login]         │
└─────────────────────────┘
          ↓
    ✅ Login successful


DAST Scanner (Attacking):
┌─────────────────────────────────────────┐
│  Login Form                             │
├─────────────────────────────────────────┤
│ Username: admin' OR '1'='1              │ ← SQL injection
│ Password: anything                      │
│         [Login]                         │
└─────────────────────────────────────────┘
          ↓
    ⚠️ Logged in as admin! VULNERABILITY FOUND!

┌─────────────────────────────────────────┐
│ Username: <script>alert('XSS')</script> │ ← XSS attack
│ Password: test                          │
│         [Login]                         │
└─────────────────────────────────────────┘
          ↓
    ⚠️ Script executed! VULNERABILITY FOUND!

┌─────────────────────────────────────────┐
│ Username: alice                         │
│ Password: (empty)                       │ ← Empty field
│         [Login]                         │
└─────────────────────────────────────────┘
          ↓
    ✅ Error handled properly - No vulnerability
```

---

day - 13

## MongoBleed Vulnerability

### Definition:

MongoBleed is a critical security vulnerability in MongoDB that allows attackers to leak sensitive data from server memory through crafted queries. Similar to the infamous Heartbleed bug, it exploits improper memory handling to read data that should be inaccessible, potentially exposing passwords, API keys, user data, and other secrets stored in adjacent memory locations.

Key idea: Trick MongoDB into reading beyond intended boundaries and reveal secrets from memory.

**Why It Matters**

Impact of MongoBleed:

🔓 Leak sensitive data (passwords, tokens, PII)
💳 Expose credit card numbers and payment info
🔑 Steal API keys and authentication tokens
📊 Access other users' data from memory
🎯 No authentication required in some cases

Severity: CRITICAL (CVSS Score: 9.8/10)

### Example:
```
Normal MongoDB Query:
┌─────────────────────────────────────┐
│ Query: Find user with id="123"     │
│        Return: {name, email}       │
├─────────────────────────────────────┤
│ Memory Buffer:                     │
│ ┌──────────────────┐              │
│ │ John | john@x.com │ ← Intended  │
│ └──────────────────┘              │
└─────────────────────────────────────┘
Result: ✅ Only requested data returned


MongoBleed Attack:
┌─────────────────────────────────────────────┐
│ Query: Crafted to read extra memory        │
├─────────────────────────────────────────────┤
│ Memory Buffer:                             │
│ ┌─────────────────────────────────────────┐│
│ │ John | john@x.com | pass123 | API_KEY   ││ ← Leak!
│ │ alice@y.com | token456 | CardNum: 1234  ││ ← Leak!
│ │ admin:secretpass | session_xyz          ││ ← Leak!
│ └─────────────────────────────────────────┘│
└─────────────────────────────────────────────┘
Result: ⚠️ Sensitive data from memory leaked!
```

---

day - 14

## C4 Model

### Definition:

The C4 Model is a "maps-of-your-code" approach to software architecture. Created by Simon Brown, it helps teams describe software architecture at different levels of abstraction, much like how you would use Google Maps to zoom from a continent down to a street view.

Rather than using complex UML diagrams that often become outdated, C4 uses four hierarchical levels to make system designs easy to understand for both technical and non-technical stakeholders.

**The 4 Levels of C4**

1. Level 1: System Context Diagram
This is the "big picture." It shows your software system as a single box in the center, surrounded by its users and the other systems it interacts with.

- Audience: Everyone (Management, Business, Developers).
- Focus: External dependencies and actors.

2. Level 2: Container Diagram
"Zooming in" to the system. A Container represents a high-level technical building block, such as a web application, a mobile app, a database, or a microservice.

- Audience: Technical staff (Devs, Ops, Architects).
- Focus: High-level technology choices and how containers communicate.

3. Level 3: Component Diagram
"Zooming in" to an individual container. It breaks a container down into its internal components (e.g., a "Security Manager," "Email Controller," or "Payment Gateway").

- Audience: Developers and Architects.
- Focus: Internal logical structure of a service.

4. Level 4: Code Diagram
The finest level of detail. This usually maps to UML Class Diagrams or entity-relationship diagrams.

- Audience: Developers.
- Focus: How the code is actually implemented. (Note: Often skipped or auto-generated as it changes too quickly).

### Example:
Internet Banking System:

```
| Level | Description |
| :--- | :--- |
| **Context** | A Personal Customer uses the Banking System to view balances. The system interacts with an external Mainframe Banking System and an Email Service. |
| **Container** | Inside the Banking System, there is a Web App (React), a Mobile App (iOS/Android), an API Application (Java/Spring Boot), and a Database (PostgreSQL). |
| **Component** | Inside the API Application, there is a Sign-in Controller, a Reset Password Component, and an Accounts Summary Component. |
| **Code** | A Class Diagram showing how the Sign-in Controller uses the Security Framework classes. |
```

---

day - 15

## SAFe 5.0 Framework

### Definition:

SAFe (Scaled Agile Framework) 5.0 is a structured methodology for implementing Agile practices at enterprise scale. It provides a knowledge base of integrated principles, practices, and competencies for achieving business agility across large organizations with multiple teams working on complex projects.

**Core Components:      **

- Team Level: Individual Agile teams (5-11 people) using Scrum/Kanban
- Program Level: Agile Release Train (ART) - multiple teams working together
- Large Solution Level: Coordinating multiple ARTs for very large systems
- Portfolio Level: Strategic alignment and lean portfolio management

**Key Principles**

✓ Economic View - Make decisions based on economics
✓ Systems Thinking - Understand the big picture
✓ Assume Variability - Plan for uncertainty
✓ Build Incrementally - Deliver in small batches
✓ Base on Objectives - Align to measurable goals
✓ Visualize & Limit WIP - Make work visible, reduce overload
✓ Apply Cadence - Regular, predictable schedules
✓ Unlock Motivation - Empower people
✓ Decentralize Decisions - Speed up execution
✓ Organize Around Value - Focus on customer outcomes

### Example:
Banking Software Development

```
Without SAFe:
Mobile team, web team, and backend team work independently
Different release schedules create integration nightmares
Strategic initiatives get lost in translation
18-month project with big-bang release
With SAFe 5.0:
Portfolio Level
Strategic theme: "Improve customer digital experience by 40% in 12 months"

Epic: Digital Wallet Integration

Budget: $2M
Business outcome: Reduce payment friction
Program Level (ART)
Agile Release Train: "Digital Banking Platform"

Teams involved:
Mobile App Team (iOS/Android)
Web Frontend Team
API/Backend Team
Security Team
UX/Design Team
Program Increment (PI): 10-week cycle


Week 1-2:   PI Planning (all teams align on objectives)
Week 3-9:   Development in 2-week sprints
Week 10:    Innovation & Planning + System Demo
PI Objectives (Quarter 1):

✓ Users can add credit/debit cards to wallet
✓ P2P transfers within the bank
✓ Security compliance certification
Team Level
Mobile App Team - Sprint 1 (2 weeks):


Stories:
- As a user, I can navigate to wallet section (5 pts)
- As a user, I can scan my card to add it (8 pts)
- As a user, I see confirmation of card added (3 pts)

Daily Standup → Work on stories → Sprint Review → Retrospective
Integration
System Demo (bi-weekly):

All teams demonstrate integrated functionality
Mobile + Backend + Web working together
Stakeholders provide feedback
```

---

day - 19

## HyperLogLog

### Definition:
HyperLogLog (HLL) is a probabilistic algorithm used to estimate the cardinality (number of unique elements) in a very large dataset using minimal memory. Instead of storing all unique values, it uses statistical approximations to count distinct items with ~2% error rate while using only kilobytes of memory instead of gigabytes.

**Core Characteristics:**

- Space Efficiency: Uses ~1.5 KB to count billions of unique items
- Accuracy: Typical error rate of ±2%
- Speed: Constant time operations O(1) for add and count
- Mergeable: Can combine multiple HyperLogLog structures

**How It Works (Simplified):**

1. Hash each element → Get a binary number
2. Count leading zeros in the hash
3. Keep track of the maximum zeros seen (per bucket)
4. Estimate cardinality from these maximums

**Why this works:**

Rare events (like many leading zeros) indicate a large dataset.

**Analogy:**

- If you flip coins and see 10 heads in a row, you probably flipped a lot of times
- HyperLogLog uses this probability theory with hash values

### Example:
Tracking Daily Unique Visitors

```
The Problem
You run a popular website with 100 million daily visitors. You need to answer:

- "How many unique visitors today?"
- "How many unique visitors this week?"

HyperLogLog Approach

import hyperloglog

# Create HyperLogLog counter (uses ~1.5 KB)
hll = hyperloglog.HyperLogLog(0.01)  # 1% error rate

# Add visitors
hll.add("user_12345")
hll.add("user_67890")
hll.add("user_12345")  # Duplicate, automatically handled

# Estimate count
estimated_count = len(hll)
print(f"~{estimated_count} unique visitors")

Memory Required: ~1.5 KB (regardless of actual count!)
```

---
