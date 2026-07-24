---
icon: ranking-star
---

# Send Txn Fee Tiers

The fee tier system dynamically adjusts a user’s minimum tip and TPS allocation based on their rolling tip volume over a rolling 7 day period. The system is designed to reward high tipping users with lower minimum tip requirements and higher rate limits.

The system runs on a rolling 7-day window, evaluating all users every hour.

Tier changes are immediate, downgrades are delayed by a grace period of 7 days.

<table><thead><tr><th width="86.51953125">Tiers</th><th width="129.85546875">Minimum tip</th><th>Tip volume thresholds rolling 7 days</th><th width="118.2421875">single txn methods</th><th width="122.61328125">bundle txn method</th><th>QoS rate limiting</th></tr></thead><tbody><tr><td>Free</td><td>0.001</td><td>N/A</td><td>5 TPS</td><td>0 TPS</td><td>every 2 hours</td></tr><tr><td>VIP 1</td><td>0.0001</td><td>3</td><td>20 TPS</td><td>5 TPS</td><td>N/A</td></tr><tr><td>VIP 2</td><td>0.0001</td><td>5</td><td>40 TPS</td><td>10 TPS</td><td>N/A</td></tr><tr><td>VIP 3</td><td>0.00001</td><td>8</td><td>80 TPS</td><td>20 TPS</td><td>N/A</td></tr></tbody></table>

Users may view their current tier stats on [portal.astralane.io](http://portal.astralane.io)

### QoS rate limiting

If your transactions have a high failure rate, a QoS penalty will apply to your execution priority. For example, if only 1% of your transactions land successfully over the last 1 hour, access to certain exclusive paths will be throttled/reset rather than continously available, including lower rate limits applied to all endpoints.

### Custom Limits

High volume and enterprise users may reach out via discord/telegram for custom minimum tip/rate limit configurations and tip rebates/refunds.

### FAQ’s

<details>

<summary>Do different tiers receive different priority?</summary>

No, all tiers receive the same scheduling priority, our routers run in pure FIFO mode, unless you are QoS rate limited because of spam/abuse. QoS rate limits only apply to users on the free tier.

</details>

<details>

<summary>How do these tiers work if i am actively tipping to Astralane?</summary>

Suppose you move from the Free tier to VIP 1 after tipping more than 3 SOL over the previous rolling 7-day period. You are immediately upgraded and receive the benefits of VIP 1, including a reduced minimum tip of 0.0001 SOL per transaction, along with higher TPS limits for both single transaction and bundle transaction methods.

If your rolling 7-day tip volume later drops below the 3 SOL threshold (for example, to 2 SOL), you are not downgraded immediately. Instead, you remain on VIP 1 and continue receiving its benefits during a 7-day grace period. Only after the grace period expires, if your rolling volume is still below the required threshold, will you be downgraded back to the Free tier.

Similarly, suppose you move from VIP 1 to VIP 2 after exceeding the 5 SOL threshold, and your rolling 7-day volume later falls significantly below it (for example, to 2 SOL). You are not immediately downgraded. Instead, you remain on VIP 2 and continue receiving its benefits during a 7-day grace period.

After the grace period ends, if your volume still does not qualify for VIP 2, you are downgraded one tier at a time, moving first to VIP 1. If your rolling volume still remains below the VIP 1 threshold (3 SOL), another 7-day grace period begins on VIP 1. Only after this second grace period, if your volume remains below the requirement, will you be downgraded to the Free tier.

</details>

<details>

<summary>How do i get access to sendBundle?</summary>

Available in the Free tier. This feature is available for all users at different TPS according to the current tier.

</details>

<details>

<summary>How do i get access to Quic?</summary>

All users have access to Quic, even free plan users.

</details>

<details>

<summary>How do i get private tip wallets to reduce write lock conflicts?</summary>

All users have access to check write lock contention on our public tip wallets using the blockline interface - kindly use this data to do a weighted round robin based on write lock contention every 1-2 days synced. If you are still noticing poor landing priority when fee / tip is higher - you can reach out to us for optimization suggestions and more tip wallets.

</details>

