# Datafund Provenance Toolkit

Store data with cryptographic provenance on the [Swarm](https://ethswarm.org) decentralized network. Every upload is hashed, optionally signed by a notary, and can be anchored on-chain — giving you an immutable, verifiable record of what was stored, when, and by whom.

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

              Blockchain RPC (Base Sepolia)
       DataProvenance smart contracts (Solidity)
```

All three clients (SDK, CLI, MCP Server) can anchor data hashes **directly** on-chain via their own chain client.
The gateway handles Swarm storage, stamp management, and notary signing — it has no blockchain endpoints.

## Components

| Component | Repo | Language | Description |
|-----------|------|----------|-------------|
| **Smart Contracts** | [ConsentsBasedDataProvenance](https://github.com/datafund/ConsentsBasedDataProvenance) | Solidity | Consent management and on-chain data provenance (Base Sepolia) |
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
- **Free tier** — rate-limited access for development and testing (3 requests/min)
- **x402 payments** — pay-per-request access using USDC on Base chain
- **Integrity verification** — automatic SHA256 hash check on download

**Direct to blockchain** (SDK, CLI, and MCP Server):

- **Blockchain anchoring** — register data hashes on the DataProvenance smart contract (Base Sepolia)
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

### CLI (Python)

The CLI is not on PyPI. Install it from a release tag:

```bash
pip install "swarm-provenance-uploader @ git+https://github.com/datafund/swarm_provenance_CLI@v0.11.0"
```

Paid use with x402 needs the `[x402]` extra: `pip install "swarm-provenance-uploader[x402] @ git+https://github.com/datafund/swarm_provenance_CLI@v0.11.0"`.

```bash
# Upload a file on the free tier
swarm-prov-upload --free upload --file /path/to/data.txt --usePool

# Download and verify
swarm-prov-upload download <swarm_hash> --output-dir ./downloads
```

- `--free` uses the gateway's free tier (rate-limited, no payment). It is a global option, so it goes before the command. Without it the gateway answers "Payment required".
- `--usePool` takes a ready postage stamp from the gateway's pool (a few seconds) instead of buying a new one (over a minute).

If the upload fails with "Failed to acquire stamp from pool: 429" or "No stamps available in pool", the pool can't serve you right now. The pool has a shared daily allowance that resets at 00:00 UTC. Run the same command without `--usePool`; it buys a new stamp, which takes a few minutes. To see whether the pool has stamps before you upload, run `swarm-prov-upload --free stamps pool-status`. A 429 at other steps comes from the general rate limit: wait a minute and try again.

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

## Gateway

The public gateway is available at `https://provenance-gateway.datafund.io`. To self-host, see [swarm_connect](https://github.com/datafund/swarm_connect).

## Status

This toolkit is in **alpha / proof-of-concept** stage. Storage on Swarm is rented — data persistence depends on postage stamp validity.

## Links

- [datafund.io](https://datafund.io) — Datafund
- [github.com/datafund](https://github.com/datafund) — all repos

## License

MIT
