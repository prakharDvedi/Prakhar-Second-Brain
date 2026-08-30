# Resume

Master resume / achievement ledger — evidence-backed, not a polished one-pager. Pull from here when writing a role-specific resume. Last rebuilt: 2026-08-30, from [[Companies/Brinavv]] and [[Companies/Pear Protocol]] (pointing to [[02 Work/Pear Protocol/Resume Evidence]]). SuprHire not included — no repo access as of this writing.

## Snapshot

- **Name:** Prakhar Dwivedi
- **Current role:** AI & Web3 Intern, Pear Protocol / Pear Foundation — working on Agent Pear (per [[01 Self/My Identity]])
- **Direction:** Software engineering × backend systems × applied AI — agent reliability, observability, security, RAG/retrieval (per [[01 Self/My Identity#What Kind of Engineer Am I Becoming?]])
- **Education:** Not recorded in vault — needs manual entry.

---

## Experience

### Pear Protocol — AI & Web3 Intern (2026-07 – present)

Full record: [[Companies/Pear Protocol]] → [[02 Work/Pear Protocol/Resume Evidence]]. Selected strongest entries:

**Root-caused a silent trade-execution defect from a cross-lingual production trace**
- Traced a Dutch-language production session where an explicit "5% profit" instruction was silently overridden by a documented 100%-PnL default
- System: streaming chat DAG context propagation (`agent-pear-v2`)
- Result: fixed with a one-line-pair change; new regression spec verified to fail pre-fix, pass post-fix (12 suites / 66 tests green)
- [[02 Work/Pear Protocol/Resume Evidence#Root-caused a silent trade-execution defect from a cross-lingual production trace|Evidence]] (PR #248)

**Designed a confirm-time margin safety gate**
- Added a fail-closed check against fresh `availableToTrade` before any leverage/order write, 100bps configurable buffer, reduce-only explicitly exempted
- System: `open_basket` trade-execution path
- Result: a safety feature shipped ahead of a failure mode, not reactive to one; focused tests cover insufficient/unavailable/changed-balance cases
- [[02 Work/Pear Protocol/Resume Evidence#Designed a confirm-time margin safety gate|Evidence]] (PR #569)

**Found and closed a crash silently substituting a user's trade request**
- Found by manually triaging 5 days of Langfuse ERROR-level traces, not from a bug report
- System: TRIGGER/LADDER JSON validation, `open_position` plan usecase
- Result: malformed input now fails as a named `PlanningError` instead of crashing and silently retrying with a different (wrong) trigger type
- [[02 Work/Pear Protocol/Resume Evidence#Found and closed a crash that was silently substituting a user's trade request|Evidence]] (PR #584)

**Closed a recurring 13-in-3-days production error at its remaining call sites**
- Verified via live Discord alert + trace that a fix covered only one of several shared entry points, then closed the rest
- System: `DagService`, ambiguous HIP-3 symbol resolution
- Result: full DAG suite 8/8 files, 60/60 tests; new spec proves the fix isn't a blanket catch-all
- [[02 Work/Pear Protocol/Resume Evidence#Closed a recurring 13-in-3-days production error at its remaining call sites|Evidence]] (PRs #431, #514)

**Disclosed a hidden ~2x margin exposure on leverage-clamped trades**
- Traced to a real user's numbers ($1,000 stated margin → $1,999.95 actual), diagnosed as a disclosure gap rather than an engine bug
- System: trade-execution composer prompt
- Result: fixed at the correct layer (prompt) without touching intentional engine behavior
- [[02 Work/Pear Protocol/Resume Evidence#Disclosed a hidden ~2x margin exposure on leverage-clamped trades|Evidence]] (PR #593)

**Quantified and fixed retry-storming on rebalance weight validation**
- Measured 4 traces / 21 rejected attempts / drift up to 15% (one trace: 10 back-to-back self-corrections) before choosing rescale-over-reject
- System: `plan-rebalance.usecase.ts`, shared by agent + MCP adapters
- Result: provably safe rescale (per-leg weights already `.positive().max(1)`, so sum > 0 before division)
- [[02 Work/Pear Protocol/Resume Evidence#Quantified and fixed retry-storming on rebalance weight validation|Evidence]] (PR #642, open as of 2026-08-30)

**Removed an unsigned-webhook fallback from the signals security surface**
- System: Elfa signal-trigger HMAC verification
- Result: every armed production webhook now requires signature verification; pre-existing armed rows detected as drift and re-armed post-deploy; 80/80 tests passing
- [[02 Work/Pear Protocol/Resume Evidence#Removed an unsigned-webhook fallback from the signals security surface|Evidence]] (PR #555)

**Breadth across a full new execution venue (Lighter integration)**
- Touched Hyperliquid-only assumptions in 5-6 structurally different places across `apps/agent`, `packages/execution`, and the KB pipeline
- Result: verified live against the gateway (227/227 Lighter instruments), KB eval held at retrieval @1/@3/@5 97/100/100
- [[02 Work/Pear Protocol/Resume Evidence#Breadth across a full new execution venue (Lighter integration)|Evidence]] (PRs #556-589, teammate-shared umbrella PRs)

---

### Brinavv — Role unclear, see [[Companies/Brinavv]] (2026-02 – 2026-07)

Full record: [[Companies/Brinavv]]. Selected strongest entries:

**Built the foundational HR/employee/payroll module from scratch**
- Action: authentication middleware, employee CRUD, payroll config, leave assignment, across 8+ commits and 2 PRs
- System: `School-ERP-Backend`, Node/Express/MongoDB
- Result: PR #81 (7,229/5,397 lines, 242 files), PR #97 (4,269/2,782 lines, 147 files) — first working HR/payroll/leave slice of the product
- [[Companies/Brinavv#HR/Employee module: from nothing to a working payroll + leave system|Evidence]]

**Made the call to extract Employee into its own module**
- Action: pulled employee logic out of the HR folder into a top-level module, updating every import path in the same change
- Result: unblocked later features (attendance, biometric integration) to depend on employee directly rather than through HR
- [[Companies/Brinavv#Restructured the employee module out of the HR folder|Evidence]] (PR #110)

**Shipped biometric device integration for attendance**
- Action: new `src/integrations/biometric/` module wired into existing attendance logic, plus a payroll/leave-validation hardening pass in the same PR
- Result: largest single PR on the repo (2,710/809 lines, 35 files), ~3-month-lived branch
- [[Companies/Brinavv#Biometric attendance integration + payroll/leave hardening (largest single change)|Evidence]] (PR #142)

**Built the frontend for the entire HR/payroll/leave/attendance feature set**
- Action: staff directory, CSV bulk-import UX, attendance registers, leave-group management UI, and a browser-based biometric simulator to test hardware-dependent logic without a device
- System: React 19 + Redux Toolkit, Vite
- Result: 4 PRs, largest 4,594/1,521 lines across 35 files
- [[Companies/Brinavv#Frontend: HR/payroll/leave/attendance UI, CSV bulk-import UX, biometric simulator|Evidence]] (PRs #332, #346, #354, #364)

---

### SuprHire

No entries — repos (`Pulkit0729/suprhire-backend`, `suprhire-ai-recruit`) identified from prior session history but return 404 under the current GitHub account. Skipped for this pass; revisit once access is confirmed. See [[Resume Claims Needing Manual Verification]] below.

---

## Projects

No dedicated project notes currently exist in the vault beyond the company work above (MedBud is referenced in [[01 Self/My Identity#Building|My Identity]] as taken to the Best Possible AI Hackathon finale at AltSpace Dharamshala, but has no standalone project note yet). Not populated further per instruction to avoid overpopulating without real evidence.

---

## Skills Backed by Evidence

### Backend
- Node.js / Express (Brinavv) — [[Companies/Brinavv]]
- NestJS / TypeScript (Pear) — `apps/agent` framework, confirmed via `package.json`
- MongoDB + Mongoose (Brinavv); PostgreSQL + Prisma (Pear)
- Redis / ioredis (Pear) — atomic-consume patterns, rate-limit counters
- Zod schema validation (Pear) — used as the actual fix mechanism in PR #584, not incidental

### AI / Agent Systems (Pear only — no comparable evidence at Brinavv)
- Multi-model LLM routing under provider instability (Gemini, GPT-OSS, MiMo swaps, live-verified each time)
- LLM observability — built `formatLlmError()`, a shared error-enrichment utility, later used to diagnose a live Gemini incident
- RAG/KB retrieval — embedding-based knowledge base with a dual-maintenance discipline (doc + index + rebuilt artifact + eval)
- Prompt-injection defense — treating client-supplied draft state as untrusted, fenced input

### Trade-execution / Financial-safety design (Pear only)
- Fail-closed safety gates (margin checks, TWAP protection, TRIGGER/LADDER schema validation)
- Disclosure-correctness fixes (margin-change disclosure, hidden 2x exposure)
- Data-backed validation design (measured trace evidence before choosing rescale-over-reject)

### Frontend
- React 19 + Redux Toolkit + Vite (Brinavv) — [[Companies/Brinavv#Technologies]]

### Platform / Messaging
- Telegram bot integration via grammY (Pear) — command routing, delivery-ordering observability, multi-account UX
- CSV/Excel bulk import pipelines (`xlsx`) — both companies (Brinavv HR import; not independently confirmed at Pear)

### Not claimed
- No skill is listed here solely because it appears in a `package.json` — every entry above traces to a specific PR or file cited in [[Companies/Brinavv]] or [[02 Work/Pear Protocol/Resume Evidence]].

---

## Metrics Bank

| Company | Metric | Value | Evidence |
|---|---|---:|---|
| Pear | Self-opened merged PRs (2026-07-07 – 2026-08-30) | 62 | [[02 Work/Pear Protocol/My Work at Pear Protocol]] |
| Pear | Authored commits (same window) | 130 | [[02 Work/Pear Protocol/My Work at Pear Protocol]] |
| Pear | Recurring ambiguous-symbol crash, pre-fix frequency | 13 occurrences / 3 days | PR #514 |
| Pear | Rebalance-weight rejections quantified before fix | 21 rejected attempts / 4 traces, drift up to 15% | PR #642 |
| Pear | Lighter instruments verified live at correct margin mode | 227 / 227 | PR #556-564 cluster |
| Pear | KB retrieval eval (post-Lighter changes) | @1/@3/@5 = 97/100/100 | PR #520, #559, #589 |
| Pear | Internal-leak guard test coverage (adversarial + benign) | 60 / 60 passing | PR #639 |
| Pear | Signal-evidence test coverage | 40 / 40 passing | PR #640 |
| Pear | DM Telegram command-menu reduction (`/aipear` unlisting) | 22 → 21 commands | PR #327, live-verified |
| Brinavv | Largest single PR (biometric integration) | 2,710 / 809 lines, 35 files | PR #142 |
| Brinavv | Foundational HR/payroll PR | 7,229 / 5,397 lines, 242 files | PR #81 |
| Brinavv | Backend PRs authored | 8 | [[Companies/Brinavv]] |
| Brinavv | Frontend PRs authored | 4 | [[Companies/Brinavv]] |
| Brinavv | Automated test coverage | 0 (test script is a stub) | `package.json`, verified directly |

Only metrics with a direct repo/PR/test-output source are listed. No percentage-improvement or production-impact numbers are included for Brinavv — none exist in the repo.

---

## Strongest Career Evidence

Ranked across both companies — not by recency.

1. **Root-caused a silent trade-execution defect from a cross-lingual production trace** (Pear, #248). Direct trace reading across a language barrier, one-line-pair fix, regression-proven. High confidence, high technical depth.
2. **Designed a confirm-time margin safety gate** (Pear, #569). Proactive safety design, not reactive. High confidence, strong ownership signal.
3. **Found and closed a silent trade-substitution crash** (Pear, #584). Found via deliberate 5-day trace triage, not a bug report. High confidence.
4. **Quantified retry-storming before fixing it** (Pear, #642, open). Data-backed validation design with a stated, provable safety property. High confidence.
5. **Biometric attendance integration** (Brinavv, #142). Largest single PR across either company by lines/files/duration; new hardware-integration module built solo. High confidence on scope, no external impact metric.
6. **Employee module extraction** (Brinavv, #110). An unprompted architectural-boundary call, not a feature request. High confidence.
7. **Disclosed a hidden ~2x margin exposure** (Pear, #593). Traced to one real user's numbers; correctly diagnosed as disclosure, not a math bug. High confidence.
8. **Foundational HR/payroll/leave build** (Brinavv, #81, #97). Two of the largest PRs in either repo; first working slice of a real product module. High confidence on scope, qualitative-only impact.
9. **Removed an unsigned-webhook fallback** (Pear, #555). Concrete security hardening with an explicit rollout/re-arm plan. High confidence.
10. **Breadth across the Lighter execution-venue integration** (Pear, multiple PRs). Same root assumption found and fixed in 5-6 structurally different places, including inside a teammate's shared branch. Medium confidence (breadth claim is a synthesis, individual facts confirmed).
11. **Live-verified every emergency model swap** (Pear, #384-#479 sequence). Operational discipline sustained across repeated pressure, not just once. High confidence.
12. **Asymmetric downgrade-only reduce-only verification gate** (Pear, #337). Deliberately can-only-downgrade safety design. High confidence.
13. **Frontend biometric simulator** (Brinavv, #354). A specific, deliberate test-without-hardware tool — shows thinking about testability, not just shipping UI.

---

## Resume Claims Needing Manual Verification

- **SuprHire — entire company.** Repos identified (`Pulkit0729/suprhire-backend`, `suprhire-ai-recruit`) but inaccessible (404) under the current GitHub account. Needs either account access, owner-granted collaborator access, or a different source before any SuprHire evidence can be built.
- **Exact job titles at Brinavv and Pear.** Pear has "AI & Web3 Intern" from [[01 Self/My Identity]]; Brinavv has no title anywhere in available sources.
- **Employment dates (start/end) for Brinavv.** Only first/last commit timestamps are known (2026-02-16 → 2026-07-18); actual engagement window could be longer or shorter.
- **Actual start date at Pear.** 2026-07-07 is the earliest commit in retained git history — may not be the real start date. See [[02 Work/Pear Protocol/My Work at Pear Protocol#Missing Context Worth Adding Manually]].
- **Team size at both companies.** Brinavv's backend repo has 14 total contributors; how work was divided is unknown. Pear's Lighter integration shows interleaved commits with teammate GPSxtreme, but planning/division-of-labor isn't in the repo.
- **Any production-impact numbers** (users affected, revenue, cost savings, support tickets) — none exist in either company's repo evidence. Every "impact" line in this file is either a test-coverage number, a traced-incident count, or explicitly marked qualitative.
- **Brinavv product scale** — number of schools/students/staff actually using the ERP, if any went live, is unrecoverable from GitHub.
- **Current PR states** — Pear PRs open as of 2026-08-30 (#639, #640, #641, #642) and Brinavv's #364 need a live status check before being cited as "shipped."

## Knowledge Base

- [[04 Knowledge/Knowledge]] — evidence-linked technical knowledge extracted from the work and project record.
- Strongest areas: [[04 Knowledge/Web3 & Trading/Trading Execution Safety|Trading Execution Safety]], [[04 Knowledge/Distributed Systems & Reliability/Retry Semantics & Failure Recovery|Retry Semantics & Failure Recovery]], [[04 Knowledge/AI & LLM Systems/LLM Observability & Evaluation|LLM Observability & Evaluation]], [[04 Knowledge/AI & LLM Systems/RAG & Retrieval|RAG & Retrieval]], and [[04 Knowledge/AI & LLM Systems/Structured Outputs & Validation|Structured Outputs & Validation]].

---

*Rebuilt 2026-08-30. Brinavv and Career Journey sections are new (from GitHub archaeology, `Navodayaraj/School-ERP-Backend` + `Navodayaraj/Frontend_Fianl`, author filter `prakharDvedi`). Pear Protocol section reuses [[02 Work/Pear Protocol/Resume Evidence]] without re-deriving it. SuprHire omitted — access blocked.*
