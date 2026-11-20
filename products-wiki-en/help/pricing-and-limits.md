---
description: Every tier, minimum and limit, for every product, in one place
icon: ranking-star
---

# Pricing and limits

Astralane charges through tips, not monthly fees. You can tip in SOL, WSOL, USDC or USDT. Your tier depends on how much you tipped over a rolling window, and it sets your minimum tip and your rate limits.

## Iris (sending)

| Tier | Tips in the last 7 days | Default minimum tip | Single-transaction methods | sendBundle |
| --- | --- | --- | --- | --- |
| Free | Under 4 SOL | 0.001 SOL | 5 TPS | Off, on request |
| VIP 1 | 4 SOL | 0.0001 SOL | 20 TPS | 5 TPS |
| VIP 2 | 5 SOL | 0.0001 SOL | 40 TPS | 10 TPS |
| VIP 3 | 8 SOL | 0.00001 SOL | 80 TPS | 20 TPS |

* Tiers are checked every hour. An upgrade applies within the hour.
* A downgrade waits 7 days, then moves down one tier at a time.
* To upgrade now, use **Upgrade API key** in the [portal](https://portal.astralane.io) and pay the tier's threshold in tips.
* Higher tiers get lower minimums and higher limits. These are defaults; custom minimums and limits can be agreed per key.
* The minimums are for SOL and WSOL. USDC and USDT tips have their own minimum: see [Tips and fees](../send-transactions/tips-and-fees.md#minimums-for-token-tips).

### When you get throttled

If very few of your transactions land, for example 1% over the last hour, the Astralane team may lower your rate limits and take you off some routes until your landing rate recovers. Your current tier and status are in the portal.

## Shreds

| Tier | Registered tips in the last 24 hours | What you get |
| --- | --- | --- |
| Tier 1 | 0.2 SOL | Baseline leader coverage, for development and testing |
| Tier 2 | 0.5 SOL | Wider leader coverage, for production |

Tips count once you register them through ShredPay. Downgrades wait 24 hours and step down one tier at a time. See [Shreds](../data-streams/shreds.md).

## Decoded shreds and postpacks

| Product | Price |
| --- | --- |
| Decoded shreds | Free for tier-2 shred users (0.5 SOL of registered tips in 24 hours) |
| Postpacks | Tips registered through PostpackPay |

## Connection limits

| Service | Limit |
| --- | --- |
| QUIC | 10 connections per key, 64 streams per connection, 30 s idle timeout |
| WebSocket | 5,000 connections per key, 2 per IP, 300 s idle timeout |
| Tip stream | 10 requests per second per IP, 2 WebSocket connections per IP |
| sendBundle | Up to 4 transactions per bundle |
| sendBatch | Up to 25 transactions per request |
| Transaction size | 1,232 bytes for legacy and v0 transactions, 4,096 bytes for v1 |


## Custom terms

High-volume users can ask for custom minimums and limits, private tip wallets, rebates and colocation. See [Access and support](../start-here/access-and-support.md).
