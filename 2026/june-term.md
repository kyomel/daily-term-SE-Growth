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
