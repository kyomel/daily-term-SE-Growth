day - 1

## Modulo Hashing

### Definition:

Modulo Hashing is a technique that uses the remainder operator (%) to map any input (like a user ID) into a fixed range of buckets. It's the simplest way to answer: "Given this thing, which slot should it go into?"

In one line:

bucket=hash(key)modn

Where n is the number of buckets (servers, partitions, slots, etc.).

Analogy — A Simple Hotel Key System
Imagine a hotel with 4 floors and a dumbwaiter that delivers room-service orders by floor number. The chef looks at the room number and does:

"Room 307 → last digit is 7 → 7 mod 4 = 3 → Floor 3"

Room Number	Last Digit	digit mod 4	Floor
301	1	1 mod 4 = 1	1st floor
405	5	5 mod 4 = 1	1st floor
512	2	2 mod 4 = 2	2nd floor
307	7	7 mod 4 = 3	3rd floor
720	0	0 mod 4 = 0	4th floor (floor 0)
The chef never needs to memorize which room is on which floor — the math does it instantly.

The Formula
index=hash(key)modn

hash(key) — turns the key into a number (for strings, use a hash function like MD5 or hashCode())
mod n — wraps that number into the range [0,n−1]

### Example:

Picking a Server (Distributed Systems)

```
import hashlib

def pick_server(user_id: str, servers: list) -> str:
    # Step 1: Hash the user ID to a number
    hash_bytes = hashlib.md5(user_id.encode()).digest()
    hash_int = int.from_bytes(hash_bytes[:8], 'big')  # first 8 bytes → integer
    
    # Step 2: Modulo by number of servers
    index = hash_int % len(servers)
    
    return servers[index]

# Usage
servers = ["Server-A", "Server-B", "Server-C", "Server-D"]

print(pick_server("alice@example.com", servers))  # e.g., Server-C
print(pick_server("bob@example.com",   servers))  # e.g., Server-A
print(pick_server("alice@example.com", servers))  # Always Server-C! (deterministic)
🗄️ Classic Hash Table Example

class SimpleHashTable:
    def __init__(self, size=10):
        self.size = size
        self.table = [None] * size
    
    def _index(self, key: str) -> int:
        # Sum of ASCII values, then modulo
        total = sum(ord(char) for char in key)
        return total % self.size
    
    def put(self, key: str, value):
        idx = self._index(key)
        self.table[idx] = value
    
    def get(self, key: str):
        idx = self._index(key)
        return self.table[idx]

# Usage
ht = SimpleHashTable(size=5)
ht.put("apple",  "🍎")
ht.put("banana", "🍌")
ht.put("cherry", "🍒")

print(ht.get("apple"))   # 🍎
print(ht.get("banana"))  # 🍌
```

---