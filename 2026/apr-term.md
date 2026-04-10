day - 1

## Garbage First Garbage Collector (G1GC)

### Definition:

Garbage First Garbage Collector (G1GC) is a region-based, server-oriented garbage collector introduced in Java 7 and made the default GC in Java 9 — designed to provide high throughput with predictable, configurable pause times by dividing the heap into equal-sized regions, prioritizing collection of regions with the most garbage first (hence "Garbage First"), and incrementally reclaiming memory without requiring a full heap collection.

G1GC was built to replace the Concurrent Mark Sweep (CMS) and Parallel GC collectors for applications that need both large heaps AND low, predictable pause times — the two goals that older collectors forced you to choose between.

— openjdk.org

The Fundamental Concept — Regions
The single most important thing to understand about G1GC:


Traditional GC Heap Layout:
┌──────────────────┬──────────────┬──────────────────────────┐
│   Young Gen      │  Survivor    │      Old Generation       │
│   (Eden)         │   Spaces     │                          │
│                  │              │                          │
│  New objects     │ Surviving    │  Long-lived objects       │
│  created here    │ objects      │                          │
└──────────────────┴──────────────┴──────────────────────────┘
Fixed, contiguous regions — must collect entire generation at once

G1GC Heap Layout:
┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ E  │ S  │ O  │ O  │ E  │ O  │ S  │ E  │ H  │ O  │ E  │ O  │
├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
│ O  │ E  │ O  │ E  │ O  │ O  │ E  │ O  │ O  │ E  │ S  │ O  │
├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
│ O  │ O  │ E  │ H  │ O  │ E  │ O  │ O  │ E  │ O  │ O  │ E  │
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘

E = Eden region    (new objects)
S = Survivor region (surviving young objects)
O = Old region     (long-lived objects)
H = Humongous region (very large objects, > 50% of region size)

Key differences:
  ✅ Heap split into 2,048 equal-sized regions (1MB–32MB each)
  ✅ Any region can play any role — flexible assignment
  ✅ Regions collected INDEPENDENTLY — no full heap needed
  ✅ G1GC picks the regions with MOST garbage first

### Example:

E-Commerce Order Processing Service
A Java-based order processing service handles variable traffic — quiet at night, massive spikes during flash sales. Here is how G1GC behavior changes across scenarios.

```
Order processing service benchmark:
  8GB heap, 5,000 orders/minute, 4-hour test

                    Parallel GC    CMS        G1GC
                    ───────────    ───────    ────────────────────
  Max pause:        4,200ms ❌    850ms ⚠️   98ms  ✅
  Avg pause:        1,800ms ❌    120ms ✅    45ms  ✅
  Throughput:       97%    ✅     91%   ⚠️   94%   ✅
  Heap fragmented:  No     ✅     YES   ❌    No    ✅
  Full GC events:   2      ⚠️    8     ❌    0     ✅
  Predictable?:     No     ❌     No    ❌    YES   ✅
  Tuning effort:    Low    ✅     High  ❌    Medium ✅

  Orders lost to timeout (pause > 500ms = user-facing error):
  Parallel GC:      1,240 orders lost ❌
  CMS:                180 orders lost ⚠️
  G1GC:               0   orders lost ✅
```

---

day - 6

## AI-Assisted Testing

### Definition:
AI-Assisted Testing is the practice of using artificial intelligence and machine learning models — including large language models (LLMs), code generation models, and intelligent analysis engines — to augment, accelerate, and enhance software testing activities across the entire testing lifecycle: from test generation and maintenance to test execution, analysis, bug detection, and coverage optimization — while keeping humans in control of testing strategy and quality decisions.

AI-Assisted Testing does not replace testers — it eliminates the tedious, repetitive, and time-consuming parts of testing so human testers can focus on what machines cannot do: creative exploration, risk judgment, and understanding user intent.

Background — Why AI-Assisted Testing Exists
Software testing has always been the bottleneck of software delivery:


