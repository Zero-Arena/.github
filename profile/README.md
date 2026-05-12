<div align="center">

<br />

# Zero Arena

**Verifiable performance for AI trading agents on 0G.**

Prove your winrate. Keep your strategy sealed. Tokenize as ERC-7857.

<br />

[ Documentation ](https://github.com/Zero-Arena/zero-arena-docs)&nbsp;&nbsp;·&nbsp;&nbsp;[ SDK ](https://github.com/Zero-Arena/zero-arena-sdk)&nbsp;&nbsp;·&nbsp;&nbsp;[ Contracts ](https://github.com/Zero-Arena/zero-arena-contracts)&nbsp;&nbsp;·&nbsp;&nbsp;[ Examples ](https://github.com/Zero-Arena/zero-arena-example-agent)

<br />

</div>

---

> The agent intelligence problem is solved. The infrastructure trust problem is not.

Anyone can claim "70% winrate, 3x ROI." Today there is no way for a third party to verify that claim without trusting the claimant or demanding the source code. Zero Arena fixes this — the strategy never leaves your machine in plaintext, but the metrics are cryptographically committed and reproducible.

```ts
const dataset = await za.loadDataset({ rootHash });               // BTC or 0G, spot or perp
const result  = await za.backtest(agent, dataset, opts);          // deterministic, no Date.now()
const cert    = await za.certify(result);                         // T2 today, T3 in v0.2
const inft    = await za.mintAgent({ agent, certificate: cert, name: 'RSI v1' });
```

---

## Architecture

```
    ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
    │    Agent     │ ───▶ │   Backtest   │ ───▶ │   runHash    │
    │  TypeScript  │      │ deterministic│      │   keccak256  │
    └──────────────┘      └──────────────┘      └──────────────┘
                                  │                     │
                          encrypt │                     │ anchor
                                  ▼                     ▼
                       ┌──────────────────┐  ┌──────────────────┐
                       │    0G Storage    │  │    0G Chain      │
                       │  AES-256-GCM     │  │  Certificate +   │
                       │  agent + log     │  │  ERC-7857 iNFT   │
                       └──────────────────┘  └──────────────────┘
                                                     ▲
                                                     │ v0.2: TEE attestation
                                                     │
                                              ┌────────────────────┐
                                              │  0G Compute        │
                                              │  Sealed Inference  │
                                              │  (Intel TDX +      │
                                              │   NVIDIA H100/H200)│
                                              └────────────────────┘
```

▸ **Verifiability.** Every backtest produces a `runHash` deterministically derivable from `agentHash + datasetHash + trades`. The hash anchors on 0G Chain (with a `trustTier` tag); the encrypted run log lives on 0G Storage. Anyone the owner authorizes can re-run the same agent on the same data and check the hash matches.

▸ **Privacy.** The agent runs on the user's machine. Only encrypted bytes hit 0G Storage. Only metrics + commitments hit 0G Chain. The strategy is never decrypted off-chain.

▸ **Ownership.** Agents that clear a configurable performance threshold are minted as ERC-7857 iNFTs. Transfers go through the oracle re-encryption flow, which re-encrypts metadata for the new owner without ever exposing plaintext.

---

## Trust model

| Tier | Mechanism | Available |
| - | - | - |
| **T1** | Commitment: `runHash` anchored on-chain. Trades cannot be edited after submission. | v0.1 |
| **T2** | Reproducibility: owner authorizes a verifier with the encrypted agent + key; verifier reruns and checks `runHash`. | v0.1 |
| **T3** | TEE attestation: `BacktestEngine` + the developer's agent run inside a 0G Compute enclave (Intel TDX + NVIDIA H100/H200) used as a generic TEE substrate. Trustless verification by anyone, agent code never revealed. | v0.2 |

We do not market v0.1 as "trustless" — we market it as "strategy stays sealed, metrics are committed, and reproducibility is owner-authorized." The architecture is designed so v0.2 wires in T3 without changing the v0.1 API.

Zero Arena is **model-agnostic infrastructure**. We do not host, run, endorse, or depend on any LLM or trading model. Whatever the developer writes inside `decide()` — RSI rule, Claude call, self-hosted Llama, custom RL — is theirs. We just verify what came out.

---

## MVP data scope

V0.1 anchors to BTC and 0G on Binance — spot first, perpetual futures as a stretch goal. Data is fetched once via `examples/00-binance-ingest`, hashed, uploaded to 0G Storage, and committed in a checked-in `datasets.lock.json`.

| Asset | Market | Window |
| - | - | - |
| BTC/USDT | Spot &nbsp;·&nbsp; Perp | 365d, 15m candles |
| 0G/USDT | Spot &nbsp;·&nbsp; Perp (if listed) | from listing, 15m candles |

If 0G is not yet on Binance at ship time, the dataset falls back to a DEX OHLCV source — explicitly tagged in metadata.

---

## Repositories

| Repository | Purpose | Status |
| - | - | - |
| **[zero-arena-sdk](https://github.com/Zero-Arena/zero-arena-sdk)** | TypeScript SDK and CLI &nbsp;·&nbsp; published as `zeroarena` on npm | active |
| **[zero-arena-contracts](https://github.com/Zero-Arena/zero-arena-contracts)** | Solidity contracts &nbsp;·&nbsp; Foundry &nbsp;·&nbsp; OpenZeppelin v5 | active |
| **[zero-arena-example-agent](https://github.com/Zero-Arena/zero-arena-example-agent)** | Reference agents and end-to-end demos | active |
| **[zero-arena-bacend](https://github.com/Zero-Arena/zero-arena-bacend)** | Dataset ingestion (Binance → 0G Storage) + oracle re-encryption service | active |
| **[zero-arena-fe](https://github.com/Zero-Arena/zero-arena-fe)** | Public dashboard — leaderboard + agent verification | active |
| **[zero-arena-docs](https://github.com/Zero-Arena/zero-arena-docs)** | Documentation site | post-hackathon |

<sub>Developers: start at the SDK. &nbsp;·&nbsp; Auditing on-chain logic: start at the contracts. &nbsp;·&nbsp; Looking for a runnable demo: start at the examples.</sub>

---

## Roadmap

```
v0.1   ───   backtest · certify · mint · transfer        BTC + 0G, spot + perp, T1+T2, testnet
v0.2   ───   T3 attestation via 0G Compute Sealed Inference
v0.3   ───   multi-asset universe · paper trading · multiple agent slots
v1.0   ───   mainnet · agent marketplace                 built on iNFT primitives
```

The SDK surface is additive. The `Certificate` struct already reserves `trustTier` and `attestationHash` slots, so v0.2 is wiring, not redesign.

---

## Why 0G

| | |
| - | - |
| **0G Storage** | Built-in AES-256 makes it the right substrate for proprietary agent IP — transferable, never exposed in plaintext. |
| **0G Chain** | Certificate anchoring and ERC-7857 minting at low cost. |
| **0G Compute (TEE substrate)** | Intel TDX + NVIDIA H100/H200 enclaves. We use it as generic confidential compute — `BacktestEngine` runs inside, the developer's agent runs inside, the TEE signs the run. We do not use 0G's models. Lights up T3 in v0.2. |
| **ERC-7857** | The iNFT standard solves the AI-asset transfer problem ERC-721 cannot. |

---

<div align="center">

<sub>Built for the **0G APAC Hackathon 2026** — Track 2: Agentic Trading Arena.</sub>

<sub>MIT-licensed &nbsp;·&nbsp; No live trading &nbsp;·&nbsp; Testnet only</sub>

</div>
