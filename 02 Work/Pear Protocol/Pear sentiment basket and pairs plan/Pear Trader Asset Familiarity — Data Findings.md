
Status: open. Data-quality investigation done; ranking logic and architecture not yet decided. *(Update 2026-09-29: ranking decided by Jeyd — 65/35 current/historical distinct-trader asset score, §11. Implementation not started.)* *(Update 2026-09-29, later: pre-implementation validation done, §12. Feasible with pipeline changes; blockers and decisions listed there.)*
Filed: 2026-09-29.
Plan / repo analysis: [[Better Trading]]
Final plan (2026-09-30): [[Familiar-Asset Ranking — Final Plan]]

## What this is

Goal: generate basket and pair ideas around assets Pear users already trade, rather than arbitrary ones. Before designing anything, I checked what "assets Pear users trade most" actually means in the data.

All queries were read-only against three DBs. The URLs came from `apps/agent/.env` and were never printed.

- **Pear BE Postgres** (`BE_READONLY_DATABASE_URL`)
- **Context DB**
- **Quant DB.** Could not connect: its certificate chain isn't trusted locally. I did not disable TLS verification.

Snapshot date: 2026-09-29. Windows are the last 7 and 30 days.

## 1. How the BE trade tables fit together

Tables available to the read-only role: `fills`, `positions`, `executions`, `instruments`, `client_ids`, `triggers`, `trade_accounts`, `users`. The role cannot create temp tables, so queries must use inline CTEs.

Join integrity over 30 days (110,795 Pear-placed fills):

| Check | Count |
|---|---|
| Fills without a position | 0 |
| Fill vs position user mismatch | 0 |
| Fill vs position account mismatch | 0 |
| Fills whose execution is missing | 0 |
| Execution vs fill position mismatch | 0 |
| Execution vs fill user mismatch | 0 |
| Fill vs execution `client_id` differs | 15 |
| Symbols not matching an `instruments` row | 0 |

Canonical joins: `fills.execution_id → executions`, `fills.position_id → positions`, and `(fills.connector, fills.symbol) → instruments (connector, instrument_id)`. `fills.symbol` is the instrument id, e.g. `159` = HYPE on Hyperliquid.

## 2. What counts as a Pear-initiated trade

### Most fills are not Pear activity

Last 30 days, excluding reconciled and synthetic fills:

| Kind | Fills | Distinct users |
|---|---|---|
| Engine-placed (`execution_id` set, not external) | 105,658 | 337 |
| No execution, not external | 335 | 22 |
| Imported external (`is_external=true`) | 313,018 | 319 |

About **75% of fills are the same wallets trading outside Pear**, imported as `creation_source='IMPORT'` / `origin='EXTERNAL'`. The flag `execution_id IS NOT NULL` identifies an engine-placed fill exactly.

### Most Pear-placed buy fills are not user decisions

Pear-placed fills by execution type, 30 days (largest rows):

| Intent | reduce_only | Scheduled | Fills | Users | Executions |
|---|---|---|---|---|---|
| ADJUST | false | yes | 64,770 | 40 | 25,385 |
| OPEN | false | no | 9,635 | 299 | 2,760 |
| ADJUST | false | no | 8,306 | 176 | 1,878 |
| CLOSE | true | no | 8,198 | 245 | 2,061 |
| ADJUST | true | no | 4,380 | 101 | 883 |
| ADJUST | true | yes | 4,327 | 20 | 1,575 |
| CLOSE | true | trigger | 2,308 | 125 | 776 |
| ADJUST | false | trigger | 1,570 | 30 | 512 |
| OPEN | false | yes | 819 | 35 | 248 |
| OPEN | false | trigger | 312 | 34 | 122 |

- About **70% of non-reduce Pear fills are scheduled ADJUST executions** (auto-rebalance / TWAP) from only 40 users. Counting `reduce_only=false` fills as "opens" inflates notional and position counts.
- A user's trade decision = an **`OPEN` execution with `status='COMPLETED'`**.
- OPEN executions over 30 days: 3,132 completed, 219 failed, 1 active. All 219 failed ones produced **0 fills**.

OPEN executions by client, 30 days:

| Client | Open executions | Users | Positions |
|---|---|---|---|
| PEARUI (web) | 2,476 | 272 | 2,296 |
| AGENTPEAR (agent) | 824 | 85 | 795 |
| no client | 35 | 10 | 29 |
| MCP | 9 | 2 | 9 |
| SENYA | 6 | 1 | 2 |
| PEARPRO | 1 | 1 | 1 |

`client_ids` codes: AGENTPEAR, APITRADERS, MCP, PEARPRO, PEARUI, SENYA.

Users by role: 2,388 basic, 62 pro, 2 vault.

## 3. Ticker normalization

The same asset appears under many spellings:

- Hyperliquid dex prefixes: `xyz:`, `hyna:`, `km:`, `flx:`, `mkts:`, `io:`, `para:`. NVDA exists as 8 instruments; BTC and TSLA as 8 each.
- Scale prefixes: `kPEPE` (HL), `1000PEPE` (binance, lighter, bybit, `hyna:`), `PEPE` (okx).

Normalization: strip the dex prefix and the `k` / `1000` prefix, keep a HIP-3 flag. Watch for collisions, e.g. `GAS` the token vs `flx:GAS` natural gas.

Coverage of Pear OPEN fills, 30 days:

| Connector | Type | Fills | Distinct bases |
|---|---|---|---|
| hyperliquid | perp | 5,329 | 162 |
| lighter | perp | 4,834 | 139 |
| hyperliquid | hip3 | 610 | 70 |

## 4. Bots and test accounts

Across 308 users with completed opens in 30 days:

| Screen | Users |
|---|---|
| Best-trade rule: median hold < 5 min AND median open < $50 | 8 |
| Median open < $20 | 8 |
| > 20 opens per active day | 1 |
| Median hold < 2 min | 14 |
| Used API-style clients (APITRADERS / SENYA / MCP) | 3 |
| No client on any open | 4 |

Notable accounts (IDs truncated):

- `7c1901bc`: 119 AGENTPEAR opens in 3 days, $11 median open, 0.5 min median hold. Looks like our own test traffic.
- `cf12a0e0`: 84 opens over 30 days, $25 median, 3.2 min hold.

Heavy scheduled-ADJUST users are real, large accounts (auto-rebalance), so I kept the users and dropped their ADJUST fills. Top scheduled-ADJUST counts:

| User | Scheduled ADJUST executions | Positions |
|---|---|---|
| f8422a09 | 9,198 | 34 |
| e69241c3 | 4,115 | 10 |
| b5497c07 | 3,208 | 101 |
| 6a0cbdb0 | 2,577 | 3 |
| 00c71f5e | 2,423 | 5 |
| 38867505 | 1,726 | 53 |

Bots barely move distinct-user counts.

## 5. Assets ranked by distinct Pear users

Method: completed OPEN executions, Pear-placed legs only, normalized tickers, vaults and the 8 bot-rule users excluded.

Totals: **298 users** over 30 days, **158** over 7 days. 252 distinct assets over 30 days, 180 over 7 days.

**87% of opened legs are part of a pair or basket.**

