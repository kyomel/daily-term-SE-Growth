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

day - 2

## LSM Trees (Log-Structured Merge-Trees)

### Definition:

LSM Trees are a data structure designed for write-heavy workloads. Instead of updating data in place (like a B-tree), they just append every change to a log, then lazily sort and merge things in the background. The mantra: write fast now, organize later.

In one sentence: Turn random writes into sequential writes, then periodically tidy up.

Analogy — The Receptionist's Logbook
Picture a busy doctor's office. Patients arrive constantly. The receptionist has two tools:

Tool	What happens	LSM Equivalent
📝 Logbook on the desk	Scribbles every arrival instantly, in order of arrival. Super fast — just append to the page.	Memtable + WAL (in-memory + append-only log)
🗄️ Filing cabinet	During quiet moments, she takes names from the logbook and files them alphabetically into patient folders.	SSTables (sorted files on disk)
♻️ Merging old folders	Eventually, she merges duplicate folders, keeping only the latest record for each patient.	Compaction
Writing (new patient arrives): Just scribble in the logbook. Zero searching. Lightning fast.
Reading (find patient info): Check the logbook first, then check the filing cabinet. Slightly more work for reads, but writes fly.

How It Works

    WRITE ──▶ [ Memtable (sorted, in RAM) ]    ← fastest
                    │
                    │ (when full, flush to disk)
                    ▼
              [ SSTable (sorted file) ]        ← disk, immutable
              [ SSTable (sorted file) ]
              [ SSTable (sorted file) ]
                    │
                    │ (background compaction)
                    ▼
              [ Merged SSTable ]               ← fewer files, garbage collected

### Example:

Simplified LSM Key-Value Store in Python
This is a dramatically simplified version to illustrate the mechanics. Real LSM engines use skip lists, bloom filters, binary search, and much more.

```
import bisect
import json
import os

class MiniLSM:
    def __init__(self, db_path="minilsm_data"):
        self.db_path = db_path
        os.makedirs(db_path, exist_ok=True)
        self.memtable = {}          # in-memory sorted dict
        self.sstables = []          # list of filenames on disk
        self.memtable_limit = 3     # flush after 3 entries (tiny for demo)

    def put(self, key, value):
        """Just write to memtable. Super fast!"""
        self.memtable[key] = value
        print(f"✍️  memtable ← {key}: {value}")

        if len(self.memtable) >= self.memtable_limit:
            self._flush()

    def get(self, key):
        """Read path: check memtable first, then SSTables (newest first)."""
        # 1. Check memtable (fastest, in RAM)
        if key in self.memtable:
            print(f"🔍 Found '{key}' in memtable")
            return self.memtable[key]

        # 2. Check SSTables, newest to oldest
        for sst_filename in reversed(self.sstables):
            with open(sst_filename, 'r') as f:
                data = json.load(f)
                if key in data:
                    print(f"🔍 Found '{key}' in {sst_filename}")
                    return data[key]
        return None

    def _flush(self):
        """Sort memtable, write to disk as a new SSTable, then clear memtable."""
        sst_name = f"{self.db_path}/sst_{len(self.sstables)+1:03d}.json"
        with open(sst_name, 'w') as f:
            # Sort by key before writing (real SSTables are always sorted)
            sorted_data = dict(sorted(self.memtable.items()))
            json.dump(sorted_data, f)

        self.sstables.append(sst_name)
        print(f"📦 Flushed → {sst_name} ({len(self.memtable)} entries)")
        self.memtable.clear()

    def compact(self):
        """Merge all SSTables into one, keeping only the latest value per key."""
        if len(self.sstables) < 2:
            return

        merged = {}
        # Read from oldest to newest, so newer values overwrite older
        for sst_filename in self.sstables:
            with open(sst_filename, 'r') as f:
                data = json.load(f)
                merged.update(data)   # newer overwrites older
            os.remove(sst_filename)   # delete old file

        # Write the single merged file
        new_sst = f"{self.db_path}/sst_merged.json"
        with open(new_sst, 'w') as f:
            json.dump(dict(sorted(merged.items())), f)

        self.sstables = [new_sst]
        print(f"♻️  Compacted → {new_sst} ({len(merged)} keys)")

# --- Usage ---
db = MiniLSM()

db.put("name", "Alice")     # memtable ←
db.put("city", "Paris")     # memtable ←
db.put("name", "Bob")       # overwrites Alice in memtable
db.put("age",  "30")        # triggers flush! (limit=3 reached)

print("\nReading...")
print(db.get("name"))        # checks memtable first
print(db.get("city"))        # checks SSTable
db.compact()                 # merge everything (real DBs do this in background)
Output:


✍️  memtable ← name: Alice
✍️  memtable ← city: Paris
✍️  memtable ← name: Bob
📦 Flushed → minilsm_data/sst_001.json (3 entries)
✍️  memtable ← age: 30

Reading...
🔍 Found 'name' in memtable
🔍 Found 'city' in minilsm_data/sst_001.json
♻️  Compacted → minilsm_data/sst_merged.json (4 keys)
Notice: name: Bob overwrites name: Alice because it was written later — no in-place update, just a newer entry that takes precedence.
```

