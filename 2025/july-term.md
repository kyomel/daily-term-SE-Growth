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