The Testing Crisis (before AI assistance):

  Modern application has:
    → 500,000 lines of code
    → 3,000+ functions and methods
    → 150+ API endpoints
    → Dozens of UI flows and edge cases
    → New features shipped every 2 weeks

  Human testing capacity:
    → QA team: 5 engineers
    → Write tests manually: ~10 tests/engineer/day
    → Maintain existing tests: eats 40% of their time
    → Regression suite: 4 hours to run
    → Flaky tests: 15% fail randomly → ignored

  The result:
    ❌ Only 40% of code paths have any test coverage
    ❌ Developers wait 4 hours for feedback from CI
    ❌ 60% of QA time spent maintaining old tests
    ❌ New features shipped with minimal test coverage
    ❌ Bugs reach production regularly
    ❌ "We don't have time to write tests"

  AI-Assisted Testing changes the equation:
    ✅ Generate 100 tests in seconds, not days
    ✅ Self-healing tests that fix themselves after UI changes
    ✅ Intelligent test selection — run only what matters
    ✅ Automatic edge case and boundary detection
    ✅ Visual regression caught automatically
    ✅ QA engineers focus on strategy, not maintenance

### Example:
Payment API Testing with AI
A fintech company builds a new payment processing API. Here is how AI-assisted testing works across the full development cycle.

```
# AI generates comprehensive API tests from the OpenAPI spec
# Tool: Schemathesis + AI enhancement, or Postman AI, or custom LLM

# AI-GENERATED: test_payment_api.py

import pytest
import requests

BASE_URL = "http://localhost:8080"

class TestPaymentAPIGenerated:
    """Tests generated by AI from OpenAPI specification"""

    # ── Valid Requests ─────────────────────────────────────

    @pytest.mark.parametrize("currency", ["USD", "EUR", "GBP"])
    def test_valid_currencies(self, currency):
        # AI: test every value in the enum
        response = requests.post(f"{BASE_URL}/payments", json={
            "cardToken":  "tok_valid",
            "amount":     100.0,
            "currency":   currency,
            "customerId": "CUST-123"
        })
        assert response.status_code == 201

    @pytest.mark.parametrize("amount", [
        0.01,      # minimum valid
        1.00,      # normal
        100.00,    # normal
        49999.99,  # just under limit
        50000.00   # exactly at limit
    ])
    def test_valid_amounts(self, amount):
        # AI: test boundary values for amount field
        response = requests.post(f"{BASE_URL}/payments", json={
            "cardToken":  "tok_valid",
            "amount":     amount,
            "currency":   "USD",
            "customerId": "CUST-123"
        })
        assert response.status_code == 201

    # ── Invalid Requests — AI Found These ────────────────

    @pytest.mark.parametrize("bad_payload,expected_status", [
        # Missing required fields
        ({},                                                400),
        ({"cardToken": "tok"},                             400),
        ({"amount": 100, "currency": "USD"},               400),

        # Invalid values
        ({"cardToken": "", "amount": 100,
          "currency": "USD", "customerId": "C"},           400),
        ({"cardToken": "tok", "amount": 0,      # zero
          "currency": "USD", "customerId": "C"},           400),
        ({"cardToken": "tok", "amount": -1,     # negative
          "currency": "USD", "customerId": "C"},           400),
        ({"cardToken": "tok", "amount": 50001,  # over limit
          "currency": "USD", "customerId": "C"},           422),
        ({"cardToken": "tok", "amount": 100,
          "currency": "JPY",                    # invalid enum
          "customerId": "C"},                              400),

        # Type mismatches
        ({"cardToken": 12345, "amount": 100,    # number not string
          "currency": "USD", "customerId": "C"},           400),
        ({"cardToken": "tok", "amount": "100",  # string not number
          "currency": "USD", "customerId": "C"},           400),

        # Injection attempts (AI adds security cases)
        ({"cardToken": "'; DROP TABLE payments;--",
          "amount": 100, "currency": "USD",
          "customerId": "C"},                              400),
        ({"cardToken": "<script>alert(1)</script>",
          "amount": 100, "currency": "USD",
          "customerId": "C"},                              400),
    ])
    def test_invalid_payloads(self, bad_payload, expected_status):
        response = requests.post(f"{BASE_URL}/payments",
                                 json=bad_payload)
        assert response.status_code == expected_status

    # AI also generates: response schema validation,
    # rate limiting tests, concurrent request tests,
    # idempotency tests, timeout behavior tests...

# 35+ tests generated from OpenAPI spec in seconds
# Without AI: would take QA engineer 1-2 days to write ✅
```

