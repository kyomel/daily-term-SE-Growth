day - 3

## Huffman Coding

### Definition:

Huffman Coding is a lossless data compression algorithm that assigns variable-length binary codes to characters, with more frequent characters getting shorter codes and less frequent characters getting longer codes. It builds an optimal binary tree (Huffman tree) where the path from root to each character determines its unique binary code, minimizing the total number of bits needed to represent the data.

**Key Properties:**

- Variable-length codes: Frequent characters = short codes, rare characters = long codes
- Prefix-free: No code is a prefix of another (ensures unique decoding)
- Optimal: Produces the minimum possible average code length
- Lossless: Perfect reconstruction of original data
- Greedy algorithm: Builds tree by repeatedly merging least frequent nodes

**Core Concepts:**

- Frequency analysis: Count how often each character appears
- Binary tree: Structure where each leaf is a character
- Priority queue: Used to build tree efficiently
- Bit encoding: Path from root to leaf gives character's code

### Example:

Compress the text "HELLO WORLD" efficiently

```
class HuffmanNode {
  constructor(char, freq, left = null, right = null) {
    this.char = char;
    this.freq = freq;
    this.left = left;
    this.right = right;
  }

  isLeaf() {
    return this.left === null && this.right === null;
  }
}

class HuffmanCoding {
  constructor(text) {
    this.text = text;
    this.frequencyMap = new Map();
    this.codes = new Map();
    this.tree = null;
  }

  // Step 1: Count character frequencies
  buildFrequencyMap() {
    console.log('📊 Building frequency map...');

    for (let char of this.text) {
      this.frequencyMap.set(char, (this.frequencyMap.get(char) || 0) + 1);
    }

    console.log('Character frequencies:');
    for (let [char, freq] of this.frequencyMap) {
      const displayChar = char === ' ' ? 'SPACE' : char;
      console.log(`  ${displayChar}: ${freq}`);
    }

    return this.frequencyMap;
  }

  // Step 2: Build Huffman tree using priority queue
  buildHuffmanTree() {
    console.log('\n🌳 Building Huffman tree...');

    // Create priority queue (min-heap) with leaf nodes
    const priorityQueue = [];

    for (let [char, freq] of this.frequencyMap) {
      priorityQueue.push(new HuffmanNode(char, freq));
    }

    // Sort by frequency (ascending)
    priorityQueue.sort((a, b) => a.freq - b.freq);

    console.log('Initial nodes (sorted by frequency):');
    priorityQueue.forEach(node => {
      const displayChar = node.char === ' ' ? 'SPACE' : node.char;
      console.log(`  ${displayChar}: ${node.freq}`);
    });

    // Build tree by repeatedly merging two smallest nodes
    let step = 1;
    while (priorityQueue.length > 1) {
      console.log(`\nStep ${step}: Merging two smallest nodes`);

      // Remove two nodes with smallest frequency
      const left = priorityQueue.shift();
      const right = priorityQueue.shift();

      console.log(`  Merging: ${this.nodeToString(left)} + ${this.nodeToString(right)}`);

      // Create new internal node with combined frequency
      const merged = new HuffmanNode(
        null, // Internal nodes don't have characters
        left.freq + right.freq,
        left,
        right
      );

      console.log(`  Result: Internal node with frequency ${merged.freq}`);

      // Insert back into queue in correct position
      let inserted = false;
      for (let i = 0; i < priorityQueue.length; i++) {
        if (merged.freq <= priorityQueue[i].freq) {
          priorityQueue.splice(i, 0, merged);
          inserted = true;
          break;
        }
      }
      if (!inserted) {
        priorityQueue.push(merged);
      }

      step++;
    }

    this.tree = priorityQueue[0];
    console.log('\n✅ Huffman tree built successfully!');
    return this.tree;
  }

  // Step 3: Generate codes by traversing tree
  generateCodes(node = this.tree, code = '') {
    if (!node) return;

    if (node.isLeaf()) {
      // Leaf node: assign code to character
      this.codes.set(node.char, code || '0'); // Handle single character case
    } else {
      // Internal node: traverse children
      this.generateCodes(node.left, code + '0');  // Left = 0
      this.generateCodes(node.right, code + '1'); // Right = 1
    }
  }

  // Step 4: Encode the text
  encode() {
    console.log('\n🔤 Generated Huffman codes:');

    this.generateCodes();

    // Display codes sorted by frequency (most frequent first)
    const sortedCodes = Array.from(this.codes.entries())
      .sort((a, b) => this.frequencyMap.get(b[0]) - this.frequencyMap.get(a[0]));

    for (let [char, code] of sortedCodes) {
      const displayChar = char === ' ' ? 'SPACE' : char;
      const freq = this.frequencyMap.get(char);
      console.log(`  ${displayChar}: ${code} (frequency: ${freq}, saves ${8 - code.length} bits per occurrence)`);
    }

    // Encode the text
    let encodedText = '';
    for (let char of this.text) {
      encodedText += this.codes.get(char);
    }

    console.log('\n📝 Encoding process:');
    console.log(`Original: "${this.text}"`);
    console.log('Character by character:');

    let currentEncoded = '';
    for (let char of this.text) {
      const code = this.codes.get(char);
      currentEncoded += code;
      const displayChar = char === ' ' ? 'SPACE' : char;
      console.log(`  ${displayChar} → ${code}`);
    }

    console.log(`\nFinal encoded: ${encodedText}`);

    return encodedText;
  }

  // Step 5: Decode the binary string
  decode(encodedText) {
    console.log('\n🔓 Decoding process:');
    console.log(`Encoded: ${encodedText}`);

    let decodedText = '';
    let currentNode = this.tree;

    console.log('Bit by bit decoding:');
    let bitIndex = 0;

    for (let bit of encodedText) {
      console.log(`  Bit ${bitIndex + 1}: ${bit}`);

      if (bit === '0') {
        currentNode = currentNode.left;
        console.log(`    → Go left`);
      } else {
        currentNode = currentNode.right;
        console.log(`    → Go right`);
      }

      if (currentNode.isLeaf()) {
        const char = currentNode.char;
        decodedText += char;
        const displayChar = char === ' ' ? 'SPACE' : char;
        console.log(`    → Found character: ${displayChar}`);
        console.log(`    → Reset to root for next character`);
        currentNode = this.tree; // Reset to root
      }

      bitIndex++;
    }

    console.log(`\nDecoded: "${decodedText}"`);
    return decodedText;
  }

  // Calculate compression statistics
  getCompressionStats() {
    const originalBits = this.text.length * 8; // ASCII = 8 bits per character

    let compressedBits = 0;
    for (let char of this.text) {
      compressedBits += this.codes.get(char).length;
    }

    const compressionRatio = ((originalBits - compressedBits) / originalBits * 100);

    return {
      originalSize: originalBits,
      compressedSize: compressedBits,
      savedBits: originalBits - compressedBits,
      compressionRatio: compressionRatio.toFixed(1)
    };
  }

  // Helper method for displaying nodes
  nodeToString(node) {
    if (node.isLeaf()) {
      const displayChar = node.char === ' ' ? 'SPACE' : node.char;
      return `${displayChar}(${node.freq})`;
    } else {
      return `Internal(${node.freq})`;
    }
  }

  // Visualize the tree structure
  visualizeTree(node = this.tree, prefix = '', isLast = true) {
    if (!node) return;

    console.log(prefix + (isLast ? '└── ' : '├── ') + this.nodeToString(node));

    if (!node.isLeaf()) {
      const newPrefix = prefix + (isLast ? '    ' : '│   ');
      this.visualizeTree(node.left, newPrefix, false);
      this.visualizeTree(node.right, newPrefix, true);
    }
  }
}

// Demonstration
function demonstrateHuffmanCoding() {
  const text = "HELLO WORLD";
  console.log('🗜️ HUFFMAN CODING DEMONSTRATION');
  console.log('===============================');
  console.log(`Input text: "${text}"`);
  console.log(`Text length: ${text.length} characters\n`);

  const huffman = new HuffmanCoding(text);

  // Step 1: Build frequency map
  huffman.buildFrequencyMap();

  // Step 2: Build Huffman tree
  huffman.buildHuffmanTree();

  // Visualize the tree
  console.log('\n🌳 Huffman Tree Structure:');
  huffman.visualizeTree();

  // Step 3 & 4: Generate codes and encode
  const encoded = huffman.encode();

  // Step 5: Decode to verify
  const decoded = huffman.decode(encoded);

  // Statistics
  console.log('\n📈 Compression Statistics:');
  const stats = huffman.getCompressionStats();
  console.log(`Original size: ${stats.originalSize} bits (${text.length} × 8 bits/char)`);
  console.log(`Compressed size: ${stats.compressedSize} bits`);
  console.log(`Space saved: ${stats.savedBits} bits`);
  console.log(`Compression ratio: ${stats.compressionRatio}%`);

  // Verification
  console.log('\n✅ Verification:');
  console.log(`Original:  "${text}"`);
  console.log(`Decoded:   "${decoded}"`);
  console.log(`Match: ${text === decoded ? '✅ Perfect!' : '❌ Error!'}`);
}

demonstrateHuffmanCoding();
```

