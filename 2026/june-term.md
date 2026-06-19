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

Room Number Last Digit digit mod 4 Floor
301 1 1 mod 4 = 1 1st floor
405 5 5 mod 4 = 1 1st floor
512 2 2 mod 4 = 2 2nd floor
307 7 7 mod 4 = 3 3rd floor
720 0 0 mod 4 = 0 4th floor (floor 0)
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

Tool What happens LSM Equivalent
📝 Logbook on the desk Scribbles every arrival instantly, in order of arrival. Super fast — just append to the page. Memtable + WAL (in-memory + append-only log)
🗄️ Filing cabinet During quiet moments, she takes names from the logbook and files them alphabetically into patient folders. SSTables (sorted files on disk)
♻️ Merging old folders Eventually, she merges duplicate folders, keeping only the latest record for each patient. Compaction
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

Step What happens U-Net Equivalent
1️⃣ Squinting Steps back, squints, sees the big shapes — "There's a mountain, a lake, some trees" Contracting path (Encoder) — captures high-level what
2️⃣ Zooming in Leans in close, fills in the edges, restores sharp boundaries — "The shoreline goes exactly here" Expanding path (Decoder) — recovers precise where
🔗 Cross-checking Glances back at the original photo constantly so no detail is lost Skip connections — shuttle fine details from early layers to later layers
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

Approach What happens Problem
❌ Call the office for every order "Give me one check number!" → wait → get #42. Next order: call again → #43. Slow, bottleneck, waiters standing around on the phone
✅ Hi-Lo: Grab a block Waiter A: "Give me 100 numbers!" → gets block #0 (covers #0–#99). Waiter B: gets block #1 (covers #100–#199). Both hand out numbers locally without calling again. Fast, no waiting, office is barely bothered
Role Real World Hi-Lo Algorithm
Central office Master sequence / DB table Hi value — fetched occasionally
Waiter's own pad Local counter (0–99) Lo value — incremented in memory
Check number Final unique order number (Hi × blockSize) + Lo
"Give me another block" Waiter calls office when pad runs out Next database fetch when Lo hits blockSize
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

🛒 Order 📦 Warehouse 🚚 Shipping 🏠 Delivery
System → System → Provider → Confirmation
Without a tracking number With a tracking number
"Where's my order??" → Shrug. Call every department. Enter tracking # → See exactly where it is
Warehouse error? Nobody knows which order is affected. "Order #TRK-8821 failed at warehouse — re-pick it"
Customer complains? 20-minute investigation across 5 systems. Search logs for TRK-8821 → instant full timeline
A Correlation ID is that tracking number — for every single request inside your system.

How It Works

Client Service A Service B Service C
│ │ │ │
│ GET /order/42 │ │ │
│ X-Correlation-ID: abc │ │ │
│─────────────────────────▶│ │ │
│ │ POST /inventory │ │
│ │ X-Correlation-ID: abc │
│ │───────────────────▶│ │
│ │ │ PUBLISH order.paid│
│ │ │ correlationId=abc │
│ │ │───────────────────▶│
│ │ │ │
│ ◀─── 200 OK ───────────│◀───────────────────│◀───────────────────│
│ │ │ │
▼ ▼ ▼ ▼
┌──────────────────────────────────────────────────────────────────────┐
│ Every log line, every message, every DB row stamped with: "abc" │
│ Search "abc" → full timeline instantly │
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

Approach What it looks like CodingBooth
❌ One messy kitchen Global Node.js, Python 2.7 and 3.11, three JDKs, a broken JAVA_HOME, and a pip package you installed once in 2023 and forgot about The "project residue" problem
✅ Pop-up kitchen pods Open the Italian pod → everything for pasta. Open the Thai pod → wok, fish sauce, jasmine rice. French pod → butter, wine, copper pans. Each project has its own .booth/ folder with exactly what it needs
Cooking Concept Real World CodingBooth
Pop-up kitchen pod A self-contained cooking station Booth — a containerized dev environment
Recipe card taped to the pod Lists exactly what's inside Boothfile — declares the environment
Pod setup instructions "Plug in, turn on gas, start cooking" ./booth — one command brings it up
Cleaning up Fold the pod away, counter is spotless ./booth down — nothing left on the host

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

