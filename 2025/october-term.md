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

day - 7

## NFA(Nondeterministic Finite Automaton)

### Definition:
A Nondeterministic Finite Automaton (NFA) is a computational model that can be in multiple states simultaneously and can make transitions without consuming input (epsilon transitions). Unlike a DFA, an NFA can have multiple possible transitions from a state on the same input symbol, making it "nondeterministic."

**Key Properties:**
- Multiple transitions: Can have several paths from one state on same input
- Epsilon transitions: Can move between states without reading input (ε-transitions)
- Multiple current states: Can be in several states at once
- Accepts if any path accepts: Input accepted if at least one execution path reaches an accept state
- More expressive: Easier to design than DFAs for complex patterns

**Formal Definition:**
An NFA is a 5-tuple: (Q, Σ, δ, q₀, F) where:

Q: Finite set of states
Σ: Input alphabet
δ: Transition function Q × (Σ ∪ {ε}) → P(Q) (returns set of states)
q₀: Initial state
F: Set of accept states

### Example:
Email Validation
```
# NFA for simplified email: [a-z]+@[a-z]+\.com
email_nfa = NFA(
    states={'start', 'username', 'at', 'domain', 'dot', 'com', 'accept'},
    alphabet=set('abcdefghijklmnopqrstuvwxyz@.'),
    transitions={
        # Username part (letters before @)
        ('start', letter): {'username'} for letter in 'abcdefghijklmnopqrstuvwxyz'
    } | {
        ('username', letter): {'username'} for letter in 'abcdefghijklmnopqrstuvwxyz'
    } | {
        # @ symbol
        ('username', '@'): {'at'},
        
        # Domain part (letters after @)
        ('at', letter): {'domain'} for letter in 'abcdefghijklmnopqrstuvwxyz'
    } | {
        ('domain', letter): {'domain'} for letter in 'abcdefghijklmnopqrstuvwxyz'
    } | {
        # .com ending
        ('domain', '.'): {'dot'},
        ('dot', 'c'): {'c'},
        ('c', 'o'): {'o'},
        ('o', 'm'): {'accept'}
    },
    start_state='start',
    accept_states={'accept'}
)
```

---

day - 8

## Behaviour Driven Development(BDD)

### Definition:
Behavior Driven Development (BDD) is a software development methodology that focuses on defining and testing software behavior through collaboration between developers, testers, and business stakeholders. It uses natural language specifications to describe how the software should behave from the user's perspective, creating a shared understanding of requirements.

**Key Properties:**
- Natural language: Uses plain English to describe behavior
- Collaboration focused: Bridges gap between technical and non-technical teams
- Outside-in approach: Starts with user behavior, works inward to implementation
- Living documentation: Specifications serve as both requirements and tests
- Gherkin syntax: Common format using Given-When-Then structure

**Core Concepts:**
- Given: Initial context/preconditions
- When: Action or event that triggers behavior
- Then: Expected outcome or result
- And/But: Additional conditions or outcomes

### Example:
User login to e-commerce website
```
Feature: User Login
  As a customer
  I want to log into my account
  So that I can access my personal dashboard and order history

  Scenario: Successful login with valid credentials
    Given I am on the login page
    When I enter valid email "john@example.com"
    And I enter valid password "mypassword123"
    And I click the "Login" button
    Then I should be redirected to my dashboard
    And I should see "Welcome back, John!"

  Scenario: Login with invalid credentials
    Given I am on the login page
    When I enter email "john@example.com"
    And I enter incorrect password "wrongpassword"
    And I click the "Login" button
    Then I should remain on the login page
    And I should see error message "Invalid email or password"

  Scenario: Account lockout after multiple failed attempts
    Given I am on the login page
    And I have already failed to login 2 times
    When I enter email "john@example.com"
    And I enter incorrect password "wrongpassword"
    And I click the "Login" button
    Then I should see "Account temporarily locked. Try again in 15 minutes"
    And the login form should be disabled
```
BDD Process Flow:
```
1. Discovery Workshop
   ↓
2. Write Feature Files (Gherkin)
   ↓
3. Implement Step Definitions
   ↓
4. Run Tests (Red)
   ↓
5. Implement Feature (Green)
   ↓
6. Refactor (Clean)
   ↓
7. Repeat
```

---

day - 9

## Payload Compression

### Definition:
Payload Compression is a technique that reduces the size of data being transmitted over a network by encoding it in a more space-efficient format. It decreases bandwidth usage, speeds up data transfer, and reduces costs, especially important for large datasets, APIs, and web applications.

**Key Properties:**
- Size reduction: Compresses data before transmission
- Automatic handling: Usually transparent to applications
- Multiple algorithms: gzip, deflate, brotli, etc.
- Trade-off: CPU time for bandwidth savings
- Reversible: Original data perfectly reconstructed

### How It Works:
1. Client requests data with compression preference
2. Server compresses response using specified algorithm
3. Data transmitted in compressed format
4. Client decompresses and uses original data

### Example:
HTTP Compression Example:
```
// Client Request
GET /api/users HTTP/1.1
Host: api.example.com
Accept-Encoding: gzip, deflate, br

// Server Response
HTTP/1.1 200 OK
Content-Type: application/json
Content-Encoding: gzip
Content-Length: 420
Original-Content-Length: 1247

[compressed binary data]
```

---

day - 10

## Speculative Decoding

### Definition:
Speculative Decoding is an optimization technique for large language models (LLMs) that speeds up text generation by using a smaller, faster "draft" model to predict multiple tokens ahead, then having the larger "target" model verify and accept/reject these predictions in parallel. This reduces the number of sequential forward passes needed.

