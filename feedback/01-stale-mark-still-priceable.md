# getMaxPerpOrderSize reports priceable: true for a market whose mark price is 64 h stale (ADLT)

Severity: major
Area: SDK

## Steps to reproduce
1. Connect read-only with `@somnia-chain/markets-sdk@0.32.1` to Hideki testnet, as in "Perps on Testnet for Developers":
   `new SomniaMarkets({ chain: hidekiTestnet, indexerUrl: "https://hideki-dev.smk.somnia.host/v1/graphql", addresses: SOMNIA_HIDEKI_TESTNET_ADDRESSES })`
2. `const m = (await markets.client.listPerpMarkets()).find(x => x.baseSymbol === "ADLT")`
3. Note `m.markPriceUpdatedAt`. On 2026-11-11 it was about **63.8 hours** in the past (pool `0xe3f6dcda98595764970e5a6ffd167af1d1b35123`).
4. Call `markets.client.getMaxPerpOrderSize({ pool: m.poolAddress, marginBank: m.marginBank, account: "0x000000000000000000000000000000000000dEaD", isBid: true, price: m.markPrice })`, with the price rounded to `tickSize`.

## Expected
The developer docs say: "It answers `priceable: false` when the market's mark price is stale, and then quotes no size at all." A mark price that is 64 h old should be stale, so we expect `priceable: false`, the same as for the LOADTEST markets with no mark.

## Actual
`priceable: true` and `markPrice: 3450000000000000000000` (3450), the same value the indexer last updated about 64 h earlier (`asOfBlock: 47985389`). For comparison: ETH (mark updated 0.8 h earlier) is `priceable: true`; LOADTEST (no mark) is `priceable: false`.

So one of these is wrong:
- the staleness check that `priceable` is documented to apply; or
- `markPriceUpdatedAt` from the indexer, if the on-chain mark is in fact fresh.

Either way, a bot that trusts `priceable` would size orders against a mark that is days old.

## Evidence
- `getMaxPerpOrderSize` response (ADLT): `{ priceable: true, placeable: false, limitedBy: "collateral", markPrice: "3450000000000000000000", asOfBlock: "47985389" }`
- `listPerpMarkets` (ADLT): `markPriceUpdatedAt` about 63.8 h before the query; ADLT is the only non-LOADTEST market older than 1 h.

## Environment
`@somnia-chain/markets-sdk` 0.32.1 (browser ESM build), Hideki testnet (chain 50383), indexer `https://hideki-dev.smk.somnia.host/v1/graphql`, read-only (no wallet). Queried 2026-11-11.
