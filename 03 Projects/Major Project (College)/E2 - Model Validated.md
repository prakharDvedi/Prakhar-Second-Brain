---
type: project
area: college
status: target
created: 2026-10-07
---

# E2: model validated

Gate: the simulator matches the theory, and the chain freshness results exist. Target readiness: after build Day 6. Previous: [[E1 - Problem and Plan]]. Index: [[00 Index]].

## Deliverables

- [ ] `age_model.py`: analytic mean age and P(age > L)
- [ ] `sim_pipeline.py`: SimPy chain of periodic jobs
- [ ] Exp 1 and Exp 2 scripts, figures F2 to F4
- [ ] Results tables below filled
- [ ] Report chapters 3 to 5 drafted (see [[E4 - Final Report and Demo]])
- [ ] 8 to 10 slides

## Simulator spec

**Entities.** Timer, Job (producer), Store, Consumer.

**Events.** Slot start, attempt success or fail (prob p), delivery after delay D, consumer read at fixed gap g.

**State.** Store holds the generation time of the freshest delivered update.

**Outputs per run.** Age at each consumer read, time-average age of the store, fraction of reads with age > L.

**Run length.** At least 100,000 producer slots per setting, 30 seeds, report mean and 95% CI.

**Determinism.** Seeded RNG, each figure reproducible with one command.

## Exp 1: analytic vs simulated mean age

- **Hypothesis.** Simulated time-average age equals D + T(1+p)/(2(1-p)) within the confidence interval.
- **Input.** Single stage, geometric failure.
- **Vary.** p in {0.01, 0.05, 0.1, 0.2, 0.4}, T in {1, 6, 24} h, D in {0.1, 1} h.
- **Metric.** Relative error and CI coverage.
- **Figure F3.** Analytic curve with simulated points and CI bars.
- **Interpretation.** Agreement validates the simulator, so later policy results are credible.
- **Acceptance.** Relative error under 1% for at least 90% of settings, otherwise find the bug before moving on.

| p | T | D | Analytic | Simulated mean | 95% CI | Rel. error |
|---|---|---|---|---|---|---|
| | | | | | | |

## Exp 2: freshness of the chain

- **Hypothesis.** P(age > L) at a fixed-time read equals p^m, and the limit L only matters in steps of T.
- **Input.** Three-job chain, consumer reads at gap g after each producer.
- **Vary.** p, g in {2, 3, 6} h, L from 1.25g to 3T.
- **Metric.** P(age > L), mean age at read, fraction of reads falling back.
- **Figures.** F2 (age sawtooth, with and without failure), F4 (P(age > L) vs p for several g).
- **Interpretation.** Show the step structure: L between g and g + T has the same exceedance probability as any L in that interval, so a tighter limit there buys nothing. State what this implies for choosing L.
- **Acceptance.** Simulated P matches p^m within CI for all settings.

| p | g | L | m | Analytic p^m | Simulated | 95% CI |
|---|---|---|---|---|---|---|
| | | | | | | |

## Code checks (pytest)

- p = 0 gives mean age D + T/2
- p = 0.5 matches D + 1.5T
- Seeded run reproduces exactly
- P(age > L) is monotone in p and non-increasing in L

## E2 slide outline

1. Recap of problem and model (1 slide)
2. Derivation of mean age (1 slide)
3. Simulator design (1 slide)
4. Exp 1 result F3 (1 slide)
5. Exp 2 result F4 and the step observation (2 slides)
6. What is next (1 slide)

## E2 viva prep

- Why age and not latency?
- Derive mean age.
- How do you know the simulator is right?
- What are the assumptions behind independent failure?
- What happens if failures are correlated across days?