---

day - 4

## JIT(Just In Time) Compiler

### Definition:

JIT Compiler is a compilation technique that compiles code during runtime rather than beforehand. It translates high-level code (like Java bytecode or JavaScript) into native machine code at the moment it's needed for execution. JIT compilers analyze running code to identify frequently executed "hot spots" and optimize them aggressively, providing a balance between compilation speed and execution performance.

**Key Properties:**

- Runtime compilation: Code is compiled while the program runs
- Adaptive optimization: Optimizes based on actual usage patterns
- Hot spot detection: Focuses optimization on frequently executed code
- Profile-guided: Uses runtime profiling to make optimization decisions
- Dynamic: Can recompile and re-optimize code as patterns change

**Core Concepts:**

- Interpretation: Initial execution without compilation
- Compilation threshold: When code becomes "hot enough" to compile
- Optimization levels: Multiple tiers of increasingly aggressive optimization
- Deoptimization: Fall back when assumptions prove wrong
- Code cache: Store compiled machine code for reuse

### Example:

JavaScript V8 Engine

```
// JavaScript JIT optimization example
function hotFunction(arr) {
    let sum = 0;
    // V8 will optimize this loop after many iterations
    for (let i = 0; i < arr.length; i++) {
        sum += arr[i];  // Type specialization: assumes numbers
    }
    return sum;
}

// Phase 1: Interpretation (Ignition interpreter)
for (let i = 0; i < 100; i++) {
    hotFunction([1, 2, 3, 4, 5]);
}

// Phase 2: Basic compilation (TurboFan compiler)
for (let i = 0; i < 10000; i++) {
    hotFunction([1, 2, 3, 4, 5]);  // Now compiled with type assumptions
}

// Phase 3: Deoptimization if assumptions break
hotFunction([1, 2, 3.14, "string"]);  // Mixed types → deoptimize back to interpreter
```

