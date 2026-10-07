# Decisions

Navigation only — open the note for context, tradeoffs, and evidence.

- [[Rebalance Weight Validation — Reject vs Rescale]] — rescale rebalance weights proportionally instead of rejecting on sum != 1.
- [[Reduce-Only Verification Source]] — use a BE-mediated live-position re-read (not direct exchange access) to verify a reduce-only downgrade.
- [[HIP-3 Margin Backfill — Gate vs Remove]] — gate the margin backfill on account abstraction mode instead of removing it.
- [[Trade-Account Alias Resolution]] — resolve an account alias only when it's unique across the full account list; fail closed on ambiguity.
- [[Elfa Webhook Trust Model]] — require HMAC on every production webhook; keep an unsigned path dev-only.
- [[Model Routing Under Provider Instability]] — distinguish rate-limiting from deprecation, then live-verify every model swap before merging.
- [[Directional Baskets — Skip Pair-Stat Enrichment]] — one-sided baskets bypass pair-stat enrichment entirely rather than computing meaningless spread stats.
- [[Internal-Leak Guard — Contextual Frame vs Unconditional Deny-List]] — gate model/provider name matches on disclosure framing instead of an unconditional deny-list.
- [[TWAP Close-All — Explicit Fallback Required]] — close-all stays market-only unless the user explicitly chooses a different execution mode.
- [[Confirm-Time Margin Gate]] — add a fail-closed margin check at confirm time, exempting reduce-only adjustments.
- [[Familiar-Asset Ranking — Final Plan]] — rank assets by 0.65 current holders + 0.35 90-day openers (distinct Pear traders), union the top 20 into the daily narrative universe, and pick pair vs basket per idea on evidence (§11).
- [[Coinglass HL Wallet Distribution — Retire, Don't Mute]] — retire the 2 dead HL wallet endpoints and say wallet data is unavailable, instead of muting a tool that fails on every call.
