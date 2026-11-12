# The "Scale quantity by the base asset's decimals" table omits 15 live markets

Severity: normal
Area: docs

## Steps to reproduce
1. Open https://app.testnet.dreamdex.io/docs/developers/perps-testnet, section "Raw and operator calls", subsection "Scale quantity by the base asset's decimals".
2. Compare its table with `(await markets.client.listPerpMarkets()).map(m => [m.baseSymbol, m.baseDecimals])` on Hideki testnet.

## Expected
The table lists every perps market, or says it is a partial example.

## Actual
The table lists 26 symbols. These live markets are missing (all `baseDecimals: 18` on 2026-11-11): **WTI, XAG, XAU, GOOGL, TSLA, HYNX, MSTR, MRVL, LITE, INTC, HOOD, COIN, AMD, ADLT**, plus the internal **LOADTEST** markets.

The page says "Read them rather than copying this table", but a reader who uses the table as a lookup (the page itself shows that a wrong scale fails silently, with `InsufficientCollateral()`) has nothing for 14 tradeable markets. The fix is to add them, or to label the table "examples" and point to `baseDecimals` on the market row as the only source.

## Evidence
`listPerpMarkets()` output (symbol: baseDecimals) for the missing ones: WTI 18, XAG 18, XAU 18, GOOGL 18, TSLA 18, HYNX 18, MSTR 18, MRVL 18, LITE 18, INTC 18, HOOD 18, COIN 18, AMD 18, ADLT 18.

## Environment
Docs page as of 2026-11-11; `@somnia-chain/markets-sdk` 0.32.1, Hideki testnet (chain 50383).
