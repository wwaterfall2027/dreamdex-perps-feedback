# Funding docs omit the +0.01% interest component: longs pay on 18 markets where the perp trades below the index

Severity: normal
Area: docs

## Steps to reproduce
1. Open "Funding" (`/docs/trading/perps-testnet/funding`). It says:
   - "Perp above the index (positive funding): longs pay shorts."
   - "Perp below the index (negative funding): shorts pay longs."
   It gives no formula for the rate itself.
2. Read each market's funding state with `@somnia-chain/markets-sdk@0.32.1` (read-only, Hideki testnet):
   ```ts
   const st = await markets.client.getPerpState(m.poolAddress)
   // st.emaPremium and st.fundingRate, both 1e18 = 100%
   ```

## Expected
When the premium is negative (perp below the index), the funding rate is negative and shorts pay longs, as the page says.

## Actual
On 18 of the 34 live markets the premium is negative but the funding rate is **+0.0100%**, so longs pay shorts. Examples on 2026-11-12:

| Market | `emaPremium` | `fundingRate` |
|---|---|---|
| SOL | −0.01953% | +0.0100% |
| AMD | −0.00478% | +0.0100% |
| ZEC | −0.00509% | +0.0100% |
| ZRO | −0.00486% | +0.0100% |
| HYPE | −0.00336% | +0.0100% |

On all 34 markets the rate fits one formula exactly: an interest component of 0.01% per 8 h, with the difference clamped to ±0.05%:

```
fundingRate = premium + clamp(0.01% − premium, −0.05%, +0.05%)
```

It checks out at both ends of the clamp too: BTC premium +0.1519% gives rate +0.1019%, and ETH premium −0.1248% gives −0.0748%.

So:
- Funding is **not** zero-centred. For any premium between −0.04% and +0.06% the rate is a fixed +0.01% per 8 h, which longs pay.
- The sign of the rate is not the sign of "perp vs index", which is the only rule the page gives.

Why it matters: a funding-arbitrage or carry bot built from the page expects to be paid for holding a long when the perp is below the index, and pays instead. The worked example on the page (0.08%) cannot be reproduced from a premium without the formula.

Suggested fix: document the interest rate, the clamp, and that `emaPremium` is a time-weighted average of the premium over the window.

## Evidence
`getPerpState` on 2026-11-12, Hideki testnet:
- SOL `0xb483ec…`: `emaPremium: -195308746363374`, `fundingRate: 100000000000000`
- BTC `0x4181dd…`: `emaPremium: 1519355828312783`, `fundingRate: 1019355828312783` (= premium − 0.05%)
- ETH `0xd6cb1d…`: `emaPremium: -1247607942303512`, `fundingRate: -747607942303512` (= premium + 0.05%)

`repro/reproduce.html` (section 07) checks the formula on every market and lists the markets where longs pay with a negative premium.

## Environment
`@somnia-chain/markets-sdk` 0.32.1 (browser ESM build), Hideki testnet (chain 50383), indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, read-only (no wallet). Queried 2026-11-12.