---

day - 3

## U-Net Architecture

### Definition:

U-Net is a convolutional neural network designed for image segmentation — the task of classifying every single pixel in an image. Its signature "U" shape combines a contracting path (which captures what is in the image) with an expanding path (which pinpoints where it is), linked by skip connections that preserve fine details.

In one sentence: U-Net takes an image in and spits out a same-sized map where each pixel gets a label.

Analogy — The Artist's Two-Step Process
Imagine an artist painting a detailed landscape from a blurry photo:

Step	What happens	U-Net Equivalent
1️⃣ Squinting	Steps back, squints, sees the big shapes — "There's a mountain, a lake, some trees"	Contracting path (Encoder) — captures high-level what
2️⃣ Zooming in	Leans in close, fills in the edges, restores sharp boundaries — "The shoreline goes exactly here"	Expanding path (Decoder) — recovers precise where
🔗 Cross-checking	Glances back at the original photo constantly so no detail is lost	Skip connections — shuttle fine details from early layers to later layers
Without the skip connections (cross-checking), you'd get a blurry blob. With them, you get sharp, pixel-perfect boundaries.

### Example:

U-Net in PyTorch (Simplified)

```
This is a clean, minimal U-Net for binary segmentation (e.g., "cell vs. background"):


import torch
import torch.nn as nn

class DoubleConv(nn.Module):
    """Two convolutions + batch norm + ReLU (the basic building block)."""
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, kernel_size=3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.Conv2d(out_ch, out_ch, kernel_size=3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
        )

    def forward(self, x):
        return self.conv(x)


class UNet(nn.Module):
    def __init__(self, in_channels=1, out_channels=1):
        super().__init__()

        # ── Encoder (contracting path) ──
        self.enc1 = DoubleConv(in_channels, 64)    # 572 → 570
        self.enc2 = DoubleConv(64, 128)            #    → 140
        self.enc3 = DoubleConv(128, 256)           #    →  68
        self.enc4 = DoubleConv(256, 512)           #    →  32

        self.pool = nn.MaxPool2d(kernel_size=2, stride=2)

        # ── Bottleneck ──
        self.bottleneck = DoubleConv(512, 1024)    #    →  28

        # ── Decoder (expanding path) ──
        self.up4 = nn.ConvTranspose2d(1024, 512, kernel_size=2, stride=2)
        self.dec4 = DoubleConv(1024, 512)          # 512(skip) + 512(up)

        self.up3 = nn.ConvTranspose2d(512, 256, kernel_size=2, stride=2)
        self.dec3 = DoubleConv(512, 256)           # 256(skip) + 256(up)

        self.up2 = nn.ConvTranspose2d(256, 128, kernel_size=2, stride=2)
        self.dec2 = DoubleConv(256, 128)           # 128(skip) + 128(up)

        self.up1 = nn.ConvTranspose2d(128, 64, kernel_size=2, stride=2)
        self.dec1 = DoubleConv(128, 64)            # 64(skip) + 64(up)

        # ── Output ──
        self.out_conv = nn.Conv2d(64, out_channels, kernel_size=1)

    def forward(self, x):
        # Encoder
        e1 = self.enc1(x)                          # [B,  64, H,   W]
        e2 = self.enc2(self.pool(e1))              # [B, 128, H/2, W/2]
        e3 = self.enc3(self.pool(e2))              # [B, 256, H/4, W/4]
        e4 = self.enc4(self.pool(e3))              # [B, 512, H/8, W/8]

        # Bottleneck
        b = self.bottleneck(self.pool(e4))         # [B, 1024, H/16, W/16]

        # Decoder with skip connections
        d4 = self.up4(b)                           # [B, 512, H/8, W/8]
        d4 = torch.cat([d4, e4], dim=1)            # Skip! → [B, 1024, H/8, W/8]
        d4 = self.dec4(d4)

        d3 = self.up3(d4)                          # [B, 256, H/4, W/4]
        d3 = torch.cat([d3, e3], dim=1)            # Skip! → [B, 512, H/4, W/4]
        d3 = self.dec3(d3)

        d2 = self.up2(d3)                          # [B, 128, H/2, W/2]
        d2 = torch.cat([d2, e2], dim=1)            # Skip! → [B, 256, H/2, W/2]
        d2 = self.dec2(d2)

        d1 = self.up1(d2)                          # [B, 64, H, W]
        d1 = torch.cat([d1, e1], dim=1)            # Skip! → [B, 128, H, W]
        d1 = self.dec1(d1)

        return self.out_conv(d1)                   # [B, out_ch, H, W]


# ── Usage Example ──
model = UNet(in_channels=3, out_channels=1)       # RGB in, binary mask out
dummy_image = torch.randn(1, 3, 572, 572)          # batch of 1, 572×572 RGB
mask = model(dummy_image)

print(f"Input shape:  {dummy_image.shape}")        # [1, 3, 572, 572]
print(f"Output shape: {mask.shape}")               # [1, 1, 572, 572]  ← same spatial size!
```

