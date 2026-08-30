---
type: knowledge
domain: Distributed Systems & Reliability
confidence: deep
last_reviewed: 2026-08-30
---

# Real-Time Communication & Delivery Ordering

## In Simple Terms

Real-time systems connect long-lived clients to changing backend state. Delivery is not only about sending a message; it is also about connection state, ordering, retries, correlation, and what happens when a client disappears.

## What I Understand

ChatApp and Redis-chatApp use Socket.IO/WebSockets, Redis for shared presence and rate limits, MongoDB for durable messages, and an in-memory map for socket routing. eVakeel2.0 has a Fastify WebSocket chat endpoint. Pear's Telegram surface adds a production case where outbound text and media may appear out of order, and the first step was to instrument `send_start`, `send_complete`, and `send_error` per turn before claiming a backend bug.

## Why It Matters

An apparently simple ordering issue may be caused by concurrent jobs, queue order, API completion order, or client rendering. Without correlation and message IDs, debugging becomes guesswork.

## How I Learned This

- [[ChatApp]]
- [[Redis-chatApp]]
- [[eVakeel2.0]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Telegram Outbound Delivery Ordering — Unresolved Investigation]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Linkwallet Poll Crash Paged On-Call for an Expected Telegram Event]]

## Real Failure Modes I've Seen

- Telegram media from an older turn appeared after newer text; root cause remains open because no pre-instrumentation evidence existed.
- A timer callback's rejected Telegram request escaped the normal middleware error boundary and paged on-call.
- In-memory socket state does not support horizontal scaling without a shared adapter or session-affinity strategy.
- Redis failure can remove presence while core chat remains usable if degradation is intentional.

## Engineering Tradeoffs

- In-memory routing is fast but process-local.
- Redis pub/sub and adapters improve multi-instance coordination but add dependency and operational complexity.
- Instrumentation should correlate turns without logging message content unnecessarily.
- Graceful degradation preserves core messaging but may provide weaker presence or rate-limit behavior.

## Patterns / Rules I Currently Believe

**Strong evidence:** Establish event correlation and completion order before deciding whether a user-visible ordering problem is backend or client-side.

**Current belief:** Every asynchronous callback needs an explicit error boundary; framework-level catches rarely cover timers, workers, or external callbacks automatically.

**Open question:** The Pear Telegram ordering investigation has not yet established the actual ordering failure.

## Examples From My Work

- ChatApp rate limits messages with atomic Redis counters and keeps durable messages in MongoDB.
- Pear uses `AsyncLocalStorage` to associate terminal sends with the originating turn.
- eVakeel separates frontend HTTP calls, backend WebSocket handling, and the Python retrieval service.

## Things I Still Don't Fully Understand

- Delivery guarantees and ordering across multiple workers and queues.
- Socket.IO adapter behavior under reconnects and network partitions.

## Related

- [[Redis & Caching]]
- [[Context Propagation & Request-Scoped State]]
- [[Retry Semantics & Failure Recovery]]
- [[LLM Observability & Evaluation]]