| # | Asset | Users 7d | Users 30d | Long users | Short users | Opens 30d | % multi-leg | Notional 30d | % users via agent |
|---|---|---|---|---|---|---|---|---|---|
| 1 | BTC | 64 | 135 | 104 | 81 | 612 | 79 | $6.16M | 37 |
| 2 | ZEC | 71 | 129 | 73 | 76 | 341 | 90 | $0.66M | 22 |
| 3 | ETH | 53 | 127 | 103 | 63 | 512 | 87 | $4.38M | 35 |
| 4 | HYPE | 46 | 120 | 112 | 19 | 395 | 90 | $0.99M | 30 |
| 5 | SOL | 58 | 109 | 83 | 43 | 281 | 89 | $1.27M | 22 |
| 6 | NEAR | 55 | 85 | 67 | 25 | 204 | 84 | $0.42M | 25 |
| 7 | UNI | 31 | 81 | 35 | 54 | 195 | 87 | $0.45M | 23 |
| 8 | LIT | 24 | 75 | 67 | 21 | 230 | 80 | $0.39M | 28 |
| 9 | ENA | 29 | 71 | 62 | 18 | 191 | 70 | $0.83M | 34 |
| 10 | PUMP | 16 | 70 | 57 | 20 | 177 | 86 | $0.41M | 21 |
| 11 | SUI | 28 | 59 | 30 | 38 | 125 | 89 | $0.17M | 29 |
| 12 | AERO | 27 | 59 | 54 | 7 | 98 | 94 | $0.23M | 20 |
| 13 | XRP | 24 | 52 | 25 | 32 | 119 | 76 | $0.65M | 31 |
| 14 | ARB | 16 | 49 | 23 | 31 | 138 | 83 | $0.27M | 37 |
| 15 | XMR | 20 | 46 | 39 | 11 | 67 | 75 | $0.07M | 15 |
| 16 | LINK | 14 | 44 | 31 | 16 | 75 | 83 | $0.21M | 27 |
| 17 | WLD | 21 | 43 | 29 | 14 | 92 | 72 | $0.50M | 23 |
| 18 | ADA | 19 | 40 | 10 | 33 | 84 | 98 | $0.13M | 25 |
| 19 | PEPE | 10 | 40 | 17 | 24 | 58 | 95 | $0.08M | 33 |
| 20 | DOGE | 13 | 39 | 13 | 28 | 66 | 91 | $0.10M | 26 |
| 21 | AAVE | 16 | 36 | 29 | 10 | 53 | 87 | $0.18M | 28 |
| 22 | AVAX | 13 | 35 | 18 | 21 | 65 | 92 | $0.10M | 31 |
| 23 | VVV | 12 | 34 | 26 | 11 | 87 | 63 | $0.30M | 26 |
| 24 | PONS | 6 | 33 | 19 | 17 | 92 | 85 | $0.23M | 24 |
| 25 | BNB | 17 | 32 | 22 | 13 | 74 | 99 | $0.18M | 16 |
| 26 | ETHFI | 7 | 31 | 30 | 3 | 66 | 83 | $0.11M | 16 |
| 27 | JUP | 11 | 30 | 23 | 9 | 55 | 95 | $0.04M | 27 |
| 28 | ONDO | 8 | 29 | 15 | 14 | 40 | 90 | $0.07M | 24 |
| 29 | APT | 10 | 28 | 6 | 22 | 41 | 90 | $0.04M | 18 |
| 30 | LTC | 12 | 27 | 4 | 23 | 38 | 89 | $0.08M | 19 |

Top 7 days by distinct users: ZEC 71, BTC 64, SOL 58, NEAR 55, ETH 53, HYPE 46, UNI 31, ENA 29, SUI 28, AERO 27.

Direction skew:
- Mostly longed: HYPE (112/19), LIT, ENA, AERO (54/7), ETHFI (30/3).
- Mostly shorted: ADA (10/33), LTC (4/23), APT (6/22), DOGE.

### First-pass (fill-based) ranking, for comparison

My first pass counted non-reduce Pear-placed fills. That still included scheduled ADJUST fills, so notional and position counts were inflated. Distinct users came out close to the clean numbers: BTC 145 / ETH 137 / ZEC 132 / HYPE 129 / SOL 118 over 30 days. Pear users with any opening fill: 180 over 7 days, 323 over 30 days.

Per-user notional concentration in that pass (largest single user's share of the asset's 30-day notional): AERO 60%, LTC 62%, xyz:GOLD 57%, JUP 59%, ONDO 53%, kPEPE 51%. **So ranking by notional is unreliable.**

## 6. Comparison with the basket pipeline (`pickDailyAssets`)

`pickDailyAssets` picks the top 10 non-HIP-3 assets by Coinglass open interest, plus 7 hard-coded HIP-3 tickers (TSLA, NVDA, GOOGL, AMZN, PLTR, MSTR, META).

Current OI top 10 from the context DB: ETH, BTC, HYPE, ZEC, SOL, NEAR, XRP, LIT, PUMP, ENA.

**It overlaps 9/10 with Pear's top 10 by distinct users.** Only UNI (Pear #7) is missing; XRP is extra.

→ Injecting a global top-N into `pickDailyAssets` would be close to a no-op. The value is in ranks 11–30, in pair compositions, and in direction skew.

### Candidate-universe cut

`SentimentService.getAssetRowsByCategory` keeps only the **20 most recently analysed assets per category** (within 14 days). In the context DB snapshot on 2026-09-29, Pear's top assets fall outside that cut:

| Asset | Category | Last sentiment run | Rank in category | In universe |
|---|---|---|---|---|
| BTC | Layer1 | 09-28 14:20 | 29 | no |
| ZEC | Layer1 | 09-28 14:20 | 33 | no |
| ETH | Layer1 | 09-28 12:00 | 34 | no |
| HYPE | Layer1 | 09-28 14:20 | 30 | no |
| SOL | Layer1 | 09-28 14:20 | 31 | no |
| NEAR | AI | 09-28 14:20 | 14 | yes |
| UNI | Defi | 09-28 17:16 | 30 | no |
| LIT | Others | 09-28 14:20 | 40 | no |
| ENA | Defi | 09-28 12:00 | 31 | no |
| PUMP | Meme | 09-28 14:20 | 25 | no |

The cause is that a 173-asset sentiment batch at 09-28 17:16 pushed the daily-picked assets out of the Layer1/Defi/Others top-20. That batch is consistent with the weekly-sentiment re-run done during the Hetzner cutover (see [[My Work at Pear Protocol]], 2026-09-28 entry).

Sentiment batches in the context DB, last 8 days:

| Run | Assets |
|---|---|
| 09-27 12:00 | 11 |
| 09-28 08:49 | 12 |
| 09-28 12:00 | 3 |
| 09-28 14:20 | 14 |
| 09-28 17:16 | 173 |

**Caveat, unverified.** The context DB behind `apps/agent/.env` has **no `AssetSentiment` or `CategorySentiment` rows from 09-08 to 09-26**; history starts 09-27. Two possibilities: it's a dev DB, or ingest writes rolled back before PR #1034. So I have not concluded anything about what production's 14:00 UTC basket run actually saw day by day. The code mechanism (`take: 20` ordered by recency) is confirmed; the production impact is not. *(Superseded 2026-09-29: not a dev DB and not a gap — `AssetSentiment` is latest-only (overwritten per asset), and this DB is production. The cutoff effect is quantified in §12.1. See also [[Asset Sentiment Playground — Research & Experiments]].)*

