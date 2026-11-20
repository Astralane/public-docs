---
description: Why a transaction did not land, and what to check first
icon: life-ring
---

# Troubleshooting

## Accepted is not landed

A signature from Iris means Iris accepted your transaction. Whether it lands depends on the leader, the tip, the fee and the accounts it touches. Check what happened with `getSignatureStatuses` on any RPC.

| What you see | What it means | Where to look |
| --- | --- | --- |
| Landed and succeeded | Done | |
| Landed but failed | The transaction reached a block and the program returned an error. This is not a delivery problem | The transaction logs |
| Never landed, blockhash expired | It was not included before its blockhash ran out | The checks below |
| No signature returned | QUIC and WebSocket never return one. Over JSON-RPC, read the error | [sendTransaction errors](../api-reference/sendtransaction.md#errors) |

## Why a transaction did not land

Check these in order. Most problems are the first three.

1. **Tip below your tier's minimum.** It is rejected. The minimum on the free tier is 0.001 SOL. See [Tips and fees](../send-transactions/tips-and-fees.md).
2. **Tip wallet contention.** Many senders tipping the same wallet lock it, and your transaction waits. Use a different wallet for each send.
3. **Blockhash too old.** Fetch a fresh blockhash right before signing.
4. **Rate limit, or method not enabled.** Over JSON-RPC you get HTTP `429`, or `401` if the method is not enabled on your key. Over QUIC and WebSocket the extra transactions are dropped without an error. Compare your send rate with your tier's TPS in [Pricing and limits](../help/pricing-and-limits.md).
5. **Throttled for a low landing rate.** If almost none of your transactions land, the Astralane team may lower your limits for a while. The portal shows your status.
6. **Priority fee too low.** Leaders that do not run Jito weigh the priority fee, not the tip. Set a normal compute unit price, or use [sendIdeal](../send-transactions/send-ideal.md).
7. **Bundle reverted.** If one transaction in a bundle fails, none of them land. Simulate each transaction first.
8. **Same transaction resent.** Iris ignores a signature it has seen in the last 90 seconds. `sendTransaction` returns the signature again without forwarding it; other methods return `contains duplicate signature`. To retry, sign a new transaction with a fresh blockhash.

## Testing Astralane against another sender

To compare fairly:

* Do not send the same signed transaction through two senders and count which one landed it. Only one can land it, and the result says more about timing than speed.
* Use separate transactions with the same tip and fee, sent at the same moment, over a full day.
* Let a new key warm up for a few minutes first.

## What to send support

Open a ticket on [Discord](https://discord.gg/2UfWGtUDtN) with:

* the signatures, and the slot you expected each one to land in
* the region and transport you used (JSON-RPC, QUIC, WebSocket and so on)
* your tip and priority fee
* roughly how many transactions you send per second
