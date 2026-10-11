---
type: project
area: college
status: drafted
created: 2026-10-09
---

# E1: problem and plan (ECE version)

Gate: panel accepts the problem, the model and the plan. Target readiness: after build Day 2. Index: [[00 Index (ECE)]]. Original non-ECE version: [[E1 - Problem and Plan]].

## 1. Title

Age-of-Information, Duty-Cycle and Fault-Detection Analysis of a Wireless Sensor Status-Update Network

## 2. Abstract (about 170 words)

Wireless sensor networks report periodic readings to a monitor over lossy radio links. A packet can be lost to noise or fading, a transmitter can be held back by a regulatory duty-cycle limit, and a sensor can itself produce stuck or spiked readings, so the monitor may act on older or wrong data than it believes. This project models a sensor, relay and monitor path as periodic update processes and measures freshness with the age-of-information metric. Packet loss probability is computed from SNR for AWGN and Rayleigh channels. Mean age under periodic updates with geometric loss is derived and checked against a discrete-event simulation. The duty-cycle limit is modelled as a token bucket and retry policies are compared, including redundant links whose failures are correlated through a common cause. A small detection stage addresses faulty sensors: a run-length test for stuck-at readings and a Hampel filter for spikes, evaluated by injecting faults of known type into public sensor data and reporting ROC, false alarms per day and delay. All parameters are modelled as swept ranges. No claim is made about any deployed network.

## 3. Problem statement

For a sensor node reporting through a lossy, duty-cycle-limited radio path with possible sensor faults, find (a) the age of the data at the monitor and the probability it exceeds a freshness limit, (b) how retry and redundancy policies change both, and (c) how reliably stuck and impulsive readings can be detected, at a controlled false-alarm rate.

## 4. Motivation

- A monitor often uses the last value it received, with no sign of how old it is.
- Latency is not enough: a packet can be delivered quickly and the data still be old because earlier packets were lost.
- Retry and redundancy are usually set by intuition. Independent losses are assumed and rarely checked, yet two links can share interference or a backhaul.
- Duty-cycle limits on licence-free bands cap how much a node may transmit, so recovering from loss has a cost.
- Faulty readings (stuck, spiked, wrong gain) look like valid numbers to a consumer.
- The tools already exist in communications and reliability theory. This project applies them to a sensor status-update path and checks them quantitatively.

## 5. Objectives

1. Compute packet loss probability from SNR for AWGN and Rayleigh channels.
2. Model a sensor, relay and monitor path as periodic updates and define freshness with age of information.
3. Derive mean age and the probability that age exceeds a limit, and validate against simulation.
4. Model a duty-cycle limit as a token bucket and compare retry and pacing policies.
5. Quantify how a common cause between redundant links breaks the independence assumption.
6. Build and evaluate stuck-at and spike detectors using fault injection on public sensor data.

## 6. Scope

**In.** Channel to loss model, age-of-information model, token bucket, retry policies, redundant link reliability, stale and Hampel detectors, simulation, public sensor data.

**Out.** Hardware, real radio measurements, protocol stack implementation, machine learning, energy optimisation, any proprietary data.

## 7. System model

Block diagram (Fig F1):

```
Sensor (sample every T)
  -> Transmitter --[channel: SNR -> BER -> loss p]--> Relay
          |  rejected by duty-cycle limiter (token bucket r, b)
          |    -> retry policy (none | fixed | backoff+jitter | AIMD)
  -> Relay store (reading, generation time)
  -> Relay forwards at fixed gap g after the sensor slot --[channel]--> Monitor
          |  missing or older than L -> use redundant link (independent | common cause)
  -> Metrics: age at read, P(age > L), goodput, completion time

Sensor data path:
Sensor stream -> fault injector -> [stale run-length | Hampel] -> flags
  -> ROC, false alarms/day, delay
```

Assumptions: independent loss per attempt with probability p unless stated; constant airtime and processing delay D; the monitor reads at a fixed clock time; time is continuous in the analysis and discrete-event in simulation; one hop is analysed in closed form, the two-hop path is simulated.

## 8. Parameters (all assumed and swept, none measured)

| Symbol | Meaning | Sweep |
|---|---|---|
| T | Sampling period | 10 s, 60 s, 600 s |
| D | Airtime plus processing delay | 0.1, 1 s |
| gamma | Mean SNR per bit (Eb/N0) | swept to cover p from 0.01 to 0.2 |
| Lp | Packet length | 128, 256, 1024 bits |
| p | Loss probability per attempt | 0.01, 0.05, 0.1, 0.2 (from gamma and Lp, or set directly) |
| g | Gap between sensor slot and read | 0.1T, 0.25T, 0.5T |
| L | Freshness limit | 1.25g to 3T |
| d, tau | Duty cycle and airtime per packet | d = 1% as the example, r = d / tau |
| r | Limiter sustained rate | 1 to 20 packets/min |
| b | Bucket depth | 5, 20, 50 |
| N | Nodes reporting in one burst (event trigger) | 20, 50, 100 |
| beta | Common-cause fraction between redundant links | 0, 0.1, 0.3, 0.5, 1 |
| q | Per-sample probability the reading changes (data-derived) | measured from the public dataset |
| k | Stale run length threshold | derived from false-alarm target |
| w, c | Hampel window and multiplier | 5 to 21, 2.5 to 4 |
| delta | ADC step (floor for the Hampel scale) | from dataset resolution |

