---
name: peer-review
description:
  3-agent architecture audit — architecture, quality, security — using prime-level agents in
  parallel for deep independent review
version: 2.0.0
category: engineering
tags: audit, code-review, architecture, performance, quality, security, peer-review
---

# Peer Review — 3-Agent Architecture Audit

You are the **peer review lead**. You orchestrate a deep, thorough audit using 3 parallel
prime-level agents, each reviewing the same subsystem from a different perspective. The agents
work independently with no knowledge of each other's findings. You synthesize their reports into
a single actionable document.

This is expensive (prime agents use the same model as you) but finds issues that cheaper agents
cannot — lifecycle invariants, concurrency gaps, cross-module contracts, and implicit assumptions
that only surface under deep reasoning.

## When to Use

- After significant system changes (new persistence layers, entry points, concurrency model)
- Before shipping a major feature that touches core infrastructure
- After an incident — audit the affected subsystem for related weaknesses
- Periodic health checks on critical subsystems

**Not for:** routine code reviews, single-file changes, or tasks that general/utility workers handle.

## How It Works

### Step 1 — Scope the Audit

Before dispatching agents, identify:

1. **The subsystem** — what area is being reviewed (e.g., "message lifecycle", "auth middleware")
2. **The architecture doc** — if one exists, agents read it first
3. **The key files** — 5-10 files that form the critical path (keep it focused)
4. **The desired guarantees** — numbered, specific, verifiable properties the system MUST deliver
5. **Existing tests** — point agents to test files so they can assess coverage

### Step 2 — Dispatch Three Prime Agents in Parallel

Each agent gets the system overview, the guarantees, and the file list. Each reviews from a
different perspective. **Do not hint at known bugs** — the value is independent discovery.

## The Three Agents

### 1. Architecture & Design

Perspective: senior architect who just inherited this system.

Evaluates whether the implementation delivers its stated guarantees by design or by accident.
Looks for: single source of truth violations, implicit assumptions, missing invariants,
structural gaps where guarantees rely on developer discipline rather than enforcement, module
boundary violations, coupling, dependency direction, and scalability.

Prompt template:

```
You are a senior architect reviewing the [SUBSYSTEM] of this codebase.

## System Overview
[2-3 sentences describing what the subsystem does, its entry points, and persistence model]

## Desired Guarantees
The system MUST provide these guarantees:
1. [Guarantee 1 — e.g., "Context fidelity — the LLM must never see stale data"]
2. [Guarantee 2 — e.g., "Single source of truth — one authoritative state at any time"]
3. [Guarantee 3]
4. [Guarantee 4]
5. [Guarantee 5]

## Files To Read
Start with the architecture doc, then trace the implementation:
- [file1] — [purpose]
- [file2] — [purpose]
- ...

## Your Task
Review whether the implementation actually delivers the guarantees above. For each guarantee:
- Does the architecture support it by design, or does it rely on implicit assumptions?
- Are there code paths where the guarantee could be violated?
- Are there tests that enforce the guarantee?

Think like an architect who just inherited this system. You have no insider knowledge
of recent bugs or fixes. Report what you find.
```

### 2. Quality & Resilience

Perspective: senior QA engineer trying to break the system.

Evaluates crash resilience, timing sensitivity, resource leaks, dead code traps, concurrency
correctness, performance scaling, error handling completeness, and defensive coding patterns.
Focuses on worst-case scenarios.

Prompt template:

```
You are a senior quality engineer reviewing the [SUBSYSTEM] for resilience and performance.

## System Overview
[2-3 sentences — same as Agent 1]

## Desired Properties
The system MUST be:
1. [Property 1 — e.g., "Resilient to crashes — recover to consistent state on restart"]
2. [Property 2 — e.g., "Resilient to timing — insertion order preserved regardless of clock"]
3. [Property 3 — e.g., "No resource leaks — deleted entities leave no orphaned state"]
4. [Property 4 — e.g., "No dead code traps — unused exports removed"]
5. [Property 5 — e.g., "Correct under concurrency — exclusive access enforced"]
6. [Property 6 — e.g., "Performance proportional to data size — bounded, not O(n)"]

## Files To Read
- [file1] — [purpose]
- [file2] — [purpose]
- ...

## Existing Tests
- [test-file1]
- [test-file2]

## Your Task
For each property, read the implementation and assess:
- Does the code actually deliver this property?
- What happens in the worst case? (crash at the worst moment, rapid-fire operations, large datasets)
- Are the tests sufficient to catch regressions?
- What would you flag in a code review?

Think like a QA engineer trying to break the system. You have no insider knowledge of recent bugs.
Report what you find.
```

### 3. Security & Correctness

Perspective: security engineer auditing trust boundaries and data integrity.

Evaluates input validation at system boundaries, authorization enforcement, data integrity
across storage layers, correctness of error handling paths, and whether error recovery leaves
the system in a consistent state.

Prompt template:

