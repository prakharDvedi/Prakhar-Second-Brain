---
type: project
area: college
status: target
created: 2026-10-09
---

# E2: model validated (ECE version)

Gate: the channel model gives p, the simulator matches the theory, and the freshness results exist. Target readiness: after build Day 6. Previous: [[E1 - Problem and Plan (ECE)]]. Index: [[00 Index (ECE)]]. Original: [[E2 - Model Validated]].

## Deliverables

- [ ] `channel.py`: BER and PER vs SNR
- [ ] `age_model.py`: analytic mean age and P(age > L)
- [ ] `sim_network.py`: SimPy sensor, relay, monitor
- [ ] Exp 0, 1 and 2 scripts, figures F2 to F5
- [ ] Results tables below filled
- [ ] Report chapters 3 to 5 drafted (see [[E4 - Final Report and Demo (ECE)]])
- [ ] 8 to 10 slides

## Exp 0: SNR to packet loss

- **Hypothesis.** A simulated bit-error count over many packets matches the closed forms in Eq 0 for AWGN and Rayleigh, and Rayleigh needs far more SNR for the same p.
- **Input.** BPSK bits through AWGN, and through flat Rayleigh fading, uncoded.
- **Vary.** Eb/N0 from 0 to 30 dB, Lp in {128, 256, 1024} bits.
- **Metric.** Simulated BER and PER vs formula, and the SNR needed for p in {0.01, 0.05, 0.1, 0.2}.
- **Figure F2.** PER vs SNR, curves and simulated points.
- **Interpretation.** Gives the SNR meaning of each swept p, which is the link between the radio and the age results.
- **Acceptance.** Simulated BER within the confidence interval of the formula for all SNR points with enough errors counted.

| Channel | Lp | Eb/N0 | BER formula | BER sim | PER formula | PER sim |
|---|---|---|---|---|---|---|
| | | | | | | |

## Simulator spec

**Entities.** Timer, Sensor (producer), Channel, Relay store, Monitor.

**Events.** Slot start, transmission success or loss (prob p), delivery after delay D, relay forward at fixed gap g, monitor read.

**State.** Each store holds the generation time of the freshest delivered reading.

**Outputs per run.** Age at each monitor read, time-average age, fraction of reads with age > L.

**Run length.** At least 100,000 sensor slots per setting, 30 seeds, report mean and 95% CI.

**Determinism.** Seeded RNG, each figure reproducible with one command.

## Exp 1: analytic vs simulated mean age

- **Hypothesis.** Simulated time-average age equals D + T(1+p)/(2(1-p)) within the confidence interval.
- **Input.** Single link, geometric loss.
- **Vary.** p in {0.01, 0.05, 0.1, 0.2, 0.4}, T in {10, 60, 600} s, D in {0.1, 1} s.
- **Metric.** Relative error and CI coverage.
- **Figure F4.** Analytic curve with simulated points and CI bars.
- **Interpretation.** Agreement validates the simulator, so later policy results are credible.
- **Acceptance.** Relative error under 1% for at least 90% of settings, otherwise find the bug before moving on.

| p | T | D | Analytic | Simulated mean | 95% CI | Rel. error |
|---|---|---|---|---|---|---|
| | | | | | | |

## Exp 2: freshness of the two-hop path

- **Hypothesis.** For one hop, P(age > L) at a fixed-time read equals p^m and L only matters in steps of T. The two-hop path is worse than one hop, and the simulation shows by how much.
- **Input.** Sensor to relay to monitor, relay forwards its latest stored reading at gap g after the sensor slot. One-hop case is checked against Eq 3. Two-hop case is simulation only.
- **Vary.** p per hop, g in {0.1T, 0.25T, 0.5T}, L from 1.25g to 3T.
- **Metric.** P(age > L), mean age at read, fraction of reads served from a stale relay copy.
- **Figures.** F3 (age sawtooth, with and without loss), F5 (P(age > L) vs p for several g).
- **Interpretation.** Show the step structure: L between g and g + T has the same exceedance probability as any L in that interval, so a tighter limit there buys nothing. State what this implies for choosing L.
- **Acceptance.** One-hop simulated P matches p^m within CI for all settings.

| p | g | L | m | Analytic p^m | Simulated | 95% CI |
|---|---|---|---|---|---|---|
| | | | | | | |

## Code checks (pytest)

- p = 0 gives mean age D + T/2
- p = 0.5 matches D + 1.5T
- BER formula at a known point (for example gamma = 0 dB) matches a hand value
- Seeded run reproduces exactly
- P(age > L) is monotone in p and non-increasing in L

## E2 slide outline

1. Recap of problem and model (1 slide)
2. SNR to loss, Eq 0 and F2 (1 slide)
3. Derivation of mean age (1 slide)
4. Simulator design (1 slide)
5. Exp 1 result F4 (1 slide)
6. Exp 2 result F5 and the step observation (2 slides)
7. What is next (1 slide)

## E2 viva prep

- Why age and not latency?
- Derive mean age.
- Where does p come from, and what is assumed (uncoded BPSK, independent bit errors)?
- How do you know the simulator is right?
- What happens if losses are correlated in time, for example a fade lasting several slots?