day - 10

## Frame Buffer Hashing

### Definition:

Frame Buffer Hashing is a visual regression testing technique that verifies rendered graphics by reading the raw GPU framebuffer (the pixel data before any compression or encoding), computing a cryptographic hash of those bytes, and comparing it against a known-good reference hash. If the hashes match, every pixel is identical. If they don't, something changed.

In one sentence: "Take a fingerprint of the raw pixels straight from the GPU, then compare it to the golden fingerprint — if they match, the render is perfect."

Analogy — The Painting Authenticator
Imagine an art authenticator verifying that a famous painting hasn't been tampered with. She has two approaches:

Approach What happens Result
📸 Take a photo, then compare photos Photo compression introduces artifacts. Two identical paintings shot with different camera settings produce different JPEGs. "They don't match!" → false alarm 😤
🔬 Scan the canvas directly She places a high-res scanner directly on the original canvas, captures raw pigment data, and hashes it. Same canvas = same hash. Always. "Hash matches. It's authentic." ✅
The photo is like a PNG screenshot — encoded, compressed, lossy. The direct scan is like the raw framebuffer — pure, unmodified pixel data from the GPU.

### Example:

Frame Buffer Hashing in Practice

```
Here's a simplified version based on real GPU testing frameworks (Intel GPU Tools, Chromium's video frame validator):


import hashlib
import struct
from dataclasses import dataclass
from typing import Dict

@dataclass
class FrameBuffer:
    """Represents a raw GPU framebuffer mapped to CPU memory."""
    width: int
    height: int
    bytes_per_pixel: int  # e.g., 4 for RGBA
    raw_bytes: bytes       # raw pixel data straight from the GPU

def hash_framebuffer(fb: FrameBuffer) -> str:
    """
    Compute an MD5 hash of the raw framebuffer bytes.
    No encoding, no compression — pure pixel data.
    """
    # In real GPU testing, you'd map the framebuffer from GPU memory:
    #   ptr = glMapBuffer(GL_PIXEL_PACK_BUFFER, GL_READ_ONLY)
    #   fb.raw_bytes = ptr[0 : width * height * bpp]

    return hashlib.md5(fb.raw_bytes).hexdigest()


# ── Golden Reference Store ──
# Stored once when the test is first created.
# A few KB for hundreds of frames (vs. GBs of PNGs).
GOLDEN_HASHES: Dict[str, str] = {
    "main_menu":        "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
    "settings_screen":  "1f2e3d4c5b6a79808182838485868788",
    "gameplay_frame_1": "deadbeefcafebabedeadbeefcafebabe",
}


def visual_regression_test(scene_name: str, fb: FrameBuffer) -> bool:
    """
    Run a single visual regression test.
    Returns True if the frame matches the golden reference.
    """
    actual_hash = hash_framebuffer(fb)
    expected_hash = GOLDEN_HASHES.get(scene_name)

    if expected_hash is None:
        # New scene — store this as the golden reference
        print(f"✨ No golden hash for '{scene_name}'. Storing: {actual_hash}")
        GOLDEN_HASHES[scene_name] = actual_hash
        return True

    if actual_hash == expected_hash:
        print(f"✅ {scene_name}: PASS (hash matches)")
        return True
    else:
        print(f"❌ {scene_name}: FAIL")
        print(f"   Expected: {expected_hash}")
        print(f"   Got:      {actual_hash}")
        # Save the actual frame as a PNG for human review
        save_frame_as_png(fb, f"failure_artifacts/{scene_name}.png")
        return False


def save_frame_as_png(fb: FrameBuffer, path: str):
    """Only save a PNG when the test FAILS — for manual inspection."""
    # Convert raw bytes to PNG (using PIL, stb_image, etc.)
    # This is the ONLY time encoding happens — after failure.
    print(f"   📸 Saved failure artifact: {path}")


# ── Usage ──
# Simulate a correct render
correct_frame = FrameBuffer(
    width=1920, height=1080, bytes_per_pixel=4,
    raw_bytes=b'\xff\x00\x00\xff' * (1920 * 1080)  # all red pixels
)
test1 = visual_regression_test("main_menu", correct_frame)

# Simulate a rendering bug — one pixel is wrong
buggy_bytes = bytearray(b'\xff\x00\x00\xff' * (1920 * 1080))
buggy_bytes[0] = 0xfe  # slightly different red in first pixel
buggy_frame = FrameBuffer(
    width=1920, height=1080, bytes_per_pixel=4,
    raw_bytes=bytes(buggy_bytes)
)
test2 = visual_regression_test("main_menu", buggy_frame)
Output:


✅ main_menu: PASS (hash matches)
❌ main_menu: FAIL
   Expected: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6
   Got:      7f8e9d0c1b2a39485748392a1b0c0d0e
   📸 Saved failure artifact: failure_artifacts/main_menu.png
```

