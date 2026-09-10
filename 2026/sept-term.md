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

day - 7

## LoRA (Low-Rank Adaptation)

### Definition:

LoRA is a **parameter-efficient fine-tuning (PEFT) technique** that adapts a large pre-trained model (an LLM, diffusion model, or any Transformer) to a new task *without touching the original weights*. It freezes the full pre-trained model and injects tiny **trainable "adapter" matrices** beside the big weight matrices, so only a minuscule fraction of the network — often under 1% of parameters — actually learns.

The trick comes from an observation about fine-tuning: when you adapt a huge pre-trained model, the learned *change* to its weights is surprisingly **low-rank**. A full weight matrix W (say 4096×4096) barely moves during adaptation — the adjustment ΔW needed to capture a new skill lives on a much smaller number of effective dimensions. So instead of learning all ~16.7M entries of ΔW directly, LoRA *factorizes* it into two skinny matrices: A (rank r × d) and B (d × rank r). Their product B·A reproduces a low-rank approximation of ΔW with only 2·d·r parameters instead of d². With r = 8, that's 65,536 vs 16.7M — about 250× fewer learnable parameters for that layer.

The forward pass becomes: **h = W₀x + (α/r)·BAx** — the frozen path carries all the pre-trained knowledge, and the small learned branch adds a task-specific correction. After training, the correction can be *merged* back into W (W′ = W₀ + (α/r)·BA), leaving a single normal model with zero extra inference cost.

FULL FINE-TUNING vs LoRA:
═══════════════════════════════════════════════════════════════

  FULL FINE-TUNING — every weight learns, everything must fit in memory:
  ─────────────────────────────────────────────────────────────

          [W: d×d] ─── gradient ──► [optimizer state]      GPU MEMORY
           ALL 4B·L             ALL weights move           = weights × ~16
           params trainable     (even if most barely do)     (fp16 + grads
                                                              + Adam)
          • New copy per task: one full model per skill        ▲
          • Checkpoint = the whole model (GBs per save)        │
          • Needs many big GPUs — the optimizer alone          │
            is 2× the model size.                              │
                                                               ▼
                                                        7B model ≈ 110–140 GB
                                                        → several 80GB GPUs

  LORA — frozen base + a tiny learnable detour per layer:
  ─────────────────────────────────────────────────────────────

          [W: d×d]  FROZEN ──► h = W₀x + (α/r)·BAx
               ▲                    │
               │            ┌───────┴────────┐
          no gradient    [A: r×d]      [B: d×r]   ◄── ONLY these learn
               │         random init   zeros init      (2·d·r params)
               │                                        per layer
               │                        GPU MEMORY
               │                        ≈ weights × ~2–3
               ▼                        + a few MB of adapters
        never updated                  → 7B model fits on ONE
                                        24GB consumer GPU
                                         (QLoRA: even one 8–12GB)

          • ONE base model + MANY adapters = many specialist
            skills, each a few MB on disk.
          • Merge B·A into W (W′ = W₀ + (α/r)·BA) → identical
            speed at inference, no adapter overhead.
          • Or keep adapters separate → hot-swap skills live.

  ┌────────────────────────────────────────────────────────────┐
  │  KEY IDEA: the pretrained model already knows 99.9% of     │
  │  what it needs. LoRA learns only the small, low-rank       │
  │  "delta" that points that knowledge at your task.          │
  └────────────────────────────────────────────────────────────┘

Key mechanics worth knowing:

- **r (rank)** — the width of the adapter matrices. Typical values 8–64. Higher r = more capacity (and more memory); lower r = cheaper, often almost as good. r is the main dial you tune.
- **α (alpha)** — a scaling factor applied as α/r. It controls how strongly the adapter correction is applied, not the rank itself. Common lore: set α ≈ 2×r and tune from there (e.g., r=16, α=32).
- **Which layers to target** — originally the attention projections (q, v). In practice people adapt q, k, v, o and often the feed-forward layers too; more targets = more capacity.
- **LoRA vs QLoRA** — QLoRA (2023) adds 4-bit quantization of the *frozen* base model while training the LoRA adapters in higher precision. That collapses the memory of the frozen part (7B base from ~14GB fp16 to ~4–5GB), which is why fine-tuning even 7B–70B class models is now possible on a single consumer GPU.
- **DoRA and the rest** — DoRA (Weight-Decomposed LoRA, 2024) decomposes weights into magnitude and direction and applies the low-rank update only to direction, beating plain LoRA on accuracy at the same rank; the field keeps iterating, but LoRA is the foundation they all build on.
- **Why it matters for serving** — because the base stays untouched, one GPU can hold a single frozen model and swap between many adapters per request (multi-LoRA serving: skill per tenant, per language, per task) — the adapter is the product, the base is shared infrastructure.

### Example:

RumahKode, a small studio, runs a customer-support copilot on a 7B open model for their Indonesian e-commerce client. It must answer in casual Indonesian ("santai", code-switching with English) and follow a strict refund policy. They own one 24GB consumer GPU and cannot rent a cluster for every experiment.

Option A (full fine-tuning) is a non-starter: a 7B model needs ~110–140 GB of VRAM just for weights + gradients + optimizer — 6× their whole GPU. Option B is to fine-tune with LoRA (they use Hugging Face PEFT; with r=16, α=32 on the attention + MLP projections, ~0.2% of parameters train).

```
THE PIPELINE — one frozen base, a small learnable detour
═══════════════════════════════════════════════════════════════

                ┌────────────────────────────────────────────┐
   1000 support │  DATASET: "user says X → agent replies Y"  │
   tickets/hour │  casual Indonesian + refund policy rules   │
                └────────────────────┬───────────────────────┘
                                     ▼
   ┌───────────────────────────────────────────────────────────┐
   │  BASE MODEL 7B — FROZEN (never updated, zero gradient)    │
   │                                                            │
   │  each layer:  h = W₀x + (α/r)·BAx                         │
   │                    ▲          ▲                            │
   │                    │          └── [A][B] ← ONLY TRAINABLE │
   │                    └─ original knowledge, untouched       │
   └──────────────────────────┬────────────────────────────────┘
                              ▼
                    train 3 epochs on the 24GB GPU
                    (weights ~14GB fp16 + adapter few MB)
                              ▼
                  LoRA adapter ≈ 26 MB on disk
                  (vs a ~14 GB full-model checkpoint)
                              ▼
              ┌───────────────────────────────┐
              │  MERGE:  W′ = W₀ + (α/r)·BA   │  ← deploy as ONE
              │  → zero extra latency, same    │    model, no special
              │    model file format           │    runtime needed
              └───────────────────────────────┘
```

