---
description: The only MCP you need on solana.
icon: user-robot
---

# Astralane MCP

The Astralane MCP Server connects your AI assistant directly to Astralane's Solana infrastructure. Ask Claude, Cursor, Gemini, or any MCP-compatible AI to check endpoint latency, optimize tips, protect swaps from sandwich attacks, build bundles, and generate production-ready integration code: without leaving your chat window.

**No API key required to connect. No authentication. Just add the URL and start.**

***

### Server URL

```
https://mcp.astralane.io
```

Transport: SSE (Server-Sent Events) over HTTPS.

***

### Connect Your AI Client

#### Claude Code (CLI)

```bash
claude mcp add astralane --transport sse https://mcp.astralane.io
```

Append `--scope user` to apply globally across all projects, or `--scope project` to write a `.mcp.json` in the current directory.

Manual config (`~/.claude.json` or `.claude/settings.json`):

```json
{
  "mcpServers": {
    "astralane": {
      "url": "https://mcp.astralane.io"
    }
  }
}
```

> Claude Code is the most capable client for Astralane workflows: it runs shell commands natively, so tools that return bash scripts (latency tests, health checks, transaction lookups) execute automatically without copy-pasting.

***

#### VS Code

```bash
code --add-mcp '{"type":"http","name":"astralane","url":"https://mcp.astralane.io"}'
```

Or create `.vscode/mcp.json` in your project:

```json
{
  "mcpServers": {
    "astralane": {
      "url": "https://mcp.astralane.io"
    }
  }
}
```

***

#### Cursor

Inject config from the terminal:

```bash
# Project-scoped
mkdir -p .cursor && echo '{"mcpServers":{"astralane":{"url":"https://mcp.astralane.io"}}}' > .cursor/mcp.json

# Global
mkdir -p ~/.cursor && echo '{"mcpServers":{"astralane":{"url":"https://mcp.astralane.io"}}}' > ~/.cursor/mcp.json
```

Or go to **Settings → Features → MCP → + Add new MCP server** and enter:

* **Name:** `astralane` · **Type:** `sse` · **URL:** `https://mcp.astralane.io`

***

#### Windsurf

Windsurf uses `"serverUrl"` instead of `"url"`:

```bash
mkdir -p .windsurf && echo '{"mcpServers":{"astralane":{"serverUrl":"https://mcp.astralane.io"}}}' > .windsurf/mcp.json
```

Global path: OS application support directory under Windsurf settings.

***

#### Gemini CLI

Open `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "astralane": {
      "type": "sse",
      "url": "https://mcp.astralane.io"
    }
  }
}
```

***

#### OpenAI Codex (CLI)

```bash
codex mcp add astralane --url https://mcp.astralane.io
```

Or via `~/.codex/config.toml`:

```toml
[mcp_servers.astralane]
url = "https://mcp.astralane.io"
```

***

#### Custom Agents (Python / Node)

Python MCP SDK:

```python
from mcp import ClientSession
from mcp.client.sse import sse_client

async with sse_client("https://mcp.astralane.io") as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        result = await session.call_tool("astralane_endpoints", {})
```

Node.js MCP SDK:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { SSEClientTransport } from "@modelcontextprotocol/sdk/client/sse.js";

const client = new Client({ name: "my-agent", version: "1.0.0" }, { capabilities: {} });
const transport = new SSEClientTransport(new URL("https://mcp.astralane.io"));
await client.connect(transport);
const result = await client.callTool({ name: "astralane_endpoints", arguments: {} });
```

***

#### Quick Reference

| Client       | CLI command                                                         | Config file               | Key property  |
| ------------ | ------------------------------------------------------------------- | ------------------------- | ------------- |
| Claude Code  | `claude mcp add astralane --transport sse https://mcp.astralane.io` | `~/.claude.json`          | `"url"`       |
| VS Code      | `code --add-mcp '{"type":"http","name":"astralane","url":"..."}'`   | `.vscode/mcp.json`        | `"url"`       |
| OpenAI Codex | `codex mcp add astralane --url https://mcp.astralane.io`            | `~/.codex/config.toml`    | `url =`       |
| Cursor       | _file injection_                                                    | `~/.cursor/mcp.json`      | `"url"`       |
| Windsurf     | _file injection_                                                    | Windsurf settings JSON    | `"serverUrl"` |
| Gemini CLI   | _file edit_                                                         | `~/.gemini/settings.json` | `"url"`       |

