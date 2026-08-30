# Healthcare Check — Status

  

Handoff doc for whoever (human or AI) picks this up next. Read this before `plan.md` (the

original spec, still present locally but gitignored — ask the user for it if you don't have it in

context). This file is the "what's actually true right now" doc; `plan.md` is the "what we

originally intended" doc. When they disagree, this file wins.

  

---

  

## Where things stand

  

**M0–M7 done. M8 partially done (UI polish complete; deploy hardening not started).**

  

| Milestone | Status | Notes |

|---|---|---|

| M0 — Scaffolding, auth, deploy skeleton | ✅ | Next.js + Supabase auth (email/password) |

| M1 — Schema + seed data | ✅ | Core tables + curated `reference_items` demo dataset |

| M2 — Upload flow | ✅ | Signed-URL upload to private Storage bucket |

| M3 — Groq provider + extraction pipeline | ✅ | `pdf-parse`-equivalent (pdf.js) + Groq vision fallback |

| M4 — Deterministic audit engine | ✅ | Price/quantity/duplicate/package-overlap/unexplained checks, no LLM |

| M5 — Explanation, questions, findings UI | ✅ | **Workflow A (hospital documents) complete end-to-end** |

| M6 — Insurance extraction + policy summary | ✅ | Policy field extraction + `coverage_gap` deterministic checks |

| M7 — Policy retrieval (FTS) + Q&A + comparison | ✅ | **Workflow B (insurance) complete end-to-end** |

| M8 — UI polish (landing/case page, loading states, findings/evidence) | ✅ | No colors/animations added, grayscale only — see changelog |

| M8 — Deploy hardening (responsive pass, error states, real deploy) | ⬜ **next** | Not started — see "Next up" below |

  

Both demo workflows (hospital-bill audit, insurance Q&A + comparison) work end-to-end and have

been verified live against real Groq calls and the real hosted Supabase project — not just unit

logic. See the changelog below for exactly what was tested.

  

---

  

## Architecture snapshot

  

- Next.js (App Router, TS, Tailwind) in `healthcare-check/`, one deployable app, no separate backend.

- Supabase: Postgres (RLS everywhere) + Auth + Storage. **Hosted cloud project**, not the local

  `supabase start` stack — `healthcare-check/supabase/config.toml` exists but is unused; nothing

  runs locally except the Next.js dev server.

- Groq for all LLM calls (extraction/classification, audit-question phrasing, policy Q&A),

  behind no formal abstraction yet — each `lib/documents/*.ts` file makes its own `fetch` call to

  Groq's OpenAI-compatible endpoint. (`plan.md` describes a `LLMProvider` interface; that

  abstraction was never actually built — every call site just calls Groq directly. If you add a

  second provider, extract the interface then, not before.)

- Deterministic logic (arithmetic, matching, capping) always stays in plain TypeScript —

  `lib/audit/*`, `lib/insurance/compare.ts`, `lib/rag/policy.ts`'s derived-fact computation. The

  LLM is never asked to do arithmetic; see M7 entry below for why this matters in practice.

- Policy retrieval is plain Postgres full-text search (generated `tsvector` column + GIN index +

  a `ts_rank`-ordering RPC). No pgvector, no embeddings, per `plan.md` §D's phased approach —

  don't add these unless FTS recall is demonstrably failing in a real (not synthetic-fixture) test.

- **Case creation is lazy, not eager.** "New case" navigates to `/case/new` (a stateless shell,

  `components/new-case-shell.tsx`) with no DB row behind it. A `cases` row is only inserted on the

  first real action — editing the title away from empty, or picking a file to upload — via a

  shared `ensureCase()` so whichever fires first is the only thing that creates one. Do not revert

  this to eager creation (`POST /api/cases` directly from the "New case" click) — that was the

  exact bug this fixed: every click, including immediately navigating away, used to leave a

  permanent "Untitled case" row behind. If you touch `NewCaseButton`, `NewCaseShell`, or

  `UploadDocument`'s `caseId: string | null` / `ensureCase` prop pair, keep this property.

  

---

  

## Running this locally

  

