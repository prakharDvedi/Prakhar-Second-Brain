---
type: project
area: college
status: planning
created: 2026-10-06
---

# Major Project: combined plan (pipeline freshness and reliability + feed fault detection)

Decision: keep the internship work as the spine, add one small signal-processing layer. No equaliser, no hardware, no hedge-ratio or trading work. Parent analysis: [[Major Project - ECE Direction]] (this is Project 2 plus a slice of Project 3).

Evidence tags: **[built]** vault shows I did it, **[consumed]** used but not written by me, **[assumption]** unverified, **[new]** to be written for the project.

## Title

*Age-of-Information, Rate-Limit and Fault-Detection Analysis of a Scheduled Market-Data Pipeline*

Plain version: how old is the data when the next job uses it, what rate limits, retries and fallbacks do to that, and how to catch bad input data.

## Abstract (about 170 words)

Automated trading and analytics systems depend on chains of scheduled jobs in which each stage consumes data produced by the previous one. A stage can fail, be throttled by an upstream API, or receive a stale or corrupted feed, so downstream consumers silently act on older or wrong data than intended. This project models such a pipeline as periodic update processes and measures freshness with the age-of-information metric. Mean age under periodic updates with geometric failures is derived and checked against a discrete-event simulation. An upstream rate limiter is modelled as a token bucket and retry policies are compared, including fallbacks whose failures are correlated through a shared quota. A small detection stage addresses corrupted input: a run-length test for stale values and a Hampel filter for spikes, evaluated by injecting faults of known type into public price histories and reporting ROC, false alarms per day and delay. The schedule, timeouts, freshness rule and fault types come from a real daily pipeline encountered during an internship on a crypto derivatives trading agent; parameters that were not measured are stated as assumptions. No claim is made about trading performance.

## Problem statement

For a chain of periodic jobs with random failure, delay, rate-limit loss and corrupted input, find the age of data seen by each consumer, the probability it exceeds a freshness limit, how retry and fallback policies change that, and how reliably bad input samples can be detected.

## Objectives

1. Model the real schedule (03:00 OI sync, 10:00 ranking, 12:00 sentiment, 14:00 baskets) as a chain of update processes.
2. Derive mean age and P(age > limit), verify by simulation.
3. Model the upstream limit as a token bucket and compare retry and pacing policies.
4. Quantify fallback failure under independent vs shared-quota (common-cause) failure.
5. Build and evaluate stale-value and spike detectors with injected faults.

## Where each piece comes from

| Piece | Source in my work | Tag |
|---|---|---|
| Schedule, ranking key (48h TTL, ignored after 36h), first-day "missing or stale" at 14:00 | Familiar-asset ranking job, Hetzner timers | [built] |
| Fallback off a rate-limited provider, injected-429 trial 6/6, worst 29.8s vs 30s tier budget | #923 | [built] |
| Transaction timeout: 5s default vs ~30ms path | #1034 | [built] |
| OI sync 28 Sep: 77/188 ok, 111 failed all 429 | Step 3 note | [built] observation, cause [assumption] |
| Stale price cache (merged old prices while its timestamp advanced) | Lighter bring-up, 18 to 20 Aug | [built] |
| Scale-prefix mix-up (`1000PEPE` vs `kPEPE`), wrong-venue price | #481, #557 | [built] |
| Detectors, simulator, analysis | none | [new] |

## Model and equations

- Age at consumer: Delta(t) = t - u(t), u = generation time of the freshest delivered update.
- Periodic updates, period T, constant delay D, failure probability p per attempt: mean age = D + T(1+p) / (2(1-p)). Derive from E[Y^2]/(2E[Y]) with geometric inter-delivery time. At p = 0 it reduces to D + T/2.
- Consumer reading at a fixed clock time: age at read = 4h + 24h x k for k consecutive producer failures. With 48h TTL and a 36h rule, k = 0 gives 4h, k = 1 gives 28h, k = 2 gives 52h, past both limits. P(fall back) = p^2 for independent daily failures. **[derived from the notes, verify against the real timer and consumer code]**. Consequence to test: on this schedule the 36h rule and the TTL trip at the same k, so the 36h rule only matters for off-schedule reruns.
- Token bucket: rate r, depth b. A 50-parallel burst against a smaller bucket is shed. Sweep r around the assumed ~80 req/min.
- Fallback: P(system fail) = p1 p2 if independent. Beta-factor model for common cause: fraction beta of each failure is shared.
- Timeout identity: N sequential round trips of RTT fail a timeout T_out when N x RTT > T_out. At RTT 30ms and 5s, N above about 167 **[arithmetic only, real N unknown]**.
- Zero-failure bound: 0 failures in n = 6 trials gives a 95% upper bound of 1 - 0.05^(1/6), about 39%.
- Stale test: if a price changes at each sample with probability q, P(k identical in a row) = (1-q)^k. Choose k for a false-alarm target per day.
- Hampel: flag if |x_t - median_w| > c x 1.4826 x MAD_w.

