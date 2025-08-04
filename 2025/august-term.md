day - 1

## HTTP Push and Pull

### Definition:

- HTTP Pull: Traditional HTTP model where the client initiates requests to the server to retrieve data. The server responds only when requested.
- HTTP Push: Server-initiated communication where the server sends data to the client without the client explicitly requesting it at that moment.

### Example:

- HTTP Pull:

```
REST API calls
GET /api/users/123
```

- HTTP Push:
  Server-Sent Events (SSE)
  WebSocket
  Push Notifications
  HTTP/2 Server Push

---

## Law of Demeter (LoD)

### Definition:

The Law of Demeter (also known as the Principle of Least Knowledge) is a design guideline that states an object should only communicate with its immediate friends and not with strangers. It promotes loose coupling by limiting the knowledge an object has about other objects.

\*\*Core Rule:"Don't talk to strangers"

### Example:

```
// GOOD: Following LoD - direct communication only
public class OrderProcessor {
    public void processOrder(Customer customer) {
        // Let customer handle its own business
        String city = customer.getCityName();
        double tax = customer.getTaxRate();
    }
}

public class Customer {
    private Address address;

    // Customer knows how to get its city name
    public String getCityName() {
        return address.getCityName();
    }

    // Customer knows how to get tax rate
    public double getTaxRate() {
        return address.getTaxRate();
    }
}
```

---

day - 2

## Space Complexity

### Definition:

Space complexity is the amount of memory an algorithm needs to run as a function of the size of its input.

### Example:

If an algorithm uses a fixed amount of memory regardless of input size, it has O(1) space complexity.
If extra space grows with input, for example, storing an array of size n, it has O(n) space complexity.

Summary:
Space complexity measures an algorithm’s memory usage as input size increases.

---

## Time Complexity

### Definition:

Time complexity measures how the running time of an algorithm increases as the size of the input grows.

### Example:

If an algorithm examines every element in a list of size n, , its time complexity is O(n).
If it uses nested loops through the list, its time complexity is O(n²).

Summary:
Time complexity describes how the execution time of an algorithm grows with input size.

---
