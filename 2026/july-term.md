day - 1

## Loop Engineering

### Definition:

Loop Engineering is the discipline of designing, optimizing, and debugging the iterative feedback loops that an AI agent uses to reason, act, and learn from results. Instead of a single linear prompt → response flow (like a traditional LLM call), an agentic system runs in a loop: perceive → think → act → observe → think again → act again → repeat until the goal is reached.

The core loop that every agent system runs looks like this:

                    ┌─────────────────────────────┐
                    │     1. PERCEIVE              │
                    │   (read tools, environment)  │
                    └─────────────┬───────────────┘
                                  ▼
                    ┌─────────────────────────────┐
                    │     2. REASON                │
                    │   (plan next action,         │
                    │    evaluate context)         │
                    └─────────────┬───────────────┘
                                  ▼
                    ┌─────────────────────────────┐
                    │     3. ACT                   │
                    │   (call a tool: terminal,    │
                    │    read_file, web_search)    │
                    └─────────────┬───────────────┘
                                  ▼
                    ┌─────────────────────────────┐
                    │     4. OBSERVE               │
                    │   (tool returns output)      │
                    └─────────────┬───────────────┘
                                  │
                          ┌───────┴───────┐
                          │               │
                      Goal met?      Not yet?
                          │               │
                          ▼               │
                    ┌──────────┐          │
                    │   DONE   │          │
                    └──────────┘          │
                          ▲               │
                          └───────────────┘

Why the loop matters:

A non-agentic LLM call is one-shot: prompt → response. If the response is wrong, you have to manually correct it. An agentic loop turns this into a self-correcting process: the agent can run a command, see the output, realize it was wrong, and try a different approach — all without human intervention.

Key loop engineering patterns:

| Pattern | What It Does | Example |
|---------|-------------|---------| (2/11)
| ReAct Loop | Reason → Act → Observe — interleave thinking with tool use | "I need to find the file. Let me search for it. I found it. Now let me read it. I see the bug. Now let me fix it." |
| Reflection Loop | Generate output → Critique output → Refine | Write code → Check for bugs → Fix bugs → Recheck |
| Retry Loop | Try → Fail → Backoff → Retry | Call API → Timeout → Wait 1s → Retry (exponential backoff) |
| Supervision Loop | Act → Human approves? → Proceed or Abort | Deploy to prod → Ask user "Approve this change?" → Yes/No |
| Tree-of-Thought | Branch → Evaluate → Prune → Expand best branch | Generate 3 approaches → Score each → Explore the best one → Repeat |
| Planning Loop | Plan steps → Execute step 1 → Update plan → Execute step 2 | "Step 1: clone repo. Done. Step 2: install deps. Failed, try alternative." |

### Example:

A comparison of a non-agentic (single-shot) approach vs an agentic loop approach for the same task — finding and fixing a bug in a Python file.

