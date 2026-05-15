# Zero Arena

> **Verifiable performance for AI trading agents.** Prove your winrate without leaking your strategy. Backtest → Certificate → ERC-7857 mint, in 5 lines of TypeScript.

[![0G](https://img.shields.io/badge/built%20on-0G-black)](https://0g.ai) [![License](https://img.shields.io/badge/license-MIT-black)](./LICENSE) [![npm](https://img.shields.io/npm/v/zeroarena?color=22c55e&label=zeroarena)](https://www.npmjs.com/package/zeroarena) [![Dashboard](https://img.shields.io/badge/dashboard-live-22c55e)](https://zero-arena-fe.vercel.app) [![Oracle](https://img.shields.io/badge/oracle-live-22c55e)](https://transfer-oracle-production-f390.up.railway.app/health) [![X](https://img.shields.io/badge/X-%400arena__labs-black?logo=x&logoColor=white)](https://x.com/0arena_labs)

Anyone can claim "70% winrate, 3x ROI." There is no way for a third party to verify without trusting the claimant or demanding the source. Zero Arena closes the gap: deterministic backtest, encrypted run log on 0G Storage, certificate anchored on 0G Chain, ERC-7857 iNFT mint. The strategy never leaves your machine in plaintext.

---

## Live deployments

| Component | URL | Notes |
| - | - | - |
| Public dashboard | [zero-arena-fe.vercel.app](https://zero-arena-fe.vercel.app) | Leaderboard + agent detail, reads chain directly |
| SDK on npm | [`zeroarena@0.2.1`](https://www.npmjs.com/package/zeroarena) | `npm install zeroarena` |
| Transfer-oracle | [transfer-oracle-production-f390.up.railway.app](https://transfer-oracle-production-f390.up.railway.app/health) | ERC-7857 re-encryption signer |
| Season keeper | _(internal, no public URL)_ | Auto-settle daemon, polls every 60s |
| 0G Chain RPC | `https://evmrpc-testnet.0g.ai` | Galileo testnet |
| 0G Storage indexer | `https://indexer-storage-testnet-turbo.0g.ai` | |
| 0G Explorer | [chainscan-galileo.0g.ai](https://chainscan-galileo.0g.ai) | |
| Follow on X | [@0arena_labs](https://x.com/0arena_labs) | Updates, demos, roadmap |

### Contracts (Galileo testnet, chainId 16602)

| Contract | Address |
| - | - |
| `AgentCertificate` | [`0x77f29d2a7BcAC679812d9a0FB1c7508eDA6B087e`](https://chainscan-galileo.0g.ai/address/0x77f29d2a7BcAC679812d9a0FB1c7508eDA6B087e) |
| `ZeroArenaINFT` | [`0xF7162ecbdB11DE4704043D4aF93B4030AD61700e`](https://chainscan-galileo.0g.ai/address/0xF7162ecbdB11DE4704043D4aF93B4030AD61700e) |
| `ReencryptionOracle` | [`0x733667CEBB27e310a8fb60799Af73A8C1fe501b2`](https://chainscan-galileo.0g.ai/address/0x733667CEBB27e310a8fb60799Af73A8C1fe501b2) |
| `LiveCertificate` | [`0x2c71fe022E4698f8fD63384A19Cd69D72a714b4d`](https://chainscan-galileo.0g.ai/address/0x2c71fe022E4698f8fD63384A19Cd69D72a714b4d) |
| `Season` | [`0x8fb87CE34b4e8F4C65eeB6752b0168EC37806CF3`](https://chainscan-galileo.0g.ai/address/0x8fb87CE34b4e8F4C65eeB6752b0168EC37806CF3) |

---

## Architecture — infrastructure vs owner-operated

Zero Arena ships **infrastructure**, not a hosted service. Two clear layers:

```
┌─ Hosted by Zero Arena (public good) ─────────────────────┐
│                                                          │
│   transfer-oracle    season-keeper    FE dashboard       │
│   (Railway)          (Railway)        (Vercel)           │
│   ↓                  ↓                ↓                  │
│   signs proofs       polls + settles  reads chain        │
│                                                          │
└──────────────────────────────────────────────────────────┘
                          │
            ┌─────────────┼──────────────┐
            ▼             ▼              ▼
       0G Chain      0G Storage      Galileo RPC
       (Certs +      (Encrypted      (public)
        iNFTs)        run logs)
            ▲             ▲
            │             │
┌─ Run by you (the agent owner) ───────────────────────────┐
│                                                          │
│   zeroarena (npm)    paper daemon     your wallet        │
│   - backtest         - per-iNFT       - signs txs        │
│   - certify          - 24/7 if you    - holds AES keys   │
│   - mint               want live cert                    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

The split is intentional: we never want to hold your strategy. The paper daemon code is shipped in [`zero-arena-bacend/`](./zero-arena-bacend/) as a reference impl that owners self-deploy.

---

## Whose key is whose

| Role | Key | Where | When |
| - | - | - | - |
| **Agent developer (you)** | `PRIVATE_KEY` | your project's `.env` ([template](https://github.com/Zero-Arena/zero-arena-example-agent/blob/main/.env.example)) | Every `certify` + `mintAgent` call |
| Oracle operator | `ORACLE_PRIVATE_KEY` | [`zero-arena-be/.env.transfer-oracle.example`](https://github.com/Zero-Arena/zero-arena-be/blob/main/.env.transfer-oracle.example) | Serving `transferAgent` proofs |
| Paper / season operator | `OPERATOR_PRIVATE_KEY` | [`zero-arena-be/.env.paper.example`](https://github.com/Zero-Arena/zero-arena-be/blob/main/.env.paper.example) | Paper epoch commits, season settles |
| Contract deployer | `DEPLOYER_PRIVATE_KEY` | [`zero-arena-contracts/.env.example`](https://github.com/Zero-Arena/zero-arena-contracts/blob/main/.env.example) | One-time, already done |

**Agent devs only need row 1.** If anything asks you for an oracle / deployer key, that's a bug.

---

## Repositories

Each is a standalone GitHub repo. Folder names map 1:1 to repos.

| Repo | Purpose |
| - | - |
| [`zero-arena-sdk`](https://github.com/Zero-Arena/zero-arena-sdk) | `zeroarena` npm package — TypeScript SDK + CLI |
| [`zero-arena-contracts`](https://github.com/Zero-Arena/zero-arena-contracts) | Solidity contracts. Foundry. |
| [`zero-arena-example-agent`](https://github.com/Zero-Arena/zero-arena-example-agent) | 8 reference agents, multi-mint orchestrator, season scripts |
| [`zero-arena-be`](https://github.com/Zero-Arena/zero-arena-be) | Backend services: transfer-oracle, season-keeper, paper (ref impl) |
| [`zero-arena-fe`](https://github.com/Zero-Arena/zero-arena-fe) | Next.js dashboard |

- Building an agent? → [`zero-arena-sdk`](https://github.com/Zero-Arena/zero-arena-sdk) then [`01-rsi-spot-btc`](https://github.com/Zero-Arena/zero-arena-example-agent/tree/main/01-rsi-spot-btc).
- Auditing contracts? → [`zero-arena-contracts`](https://github.com/Zero-Arena/zero-arena-contracts).
- Working on the dashboard? → [`zero-arena-fe`](https://github.com/Zero-Arena/zero-arena-fe).
- Running infrastructure? → [`zero-arena-be`](https://github.com/Zero-Arena/zero-arena-be).
- Cutting a release? → [`zero-arena-sdk/RELEASE.md`](https://github.com/Zero-Arena/zero-arena-sdk/blob/main/RELEASE.md).

---

## Quick start

```bash
npx zeroarena init my-agent
cd my-agent
# Paste your wallet key when prompted (or generate one: cast wallet new)
npm start
```

The wizard scaffolds an `agent.ts`, pre-pins Galileo addresses, runs backtest → certify → mint end-to-end. The dashboard auto-picks up your new iNFT within a minute.

Or manually:

```ts
import { ZeroArena, Agent } from 'zeroarena';

const za = new ZeroArena({
  rpc: 'https://evmrpc-testnet.0g.ai',
  indexer: 'https://indexer-storage-testnet-turbo.0g.ai',
  privateKey: process.env.PRIVATE_KEY!,
  addresses: {
    AgentCertificate:   '0x77f29d2a7BcAC679812d9a0FB1c7508eDA6B087e',
    ZeroArenaINFT:      '0xF7162ecbdB11DE4704043D4aF93B4030AD61700e',
    ReencryptionOracle: '0x733667CEBB27e310a8fb60799Af73A8C1fe501b2',
  },
});

class RsiAgent extends Agent {
  decide(obs) {
    if (obs.rsi14 < 30) return { direction: 1, size: 0.5 };
    if (obs.rsi14 > 70) return { direction: 0, size: 0 };
    return { direction: obs.position > 0 ? 1 : 0, size: obs.position > 0 ? 0.5 : 0 };
  }
}

const dataset = await za.loadDataset({ rootHash: '0xabc…' });
const result  = await za.backtest(new RsiAgent(), dataset, { initialBalance: 10_000, market: 'spot' });
const cert    = await za.certify(result);
const inft    = await za.mintAgent({ agent: new RsiAgent(), certificate: cert, name: 'RSI v1' });
```

Full walkthrough: [`01-rsi-spot-btc/`](https://github.com/Zero-Arena/zero-arena-example-agent/tree/main/01-rsi-spot-btc).

---

## Trust model

| Tier | What it proves | Available |
| - | - | - |
| **T1 — Commitment** | `runHash` anchored on-chain; trades immutable after submission. | v0.1 |
| **T2 — Reproducibility** | Owner shares encrypted agent + AES key → verifier reruns → same `runHash`. | v0.1 |
| **T3 — TEE attestation** | Engine + agent run inside a 0G Compute enclave; trustless verification, code never revealed. | v0.4 |

v0.2 ships **T1 + T2**. The `Certificate` struct reserves the `trustTier` and `attestationHash` slots — v0.4 is wiring, not redesign. Details in [`CLAUDE.md`](./CLAUDE.md).

Zero Arena is **model-agnostic** — we don't bundle, recommend, or depend on any LLM or trading model. Whatever you put in `decide()` is yours.

---

## MVP data scope (v0.2)

Spot only. The perp engine is implemented and reachable from the SDK, but the canonical demo path + leaderboard ranking + paper-engine support all target spot for v0.2. Full perp coverage promoted to v0.3.

| Asset | Market | Granularity | Window |
| - | - | - | - |
| BTC/USDT | Spot | 15m | last 365 days |
| 0G/USDT | Spot (or DEX fallback) | 15m | from listing |

Dataset ingest is in the SDK CLI: `npx zeroarena dataset ingest …` normalizes Binance OHLCV, hashes it, uploads to 0G Storage, and rotates `datasets.lock.json`.

---

## Roadmap

- **v0.1** — backtest, certify, mint, transfer. BTC + 0G spot. T1 + T2. Galileo testnet. ✅
- **v0.2** — Paper-engine (`LiveCertificate`), seasons (`Season`), live leaderboard, spot canonical. ✅
- **v0.3** — perpetual futures promoted to canonical (funding accrual, isolated-margin liquidation), multi-asset universe, additional dataset slots per iNFT.
- **v0.4** — T3 via 0G Compute Sealed Inference. TEE-attested oracle + paper daemon.
- **v1.0** — mainnet, public agent marketplace.

The v0.1 API is additive; later phases are wiring.

---

## License

MIT. Galileo testnet today; mainnet in v1.0.
