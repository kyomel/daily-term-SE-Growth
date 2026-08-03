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
An AI coding assistant needs to (1) look up a user's GitHub repositories, then (2) open a specific file from those results.
═══════════════════════════════════════════════════════════════
  STATEFUL MCP SERVER (Remembers between calls)
═══════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────────┐
  │                                                         │
  │  CALL 1: Assistant asks: "List my repos"                │
  │                                                         │
  │  Assistant ──► [Stateful Server]                        │
    │               Server: "Who is 'my'?"                    │
    │               → Assistant already authenticated in      │
    │                 a previous session, server remembers    │
    │               → Server returns: [repo-A, repo-B, ...]   │
    │               → Server STORES the repo list internally  │
    │                                                         │
    │  CALL 2: Assistant asks: "Open repo-A"                  │
    │                                                         │
    │  Assistant ──► [Stateful Server]                        │
    │               Server: "I remember repo-A from the       │
    │                 list I returned last time"              │
    │               → No need to re-authenticate or re-fetch  │
    │               → Server opens repo-A                    │
    │                                                         │
    │  DANGER: If the server crashes between CALL 1 and       │
    │  CALL 2, the remembered state is LOST. The assistant    │
    │  must re-authenticate and re-fetch everything.          │
    │                                                         │
    └─────────────────────────────────────────────────────────┘
  
  
  ═══════════════════════════════════════════════════════════════
    STATELESS MCP SERVER (Forgets everything between calls)
  ═══════════════════════════════════════════════════════════════
  
    ┌─────────────────────────────────────────────────────────┐
    │                                                         │
    │  CALL 1: Assistant asks: "List repos for user Alice"    │
    │                                                         │
    │  Assistant ──► [Stateless Server]                       │
    │               Server: "Repos for which user?"           │
    │               → Assistant includes EVERYTHING:          │
    │                 "List repos for Alice, auth: token X"   │
    │               → Server returns: [repo-A, repo-B, ...]   │
      │               → Server FORGETS everything              │
      │                                                         │
      │  CALL 2: Assistant asks: "Open repo-A"                  │
      │                                                         │
      │  Assistant ──► [Stateless Server]                       │
      │               Server: "Open repo-A for which user?"     │
      │               → Assistant sends EVERYTHING again:       │
      │                 "Open repo-A for Alice, auth: token X"  │
      │               → Server opens repo-A                    │
      │               → Server FORGETS everything again         │
      │                                                         │
      │  KEY: Every request is COMPLETE on its own.             │
      │  If the server crashes between calls — no problem.      │
      │  Next request carries all the context again.            │
      │                                                         │
      └─────────────────────────────────────────────────────────┘
```

---
