day - 1

## Deadlock

### Definition:

A deadlock is a situation where two or more processes are blocked indefinitely, each waiting for the other to release a resource that they need to continue execution. None of the processes can proceed, creating a circular dependency.

### Example:

- Banking System Scenario

```
# Process A needs Account1 and Account2
# Process B needs Account2 and Account1

Process A:
1. Lock Account1 ✅
2. Wait for Account2 ⏳ (held by Process B)

Process B:
1. Lock Account2 ✅
2. Wait for Account1 ⏳ (held by Process A)

Result: Both processes wait forever! 🔄
```

- Traffic Intersection Analogy
  Imagine four cars at a 4-way intersection:

1. Each car occupies one lane
2. Each needs the lane occupied by the next car
3. No car can move forward
4. Result: Complete gridlock!

---

day - 2

## Behavioral Design Patterns

### Definition:

Behavioral design patterns are a category of design patterns that focus on the interactions and communication between objects. They help define how objects collaborate and distribute responsibility among them, making it easier to manage complex control flow and communication in a system.

### Example:

- Chain of Responsibility
- Command
- Iterator
- Mediator
- Memento
- Observer
- State
- Strategy
- Template Method
- Visitor

reference: [Link-1](https://refactoring.guru/design-patterns/behavioral-patterns)
[Link-2](https://www.geeksforgeeks.org/system-design/behavioral-design-patterns)

---
