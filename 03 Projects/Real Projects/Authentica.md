# Authentica

*Part of [[Projects]] → [[Real Projects]]*

**Repo:** [Authentica](https://github.com/prakharDvedi/Authentica)
**Visibility:** Public
**Language:** TypeScript
**Description:** Blockchain-backed provenance system for verifying AI-generated content, prompts, and creator ownership.

---

# Authentica

A blockchain-based provenance system for AI-generated content. Cryptographically links a creator, their prompt, the AI output, and a timestamp — storing the proof immutably on Ethereum.

---

## Problem

Generative AI makes it trivial to produce content but impossible to prove authorship after the fact. There is no reliable way to verify who created a specific output, what prompt they used, or when it was made. Authentica solves this with tamper-proof on-chain records.

---

## How It Works

```
Prompt + Creator + Timestamp
         │
         ▼
    SHA-256 Hash
         │
         ├── Stored on Ethereum (Sepolia) via ProofOfArt.sol
         └── Content stored on IPFS (Pinata)
                    │
                    ▼
          Verifiable PDF Certificate with QR code
```

Anyone can verify a certificate by submitting the hash to the `/verify` page. The contract returns the original creator address, timestamp, and IPFS link.

---

## Features

**Core**
- SHA-256 cryptographic proof linking prompt, output, creator, and timestamp
- Immutable on-chain storage via Solidity smart contract (Sepolia testnet)
- Decentralized content storage on IPFS
- Downloadable PDF certificates with QR verification codes
- WalletConnect / RainbowKit wallet integration

**Security**
- Optional AES-256-GCM encryption for private content, decryptable only via wallet signature
- Multi-method tamper detection: CLIP embeddings, pixel diffing, histogram analysis, structural comparison
- LSB steganography detection using chi-square tests, entropy analysis, and RS analysis
- Face verification (hash-only — no image stored)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14, TypeScript, Tailwind CSS |
| Web3 | Wagmi, RainbowKit, ethers.js |
| Blockchain | Solidity, Hardhat, Ethereum (Sepolia) |
| Storage | IPFS via Pinata |
| AI Generation | Stability AI API |
| Tamper Detection | CLIP embeddings (Python service), canvas pixel analysis |
| Security | SHA-256, AES-256-GCM |

---

## Local Setup

**Prerequisites:** Node.js 18+, MetaMask, Stability AI key, Pinata JWT, WalletConnect project ID

```bash
# Install dependencies
npm install

# Create .env.local
STABILITY_API_KEY=
IPFS_API_URL=https://api.pinata.cloud
IPFS_AUTH=
NEXT_PUBLIC_CONTRACT_ADDRESS=
NEXT_PUBLIC_RPC_URL=
NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID=

# Deploy smart contract
npm run compile-contract
npm run deploy-contract

# Update NEXT_PUBLIC_CONTRACT_ADDRESS with deployed address, then:
npm run dev
```

Open http://localhost:3000

---

## Project Structure

```
├── app/
│   ├── api/
│   │   ├── generate/        # AI generation + hashing endpoint
│   │   ├── compare/         # Tamper detection endpoint
│   │   ├── verify/          # On-chain verification
│   │   └── metadata/        # IPFS metadata retrieval
│   ├── create/              # Creation flow
│   └── verify/              # Public verification page
├── contracts/
│   └── ProofOfArt.sol       # Main Solidity contract
├── lib/
│   ├── ai.ts                # Stability AI integration
│   ├── blockchain.ts        # Contract interaction
│   ├── crypto.ts            # SHA-256 hashing
│   ├── ipfs.ts              # IPFS upload/retrieval
│   ├── steganography.ts     # LSB detection
│   ├── imageComparison.ts   # Tamper detection
│   └── certificate.ts       # PDF generation
└── services/
    └── clip_service.py      # Optional CLIP embedding service
```

---

## Tamper Detection

Two-layer approach:

**Primary:** CLIP embedding comparison — measures cosine similarity between original and submitted image vectors. Catches semantic-level modifications that pixel diffing misses.

**Fallback (no CLIP service):** Canvas-based pixel analysis combining structural similarity, histogram correlation, and raw pixel difference. Runs entirely in-browser.

Steganography detection runs independently using 5 methods — LSB bias analysis, chi-square test, entropy analysis, RS analysis, and sample pair analysis — and returns a confidence score.

---
