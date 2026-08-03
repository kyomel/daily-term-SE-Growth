day - 3

## Stateless MCP

### Definition:

Stateless MCP refers to the design and operation of Model Context Protocol (MCP) servers that do not retain any memory or session state between requests — each request is handled independently, in complete isolation from all previous requests. 

MCP (Model Context Protocol) is the open standard that connects AI models to external tools and data sources. An MCP server exposes tools, resources, and prompts that an AI model can call. Stateless describes how those servers handle requests: no conversation history, no cached context, no session data carried from one call to the next.

The concept comes from the same principle as stateless HTTP, stateless functions, and stateless services: each request contains everything the server needs to process it.

STATEFUL vs. STATELESS MCP SERVER:
═══════════════════════════════════════════════════════════════

  STATEFUL MCP Server (keeps memory between requests):
  ──────────────────────────────────────────────────────────

  Request 1 ──► [MCP SERVER] ──► "I remember you said X"
                    │
                    │  stores session state internally
                    ▼
  Request 2 ──► [MCP SERVER] ──► "Continuing from where
                                    we left off..."

  ┌─────────────────────────────────────────────────────┐
  │  The server remembers:                              │
  │  • Conversation history                             │
  │  • User identity and preferences                    │
  │  • Authentication tokens                            │
  │  • Data from previous requests                      │
  │  • Which client is calling                          │
  │                                                      │
  │  PROS: richer context, continuity                   │
  │  CONS: server-side memory, scaling issues,          │
  │        state can become stale or corrupted          │
  └─────────────────────────────────────────────────────┘


  STATELESS MCP Server (no memory between requests):
  ──────────────────────────────────────────────────────

  Request 1 ──► [MCP SERVER] ──► process → respond
                    │
                    │  discards everything after responding
                    ▼
  Request 2 ──► [MCP SERVER] ──► process → respond
                                 (same as if request 1
                                  never happened)

  ┌─────────────────────────────────────────────────────┐
  │  The server remembers NOTHING between requests.      │
  │  Each request is self-contained.                     │
  │                                                      │
  │  PROS: trivially scalable, stateless = easy to       │
  │        run anywhere, no state to corrupt,            │
  │        fault-tolerant                               │
    │  CONS: client must send ALL context every time,      │
    │        no built-in continuity                        │
    └─────────────────────────────────────────────────────┘

### Example:

A visual comparison of how Stateless vs. Stateful MCP servers handle the same two consecutive requests from an AI assistant.

```

```

---
