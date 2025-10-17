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

day - 14

## Grouped-Query Attention (GQA)

### Definition:
Grouped-Query Attention (GQA) is an optimization technique for transformer models that reduces memory usage and computational cost by sharing key-value (KV) pairs across multiple query heads. Instead of having separate KV pairs for each attention head, GQA groups multiple query heads to share the same KV pairs, creating a middle ground between Multi-Head Attention (MHA) and Multi-Query Attention (MQA).

**Key Properties:**
- Shared KV pairs: Multiple query heads share same key-value computations
- Memory efficient: Reduces KV cache size during inference
- Quality preservation: Better than MQA, close to full MHA performance
- Flexible grouping: Can configure different group sizes
- Inference speedup: Faster generation with less memory bandwidth

### Example:
Language Model Inference:
```
class LanguageModelWithGQA:
    def __init__(self, vocab_size=50000, hidden_size=4096, num_layers=32):
        self.num_layers = num_layers
        self.layers = [
            GroupedQueryAttention(
                hidden_size=hidden_size,
                num_heads=32,
                num_kv_heads=8  # 4:1 ratio
            ) for _ in range(num_layers)
        ]
        self.kv_cache = {}  # Store past key-values for fast generation
    
    def generate_text(self, prompt, max_length=100):
        tokens = self.tokenize(prompt)
        generated = []
        
        for step in range(max_length):
            # Forward pass with KV caching
            logits, new_cache = self.forward(tokens, use_cache=True)
            
            # Sample next token
            next_token = self.sample(logits)
            generated.append(next_token)
            
            # Update cache and continue with just the new token
            self.update_cache(new_cache)
            tokens = [next_token]  # Only process new token
            
        return self.detokenize(generated)
    
    def forward(self, tokens, use_cache=False):
        x = self.embed(tokens)
        new_cache = []
        
        for i, layer in enumerate(self.layers):
            past_kv = self.kv_cache.get(i) if use_cache else None
            x, current_kv = layer(x, past_key_value=past_kv)
            new_cache.append(current_kv)
        
        logits = self.lm_head(x)
        return logits, new_cache
    
    def memory_usage_comparison(self, seq_length=2048):
        # GQA memory per layer
        gqa_mem = 8 * seq_length * 128 * 2 * 4  # 8 KV heads
        
        # Traditional MHA memory per layer  
        mha_mem = 32 * seq_length * 128 * 2 * 4  # 32 KV heads
        
        # Total for all layers
        total_gqa = gqa_mem * self.num_layers / (1024**3)  # GB
        total_mha = mha_mem * self.num_layers / (1024**3)  # GB
        
        print(f"GQA total KV cache: {total_gqa:.2f} GB")
        print(f"MHA total KV cache: {total_mha:.2f} GB") 
        print(f"Memory savings: {total_mha - total_gqa:.2f} GB")
        
        return total_gqa, total_mha

# Example: Llama2-70B style model
model = LanguageModelWithGQA(hidden_size=8192, num_layers=80)

# For 2048 token context:
gqa_mem, mha_mem = model.memory_usage_comparison(2048)
# Output:
# GQA total KV cache: 6.71 GB
# MHA total KV cache: 26.84 GB
# Memory savings: 20.13 GB (4x reduction!)
```

---

day - 15

## Progressive Scaling Patterns

### Definition:
Progressive Scaling Patterns are architectural strategies that allow systems to grow incrementally from simple, single-instance setups to complex, distributed systems without requiring complete rewrites. These patterns enable businesses to start small and scale gradually as demand increases, evolving the architecture step-by-step while maintaining functionality and minimizing risk.

**Key Properties:**
- Incremental growth: Scale in manageable steps, not giant leaps
- Backward compatibility: Each stage works with previous components
- Risk mitigation: Gradual changes reduce deployment risks
- Cost optimization: Pay for complexity only when needed
- Smooth transitions: No dramatic architectural overhauls

