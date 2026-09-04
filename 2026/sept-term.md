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

day - 3

## Serverless Cold Start

### Definition:

Serverless Cold Start is the latency penalty a serverless platform pays — and your users feel — when it must **create and initialize a brand-new execution environment** (sandbox, runtime, and your code) before it can run a function that was just invoked. A *warm* request finds an environment that is already alive and skips straight to running the handler; a *cold* request has to build the whole thing from zero first.

The important framing: a cold start is **not a bug — it is the deliberate price of scale-to-zero economics**. Because the platform reclaims idle environments after minutes of silence, you pay exactly nothing while your function sits unused. The first request after that silence is the one that pays the full setup bill. Latency and the pay-per-use billing model are two sides of the same coin: the platform can only charge you for what actually runs if it is free to destroy what isn't running.

That also tells you the three triggers — every cold start is one of these:
1. **Idle reclamation** — environment killed after N minutes without traffic; the next request rebuilds it.
2. **A fresh deploy** — a new code version means new environments, so releases can trigger a burst of cold starts.
3. **Scale-out (the sneaky one)** — N concurrent requests arrive but only M < N environments are warm; the extra N−M cold-start *all at the same time*, in the middle of your busiest moment — precisely when extra latency hurts the most.

COLD vs WARM — what actually happens on each path:

```
COLD START vs WARM START
════════════════════════════════════════════════════════════

  COLD START — no environment ready, request pays full setup:

   request ──► ┌───────────────────────────────────────────────┐
               │  1. ALLOCATE a sandbox (micro-VM / container) │  50–100 ms
               │  2. DOWNLOAD your deployment package          │  50–500 ms
               │     (grows with bundle size!)                 │
               │  3. BOOT the language runtime                 │  50–1,000 ms
               │     (interpreter fast, JVM/.NET slow)         │
               │  4. RUN YOUR init code                        │  0 ms – seconds
               │     imports, SDK clients, DB connections      │  ◄── your lever
               │  5. INVOKE the handler ────────────────► done │
               └───────────────────────────────────────────────┘
                 total added latency: ~100 ms to several seconds
                 THE USER WAITS FOR ALL OF IT BEFORE THE
                 FIRST BYTE OF ACTUAL WORK HAPPENS.

  WARM START — environment kept alive, same request later:

   request ──► ┌────────────────────────────────┐
               │  thaw (~1–10 ms)               │
               │  step 4 results (DB pool, SDK  │
               │  clients) are STILL ALIVE      │
               │  5. INVOKE handler ────► done  │   single-digit ms
               └────────────────────────────────┘
                 init code runs ONCE per env,
                 then is reused across requests
```

How bad is it, honestly? It depends mostly on runtime and bundle size. Interpreted runtimes (JavaScript, Python) typically cold-start in **200–400 ms**; compiled ones (Go, Rust) can stay under ~100–300 ms; VM-based runtimes (JVM, .NET) take **500 ms to several seconds** — which is why snapshot-restore features (freeze a booted runtime, restore it in milliseconds instead of re-booting) exist and cut that figure dramatically. And *frequency* is the half of the story most explainers skip: steady production traffic sees cold starts on under 1% of invocations — but a function invoked once per hour cold-starts almost every single call, and development environments can see cold rates of 30–90%. Tail latencies (p99) run 2–3× the median, which is exactly what your latency budget actually cares about.

Because every cold start is a small bill in *time*, teams climb a **mitigation ladder** — cheapest first:

```
THE MITIGATION LADDER (cost order)
══════════════════════════════════
  FREE (code-level)      shrink the deployment bundle & prune
                         dependencies  ← biggest lever most teams
                         never pull; lazy-load heavy imports;
                         open DB connections in init scope so
                         warm requests reuse them
  CHEAP (config)         raise memory (CPU scales with it);
                         enable snapshot-restore / fast-snapshot
                         features where the platform offers them
  FRAGILE (folk remedy)  keep-warm "ping" every few minutes —
                         keeps ONE environment warm, does nothing
                         on scale-out, and lies to dashboards
  PAID (definitive)      provisioned concurrency / minimum
                         instances: N environments always ready,
                         cold starts eliminated up to N — but you
                         reintroduce always-on cost into a
                         pay-per-use model (the irony is priced in)
  ARCHITECTURAL          isolate-based edge runtimes (V8 isolates:
                         single-digit ms to spawn) trade a full
                         runtime for a sandboxed web-standard env
```

The whole discipline boils down to one question: **is a human waiting on this call?** If yes, cold starts are a UX bug and deserve the ladder. If the work is async — a queued job, a webhook, a scheduled task — the tail is absorbed invisibly and you should spend nothing on it.

### Example:

A ticketing platform ("TiketKilat") runs a flash sale at 15:00 sharp. The checkout function is serverless, pay-per-use — and had zero traffic for the 30 minutes before the sale, so the platform reclaimed every idle environment.

