day - 2

## Graceful Degradation

### Definition:

Graceful Degradation is a design principle where a system continues to function at a reduced level when part of it fails — rather than breaking completely.

The goal is to keep core functionality alive, even when things go wrong.

**Simple Analogy**
🏠 A house with a backup generator — when the main power goes out, you don't lose everything. You still get lights and the fridge running, just not the full power grid.

**Visual Concept**

All features working → Some features fail → Core still works
[ ✅ Full Experience ] [ ⚠️ Degraded ] [ 🔴 NOT broken ]

### Example:

Imagine an e-commerce website where a recommendation service goes down.

```
With Graceful Degradation:
User visits product page
        ↓
  Calls Recommendation API  →  ❌ API timeout / down
        ↓
  Fallback: Show static "Popular Items" list instead
        ↓
  Page still loads, user still shops 🛍️

async function getRecommendations(userId) {
  try {
    const res = await fetch(`/api/recommendations/${userId}`);
    return await res.json();
  } catch (error) {
    // 👇 Graceful fallback when service is down
    return getStaticPopularItems();
  }
}
**More Quick Examples**

| Scenario         | Failure            | Graceful Response                          |
| ---------------- | ------------------ | ------------------------------------------ |
| 🗺️ Maps app      | Network slow       | Show cached map tiles already downloaded   |
| 📧 Web email     | Offline            | Still let user draft/edit emails           |
| 🎵 Streaming app | CDN issue          | Drop video quality, don't stop playback    |
| 🌐 Website       | JS fails to load   | Page still readable as plain HTML          |

**Graceful Degradation vs Fault Tolerance**

| Aspect     | Graceful Degradation         | Fault Tolerance                        |
| ---------- | ---------------------------- | -------------------------------------- |
| On failure | Reduced functionality        | No impact, seamless                    |
| Goal       | Keep core alive              | Zero downtime                          |
| Example    | Show cached data             | Automatic failover to backup server    |
```

---

day - 3

## Cosine Similarity

### Definition:

Cosine Similarity is a metric that measures how similar two things are by calculating the cosine of the angle between their vector representations — focusing on direction, not size.

The smaller the angle between two vectors, the more similar they are.

Simple Analogy
🧭 Two people walking — it doesn't matter if one walks faster or slower. If they're walking in the same direction, they're going to the same place. Cosine similarity measures direction alignment, not speed.

The Formula
Cosine Similarity(A,B)=
∥A∥×∥B∥
A⋅B
​

Where:

- A⋅B = dot product of vectors A and B
- ∥A∥ and ∥B∥ = magnitudes (lengths) of each vector

Result Range
| Value | Meaning |
|-------|------------------------------------------|
| 1 | Identical direction (perfectly similar) |
| 0 | Perpendicular (no similarity) |
| -1 | Opposite direction (completely dissimilar) |

### Example:

Text Similarity
Imagine comparing two sentences to see how similar they are.

```
Sentence A: "I love cats"
Sentence B: "I love dogs"
Step 1 — Build a word frequency table
Word	Sentence A	Sentence B
I	1	1
love	1	1
cats	1	0
dogs	0	1
So the vectors are:

A=[1,1,1,0]
B=[1,1,0,1]

Step 2 — Calculate dot product
A⋅B=(1×1)+(1×1)+(1×0)+(0×1)=2

Step 3 — Calculate magnitudes
∥A∥=
1
2
 +1
2
 +1
2
 +0
2

​
 =
3
​
 ≈1.732

∥B∥=
1
2
 +1
2
 +0
2
 +1
2

​
 =
3
​
 ≈1.732

Step 4 — Final score
Cosine Similarity=
1.732×1.732
2
​
 =
3
2
​
 ≈0.667

✅ Score of 0.667 — the sentences are moderately similar (share structure and some words, but differ in one key word)
```

---

day - 4

## JSON Bandwidth Inflation

### Definition:

JSON Bandwidth Inflation is the phenomenon where JSON, being a textual format, tends to be verbose, which results in increased network bandwidth usage and higher latencies. In simple terms — JSON wastes space by sending more bytes than the actual data needs, because of its human-readable, text-based nature.

JSON is designed to be readable by humans, not optimized for machines — and that readability comes at a bandwidth cost.

**Simple Analogy**
📦 Imagine shipping a small gift. Instead of a tight box, you use a giant cardboard box filled with bubble wrap just to label what's inside. The gift is small, but the package is huge. JSON is the oversized box.

**Why Does It Happen?**
JSON is a textual format, which tends to be verbose. Every response must carry:

| Overhead Type           | Example                                 |
| ----------------------- | --------------------------------------- |
| Repeated key names      | "username" repeated in every object     |
| No type compression     | Numbers stored as plain text: "age": 25 |
| Whitespace & formatting | Spaces, newlines, indentation           |
| Redundant structure     | Brackets, quotes, colons everywhere     |

### Example:

Imagine an API returning a list of 3 users

```
Option 1 — Minified JSON (remove whitespace)

[{"id":1,"username":"alice","email":"alice@email.com","age":28,"is_active":true},{"id":2,"username":"bob","email":"bob@email.com","age":34,"is_active":false}]
Option 2 — Restructured (separate keys from values)

{
  "fields": ["id", "username", "email", "age", "is_active"],
  "rows": [
    [1, "alice", "alice@email.com", 28, true],
    [2, "bob",   "bob@email.com",   34, false],
    [3, "carol", "carol@email.com", 22, true]
  ]
}
✅ Keys are declared once, not repeated per row — massive savings at scale!

Option 3 — Switch to Binary Format (Protobuf / MessagePack)

# Binary encoding — not human readable but 20–60% smaller
\x01\x05alice\x11alice@email.com\x1c\x01 ...
```

---
