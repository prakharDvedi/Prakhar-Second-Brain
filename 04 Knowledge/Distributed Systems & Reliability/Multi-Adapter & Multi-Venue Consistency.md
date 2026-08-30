---
type: knowledge
domain: Distributed Systems & Reliability
confidence: deep
last_reviewed: 2026-08-30
---

# Multi-Adapter & Multi-Venue Consistency

## In Simple Terms

When several adapters, venues, or services perform the same logical operation, correctness depends on shared assumptions: the same schema, the same resolved account, the same source of truth, and the same user-visible semantics.

## What I Understand

Pear's Lighter integration exposed Hyperliquid-only assumptions across pricing, margin mode, account binding, asset caches, KB wording, and Telegram UX. The agent/Telegram and MCP adapters also reached the same trade usecases with different validation behavior. The fixes were to resolve venue from the bound account rather than model text, pin the account per invocation, use the same schema across adapters, and make checks derive shared facts from the same source.

## Why It Matters

Adding a second venue or adapter multiplies hidden assumptions. A system can pass unit tests for each component while a read comes from venue A, a write goes to venue B, or one adapter accepts a shape another rejects.

## How I Learned This

- [[02 Work/Pear Protocol/Features Owned#Multi-venue execution: Lighter integration]]
- [[02 Work/Pear Protocol/Decisions/Trade-Account Alias Resolution]]
- [[02 Work/Pear Protocol/Incidents & Investigations/TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade]]
- [[02 Work/Pear Protocol/Engineering Lessons#Two checks meant to agree on the same fact need to derive it from the same source]]
- [[02 Work/Pear Protocol/Engineering Lessons#A narrow fix should be checked against sibling call sites of the same root cause]]

## Real Failure Modes I've Seen

- Price reads assumed Hyperliquid even when the resolved account was on Lighter.
- Explicit and omitted account selections could resolve to different memoized keys within one invocation.
- A stale price could survive a successful snapshot because the cache merged rather than replaced.
- Agent/Telegram condition validation lagged behind MCP validation and enabled a dangerous silent retry.
- A validator and stats tool independently defaulted omitted platform information and contradicted each other.

## Engineering Tradeoffs

- Binding to resolved account state is safer than trusting model-provided venue text, but it requires account resolution before reads.
- Rebuilding enums per invocation avoids stale capability assumptions, at some cost in work and code complexity.
- Shared schemas reduce drift, but they may need adapter-specific normalization before validation.
- Sequencing and explicit user choices can be slower than implicit fallbacks, but preserve intent.

## Patterns / Rules I Currently Believe

**Strong evidence:** Resolve once, bind the operation to that result, and make every adapter use the same schema and source of truth for shared facts.

**Current belief:** A second integration is a test of the original system's assumptions, not just another connector implementation.

**Open question:** The vault does not establish a complete long-term abstraction boundary for additional venues beyond Lighter.

## Examples From My Work

- Live check: 227/227 Lighter instruments reported `marginMode="any"`.
- KB retrieval remained strong after venue-aware documentation changes.
- Agent and MCP account-resolution suites passed 18/18 and 7/7 at review time.

## Things I Still Don't Fully Understand

- How to formalize venue capability matrices and contract tests for many connectors.
- How to manage cross-venue differences without leaking connector-specific rules everywhere.

## Related

- [[Context Propagation & Request-Scoped State]]
- [[Structured Outputs & Validation]]
- [[APIs & Layered Backend Architecture]]
- [[Trading Execution Safety]]
- [[PostgreSQL & Data Modeling]]

