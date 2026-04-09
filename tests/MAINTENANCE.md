# Test Suite Maintenance Guide

The test suite is intentionally pragmatic. Its job is to be a **safety net, not a straitjacket**:
catch crashes, broken invariants, and pipeline regressions without freezing internal implementation details.

---

## File Layout

| File | Purpose |
|---|---|
| `conftest.py` | Shared fixtures (price series, DataFrames, articles, signal dicts, mocks) |
| `test_core.py` | Core sanity/invariant tests for pure functions |
| `test_pipeline.py` | Smoke tests for end-to-end orchestration |
| `test_logic_coverage.py` | Additional edge coverage for scoring, sentiment, dedup, contradictions |
| `test_storage_and_scan.py` | Storage retention/cleanup, round-trip, dry-run scan, synthetic backtest |

The suite is no longer fixed to a tiny count. Add tests when they improve confidence on important logic.

---

## What each test file covers

### `test_core.py` and `test_logic_coverage.py`
Pure logic tests. These should verify either:
- The function runs without crashing, or
- A hard invariant/range, or
- A stable business rule (for example clamping, dedup threshold behavior, or contradiction detection).

Prefer robust assertions over fragile full-structure snapshots.

### `test_pipeline.py` and `test_storage_and_scan.py`
Integration/smoke coverage for:
- `analyse_asset()` and `run_full_scan()` with network mocked,
- `run_scan(dry_run=True)` execution,
- storage read/write/retention behaviors,
- synthetic backtest evaluation.

These tests should avoid enforcing unstable presentation details, but they should assert
meaningful pipeline outcomes (no crash, sane outputs, expected side effects).

---

## Import paths

All test files use the new package-based imports:

| Import | Module |
|---|---|
| `from app.analysis import X` | Re-export shim in `app/analysis.py` |
| `from storage.storage import X` | Storage module in `storage/storage.py` |
| `from src.price import X` | Price logic in `src/price.py` |
| `from src.signals import X` | Signal logic in `src/signals.py` |
| `from src.sentiment import X` | Sentiment logic in `src/sentiment.py` |
| `from src.news import X` | News logic in `src/news.py` |
| `from src.engine import X` | Engine orchestration in `src/engine.py` |

`conftest.py` imports `storage.storage as storage` so that `monkeypatch.setattr` targets the correct module object.

Network calls are mocked at the point of use in `src/engine.py`:
- `"src.engine.fetch_price_history"`
- `"src.engine.fetch_news_articles"`
- `"src.engine.analyse_market_context"`

---

## When to update tests

| Change | Action required |
|---|---|
| New key added to `analyse_asset()` result | Usually nothing; avoid strict key snapshots |
| `analyse_asset()` top-level contract changed | Update smoke expectations in `test_pipeline.py` |
| RSI/ROC formula replaced | Re-check range/sign invariants and edge-case behavior |
| Signal score clamping removed | Clamping tests should fail intentionally |
| Dedup threshold changed | Review boundary tests in `test_logic_coverage.py` |
| Storage retention windows changed | Review age-based tests in `test_storage_and_scan.py` |
| Functions moved between `src/` modules | Update patch targets and imports accordingly |

---

## Adding a new test

Before adding a test, ask:
- **Does it protect against a real regression risk?** Good.
- **Does it verify an invariant, edge case, or side effect that matters?** Good.
- **Is it brittle to harmless refactors?** If yes, simplify it.

---

## Running the tests

```bash
# Standard run
pytest

# Single file
pytest tests/test_core.py -v

# With full traceback
pytest --tb=long
```