```
import re
import json
import time
from dataclasses import dataclass, field
from typing import Optional, List


# ═══════════════════════════════════════════════
#  Simulated Environment
# ═══════════════════════════════════════════════

# A simulated file with a bug
BUGGY_FILE = """
def calculate_discount(price: float, is_member: bool) -> float:
    \"\"\"Apply member discount to price.\"\"\"
    if is_member:
        return price * 0.9  # 10% discount
    else:
        return price

def checkout(items: list[dict]) -> float:
    \"\"\"Calculate total for all items in cart.\"\"\"
    total = 0
    for item in items:
        total += item["price"]
    member = items[0]["is_member"]  # 🐛 BUG: assumes all items have same member status
    return calculate_discount(total, member)

def test_checkout():
    \"\"\"Test the checkout function.\"\"\"
    cart = [
        {"name": "shirt", "price": 25.0, "is_member": True},
        {"name": "pants", "price": 50.0, "is_member": True},
    ]
    result = checkout(cart)
    expected = (25.0 + 50.0) * 0.9
    print(f"Result: {result}, Expected: {expected}")
    assert result == expected, f"Mismatch: {result} != {expected}"
    print("✅ Test passed!")

test_checkout()
"""


# ═══════════════════════════════════════════════
#  Approach 1: NON-AGENTIC (Single Shot)
# ═══════════════════════════════════════════════

class SingleShotAgent:
    """No loop. One prompt, one answer. Cannot self-correct."""

    def run(self, task: str) -> dict:
        # Simulating a single LLM call — no observation, no iteration
        print(f"📝 Task: {task}")
        print(f"🤖 Response: 'The bug is in line 12: 'member = items[0][\"is_member\"]'")
        print(f"              This assumes all items share the same member status.")
        print(f"              Fix: aggregate member status per item.'")
        print(f"⚠️  But is the fix correct? The agent can't check.")
        "Fix: Instead of taking member status from items[0], "
                        "check each item individually. If ANY item requires member "
                        "pricing logic, we need to apply discount per item or "
                        "determine member status differently."
                    )
        
                if action == "run_test":
                    # Simulate running the test
                    result = (25.0 + 50.0) * 0.9
                    return f"Test output: Result: {result}, Expected: {result}. ✅ Test passed!"
        
                if action == "verify_bug_exists":
                    # The test passes because both items have is_member=True!
                    # The bug only manifests with mixed membership.
                    return (
                        "⚠️  Test passed because both test items have is_member=True. "
                        "The bug is still there — it just doesn't show in this test case. "
                        "Need a better test with mixed member/non-member items to reproduce."
                    )
        
                if action == "reproduce_bug":
                    # Create a test case that triggers the bug
                    return (
                        "Edge case test: cart with mixed membership\n"
                        "cart = [\n"
                        '    {"name": "shirt", "price": 25.0, "is_member": True},\n'
                        '    {"name": "pants", "price": 50.0, "is_member": False},\n'
                        "]\n"
                        "Expected: 25*0.9 + 50 = 72.5\n"
                        "Got: (25+50)*0.9 = 67.5\n"
                        "❌ Bug reproduced! Non-member item got a discount it shouldn't."
                    )
        
                return "Unknown action"
        
            def run(self, task: str) -> List[AgentAction]:
                print(f"📝 Task: {task}\n")
                print(f"🔄 AGENTIC LOOP — Step-by-Step\n")
                print(f"{'='*60}")
        
                loop = [
                    {
                        "thought": "First, I need to read the file to understand the bug.",
                        "action": "read_file",
                        "args": {},
                    },
                    {
                    "thought": "Let me search for suspicious patterns — 'items[0]' is often a bug.",
                                    "action": "search",
                                    "args": {"pattern": "items[0]"},
                                },
                                {
                                    "thought": "Found 'items[0][\"is_member\"]'. This assumes all items share the same member status — that's the bug.",
                                    "action": "analyze_bug",
                                    "args": {},
                                },
                                {
                                    "thought": "Let me verify by creating a test case that exposes the bug with mixed membership.",
                                    "action": "reproduce_bug",
                                    "args": {},
                                },
                                {
                                    "thought": "Bug confirmed. The fix should calculate discount per-item based on each item's membership, not assume uniformity.",
                                    "action": "propose_fix",
                                    "args": {},
                                },
                            ]
                    
                            for i, step in enumerate(loop, 1):
                                start = time.time()
                    
                                print(f"\n🔷 Step {i}/{len(loop)}")
                                print(f"   💭 {step['thought']}")
                                print(f"   🛠️  Action: {step['action']}({step['args']})")
                    
                                result = self._simulate_tool_call(step["action"], step["args"])
                                elapsed = (time.time() - start) * 1000
                    
                                print(f"   📥 Result:")
                                for line in result.split("\n")[:5]:
                                    print(f"      {line}")
                                if len(result.split("\n")) > 5:
                                    print(f"      ... ({len(result.split('\\n')) - 5} more lines)")
                    
                                self.history.append(AgentAction(
                                    step=i,
                                    thought=step["thought"],
                                    action=step["action"],
                                    result=result,
                                    duration_ms=round(elapsed, 1),
                                ))
                    
                            # Final reflection
                            reflection = (
                                f"\n✅ Loop complete after {len(loop)} steps.\n"
                                f"   Diagnosis: Found bug at line 12 (items[0]['is_member'] assumes uniform membership)\n"
                                f"   Reproduced: Created edge case that triggers the bug\n"
                                            f"   Verification: Non-member items incorrectly get discount\n"
                                            f"   Confidence: HIGH — bug is understood and reproducible\n"
                                        )
                                        print(reflection)
                                
                                        return self.history
                                
                                
                                # ═══════════════════════════════════════════════
                                #  Run Comparison
                                # ═══════════════════════════════════════════════
                                
                                print("📐 LOOP ENGINEERING — Agentic vs Non-Agentic Comparison")
                                print("=" * 70)
                                
                                # Approach 1
                                print("\n❌ APPROACH 1: NON-AGENTIC (Single Shot)")
                                print("-" * 40)
                                single = SingleShotAgent()
                                single.run("Find the bug in checkout()")
                                
                                # Approach 2
                                print("\n✅ APPROACH 2: AGENTIC LOOP (Perceive → Think → Act → Observe → Repeat)")
                                print("-" * 40)
                                looped = AgenticLoop()
                                actions = looped.run("Find the bug in checkout()")
                                
                                # Summary comparison
                                print(f"\n{'='*70}")
                                print(f"📊 COMPARISON SUMMARY")
                                print(f"{'='*70}")
                                print(f"\n{'Dimension':<35} {'Single Shot':<18} {'Agentic Loop':<18}")
                                print(f"{'-'*70}")
                                print(f"{'Steps taken':<35} {'1 (one answer)':<18} {'5 (iterative)':<18}")
                                print(f"{'Self-corrects?':<35} {'❌ No':<18} {'✅ Yes — rethinks':<18}")
                                print(f"{'Can verify?':<35} {'❌ No':<18} {'✅ Yes — tests it':<18}")
                                print(f"{'Handles edge cases?':<35} {'❌ No':<18} {'✅ Yes — probes':<18}")
                                print(f"{'Finds root cause?':<35} {'❌ Surface only':<18} {'✅ Yes — deep':<18}")
                                print(f"{'Confidence':<35} {'Low (unverified)':<18} {'High (tested)':<18}")
                                print(f"{'Token cost':<35} {'Low':<18} {'Higher (but verified)':<18}")
                                
                                print(f"\n{'='*70}")
                                print(f"🏁 KEY INSIGHT:")
                                print(f"   The single-shot agent found the bug once (good!).")
                                print(f"   But it couldn't verify its fix. The agentic loop went")
                                print(f"   deeper — it discovered the test didn't reproduce the bug,")
                                print(f"   created a proper edge case, confirmed the root cause,")
                                print(f"   and built confidence through iteration.")
                                print(f"   That's the power of loop engineering.")
                                📐 LOOP ENGINEERING — Agentic vs Non-Agentic Comparison
                                ======================================================================
                                
                                ❌ APPROACH 1: NON-AGENTIC (Single Shot)
                                ----------------------------------------
                                📝 Task: Find the bug in checkout()
                                🤖 Response: 'The bug is in line 12...'
                                ⚠️  But is the fix correct? The agent can't check.
                                
                                ✅ APPROACH 2: AGENTIC LOOP (5 Steps)
                                ----------------------------------------
                                🔷 Step 1: Read file to understand the bug
                                🔷 Step 2: Search for suspicious patterns — found items[0]
                                🔷 Step 3: Analyze — "items[0]['is_member'] assumes uniform membership"
                                🔷 Step 4: Reproduce — created edge case that triggers the bug
                                🔷 Step 5: Propose verified fix with high confidence
                                
                                ======================================================================
                                📊 COMPARISON SUMMARY
                                ======================================================================
                                
                                Dimension                        Single Shot        Agentic Loop      
                                ----------------------------------------------------------------------
                                Self-corrects?                   ❌ No              ✅ Yes — rethinks  
                                Can verify?                      ❌ No              ✅ Yes — tests it  
                                Handles edge cases?              ❌ No              ✅ Yes — probes    
                                Finds root cause?                ❌ Surface only     ✅ Yes — deep      
                                Confidence                       Low (unverified)   High (tested)      
```