**Key Properties:**
- Two-model approach: Small draft model + large target model
- Parallel verification: Target model checks multiple tokens at once
- Mathematically equivalent: Same output distribution as normal decoding
- Speed improvement: 2-4x faster generation without quality loss
- Speculative execution: Generate candidates optimistically

**How It Works:**
1. Draft model quickly generates multiple token candidates
2. Target model evaluates all candidates in parallel
3. Accept/reject tokens based on probability comparison
4. Continue from longest accepted sequence
5. Repeat until completion

### Example:
ChatBot Response:
```
class ChatBotWithSpeculativeDecoding:
    def __init__(self):
        # Draft: Fast 7B model, Target: Accurate 70B model
        self.draft_model = load_model("llama-7b")
        self.target_model = load_model("llama-70b") 
        self.acceptance_rate = 0.75  # 75% of draft tokens accepted on average
    
    def generate_response(self, user_message):
        prompt = f"User: {user_message}\nAssistant: "
        
        response_tokens = []
        current_input = self.tokenize(prompt)
        
        while not self.is_complete(response_tokens):
            # Draft model quickly suggests next 5 tokens
            draft_sequence = self.draft_model.generate_fast(
                current_input, 
                num_tokens=5
            )
            # Example: ["I", "think", "the", "best", "approach"]
            
            # Target model evaluates all 5 tokens at once
            verification_result = self.target_model.verify_batch(
                current_input,
                draft_sequence
            )
            # Example: Accept ["I", "think", "the"], Reject ["best", "approach"]
            
            # Add accepted tokens to response
            accepted = verification_result.accepted_tokens
            response_tokens.extend(accepted)
            current_input.extend(accepted)
            
            # If no tokens accepted, target model generates one
            if len(accepted) == 0:
                fallback_token = self.target_model.generate_single(current_input)
                response_tokens.append(fallback_token)
                current_input.append(fallback_token)
        
        return self.detokenize(response_tokens)

# Performance comparison:
# Traditional: User asks question → 3.2 seconds for response
# Speculative: User asks question → 1.1 seconds for response (3x faster!)
```

---

day - 13

## Temporal Debugging Patterns

### Definition:
Temporal Debugging Patterns are debugging techniques that help developers understand and troubleshoot issues that occur across time, especially in asynchronous, distributed, or event-driven systems. These patterns capture the temporal sequence of events, state changes, and interactions to identify problems that only manifest over time or under specific timing conditions.

**Key Properties:**
- Time-aware: Focus on when events occur, not just what happens
- Sequence tracking: Monitor order of operations across time
- State evolution: Track how system state changes over time
- Async-friendly: Handle non-linear execution flows
- Historical context: Maintain timeline of past events

**Common Temporal Issues:**
- Race conditions
- Timing-dependent bugs
- Memory leaks over time
- Performance degradation
- State corruption in async systems

### Example:
Shopping cart sometimes loses items randomly
```
class TemporalCartDebugger {
  constructor() {
    this.timeline = [];
    this.stateSnapshots = [];
  }
  
  logEvent(event, data) {
    const timestamp = Date.now();
    const entry = {
      timestamp,
      event,
      data,
      stackTrace: new Error().stack,
      cartState: JSON.parse(JSON.stringify(this.cart))
    };
    
    this.timeline.push(entry);
    console.log(`[${timestamp}] ${event}:`, data);
  }
  
  addToCart(userId, productId) {
    this.logEvent('ADD_TO_CART_START', { userId, productId });
    
    // Simulate async operation
    setTimeout(() => {
      this.logEvent('ADD_TO_CART_DB_WRITE', { productId });
      this.cart.items.push(productId);
      this.logEvent('ADD_TO_CART_COMPLETE', { 
        productId, 
        cartSize: this.cart.items.length 
      });
    }, Math.random() * 100); // Random delay reveals race condition!
  }
  
  removeFromCart(userId, productId) {
    this.logEvent('REMOVE_FROM_CART_START', { userId, productId });
    
    setTimeout(() => {
      const index = this.cart.items.indexOf(productId);
      if (index > -1) {
        this.cart.items.splice(index, 1);
        this.logEvent('REMOVE_FROM_CART_COMPLETE', { 
          productId, 
          cartSize: this.cart.items.length 
        });
      }
    }, Math.random() * 50);
  }
  
  analyzeTimeline() {
    console.log('\n=== TEMPORAL ANALYSIS ===');
    
    // Find overlapping operations
    const overlaps = this.findOverlappingOperations();
    if (overlaps.length > 0) {
      console.log('⚠️ RACE CONDITIONS DETECTED:');
      overlaps.forEach(overlap => {
        console.log(`- ${overlap.op1} and ${overlap.op2} overlapped`);
      });
    }
    
    // Check for unexpected state changes
    this.detectUnexpectedStateChanges();
  }
}

// Usage reveals the bug:
const debugger = new TemporalCartDebugger();

debugger.addToCart(123, 'product1');
debugger.addToCart(123, 'product2');
debugger.removeFromCart(123, 'product1'); // Race condition!

// Output:
// [1234567890] ADD_TO_CART_START: {userId: 123, productId: 'product1'}
// [1234567891] ADD_TO_CART_START: {userId: 123, productId: 'product2'}  
// [1234567892] REMOVE_FROM_CART_START: {userId: 123, productId: 'product1'}
// [1234567935] REMOVE_FROM_CART_COMPLETE: {productId: 'product1', cartSize: 0}
// [1234567956] ADD_TO_CART_COMPLETE: {productId: 'product1', cartSize: 1}  
// [1234567978] ADD_TO_CART_COMPLETE: {productId: 'product2', cartSize: 2}
// ⚠️ RACE CONDITION: Item removed before it was fully added!
```

---
