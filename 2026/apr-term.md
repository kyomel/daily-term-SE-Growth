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