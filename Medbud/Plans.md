# Healthcare Check — MVP Implementation Plan

  

Status: **Approved direction, updated per feedback — ready to start M0.**

Scope: one-day hackathon MVP, deployed vertical slice.

Priority order: **Workflow A (hospital documents) end-to-end first, Workflow B (insurance) second.**

  
## 0. Decisions (updated)

  

1. **LLM provider: Groq directly, not OpenRouter.** You already have a Groq key — use it. The provider is still hidden behind a single abstraction (§C), so swapping to OpenAI later (your stated plan) is a config change plus one small adapter, not a rewrite of call sites.

2. **OCR approach: vision-capable LLM, no traditional OCR pipeline.** Unchanged from before. Groq hosts vision-capable Llama models (exact model ID to confirm at implementation time, since Groq's hosted lineup changes — currently Llama 4 Scout/Maverick and/or Llama 3.2 Vision are the candidates). Same tradeoff as before: simplest path to structured output in a day, works best on clean/synthetic demo documents.

3. **RAG: start simple, not embeddings-first.** Insurance policy retrieval starts as plain page/section text storage + keyword / Postgres full-text search. No Hugging Face, no pgvector, no embeddings on day one. This is upgradeable later without a schema rewrite (see §D, §G).

4. **Auth: email + password.** Unchanged — avoids email-deliverability timing risk during a live demo.

5. **Pipeline runs synchronously per document**, not a queue. Unchanged — fallback documented in §K if latency is a problem.

6. **Build order: hospital workflow fully working before insurance workflow starts.** Insurance (RAG Q&A, coverage comparison) only begins once upload → extract → audit → evidence → explain → questions is solid end-to-end for hospital documents.

  

---

  

## A. Architecture

  

```

Browser (Next.js App Router, TS, Tailwind)

   │

   ├─ Supabase Auth (email/password) — session cookie via @supabase/ssr

   │

   ├─ Upload → Supabase Storage (private bucket, signed URLs)

   │

   └─ Next.js Route Handlers (server-only)

         │

         ├─ Extraction stage   → Groq (vision-capable model)         [LLM]

         ├─ Normalization      → same LLM call, structured JSON out  [LLM]

         ├─ Audit engine       → plain TypeScript, no LLM             [code]

         ├─ Reference lookup   → Postgres table match (curated data)  [code]

         ├─ Policy retrieval   → page/section text + keyword search   [code]

         │                       (Postgres full-text search; vector

         │                        search added later only if needed)

         ├─ Explanation/Qs     → Groq                                 [LLM]

         └─ Persistence        → Supabase Postgres (RLS-scoped)       [code]

  

Supabase Postgres: relational data (pgvector extension NOT enabled day-1)

Supabase Storage: private, per-user-scoped paths, signed URLs only

Vercel: hosts the single Next.js app (frontend + API routes)

```

  

**One deployable app.** No separate backend service.

  

---

  

## B. Stack — confirmed

  

| Component | Decision |

|---|---|

| Next.js + TS + Tailwind | Confirmed, no change. |

| Supabase (Postgres/Auth/Storage) | Confirmed. |

| Separate backend/microservice | **Rejected.** Plain TS covers PDF text extraction (`pdf-parse`), HTTP calls to Groq, Supabase client. No genuine need for a second service. |

| Groq (LLM) | Confirmed as initial provider, behind an abstraction so OpenAI can replace it later with a config change + one adapter file. |

| Vercel | Confirmed. |

| Vector DB / pgvector / embeddings | **Deferred, not rejected.** Not part of the day-1 build. Add only if plain-text retrieval proves too imprecise (§D). |

  

---

  

## C. AI architecture

  

**Extraction / OCR — vision-capable Groq model, no traditional OCR pipeline.**

Same reasoning as before: a vision LLM reading a page image directly and returning structured JSON collapses OCR + layout understanding + extraction into one step. Accuracy tradeoff accepted given the brief allows curated/synthetic demo documents.

  

Document handling split (unchanged):

- **Digital-native PDF** (embedded text layer): extract with `pdf-parse` first — cheap, exact, no LLM vision call needed.

- **Scanned PDF or photographed image**: send page image(s) to the Groq vision model. If Groq's current hosted model doesn't accept PDFs natively, images (JPG/PNG) are the safe fallback path — treat "give me a PDF and I'll rasterize it server-side" as out of scope for day 1 (this rasterization step was flagged as the top pipeline risk in the original plan and is easiest to avoid by scoping to images for the scanned case).

  

**Extraction + classification stay combined in one LLM call** (doc_type + structured fields via one JSON schema) — halves LLM round-trips vs. separate classify-then-extract stages, no accuracy cost.

  

**Reasoning/explanation model** — separate Groq call, text-only, takes the audit engine's structured findings + reference evidence, produces explanation + evidence summary + hospital-facing questions.

  

**Deterministic logic stays in TypeScript**: totals, duplicate detection, price comparisons, quantity flags, insurance limit math. The LLM never does arithmetic that code can do exactly.

  

**Model/provider abstraction** — unchanged in shape, Groq is just the first implementation:

```

LLMProvider {

  extractDocument(input: {images?, text?}, schema: JSONSchema): Promise<T>

  explain(findings, evidence): Promise<{summary, questions}>

  chat(messages): Promise<string>          // insurance Q&A, once built

}

```

One concrete implementation targets Groq's OpenAI-compatible chat-completions endpoint; model name is an env var (`LLM_MODEL_EXTRACT`, `LLM_MODEL_REASON`). Switching to OpenAI later = new adapter file implementing the same interface + env var change, no call-site changes.

  

**Embeddings — not built day 1.** No Hugging Face call, no embedding column, no vector index. See §D for what replaces it initially and when to add it.

  

**Reranking — not applicable yet**, since there's no vector search to rerank.

  

**Specialized medical/insurance models — skipped for MVP**, unchanged reasoning: a general LLM with a tight JSON-schema prompt outperforms a generic NER model on messy real-world documents without adding a dependency.

  

---

  

## D. RAG decision (revised: phased, not embeddings-first)

  

| Use case | Data shape | Day-1 approach | Later upgrade |

|---|---|---|---|

| **Insurance policy Q&A** | Long prose PDF, paraphrased questions | Store policy text as page/section rows; retrieve via Postgres full-text search (`tsvector`/`ts_rank`) plus simple keyword/heading match; feed top-N matched sections to the LLM with page numbers for citation | Add `pgvector` + Hugging Face embeddings only if keyword search demonstrably misses paraphrased questions in testing |

| Medicine/procedure reference pricing | Small, curated, structured | Postgres table match on normalized name (+ `pg_trgm` fuzzy match for spelling variance) | Not RAG, ever — this is a lookup problem regardless of scale |

  

Why this is defensible for the insurance case specifically: policy documents are section-headed and clause-numbered by nature ("Section 4: Exclusions", "Clause 12: Room Rent"), so keyword/full-text search against page+section-tagged rows already gets you most of the way to "find the room-rent clause" without semantic search. Embeddings buy recall on odd phrasings the keyword search misses — a real gap, but one to measure before paying for it in build time.

  

**Net effect: no pgvector extension, no HF token, no embedding calls in the day-1 build.** These become a scoped follow-up task, not a blocker for either demo.

  

---

  

## E. Hugging Face models — status

  

| Model | Purpose | Use now? |

|---|---|---|

| Small sentence-transformer (embeddings) | Policy chunk semantic search | **No — deferred.** Only added if full-text search proves insufficient (§D). |

| Reranker | Rerank retrieved chunks | No — irrelevant until embeddings exist. |

| LayoutLMv3 / Donut | Structured OCR | No — vision LLM covers this at MVP scope. |

| BioBERT / med7 | Medical NER | No — general LLM + JSON schema is more practical for messy inputs. |

  

Hugging Face is not part of the day-1 dependency list at all. Nothing above blocks either demo.

  

---

  

## F. Credentials checklist (updated)

  

### Required today

| Credential | Why | Server-only? |

|---|---|---|

| Supabase project URL + anon key | Client auth session, client-side Storage/DB calls under RLS | Anon key is public by design |

| Supabase service role key | Signed-URL generation, case deletion across storage+DB | **Yes — never expose to browser** |

| Groq API key | All LLM calls (extraction, reasoning) | **Yes** |

| Vercel account | Deployment target | n/a |

| GitHub account | Public repo, Vercel git integration | n/a |

  

### Optional later

| Credential | Why | When |

|---|---|---|

| OpenAI API key | Your planned provider swap once things are running | Post-MVP, config + one adapter file |

| Hugging Face API token | Embeddings, if full-text policy search proves insufficient | Only if §D's upgrade trigger is hit |

| Dedicated OCR provider (Textract/Document AI) | Only if vision-LLM extraction accuracy is insufficient on real documents | Fallback, not day-1 |

| External medicine pricing API | Replace the curated static reference dataset | Post-MVP |

  

---

  

## G. Database schema (minimum viable, revised)

  

```

cases

  id, user_id (fk auth.users), kind (hospital|insurance), title,

  status (processing|ready|error), created_at

  

documents

  id, case_id (fk), storage_path, doc_type

    (estimate|bill|prescription|quotation|policy|approval),

  original_filename, mime_type, page_count,

  extracted_raw jsonb,      -- full structured extraction, not raw file

  created_at, deleted_at

  

extracted_items

  id, document_id (fk), item_type

    (medicine|procedure|test|consumable|charge|misc),

  name, normalized_name, quantity, unit_price, total_price,

  source_page, raw_text, confidence

  

reference_items                 -- curated, structured, small

  id, category (medicine|procedure), name, normalized_name,

  reference_price, unit, source_name, source_url,

  effective_date, retrieved_at, notes

  

policy_chunks                   -- insurance docs only, plain text search

  id, document_id (fk), chunk_index, page, section_title, content

  -- full-text search index (GIN on to_tsvector(content))

  -- NOTE: no embedding column day-1; added via migration only if

  -- the retrieval upgrade in §D is actually triggered

  

audit_findings

  id, case_id (fk), document_id (fk nullable), finding_type

    (price|quantity|duplicate|package_overlap|unexplained|

     medicine_savings|coverage_gap),

  title, description, evidence jsonb, confidence (high|medium|low),

  related_item_id (fk nullable, extracted_items)

  

questions

  id, case_id (fk), finding_id (fk nullable), question_text

```

  

Same 8 tables as before; only change is `policy_chunks` drops its `embedding` column until/unless needed. All tables get RLS: `case.user_id = auth.uid()`, cascaded via the case/document relationship.

  

---

  

## H. API routes (minimum surface, revised)

  

```

POST   /api/cases                      create case

GET    /api/cases/:id                  case + documents + findings + questions

DELETE /api/cases/:id                  delete case (storage objects + DB rows)

  

POST   /api/documents                  create document row + return signed upload URL

POST   /api/documents/:id/process      run extraction → normalize → audit →

                                        explain → persist (synchronous)

  

POST   /api/insurance/:caseId/ask      full-text search over policy_chunks,

                                        top-N sections → LLM answer + page citation

POST   /api/insurance/:caseId/compare  estimate-vs-policy deterministic comparison

```

  

Still 7 routes. Upload goes browser → Supabase Storage directly via signed URL.

  

---

  

## I. Folder structure (revised)

  

```

/app

  /(auth)/login/page.tsx

  /(auth)/signup/page.tsx

  /dashboard/page.tsx

  /case/[id]/page.tsx                  results view

  /api/cases/route.ts

  /api/cases/[id]/route.ts

  /api/documents/route.ts

  /api/documents/[id]/process/route.ts

  /api/insurance/[caseId]/ask/route.ts

  /api/insurance/[caseId]/compare/route.ts

/lib

  /supabase/client.ts, server.ts        browser/server Supabase clients

  /llm/provider.ts                      LLMProvider interface + Groq impl

  /llm/schemas.ts                       JSON schemas for extraction outputs

  /audit/                               deterministic checks

    price.ts, duplicates.ts, quantity.ts,

    package-overlap.ts, unexplained.ts, savings.ts, insurance-coverage.ts

  /rag/policy.ts                        chunking + full-text search retrieval

  /db/queries.ts                        typed query helpers

/supabase

  /migrations/*.sql

  /seed/reference_items.sql             curated NPPA/NLEM demo data

/components

  UploadCard, ProcessingStatus, FindingCard, EvidencePanel, QuestionList

plan.md

```

  

`/lib/embeddings/` intentionally does not exist yet — added only if §D's upgrade path is triggered.

  

---

  

## J. Implementation sequence (hospital workflow first)

  

**M0 — Scaffolding, auth, deploy skeleton** (~1.5h)

Files: Next.js init, Tailwind, Supabase project, `.env`, `/lib/supabase/*`, login/signup pages, empty `/dashboard`.

Output: deployed empty app, working login.

Test: sign up, log in, session persists, deployed on Vercel.

  

**M1 — Schema + seed data** (~1h)

Files: `/supabase/migrations`, `/supabase/seed/reference_items.sql` (~20-30 curated medicines/procedures, labeled as demo-curated).

Output: tables + RLS live, seed data queryable.

Test: RLS blocks cross-user access (verify with two test accounts).

  

**M2 — Upload flow** (~1h)

Files: `/api/cases`, `/api/documents`, `UploadCard`.

Output: authenticated user uploads a PDF/image to a private bucket, row created.

Test: uploaded file is not publicly fetchable without a signed URL.

  

**M3 — Groq provider + extraction pipeline** (~2h)

Files: `/lib/llm/provider.ts` (Groq implementation), `/lib/llm/schemas.ts`, `/api/documents/[id]/process` (extraction half).

Output: given a document, get back validated structured JSON (doc_type + line items).

Test: run against one synthetic hospital estimate and one synthetic prescription, inspect JSON by hand.

  

**M4 — Deterministic audit engine** (~2h)

Files: `/lib/audit/*.ts` (hospital-side checks only: price, quantity, duplicate, package overlap, unexplained, medicine savings).

Output: audit_findings rows from extracted_items + reference_items.

Test: fixture with a known duplicate/known overcharge fires; clean fixture doesn't.

  

**M5 — Explanation, questions, findings UI** (~2h)

Files: reasoning half of `/api/documents/[id]/process`, `/case/[id]/page.tsx`, `FindingCard`, `EvidencePanel`, `QuestionList`.

Output: **Workflow A complete end-to-end** — upload → findings with evidence/confidence/source → generated questions.

Test: full Demo 1 flow works against a synthetic hospital estimate, in the browser, on the deployed URL.

  

--- checkpoint: hospital workflow demo-ready before starting insurance ---

  

**M6 — Insurance extraction + policy summary** (~1.5h)

Files: policy extraction (reuses M3 pipeline with a policy schema), `insurance-coverage.ts` audit checks, insurance summary section of `/case/[id]/page.tsx`.

Output: upload policy → get coverage summary (sum insured, room-rent limit, co-pay, exclusions, etc.) with evidence.

Test: run against a synthetic policy, verify extracted limits match the source document by hand.

  

**M7 — Policy retrieval (full-text) + Q&A + estimate comparison** (~1.5h)

Files: `/lib/rag/policy.ts` (chunk-by-section on ingest, `tsvector` search), `/api/insurance/:caseId/ask`, `/api/insurance/:caseId/compare`.

Output: **Workflow B complete end-to-end** — ask a free-text question, get an answer with cited page/section; upload estimate + policy, get a patient-cost breakdown with the "why" line items.

Test: full Demo 2 flow against a synthetic policy + estimate pair; verify at least 3 varied questions return correctly cited answers.

  

**M8 — Polish + deploy hardening** (~1.5h)

Files: landing page, loading-stage UI, responsive pass, case deletion, error states.

Output: both demos work from a cold start on the deployed Vercel URL.

Test: full run-through of both demos on the live URL, on mobile viewport.

  

**Buffer** (~1h).

  

**Stretch, only if all of the above is done with time left:** upgrade `policy_chunks` retrieval to embeddings/pgvector per §D's upgrade path. Not scheduled into the day-1 budget.

  

Total scheduled: ~12h + buffer.

  

---

  

## K. Risk assessment (updated)

  

**1. Vision-LLM extraction accuracy on real-world document layouts.**

*Fallback:* unchanged — rehearse against 2-3 known-good synthetic documents; keep "confidence: low" / "insufficient information" paths genuinely reachable in the UI.

  

**2. Serverless execution time** on the synchronous `/process` route.

*Fallback:* unchanged — Groq is already the fast option here (that's Groq's main selling point), so this risk is lower than in the OpenRouter version of the plan. If still tight, split extraction and explanation into two client-sequenced calls rather than building a queue.

  

**3. Groq free-tier rate limits during the live demo**, especially on vision-capable models, which tend to have tighter limits than text models.

*Fallback:* rehearse the exact demo flow beforehand; know Groq's current rate limit for the specific vision model in use before demo day; have the OpenAI key path ready as a same-day fallback since the provider abstraction (§C) makes it a config swap, not a rewrite.

  

---

  

## What I need from you to start

  

1. Groq API key.

2. Supabase project created (URL + anon key + service role key).

  

Once you share those (as env vars, not pasted in chat), M0 starts.

  

---

  

## L. Medicine Price Discovery

  

**Status: approved direction; documentation only. No implementation has started.**

  

### Goal

  

Evolve medicine checking from the current exact-name + single-reference-price approach into canonical medicine identity plus multiple structured price references.

  

The goal is to make the existing medicine comparison substantially more useful while preserving the current **Potential savings to investigate** value proposition.

  

### Approved architecture

  

```

Hospital bill

    |

Structured medicine extraction

    |

Medicine identity resolver

    |

Canonical medicine

    |

  +----------------------+----------------------+

  | NPPA ceiling price   | Jan Aushadhi/PMBI    |

  |                      | listed MRP           |

  +----------------------+----------------------+

    |

Unit / pack compatibility

    |

Deterministic comparison

    |

Finding + source-specific evidence

```

  

### MVP sources

  

- NPPA official ceiling prices.

- Jan Aushadhi / PMBI listed MRP data.

  

#### NPPA

  

Retain the existing reviewed 65-row dataset. Preserve, per row:

  

- SO number;

- SO date;

- source URL;

- published unit;

- raw source price information;

- retrieval/source metadata.

  

NPPA remains the primary reference for the current MVP. An NPPA ceiling price must not be described as a hospital tariff, market price, or guaranteed recoverable amount.

  

#### Jan Aushadhi / PMBI

  

Ingest an official structured PMBI snapshot. Preserve:

  

- product ID;

- drug code;

- generic name;

- unit size;

- MRP;

- therapeutic group;

- retrieval timestamp;

- raw/source information.

  

Do not invent effective dates when the source does not provide them. Treat PMBI MRP separately from NPPA ceiling price, with distinct labels and semantics in the UI.

  

Initially import only products that can be matched safely to the canonical medicine catalog. Do not force the entire PMBI dataset into matches.

  

### Medicine identity

  

Resolve, where explicitly available in the bill:

  

- brand name;

- active ingredient(s) / salt;

- strength;

- dosage form;

- route;

- combination components;

- pack text;

- pack quantity and unit.

  

Use structured LLM extraction followed by deterministic normalization and matching. Do not infer a unit that is not present in the bill. If the bill says “Ceftriaxone 1g injection” but does not establish whether the price is per vial, ml, or another unit, keep the comparison unavailable.

  

Normalize mechanically:

  

- 1 g -> 1000 mg;

- tab -> tablet;

- cap -> capsule;

- inj -> injection;

- casing, spacing, and punctuation.

  

Never let normalization collapse meaningful differences:

  

- 500 mg vs 1 g;

- tablet vs injection;

- per ml vs per vial;

- single ingredient vs combination;

- one tablet vs a 10-tablet pack.

  

### Matching hierarchy

  

Use this order:

  

1. exact canonical identity;

2. reviewed alias;

3. constrained fuzzy candidate generation;

4. unresolved.

  

Fuzzy matching must only generate candidates. It must never silently approve a medicine when strength, dosage form, combination contents, or pack/unit differs.

  

If uncertain, surface **Medicine details could not be verified** rather than producing a potentially wrong price comparison.

  

### Combination medicines

  

Combination medicines must fail closed. For example, amoxicillin 500 mg + clavulanic acid 125 mg must never match plain amoxicillin 500 mg.

  

Require an exact component-set match, including strength and dosage form. Never match a combination to one of its individual ingredients.

  

### Unit safety

  

Incompatible or insufficiently evidenced units must produce `unit_unverified` / unresolved behavior. They must not generate a price difference or potential-savings calculation.

  

For a valid comparison, require:

  

- exact ingredient(s);

- exact strength;

- exact dosage form/route where relevant;

- exact combination contents;

- compatible pack/sale unit;

- sufficient bill evidence.

  

### Database architecture

  

Keep `reference_items` for backward compatibility and non-medicine references. It must not become the permanent multi-source medicine model.

  

Add canonical medicine/product, component, alias, and multi-source price-observation models:

  

- `medicine_products`;

- `medicine_product_components`;

- `medicine_aliases`;

- `medicine_price_observations`.

  

Add medicine identity and match fields to `extracted_items`. Keep the existing 65 NPPA rows during the transition; do not delete or break the current data.

  

### Comparison behavior

  

Show NPPA and PMBI separately with their correct semantics. For example:

  

```

Ceftriaxone 1g Injection

  

Hospital charge       Rs 480 / vial

NPPA ceiling price    Rs 63.88 / vial

Jan Aushadhi MRP      Rs XX / pack

Potential difference  Rs XXX

```

  

Do not combine NPPA and PMBI differences into one number. Keep the existing case-level aggregate:

  

**Potential savings to investigate: Rs X**

  

Prefer NPPA as the primary source for that aggregate in the MVP.

  

Keep the existing disclaimer:

  

> Estimated from the available reference price. This does not guarantee that this amount is recoverable or that the hospital charge is unlawful.

  

### Patient-facing framing

  

Findings should clearly distinguish:

  

- Hospital charge;

- NPPA ceiling price;

- Jan Aushadhi MRP;

- potential price difference;

- source date or timestamp where relevant;

- source links and provenance.

  

Retain **Potential savings to investigate** and the existing soft, question-generating framing. Never imply that a price difference proves unlawful billing or guarantees recoverability.

  

### Explicitly out of scope

  

#### RAG

  

RAG is out of scope. Medicine pricing is structured identity plus structured lookup plus deterministic comparison. The existing policy RAG solves a different problem and remains separate.

  

#### Embeddings and pgvector

  

Embeddings and `pgvector` are out of scope for medicine pricing.

  

#### Live online pharmacy scraping

  

Live online pharmacy scraping is out of scope for the MVP. Do not scrape online pharmacies or use search snippets as price data. Do not scrape Tata 1mg, Apollo, PharmEasy, Netmeds, or similar sites.

  

There is no new backend service or unnecessary npm package in the approved MVP direction. In a later Phase 2, consider legitimate/licensed online-price APIs or other structured providers if the product progresses.

  

### Proposed implementation sequence

  

1. Identity extraction and normalization.

2. Identity regression fixtures.

3. Canonical medicine/product/component/alias tables.

4. Link existing NPPA rows.

5. Import a reviewed PMBI snapshot.

6. Replace exact `normalized_name` matching with the resolver.

7. Add multi-source audit evidence.

8. Update the finding UI.

9. Add unit-safety regression tests.

10. Add refresh tooling later.

  

### Critical regression cases

  

- Paracetamol 500 mg != 650 mg.

- Ceftriaxone 500 mg != 1 g.

- tablet != injection.

- single ingredient != combination.

- explicit vial != unspecified injection.

- per-ml != per-vial.

- compatible unit -> comparison allowed.

- incompatible or unknown unit -> no savings.

  

## M. Multilingual support — approved Phase 1 plan

  

### Decision

  

Proceed with **Phase 1: Hindi + English policy Q&A**.

  

The MVP will demonstrate the highest-value multilingual workflow:

  

```text

Hindi / Hinglish question

  -> compact English retrieval terms

  -> existing policy full-text retrieval

  -> grounded English policy excerpts + deterministic facts

  -> Hindi or English explanation

  -> unchanged page and section citations

```

  

Do not localize the entire application shell before this workflow is reliable. Do not add full

automatic language detection to the MVP. The user explicitly selects `English` or `हिंदी`; language

detection can be considered later for convenience, but must not become a hidden source of retrieval

or answer-language bugs.

  

### Language contract

  

Keep these values separate throughout the request and response flow:

  

```text

original_question = user's Hindi, Hinglish, or English input

retrieval_query   = compact English search terms for PostgreSQL FTS

retrieved_chunks  = original English policy text with page/section provenance

answer_language   = selected `en` or `hi`

citations         = page/section labels from retrieved chunks, unchanged

```

  

The original question must remain available for debugging and answer generation. Retrieval terms

must not replace it.

  

### Retrieval rewrite

  

Add a small, structured rewrite step before `searchPolicyChunks`:

  

- accept Hindi, Hinglish, mixed-language, and English input;

- return only compact English keywords or noun phrases, not a translated full sentence;

- preserve policy concepts and domain terms such as `room rent`, `ICU`, `co-pay`, `deductible`,

  `waiting period`, `exclusion`, and `sub-limit`;

- never produce numeric policy facts or perform calculations;

- fall back to the original question if rewriting fails;

- keep the rewrite observable in server logs or a debug-safe internal result without logging

  sensitive policy content unnecessarily.

  

Example:

  

```text

Meri policy mein room rent ka limit kya hai?

  -> room rent limit accommodation eligibility

```

  

The rewrite is solely for recall against the existing English policy chunks. It must not alter

canonical medicine/procedure identities, extracted policy facts, calculations, or evidence.

  

### Grounded answer generation

  

Extend the policy Q&A contract so the model receives the original question, retrieved English

excerpts, deterministic policy facts, and the selected answer language. The model must:

  

- answer only from retrieved excerpts and precomputed facts;

- answer in `answer_language`;

- preserve all amounts, percentages, limits, and durations exactly;

- never translate or reinterpret citation labels;

- return the existing `basis`, `confidence`, and citation structure;

- continue to refuse confidently when the evidence is insufficient.

  

The English policy clause remains the source of truth. A future UI enhancement may expose the

original clause beside the localized explanation, but the first implementation only needs to keep

the existing citation behavior intact.

  

### UI scope for Phase 1

  

- Add an `English | हिंदी` selector to the policy Q&A experience.

- Send `answer_language` with every policy question request.

- Localize the policy-question heading, input placeholder, button, loading state, errors, basis

  labels, confidence labels, and suggested questions.

- Keep the rest of the app shell in English for this phase.

- Support Hinglish input even when `हिंदी` is selected.

  

### Verification and acceptance criteria

  

Add fixtures and tests for:

  

- English input retrieving the same policy chunk as today;

- Hindi input such as `मेरी पॉलिसी में रूम रेंट की सीमा क्या है?`;

- Hinglish input such as `room rent limit kitna hai`;

- compact rewrite output rather than a verbose translated sentence;

- answer language differing from retrieval language;

- unchanged page/section citations;

- exact preservation of rupee amounts, percentages, and durations;

- safe refusal for an unanswerable question;

- rewrite failure falling back without breaking existing English Q&A.

  

The end-to-end demo is successful when a user can ask in Hindi or Hinglish, retrieve the English

policy clause, and receive a Hindi answer with the same citation.

  

### Deferred phases

  

**Phase 2 — localized policy results:** localize coverage summaries, finding explanations, and

suggested hospital/insurer questions while keeping the structured result and evidence canonical.

  

**Phase 3 — additional Indian languages:** extend the same locale and retrieval contracts to

regional languages after Hindi evaluation shows reliable retrieval, grounding, and terminology.

  

Full automatic language detection and whole-app shell localization remain deferred until they are

demonstrably useful.