---

day - 2

## Half Engineer, Half AI Systems Architect

### Definition:

The modern Senior Developer role has silently split into two jobs disguised as one. The first half is the traditional engineering role we all trained for — systems design, code review, debugging, mentoring. The second half is a new role that didn't exist three years ago: AI Systems Architect — designing, governing, and optimizing the AI-assisted workflows that the entire team depends on every day.

This isn't about becoming a data scientist or training ML models. It's about being the person who answers questions like:

- "Which tasks should AI generate vs. just assist vs. never touch?"
- "Which model is best for code review vs. testing vs. architecture brainstorming?"
- "How do we know when AI output is wrong in subtle, non-obvious ways?"
- "Why did my teammate get terrible output from the same prompt that works for me?"

The role split visualized:
The Senior Developer (2026)
┌──────────────────────────────┐
│                              │
┌──────────────┴──────────────┐               │
│                             │               │
▼                             ▼               │
┌──────────────────┐   ┌──────────────────────┐    │
│   ENGINEER (½)   │   │ AI SYSTEMS ARCHITECT │    │
│                  │   │       (½)            │    │
│ • Systems design │   │ • AI workflow design │    │
│ • Code review    │   │ • Model selection    │    │
│ • Testing        │   │ • Quality governance │    │
│ • Debugging      │   │ • Prompt libraries   │    │
│ • On-call        │   │ • Failure mode maps  │    │
│ • Mentoring      │   │ • Team AI culture    │    │
└──────────────────┘   └──────────────────────┘    │
│                             │               │
└──────────────┬──────────────┘               │
▼                              │
Companies pay 18-31%                     │
premium for both halves                  │
────────────────────────                 │
          ┌────────────────────┘
          ▼
