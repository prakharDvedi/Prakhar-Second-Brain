---
type: knowledge
domain: Frontend & Product Surfaces
confidence: applied
last_reviewed: 2026-08-30
---

# Next.js & React Product Surfaces

## In Simple Terms

Frontend engineering is not only rendering screens; it is preserving backend contracts, user intent, loading/error states, authentication boundaries, and the behavior of asynchronous product actions in a surface people can actually use.

## What I Understand

I have built or maintained React/Vite and Next.js applications across Brinavv, MedBud, the outreach dashboard, portfolio, eVakeel, Authentica, ChatApp, and several learning projects. I have used App Router/API routes, server-side database reads, route protection, streaming route handlers, Redux/Zustand, and browser-side simulators for hardware-dependent flows.

The strongest product-surface lessons came from Pear's Telegram interface and Brinavv's biometric simulator: a capability must be exposed only where it works, and hardware/integration behavior needs a testable UI path.

## Why It Matters

Users experience the whole contract. A backend can be correct while buttons appear in the wrong chat, a command is advertised where it does nothing, an error is hidden behind generic UI, or a streamed result is handled like a normal return value.

## How I Learned This

- [[05 Career/Companies/Brinavv]]
- [[MediBuddy]]
- [[Personalized-Outreach-Dashboard]]
- [[portfolio]]
- [[eVakeel2.0]]
- [[Authentica]]
- [[ChatApp]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-07-23 — /aipear was advertised in DMs where it silently did nothing]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-07-30 – 07-31 — Group trade pitches showed execution buttons that only worked in DM]]

## Real Failure Modes I've Seen

- Pear advertised `/aipear` in private-chat menus even though the command handler returned before the fallback route ran.
- Group trade pitches displayed execution buttons that only worked in DMs.
- A streaming generation route needed a `ReadableStream`; server actions could not push tokens as they arrived.
- Brinavv added a browser biometric simulator because physical hardware was not a practical test dependency.
- MedBud documents stale router-cached 404s, persistent browser cookies, and incomplete deploy-hardening as real debugging considerations.

## Engineering Tradeoffs

- Server-rendered/database-backed pages reduce client data plumbing, while streaming routes improve perceived latency but require explicit client protocol handling.
- A simulator increases testability but is not a replacement for device integration testing.
- More explicit loading/error states make flows longer to build but prevent silent failure and user confusion.

## Patterns / Rules I Currently Believe

**Strong evidence:** Product surfaces must reflect the actual capability and context of the backend path; do not expose controls merely because an endpoint exists.

**Current belief:** Frontend verification should include the user journey and the backend contract, not only component rendering.

**Open question:** I have not yet measured frontend performance systematically or built a durable visual-regression workflow.

## Examples From My Work

- MedBud's app-wide English/Hindi selector preserves locale across refreshes while canonical evidence stays unchanged.
- Outreach uses a streaming route handler for generation and stores extracted prospect context to avoid repeated scraping.
- Brinavv's `BiometricSimulator.jsx` exercises attendance ingestion without physical hardware.

## Things I Still Don't Fully Understand

- Advanced React concurrency and performance profiling.
- Accessibility testing and resilient client behavior during reconnects or partial backend failure.

## Related

- [[APIs & Layered Backend Architecture]]
- [[Real-Time Communication & Delivery Ordering]]
- [[Authentication, Authorization & Trust Boundaries]]
- [[Containers, CI & Deployment]]

