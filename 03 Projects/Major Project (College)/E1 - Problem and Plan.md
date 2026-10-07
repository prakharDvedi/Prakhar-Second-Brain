---
type: project
area: college
status: drafted
created: 2026-10-07
---

# E1: problem and plan

Gate: panel accepts the problem, the model and the plan. Target readiness: after build Day 2. Index: [[00 Index]].

## 1. Title

Age-of-Information, Rate-Limit and Fault-Detection Analysis of a Scheduled Data Pipeline

## 2. Abstract (about 170 words)

Automated analytics systems depend on chains of scheduled jobs in which each stage consumes data produced by the previous one. A stage can fail, be throttled by an upstream API, or receive a stale or corrupted feed, so downstream consumers silently act on older or wrong data than intended. This project models such a pipeline as periodic update processes and measures freshness with the age-of-information metric. Mean age under periodic updates with geometric failures is derived and checked against a discrete-event simulation. An upstream rate limiter is modelled as a token bucket and retry policies are compared, including fallbacks whose failures are correlated through a shared quota. A small detection stage addresses corrupted input: a run-length test for stale values and a Hampel filter for spikes, evaluated by injecting faults of known type into public price histories and reporting ROC, false alarms per day and delay. The problem classes come from pipelines encountered during an internship on a data-driven trading platform; all parameters are modelled as swept ranges. No claim is made about trading performance.

## 3. Problem statement

For a chain of periodic jobs with random failure, delay, rate-limit loss and corrupted input, find (a) the age of data seen by each consumer and the probability it exceeds a freshness limit, (b) how retry and fallback policies change both, and (c) how reliably stale and impulsive samples in an input price stream can be detected, at a controlled false-alarm rate.

## 4. Motivation

- Downstream jobs often use whatever data exists, with no sign of how old it is.
- Latency is not enough: data can be fast to deliver and still old because updates were lost.
- Retry and fallback are usually designed by intuition. Independence of failures is assumed and rarely checked.
- Corrupted samples (stale, spiky, wrong scale) look like valid numbers to a consumer.
- Standard tools for all of this already exist in communications and reliability theory. This project applies them to a data pipeline and checks them quantitatively.

**Link to internship (one paragraph, generic).** During an internship on a data-driven trading platform I worked with scheduled data pipelines that went stale, hit upstream rate limits and received corrupted prices. This project studies those problem classes formally. It is a model-based study. It does not describe or reproduce any production system.

## 5. Objectives

1. Model a chain of dependent periodic jobs and define freshness with age of information.
2. Derive mean age and the probability that age exceeds a limit, and validate against simulation.
3. Model an upstream rate limiter as a token bucket and compare retry and pacing policies.
4. Quantify how a shared-quota fallback breaks the independence assumption.
5. Build and evaluate stale-value and spike detectors using fault injection on public price data.

## 6. Scope

**In.** Age-of-information model, token bucket, retry policies, fallback reliability, stale and Hampel detectors, simulation, public price data.

**Out.** Any trading logic, hedge ratios, backtests, asset ranking, machine learning, hardware, real company data, real incident numbers.

## 7. System model

Block diagram (Fig F1):

```
Timer (period T)
  -> Job 1 (producer) --calls--> [Rate limiter: token bucket (r, b)]
          |  rejected -> retry policy (none | fixed | backoff+jitter | AIMD)
  -> Store (result, generation time, TTL)
  -> Job 2 (consumer/producer), reads at fixed gap g after Job 1
          |  missing or older than L -> fallback tier (independent | shared quota)
  -> Job 3 ...
  -> Metrics: age at read, P(age > L), goodput, completion time

Input stream path:
Price feed -> fault injector -> [stale run-length | Hampel] -> flags
  -> ROC, false alarms/day, delay
```

Assumptions: independent failure per attempt with probability p unless stated; constant processing delay D; consumers read at a fixed clock time; time is continuous in the analysis and discrete-event in simulation.

## 8. Parameters (all assumed and swept, none observed)

| Symbol | Meaning | Sweep |
|---|---|---|
| T | Job period | 24 h (also 1 h, 6 h) |
| D | Processing/delivery delay | 0.1, 1 h |
| p | Failure probability per attempt | 0.01, 0.05, 0.1, 0.2 |
| g | Gap between producer and consumer read | 2, 3, 6 h |
| L | Freshness limit | 1.25g to 3T |
| r | Limiter sustained rate | 20 to 200 calls/min |
| b | Bucket depth | 10, 50, 100 |
| N | Parallel burst size | 50, 100, 200 |
| beta | Common-cause fraction | 0, 0.1, 0.3, 0.5, 1 |
| q | Per-sample price-change probability (data-derived) | measured from public data |
| k | Stale run length threshold | derived from false-alarm target |
| w, c | Hampel window and multiplier | 5 to 21, 2.5 to 4 |