## 9. Derivations (E1 must contain these on paper)

**(0) Link loss.** BPSK in AWGN: BER = Q(sqrt(2 gamma)). BPSK in flat Rayleigh fading: BER = 0.5 (1 - sqrt(gamma / (1 + gamma))). A packet of Lp bits without coding is lost if any bit is wrong: p = 1 - (1 - BER)^Lp. This gives the sweep link between SNR and p. Uncoded and independent bit errors are the stated assumptions.

**(1) Age.** Delta(t) = t - u(t), where u(t) is the generation time of the freshest delivered update.

**(2) Mean age, periodic updates, geometric loss.** Attempts every T succeed with q' = 1 - p, independently. Inter-delivery time Y = T G with G geometric on {1, 2, ...}: E[G] = 1/q', E[G^2] = (1+p)/q'^2. After each delivery the age resets to D and grows with slope 1 for a time Y. Area per cycle = D Y + Y^2/2. Mean age = D + E[Y^2] / (2 E[Y]) = D + T(1+p) / (2(1-p)). At p = 0 this is D + T/2.

**(3) Fixed-time read.** The monitor reads a gap g after the sensor slot. If the last k slots were lost, age at read = g + kT. P(k >= m) = p^m. P(age > L) = P(g + kT > L) = p^m with m = floor((L-g)/T) + 1 for L > g. Example: g = 0.25T, L = 1.25T gives m = 2 and P = p^2.

**(4) Token bucket burst.** Bucket depth b full, N packets arrive at once, rejected packets are not queued. Accepted = min(N, b), fraction min(1, b/N) (plus any tokens refilled during the burst). With ideal pacing at rate r, completion time is about (N - b)/r. The duty-cycle limit gives r = d / tau.

**(5) Common-cause failure.** Each link fails with probability p. A fraction beta of a failure is shared. P(both fail) is approximately beta p + ((1-beta) p)^2 for small p. At beta = 0 it is p^2, at beta = 1 it is p.

**(6) Zero-failure bound.** After n trials with no failure, the one-sided upper confidence bound on p at level alpha is 1 - alpha^(1/n). For n = 6, alpha = 0.05 it is about 0.39.

**(7) Stale test.** If the reading changes at each sample with probability q, P(k identical in a row) = (1-q)^k. Expected false alarms per day is about (samples/day) x (1-q)^k, ignoring run overlap. Choose k for a target. For a quantised sensor q depends on noise sigma against ADC step delta, which is why it is measured from data.

**(8) Hampel.** Flag x_t if abs(x_t - median_w) > c x 1.4826 x MAD_w. The factor makes MAD consistent with sigma for Gaussian data. Median and MAD have a 50% breakdown point.

## 10. Methodology summary

1. Compute p from SNR and packet length (Eq 0) and plot PER vs SNR.
2. Implement a SimPy simulator of the sensor, relay and monitor path and validate against (2) and (3).
3. Add the duty-cycle limiter and policies, compare success, time and retry amplification.
4. Add the redundant link with a common cause, compare with (5).
5. Build the injector and detectors on public sensor data, tune on one half, test on different fault parameters in the other half.
6. Report everything with confidence intervals from repeated runs.

## 11. Experiments (full specs in E2 and E3)

| # | Experiment | Stage |
|---|---|---|
| 0 | SNR to loss probability, AWGN vs Rayleigh | E2 |
| 1 | Analytic vs simulated mean age | E2 |
| 2 | P(age > L) vs p for the two-hop path | E2 |
| 3 | Duty-cycle limiter and retry policies | E3 |
| 4 | Redundant link, independent vs beta-factor | E3 |
| 5 | Zero-failure evidence | E3 |
| 6 | Stuck-at and spike detection | E3 |

## 12. Data

- Public sensor logs, first candidate the Intel Berkeley Research Lab temperature data [verify access, size and licence before Day 10].
- Fallback: a synthetic signal (slow diurnal sine plus AR(1) noise, quantised to an ADC step), labelled synthetic.
- Synthetic faults injected and labelled synthetic.
- No proprietary, company or user data.

## 13. Timeline

Days 1 to 2: parameter table, derivations (this document). Days 3 to 11: build and experiments. Days 12 to 14: figures, demo, draft report. See [[E2 - Model Validated (ECE)]] onward.

## 14. Risks

| Risk | Mitigation |
|---|---|
| "This is just a simulation" | Closed-form derivations, results checked against theory, standard communications models |
| "Where is the radio part" | Eq 0 and Exp 0 tie p to SNR, modulation and packet length, and the limiter is a real regulatory rule |
| Public data limits | Shorter history, or the synthetic signal for the detection chapter |
| Simulation looks arbitrary | Every parameter swept, sensitivity plots, analytic checks |
| Over-claiming | Do-not-claim list in [[E4 - Final Report and Demo (ECE)]] |

## 15. E1 deliverable checklist

- [ ] Abstract, problem statement, objectives (this note)
- [ ] F1 block diagram drawn cleanly
- [ ] F3 age sawtooth sketch
- [ ] Derivations (0), (2), (3), (5), (6) written by hand and checked
- [ ] Parameter table accepted
- [ ] Scope cuts stated
- [ ] Guide's OK on the topic noted in [[00 Index (ECE)]]
- [ ] 6 to 8 slides from this note
- [ ] Three questions rehearsed: why is this ECE, what is age of information vs latency, where do the parameters come from
