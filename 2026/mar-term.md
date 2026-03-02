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
