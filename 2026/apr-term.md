day - 1

## Garbage First Garbage Collector (G1GC)

### Definition:

Garbage First Garbage Collector (G1GC) is a region-based, server-oriented garbage collector introduced in Java 7 and made the default GC in Java 9 — designed to provide high throughput with predictable, configurable pause times by dividing the heap into equal-sized regions, prioritizing collection of regions with the most garbage first (hence "Garbage First"), and incrementally reclaiming memory without requiring a full heap collection.

G1GC was built to replace the Concurrent Mark Sweep (CMS) and Parallel GC collectors for applications that need both large heaps AND low, predictable pause times — the two goals that older collectors forced you to choose between.

— openjdk.org

The Fundamental Concept — Regions
The single most important thing to understand about G1GC:


Traditional GC Heap Layout:
┌──────────────────┬──────────────┬──────────────────────────┐
│   Young Gen      │  Survivor    │      Old Generation       │
│   (Eden)         │   Spaces     │                          │
│                  │              │                          │
│  New objects     │ Surviving    │  Long-lived objects       │
│  created here    │ objects      │                          │
└──────────────────┴──────────────┴──────────────────────────┘
Fixed, contiguous regions — must collect entire generation at once

G1GC Heap Layout:
┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ E  │ S  │ O  │ O  │ E  │ O  │ S  │ E  │ H  │ O  │ E  │ O  │
├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
│ O  │ E  │ O  │ E  │ O  │ O  │ E  │ O  │ O  │ E  │ S  │ O  │
├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
│ O  │ O  │ E  │ H  │ O  │ E  │ O  │ O  │ E  │ O  │ O  │ E  │
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘

E = Eden region    (new objects)
S = Survivor region (surviving young objects)
O = Old region     (long-lived objects)
H = Humongous region (very large objects, > 50% of region size)

Key differences:
  ✅ Heap split into 2,048 equal-sized regions (1MB–32MB each)
  ✅ Any region can play any role — flexible assignment
  ✅ Regions collected INDEPENDENTLY — no full heap needed
  ✅ G1GC picks the regions with MOST garbage first

### Example:

E-Commerce Order Processing Service
A Java-based order processing service handles variable traffic — quiet at night, massive spikes during flash sales. Here is how G1GC behavior changes across scenarios.

```
Order processing service benchmark:
  8GB heap, 5,000 orders/minute, 4-hour test

                    Parallel GC    CMS        G1GC
                    ───────────    ───────    ────────────────────
  Max pause:        4,200ms ❌    850ms ⚠️   98ms  ✅
  Avg pause:        1,800ms ❌    120ms ✅    45ms  ✅
  Throughput:       97%    ✅     91%   ⚠️   94%   ✅
  Heap fragmented:  No     ✅     YES   ❌    No    ✅
  Full GC events:   2      ⚠️    8     ❌    0     ✅
  Predictable?:     No     ❌     No    ❌    YES   ✅
  Tuning effort:    Low    ✅     High  ❌    Medium ✅

  Orders lost to timeout (pause > 500ms = user-facing error):
  Parallel GC:      1,240 orders lost ❌
  CMS:                180 orders lost ⚠️
  G1GC:               0   orders lost ✅
```

---