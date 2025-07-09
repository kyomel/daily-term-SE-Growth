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

day - 3

## Database Normalization

### Definition:

Database Normalization is the process of organizing data in a database to reduce redundancy and improve data integrity by dividing large tables into smaller, related tables and defining relationships between them.
**Normalization Forms:**
1NF: Each column contains atomic values, no repeating groups
2NF: 1NF + All non-key attributes fully depend on primary key
3NF: 2NF + No transitive dependencies
BCNF: 3NF + Every determinant is a candidate key

### Example:

**Student Database Normalization**

Before Normalization:

```
Students Table:
+----+----------+-------------+------------------+----------------+
| ID | Name     | Courses     | Instructors      | Departments    |
+----+----------+-------------+------------------+----------------+
| 1  | John     | Math, CS    | Smith, Johnson   | Math, CS       |
| 2  | Sarah    | Physics     | Brown            | Physics        |
+----+----------+-------------+------------------+----------------+
```

After 1NF(First Normal Form):

```
Students Table:
+----+----------+
| ID | Name     |
+----+----------+
| 1  | John     |
| 2  | Sarah    |
+----+----------+

Student_Courses Table:
+----+---------+------------+------------+
| ID | Course  | Instructor | Department |
+----+---------+------------+------------+
| 1  | Math    | Smith      | Math       |
| 1  | CS      | Johnson    | CS         |
| 2  | Physics | Brown      | Physics    |
+----+---------+------------+------------+
```

After 2NF & 3NF:

```
Students Table:
+----+----------+
| ID | Name     |
+----+----------+
| 1  | John     |
| 2  | Sarah    |
+----+----------+

Courses Table:
+-----------+-------------+----------------+
| Course_ID | Course_Name | Department_ID  |
+-----------+-------------+----------------+
| 101       | Math        | 1              |
| 102       | CS          | 2              |
| 103       | Physics     | 3              |
+-----------+-------------+----------------+

Departments Table:
+---------------+-----------------+
| Department_ID | Department_Name |
+---------------+-----------------+
| 1             | Mathematics     |
| 2             | Computer Science|
| 3             | Physics         |
+---------------+-----------------+

Instructors Table:
+---------------+----------+---------------+
| Instructor_ID | Name     | Department_ID |
+---------------+----------+---------------+
| 1             | Smith    | 1             |
| 2             | Johnson  | 2             |
| 3             | Brown    | 3             |
+---------------+----------+---------------+

Enrollments Table:
+------------+-----------+---------------+
| Student_ID | Course_ID | Instructor_ID |
+------------+-----------+---------------+
| 1          | 101       | 1             |
| 1          | 102       | 2             |
| 2          | 103       | 3             |
+------------+-----------+---------------+
```