---

day - 4

## SSO Migration

### Definition:

SSO migration is the process of moving a company's single sign-on system from one identity provider to another — for example, switching from Okta to Microsoft Entra ID — without locking users out, breaking app integrations, or forcing everyone to reset their passwords on the same day.

Think of it like changing the locks on every door in a massive office building while employees are still walking through those doors. You can't just swap all the locks at once at 9 AM. Instead, you phase it room by room, test each new key, and keep both the old and new locks working during the transition.

The Core Challenge

                    ┌─────────────────────┐
    Current state:  │       ONE IDP       │   ← e.g., Okta
                    │  (all apps trust it) │
                    └──────┬──────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
     ┌────────┐      ┌────────┐      ┌────────┐
     │ GitHub │      │ Jira   │      │  Zoom  │   ...and 80+ more apps
     └────────┘      └────────┘      └────────┘
You need to point every single arrow to a new Identity Provider (IdP) — without those apps ever trusting a broken or missing identity.

### Example:

Okta → Microsoft Entra ID

```
A mid-size company with 1,500 employees and 85 SaaS apps decides to migrate from Okta to Microsoft Entra ID (formerly Azure AD). Here's a staged plan:

Phase	What Happens	User Experience
1. Prep	Inventory all 85 apps. Map SAML/OIDC configs from Okta to Entra. Identify which apps support multi-IdP.	Nothing changes.
2. Parallel Run	Entra ID is set up as a second IdP. A pilot group (IT team, 10 people) uses Entra for a week.	Pilot users see a slightly different login page. Everyone else: no change.
3. Phased Cutover	Roll out in waves — Finance dept on Monday, Engineering on Wednesday, etc.	User clicks "Sign in with Google" on Jira → now sees "Sign in with Microsoft" instead. Same username. No password reset.
4. Coexistence	Okta stays live for 2 weeks while Entra handles 100% of traffic. Rollback is one config change away.	Transparent.
5. Decommission	Okta tenant emptied, contracts canceled, old certs revoked.	Done.
```

