# Datafund Provenance Toolkit

Store data with cryptographic provenance on the [Swarm](https://ethswarm.org) decentralized network. Every upload is hashed, optionally signed by a notary, and can be anchored on-chain — giving you an immutable, verifiable record of what was stored, when, and by whom.

> For a non-technical overview, visit [provenance.datafund.io](https://provenance.datafund.io)

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Your Application                  │
├──────────┬──────────────┬───────────────────────────┤
│   SDK    │     CLI      │        MCP Server         │
│ (TS/JS)  │  (Python)    │   (AI agent integration)  │
└────┬─────┴──────┬───────┴────────────┬──────────────┘
     │            │                    │
     ▼            ▼                    ▼
┌─────────────────────────────────────────────────────┐
│              Provenance Gateway                      │
│         (FastAPI — swarm_connect)                    │
├─────────────────────┬───────────────────────────────┤
│    Swarm Network    │   Blockchain (Base Sepolia)   │
│  (decentralized     │   (DataProvenance contract,   │
│   storage)          │    on-chain anchoring)        │
└─────────────────────┴───────────────────────────────┘
```

## Components

| Component | Repo | Language | Description |
|-----------|------|----------|-------------|
| **SDK** | [swarm_provenance_SDK](https://github.com/datafund/swarm_provenance_SDK) | TypeScript | Library for browser and Node.js apps |
| **CLI** | [swarm_provenance_CLI](https://github.com/datafund/swarm_provenance_CLI) | Python | Command-line tool for uploads, downloads, and stamp management |
| **MCP Server** | [swarm_provenance_mcp](https://github.com/datafund/swarm_provenance_mcp) | Python | AI agent integration via Model Context Protocol |
| **Gateway** | [swarm_connect](https://github.com/datafund/swarm_connect) | Python | FastAPI server bridging clients to a Swarm Bee node |
| **Landing Page** | [provenance-landing](https://github.com/datafund/provenance-landing) | Astro | [provenance.datafund.io](https://provenance.datafund.io) |

## Key Features

All client tools (SDK, CLI, MCP) share these capabilities through the gateway:

- **Provenance metadata** — every upload wraps data with SHA256 hash, timestamp, and encoding info
- **Notary signing** — optional EIP-191 cryptographic signature proving data authenticity
- **Blockchain anchoring** — register data hashes on the DataProvenance smart contract (Base Sepolia)
- **Stamp management** — purchase, pool, extend, and monitor postage stamps for Swarm storage
- **x402 payments** — pay-per-request access using USDC on Base chain
- **Integrity verification** — automatic SHA256 hash check on download

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

```bash
pip install -e "git+https://github.com/datafund/swarm_provenance_CLI.git#egg=swarm-provenance-uploader"
```

```bash
# Upload a file
swarm-prov-upload upload --file /path/to/data.txt

# Download and verify
swarm-prov-upload download <swarm_hash> --output-dir ./downloads
```

### MCP Server (AI Agents)

```bash
git clone https://github.com/datafund/swarm_provenance_mcp.git
cd swarm_provenance_mcp
pip install -e .
```

Add to your Claude Desktop config (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "swarm-provenance": {
      "command": "swarm-provenance-mcp",
      "env": {
        "SWARM_GATEWAY_URL": "https://provenance-gateway.datafund.io"
      }
    }
  }
}
```

## Gateway

The public gateway is available at `https://provenance-gateway.datafund.io`. To self-host, see [swarm_connect](https://github.com/datafund/swarm_connect).

## Status

This toolkit is in **alpha / proof-of-concept** stage. Storage on Swarm is rented — data persistence depends on postage stamp validity.

## Links

- [provenance.datafund.io](https://provenance.datafund.io) — product overview
- [datafund.io](https://datafund.io) — Datafund
- [github.com/datafund](https://github.com/datafund) — all repos

## License

MIT
