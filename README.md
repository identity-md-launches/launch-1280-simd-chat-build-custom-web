# Pulse — personal crypto trading intelligence

Pulse is a responsive research and paper-trading workspace based on [Jev-X Sentiment Analysis](https://github.com/brainstormity/Jev-X-Sentiment-Analysis). The finished static website is in **`dist/`**, alongside the complete React/TypeScript source and npm lockfile. It runs immediately with clearly labeled, deterministic synthetic data. An optional Python service connects live sources; the website does not require that service to explore its tools.

![Pulse market overview](artifacts/overview-desktop.png)

## Install, develop, preview

Use Node.js 22 or later and npm. From this directory:

```sh
npm ci
npm run dev
```

Open the local URL Vite prints, normally `http://localhost:5173`. To preview the **delivered production export**:

```sh
npm run preview
```

Open the printed URL, normally `http://localhost:4173`. You can also preview without installing any npm packages:

```sh
python3 -m http.server 4173 --directory dist
```

Use an HTTP server; opening the HTML directly as a `file:` URL is not supported by browser module loading.

## Rebuild and publish

```sh
npm run typecheck
npm test
npm run build
npm run check:budget
```

Publish **the contents of `dist/`**, including `assets/`, `favicon.svg`, and `og-image.png`, to any static web host. Keep these files together. Vite uses `base: './'`; scripts, styles, the local font, favicon, and share image have relative URLs. Navigation uses hashes (`#signals`, `#backtest`), so a gateway subpath or ENS-hosted static site needs no server-side route rewrites. The publisher serves this export without rebuilding; regenerate and include `dist/` after any source change.

The default app makes no external runtime requests. Fonts, icons, and data needed for the demo are local. Live mode requires access to the optional service and its upstream providers. For link-preview services that require an absolute Open Graph image URL, set `og:image` in `index.html` to the final public image URL after choosing a deployment origin, then rebuild. The included relative image works as a local asset and avoids guessing your hostname.

Keep dependencies and caches out of the submission. `PATH_BUDGET.md` defines the explicit ignore-path budget; `npm run check:budget` inventories deliverable files and produces a compressed check archive in `/tmp`, never in the repository. There is no dependency archive, vendored registry, or Git submodule.

## Use the workspace

| View | Primary workflow |
| --- | --- |
| **Overview** | Choose BTC, ETH, SOL, or BNB. Select **Analyze market** to review the current rules-based decision. Switch chart intervals, toggle indicators, and inspect candles with the arrows. |
| **Sentiment** | Inspect the weighted score and individual social, news, and on-chain sources. Filter by source or search the activity feed. |
| **Trading signals** | Save buy/sell thresholds, an RSI ceiling, source weights, and risk defaults. Zero disables a source. Review and record a paper order; export the local journal. |
| **Strategy lab** | Set thresholds, capital, fees, slippage, stops, and targets. Run a historical backtest, inspect the equity curve and trade ledger, or import/export CSV. |
| **Risk calculator** | Choose long or short, enter capital and price levels, and calculate risk-based position size. Optionally apply a 2 × ATR stop suggestion. |
| **Connections** | Connect your data service, pause/resume refresh, or return to demo data. Download Kraken, Binance, or Coinbase configuration templates for manual or automated integration. |

The **Help & guide** view explains the workflow inside the app. `/` opens asset search when focus is outside an input. All visible times use UTC. Indicators and numeric tables have text equivalents. There are no mobile, desktop, email, or push notifications.

Saved strategies, paper orders (up to 500), and the service URL use local browser storage. No provider or exchange secrets are requested or stored by the browser. The app always starts in demo mode; selecting **Connect service** explicitly enables live polling. If local storage is unavailable, the tools still work for the current session and report that persistence failed. Strategy lab keeps each asset's dataset, provenance, range, parameters, and results while you navigate or switch assets. **Load saved signal rules** applies the saved thresholds and risk rules without replacing historical data or changing capital, fees, or slippage; changed parameters clear outdated results. This research state is held in memory until a reload or tab close. Export data and trades before leaving the session to retain them.

Paper trades are a decision journal, not an exchange simulator or a maintained portfolio. They record a synthetic fill at the displayed price and do not reserve balances or monitor subsequent exits.

## Live sources and the Jev-X foundation

The companion service retains Jev-X's **Python / FastAPI / CCXT / SQLite / optional TypeSafe SDK** foundation. The original plain-JavaScript/Jinja terminal has been replaced with a Vite/React/TypeScript static client to meet the static-hosting requirement. Social preprocessing retains keyword polarity, author diversity, a high-engagement/recent stratified sample, and database deduplication. The deterministic default is explicitly identified; the app never invents a confidence probability.

| Input | Live implementation | Configuration |
| --- | --- | --- |
| Prices and OHLCV | Kraken public spot data through CCXT; closed 5-minute candles | No key required. Exchange listing and regional availability apply. |
| Social | X posts through TwitterAPI.io; 50–1,000 requested posts; 24-hour lookback | Server-side `TWITTER_API_KEY`; optional `PULSE_SOCIAL_SAMPLE` |
| News | CoinDesk and Cointelegraph RSS headlines; per-asset keyword polarity | No key required |
| On-chain | Provider-neutral HTTP adapter with score, observation count, timestamp, and methodology | Owner-set `PULSE_ONCHAIN_URL`; optional `PULSE_ONCHAIN_TOKEN` |
| Jev model | Optional TypeSafe System One social-score evaluation on the stratified sample | Server-side `TYPESAFE_API_KEY` |

Complete service setup, endpoint contracts, source freshness, and the on-chain JSON schema are documented in [service/README.md](service/README.md). Exchange configuration and the bounded dry-run runner are documented in [templates/README.md](templates/README.md).

Live data is polled every 30 seconds while the tab is visible and refresh is enabled. It is periodic retrieval, not an exchange tick stream. Unavailable providers never silently become synthetic live data. Missing enabled sources or scores older than 15 minutes block a signal. Live snapshots older than 2 minutes also block signals, including while refresh is paused. Source weights apply to the current signal; historical CSV scores are already aggregated.

## Calculation details and limits

- **Sentiment:** normalized weighted mean, 0–100. Default weights are social 45, news 35, on-chain 20. A score is directional context, not a probability of profit. Keyword scoring does not reliably understand negation, irony, coordinated posts, or source credibility. On-chain direction depends on the provider's disclosed methodology.
- **Charts:** candlesticks and volume at 5m, 15m, 1h, and 4h; session VWAP resets at 00:00 UTC; RSI uses Wilder's 14-period smoothing; MACD uses 12/26 EMAs and a 9-period signal. Chart indicators recalculate at the selected interval. Signal rules use the underlying 5-minute candles. Charts below 480px show 32 bars and three time ticks; wider charts show 64 bars and six ticks. Visible candle inspection retains two decimal places.
- **Position sizing:** the smaller of `capital × risk% / abs(entry − stop)` and `capital / entry`. No leverage is assumed. Planned loss excludes fees, slippage, funding, and liquidation; a stop cannot guarantee its fill price. ATR guidance uses the mean true range of the last 14 complete 5-minute bars, multiplied by two.
- **Backtesting:** one long position at a time; previous closed-bar signal, next-open fill; adverse slippage and fees on both sides; stop-first when both stop and target touch the same candle; gap exits at the open; final positions close at the final close. Position size is limited by cash including entry fees. Drawdown uses bar-close liquidation equity. Buy-and-hold comparison is before costs. No shorting, funding, latency, liquidity, partial fills, or intrabar tick simulation is modeled.
- **Historical data:** the bundled seven-day series is synthetic, not an empirical performance claim. Import up to 25,000 chronological candles / 5 MiB using `time,open,high,low,close,volume,sentiment`. Unix seconds denote candle opening time; each score must have been available by that candle's close. Empty sentiment cells do not trigger entries/exits. OHLC consistency, ordering, finite values, and score bounds are validated.

This software is for personal research and paper trading. Live execution is intentionally outside the static site's scope: templates contain no real keys, and the included runner refuses to submit real orders. Backtests and heuristic scores are not financial advice or evidence of future returns.

## Validation actually performed

See [artifacts/validation.md](artifacts/validation.md) for the six-domain Better Interface review, findings, fixes, evidence, and limitations. Machine-readable browser results and measurements are in [artifacts/browser-results.json](artifacts/browser-results.json).

Worker checks during the revision on 2026-10-11:

- `npm run typecheck` — passed.
- `npm test` — 16 numerical, strategy, import, and transport checks passed.
- `npm run build` — passed; production output generated in `dist/` with relative asset URLs.
- `npm run check:browser` — Chromium interaction checks, 14 axe audits, and responsive checks passed. The worker used an explicit locally installed Chromium path through `PULSE_CHROMIUM`.
- `node tests/revision-browser.mjs` — six regression groups passed: all four intervals at seven widths, visible OHLC precision, imported/recorded history retention, separate asset state, rule loading, and shared typography/copy. See [before measurements](artifacts/revision-before.json), [after measurements](artifacts/revision-results.json), and the five finding responses in `.imd-responses.json`.
- `npm run check:budget` — final measured inventory is in [artifacts/budget.json](artifacts/budget.json).

To respect this revision's protected dependency paths, the worker installed the unchanged lockfile and ran frontend commands in a matching `test/scratch/workspace/` copy. Its built `dist/` was copied into the root and byte-compared; the browser tool also inspected the root export. Exact commands are in the validation record. The previous accepted round reported 13 Python tests and a public price/news smoke check; those were not rerun in this frontend-only revision. No paid, owner-specific, or trading integration was exercised.

To repeat browser checks locally:

```sh
npx playwright install chromium
npm run build
npm run check:browser
node tests/revision-browser.mjs
```

On a machine with an existing Chromium installation, set `PULSE_CHROMIUM` to its executable instead. The test script owns and closes its HTTP preview and browser in one foreground process. It serves the real `dist/` under `/preview/`, tests forms, filters, downloads, chart controls, paper orders, storage, live-connection recovery, keyboard focus, and reflow. Real provider availability is not implied by fixture-based transport tests. Screen-reader, physical-device, Safari/Firefox, and native browser zoom checks remain unperformed. 200% CSS text enlargement was checked separately.

## Repository map

```text
src/                 React interface, chart components, calculation engine
public/              Favicon and social preview image
dist/                Complete static production export, ready to publish
service/             Optional read-only Python aggregation service and lockfile
templates/           Exchange integration instructions and dry-run runner
tests/               Numerical, service and browser interaction validation
scripts/             Preview harness, social image generation and bundle budget
artifacts/           Screenshots, actual validation results and final inventory
DESIGN.md            Implemented design system and responsive behavior
THIRD_PARTY_NOTICES.md Attribution and license references
```

The design follows the pinned Better Interface guide, adapted from Jakub Krehel's MIT-licensed work, with the documentation method adapted from Paul Bakaus's Apache-2.0-licensed Impeccable. Full notices are retained in `licenses/`.