---

day - 5

## Hi-Lo Algorithm

### Definition:

The Hi-Lo Algorithm is an ID generation strategy that produces unique identifiers by combining two numbers: a "Hi" value fetched occasionally from a shared source (like a database), and a "Lo" value that increments locally in memory. The formula:

ID=(Hi×blockSize)+Lo

In one sentence: Grab a big chunk of IDs from the database once, then hand them out locally until you run out — repeat. This avoids a database round-trip for every single ID.

Analogy — The Waiter's Check Numbers
Imagine a restaurant where every order needs a unique check number, and the central office owns the master list. Three waiters are taking orders simultaneously:

Approach	What happens	Problem
❌ Call the office for every order	"Give me one check number!" → wait → get #42. Next order: call again → #43.	Slow, bottleneck, waiters standing around on the phone
✅ Hi-Lo: Grab a block	Waiter A: "Give me 100 numbers!" → gets block #0 (covers #0–#99). Waiter B: gets block #1 (covers #100–#199). Both hand out numbers locally without calling again.	Fast, no waiting, office is barely bothered
Role	Real World	Hi-Lo Algorithm
Central office	Master sequence / DB table	Hi value — fetched occasionally
Waiter's own pad	Local counter (0–99)	Lo value — incremented in memory
Check number	Final unique order number	(Hi × blockSize) + Lo
"Give me another block"	Waiter calls office when pad runs out	Next database fetch when Lo hits blockSize
The office is contacted once per 100 orders, not once per order. That's the Hi-Lo magic.

### Example:

Hi-Lo ID Generator in Python

```
import threading

class HiLoGenerator:
    """
    Generates unique IDs without hitting the DB for every ID.
    Fetches a new 'Hi' block when the local 'Lo' runs out.
    """
    def __init__(self, block_size=100):
        self.block_size = block_size
        self.hi = -1          # Current high block (will trigger fetch on first next())
        self.lo = block_size  # Start at max to trigger immediate fetch
        self.lock = threading.Lock()

    def _fetch_next_hi(self):
        """
        Simulate a DB call to get the next available Hi block.
        In reality this would be:
            SELECT nextval('hi_sequence') FROM dual;  -- Oracle/PostgreSQL
            UPDATE hi_table SET hi = hi + 1;           -- manual table
        """
        # Simulated: in real code, this hits the database
        self._db_value = getattr(self, '_db_value', -1) + 1
        print(f"  📞 DB CALL → fetched Hi block #{self._db_value}")
        return self._db_value

    def next_id(self) -> int:
        """
        Returns the next unique ID.
        Calls the database ONLY when the local block is exhausted.
        """
        with self.lock:
            if self.lo >= self.block_size:
                # Ran out of local numbers → fetch new Hi block
                self.hi = self._fetch_next_hi()
                self.lo = 0

            generated_id = (self.hi * self.block_size) + self.lo
            self.lo += 1
            return generated_id


# ── Usage ──
gen = HiLoGenerator(block_size=5)   # Small block for demo

print("Generating IDs:")
for i in range(12):
    print(f"  → ID #{gen.next_id()}")
Output:


Generating IDs:
  📞 DB CALL → fetched Hi block #0
  → ID #0
  → ID #1
  → ID #2
  → ID #3
  → ID #4
  📞 DB CALL → fetched Hi block #1     ← Only 3 DB calls for 12 IDs!
  → ID #5
  → ID #6
  → ID #7
  → ID #8
  → ID #9
  📞 DB CALL → fetched Hi block #2
  → ID #10
  → ID #11
Only 3 database calls for 12 IDs. With block_size=1000, you'd need just 1 DB call per 1000 IDs.
```

---

day - 8

## Correlation ID

### Definition:

A Correlation ID is a unique identifier attached to a request that travels with it across every service it touches in a distributed system. It's the breadcrumb trail that lets you trace a single user action through dozens of microservices, message queues, and databases.

In one sentence: Tag a request at the door with a unique ID, then use that ID to follow it everywhere it goes.

