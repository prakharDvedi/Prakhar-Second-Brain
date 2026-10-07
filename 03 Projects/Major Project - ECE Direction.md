---
type: project
area: college
status: planning
created: 2026-10-06
---

# Major Project: ECE direction from the Pear work

Planning note for the B.Tech ECE major project. Built from what the vault actually records about the Pear internship, then translated into ECE terms. Nothing here is a claim about what was built. Where the vault is silent, the note says so.

Evidence tags used below:
- **[built]** the vault shows I wrote or changed it (PR, commit, trace)
- **[consumed]** the system uses it and I debugged around it, but the vault has no evidence I wrote the estimator or algorithm
- **[new]** would be written for the college project
- **[assumption]** not verified, check before relying on it

Source notes: [[My Work at Pear Protocol]], [[Features Owned]], [[Better Trading]], [[Familiar-Asset Ranking — Final Plan]], [[BNB-AVAX Pair Orientation Was Lost at the Agent Boundary]], [[Step 3 — Tokenomist + OI Canaries on Hetzner]], [[1st Oct 2026]].

---

## 1. Honest read of the internship from an ECE angle

What the vault shows, sorted by how ECE-shaped it really is.

| Work in the vault | What it is, honestly | ECE-able? |
|---|---|---|
| Rolling z, hedge ratio k, correlation, cointegration in `pair_stats_*` and `/stats/basket`, used as the quant gate (reject k<0, require a hedged direction) | **[consumed]**. Computed by other services (`pear-stat-rs`, quant DB). The vault has no evidence I wrote the estimators. I used them, built pair-vs-basket selection on top (#1067, closed), and traced orientation bugs | Yes, strongly: estimation, adaptive filtering, detection |
| Spread = price(A) - beta x price(B), z stored in a canonical orientation, reverse regression is "a different model, not a sign-flipped copy" (repo KB, quoted in the BNB/AVAX note) | **[consumed]** concept, **[built]** investigation | Yes: OLS asymmetry, errors-in-variables |
| Schedules, timeouts, retries, fallbacks, rate limits (#923 fallback, #1034 transaction timeout, #796 concurrency, OI 429s, Hetzner timers, ranking key TTL 48h / stale after 36h) | **[built]** mostly, with real numbers | Yes, as networks / queueing / reliability / freshness |
| Data-integrity bugs: price cache that merged old prices while its timestamp advanced, `1000PEPE` vs `kPEPE` scale prefix, price anchored to the wrong venue | **[built]** (fixes) | Yes, as fault detection in sensor-like feeds |
| Margin gate (100bps buffer, fail-closed, reduce-only exempt), leverage clamp doubling margin ($1,000 asked, $1,999.95 resulting) | **[built]** | Weak. A pre-trade threshold check is not a feedback loop. See idea 6 |
| 65/35 ranking, adoption metric, frozen baseline, replay harness, `paper-shadow.ts` | **[built]** | Mostly analytics. Small parts are usable |
| LLM agents, prompts, Telegram, KB retrieval, model routing | **[built]** | No. This is software/AI engineering. Do not dress it up |

On "prices as signals": defensible when a model is stated. A price series sampled hourly is a non-stationary, heavy-tailed stochastic process. It is not "signal plus noise" until I say what the signal is (for example a latent equilibrium level of a spread). Every project below states its model first. Where an idea needs the analogy stretched past that, it is marked weak.

---

## 2. Project ideas

Scores: effort 5 = hardest. ECE defensibility 5 = easiest to defend. Authenticity 5 = closest to work I did.

### Idea 1. Adaptive hedge-ratio tracking and adaptive-threshold excursion detection

- **Titles.** ECE: *Adaptive Estimation and CFAR-Style Detection on Mean-Reverting Spreads of Crypto Perpetual Futures.* Plain: track how two prices relate over time and flag when the gap between them is unusually wide.
- **Core problem.** A fixed rolling window trades lag against noise, assumes the hedge ratio is constant, and a z threshold assumes Gaussian noise. How much do adaptive estimators help, and how wrong is the nominal false-alarm rate?
- **Internship link.** **[consumed]** pair/basket quant gate, rollZ and k in tool output, BNB/AVAX orientation incident, #1075 (stored pairs lose z and hedge when the row is reverse-oriented, filed by me), replay finding that only 1.7% of HL candidate pairs were cointegrated and 23% of attempts had a strong pair but 4% made one.
- **ECE link.** Adaptive filtering (RLS, forgetting factor), Kalman state estimation, system identification, detection theory (ROC, Neyman-Pearson, CA-CFAR), stochastic processes (AR(1)/OU, autocorrelation).
- **Implementation.** In: hourly closes for 10 to 20 perp pairs plus synthetic spreads with known time-varying beta. Pipeline: log prices, align, estimate beta (rolling OLS, RLS, Kalman), form spread and innovation, standardise, threshold. Out: beta trajectories, tracking error, ROC, empirical vs nominal false alarm, detection delay. Python (numpy, scipy, statsmodels, matplotlib). Math: state-space model, Kalman recursion, RLS, Student-t tails.
- **Block diagram.** Full spec in section 4.1.
- **Evaluation.** Beta tracking MSE vs forgetting factor (expect a U shape), lag after a beta step, innovation whiteness (Ljung-Box), spread stationarity and half-life, ROC and Pfa, detection delay.
- **Work split.** Done: nothing in the estimator sense. Now: read the repo's pair-stats code and KB to document exactly what the production definitions are. New: all three estimators, synthetic generator, detector, evaluation.
- **Effort 3. ECE 5. Authenticity 3.**
- **Risk.** "Did you build the stats engine?" (no, say so). "Does it make money?" (not claimed). Panel may say finance. Answer with the model and the measurements.

### Idea 2. Structural-break detection for spread relationships

- **Titles.** ECE: *Sequential Change Detection (CUSUM/GLR) for Breakdown of Pairwise Price Relationships.* Plain: tell quickly when a pair stops behaving like a pair.
- **Core problem.** Detection delay vs false-alarm rate when the relationship changes.
- **Internship link.** **[consumed]** hedge validity gate, 1.7% cointegration rate, stored pairs with no cointegration and |roll z| < 1.
- **ECE link.** Sequential detection, Page's CUSUM, ARL0/ARL1 trade-off, Lorden-style optimality.
- **Implementation.** Residual or innovation stream from Idea 1, CUSUM and GLR on mean and variance shifts, injected breaks of known size. Out: ARL curves, delay vs shift size.
- **Block diagram.** Innovation stream, running statistic, threshold h, alarm, delay measurement.
- **Evaluation.** ARL0 target, delay vs shift, false alarms per month on real data.
- **Work split.** New: everything. Natural stretch chapter for Idea 1.
- **Effort 3. ECE 4. Authenticity 3.**
- **Risk.** Real breaks have no ground truth. Use injected breaks plus a few documented real ones.

### Idea 3. Streaming fault detection in exchange price feeds

- **Titles.** ECE: *Robust Streaming Fault Detection in Exchange Price Feeds using Median Filtering, Redundancy Voting and Sequential Tests.* Plain: catch bad prices (stale, spiky, wrong scale, wrong venue) before the trading logic uses them.
- **Core problem.** Feeds fail quietly. Which cheap detectors catch which faults, at what false-alarm cost?
- **Internship link.** **[built]** fixes for the price store that merged old prices while its freshness timestamp advanced, the `1000PEPEUSDT` vs `kPEPE` scale mapping (#481), and the HL-only price anchoring that led to `execution_venue_price` (#557). The detectors are **[new]**.
- **ECE link.** Robust DSP (median/Hampel filtering, impulsive noise), CUSUM, sensor redundancy and median-of-three voting (TMR), fault injection methodology.
- **Implementation.** Full spec in section 4.3.
- **Evaluation.** Precision, recall, false alarms per day, detection delay, ROC over window and threshold.
- **Work split.** Done: the fault taxonomy from real bugs. New: detectors, injector, evaluation.
- **Effort 3. ECE 4. Authenticity 3.**
- **Risk.** Injected faults are synthetic. Say so on every results slide.

### Idea 4. Reliability and latency model of a rate-limited pipeline with retries and fallbacks

- **Titles.** ECE: *Queueing, Rate-Limiting and Redundancy Analysis of a Multi-Tier Fallback Pipeline.* Plain: how rate limits, retries and backup providers change completion time and failure probability.
- **Core problem.** Naive reliability math assumes independent failures. A fallback on the same quota fails together.
- **Internship link.** **[built]** #923 (composer fallback moved off the rate-limited provider, step-2 stream bug, injected-429 trial: 6/6 recovered, worst 29.8s against a 30s per-tier budget), OI sync on 28 Sep (77/188 ok, 111 failures all 429, started 1s into batch 2). The ~80 req/min cap behind that is **[assumption]**, unconfirmed.
- **ECE link.** Token-bucket shaping, AIMD and backoff (TCP, Ethernet), queueing, series/parallel reliability, common-cause (beta-factor) failure.
- **Implementation.** SimPy discrete-event simulation against a mock rate-limited server (clearly a simulation). Compare no-retry, fixed, exponential backoff with jitter, AIMD pacing. Out: goodput, completion time, failure probability.
- **Block diagram.** Client pool, pacing policy, token-bucket server, failure/429 path, retry scheduler, fallback tier, metrics.
- **Evaluation.** Completion time and success vs policy and load, sensitivity to the assumed cap, independent vs correlated fallback.
- **Work split.** Done: real incidents and numbers. New: model and simulator.
- **Effort 3. ECE 3. Authenticity 5.**
- **Risk.** "This is software." Needs derivations in the report. Folded into Project 2 below rather than standing alone.

### Idea 5. Information freshness (age of information) of a scheduled multi-stage pipeline

- **Titles.** ECE: *Age-of-Information Analysis of a Scheduled, Multi-Stage Market-Data Pipeline.* Plain: how old is the data when the next job uses it, and what happens when a stage fails.
- **Core problem.** Downstream jobs silently use older data than intended, or fall back.
- **Internship link.** **[built]** ranking job (10:00 UTC, key TTL 48h, ignored if older than 36h), sentiment 12:00, baskets 14:00, the first prod day where 14:00 ran before the key existed and logged "missing or stale", plus the price-cache bug where the timestamp advanced while data was stale (the freshness metric itself was wrong).
- **ECE link.** Age of information (Kaul, Yates, Gruteser), renewal processes, delay and loss, scheduling.
- **Implementation.** Full spec in section 4.2.
- **Work split.** Done: schedule, incidents, timings. New: analysis, simulator, policy comparison.
- **Effort 3. ECE 4. Authenticity 4.**
- **Risk.** Panel may ask for communication-channel relevance. Answer: the producer-consumer chain is an update-delivery system, which is the AoI setting.

### Idea 6. Closed-loop leverage and margin control

- **Titles.** ECE: *Volatility-Feedback Leverage Control with Liquidation First-Passage Analysis.* Plain: shrink leverage when risk rises so the account stays away from liquidation.
- **Internship link.** **[built]** confirm-time margin gate (#569), leverage-clamp margin disclosure (#593). Leverage derived from spread volatility already exists in the generator **[consumed]**.
- **ECE link.** EWMA volatility is a first-order IIR low-pass filter, gain scheduling, stochastic first-passage. 
- **Core problem and evaluation.** Keep margin ratio above a limit under random returns. Metric: probability of breaching a limit, leverage lag after a vol jump.
- **Work split.** New: nearly all.
- **Effort 4. ECE 3. Authenticity 3.**
- **Risk.** The margin gate is a threshold check, not a loop. You do not control the market, so the only plant is account equity. "Where is your plant model and transfer function?" is an easy kill. Do not pick this.

### Idea 7. Validating LLM sentiment signals as a detection problem

- **Titles.** ECE: *Hypothesis-Testing Evaluation of Directional Signals against Forward Returns.* Plain: do the sentiment calls beat a coin flip.
- **Internship link.** **[built]** #1058 playground that scores calls against HL price at +1/3/7d. Day 1 logged, no accuracy result. `paper-shadow.ts` 10/10 positive overnight (+0.45% to +6.68%) is preliminary.
- **ECE link.** Binary hypothesis testing, ROC, confidence intervals. Thin.
- **Why weak.** About 17 assets a day for a few days. 60% hit rate on n=50 gives a one-sided binomial p of about 0.10, so nothing is provable. Needs months of data.
- **Effort 2. ECE 3. Authenticity 4.** Use as a small section at most.

### Idea 8. Weighted signal-fusion ranking. **Rejected.**

0.65 x current + 0.35 x historical is a product decision made with Jeyd, not an estimated or optimised weighting. Normalising and summing is analytics. Only "rank stability under noise" (1.9 top-20 swaps a day, no hysteresis, a Schmitt-trigger idea) is usable, and that is one paragraph, not a project.
Effort 2. ECE 2. Authenticity 4.

### Idea 9. ML classification of spread reversion. **Rejected as standalone.**

A classifier on rolling features is a data-science project. Panel will ask what makes it ECE, and the honest answer is nothing unless it sits on a stated signal model. Lookahead bias and tiny effective sample are the usual traps. Can appear as a baseline inside Idea 1.
Effort 3. ECE 2. Authenticity 2.

### Idea 10. Information-theoretic view of signals. **Rejected.**

Mutual information between a z-score and a forward return, or "channel capacity of the market", is the forced analogy the brief warns about. MI estimation on small, autocorrelated samples is biased and hard to defend.
Effort 4. ECE 2. Authenticity 1.

**Also rejected on purpose:** FPGA/Arduino/sensor layers. No experiment in the vault needs hardware, so adding it would be decoration.

---

## 3. Comparison

| Rank | Project | ECE Alignment | Internship Alignment | Implementation Effort | Academic Depth | Viva Defensibility |
|---|---|---|---|---|---|---|
| 1 | Adaptive hedge-ratio tracking + adaptive-threshold detection (Idea 1) | 5 | 3 | 3 | 5 | 5 |
| 2 | Information freshness of a scheduled pipeline (Idea 5, with Idea 4 folded in) | 4 | 4 | 3 | 4 | 4 |
| 3 | Streaming fault detection in price feeds (Idea 3) | 4 | 3 | 3 | 4 | 4 |
| 4 | Rate-limit / retry / fallback reliability (Idea 4) | 3 | 5 | 3 | 3 | 3 |
| 5 | Structural-break detection (Idea 2, stretch for Idea 1) | 4 | 3 | 3 | 4 | 4 |
| 6 | Closed-loop leverage control (Idea 6) | 3 | 3 | 4 | 3 | 3 |
| 7 | Sentiment-signal validation (Idea 7) | 3 | 4 | 2 | 2 | 3 |
| 8 | Weighted signal-fusion ranking (Idea 8) | 2 | 4 | 2 | 1 | 2 |
| 9 | ML classification of reversion (Idea 9) | 2 | 2 | 3 | 2 | 2 |
| 10 | Information-theoretic view (Idea 10) | 2 | 1 | 4 | 3 | 1 |

Effort: 5 = hardest. Everything else: 5 = best.

Top three: **P1 = Idea 1**, **P2 = Idea 5 with Idea 4 inside it**, **P3 = Idea 3**. Idea 2 is a stretch chapter for P1.

---

## 4. The three finalists

### 4.1 Project 1. Adaptive hedge-ratio tracking and adaptive-threshold detection

**Final title.** Adaptive Estimation and CFAR-Style Detection on Mean-Reverting Spreads of Cryptocurrency Perpetual Futures: A Kalman and Recursive Least Squares Study.

**Abstract (about 190 words).**
Pair trading in crypto perpetual futures depends on two quantities: the hedge ratio that combines two prices into a spread, and a rolling z-score that flags when the spread is stretched. In production systems both are usually computed with fixed rolling windows, which trade lag against noise and treat the hedge ratio as constant. This project treats the hedge ratio as a slowly varying state and compares three estimators on the same data: rolling ordinary least squares, recursive least squares with a forgetting factor, and a Kalman filter with a random-walk state model. It then treats the z-score as an adaptive-threshold detector, closely related to cell-averaging CFAR detection in radar, and measures how closely its empirical false-alarm rate follows the nominal Gaussian value on heavy-tailed data. Estimators are first validated on synthetic spreads with known parameters, then evaluated on public hourly Hyperliquid and Binance perpetual price histories using tracking error, spread stationarity, innovation whiteness, detection ROC and detection delay. The work is motivated by orientation and hedge-ratio failure modes met during an internship on a crypto derivatives trading agent. It reports signal-processing performance only and makes no claim of trading profitability.

**Exact problem statement.** Given log prices y_t and x_t of two perpetuals, estimate alpha_t and beta_t in y_t = alpha_t + beta_t x_t + e_t online, and decide at each t whether the spread is in a statistically unusual excursion, such that estimation error is minimised and the false-alarm rate is controlled and measurable.

**Objectives.**
1. Implement rolling OLS, RLS (forgetting factor) and a Kalman filter for time-varying alpha and beta, and show their equivalence and differences analytically.
2. Quantify tracking error, lag and noise sensitivity on synthetic data with known truth.
3. Evaluate innovation whiteness, spread stationarity and half-life on real hourly data.
4. Formulate z-threshold detection as an adaptive-threshold (CA-CFAR-style) detector and compare nominal vs empirical false-alarm rate, ROC and delay for fixed-window z, EWMA z and Kalman standardised innovation.
5. (Stretch) Add CUSUM on innovations for relationship breakdown (Idea 2).

**Methodology.**
- Model: y_t = h_t^T theta_t + v_t, h_t = [1, x_t]^T, theta_{t+1} = theta_t + w_t, v ~ N(0, R), w ~ N(0, Q).
- Kalman: predict P^- = P + Q. Innovation e_t = y_t - h_t^T theta^-. S_t = h^T P^- h + R. Gain K = P^- h / S_t. Update theta = theta^- + K e_t. Standardised innovation e_t / sqrt(S_t) is the Kalman z-score.
- RLS with forgetting lambda: effective memory 1/(1-lambda). It matches a Kalman filter with Q proportional to P, so the comparison is principled.
- Rolling OLS: rectangular window N, lag about (N-1)/2.
- Detector: threshold k times a locally estimated scale, reference window with a guard region so the current point does not contaminate its own scale estimate (CA-CFAR guard cells).
- Truth for detection: a crossing is a true event if the spread moves at least X% toward its mean within H bars. X and H are swept and reported.
- Walk-forward splits only. No parameter tuned on the test segment.

**Architecture (block diagram).**
```
Public price feed (HL / Binance, 1h)
  -> Clean + align timestamps, log transform, gap handling
  -> Estimator bank: [Rolling OLS | RLS(lambda) | Kalman(Q,R)]
  -> Spread e_t and variance S_t
  -> Standardiser: [rolling z | EWMA z | innovation / sqrt(S_t)]
  -> Threshold detector (k, guard cells)
  -> Event labeller (reversion within H bars)  ->  Metrics: ROC, Pfa, delay
Side path: Synthetic generator (known beta_t, OU spread, t-noise)
  -> same chain -> error vs truth
Stretch: innovations -> CUSUM -> break alarm
```

**Mathematical concepts.** State-space models, Kalman recursion, RLS and forgetting, bias-variance and lag, AR(1)/OU and half-life = -ln2 / ln(phi), autocorrelation and PSD, Ljung-Box, Engle-Granger critical values, beta_yx x beta_xy = rho^2 (orientation asymmetry), total least squares, Neyman-Pearson, ROC, CFAR, Student-t tails.

**Data.** Public hourly candles: Hyperliquid info API and Binance public data archive **[assumption: verify limits, availability and terms]**. Synthetic data generated in code and labelled as synthetic. **Do not use Pear's quant DB, `made_baskets`, or any BE data.** The BE database holds real users' trades. Using Pear data in a college report needs written permission, and user data should never leave Pear.

**Experiments.**
1. Synthetic: beta step and ramp, sweep lambda, Q/R and N. Plot MSE and lag.
2. Synthetic: sweep noise level. Plot tracking error vs SNR-like ratio var(w)/var(v).
3. Real: 10 to 20 liquid pairs, three estimators. Report half-life, ADF-style stationarity with Engle-Granger critical values, innovation whiteness.
4. Detection: ROC per standardiser, Pfa nominal vs empirical at k = 1.5, 2, 2.5, 3, kurtosis of residuals.
5. Orientation: OLS(y on x) vs OLS(x on y) vs TLS. Show beta_yx x beta_xy = rho^2 on real data.
6. (Stretch) CUSUM ARL0 vs delay on injected breaks.

**Expected results.** To be confirmed, not promised. Adaptive estimators should lag less than rolling OLS at equal noise. MSE vs lambda should show an interior optimum. Empirical false-alarm rate should exceed the Gaussian nominal at fixed k because of fat tails and autocorrelation. If results come out differently, report them as found.

**Software stack.** Python 3.11, numpy, scipy, pandas, statsmodels, matplotlib, pytest, Jupyter. Kalman and RLS written by hand (do not hide them behind a library). MATLAB is optional if the department prefers it.

**Report chapters.**
1. Introduction and motivation (internship context, honest scope)
2. Background: stochastic processes, adaptive filters, detection theory
3. System model and problem formulation
4. Estimators (OLS, RLS, Kalman) and their relation
5. Detection and CFAR formulation
6. Implementation and data
7. Experiments and results
8. Discussion, limitations
9. Conclusion and future work
Appendices: derivations, code listing, reproducibility notes

**Demonstration.** A notebook that replays one pair: the three beta estimates against truth on synthetic data, then on real data with sliders for lambda and k updating the ROC and the detections live. Plus a one-page printed derivation of the Kalman/RLS link.

**Viva questions and answers.**
1. *Why is this ECE and not finance?* The problem is time-varying parameter estimation and detection with a controlled false-alarm rate. That is system identification, adaptive filtering and detection theory. Finance only supplies data and the motivating use.
2. *Is a price a signal? What is the noise?* A sampled stochastic process, not signal plus noise by nature. I state a model: spread = latent equilibrium plus mean-reverting part plus measurement noise. Claims hold under that model, and synthetic tests are where truth is known.
3. *Why Kalman over rolling OLS?* Under the random-walk state model it is the MMSE linear estimator and it adapts its gain from P. Rolling OLS is a rectangular window with abrupt forgetting. If the model is wrong it is not optimal, so I measure it instead of assuming it.
4. *How did you choose Q and R?* Grid on a training segment, plus innovation-based checks (innovations should be white). Walk-forward so no look-ahead. I report sensitivity to Q/R.
5. *How does this relate to LMS/RLS?* RLS forgetting lambda gives memory 1/(1-lambda) and equals a Kalman filter with Q proportional to P. Memory length is the tracking-vs-misadjustment trade-off.
6. *What is the CFAR analogy and where does it break?* CA-CFAR sets a threshold from a locally estimated noise level with guard cells. The rolling z does the same. It breaks because the noise is not Gaussian or stationary, my "target" is a labelling rule I define, and neighbouring samples are correlated. I measure that gap instead of hiding it.
7. *How is ground truth defined for detection?* Synthetic data has it by construction. For real data a crossing counts as true if the spread reverts X% within H bars. X and H are swept, and the dependence is reported as a limitation.
8. *Why do regressions of y on x and x on y disagree?* beta_yx x beta_xy = rho^2, so they are not inverses unless rho = 1. This is an errors-in-variables effect. I fix the orientation and test the TLS alternative. It is the same reason a sign-flipped z is not the reverse regression.
9. *Did you build the production stats engine?* No. The system I worked on computes pair stats in separate services. I consumed them, traced orientation and hedge-validity failures, and this project is my own implementation on public data.
10. *Does it make money?* Not claimed. The project measures estimation and detection performance. Any return number would be illustrative only.

**Internship items I can cite.** The BNB/AVAX orientation investigation (finding: the LLM invented the sign flip, backend was correct). #1075 (reverse-oriented rows lose z and hedge). The replay measurements of pair-evidence prevalence (1.7% cointegrated, 23% strong vs 4% made). The pair-vs-basket selection work (#1067, closed unmerged). Backtested returns covering 7 of 1,338 candidate pairs. #905 and #947 landed related fixes but the vault does not say I wrote them, so check `git log` before citing.

**Do NOT claim.** That I wrote or tuned the production z-score or hedge-ratio engine. Any trading profit or edge (the paper-shadow 10/10 positive overnight is preliminary, the replay arm returns were driven by one -31% idea). That Kalman "beats" rolling OLS in general. Any hardware.

---

### 4.2 Project 2. Age-of-information and reliability of a scheduled, rate-limited pipeline

**Final title.** Age-of-Information and Reliability Analysis of a Scheduled, Rate-Limited, Multi-Stage Market-Data Pipeline.

**Abstract (about 190 words).**
Automated trading and analytics systems depend on chains of scheduled jobs in which each stage consumes data produced by the previous one. A stage can fail, be throttled by an upstream API, or run before its input exists, so downstream consumers silently act on older data than intended. This project models such a pipeline as a sequence of periodic update processes and measures information freshness using the age-of-information metric. Delay and loss inputs come from an upstream rate limiter modelled as a token bucket, from retry policies (immediate, fixed, exponential backoff with jitter, additive-increase multiplicative-decrease pacing), and from multi-tier fallbacks whose failures may be correlated through a shared quota. Analytical expressions for mean age under periodic updates with geometric failures are derived and checked against a discrete-event simulation. The schedule, timeouts, freshness rule and failure modes are taken from a real daily pipeline encountered during an internship on a crypto derivatives trading agent; simulation parameters that were not measured are stated as assumptions. The project reports freshness, completion time and failure probability for each policy, and identifies where a common-cause failure breaks the independence assumption behind naive reliability estimates.

**Exact problem statement.** For a chain of periodic jobs with random failure, delay and rate-limit loss, find the age distribution seen by each consumer, the probability it exceeds a freshness limit, and how retry and fallback policies change both.

**Objectives.**
1. Model the real schedule (03:00 OI sync, 10:00 ranking, 12:00 sentiment, 14:00 baskets) as a chain of update processes.
2. Derive mean age and P(age > limit) for periodic updates with i.i.d. failure, and verify by simulation.
3. Model the upstream limit as a token bucket and compare retry/pacing policies on goodput and completion time.
4. Quantify the effect of a shared-quota fallback with a common-cause factor.
5. Check the freshness rule (stale after 36h, TTL 48h) against the schedule.

**Methodology.**
- Age at consumer: Delta(t) = t - u(t), u = generation time of the freshest delivered update.
- Periodic updates with period T, constant delay D, failure prob p per attempt: mean age = D + T(1+p) / (2(1-p)). At p = 0 this reduces to D + T/2. Derive it yourself from E[Y^2] / (2E[Y]) with geometric inter-delivery time, then verify in simulation.
- Consumer reading at a fixed clock time: age at read = 4h + 24h x k for k consecutive producer failures. With the ranking key (TTL 48h, stale if over 36h) k = 0 gives 4h, k = 1 gives 28h, k = 2 gives 52h, which is past both limits. P(fall back) = p^2 for independent daily failures. **[derived from the notes, verify against the real timer and consumer code]**. A consequence worth testing: on this schedule the 36h rule and the 48h TTL trip at the same k, so the 36h rule only matters for off-schedule reruns.
- Token bucket: rate r, depth b. A burst of 50 parallel calls with depth less than 50 is shed. Compare with paced and AIMD senders.
- Fallback: P(system fail) = p1 p2 if independent. Beta-factor model for common cause: fraction beta of each failure is shared.
- Latency identity: a transaction of N sequential round trips of RTT fails a timeout T_out when N x RTT > T_out. At RTT 30ms and a 5s default that is N above about 167 **[arithmetic from the numbers in the notes, N itself not known]**.

**Architecture.**
```
Schedule (cron/systemd timers)
  -> Stage 1 producer (OI sync)  --API calls--> [Rate limiter: token bucket(r,b)]
        |-- 429 -> retry policy (none | fixed | exp+jitter | AIMD) -> back to limiter
  -> Store (key, TTL, timestamp)
  -> Stage 2 consumer/producer (ranking -> sentiment -> baskets)
        |-- missing/stale -> fallback tier (independent | shared quota)
  -> Metrics: age at read, P(age > limit), goodput, completion time
```

**Mathematical concepts.** Renewal theory, age of information, geometric and compound failure, token-bucket shaping, AIMD, exponential backoff, queueing basics, series/parallel reliability, common-cause failure, confidence bounds for zero-failure tests, Little's law as a simulation check.

**Data.** The schedule, timeouts and incident numbers in my own work log. Simulation parameters otherwise. Production logs and traces may be confidential, so use only the schedule shape and numbers Pear is happy to see published, and get that confirmed. **No user data.**

**Experiments.**
1. Analytic vs simulated mean age over p and T.
2. P(age > 36h) vs p, including a rerun-time sweep.
3. Token bucket with cap swept around the assumed 80 req/min: 50-parallel vs paced vs AIMD vs backoff-with-jitter. Plot success, completion time, retry amplification.
4. Fallback failure probability, independent vs beta-factor.
5. Zero-failure evidence: 6 recovered of 6 gives a 95% upper bound of about 39% on failure probability (1 - 0.05^(1/6)). Show why six trials cannot certify reliability and how much simulation extends it, under stated assumptions.

**Expected results.** To be confirmed. Backoff with jitter should beat fixed retry under a shared cap. A burst larger than the bucket should lose a large fraction of calls. A shared quota should push system failure probability well above p1 x p2. Report whatever comes out.

**Software stack.** Python, SimPy, numpy, scipy, matplotlib, pytest.

**Report chapters.** 1 Introduction. 2 Background (AoI, shaping, reliability). 3 The real pipeline and what was observed. 4 Analytical model. 5 Simulator and validation. 6 Policy experiments. 7 Reliability under correlated failure. 8 Discussion and limits. 9 Conclusion. Appendices: derivations, simulator code, parameter table with source tag (observed / assumed).

**Demonstration.** A SimPy-driven live plot: the age sawtooth for each stage, sliders for failure rate, cap and policy, with analytic mean age overlaid on the simulated one.

**Viva questions and answers.**
1. *Isn't this software engineering?* The system studied is software, but the tools are standard networks and communications theory: shaping, AIMD, backoff, queueing, age of information, redundancy. The contribution is the derivation and the policy comparison.
2. *Why age and not latency?* Latency is per-update delay. Age also grows between updates, so update rate and loss matter. A job can be fast and still read old data.
3. *Derive mean age for periodic updates.* D + E[Y^2]/(2E[Y]). For geometric retries on a period T, E[Y] = T/(1-p), E[Y^2] = T^2 (1+p)/(1-p)^2, so age = D + T(1+p)/(2(1-p)).
4. *Where do the parameters come from?* Schedule, timeouts and the 28 Sep OI run from real logs. The ~80 req/min cap is a hypothesis, so I sweep around it.
5. *Why a token bucket?* It is the standard model of a rate limiter: sustained rate r, burst b. A 50-parallel burst against a smaller bucket is shed.
6. *Is retry safe?* Naive retry multiplies load on a limited resource. Backoff with jitter decorrelates retries, like collision backoff in Ethernet. I show load vs goodput.
7. *Why do independent-failure formulas fail for fallbacks?* The earlier fallback shared a quota with the primary, so one 429 hit both. Changing provider removes that cause. A beta-factor model shows the effect.
8. *How do you validate the simulator?* Analytic vs simulated age, Little's law, and one replay of a real observed run.
9. *Six successful injected trials. Is it reliable?* No. Zero failures in six trials only bounds failure probability below about 39% at 95%. Simulation extends it under stated assumptions.
10. *Can you publish production details?* Only what Pear allows. No user data, and I labelled what is observed vs assumed.

**Internship items I can cite.** #923 (fallback to a different provider, stream path fix, trial table), #1034 (transaction timeout over the proxy path), #796 (platform loops to `Promise.all`; the 3h to about 1h figure is expected, not observed), the Hetzner timers and cutover, the ranking key TTL and freshness rule, the first-day "missing or stale" fallback at 14:00, the OI 28 Sep 429 observation, the Redis client duplication and cron-collision incidents.

**Do NOT claim.** The OI cap cause as confirmed (hypothesis). The 3h to 1h speedup as measured. That 6/6 proves reliability. That fallback independence is the real-world value. That I invented AoI.

---

### 4.3 Project 3. Streaming fault detection in exchange price feeds

**Final title.** Robust Streaming Fault Detection in Exchange Price Feeds using Median Filtering, Redundancy Voting and Sequential Tests.

**Abstract (about 190 words).**
Trading systems consume price feeds from several venues and indexes. Faults in these feeds, such as stale values, impulsive outliers, missing samples, wrong scale factors and mismatched venues, can pass unnoticed and corrupt downstream calculations. This project formulates fault detection for streaming perpetual-futures prices as a detection problem on a sampled signal. Four detectors are compared: a Hampel (median and median-absolute-deviation) filter for impulsive outliers, a run-length test for stale values with a threshold derived from the tick-change probability, a CUSUM test for persistent level shifts, and a median-of-three redundancy vote across venues. Because labelled real faults are rare, faults of known type, size and duration are injected into public price histories from Hyperliquid and Binance, and the injection process is documented and reproducible; a small set of naturally occurring anomalies is reviewed manually. Detectors are evaluated by precision, recall, false alarms per day, detection delay and the receiver operating characteristic as window length and threshold vary. The fault taxonomy is drawn from data-integrity bugs encountered during an internship on a crypto derivatives trading agent, including a price cache that kept stale values while its freshness timestamp advanced.

**Exact problem statement.** Given one to three streaming price series per asset at fixed sampling, flag samples that are faulty (stale, impulsive, gap, scale, venue mismatch) with controlled false-alarm rate and minimum delay.

**Objectives.**
1. Define a fault taxonomy and a reproducible injector.
2. Implement Hampel, run-length stale test, CUSUM and median-of-three voting.
3. Derive detector thresholds from stated false-alarm targets, not by hand-tuning.
4. Evaluate precision, recall, delay, false alarms per day, ROC.
5. Separate genuine market events from faults using cross-venue agreement.

**Methodology.**
- Hampel: flag if |x_t - median_w| > c x 1.4826 x MAD_w. Median and MAD have a 50% breakdown point, unlike mean and std, where one outlier inflates its own threshold.
- Stale: with a per-sample probability q of a price change at the chosen resolution, P(k identical in a row) = (1-q)^k. Pick k for a false-alarm target per day.
- CUSUM: S_t = max(0, S_{t-1} + (z_t - kappa)), alarm at S_t > h. Choose h from a target ARL0 by simulation.
- Voting: median of three venues. Deviation threshold from the empirical basis distribution, since venues legitimately differ.
- Scale fault (1000x): ratio test against the median across sources.
- Evaluation: injected faults of swept type, size and duration, tuned on one half and tested on different fault parameters in the other half.

**Architecture.**
```
Venue feeds A (HL), B (Binance), C (third source, optional)
  -> Align timestamps / resample / gap marker
  -> Detectors in parallel:
       [Hampel] [Stale run-length] [CUSUM] [Cross-venue median vote] [Scale ratio]
  -> Fault flag + type + confidence
  -> Metrics (vs injected labels): precision, recall, delay, FA/day, ROC
Fault injector -> corrupts a copy of feed A before the detectors
```

**Mathematical concepts.** Robust statistics (breakdown point), order statistics, run-length and geometric tails, CUSUM and ARL, ROC, redundancy and voting (TMR), impulsive noise and nonlinear filters, sampling and aliasing for gap handling.

**Data.** Public candles and trades from Hyperliquid and Binance **[assumption: verify coverage, resolution and terms; a third source such as Lighter public data is optional and unverified]**. Faults are synthetic and labelled as such. A short list of real anomalies (wicks, outages) is reviewed by hand.

**Experiments.**
1. ROC per detector over window and threshold for each fault type.
2. False alarms per day on a clean month.
3. Detection delay vs fault size.
4. Real wicks: does cross-venue voting separate a single-venue glitch from a market-wide move.
5. Computational cost per sample for the median window.

**Expected results.** To be confirmed. Median-based detectors should keep their false-alarm rate where mean/std versions degrade with outliers present. Stale detection should be near-perfect on long runs and weak on short ones. Voting should catch single-venue faults and miss a common-cause one.

**Software stack.** Python, numpy, scipy, pandas, matplotlib, pytest. Optional baseline: an isolation forest, clearly labelled as a baseline.

**Report chapters.** 1 Introduction. 2 Background (robust statistics, change detection, redundancy). 3 Fault taxonomy from real incidents. 4 Detectors. 5 Fault injection and evaluation design. 6 Results. 7 Discussion. 8 Conclusion. Appendices: injector spec, parameter tables, code.

**Demonstration.** A replay of a real price stream with a fault injected live. Detectors light up with type and delay, running precision and recall on screen.

**Viva questions and answers.**
1. *Injected faults make this circular, no?* Injection is standard in fault-detection evaluation because real faults are rare. I document it, tune on one half, test on different parameters, and say synthetic scores are an optimistic bound.
2. *Why median and MAD?* Breakdown point 50% vs 0%. One outlier inflates the std and hides itself. The 1.4826 factor makes MAD consistent with sigma for Gaussian data.
3. *What does the Hampel filter do in the frequency domain?* It is nonlinear, so no transfer function. A linear low-pass would smear an impulse into its neighbours. The median removes it and keeps edges.
4. *How is the stale threshold set?* From P(no change in k samples) under the observed change probability, so it matches a chosen false-alarm per day.
5. *Why CUSUM?* It is optimal for a known shift in the sense of minimising delay for a given ARL0. Threshold h comes from a target ARL0 by simulation.
6. *What does redundancy voting cost and miss?* Median of three tolerates one bad source. It fails on common-cause faults and needs a basis allowance between venues.
7. *What about a real flash crash?* It is not a fault. It shows up in every venue, which voting uses to tell it apart from a single-venue glitch.
8. *Why not a neural net or isolation forest?* Interpretable detectors with known error properties come first. ML needs labelled faults I do not have. I can include one as a labelled baseline.
9. *Real-time cost?* Median over a window of w is O(w log w) naively and O(log w) with a heap. Show per-sample time against the sampling interval.
10. *What is from the internship?* The failure taxonomy: stale cache with an advancing timestamp, scale-prefix mapping, venue mismatch. The detectors are new work.

**Internship items I can cite.** The price store fix (replace-not-merge semantics, in the Lighter bring-up work of 18 to 20 Aug), #481 (quant IDs vs HL tickers), #557 (`execution_venue_price`, price anchored to the account's actual venue), #588/#589 (wiring gaps from an assumed venue). Cite as the source of the fault types.

**Do NOT claim.** That I built a production feed monitor. That injected-fault scores are real-world accuracy. That any venue's data was verified beyond what the project shows.

---

## 5. Recommendation

**Pick Project 1** (adaptive hedge-ratio tracking plus adaptive-threshold detection), with Idea 2 as the stretch chapter.

Against the six criteria:
1. Related to the internship: yes, but be exact. The concepts and failure modes are real and I debugged around them. I did not write the estimators. Say "consumed and investigated", not "built".
2. Technical enough: yes. Derivations, three estimators, detection theory.
3. ECE defensible: the strongest of the three. Adaptive filtering, Kalman, detection, stochastic processes are core ECE.
4. Achievable: public data, standard algorithms, no hardware, no infra. Smallest build.
5. Graphs, equations, diagrams: tracking error curves, ROC, Pfa vs k, frequency-style window comparisons, a clean block diagram.
6. Not CRUD: there is no application. It is analysis.

**Trade-off.** Project 2 is the most authentic (Authenticity 4 vs 3) and the right pick if the panel values the internship story over core-ECE content. Its weak point is "this is software". If the department is strict about ECE subjects, Project 1 is safer. A reasonable hybrid is Project 1 as the project and one internship-report chapter on the Project 2 timing and freshness findings. That chapter would be separate, not a second project.

**Smallest useful next step.** Pull one month of hourly candles for two liquid pairs, write the Kalman and rolling OLS estimators on a synthetic pair with a beta step, and plot the lag. One plot tells whether this is worth the next two months.

## Still unknown

- Evaluation date and guide's subject area (changes how much MATLAB or communications framing is wanted).
- Who wrote the production hedge-ratio and z-score code. `git log` on the stats services would settle it, if I have access.
- Pear's position on citing schedules, timings and incident numbers in a college report.
- Public data availability and limits for Hyperliquid, Binance and Lighter at the resolutions needed.