***

### Verify the Connection

After connecting, ask your AI:

> _"List all Astralane endpoints."_

If the AI calls `astralane_endpoints` and returns a table of global gateways, the connection is active.

***

### API Keys

Connecting to the MCP server requires no key. However, some tools generate code or commands that call external APIs: those require keys you obtain separately.

| Key                   | Used by                                                                                                      | Where to get it                                    |
| --------------------- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------- |
| **Astralane API key** | All transaction submission tools (`sendTransaction`, `sendBundle`, `sendIdeal`, `getNonce`, QUIC, WebSocket) | [portal.astralane.io](https://portal.astralane.io) |
| **SolTop API key**    | `astralane_investigate_sandwich`: forensic MEV analysis                                                      | [soltop.sh](https://soltop.sh)                     |
| **Helius API key**    | `astralane_get_priority_fees`: real-time priority fee estimates                                              | [helius.dev](https://helius.dev)                   |
| **Allora API key**    | `astralane_allora_inference`: AI price forecasting (optional; public preview works without a key)            | [allora.network](https://allora.network)           |

**No key required:** DexScreener, RugCheck, Jito tip floor, pump.fun API, Solana public RPC.

***

### Astralane Core Tools

#### Endpoint & Network

| Tool                                 | What it does                                                                                                                             |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `astralane_endpoints`                | List all global gateways with IP, region, datacenter, and ASN. Filter by EU / US / APAC / Global.                                        |
| `astralane_suggest_closest_endpoint` | Returns a bash ping script to run locally. AI executes it, parses output, and reports the lowest-latency host.                           |
| `astralane_health`                   | Returns a curl health-check script for one or all endpoints. Confirms reachability and HTTP status.                                      |
| `astralane_recommend_endpoint`       | Given your use case (QUIC / WebSocket / binary / batch / bundle / single), returns the exact URL, path, protocol, and connection limits. |

#### Tips & Fee Optimization

| Tool                              | What it does                                                                                                                     |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `astralane_get_live_tips`         | Returns curl + WebSocket commands to fetch live tip percentiles (25th–99th) from `tips.astralane.io`. Jito fallback included.    |
| `astralane_tip_info`              | Returns all Astralane tip wallet addresses, minimum tip (10,000 lamports), and write-lock rotation advice.                       |
| `astralane_optimize_tip_volume`   | Given target TPS, congestion level, and live tip data: returns the exact lamport amount to use.                                  |
| `astralane_optimize_tip_ratio`    | Calculates the optimal tip-to-priority-fee split for your goal: `top_of_block`, `fast_landing`, `cost_optimal`, or `sendIdeal`.  |
| `astralane_tip_write_lock_wallet` | Given your transaction's writable accounts, returns a tip wallet that avoids write-lock contention: improves parallel execution. |
| `astralane_estimate_rebates`      | Estimates monthly tip rebates based on transaction volume, submission method, and exclusivity routing.                           |

#### Transaction Submission

| Tool                           | What it does                                                                                                |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `astralane_get_nonce`          | Returns code to call `getNonce` and retrieve a managed durable nonce for `sendIdeal`.                       |
| `astralane_check_transaction`  | Returns curl commands for `getSignatureStatuses` and `getTransaction` on a given signature.                 |
| `astralane_send_ideal_builder` | Step-by-step guide and TypeScript/Rust code for the `sendIdeal` dual-variant nonce-racing flow.             |
| `astralane_mev_bundle_builder` | Complete guide and code for `sendBundle`: atomicity rules, tip placement, compute limits.                   |
| `astralane_integration_guide`  | Full walkthrough for a given submission method (`sendTransaction`, `sendBundle`, `sendIdeal`, `sendBatch`). |

#### MEV & Sandwich Protection

| Tool                             | What it does                                                                                                                                                                                       |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `astralane_sandwich_protection`  | Configures `mev-protect`, `dontFrontRun`, and `jitoDontFrontRun` flags for your strategy (conservative / balanced / aggressive). Optionally queries the Solana leader schedule.                    |
| `astralane_investigate_sandwich` | Given a transaction signature and a SolTop API key, returns a 6-step forensic investigation: detects frontrun, backrun, sandwich, and validator collusion. Powered by [SolTop](https://soltop.sh). |

#### Protocol Guides

| Tool                              | What it does                                                                                                                 |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `astralane_quic_guide`            | QUIC implementation guide: connection setup, 64 concurrent streams, backpressure, MEV-protected port 9000 vs standard 7000.  |
| `astralane_websocket_guide`       | WebSocket guide: binary fire-and-forget, heartbeat handling, connection limits (5000/key, 2/IP), idle timeout.               |
| `astralane_binary_protocol_guide` | `/irisb` binary endpoint: 2-byte length-prefixed frames, `application/octet-stream`, latency vs JSON-RPC.                    |
| `astralane_tip_stream_guide`      | Guide to the `tips.astralane.io/tip_stream` WebSocket: 15-minute rolling percentile snapshots, rate limits, reconnect logic. |
| `astralane_shredstream_guide`     | ShredStream setup: Tier 1 (0.2 SOL/24h) vs Tier 2 (0.5 SOL/24h), UDP shred reception, state reconstruction timing.           |

#### Documentation

| Tool                        | What it does                                                                                    |
| --------------------------- | ----------------------------------------------------------------------------------------------- |
| `astralane_docs`            | Queries Astralane documentation for any topic. Use for anything not covered by a specific tool. |
| `astralane_subsidy_request` | Generates a pre-filled message to request fee subsidies for high-volume operators.              |

***

### Third-Party & Extended Tools

Tools in this section are powered by external APIs and bundled into the MCP server for a unified Solana development experience.

#### Solana RPC Tools

Standard JSON-RPC methods wrapped to return ready-to-execute curl commands. Works with any Solana RPC endpoint.

| Tool                             | What it does                                                                               | External API                 |
| -------------------------------- | ------------------------------------------------------------------------------------------ | ---------------------------- |
| `astralane_get_priority_fees`    | Priority fee estimates (low → very high) with lamport conversion. Requires Helius API key. | [Helius](https://helius.dev) |
| `astralane_simulate_transaction` | Dry-run a serialized transaction before submission.                                        | Solana RPC                   |
| `astralane_get_latest_blockhash` | Fetch current blockhash with commitment level.                                             | Solana RPC                   |
| `astralane_get_balance`          | Check SOL balance for any pubkey.                                                          | Solana RPC                   |
| `astralane_ledger_explorer`      | Query slot history, block production, and validator performance.                           | Solana RPC                   |
| `astralane_whale_tracker`        | Commands to watch large wallet activity via `getProgramAccounts`.                          | Solana RPC                   |

#### DexScreener & Token Intelligence

| Tool                             | What it does                                                                     | External API                           | Key needed |
| -------------------------------- | -------------------------------------------------------------------------------- | -------------------------------------- | ---------- |
| `astralane_dexscreener_trending` | Trending Solana pools and top-boosted tokens. Filter by minimum liquidity.       | [DexScreener](https://dexscreener.com) | No         |
| `astralane_token_security`       | RugCheck risk score, freeze authority, mint authority, top holder concentration. | [RugCheck](https://rugcheck.xyz)       | No         |
| `astralane_price_impact`         | Estimated price impact and slippage for a swap amount on a given pool.           | [DexScreener](https://dexscreener.com) | No         |

#### Expert & Research Tools

| Tool                         | What it does                                                                                                                      | External API                             | Key needed |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ---------- |
| `astralane_sbpf_audit_guide` | sBPF on-chain program security audit: top 5 vulnerabilities: missing signers, re-initialization, PDA injection, integer overflow. | :                                        | No         |
| `astralane_allora_inference` | AI-driven price inference and forecasting for a token mint across 5m / 1h / 24h timeframes.                                       | [Allora Network](https://allora.network) | Optional   |

***

### External APIs Used

| API                    | Purpose                                                                    | Auth                    | Docs                                                 |
| ---------------------- | -------------------------------------------------------------------------- | ----------------------- | ---------------------------------------------------- |
| **Astralane Iris**     | Transaction submission (sendTransaction, sendBundle, sendIdeal, sendBatch) | API key via `?api-key=` | [portal.astralane.io](https://portal.astralane.io)   |
| **Astralane Tips**     | Live tip percentile snapshots and WebSocket stream                         | None                    | `tips.astralane.io`                                  |
| **Jito**               | Tip floor fallback when Astralane tips unavailable                         | None                    | `bundles.jito.wtf/api/v1/bundles/tip_floor`          |
| **SolTop**             | MEV forensics: sandwich, frontrun, backrun detection                       | Bearer token            | [soltop.sh](https://soltop.sh)                       |
| **Helius**             | Priority fee estimation                                                    | API key                 | [helius.dev](https://helius.dev)                     |
| **DexScreener**        | Trending pools, token search, pool metadata                                | None                    | [docs.dexscreener.com](https://docs.dexscreener.com) |
| **RugCheck**           | Token security report: risk score, authority checks, holder distribution   | None                    | [rugcheck.xyz](https://rugcheck.xyz)                 |
| **Allora Network**     | AI price inference and forecasting                                         | Optional                | [allora.network](https://allora.network)             |
| **Pump.fun**           | Token metadata IPFS upload, bonding curve interaction                      | None                    | [pump.fun](https://pump.fun)                         |
| **Jupiter**            | Referenced in integration guides for swap aggregation context              | None                    | [jup.ag](https://jup.ag)                             |
| **Solana Mainnet RPC** | Blockhash, balance, simulation, signature status                           | None                    | `api.mainnet-beta.solana.com`                        |

***

### Example Workflows

**Find your fastest endpoint:**

> _"Run a latency test to all Astralane endpoints and tell me which one I should use."_

AI pings all gateways from your machine, reports the winner with datacenter and ASN for colocation planning.

**Optimize tips in real time:**

> _"Fetch the live tip floor and tell me the exact lamport amount for a swap that needs to land next slot."_

AI fetches live percentiles from `tips.astralane.io`, applies your TPS tier and congestion multiplier, returns a specific lamport value.

**Debug a failed transaction:**

> _"My transaction \[SIG] didn't land. Was it sandwiched?"_

AI checks signature status, fetches full transaction details, then runs the SolTop sandwich investigation.

**Build a bundle:**

> _"Generate TypeScript for a 2-transaction arbitrage bundle with MEV protection."_

AI generates complete `sendBundle` code with correct tip placement, atomicity rules, and write-lock-safe wallet selection.

**Snipe a new token safely:**

> _"Check token \[MINT] for rug indicators, then set up sendIdeal for a fast snipe."_

AI runs RugCheck, and if clean, generates the full `sendIdeal` dual-variant nonce flow.

***

### Architecture Notes

The server is read-only and advisory:

* **No key handling.** Private keys never touch the MCP server.
* **No proxying.** RPC calls, latency tests, and market data queries run on your machine via returned scripts.
* **No stored state.** Each session is independent.

Latency-sensitive operations (transaction submission, market data) stay local. The AI handles reasoning, code generation, and data interpretation.

***

### Support

* Discord: [discord.gg/2UfWGtUDtN](https://discord.gg/2UfWGtUDtN)
* Telegram: [t.me/astralanecommunity](https://t.me/astralanecommunity)
* Portal: [portal.astralane.io](https://portal.astralane.io)
* Astralane Docs: [astralane.gitbook.io/docs](https://astralane.gitbook.io/docs)
