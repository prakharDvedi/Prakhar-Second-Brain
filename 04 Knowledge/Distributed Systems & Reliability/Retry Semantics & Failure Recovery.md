---
type: knowledge
domain: Distributed Systems & Reliability
confidence: deep
last_reviewed: 2026-08-30
---

# Retry Semantics & Failure Recovery

## In Simple Terms

A retry is not automatically safe recovery. It may repeat a side effect, observe a different world, change the request, amplify load, or hide the original failure. The first question is what exactly is being retried.

## What I Understand

Pear work gave me repeated examples: a bounded planner recovery for a schema-only failure, a composer promise of a retry loop that did not exist, a signal retry that re-measured a condition instead of preserving its original evidence, a rebalance validator that caused 21 rejected attempts, and a timeout path that could recover a completed read-only card but must still rethrow around money-moving tools.

I also learned to distinguish a stale order rejection from a failed user intent: a fresh read may show that the requested end-state was already achieved, but the verification gate must be downgrade-only.

## Why It Matters

Retries can turn correctness bugs into silent data or trade changes. They can also create storms that consume provider capacity and make the original issue harder to diagnose.

## How I Learned This

- [[02 Work/Pear Protocol/Incidents & Investigations/Composer Promised a Retry Capability That Didn't Exist]]
- [[02 Work/Pear Protocol/Incidents & Investigations/TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade]]
- [[02 Work/Pear Protocol/Engineering Lessons#Fresh measurements can change what a retry is actually retrying]]
- [[02 Work/Pear Protocol/Engineering Lessons#A validation rule can be arithmetically correct and still be the wrong check]]
- [[02 Work/Pear Protocol/Engineering Lessons#A downgrade-only gate is safer than a symmetric recheck]]
- [[02 Work/Pear Protocol/Decisions/Rebalance Weight Validation — Reject vs Rescale]]

## Real Failure Modes I've Seen

- A malformed structured request was retried as a different request type and succeeded, silently changing user intent.
- Rebalance weights caused 21 rejected attempts across four traces, including 10 self-corrections in one trace.
- A signal retry independently re-measured a crossing that had already reversed, so it was no longer verifying the original event.
- A timeout after a completed read-only tool step discarded a useful result; the safe recovery was narrowly gated and excluded writes.
- The composer promised background retries without any scheduler, loop, or notification hook.

## Engineering Tradeoffs

- Strict rejection protects invariants but may burn retries when the input has harmless arithmetic drift.
- Rescaling preserves relative intent but must be bounded by upstream positivity/max constraints and an explicit proof that division is safe.
- Recovery improves UX only when it preserves intent and has a known capability behind it.
- Fresh verification costs another read and can still be imperfect; asymmetry bounds the risk.

## Patterns / Rules I Currently Believe

**Strong evidence:** Define retry identity, side-effect safety, evidence freshness, and stop conditions before adding a retry. Never imply a background capability that is not implemented.

**Current belief:** A retry should either replay the same logical event or explicitly be treated as a new attempt with new evidence.

**Open question:** I have not yet implemented or operated a general idempotency-key strategy across distributed writes.

## Examples From My Work

- Planner recovery runs once and only for `planner_no_output`.
- Rebalance weights are rescaled outside tolerance while clean sums remain unchanged.
- Reduce-only failures can move only from failed to benign after positive fresh-state confirmation.

## Things I Still Don't Fully Understand

- Queue retry policies, dead-letter handling, and exactly-once illusions at scale.
- How to choose backoff and budgets from measured provider and user behavior.

## Related

- [[Structured Outputs & Validation]]
- [[LLM Observability & Evaluation]]
- [[Context Propagation & Request-Scoped State]]
- [[Trading Execution Safety]]
- [[Redis & Caching]]

