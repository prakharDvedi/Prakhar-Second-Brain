# Current Focus

## Work

- Pear Protocol: two clusters of work open as of the latest daily logs — [[28th Aug 2026]] (internal-leak guard tuning, self-diagnosing signal failures, Telegram delivery-ordering observability — PRs #639/#640/#641, all open) and [[30th Aug 2026]] (rebalance weight validation — PR #642, open).
- See [[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]] and [[My Work at Pear Protocol#2026-08-30 — Rebalance weights were rejected instead of rescaled (open, current branch)]] for what these are.

## Projects

- **Medbud (Healthcare Check)** — M0–M7 done, M8 partially done per [[Status|Medbud/Status]]. Explicitly open: responsive/mobile-viewport pass, case deletion, error-boundary/empty-state design, and the actual Vercel deploy (never deployed anywhere beyond local `docker compose`).

## Learning

- *(No explicit current learning area found in recent notes — leaving sparse rather than guessing.)*

## Personal

- *(No recent note explicitly supports a current personal focus item — leaving sparse.)*

## Open Loops

- Telegram outbound delivery ordering: instrumentation shipped, root cause not yet established — see [[Telegram Outbound Delivery Ordering — Unresolved Investigation]].
- Internal-leak guard: same-day follow-up already found and fixed one over-narrowing; whether further gaps exist is inherently open-ended — see [[Internal-Leak Guard Blocked Legitimate AI-Industry Reporting]].
- PRs #639, #640, #641, #642 — all open as of 2026-08-30; merge status needs checking against live PR state.
- Medbud: no `pg` CLI/dependency in the container (migrations use a throwaway-script workaround) — flagged in [[Status|Medbud/Status]] as "a real gap, not a stylistic choice," not yet fixed.
- Medbud: junk "Untitled case" rows from before the lazy-creation fix still sit in the DB with no delete tooling yet.

## Things I'm Intentionally Not Doing

*(Only explicit statements found — scoped to Medbud, not a general life claim.)*

- Medbud: not building RAG/embeddings/pgvector for medicine-price matching — structured identity + structured lookup + deterministic comparison only ([[Plans|Medbud/Plans]] §L).
- Medbud: not scraping online pharmacies (Tata 1mg, Apollo, PharmEasy, Netmeds, etc.) for price data.
- Medbud: not localizing the whole app shell or adding automatic language detection before the Hindi/Hinglish policy-Q&A workflow is proven reliable ([[Plans|Medbud/Plans]] §M).
