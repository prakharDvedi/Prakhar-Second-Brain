---
type: project
area: college
status: target
created: 2026-10-09
---

# E4: final report, demo and viva (ECE version)

Gate: complete report, working demo, rehearsed viva. Previous: [[E3 - Policies and Detection (ECE)]]. Index: [[00 Index (ECE)]]. Original: [[E4 - Final Report and Demo]].

## Deliverables

- [ ] Final report (chapters below)
- [ ] Live demo notebook
- [ ] Reproducible repo, one command per figure
- [ ] Results summary table
- [ ] Viva prep sheet
- [ ] 15 to 20 slides

## Report structure and page targets

| Ch | Title | Content | Pages | Source |
|---|---|---|---|---|
| 1 | Introduction | Motivation, problem, objectives, scope | 4 | [[E1 - Problem and Plan (ECE)]] |
| 2 | Background | Digital modulation and fading, age of information, token bucket and ARQ, reliability and common cause, robust detection | 9 | references |
| 3 | System model | Sensor, relay, monitor, parameters (swept), assumptions, F1 | 5 | E1 |
| 4 | Analytical model | Derivations (0) to (8) with worked examples | 8 | E1 |
| 5 | Channel, simulator and validation | Exp 0 to 2, F2 to F5, pytest checks | 8 | [[E2 - Model Validated (ECE)]] |
| 6 | Policy experiments | Exp 3 to 5, F6 to F10 | 10 | [[E3 - Policies and Detection (ECE)]] |
| 7 | Sensor fault detection | Injector, detectors, Exp 6, F11 to F13 | 9 | E3 |
| 8 | Discussion and limits | What was shown, what was assumed, what was not tested | 4 | all |
| 9 | Conclusion and future work | Short | 2 | all |
| A | Appendices | Derivations in full, parameter table, injector spec, code listing, reproducibility | 8 | repo |

Total about 67 pages. Adjust to the department's template.

## Front and back matter

Title page, certificate, declaration, acknowledgements, abstract, contents, lists of figures, tables and symbols, references.

## Results summary table (fill from E2, E3)

| Question | Result | Figure |
|---|---|---|
| What SNR does each loss probability p correspond to? | | F2 |
| Does the simulator match theory? | | F4 |
| Where does the freshness limit matter? | | F5 |
| Which retry policy is robust to an unknown duty-cycle limit? | | F7, F8 |
| How much does a common cause cost a redundant link? | | F9 |
| What does a small clean trial certify? | | F10 |
| Which detector handles which sensor fault? | | F11 to F13 |

## Demo script (about 8 minutes)

1. (1 min) Problem in one slide, F1.
2. (1 min) SNR slider: show PER vs SNR and the resulting p, F2.
3. (2 min) Notebook: age sawtooth for a link. Move the loss slider, analytic mean age line moves with the simulated one.
4. (2 min) Move the duty-cycle and policy controls, show completion time and amplification change.
5. (1 min) Toggle the common cause, show two-link failure probability jump.
6. (1 min) Replay a sensor stream, inject a fault live, detectors flag it, running precision and recall shown.

Backup: pre-rendered screenshots (F14) and a recorded run in case the notebook fails. The existing artifact page can be reworked to this ECE setting for the live demo.

## Reproducibility checklist

- [ ] README with environment and one command per figure
- [ ] Fixed seeds
- [ ] Data download script for public data, no data committed if large
- [ ] pytest passes
- [ ] Parameter table in the repo matches the report

## Do NOT claim

- Measurements from a real radio or deployed network.
- That formula-based p matches a real coded, interleaved link.
- That injected-fault scores are real-world accuracy.
- That six clean trials prove reliability.
- That independent losses hold in practice.
- That age of information is original work.

## Viva prep: ten questions

1. Why is this ECE? It uses digital communications (BER, fading, SNR to packet loss), traffic shaping, reliability and detection theory, derived and checked numerically.
2. Why age and not latency? Age grows between updates, latency does not capture lost updates.
3. Derive mean age. D + E[Y^2]/(2E[Y]), geometric Y.
4. How do you know the simulator is right? Matches (2) and (3), pytest checks, CIs.
5. Where do parameters come from? Assumed and swept. p comes from Eq 0 at a stated SNR, none measured on hardware.
6. Why a token bucket? Standard rate-limiter model, and a duty-cycle limit is a real regulatory cap on airtime.
7. Is retry safe? Amplification and collisions, jitter and AIMD reduce them.
8. Why does a common cause break p1 p2? Shared interference or backhaul, equation (5).
9. Why median and MAD? Breakdown point 50%, one outlier does not hide itself.
10. Is the data real? The sensor data is a public dataset. Faults and losses are simulated and labelled synthetic.

## Final-week checklist

- [ ] Every figure regenerated from the repo
- [ ] Every equation checked numerically
- [ ] Synthetic labels on every fault-injection and loss-simulation figure
- [ ] Dataset licence and citation noted
- [ ] Citations verified
- [ ] Plagiarism check
- [ ] Demo rehearsed twice, backup ready
