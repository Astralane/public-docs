---
icon: circle-dollar-to-slot
---

# Tip in tokens or ALTs

You can pay the tip in WSOL, USDC or USDT instead of SOL. That lets you budget tips in dollars and pay them from the token balance you already trade with. SOL tips work exactly as before.

A token tip goes to the tip wallet's Associated Token Account (ATA) for that token, not to the wallet address itself. Every tip wallet already has an ATA for all three tokens, so you never need to create one.

1. Pick a tip wallet from the [list here](../endpoints-and-configs/).
2. Derive that wallet's ATA for the token you are tipping in.
3. Add an SPL Token `Transfer` or `TransferChecked` from your own token account to that ATA, in the same transaction.
4. Send the transaction through your usual Iris endpoint.

Amounts are in the token's smallest unit:

<table><thead><tr><th width="81.296875">Token</th><th>Mint</th><th width="95.9296875">Decimals</th><th>Example</th></tr></thead><tbody><tr><td>WSOL</td><td><code>So11111111111111111111111111111111111111112</code></td><td>9</td><td>0.001 WSOL = <code>1,000,000</code></td></tr><tr><td>USDC</td><td><code>EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v</code></td><td>6</td><td>0.25 USDC = <code>250,000</code></td></tr><tr><td>USDT</td><td><code>Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB</code></td><td>6</td><td>0.25 USDT = <code>250,000</code></td></tr></tbody></table>

A USDC tip in TypeScript:

```javascript
import { PublicKey } from "@solana/web3.js";
import { createTransferCheckedInstruction, getAssociatedTokenAddressSync } from "@solana/spl-token";

const USDC = new PublicKey("EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v");
const TIP_WALLET = new PublicKey("astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF");
const TIP_AMOUNT = 250_000n; // 0.25 USDC

const fromAta = getAssociatedTokenAddressSync(USDC, payer.publicKey); // your USDC account
const tipAta = getAssociatedTokenAddressSync(USDC, TIP_WALLET);       // the tip wallet's USDC account

// Add this instruction to the transaction you send through Iris
const tipIx = createTransferCheckedInstruction(fromAta, USDC, tipAta, payer.publicKey, TIP_AMOUNT, 6);
```

### How token tips are valued <a href="#how-token-tips-are-valued" id="how-token-tips-are-valued"></a>

WSOL counts 1:1 with SOL. USDC and USDT are converted to their SOL value at a cached market price (cmc 3 min average price), and Iris uses that value both to check your tier's minimum and to route the transaction. Because the price moves, tip a little above your minimum rather than exactly at it.

### Tip accounts in an Address Lookup Table <a href="#tip-accounts-in-an-address-lookup-table" id="tip-accounts-in-an-address-lookup-table"></a>

In a v0 transaction, the tip account can come from an Address Lookup Table (ALT) instead of being listed in full. Add the SOL tip wallet to your ALT, or, for a token tip, the tip wallet's ATA for that token, and reference it from your transaction. Iris resolves the account when it reads the tip.

This saves space when your transaction already uses an ALT, for example for a swap's accounts. The tip account then takes a 1-byte index instead of a 32-byte address. Since you rotate tip wallets, add every wallet you rotate through, or its token accounts, to the table.

### Refunds <a href="#refunds" id="refunds"></a>

Transactions that land for less than they tipped can earn part of the tip back. See [Tip refunds](tip-refunds.md).
