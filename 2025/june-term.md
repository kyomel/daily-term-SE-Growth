day - 28

## Polya Problem Solving Technique

### definition:

George Polya developed a systematic approach to problem-solving with four key steps:

1. Understand the problem
   *What is the unknown?
   *What are the given data?
   \*What are the conditions?
2. Devise a plan
   *Look for patterns
   *Work backwards
   *Use analogies
   *Break into smaller problems
3. Carry out the plan
   *Execute your strategy
   *Check each step
   \*Be patient and persistent
4. Look back
   *Check the result
   *Can you verify the answer?
   \*Can you use this method elsewhere?

### example:

Problem: At a party with 10 people, if everyone shakes hands with everyone else exactly once, how many handshakes occur?

- Step 1: Understand 🤔
  Unknown: Total number of handshakes
  Given: 10 people, each shakes hands once with every other person
  Condition: No person shakes hands with themselves
- Step 2: Plan 📊
  Strategy: Use combinations formula
  Each handshake involves exactly 2 people
  Choose 2 people from 10: C(10,2)
- Step 3: Execute 🔢  
  C(10,2) = 10!/(2!(10-2)!) = (10 × 9)/2 = 45
- Step 4: Verify ✨
  - Check: Person 1 shakes 9 hands, Person 2 shakes 8 remaining, etc.
  - Sum: 9+8+7+6+5+4+3+2+1 = 45 ✅
  - Answer: 45 handshakes

---

day - 30

## gRPC

### definition:

gRPC is a high-performance Remote Procedure Call (RPC) framework that can run in various environments and efficiently connect services in and across data centers.

By default, gRPC employs Protocol Buffers to define message structures and RPC services.

### example:

There are four types of gRPC service methods:

1. **Unary RPC**

The client sends a request and receives a response from the server. Similar with REST API style.

```proto
rpc SayHello(HelloRequest) returns (HelloResponse);
```

2. **Server streaming RPC**

The client sends a request and receives a stream of responses from the server. The client reads the responses until there are no responses.

```proto
rpc GetBooks(GetBooksRequest) returns (stream GetBooksResponse) {};
```

3. **Client streaming RPC**

The client sends a stream of requests and receives a response from the server. When the client finished sending the requests, the server handle the requests and returns a responses.

```proto
rpc AddBatchBook(stream AddBatchBookRequest) returns (AddBatchBookResponse) {};
```

4. **Bidirectional streaming RPC**

The client and the server send a sequence of messages using a read-write stream. The two streams operate independently in any order. For example, the server wait to receive all the client messages before returning the responses.

```proto
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
```

This is the example of [gRPC implementation in Go](https://github.com/nadirbasalamah/books-grpc). This example covers the basic usages of gRPC.

---