**Evolution Stages:**
Monolith → 2. Modular Monolith → 3. Microservices → 4. Distributed Systems

### Example
Stage 1: Simple Monolith (MVP - 100 users)
```
// Single Node.js application
const express = require('express');
const sqlite3 = require('sqlite3');

class EcommerceApp {
  constructor() {
    this.app = express();
    this.db = new sqlite3.Database('ecommerce.db');
    this.setupRoutes();
  }
  
  setupRoutes() {
    // All functionality in one app
    this.app.get('/products', this.getProducts.bind(this));
    this.app.post('/orders', this.createOrder.bind(this));
    this.app.get('/users/:id', this.getUser.bind(this));
    this.app.post('/payments', this.processPayment.bind(this));
  }
  
  async getProducts(req, res) {
    // Simple database query
    const products = await this.queryDb('SELECT * FROM products');
    res.json(products);
  }
  
  async createOrder(req, res) {
    const { userId, productIds, paymentInfo } = req.body;
    
    // Everything happens in one transaction
    await this.db.run('BEGIN TRANSACTION');
    try {
      const orderId = await this.insertOrder(userId, productIds);
      await this.updateInventory(productIds);
      await this.processPayment(paymentInfo);
      await this.sendOrderEmail(userId, orderId);
      await this.db.run('COMMIT');
      
      res.json({ orderId, status: 'success' });
    } catch (error) {
      await this.db.run('ROLLBACK');
      res.status(500).json({ error: error.message });
    }
  }
}

// Simple deployment
const app = new EcommerceApp();
app.listen(3000, () => console.log('E-commerce running on port 3000'));

// Pros: Simple, fast development, easy debugging
// Cons: Single point of failure, hard to scale specific parts
```

Stage 2: Load Balanced Monolith (1,000 users)
```
// Add load balancer + multiple instances
// nginx.conf
upstream backend {
    server app1:3000;
    server app2:3000;
    server app3:3000;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
    }
}

// Separate database
class EcommerceApp {
  constructor() {
    this.app = express();
    // Move to external database
    this.db = new PostgreSQL({
      host: 'postgres-server',
      database: 'ecommerce',
      pool: { min: 5, max: 20 }
    });
    
    // Add session store for multiple instances
    this.sessionStore = new RedisStore({
      host: 'redis-server'
    });
  }
  
  // Same application logic, but now scalable
}

// docker-compose.yml for easy scaling
version: '3.8'
services:
  app:
    build: .
    deploy:
      replicas: 3  # Easy to increase
  
  postgres:
    image: postgres:13
    
  redis:
    image: redis:7
    
  nginx:
    image: nginx
    ports:
      - "80:80"

// Pros: Can handle more load, database separated
// Cons: Still monolithic code, scaling is all-or-nothing
```

Stage 3: Modular Monolith (10,000 users)
```
// Separate modules within same codebase
class EcommerceApp {
  constructor() {
    this.app = express();
    
    // Modular architecture
    this.productService = new ProductService(this.db);
    this.orderService = new OrderService(this.db);
    this.userService = new UserService(this.db);
    this.paymentService = new PaymentService(this.db);
    this.inventoryService = new InventoryService(this.db);
    
    this.setupRoutes();
  }
  
  setupRoutes() {
    // Route to appropriate service
    this.app.use('/products', this.createProductRoutes());
    this.app.use('/orders', this.createOrderRoutes());
    this.app.use('/users', this.createUserRoutes());
    this.app.use('/payments', this.createPaymentRoutes());
  }
  
  createOrderRoutes() {
    const router = express.Router();
    
    router.post('/', async (req, res) => {
      const { userId, productIds, paymentInfo } = req.body;
      
      try {
        // Services communicate through method calls
        const user = await this.userService.getUser(userId);
        const products = await this.productService.getProducts(productIds);
        const inventory = await this.inventoryService.checkAvailability(productIds);
        
        if (!inventory.available) {
          return res.status(400).json({ error: 'Insufficient inventory' });
        }
        
        const order = await this.orderService.createOrder(user, products);
        await this.inventoryService.reserveItems(productIds);
        await this.paymentService.processPayment(paymentInfo);
        
        res.json({ orderId: order.id, status: 'success' });
      } catch (error) {
        res.status(500).json({ error: error.message });
      }
    });
    
    return router;
  }
}

// Each service is a separate class with clear boundaries
class OrderService {
  constructor(database) {
    this.db = database;
  }
  
  async createOrder(user, products) {
    return await this.db.orders.create({
      userId: user.id,
      items: products,
      total: products.reduce((sum, p) => sum + p.price, 0),
      status: 'pending'
    });
  }
  
  async getOrder(orderId) {
    return await this.db.orders.findById(orderId);
  }
}

// Pros: Clear separation of concerns, easier to test modules
// Cons: Still deployed as one unit, shared database
```

