---
type: project
area: college
status: active
created: 2026-10-07
---

# Major Project (college set): index

Anonymised set for submission. It names no company, provider, venue or internal system and uses no production numbers. All parameters are symbols or swept ranges. The private version with real details is [[Major Project - Combined Plan]]. Do not copy numbers from there into these files.

**Title.** Age-of-Information, Rate-Limit and Fault-Detection Analysis of a Scheduled Data Pipeline

## Status board

| Stage | Doc | Gate | Status |
|---|---|---|---|
| E1 | [[E1 - Problem and Plan]] | Problem, model, derivation, plan | drafted, needs review |
| E2 | [[E2 - Model Validated]] | Simulator matches theory, freshness results | target defined |
| E3 | [[E3 - Policies and Detection]] | Rate limit, fallback, detection results | target defined |
| E4 | [[E4 - Final Report and Demo]] | Report, demo, viva | target defined |

Evaluation dates: unknown. Fill in here: E1 ____ E2 ____ E3 ____ E4 ____

## Open before starting

- [ ] One-line OK from manager: "describe the pipeline generically, no names or numbers"
- [ ] Read offer letter / NDA confidentiality clause
- [ ] Confirm guide's subject area and whether MATLAB is expected
- [ ] Verify public price data access and limits (two public exchange APIs)

## Ground rules

1. Every number in a result is derived, simulated or from public data. None from production.
2. Every parameter is a symbol or a swept range, tagged "assumed (swept)".
3. Synthetic data (fault injection, simulated failures) is labelled synthetic on every figure.
4. No trading performance claim anywhere.
5. Code and data live in the repo below, not in this vault.

## Repo layout (code, outside the vault)

```
major-project/
  README.md                  how to reproduce every figure
  src/
    age_model.py             analytic mean age, P(age > L)
    sim_pipeline.py          SimPy chain of periodic jobs
    token_bucket.py          limiter + retry policies
    reliability.py           fallback, beta-factor, zero-failure bound
    detectors.py             stale run-length, Hampel
    inject.py                fault injector
    metrics.py               ROC, FA/day, delay
  experiments/
    exp1_age_validation.py
    exp2_freshness_chain.py
    exp3_rate_limit_policies.py
    exp4_fallback.py
    exp5_zero_failure.py
    exp6_detection.py
  data/                      public price history only (gitignored if large)
  figures/                   F1.. generated, numbered as below
  tests/                     pytest: formula checks, detector sanity
  report/                    LaTeX or docx sources
```

## Figure register (numbering fixed now, filled by stage)

| Fig | Content | Stage |
|---|---|---|
| F1 | Pipeline block diagram | E1 |
| F2 | Age sawtooth, one stage, with and without failure | E1 sketch, E2 final |
| F3 | Analytic vs simulated mean age vs p, several T | E2 |
| F4 | P(age > L) vs p for the chain, several read gaps | E2 |
| F5 | Token bucket success vs burst size N and depth b | E3 |
| F6 | Completion time and success per retry policy vs limit r | E3 |
| F7 | Retry amplification (calls sent / calls needed) per policy | E3 |
| F8 | System failure probability, independent vs beta-factor | E3 |
| F9 | Zero-failure upper bound vs number of trials | E3 |
| F10 | Detector ROC per fault type | E3 |
| F11 | False alarms per day vs threshold | E3 |
| F12 | Detection delay vs fault size | E3 |
| F13 | Demo screenshot set | E4 |

## Equation register

| Eq | Statement | Where derived |
|---|---|---|
| (1) | Age: Delta(t) = t - u(t) | E1 |
| (2) | Mean age, periodic, geometric failure: D + T(1+p)/(2(1-p)) | E1 |
| (3) | Fixed-time read: P(age > L) = p^m, m = floor((L-g)/T) + 1 | E1 |
| (4) | Token bucket accepted fraction for instantaneous burst: min(1, b/N) | E1 |
| (5) | Beta-factor: P_sys ~ beta p + ((1-beta) p)^2 | E1 |
| (6) | Zero-failure bound: 1 - alpha^(1/n) | E1 |
| (7) | Stale test: false alarms ~ samples/day x (1-q)^k | E1 |
| (8) | Hampel: flag if abs(x - med) > c x 1.4826 x MAD | E1 |

## References to collect (verify details before citing)

- Kaul, Yates, Gruteser, "Real-time status: how often should one update?", INFOCOM 2012
- Yates et al., "Age of Information: An Introduction and Survey", IEEE JSAC 2021
- Hampel, robust estimation, 1974 (and a modern MAD outlier reference)
- Page, CUSUM, 1954 (only if the CUSUM stretch is added)
- A standard text on traffic shaping / token bucket and on reliability engineering (common-cause failure, beta-factor)
- A SimPy reference