- PONS: tradable on every venue (hyperliquid, lighter, lighter_rhc, binance, bybit, okx), 33 Pear users over 30 days, but **absent from the sentiment universe**, so the basket generator can never pick it.

## 7. What the popular assets really are

Top pair/basket compositions by distinct users (30 days):

| Composition | Users 30d | Users 7d | Opens |
|---|---|---|---|
| Long AERO / Short UNI | 38 | 13 | 48 |
| Long ETH / Short BTC | 34 | 15 | 96 |
| Long SOL / Short ZEC | 34 | 33 | 50 |
| Long BTC / Short ETH | 20 | 4 | 58 |
| Long XMR / Short ZEC | 16 | 12 | 22 |
| Long NEAR / Short ZEC | 16 | 15 | 18 |
| Long HYPE / Short ETH | 13 | 4 | 21 |
| Long PUMP / Short SOL | 12 | 0 | 16 |
| Long SKY / Short BTC | 10 | 10 | 11 |
| Long HYPE / Short LIT | 8 | 2 | 8 |
| Long BTC / Short MSTR | 7 | 1 | 12 |
| Long ETH / Short DOGE | 6 | 1 | 7 |
| Short BTC / Short ETH / Short SOL | 6 | 0 | 7 |
| Long XLM / Short SUI | 6 | 0 | 6 |
| Long ENA, HYPE, LIT, ZEC (basket) | 5 | 1 | 7 |
| Long PENGU / Short PEPE | 5 | 0 | 6 |
| Long DRAM / Short SKHY | 5 | 0 | 5 |
| Long BTC / Long ETH | 5 | 2 | 8 |
| Long HYPE / Short SOL | 5 | 1 | 10 |
| Long LIT / Short ETH | 5 | 1 | 7 |

Daily distinct users (day of month : users), last 14 days:

| Asset | Pattern |
|---|---|
| AERO | quiet (1–2/day) → **20 on the 19th** → 2–8/day |
| UNI | 1–3/day → **21 on the 19th** → 4–9/day |
| ZEC | 3–7/day, 14 on the 22nd, **24 on the 27th, 31 on the 28th** |
| NEAR | 2–11/day, **20 on the 27th**, 14 on the 28th |
| LIT | steady 1–8/day |
| PUMP | 1–5/day, 9 on the 28th |
| SUI | 1–7/day, 13 on the 21st |
| PONS | 1–3/day |

Observations:
- AERO and UNI popularity is essentially **one pair** (Long AERO / Short UNI), with a single-day spike on the 19th.
- ZEC popularity is mostly **two pairs** (Long SOL / Short ZEC, Long NEAR / Short ZEC), concentrated on the 27th–28th.
- The only enduring pairs are **BTC/ETH**, in both directions.
- Emerging: Long SKY / Short BTC (all 10 users in the last 7 days), Long XMR / Short ZEC.
- Hypothesis, not checked: the spikes are campaign / featured-trade / notification days. Not yet correlated with our notification logs. *(Partly answered for the 27th–28th ZEC/NEAR spike: it came from outside X posts, not Pear. See §9.)*

## 8. Ranking recommendation (not decided) *(superseded by Jeyd's 65/35 decision — see §11)*

- **Event:** a completed OPEN execution. Pear-placed legs only, normalized tickers, vaults and the bot rule excluded.
- **Primary signal:** distinct users over 30 days, **spike-dampened.** Either count user-days with a per-day cap, or flag an asset as event-driven when its biggest single day exceeds ~40% of its 30-day users.
- **Two levels:** assets (with long/short skew) and **pair/basket compositions**. Compositions are the more useful seeds, since 87% of trading is multi-leg.
- **Where it adds value:**
  1. popular compositions and direction skew into basket/pair generation;
  2. boost the mid-ranked assets (UNI, SUI, AERO, ARB, XMR, ADA…), not the majors;
  3. fix the 20-per-category recency cut so high-interest assets can't silently drop out.
- **Drop notional as a ranking input:** it's dominated by scheduled ADJUST fills and a few large accounts.

## 9. Case study: Long SOL / Short ZEC (2026-09-29)

Question: is the familiarity signal picking up organic user preferences, or Pear's own earlier promoted ideas?

**Answer: neither.** It came from an outside trader on X. Most users trading it were **new to Pear** and copying the tweet.