Analogy — The Package Tracking Number
Imagine you order a jacket online. The order triggers a chain of events:


  🛒 Order       📦 Warehouse     🚚 Shipping     🏠 Delivery
   System    →    System     →     Provider   →   Confirmation
Without a tracking number	With a tracking number
"Where's my order??" → Shrug. Call every department.	Enter tracking # → See exactly where it is
Warehouse error? Nobody knows which order is affected.	"Order #TRK-8821 failed at warehouse — re-pick it"
Customer complains? 20-minute investigation across 5 systems.	Search logs for TRK-8821 → instant full timeline
A Correlation ID is that tracking number — for every single request inside your system.

How It Works

  Client                    Service A            Service B            Service C
    │                          │                    │                    │
    │  GET /order/42           │                    │                    │
    │  X-Correlation-ID: abc   │                    │                    │
    │─────────────────────────▶│                    │                    │
    │                          │  POST /inventory   │                    │
    │                          │  X-Correlation-ID: abc                 │
    │                          │───────────────────▶│                    │
    │                          │                    │  PUBLISH order.paid│
    │                          │                    │  correlationId=abc │
    │                          │                    │───────────────────▶│
    │                          │                    │                    │
    │  ◀─── 200 OK ───────────│◀───────────────────│◀───────────────────│
    │                          │                    │                    │
    ▼                          ▼                    ▼                    ▼
  ┌──────────────────────────────────────────────────────────────────────┐
  │  Every log line, every message, every DB row stamped with: "abc"     │
  │  Search "abc" → full timeline instantly                              │
  └──────────────────────────────────────────────────────────────────────┘

### Example:

End-to-End in a Microservice System

```
1️⃣ Frontend / Gateway — Create or Forward the ID

# API Gateway (or first service that handles the request)
from flask import Flask, request
import uuid
import requests

app = Flask(__name__)

@app.before_request
def attach_correlation_id():
    # If the caller already sent one, use it. Otherwise, create it.
    correlation_id = request.headers.get('X-Correlation-ID')
    if not correlation_id:
        correlation_id = str(uuid.uuid4())
    
    # Store it so every downstream call can access it
    request.correlation_id = correlation_id
    print(f"🔖 [{correlation_id}] Request started: {request.method} {request.path}")

@app.route('/order', methods=['POST'])
def create_order():
    order_data = request.json
    cid = request.correlation_id

    print(f"📦 [{cid}] Creating order for: {order_data['item']}")

    # Call service B — PASS THE CORRELATION ID
    resp = requests.post(
        "http://inventory-service/check",
        json={"item_id": order_data["item"]},
        headers={"X-Correlation-ID": cid}        # ← THE KEY LINE
    )
    print(f"📋 [{cid}] Inventory responded: {resp.status_code}")
    return {"status": "ok", "correlation_id": cid}
2️⃣ Middle Service — Receive It, Log It, Forward It

# Inventory Service
from flask import Flask, request

app = Flask(__name__)

@app.route('/check', methods=['POST'])
def check_inventory():
    cid = request.headers.get('X-Correlation-ID', 'no-cid')
    
    print(f"🏭 [{cid}] Checking inventory for item_id={request.json['item_id']}")
    print(f"🏭 [{cid}] Stock found: 23 units")

    # Simulate a database insert with correlation ID
    # INSERT INTO inventory_logs (item_id, action, correlation_id) VALUES (42, 'checked', 'abc')

    return {"available": True}
3️⃣ Logging — The Real Payoff

import logging
import sys

# Structured logging that always includes the correlation ID
formatter = logging.Formatter(
    '{"time": "%(asctime)s", "level": "%(levelname)s", "cid": "%(cid)s", "msg": "%(message)s"}'
)

handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(formatter)

logger = logging.getLogger("service")
logger.addHandler(handler)

# Usage in any service
def log(cid, message, level="info"):
    extra = {"cid": cid}
    getattr(logger, level)(message, extra=extra)

log("abc-123", "Order created")        # {"cid": "abc-123", "msg": "Order created"}
log("abc-123", "Payment processed")    # {"cid": "abc-123", "msg": "Payment processed"}
log("abc-123", "Shipment dispatched")  # {"cid": "abc-123", "msg": "Shipment dispatched"}
4️⃣ Searching — The Magic Moment
Now in your log aggregator (ELK, Splunk, Datadog, Grafana Loki):


# Find EVERYTHING related to a single user request
$ grep "abc-123" /var/log/*.json

[12:03:01] abc-123  →  Gateway:    Request started POST /order
[12:03:01] abc-123  →  Orders:     Creating order for "jacket"
[12:03:02] abc-123  →  Inventory:  Checking stock for item #42
[12:03:02] abc-123  →  Inventory:  Stock: 23 units
[12:03:03] abc-123  →  Payment:    Charging $89.99
[12:03:03] abc-123  →  Payment:    Payment approved
[12:03:04] abc-123  →  Warehouse:  Pick ticket created #WH-551
[12:03:04] abc-123  →  Orders:     Order complete → 200 OK
One search. The full story. Every service. In order.
```

