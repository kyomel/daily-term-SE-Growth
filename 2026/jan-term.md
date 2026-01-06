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

day - 2

## Sieve Algorithm or Sieve of Eratosthenes

### Definition:

The Sieve Algorithm (specifically Sieve of Eratosthenes) is an ancient, efficient method for finding all prime numbers up to a given limit. It works by systematically "crossing out" multiples of each prime, leaving only primes unmarked.

Key idea: Instead of testing each number individually, eliminate all non-primes in batches.

**Why It Matters**

- Fast: Much faster than checking each number individually
- Simple: Easy to understand and implement
- Efficient: Great for finding many primes at once (up to millions)

**How It Works (Simple Steps)**

- List all numbers from 2 to n
- Start with the first unmarked number (it's prime)
- Cross out all its multiples (they're composite)
- Repeat with the next unmarked number
- Continue until done

### Example:

Find all primes up to 30:

```
Step 1: List numbers 2-30
2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30

Step 2: 2 is prime, cross out multiples of 2
2  3  ✗  5  ✗  7  ✗  9  ✗  11 ✗  13 ✗  15 ✗  17 ✗  19 ✗  21 ✗  23 ✗  25 ✗  27 ✗  29 ✗

Step 3: 3 is prime, cross out multiples of 3
2  3  ✗  5  ✗  7  ✗  ✗  ✗  11 ✗  13 ✗  ✗  ✗  17 ✗  19 ✗  ✗  ✗  23 ✗  25 ✗  ✗  ✗  29 ✗

Step 4: 5 is prime, cross out multiples of 5
2  3  ✗  5  ✗  7  ✗  ✗  ✗  11 ✗  13 ✗  ✗  ✗  17 ✗  19 ✗  ✗  ✗  23 ✗  ✗  ✗  ✗  ✗  29 ✗

Step 5: Continue with 7, 11, 13...

Final Result (primes): 2, 3, 5, 7, 11, 13, 17, 19, 23, 29
```

---

day - 5

## AutoML Frameworks

### Definition:

AutoML (Automated Machine Learning) Frameworks are software tools that automatically handle the complex, time-consuming parts of building machine learning models. They automate tasks like data preprocessing, feature engineering, algorithm selection, and hyperparameter tuning.

Key idea: You provide the data, AutoML does the hard work, and you get a trained model.

**Why It Matters**

Traditional ML requires:

- Deep expertise in algorithms
- Manual feature engineering
- Trial-and-error for best parameters
- Hours/days of experimentation

AutoML provides:

- Automated model selection
- Works for non-experts
- Faster time to deployment
- Often better results than manual tuning

### Example:

```
from autosklearn.classification import AutoSklearnClassifier

# Just 3 lines!
automl = AutoSklearnClassifier(time_left_for_this_task=3600)
automl.fit(X_train, y_train)
predictions = automl.predict(X_test)

# AutoML automatically tried 100+ model configurations!
```

Popular AutoML Frameworks
| Framework | Description | Best For |
|-----------------|-----------------------------------|------------------------------|
| Auto-sklearn | Built on scikit-learn | Tabular data, Python users |
| H2O AutoML | Enterprise-grade AutoML | Business applications |
| Google AutoML | Cloud-based, no-code | Beginners, quick prototypes |
| TPOT | Uses genetic algorithms | Research, optimization |
| AutoKeras | Deep learning focused | Image/text data |
| PyCaret | Low-code ML library | Fast experimentation |

---

day - 6

## Spot Instances

### Definition:

Spot Instances are heavily discounted cloud computing resources (virtual machines) that use spare capacity from cloud providers. They can save you up to 90% compared to regular On-Demand pricing, but come with a trade-off: the cloud provider can reclaim them at any time with short notice (30 seconds to 2 minutes).

Key idea: Cheap cloud servers that can be interrupted when the provider needs the capacity back.

**Why It Matters**
💰 Huge savings: 60-90% cheaper than regular pricing
⚡ Same performance: Identical to standard instances
⚠️ Interruptible: Can be terminated with minimal warning
🎯 Perfect for flexible workloads: Not for critical applications

**How It Works**

Cloud Provider's Perspective:

1. Has 1000 servers in data center
2. Only 600 are being used by regular customers
3. Offers remaining 400 at steep discount as "Spot Instances"
4. If regular customers need them → reclaim from Spot users

Your Perspective:

1. Rent cheap server (90% off!)
2. Run your workload
3. Might get interrupted (2-minute warning)
4. Save tons of money! 💵

### Example:

Scenario: Video Rendering Farm

```
Option 1: On-Demand Instances

Cost: $1.00/hour per server
Workload: Render 100 videos (10 hours)
Servers needed: 10
Total cost: $1.00 × 10 servers × 10 hours = $100
Reliability: ✅ Guaranteed

Option 2: Spot Instances

Cost: $0.10/hour per server (90% discount!)
Workload: Render 100 videos (10 hours)
Servers needed: 10
Total cost: $0.10 × 10 servers × 10 hours = $10
Reliability: ⚠️ Might be interrupted
Savings: $90 (90% saved!) 🎉

Result: If 2 out of 10 servers get interrupted and you restart them, you still save ~$80!
```

---
