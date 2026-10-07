---
type: project
area: college
status: target
created: 2026-10-07
---

# E3: policies and detection

Gate: rate-limit and fallback results, plus the detection chapter, with limits stated. Target readiness: after build Day 11. Previous: [[E2 - Model Validated]]. Index: [[00 Index]].

## Deliverables

- [ ] `token_bucket.py`, `reliability.py`, `detectors.py`, `inject.py`, `metrics.py`
- [ ] Exp 3 to Exp 6 scripts, figures F5 to F12
- [ ] Results tables below filled
- [ ] Report chapters 6 and 7 drafted
- [ ] 10 to 12 slides

## Exp 3: rate limit and retry policies

- **Hypothesis.** Backoff with jitter or AIMD pacing beats immediate retry under a shared limit, and a burst larger than the bucket loses a large fraction of calls.
- **Input.** N parallel calls against a token bucket (r, b). Rejected calls are retried per policy.
- **Policies.** No retry, fixed delay, exponential backoff with jitter, AIMD pacing.
- **Vary.** N in {50, 100, 200}, b in {10, 50, 100}, r in {20 to 200}/min.
- **Metrics.** Success fraction, completion time, retry amplification (calls sent / calls needed).
- **Figures.** F5 (success vs N, b), F6 (completion time and success per policy vs r), F7 (retry amplification).
- **Interpretation.** Show the trade-off between completion time and load placed on the limiter, and which policy is robust when r is unknown (the point of AIMD).
- **Acceptance.** Theory (4) matches the simulated first-burst acceptance.

| Policy | N | b | r | Success | Completion time | Amplification |
|---|---|---|---|---|---|---|
| | | | | | | |

## Exp 4: fallback reliability

- **Hypothesis.** A shared quota raises system failure probability far above p1 p2, towards p as beta grows.
- **Input.** Two tiers, each failing with p, common-cause fraction beta.
- **Vary.** p in {0.01 to 0.2}, beta in {0, 0.1, 0.3, 0.5, 1}.
- **Metric.** P(both fail) simulated vs equation (5).
- **Figure F8.** System failure vs beta for several p, log scale.
- **Interpretation.** A fallback only helps to the extent its failures are independent of the primary. Moving the fallback to a different upstream resource lowers beta.

| p | beta | Eq. (5) | Simulated | 95% CI |
|---|---|---|---|---|
| | | | | |

## Exp 5: zero-failure evidence

- **Hypothesis.** A small clean trial bounds reliability only weakly.
- **Metric.** Upper bound 1 - alpha^(1/n) vs n at alpha = 0.05, plus the bound after n_sim simulated trials under stated assumptions.
- **Figure F9.** Bound vs n (log x). Mark n = 6 at about 0.39.
- **Interpretation.** Six clean runs do not certify reliability. Simulation extends it only as far as the assumptions hold.

## Exp 6: stale and spike detection

**Data.** Public price history from two public exchange APIs [verify]. One clean month per asset for false-alarm measurement. Split time in half: tune on first half, test on second with different fault parameters.

**Fault injector (`inject.py`).**

| Type | Parameters swept | Label |
|---|---|---|
| Stale | run length 3, 5, 10, 30 samples | start and end index |
| Spike | size 2, 5, 10 x local sigma, single and burst of 3 | index |
| Gap | 1, 5, 20 missing samples | index range |
| Scale | x1000 step for 10 samples | index range |

All injected faults are synthetic. Label every figure "synthetic faults".

**Detectors.**

- Stale run-length: k from (1-q)^k target per day.
- Hampel: w in {5, 11, 21}, c in {2.5, 3, 4}.
- Baseline: mean and standard deviation z-score.

**Metrics.** Precision, recall, false alarms per day on clean data, detection delay in samples, ROC over threshold.

**Figures.** F10 (ROC per fault type), F11 (false alarms/day vs threshold), F12 (delay vs fault size).

**Hypotheses.**

- Hampel keeps recall as spike density grows, where mean/std z-score is masked by its own outliers.
- Stale detection is near-perfect on long runs and weak on short ones, by the geometric tail.
- A scale fault is caught by any robust detector, a gap needs a separate rule.

| Detector | Fault | Size | Precision | Recall | FA/day | Delay |
|---|---|---|---|---|---|---|
| | | | | | | |

## Limits to state in the report

- Injected faults are synthetic, scores are an optimistic bound.
- Failure probability and limiter parameters are assumed and swept.
- Independence of failures across days is an assumption.
- Real market events (flash moves) are not faults and will be flagged. State how multi-source agreement could separate them, but do not claim it was tested.

## E3 slide outline

1. Recap and E2 result (1)
2. Rate limit model and policies (1)
3. Exp 3 results F5 to F7 (2)
4. Fallback and common cause, F8 (1)
5. Zero-failure bound F9 (1)
6. Detection setup and injector (1)
7. Detection results F10 to F12 (2)
8. Limits (1)
9. Remaining work (1)

## E3 viva prep

- Why a token bucket?
- Is retry safe? Retry amplification and storms.
- Why does a shared quota break the independence formula?
- Why can six clean trials not prove reliability?
- Why median and MAD, what is the breakdown point?
- How is the stale threshold derived?
- Are injected faults circular?