Raw count: **37 users, 54 opens** (§7's 34 is after bot exclusion).

### How it's represented

- **`executions`:** one `intent='OPEN'` row per open.
  - `context` = `{payload:{legs,type}, position:{}}`. Each leg has an instrument id, a `mode:"USD"` amount and a side. SOL = `5`, ZEC = `214` on Hyperliquid; SOL = `2`, ZEC = `90` on Lighter.
  - `plan` = planned price and quantity per leg.
  - **No source, suggestion, ticket, campaign or referral field.**
- **`position_key`:** a per-user/account hash. All 37 keys have exactly 1 user, so it is **not** the composition key.
- **Fills:** 1 BUY fill for SOL and 1 SELL fill for ZEC per execution.
- **Positions:**
  - `metadata.initialExposureUSD` = signed entry USD per instrument, e.g. `{"5":"+59.8","214":"-66.4"}`.
  - `positions.exposure` = live **coin quantity, not USD** (e.g. `22.46` SOL ≈ $2.7k). Easy to misread.
- **Sizing:** every open used **equal USD on both legs** (web UI default), with varied amounts: $20, $50, $150, $200, $250, $500, $667…
  - Most common: $250 ×4; $667, $150, $500 and $200 ×3 each.
  - This doesn't look like everyone clicking one preset.

### Timing, client and who the users were

| Day (UTC) | Client | Venue | Opens | Users |
|---|---|---|---|---|
| 09-03 | PEARUI | hyperliquid | 1 | 1 |
| 09-27 (19:43 → 23:34) | PEARUI | hyperliquid | 12 | 10 |
| 09-27 (19:43 → 21:54) | PEARUI | lighter | 7 | 6 |
| 09-28 | PEARUI | hyperliquid | 23 | 14 |
| 09-28 | PEARUI | lighter | 4 | 4 |
| 09-29 (to 08:28) | PEARUI | hyperliquid | 7 | 4 |

- **All 54 opens came through the web UI (PEARUI). None came through the agent.**
- **22 of 37 users made their first-ever Pear trade with this pair.**
  - 5 of those 22 already had venue history (imported external fills).
  - 10 users had 1–5 earlier Pear opens; 5 were established users.
  - 21 of 37 opened other trades afterwards (median 1 later open).

### Origin

Nothing Pear-side:
- No `SignalNotification`, `TouchCandidate` or chat trade link with SOL+ZEC in the agent DB.
- The only related row is one agent chat on 09-27 21:55, asking a user how to proceed with an SOL/ZEC position they already held.
- No Pear-authored tweet since 09-24 in the context DB.
- `made_baskets` not checked (quant DB certificate not trusted locally). *(Update 2026-09-29: quant DB read in §12. No SOL/ZEC-style trends were traced to made baskets; only ~2% of generated compositions get opened at all.)*

The context DB's tweet table (`TweetContext`) shows the source:

| UTC | Author | Content (views) |
|---|---|---|
| 09-27 14:21 | trader1sz | recommended to try @pear_protocol for $XMRZEC (11.5k) |
| 09-27 18:08 | trader1sz | Pear the only place for XMR/ZEC cross perp (12.4k) |
| **09-27 19:06** | **trader1sz** | **"does @pear_protocol have $SOLZEC … low risk high reward" (19.6k)** |
| 09-27 20:21 | trader1sz | testing Pear with "a small punt… few hundred dollars" (13k) |
| 09-27 20:33 | hufhaus9 | "tailed on this SOLZEC, link here" (982) |
| 09-27 21:47 | trader1sz | "might punt $NEARZEC long on @pear_protocol" (8.7k) |
| 09-28 12:50 | trader1sz | "i am long $NEARZEC on @pear_protocol" (10.3k) |

- **The first Pear SOL/ZEC open (19:43) came 37 minutes after the $SOLZEC tweet.**
- This also explains §7's **Long XMR / Short ZEC (16 users)** and **Long NEAR / Short ZEC (16 users)**, and the ZEC/NEAR spikes on the 27th–28th.
- Unknown: whether the shared t.co links were Pear share or referral links. The BE read-only role has no referral table.

### Outcomes

| Group | Positions | Users | Median hold | Result |
|---|---|---|---|---|
| Closed by user | 21 | 15 | **0.2 h** (15 closed < 1 h) | 5 winners / 16 losers, total −$44, median −0.20% (≈ fees) |
| Closed by netting | 4 | 4 | 19 h | 1 / 3, total −$236 |
| Still open (09-29) | 29 | 29 | 26.8 h so far | **25 / 29 in profit, ≈ +$3.2k**, median **+9.3%** on each leg's notional |

Open positions were marked to the latest Pear fill price (HL: SOL $119.6, ZEC $1,421.4; Lighter: SOL $118.98, ZEC $1,404.5), so treat that as an estimate. ZEC fell from ~$1,600 to ~$1,420 over the period.

Two behaviours: quick "test Pear with a small punt" trades that get abandoned, and holders who are currently winning.

### `basketPositions` check

The canonical basket key is sha256 of `{"long":[sorted ids],"short":[sorted ids]}` (`deriveBasketKey`, `@pear-protocol/utils`). Queried read-only via `POST /markets/baskets/positions`:

| Venue | Composition | Opened | Traders |
|---|---|---|---|
| hyperliquid | Long SOL / Short ZEC | 43 | 28 |
| hyperliquid | Long ZEC / Short SOL | 3 | 3 |
| lighter | Long SOL / Short ZEC | 11 | 9 |
| lighter | Long ZEC / Short SOL | 0 | 0 |

HL + Lighter = **54 opened / 37 traders**, matching the BE count exactly. Almost all of the composition's all-time activity (every open but the one on 09-03) is from 09-27 onward.

- `basketPositions` is therefore a valid Pear-only popularity source per composition.
- It is cumulative and has no attribution, so it can't show *why* or *how recently*.

### Implications for the familiarity layer

- The top recent compositions are **influencer-driven acquisition bursts**, not steady preferences of existing users. A plain distinct-user count ranks SOL/ZEC #3 over 30 days and #1 over 7 days.
- Signals that would separate a preference from a burst (all computable from BE tables):
  - burst onset: largest single day's share of users;
  - share of first-time Pear users;
  - share closed within 1 hour;
  - share still held or reopened later.
- Bursts like this may deserve their own fast-reacting product (a "trending on Pear now" feed or touch item), separate from "assets you're familiar with".

## 10. Top-10 compositions — behavioural archetypes (2026-09-29)

No scoring formula yet. This section inspects rows to see which behaviours exist before designing the ranking.

Metric definitions:
- **Users:** distinct users with completed OPEN executions, last 30 days. Vaults excluded; **bots not excluded**.
- **Biggest day %:** the busiest day's distinct users ÷ total users.
- **First-time %:** users whose first-ever Pear OPEN was this composition.
- **Established %:** users with more than 5 earlier Pear opens.
- **Repeat %:** users who opened the composition on 2 or more distinct days.
- **Top user's share of opens:** detects single-account churn.

| Pair | Users | Opens | Active days | Biggest day % | First-time % | Established % | Repeat % | <1h close % | Still open % | Agent % | Median open | Top user's share of opens | First seen |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Long AERO / Short UNI | 38 | 48 | 10 | 50 (09-19) | 39 | 47 | 13 | 0 | 21 | 23 | $974 | 8% | 09-19 |
| Long SOL / Short ZEC | 37 | 54 | 4 | 49 (09-28) | 59 | 14 | 5 | 28 | 52 | 0 | $726 | 13% | 09-03 |
| Long ETH / Short BTC | 35 | 98 | 24 | 20 (09-23) | 20 | 54 | 31 | 42 | 6 | 36 | $2,783 | 19% | 08-30 |
| Long BTC / Short ETH | 24 | 99 | 22 | 21 (09-11) | 21 | 58 | 33 | 78* | 2 | 35 | $99* | 27%* | 08-31 |
| Long NEAR / Short ZEC | 18 | 20 | 5 | 39 (09-27) | 33 | 17 | 0 | 15 | 55 | 5 | $968 | 10% | 09-09 |
| Long XMR / Short ZEC | 16 | 22 | 9 | 31 (09-27) | 44 | 25 | 13 | 27 | 23 | 9 | $398 | 23% | 08-31 |
| Long HYPE / Short ETH | 15 | 80 | 28 | 20 (09-11) | 13 | 73 | 33 | 85 | 1 | 4 | $25 | **71% (bot)** | 08-30 |
| Long PUMP / Short SOL | 12 | 16 | 4 | 83 (09-07) | 0 | 75 | 8 | 0 | 0 | 38 | $4,500 | 19% | 09-03 |
| Long SKY / Short BTC | 10 | 11 | 1 | 100 (09-28) | 0 | 90 | 0 | 0 | 73 | 45 | $227 | 18% | 09-28 |
| Long HYPE / Short LIT | 8 | 8 | 6 | 25 (09-04) | 38 | 63 | 0 | 38 | 25 | 25 | $5,011 | 13% | 08-30 |

\*BTC/ETH's short holds and tiny median come from its heaviest openers: `cf12a0e0` (27 opens, bot-rule-flagged in §4), `bc854ed9` (15), `7c1901bc` (12, likely our test account), `b426d481` (10). HYPE/ETH is 57 of 80 opens from `cf12a0e0`.

### Likely sources

| Pair | Likely source | Confidence | Evidence |
|---|---|---|---|
| AERO/UNI | Copy trade of **@koolkrypto223**, amplified by **hufhaus9** | medium | Opens from 03:16 on 09-19 (20 opens, 19 users that day). hufhaus9 at 12:47: "copy trading @koolkrypto223 on his AERO/UNI pair trade … link". koolkrypto223 isn't in our captured tweets, so the original post is unverified. Agent user at 08:38: "can you help me long AERO and short UNI". |
| SOL/ZEC | **trader1sz** tweet | high | See §9. |
| NEAR/ZEC | **trader1sz** | high | $NEARZEC posts 09-27 21:47 and 09-28 12:50. |
| XMR/ZEC | **trader1sz** | high | "short $ZEC and long $XMR" (09-25 20:39, 16k views), repeated 09-27. Small earlier seeding from 08-31. |
| ETH/BTC, BTC/ETH | Organic, a steady majors pair | high | 22–24 active days, no spike, mostly established users. |
| HYPE/ETH | Bot / test churn | high | 71% of opens from one bot-flagged account, $25 median, 85% closed within 1 h. |
| PUMP/SOL | Pear community / affiliated promoter | medium | First opens 08:32 on 09-07, all by established users, $4.5k median. Agent users at 08:36 "what do you think of PUMP/SOL", 08:51 "long PUMP short SOL". hufhaus9 PUMP tweet at 08:44, just after the first opens. |
| SKY/BTC | **Copying one whale's ~$100k TWAP** | high | `00c71f5e` opened at 09:50 on 09-28 (PEARUI), then 876 scheduled ADJUST executions ≈ $99.8k until 13:30. From 09:51, agent users pasted "🔄 TWAP Started · Long SKY 46% / Short BTC 54% · Size: $100,000" ("Can I do something similar on my account"). Followers opened 09:52–15:33, some running their own TWAPs (`b4543b3c` ≈ $58k). The 16 SKY `TouchCandidate`s that day were `s3_position` funding check-ins, not suggestions. |
| HYPE/LIT | Organic market narrative | medium | "HYPE/LIT hit a new all time low" tweets (x256xx, HYPEconomist) on 09-03/04; spread over 6 days. |

Pear-side check: the agent DB is production-like (1.3–2.3k chat messages a week, 380 wallet links, journal rows up to 09-29). For every composition there was **no `SignalNotification` or suggestion-type `TouchCandidate`** pushing it. Agent chats show users asking the agent to place a pair they had already chosen.

hufhaus9 looks **Pear-affiliated**: leaderboard prize posts ("prize pool is 10% of pear treasury"), feature promotion, "welcome" posts. That makes AERO/UNI and PUMP/SOL partly Pear-promoted in effect, even though no Pear system pushed them. Unverified whether the account is official.

### Archetypes

| Archetype | Examples | Signature in the numbers |
|---|---|---|
| **Durable pair** | ETH/BTC (both directions) | Many active days, biggest day ≤ ~20%, mostly established users, ~30% repeat users |
| **Social spike / new-user acquisition** | SOL/ZEC, XMR/ZEC, NEAR/ZEC, AERO/UNI | Biggest day 30–50%, **33–59% first-time**, few repeats, many still open |
| **Community / campaign pair** | PUMP/SOL (AERO/UNI in part) | Biggest day ≥ 80%, **0% first-time**, established users, large size |
| **One-day whale copy** | SKY/BTC | 1 active day (100%), 0% first-time, 90% established, heavy agent share |
| **Bot churn** | HYPE/ETH, part of BTC/ETH | One account dominates opens, tiny size, > 75% closed within 1 h |
| **Narrative pair** | HYPE/LIT | Moderately spread, small user count, matches market chatter |

*(Update 2026-09-29: per Jeyd, these archetypes are diagnostics/caveats only, not a separate layer to build. See §11.)*

**Takeaway:** only BTC/ETH is a durable preference in the top 10. The layer as specced would mostly rediscover influencer and copy-trade bursts, not Pear's own suggestions (none found). Design the ranking only after deciding which archetypes each product surface should use.

## 11. Product decision: 65/35 Pear-native asset ranking (Jeyd, 2026-09-29)

Status report for Jeyd/Huf (Notion, private draft): [Familiar-Asset Pair/Basket Ideas](https://app.notion.com/p/3ea4641ea90c81af8bebe1578a9c28e0)

### Decision

- **Two Pear-native asset signals, both counted in distinct traders** (not fills or notional):
  - **Current** = assets most held by distinct Pear traders right now.
  - **Historical** = assets most broadly opened in completed Pear OPEN executions over a longer window.
- `final_asset_score = 0.65 × current_popularity + 0.35 × historical_popularity`
- Normalize tickers across venues and remove vault/bot/test noise before scoring.
- Take the top ~20 assets → narrative analysis → generate **pair or basket** ideas (equal importance) → existing quant validation stays the gate.
  - Popularity decides where to look; narrative + quant decide whether an idea is good.
- **No separate trending system; don't over-focus on social attribution.** If Pear traders are trading an asset, that is valid signal whatever caused it. The §9–§10 findings remain caveats/diagnostics only.
- Composition and direction data are supporting context only (87% of opens are multi-leg). No composition recommender yet.

### Definitions I chose (flag to Jeyd if different)

- **Current:** distinct users holding a `positions.status IN ('OPEN','OPENING')` position with `creation_source='ENGINE'` and `origin<>'EXTERNAL'`, with a leg in the asset (legs from `metadata.initialExposureUSD`). Divided by all current holders.
- **Historical:** distinct users with a completed OPEN execution touching the asset (Pear-placed fills), **last 90 days**. Divided by all 90-day openers.
- **Bots:** the best-trade rule over the last 30 days (median hold < 5 min AND median open < $50). Vaults excluded.

### Preview (read-only, 2026-09-29)

Base: **153** users currently hold an open Pear position; **480** opened in the last 90 days. Pear OPEN history starts **2026-05-26**, so the 90-day window covers ~4 months of data.

| Rank | Asset | Current holders | Current % | Openers (90d) | Historical % | Score % | Hist. rank | Current rank |
|---|---|---|---|---|---|---|---|---|
| 1 | BTC | 49 | 32.5 | 253 | 53.6 | 39.9 | 1 | 2 |
| 2 | ZEC | 57 | 37.7 | 178 | 37.7 | 37.7 | 4 | 1 |
| 3 | SOL | 48 | 31.8 | 167 | 35.4 | 33.0 | 5 | 3 |
| 4 | HYPE | 38 | 25.2 | 215 | 45.6 | 32.3 | 3 | 4 |
| 5 | ETH | 36 | 23.8 | 225 | 47.7 | 32.2 | 2 | 5 |
| 6 | NEAR | 30 | 19.9 | 116 | 24.6 | 21.5 | 7 | 6 |
| 7 | LIT | 22 | 14.6 | 146 | 30.9 | 20.3 | 6 | 8 |
| 8 | UNI | 26 | 17.2 | 94 | 19.9 | 18.2 | 10 | 7 |
| 9 | PUMP | 20 | 13.2 | 105 | 22.2 | 16.4 | 8 | 10 |
| 10 | ENA | 17 | 11.3 | 98 | 20.8 | 14.6 | 9 | 12 |
| 11 | AERO | 21 | 13.9 | 73 | 15.5 | 14.5 | 15 | 9 |
| 12 | SUI | 17 | 11.3 | 81 | 17.2 | 13.3 | 12 | 12 |
| 13 | LINK | 18 | 11.9 | 65 | 13.8 | 12.6 | 18 | 11 |
| 14 | XRP | 13 | 8.6 | 88 | 18.6 | 12.1 | 11 | 17 |
| 15 | WLD | 16 | 10.6 | 67 | 14.2 | 11.9 | 17 | 14 |
| 16 | ADA | 14 | 9.3 | 72 | 15.3 | 11.4 | 16 | 15 |
| 17 | AAVE | 14 | 9.3 | 62 | 13.1 | 10.6 | 21 | 15 |
| 18 | XMR | 11 | 7.3 | 74 | 15.7 | 10.2 | 13 | 21 |
| 19 | BNB | 12 | 7.9 | 59 | 12.5 | 9.5 | 22 | 20 |
| 20 | AVAX | 11 | 7.3 | 55 | 11.7 | 8.8 | 26 | 21 |

Just below: ONDO 8.8, VVV 8.5, ETHFI 8.5, PEPE 8.3, DOGE 8.2.

### Main caveat (verified by the preview)

- **Ranks 1–10 = 9/10 of today's market-OI top 10** (`pickDailyAssets`). UNI is new; XRP drops to #14.
- The incremental value is likely:
  1. the **65% current signal**, which reorders toward what Pear traders hold now (ZEC #1 current vs. #4 historical; AERO #9 current vs. #15 historical);
  2. **ranks 11–20**, the Pear-specific mid-tier (AERO, SUI, LINK, WLD, ADA, AAVE, XMR, BNB, AVAX), which today get only weekly sentiment;
  3. **composition/direction context** (e.g. HYPE mostly longed; ADA/LTC/APT mostly shorted; SOL/ZEC and BTC/ETH common pairings).
- The top 20 contains **no HIP-3 equities**. Whether the forced equity pass and the 7 hard-coded HIP-3 sentiment picks stay is an open decision.
- Of ranks 11–20, SUI and ADA were outside the 20-per-category candidate list in the §6 snapshot (AERO, LINK, WLD, AAVE, XMR, BNB, AVAX were inside). This depends on the same unverified production cutoff.

### Integration points

- **`pickDailyAssets`** (`apps/context/src/sentiment/services/sentiment-cron.service.ts`): feed the ranked top ~20 into daily narrative. Today's budget is ~17 assets (10 OI + 7 HIP-3), so the cost is similar.
- **`MadeBasketsService.generateBasket`** (apps/context): the ranked universe plus composition/direction context enter as input. It already produces both baskets and 1x1 pairs; quant validation is unchanged.
- One nightly read-only aggregation (BE access already exists in apps/agent via `best-trade.reader.ts`). No new tables or ingestion.

### Do not build yet

- No separate trending product.
- No automated social attribution.
- No complex behavioural classifier.
- No recommendation/scoring engine beyond the 65/35 asset ranking.
- No cohort modelling or per-user generation.
- No composition-level recommender.

### Next experiment

1. Compute the 65/35 ranking daily for ~7 days (read-only); check stability and how much ranks 11–20 differ from the OI picks.
2. Verify the 20-per-category cutoff and quant/sentiment mappings for the top 20 against production.
3. Define the success metric (`basketPositions` + Pear OPENs on generated compositions) before shipping.

## 12. Pre-implementation validation of the 65/35 plan (2026-09-29)

All read-only; no repo code or production data changed.

Sources:
- Pear BE (read-only role).
- Context DB, in `BEGIN READ ONLY`.
- Quant DB, via libpq-compatible SSL (`uselibpqcompat=true`: encrypted without CA verification — the same as the app's Prisma `sslmode=require`).
- Agent DB.
- One `GET asset_index:full` from agent Redis.

**Verdict:** nothing blocks the plan at the data level, and 65/35 is about as stable as the alternatives. But as the code stands, the ranking barely reaches the generator, and generated ideas are rarely traded today (~2%).

### Corrections to earlier findings

- **§6 "dev DB" caveat — wrong.**
  - `AssetSentiment` is **latest-only**: `ingestAssetRun` does `updateMany where assetId`, overwriting `runAt` and `category` (213 assets → 213 rows).
  - The missing 09-08…09-26 rows are an artifact of that, not a gap. The context DB behind `apps/agent/.env` **is production**.
  - Independently confirmed from Railway logs in [[Asset Sentiment Playground — Research & Experiments]].
- **"HYPE has no quant ID" — wrong.** That came from a **test fixture** (`packages/shared/src/asset-index/reader.spec.ts:760`). Production has `HYPEUSDT` in `asset_platforms` and in the asset index. (Also claimed in [[Better Trading]] §5.)

### 1. Candidate cutoff

- **Every basket path** gets candidates from `getAssetRowsByCategory` (take 20 per category, ordered by `runAt`):
  - OpenRouter `assets_by_category`;
  - Claude MCP bridge (`apps/context/src/claude/pear-tools.ts:117`);
  - Claude precomputed list;
  - Claude research-stage `buildCandidateUniverse`.
- Daily 12:00 picks get the newest `runAt` and rank first. Everything else shares a weekly-run `runAt`, in large tied groups, and Postgres orders tied rows arbitrarily.

Category sizes (fresh within 14 days) vs. the 20-slot cut:

| Category | Fresh assets | Distinct runAt | Largest tie group |
|---|---|---|---|
| Others | 44 | 4 | 39 |
| Layer1 | 39 | 4 | 28 |
| Defi | 35 | 4 | 30 |
| Meme | 29 | 4 | 24 |
| AI | 14 | 2 | 13 |
| Gaming | 14 | 2 | 13 |
| Layer2 | 12 | 2 | 11 |
| HIP-3 (7 categories) | 1–8 each | 1–4 | ≤ 4 |

Pear top-20 position at today's 14:00 run:

| Asset | Category | runAt | Rank | Tie group | Outcome |
|---|---|---|---|---|---|
| BTC, ETH, HYPE, SOL, XRP, ZEC | Layer1 | 09-29 12:00 | 1–6 | 6 | always in |
| NEAR | AI | 09-29 12:00 | 1 | 1 | in |
| LIT | Others | 09-29 12:00 | 1 | 1 | in |
| ENA | Defi | 09-29 12:00 | 1 | 1 | in |
| PUMP | Meme | 09-29 12:00 | 1 | 1 | in |
| XMR | HIP_3_crypto | 09-28 08:49 | 1 | 1 | in |
| AVAX, BNB, SUI | Layer1 | 09-28 17:16 | 7–34 | 28 (14 slots) | ~50% chance each |
| UNI, AERO, LINK, AAVE | Defi | 09-28 17:16 | 2–31 | 30 (19 slots) | ~37% dropped each |
| WLD | AI | 09-28 17:16 | 2–14 | 13 | in |
| ADA | Layer1 | 09-28 08:49 | 35 | 3 | **always dropped** |

- **Blocks?** It blocks the *effect*, not the build.
- **Smallest safe fix:** add the ranked top 20 to `pickDailyAssets` as a **union**, keeping the OI top 10 + 7 HIP-3. They then get the newest `runAt` daily and survive deterministically (≤ 10 per category; Layer1 would hold 10).
  - Daily sentiment goes from ~17 to ~27 assets. The OI top 10 is already inside the Pear top 20.
  - Don't change the shared `take: 20`.

### 2. Asset coverage (all 20 PASS)

The chain is: BE ticker → normalized → `AssetSentiment` row/category → listed in `asset_platforms` → quant ID → asset-index core listing → `pair_stats_4h`.

| Assets | Category | HL / Lighter / SYMM listed (days_old) | Quant ID | Asset-index listing | Pair partners (4h, 09-29 08:00) |
|---|---|---|---|---|---|
| BTC, ZEC, SOL, HYPE, ETH, XRP, SUI, ADA, BNB, AVAX | Layer1 | ✅ ✅ ✅ (146) | `<T>USDT` | `<T>(core)` | 770 |
| NEAR, WLD | AI | ✅ ✅ ✅ | ✅ | core | 770 |
| LIT | Others | ✅ ✅ ✅ | ✅ | core | 770 |
| UNI, ENA, AERO, LINK, AAVE | Defi | ✅ ✅ ✅ | ✅ | core | 770 |
| PUMP | Meme | ✅ ✅ ✅ | ✅ | core | 770 |
| XMR | **HIP_3_crypto** ⚠️ | ✅ ✅ ✅ | ✅ | core | 770 |

- Asset index: 800 entries. I replicated `resolveQuantAssetId` and `resolveHlBasketListing` against it.
- `/stats/basket` was not called live; its prerequisites pass.
- XMR is filed under a HIP-3 category, so it only enters baskets when a HIP-3 category is chosen (including the equity pass). A categorisation quirk, not a blocker.
- **Action:** have the ranking job drop or flag assets without a sentiment row or quant ID (PONS-style gaps will recur).

### 3. Equities

- The forced pass is `refreshPlatform(..., allowedCategories: HIP_3_*)` per platform, seeded with the general run (`made-baskets-cron.service.ts`). Its daily sentiment comes from the 7 hard-coded HIP-3 assets in `pickDailyAssets`.
- The Pear top 20 has no equities most days (NVDA, GOLD and MU occasionally cross in).
- But **84 of 308 Pear traders (27%) opened a HIP-3 market in 30 days**, only 325 of 3,105 opens (10%). Interest is real but spread across many tickers.
- **Not a blocker** if the ranking is a union. Replacing `pickDailyAssets` would make HIP-3 sentiment weekly-stale.
- **Options (product decision):**
  - keep the pass separate (equity baskets guaranteed; ranking has no effect on them);
  - fold equities into the ranking (they'd mostly fall below the cut);
  - hybrid: rank HIP-3 assets by 65/35 to choose the pass's daily sentiment assets instead of the hard-coded 7.

### 4. Weight/window stability

Replay at 14:00 UTC for 09-01 → 09-28 (28 days). Current holders ranged 89–149 (jump 114 → 149 on 09-28); 90-day openers 317 → 457.

| Variant | Top-20 swaps/day | Max | Days with 0 swaps | Top-10 swaps/day | Distinct assets ever in top 20 | Top-20 overlap day 1 vs day 28 |
|---|---|---|---|---|---|---|
| **65/35, 90d (chosen)** | **1.89** | 4 | 3 | **0.89** | 34 | 16 |
| 70/30, 90d | 1.96 | 4 | 2 | 0.96 | 35 | 16 |
| 60/40, 90d | 1.93 | 4 | 3 | 0.70 | 33 | 16 |
| 65/35, 30d | 2.15 | 4 | 2 | 0.81 | 35 | 16 |
| 90d history only | 0.44 | 1 | 15 | 0.15 | 26 | 16 |
| 30d history only | 0.67 | 2 | 10 | 0.19 | 28 | 15 |
| Current holders only | 2.56 | 4 | 0 | 1.22 | 43 | 15 |

- Churn sits at ranks ~15–25 (WLD, ARB, PEPE, BNB, GOLD, VVV, ETHFI…). The top 10 is nearly fixed.
- 65/35 top 20 on 09-01: BTC, ETH, HYPE, SOL, LIT, ZEC, XRP, PUMP, SUI, ADA, ENA, APT, NEAR, TAO, UNI, XMR, AAVE, AVAX, PENGU, DOGE.
- 65/35 top 20 on 09-28: BTC, ZEC, HYPE, ETH, SOL, LIT, NEAR, UNI, PUMP, SUI, ENA, AERO, XRP, LINK, WLD, AAVE, XMR, ADA, AVAX, ARB.
- Caveat: the bot filter used today's 30-day bot set (minor look-ahead).
- **Not a blocker.** Optional smoothing: keep an asset until it falls below rank 25.

### 5. Measurement feasibility

- **Baseline:** 778 unique generated compositions over 21 days (797 `made_baskets` rows) → **17 opened on Pear within 7 days of generation, by 10 distinct users** (exact long/short match after normalization). ≈ 2%.
- **Generator shape (21 days):** only **3 of 797 are 1x1 pairs (0.4%)**; 249 directional, 545 multi-leg. "Pair = basket" isn't how it behaves today.
- **Venues:**
  - Pear-engine OPENs over 60 days: hyperliquid 4,796 (394 users), lighter 1,455 (133), bybit 5 (1). **No SYMM.**
  - Yet 255 of 797 generated rows target SYMMIO (HL 289, Lighter 253). Where SYMM trades happen is unverified.
- **Run timing:** `made_baskets` wrote at 03:00 until mid-September, then 14:00 daily from 09-21 (the Hetzner cutover).

What attribution is possible:
- **Exposure-based (feasible now):** `made_baskets` (`id`, `run_at`, legs as display names) → later completed OPENs with the same composition or assets.
- **`basketPositions` (feasible):** legs → instrument ids → `deriveBasketKey`. It's cumulative, so it needs snapshots at generation time and later. A cross-check only.
- **Causal (missing):**
  - `executions.context` has legs only, with no source field.
  - The web UI records no suggestion ID.
  - Agent tickets carry only `signalNotifId` (`trade-ticket.types.ts:160`; funnel writes in `execute-ticket.usecase.ts:278,509`). There is no `madeBasketId`.
- **Action:** snapshot the baseline before shipping. Later, add an optional `madeBasketId` to agent tickets following the `signalNotifId` pattern. Web-UI attribution needs frontend work outside this repo.

### 6. Diagnostics only (not ranking features)

Top 20, last 30 days (vaults excluded, bots not excluded):

| Asset | Users | Active days (of 31) | Busiest day % | First-time % | Closed < 1 h % |
|---|---|---|---|---|---|
| BTC | 140 | 31 | 21 | 21 | 44 |
| ZEC | 134 | 31 | 25 | 41 | 17 |
| ETH | 133 | 31 | 21 | 23 | 44 |
| HYPE | 125 | 31 | 17 | 23 | 32 |
| SOL | 115 | 31 | 23 | 33 | 23 |
| NEAR | 90 | 29 | 23 | 24 | 19 |
| UNI | 79 | 29 | 27 | 33 | 15 |
| LIT | 77 | 31 | 16 | 19 | 20 |
| ENA | 71 | 31 | 17 | 17 | 15 |
| PUMP | 70 | 30 | 31 | 19 | 13 |
| AERO | 60 | 20 | 33 | 33 | 6 |
| SUI | 59 | 25 | 22 | 24 | 10 |
| XRP | 52 | 28 | 25 | 12 | 17 |
| XMR | 47 | 21 | 17 | 26 | 16 |
| LINK | 45 | 22 | 24 | 22 | 15 |
| WLD | 44 | 21 | 25 | 14 | 7 |
| ADA | 40 | 26 | 20 | 30 | 13 |
| AAVE | 37 | 22 | 19 | 19 | 4 |
| AVAX | 36 | 23 | 19 | 28 | 14 |
| BNB | 33 | 25 | 18 | 21 | 19 |

- **At the asset level, bursts are diluted:** every top-20 asset traded on 20–31 of 31 days, and no single day holds more than 33% of its users. The composition-level spikes (SOL/ZEC 49% in one day, SKY/BTC 100%) mostly disappear, which supports ranking by asset.
- BTC/ETH's 44% fast-close rate is bot-inflated (`cf12a0e0`, `7c1901bc`). The ranking's bot filter handles this.

### A) Safe to implement now

- The read-only ranking computation (SQL verified).
- Union into `pickDailyAssets`.
- Ranked universe + long/short skew as context in the `generateBasket` input.
- Per-run logging of the ranked universe, the diagnostics, and baseline generated-idea open rates.
- **Agent-side use:** the chat pair/basket nodes already receive a personal context block (trade profile + exposure) via `node-executor.service.ts:105-125`. Add one "assets Pear traders are active in now" line from the same Redis key. No new node or tool. Optionally rank `quant_recent_made_baskets` by it.

### B) Technical blockers

1. **Transport into apps/context.**
   - BE access (`pg` + `BE_READONLY_DATABASE_URL`) exists only in `apps/agent`.
   - `SentimentRunnerModule` (the 12:00 one-shot) has no agent-Redis client.
   - Options: an apps/agent nightly job (before 12:00 UTC) writing a Redis key, plus a reader wired into the runners; or give apps/context the BE URL.
2. **Unknown production generator config.** If the Claude research stage is on, it shortlists 4 per category and researches at most 12 assets (`CLAUDE_BASKET_*`), which caps what's considered. Check `BASKET_LLM_PROVIDER` / `CLAUDE_BASKET_RESEARCH_STAGE` on Hetzner.
3. **Pair output ≈ 0 (0.4%).** Equal pair importance needs a generator change (a dedicated pair pass or prompt change).

### C) Product decisions (Jeyd / another human)

- Equities: separate pass vs. fold in vs. hybrid.
- Pair vs. basket balance: pairs per day, and whether a dedicated pass.
- SYMM: a third of generation targets a venue with no Pear-engine OPENs visible.
- Confirm signal definitions (current = open Pear-engine positions; historical = 90d).
- Accept the sentiment cost rise (~17 → ~27 assets/day).
- Success metric and target lift (baseline ≈ 2% exact-composition opens within 7 days).

### D) Files/modules likely to change

- **New in `apps/agent`:** ranking reader/compute (e.g. `src/pear-asset-ranking/`), `src/main-pear-asset-ranking.ts` + runner module, a Hetzner unit/timer in `apps/agent/deploy/hetzner/`. Reuse the `best-trade/best-trade.reader.ts` and `ai/background/user-value-score.processor.ts` patterns.
- **`packages/shared`:** a Redis-key reader for the ranked universe.
- **`apps/context/src/sentiment/services/sentiment-cron.service.ts`:** `pickDailyAssets` union.
- **`apps/context/src/sentiment/sentiment-runner.module.ts`, `apps/context/src/basket-runner.module.ts`:** bind the agent-Redis client.
- **`apps/context/src/sentiment/services/made-baskets.service.ts`:** context block. A pair pass touches `made-baskets-cron.service.ts` + basket prompts.
- **Agent context line:** `apps/agent/src/ai/dag/executor/node-executor.service.ts` (+ a small personalization helper).
- **Later, for measurement:** `apps/agent/src/ai/trade-tickets/trade-ticket.types.ts`, `execute-ticket.usecase.ts` (optional `madeBasketId`).

### Suggested order

1. Snapshot the baseline.
2. Ranking job in `apps/agent` + the agent context line. The smallest visible slice, independent of the basket pipeline.
3. In parallel: `pickDailyAssets` union + `generateBasket` context.
4. After the equities and pair-count decisions: the pair pass.

## Open questions

- [x] Check the 20-per-category cut against **production's** context DB, and what the 14:00 UTC basket run actually saw. *(Done 2026-09-29, §12.1: ADA always dropped; SUI/BNB/AVAX ~50%, UNI/AERO/LINK/AAVE ~37% dropped. Fix: union into `pickDailyAssets`.)*
- [x] Why the agent `.env` context DB has no sentiment rows from 09-08 to 09-26 (dev DB, or pre-#1034 rollbacks?). *(Resolved: latest-only table, production DB. §12 corrections; [[Asset Sentiment Playground — Research & Experiments]].)*
- [x] Read `made_baskets` once the quant DB certificate is trusted locally. *(Done via libpq-compatible SSL, §12.5.)*
- [ ] Confirm what caused the spikes on the 19th (AERO/UNI) and 27th–28th (ZEC/NEAR). *(Update 2026-09-29: the 27th–28th spike traced to outside X posts, see §9. AERO/UNI on the 19th is still open.)*
- [ ] Whether the shared t.co links in the §9 tweets were Pear share/referral links.
- [ ] Run the same origin trace for Long AERO / Short UNI and Long SKY / Short BTC. *(Update 2026-09-29: done in §10. AERO/UNI → koolkrypto223 via hufhaus9, medium confidence; SKY/BTC → copy of one ~$100k TWAP, high confidence.)*
- [ ] Find koolkrypto223's original AERO/UNI post (not in `TweetContext`).
- [ ] Confirm whether hufhaus9 is an official Pear account.
- [ ] How the SKY/BTC "TWAP Started" alert reached other users (group chat? screenshot?).
- [x] ~~Decide which archetypes each product surface should use (familiar-asset seeding vs. "trending on Pear now" vs. excluded), then design the ranking.~~ *(Resolved 2026-09-29: Jeyd chose a single 65/35 current/historical asset ranking, no trending layer — §11.)*
- [ ] Confirm my signal definitions with Jeyd: current = open Pear-engine positions; historical = 90-day completed OPENs; each normalized by total traders in its window.
- [x] Is the 65/35 ranking stable day to day, or reshuffled by single-day bursts? *(Done, §12.4: ~1.9 top-20 swaps/day, 0.9 in the top 10; comparable to 70/30 and 60/40.)*
- [ ] Keep or drop the forced HIP-3 equity pass, given the Pear top 20 has no equities? *(Options and consequences in §12.3; still a product decision. 27% of traders touched HIP-3 in 30 days.)*
- [x] Map the top 20 to quant IDs and sentiment categories (HYPE quant-ID gap; PONS-style category gaps). *(Done, §12.2: all 20 PASS. The HYPE gap was a test fixture. XMR is categorised HIP_3_crypto.)*
- [ ] Pair vs. basket balance: the generator makes 0.4% 1x1 pairs. How many pairs a day, and a dedicated pass? (§12.5)
- [ ] SYMM: 255 of 797 generated rows target SYMMIO, but the BE shows no Pear-engine OPENs there. Keep or drop, or confirm where SYMM trades happen.
- [ ] Production generator config on Hetzner: `BASKET_LLM_PROVIDER` / `CLAUDE_BASKET_RESEARCH_STAGE` (the research stage caps at 4/category, 12 total).
- [ ] Success metric and target lift vs. the ≈ 2% baseline (§12.5).
- [ ] Decide the ranking logic, then the architecture in [[Better Trading]].

## Evidence

- Queries: session scratchpad scripts (read-only; the BE role enforces read-only, the context DB was queried inside `BEGIN READ ONLY`).
- Code paths: `apps/context/src/sentiment/services/sentiment-cron.service.ts` (`pickDailyAssets`), `packages/shared/src/services/sentiment.service.ts` (`getAssetRowsByCategory`), `apps/agent/src/best-trade/best-trade.reader.ts` / `best-trade.ts` (BE SQL and bot rule).
- Related: [[Better Trading]], [[My Work at Pear Protocol]], [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner]].