---

day - 11

## Perag

### Definition:

Perag (short for Personal RAG) is a local, no-cloud, command-line RAG tool that gives your AI assistant long-term memory by making your personal documents searchable. It works entirely on your machine — no data ever leaves your computer.

Instead of shoving hundreds of files into an AI's context window (which would overflow it), Perag:
Chunks your documents into small pieces
Embeds them into vector representations
Stores them in a local SQLite database
Retrieves only the most relevant chunks when you ask a question

It's designed as a UNIX pipeline (chunk | embed | ingest | query) — composable, transparent, and scriptable.

### Example:

Say you have a folder of contracts:

```
~/projects/
├── nda_2024.pdf
├── service-agreement.docx
└── meeting-notes.md


Step 1 — Initialize
cd ~/projects
perag init


Step 2 — Add documents
perag add nda_2024.pdf service-agreement.docx meeting-notes.md


Step 3 — Ask questions to your AI assistant
"What does the NDA say about the notice period?"

Your AI assistant automatically runs perag query "notice period NDA" under the hood, retrieves the relevant chunk(s), and answers based on what's actually in your document — not a guess.

What happens behind the scenes
nda_2024.pdf
    ↓ perag chunk
[chunk1, chunk2, ...]
    ↓ perag embed (local model)
[chunk1+vector, chunk2+vector, ...]
    ↓ perag ingest
(sqlite-vec database)
    ↓ perag query "notice period NDA"
→ Returns: "The agreement shall terminate upon 30 days written notice..."


Key points:
✅ 100% local — no cloud, no account, no API key needed (by default)
✅ Works with Claude Code and other CLI-based AI assistants (1/2)
✅ Privacy-first — your documents never leave your machine
🔗 Composable via UNIX pipes — each step is its own command
```

---

day - 12

## Static Pipelines (DAGs)

### Definition:

A Static Pipeline is a workflow structured as a Directed Acyclic Graph (DAG) where each node is a task and each edge is a dependency (A → B means "A must finish before B starts"). Directed means every edge has a clear direction — data or control flows one way. Acyclic means there are no loops — the pipeline cannot circle back to an earlier stage, guaranteeing it will always terminate. Static means the graph structure is fixed at definition time, not dynamically rewired during execution.

In one sentence: A pre-planned, loop-free dependency graph that guarantees tasks run in the correct order, every time.

Analogy — The Assembly Line Kitchen 🏭

Imagine a high-end sushi restaurant with a conveyor belt. The kitchen is split into stations:

Rice Cooker → Fish Prep → Rolling → Plating → Customer

Directed: Rice must be cooked before rolling. You can't roll sushi without rice.
Acyclic: There's no "send the plated sushi back to the rice cooker" — the belt only moves forward.
Static: The manager designed this layout Monday morning and it stays the same all week. Every order follows the exact same path.

If a customer orders miso soup too, the pipeline forks:
┌→ Rolling → Plating → Customer
Rice Cooker ──┤
└→ Soup Pot → Plating → Customer

