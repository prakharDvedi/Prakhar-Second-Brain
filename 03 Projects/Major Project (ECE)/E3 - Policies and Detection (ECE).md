---
type: project
area: college
status: target
created: 2026-10-09
---

# E3: policies and detection (ECE version)

Gate: duty-cycle and redundancy results, plus the detection chapter, with limits stated. Target readiness: after build Day 11. Previous: [[E2 - Model Validated (ECE)]]. Index: [[00 Index (ECE)]]. Original: [[E3 - Policies and Detection]].

## Deliverables

- [ ] `token_bucket.py`, `reliability.py`, `detectors.py`, `inject.py`, `metrics.py`
- [ ] Exp 3 to Exp 6 scripts, figures F6 to F13
- [ ] Results tables below filled
- [ ] Report chapters 6 and 7 drafted
- [ ] 10 to 12 slides

## Exp 3: duty-cycle limit and retry policies

- **Hypothesis.** Backoff with jitter or AIMD pacing beats immediate retry under a shared limit, and a burst larger than the bucket loses a large fraction of packets.
- **Input.** N nodes report at once after an event, through a limiter with sustained rate r = d / tau and depth b. Rejected packets are retried per policy.
- **Policies.** No retry, fixed delay, exponential backoff with jitter, AIMD pacing.
- **Vary.** N in {20, 50, 100}, b in {5, 20, 50}, r from 1 to 20 packets/min.
- **Metrics.** Success fraction, completion time, retry amplification (packets sent / packets needed).
- **Figures.** F6 (success vs N, b), F7 (completion time and success per policy vs r), F8 (retry amplification).
- **Interpretation.** Show the trade-off between completion time and load placed on the channel, and which policy is robust when r is unknown (the point of AIMD).
- **Acceptance.** Theory (4) matches the simulated first-burst acceptance.
- **Stretch.** Add shared-channel collisions with slotted ALOHA, where throughput is G e^(-G), to show why immediate retry can collapse goodput. Only if time allows.

| Policy | N | b | r | Success | Completion time | Amplification |
|---|---|---|---|---|---|---|
| | | | | | | |

## Exp 4: redundant link reliability

- **Hypothesis.** A common cause raises the two-link failure probability far above p^2, towards p as beta grows.
- **Input.** Two links (for example two channels or two gateways), each failing with p, common-cause fraction beta from shared interference, power or backhaul.
- **Vary.** p in {0.01 to 0.2}, beta in {0, 0.1, 0.3, 0.5, 1}.
- **Metric.** P(both fail) simulated vs equation (5).
- **Figure F9.** System failure vs beta for several p, log scale.
- **Interpretation.** A second link only helps to the extent its failures are independent of the first. Moving it to a different frequency, antenna position or backhaul lowers beta.

| p | beta | Eq. (5) | Simulated | 95% CI |
|---|---|---|---|---|
| | | | | |

## Exp 5: zero-failure evidence

- **Hypothesis.** A small clean trial bounds reliability only weakly.
- **Metric.** Upper bound 1 - alpha^(1/n) vs n at alpha = 0.05, plus the bound after n_sim simulated trials under stated assumptions.
- **Figure F10.** Bound vs n (log x). Mark n = 6 at about 0.39.
- **Interpretation.** Six clean test transmissions do not certify a link. Simulation extends it only as far as the assumptions hold.

## Exp 6: stuck-at and spike detection

**Data.** Public sensor logs [verify]. One clean stretch per sensor for false-alarm measurement. Split time in half: tune on the first half, test on the second with different fault parameters. Fallback is the synthetic signal from E1 section 12.

**Fault injector (`inject.py`).**

| Type | Parameters swept | Label |
|---|---|---|
| Stuck-at (stale) | run length 3, 5, 10, 30 samples | start and end index |
| Spike | size 2, 5, 10 x local sigma, single and burst of 3 | index |
| Dropout | 1, 5, 20 missing samples | index range |
| Gain error | x10 step for 10 samples | index range |

All injected faults are synthetic. Label every figure "synthetic faults".

**Detectors.**

- Stuck-at run-length: k from the (1-q)^k target per day.
- Hampel: w in {5, 11, 21}, c in {2.5, 3, 4}, scale floor set to the ADC step delta so a frozen window does not give MAD = 0.
- Baseline: mean and standard deviation z-score.

**Metrics.** Precision, recall, false alarms per day on clean data, detection delay in samples, ROC over threshold.

**Figures.** F11 (ROC per fault type), F12 (false alarms/day vs threshold), F13 (delay vs fault size).

**Hypotheses.**

- Hampel keeps recall as spike density grows, where mean/std z-score is masked by its own outliers.
- Stuck-at detection is near-perfect on long runs and weak on short ones, by the geometric tail.
- A gain error is caught by any robust detector, a dropout needs a separate rule.
- q depends on quantisation, so a coarse ADC makes honest repeated readings look stuck and forces a larger k.

| Detector | Fault | Size | Precision | Recall | FA/day | Delay |
|---|---|---|---|---|---|---|
| | | | | | | |

## Limits to state in the report

- Injected faults are synthetic, scores are an optimistic bound.
- Loss probability and limiter parameters are assumed and swept, not measured on a real radio.
- Uncoded BPSK and independent bit errors are assumptions. Coding and interleaving would change p.
- Independence of losses across slots is an assumption. Real fades last several slots.
- A real physical event (a genuine temperature jump) is not a fault and will be flagged. State how neighbouring sensors could separate them, but do not claim it was tested.

## E3 slide outline

1. Recap and E2 result (1)
2. Duty-cycle model and policies (1)
3. Exp 3 results F6 to F8 (2)
4. Redundant link and common cause, F9 (1)
5. Zero-failure bound F10 (1)
6. Detection setup and injector (1)
7. Detection results F11 to F13 (2)
8. Limits (1)
9. Remaining work (1)

## E3 viva prep

- Why a token bucket for a duty-cycle limit?
- Is retry safe? Retry amplification and collision collapse.
- Why does a common cause break the independence formula?
- Why can six clean trials not prove reliability?
- Why median and MAD, what is the breakdown point?
- How is the stuck-at threshold derived, and how does ADC resolution enter?
- Are injected faults circular?
