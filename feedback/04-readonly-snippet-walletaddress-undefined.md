# Docs snippets mix read-only setup with `markets.walletAddress!`, which is undefined without a key

Severity: normal
Area: docs

## Steps to reproduce
1. Follow "Connect" on the developer page and leave out `privateKey`, which the snippet's own comment allows ("leave out for read-only").
2. Run the next snippet, "Get test funds":
   `const account = markets.walletAddress!`
   `await markets.client.getPerpVoucherEligibility(btc.marginBank, account)`

## Expected
Either the snippets state that a key is needed from this point on, or they show how to pass an explicit address for read-only checks.

## Actual
`markets.walletAddress` is `undefined` in read-only mode. The TypeScript non-null assertion (`!`) hides it at compile time, so `account` is `undefined` at runtime and is passed into `getPerpVoucherEligibility` and then `getMaxPerpOrderSize({ ..., account })` in the following snippet. Passing an explicit address works (e.g. `getPerpVoucherEligibility(marginBank, "0x...dEaD")` returns `{ eligible: true, grantAmount: 1000e18 }`), but the docs never say so.

## Evidence
In a read-only session, `String(markets.walletAddress)` was `"undefined"`. With an explicit address, eligibility and sizing returned normally.

## Environment
`@somnia-chain/markets-sdk` 0.32.1, Hideki testnet (chain 50383), read-only, 2026-11-11.