"Senior Developer" title now
implicitly means both roles

The four new responsibilities of the AI Systems Architect half:

| # | Responsibility | What It Means Day-to-Day |
|---|----------------|--------------------------|
| 1 | Workflow Design | Decide which tasks are AI-generated (write tests), AI-assisted (draft PR description), AI-reviewed (scan for security issues), or human-only (approve production deployments). Move from accidental habits to deliberate design. | (2/8)
| 2 | Model Selection & Configuration | Choose the right model for each task — cheap flash for quick drafts, powerful pro for complex reasoning. Set context window strategies, configure fallbacks when a model is down, decide when to use open-source vs. proprietary. |
| 3 | Quality Governance | Map the failure modes of AI tooling. When does the AI suggest plausible-looking code that's actually wrong? What security patterns does it consistently mishandle? What naming conventions does it ignore? Build institutional knowledge about "right-looking wrong answers." |
| 4 | Team Prompting Culture | Build and maintain a shared prompt library (versioned, reviewed like code). Diagnose why a teammate got bad output. Review prompts in code reviews, not just the code itself. This is described as the single most leveraged thing a senior developer can do. |

### Example:

A comparison of how a Pure Engineer vs. a Half Engineer, Half AI Systems Architect handles the same scenario — a team adopting AI coding tools.

Scenario: Your 12-person microservices team just started using AI coding assistants. The team is shipping faster, but you're starting to see inconsistent code quality, weird security patterns, and some developers get great results while others get garbage.

✅ ENGINEER + AI SYSTEMS ARCHITECT Approach: 

```
┌─────────────────────────────────────────────────────────────┐
│ PHASE 1: AUDIT & MAP (Week 1)                               │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ • Run a 1-week audit: which tasks use AI?               │ │
│ │ • Categories: test writing, PR descriptions,             │ │
│ │   debugging, architecture brainstorming, code gen        │ │
│ │ • Identify: who gets good results, who struggles         │ │
│ │ • Map failure modes: naming convention errors,           │ │
│ │   security blind spots, hallucinated dependencies        │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                              │
│ PHASE 2: DESIGN THE WORKFLOW (Week 2)                        │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ • Define autonomy levels per task type:                  │ │
│ │                                                          │ │
│ │   Task Type          | AI Role        | Human Check      │ │
│ │   ──────────────────────────────────────────────────     │ │
│ │   Unit tests         | GENERATE       | Quick review     │ │
│ │   Boilerplate code   | GENERATE       | Review required  │ │
│ │   PR descriptions    | ASSIST (draft) | Edit & approve   │ │
│ │   Auth logic         | ASSIST (review)| Engineer writes  │ │
│ │   Security-sensitive | REVIEW only    | Human writes     │ │
│ │   Production deploy  | NOT USED       | Human only       │ │
│ │                                                          │ │
│ • Document this as the team's "AI Workflow Charter"        │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                              │
│ PHASE 3: BUILD INFRASTRUCTURE (Week 3)                       │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ • Set up a shared prompt library in the repo:            │ │
│ │   .prompts/                                              │ │
│ │   ├── review-pr.md           # PR review prompt          │ │
│ │   ├── write-tests.md         # Test generation           │ │
│ │   ├── debug-error.md         # Error diagnosis           │ │
│ │   └── security-review.md     # Security scan prompt      │ │
│ │                                                          │ │
│ │ • Create a prompt review checklist:                      │ │
│ │   □ Does the prompt include context (codebase, project)? │ │
│ │   □ Does it specify output format (code, explanation)?   │ │
│ │   □ Does it constrain behavior (no hallucinations)?      │ │
│ │   □ Does it include examples (few-shot)?                 │ │
│ │   □ Is there a version history?                          │ │
│ │                                                          │ │
│ │ • Configure model routing:                               │ │
│ │   - Quick drafts → cheap/fast model (flash tier)         │ │
│ │   - Complex architecture → powerful model (pro tier)     │ │
│ │   - Security review → conservative model                 │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                              │
│ PHASE 4: GOVERN & ITERATE (Ongoing)                          │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ • Weekly "AI output review" in team retro               │ │
│ │ • Track failure modes in a shared doc                   │ │
│ │ • Update prompt library as patterns emerge              │ │
│ │ • Mentoring: "Here's why your prompt gave bad output"   │ │
│ │ • New hire onboarding: includes AI workflow training     │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

  Result: Consistent quality across the team. Senior is a force
          multiplier, not a bottleneck. AI tool usage is
          deliberate and governed, not chaotic. New hires ramp
          faster. Failure modes are documented and avoided.
```

---
