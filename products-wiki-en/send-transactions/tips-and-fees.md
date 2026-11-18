---
description: Minimum tips, tip wallets, token tips, and how much to tip
icon: coins
---

# Tips and fees

Every transaction sent through Iris pays a tip: a transfer to one Astralane tip wallet, inside the transaction. You can tip in SOL, WSOL, USDC or USDT. Ghost transactions are the only exception: they pass the tip as a parameter.

## Minimum tip

| Tier | Minimum tip | In lamports |
| --- | --- | --- |
| Free | 0.001 SOL | 1,000,000 |
| VIP 1 | 0.0001 SOL | 100,000 |
| VIP 2 | 0.0001 SOL | 100,000 |
| VIP 3 | 0.00001 SOL | 10,000 |

The minimum is a floor. Tips below it are rejected. A token tip counts at its SOL value, as described in [How token tips are valued](#how-token-tips-are-valued). How tiers are earned is in [Pricing and limits](../help/pricing-and-limits.md).

<!-- review: Confirm these minimums and add the exact error a user gets below them. All current code samples and the MCP use 10,000 lamports, which is the VIP 3 minimum only. -->

## Tip in WSOL, USDC or USDT

You can pay the tip in WSOL, USDC or USDT instead of SOL. That lets you budget tips in dollars and pay them from the token balance you already trade with. SOL tips work exactly as before.

A token tip goes to the tip wallet's Associated Token Account (ATA) for that token, not to the wallet address itself. Every tip wallet already has an ATA for all three tokens, so you never need to create one.

1. Pick a tip wallet from the [list below](#tip-wallets).
2. Derive that wallet's ATA for the token you are tipping in.
3. Add an SPL Token `Transfer` or `TransferChecked` from your own token account to that ATA, in the same transaction.
4. Send the transaction through your usual Iris endpoint.

Amounts are in the token's smallest unit:

| Token | Mint | Decimals | Example |
| --- | --- | --- | --- |
| WSOL | `So11111111111111111111111111111111111111112` | 9 | 0.001 WSOL = `1,000,000` |
| USDC | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` | 6 | 0.25 USDC = `250,000` |
| USDT | `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB` | 6 | 0.25 USDT = `250,000` |

A USDC tip in TypeScript:

```typescript
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

<!-- review: All 51 tip-wallet token accounts (17 wallets x WSOL, USDC and USDT) were checked on-chain on 6 Nov 2026, and every one exists. We can share the full list of 51 addresses if you would rather users paste them than derive them. -->

## How token tips are valued

WSOL counts 1:1 with SOL. USDC and USDT are converted to their SOL value at a cached market price, and Iris uses that value both to check your tier's minimum and to route the transaction. Because the price moves, tip a little above your minimum rather than exactly at it.

<!-- review: How often is the cached price refreshed? Do token tips count toward the VIP tiers (tips over 7 days) and toward tip refunds at their SOL value? Can a token tip be registered through ShredPay? -->

## Tip accounts in an Address Lookup Table

In a v0 transaction, the tip account can come from an Address Lookup Table (ALT) instead of being listed in full. Add the SOL tip wallet to your ALT, or, for a token tip, the tip wallet's ATA for that token, and reference it from your transaction. Iris resolves the account when it reads the tip.

This saves space when your transaction already uses an ALT, for example for a swap's accounts. The tip account then takes a 1-byte index instead of a 32-byte address. Since you rotate tip wallets, add every wallet you rotate through, or its token accounts, to the table.

<!-- review: Does ALT support cover v1 transactions as well as v0? -->

## How much to tip

The [tip stream](../data-streams/tip-stream.md) shows what transactions that landed through Astralane paid in the last 15 minutes. On 19 Oct 2026 at 06:35 UTC it read:

| Percentile | Lamports | SOL |
| --- | --- | --- |
| 25th | 12,500 | 0.0000125 |
| 50th | 100,000 | 0.0001 |
| 75th | 1,000,000 | 0.001 |
| 95th | 2,000,000 | 0.002 |
| 99th | 5,000,001 | 0.005 |

Around the median usually lands. Use the 95th percentile when the market is busy. The numbers describe tips that landed; they are not a landing probability.

## Tip wallets

Pick a different wallet for each send. Tips to one wallet lock that account, so transactions that share a wallet queue behind each other. For a token tip, send to the wallet's token account, as described in [Tip in WSOL, USDC or USDT](#tip-in-wsol-usdc-or-usdt).

All 17 wallets, ready to paste into your config:

```
astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF,
astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm,
astra9xWY93QyfG6yM8zwsKsRodscjQ2uU2HKNL5prk,
astraRVUuTHjpwEVvNBeQEgwYx9w9CFyfxjYoobCZhL,
astraEJ2fEj8Xmy6KLG7B3VfbKfsHXhHrNdCQx7iGJK,
astraubkDw81n4LuutzSQ8uzHCv4BhPVhfvTcYv8SKC,
astraZW5GLFefxNPAatceHhYjfA1ciq9gvfEg2S47xk,
astrawVNP4xDBKT7rAdxrLYiTSTdqtUr63fSMduivXK,
AstrA1ejL4UeXC2SBP4cpeEmtcFPZVLxx3XGKXyCW6to,
AsTra79FET4aCKWspPqeSFvjJNyp96SvAnrmyAxqg5b7,
AstrABAu8CBTyuPXpV4eSCJ5fePEPnxN8NqBaPKQ9fHR,
AsTRADtvb6tTmrsqULQ9Wji9PigDMjhfEMza6zkynEvV,
AsTRAEoyMofR3vUPpf9k68Gsfb6ymTZttEtsAbv8Bk4d,
AStrAJv2RN2hKCHxwUMtqmSxgdcNZbihCwc1mCSnG83W,
Astran35aiQUF57XZsmkWMtNCtXGLzs8upfiqXxth2bz,
AStRAnpi6kFrKypragExgeRoJ1QnKH7pbSjLAKQVWUum,
ASTRaoF93eYt73TYvwtsv6fMWHWbGmMUZfVZPo3CRU9C
```

Shreds and postpacks use their own tip wallets, listed on [Shreds](../data-streams/shreds.md) and [Postpacks](../data-streams/postpacks.md).

<!-- review: Today three pages give three different rotation rules (every send, by region, weighted by write-lock contention). This page keeps one. A call that returns the current wallets would stop bots hard-coding this list. -->

## Tip or priority fee

Some leaders run Jito and weigh tips; others run Agave or Firedancer and weigh priority fees. Iris sends through both paths, so most transactions need a tip at or above the minimum and a normal priority fee. If you cannot decide, [sendIdeal](send-ideal.md) sends one tip-heavy and one fee-heavy version and lets whichever lands first win.

## Refunds

Transactions that land for less than they tipped can earn part of the tip back. See [Tip refunds](tip-refunds.md).
