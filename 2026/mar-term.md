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

day - 5

## Union Find (DSU)

### Definition:

Union Find (also known as Disjoint Set Union / DSU) is a data structure that efficiently tracks which elements belong to the same group, and supports two core operations — merging groups and checking if two elements are in the same group.

Think of it as managing a collection of non-overlapping groups, where you can quickly unite two groups or ask "are these two things connected?"

Simple Analogy
👥 Imagine a school with students forming friend groups. At first, everyone is their own group. As friendships form, groups merge together. Union Find lets you instantly answer "Are Alice and Bob in the same friend group?" — even after hundreds of merges.

The 2 Core Operations
| Operation | Description |
|-----------|-------------|
| find(x) | Find which group/set element x belongs to (returns the root/representative) |
| union(x, y) | Merge the groups containing x and y into one |

### Example:

Imagine checking if computers in a network are connected

```
Always attach the smaller tree under the larger tree to avoid tall unbalanced chains:


Bad (no rank):        Good (with rank):
  0                       0
  |                     / | \
  1                    1  2  3
  |
  2
  |
  3
Time Complexity
Operation	Naive	With Both Optimizations
find(x)	O(n)	O(α(n)) ≈ O(1)
union(x, y)	O(n)	O(α(n)) ≈ O(1)
α(n) is the inverse Ackermann function — grows so slowly it's effectively constant for all practical input sizes.
```

---

day - 6

## Bit Flips

### Definition:

A Bit Flip is when a single binary digit (bit) unexpectedly changes its value — from 0 to 1, or from 1 to 0 — due to hardware faults, radiation, or other external forces, causing silent data corruption without the program knowing.

A bit flip is a single letter typo in your computer's memory — one character changes, and the meaning of the entire word shifts.

Simple Analogy
🎲 Imagine writing the number 6 on a whiteboard in binary: 0110. A cosmic ray hits the board and the first digit changes to 1, making it 1110 — which is 14. Nobody erased the board intentionally, but the number is now completely wrong and you'd never know unless you double-checked.

Binary Basics (Quick Refresher)
| Decimal | Binary |
|---------|--------|
| 6 | 0110 |
| 14 | 1110 |

A single bit flip in position 4: 0110 → 1110 = 6 becomes 14 instantly.

$$
\begin{align*}
0110_2 &= 6_{10} \\
\text{bit flip} \\
1110_2 &= 14_{10}
\end{align*}
$$

​

What Causes Bit Flips?
| Cause | Description |
| ----- | ----------- |
| ☄️ Cosmic rays | High-energy particles from space hit memory cells and flip bits |
| ⚡ Voltage glitches | Power supply instability causes incorrect bit states |
| 🌡️ Overheating | Heat degrades electronic components, causing unreliable behavior |
| 🏭 Manufacturing defects | Impurities in memory chips lead to unstable bits |
| 🔧 Aging hardware | Components degrade over time and become unreliable |
| 🦠 Software bugs | Code accidentally overwrites a wrong memory location |
| 🔐 Malicious attacks | Hackers intentionally flip bits to exploit system vulnerabilities |

**Visual — What a Bit Flip Looks Like**

Original data in memory:
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 1 │ 0 │ 1 │ 1 │ 0 │ 1 │ = 109 decimal = ASCII letter 'm'
└───┴───┴───┴───┴───┴───┴───┴───┘

         ☄️ cosmic ray hits bit 5
                   ↓

┌───┬───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 1 │ 0 │ 1 │ 0 │ 0 │ 1 │ = 105 decimal = ASCII letter 'i'
└───┴───┴───┴───┴───┴───┴───┴───┘

'm' silently became 'i' — with no error thrown ❌

### Example:

In 2003, a voting machine in Belgium recorded 4,096 extra votes for one candidate out of nowhere. Investigators traced it back to a single bit flip in memory — caused by cosmic radiation.

```
Correct vote count:    13,000  =  ...0011001011010000
After bit flip:        17,096  =  ...0100001011010000
                            ↑
                     this bit flipped → 4,096 extra votes added
A single cosmic ray decided an election. 🗳️
```

---

day - 9

## Prim's Algorithm

### Definition:

Prim's Algorithm is a greedy algorithm used to find the Minimum Spanning Tree (MST) of a weighted, undirected graph. It builds the MST by starting from any node and greedily adding the cheapest edge that connects a visited node to an unvisited node — one edge at a time — until all nodes are included.

A Minimum Spanning Tree is a subset of edges that connects all nodes in a graph with the minimum possible total edge weight, without forming any cycles.

💡 Simple Analogy
Imagine you're a city planner connecting 5 towns with roads. You want every town connected, but you want to spend as little money as possible on road construction. Prim's Algorithm says: "Start at one town, always build the cheapest road to the nearest unconnected town next."

⚙️ How It Works — Step by Step
| Step | Action |
|------|--------|
| 1 | Start at any node — mark it as visited |
| 2 | Look at all edges from visited nodes to unvisited nodes |
| 3 | Pick the cheapest edge |
| 4 | Add that edge and its new node to the MST |
| 5 | Repeat steps 2–4 until all nodes are visited |

### Example:

Graph Setup

```
        2       3
   A ——————B ——————C
   |       |       |
 6 |     8 |     7 |
   |       |       |
   D——————E ——————F
        5       4

Edge List with Weights
| Edge | Weight |
|------|--------|
| A–B  | 2      |
| A–D  | 6      |
| B–C  | 3      |
| B–E  | 8      |
| C–F  | 7      |
| D–E  | 5      |
| E–F  | 4      |

🔍 Prim's Walkthrough — Starting at Node A
Step 1 — Start at A


Visited: [A]
Available edges: A–B(2), A–D(6)
✅ Pick cheapest → A–B (weight 2)
Step 2 — Add B


Visited: [A, B]
Available edges: A–D(6), B–C(3), B–E(8)
✅ Pick cheapest → B–C (weight 3)
Step 3 — Add C


Visited: [A, B, C]
Available edges: A–D(6), B–E(8), C–F(7)
✅ Pick cheapest → A–D (weight 6)
Step 4 — Add D


Visited: [A, B, C, D]
Available edges: B–E(8), C–F(7), D–E(5)
✅ Pick cheapest → D–E (weight 5)
Step 5 — Add E


Visited: [A, B, C, D, E]
Available edges: C–F(7), E–F(4), B–E(skip–visited)
✅ Pick cheapest → E–F (weight 4)
✅ All nodes visited — MST complete!
```

---