reference: [Link-1](https://www.geeksforgeeks.org/dbms/introduction-of-database-normalization/)
[Link-2](https://www.datacamp.com/tutorial/normalization-in-sql)

---

day - 4

## Test Driven Development

### Definition:

Test Driven Development (TDD) is a software development methodology where you write tests before writing the actual code. It follows a simple cycle: Red → Green → Refactor.
TDD Cycle:

- Red: Write a failing test
- Green: Write minimal code to make the test pass
- Refactor: Improve the code while keeping tests passing

### Example:

Calculator Addition Function
**Step 1: Red - Write Failing Test**

```
import unittest

class TestCalculator(unittest.TestCase):
    def test_add_two_numbers(self):
        calc = Calculator()
        result = calc.add(2, 3)
        self.assertEqual(result, 5)

    def test_add_negative_numbers(self):
        calc = Calculator()
        result = calc.add(-1, -2)
        self.assertEqual(result, -3)

if __name__ == '__main__':
    unittest.main()
```

**Step 2: Green - Write Minimal Code**

```
class Calculator:
    def add(self, a, b):
        return a + b
```

**Step 3: Refactor - Improve Code**

```
class Calculator:
    def add(self, a: int, b: int) -> int:
        """Add two numbers and return the result."""
        if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
            raise TypeError("Both arguments must be numbers")
        return a + b
```

reference: [Link-1](https://martinfowler.com/bliki/TestDrivenDevelopment.html)

---

day - 7

## Lazy Loading

### Definition:

Lazy Loading is a design pattern that delays the initialization or loading of resources until they are actually needed, rather than loading everything upfront. This improves performance by reducing initial load time and memory usage.

**Benefits vs Trade-offs**

- Benefits: Faster initial load, Lower memory usage, Better user experience
- Trade-offs: Potential delay when accessing, More complex implementation, Possible loading indicators needed

### Example:

- Web Development - Image Lazy Loading

```
<!-- Images load only when they come into viewport -->
<img src="placeholder.jpg" data-src="actual-image.jpg" loading="lazy" alt="Example">
```

- JavaScript - Module Lazy Loading

```
// Module is loaded only when function is called
async function loadFeature() {
    const module = await import('./heavy-feature.js');
    return module.default();
}
```

---

day - 8

## Black Box Testing

### Definition:

Black Box Testing is a testing technique that mainly focuses on testing the system functionalities without knowing the actual implementations.

### Example:

There are many testing techniques that actually implement Black Box Testing:

- E2E Testing (End-to-End Testing)
- System Testing
- User Acceptance Testing (UAT)

These are the tool examples for Black Box Testing

| **Tool**   | **Description**                                                                                                                |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Postman    | E2E API tool. Support other protocols like graphQL and gRPC                                                                    |
| Playwright | E2E Web automation testing tool. Using JS / TS as a primary language                                                           |
| Katalon    | E2E Multi-platform automation tool. Suitable for testing across different platforms like API, web and mobile. Free for 30 days |

---

## Eager Loading

### Definition:

Eager Loading is a technique where data or resources are loaded immediately when they are declared or referenced, rather than waiting until they are actually needed. This approach prioritizes upfront loading to ensure data is readily available when accessed.

- **Eager Loading vs Lazy Loading**
  |Aspect |Eager Loading |Lazy Loading |
  |--- |--- |--- |
  |Loading Time |Load immediately/upfront |Load when needed |
  |Memory Usage |Higher initial memory consumption |Lower initial memory usage |
  |Performance |Faster access after loading |Slower first access, faster subsequent |
  |Network Requests |More upfront requests |Fewer initial requests |
  |Use Case |Critical data needed immediately |Optional/conditional data |

### Example:

- Database Query

```
# Eager Loading Example - SQLAlchemy
from sqlalchemy.orm import joinedload

# Load user with all their posts immediately
user = session.query(User)\
    .options(joinedload(User.posts))\
    .filter(User.id == 1)\
    .first()

# Posts are already loaded - no additional query needed
print(user.posts)  # ✅ Data already available
```

- Javascript

```
// Eager Loading Example - JavaScript
class ImageGallery {
    constructor(imageUrls) {
        this.images = [];
        this.loadAllImages(imageUrls); // Load immediately
    }

    loadAllImages(urls) {
        urls.forEach(url => {
            const img = new Image();
            img.src = url; // Load all images upfront
            this.images.push(img);
        });
    }
}

// All images start loading immediately
const gallery = new ImageGallery(['img1.jpg', 'img2.jpg', 'img3.jpg']);
```

---

day - 9

## CARL Interview Method

### Definition:

CARL is a structured interview technique used to evaluate candidates' past experiences and problem-solving abilities through specific storytelling components:

- Context - The situation or background
- Action - What the candidate specifically did
- Result - The outcome achieved
- Learning - What was learned from the experience

### Example:

**Leadership Challenge**

Question: "Tell me about a time when you had to lead a difficult project."

Candidate's CARL Response:

🔍 Context:

"In my previous role as a software team lead, we had a critical e-commerce platform that was experiencing 40% performance degradation during peak hours, affecting customer sales."

⚡ Action:

"I assembled a cross-functional team, implemented daily standups, analyzed the codebase to identify bottlenecks, prioritized database optimization, and coordinated with DevOps for infrastructure scaling."

📊 Result:

"We reduced page load times by 60%, improved system stability to 99.8% uptime, and recovered $2M in potential lost sales within 3 weeks."

📚 Learning:

"I learned the importance of proactive monitoring and that effective leadership requires both technical problem-solving and clear communication across departments."

---
