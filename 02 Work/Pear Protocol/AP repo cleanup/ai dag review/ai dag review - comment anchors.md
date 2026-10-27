# ai/dag review - comment anchors

Parked for later. To be picked up properly someday.

- Repo: agent-pear-v2, folder `apps/agent/src/ai/dag`
- Logged: 27th Oct 2026
- Source: AI-generated maintainability audit of the folder. Line numbers are from that day and will drift. Usage claims (dead code, duplicates) came from repo-wide greps in that session; nothing here was changed in code as a result.
- Incident-id and history comments were already deleted from the code, so they have no anchors. Mention them in the PR description instead.
- Related: [[Pear Protocol]]

Review-comment anchors, using the line numbers at the time. These are comment targets only. Anchors are grouped by finding.

**Node list consolidation (P2)**

| Location | Comment about |
|---|---|
| `types/index.ts:22` (`ExecutableNode`) | This enum should be the single node list. The other copies derive from it. |
| `node-registry.ts:14` (`PlannerNodeId`) | A third restatement of the same ids. |
| `node-registry.ts:264` (`EXECUTABLE_PLANNER_NODE_IDS`) | Duplicates the enum. Use `Object.values(ExecutableNode)`. |
| `node-registry.ts:283` (`PLANNER_TO_EXECUTABLE`) | Identity map: every value equals its key. Delete it. |
| `node-registry.ts:344` (`NODES_WITH_DEPENDENCIES`) | The same ten strings as `EXECUTABLE_PLANNER_NODE_IDS`. |
| `node-registry.ts:310`, `executor/orchestrator.service.ts` (`filterAllowedNodes`), `executor/node-executor.service.ts:420` | The same `Object.entries(PLANNER_TO_EXECUTABLE).find(...)` reverse lookup, each followed by a cast. |
| `executor/orchestrator.service.ts:258` (`doneMessage`) and `:296` (`buildNodeOutput`) | 10-way switches over the nodes. A missing node falls through silently, and `buildNodeOutput` is mostly `findings[node]`. |
| `executor/orchestrator.service.ts` (`normalizeDependencies`) | The `depMap` literal restates all ten nodes. Weakest item: `DependencyMap` is `Record<ExecutableNode, …>`, so the compiler already catches a missing key. |
| `dag.service.ts:554` (`getActiveBranches`) | Only used for the `Plan: [...]` log and the `nodesUsed` span attribute. Omits `scheduler` (9 of 10 nodes), so scheduler-only plans log `none`. `getActiveNodes(plan).join(" + ")` gives the same text and covers all nodes. |
| `services/planner-adapter.service.ts:219`, `:269` and `createDefaultPlan` | The default-deps literal is written three times. |
| `builders/context-assembler.service.ts:76` (`NODE_LABELS`) | `Record<string,string>`, so a missing node isn't caught by the compiler. |
| `services/clarify.ts:13` (`PLANNER_NODE_IDS`) | Another copy of the node ids. |

**`dag.service.ts` duplicated tails (P2)**

| Location | Comment about |
|---|---|
| `dag.service.ts:545`, `:1012`, `:1306`, `:1516` | The same leak-guard `logger.error` repeated four times. Extract `reportLeak`. |
| `dag.service.ts:1267`-`1280` and `:1575`-`1590` | The chart and trade-link tail is repeated here and again in `composer.service.ts:369` and `:569`. Share one helper. |
| `dag.service.ts:1274` and `:1580` vs `composer.service.ts:375` and `:575` | `.catch(() => [])` exists in `dag.service` but not in the composer. The copies have already drifted. |
| `dag.service.ts:396`, `:1078`, `:1707` | The `callerFacing` plus detail plus warn/error branch is copied three times. |
| `dag.service.ts:271` (`run`) and `:719` (`runAsyncInner`) | The terminal clarify and dialed-decline handling is written twice. The terminal `sources` yield payload recurs in `runAsyncInner` and `runAdvancedAsyncInner`. |
| `dag.service.ts` (all four entry points) | The inline `{ question: string; options: string[] }` should be `SuggestedAction`. |
| `dag.service.ts:710` vs `executor/orchestrator.service.ts:132` | The same `stopRequested`, copied verbatim. |

