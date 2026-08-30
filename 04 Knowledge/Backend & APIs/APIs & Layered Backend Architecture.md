---
type: knowledge
domain: Backend & APIs
confidence: deep
last_reviewed: 2026-08-30
---

# APIs & Layered Backend Architecture

## In Simple Terms

An API is a contract between callers and the system behind it. Good boundaries make it clear where input is validated, where business rules live, and where persistence or transport details stop leaking across the application.

## What I Understand

The Finance API uses a route → controller → service → repository → database split: controllers shape HTTP concerns, services enforce business rules, and repositories own Prisma queries. eVakeel2.0 uses a Fastify backend plus a separate Python embedding service. Pear has multiple adapters—agent/Telegram and MCP—feeding shared execution usecases, which makes contract parity important.

I have also built Next.js route handlers and API routes in portfolio, MedBud, outreach, and the Finance project. The level of operational depth is strongest in Pear, while the layered API design is clearest in the Finance project.

## Why It Matters

Most production bugs occur at seams: a caller passes the wrong shape, a route never wires an existing capability, or two adapters make different assumptions about the same usecase. Explicit boundaries make those differences testable.

## How I Learned This

- [[03 Projects/Real Projects/Finance-Data-Processing-and-Access-Control-Backend]]
- [[eVakeel2.0]]
- [[MediBuddy]]
- [[Personalized-Outreach-Dashboard]]
- [[02 Work/Pear Protocol/Engineering Lessons#A wiring gap can look identical to broken logic]]
- [[02 Work/Pear Protocol/Engineering Lessons#An unvalidated shortcut on one of two parallel adapters reopens a hole the other already closed]]

## Real Failure Modes I've Seen

- An existing `allowLighter` capability was not wired into the surfaces that needed it, so the feature appeared broken even though the lower-level logic existed.
- The agent/Telegram adapter accepted a JSON-string shortcut that bypassed the validation already present in the MCP adapter.
- A stale API contract or missing execution field caused response handling to crash; the fix was to make the boundary tolerate the actual SDK response shape.
- eVakeel documents partial routes and a Docker Compose layout that no longer fully matches the repository, showing that an API surface can exist without being complete end to end.

## Engineering Tradeoffs

- More layers add files and indirection, but reduce coupling and make testing responsibilities clearer.
- Sharing a usecase across adapters reduces duplicate business logic, but requires explicit contract-parity tests.
- A thin route handler is easier to reason about, but only if service-level validation is not silently omitted.

## Patterns / Rules I Currently Believe

**Strong evidence:** Validate and normalize at the boundary closest to the untrusted input, then keep business rules in a shared layer.

**Current belief:** When a feature “does not work,” trace the full route from transport input to usecase before redesigning the core logic.

**Open question:** I have not yet documented a consistent versioning or backward-compatibility strategy for these APIs.

## Examples From My Work

- Finance registration always creates a viewer; admin role changes are separate and protected.
- MedBud keeps a single Next.js deployable app instead of introducing a backend service without a real need.
- Pear hardened two structurally different DAG entry points instead of adding a blanket catch-all.

## Things I Still Don't Fully Understand

- How to evolve public API contracts across independently deployed clients.
- When a shared usecase should be split because adapters genuinely need different semantics.

## Related

- [[Structured Outputs & Validation]]
- [[Multi-Adapter & Multi-Venue Consistency]]
- [[Authentication, Authorization & Trust Boundaries]]
- [[PostgreSQL & Data Modeling]]

