# Pear Protocol

Full engineering record already lives here — this note is a pointer + summary, not a re-parse. Per-company rule: do not re-derive Pear from repo; source of truth is the existing evidence bank.

- Full timeline, PR-by-PR: [[02 Work/Pear Protocol/My Work at Pear Protocol]]
- Condensed resume-ready entries: [[02 Work/Pear Protocol/Resume Evidence]]
- Feature ownership map: [[02 Work/Pear Protocol/Features Owned]]
- Decisions log: [[02 Work/Pear Protocol/Decisions/Decisions]]
- Incidents/investigations: [[02 Work/Pear Protocol/Incidents & Investigations/Incidents & Investigations]]
- Engineering lessons: [[02 Work/Pear Protocol/Engineering Lessons]]

## Role

**AI & Web3 Intern**, per [[01 Self/My Identity]] — working on Agent Pear (the AI trading agent). Exact start date not independently confirmed; earliest commit in retained git history is 2026-07-07, but [[02 Work/Pear Protocol/My Work at Pear Protocol#Missing Context Worth Adding Manually|the work-history note itself flags]] this may just be the start of retained history, not the actual start date.

## Dates

2026-07-07 → present (2026-08-30 as of last evidence pull). Active/ongoing.

## What the company/product does

Pear Protocol / Pear Foundation builds **Agent Pear**, an AI trading agent for crypto derivatives (pair trades, baskets, multi-venue execution across Hyperliquid and Lighter) reachable via Telegram and web chat, with an MCP surface for external hosts.

## My Work

See [[02 Work/Pear Protocol/My Work at Pear Protocol#Timeline]] for the full dated timeline (2026-07-07 → 2026-08-30, 62 self-opened merged PRs, 130 authored commits). Not re-summarized here to avoid drift between two copies of the same evidence — edit the source note, not this pointer.

## Technologies

TypeScript, NestJS, Zod, grammY (Telegram), ioredis/Redis, Langfuse (LLM observability), Postgres/Prisma, OpenRouter multi-model routing (Gemini, GPT-OSS, MiMo), Hyperliquid + Lighter exchange SDKs, MCP protocol, embedding-based KB retrieval.

## Strongest Evidence

Full ranked list: [[02 Work/Pear Protocol/My Work at Pear Protocol#Strongest Resume / Career Evidence]]. Top entries, unchanged from that ranking:

1. Root-caused a silent trade-execution defect from a cross-lingual production trace (#248).
2. Designed a confirm-time margin safety gate (#569).
3. Found and closed a crash silently substituting a user's trade request (#584).
4. Closed a recurring 13-in-3-days production error at its remaining call sites (#431, #514).
5. Disclosed a hidden ~2x margin exposure on leverage-clamped trades (#593).
6. Quantified and fixed retry-storming on rebalance weight validation (#642).
7. Removed an unsigned-webhook fallback from the signals security surface (#555).
8. Built an asymmetric, downgrade-only verification gate for reduce-only failures (#337).
9. Tuned an LLM-output security guard against a reproduced false-positive/negative pair (#639+follow-up).
10. Built the shared LLM-error observability layer later used to diagnose a Gemini incident (#258, #300).

## Missing Context

Unchanged from the source note — see [[02 Work/Pear Protocol/My Work at Pear Protocol#Missing Context Worth Adding Manually]]. Notably: real user-impact scale of any incident, team division of labor on Lighter integration, actual start date, current status of PRs open as of 2026-08-30.
