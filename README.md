# AI Capex Boom Tracker

A self-contained, single-file web app that tracks the **AI infrastructure capex boom** — 59 stocks across 12 segments of the AI compute supply chain, with live quotes, per-stock trendlines, and a market-cap-weighted index comparing cap-weighted vs equal-weighted performance.

No build step. No dependencies. No backend. One HTML file, your browser, and a free API key.

## Quick start

1. Download `index.html` and open it in any modern browser (double-click is enough).
2. Create a free account at [finnhub.io](https://finnhub.io) and copy your **API key** from the dashboard front page.
3. Paste the key into the app and hit **Save & connect**.

The key is stored only in your browser's `localStorage`. There is no server — every request goes straight from your machine to Finnhub.

## The watchlist

| Group | Tickers |
| --- | --- |
| AI Chips & Compute | NVDA, AMD, AVGO, MRVL, ARM, QCOM, INTC, AMBA |
| Hyperscalers & Cloud | MSFT, GOOGL, AMZN, META, ORCL |
| Neoclouds | CRWV, NBIS, IREN, APLD, CORZ |
| Memory & Storage | MU, WDC, STX, SNDK, PSTG |
| Foundry & Fab Equipment | TSM, ASML, AMAT, LRCX, KLAC, TER |
| Networking | ANET, CRDO, ALAB, CSCO, JNPR |
| Servers & Hardware | SMCI, DELL, HPE, HPQ, CLS |
| Data Center REITs | EQIX, DLR, IRM |
| Cooling & Thermal | MOD, JCI, TT, CARR |
| Optical & Photonics | COHR, GLW, LITE, CIEN |
| AI Software & Data | PLTR, SNOW, DDOG, NET, SNPS, CDNS |
| Robotics & Industrial | IRBT, ROK, SYM |

## Features

- **Live quote cards** — price, day change (absolute + %), per stock, grouped in tabs.
- **Progressive trendlines** — a `<canvas>` sparkline per stock that adds a point on every refresh and grows over time. Flat lines while the market is closed — the app never invents prices.
- **AI Capex Index** — a market-cap-weighted basket of all 59 names, plotted against an equal-weighted version of the same basket. The gap between the two lines shows whether the boom is concentrated in the giants or broadening to smaller names.
- **Rate-limit-aware fetching** — quotes are pulled in paced batches with automatic retries, designed to stay inside Finnhub's free tier (60 calls/min) with a 59-symbol watchlist.
- **Honest failure states** — invalid keys, rate limits, network blocks, and delisted symbols are each reported explicitly. No placeholder data ever shares the screen with real prices.

## How it works

**Data.** Quotes come from Finnhub's `/quote` endpoint; market-cap weights (for the index) from `/stock/profile2`. The free tier has no historical price data, so instead of fake seed data the app **accumulates prices on every refresh** (one point per symbol per cycle, capped at 240 points ≈ 12 hours of runtime) and stores them in `localStorage`. Your history grows the more you use it, and it persists between sessions.

**Index methodology.** Each stock is normalized to the first price the app ever saw for it, then blended:

```
index_level = Σ  weightᵢ × (priceᵢ / base_priceᵢ)
```

Cap weights come from current market capitalization (refreshed weekly); the index starts at 1.0, so the level reads directly as "the basket is up X% since tracking began." The equal-weight line applies the same formula with `weightᵢ = 1/n`.

**Rate limiting.** Finnhub's free tier allows ~60 API calls per minute. A full watchlist refresh fires 59 quote calls, so the app fetches in batches of 12 with 21-second pauses, paints cards progressively as each batch lands, and retries failed symbols once before giving up. Refresh cadence is 3 minutes — deliberate, to keep the average call rate low. This is a small, real example of living inside an API quota.

**Storage layout** (all in `localStorage`):

| Key | Contents |
| --- | --- |
| `finnhub_key` | your API key |
| `capex_hist_v1` | per-symbol price history |
| `capex_bases_v2` | per-symbol baseline price for the index |
| `capex_mcaps_v2` | cached market caps (weekly) |
| `capex_index_v2` | index history (cap-weighted + equal-weighted) |

Clearing your browser data resets the history; prices recover instantly, accumulated trends do not.

## Project layout

```
index.html   — the entire app (HTML + CSS + JS, ~600 lines)
README.md
LICENSE
```

## Notes

- US market hours are 15:30–22:00 CET; quotes only move while the market is open.
- The web app is built to run as a local file. If you embed it on a website, keep the same CORS considerations in mind (Finnhub allows cross-origin requests).
- Ticker list lives in the `WATCHLIST` constant at the top of the script — add or remove names there; everything else adapts automatically.

## Roadmap ideas

- Group-level mini-indices (chips vs clouds vs software rotation)
- Week-in-review leaderboard (best/worst performers since tracking began)
- Price alerts via browser notifications
- CSV export of accumulated history

## Disclaimer

This is a personal tracking tool, not investment advice. No warranty, express or implied. Data comes from third-party APIs and may be delayed or wrong; verify anything that matters from a proper source.

## License

MIT — see [LICENSE](LICENSE).
