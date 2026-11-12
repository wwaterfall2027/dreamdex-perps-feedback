# dreamDEX Perps feedback (Feedback track)

Reproducible reports against the dreamDEX perps testnet (Hideki, chain 50383) and `@somnia-chain/markets-sdk` 0.32.1, in the format requested by the dreamDEX Perps Builder Bounty: one Markdown file per issue, in `feedback/`.

| # | Title | Severity | Area |
|---|---|---|---|
| 01 | [`getMaxPerpOrderSize` reports `priceable: true` for a market whose mark price is 64 h stale (ADLT)](feedback/01-stale-mark-still-priceable.md) | major | SDK |
| 02 | [`listPerpMarkets` returns 5 internal "LOADTEST" markets alongside real ones](feedback/02-loadtest-markets-in-public-list.md) | normal | API |
| 03 | [The base-decimals table omits 15 live markets](feedback/03-base-decimals-table-incomplete.md) | normal | docs |
| 04 | [Read-only setup plus `markets.walletAddress!` gives `undefined`](feedback/04-readonly-snippet-walletaddress-undefined.md) | normal | docs |

Everything was reproduced read-only (no wallet and no funds needed) on 2026-11-11.

## Live reproduction with the Markets SDK

[epro/reproduce.html](repro/reproduce.html) uses `@somnia-chain/markets-sdk@0.32.1` (read-only: no wallet and no keys) to reproduce all four reports against Hideki testnet. Open it from any static server; it prints, for example:

```text
[01] ADLT: markPriceUpdatedAt 65.9 h ago -> priceable=true
[02] LOADTEST markets returned: 5
[03] symbols not in the docs table: LOADTEST, WTI, XAG, XAU, GOOGL, TSLA, HYNX, MSTR, MRVL, LITE, INTC, HOOD, COIN, AMD, ADLT
[04] read-only markets.walletAddress = undefined
```
