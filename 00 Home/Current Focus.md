# Current Focus

## Work

- Pear Protocol: PRs #639/#640/#641 (internal-leak guard tuning, self-diagnosing signal failures, Telegram delivery-ordering observability — [[28th Aug 2026]]) and #642 (rebalance weight validation — [[30th Aug 2026]]) all merged 2026-08-31 via the `development → main` promotion PR #664. These clusters are closed as of this check.
- See [[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]] and [[My Work at Pear Protocol#2026-08-30 — Rebalance weights were rejected instead of rescaled (open, current branch)]] for what these are (headers still say "open" — not yet updated in that note, but confirmed merged per above).
- [[Agent Pear Revamp]] — team is planning to redesign AP. Codebase-grounded comparison against Oracle/DEMI done: verdict is don't adopt wholesale, AP already covers most of it via plan-token/ticket/reconciliation protocols. One real gap found (autonomous/unattended execution has no bound-policy primitive yet). Waiting to bring verdict back to senior and confirm revamp scope before prototyping anything.
- Separate AP work item: live leaderboard + campaign context (Epic #643 → Story #644 / Story #645). The original 8 stacked scaffold PRs (#646–#653) were closed same-day as unnecessary/premature. Leaderboard axis reopened leaner as #659 → #660 (open). Campaign axis (Story #645) is back to unstarted — no replacement PR yet. A separate #665 (chart entry/TP/SL annotations) also opened same day; relation to this epic unconfirmed. See [[Leaderboard & Campaign Context Ingestion]] and [[31st Aug 2026]].

## Projects

- **Medbud (Healthcare Check)** — M0–M7 done, M8 partially done per [[Status|Medbud/Status]]. Explicitly open: responsive/mobile-viewport pass, case deletion, error-boundary/empty-state design, and the actual Vercel deploy (never deployed anywhere beyond local `docker compose`).

## Learning

- *(No explicit current learning area found in recent notes — leaving sparse rather than guessing.)*

## Personal

- *(No recent note explicitly supports a current personal focus item — leaving sparse.)*

## Open Loops

- Telegram outbound delivery ordering: instrumentation shipped, root cause not yet established — see [[Telegram Outbound Delivery Ordering — Unresolved Investigation]].
- Internal-leak guard: same-day follow-up already found and fixed one over-narrowing; whether further gaps exist is inherently open-ended — see [[Internal-Leak Guard Blocked Legitimate AI-Industry Reporting]].
- Agent Pear Revamp: verdict reached (bounded execution policy for automation is the one worth prototyping), not yet confirmed with senior whether autonomous execution is actually in revamp scope — see [[Agent Pear Revamp]].
- Leaderboard + campaign context: original 8-PR scaffold stack closed as unnecessary; leaderboard axis now in progress under #659 → #660, campaign axis unstarted again (no PR open). VPS ownership question resolved as "not in either repo" (grep-clean); three MVP defaults (source lists, campaign-asset linkage, GH scope) still await stakeholder sign-off — unclear if they still apply once the campaign axis restarts. See [[Leaderboard & Campaign Context Ingestion]] and [[31st Aug 2026]].
- Medbud: no `pg` CLI/dependency in the container (migrations use a throwaway-script workaround) — flagged in [[Status|Medbud/Status]] as "a real gap, not a stylistic choice," not yet fixed.
- Medbud: junk "Untitled case" rows from before the lazy-creation fix still sit in the DB with no delete tooling yet.

## Things I'm Intentionally Not Doing

*(Only explicit statements found — scoped to Medbud, not a general life claim.)*

- Medbud: not building RAG/embeddings/pgvector for medicine-price matching — structured identity + structured lookup + deterministic comparison only ([[Plans|Medbud/Plans]] §L).
- Medbud: not scraping online pharmacies (Tata 1mg, Apollo, PharmEasy, Netmeds, etc.) for price data.
- Medbud: not localizing the whole app shell or adding automatic language detection before the Hindi/Hinglish policy-Q&A workflow is proven reliable ([[Plans|Medbud/Plans]] §M).
