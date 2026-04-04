# PulseEngine — Roadmap

This document outlines where PulseEngine is going and why. It exists so contributors can see the direction, pick a lane, and build something that matters.

The project is split into two surfaces sharing one core engine:

- **Local app** — the full product. Runs entirely on your machine. No cloud, no accounts, no data leaving your device. This is the priority.
- **Web demo** — a restricted live preview hosted on Streamlit Community Cloud. Drives awareness and downloads. Not the end goal.

---

## Current state — v0.2.0

- 24 tracked assets across Commodities, Cryptocurrency, Tech Stocks, and Market Indices
- VADER sentiment engine with injected financial lexicon
- 12 RSS feeds ingested in parallel with Jaccard deduplication
- RSI, momentum, trend strength, and 8-category event detection
- Per-asset-class signal weighting profiles
- Background scan daemon refreshing all assets every 30 minutes
- Compressed snapshot storage with tiered retention (7 / 30 / 60 days)
- Backtesting module with hit-rate evaluation
- Streamlit live demo at [pulseengine.streamlit.app](https://pulseengine.streamlit.app/)
- Docker support, 14 tests, full documentation

**What's missing:** arbitrary ticker support, a local installer, a desktop executable, and a proper financial NLP model.

---

## v0.3 — Foundation split + arbitrary tickers

> **This is the critical path milestone. Everything after it depends on it.**

### Repo restructure

```
pulseengine/
  core/        ← shared engine (app.py, config.py, storage.py, backtest.py)
  local/       ← Streamlit dashboard, full feature set
  web/         ← restricted demo build
  docs/
  tests/
```

### Arbitrary ticker support

Right now every asset has a handcrafted keyword list in `config.py`. Scaling to arbitrary tickers means:

- User can type any valid ticker symbol into the dashboard
- Keywords are auto-generated from company name, ticker symbol, and executive names
- Low-news-volume stocks are handled gracefully with fallback signal behaviour
- Contributors can add stocks without touching `config.py`

### Other v0.3 deliverables

- Local installer script — one command, no friction, no manual dependency wrangling
- Close issue backlog: #10 (last scanned message), #11 (config.py docstrings), #12 (signal score legend)
- This `ROADMAP.md` published and linked from the README

---

## v0.4 — Desktop experience (Local track)

The goal here is "download and double-click." The user should never need to open a terminal.

- **PyInstaller EXE** — Windows first, then macOS and Linux
- **Launcher script** — starts the Streamlit server as a subprocess, waits for it to be ready, opens the browser automatically, shows a system tray icon
- **Clean shutdown** — closing the tray icon stops the server gracefully
- **GitHub Actions build pipeline** — on every release tag, automatically build and attach platform binaries to the GitHub Release
- **First-run setup** — lightweight wizard that configures the data directory and confirms dependencies on first launch

### Web demo (parallel, ongoing)

The web demo is not a separate project — it runs off the same core and stays deliberately limited:

- Streamlit Community Cloud — free forever, Streamlit absorbs the hosting cost
- 24 fixed assets only — no arbitrary ticker lookup
- No backtesting, no historical snapshots, no export
- Fully stateless — nothing is stored server-side, ever
- GitHub Pages landing page: what PulseEngine is, download button, live demo link

> **Privacy headline:** *"We store nothing. Ever."* — the web demo is architecturally incapable of retaining user data.

---

## v0.5 — Local intelligence (Local track)

This is where the local app becomes genuinely independent of any external service.

- **FinBERT** — a proper financial NLP model that runs entirely locally. Downloaded once on first run, cached permanently. Meaningfully better sentiment accuracy than VADER for financial news.
- **Offline mode** — when the network is unavailable, serve cached data and flag signal staleness clearly
- **Export** — CSV and PDF export of signal reports and backtest results
- **Backtesting improvements** — lag correction, rolling validation, signal weight evaluation against historical data
- **Test coverage** — property-based tests for pure functions (RSI, Jaccard, signal scoring), integration coverage for the full pipeline
- **Custom RSS feeds** — users can add their own news sources via config

---

## v1.0 — Full market coverage

This milestone is the vision fully realised. Timeline depends on community growth.

- **Dynamic asset discovery** — the system finds and begins covering stocks automatically, not just ones explicitly configured
- **All stocks** — any exchange, any ticker, any news volume level
- **Auto-generated company profiles** — executive names, subsidiary names, product lines, all derived automatically and used for news correlation
- **News routing at scale** — RSS feeds alone won't cut it at this volume; proper news ingestion pipeline
- **Signal weight auto-tuning** — weights validated against rolling historical data rather than hand-tuned intuition
- **Alert system** — when a signal crosses a configurable threshold, the user gets a local desktop notification
- **Community-maintained sector profiles** — contributors own their domain. A contributor who covers semiconductors maintains the keyword and weighting profiles for that sector.

---

## What's explicitly out of scope

These are things PulseEngine will not become:

- **A trading platform.** No order execution, no brokerage integration, no portfolio management.
- **A SaaS product.** No user accounts, no subscriptions, no cloud lock-in.
- **A paid tool.** Free forever, MIT licensed.
- **A data vendor.** All data sourced from free public feeds. No proprietary data.

---

## How to contribute

The issue tracker is the best place to start. Issues are tagged by difficulty and area:

- `good first issue` — self-contained, no domain knowledge required
- `backend` — core engine, data pipeline, storage
- `frontend` — dashboard UI, Streamlit components
- `docs` — documentation, examples, guides
- `medium` — meaningful scope, some codebase familiarity needed

If you want to work on something not in the issue tracker, open an issue first and describe what you're planning. This avoids duplicate work and makes sure your contribution fits the direction.

Read [CONTRIBUTING.md](../CONTRIBUTING.md) before opening a pull request.

---

## Architecture note for contributors

The `core/` directory is the shared foundation. Changes there affect both the local app and the web demo. Keep it clean, well-tested, and free of surface-specific logic.

The `local/` directory can be heavy — FinBERT, backtesting, snapshot storage, export. No apologies for size or compute requirements here.

The `web/` directory must stay lightweight. No local model inference, no file I/O, no state between requests. If a feature can't run statelessly in a browser session, it belongs in `local/` only.

---

*PulseEngine is MIT licensed. See [LICENSE](../LICENSE) for the full text.*
*This is not financial advice. See [DISCLAIMER.md](DISCLAIMER.md) for the full disclaimer.*
