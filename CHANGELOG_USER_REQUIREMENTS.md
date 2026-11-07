# User-requested project update — 2026-11-07

Base project: `po3-mexc-futures-scanner` from the supplied v11 ZIP.

## Implemented

1. Production target: up to 500 CMC-ranked cryptocurrency derivative assets, then strict verification against MEXC Futures. Only active, tradable, genuine USDT-quoted and USDT-settled perpetual contracts pass. The existing MEXC contract/detail and ticker validation is preserved.
2. Production candle timeframe: 4H only. No 1H or 1D candle fetches are scheduled by the production configuration.
3. Global volume ranking: scheduled refresh every four hours (`40 */4 * * *` UTC). The scanner still runs hourly and reuses the cached ranking between refreshes.
4. Signal freshness: current forming 4H candle plus the previous two closed 4H candles. Older entries are rejected as stale.
5. CMC only: three distinct API keys are required as `CMC_API_KEY_1`, `CMC_API_KEY_2`, and `CMC_API_KEY_3`. The CMC client rotates keys after API, authentication, rate-limit, or quota errors. Duplicate keys are rejected. There is no CoinGecko, CoinGlass, spot, exchange-volume, or MEXC-only ranking fallback.
6. PO3 remains SHORT-only. No BUY/LONG alerts and no order placement.

## Preserved

- Existing PO3 strategy implementation and its configured inputs, entry, SL, TP and state-machine behavior.
- Existing MEXC contract verification and exclusion filters.
- Existing duplicate ledger, Telegram delivery handling, state-branch persistence and hourly scan workflow.

## Important limitations

- CMC's ranking aggregation is partial: it fetches up to the configured top 40 derivatives exchanges and up to 1,000 markets per exchange by default. It should not be described as a complete census of every global futures venue.
- 500 is a target, not a guaranteed count. If fewer than 500 assets pass strict MEXC verification and liquidity/contract filters, the scanner reports and scans fewer; it never pads the list.
- Three distinct API keys are required, but they do not guarantee three times the quota. Each account's actual CMC plan and remaining credits govern usage.
- The scanner may reuse a recent CMC ranking cache if all keys fail. If no acceptable CMC cache exists, it halts. It never switches to MEXC-only volume ranking.
- Live CMC, MEXC, Telegram and GitHub Actions execution has not been performed in this environment. Local tests use mocked HTTP/local servers only.

## Verification

- `python -m compileall -q .` — passed.
- `pytest -q` — 250 passed.
- Includes a mocked CMC key-rotation test (key 1 rejected, key 2 succeeds), strict no-fallback tests, MEXC contract filtering tests, 4H-only production configuration tests, current-plus-previous-two candle-window tests, duplicate suppression and local HTTP end-to-end tests.
