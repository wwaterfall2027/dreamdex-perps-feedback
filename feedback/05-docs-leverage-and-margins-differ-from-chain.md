# Perps docs state the wrong max leverage and margin thresholds for 12 of 21 markets (40x tier undocumented)

Severity: major
Area: docs

## Steps to reproduce
1. Open "Perps on Testnet" (`/docs/trading/perps-testnet`). Its market table gives each market's "Max leverage" and "Initial / maintenance margin", and says the figures "are read from the live markets".
2. Open "Margin & Liquidations" (`/docs/trading/perps-testnet/margin-and-liquidations`). It says max leverage is "20x, 10x or 5x on testnet" and gives the thresholds per tier (20x: 5% / 2.5% / 1.25%, 10x: 10% / 5% / 2.5%, 5x: 20% / 10% / 5%).
3. Read the live parameters for each market with `@somnia-chain/markets-sdk@0.32.1` (read-only, Hideki testnet):
   ```ts
   const perps = await markets.client.listPerpMarkets()
   const m = perps.find((x) => x.baseSymbol === "BTC")!
   const rp = await markets.client.getPerpRiskParams(m.poolAddress)
   // rp.initialMarginBps, rp.maintenanceMarginBps, rp.closeOutMarginBps
   ```
   The indexed row agrees: `m.initialMarginBps`.

## Expected
The table and the margin page match the live markets, as the table claims.

## Actual
12 of the 21 documented markets differ. Every market the docs call 20x except XRP is 40x on chain, and every market the docs call 5x is 10x on chain:

| Market | Docs: max leverage, IM / MM | Chain: max leverage, IM / MM (`getPerpRiskParams`) |
|---|---|---|
| BTC, ETH, HYPE, SOL | 20x, 5% / 2.5% | **40x, 2.5% / 1.25%** (close-out 0.62%) |
| 1000PEPE, LIT, PUMP, TAO, VVV, WLD, ZEC, ZRO | 5x, 20% / 10% | **10x, 10% / 5%** (close-out 2.5%) |
| XRP | 20x, 5% / 2.5% | 20x, 5% / 2.5% (matches) |
| ADA, AVAX, BNB, DOGE, LINK, NEAR, SUI, XLM | 10x, 10% / 5% | 10x, 10% / 5% (matches) |

`getPerpSystemConfig(marginBank).maxLeverageLimit` is `40`, so the 40x tier is real, but the margin page lists only 20x, 10x and 5x. The 5x tier does exist on chain, only for markets the table does not list (AMD, COIN, HOOD, HYNX, INTC, LITE, MRVL, MSTR at 20% initial margin).

Why it matters:
- Liquidation is driven by the maintenance margin. Anyone computing a liquidation price or a safety buffer from the docs is off by a factor of 2 on 12 markets. For example, the docs say a full-leverage BTC position is liquidated after "roughly 2.5%"; on chain it is 1.25% at 40x.
- A bot that caps its own leverage from the table under-uses BTC, ETH, SOL and HYPE, and over-estimates the margin it needs on the eight "5x" markets.

A possible source of the 20x figure: the paper USDso campaign has `leverageCapX: 20` (`getPerpVoucherCampaign(marginBank)`). That is a cap on voucher accounts, not the market's maximum leverage, and the docs do not mention it.

## Evidence
`getPerpRiskParams` on 2026-11-12, Hideki testnet:
- BTC `0x4181dd…`: `initialMarginBps: 250, maintenanceMarginBps: 125, closeOutMarginBps: 62`
- ETH: `250 / 125 / 62`
- TAO: `1000 / 500 / 250` (docs: 20% / 10%)
- ZEC: `1000 / 500 / 250` (docs: 20% / 10%)
- 1000PEPE: `1000 / 500 / 250` (docs: 20% / 10%)
- XRP: `500 / 250 / 125` (matches the docs)
- `getPerpSystemConfig`: `maxLeverageLimit: 40`
- `getPerpVoucherCampaign`: `leverageCapX: 20`

`repro/reproduce.html` (section 05) prints the full comparison for all 21 markets.

## Environment
`@somnia-chain/markets-sdk` 0.32.1 (browser ESM build), Hideki testnet (chain 50383), indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, read-only (no wallet). Queried 2026-11-12.
