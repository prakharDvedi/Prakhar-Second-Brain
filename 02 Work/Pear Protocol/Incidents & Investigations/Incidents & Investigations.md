# Incidents & Investigations

Navigation only — open the note for the full write-up.

## Execution / Trading

- [[Silent TP-SL Default on the Streaming Chat Path]] — streaming chat path silently reverted a user's TP/SL instruction to a 100% default. Resolved.
- [[Stale Recall Memory and an Unpersisted skipRisk Flag Resurfaced Old TP-SL Preferences]] — same symptom shape recurred via a different cause: unpersisted `skipRisk` + a stale multi-day recall note + prompt-only memory precedence. Root cause confirmed via live model replay; the "recent Gemma→GPT-OSS switch" hypothesis rejected as stated. **Diagnosed — fix not yet implemented.**
- [[Partial Fill Alert Blamed the Wrong Cause]] — partial-fill alerts asserted a hardcoded "$11 minimum" cause that a real incident's own numbers disproved. Resolved.
- [[HIP-3 Margin Backfill Passed Orders It Could Never Fund]] — No-Dex-Abstraction accounts passed margin validation for orders that failed at execution. Resolved.
- [[Reduce-Only Rejection Didn't Mean the Trade Failed]] — a "would increase position" rejection was always treated as failed, even when the user's target was already met. Resolved.
- [[TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade]] — an unvalidated condition crashed, then the model silently retried with a different trade type. Resolved.

## Agent / LLM

- [[Composer Fabricated the Current Time]] — no deterministic clock at the composer boundary, so it guessed a wrong time. Resolved.
- [[Finished PnL Card Discarded on an Unrelated Timeout]] — a completed tool result was discarded when a later step timed out. Resolved.
- [[HIP-3 Chart Silent Fallback to Text]] — chart requests for HIP-3 assets silently dropped to text with no error. Resolved.
- [[Composer Promised a Retry Capability That Didn't Exist]] — composer fabricated a background-retry capability after a timeout. Resolved.
- [[Chasing the Corrupted Thought Signature Failure]] — recurring Gemini/OpenRouter failure, observability fixed, root cause not confirmed. Diagnosed, not confirmed resolved.
- [[Ambiguous HIP-3 Symbol Crashed the Whole Reply, Repeatedly]] — a dual-listed ticker crashed replies at multiple call sites, recurring 13x in 3 days. Resolved.
- [[Internal-Leak Guard Blocked Legitimate AI-Industry Reporting]] — the leak-detection guard blocked legitimate AI-industry news. Resolved, ongoing tuning surface.
- [[Kalshi Odds Query Fell Into a Planner Routing Dead Zone With No Empty-Plan Fallback]] — a Kalshi "odds" query hit a node trigger-text conflict (research includes macro, excludes live price; marketScanner doesn't cover Kalshi either) and the planner emitted an all-null decision nothing downstream caught. Diagnosed, not confirmed resolved — no fix implemented.
- [[BNB-AVAX Pair Orientation Was Lost at the Agent Boundary]] — canonical pair-stat orientation was flattened into prose; the pair LLM later invented an inverse-sign transformation, while a separate frontend display path can also mix requested labels with canonical scores. Diagnosed; no production fix implemented.

## Telegram

- [[Linkwallet Poll Crash Paged On-Call for an Expected Telegram Event]] — an unguarded poll let an expected Telegram 403 escape as an unhandled rejection. Resolved.
- [[Telegram Outbound Delivery Ordering — Unresolved Investigation]] — possible outbound send-ordering bug vs. client rendering quirk. **Open — no root cause yet.**

## Infrastructure / Data

- [[Token-Unlock Amounts Shown as Confirmed Facts]] — projected token-unlock ceilings were shown to users as confirmed amounts. Resolved.
- [[One Service URL Fed Two Different Services, Nulling Basket Stats]] — one `STATS_BASE_URL` fed both `/categories` and `/stats/basket` on different services; repointing it 404'd basket-stats enrichment and nulled the columns behind a misleading `No baskets generated` alert. Resolved.