---

day - 5

## Sandbox Architecture

### Definition:

Sandbox Architecture is a security design pattern that creates isolated execution environments where untrusted code can run safely without affecting the host system or other applications. Like a child's sandbox that contains sand and prevents it from spreading everywhere, a security sandbox contains potentially dangerous code and prevents it from accessing sensitive resources, making unauthorized network calls, or damaging the system.

**Key Properties:**

- Isolation: Code runs in a restricted environment separate from the host
- Controlled access: Limited permissions to system resources
- Containment: Prevents malicious code from escaping or causing damage
- Monitoring: All activities within the sandbox are observable
- Disposable: Sandbox can be destroyed and recreated easily

**Core Concepts:**

- Security boundaries: Clear separation between trusted and untrusted code
- Privilege restriction: Minimal permissions principle
- Resource limitations: CPU, memory, disk, and network constraints
- API filtering: Only allowed operations can be performed
- Process isolation: Separate processes or containers for untrusted code

### Example:

Allow users to upload and run custom JavaScript code on your website without compromising security

```
// Chrome's multi-process architecture
const browserSandbox = {
  browserProcess: {
    privileges: 'FULL',
    responsibilities: ['UI', 'Network', 'File System', 'Process Management']
  },

  rendererProcess: {
    privileges: 'RESTRICTED',
    responsibilities: ['Render Web Pages', 'Execute JavaScript'],
    restrictions: [
      'No direct file system access',
      'No direct network access',
      'No system API access',
      'Must communicate via IPC'
    ]
  },

  pluginProcess: {
    privileges: 'MINIMAL',
    responsibilities: ['Run Browser Plugins'],
    restrictions: [
      'Heavily restricted API access',
      'Limited memory allocation',
      'Network access via browser process only'
    ]
  }
};
```

