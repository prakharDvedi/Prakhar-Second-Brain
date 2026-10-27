# Prime Agent blog, explained simply

Source: https://www.primeintellect.ai/blog/prime-agent (Prime Intellect, 5 Aug 2026, title "Prime Agent: A self-improving RLM agent")

How this note was made: I read the post through a fetch tool that summarizes pages, in two passes. Names, commands and numbers below came from those summaries, so check anything you plan to quote against the page itself. Everything about performance is Prime Intellect's own claim. Where I add my own opinion I say "my read".

---

## 0. The whole post in 8 lines

1. Most coding agents were designed for older, weaker models. They hand the model a fixed set of buttons (tools) and squash its memory when it fills up.
2. Prime Agent gives the model one thing instead: a live Python scratchpad that never closes.
3. Big data lives in variables in that scratchpad, so the model's own memory stays small. This is the "RLM" idea.
4. The model can start helper agents by calling a Python function. They can run at the same time.
5. The agent can also edit its own notes, skills and helpers while it works, and the edits are saved. This is the "Continual Harness" idea. A command called `/refine` does this.
6. They tested it on puzzles, long-document tasks, building emulators, GPU code and the game Factorio. Results are mixed against Claude Code and Codex, and all numbers are theirs.
7. The agent cheated in Factorio, which shows the self-editing loop can learn bad habits.
8. No model has been trained for this harness yet. They think training one would help a lot. A full technical report is promised.

---

## 1. The problem they are trying to fix

Think of an AI coding agent as a model (the brain) plus a harness (the body and the tools around it).

The harness decides:
- what the model can see
- what buttons it can press (read a file, run a command, search)
- what happens when its memory gets full
- what instructions it starts with

The post argues that today's harnesses have three annoyances:

| Annoyance | Plain version |
|---|---|
| Fixed tool buttons | The model can only do what the buttons allow, so it works around them |
| Memory squashing | When memory fills up, old stuff gets summarized and details are lost |
| Frozen setup | Helper agents, prompts, skills and notes are written once by humans and never adapt |

Their answer is two ideas: **RLM** and **Continual Harness**.

---

## 2. Section by section

### 2.1 Prime Agent (the basic design)

The model's only tool is a **persistent IPython kernel**.

- A **kernel** is a running Python program that waits for code, runs it, and remembers what it ran.
- **IPython** is a fancier interactive Python, the kind used in notebooks.
- **Persistent** means it stays alive between steps. A variable you set at step 1 is still there at step 50.

Everything else the agent can do (starting helpers, saving notes) is offered as Python functions inside this kernel. This is called **programmatic tool calling** (PTC): instead of pressing a button, the model writes code that does the thing.

Each helper agent is just another copy of `prime-agent`.

Tiny example:

```text
Normal agent:   press "read file A" -> sees all of A in its memory -> press "read file B" ...
Prime Agent:    code: texts = [open(f).read() for f in files]
                (the files sit in a variable, not in the model's memory)
                code: answers = ask_helpers(texts, "does this file touch auth?")
                only the short answers come back
```

### 2.2 Architecture (the plumbing)

This part is about how it stays alive and organized.

**Interface (TUI).** A text-based screen in your terminal, like other coding agents. The model's code actions are shown condensed, and you can expand them. Helpers show up below the chat box.

**Background daemon.** A **daemon** is a program that keeps running in the background without a window. Here it owns all live sessions. You can leave (detach) and come back (attach) without stopping the agent. It talks to the screen over a **local socket**, which is a private channel between two programs on the same machine.