```
FLASH SALE — ONE ENDPOINT, THREE MOMENTS
════════════════════════════════════════════════════════════

  14:30–15:00  no traffic → platform reclaims ALL environments

  ┌─ 15:00:00.000  request #1 arrives ── COLD ─────────────────┐
  │  allocate sandbox → download package → boot runtime →      │
  │  init SDKs + DB pool → finally run handler                 │
  └────────────────────────────────────────────────────────────┘
              user #1 waits 1.3 s (spinner, rage, refresh)

  ┌─ 15:00:00.300  requests #2–#10 arrive — THE SPIKE ────────┐
  │  env #1 is warm now BUT handles one request at a time     │
  │  → #2–#10 each need their OWN environment                 │
  │  → 9 MORE cold starts, all simultaneous                   │
  │    ◄── scale-out: cold starts land exactly when           │
  │        traffic is highest (worst possible timing)         │
  └───────────────────────────────────────────────────────────┘
              first 30 s of the sale: p99 ≈ 1.4 s
              (users abandon carts, sale page trends on X
               for the wrong reason)

  ┌─ 15:00:45  autoscaler finally has 40 warm envs ──────────┐
  │  requests #200+ ── WARM ── 30–50 ms each                 │
  └──────────────────────────────────────────────────────────┘
              p99 five minutes later: ~45 ms — smooth,
              but the first 30 seconds already happened
```

The fix is a direct application of the ladder. TiketKilat *knows* the sale is at 15:00 — that's not a surprise, it's a schedule. So before launch day they:

- **Schedule-based provisioned concurrency**: warm 30 environments at 14:59:30 so the first wave of requests lands on ready envs (predictable traffic → pre-warm ahead of the peak instead of reacting to it).
- **Slim the bundle**: the checkout function was importing a whole SDK for one call — tree-shaken down from 41 MB to 6 MB, shaving hundreds of ms off the unavoidable cold starts.
- **Move the non-blocking work off the hot path**: the confirmation email is pushed to a queue and processed by a separate function where a 1-second cold start is invisible — nobody is staring at it.

```
WITH THE FIX — same flash sale, same spike:
═══════════════════════════════════════════
  14:59:30  provisioned envs spin up (N = 30) ── paid, ready
  15:00:00  request #1 ──► WARM env ──► 45 ms      ✓ no spinner
  15:00:00  requests #2–#100 ──► scale-out absorbs on warm pool
  15:00:05  a few cold starts only if traffic exceeds N
            (rare, and ~300 ms now, not 1.4 s)
```

The punchline: serverless never removes latency — it *relocates* it from idle time to the first request after idle. Cold starts can't be eliminated for free; they can only be (a) shrunk with leaner code, (b) skipped on the paths where a human waits, and (c) paid away with always-warm capacity exactly where the spike is predictable. The engineering skill is knowing which of the three applies per endpoint — and never trusting a keep-warm ping to save you.

---

day - 4

## Structured Concurrency

### Definition:

Structured Concurrency is a programming model that guarantees **no concurrent task can outlive the code block that created it**: whenever a block spawns child tasks, those children must finish — or be cancelled — before the block is allowed to exit. It applies the same nesting discipline that *structured programming* gave to control flow back in the 1960s (no more `goto`, everything nests inside `if`/`while`/function calls) to the world of threads, goroutines, and coroutines, which never got that discipline — they were the last `goto` in modern code.

In the classic unstructured model, a task's lifetime is best described as "whoever spawned it *hopes* it finishes." A child routinely outlives its parent (a leak), or its parent dies first and leaves it an orphan that happily writes to closed databases and logs after the response was already sent, or it fails and the error vanishes because nothing is attached to it to hear the scream. Structured Concurrency makes all three impossible by construction — not by discipline, but by the language runtime:

1. **Scope**: tasks can only be spawned inside a delimited scope — a `StructuredTaskScope` (Java), a `coroutineScope { }` (Kotlin), an `errgroup.Group` (Go), a Trio/AnyIO *nursery* (Python).
2. **Fork-join**: the scope cannot return until *every* child has joined. No code after the scope runs while children are still in flight.
3. **Cancel-on-failure**: the first child error automatically cancels all remaining siblings, then propagates up like a normal exception — no zombie fan-out silently half-succeeding.

UNSTRUCTURED vs STRUCTURED — what happens to the children:

```
UNSTRUCTURED CONCURRENCY            STRUCTURED CONCURRENCY
(fire-and-forget)                   (scoped fork-join)
══════════════════════════          ══════════════════════════

 main() ─ spawn ──►┌ worker A ┐     main() ──► ┌──────────────────────────┐
        │          │ (slow)   │                │ scope {                  │
        │          └──────────┘                │   fork A ──┐             │
        │          ┌ worker B ┐                │   fork B ──┼─┐           │
        │          │ (fails!)  │               │   fork C ──┼─┼─┐         │
        │          └──────────┘                │            │ │ │         │
        ▼          nobody hears                │   join A ◄─┘ │ │         │
   main RETURNS    B's error                   │   join B ◄───┘ │         │
   (frame gone)                                │   join C ◄─────┘         │
        │                                     │ }  ── error? cancel       │
        │   worker A STILL RUNNING ──►        │      remaining siblings   │
        │   writes to a DB pool the           └──────────────────────────┘
        │   parent already closed                     │
        ▼                                            ▼
   GHOST WORK after "done":               main() returns ONLY after all
   leaked memory, late panics,            children are joined — the task
   silent partial failure                 tree mirrors the call stack:
                                          no orphans, no ghosts, no
   CHILD LIFETIME: unbounded              swallowed errors.
   CHILD LIFETIME: bounded by scope ──►   ▲
                                          └─ error handling in ONE place
```

The mental model: **the tree of running tasks should look exactly like the tree of the call stack** — a parent should never be able to move on, or die, while its descendants are still alive somewhere in the dark. If a piece of work genuinely must outlive its caller (background telemetry, a confirmation email), structured concurrency doesn't forbid it — it forces you to make that detachment *explicit* (hand it to a queue or a separate process) instead of letting it happen by accident.

Structured Concurrency went mainstream through Project Loom: previewed in Java since Java 19, it was finalized as **JEP 507 in Java 26 (2026)** via `StructuredTaskScope`. Kotlin (coroutines), Swift (task groups), Python (Trio/AnyIO nurseries) and Go (`errgroup` + `context`) ship the same idea — each with its own flavor.

### Example:

"TokoOnline" adds a checkout endpoint. Before returning an order confirmation it must do three independent remote calls in parallel: reserve inventory (RPC), charge the payment (RPC), and run a fraud check. The tempting first version is three fire-and-forget goroutines:

```
NAIVE VERSION — handler exits while workers are still flying:

 request ──► handler
              ├─ go reserveInventory() ──(150 ms)──► done
              ├─ go chargePayment()    ──(300 ms)──► done
              └─ go fraudCheck()       ──(fails @ 200 ms,
              │                          nobody is watching)
              ▼
   ~15 ms later : handler returns "200 OK ✓"  ← LIES,
                  nothing actually finished
   200 ms later : fraud check FAILS → order ships anyway,
                  money moves, no one ever learns
   310 ms later : workers touch the request-scoped DB pool
                  the handler already closed → PANIC
                  in the logs AFTER the response was sent
                  (the ghost crash — your on-call pager
                   ringing about a request that "succeeded")
```

All three classic failures in one screenshot: parent returned before children (lie), a child failed silently (swallowed error), and children outlived the parent (ghost work). The fix is to put the fan-out inside a scope — in Go, `errgroup` from `golang.org/x/sync`:

```go
g, ctx := errgroup.WithContext(r.Context())   // scope begins

g.Go(func() error { return reserveInventory(ctx) })  // fork
g.Go(func() error { return chargePayment(ctx) })     // fork
g.Go(func() error { return fraudCheck(ctx) })        // fork

if err := g.Wait(); err != nil {   // join: waits for ALL children
    return 502                      // first error cancels the rest
}                                   // via ctx before it returns
return 200                          // "OK" is now a TRUE statement:
                                    // all three really completed
```

```
WITH ERRGROUP — the scope cannot exit until every child is joined:

 scope ──► ┌────────────────────────────────────────────────┐
           │ g.Go(reserve) ──(150 ms)──► ok ───────────────┐ │
           │ g.Go(charge)   ──(300 ms)──► ok ────────────┐ │ │
           │ g.Go(fraud)    ──(200 ms)──► ERROR ────► ┌──┘ │ │
           └──────────────────────────────────────────┴────┘ │
                      │  errgroup cancels ctx ────────────────┘
                      ▼
           g.Wait() returns the fraud error
                      ▼
           handler → 502, order rolled back,
           no ghost goroutines, no silent success
```

Same three properties, now enforced by the runtime instead of by hope: the handler's `return 200` physically cannot run before all three children joined; the fraud error is impossible to lose — `g.Wait()` either returns `nil` (all OK) or the first failure; and on failure the context cancels the two siblings still in flight, so no half-charged order limps on.

The punchline: structured concurrency is "**the call stack, but for parallel work**." Its three guarantees — no orphaned tasks, no swallowed errors, cleanup in exactly one place — turn concurrency bugs from a class of mystery (races you debug at 2 AM) into plain control flow you can read top to bottom. The remaining honest use of fire-and-forget isn't "we'll deal with it later" — it's work you *deliberately* detach to a queue or background process where outliving the request is the correct, visible design. Concurrency stopped being special the moment we stopped letting it escape the block that owns it.

---