---

day - 6

## Conway's Law

### Definition:

Conway's Law states that "organizations which design systems... are constrained to produce designs which are copies of the communication structures of these organizations." In simple terms, the software architecture you build will mirror how your teams are organized and communicate. If your teams are siloed, your software will be siloed; if your teams collaborate well, your software will be well-integrated.

**Key Properties:**

- Organizational mirror: Software structure reflects team structure
- Communication patterns: How teams talk affects how code is structured
- Inevitable influence: Happens whether you plan for it or not
- Structural constraint: Team boundaries become system boundaries
- Feedback loop: System structure can reinforce organizational structure

**Core Concepts:**

- Team boundaries: Separate teams create separate system components
- Communication overhead: Cross-team coordination creates system coupling
- Ownership domains: Teams own specific parts of the system
- Interface design: Team interactions define system interfaces
- Reverse Conway maneuver: Reorganize teams to get desired architecture

### Example:

cross-functional feature teams

```
🚀 CROSS-FUNCTIONAL FEATURE TEAMS
=================================

User Management Team (5 people)
├── 1 Frontend Developer
├── 1 Backend Developer
├── 1 Mobile Developer
├── 1 Database Expert
└── 1 DevOps Engineer
└── Owns: User registration, authentication, profiles

Product Catalog Team (5 people)
├── 1 Frontend Developer
├── 1 Backend Developer
├── 1 Mobile Developer
├── 1 Database Expert
└── 1 DevOps Engineer
└── Owns: Product display, search, recommendations

Order Processing Team (5 people)
├── 1 Frontend Developer
├── 1 Backend Developer
├── 1 Mobile Developer
├── 1 Database Expert
└── 1 DevOps Engineer
└── Owns: Shopping cart, checkout, order management

Payment Team (5 people)
├── 1 Frontend Developer
├── 1 Backend Developer
├── 1 Mobile Developer
├── 1 Database Expert
└── 1 DevOps Engineer
└── Owns: Payment processing, billing, refunds
```

---

day - 7

## Core Event-Driven Architecture (EDA) Patterns

### Definition:

Event-Driven Architecture (EDA) is a design pattern where components communicate through the production and consumption of events. Instead of direct service-to-service calls, systems emit events when something significant happens, and interested parties subscribe to and react to these events. This creates loosely coupled, scalable systems where components don't need to know about each other directly.

**Key Properties:**

- Asynchronous communication: Events are processed without blocking the producer
- Loose coupling: Producers and consumers don't directly depend on each other
- Event-first design: Business logic is expressed as events and reactions
- Temporal decoupling: Producers and consumers don't need to be online simultaneously
- Scalable: Easy to add new event consumers without changing producers

