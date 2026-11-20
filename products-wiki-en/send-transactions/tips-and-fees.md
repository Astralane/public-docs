---
description: Minimum tips, tip wallets, token tips, and how much to tip
icon: coins
---

# Tips and fees

Every transaction sent through Iris pays a tip: a transfer to one Astralane tip wallet, inside the transaction. You can tip in SOL, WSOL, USDC or USDT. Ghost transactions are the only exception: they pass the tip as a parameter.

## Minimum tip

| Tier | Default minimum tip | In lamports |
| --- | --- | --- |
| Free | 0.001 SOL | 1,000,000 |
| VIP 1 | 0.0001 SOL | 100,000 |
| VIP 2 | 0.0001 SOL | 100,000 |
| VIP 3 | 0.00001 SOL | 10,000 |

These are the defaults; a custom minimum can be agreed per key. The minimum is a floor. Over JSON-RPC a tip below it is rejected with `-32600` `Invalid Request: min tip not paid for any asset`; over plain text and binary you get HTTP `400` with the same message; over QUIC and WebSocket the transaction is dropped. Token tips have their own minimums, described in [Minimums for token tips](#minimums-for-token-tips). How tiers are earned is in [Pricing and limits](../help/pricing-and-limits.md).

## Tip in WSOL, USDC or USDT

You can pay the tip in WSOL, USDC or USDT instead of SOL. That lets you budget tips in dollars and pay them from the token balance you already trade with. SOL tips work exactly as before.

A token tip goes to the tip wallet's Associated Token Account (ATA) for that token, not to the wallet address itself. Every tip wallet already has an ATA for all three tokens, so you never need to create one.

1. Pick a tip wallet from the [list below](#tip-wallets).
2. Derive that wallet's ATA for the token you are tipping in.
3. Add an SPL Token `Transfer` or `TransferChecked` from your own token account to that ATA, in the same transaction.
4. Send the transaction through your usual Iris endpoint.

Iris only counts a token tip that:

* uses the SPL Token program. Token-2022 transfers are not counted.
* goes to the tip wallet's associated token account, not any other token account it owns.
* is a top-level instruction in your transaction. A transfer made inside another program's instruction is not seen.

A `TransferChecked` with the wrong mint or decimals rejects the whole transaction.

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

### Tip wallet token accounts

Instead of deriving it, you can copy the token account for each tip wallet from this table. These are the SPL Token associated token accounts Iris counts.

| Tip wallet | WSOL | USDC | USDT |
| --- | --- | --- | --- |
| `astrazznxsGUhWShqgNtAdfrzP2G83DzcWVJDxwV9bF` | `G2cGwrLbiXA4gKsGuUbsM8ZwSQPRe1uv5BxmyTqZpLgA` | `4KUiw9ct6H2DtjQ65wjPh9EuAkRjT5ABohRJ6WK8VfTa` | `D2T61t56rwJos3oCxHmcQYQdnES4FfR2KYYQsGPSfHb1` |
| `astra4uejePWneqNaJKuFFA8oonqCE1sqF6b45kDMZm` | `HiTFQfXAyB5WXhWVsX6BvjBUxUji2146uHftjU3dZQLe` | `7mPs8UuZDpDxg7BaCLwj1ccfsFAsjk9eSi41XCx54PEZ` | `7gtf2Ay4tkf15fh1vjU642X6EJynFY3Wcon6asXWyqoC` |
| `astra9xWY93QyfG6yM8zwsKsRodscjQ2uU2HKNL5prk` | `98HqoC8NBgzfti64VAv3AjSr4AEypUxmqGg96rcZh86T` | `GhWtEHgZbBj9muW5FXmopMEx4WKTzauBrTFh9iqYarMU` | `6rZCXMmPhu2WNrgZVVhicUkG6Kiz2vaDYpFuif114dcH` |
| `astraRVUuTHjpwEVvNBeQEgwYx9w9CFyfxjYoobCZhL` | `2PjGQYeJDqND2XTJWgySzZGJgjL4yx2mPECq1HdgHGjC` | `6XMcsS7UABKGo9vHNtkpvo7e6ChzaPhHgnAPjqkt8RTB` | `hexmWc82YMrZMwy4fKngU7eUASvydt6eqHpVnMRd5go` |
| `astraEJ2fEj8Xmy6KLG7B3VfbKfsHXhHrNdCQx7iGJK` | `Hw96uLDaPDgcYCYYUsL3VfYB2MTZiDQ1ormz119aaJUv` | `H5Sisc8zTsjU23iVMQT2aBoYB6Ls5pV2UozNZJuH3Hvz` | `Dx9xTR28x4jri6uiFkxDNHRJejKogmdHLfUjLByvekMQ` |
| `astraubkDw81n4LuutzSQ8uzHCv4BhPVhfvTcYv8SKC` | `nEsxXB1xt3x2W35jQF6E7QGKGfmrKFcv6YYnEsBacmh` | `Hc158Ed7sRyrXeudxAWd7f2GE35QyPMsWTGCjd8smm96` | `DmAK8dVph1Wz4T5UMRyw7WhPrjWmnjZNFzibF4HhuGi2` |
| `astraZW5GLFefxNPAatceHhYjfA1ciq9gvfEg2S47xk` | `HkNkNrcLLbXPVwNDZVixWD38cVt98exLFkftSdMXzmNJ` | `3ufjLLUCmtpkLi2Mx12z7z7WYX2TomFZ5WstKY6RKF5b` | `3Hgem8S9FGDojRoYpCL3Pd23NKX7MGm8sZU4Y3gM2XJP` |
| `astrawVNP4xDBKT7rAdxrLYiTSTdqtUr63fSMduivXK` | `EwLVD3YDzLg7idccmgzqLc26rDYrvyhWf9jEkKgtEjRU` | `DutwtCXhneSszhdanitiiXcmFsH5mMwJHRYphBjumRtd` | `4DwbWTMAdD4f9P3yvXgDe8i1A6eJSH64jia4nVgwe4fR` |
| `AstrA1ejL4UeXC2SBP4cpeEmtcFPZVLxx3XGKXyCW6to` | `G4TKzDPGZyHZT48TRALCQzwAYRmuefZ8KmKQmDL4KsfG` | `CQo8HPMSVepBKW1aNXjUGepzxYqG1zLXZUJV3rHv21rd` | `6qRj58LBdhCYfemhk7BYBFYXVyS3BGf64w1HLb31MTFm` |
| `AsTra79FET4aCKWspPqeSFvjJNyp96SvAnrmyAxqg5b7` | `A6nT74ZuHiwEhCuHgDvtUJtaspzcvC1Xo39XmD79xTdx` | `iwFZ3MFrtcMYL7oQrfPVRPaLoFyZ7byohdfsxWKktrq` | `Zev1Mkw2tWs7NVw36nCRy4tF3MS7fo5PyuqMqatph9V` |
| `AstrABAu8CBTyuPXpV4eSCJ5fePEPnxN8NqBaPKQ9fHR` | `EabPzKurLvdKzQj8ffmgRga6qPDTd5hv34L7Mnb6URwq` | `G9hTY51eBbQ6aM15Mzs6LUmBTXB7jo5NQNPnzFKcKoQS` | `1adSMs4kQKrzR5y2EZVJ7rJfzmwKj4AGN3VPoTL6qer` |
| `AsTRADtvb6tTmrsqULQ9Wji9PigDMjhfEMza6zkynEvV` | `3qoDx124YVLcMnbXSGc5otahqcxNBeX1AL2KKcepmzVj` | `2rEjmAQYCZdXTLrhxDdMqMZVvaREcju7SqvNoT6FstRy` | `4v54FBbHBaDtc4XzKpXpTaaoBuUgWUqr7VTxL8bMquyT` |
| `AsTRAEoyMofR3vUPpf9k68Gsfb6ymTZttEtsAbv8Bk4d` | `EcUGMgQLYHxBvBKnJLBCPG3no6ANnxbTbJY974PdyHA4` | `7AkKb6Cd3HbnTxKAEHv8JS7rxMykmnJZvvH7CxgHurKz` | `8hKuRBtyXXFdTUE11exZVizBdLkJemKGdRSt2sX9foDu` |
| `AStrAJv2RN2hKCHxwUMtqmSxgdcNZbihCwc1mCSnG83W` | `4Sk3Xs6j6fC3VQwd53kgK68G4DHDTyjPJSY4YYTRCeEL` | `GSfxtoF6kvbMf1GtRqsQ3gPQE7BQrWQktL1Xun5PZWR2` | `HkogeeusKwb17G8mNCW25iCi7Ceuf9UNUh7yMf8GKhuw` |
| `Astran35aiQUF57XZsmkWMtNCtXGLzs8upfiqXxth2bz` | `BUSK5KAHj776qiRfoqACqvKiGBhKYZCHKPnAs4zo8N5e` | `5THHwZ1SvbqsDHFEPuSkWcFYSTFn8YdfzA7Q6J7CErsu` | `68yAeqd7SmtXwvEJ6B11puy3cYF53h7UvdT9tuBZddrJ` |
| `AStRAnpi6kFrKypragExgeRoJ1QnKH7pbSjLAKQVWUum` | `92A4xc3sDKKakpDDPeiQrLqbchpRRgdA45DBwZbTTfqi` | `2M4HQPyqf2vTPTPYXsTjN12c853g3wDohX2vyCvP7Kbn` | `ECht2uD6cVC6MkDd3Gp2Whm3ac3pshQmmVMxxgAnsqwK` |
| `ASTRaoF93eYt73TYvwtsv6fMWHWbGmMUZfVZPo3CRU9C` | `2bQbSyQUnecPHhUvpNkbTGdFMSQCd6xsvaXDeq74sy5D` | `eMdXDFvUkQPHGxQ6CREaTTX6KzyyRxzNVkjvcCDqPoJ` | `4LoX8QJ1Zgca4yfJo5wkE4cM8bZQvcvBTavt259tR6H4` |

## Minimums for token tips

Each asset is checked against its own minimum. A transaction passes if any one of them is met:

| Asset | Minimum |
| --- | --- |
| SOL | Your tier's minimum |
| WSOL | Your tier's minimum, counted 1:1 with SOL |
| USDC | 0.1 USDC (`100,000`) unless agreed otherwise |
| USDT | 0.1 USDT (`100,000`) unless agreed otherwise |

Amounts in different assets are not added together. Half the minimum in SOL plus half in WSOL does not pass.

## How token tips are valued

For routing, Iris converts USDC and USDT tips to their SOL value at a market price refreshed about every minute. WSOL counts 1:1 with SOL. The SOL value decides which routes a transaction can use; it does not change the minimums above.

Token tips count toward your VIP tier. They are not refunded: [tip refunds](tip-refunds.md) are calculated on SOL tips only. ShredPay and PostpackPay only register SOL tips, and do not support Address Lookup Tables.

## Tip accounts in an Address Lookup Table

In a v0 transaction, the tip account can come from an Address Lookup Table (ALT) instead of being listed in full. Add the SOL tip wallet to your ALT, or, for a token tip, the tip wallet's ATA for that token, and reference it from your transaction. Iris resolves the account when it reads the tip.

This saves space when your transaction already uses an ALT, for example for a swap's accounts. The tip account then takes a 1-byte index instead of a 32-byte address. Since you rotate tip wallets, add every wallet you rotate through, or its token accounts, to the table.

If Iris cannot resolve the table, the tip is not counted and the transaction fails the minimum check. v1 transactions have no lookup tables, so list the tip account directly.

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

To fetch the current list instead of hard-coding it, call `getTipWallets`. No API key is needed:

```bash
curl -s -X POST https://api.gateway.astralane.io/public-api \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"getTipWallets"}'
```

It returns the wallet addresses as an array in `result`.

Shreds and postpacks use their own tip wallets, listed on [Shreds](../data-streams/shreds.md) and [Postpacks](../data-streams/postpacks.md).

## Tip or priority fee

Some leaders run Jito and weigh tips; others run Agave or Firedancer and weigh priority fees. Iris sends through both paths, so most transactions need a tip at or above the minimum and a normal priority fee. If you cannot decide, [sendIdeal](send-ideal.md) sends one tip-heavy and one fee-heavy version and lets whichever lands first win.

## Refunds

Transactions that land for less than they tipped can earn part of the tip back. See [Tip refunds](tip-refunds.md).
