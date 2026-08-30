---
type: knowledge
domain: AI & LLM Systems
confidence: deep
last_reviewed: 2026-08-30
---

# RAG & Retrieval

## In Simple Terms

Retrieval is the part of an AI system that decides which source material the model is allowed to see. The useful lesson from my projects is that retrieval is not automatically an embeddings problem: the retrieval method should fit the data and be measured against real questions.

## What I Understand

I have used three different retrieval shapes:

- eVakeel2.0 uses a local legal corpus, InLegalBERT embeddings, a generated vector store, and Redis caching.
- MedBud stores page/section-aware policy chunks in PostgreSQL and uses full-text search, keeping the original question separate from compact retrieval terms and preserving page citations.
- Pear's knowledge base uses a committed embedding artifact and a dual-maintenance workflow: update the source document, rebuild the artifact, and run retrieval evaluation.

Document Intelligence also used sentence-transformer relevance scoring over extracted PDF sections. These are related systems, but not interchangeable: semantic retrieval is useful for paraphrases, while structured medicine pricing in MedBud is deliberately a lookup problem, not RAG.

## Why It Matters

Retrieval errors become confident answer errors. A system can have a good model and still answer incorrectly if it retrieves the wrong chunk, uses a stale index, loses source location, or silently searches with overly strict semantics.

## How I Learned This

- [[eVakeel2.0]]
- [[MediBuddy]]
- [[document-intelligence-system]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-17 – 08-20 — The agent claimed a live product feature "doesn't exist"]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue]]
- [[Medbud/Status]]
- [[Medbud/Plans]]

## Real Failure Modes I've Seen

- A stale Pear knowledge document continued to describe triggers as Hyperliquid-only after Lighter became a supported venue; the document and embedding artifact both had to be corrected.
- In MedBud, raw `websearch_to_tsquery` ANDed words in a multi-topic question and made recall near-zero for many questions. The query builder was changed to OR semantics.
- A generated vector artifact or committed retrieval index can become inert or stale if the source document is changed without rebuilding it.
- eVakeel's repository documents a vector store and cache, but also records that some routes and deployment files are incomplete; retrieval components should not be assumed to mean the whole product is production-ready.

## Engineering Tradeoffs

- PostgreSQL full-text search is simpler and easier to explain than embeddings for section-headed insurance policies; embeddings are a measured upgrade path, not a default.
- Local vectors are easy to ship and inspect, but require explicit ingestion and refresh discipline.
- Caching retrieval results reduces repeated work, but stale-cache behavior becomes part of correctness.
- Page/section provenance can matter more than a slightly better semantic match when the user needs to verify the source.

## Patterns / Rules I Currently Believe

**Strong evidence:** Retrieval changes should be evaluated with a known query set, and any generated artifact must be rebuilt with its source change. Evidence and citations should travel with the retrieved text.

**Current belief:** Start with the simplest retrieval method that can satisfy the actual corpus, then upgrade only when measured misses justify the added model and index complexity.

**Open question:** I have not yet documented production-scale retrieval monitoring, chunk-size experiments, or reranking behavior.

## Examples From My Work

- Pear KB evaluation held at retrieval @1/@3/@5 = 97/100/100 after Lighter changes; new Lighter queries ranked first.
- MedBud keeps English policy clauses as the canonical retrieved evidence even when the answer is generated in Hindi or Hinglish.
- eVakeel2.0 calls a Python embedding service on a cache miss, compares against `vectors.json`, and caches retrieval results in Redis.

## Things I Still Don't Fully Understand

- How retrieval quality changes with corpus growth and chunking choices.
- When to introduce a reranker or hybrid lexical/semantic retrieval.
- How to operate and migrate a vector index safely at larger scale.

## Related

- [[Structured Outputs & Validation]]
- [[LLM Reliability & Grounding]]
- [[LLM Observability & Evaluation]]
- [[Redis & Caching]]
- [[PostgreSQL & Data Modeling]]
