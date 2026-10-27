# Elfa Endpoints and Plan Gating

Checked 22 Oct 2026 against the code in `apps/agent` and `apps/twitter`, a live `/v2/key-status` probe, and docs.elfa.ai/get-started/pricing.

## Plans (docs)
- PAYG: 60 RPM, prepaid credits, 30-day v3 history.
- Grow: 120,000 credits/month ($870; pre-Oct-2026 subscribers keep 40k at $290), 120 RPM, 60-day v3 history. Everything in PAYG **except streaming AI Chat (Ask Elfa)**.
- Enterprise: custom credits/RPM, **streaming AI Chat**, 2-year v3 history, support.
- API v3 is free on PAYG and Grow until 31 Oct 2026, then credit-priced.

## Our key (`ap-prod`)
`tier: enterprise`, 10M credit limit, 150 RPM, billing `arrears`, no overage cap. Scopes include all v3 groups and `chat`.

## Endpoints we call today (all v2)
- Data: `/v2/data/top-mentions`, `keyword-mentions`, `token-news`, `event-summary`, `trending-narratives`; `/v2/aggregations/trending-tokens`
- Auto: `/v2/auto/queries` (create, get, delete, cancel, validate), `/v2/auto/executions/{id}`, `/v2/auto/chat` (Builder Chat), `/v2/auto/validate-symbol/{ex}/{sym}`
- Account: `/v2/key-status`

None are in the Enterprise-only set. We call no `/v3/*` and no `/v2/chat` or chat-stream.

## Update 24th Oct 2026
Elfa paused on 23 Oct (Railway env renamed to `ELFA_API_KEY_PAUSED`); key still `active`, usage today 0, month 233,461. What we need from Elfa and what to replace: [[Elfa Replacement — What Agent Pear Actually Needs]].
