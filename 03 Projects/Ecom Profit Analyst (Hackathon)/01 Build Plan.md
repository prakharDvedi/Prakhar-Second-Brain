# Build Plan (2 days, technical)

Product view of each stage lives in the repo issues. This note is the how. Stack reasons are in [[02 Decisions]].

Time is a guess for one person. Stage numbers match the repo issues.

## Repo layout

```text
ecom-profit-analyst/
  README.md
  pyproject.toml
  .env.example           ANTHROPIC_API_KEY, MODEL, MAX_USD, MAX_TURNS
  data/
    raw/                 Olist csvs (gitignored)
    messy/               output of make_messy.py (small sample committed)
    truth.json           planted problems and true answers
  scripts/
    make_messy.py        seeded mess + synthetic costs and ad spend
  agent/
    loop.py              tool use loop, caps, resume
    tools.py             tool schemas and handlers
    state.py             ledger and run state, read/write json
    metrics.py           trusted metric registry (the verifier uses this)
    verify.py            claim check
    report.py            fills the report template from verified claims
    prompts.py           system prompt
  evals/
    run_eval.py          N runs, scores, writes evals/results/*.json
  runs/                  per run state and outputs (gitignored)
  plan/                  cfo.ai export
```

## Day 1: get a traced agent that cleans and computes

### Block 1 (1h): Stage 0 setup
1. Create venv, install `anthropic`, `duckdb`, `pandas`, `matplotlib`, `python-dotenv`.
2. Install neatlogs and Entire. **Read their docs first, I have not checked the install steps.** Write down the exact commands in the README.
3. Wire neatlogs around one dummy LLM call. Open the trace. Done when I can see one span tree.
4. Turn on Entire checkpoints and make one commit that has a checkpoint.
5. Open cfo.ai and create an empty model file. Save in `plan/`.

### Block 2 (1.5h): Data and mess generator
1. Download Olist from Kaggle into `data/raw/`.
2. Write `scripts/make_messy.py` with `random.seed(42)`:
   - tables out: `orders.csv`, `order_items.csv`, `refunds.csv`, `product_costs.csv`, `ad_spend.csv`
   - `product_costs.csv`: cost as a fraction of price per product, with 5% of products missing
   - `refunds.csv`: built from low review scores plus a few planted high return products, plus 10 refunds with no matching order
   - `ad_spend.csv`: spend per channel per month, two channels, one planted bad channel
   - plant: 3% duplicate orders, 15% of rows in a second currency with a rate column missing in one month, one file with renamed columns
3. Write `data/truth.json`: list of planted problems with counts, and true profit for each product.
4. Done when truth.json exists and I can open the messy files.

### Block 3 (1.5h): Tools and loop (Stage 1)
1. `tools.py` first set:
   - `list_files(dir)` returns name, rows, columns
   - `profile_file(name)` returns column types, null counts, sample rows, duplicate key counts
   - `run_sql(query)` read only DuckDB over loaded files, result capped at 200 rows, returns error text on failure
   - `record_fix(problem, action, rows_affected, reason)` appends to the ledger
   - `record_claim(metric, params, value, text)` appends a claim
   - `finish()` ends the run
2. `loop.py`: send messages with tool schemas, run tool calls, append results, repeat. Stop on `finish`, max turns or max dollars. Count tokens and cost from the response usage fields.
3. Wrap each model call and each tool call as a neatlogs span. **Not checked: neatlogs span API, confirm in docs.**
4. Done when the agent lists the files, profiles them and prints a plain summary. Save a trace.

### Block 4 (2h): Cleaning (Stage 2)
1. System prompt tells the agent: profile every file, list problems, fix by running SQL that creates `clean_*` views, log each fix with `record_fix`, never delete without a ledger entry.
2. Add `create_view(name, sql)` so cleaned tables are named and reusable. Only names starting with `clean_` allowed.
3. Run it. Compare ledger against `truth.json` problems. Write a small `evals/score_cleaning.py` for this.
4. Done when most planted problems are found. Note which are missed, that is the first thing to fix with traces.