After training, the copilot holds the refund policy and the right tone — without the studio ever training more than a few MB of parameters, and without renting a GPU cluster.

The merge step is optional and that's where LoRA gets *really* interesting. Their second product is a fasting-app companion bot that needs three very different voices: a strict medical-accuracy mode, a relaxed "gym bro" motivator mode, and a Bahasa-Jawa-lite casual mode. Instead of three full fine-tunes (three × 14GB+ models), they train **three 26 MB LoRA adapters on the same frozen base** and load the right one per request — or even per user:

```
MULTI-LORA SERVING — one base model, three specialist skills
═══════════════════════════════════════════════════════════════

         ┌─────────────────────────────────────────────────┐
         │            ONE FROZEN 7B BASE (14 GB)          │
         │            loaded once in GPU memory           │
         └──────▲──────────────────▲──────────────────▲───┘
                │                  │                  │
        ┌───────┴──────┐   ┌───────┴──────┐   ┌───────┴──────┐
        │ adapter 1    │   │ adapter 2    │   │ adapter 3    │
        │ "medical"    │   │ "gym bro"    │   │ "casual jv"  │
        │ 26 MB        │   │ 26 MB        │   │ 26 MB        │
        └───────┬──────┘   └───────┬──────┘   └───────┬──────┘
                │   hot-swap per request:             │
                └───────── W + B₁A₁  /  W + B₂A₂ ─────┘
                                             │
                        user asks → route → right adapter → answer
```

Storage saved: 42 GB of full checkpoints become 78 MB of adapters. GPU saved: one model resident instead of three. Iteration saved: retraining one voice's adapter never touches the other voices — no regression risk across skills. And each adapter trains in a fraction of the time of a full fine-tune, so the studio can experiment daily instead of weekly.

The punchline: LoRA separates the two things fine-tuning used to conflate — *knowledge* (expensive, lives in the big frozen weights) and *behavior* (cheap, lives in the tiny adapter delta). When adapting a model stopped meaning "re-train everything" and started meaning "attach a few MB of learned direction," fine-tuning went from a cluster-scale, weekly operation to a single-GPU, per-product one. For anyone running models on a modest box, LoRA isn't an optimization trick — it's the difference between fine-tuning being possible at all and not.

---

day - 8 

## Idempotency Keys

### Definition:

An **idempotency key** is a client-generated unique identifier sent with a mutating HTTP request (typically as an `Idempotency-Key` header on `POST`/`PATCH`) that lets the server recognize retries of the *same logical operation* and answer them with the stored result — instead of executing the side effect again.

It exists because of a brutal asymmetry in networks: **a lost response does not mean the request never arrived.** HTTP gives you safe methods (`GET`, `PUT`, `DELETE` are idempotent by spec — repeating them is harmless) but `POST` is a "fire once" verb with no such guarantee. When a client times out, it cannot know whether the server committed the charge, booked the seat, or sent the email. So it retries — and "retry" is where duplicates are born. At-least-once delivery is the default of the real world; idempotency keys are how you survive it.

The name comes from algebra: an operation is *idempotent* when doing it twice equals doing it once (`f(f(x)) = f(x)`). The pattern does not literally make the operation run once — it makes the **system converge on one visible result**, no matter how many times the request arrives. That distinction ("deterministic convergence", not "exactly-once") is the whole game in distributed systems.

NON-IDEMPOTENT POST — same request sent twice = side effect twice:
════════════════════════════════════════════════════════════════════

  CLIENT                          SERVER
    │ 1. POST /charge $50          │
    │────────────────────────────► │
    │                              │  charge $50    ┌───────────────┐
    │                              │───────────────►│ bank: -$50    │
    │ 2. response LOST             │                └───────────────┘
    │◄───────── network ✂ ──────── │  (client times out)
    │                              │
    │ 3. retry POST /charge $50    │
    │    (no key — looks identical)│
    │────────────────────────────► │
    │                              │  charge $50 AGAIN  ┌───────────────┐
    │                              │───────────────────►│ bank: -$50    │
    │◄──────────────────────────── │  200 OK            │ TOTAL: -$100  │
    │                              │                    └───────────────┘
    RESULT: one intent, two charges. The retry was "safe" from the
    client's view — it never saw a response — but the server could
    not tell the two requests apart.


IDEMPOTENT POST — same key = same logical operation:
════════════════════════════════════════════════════════════════════

  CLIENT                          SERVER
    │ 1. POST /charge $50          │
    │    Idempotency-Key: K-3f9a   │
    │────────────────────────────► │
    │                              │  key K-3f9a seen before?
    │                              │      │ NO
    │                              │      ▼
    │                              │  charge $50   ┌──────────────────────┐
    │                              │  save:        │ IDEMPOTENCY STORE    │
    │                              │  K-3f9a → 200 │ K-3f9a │ 200 {charge} │
    │ 2. response LOST             │               └──────────────────────┘
    │◄───────── network ✂ ──────── │  (client times out)
    │                              │
    │ 3. retry POST /charge $50    │
    │    SAME Idempotency-Key      │
    │────────────────────────────► │
    │                              │  key K-3f9a seen before?
    │                              │      │ YES ──► replay saved response
    │◄──────────────────────────── │  NO second charge!
    │  200 OK (replayed)           │
    RESULT: one intent, one charge, retries are free.

  ┌──────────────────────────────────────────────────────────────────┐
  │  KEY IDEA: the server remembers the OUTCOME of a key, so a       │
  │  retry with that key is answered from memory, never re-executed. │
  └──────────────────────────────────────────────────────────────────┘