## Architecture

```
Schedule (timers)
  -> Stage 1 producer (OI sync) --API calls--> [Token bucket (r, b)]
        |-- 429 -> retry policy (none | fixed | exp+jitter | AIMD)
  -> Store (key, TTL, timestamp)
  -> Stage 2/3 consumers (ranking -> sentiment -> baskets)
        |-- missing or stale -> fallback tier (independent | shared quota)
  -> Metrics: age at read, P(age > limit), goodput, completion time

Separate small path:
Public price feed -> fault injector -> [stale run-length | Hampel] -> flags
  -> ROC, false alarms/day, delay
```

## Experiments

1. Analytic vs simulated mean age over p and T (validation).
2. P(age > 36h) vs p on the real schedule, with a rerun-time sweep.
3. Token bucket: 50-parallel vs paced vs AIMD vs backoff with jitter, cap swept around 80/min. Success, completion time, retry amplification.
4. Fallback failure, independent vs beta-factor.
5. Zero-failure evidence: why 6/6 cannot certify reliability and how simulation extends it under stated assumptions.
6. Detection: ROC of stale and Hampel detectors on injected faults (type, size, duration swept), false alarms per day on a clean month, delay vs fault size.

## Data

- Schedule, timeouts, incident numbers from my own notes, only what Pear permits publishing. No user data. Do not touch the BE or quant DBs.
- Public price history, Hyperliquid and Binance **[assumption: verify limits, coverage, terms]**.
- Injected faults are synthetic and labelled as such on every result.

## Stack

Python, SimPy, numpy, scipy, pandas, matplotlib, pytest, Jupyter.

## Report chapters

1. Introduction and internship context
2. Background: age of information, rate limiting, reliability, robust detection
3. The real pipeline and what was observed (observed vs assumed table)
4. Analytical model
5. Simulator and validation
6. Policy experiments (retry, rate limit, fallback)
7. Feed fault detection
8. Discussion and limits
9. Conclusion
Appendices: derivations, parameter table with source tags, code, injector spec

## Demonstration

A live notebook: age sawtooth per stage with sliders for failure rate, cap and policy, analytic mean age overlaid on the simulated one. Then a price stream with a fault injected, the detector lighting up, running ROC.

## Do NOT claim

- The 80 req/min cause of the 28 Sep 429s as confirmed.
- 3h to 1h speedup (#796) as measured.
- That 6/6 recovered proves reliability.
- That injected-fault scores are real-world accuracy.
- That I invented age of information.
- Any trading performance.

## Two-week build order

| Day | Work | Output |
|---|---|---|
| 1 | Ask Pear what can be published. List observed vs assumed parameters from the vault. Repo and SimPy setup | Parameter table v1, permission answer |
| 2 | Derive mean age on paper. Write the model section | Derivation page |
| 3 to 4 | Simulator core: periodic producer, failure p, consumer at fixed time. Validate against the formula | Exp 1 plot, sawtooth plot |
| 5 to 6 | Real schedule chain, k-failure ages, P(fall back) = p^2, 36h vs TTL check against the actual timer and consumer code (read only) | Exp 2 plot, note on the 36h rule |
| 7 to 8 | Token bucket and policies, cap sweep, 50-parallel burst | Exp 3 plots |
| 9 | Fallback independent vs beta-factor, zero-failure bound | Exp 4 and 5 |
| 10 to 11 | Pull public price data, fault injector, stale and Hampel detectors, ROC, false alarms per day | Exp 6 plots |
| 12 | Consolidate figures and tables, tidy parameter provenance | Figure set |
| 13 to 14 | Demo notebook, report chapters 1 to 4 drafted, buffer for slips | Demo, draft |

Two weeks gets the results and a demo. The full report continues into the later evaluations.

## Evaluation plan (E1 to E4)

Evaluation dates are unknown, so these are relative. Map them onto the real calendar once known.

**E1: problem and plan.** Show: title, abstract, problem statement, objectives, block diagram, the mean-age derivation, parameter table with observed vs assumed tags, scope cuts, Pear permission status. Ready after Day 2.

**E2: model validated.** Show: working simulator, analytic vs simulated mean age (Exp 1), real-schedule freshness results (Exp 2), the 36h rule finding. Ready after Day 6.

**E3: policies and detection.** Show: token-bucket and retry comparison (Exp 3), fallback reliability (Exp 4 and 5), detection ROC (Exp 6), limits stated. Ready after Day 11.

**E4: final.** Show: complete report, live demo, results table, viva prep (do-not-claim list, observed vs assumed table), clean code and reproducibility notes.