```
You are a security engineer reviewing the [SUBSYSTEM] for correctness and safety.

## System Overview
[2-3 sentences — same as Agent 1]

## Trust Boundaries
Identify and verify every point where:
1. External input enters the system (API endpoints, file reads, user messages)
2. Data crosses storage boundaries (memory -> DB, DB -> cache, cache -> LLM)
3. Permissions are checked or assumed
4. Error paths could leak state or skip cleanup

## Files To Read
- [file1] — [purpose]
- [file2] — [purpose]
- ...

## Your Task
For each trust boundary:
- Is input validated before use?
- Could malformed input cause inconsistent state?
- Are error paths safe? (no partial commits, no leaked resources)
- Are authorization checks present where needed, or assumed from caller context?
- Could a malicious or buggy caller bypass safety checks?

Think like a security auditor who assumes nothing about caller behavior.
Report what you find.
```

## Priority Levels

Rank every finding using exactly these levels:

| Priority        | Meaning                                                                     |
| --------------- | --------------------------------------------------------------------------- |
| **P1 Critical** | Immediate risk of crash, data loss, security breach, or severe resource leak |
| **P2 High**     | Significant reliability, performance, or design issue likely to cause production problems |
| **P3 Medium**   | Noticeable problem that degrades quality but is not immediately dangerous |
| **P4 Low**      | Minor issue worth fixing when nearby code is touched |
| **P5 Info**     | Observation or suggestion, not a defect |

## Step 3 — Synthesize Findings

When all three agents return:

1. **Deduplicate** — multiple reviewers often flag the same issue from different angles. Keep the most detailed version and note which perspectives identified it.
2. **Rank** — sort all findings P1 first, P5 last.
3. **Confirm strengths** — what multiple reviewers agreed is solid.
4. **Identify test gaps** — what should be tested but isn't.
5. **Produce a fix checklist** — ordered by priority with effort estimates.
6. **Note discoverability** — for each finding, state whether cheaper agents (general/utility) could have found it. This helps calibrate future audit decisions.

## Report Format

Write the final report as markdown:

```markdown
# Peer Review: [Subsystem Name] — YYYY-MM-DD

## Summary

- **Scope**: (what was audited)
- **Agents**: 3 prime (architecture, quality, security)
- **Duration**: (wall time)
- **Findings**: X total (P1: N, P2: N, P3: N, P4: N, P5: N)
- **Key Risks**: (1-3 sentence summary of the most critical findings)

## Overall Assessment

[1-2 sentences: is the system sound, or are there structural concerns?]

## Findings by Priority

| # | Priority | Finding | Agents | Discoverable by cheaper agents? |
|---|----------|---------|--------|-------------------------------|
| 1 | P1 | [description] | Arch + Quality | No — requires lifecycle reasoning |
| 2 | P2 | [description] | Quality | Maybe |
| ... | ... | ... | ... | ... |

### P1 — Critical

(detailed findings with evidence, impact, and fix — or "None" if clean)

### P2 — High

...

### P3-P5

...

## Strengths Confirmed

- [Strength 1 — confirmed by multiple reviewers]
- [Strength 2]

## Test Gaps

| Gap | Severity |
|-----|----------|
| [Missing test] | Medium |
| [Missing test] | Low |

## Recommended Actions

| Priority | Action | Effort |
|----------|--------|--------|
| P1 | [Fix description] | [time estimate] |
| P2 | [Fix description] | [time estimate] |
| ... | ... | ... |
```

## Cost Expectations

| Metric | Typical Range |
|--------|--------------|
| Wall time | 3-5 minutes (agents run in parallel) |
| API calls | 8-15 per agent |
| Total tokens | 400k-800k across all agents |
| Cost | $5-15 depending on model and file count |
| Maestro context growth | ~15-25k tokens (only summaries return) |

The agents' internal context (100k+ each) is discarded after completion. Only the final summary
returns to maestro, keeping the conversation context small and compactable.

## Best Practices

1. **Be specific about guarantees** — "Messages must be returned in insertion order regardless of timestamp precision" beats "data must be consistent."
2. **Architecture doc first** — agents that read design intent before implementation produce better findings.
3. **Include test files** — agents that see existing coverage identify gaps more accurately.
4. **Don't hint at known bugs** — independent discovery is the point. Hinting produces confirmation, not discovery.
5. **5-10 files max** — agents that read 20+ files spend context on reading, not reasoning.
6. **Run after significant changes** — the cost/benefit is best for subsystem-level audits.

## Must-Detect Issues

Regardless of agent perspective, the audit as a whole must surface these if present:

- Crash risks (unhandled exceptions, null dereferences, panic paths)
- Data integrity violations (stale caches, missing invalidation, orphaned state)
- Memory leaks (unbounded maps, event listener buildup, unclosed resources)
- Concurrency bugs (race conditions, missing locks, shared mutable state)
- Security exposures (injection, auth bypass, secrets in code, missing validation)

## Constraints

- Do not fabricate findings. If the code is clean, say so.
- Do not pad the report with low-value observations to look thorough.
- Every finding must have concrete evidence (file path, line number, or pattern) and a concrete fix.
- The report is the deliverable. Make it scannable, actionable, and honest.
