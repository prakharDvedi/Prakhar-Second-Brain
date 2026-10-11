# Decisions

Status tags: **confirmed** means decided by me. **assumed** means my reasoning but not tested yet. **not checked** means a fact I have not looked up.

## Stack

| Part | Choice | Why | Status |
|---|---|---|---|
| Language | Python 3.10+ | Data work is easiest here. 3.10.11 is already installed on this machine. | confirmed (version seen) |
| Model | Claude through the Anthropic SDK, model id read from an env var | One place to swap models. Default `claude-sonnet-5-5`, cheaper `claude-haiku-5-5` for simple steps. | assumed |
| Agent loop | Hand written tool use loop, no agent framework | The loop is about 100 lines. A framework hides the steps that neatlogs needs to show. | assumed |
| Data engine | DuckDB | Runs SQL on CSV files with no server. One file, fast. | assumed |
| Dataframes | pandas only for loading and the mess generator | Keep it small. | assumed |
| Charts | matplotlib to PNG | No web app to build. | assumed |
| Interface | CLI. Command takes a folder and a question. | A UI is not scored. Time goes to the agent. | assumed |
| Output | `report.md` plus PNG charts plus `ledger.json` | Easy to read, easy to show in the video. | assumed |
| Tracing | neatlogs | Required. | not checked: install steps and API |
| Code history | Entire checkpoints | Required. | not checked: install steps and API |
| Business plan | cfo.ai | Required. | not checked: how export works |

## Update 2026-10-09: tool facts checked against official pages

Replaces the "not checked" tags above for the three tools. Sources: docs.neatlogs.com/sdk/python, docs.entire.io/quickstart, neatlogs.com/hackathon.

- **neatlogs** (confirmed from docs): `pip install -U "neatlogs[anthropic]"`, Python 3.10 to 3.13. `neatlogs.init(workflow_name=..., instrumentations=["anthropic"])` before importing anthropic. Key from env `NEATLOGS_API_KEY`, without a key spans are made but not sent. `@neatlogs.span(kind=...)` with kinds WORKFLOW, AGENT, CHAIN, TOOL and others. Call `neatlogs.flush()` and `shutdown()` at the end of scripts. Installed version here: 1.4.25.
- **Entire** (confirmed from docs): Windows install is `irm https://entire.io/install.ps1 | iex`. Then `entire enable -y --agent claude-code` inside the repo, which adds git hooks. A checkpoint is made when the agent changes code and I commit, and Entire asks to link the commit to the session. `entire checkpoint list` shows them. entire-graph is a CLI plugin with its own docs pages. Not yet read.
- **cfo.ai** (from a news search, weaker): new AI CFO product called Ari, connects to Sheets and Excel, 14 day free trial. Input format for the hackathon plan not checked.
- **Hackathon rules** (confirmed from the neatHack page): build period Oct 10 to 11 starting 12:00 AM IST, kickoff 9:00 AM IST. Submission Oct 12, 7:00 to 11:55 PM IST, on X by quote tweeting the official neatlogs post. Needs a public repo with commits dated during the hackathon, trace links, Entire output, cfo.ai plan, a recovery run, before/after numbers, posts tagged #neatHack. AWS credits (Bedrock) for the first 200 teams to check in, Crustdata 5000 credits for everyone.
- **Open risk:** the repo's first commit (readme, license) is dated 2026-10-09 14:51 IST, before the build period. Code commits wait until after 12:00 AM IST on Oct 10.

## Data

- Base data: Olist Brazilian e-commerce public dataset from Kaggle. Has orders, items, payments, products and reviews. Needs a Kaggle login to download (my action).
- Not in Olist: product cost and ad spend. I generate both with a seeded script and say so openly in the README.
- License: I recall Olist as CC BY-NC-SA 4.0 (non commercial). **not checked**. Read the license on the Kaggle page before the demo and put the credit in the README. Fine for a hackathon, matters if this becomes a product.
- Mess is planted by `make_messy.py` with a fixed seed. Because I plant it, I know the right answers and can score the agent.

## Key design decisions

1. **Verifier is code, not a second model.** The agent writes claims as structured objects that name a metric from a fixed registry. A deterministic function recomputes the metric from the cleaned data. Compare to the claimed value within a tolerance. A model checking a model is not evidence.
2. **Agent cannot write the final report text with free numbers.** Numbers in the report come from a template filled with verified claims only. A claim that fails is retried once, then dropped and listed under "could not verify".
3. **Cleaning is logged, never silent.** Every fix and every excluded row goes to the ledger with a reason. Ledger is shown in the report.
4. **Run state on disk.** After every tool call the state is written to `runs/<id>/state.json` so a stopped run can continue.
5. **Hard caps.** Max turns, max tokens and max dollars per run. The loop stops and says why.
6. **Eval harness built early.** Because mess is planted with a known seed, the same script scores every run. That gives the before and after comparison for neatlogs.

## Out of scope on purpose

- Web UI, login, payments
- Live Shopify connection (files only)
- Forecasting
- Multi agent setup. One agent with good tools is enough. Revisit only if a clear need shows up.

## What I need to do (not the agent)

See the end of [[01 Build Plan]].

## Related
[[00 Index]] · [[01 Build Plan]]
