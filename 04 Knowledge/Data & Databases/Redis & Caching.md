---
type: knowledge
domain: Data & Databases
confidence: deep
last_reviewed: 2026-08-30
---

# Redis & Caching

## In Simple Terms

Redis is useful for fast, shared, short-lived state—but only when the system is clear about what is authoritative, how stale data is tolerated, and what must be atomic.

## What I Understand

In ChatApp and Redis-chatApp, Redis holds online presence and rate-limit counters while MongoDB holds persistent messages and history; an in-memory map handles connection-specific socket lookup. The rate limiter relies on atomic `INCR` plus TTL, and the app deliberately degrades when Redis is unavailable.

In eVakeel2.0, Redis caches retrieval results. In Pear, Redis supports atomic callback consumption, command counters, and other shared state. Lighter work also exposed a related cache correctness issue: a successful price snapshot, including an empty one, must replace the prior snapshot rather than merge stale symbols into it.

## Why It Matters

A cache or ephemeral store is part of system behavior even when it is not the source of truth. Stale values can cause wrong prices, duplicate actions, incorrect presence, or unsafe decisions.

## How I Learned This

- [[ChatApp]]
- [[Redis-chatApp]]
- [[eVakeel2.0]]
- [[02 Work/Pear Protocol/Features Owned#Telegram / product surface]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue]]

## Real Failure Modes I've Seen

- A merged price snapshot could retain an omitted symbol's stale price while advancing its freshness timestamp.
- Redis failure should not break core chat delivery; it should remove presence/rate-limit capabilities and preserve messaging where safe.
- A callback that is supposed to be single-use needs atomic consume (`getDel`), not a separate read and delete.
- A cache hit can conceal an underlying retrieval or service failure; eVakeel's cache is explicitly part of the retrieval path.

## Engineering Tradeoffs

- Redis provides speed and shared state, but adds an operational dependency.
- In-memory state is faster and simpler for local connections, but does not survive restarts or coordinate across instances.
- Graceful degradation preserves core UX but can weaken non-core guarantees such as presence or rate limiting.
- Replace semantics avoid stale retention, while merge semantics can reduce payload churn; correctness decides which is valid.

## Patterns / Rules I Currently Believe

**Strong evidence:** Define the source of truth and the invalidation/expiry semantics before calling something a cache. Use atomic operations when correctness depends on single consumption or shared counters.

**Current belief:** “Freshness” metadata is only meaningful if the snapshot semantics are explicit; an empty successful response can be a valid replacement.

**Open question:** I have not yet operated Redis in a highly available or clustered production environment.

## Examples From My Work

- Chat rate limiting: atomic counter plus 60-second TTL.
- Pear suggested-action callbacks: atomic single-consume behavior.
- Lighter price cache: replace-not-merge semantics.
- eVakeel retrieval: Redis checked before invoking the embedding service.

## Things I Still Don't Fully Understand

- Redis replication, failover, cluster behavior, and consistency under partitions.
- How to choose TTLs from user-visible freshness requirements.

## Related

- [[PostgreSQL & Data Modeling]]
- [[Retry Semantics & Failure Recovery]]
- [[Real-Time Communication & Delivery Ordering]]
- [[RAG & Retrieval]]

