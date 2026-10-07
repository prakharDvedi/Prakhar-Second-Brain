# Codebase Retrieval with codectx (Graphify + Embeddings)

Goal: stop coding agents (and me) from re-grepping the whole repo every session. One command, `codectx`, answers "where does X live / what depends on it" from a persistent index instead.

Built 2026-09-27 for `agent-pear-v2`, but it works for **any git repo** — it indexes whatever repo you run it in. See [[27th Sep 2026]].

### Quick use

From anywhere inside a repo (root or any subfolder):

```bash
codectx "where is leverage validated and what depends on it?"   # main use
codectx --explain "DagService"            # a symbol's callers/callees
codectx --affected "planSetLeverageUsecase()"  # what breaks if I change it
codectx --path "ChatController" "DagService"   # how two things connect
codectx --status                          # index health (no API calls)
codectx --refresh                         # forced rebuild + embedding sync (no LLM)
codectx --enrich                          # optional LLM doc reading (costs money)
codectx --budget 500 "..."                # cap output (~tokens, default 1200)
```

In Claude Code: `/codebase <question>` runs the same thing.

For a flow question (e.g. "how does a trade go from Telegram to execution"), ask **one short question per hop**, not one long one:
`"telegram text message handler"` → `"queue worker that runs the agent"` → `"trade tool that stages a ticket"` → `"ticket confirm execution"` → `"result delivered back to telegram"`, then `--explain` the hits.

Output is **orientation, not proof** — it gives symbol + `file:line-range`; always open the source before trusting it.

### The mental model

Three layers, all stored **outside the repo** (repo stays clean — verified with `git status` before/after):

1. **Graph (Graphify, free, local).** Parses every code file's syntax and builds a map: ~13.5k nodes (functions, classes, files) and ~36k edges (calls, imports, references, inherits). No LLM.
2. **Embeddings (OpenRouter, cheap).** Each of ~12.4k entities gets a meaning-vector (`openai/text-embedding-3-small`, 1024 dims), so plain-English questions work without exact names.
3. **Hybrid search (`codectx`).** question → top semantic matches → follow graph edges 1–2 hops (hub nodes not expanded) → rerank + dedupe, tests demoted unless the question is about tests → compact list of symbols, relationships, files with line ranges.

Without an API key it still works, falling back to **keyword** matching + the graph (header says `lexical fallback: <reason>`).

### Where things live

| What | Path |
|---|---|
| The program | `~/.local/share/code-intelligence/tools/codectx.py` (one Python script, run via `uv`) |
| Command shims | `~/.local/bin/codectx` (Git Bash), `~/.local/bin/codectx.cmd` (PowerShell/cmd) |
| Per-repo index | `~/.local/share/code-intelligence/<owner>-<repo>/` → `graphify/graphify-out/` (graph), `embeddings/index.sqlite` (vectors), `state.json`, `config.json` |
| Claude Code command | `~/.claude/commands/codebase.md` |
| Agent instructions | "Codebase retrieval (codectx)" section at the end of `~/.claude/CLAUDE.md` |
| Benchmark tool | `codectx-bench` → `~/.local/share/code-intelligence/tools/codectx_bench.py` |
| Graphify itself | `uv tool` install, `graphifyy` 0.9.69 with the `[openai]` extra |

### API key

- Uses `OPENROUTER_API_KEY` (or `OPENAI_API_KEY`). Stored once as a Windows **user** env var — never in a file, never printed.
- Needed only to embed each **question** (~$0.0000002/query) and to re-embed changed code. The stored vectors are local.
- If a session started before the key was set, `codectx` reads it straight from the saved user env var, so no restart is needed.
- Set it (copies from `apps/agent/.env` without printing):

```powershell
[Environment]::SetEnvironmentVariable("OPENROUTER_API_KEY", ((Get-Content apps\agent\.env | Select-String '^\s*OPENROUTER_API_KEY\s*=').Line -replace '^\s*OPENROUTER_API_KEY\s*=\s*','' -replace '^["'']|["'']$',''), "User")
```

### Config (env vars)