## 9. Derivations (E1 must contain these on paper)

**(1) Age.** Delta(t) = t - u(t), where u(t) is the generation time of the freshest delivered update.

**(2) Mean age, periodic updates, geometric failure.** Attempts every T succeed with q = 1 - p, independently. Inter-delivery time Y = T G with G geometric on {1, 2, ...}: E[G] = 1/q, E[G^2] = (1+p)/q^2. After each delivery the age resets to D and grows with slope 1 for a time Y. Area per cycle = D Y + Y^2/2. Mean age = D + E[Y^2] / (2 E[Y]) = D + T(1+p) / (2(1-p)). At p = 0 this is D + T/2.

**(3) Fixed-time read.** The consumer reads a gap g after the producer slot. If the last k slots failed, age at read = g + kT. P(k >= m) = p^m. P(age > L) = P(g + kT > L) = p^m with m = floor((L-g)/T) + 1 for L > g. Example: g = 3 h, T = 24 h, L = 30 h gives m = 2 and P = p^2.

**(4) Token bucket burst.** Bucket depth b full, N calls arrive at once, rejected calls are not queued. Accepted = min(N, b), fraction min(1, b/N) (plus any tokens refilled during the burst). With ideal pacing at rate r, completion time is about (N - b)/r.

**(5) Common-cause failure.** Each tier fails with probability p. A fraction beta of a failure is shared. P(both fail) is approximately beta p + ((1-beta) p)^2 for small p. At beta = 0 it is p^2, at beta = 1 it is p.

**(6) Zero-failure bound.** After n trials with no failure, the one-sided upper confidence bound on p at level alpha is 1 - alpha^(1/n). For n = 6, alpha = 0.05 it is about 0.39.

**(7) Stale test.** If the price changes at each sample with probability q, P(k identical in a row) = (1-q)^k. Expected false alarms per day is about (samples/day) x (1-q)^k, ignoring run overlap. Choose k for a target.

**(8) Hampel.** Flag x_t if abs(x_t - median_w) > c x 1.4826 x MAD_w. The factor makes MAD consistent with sigma for Gaussian data. Median and MAD have a 50% breakdown point.

## 10. Methodology summary

1. Implement a SimPy simulator of the chain and validate against (2) and (3).
2. Add the limiter and policies, compare success, time and retry amplification.
3. Add shared-quota fallback, compare with (5).
4. Build the injector and detectors on public price data, tune on one half, test on different fault parameters in the other half.
5. Report everything with confidence intervals from repeated runs.

## 11. Experiments (full specs in E2 and E3)

| # | Experiment | Stage |
|---|---|---|
| 1 | Analytic vs simulated mean age | E2 |
| 2 | P(age > L) vs p for the chain | E2 |
| 3 | Token bucket and retry policies | E3 |
| 4 | Fallback independent vs beta-factor | E3 |
| 5 | Zero-failure evidence | E3 |
| 6 | Stale and Hampel detection | E3 |

## 12. Data

- Public price history from two public exchange APIs [verify limits, coverage, terms before Day 10].
- Synthetic faults injected and labelled synthetic.
- No company, user or production data.

## 13. Timeline

Days 1 to 2: permissions, parameter table, derivations (this document). Days 3 to 11: build and experiments. Days 12 to 14: figures, demo, draft report. See [[E2 - Model Validated]] onward.

## 14. Risks

| Risk | Mitigation |
|---|---|
| "This is software engineering" | Derivations in report, standard network and communications models, results checked against theory |
| Public data limits | Fall back to a shorter history, synthetic stream for the detection chapter if needed |
| Simulation looks arbitrary | Every parameter swept, sensitivity plots, analytic checks |
| Over-claiming | Do-not-claim list in [[E4 - Final Report and Demo]] |

## 15. E1 deliverable checklist

- [ ] Abstract, problem statement, objectives (this note)
- [ ] F1 block diagram drawn cleanly
- [ ] F2 age sawtooth sketch
- [ ] Derivations (2), (3), (5), (6) written by hand and checked
- [ ] Parameter table accepted
- [ ] Scope cuts stated
- [ ] Permission status noted in [[00 Index]]
- [ ] 6 to 8 slides from this note
- [ ] Three questions rehearsed: why is this ECE, what is age of information vs latency, where do the parameters come from
