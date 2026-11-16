# AI Capex Boom Tracker

A self-contained, single-file web app that tracks the **AI infrastructure capex boom** — 58 stocks across 12 segments of the AI compute supply chain, with live quotes, real 1-month trendlines, and a market-cap-weighted index comparing cap-weighted vs equal-weighted performance.

No build step. No dependencies. No backend. No API key. One HTML file and your browser. (The only external asset is the optional pixel-font stylesheet used by the retro theme; the app works fine without it.)

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
- **Three timeframes — 1M / 1W / 1D** — one click switches the whole board: monthly daily closes, weekly 15-minute bars, or intraday 5-minute bars. The index recomputes per timeframe. Day change always means "vs yesterday's close" on every view — never vs the last bar, which on intraday views would read ~0%. The 1D chart anchors its line at yesterday's close, so overnight gaps (earnings pops, gap-ups) show up in the chart instead of contradicting the day change.
- **Chart hover** — move the cursor over any chart to read exact values: the index chart shows a crosshair with the date/time and the % of both index lines, and each stock sparkline shows price + time in its caption.
- **News ticker + 📰 News tab** — the 20 newest AI-market headlines scroll continuously across a bar pinned to the top of the page, with a colored dot per source. Hovering pauses the scroll, clicking a headline opens the story, and the feed refreshes itself every 30 minutes. The News tab shows the full list with per-source filter chips. Sources: TechCrunch AI, Ars Technica AI, MIT Tech Review, ZDNET AI and MarkTechPost via RSS (the parser handles both RSS and Atom formats).
- **Retro CRT layout** — a `RETRO` toggle in the header switches the whole app into an 80s/90s green-phosphor terminal: Press Start 2P headers, VT323 body text, chunky square borders, scanlines and a soft CRT glow. The charts re-skin to the phosphor palette with thicker lines, but render at full resolution so values and tooltips stay crisp. Your choice is remembered; the pixel fonts load from Google Fonts and fall back to monospace offline.
- **Real 1-month trendlines** — a `<canvas>` sparkline per stock drawn from actual daily closes on Yahoo Finance. No waiting for history to accumulate — you get a month of real data on first load.
- **AI Capex Index** — a market-cap-weighted basket of all 58 names, plotted against an equal-weighted version of the same basket. The gap between the two lines shows whether the boom is concentrated in the giants or broadening to smaller names.
- **No key, no quota** — quotes come from Yahoo's unofficial `spark` endpoint, 15 symbols per request, so a full refresh is 4 calls instead of 58.
- **Resilient fetching** — requests go through a chain of CORS proxies (allorigins → codetabs → r.jina.ai); a proxy that fails goes into a 10-minute cooldown and the chain self-heals. Failed symbols get an automatic retry pass.
- **Honest failure states** — network blocks, dead proxies, and symbols without data are each reported explicitly. No placeholder data ever shares the screen with real prices.

## How it works

**Data.** Quotes and history come from Yahoo Finance's unofficial endpoints:

- `/v8/finance/spark?symbols=…&range={1mo|5d|1d}&interval={1d|15m|5m}` — batched, 15 symbols per call, returns the timeframe's closes plus the live price and previous close.
- `/v8/finance/chart/{sym}?range=…&interval=…` — single-symbol fallback for stragglers.

Browsers can't call Yahoo directly (it sends no CORS header), so every request is routed through a public proxy. If one proxy is down, the next in the chain takes over.

**Index methodology.** Each stock is normalized to its first close of the selected window, then blended:

```
index_level = Σ  weightᵢ × (priceᵢ / first_closeᵢ)
```

Both lines start at 1.0, so the level reads directly as "the basket is up X% over this window." Cap weights are fixed market caps baked into the file (as of 14 Nov 2026); the equal-weight line applies `weightᵢ = 1/n`. Stocks are forward-filled onto the union of trading dates, so US holidays don't dent the line; stocks that haven't reported a bar yet in the window are held at their base price rather than excluded, so the basket is complete from the first point.

**Storage.** Per-timeframe caches (`capex_cache_v1_1M` / `_1W` / `_1D`) hold the latest quotes snapshot so switching views paints instantly and fetches fresh data in the background; `capex_tf` remembers your preferred timeframe between sessions. A generation counter guards refreshes: if you switch timeframes mid-refresh, the stale fetch discards itself instead of writing old data onto the new view.

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

- **Nov 2026** — News ticker + News tab: 5 AI-market RSS feeds (TechCrunch, Ars Technica, MIT Tech Review, ZDNET, MarkTechPost) with a scrolling ticker pinned to the top; proxy chain now tracks dead proxies separately for quotes vs news.
- **Nov 2026** — Retro CRT theme toggle: green-phosphor layout with Press Start 2P + VT323 fonts, charts re-skinned via CSS-variable palette at full resolution; preference persisted.
- **Nov 2026** — Chart hover: crosshair with dates and values on the index chart, price + time on stock sparklines.
- **Nov 2026** — 1D charts now anchor at yesterday's close, so overnight gaps (a stock that gaps up +27% then fades no longer shows a falling chart next to a big green day change).
- **Nov 2026** — Added the 1M / 1W / 1D timeframe switcher (daily, 15-min, and 5-min bars), with per-timeframe caching and a generation guard for mid-refresh switches.
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