**Orchestrator duplication (P2)**

| Location | Comment about |
|---|---|
| `executor/orchestrator.service.ts:38` and `:141` | The span-per-node body and the failure `forEach` repeat in both functions. Extract one `runLayer`. These are the paths that run the money-moving nodes. |
| `dag.service.ts:798`, `:1459`, `executor/orchestrator.service.ts` (`THINKING_TICK_MS`) | The 1400 ms ticker loop is written three times. |
| `dag.service.ts:948`, `:1496` | The 1500 ms stop-probe is hardcoded twice. |

**Dependency direction (P2)**

| Location | Comment about |
|---|---|
| `dag.service.ts:164` (`forwardRef`) | Likely caused by `advanced/tools.ts` injecting `DagService` while agents import `dag/types`. Verify after moving shared types. |
| `services/hyperliquid-market.service.ts` (class) and `services/coingecko.service.ts` | Shared infrastructure with 22 importers outside the folder. Move out of `ai/dag`. |
| `services/hyperliquid-market.module.ts` | Exists only to avoid a cycle with `ChartModule`. |
| `services/hyperliquid-market.service.ts:692`-`:730` and `:931` | Transport/retry and TA rendering are mixed in one class with listing resolution. |

**Dead code (P3)**

| Location | Comment about |
|---|---|
| `dag.module.ts:4` | No importers; every provider is registered in `ai.module.ts`. |
| `services/research-web.service.ts:24` and `:34` | `modelOverride` is never passed by any caller. |
| `dag.service.ts:570` (`execute`) | No callers in `apps/`. |
| `dag.service.ts:694` (`compose`, `_plan`) and `:671` (`executeResearchTask`, `_userInput`) | Unused parameters. |
| `executor/orchestrator.service.ts:283` (`buildNodeInput`, `_userInput`) | Unused parameter. |
| `types/index.ts:139` (`ComposerInput`) | Referenced nowhere else. |
| `types/index.ts:105`-`:114` | `TextResult` aliases. Four are used only by `coingecko`, and the rest collide in name with different `agents/*/schema.ts` exports. |

**Weak typing and casts (P3)**

| Location | Comment about |
|---|---|
| `dag.service.ts:98` | `stripPlanTelemetry` takes `any`. |
| `types/index.ts:99` | `steps?: any[]`, which is why `node-executor.service.ts:370`-`:376` reads v6 and legacy field names untyped. |
| `services/planner-adapter.service.ts:309`, `:333` | `as unknown as Record`, `as never`. |
| `executor/orchestrator.service.ts` (`buildNodeInput`) and `executor/node-executor.service.ts:86` | `node as string as keyof typeof plan`. |
| `dag.service.ts:302`, `:1224`, `:1652` | `as AppType`, then `String(appType)`. |
| `types/index.ts:217` | `ExecutableNode \| "compose" \| string` collapses to `string`. |
| `node-registry.ts:384` | `ALLOWED_DEPS_BY_NODE` is `readonly string[]` instead of a node type. |
| `types/index.ts:226`, `:231` | The platform union is repeated in the input and result types. |

**Swallowed failures and logging (P3)**

| Location | Comment about |
|---|---|
| `dag.service.ts:219` | `.catch(() => {})` on the session-surface write. |
| `services/planner-adapter.service.ts:61` | Every failure is logged as `planner:timeout`. |
| `services/composer.service.ts:171` | Bare `catch {}`. |
| `services/hyperliquid-market.service.ts:187`, `:227`, `:252` | Bare `catch` blocks that return `false` or `null`. |
| `services/composer.service.ts:450`, `:628`, `services/research-web.service.ts:36` | Log-and-rethrow, and `node-executor` logs the same error again. |

**Possible dead code, needs verification**

| Location | Comment about |
|---|---|
| `builders/context-assembler.service.ts:90` | A stateless injectable with one method and one caller. Check how the specs mock it first. |
| `types/index.ts:118` (`LeaderboardResult`) | Same name as `packages/execution/src/contracts/leaderboard-result.ts`. Not diffed yet. |