- `CODECTX_EMBED_MODEL` — default `openai/text-embedding-3-small`. `voyageai/voyage-code-4` also works on OpenRouter (6× price).
- `CODECTX_EMBED_DIM` — default `1024` (confirmed honored by OpenRouter for text-embedding-3-small); `0` = model native.
- `CODECTX_LLM_MODEL` — model for `--enrich`, default `qwen/qwen3.8-flash` (see gotchas — it didn't work well).
- Changing model or dim is detected: the old index is never mixed with new vectors; queries fall back and tell you to run `--refresh`, which rebuilds.

### Incremental updates

- Each query computes a cheap fingerprint (HEAD + dirty files + mtimes). Unchanged → no work, ~3s query. Changed → `graphify update` (~60s) then re-embed only changed entities.
- So: first query after editing/switching branch ≈ 1 min, the rest ≈ 3s.
- After a big refactor that deletes code, run `codectx --refresh` (graphify refuses to shrink the graph without `--force`).

### Costs (measured on this repo)

| | Cost |
|---|---|
| Graph build/update | free |
| Full embedding pass (12,393 entities) | ≈ $0.03 |
| Each query | ≈ $0.0000002 |
| Total spent during setup | $0.18 measured (OpenRouter key usage delta) — mostly the failed `--enrich` attempt; the split between the two is estimated |

### Gotchas (learned the hard way)

- **Claude desktop app virtualizes `%LOCALAPPDATA%`.** It's an MSIX package, so anything it writes under AppData lands in `AppData\Local\Packages\Claude_…\LocalCache\…` — invisible to a normal terminal. That's why the index lives in `~/.local/share`, not `%LOCALAPPDATA%`. See [[Engineering Lessons#A packaged desktop app's writes to AppData can land in a private virtualized copy]].
- **Graphify re-parses all JS/TS on every update.** It deliberately skips its cache for JS/TS (cross-file resolution), so `graphify update` is ~60s on this repo regardless. The fingerprint check is what keeps queries fast.
- **`--enrich` (LLM doc reading) is opt-in and wasn't useful yet.** With `qwen/qwen3.8-flash`, replies kept hitting the output limit, graphify kept halving batches (46 → 23 → … → 3), and each cut-off attempt was billed — ~$0.15 for little/nothing saved before I stopped it. It also needs `uv tool install "graphifyy[openai]"`. If retried, use a model with a large output limit and no thinking step. Plain `--refresh` never calls an LLM.
- **Stopping a background run from Claude Code may not kill it.** The task-stop ended the wrapper shell but `codectx`/`graphify` kept running; had to `taskkill /PID <sh.exe pid> /T /F` the tree.
- **Runtime hops are invisible to the graph.** LLM tool dispatch, BullMQ queues, DI — `codectx --path` finds no edge from `TelegramService` to `runTicketExecution()`. Semantic queries still find each side; bridge the gap by reading code.
- `.sql` files aren't indexed (`graphifyy[sql]` not installed).

### Measuring whether it helps

`codectx-bench` reads Claude Code transcripts (`~/.claude/projects/<proj>/*.jsonl`) and compares runs: wall time, API round-trips, tool calls, grep vs targeted reads, context tokens, and a crude quality check (`--expect sym1,sym2` = functions the answer should mention).

```bash
codectx-bench <session-id> <session-id> --expect "onText,handleText,..."   # compare past chats, free
codectx-bench --ab "PROMPT" --expect "..."   # runs PROMPT twice headless: with vs without codectx (costs 2 Claude runs)
```

Results for "trace a trade from Telegram → execution → reply" (2026-09-27):

| | Run 1 — keyword fallback, then grep | Run 2 — semantic codectx |
|---|---|---|
| Wall time | 148s | 94s (−36%) |
| Model round-trips | 20 | 10 (−50%) |
| grep/find searches | 28 | 10 (−64%) |
| Context processed | 1.73M tok | 0.88M tok (−49%) |
| Key functions in answer | 9/10 | 9/10 |

Caveat: Run 1 still ran codectx once (keyword mode), so it's not a pure "without" baseline; n=1 each.

**Clean headless A/B (2026-09-27, `codectx-bench --ab`, Opus 5.5, same prompt, read-only tools; "without" = codectx blocked + told not to use it):**

| | With codectx | Without codectx | Change |
|---|---|---|---|
| Cost (Claude, from `claude -p`) | $0.89 | $1.15 | **−22%** |
| Wall time | 178s | 166s | +7% (8 codectx calls × ~3s) |
| API round-trips | 23 | 31 | −26% |
| grep/find searches | 10 | 17 | −41% |
| Tool output read | 102k chars | 171k chars | −40% |
| Context processed | 2.29M tok | 3.08M tok | −26% |
| Key functions in answer | 10/10 | 9/10 (missed `buildTicketCtaMarkup`) | +1 |

Read: cheaper and more complete, not faster. n=1 per arm — single runs vary; treat as directional. Sessions `d50ccd11` (with) / `7cbf4f8c` (without).

### Uninstall

Delete `~/.local/share/code-intelligence/`, `~/.local/bin/codectx*`, `~/.claude/commands/codebase.md`, and the "Codebase retrieval (codectx)" section in `~/.claude/CLAUDE.md`. Optionally `uv tool uninstall graphifyy`.