---

day - 9

## CodingBooth

### Definition:

CodingBooth is a tool that gives every project its own isolated, reproducible development environment declared in the repository itself. One command brings it up; one command tears it down. Nothing installs on your host machine — all toolchains, runtimes, and dependencies live inside a container that travels with the project.

In one sentence: "A booth per project. A clean host. A repo that brings its own environment."

Analogy — The Pop-Up Kitchen
Imagine a chef who cooks three different cuisines — Italian, Thai, and French. Instead of cramming every ingredient and tool into one chaotic kitchen, she uses pop-up kitchen pods:

Approach	What it looks like	CodingBooth
❌ One messy kitchen	Global Node.js, Python 2.7 and 3.11, three JDKs, a broken JAVA_HOME, and a pip package you installed once in 2023 and forgot about	The "project residue" problem
✅ Pop-up kitchen pods	Open the Italian pod → everything for pasta. Open the Thai pod → wok, fish sauce, jasmine rice. French pod → butter, wine, copper pans.	Each project has its own .booth/ folder with exactly what it needs
Cooking Concept	Real World	CodingBooth
Pop-up kitchen pod	A self-contained cooking station	Booth — a containerized dev environment
Recipe card taped to the pod	Lists exactly what's inside	Boothfile — declares the environment
Pod setup instructions	"Plug in, turn on gas, start cooking"	./booth — one command brings it up
Cleaning up	Fold the pod away, counter is spotless	./booth down — nothing left on the host

### Example:

A Real Boothfile
Here's a complete CodingBooth setup for a Svelte + Firebase blog with Claude Code AI assistance:

```
.booth/Boothfile

# .booth/Boothfile
# syntax=codingbooth/boothfile:1
# Configured by: booth config --no-tui --overwrite --variant codeserver --port 13579 --expose 5173 --select firebase+credential/claude-code+auto-accept+credential+settings-cache

setup svelte        # Node.js + Svelte toolchain
setup firebase      # Firebase CLI for deploys
setup claude-code   # Claude Code AI assistant
Two setup lines and the entire development environment is declared. No FROM, no ARG, no shell wrangling. Each setup line maps to a curated install script.

.booth/config.toml

# .booth/config.toml
variant = "codeserver"
port    = "13579"

run-args = [
    "-v", "~/.config/configstore/firebase-tools.json:/etc/cb-home-seed/.config/configstore/firebase-tools.json:ro",
    "-v", "~/.claude.json:/etc/cb-home-seed/.claude.json:ro",
    "--publish", "5173:5173"
]
variant = "codeserver" — opens a browser-based VS Code
Credential mounts (:ro = read-only) pass your Firebase and Claude keys into the booth
--publish 5173:5173 — forwards the Vite dev server to your host browser
Usage

# Bring the environment up (one command)
./booth

# → Browser opens at http://localhost:13579
# → Full VS Code in the browser
# → Svelte dev server running on port 5173
# → Firebase CLI ready to deploy
# → Claude Code ready to assist
# → NOTHING installed on the host

# When done:
./booth down   # Tears it all down, host is clean
```

---