---

day - 7

## Load Shedding Architecture

### Definition:

Load Shedding Architecture is a deliberate system design strategy where a service or system intentionally drops, rejects, or degrades a portion of incoming requests when it detects it is operating beyond its safe capacity — protecting core system stability and serving remaining requests well, rather than attempting to process everything and collapsing entirely under the overload.

Load Shedding is the engineering equivalent of a controlled sacrifice — deliberately letting some work fail gracefully so that the system as a whole continues to function, rather than letting everything fail catastrophically.

Background — Why Load Shedding Exists
Every system has a capacity limit — and what happens beyond that limit defines survivability:


What happens when traffic exceeds capacity:

  WITHOUT Load Shedding:
    Traffic doubles beyond capacity
         │
         ▼
    All requests slow down
         │
         ▼
    Threads/connections exhausted
         │
         ▼
    Memory fills, GC thrashes
         │
         ▼
    Response times: 30 seconds
         │
         ▼
    Entire system collapses ❌
    ALL users get nothing
    Recovery time: minutes to hours

  WITH Load Shedding:
    Traffic doubles beyond capacity
         │
         ▼
    System detects overload
         │
         ▼
    Sheds 50% of lowest-priority requests
    (returns HTTP 429 / 503 immediately)
         │
         ▼
    Remaining 50% served normally ✅
    Response times: still fast
         │
         ▼
    System stays healthy and responsive
    50% of users fully served
    50% get fast "try again" response

  Partial service > total collapse — always

### Example:
Streaming Platform During Live Event
A streaming platform hosts a massive live concert — expecting 10x normal traffic. Here is how Load Shedding Architecture keeps the platform alive.

```
                     Internet
                        │
                 50M concurrent users
                 (normal: 5M)
                        │
                        ▼
           ┌────────────────────────┐
           │      CDN Layer         │
           │  (handles static       │
           │   assets, edge cache)  │
           └───────────┬────────────┘
                       │ dynamic requests only
                       ▼
           ┌────────────────────────┐
           │   API Gateway with     │
           │   Load Shedding Gate   │◀─── Real-time metrics
           │                        │     collector
           │   Priority classifier  │
           │   Capacity monitor     │
           │   Shed decision engine │
           └──────┬─────────────────┘
                  │
      ┌───────────┼─────────────┐
      │           │             │
      ▼           ▼             ▼
  TIER 1       TIER 2       TIER 3
  Protected    Important    Deferrable
  Stream API   Search API   Recommend
  Auth API     Profile API  API (shed)
  Payment API  Chat API     Social API
```

---

day - 8

## GeoDNS

### Definition:

GeoDNS (Geographic Domain Name System) is a DNS-based traffic routing technique that returns different IP addresses in response to DNS queries based on the geographic location of the requester — directing users to the nearest, fastest, or most appropriate server automatically, without any change to the user's browser, application, or behavior.

GeoDNS turns the DNS lookup — the first step every internet connection makes — into an intelligent traffic router that silently directs each user to the optimal server for their location, the moment they type a URL.

Background — Why GeoDNS Exists
The internet is global. Servers are not everywhere. Physics is unavoidable:


The Latency Problem — without GeoDNS:

  One server in New York serves the entire world:

  User in New York    → 8ms latency    ✅ fast
  User in London      → 95ms latency   ⚠️ acceptable
  User in Mumbai      → 220ms latency  ❌ slow
  User in Tokyo       → 190ms latency  ❌ slow
  User in São Paulo   → 160ms latency  ❌ slow
  User in Sydney      → 240ms latency  ❌ very slow

  Every DNS query for "api.example.com"
  returns the SAME IP: 203.0.113.10 (New York)
  regardless of where the user is located

  Impact:
    ❌ 80% of global users experience poor performance
    ❌ Single point of failure — NY goes down = world goes down
    ❌ All global traffic crosses the Atlantic/Pacific
    ❌ Bandwidth costs are enormous on one datacenter
    ❌ Compliance issues — EU data must stay in EU

  With GeoDNS:
    User in New York  → 203.0.113.10 (New York)    8ms  ✅
    User in London    → 185.12.45.22 (Frankfurt)   18ms ✅
    User in Mumbai    → 103.45.67.89 (Singapore)   28ms ✅
    User in Tokyo     → 210.32.45.11 (Tokyo)        5ms ✅
    User in São Paulo → 177.88.32.45 (São Paulo)   12ms ✅
    User in Sydney    → 202.45.67.88 (Sydney)       9ms ✅

  Each user automatically routed to nearest server
  Zero configuration change on the user side

### Example:

Global E-Commerce Platform
An e-commerce company sells worldwide and uses GeoDNS to ensure every shopper gets fast, compliant, locally relevant service.

```
E-Commerce Platform — 3-month comparison:

                        Before GeoDNS    After GeoDNS    Change
                        ─────────────    ────────────    ──────
Global avg latency:     185ms            18ms           ↓ 90% ✅
Asia-Pacific latency:   230ms            12ms           ↓ 95% ✅
Europe latency:         95ms             15ms           ↓ 84% ✅
South America latency:  165ms            11ms           ↓ 93% ✅

Page load time (global  4.8 seconds      1.1 seconds    ↓ 77% ✅
  avg, including assets)

Conversion rate:        2.1%             3.4%           ↑ 62% ✅
  (faster = more sales — 100ms latency ≈ 1% conversion drop)

Uptime (global):        99.2%            99.97%         ✅
  (single datacenter vs regional failover)

GDPR compliance:        Manual process   Automatic ✅
  EU data in EU:        (risky)          (enforced by routing)

Bandwidth costs:        $48,000/mo       $31,000/mo     ↓ 35% ✅
  (regional servers — shorter data paths, cheaper egress)

Customer complaints     847/month        94/month       ↓ 89% ✅
  about slowness:
```

---

day - 9

## Real-Time Ad Bidding Systems (RTB)

### Definition:

Real-Time Bidding (RTB) is an automated digital advertising process where ad impressions are bought and sold through instant auctions that occur in the time it takes a webpage to load — typically under 100 milliseconds. Every time a user visits a webpage, advertisers automatically compete in a live auction to show their ad to that specific user, with the highest bidder winning the ad slot.

An impression is simply one instance of an ad being displayed to a user on a webpage or app.

💡 Simple Analogy
Imagine a lightning-fast stock exchange, but instead of trading company shares, you're trading eyeballs on a screen. The moment someone visits a webpage, thousands of advertisers instantly bid for the right to show that specific person their ad — all before the page even finishes loading.

### Example:

Scenario: Nike Targeting a Runner
Sarah is a 28-year-old marathon runner. She visits ESPN.com to check scores.

```
What happens behind the scenes in 100ms:


1️⃣  ESPN's SSP fires a Bid Request:
    ┌─────────────────────────────────────────┐
    │ User ID:      #XJ-4829                  │
    │ Age:          28                        │
    │ Interests:    Running, Fitness, Sports  │
    │ Location:     Chicago, IL               │
    │ Device:       iPhone 15                 │
    │ Page:         ESPN.com/scores           │
    │ Ad Slot:      300x250 banner, above fold│
    └─────────────────────────────────────────┘

2️⃣  Ad Exchange broadcasts to all DSPs

3️⃣  DSPs evaluate and bid:
    Nike DSP    → $4.20  ✅ (Sarah matches runner profile)
    Adidas DSP  → $3.80
    Netflix DSP → $1.20  (not their target)
    Booking.com → $2.50

4️⃣  Nike WINS at $4.20

5️⃣  Nike's "Spring Running Shoes" ad appears
    on Sarah's ESPN page instantly 🏃‍♀️👟
```

