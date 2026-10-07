---
type: project
area: college
status: target
created: 2026-10-07
---

# E4: final report, demo and viva

Gate: complete report, working demo, rehearsed viva. Previous: [[E3 - Policies and Detection]]. Index: [[00 Index]].

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
| 1 | Introduction | Motivation, problem, objectives, scope, internship context paragraph | 4 | [[E1 - Problem and Plan]] |
| 2 | Background | Age of information, token bucket and AIMD, reliability and common cause, robust detection | 8 | references |
| 3 | System model | Chain, parameters (swept), assumptions, F1 | 5 | E1 |
| 4 | Analytical model | Derivations (1) to (8) with worked examples | 8 | E1 |
| 5 | Simulator and validation | Design, Exp 1 and 2, F2 to F4, pytest checks | 7 | [[E2 - Model Validated]] |
| 6 | Policy experiments | Exp 3 to 5, F5 to F9 | 10 | [[E3 - Policies and Detection]] |
| 7 | Feed fault detection | Injector, detectors, Exp 6, F10 to F12 | 9 | E3 |
| 8 | Discussion and limits | What was shown, what was assumed, what was not tested | 4 | all |
| 9 | Conclusion and future work | Short | 2 | all |
| A | Appendices | Derivations in full, parameter table, injector spec, code listing, reproducibility | 8 | repo |

Total about 65 pages. Adjust to the department's template.

## Front and back matter

Title page, certificate, declaration, acknowledgements, abstract, contents, lists of figures, tables and symbols, references.

## Results summary table (fill from E2, E3)

| Question | Result | Figure |
|---|---|---|
| Does the simulator match theory? | | F3 |
| Where does the freshness limit matter? | | F4 |
| Which retry policy is robust to an unknown limit? | | F6, F7 |
| How much does a shared quota cost? | | F8 |
| What does a small clean trial certify? | | F9 |
| Which detector handles which fault? | | F10 to F12 |

## Demo script (about 8 minutes)

1. (1 min) Problem in one slide, F1.
2. (2 min) Notebook: age sawtooth for a stage. Move the failure-rate slider, analytic mean age line moves with the simulated one.
3. (2 min) Move the limiter and policy controls, show completion time and amplification change.
4. (1 min) Toggle shared quota, show failure probability jump.
5. (2 min) Replay a price stream, inject a fault live, detectors flag it, running precision and recall shown.

Backup: pre-rendered screenshots (F13) and a recorded run in case the notebook fails.

## Reproducibility checklist

- [ ] README with environment and one command per figure
- [ ] Fixed seeds
- [ ] Data download script for public data, no data committed if large
- [ ] pytest passes
- [ ] Parameter table in the repo matches the report

## Do NOT claim

- Any real system's numbers or behaviour. This is a model.
- Trading performance of any kind.
- That injected-fault scores are real-world accuracy.
- That six clean trials prove reliability.
- That independent failures hold in practice.
- That age of information is original work.

## Viva prep: ten questions

1. Why is this ECE and not software? Standard communications and reliability models, derived and checked numerically.
2. Why age and not latency? Age grows between updates, latency does not capture lost updates.
3. Derive mean age. D + E[Y^2]/(2E[Y]), geometric Y.
4. How do you know the simulator is right? Matches (2) and (3), pytest checks, CIs.
5. Where do parameters come from? Assumed and swept. None observed, none from a real system.
6. Why a token bucket? Standard rate-limiter model, burst and sustained rate.
7. Is retry safe? Amplification and storms, jitter and AIMD reduce them.
8. Why does a shared quota break p1 p2? Common-cause failure, equation (5).
9. Why median and MAD? Breakdown point 50%, one outlier does not hide itself.
10. Is this derived from a real system? It is motivated by problem classes met in an internship. Internal details cannot be shared, the model is generic by design.

## Final-week checklist

- [ ] Every figure regenerated from the repo
- [ ] Every equation checked numerically
- [ ] No names or numbers from any real system in text, figures, filenames or code comments
- [ ] Synthetic labels on every fault-injection figure
- [ ] Citations verified
- [ ] Plagiarism check
- [ ] Demo rehearsed twice, backup ready
