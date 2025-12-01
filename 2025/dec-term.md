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
