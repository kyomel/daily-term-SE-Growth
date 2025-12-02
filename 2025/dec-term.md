day - 1

## Dark Deployments

### Definition:

Dark Deployments (also called Dark Launches) is a deployment strategy where new features or code changes are released to production but remain hidden from regular users. The new functionality runs silently in the background, allowing teams to test performance, stability, and behavior in a real production environment without any user impact.

**Key characteristics:**

- Code deployed but invisible to users
- Tests with real production traffic
- Zero user-facing risk during testing
- Enables gradual rollout when ready
- Validates performance before full release

### Example:

Scenario: E-commerce site wants to deploy a new search algorithm

```
Traditional Deployment (Risky):

Monday: Deploy new search algorithm
↓
Users immediately see different results
↓
Bug discovered: "laptop" search returns "shoes"
↓
Customer complaints flood support
↓
Emergency rollback required
Dark Deployment (Safe):

Monday: Deploy new search algorithm (hidden)
↓
Users still see OLD search results
↓
NEW algorithm runs silently in background
↓
Compare results, measure performance
↓
Fix issues without any user impact
↓
Friday: Confident? Make it visible to users
```

---

day - 2

## Tromboning

### Definition:

Tromboning is a network inefficiency where traffic takes an unnecessarily long, looping path to reach its destination - going far away only to come back to a nearby location. The name comes from the shape of a trombone slide, where air travels out and back. This wastes bandwidth, increases latency, and adds unnecessary load on network devices.

**Key characteristics:**

- Traffic travels longer distance than necessary
- Increases network latency significantly
- Wastes bandwidth and resources
- Common in data centers, VPNs, and cloud environments
- Often caused by poor network design or security requirements

### Example:

Office Communication Problem
Scenario: Two employees in the same building want to share a file

```
❌ Tromboning (Inefficient):

Alice (Floor 1)                    Bob (Floor 2)
     │                                  ▲
     │                                  │
     ▼                                  │
┌─────────────────────────────────────────────────────┐
│                 LOCAL NETWORK                       │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │   CORPORATE     │
            │   HEADQUARTERS  │    ← 500 miles away!
            │   (Firewall)    │
            └─────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────┐
│                 LOCAL NETWORK                       │
└─────────────────────────────────────────────────────┘
                     │
                     ▼
              Bob receives file
Path Traveled: 1000+ miles round trip
Latency: 200ms
Result: Slow file transfer between neighbors!

✅ Optimal Path (No Tromboning):

Alice (Floor 1)
     │
     │  Direct local path
     │
     ▼
Bob (Floor 2)
Path Traveled: 50 feet
Latency: 2ms
Result: Fast, efficient transfer!
```

---