### Block 5 (1.5h): Metrics and profit (Stage 3)
1. `metrics.py` registry, each is a pure function of the clean views:
   - `revenue_by_product`, `refund_rate_by_product`, `shipping_by_product`, `cogs_by_product`, `ad_cost_by_product`, `profit_by_product`
   - ad cost split rule: by revenue share within the month and channel. Write the rule in a docstring so the agent can quote it.
2. Give the agent `get_metric(name, params)` that calls the registry. The agent can also use `run_sql` to explore, but claims must name a registry metric.
3. Hand check 5 products against raw rows. Done when they match.

### End of day 1 (30m)
- Post 1 and Post 2 in public (issue #11). Include a screenshot of a messy file and the first trace.
- Commit with checkpoint. Close or comment on issues #1 to #4.

## Day 2: verification, recovery, evidence, business, video

### Block 6 (1.5h): Investigation (Stage 4)
1. Add tools: `drilldown(metric, product_id, by)` where `by` is one of channel, month, variant, region. It wraps registry metrics.
2. Prompt: pick the 3 worst products by profit, drill until a cause is stated or 5 calls are spent, then record a claim with a confidence of low, medium or high.
3. Done when the planted high return product is found and named as a returns problem.

### Block 7 (2h): Verifier and report (Stage 5)
1. Claim object: `{id, metric, params, claimed_value, text}`.
2. `verify.py`: call `metrics.<metric>(**params)`, compare with `claimed_value`. Pass if within 0.5% or 1 cent. Return pass, fail with the true value, or error.
3. Failed claim: send the true value back to the agent once so it can fix the claim. Still failing, drop it and list it as unverified.
4. `report.py`: fill `report.md` from a template. Sections: summary, products losing money, why, actions, data fixes (from ledger), assumptions, verification (N of N). Charts: profit bar, return rate bar.
5. Test the checker by editing one value by hand.
6. Done when a run ends with a report and a line like "18 of 18 numbers verified".

### Block 8 (1.5h): Recovery (Stage 6)
1. Tool errors return text, never raise out of the loop.
2. Add a check in the loop: if the same tool call fails 3 times, add a message telling the agent to change approach. After 6 failures, stop that branch and write it in the report.
3. Join row count guard: `run_sql` result includes `rows` and a warning if a query on `clean_orders` returns more rows than the order table has.
4. State: write `runs/<id>/state.json` after each tool call (messages summary, ledger, claims, spend). Add `--resume <id>`.
5. Break a file on purpose and stop a run halfway. Both should work.

### Block 9 (2h): Evals and fixes (Stage 7)
1. `evals/run_eval.py`: run the agent 5 times on the same messy data. Score per run:
   - planted problems found (from ledger vs truth.json)
   - claims verified / claims made
   - profit error vs truth on top 10 products
   - dollars and seconds per run, read from the run state
2. Save results as JSON with a label (baseline, after fix 1, after fix 2).
3. Open the worst traces in neatlogs. Pick 2 real failures. Fix each in code or prompt. Re-run the eval. Keep the before and after files in `evals/results/`.
4. Do not invent failures. If runs are clean, say so and use the cleaning misses from block 4.

### Block 10 (1h): Business plan (Stage 8)
1. Take average cost per run from the eval results.
2. In cfo.ai: price per report (try 2 or 3 prices), cost per report, gross margin, customers in a base and a growth case, a fixed monthly cost line for hosting, runway. **Not checked: cfo.ai input format. Use the same structure as the example in the track brief.**
3. Export or screenshot the plan into `plan/`.

### Block 11 (1.5h): Video and submit (Stage 9)
Video script, about 3 minutes:
1. 20s: the seller problem and the messy files
2. 40s: run the agent, show it cleaning and drilling
3. 30s: report with "N of N verified". Then edit one number and show the checker catch it
4. 40s: neatlogs trace of a real failure, the fix, the before and after scores
5. 20s: Entire checkpoint on that fix
6. 20s: cfo.ai plan with the measured cost per report
7. 10s: what is next

Then: README with run steps and results table, post 3 and post 4, submit.

## Risks and fallbacks

| Risk | Fallback |
|---|---|
| neatlogs or Entire setup takes long | Timebox 1h. Ask in the hackathon channel. Keep a log of what failed, it is also content for a post. |
| Agent misses planted problems | That is the eval story. Fix the prompt and tools, show the score change. |
| Cost per run too high | Use the cheaper model for profiling and cleaning, the bigger one for investigation. Cap with `MAX_USD`. |
| Behind schedule | Cut block 6 depth first (fewer drilldown types). Never cut verification or evals. |

## What I need to do myself

1. Make a Kaggle login and download the Olist dataset into `data/raw/`.
2. Create an Anthropic API key and put it in `.env`.
3. Get access to neatlogs, Entire and cfo.ai (accounts, keys). Send me the docs links or paste the install steps so the plan stops saying "not checked".
4. Decide a submission link or form and the deadline. Not in the brief I was given.
5. Post the public updates (4 posts, issue #11). I can draft them in your voice. You press post.
6. Record the video.
7. Read the Olist license before the demo.

## Related
[[00 Index]] · [[02 Decisions]]

## Update 2026-10-10: plan for the GST import auditor

Replaces the e-commerce blocks above for the chosen project. Same 2 day shape, same stage numbers as repo issues #1 to #11. Stack, loop, ledger, caps, resume, neatlogs, Entire and cfo.ai carry over from [[02 Decisions]]. Scope and checks are in [[03 Alternative - GST Import Auditor]].

### Repo layout changes
- `scripts/seed_books.py` replaces `make_messy.py`: fixed seed, fake org, about 25 bills with planted errors, writes `data/truth.json`
- `agent/books.py`: client with list vendors, list bills, get bill, update bill. Default backend reads the generated data. Flags make it return 429, expired token and update error. Zoho backend optional
- `agent/rules.py`: the 7 checks, one test each
- `agent/rates.py`: 6 to 8 service types with SAC and rate (unconfirmed until a CA checks)
- `agent/verify.py`: read back the stored bill and re-run the rules

### Day 1
- Block 1 (1h, Stage 0): same setup as above. Add README note that the data is generated
- Block 2 (1.5h, Stage 1): `seed_books.py`, `truth.json`, books client
- Block 3 (1.5h, Stage 1): tools and loop in read-only audit mode, first neatlogs trace
- Block 4 (2h, Stage 2): `rules.py` with a test per rule, score findings against `truth.json`
- Block 5 (1.5h, Stage 3): service classification from description, rate and SAC from the table only
- End of day: posts 1 and 2 (#11), commit with checkpoint

### Day 2
- Block 6 (1.5h, Stage 4): vendor history drilldown, safe to fix or human review
- Block 7 (2h, Stage 5): apply fix, read back, re-run rules, report with RCM output tax vs input credit totals and N of N verified
- Block 8 (1.5h, Stage 6): failure flags on the fake client (429, expired token, update error), backoff, human review path, resume
- Block 9 (2h, Stage 7): evals over several runs, score planted errors found, fixes verified, wrong fixes (must be 0), false alarms, cost, time. Fix 2 real failures, keep before and after
- Block 10 (1h, Stage 8): cfo.ai plan from measured cost per audit
- Block 11 (1.5h, Stage 9): video, README, posts 3 and 4, submit

Cut order if behind: checks 6 and 7, then the vendor history drilldown. Never cut read-back verification or evals.

### Risks
- Rules and rate table not confirmed by a CA. Say so in the README and video
- Generated data can look too easy. Plant vague descriptions and repeated vendor errors so the agent has real decisions to make
