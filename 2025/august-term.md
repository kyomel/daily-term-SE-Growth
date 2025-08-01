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
