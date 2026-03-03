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
