# listPerpMarkets returns 5 internal "LOADTEST" markets alongside real ones

Severity: normal
Area: API

## Steps to reproduce
1. Connect read-only with `@somnia-chain/markets-sdk@0.32.1` to Hideki testnet, as in the developer docs.
2. `const perps = await markets.client.listPerpMarkets()`
3. `perps.filter(m => m.baseSymbol === "LOADTEST").map(m => m.poolAddress)`

## Expected
Only tradeable markets, or a flag (e.g. `hidden`, `status`) so integrators can tell internal load-test markets from real ones. The docs never mention LOADTEST.

## Actual
40 markets are returned, and 5 of them are `baseSymbol: "LOADTEST"`:
- `0x736850b162802db2b54d766c686e65e1b31eb954`
- `0x7d19508851d5397fd0e3b648fe30982bf86268a5`
- `0xf909076d4abc6e7993e499b533e2b2a8cfbe4dd4`
- `0xb738977760660b455b0ef9e7fa3eec0205c1292d`
- `0x7982b98b08f9956e6665366ae93c466b44b044a5`

One has no mark price at all (`getMaxPerpOrderSize` gives `priceable: false`). The others have marks 13 to 35 h old. All five share the same `baseSymbol`, so code that indexes markets by symbol (as the docs' `perps.find(m => m.baseSymbol === "BTC")` pattern encourages) silently picks an arbitrary one. A bot that iterates "all markets" also wastes calls on them, or errors on them.

## Evidence
`listPerpMarkets()` returned 40 rows on 2026-11-11. The 5 LOADTEST pool addresses are listed above, all with marginBank `0x478769e1177c0773c43a593dccc4ad15f15f1394` and quote token `0xed7993a74dae1e6656b704a424dcd8711aaf4f72`.

## Environment
`@somnia-chain/markets-sdk` 0.32.1, Hideki testnet (chain 50383), indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, read-only.
