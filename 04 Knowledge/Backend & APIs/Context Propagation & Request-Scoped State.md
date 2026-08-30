---
type: knowledge
domain: Backend & APIs
confidence: deep
last_reviewed: 2026-08-30
---

# Context Propagation & Request-Scoped State

## In Simple Terms

Context is data that must stay attached to one request or one user action as it crosses layers. If it is dropped, defaulted, or accidentally shared with another request, the system may do the wrong thing while every individual function still appears to work.

## What I Understand

Pear gave me the clearest examples. The streaming DAG path passed empty `userInput` and `history` into `buildDagContext`, while the synchronous path passed the real values. The fix was to mirror the context construction and test both entry points. Later, Lighter integration required pinning the resolved trade account for one agent invocation so an explicit read and an omitted write could not use different accounts.

I have also used request-scoped runtime context for a live Basket Builder draft and `AsyncLocalStorage` to correlate Telegram sends to overlapping turns without putting message content in the observer.

## Why It Matters

Context bugs are especially dangerous because the code often returns a valid-looking response. A missing history value can change a financial instruction; a stale account binding can read one venue and write to another.

## How I Learned This

- [[02 Work/Pear Protocol/Incidents & Investigations/Silent TP-SL Default on the Streaming Chat Path]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue]]
- [[02 Work/Pear Protocol/Features Owned#Basket generation]]
- [[02 Work/Pear Protocol/Features Owned#Observability / agent infrastructure]]
- [[02 Work/Pear Protocol/Engineering Lessons#Two entry points meant to behave identically can silently diverge]]

## Real Failure Modes I've Seen

- Streaming and synchronous paths silently diverged because one constructed context with `"", []`.
- An account resolver memoized explicit and omitted account calls under different keys, making a read/write pair vulnerable to stale default reuse.
- A draft-aware agent needs invocation-scoped state; persisting the live draft would have changed the intended lifecycle.
- Overlapping Telegram turns needed isolation; a shared mutable correlation value could have made one turn inherit another turn's identifiers.

## Engineering Tradeoffs

- Passing context explicitly is easier to inspect, while ambient context can reduce plumbing but needs strong isolation tests.
- Pinning state for one invocation improves consistency but means later account changes do not affect an in-flight operation.
- Request-scoped context is useful for observability and draft reasoning, but it should not become accidental persistence.

## Patterns / Rules I Currently Believe

**Strong evidence:** Test what each entry point hands downstream, not merely whether both return a response. Bind read and write operations to the same resolved request context.

**Current belief:** Treat context construction as a contract with its own tests, especially when sync, streaming, worker, and adapter paths coexist.

**Open question:** I have not yet established a general policy for context cancellation and lifetime across long-running jobs.

## Examples From My Work

- `DagContext` now receives real input/history on `runAsync`.
- Lighter account selection is pinned within `TradeExecutionAgent.invoke()`.
- Telegram delivery observations use `AsyncLocalStorage` for per-turn correlation.

## Things I Still Don't Fully Understand

- How to propagate context safely across queues and process boundaries.
- Which context fields should be immutable after request start.

## Related

- [[Multi-Adapter & Multi-Venue Consistency]]
- [[Real-Time Communication & Delivery Ordering]]
- [[Retry Semantics & Failure Recovery]]
- [[APIs & Layered Backend Architecture]]

