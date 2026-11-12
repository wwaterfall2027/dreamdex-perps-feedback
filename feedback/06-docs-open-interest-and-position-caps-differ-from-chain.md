# Perps docs: open interest caps are wrong for 20 of 21 markets, and the per-account cap is not "a tenth" of it

Severity: normal
Area: docs

## Steps to reproduce
1. Open "Perps on Testnet" (`/docs/trading/perps-testnet`). The market table has an "Open interest cap" column, and the text below it says: "A single account is capped at a tenth of it on most markets."
2. "Perps on Testnet for Developers" (`/docs/developers/perps-testnet`) repeats it: "Each account has a position cap per market, a tenth of the market's open interest cap on most markets."
3. Read the live caps with `@somnia-chain/markets-sdk@0.32.1` (read-only, Hideki testnet):
   ```ts
   const rp = await markets.client.getPerpRiskParams(m.poolAddress)
   // rp.maxOpenInterest and rp.maxPositionSize, in raw base units (m.baseDecimals)
   ```

## Expected
The documented open interest caps match `maxOpenInterest`, and `maxPositionSize` is about a tenth of it on most markets.

## Actual
Only TAO matches. On every other documented market the on-chain cap is 2x or 4x the documented one, and the per-account cap is a tenth of it on 1 market out of 21:

| Market | Docs OI cap | Chain `maxOpenInterest` | Chain `maxPositionSize` / `maxOpenInterest` |
|---|---|---|---|
| BTC | 246.81545 | 493.6309 (2x) | 20.3% |
| ETH | 3,995.6125 | 7,991.225 (2x) | 5.0% |
| HYPE | 23,812.72 | 47,625.44 (2x) | 12.6% |
| SOL | 47,654.44 | 95,308.88 (2x) | 5.0% |
| XRP | 344,632 | 1,378,528 (4x) | 5.0% |
| ADA | 2,254,678 | 9,018,712 (4x) | 5.0% |
| AVAX | 66,672.2 | 266,688.8 (4x) | 5.0% |
| BNB | 693.975 | 2,775.9 (4x) | 5.0% |
| DOGE | 5,613,590 | 22,454,360 (4x) | 5.0% |
| LINK | 42,643.6 | 170,574.4 (4x) | 5.0% |
| NEAR | 250,410 | 1,001,640 (4x) | 5.0% |
| SUI | 635,009 | 2,540,036 (4x) | 5.0% |
| XLM | 2,677,780 | 10,711,120 (4x) | 5.0% |
| 1000PEPE | 130,285,600 | 521,142,400 (4x) | 2.5% |
| LIT | 138,896.6 | 555,586.4 (4x) | 2.5% |
| PUMP | 87,993,700 | 351,974,800 (4x) | 2.5% |
| TAO | 2,185.491 | 2,185.491 (matches) | **10.0%** |
| VVV | 21,786.14 | 87,144.56 (4x) | 2.5% |
| WLD | 1,264,000 | 5,056,000 (4x) | 2.5% |
| ZEC | 420 | 1,680 (4x) | 25.0% |
| ZRO | 250,599.5 | 1,002,398 (4x) | 2.5% |

So the per-account cap ranges from 2.5% to 25% of the market cap, and is a tenth on TAO only.

Why it matters: `MaxPositionSizeExceeded()` is a revert a bot has to plan around. The docs send developers to "a tenth of the open interest cap", which is wrong on 20 markets, and the documented open interest caps understate capacity by 2x to 4x. (`getMaxPerpOrderSize` does return the right `maxPositionSize`, so the fix is in the docs.)

## Evidence
`getPerpRiskParams` on 2026-11-12, Hideki testnet, for example:
- BTC: `maxOpenInterest: 49363090000` (493.6309 BTC at 8 decimals), `maxPositionSize: 10000000000` (100 BTC)
- ETH: `maxOpenInterest: 7991225000000000000000`, `maxPositionSize: 399561200000000000000`
- ZEC: `maxOpenInterest: 168000000000` (1,680 ZEC), `maxPositionSize: 42000000000` (420 ZEC, which is the figure the docs give as the market-wide cap)
- TAO: `maxOpenInterest: 2185491000000`, `maxPositionSize: 218549000000` (the only 10% case)

`repro/reproduce.html` (section 06) prints the full table for all 21 markets.

## Environment
`@somnia-chain/markets-sdk` 0.32.1 (browser ESM build), Hideki testnet (chain 50383), indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, read-only (no wallet). Queried 2026-11-12.
