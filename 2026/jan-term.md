day - 1

## Principle of Least Surprise

### Definition:

Principle of Least Surprise (also called Principle of Least Astonishment) is a design principle stating that a system should behave in a way that users expect, minimizing confusion and unexpected outcomes.

In simple terms: Things should work the way people naturally think they work.

**Why It Matters**  
When software violates this principle, users:

- Make mistakes more often
- Feel frustrated
- Take longer to learn the system
- Lose trust in the interface

### Examples:

Good (Follows POLS)

```
# Expected behavior - delete() actually deletes
def delete(item):
    item.clear()  # Actually removes the data
    return True   # Confirms deletion

user = {"name": "Alice"}
delete(user)
# User expects: user is deleted ✓
# Reality: user is deleted ✓
```

---
