---
type: knowledge
domain: Web3 & Trading
confidence: applied
last_reviewed: 2026-08-30
---

# Wallets & Account Binding

## In Simple Terms

Wallet and account integration is about binding a user action to the correct identity and execution target, then making that binding explicit enough that later reads and writes cannot silently drift.

## What I Understand

In Authentica, wallet signatures, Ethereum Sepolia, Solidity, IPFS, and WalletConnect/RainbowKit connect a creator to an immutable provenance record. In Pear, wallet linking and multi-account execution required venue selection, alias resolution, account UUID persistence, and invocation-scoped account pinning. These are different products, but both taught the importance of explicit identity binding.

## Why It Matters

An ambiguous or stale account choice can send a valid transaction to the wrong place. A wallet-linked proof also loses its meaning if the creator, content hash, timestamp, or storage reference is not bound consistently.

## How I Learned This

- [[Authentica]]
- [[02 Work/Pear Protocol/Decisions/Trade-Account Alias Resolution]]
- [[02 Work/Pear Protocol/Features Owned#Multi-venue execution: Lighter integration]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm]]

## Real Failure Modes I've Seen

- A human-readable trade-account alias could collide across supported and unsupported venues; naive fallback was rejected in favor of uniqueness across the full account list.
- Explicit and omitted account-selection calls could resolve to different memoized keys within one invocation.
- `/linkwallet` moved from typed venue arguments to inline venue selection so the user choice was explicit.
- Authentica's proof model depends on a consistent hash linking prompt, output, creator, and timestamp before on-chain storage.

## Engineering Tradeoffs

- UUIDs are unambiguous but poor user-facing identifiers; aliases improve UX only when collisions fail closed.
- A wallet signature provides strong identity evidence, but wallet UX and chain/network configuration add friction.
- Persisting a resolved account UUID improves later consistency, but a stale account binding must be handled deliberately.

## Patterns / Rules I Currently Believe

**Strong evidence:** Resolve identity and execution target before the side effect, and persist the canonical identifier rather than the ambiguous display name.

**Current belief:** User-facing account selection should be explicit whenever more than one execution target exists.

**Open question:** I have not yet operated production wallet custody, chain reorg handling, or transaction replacement flows.

## Examples From My Work

- Pear account-resolution suites: agent 18/18 and MCP 7/7 at review time.
- Authentica stores the hash proof on Ethereum Sepolia and content through IPFS while exposing public verification.

## Things I Still Don't Fully Understand

- Nonce management, confirmations, reorgs, and transaction lifecycle recovery.
- Production key management and wallet security beyond application-level signing flows.

## Related

- [[Trading Execution Safety]]
- [[Multi-Adapter & Multi-Venue Consistency]]
- [[Authentication, Authorization & Trust Boundaries]]
- [[Structured Outputs & Validation]]