Stage 4: Microservices (100,000+ users)
```
// Separate applications for each service

// Product Service (products-service/)
class ProductService {
  constructor() {
    this.app = express();
    this.db = new MongoDB({ collection: 'products' });
    this.cache = new Redis();
    this.setupRoutes();
  }
  
  setupRoutes() {
    this.app.get('/products', async (req, res) => {
      // Check cache first
      const cached = await this.cache.get('products:all');
      if (cached) {
        return res.json(JSON.parse(cached));
      }
      
      const products = await this.db.find({});
      await this.cache.setex('products:all', 300, JSON.stringify(products));
      res.json(products);
    });
    
    this.app.get('/products/:id', async (req, res) => {
      const product = await this.db.findById(req.params.id);
      res.json(product);
    });
  }
}

// Order Service (orders-service/)
class OrderService {
  constructor() {
    this.app = express();
    this.db = new PostgreSQL({ table: 'orders' });
    this.eventBus = new EventBus(); // For communication
    this.setupRoutes();
  }
  
  setupRoutes() {
    this.app.post('/orders', async (req, res) => {
      const { userId, productIds, paymentInfo } = req.body;
      
      try {
        // Make HTTP calls to other services
        const user = await this.callUserService(userId);
        const products = await this.callProductService(productIds);
        const inventory = await this.callInventoryService(productIds);
        
        if (!inventory.available) {
          return res.status(400).json({ error: 'Insufficient inventory' });
        }
        
        // Create order
        const order = await this.createOrder(user, products);
        
        // Publish events for other services
        await this.eventBus.publish('order.created', {
          orderId: order.id,
          userId,
          productIds,
          total: order.total
        });
        
        res.json({ orderId: order.id, status: 'pending' });
      } catch (error) {
        res.status(500).json({ error: error.message });
      }
    });
  }
  
  async callUserService(userId) {
    const response = await fetch(`http://user-service:3001/users/${userId}`);
    return await response.json();
  }
  
  async callProductService(productIds) {
    const response = await fetch(`http://product-service:3002/products`, {
      method: 'POST',
      body: JSON.stringify({ ids: productIds }),
      headers: { 'Content-Type': 'application/json' }
    });
    return await response.json();
  }
}

// Event-driven communication
class EventBus {
  constructor() {
    this.kafka = new KafkaClient();
  }
  
  async publish(topic, data) {
    await this.kafka.send({
      topic,
      messages: [{ value: JSON.stringify(data) }]
    });
  }
  
  async subscribe(topic, handler) {
    await this.kafka.subscribe({ topic });
    await this.kafka.run({
      eachMessage: async ({ message }) => {
        const data = JSON.parse(message.value.toString());
        await handler(data);
      }
    });
  }
}

// Inventory Service listens for order events
class InventoryService {
  constructor() {
    this.eventBus = new EventBus();
    this.setupEventHandlers();
  }
  
