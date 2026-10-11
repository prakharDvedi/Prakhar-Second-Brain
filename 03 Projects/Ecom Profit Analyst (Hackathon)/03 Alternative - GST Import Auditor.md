# Alternative idea: GST import auditor (Zoho Books)

Status: scoped, not chosen. The e-commerce plan in [[00 Index]] is still the default until the go/no-go check below passes.

Idea source: advice to find a time-consuming accountant task around ERP tools (Zoho, NetSuite, QuickBooks) and the pain of foreign vendors selling in India under GST. Rules research was done on 2026-10-09 from secondary sources only (consultancy blogs). Nothing here is verified against the statute.

## One-line scope

An agent that audits the foreign-vendor bills in a Zoho Books (India) organisation, finds the ones booked wrongly for reverse charge (RCM), fixes them in a test org and proves the fix by reading the bill back.

## In scope

Buyer side only. An Indian business buying services from a foreign vendor.

Checks, each a coded rule:
1. Vendor has `gst_treatment = overseas` but a bill has `is_reverse_charge_applied = false`
2. Line item has no `reverse_charge_tax_id`
3. Line tax rate or `tax_id` does not match the service type
4. `destination_of_supply` is wrong (it defaults to the org home state)
5. Line has no `hsn_or_sac`
6. Vendor marked `business_none` or `consumer` when it is a foreign business vendor
7. Duplicate `bill_number` for the same vendor

## Out of scope

- Foreign seller side (OIDAR, GSTR-5A, simplified registration). That is the vendor's compliance, not the buyer's books.
- Import of goods and Bill of Entry matching (a good later case, not for 2 days)
- Filing returns, GSTR-2B reconciliation
- Anything that avoids tax. The tool only finds mistakes and missed credits.

## Agent loop

```text
list overseas vendors  (GET /contacts)
→ list bills in the period  (GET /bills)
→ for each bill: get it  (GET /bills/{id})
→ run the rule checks (deterministic function)
→ ambiguous line? agent reads the description, proposes a service type, looks up SAC and rate from a small table
→ record each finding with the evidence
→ propose a fix
→ apply the fix in the test org  (PUT /bills/{id})
→ read the bill back  (GET /bills/{id})
→ code re-runs the rules on the stored bill. Pass only if all checks now pass
→ report: bills checked, errors found, fixed and verified, left for a human, RCM liability total vs ITC total
```

Where the model decides: classifying a service from free text, picking what to investigate (vendor history, earlier bills with RCM), whether a fix is safe to apply or should go to a human.

Where code decides: every rule, every rate lookup, and the final pass or fail.

## Verification design

- Ground truth: I plant the errors myself in the test org through the API, so the answer key is known.
- Per fix: read back from Zoho, re-run the rule engine on the stored bill.
- Totals: RCM output tax and RCM input credit should match for eligible bills. A mismatch is flagged.
- Eval: N runs, score = planted errors found, fixes verified, wrong fixes (should be 0), calls used, cost, time.

## Zoho Books API facts (from official docs, fetched 2026-10-09)

- Base URL for India: `https://www.zohoapis.in/books/` (docs say replace .com with .in)
- Every call needs `organization_id` (from `GET /organizations`)
- Rate limit 100 requests per minute per org. Daily caps by plan: 1000 Free, 2000 Standard, 5000 Professional, 10000 above. Over the limit returns HTTP 429 (error codes 44, 45, 1070). This is a natural recovery case for the demo.
- Auth: OAuth 2.0 with a Self Client from the Zoho Developer Console. Access token valid 1 hour, refresh token until revoked.
- Scopes: `ZohoBooks.bills.READ,UPDATE,CREATE`, `ZohoBooks.contacts.READ,CREATE`, `ZohoBooks.settings.READ` (taxes live under settings)
- Bill endpoints: `POST /bills`, `GET /bills`, `GET /bills/{id}`, `PUT /bills/{id}`
- Bill fields (India): `gst_no`, `gst_treatment` (`business_gst`, `business_none`, `overseas`, `consumer`), `source_of_supply`, `destination_of_supply`, `is_reverse_charge_applied`, `is_inclusive_tax`
- Line fields: `reverse_charge_tax_id` (India, GCC, South Africa), `tax_id`, `hsn_or_sac`, `account_id`
- Required to create a bill: `vendor_id`, `bill_number`, `organization_id`
- Vendor contact fields: `gst_treatment`, `gst_no`, `place_of_contact` (not applicable for overseas contacts)