The soup and sushi prep can happen in parallel (they're independent), but both must hit "Plating" before reaching the customer. The DAG captures this without any manual coordination.

### Example:

ETL Data Pipeline with Airflow-style DAG

```
from datetime import datetime

# Imagine a simple DAG scheduler checks dependencies
# before running each step.

tasks = {}

def task(name, depends_on=None):
    """Register a pipeline task."""
    tasks[name] = {"depends_on": depends_on or [], "done": False}

def run(name):
    """Run a task if all its dependencies are satisfied."""
    t = tasks[name]
    for dep in t["depends_on"]:
        if not tasks[dep]["done"]:
            print(f"  ⏳ {name} waiting for {dep}...")
            return False
    print(f"  ✅ Running {name}...")
    t["done"] = True
    return True

# ─── Define the static DAG ───
task("extract_csv", depends_on=[])        # Root node: no deps
task("extract_api", depends_on=[])        # Root node: no deps
task("validate",    depends_on=["extract_csv", "extract_api"])
task("transform",   depends_on=["validate"])
task("load",        depends_on=["transform"])

print("🚀 Pipeline Execution:\n")

# ─── Execute in topological order ───
# (In a real DAG scheduler, this is automatic)
while not tasks["load"]["done"]:
    run("extract_csv")
    run("extract_api")
    run("validate")
    run("transform")
    run("load")

print("\n📊 Pipeline: COMPLETE")


Output:
🚀 Pipeline Execution:

  ✅ Running extract_csv...
  ✅ Running extract_api...
  ✅ Running validate...
  ✅ Running transform...
  ✅ Running load...

📊 Pipeline: COMPLETE
```

### Key Concepts:

1. **DAG (Directed Acyclic Graph)**: A graph with nodes (tasks) and edges (dependencies) that has no cycles.
2. **Topological Sort**: An ordering of nodes where every dependency comes before the dependent task.
3. **Idempotency**: Tasks should be able to run multiple times without changing the final result.
4. **State Tracking**: Each task tracks its completion status to avoid redundant work.

### Real-world Analogy:

Imagine a construction project where:
- Tasks are like building steps (foundation, framing, plumbing, etc.)
- Dependencies are like "must be done before" relationships
- The DAG scheduler is like a foreman ensuring steps happen in theß correct order

This pattern is used in:
- Data pipelines (Apache Airflow, Luigi)
- Build systems (Make, Bazel)
- Workflow engines (AWS Step Functions, Prefect)

---

day - 15

## WASI-NN (WebAssembly System Interface for Neural Networks)

### Definition:

WASI-NN is a standardized API specification that lets WebAssembly (WASM) modules perform neural network inference by offloading the computation to native backends — OpenVINO, ONNX Runtime, TensorFlow Lite, or PyTorch — through a clean, host-provided interface.

Normally, running ML inference inside WASM means either:
❌ Compiling the entire ML framework (hundreds of MB) to WASM — slow and bloated.
❌ Writing manual bindings per platform — fragile and non-portable.

WASI-NN solves this by defining a small, standard set of functions (load, init_execution_context, set_input, compute, get_output) that any WASM module can call, and any WASM runtime (Wasmer, Wasmtime, WAMR) can implement by plugging in a native backend. The WASM module stays small and portable; the heavy lifting happens on the host's GPU/NPU/CPU.

Analogy — The Film Lab 📸

Imagine you're a photographer in the 1990s. Your camera has a tiny film roll (the WASM module). It captures the raw shot (input data) and specs like "develop with Fuji process, print 4×6 glossy." You drop the roll at a film lab (the WASM runtime with WASI-NN). The lab has:

A Fuji machine (OpenVINO backend)
A Kodak machine (ONNX Runtime backend)
A printing press (PyTorch backend)

You don't care which machine does the work — you just hand over the film and a memo ("inference request"). The lab picks the right machine, runs the chemistry, and hands you back the final prints (output tensor). You never set foot in the machine room.

The film roll stays tiny. The lab does the heavy equipment work. That's WASI-NN.

### Example:

Running Image Classification via WASI-NN

```
;; WASM module using WASI-NN to classify an image
;; The module sends bytes → gets back a label index

(module
  ;; Import WASI-NN functions provided by the host runtime
  (import "wasi_nn" "load"
    (func $load (param $bytes i32) (param $len i32)
                (result i32)))        ;; returns graph handle

  (import "wasi_nn" "init_execution_context"
    (func $init_ctx (param $graph i32)
                    (result i32)))    ;; returns context handle

  (import "wasi_nn" "set_input"
    (func $set_input (param $ctx i32) (param $tensor i32))  ;; void

  (import "wasi_nn" "compute"
    (func $compute (param $ctx i32)))                       ;; void

  (import "wasi_nn" "get_output"
    (func $get_output (param $ctx i32) (param $out i32)))   ;; void

  ;; Memory for input image (224×224×3 = 150,528 bytes)
  (memory (export "memory") 1)

  ;; ─── Inference Flow ───
  (func (export "classify") (param $img_offset i32) (param $img_len i32)
                            (result i32)

    ;; Step 1: Load the model from embedded bytes
    ;;        (model stored in WASM data section at offset 0x1000)
    i32.const 0x1000      ;; model bytes start
    i32.const 50000       ;; model size in bytes
    call $load
    local.set $graph      ;; graph handle

    ;; Step 2: Create an execution context
    local.get $graph
    call $init_ctx
    local.set $ctx

    ;; Step 3: Set the input tensor (the image)
    local.get $ctx
    local.get $img_offset
    call $set_input

    ;; Step 4: Run inference
    local.get $ctx
    call $compute

    ;; Step 5: Read the result (index of highest-probability class)
    local.get $ctx
    i32.const 0x2000      ;; write output tensor here
    call $get_output

    ;; Return the top class index
    i32.load 0x2000
  )
)
```

---

day - 16

## HNSW (Hierarchical Navigable Small World)

### Definition:

HNSW is a graph-based Approximate Nearest Neighbor (ANN) search algorithm that organizes high-dimensional vectors (embeddings) into a hierarchy of navigable small-world graphs. It builds on two ideas:

1. **Navigable Small World (NSW)** — A graph where each node is connected to its nearest neighbors, creating short paths between any two nodes (the "small world" property: you can reach any node in a few hops).
2. **Multi-Layer Hierarchy** — The graph is stratified into layers. The topmost layers are sparse, containing only a few nodes with long-range edges (coarse search). The bottom layer contains all nodes with short, precise edges (fine search).

**How search works:** Starting from the top layer, you greedily navigate to the nearest node, then descend to the layer below using that node as the entry point. Repeat until you reach the bottom layer, where you perform a fine-grained search among the closest neighbors. This is like searching a map at different zoom levels — first pan across continents, then zoom into a city, then find the exact street.

**Why it matters:** HNSW is the de facto standard for vector search today. It powers vector databases like Pinecone, Qdrant, Weaviate, and pgvector's ivfflat alternatives. It achieves search complexity of O(log N) with high recall — orders of magnitude faster than brute-force k-NN.

Analogy — Google Maps Zoom Levels 🗺️

Imagine you're looking for a specific coffee shop in Jakarta.

- **Top layer (zoom level 1):** You see only major cities — Jakarta, Surabaya, Bandung. You click Jakarta.
- **Middle layer (zoom level 5):** You see Jakarta's districts — Menteng, Kemang, Kelapa Gading. You click Menteng.

### Example:

Vector Search with a Minimal HNSW Implementation

```python
import numpy as np
import heapq
import random

class SimpleHNSW:
    """Minimal HNSW for learning — not production-grade."""

    def init(self, dim: int, M: int = 16, ef_construction: int = 200):
        self.dim = dim
        self.M = M
        self.ef_construction = ef_construction
        self.nodes = []
        self.graphs = []
        self.max_level = 0
        self.enter_point = None

    def _random_level(self) -> int:
        level = 0
        while random.random() < 0.5 and level < 16:
            level += 1
        return level

    def _search_layer(self, query, entry, ef, layer):
        """Greedy search on a single layer."""
        visited = {entry}
        dist_entry = np.linalg.norm(query - self.nodes[entry])
        candidates = [(dist_entry, entry)]
        heapq.heapify(candidates)
        result = [(dist_entry, entry)]

        while candidates:
            dist_c, c = heapq.heappop(candidates)
            furthest = max(result, key=lambda x: x[0])[0]
            if dist_c > furthest:
                break
            for neighbor in self.graphs[layer].get(c, []):
                if neighbor not in visited:
                    visited.add(neighbor)
                    dist_n = np.linalg.norm(query - self.nodes[neighbor])
                    furthest = max(result, key=lambda x: x[0])[0]
                    if dist_n < furthest or len(result) < ef:
                        heapq.heappush(candidates, (dist_n, neighbor)) (2/4)
result.append((dist_n, neighbor))
                        result = heapq.nsmallest(ef, result, key=lambda x: x[0])
        return result

    def insert(self, vector):
        node_id = len(self.nodes)
        self.nodes.append(vector)
        level = self._random_level()

        while len(self.graphs) <= level:
            self.graphs.append({})
        for l in range(level + 1):
            self.graphs[l][node_id] = set()

        if self.enter_point is None:
            self.enter_point = node_id
            self.max_level = level
            return

        curr_entry = self.enter_point
        for l in range(self.max_level, level, -1):
            result = self._search_layer(vector, curr_entry, 1, l)
            curr_entry = result[0][1]

        for l in range(min(level, self.max_level), -1, -1):
            result = self._search_layer(vector, curr_entry, self.efconstruction, l)
            neighbors = [n for , n in result[:self.M]]
            self.graphs[l][node_id] = set(neighbors)
            for n in neighbors:
                self.graphs[l][n].add(node_id)
                if len(self.graphs[l][n]) > self.M:
                    n_neighbors = list(self.graphs[l][n])
                    n_dists = [(np.linalg.norm(self.nodes[n] - self.nodes[nn]), nn)
                               for nn in n_neighbors]
                    ndists.sort(key=lambda x: x[0])
                    self.graphs[l][n] = {nn for , nn in n_dists[:self.M]}
            curr_entry = result[0][1]

        if level > self.max_level:
            self.max_level = level
            self.enter_point = node_id

    def search(self, query, k=5):
        if self.enter_point is None:
            return []
        curr_entry = self.enter_point
        for l in range(self.max_level, 0, -1):
            result = self._search_layer(query, curr_entry, 1, l)
            curr_entry = result[0][1]
        result = self._search_layer(query, curr_entry, k, 0) (3/4)
return [(dist, nid) for dist, nid in heapq.nsmallest(k, result, key=lambda x: x[0])]

```

---

day - 17

## FinOps (Cloud Financial Operations)

### Definition:

FinOps (a portmanteau of **Finance** and **DevOps**) is an operational framework and cultural practice that brings engineering, finance, and business teams together to manage cloud spending responsibly. It transforms cloud cost from a passive monthly surprise into an active engineering discipline — treating cost efficiency as a first-class concern alongside performance, scalability, and reliability, not as an afterthought discovered when the invoice arrives.

The core idea is simple but powerful: **engineers who provision cloud resources should understand what those resources cost — and be empowered to optimize them.** Instead of finance teams sending a spreadsheet after the fact, FinOps makes cost visibility part of the engineering workflow.

The FinOps Foundation defines three iterative phases:

1. **Inform** — Visibility and allocation. Read the bill, tag resources, identify top cost drivers, establish a baseline.
2. **Optimize** — Efficiency through action. Right-size over-provisioned resources, move cold data to cheaper tiers, use Spot instances, purchase commitments in the correct sequence.
3. **Operate** — Continuous improvement through automation. Automate orphan cleanup, add cost checks to CI/CD pipelines, build chargeback/showback models, present unit-cost metrics to leadership.

**Why sequence matters:** The most expensive mistake in cloud cost management is buying Savings Plans or Reserved Instances *before* right-sizing. You end up paying a discounted price for waste — locked in for 1–3 years. The correct order is: right-size first, *then* commit.

Analogy — The Grocery Budget 🛒

Imagine you and your roommates share a kitchen. Every week, someone goes to the supermarket and buys whatever they want — organic avocados, premium ice cream, name-brand cereal — and puts it on a joint credit card. At the end of the month, the bill arrives and everyone is shocked. Nobody knows who bought what or why it's so expensive. Some items spoil before anyone eats them.

**FinOps is the solution:** You start by reading the receipt (inform). You tag each item with the owner's name so you know who spent what (tagging). You notice someone has been buying 5 bags of chips every week but only eating 2 — so you buy smaller portions (right-sizing). You switch from name-brand to store-brand on staples (Spot instances). You set a rule: "no single purchase over $50 without group approval" (cost-aware code review). You automate a weekly waste report — expired milk, moldy bread — that flags what to stop buying (orphaned resource cleanup). Now the kitchen runs efficiently, everyone knows their share, and there's no monthly shock.

### Example:

Building a Cost-Aware Infrastructure Pipeline — From Baseline to Block

This example shows a complete mini-FinOps workflow for an AWS environment, following the article's 4-stage roadmap in practice.

**Step 1: Establish a Baseline (Inform)**

Pull last month's cost broken down by service. Save this before any optimization — you cannot measure progress without a baseline.
bash
Save as aws-baseline-2026-05.txt — compare every future month against this
aws ce get-cost-and-usage \
--time-period Start=2026-05-01,End=2026-06-01 \
--granularity MONTHLY \
--group-by Type=DIMENSION,Key=SERVICE \
--metrics UnblendedCost \
--query 'ResultsByTime[0].Groups[*].{Service:Keys[0],Cost:Metrics.UnblendedCost.Amount}' \
--output table | sort -k3 -rn
**Step 2: Find Right-Sizing Candidates (Optimize)**

Run a Python audit to detect EC2 instances running below 20% average CPU for 14 days. These are likely over-provisioned.
python
import boto3
from datetime import datetime, timedelta

def find_rightsizing_candidates(region='us-east-1'):
ec2 = boto3.client('ec2', region_name=region)
cw = boto3.client('cloudwatch', region_name=region)

reservations = ec2.describe_instances(
Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
...

---

day - 18

## Architectural Decision Record (ADR)

### Definition:

- Short, lightweight document that captures a significant architectural 
  decision + context + rationale + trade-offs
- "Document the decision, not the details"
- Lifecycle: Proposed → Accepted → Deprecated → Superseded
- Analogi: Ship's Captain Log — for the next captain who takes the helm 🚢

### Example:

```
Full ADR document (ADR-001) for choosing PostgreSQL over DynamoDB/MongoDB 
for a User Service migration, including:
- Context (2.5M users, strong consistency needed)
- Decision (PostgreSQL 16 on RDS)
- Alternatives Considered (DynamoDB, MongoDB — with honest pros/cons)
- Consequences (positive + negative — no perfect decision)
- Related Decisions (links to ADR-000, ADR-003)
```

---

day - 19

## TSRX (TypeScript Render Extensions)

### Definition:

- TypeScript language extension by Dominic Gannaway (ex-React/Svelte core, 
  creator of Inferno, Lexical, Ripple)
- "Spiritual successor to JSX" — statement-based instead of expression-based
- Framework-agnostic: compiles to React, Preact, Solid, Vue, Ripple
- Key features: @if/@else, @for, scoped <style>, conditional hooks, prop shorthands
- Not a runtime — pure compile-time: .tsrx → AST → framework-specific .tsx
- Status: Beta, MIT license
- Analogi: Universal Translator — write once, output idiomatic code for any framework 🌐

### Example:

```
UserProfile.tsrx component showcasing:
  - @if/@else for auth state (signed in vs signed out)
  - @for for rendering recent posts list
  - Scoped <style> block auto-hashed by compiler
  - Conditional hooks (useUser inside @else — compiler generates child 
    component to satisfy Rules of Hooks)
  - Side-by-side with compiled React output
```

### Key Concepts:
- Statement-based > expression-based (compilers can optimize better)
- Framework-agnostic via plugin architecture
- Conditional hooks = no more manual component splitting for hook rules
- Incremental adoption (one .tsrx at a time)