  async setupEventHandlers() {
    await this.eventBus.subscribe('order.created', async (orderData) => {
      await this.reserveItems(orderData.productIds);
      
      await this.eventBus.publish('inventory.reserved', {
        orderId: orderData.orderId,
        productIds: orderData.productIds
      });
    });
  }
}
```

Stage 5: Distributed System with Auto-scaling (1M+ users)
```
# Kubernetes deployment with auto-scaling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: product-service
  template:
    spec:
      containers:
      - name: product-service
        image: ecommerce/product-service:v1.2.0
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: product-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: product-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

---
# API Gateway for routing
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: ecommerce-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - api.ecommerce.com

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ecommerce-routes
spec:
  http:
  - match:
    - uri:
        prefix: /products
    route:
    - destination:
        host: product-service
        subset: v1
      weight: 90
    - destination:
        host: product-service
        subset: v2
      weight: 10  # Canary deployment
  - match:
    - uri:
        prefix: /orders
    route:
    - destination:
        host: order-service
```

---

day - 16

## Redlock Algorithm

### Definition
Redlock Algorithm is a distributed locking mechanism designed to create reliable locks across multiple Redis instances. It solves the problem of achieving mutual exclusion in distributed systems by requiring a majority of Redis nodes to agree on a lock, preventing the issues that can occur with single-point-of-failure locking systems.

**Key Properties:**
- Distributed consensus: Requires majority (N/2 + 1) of Redis nodes to agree
- Fault tolerance: Works even if some Redis instances fail
- Time-based expiration: Locks automatically expire to prevent deadlocks
- Clock drift consideration: Accounts for time differences between nodes
- Mutual exclusion: Only one client can hold a lock at a time

**How It Works:**
1. Acquire lock on majority of Redis instances
2. Check elapsed time is less than lock validity
3. Use the lock for critical section
4. Release lock from all instances
5. Handle failures gracefully

### Example
Payment Processing
```
class PaymentProcessor {
  constructor() {
    this.redlock = new Redlock([
      { host: 'redis-east-1' },
      { host: 'redis-east-2' },
      { host: 'redis-west-1' },
      { host: 'redis-west-2' },
      { host: 'redis-central-1' }
    ]);
  }
  
  async processPayment(userId, amount, paymentMethod) {
    const lockKey = `payment-lock:${userId}`;
    
    try {
      // Prevent duplicate payments from same user
      const lock = await this.redlock.lock(lockKey, 30000); // 30 seconds
      
      console.log(`🔒 Processing payment for user ${userId}`);
      
      // Check user balance
      const user = await this.getUserAccount(userId);
      if (user.balance < amount) {
        throw new Error('Insufficient funds');
      }
      
      // Process payment with external provider
      const paymentResult = await this.chargePaymentMethod(paymentMethod, amount);
      
      if (paymentResult.success) {
        // Update user balance
        await this.updateUserBalance(userId, user.balance - amount);
        
        // Record transaction
        await this.recordTransaction(userId, amount, paymentResult.transactionId);
        
        console.log(`💰 Payment successful: $${amount} from user ${userId}`);
        
        await this.redlock.unlock(lock);
        return { success: true, transactionId: paymentResult.transactionId };
      } else {
        await this.redlock.unlock(lock);
        return { success: false, error: paymentResult.error };
      }
      
    } catch (error) {
      console.error(`❌ Payment failed for user ${userId}:`, error.message);
      return { success: false, error: error.message };
    }
  }
  
  async processRefund(transactionId) {
    const lockKey = `refund-lock:${transactionId}`;
    
    try {
      const lock = await this.redlock.lock(lockKey, 30000);
      
      // Ensure refund isn't processed twice
      const transaction = await this.getTransaction(transactionId);
      if (transaction.refunded) {
        throw new Error('Transaction already refunded');
      }
      
      // Process refund
      await this.processRefundWithProvider(transaction);
      await this.markTransactionRefunded(transactionId);
      
      await this.redlock.unlock(lock);
      return { success: true };
      
    } catch (error) {
      console.error(`Refund failed:`, error.message);
      return { success: false, error: error.message };
    }
  }
}