## Not checked

- Whether the free trial includes API access, and which daily cap applies during the trial. The docs list quotas per plan and do not mention trials.
- The India accounts URL for tokens. Docs say to use your own data centre accounts URL. `accounts.zoho.in` is my guess.
- Whether a test org can be set up for India GST without a real GSTIN.
- The `place_of_supply` field. The bill page marks it GCC only, India uses `source_of_supply` and `destination_of_supply`.
- Contact and bill pages were read in part. Some of the long pages were truncated.
- All GST rules. Sources disagree in places and a CA should confirm the rule table.
- Which rate each service type gets. Start with a table of 6 to 8 service types and have a CA confirm it.

## Go/no-go check (about 1 hour, before the build period)

1. Zoho India trial works, a test org with GST turned on exists
2. A token with the scopes above can call `GET /organizations` and `GET /bills`
3. `POST /bills` can create a bill for an overseas vendor with the RCM flag
4. A CA or accountant confirms the 7 checks and the rate table
Any fail means stay with e-commerce.

## Build plan, 2 days

Reuses the loop, ledger, caps, resume, neatlogs, Entire and cfo.ai from [[01 Build Plan]].

Day 1
- Zoho org, auth, `zoho.py` client with token refresh and 429 backoff
- `rules.py` (the 7 checks) with a test per rule
- Seed script that plants about 25 bills with known errors
- Agent loop in read-only audit mode, first neatlogs trace
- 2 public posts

Day 2
- Fix and read-back verification
- Recovery: 429, expired token, API error on PUT, human-review path
- Evals, read traces, fix 2 real failures, keep before and after
- cfo.ai plan (customer: small CA firms and finance teams, priced per org or per month)
- Video, posts, submit

Cut order if behind: checks 6 and 7 first, then the vendor history drilldown. Never cut read-back verification or evals.

## Comparison with the e-commerce idea

| | E-commerce profit | GST import auditor |
|---|---|---|
| Real pain, paying buyer | medium | strong |
| Original | low | higher |
| Verification | strong, data only | strong if rules are right, plus real ERP read-back |
| Risk of being wrong | low | higher (tax rules, API unknowns) |
| Data ready | needs download | needs Zoho setup and seeding |
| Demo feel | terminal and report | real accounting software |

## Related
[[00 Index]] · [[01 Build Plan]] · [[02 Decisions]]

## Update 2026-10-10: generated test data instead of a Zoho org

Decision by me: the GST import auditor is now the project. The repo was renamed to https://github.com/prakharDvedi/gst-import-auditor and issues #1 to #11 were rewritten for it (same stage numbers as [[00 Index]]).

This supersedes the "Zoho trial", "Data ready" and "Go/no-go" parts above:

- Test data is generated by us. `scripts/seed_books.py` with a fixed seed makes a fake organisation (overseas vendors, Indian vendors, about 25 bills with planted errors) and writes `data/truth.json` as the answer key.
- A small books client has the calls the agent needs: list vendors, list bills, get bill, update bill. Default backend reads the generated data. A real Zoho backend is optional and only added if time is left.
- Field names follow the Zoho Books India docs checked on 2026-10-09, so the shape stays realistic. The README and video say plainly that the data is generated.
- The fake client can return 429, an expired token and an update error on a flag. That covers the recovery stage (#7) without hitting a real rate limit.
- Go/no-go now: only step 4 still matters (a CA or accountant confirming the 7 checks and the rate table). Steps 1 to 3 (trial, token, POST /bills) are no longer blocking.
- Still not verified: all GST rules and the rate table. Same caveat as above.
