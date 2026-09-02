day - 1

## CQRS (Command Query Responsibility Segregation)

### Definition:

CQRS is an architectural pattern that separates the operations that **change** state (Commands) from the operations that **read** state (Queries), giving each its own model, its own code path, and often its own data store and scaling strategy.

It rests on a simple observation from 1970s database theory: a system does two very different jobs — a write (a "command") is a one-way action that mutates the system and has side effects; a read (a "query") is a side-effect-free lookup that returns data. When you force both through the *same* model, that model becomes a compromise that is good at neither. CQRS deliberately splits them so each half can be optimized for its own job.

A quick word on the name: "Responsibility Segregation" just means "give each responsibility its own class/module/component" — it's the same idea as Separation of Concerns applied to reads vs. writes.

TRADITIONAL (single model) vs CQRS (split models):
═══════════════════════════════════════════════════════════════

  TRADITIONAL CRUD — ONE model does everything:
  ─────────────────────────────────────────────────────────────

            ┌──────────────────────────┐
   write ──►│                          │
   (POST)   │    ONE DOMAIN MODEL      │──► reads hit the SAME
            │    ONE DATABASE          │    structure as writes
   read  ──►│    ONE SERVICE           │
   (GET)    │                          │
            └──────────────────────────┘
              • Model is a compromise: it must serve both
                mutation and query shapes.
              • Read scaling = write scaling (coupled).
              • A heavy write lock blocks reads, and vice versa.


  CQRS — TWO models, each optimized for its own job:
  ─────────────────────────────────────────────────────────────

          ┌──────────────┐         ┌──────────────┐
   write ─►│  COMMAND SIDE │        │  QUERY SIDE   │◄─ read
   (POST)  │  (Write Model)│        │ (Read Model)  │   (GET)
   ──────►│  validates    │        │  denormalized │
          │  business rules│        │  pre-joined   │
          │  persists     │        │  fast reads   │
          └──────┬───────┘         └──────────────┘
                 │      separate stores,
                 │      separate scale
                 ▼         ↑ sync (sync call or async event)
          [WRITE DB]       [READ DB / READ-ONLY REPLICA / CACHE]

              • Commands: strict, validated, consistency-first.
              • Queries: loose, denormalized, speed-first.
              • Each side scales and is deployed independently.

  ┌────────────────────────────────────────────────────────────┐
  │  KEY IDEA: the write path and read path are DIFFERENT       │
  │  shapes, so give them DIFFERENT models instead of one       │
  │  bloated compromise.                                        │
  └────────────────────────────────────────────────────────────┘

When to use it (it is NOT for every app):
  ┌────────────────────────────────────────────────────────────┐
  │  GOOD FIT (use CQRS):                                      │
  │  • Read-heavy systems: 10×–1000× more reads than writes    │
  │  • The read shape is very different from the write shape   │
  │    (write tiny/normalized, read wide/denormalized)         │
  │  • Write and read need different scaling (bursty writes,   │
  │    heavy analytic reads)                                   │
  │  • Teams need to optimize each side independently          │
  │                                                            │
  │  BAD FIT (skip it):                                        │
  │  • Simple CRUD CRUD — a single model is simpler            │
  │  • No read/write asymmetry → CQRS adds complexity          │
  │  • You need strong, immediate read-your-writes consistency │
  └────────────────────────────────────────────────────────────┘

CQRS is often (but not always) paired with **Event Sourcing** — commands produce events that are replayed to build the read model. They are separate patterns that complement each other; CQRS can exist without Event Sourcing.

### Example:

A ticketing platform where 99% of traffic is people *viewing* event seats (reads) and only a tiny fraction is actually *booking* (writes).

```
WITHOUT CQRS — every page-view and every booking hit ONE table
═══════════════════════════════════════════════════════════════

            ┌────────────────────────────────────────┐
 1000/s     │         [events] + [seats] JOIN        │
 reads ───► │    ONE relational model + indexes      │
            │                                        │
  2/s       │   • Seat lookups run heavy JOINs.      │
 writes ──► │   • Every read pays the write-model    │
            │     cost (normalization, locking).     │
            │   • A long booking transaction can     │
            │     block page-view reads.             │
            └────────────────────────────────────────┘
            PROBLEM: reads are slow, writes are rare,
            but they're stuck in the same bottleneck.


WITH CQRS — reads get a model built just for viewing
═══════════════════════════════════════════════════════════════

  Book seat (POST /book)                 View seats (GET /seats)
       │                                        ▲
       ▼                                        │
  ┌────────────────┐            ┌──────────────────────────┐
  │ COMMAND SIDE   │            │ QUERY SIDE               │
  │ write model    │            │ read model               │
  │ • validates    │            │ • pre-joined, flat rows  │
  │   seat exists  │            │ • e.g. "seat: A-12,      │
  │ • checks price │            │   row: front, price: 80, │
  │ • holds lock   │            │   status: FREE"          │
  │ • persists     │            │ • served from cache /    │
  └───────┬────────┘            │   read replica           │
          │                     └────────────▲─────────────┘
          │  emits event                        │ subscribed
          ▼  "SeatBooked"                        │ to events
  ┌──────────────────────────────────────────────────────┐
  │   ASYNC SYNC: write DB → (event) → read DB/cache     │
  │   (event bus / CDC / message queue)                  │
  └──────────────────────────────────────────────────────┘

  RESULT:
  • 1000/s page-views are served from a flat, cacheable,
    denormalized read model — sub-millisecond, no JOINs.
  • The rare 2/s bookings use strict validation and locking
    on the write side, without contending with reads.
  • The read model can be scaled out to N replicas freely;
    the write side stays small and correct.
  • Cost: tiny lag between booking and the read model
    catching up (eventual consistency on the read side).
```

