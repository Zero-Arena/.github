<div align="center">

<br />

# Zero Arena

**Verifiable AI trading agents on 0G.**

Backtest deterministically. Anchor the proof on-chain. Tokenize as ERC-7857.

<br />

[ Documentation ](https://github.com/Zero-Arena/zero-arena-docs)&nbsp;&nbsp;·&nbsp;&nbsp;[ SDK ](https://github.com/Zero-Arena/zero-arena-sdk)&nbsp;&nbsp;·&nbsp;&nbsp;[ Contracts ](https://github.com/Zero-Arena/zero-arena-contracts)&nbsp;&nbsp;·&nbsp;&nbsp;[ Examples ](https://github.com/Zero-Arena/zero-arena-examples)

<br />

</div>

---

> The agent intelligence problem is solved. The infrastructure trust problem is not.

Every AI trading agent today shares the same flaw — results live on a Notion page, a tweet, a PDF. There is no way to verify the agent ran what it claims, on the data it claims. There is no way to own that agent as an asset, transfer it, or sell it without exposing its weights.

Zero Arena fixes both, in five lines of TypeScript.

```ts
const dataset = await za.loadDataset({ rootHash });
const result  = await za.backtest(agent, dataset, opts);
const cert    = await za.certify(result);
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
                       │  AES-256 sealed  │  │  Certificate +   │
                       │  agent + log     │  │  ERC-7857 iNFT   │
                       └──────────────────┘  └──────────────────┘
```

▸ **Verifiability.** Every backtest produces a `runHash` deterministically derivable from `agentHash + datasetHash + trades`. The hash anchors on 0G Chain, the encrypted run log lives on 0G Storage. Anyone can re-run the same agent on the same data and check the hash matches.

▸ **Ownership.** Agents that clear a configurable performance threshold are minted as ERC-7857 iNFTs. The strategy stays encrypted in 0G Storage — only the holder of the data key can decrypt it. Transfers go through the ERC-7857 oracle flow, which re-encrypts metadata for the new owner without ever exposing plaintext.

---

## Repositories

| Repository | Purpose | Status |
| - | - | - |
| **[zero-arena-sdk](https://github.com/Zero-Arena/zero-arena-sdk)** | TypeScript SDK and CLI &nbsp;·&nbsp; published as `zeroarena` on npm | active |
| **[zero-arena-contracts](https://github.com/Zero-Arena/zero-arena-contracts)** | Solidity contracts &nbsp;·&nbsp; Foundry &nbsp;·&nbsp; OpenZeppelin v5 | active |
| **[zero-arena-examples](https://github.com/Zero-Arena/zero-arena-examples)** | Reference agents, sample datasets, end-to-end demos | active |
| **[zero-arena-docs](https://github.com/Zero-Arena/zero-arena-docs)** | Documentation site | post-hackathon |

<sub>Developers: start at the SDK. &nbsp;·&nbsp; Auditing on-chain logic: start at the contracts. &nbsp;·&nbsp; Looking for a runnable demo: start at the examples.</sub>

---

## Roadmap

```
v0.1   ───   backtest · certify · mint · transfer        single-asset OHLCV, testnet
v0.2   ───   paper trading via 0G Compute                multiple agent slots per iNFT
v0.3   ───   Sealed Inference for on-chain LLM agents
v1.0   ───   mainnet · agent marketplace                 built on iNFT primitives
```

The SDK surface is additive. Nothing released in v0.1 changes when later phases ship.

---

## Why 0G

Zero Arena exercises 0G at the depth it was designed for, rather than naming six components and integrating none of them deeply.

| | |
| - | - |
| **0G Storage** | Built-in AES-256 makes it the right substrate for proprietary agent IP — transferable, never exposed. |
| **0G Chain** | Certificate anchoring and ERC-7857 minting at low cost. |
| **ERC-7857** | The iNFT standard solves the AI-asset transfer problem ERC-721 cannot. |

---

<div align="center">

<sub>Built for the **0G APAC Hackathon 2026** — Track 2: Agentic Trading Arena.</sub>

<sub>MIT-licensed &nbsp;·&nbsp; No live trading &nbsp;·&nbsp; Testnet only</sub>

</div>
