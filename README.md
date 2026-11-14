# AI Capex Boom Tracker

A self-contained, single-file web app that tracks the **AI infrastructure capex boom** — 58 stocks across 12 segments of the AI compute supply chain, with live quotes, real 1-month trendlines, and a market-cap-weighted index comparing cap-weighted vs equal-weighted performance.

No build step. No dependencies. No backend. No API key. One HTML file and your browser.

## Quick start

1. Download `index.html` and open it in any modern browser (double-click is enough).

That's it. Quotes stream straight from Yahoo Finance through a public CORS proxy — nothing to sign up for, nothing to configure.

## The watchlist

| Group | Tickers |
| --- | --- |
| AI Chips & Compute | NVDA, AMD, AVGO, MRVL, ARM, QCOM, INTC, AMBA |
| Hyperscalers & Cloud | MSFT, GOOGL, AMZN, META, ORCL |
| Neoclouds | CRWV, NBIS, IREN, APLD, CORZ |
| Memory & Storage | MU, WDC, STX, SNDK, NTAP |
| Foundry & Fab Equipment | TSM, ASML, AMAT, LRCX, KLAC, TER |
| Networking | ANET, CRDO, ALAB, CSCO |
| Servers & Hardware | SMCI, DELL, HPE, HPQ, CLS |
| Data Center REITs | EQIX, DLR, IRM |
| Cooling & Thermal | MOD, JCI, TT, CARR |
| Optical & Photonics | COHR, GLW, LITE, CIEN |
| AI Software & Data | PLTR, SNOW, DDOG, NET, SNPS, CDNS |
| Robotics & Industrial | SERV, ROK, SYM |

## Features

- **Live quote cards** — price, day change (absolute + %), per stock, grouped in tabs.
- **Real 1-month trendlines** — a `<canvas>` sparkline per stock drawn from actual daily closes on Yahoo Finance. No waiting for history to accumulate — you get a month of real data on first load.
- **AI Capex Index** — a market-cap-weighted basket of all 58 names, plotted against an equal-weighted version of the same basket. The gap between the two lines shows whether the boom is concentrated in the giants or broadening to smaller names.
- **No key, no quota** — quotes come from Yahoo's unofficial `spark` endpoint, 15 symbols per request, so a full refresh is 4 calls instead of 58.
- **Resilient fetching** — requests go through a chain of CORS proxies (allorigins → codetabs → r.jina.ai); a proxy that fails goes into a 10-minute cooldown and the chain self-heals. Failed symbols get an automatic retry pass.
- **Honest failure states** — network blocks, dead proxies, and symbols without data are each reported explicitly. No placeholder data ever shares the screen with real prices.

## How it works

**Data.** Quotes and history come from Yahoo Finance's unofficial endpoints:

- `/v8/finance/spark?symbols=…&range=1mo&interval=1d` — batched, 15 symbols per call, returns a month of daily closes plus the live price.
- `/v8/finance/chart/{sym}?range=1mo&interval=1d` — single-symbol fallback for stragglers.

Browsers can't call Yahoo directly (it sends no CORS header), so every request is routed through a public proxy. If one proxy is down, the next in the chain takes over.

**Index methodology.** Each stock is normalized to its first close of the 1-month window, then blended:

```
index_level = Σ  weightᵢ × (priceᵢ / first_closeᵢ)
```

Both lines start at 1.0, so the level reads directly as "the basket is up X% over the past month." Cap weights are fixed market caps baked into the file (as of 14 Nov 2026); the equal-weight line applies `weightᵢ = 1/n`. Stocks are forward-filled onto the union of trading dates, so US holidays don't dent the line.

**Storage.** A single `localStorage` key, `capex_cache_v1`, holds the latest quotes snapshot so the app paints instantly on reload and fetches fresh data in the background.

## Project layout

```
index.html   — the entire app (HTML + CSS + JS, ~650 lines)
README.md
LICENSE
```

## Notes

- US market hours are 15:30–22:00 CET; quotes only move while the market is open.
- Yahoo's endpoints are unofficial — they can change without notice. The proxy chain and tolerant JSON parsing absorb most hiccups.
- Ticker list lives in the `WATCHLIST` constant at the top of the script — add or remove names there; everything else adapts automatically.

## Changelog

- **Nov 2026** — Migrated from Finnhub (API key required, US-only, no history on free tier) to Yahoo Finance: no key needed, real 1-month daily trends, 4 batched requests per refresh instead of 58. Dropped JNPR (delisted — HPE acquisition completed), PSTG and IRBT (recycled ticker shells on Yahoo); added NTAP (NetApp) and SERV (Serve Robotics).

## Roadmap ideas

- Group-level mini-indices (chips vs clouds vs software rotation)
- Week-in-review leaderboard (best/worst performers of the month)
- Price alerts via browser notifications
- CSV export of price history

## Disclaimer

This is a personal tracking tool, not investment advice. No warranty, express or implied. Data comes from third-party APIs and may be delayed or wrong; verify anything that matters from a proper source.

## License

MIT — see [LICENSE](LICENSE).