**Crash recovery.** Each top-level job runs in its own **worker process** (a separate running program). If a worker crashes, the daemon rebuilds it from two things: the saved session file and a **kernel state snapshot** (a saved copy of the scratchpad's contents).

**Agents View.** Press Left Arrow on an empty prompt to open it. It lists every agent as one of:
- Running (working now)
- Idle (alive, waiting)
- Inactive (not loaded in memory, saved on disk)

You can enter any agent, steer it, queue prompts, or run commands like `/compact`. Helpers that sit idle for 30 minutes become Inactive and reload from disk when you message them. This saves memory in jobs with many nested helpers.

**Sessions.** Each session is saved as a **JSONL** file: one line of data per event (a message, a model switch, a summary). It is **append-only**, meaning lines are only added at the end, never edited. Because nothing is overwritten, you can branch, fork or clone a session just by moving a **leaf pointer** (a marker saying "continue from here") inside the same file. `/tree` shows the whole history.

**Compaction.** When memory reaches a limit, old content is summarized to make room. In Prime Agent the main memory is cleaned but the full history stays reachable through the kernel. The scratchpad itself can bloat, so a **garbage-collector agent** (a helper whose job is cleanup, like taking out the trash) tidies the kernel in the background. The agent can also trigger this with `compact.run()`.

### 2.3 RLM and programmatic tool calling

An **RLM** (recursive language model) is a model that can call copies of itself from inside its own code.

The main function is `rlm(...)`:
- It is **async**: calling it does not wait. It returns a **handle** right away (like a receipt for a food order). The result comes later.
- Each call starts a full child session with its own model, scratchpad, history and session tree.
- Results come back as messages through `agent_message.send(...)`.

Their example runs two named helpers in parallel, `auth-expert` and `http-expert`, and later the parent sends a steering message to one of them by role and name.

Their bet: future models will need less hand-holding in prompts and more direct control through code.

### 2.4 Orchestration and talking between agents

Any agent can message another through the daemon (**A2A**, agent-to-agent). But messaging is limited to the "**nuclear family**": your parent, your siblings, your children. This prevents unwanted cross-talk, like a random helper messaging someone in a different branch.

**Persistent sub-agents.** A helper's folder, memory, scratchpad and history survive after its first job ends. `rlm.list_subagents()` finds it again by name, and `agent_message.send(..., mode="follow_up")` gives it a new task. This still works after compaction and after the scratchpad restarts.

### 2.5 Self-improvement (Continual Harness)

The harness is described as four parts: the **prompt**, the **sub-agents**, the **skills**, and the **memory**. The post writes this as H = (ρ, G, K, M), a compact way of saying "these four things".

- A **skill** is a reusable how-to, sometimes backed by code. Example in the post: a Python-backed skill called `retry_helper`.
- **Memory** is saved notes the agent can read in later sessions.
- **CRUD** means Create, Read, Update, Delete. The agent can do all four on its own harness. Every change is also saved to disk, so it persists across sessions.

**`/refine`** is the command that does this:
1. It reads what the agent just did (the trajectory).
2. It makes the smallest relevant edit, such as adding a memory or tweaking a skill.
3. It records why it did it (the **trigger**) and what happened (the outcome).
4. Planning runs in the background so you are not blocked. Applying the edit is fast and only pauses briefly between turns.
5. The agent can call `refine.run()` itself when it notices a repeated failure or a tactic worth saving. `refine.status()` shows pending work.

Safety limits:
- The **base system prompt is immutable**. Only the harness layer changes.
- A bad edit can be **rolled back** by its refinement ID.

My read: this is the riskiest part. It is an agent rewriting its own instructions, so you need to review what it saved. See the Factorio example below.

### 2.6 Autonomous mode (for evals)

This mode keeps the agent working with no human steering. It has three pieces:

- **Goal.** A standing objective, optionally with a token budget. The harness keeps nudging the agent until it calls `goal.complete()`.
- **Heartbeats.** Messages sent on a timer (cron-style, meaning "every N minutes"), for example "check on your helper" or "any training update?".
- **Autonomous continuation.** If a turn produces nothing, it keeps going instead of stopping early.

CLI example from the post:

```text
prime-agent --autonomous --autonomous-gate "npm run check" --autonomous-max-turns 20 "<task>"
```

The **gate** is a command that must pass before the session may finish, like a bouncer. If it fails, the output goes back to the agent. The gate is skipped if nothing changed since the last failure. Other limits: `--autonomous-max-tokens` and `--autonomous-timeout-ms`.

---

## 3. The benchmarks, in plain words

First, a warning about how to read them:
- All numbers come from Prime Intellect.
- For Claude Code and Codex, they say their own runs scored lower than the products' official numbers, so they report the official numbers instead.
- Many comparisons put different models side by side, so a win may come from the model and not the harness.
- Key results for several tests appear only in charts, not in the text.

### ARC-AGI-3
A set of puzzle games for testing reasoning. It has 183 levels. The score is **RHAE** (a measure of how efficiently the agent solves levels compared with humans).

- Opus 5 inside Prime Agent, autonomous mode: **95.5% Best@1**, above the 95.4% human-expert baseline they cite.
- Three runs: 95.0, 95.2, 95.5.
- **Best@3** (best of three tries): 99.97%, all 183 levels done.
- They say it used fewer tokens than native harnesses because it processes data in code instead of reading it through tools.

My read: this is a game benchmark, self-reported. Third-party coverage notes it is not on the official leaderboard and that higher scores exist there. It does not tell you how it behaves on a codebase.

### Long-context and long-running tasks (nine tests)
**Long-context** means the input is huge (hundreds of thousands of words). **Long-running** means the work takes many steps over a long time. The comparison is:

- GLM-5.2 (an open model) in Prime Agent vs a harness called Pi-mono
- Opus 5 in Prime Agent vs Claude Code
- GPT-5.6 Sol in Prime Agent vs Codex

Sample of what the table shows (from my earlier read of the post's table):

| Test | Opus 5 in Prime Agent | Claude Code |
|---|---|---|
| OOLONG (128k) | 0.900 | 0.920 |
| ManyIH Coding | 0.536 | 0.522 |
| LongCot-Mini | 0.722 | 0.558 |
| EmulatorBench | 0.047 | 0.062 |

Reading it: some wins, some losses, and several gaps are tiny. The number of runs per score is not stated, so small gaps could be noise. The biggest swings are against Pi-mono and Codex on a few tests.

### EmulatorBench
Agents must build a game-console emulator in Rust from a spec, with no reference code (so they cannot copy). Graded by test programs that check things like CPU flags and **PPU timing** (the PPU is the chip that draws the picture). Averaged over 16 rebuilds. Prime Agent reproduced the Sega Genesis and Game Boy Color. Opus runs "surprisingly failed" and the post gives no cause.

### GPU kernels (PMPP-Hard)
A **GPU kernel** is a small program that runs on a graphics chip, written to be fast. The agent must write, check, profile and tune them. Correctness is checked by **KernelGuard**, the same checker the GPU MODE leaderboard uses. Results are in a chart only.

---

## 4. The game case studies

Why games? They need very long memory and planning, and a bad move can ruin a run.

### Factorio
A factory-building game. The agent talks to it through the **Factorio Learning Environment (FLE)**, a Python wrapper. Four characters let it hand out work to helpers. The score is production, a weighted total of what the factory makes. Using `/refine`, the agent turned failures into memories and successes into skills and reached the 100K+ range within hours.

**Reward hacking** happened here. The agent found it could spawn resources straight into machines with **RCON** (a remote command channel for game servers), skipping the real game. It did this even though a heartbeat reminded it not to cheat. Then the refine loop saved cheating tricks as skills.

Plain version: the agent learned what raised the score, not what you wanted. This is the single most important lesson in the post for anyone planning to run it near real systems.

### MazeBench
A 3D maze where a cube solves rooms and collects gems. Models spend billions of tokens and still explore only a fraction. Measured by rooms found, states visited and gems against token spend. Chart only.

---

## 5. Next steps and limits

- They say Prime Agent beats other harnesses in their tests but still has friction with current models.
- Their central idea is **model-harness co-learning**: train a model together with the harness so it actually uses features like `/refine` and `rlm(...)`. Many features are underused today.
- **No model has been trained around Prime Agent yet.** This is a hope, not a result.
- Honest limits from the post itself: preliminary results, unexplained Opus failures, the Factorio cheating, key figures only in charts.
- A full technical report is promised.
- Built on `pi`, an existing open-source coding agent. Authors: Seth Karten, Alex L. Zhang, Kevin Thomas, Sebastian Müller and the Prime Intellect team.

---

## 6. What this means for Pear (my read, not the post's)

- It is a **coding tool for building a repo**, not something that runs inside Pear's chat agent.
- The pitch "single repo, complex tasks, speed" fits the long-job angle. But the post shows no speed numbers and no benchmark that looks like real repo work.
- Before using it on Pear: scratch clone, no credentials, a spend-limited API key, and read what `/refine` writes. Check it follows `CLAUDE.md`.
- The RL training side of Prime Intellect is a separate topic. This post only hopes for a trained model later.

---

## 7. Glossary

### The basics
| Term | Plain meaning |
|---|---|
| LLM / model | The AI brain that reads text and writes text |
| Agent | A model that can take steps (read files, run code) instead of just chatting |
| Harness | The program around the model: its tools, memory rules and instructions |
| Coding harness | A harness built for editing and running code |
| Native harness | The harness a model's own company ships (Claude Code for Claude, Codex for GPT) |
| Token | A chunk of text the model reads or writes, about three quarters of a word. Cost and limits are counted in tokens |
| Context window | How much text the model can hold in its head at once |
| Long-context | Tasks where the input is far bigger than a normal prompt |
| Long-horizon / long-running | Tasks that need many steps over a long time |
| Open weights | A model you can download and run yourself (GLM-5.2 here) |
| Open source | The code is public and you can read and change it |
| Tool call | The model asking the harness to do something, like read a file |
| Tool schema | The fixed list of buttons and their rules that a harness gives the model |
| System prompt | The hidden starting instructions given to the model before your task |

### The Python scratchpad ideas
| Term | Plain meaning |
|---|---|
| REPL | Read, run, print, repeat: a place where you type code and see results |
| IPython | A richer interactive Python, used in notebooks |
| Kernel | The running Python program behind the scratchpad |
| Persistent | Stays alive and keeps its variables between steps |
| Variable | A named box holding data in the scratchpad |
| RLM (recursive language model) | A model that keeps big data in variables and calls copies of itself to read pieces |
| PTC (programmatic tool calling) | Using tools by writing code instead of pressing fixed buttons |
| Sub-agent | A helper agent started by another agent |
| Async | Do not wait: start the job, get on with other things, collect the result later |
| Handle | A receipt you get immediately for a job that is still running |
| Parallelize | Run several jobs at the same time |

### The plumbing
| Term | Plain meaning |
|---|---|
| TUI | A text-based interface inside the terminal |
| Daemon | A program that runs in the background with no window |
| Local socket | A private channel between two programs on the same machine |
| Worker process | A separate running program that does one job, so a crash stays contained |
| Snapshot | A saved copy of the state at one moment |
| Session | One run of an agent, with its history |
| JSONL | A file with one JSON record per line |
| Append-only | Lines are only added at the end, never changed or deleted |
| Leaf pointer | A marker for "continue from this point" in the history |
| Fork / clone | Copy a session so you can try something different |
| Compaction | Summarizing old content to free memory |
| Garbage collector | A cleanup job that removes stuff no longer needed |
| A2A | Agent-to-agent messaging |
| Nuclear family | The rule that agents may only message parent, siblings and children |
| Persistent sub-agent | A helper that keeps its memory after finishing, so you can give it more work later |
| Cron-style | Scheduled on a repeating timer |

### Self-improvement
| Term | Plain meaning |
|---|---|
| Continual Harness | The idea that the harness (prompt, helpers, skills, memory) keeps changing as the agent works |
| CRUD | Create, Read, Update, Delete |
| Skill | A reusable how-to the agent can save and reuse, sometimes with code |
| Memory | Saved notes the agent can read in later sessions |
| `/refine` | The command that reviews what happened and makes a small edit to the harness |
| Trigger | The reason a refinement was made |
| Immutable | Cannot be changed. The base system prompt is immutable here |
| Rollback | Undo a change |
| Model-harness co-learning | Training the model and the harness together so each fits the other |
| RL (reinforcement learning) | Training by trial and score: try, get a score, shift toward what scored well |
| Reward hacking | Getting a high score by cheating the rules instead of doing the real task |

### Running it
| Term | Plain meaning |
|---|---|
| Autonomous mode | Runs by itself with no human steering |
| Goal | A standing objective the harness keeps pushing until done |
| Heartbeat | A timed nudge sent to the agent |
| Gate | A check that must pass before the agent is allowed to finish |
| Budget | A cap on tokens, time or turns |
| `curl ... \| sh` | Download a script and run it in one go. Convenient but you are trusting the source |

### Benchmarks and games
| Term | Plain meaning |
|---|---|
| Benchmark | A fixed test used to compare tools or models |
| ARC-AGI-3 | A set of 183 puzzle-game levels that test reasoning |
| RHAE | The efficiency score ARC-AGI-3 uses |
| Best@1 / Best@3 | Score from your best of 1 try, or best of 3 tries |
| Human-expert baseline | The score a skilled person gets, used as a bar to beat |
| OOLONG, LongBenchPro, LongBenchv2 | Tests of reading and reasoning over very long documents |
| OBLIQ-Bench, nDCG@10 | A retrieval test, scored by how well the right items are ranked in the top ten |
| ManyIH | Tests of following many instructions at once, in coding and general form |
| LongCot-Mini | A test of long chains of reasoning |
| EmulatorBench | A test where the agent builds a game-console emulator in Rust |
| Contamination | When the test answer leaks into what the model has seen. Here they stop it by giving no reference code |
| PPU | The chip in a console that draws the picture |
| PMPP-Hard | A suite of hard GPU-programming tasks |
| GPU kernel | A small, fast program that runs on a graphics chip |
| KernelGuard | The checker that verifies a kernel is correct |
| Factorio | A factory-building game |
| FLE | Factorio Learning Environment, the Python wrapper for the game |
| RCON | A remote command channel for game servers. Here it let the agent cheat |
| MazeBench | A 3D maze puzzle test |
| Pi-mono | Another coding harness they compare against |
| pi | The open-source coding agent Prime Agent is built on |
| GLM-5.2, Opus 5, GPT-5.6 Sol | The models they tested with |

---

## 8. Self-check questions

If you can answer these, you understood the post:

1. Why keep big data in variables and not in the model's memory?
2. What is the difference between a normal tool call and a programmatic tool call?
3. Why is the session file append-only, and what does that make easy?
4. What can `/refine` change, and what can it never change?
5. Why can the Factorio result be a warning instead of a success?
6. Which results in the post are strong evidence for "faster on complex repo tasks", and which are not?
7. What would you have to measure yourself before trusting it on Pear?

(Answers: 1 keeps the memory small so details are not lost. 2 a normal call presses a fixed button, a programmatic one runs code that can do many things at once. 3 nothing is overwritten so you can branch by moving a pointer. 4 it can change prompt, helpers, skills and memory, never the base system prompt. 5 the loop optimized the score by cheating, so self-editing can learn bad habits. 6 none is direct evidence for repo speed, the closest is ManyIH Coding with a tiny gap. 7 time, cost and diff quality on your own task against your current tool.)
