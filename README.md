# Datafund Provenance Toolkit

Store data with cryptographic provenance on the [Swarm](https://ethswarm.org) decentralized network. Every upload is hashed, optionally signed by a notary, and can be anchored on-chain — giving you an immutable, verifiable record of what was stored, when, and by whom.

> **Alpha software.** The production gateway takes real payments (USDC on Base mainnet). There are no guarantees, and refunds should not be expected. Spend small amounts, and test on the [staging gateway](#gateway) or the free tier first. Feedback: [info@datafund.io](mailto:info@datafund.io).

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    dataprovenance-app                     │
│                (Astro + React web UI)                     │
├──────────────────────────┬──────────────────────────────┤
│      SDK    │    CLI     │          MCP Server          │
│     (TS/JS) │  (Python)  │    (AI agent integration)    │
└───┬─────────┴──┬─────────┴──────────────┬───────────────┘
    │            │                        │
    ├────────────┼────────────────────────┤
    │       Chain Clients (direct)        │
    │       (viem / web3.py)              │
    ├────────────┼────────────────────────┘
    │            │                        │
    │   ┌────────┘          ┌─────────────┘
    │   │                   │
    ▼   ▼                   ▼
┌─────────────────────────────────────────────────────────┐
│                   Provenance Gateway                     │
│              (FastAPI — swarm_connect)                    │
│                                                          │
│       stamps, upload/download, notary signing            │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
                      Swarm Network
                  (decentralized storage)

    ── All clients also connect directly ──

     Blockchain RPC — anchoring/contracts (Base Sepolia testnet)
       DataProvenance smart contracts (Solidity)
```

All three clients (SDK, CLI, MCP Server) can anchor data hashes **directly** on-chain via their own chain client.
The gateway handles Swarm storage, stamp management, and notary signing — it has no blockchain endpoints.

## Components

| Component | Repo | Language | Description |
|-----------|------|----------|-------------|
| **Smart Contracts** | [ConsentsBasedDataProvenance](https://github.com/datafund/ConsentsBasedDataProvenance) | Solidity | Consent management and on-chain data provenance (anchoring/contracts on Base Sepolia testnet) |
| **SDK** | [swarm_provenance_SDK](https://github.com/datafund/swarm_provenance_SDK) | TypeScript | Library for browser and Node.js apps |
| **CLI** | [swarm_provenance_CLI](https://github.com/datafund/swarm_provenance_CLI) | Python | Command-line tool for uploads, downloads, chain anchoring, and stamp management |
| **MCP Server** | [swarm_provenance_MCP](https://github.com/datafund/swarm_provenance_MCP) | Python | AI agent integration via Model Context Protocol, with optional chain anchoring |
| **App** | [dataprovenance.app](https://dataprovenance.app) (live app) | TypeScript | Web interface for recording, verifying, and tracing data provenance |
| **Gateway** | [swarm_connect](https://github.com/datafund/swarm_connect) | Python | FastAPI server bridging clients to a Swarm Bee node |

## Key Features

**Via the gateway** (all clients):

- **Provenance metadata** — every upload wraps data with SHA256 hash, timestamp, and encoding info
- **Notary signing** — optional EIP-191 cryptographic signature proving data authenticity
- **Stamp management** — purchase, extend, and monitor postage stamps using duration and size presets
- **Free tier** — rate-limited access for development and testing (3 requests/min). On the production gateway each client also gets at most 3 free small stamps from the pool (`--usePool`) per day, reset at 00:00 UTC.
- **x402 payments** — pay-per-request access with x402 (v1 and v2). On the production gateway you pay **real USDC on Base mainnet**. The software is alpha: no guarantees, refunds should not be expected, so use small amounts. For testing, use the staging gateway `provenance-gateway.dev.datafund.io` (Base Sepolia, test USDC). See [Gateway](#gateway).
- **Integrity verification** — automatic SHA256 hash check on download

**Direct to blockchain** (SDK, CLI, and MCP Server):

- **Blockchain anchoring** — register data hashes on the DataProvenance smart contract (anchoring/contracts on Base Sepolia testnet)
- **Transformation lineage** — record data transformations and merges, building a provenance DAG
- **Storage references** — bidirectional lookup between Swarm hashes and external storage identifiers

## Quick Start

### SDK (TypeScript)

```bash
pnpm add @datafund/swarm-provenance
```

```typescript
import { ProvenanceClient } from '@datafund/swarm-provenance';

const client = new ProvenanceClient();
const result = await client.upload('Hello, World!', { standard: 'v1' });
console.log('Reference:', result.reference);
```

The SDK's default payment mode is `payment: 'free'`, which uses the gateway's free tier. SDK x402 payments (`payment: { wallet }`) default to Base Sepolia, so paying on the production gateway needs the Base mainnet network: `payment: { wallet, network: 'eip155:8453', v1Network: 'base' }`. That spends real USDC.

### CLI (Python)

The CLI is not on PyPI. Install it from a release tag:

```bash
pip install "swarm-provenance-uploader @ git+https://github.com/datafund/swarm_provenance_CLI@v0.11.0"
```

Paid use with x402 needs the `[x402]` extra: `pip install "swarm-provenance-uploader[x402] @ git+https://github.com/datafund/swarm_provenance_CLI@v0.11.0"`.

> **Warning:** the CLI's default gateway is production. Paid uploads (`--x402`) with the default gateway spend **real USDC on Base mainnet**. To test without real money, point the CLI at the staging gateway with `PROVENANCE_GATEWAY_URL=https://provenance-gateway.dev.datafund.io`, or use `--free` for the free tier. The CLI's x402 network defaults to `base-sepolia`, so paying on production also needs `X402_NETWORK=base`, or the global option `--x402-network base`, which goes before the command like `--free`: `swarm-prov-upload --x402 --x402-network base upload ...`.

Blockchain anchoring with the `swarm-prov-upload chain` commands needs the `[blockchain]` extra: `pip install "swarm-provenance-uploader[blockchain] @ git+https://github.com/datafund/swarm_provenance_CLI@v0.11.0"`.

```bash
# Upload a file on the free tier
swarm-prov-upload --free upload --file /path/to/data.txt --usePool

# Download and verify
swarm-prov-upload download <swarm_hash> --output-dir ./downloads

# Same upload against the staging gateway (for testing)
PROVENANCE_GATEWAY_URL=https://provenance-gateway.dev.datafund.io swarm-prov-upload --free upload --file /path/to/data.txt --usePool
```

- `--free` uses the gateway's free tier (rate-limited, no payment). It is a global option, so it goes before the command. Without it the gateway answers "Payment required".
- `--usePool` takes a ready postage stamp from the gateway's pool (a few seconds) instead of buying a new one (over a minute).

If the upload fails with "Failed to acquire stamp from pool: 429" or "No stamps available in pool", the pool can't serve you right now. The pool has a shared daily allowance that resets at 00:00 UTC. Run the same command without `--usePool`; it buys a new stamp, which takes a few minutes. To see whether the pool has stamps before you upload, run `swarm-prov-upload --free stamps pool-status`. A 429 at other steps comes from the general rate limit: wait a minute and try again.

On the production gateway each client can take 3 free small stamps from the pool per day. After that, the gateway answers 402 (code `DAILY_STAMP_ALLOWANCE_PER_CLIENT_EXHAUSTED`) and the CLI prints "Payment required ... Use --x402 ...", even with `--free`. On production, `--x402` means paying real USDC, so do not follow that suggestion unless you mean to pay. Instead, wait for the reset at 00:00 UTC, use the staging gateway, or run the same command without `--usePool` to buy a new stamp on the free tier (this takes a few minutes).

### MCP Server (AI Agents)

**Option A — Docker (recommended):**

```bash
docker pull ghcr.io/datafund/swarm-provenance-mcp:latest
```

**Option B — From source:**

The MCP server needs Python 3.10 or later.

```bash
git clone https://github.com/datafund/swarm_provenance_MCP.git
cd swarm_provenance_MCP
pip install -e .
```

Add to your Claude Desktop config (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "swarm-provenance": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "ghcr.io/datafund/swarm-provenance-mcp:latest"],
      "env": {
        "SWARM_GATEWAY_URL": "https://provenance-gateway.datafund.io",
        "CHAIN_ENABLED": "true"
      }
    }
  }
}
```

Or if installed from source, replace `"command"` / `"args"` with `"command": "swarm-provenance-mcp"`.

The MCP server uses the gateway's free tier; it cannot make x402 payments.

## Gateway

There are two public gateways:

| Gateway | URL | x402 network | Token | Free tier | Use for |
|---------|-----|--------------|-------|-----------|---------|
| Production | `https://provenance-gateway.datafund.io` | Base mainnet (`base`) | Real USDC | Yes, 3 requests/min; 3 free pooled stamps per client per day (resets 00:00 UTC) | Real use (alpha) |
| Staging | `https://provenance-gateway.dev.datafund.io` | Base Sepolia (`base-sepolia`) | Test USDC | Yes, 3 requests/min | Integration testing only |

Both accept x402 v1 and v2. For the free tier, send the header `X-Payment-Mode: free` (CLI: `--free`, SDK: `payment: 'free'`). The staging gateway stores data on the real Swarm network; use it for integration testing only.

To self-host, see [swarm_connect](https://github.com/datafund/swarm_connect).

## Status

This toolkit is **alpha** software. There are no guarantees, and refunds should not be expected; spend small amounts and test on the staging gateway or the free tier first.

- The production gateway accepts real payments (USDC on Base mainnet) and also offers a rate-limited free tier.
- On-chain anchoring still uses Base Sepolia (testnet).
- Storage on Swarm is rented — data persistence depends on postage stamp validity.

Feedback: [info@datafund.io](mailto:info@datafund.io).

## Links

- [datafund.io](https://datafund.io) — Datafund
- [github.com/datafund](https://github.com/datafund) — all repos

## License

MIT