The team gets fast, cacheable reads AND strict, correct writes — each tuned for its own workload instead of one model compromising both.

---

day - 2

## Test-Time Compute (Test-Time Scaling)

### Definition:

Test-Time Compute is the practice of deliberately spending **more computation at inference time** (when the model is answering) to get a better answer — as opposed to spending more computation at **training time** (when the model is built).

For most of AI's recent history there was only ONE dial to turn. To make a model smarter you made it bigger and fed it more data — that computation happened once, upfront, inside the training run, and got baked into the weights. At inference time the model did a single fast forward pass: prompt in, one answer out. More thinking = you had to re-train a bigger model.

Reasoning models (o1-class and successors, which became mainstream through 2025–2026) opened a SECOND dial. Instead of answering in one shot, they generate an internal "chain of thought" — they pause, explore, backtrack, and verify — and the **length and thoroughness of that hidden thinking** is itself a knob you can turn per-request. That extra per-query reasoning is exactly what "test-time compute" means. The same model, given more compute budget at runtime, produces a measurably better answer. Quality now scales with *thinking time*, not just with parameters.

CLASSICAL (training-time only) vs REASONING (test-time scaling):
═══════════════════════════════════════════════════════════════

  CLASSICAL SCALING — one dial, all spent upfront:
  ─────────────────────────────────────────────────────────────

        [TRAINING TIME — expensive, one-time]     [INFERENCE — fixed]
        ┌──────────────────────────────────┐      ┌──────────────┐
        │  MORE PARAMS        ┌──────────┐ │      │  1 fast pass  │
   dial ►│  MORE DATA    ───► │ trained  │ │ ───► │  prompt → out │
        │  MORE GPU-days      │ weights  │ │      │  (no thinking)│
        └──────────────────────────────────┘      └──────────────┘
          smarter model                    SAME cost per query,
                                            no per-question knob

  RESULT: if one answer is wrong, your only fix is to
  retrain something bigger. Expensive, slow, can't adapt
  per question.


  REASONING / TEST-TIME SCALING — a second dial at runtime:
  ─────────────────────────────────────────────────────────────

                          ┌───────────────────────────────┐
   prompt ───────────────►│  TEST-TIME COMPUTE BUDGET     │
                          │                               │
                          │   chain-of-thought ──┐        │
                          │   • try a path        │        │
                          │   • notice error      │◄───────┤  budget
                          │   • backtrack         │  up   │  spend
                          │   • try another route │       │  more =
                          │   • verify            │       │  better
                          └───────────────────────┴───────┘
                                            │  answer
                                            ▼

  • Same trained weights, but you control how "hard" it
    thinks PER REQUEST.
  • Math/code/logic → crank budget up → better accuracy.
  • Simple chit-chat → keep budget tiny → fast & cheap.
  • The trade-off moves to inference time: per-query cost
    rises because the model emits many more tokens (its
    hidden reasoning) even when unit price per token falls.

  ┌────────────────────────────────────────────────────────────┐
  │  KEY IDEA: smarter is no longer ONLY "bigger model".       │
  │  It's also "think longer on the hard questions". Two       │
  │  orthogonal scaling axes — training compute and            │
  │  test-time compute.                                        │
  └────────────────────────────────────────────────────────────┘

The dial is tunable at several levels: per deployment (a "deep reasoning" vs a "fast" model endpoint), per request (an API flag asking for more effort), or algorithmically via methods such as best-of-N sampling, majority voting (self-consistency), or letting the model search/verify before committing to an answer. The economics matter: total inference cost for hard tasks can rise sharply because a reasoning model spends many tokens thinking — so systems must decide WHEN high test-time compute is worth it.

### Example:

A math tutoring app (fitting — exactly the kind of thing you'd build) where the SAME model must handle two very different requests: "what is 7×8?" (instant) vs a hard word problem that trips up one-shot answers.

```
THE SAME MODEL, ONE REQUEST, A PER-REQUEST THINKING BUDGET
═══════════════════════════════════════════════════════════════

  REQUEST A: "7 × 8 = ?"
  ───────────────────────
     budget: LOW  (it's trivial)

        ┌──────────────────────────────┐
        │  single fast pass            │
        │  "7 × 8 = 56"                │  ~few tokens
        └──────────────────────────────┘   → cheap, ~instant

  REQUEST B: hard word problem
  ───────────────────────
     budget: HIGH (it trips up one-shot)

   prompt ──► ┌──────────────────────────────────────────────┐
              │  CHAIN-OF-THOUGHT (hidden reasoning tokens)  │
              │                                              │
              │  attempt 1: sets up wrong equation ──✗        │
              │    "wait, that double-counts the overlap"    │
              │  attempt 2: backtrack, re-model              │
              │    builds correct equation ✓                 │
              │  verify: plug answer back in, consistent ✓   │
              └──────────────────────────────────────────────┘
                                             │  commits "42"
                                             ▼
        more tokens spent → higher accuracy on THIS hard query

  WITH one-shot fast pass (no test-time compute):
     this word problem likely gets the SAME error the model
     always makes on it.
  WITH test-time compute (budget cranked up):
     the model searches internally, self-corrects, verifies,
     and lands the right answer — no retraining needed.
```

The punchline: the app didn't need a bigger, more expensive model. It needed a **budget-aware router** — spend test-time compute only where one-shot accuracy fails, and keep the cheap fast path everywhere else. That's the real engineering superpower of test-time scaling: you buy intelligence on demand, question by question, instead of buying it once in a monolithic training run.

---