```bash

docker compose up -d      # from repo root; builds nothing, just npm install && npm run dev

```

  

App is on `http://localhost:3000`. `docker-compose.yml` mounts `node_modules` as a **named

volume**, not a bind mount — so `healthcare-check/node_modules` on the host will look empty/absent;

that's expected, dependencies live inside the container/volume. Don't `npm install` on the host.

  

Secrets live in a root-level `.env` (gitignored, not committed) referenced by `docker-compose.yml`'s

`env_file`. Relevant vars: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`,

`SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_DB_URL` (direct Postgres connection string), `GROQ_API_KEY`.

  

### Applying a new DB migration

  

There's no `psql`, no `supabase` CLI, and no `pg` npm dependency installed in the container. The

pattern used for the M7 migrations (`policy_chunks` table + `search_policy_chunks` RPC, then a

follow-up fix) was:

  

1. Write the migration SQL under `healthcare-check/supabase/migrations/<timestamp>_name.sql`.

2. `docker compose exec web sh -c 'npm install pg --no-save --silent'`

3. Write a throwaway `.mjs` script that reads the file and runs it via `new pg.Client({ connectionString: process.env.SUPABASE_DB_URL })`, then `node` it inside the container.

4. Delete the throwaway script. The migration file itself stays committed as the source of truth.

  

This is a real gap, not a stylistic choice — worth fixing properly in M8 or later (install `pg` for

real, or get the `supabase` CLI into the container) rather than repeating the one-off-script dance

every time.

  

### Creating a test user without hitting email limits

  

The hosted Supabase project's auth email rate limit (2/hour) gets hit fast during iterative

testing. Don't use the signup form more than once or twice per session. Instead create users

directly, pre-confirmed, via the admin API using the service role key:

  

```bash

curl -X POST "$NEXT_PUBLIC_SUPABASE_URL/auth/v1/admin/users" \

  -H "apikey: $SUPABASE_SERVICE_ROLE_KEY" -H "Authorization: Bearer $SUPABASE_SERVICE_ROLE_KEY" \

  -H "Content-Type: application/json" \

  -d '{"email":"you+test@gmail.com","password":"...","email_confirm":true}'

