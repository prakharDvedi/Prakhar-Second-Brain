---
type: project
area: college
status: active
created: 2026-10-09
---

# Major Project (ECE version): index

ECE reframe of the college set. Same maths as [[00 Index]], new setting: a wireless sensor status-update network instead of a data pipeline. The old set is untouched and stays as the fallback. Pick one before the first evaluation.

**Title.** Age-of-Information, Duty-Cycle and Fault-Detection Analysis of a Wireless Sensor Status-Update Network

**One line.** Sensor nodes send periodic readings over a lossy radio link under a duty-cycle limit. I measure how old the data at the monitor is, how retry and redundancy policies change that, and how well simple detectors catch faulty sensor readings.

## What changed from the first set

| First set | ECE set |
|---|---|
| Chain of scheduled jobs | Sensor node, relay, gateway/monitor |
| API call fails with prob p | Packet lost on a noisy or fading channel, p computed from SNR |
| Upstream rate limit | Duty-cycle / airtime limit, modelled as a token bucket |
| Retry, backoff, AIMD | ARQ, backoff with jitter, AIMD pacing |
| Shared-quota fallback | Redundant links with a common cause (interference, shared backhaul) |
| Price feed corruption | Sensor faults: stuck-at, spike, dropout, gain error |
| Public exchange price data | Public sensor dataset, or synthetic signal |
| Internship link and NDA items | Removed, nothing proprietary involved |

Equations (1) to (8) are unchanged. One new equation (0) links SNR to p.

## Status board

| Stage | Doc | Gate | Status |
|---|---|---|---|
| E1 | [[E1 - Problem and Plan (ECE)]] | Problem, model, derivation, plan | drafted, needs review |
| E2 | [[E2 - Model Validated (ECE)]] | Channel to p, simulator matches theory | target defined |
| E3 | [[E3 - Policies and Detection (ECE)]] | Duty cycle, redundancy, detection results | target defined |
| E4 | [[E4 - Final Report and Demo (ECE)]] | Report, demo, viva | target defined |

Evaluation dates: unknown. Fill in here: E1 ____ E2 ____ E3 ____ E4 ____

## Open before starting

- [ ] Guide's OK on the topic and the ECE framing
- [ ] Confirm whether MATLAB is expected (Octave or Python otherwise)
- [ ] Verify the public sensor dataset: access, size, licence (Intel Berkeley Research Lab temperature logs is the first candidate)
- [ ] Verify the duty-cycle figure to quote (1% in the EU 868 MHz sub-band is the usual example) against the regulation text

## Ground rules

1. Every number in a result is derived, simulated or from the public dataset.
2. Every parameter is a symbol or a swept range, tagged "assumed (swept)".
3. Synthetic data (simulated losses, injected faults) is labelled synthetic on every figure.
4. Link-level numbers (BER, PER) come from stated textbook formulas, not from measuring a real radio.
5. Code and data live in the repo below, not in this vault.

## Repo layout (code, outside the vault)

```
major-project/
  README.md                  how to reproduce every figure
  src/
    channel.py               BER and PER vs SNR (AWGN, Rayleigh), p from SNR
    age_model.py             analytic mean age, P(age > L)
    sim_network.py           SimPy sensor, relay, monitor
    token_bucket.py          duty-cycle limiter + retry policies
    reliability.py           redundant link, beta-factor, zero-failure bound
    detectors.py             stale run-length, Hampel
    inject.py                sensor fault injector
    metrics.py               ROC, FA/day, delay
  experiments/
    exp0_channel_to_p.py
    exp1_age_validation.py
    exp2_freshness_two_hop.py
    exp3_duty_cycle_policies.py
    exp4_redundant_link.py
    exp5_zero_failure.py
    exp6_sensor_fault_detection.py
  data/                      public sensor data only (gitignored if large)
  figures/                   F1.. generated, numbered as below
  tests/                     pytest: formula checks, detector sanity
  report/                    LaTeX or docx sources
```

## Figure register

| Fig | Content | Stage |
|---|---|---|
| F1 | System block diagram | E1 |
| F2 | PER vs SNR, AWGN and Rayleigh, several packet lengths | E2 |
| F3 | Age sawtooth, one link, with and without loss | E1 sketch, E2 final |
| F4 | Analytic vs simulated mean age vs p, several T | E2 |
| F5 | P(age > L) vs p for the two-hop path, several read gaps | E2 |
| F6 | Token bucket success vs burst size N and depth b | E3 |
| F7 | Completion time and success per retry policy vs rate r | E3 |
| F8 | Retry amplification (packets sent / packets needed) per policy | E3 |
| F9 | Two-link failure probability, independent vs beta-factor | E3 |
| F10 | Zero-failure upper bound vs number of trials | E3 |
| F11 | Detector ROC per fault type | E3 |
| F12 | False alarms per day vs threshold | E3 |
| F13 | Detection delay vs fault size | E3 |
| F14 | Demo screenshot set | E4 |

## Equation register

| Eq | Statement | Where derived |
|---|---|---|
| (0) | Link loss: BER from SNR, p = 1 - (1 - BER)^Lp | E1, checked in E2 |
| (1) | Age: Delta(t) = t - u(t) | E1 |
| (2) | Mean age, periodic, geometric loss: D + T(1+p)/(2(1-p)) | E1 |
| (3) | Fixed-time read: P(age > L) = p^m, m = floor((L-g)/T) + 1 | E1 |
| (4) | Token bucket accepted fraction for a burst: min(1, b/N) | E1 |
| (5) | Beta-factor: P_sys ~ beta p + ((1-beta) p)^2 | E1 |
| (6) | Zero-failure bound: 1 - alpha^(1/n) | E1 |
| (7) | Stale test: false alarms ~ samples/day x (1-q)^k | E1 |
| (8) | Hampel: flag if abs(x - med) > c x 1.4826 x MAD | E1 |

## References to collect (verify details before citing)

- Kaul, Yates, Gruteser, "Real-time status: how often should one update?", INFOCOM 2012
- Yates et al., "Age of Information: An Introduction and Survey", IEEE JSAC 2021
- Proakis, Digital Communications (BER of BPSK in AWGN and Rayleigh fading)
- Bertsekas and Gallager, Data Networks (token bucket, ALOHA, ARQ)
- Abramson, ALOHA system, 1970 (only if the collision stretch is added)
- Regulation or standard text for sub-GHz duty cycle limits (ETSI EN 300 220) [verify]
- Hampel, robust estimation, 1974, and a modern MAD outlier reference
- A reliability engineering text on common-cause failure and the beta-factor
- Dataset reference for the Intel Berkeley Research Lab sensor data [verify]
- A SimPy reference