// Multiple servers can safely process payments
const processor = new PaymentProcessor();

// Server 1 attempts payment
processor.processPayment('user123', 100, 'credit-card-token');

// Server 2 attempts same payment (will be blocked by lock)
processor.processPayment('user123', 100, 'credit-card-token');

// Result: Only one payment is processed, no double-charging!
```

---

day - 17

## Long Short Term Memory(LSTM)

### Definition:
Long Short-Term Memory (LSTM) is a type of recurrent neural network (RNN) designed to learn and remember patterns in sequential data over long periods. Unlike traditional RNNs that suffer from vanishing gradients and forget information quickly, LSTMs use special "gates" to control what information to remember, forget, or output, making them excellent for tasks involving sequences like text, time series, or speech.

**Key Properties:**
- Memory cells: Store information over long sequences
- Three gates: Forget, input, and output gates control information flow
- Gradient stability: Solves vanishing gradient problem
- Sequential processing: Processes data one step at a time
- Context awareness: Remembers relevant past information

**LSTM Gates:**
- Forget Gate: Decides what to remove from memory
- Input Gate: Decides what new information to store
- Output Gate: Decides what parts of memory to output

### Example:
Text Generation
```
import string

class TextGenerator:
    def __init__(self, text_data):
        self.chars = sorted(list(set(text_data)))
        self.char_to_idx = {ch: i for i, ch in enumerate(self.chars)}
        self.idx_to_char = {i: ch for i, ch in enumerate(self.chars)}
        
        self.vocab_size = len(self.chars)
        self.model = SimpleLSTM(
            input_size=self.vocab_size,
            hidden_size=128,
            output_size=self.vocab_size,
            num_layers=2
        )
        
    def text_to_sequences(self, text, seq_length=50):
        sequences = []
        targets = []
        
        for i in range(len(text) - seq_length):
            # Input sequence
            seq = text[i:i + seq_length]
            # Target (next character)
            target = text[i + seq_length]
            
            # Convert to one-hot encoding
            seq_encoded = self.encode_sequence(seq)
            target_encoded = self.char_to_idx[target]
            
            sequences.append(seq_encoded)
            targets.append(target_encoded)
        
        return torch.FloatTensor(sequences), torch.LongTensor(targets)
    
    def encode_sequence(self, sequence):
        encoded = []
        for char in sequence:
            one_hot = [0] * self.vocab_size
            one_hot[self.char_to_idx[char]] = 1
            encoded.append(one_hot)
        return encoded
    
    def generate_text(self, seed_text, length=100, temperature=1.0):
        """Generate text starting with seed_text"""
        self.model.eval()
        generated = seed_text
        
        # Convert seed to input sequence
        current_seq = seed_text[-50:]  # Use last 50 characters
        
        for _ in range(length):
            # Encode current sequence
            seq_encoded = self.encode_sequence(current_seq)
            input_tensor = torch.FloatTensor(seq_encoded).unsqueeze(0)
            
            # Get prediction
            with torch.no_grad():
                output = self.model(input_tensor)
                
                # Apply temperature for creativity
                output = output / temperature
                probabilities = torch.softmax(output, dim=-1)
                
                # Sample next character
                next_char_idx = torch.multinomial(probabilities, 1).item()
                next_char = self.idx_to_char[next_char_idx]
                
                generated += next_char
                current_seq = current_seq[1:] + next_char  # Slide window
        
        return generated

# Example usage
shakespeare_text = """
To be or not to be, that is the question:
Whether 'tis nobler in the mind to suffer
The slings and arrows of outrageous fortune,
Or to take arms against a sea of troubles...
"""

generator = TextGenerator(shakespeare_text)

# Train the model (simplified)
sequences, targets = generator.text_to_sequences(shakespeare_text)
# ... training code ...

# Generate new text
generated = generator.generate_text("To be or not", length=100)
print("Generated text:", generated)
# Output: "To be or not to be, that is the noble mind to suffer..."
```

---