```

  

(Gmail `+tag` addresses work fine and are how every test account so far has been named —

`prakhar.dwivedi.3782+<milestone>test@gmail.com`.)

  

---

  

## Known gotchas (learned the hard way this session, don't re-discover them)

  

- **The persistent browser tool tab keeps cookies across sessions.** If you log in as a new test

  user and the page renders old data (wrong case, wrong policy numbers), don't trust it — decode

  the `sb-<project-ref>-auth-token` cookie (base64 JSON, has `.user.email`) to check who you're

  *actually* authenticated as before debugging anything else. When in doubt, clear

  `document.cookie` + `localStorage` and log in fresh.

- **Full-page `navigate()` to a dynamic route can serve a stale Next.js client-router-cached 404**,

  even when the row genuinely exists server-side. Symptom: typing a `/case/<id>` URL directly

  404s, but clicking the same link from `/dashboard` (soft nav) renders fine. If you hit this,

  navigate to `/dashboard` first and click through rather than fighting the direct URL.

- **`websearch_to_tsquery` ANDs bare words.** The first version of `search_policy_chunks` used it

  directly on the raw question text, which meant a multi-topic question (e.g. "cataract surgery

  how much... waiting periods") required every stemmed word to land in the *same* chunk — recall

  was near zero on anything but single-fact questions. Fixed in the `20260809130000` migration by

  inserting `or` between words before calling `websearch_to_tsquery` (OR semantics, still can't

  throw on weird input). If retrieval feels too strict again, check this first.

- **pdf.js (`lib/documents/pdf.ts`) flattens each page to one whitespace-joined line** — no

  newlines survive extraction. `lib/rag/policy.ts`'s chunker only splits on `Section N:` /

  `Clause N:` markers for exactly this reason, and deliberately labels chunks with just the bare

  marker (`"Section 4"`), never a guessed title — guessing a title from flattened text risks

  mislabeling a citation, which is worse than a plain section number.

- **File upload in the in-app browser tool has no native file-picker support.** The working

  pattern used throughout M6/M7 testing: base64-encode the fixture PDF, then in `javascript_tool`

  construct a `File` + `DataTransfer`, assign to the `<input type="file">`'s `.files`, and dispatch

  a `change` event. Don't try to click the file input and hope for a picker.

- **Deleting a component file while the dev server is running can leave it in a broken state** —

  the long-running `next dev` process kept an in-memory reference to `document-pages.tsx` /

  `document-items.tsx` after they were deleted (replaced by `document-details.tsx`), throwing

  `ReferenceError: DocumentPages is not defined` on every server render even though no source file

  referenced it anymore and a clean `tsc`/`eslint` passed. Deleting `.next` didn't fix it either —

  only `docker compose restart web` (a real process restart) did. If you delete/rename a component

  file and things break in ways that don't match the current source, restart the container before

  debugging further.

- **When testing async navigation via the browser tool, don't trust `location.href` checked

  immediately after triggering the action** — checked too early it'll still show the old URL even

  though the request is genuinely in flight and `router.push` fires correctly moments later.

  Verified this via server logs (clean 200/201s, correct `GET /case/[id]` at the end) before

  concluding a real bug existed. Wait a beat (or check server logs) before concluding a navigation

  didn't happen.

  

---

  

## Changelog

  

### 2026-08-09 — M8 follow-ups: lazy case creation

- Root cause: "New case" used to `POST /api/cases` eagerly on click, before the user did anything —

  navigating straight back left a permanent junk "Untitled case" row (same failure mode as a

  ChatGPT "new chat" screen persisting even if you never send a message).

- Fix: `NewCaseButton` is now a plain `Link` to `/case/new` (no API call). `app/case/new/page.tsx`

  + `components/new-case-shell.tsx` render a stateless shell; the case row is created only on the

  first real action (title edit or file upload) via a shared, ref-guarded `ensureCase()` so

  concurrent triggers can't double-create. `UploadDocument` now takes `caseId: string | null` +

  optional `ensureCase`; when `caseId` is `null` it resolves the real id as step 0 of the upload

  pipeline, then `router.push`es to the real case instead of `router.refresh()`ing in place. See

  the architecture note above — **don't revert this to eager creation.**

- New client helper `lib/cases/create-case.ts` (`POST /api/cases` wrapper), shared by the title

  path and the upload path.

- **Verified against the DB directly** (not just the UI): visiting `/case/new` and navigating away

  created zero rows; the title-only path and the upload-only path each created exactly one row.

- Hit a stale-dev-server issue mid-testing (see gotchas) — a `docker compose restart web` fixed it,

  unrelated to correctness of the feature itself.

  

### 2026-08-09 — M8 follow-ups: auto-title + editable case title, document details redesign

- Cases were always stuck at "Untitled case" with no way to rename them. Fixed two ways:

  - **Auto-fill**: `suggested_title` added to both extraction schemas

    (`lib/documents/structure.ts`, `lib/documents/policy.ts`) — organization name (hospital/insurer,

    read off the document, never invented) + document type, e.g. "City Care Hospital — Estimate".

    Applied via `lib/documents/case-title.ts`'s `maybeSetCaseTitle()`, called from both the

    structure and policy routes, but **only when the case title is still the literal default** —

    it never overwrites an auto-set or user-edited title, and only the *first* processed document

    in a case gets to name it.

  - **Manual edit**: click-to-edit title (`components/editable-case-title.tsx`) + new

    `PATCH /api/cases/[id]` route.

- Separately, fixed a real UX complaint: the old `DocumentPages`/`DocumentItems` toggles were two

  independent buttons that could both be open at once, splitting the card into cramped side-by-side

  columns and reflowing the whole page by however long the content was. Replaced with

  `components/document-details.tsx` — one toggle, two tabs (Line items / Extracted text) sharing a

  fixed `max-h-72` scrollable panel, so opening it or switching tabs never reflows anything else.

  Old `document-pages.tsx` / `document-items.tsx` deleted (nothing else referenced them).

- Verified live: uploading the synthetic estimate auto-titled the case "City Care Hospital —

  Estimate" pulled straight from the PDF header; manual rename via click-to-edit persisted

  correctly; tabbed details panel confirmed not to reflow the page on toggle/tab-switch.

  

### 2026-08-09 — M8: UI polish (landing page, case page, loading states, findings/evidence)

Scope was explicitly UI/structure only — no new features, no colors beyond the existing zinc/

black/white palette, no animation beyond a plain `animate-pulse` dot on active loading steps.

- Landing page (`app/page.tsx`) rewritten: real hero copy, top nav consistent with the rest of the

  app, two honest "how it works" columns instead of just two buttons.

- Case page decomposed from one 350-line file into `DocumentCard`, `CaseSummary`,

  `CoverageSummary`, `FindingCard` components under clear uppercase section labels, consistent

  card backgrounds, findings sorted high-confidence-first.

- New `components/processing-steps.tsx` — fixed-stage loading indicator (done/active/pending/error

  dot + label) replacing the old single-line-that-keeps-changing-text pattern. Wired into

  `UploadDocument` (**Uploading → Extracting → Understanding**) and `RunAuditButton`

  (**Checking → Preparing questions**).

- Findings made scannable: human-readable type labels instead of raw `finding_type` enum values,

  confidence shown as filled-vs-outline badges (high = solid, medium/low = outline) instead of

  uniform pills.

- New evidence/citation block, shape-driven off the `evidence` jsonb rather than a per-type switch:

  a price-comparison line when `hospital_price`/`reference_price` are present, a quoted citation

  card (page badge + source text) when `original_text`/`page` are present, one citation per

  occurrence for `duplicate` findings. `coverage_gap` findings correctly render nothing extra here

  (no document quote exists for them — the description already states the extracted number).

- Verified live in the browser, light and dark, against the real M7 test case, including a live

  "Run audit" pass through the new stepper and citation rendering on real Ceftriaxone/Paracetamol

  price findings.

  

### 2026-08-09 — NPPA reference subset imported

- Added the approved 65-row NLEM-2022 subset from the live IPDMS “All Drugs Ceiling Prices” table

  (SO 1575(E), 25-Mar-2026), with `is_demo_data = false`, exact parsed sale-unit descriptors,

  per-row provenance, and the raw source price cell preserved in `reference_items.notes`.

- Kept the 30 legacy demo rows unchanged; the imported subset is separately reviewable in

  `healthcare-check/supabase/seed/nppa_reference_items.sql`.

- Added a fail-closed unit check: incompatible or unstated tablet/capsule/vial/ml/pack context

  produces a `unit_unverified` finding and no price-overcharge comparison. NPPA findings now say

  “worth checking” and explicitly explain that the ceiling is an ex-manufacturer, pre-GST legal

  maximum—not a hospital billing cap or a market price.

- Hosted verification: 65 imported rows, all non-demo, all effective-dated 25-Mar-2026, all with

  the approved IPDMS source URL and raw source-cell notes. `tsc`, ESLint, and audit regression

  fixtures passed.

  

### 2026-08-09 — M7: policy retrieval (FTS) + Q&A + estimate comparison

- Added `policy_chunks` table (page/section-tagged text, generated `tsvector` + GIN index) and a

  `search_policy_chunks` RPC for ranked full-text search — two migrations (initial + the

  `websearch_to_tsquery` OR-semantics fix described above).

- `lib/rag/policy.ts`: section-marker chunker + `computeDerivedPolicyFacts()` (deterministically

  pre-multiplies percent-based sub-limits, e.g. "2% of sum insured", in TypeScript so the LLM

  never does arithmetic).

- `lib/documents/policy-qa.ts` + `POST /api/insurance/[caseId]/ask`: Groq call constrained to only

  the retrieved excerpts + precomputed facts, strict JSON schema with

  `basis ∈ {policy_states, calculated_from_policy, requires_confirmation}` and page/section

  citations.

- `lib/insurance/compare.ts` + `POST /api/insurance/[caseId]/compare`: pure deterministic

  estimate-vs-policy breakdown (sub-limit capping via conservative keyword match, deductible,

  co-pay), with a `whyLines[]` explaining every deduction in plain language.

- `AskPolicy` / `CompareEstimate` UI components wired into the case page.

- **Verified live** (real Groq calls, real hosted Supabase, fresh test account): 4/4 Q&A test

  questions correct (one directly-stated answer, one spanning two policy sections, one requiring

  the precomputed-percentage path, one genuinely unanswerable — correctly refused rather than

  conflating "ambulance" sub-limit with "air ambulance"). Compare flow verified by hand against a

  synthetic ₹44,760 estimate: sub-limit caps, deductible, and co-pay all arithmetically correct,

  both via direct API call and the rendered UI.

- Test data currently live in the DB (left in place on request, not cleaned up): user

  `prakhar.dwivedi.3782+m7test@gmail.com`, case `f191f486-2c35-40f7-bb6c-ab3093902325` with a

  synthetic policy + estimate uploaded and processed.

  

### 2026-08-09 — M6: insurance policy extraction + coverage summary

- `lib/documents/policy.ts` + `POST /api/documents/[id]/policy`: Groq extraction of structured

  policy fields (sum insured, room/ICU rent limits, co-pay, deductible, waiting periods,

  sub-limits, exclusions, consumables coverage) into the (already-existing-from-M1)

  `insurance_policies` table.

- `lib/audit/insurance-coverage.ts`: deterministic `coverage_gap` findings from policy fields

  alone (no LLM) — room rent/ICU caps, co-pay, deductible, uncovered consumables, sub-limits,

  waiting periods — wired into the existing `lib/audit/run.ts` audit engine.

- Upload flow auto-chains policy extraction when a document classifies as `policy`.

- Coverage summary card + "Ask the insurer:" vs "Ask the hospital:" type-aware question labels

  added to the case page.

- **Verified live** end-to-end: upload → extract → classify → policy extraction → 7 coverage_gap

  findings → 7 insurer-facing questions, all grounded, zero hallucinated numbers.

  

### Before this session (M0–M5)

Not covered in detail here — see git history / prior session context if needed. Summary: hospital

document workflow (Workflow A) built and verified end-to-end — upload → extract → structure →

deterministic audit → LLM-generated hospital-facing questions → findings UI. Reference pricing

dataset (`healthcare-check/supabase/seed.sql`) is hand-curated demo data, explicitly not

authoritative — every row's `source_name` says so on purpose.

  

---

  

## Next up: M8 remainder (deploy hardening)

  

Done already (see changelog above): landing page, case page visual cleanup, staged loading

indicators, findings/evidence clarity, auto-title + editable title, document details redesign,

lazy case creation. Still open, per `plan.md`:

- **Responsive/mobile-viewport pass** — not checked at all yet. Verify current state before

  assuming anything works below desktop width.

- **Case deletion** — no delete-case feature exists (no route, no UI). This is also why the

  handful of junk "Untitled case" rows created before the lazy-creation fix (see changelog) are

  still sitting in the DB under the test accounts below — nothing to clean them up with yet short

  of a direct DB delete.

- **Error states** — beyond the inline `text-red-600` messages already in each form/button

  component, no dedicated error-boundary or empty/broken-state design pass has happened.

- **Real deploy** — this has only ever run via `docker compose up` against the hosted Supabase

  project. Never deployed to Vercel (or anywhere) yet; that's still fully ahead.

  

### Current live test data (hosted Supabase project, not cleaned up)

Multiple test users exist under `prakhar.dwivedi.3782+<tag>test@gmail.com` (m6test, m7test, ...),

created via the admin API pattern above. Several cases exist per user, mixing real fixture data

(synthetic hospital estimate + insurance policy PDFs, described in earlier changelog entries) and

stray empty/junk cases from before the lazy-creation fix. None of this is fixture data checked

into the repo — the synthetic PDFs were generated ad hoc via `reportlab` into the session scratch

directory each time and are not preserved anywhere durable. If you need them again, regenerate

from the descriptions in the M6/M7 changelog entries rather than hunting for a saved file.