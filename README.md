# dreamDEX Perps feedback (Feedback track)

Reproducible reports against the dreamDEX perps testnet (Hideki, chain 50383) and `@somnia-chain/markets-sdk` 0.32.1, in the format requested by the dreamDEX Perps Builder Bounty: one Markdown file per issue, in `feedback/`.

| # | Title | Severity | Area |
|---|---|---|---|
| 01 | [`getMaxPerpOrderSize` reports `priceable: true` for a market whose mark price is 64 h stale (ADLT)](feedback/01-stale-mark-still-priceable.md) | major | SDK |
| 02 | [`listPerpMarkets` returns 5 internal "LOADTEST" markets alongside real ones](feedback/02-loadtest-markets-in-public-list.md) | normal | API |
| 03 | [The base-decimals table omits 15 live markets](feedback/03-base-decimals-table-incomplete.md) | normal | docs |
| 04 | [Read-only setup plus `markets.walletAddress!` gives `undefined`](feedback/04-readonly-snippet-walletaddress-undefined.md) | normal | docs |
| 05 | [Perps docs state the wrong max leverage and margin thresholds for 12 of 21 markets (40x tier undocumented)](feedback/05-docs-leverage-and-margins-differ-from-chain.md) | major | docs |
| 06 | [Open interest caps are wrong for 20 of 21 markets, and the per-account cap is not "a tenth" of it](feedback/06-docs-open-interest-and-position-caps-differ-from-chain.md) | normal | docs |
| 07 | [Funding docs omit the +0.01% interest component: longs pay on 18 markets where the perp trades below the index](feedback/07-funding-docs-omit-interest-rate-and-clamp.md) | normal | docs |

Everything was reproduced read-only (no wallet and no funds needed): 01 to 04 on 2026-11-11, 05 to 07 on 2026-11-12.

## Live reproduction with the Markets SDK

[repro/reproduce.html](repro/reproduce.html) uses `@somnia-chain/markets-sdk@0.32.1` (read-only: no wallet and no keys) to reproduce all seven reports against Hideki testnet. Open it from any static server; it prints, for example:

```text
[01] ADLT: markPriceUpdatedAt 69.6 h ago -> priceable=true
[02] LOADTEST markets returned: 5
[03] symbols not in the docs table: LOADTEST, WTI, XAG, XAU, GOOGL, TSLA, HYNX, MSTR, MRVL, LITE, INTC, HOOD, COIN, AMD, ADLT
[04] read-only markets.walletAddress = undefined
[05/06] BTC docs 20x | chain 40x (IM 2.5% / MM 1.25%) | OI cap docs 246.81545 chain 493.6309 | account cap 20.3% of OI
  -> leverage differs on 12/21, OI cap differs on 20/21, account cap is a tenth on 1/21; maxLeverageLimit = 40
[07] formula fits 34/34 markets
  negative premium but longs pay: 18 markets
    SOL (premium -0.01953%, rate +0.0100%)
```