**Core Concepts:**

- Events: Immutable facts about what happened in the system
- Event producers: Components that emit events
- Event consumers: Components that react to events
- Event bus/broker: Infrastructure that routes events
- Event store: Persistent storage for events

### Example:

Netflix

```
Event: "User clicked play on Movie X"
Reactions:
• Billing Service: Check subscription status
• Recommendation Service: Update viewing patterns
• Analytics Service: Track engagement metrics
• CDN Service: Pre-cache related content
• UI Service: Update "Continue Watching" list
```

---

day - 10

## PKCE(Proof Key for Code Exchange)

### Definition:

PKCE (Proof Key for Code Exchange) is a security extension to OAuth 2.0 designed to make the authorization code flow secure for public clients (like mobile apps and SPAs) that cannot securely store client secrets. Instead of using a static client secret, PKCE dynamically generates a cryptographic challenge and verifier pair for each authorization request, preventing authorization code interception attacks.

**Key Properties:**

- Dynamic security: Each authorization request uses unique cryptographic values
- No client secret required: Suitable for public clients (mobile apps, SPAs)
- Prevents code interception: Even if auth code is stolen, it's useless without the verifier
- Cryptographically secure: Uses SHA256 hashing with random code verifiers
- Backward compatible: Works with existing OAuth 2.0 infrastructure

**Core Concepts:**

- Code Verifier: Random cryptographic string (43-128 characters)
- Code Challenge: SHA256 hash of the code verifier (Base64 URL-encoded)
- Challenge Method: How the challenge is derived (S256 for SHA256)
- Authorization code: Still used, but protected by PKCE
- Proof of possession: Client must prove it initiated the flow

### Example:

Mobile app needs to authenticate users securely without storing secrets

