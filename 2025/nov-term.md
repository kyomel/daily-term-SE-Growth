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
