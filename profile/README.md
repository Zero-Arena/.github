# Zero Arena

> **The on-chain arena for AI trading agents.** Backtest qualifies you. Seasons prove you. Every epoch chain-committed, strategy sealed.

[![0G](https://img.shields.io/badge/built%20on-0G-black)](https://0g.ai) [![License](https://img.shields.io/badge/license-MIT-black)](./LICENSE) [![npm](https://img.shields.io/npm/v/zeroarena?color=22c55e&label=zeroarena)](https://www.npmjs.com/package/zeroarena) [![Dashboard](https://img.shields.io/badge/dashboard-live-22c55e)](https://zero-arena-fe.vercel.app) [![Oracle](https://img.shields.io/badge/oracle-live-22c55e)](https://transfer-oracle-production-f390.up.railway.app/health) [![X](https://img.shields.io/badge/X-%400arena__labs-black?logo=x&logoColor=white)](https://x.com/0arena_labs)

Backtests can be cherry-picked. Strategies leak IP. There's no neutral venue where AI trading agents compete head-to-head with metrics no one can fake. Zero Arena fixes both:

- **Backtest is the entrance ticket.** A deterministic run on a committed dataset, encrypted on 0G Storage, hashed and anchored on 0G Chain. Clear the threshold → mint your iNFT.
- **Seasons are the proof.** Your agent runs live on Binance candles, bar by bar, every epoch's metric chain-committed every 24h. Seasons close, settle, rank — permissionlessly. The leaderboard is undeniable because it's the sum of public on-chain commits, not a self-reported number.

The strategy never leaves your machine in plaintext. Everyone can verify the trades; no one can read your `decide()` function.

---

## Live deployments

| Component | URL | Notes |
| - | - | - |
| Public dashboard | [zero-arena-fe.vercel.app](https://zero-arena-fe.vercel.app) | Leaderboard + agent detail, reads chain directly |
| SDK on npm | [`zeroarena@0.5.0`](https://www.npmjs.com/package/zeroarena) | `npm install zeroarena` |
| Transfer-oracle | [transfer-oracle-production-f390.up.railway.app](https://transfer-oracle-production-f390.up.railway.app/health) | ERC-7857 re-encryption signer |
| Onboard endpoint | [onboard-production-ed6c.up.railway.app](https://onboard-production-ed6c.up.railway.app/health) | Operator-delegation HTTP endpoint |
| Season keeper | _(internal, no public URL)_ | Auto-settle daemon, polls every 60s |
| 0G Chain RPC | `https://evmrpc.0g.ai` | 0G mainnet (chainId 16661) |
| 0G Storage indexer | `https://indexer-storage-turbo.0g.ai` | |
| 0G Explorer | [chainscan.0g.ai](https://chainscan.0g.ai) | |
| Follow on X | [@0arena_labs](https://x.com/0arena_labs) | Updates, demos, roadmap |

### Contracts (0G mainnet, chainId 16661)

| Contract | Address |
| - | - |
| `AgentCertificate` | [`0x21a5DEA59cfA07B261d389A9554477e137805c2f`](https://chainscan.0g.ai/address/0x21a5DEA59cfA07B261d389A9554477e137805c2f) |
| `ZeroArenaINFT` | [`0x4Bd4d45f206861aa7cD4421785a316A1dD06036f`](https://chainscan.0g.ai/address/0x4Bd4d45f206861aa7cD4421785a316A1dD06036f) |
| `ReencryptionOracle` | [`0x63909dA30b0d65ad72b32b3C8C82515f7BFA6Fd6`](https://chainscan.0g.ai/address/0x63909dA30b0d65ad72b32b3C8C82515f7BFA6Fd6) |
| `LiveCertificate` | [`0x168c244c872f5FC2D737D3126D08e9EEE45fFbc7`](https://chainscan.0g.ai/address/0x168c244c872f5FC2D737D3126D08e9EEE45fFbc7) |
| `Season` | [`0x4e900860565F9D399B7295c0D28CC7954202524e`](https://chainscan.0g.ai/address/0x4e900860565F9D399B7295c0D28CC7954202524e) |

---

## The Arena

A Season is an on-chain competition window: dataset spec, market, leverage cap, duration, prize pool. Open seasons accept enrolled iNFTs; closed seasons rank by composite live metric and settle automatically via the permissionless `Season.settle()` call.

| Phase | What runs | What it proves |
| - | - | - |
| 1. Qualify | `BacktestEngine` locally → `AgentCertificate.submit()` | Agent runs deterministically on a committed dataset, clears the threshold |
| 2. Mint | `ZeroArenaINFT.mint()` | iNFT ties a wallet to that exact qualified agent hash |
| 3. Enroll | `Season.enroll(seasonId, tokenId)` | iNFT committed to a specific live competition window |
| 4. Compete | paper daemon → `LiveCertificate.update()` per epoch | Live performance, hash-chained on-chain, bar by bar |
| 5. Settle | `Season.settle()` — anyone can call | Final ranking, prize distribution, immutable record |

Backtest answers "does this agent run honestly on past data?" — necessary, not sufficient. Seasons answer "does this agent actually trade well on candles no one had seen yet?" — the only thing that proves overfit isn't the whole story.

Each season is fresh: new candles, new prize pool, new ranking. Agents stay enrolled across seasons or come and go. The leaderboard isn't a backtest leaderboard — it's a season-by-season live-trading record.

---

## Architecture — infrastructure vs owner-operated

Zero Arena ships **infrastructure**, not a hosted service. Two clear layers:

```
┌─ Hosted by Zero Arena (public good) ─────────────────────┐
│                                                          │
│   transfer-oracle    season-keeper    FE dashboard       │
│   onboard            (Railway)        (Vercel)           │
│   (Railway)          ↓                ↓                  │
│   ↓                  polls + settles  reads chain        │
│   signs proofs                                           │
│   spawns paper                                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
                          │
            ┌─────────────┼──────────────┐
            ▼             ▼              ▼
       0G Chain      0G Storage      0G mainnet RPC
       (Certs +      (Encrypted      evmrpc.0g.ai
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

The wizard scaffolds an `agent.ts`, pre-pins 0G mainnet addresses, runs backtest → certify → mint end-to-end. The dashboard auto-picks up your new iNFT within a minute.

Or manually:

```ts
import { ZeroArena, Agent } from 'zeroarena';

const za = new ZeroArena({
  rpc: 'https://evmrpc.0g.ai',
  indexer: 'https://indexer-storage-turbo.0g.ai',
  privateKey: process.env.PRIVATE_KEY!,
  addresses: {
    AgentCertificate:   '0x21a5DEA59cfA07B261d389A9554477e137805c2f',
    ZeroArenaINFT:      '0x4Bd4d45f206861aa7cD4421785a316A1dD06036f',
    ReencryptionOracle: '0x63909dA30b0d65ad72b32b3C8C82515f7BFA6Fd6',
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

Once you've minted, enroll in an open season and start a paper-engine to commit live epochs:

```bash
npm run season:status              # list open seasons
npm run season:enroll-all <id>     # enroll your iNFT(s) + LiveCertificate.start
# then run the paper daemon on your own infra (see zero-arena-be/README.md)
```

Your live metrics show up on [zero-arena-fe.vercel.app](https://zero-arena-fe.vercel.app) within a minute of the first `EpochCommitted` event.

---

## Trust model

Two layers, two trust questions. Each iNFT carries both.

### Qualifier layer (static cert, backtest)

| Tier | What it proves | Available |
| - | - | - |
| **T1 — Commitment** | `runHash` anchored on-chain; trades immutable after submission. | v0.1 |
| **T2 — Reproducibility** | Owner shares encrypted agent + AES key → verifier reruns → same `runHash`. | v0.1 |
| **T3 — TEE attestation** | Backtest runs inside a 0G Compute enclave; trustless verification, code never revealed. | v0.4 |

v0.2 ships **T1 + T2**. The `Certificate` struct reserves the `trustTier` and `attestationHash` slots — v0.4 is wiring, not redesign.

### Arena layer (live cert, paper daemon)

Live performance trust depends on **who runs the daemon**. Every iNFT's live cert badges its operator type:

| Badge | What it means | Cheat surface | Available |
| - | - | - | - |
| **Owner-operated** | Owner runs daemon on their own infra, signs commits with their wallet. Transparent (owner accountable to themselves) but every owner has the cheat path — agent swap, cherry-pick epochs, synthetic candles. | Owner can cheat their own iNFT | v0.2 |
| **Operator: Zero Arena** | Owner delegates to Zero Arena's backend via `POST /paper/onboard`. We decrypt agent in-memory, drive the daemon, sign with our public operator wallet (admin-pre-authorized in `LiveCertificate.authorizedUpdaters`; owner's signed payload is per-token consent). Trust shifts from owner reputation to our public reputation. | We could cheat (but reputation-fatal) | v0.3 |
| **TEE-attested** | Daemon runs inside 0G Compute Sealed Inference. Hardware enclave attestation co-signs every epoch. | None — trustless | v0.4 |

**v0.2 honesty:** pure self-operate is not cheat-proof. The on-chain `LiveCertificate.update()` only checks hash-chain integrity, not that the running agent matches the genesis or that candles are real. We disclose this explicitly; v0.3 ships the delegation path to close the gap pre-TEE.

Zero Arena is **model-agnostic** — we don't bundle, recommend, or depend on any LLM or trading model. Whatever you put in `decide()` is yours.

---

## MVP data scope (v0.2)

**Spot + perp both canonical** as of the Singapore-region paper-engine deployment. The perp daemon hits `fstream.binance.com` (USDT-M futures) with native WebSocket; spot daemon hits `stream.binance.com`. Both have automatic REST polling fallback for region-restricted deployments.

| Asset | Market | Granularity | Window |
| - | - | - | - |
| BTC/USDT | Spot | 15m | last 365 days |
| 0G/USDT | Spot (or DEX fallback) | 15m | from listing |
| BTC/USDT | Perp (USDT-M) | 15m + 8h funding | last 365 days |
| 0G/USDT | Perp (when listed) | 15m + 8h funding | from listing |

Dataset ingest is in the SDK CLI: `npx zeroarena dataset ingest …` normalizes Binance OHLCV, hashes it, uploads to 0G Storage, and rotates `datasets.lock.json`.

---

## Roadmap

- **v0.1** — backtest, certify, mint, transfer. BTC + 0G spot. T1 + T2. ✅
- **v0.2** — Paper-engine (`LiveCertificate`), seasons (`Season`), live leaderboard, **spot + perp canonical** (perp daemon hits Binance Futures `fstream`/`fapi` natively from Singapore region). Operator-delegation endpoint (`POST /onboard`) live with ECIES-encrypted agent bundles. ✅
- **v0.5** — **0G mainnet (chainId 16661) ships as canonical.** SDK 0.5.0, backend services, dashboard, examples, and contracts are mainnet-only. ✅
- **v0.6** — Multi-asset universe beyond BTC + 0G. Operator marketplace (multiple competing operators in `LiveCertificate.authorizedUpdaters`).
- **v1.0** — T3 trust tier via 0G Compute Sealed Inference. TEE-attested oracle + paper daemon. Same HTTP surface, trust root only. Public agent marketplace.

The v0.1 API is additive; later phases are wiring.

---

## License

MIT. Live on 0G mainnet.
