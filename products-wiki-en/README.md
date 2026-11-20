---
description: Low-latency transaction sending and data for Solana
icon: house
---

# Welcome to Astralane

Astralane sends Solana transactions to the current leader over staked connections, and streams block data before most of the network sees it. It is built for trading firms, market makers, searchers and trading apps.

## Start here

{% content-ref url="start-here/quickstart.md" %}
[Quickstart](start-here/quickstart.md)
{% endcontent-ref %}

Land your first transaction in five steps.

## Products

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody>
<tr><td><strong>Send transactions</strong></td><td>Iris routes each transaction to the leader over JSON-RPC, plain text, binary, QUIC or WebSocket.</td><td><a href="send-transactions/README.md">Choose how to send</a></td></tr>
<tr><td><strong>Shreds</strong></td><td>Raw shreds over UDP, direct from leaders, for teams that rebuild blocks themselves.</td><td><a href="data-streams/shreds.md">Shreds</a></td></tr>
<tr><td><strong>Decoded shreds</strong></td><td>The same stream as decoded transactions over gRPC, with filters.</td><td><a href="data-streams/decoded-shreds/README.md">Decoded shreds</a></td></tr>
<tr><td><strong>Postpacks</strong></td><td>A signal from the leader that a transaction holds its account locks, before shreds exist.</td><td><a href="data-streams/postpacks.md">Postpacks</a></td></tr>
<tr><td><strong>Tip stream</strong></td><td>What landed transactions paid in tips over the last 15 minutes.</td><td><a href="data-streams/tip-stream.md">Tip stream</a></td></tr>
<tr><td><strong>Astralane MCP</strong></td><td>Connect Claude, Cursor or Codex to Astralane's endpoints, tips and guides.</td><td><a href="astralane-mcp.md">Astralane MCP</a></td></tr>
</tbody></table>

## Get an API key

Create an account at [portal.astralane.io](https://portal.astralane.io). [Access and support](start-here/access-and-support.md) lists what works straight away and what needs approval.