How it works in practice:

- **One key per intent.** The client mints a fresh key for each new logical operation (each checkout, each booking) and reuses *that same key* for every retry of it. A UUIDv4 is the convention; never use PII or predictable counters (guessing keys would let callers collide with — and replay — other people's operations).
- **The server stores key → result.** First request with a key executes and records the outcome (status code + response body) in a dedupe store. Any later request carrying the same key short-circuits and gets the stored outcome back — Stripe even replays stored `500`s, so retries after failures behave exactly like the original failure did.
- **Key + different payload = conflict.** If a client reuses a key but sends a different body, the server must reject with `409 Conflict` — that is a bug in the client (new intent needs a new key), not a retry.
- **Keys expire.** Stripe prunes keys after ≥24 hours; a key reused after pruning starts a *new* operation, which is safe only because the original is long settled. TTL is your garbage collector — the store otherwise grows forever.
- **Standardization:** the `Idempotency-Key` header is an IETF draft (`draft-ietf-httpapi-idempotency-key-header`) and a de-facto convention that payment APIs (Stripe, Adyen, PayPal) have shipped for over a decade.

The subtle part — where naive implementations leak duplicates:

```
THE CRASH WINDOW — the store alone is not enough
════════════════════════════════════════════════
  1. business commit        2. write dedupe record    3. reply 200
  ┌──────────────────┐        ┌──────────────────┐      ┌────────┐
  │ charge $50       │        │  K-3f9a → 200    │      │  200   │
  │ tx COMMITS       │  crash │  (never written) │      └────────┘
  └──────────────────┘ ────►  └──────────────────┘
        ▲                        ▲
        └─ if the process dies between these two steps, the retry
           arrives, the key is unknown, and the charge runs again.

  Fixes (defense in depth):
  1. Write the dedupe row in the SAME database transaction as the
     business commit → both happen or neither.
  2. Backstop: a UNIQUE constraint on the natural business key
     (e.g. payment_ref = order-4711). A duplicate insert then
     collides → return the existing row instead of charging twice.
  3. Concurrency: two same-key requests in flight (double-tap).
     The store's primary key on the idempotency key is the lock —
     one insert wins, the other waits or gets a 409.
```

### Example:

Kyomel's fasting-bot launches a paid "Pro" tier. Users pay through a checkout API, and the payment service is written in Go behind a load balancer. It is 11 PM, and a user on a flaky mobile connection taps "Upgrade to Pro" on a slow bus — the request reaches the server, the charge commits, but the response dies somewhere in the tunnel before coming back. The phone retries.

```
RETRY-SAFE CHECKOUT — idempotency middleware + DB backstop
═══════════════════════════════════════════════════════════

  CLIENT (fasting-bot app)                PAYMENT SERVICE (Go)
  ┌──────────────────────────────┐
  │ POST /v1/subscriptions       │
  │ { plan: "pro" }              │
  │ Idempotency-Key:             │
  │   550e8400-e29b-41d4-a716-…  │   ← new UUID minted per checkout
  └──────────────┬───────────────┘
                 │  attempt #1
                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ IDEMPOTENCY MIDDLEWARE (dedupe gate, runs before handler)   │
  │                                                             │
  │   key "550e84…" in store?                                   │
  │      │ NO                                                   │
  │      ▼                                                      │
  │   run handler ──► charge $49 ──► INSERT subscription        │
  │                      │              │                       │
  │                      └── SAME TX ──┴─► INSERT dedupe row    │
  │                          (fix #1: commit together)          │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │ DB                      │
                    │ subscriptions            │
                    │   UNIQUE (client_ref)   │ ← fix #2: natural-key
                    │ idempotency_keys        │   backstop
                    │   PK (key), TTL 24h     │
                    └─────────────────────────┘

  ── the 200 response is lost in the tunnel; client times out ──

  CLIENT                               PAYMENT SERVICE
    │  attempt #2 (auto-retry, SAME key, SAME body)
    │────────────────────────────────►
    │                                 key "550e84…" in store?
    │                                       │ YES
    │                                       ▼
    │                                 replay stored 200 + sub id
    │◄──────────────────────────────── no handler, no new charge

  RESULT: user sees one successful upgrade. Bank shows one $49
  charge. Even if the server had crashed inside the crash window,
  the second attempt's INSERT would hit the UNIQUE(client_ref)
  backstop and return the existing subscription instead of
  creating a duplicate.
```

Three small details make this production-grade rather than demo-grade. First, the dedupe gate must be **atomic**: two retries arriving at the same millisecond (double-tap on the pay button) both check the store, both see "unknown", and both run the handler — so the store's primary key on the idempotency key is what serializes them: one INSERT wins, the other blocks and then replays. Second, the middleware should hash the request body and store it next to the key, so a same-key-different-payload retry is caught with a `409` instead of silently returning someone else's result. Third, idempotency keys are for *your* retries — a background worker, a webhook redelivery, a user mashing the button — all of them must agree on one key per intent or the pattern quietly stops working.

The punchline: idempotency keys move the burden of duplicate-safety from "hope the network behaves" to "design for the network misbehaving." They don't eliminate duplicate execution — nothing can, once a process can die between two side effects — but they guarantee that no matter how many times a request arrives, the user is charged once, the seat is booked once, and the system's answer never changes. That single property is why no serious payment, booking, or messaging API ships without them, and why the pattern is quietly becoming the default answer to every "what if the retry double-fires?" question in distributed systems.

---

day - 9

## Transactional Outbox Pattern

### Definition:

The Transactional Outbox Pattern makes **publishing an event as reliable as committing a database transaction** — by not publishing from the application at all. Instead of sending a message to a broker inside your request handler, you write the event as a row into an **outbox table, in the *same* database transaction** as the business change. A separate *relay* process later reads those rows and forwards them to the broker. Your database transaction becomes the single point of atomicity: business data and its events either commit together or not at all.

The pattern exists because of the **dual-write problem**. In any event-driven system, some operation must do two writes that no single transaction can span: (1) change rows in your database and (2) publish an event to a broker. A relational transaction cannot reach into Kafka; a broker transaction cannot reach into Postgres. So the two writes are never atomic, and whichever order you pick, you expose a window where the two systems disagree — the two classic failure modes:

- **Save-then-publish**: the order commits, the process crashes before the event goes out → the event is *lost forever*. The order exists but nothing downstream ever hears about it, and that inconsistency never heals on its own.
- **Publish-then-save**: the event goes out, then the business write rolls back → a *phantom event*. Consumers eagerly send a confirmation email, decrement stock, and charge a card for an order that does not exist.

The outbox closes both windows with one trick: the event stops being a side effect and becomes **data**. You move the publish outside the transaction and outside the request path entirely, and the message broker becomes just one more *consumer of your database*.

NAIVE DUAL-WRITE vs OUTBOX — where atomicity lives:
════════════════════════════════════════════════════════════════════════

```
NAIVE — two writes, two systems, NO shared atomicity:

  handler
    │  ① write business row        ② send event to broker
    ▼                                ▼
  ┌───────────────┐              ┌──────────────┐
  │  ORDERS  DB   │              │   BROKER     │
  └───────────────┘              └──────────────┘
      no transaction can span both ──► either write fails alone:
        • ① ok, crash before ②  → event LOST (order exists,
                                     nobody notified, never heals)
        • ② ok, ① rolls back    → PHANTOM event (consumers act on
                                     an order that doesn't exist)


OUTBOX — the event is a ROW, committed atomically with the change:

  handler
    │  BEGIN TX
    │    INSERT INTO orders (...)
    │    INSERT INTO outbox (id, aggregate_id, type,
    │                        payload, created_at)
    │  COMMIT                      ← one transaction: both or neither
    ▼
  ┌───────────────────────────────────────┐
  │            ORDERS  DB                 │   the DB is now source of
  │  ┌──────────────┐  ┌───────────────┐  │   truth for state AND events
  │  │ orders       │  │ outbox        │  │
  │  │              │  │ OrderPlaced…  │──┼──► ② relay reads new rows
  │  └──────────────┘  └───────────────┘  │     (polling OR CDC tailer)
  └───────────────────────┬───────────────┘
                          │ ③ publish — relay retries forever,
                          ▼    crash/restart loses nothing
                  ┌───────────────┐
                  │    BROKER     │   at-least-once delivery
                  └───────┬───────┘
                          │ ④ consumers MUST be idempotent
                          ▼    (duplicates are expected, not bugs)
```

  ┌────────────────────────────────────────────────────────────────┐
  │  KEY IDEA: you never "send" an event in the request path —     │
  │  you SAVE it. Delivery becomes a separate, retryable job       │
  │  owned by a relay, fully decoupled from the business write.    │
  └────────────────────────────────────────────────────────────────┘

How the relay works — two flavors, same job (read new rows → publish → mark dispatched):

```
POLLING PUBLISHER                          TRANSACTION LOG TAILING (CDC)
─────────────────────                      ─────────────────────────────
  app process, your code                    Debezium / CDC connector
  every N ms (or on commit hook):           reads the DB's write-ahead
                                            log / replication stream
  SELECT ... FROM outbox
  WHERE dispatched_at IS NULL               ┌───────────────────────┐
  ORDER BY id                               │  outbox table          │
  FOR UPDATE SKIP LOCKED                    │  INSERT ... ──► WAL    │
        │                                   └───────────┬───────────┘
        ▼                                               │ tails binary
  publish each row ──► broker                           ▼
        │                                     ┌───────────────────────┐
        ▼                                     │  Debezium / CDC       │
  UPDATE outbox                               │  ──► broker           │
  SET dispatched_at = now()                   └───────────────────────┘
        │                                     • zero app code, no
  + easy to write, test, debug                  polling latency
  + full control (batching, retry)            • but read-only window
  – adds latency (poll interval)                on your DB internals
  – watch out: MULTIPLE relay instances        • ordering preserved by
    can double-publish → make the relay         the log itself
    idempotent (UNIQUE(id)) or run ONE
```

Non-negotiable companions of the pattern:

- **At-least-once, never exactly-once.** A relay can crash after publishing but before marking the row dispatched, so the same event *will* sometimes arrive twice. Consumers must dedupe (store `processed_event_id` with a UNIQUE constraint, or make the consumer's own write idempotent by natural key). This is not a flaw — it's the honest contract that makes the whole system simple.
- **Ordering is a choice, not a given.** If per-aggregate ordering matters (events for the *same order* must arrive in sequence), either run a single relay or partition the topic by `aggregate_id` — and remember multiple relay instances polling in parallel can hand rows to the broker out of order.
- **The outbox table needs a janitor.** Dispatched rows accumulate forever if nobody deletes them. Common policy: delete rows older than N days/hours (after confirming the broker took them), or keep them briefly as an audit/replay trail. Retention is a business decision — storage is cheap, ordering guarantees are not.
- **It is not Event Sourcing.** Event sourcing stores *all* state as events and rebuilds aggregates from them; the outbox is just a delivery queue that mirrors your normal writes. They coexist happily — and so do the outbox and CQRS (day - 1): the outbox is often the reliable pipe that feeds a CQRS read model.

When you should *not* reach for it: purely synchronous request/response with no events; workloads where a lost event is genuinely tolerable (metrics, best-effort notifications) — there the pattern is pure overhead; and systems that need a broker message *before* the DB commit becomes visible. Otherwise, for anything where "this happened" must eventually reach other services exactly once per occurrence, the outbox is the standard answer — which is why it shows up in every serious microservices guide, and why CDC-based relays (Debezium + Kafka Connect) turned it into mainstream production practice through 2024–2026.

### Example:

"KopiKode" — an online coffee-subscription shop — splits checkout into services. The `orders` service is the source of truth for orders. When a customer checks out, three other services must learn about it: `billing` (charge), `inventory` (reserve beans), and `notifications` (send "order received" email). The first version did the naive dual-write — and production found both failure modes within a week:

```
NAIVE VERSION — the week from hell:

  checkout handler
    │  INSERT order (ok)
    │  kafka.send("OrderPlaced")        ── ① 3 AM: crash between the
    ▼                                      two lines → order stored,
  ┌────────────┐   ┌──────────┐            email never sent, stock
  │ orders DB  │   │  broker  │            never reserved. Support
  └────────────┘   └──────────┘            tickets: "I ordered, no
                                           confirmation, beans never
    and the reverse: a retry sent the      shipped."
    event twice for ONE order after a
    timeout → customer charged 2×,         ── ② double-send on retry:
    two emails, beans reserved 2×.             no dedupe anywhere.
```

They rebuilt it with an outbox. The checkout handler now does exactly ONE transactional write; a polling relay (their Go service, `SELECT … FOR UPDATE SKIP LOCKED`, every 100 ms) does the publishing; and consumers dedupe on `event_id`:

```
WITH THE OUTBOX — order → event → three consumers, no lost or phantom events:

  CUSTOMER taps "Checkout"
        │
        ▼
  ┌─────────────────────────────────────────────┐
  │ ORDERS SERVICE — ONE local transaction      │
  │                                             │
  │   BEGIN;                                   │
  │     INSERT INTO orders (id, sku, qty, …)   │   ← business change
  │     INSERT INTO outbox (id, aggregate_id,  │
  │       type, payload)                       │   ← the event, same tx
  │       VALUES (evt_9f2c, 'order_4711',      │
  │               'OrderPlaced', '{…}');       │
  │   COMMIT;                                  │
  │                                             │
  │   ┌──────────────┐   ┌───────────────────┐ │
  │   │ orders       │   │ outbox            │ │
  │   └──────────────┘   │ evt_9f2c OrderPl… │ │
  │                      └───────────────────┘ │
  └─────────────────────────────┬───────────────┘
                                │
                  RELAY (Go, polls every 100 ms:
                  SELECT … FOR UPDATE SKIP LOCKED)
                                │
                                │  publish evt_9f2c ──► topic "orders"
                                ▼
                     ┌──────────────────────┐
                     │        KAFKA         │   at-least-once
                     └──┬───────┬───────┬───┘
                        │       │       │
            ┌───────────┴──┐ ┌──┴──────┐ └──────────┐
            ▼              ▼          ▼            ▼
     ┌──────────────┐ ┌──────────┐ ┌──────────────┐
     │   BILLING    │ │INVENTORY │ │ NOTIFICATIONS │
     │  charge $12  │ │reserve 1 │ │  send email   │
     └──────┬───────┘ │bag "Gayo"│ └──────┬───────┘
            │         └──────────┘        │
            └─────────── all dedupe on ───┘
                 event_id: INSERT INTO processed (event_id)
                 … UNIQUE(event_id) → duplicate delivery of
                 evt_9f2c is swallowed silently, never re-applied
```

```
THE CRASH TEST — why this survives what the naive version didn't:

  relay publishes evt_9f2c ──► broker ACKs ──► crash
        │                                        │
        │         relay dies BEFORE marking       │
        ▼         outbox row dispatched           ▼
  outbox: evt_9f2c  dispatched_at = NULL    (row still pending)

  relay restarts → re-reads evt_9f2c → publishes AGAIN
        │
        ▼
  billing receives evt_9f2c twice:
     1st: INSERT processed(evt_9f2c) ✓ → charge $12
     2nd: INSERT processed(evt_9f2c) ✗ UNIQUE VIOLATION
          → caught, skipped, NO second charge

  RESULT: the event is delivered at least once, the CHARGE happens
  exactly once (thanks to the consumer's dedupe), and no order is
  ever silently missing its events. Even a crash in the relay is
  just "publish again" — the DB never lies about what happened.
```

The team's final shape: orders DB holds state + outbox; the Go relay (or, later, Debezium tailing the WAL) moves events to Kafka; billing, inventory, and notifications all dedupe by `event_id`; a nightly cleanup job deletes outbox rows dispatched more than 24 h ago. The checkout path gained a few microseconds writing one extra row — and lost an entire class of "it happened but nobody knows" incidents.

The punchline: the Transactional Outbox Pattern is the pragmatic answer to the oldest lie in distributed systems — "I'll write to my DB and then tell everyone about it." You cannot make two systems commit atomically, so you stop trying: you make the *event itself* part of the one transaction you do control, and you treat the broker as a downstream consumer that lags a little. Events become durable facts that survive crashes, restarts, and deploys by construction — and the only price is a relay you can restart freely and consumers that must tolerate (and dedupe) a duplicate now and then. That trade — atomicity where it's free, idempotency where it's needed — is why the outbox, not distributed transactions, became the default backbone of event-driven systems.

---

day - 10

## Confidential Computing

### Definition:

Confidential Computing is a **hardware-enforced runtime isolation** model: a workload runs inside a *Trusted Execution Environment (TEE)* — a cryptographically sealed region of CPU and GPU memory whose encryption keys are generated **inside the silicon and never leave it**. The consequence is the whole point of the pattern: everyone who *operates* the machine — the hypervisor, the host kernel, the cloud provider's privileged staff, a fully root-compromised host OS — sees only ciphertext where your model weights, your prompts, and your patient records used to be.

The framing that makes it click. Security has covered **two states of data** for thirty years, and quietly skipped the third:

- **at rest** → disk / object-storage encryption
- **in transit** → TLS
- **in use** → …nothing, by design

To compute on data, a CPU *must* decrypt it into DRAM — and DRAM belongs to whoever owns the host. Disk encryption ends at the bootloader; TLS ends at the socket. The moment the bytes have to be understood, they are plaintext owned by the operator. That is why every cloud workload has always carried an unstated assumption: **trust the operator**.

Confidential computing moves that assumption down into the silicon. Instead of trusting the operator, you trust the chip vendor's hardware — plus a cryptographic proof that the right code is running. That proof is **remote attestation**, and it is the second half of the pattern:

1. **Runtime isolation** — the TEE's memory is encrypted in place; the host cannot read it, and (in 2026 silicon) cannot silently re-map, replay, or roll it back either.
2. **Attestation** — the TEE can produce a hardware-signed *Evidence* artifact (an SEV-SNP report / TDX Quote / NVIDIA GPU attestation report) proving exactly which firmware, kernel, and application measurements were loaded, and that debug mode is off.

Neither half works alone. Isolation without attestation is unverifiable (you cannot tell a genuine TEE from a simulator, or from a legitimate TEE running the attacker's image). Attestation without isolation is a notarized promise that nobody enforces. Together they are the product: **"prove what is running, then hand it secrets that even its own host cannot read."**

WITHOUT CONFIDENTIAL COMPUTING — a standard VM: the host reads everything
════════════════════════════════════════════════════════════════════════════

```
                    ┌─────────────────────────────────────────────┐
   patient data ───►│  CLOUD PROVIDER'S PHYSICAL MACHINE          │
   model weights    │                                             │
   (TLS in transit) │   ┌─────────────────────────────────────┐   │
                    │   │  YOUR VM / CONTAINER                │   │
                    │   │   ┌───────────────────────────────┐ │   │
                    │   │   │  DRAM  —  PLAINTEXT           │ │   │
                    │   │   │   • prompt: "pasien Budi, …"  │ │   │
                    │   │   │   • weights: 7B fp16          │ │   │
                    │   │   │   • KV-cache: full reasoning  │ │   │
                    │   │   └───────────────────────────────┘ │   │
                    │   └─────────────────────────────────────┘   │
                    │        ▲              ▲              ▲      │
                    │   hypervisor     host kernel   provider ops │
                    │   ═════════ ALL OF THEM CAN READ ═════════► │
                    └─────────────────────────────────────────────┘

   Disk encryption and TLS BOTH END HERE — at the doorstep of the
   machine. The instant the CPU needs the bytes to compute on them,
   they are plaintext, and the operator owns the plaintext.
```

WITH CONFIDENTIAL COMPUTING — a Confidential VM (CVM) + GPU TEE
════════════════════════════════════════════════════════════════════════════

```
                    ┌─────────────────────────────────────────────┐
   patient data ───►│  CLOUD PROVIDER'S PHYSICAL MACHINE          │
   model weights    │  (operator: untrusted, and now irrelevant)  │
                    │   ┌─────────────────────────────────────┐   │
                    │   │  CONFIDENTIAL VM  ── TEE ──         │   │
                    │   │   ┌───────────────────────────────┐ │   │
                    │   │   │  DRAM — ENCRYPTED (per-page)  │ │   │
                    │   │   │  0x9f3a…  0x41bc…  0x77de…    │ │   │
                    │   │   └───────────────────────────────┘ │   │
                    │   │            ▲ decrypts ONLY here     │   │
                    │   └────────────┼────────────────────────┘   │
                    │        ┌───────┴────────┐                   │
                    │        │  CPU / GPU     │ keys are born in  │
                    │        │  SILICON       │ silicon, never    │
                    │        └───────┬────────┘ handed to the host│
                    │   AMD SEV-SNP · Intel TDX · ARM CCA ·       │
                    │   NVIDIA C-CAP (H100 / H200 / B200)         │
                    │                                             │
                    │  hypervisor ─┐                              │
                    │  host kernel ┼─► see CIPHERTEXT + MEASURE-  │
                    │  provider ops┘   MENTS, never the contents  │
                    └─────────────────────────────────────────────┘

  ┌────────────────────────────────────────────────────────────────┐
  │  KEY IDEA: the trust boundary moves from "the party running    │
  │  the hardware" to "the chip vendor's silicon + a signed proof  │
  │  of what is loaded." You stop asking the operator to be good   │
  │  and start verifying, cryptographically, that they can't peek. │
  └────────────────────────────────────────────────────────────────┘
```

The four pillars (as the Confidential Computing Consortium frames them) are a checklist you can audit against:

- **Hardware root of trust** — encryption keys are fused into the chip; the host cannot derive them.
- **Attestation** — signed Evidence of identity, initial state, and TCB (firmware microcode) version.
- **Sealed storage** — data encrypted to the TEE's *identity*, so it can only be unsealed by a TEE matching the same measurement. This is "encryption at rest" rebound to *code* instead of to a machine.
- **Secure channels** — a session key negotiated *after* attestation, so the wire is pinned to the verified enclave rather than to a hostname.

The 2026 hardware landscape, and why "the TEE" is usually several TEEs stitched together:

```
PROTECTING A FULL AI INFERENCE STACK — isolation at both ends of the bus
════════════════════════════════════════════════════════════════════════

  ┌──────────────────────────── CONFIDENTIAL VM ──────────────────────────┐
  │                                                                       │
  │  AMD SEV-SNP / Intel TDX / ARM CCA                                    │
  │  ┌───────────────────────────────────────┐                            │
  │  │ CPU TEE: guest DRAM encrypted,        │   · whole-VM isolation     │
  │  │ pinned launch measurement,            │   · 3–6% overhead on       │
  │  │ REVERSE MAP TABLE blocks remap/replay │     inference in 2026       │
  │  └───────────────┬───────────────────────┘                            │
  │                  │ PCIe — bus traffic encrypted by the CPU             │
  │  ┌───────────────▼───────────────────────┐                            │
  │  │ NVIDIA C-CAP GPU TEE (H100/H200/B200) │   · VRAM encrypted in HBM3e│
  │  │ weights + activations + KV-CACHE      │   · ~7% throughput cost    │
  │  │ live encrypted in VRAM                │   · has its OWN attestation│
  │  └───────────────────────────────────────┘     report, bound to the   │
  │                                                 CPU's — "composite    │
  │  AWS Nitro Enclaves: no persistent storage, no  attestation"          │
  │  networking by default — you must proxy every                         │
  │  byte through the parent, by design.                                  │
  └───────────────────────────────────────────────────────────────────────┘

  Rule of thumb: encrypt the CPU side only and your weights still sit in
  plaintext VRAM. Confidential inference means a CVM PLUS a CC-capable
  accelerator, with an attestation that covers BOTH.
```

What changed by 2026 — this is the reason the term moved from "compliance checkbox" to "default":

- **Overhead collapsed.** First-generation TEEs cost 30–40% throughput. Current SEV-SNP/TDX silicon sits at **3–6%** on inference, and NVIDIA C-CAP at roughly **7%** now that the memory-encryption engines live inside the HBM3e controllers. Below the variance between two cloud regions, the cost stops being a business decision.
- **Frameworks caught up.** By 2026 vLLM, TGI, and Triton ship first-class confidential modes: pass a flag, weights load across an encrypted path into a CC GPU, the KV-cache stays sealed, tokens stream out over an attested channel. Before this, every team hand-instrumented its own inference server.
- **Consumer-scale confidential AI went mainstream.** Apple's Private Cloud Compute, Meta's Private Processing, and Azure Confidential VMs / Google Confidential Space made "the operator cannot read it" a consumer-visible claim, not a B2B SKU.
- **Procurement language hardened.** Compliance text now literally reads: *model weights and inference inputs must be protected against access by the infrastructure provider, attested at workload launch, and never appear in plaintext in host memory.* That sentence is a TEE requirement written as a purchase order.

Where it fits — and where it does not:

```
┌──────────────────────────────────────────────────────────────────────┐
│  GOOD FIT (a TEE is doing real work)                                 │
│  • Regulated data you cannot show the host: health, finance, gov     │
│  • Proprietary weights + untrusted/foreign infrastructure            │
│  • Multi-party data clean rooms (two banks, one model, no raw data   │
│    ever visible to either counterparty or the cloud)                 │
│  • Cloud bursting / colo where you do not own the rack               │
│  • Sovereignty rules that forbid foreign staff touching the data     │
│                                                                      │
│  NOT WORTH IT (the operational tax buys nothing)                     │
│  • Any workload you would happily run in your own datacenter         │
│  • Public data + public model: no secret exists to protect           │
│  • Teams that will not run attestation verification — a TEE with     │
│    no verifier is decoration                                         │
│  • Latency-hard budget with autoscaling churn (attestation is part   │
│    of cold start; budget it, and pre-warm)                           │
└──────────────────────────────────────────────────────────────────────┘
```

Being honest about the threat model is what separates the deployments that hold from the ones that get breached. A TEE does **not** defend against **side channels** (microarchitectural leakage is relocated, not eliminated), **output leakage** (memory isolation does not stop a jailbroken or poisoned model from exfiltrating through its own answers), **compromised supply chain** (a poisoned weight file loads perfectly — and privately), **a lazy verifier**, or a **debug/swap path left enabled** (SEV-SNP's `DEBUG` flag and TDX debug mode must be off; the host must not be able to core-dump guest memory).

One honest comparison worth pinning, because the two get blurred constantly — and because this journal already covered **Homomorphic Encryption (FHE)**:

```
TEE (CONFIDENTIAL COMPUTING)   vs   FHE (HOMOMORPHIC ENCRYPTION)
════════════════════════════════════════════════════════════════

  TRUST ANCHOR
    TEE  a hardware vendor's silicon (AMD / Intel / ARM / NVIDIA)
    FHE  nobody — the security is mathematical, not physical

  DATA IN USE
    TEE  plaintext, but only inside a sealed + attested box
    FHE  ciphertext end to end — never decrypted anywhere

  vs MALICIOUS ADMIN / HYPERVISOR
    TEE  ✓ blocked (host sees ciphertext)
    FHE  ✓ blocked (there is nothing to see)

  vs SIDE-CHANNEL ATTACK
    TEE  ⚠ residual risk — leakage is relocated, not removed
    FHE  ✓ structurally immune (no hardware is trusted)

  POST-QUANTUM
    TEE  ⚠ AES-based, breakable by a large quantum computer
    FHE  ✓ lattice-based, believed quantum-safe

  CODE CHANGE TO ADOPT
    TEE  ~none — deploy the same binary into a TEE
    FHE  full rewrite: arithmetic must become FHE arithmetic

  TIME TO PRODUCTION
    TEE  days to weeks
    FHE  months to years, with cryptography specialists

  COST
    TEE  ~1.0–1.1× baseline (3–7% overhead in 2026)
    FHE  ~100×–1,000×+ — usually fatal for interactive work
```

The pragmatic reading: **FHE is the stronger security claim, TEEs are the only one you can actually ship this year.** They are complements — and FHE is quietly useful in exactly the niches a TEE cannot serve, like when no one at all may hold a decryption key.

### Example:

"KlinikSehat", a Jakarta health-tech, wants to launch an LLM triage assistant that reads patient records. Its constraints are brutal but normal: the hospital contracts forbid the cloud provider from ever accessing patient data; Indonesian personal-data law requires demonstrable technical controls; and the model is a fine-tuned, million-dollar asset that must not leak to the provider either. They refuse to build a datacenter. So they deploy a **confidential inference stack** — and the shape of the system changes from an architecture diagram into a *handshake*.

```
CONFIDENTIAL INFERENCE, END TO END — nothing readable until attestation passes
══════════════════════════════════════════════════════════════════════════════

   CLINIC (verifier side)                      CLOUD (untrusted operator)
  ┌───────────────────────┐
  │ client SDK / KMS      │  1. send a NONCE (freshness / anti-replay)
  │ reference values      │──────────────────────────────►  starts a CVM
  │  pinned image hash    │                               with the pinned
  │  min TCB version      │                               image hash
  │  debug = OFF          │
  └──────────┬────────────┘
             │                             2. TEE asks its HARDWARE for Evidence
             │                                ├─ SEV-SNP attestation report (VCEK-signed)
             │                                ├─ Intel TDX Quote
             │                                └─ NVIDIA GPU CC attestation report
             │                                …the GPU's report is bound to the CPU's
             │                                → the two are ONE verified composite
             │  ◄─────── signed Evidence ──────┘
             │
             ▼  3. VERIFY against policy (OPA/Rego, not a hard-coded ==):
                ├─ measurement == our image hash?             ✓
                ├─ TCB ≥ minimum microcode version?           ✓
                ├─ nonce matches the one we just sent?        ✓ (no replay)
                ├─ debug flag OFF, no core-dump path?         ✓
                └─ signature chains to AMD/Intel/NVIDIA root  ✓
                        │
                        │  ✗ ANY FAILURE → abort, release nothing.
                        │    That is the entire point of the pattern.
                        ▼
  ┌──────────────────────────────────────────────────────────────────┐
  │  4. UNSEAL — key release is GATED BY ATTESTATION                 │
  │                                                                  │
  │   KMS policy: kms:RecipientAttestation == <expected measurement> │
  │        │                                                         │
  │        │  KMS refuses to hand over the model-decryption key to   │
  │        │  anything that cannot prove what it is. A host that     │
  │        └─ steals the encrypted weights still gets 0x41bc…  ✓     │
  └──────────────────────────────────────────────────────────────────┘
                        │
                        ▼
  ┌──────────────────────────────────────────────────────────────────┐
  │  5–6. LOAD + SERVE, all inside the TEE                           │
  │                                                                  │
  │   prompt "pasien Budi, 54, diabetes, metformin…"                 │
  │        │                                                         │
  │        ▼  plaintext exists ONLY inside encrypted DRAM/VRAM       │
  │   [ CPU TEE: prompt + tokenizer ]──PCIe(encrypted)──►[ GPU TEE:  │
  │                                                        weights + │
  │                                                        KV-cache ]│
  │        │                                                         │
  │        ▼  response encrypted to the clinic's session key         │
  │   "Saran triage: …"  ──►  clinic decrypts                        │
  └──────────────────────────────────────────────────────────────────┘

  MEANWHILE, THE CLOUD ADMIN'S DASHBOARD SHOWS:
  ┌──────────────────────────────────────────────┐
  │  CVM-7f3a   ATTESTED  ✓   TCB 3.1.2          │
  │  DRAM  0x9f3a: 9d 41 bc 77 de 00 3a …        │
  │  VRAM  0x41bc: c0 ff ee 1a 9f 3a 44 …        │
  │  egress: 2.1 GB   cpu: 41%   net: ok         │
  │  ── no prompt, no weights, no records ──     │
  └──────────────────────────────────────────────┘
```

Two design decisions carry most of the weight here, and both are easy to get wrong:

**The verifier is not the cloud.** If KlinikSehat accepts the provider's own attestation service verdict, it has merely moved the trust, not removed it — the operator can lie about a quote it validates itself. The IETF RATS framework (RFC 9334) names the roles precisely: the **Attester** (the TEE) produces **Evidence**; the **Verifier** checks it against **Reference Values**; the **Relying Party** (KlinikSehat's KMS) decides whether to release the key. Two standard topologies fall out:

```
BACKGROUND CHECK MODEL — every relying party verifies independently
────────────────────────────────────────────────────────────────────

   TEE ──Evidence──► Relying Party ──Evidence──► Verifier
                          ▲                          │
                          └───── Attestation Result ─┘
   decision: the RELYING PARTY releases the key, on a FRESH result
   cost:     every service that must decide needs verifier access

PASSPORT CHECK MODEL — verify once, then present a token
────────────────────────────────────────────────────────

   TEE ──Evidence──► Verifier ──► Attestation Result (signed token)
     │
     └──presents token──► Relying Party ──► validates the SIGNATURE only
   decision: any service can decide, with NO verifier access
   cost:     a reusable token exists — so bind it to a short TTL + audience

GOOD FOR:  few, high-stakes consumers     GOOD FOR:  many services / proxies
           key release to one KMS                      fleet-wide rollout gates
```

KlinikSehat self-hosts the verifier (Trustee-style) inside its own VPC for the high-stakes key release, and lets downstream proxies accept passport-style tokens so every microservice does not need to re-verify a hardware quote.

**Patching is an attestation event.** This is the operational trap nobody sees in the architecture diagram: every routine kernel update changes the launch measurement, so the *reference values* must be provisioned to the verifier **before** the updated guests boot — otherwise a fleet-wide patch at 02:00 becomes a fleet-wide attestation outage at 02:01. The mature pattern is policy, not equality: pin the measurement for the tight deployments, allow a *minimum TCB version* window during rollouts, and keep an explicit allow-list of hardware models and a hard rule that debug mode is off. "Measurement churn" is a first-class release-engineering concern the day you adopt confidential computing, not an afterthought.

Finally, the part KlinikSehat gets right by *not* trusting the TEE for everything. The TEE protects memory, not meaning. So the triage bot keeps three controls *outside* the enclave: output filtering and redaction on the response path (because a poisoned or jailbroken model exfiltrates happily through an attested channel — the pipe is private, the payload is not), rate limits and per-tenant quotas, and ECC memory plus disabled swap as a baseline, since memory corruption inside an enclave is far harder to recover from when the host is not allowed to intervene. And because attestation adds latency, they budget it into autoscaling — pre-warming confidentially, the same lesson as any cold start, with a verifier handshake stapled to the front.

The punchline: confidential computing is the first time the phrase *"trust no one"* became an actual machine instruction. For thirty years the cloud forced a trade — get scale and elasticity, and in exchange let the operator see your plaintext. TEEs plus attestation break that trade: the operator keeps the hardware, you keep the secrets, and a signed quote from the chip decides who is lying. It does not make a system safe — a TEE is a memory-isolation primitive, not a security programme, and teams that deploy one without a verifier, without output controls, and without a patch plan are buying a certificate rather than a control. But by 2026 the overhead has fallen below the noise floor and the frameworks ship it as a flag, which is why the honest framing is no longer "should we adopt confidential computing?" but "which of our workloads can we still afford to have the provider read?"

---