---
type: knowledge
domain: Security & Trust
confidence: deep
last_reviewed: 2026-08-30
---

# Authentication, Authorization & Trust Boundaries

## In Simple Terms

Authentication answers who is calling; authorization answers what they may do; a trust boundary answers which inputs, services, and persisted facts are allowed to influence a sensitive action.

## What I Understand

I have implemented or worked with JWT and bcrypt authentication in Finance, Brinavv, ChatApp, and eVakeel; RBAC and protected admin routes in Finance; Supabase Auth/RLS/private Storage in MedBud; and HMAC verification for Pear signal webhooks. Pear also made the broader trust problem explicit: client-supplied drafts, model output, provider data, and execution state cannot all be treated as equally trustworthy.

## Why It Matters

The most damaging security bugs often come from a fallback that seems convenient: accepting a role during registration, trusting an unsigned webhook, selecting an account from an ambiguous alias, or treating model/provider text as authoritative.

## How I Learned This

- [[03 Projects/Real Projects/Finance-Data-Processing-and-Access-Control-Backend]]
- [[05 Career/Companies/Brinavv]]
- [[ChatApp]]
- [[eVakeel2.0]]
- [[MediBuddy]]
- [[02 Work/Pear Protocol/Decisions/Elfa Webhook Trust Model]]
- [[02 Work/Pear Protocol/Decisions/Trade-Account Alias Resolution]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Internal-Leak Guard Blocked Legitimate AI-Industry Reporting]]

## Real Failure Modes I've Seen

- Pear's unsigned webhook fallback left a production trust bypass and replay risk; production now requires HMAC while a dev-only escape hatch remains explicit.
- A naive account-alias fallback could select the wrong account on a collision; duplicate aliases fail closed.
- Finance registration accepting a role would allow self-escalation, so registration always creates a viewer.
- A prompt safety guard blocked legitimate public AI-industry reporting because it treated overlapping model names as unconditional secrets.
- ChatApp and eVakeel document security controls and limitations, but their project notes do not establish a full production security audit.

## Engineering Tradeoffs

- Fail-closed behavior blocks some legitimate traffic during uncertainty, but is appropriate for sensitive writes and webhook trust.
- Development escape hatches can preserve iteration speed only when configuration and deployment boundaries make them impossible in production.
- Contextual detection reduces false positives but needs adversarial and benign tests to avoid false negatives.
- Soft deletion and audit preservation improve accountability but require careful authorization on every read and mutation path.

## Patterns / Rules I Currently Believe

**Strong evidence:** Never resolve ambiguity by guessing on a sensitive action. Keep production trust requirements stronger than local-development conveniences.

**Current belief:** Treat model output, client state, webhook payloads, and provider responses as untrusted until validated against an independent contract or authenticated source.

**Open question:** I have not performed a systematic threat model or penetration test across these systems.

## Examples From My Work

- Elfa webhook HMAC hardening: 80/80 focused tests and a persisted-row re-arm plan.
- Finance RBAC separates viewer, analyst, and admin capabilities.
- MedBud uses private per-user storage and RLS-scoped database access.

## Things I Still Don't Fully Understand

- Token rotation, session revocation, and refresh-token threat models in depth.
- Distributed authorization consistency across multiple services.

## Related

- [[Prompt Injection & Safety Boundaries]]
- [[Structured Outputs & Validation]]
- [[Wallets & Account Binding]]
- [[Trading Execution Safety]]
- [[APIs & Layered Backend Architecture]]