```
// ✅ PKCE-PROTECTED OAUTH 2.0 FLOW
class SecurePKCEFlow {
  constructor() {
    this.crypto = require('crypto'); // In browser, use Web Crypto API
  }

  async demonstrateSecureFlow() {
    console.log('\n🔒 SECURE OAUTH 2.0 WITH PKCE');
    console.log('=============================');
    console.log('Solution: Cryptographic proof that client initiated the flow\n');

    // Step 1: Generate PKCE values
    console.log('1️⃣ Mobile app generates PKCE values:');
    const pkcePair = this.generatePKCEPair();

    console.log(`   Code Verifier: ${pkcePair.codeVerifier.substring(0, 20)}... (${pkcePair.codeVerifier.length} chars)`);
    console.log(`   Code Challenge: ${pkcePair.codeChallenge}`);
    console.log(`   Challenge Method: ${pkcePair.challengeMethod}`);
    console.log('   ✅ Verifier stored securely in app memory');

    // Step 2: Authorization request with code challenge
    const authUrl = this.buildPKCEAuthorizationUrl(pkcePair.codeChallenge, pkcePair.challengeMethod);
    console.log('\n2️⃣ Mobile app redirects user with PKCE challenge:');
    console.log(`   ${authUrl}`);
    console.log('   ✅ Code challenge sent, verifier stays in app');

    // Step 3: User authorizes, gets code
    const authorizationCode = 'auth_code_67890';
    console.log('\n3️⃣ User authorizes, gets redirected back:');
    console.log(`   myapp://callback?code=${authorizationCode}`);
    console.log('   ⚠️  Code can still be intercepted...');

    // Step 4: Legitimate app exchanges code with verifier
    console.log('\n4️⃣ Legitimate app exchanges code with PKCE verifier:');
    const tokenResponse = await this.exchangeCodeForToken(authorizationCode, pkcePair.codeVerifier);
    console.log('   POST /token');
    console.log('   {');
    console.log('     "grant_type": "authorization_code",');
    console.log(`     "code": "${authorizationCode}",`);
    console.log('     "client_id": "mobile_app_123",');
    console.log(`     "code_verifier": "${pkcePair.codeVerifier.substring(0, 20)}..."`);
    console.log('   }');

    if (tokenResponse.success) {
      console.log('\n✅ SUCCESS: Token exchange successful!');
      console.log(`   Access Token: ${tokenResponse.accessToken.substring(0, 20)}...`);
    }

    // Step 5: Attack scenario with PKCE protection
    console.log('\n💀 ATTACK ATTEMPT: Attacker tries to use intercepted code:');
    console.log('   Attacker has: authorization code');
    console.log('   Attacker missing: code verifier (only in legitimate app memory)');

    const attackResponse = await this.attemptAttack(authorizationCode);
    console.log('\n❌ ATTACK FAILED:');
    console.log(`   Error: ${attackResponse.error}`);
    console.log('   🛡️ PKCE protection worked!');
  }

  generatePKCEPair() {
    // Generate random code verifier (43-128 characters)
    const codeVerifier = this.generateCodeVerifier();

    // Generate code challenge (SHA256 hash of verifier)
    const codeChallenge = this.generateCodeChallenge(codeVerifier);

    return {
      codeVerifier,
      codeChallenge,
      challengeMethod: 'S256'
    };
  }

  generateCodeVerifier() {
    // Generate cryptographically random string (43-128 chars, URL-safe)
    const buffer = this.crypto.randomBytes(32); // 32 bytes = 43 chars when base64url encoded
    return this.base64URLEncode(buffer);
  }

  generateCodeChallenge(codeVerifier) {
    // SHA256 hash of code verifier, then base64url encode
    const hash = this.crypto.createHash('sha256').update(codeVerifier).digest();
    return this.base64URLEncode(hash);
  }

  base64URLEncode(buffer) {
    return buffer.toString('base64')
      .replace(/\+/g, '-')
      .replace(/\//g, '_')
      .replace(/=/g, ''); // Remove padding
  }

  buildPKCEAuthorizationUrl(codeChallenge, challengeMethod) {
    const params = new URLSearchParams({
      response_type: 'code',
      client_id: 'mobile_app_123',
      redirect_uri: 'myapp://callback',
      scope: 'read_profile read_emails',
      state: 'random_state_value',
      code_challenge: codeChallenge,
      code_challenge_method: challengeMethod
    });

    return `https://auth.example.com/oauth/authorize?${params}`;
  }

  async exchangeCodeForToken(authCode, codeVerifier) {
    // Simulate token exchange request
    console.log('   🔍 Authorization server validates:');
    console.log('     1. Code is valid and not expired');
    console.log('     2. SHA256(code_verifier) === stored code_challenge');
    console.log('     3. Client_id matches original request');

    // Simulate server-side PKCE verification
    const isValidPKCE = this.verifyPKCE(codeVerifier, 'stored_code_challenge_from_auth_request');

    if (isValidPKCE) {
      return {
        success: true,
        accessToken: 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...',
        tokenType: 'Bearer',
        expiresIn: 3600,
        refreshToken: 'refresh_token_abc123',
        scope: 'read_profile read_emails'
      };
    } else {
      return {
        success: false,
        error: 'invalid_grant',
        errorDescription: 'PKCE verification failed'
      };
    }
  }

  async attemptAttack(authCode) {
    console.log('   POST /token (attacker without code_verifier):');
    console.log('   {');
    console.log('     "grant_type": "authorization_code",');
    console.log(`     "code": "${authCode}",`);
    console.log('     "client_id": "mobile_app_123"');
    console.log('     // Missing code_verifier!');
    console.log('   }');

    // Attacker doesn't have the code verifier
    return {
      error: 'invalid_request',
      errorDescription: 'Code verifier required for PKCE'
    };
  }

  verifyPKCE(codeVerifier, storedCodeChallenge) {
    // In real implementation, server would compare like this:
    const computedChallenge = this.generateCodeChallenge(codeVerifier);
    return computedChallenge === storedCodeChallenge;
  }
}
```

---