---

day - 10

## Toy API Calls

### Definition:

Toy API Calls is a software development learning and prototyping technique where a developer makes the simplest possible, stripped-down API request — using hardcoded values, minimal parameters, and throwaway code — with the sole purpose of understanding how an API works, what it returns, and how to interact with it before writing any real production code.

A Toy API Call is intentionally not meant to be kept, scaled, or shipped — it is a scientific experiment on an API: the fastest possible way to see a real response, understand the data shape, and build mental model of how the API behaves — before investing time in proper integration.

Background — Why Toy API Calls Exist
Every API integration starts with the same problem: you don't know what you don't know:


The API Integration Problem (without Toy API Calls):

  Developer assigned task:
  "Integrate Stripe payment processing into checkout"

  Wrong approach (jump straight to production code):
    → Write PaymentService class with full abstraction
    → Wire up dependency injection
    → Write error handling for every case
    → Write unit tests with mocks
    → Submit PR for review
    → PR review reveals: "You're using the wrong API endpoint"
    → "The response shape is completely different from docs"
    → "This parameter doesn't do what you thought"
    → Entire class rewritten ❌
    → 2 days wasted ❌

  Right approach (Toy API Call first):
    → Open scratch file or terminal
    → Paste in hardcoded API key
    → Make ONE raw request to Stripe
    → See the REAL response in 60 seconds
    → Understand the actual data shape
    → NOW write the production class — correctly
    → 10 minutes of exploration saves 2 days ✅

  Toy API Calls are the API equivalent of
  "measure twice, cut once"

### Example:

Integrating OpenAI API
A developer needs to add AI-generated product descriptions to an e-commerce platform. They have never used the OpenAI API before.

```
# scratch.py — iteration 2
# Goal: Does it work for MY specific use case?
# (product description generation)

import openai
import json

client = openai.OpenAI(api_key="sk-proj-abc123hardcoded")

# Hardcoded product data — real code would pass this as parameter
product = {
    "name":     "Wireless Noise-Cancelling Headphones",
    "brand":    "SoundMax",
    "features": ["40hr battery", "ANC", "Bluetooth 5.3", "USB-C"],
    "price":    "$149"
}

# Testing different prompt approaches — toy call lets us iterate fast
response = client.chat.completions.create(
    model       = "gpt-4o",
    max_tokens  = 150,
    temperature = 0.7,
    messages    = [
        {
            "role":    "system",
            "content": "You write compelling e-commerce product descriptions."
                       " Be concise, benefit-focused, and persuasive."
        },
        {
            "role":    "user",
            "content": f"Write a 2-sentence product description for: "
                       f"{json.dumps(product)}"
        }
    ]
)

# Print just the content we care about
print("=== GENERATED DESCRIPTION ===")
print(response.choices[0].message.content)
print()
print("=== USAGE ===")
print(f"Tokens used: {response.usage.total_tokens}")
print(f"Finish reason: {response.choices[0].finish_reason}")

# OUTPUT:
# === GENERATED DESCRIPTION ===
# Experience immersive audio freedom with SoundMax's Wireless
# Noise-Cancelling Headphones — featuring advanced ANC technology
# and an impressive 40-hour battery life to keep you in your
# world, uninterrupted. Connect effortlessly via Bluetooth 5.3
# and recharge conveniently with USB-C, all for just $149.
#
# === USAGE ===
# Tokens used: 112
# Finish reason: stop

# Developer learns:
# ✅ Quality is good enough for production
# ✅ 112 tokens per description → cost is calculable ($0.0003)
# ✅ finish_reason='stop' means it completed naturally
# ✅ temperature=0.7 feels right — creative but coherent
# ✅ JSON product data works well in the prompt
# 🤔 What happens if product data is missing fields?
```

---