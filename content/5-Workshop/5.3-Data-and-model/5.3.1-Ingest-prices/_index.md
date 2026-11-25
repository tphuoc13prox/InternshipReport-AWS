---
title: "Price ingestion"
date: 2026-11-20
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---
#### Data sources (ADR-0005)

Each series comes from **a single source** for both history and daily updates, so nothing shifts where two sources would be joined.

| Asset | Target series | Trading calendar | Cross-check |
|---|---|---|---|
| Gold | GLD (ETF holding physical gold) | NYSE | NBP gold price level |
| Silver | SLV (ETF holding physical silver) | NYSE | — |
| Brent crude | BZ=F | CME | EIA API v2 (RBRTE) |
| WTI crude | CL=F | CME | EIA API v2 (RWTC) |
| USD/VND | VND=X | — | Display conversion only, never a feature |

All five series are downloaded with the `yfinance` library. The yfinance README states that the Yahoo Finance API is intended for personal use, and the team has no licence from Yahoo. The team treats this as a known risk and narrows its use:

- the web app always shows the data source and opens only after sign-in;
- the web app has no price download feature;
- gold/silver price display can be switched off with the build flag `VITE_SHOW_METAL_PRICES=false`.

#### Running locally

```bash
# load history from 2012 into data/lake (already in .gitignore)
python -m src.data.ingest_prices --start 2012-01-01 --dest local

# routine run: without --start it only reloads the last 5 sessions
python -m src.data.ingest_prices --dest local
```

The ingest Lambda (`src/data/lambda_ingest.py`) calls the same `run()` function with an S3 destination and accepts three kinds of event:

| Event | Behaviour |
|---|---|
| `{}` | Routine run: reload the last 5 sessions, write `raw/` and `clean/` |
| `{"start": "2012-01-01"}` | First history load while the bucket is empty |
| `{"dry_run": true}` | Download the last 10 sessions of the 5 main symbols, **write nothing**, return row counts and errors per symbol. Used to check whether Yahoo blocks AWS IP addresses |

#### Cleaning rules

- `pandas_market_calendars` defines valid sessions: the NYSE calendar for GLD and SLV, the CME calendar for oil.
- **Drop today's bar** in New York time, because that session has not closed yet.
- Every run **reloads the last 5 sessions** and overwrites by key, which fixes data the provider revises late. Re-running on the same day gives the same result.
- The clean table is in long format and every row has a `flag` column. Rejected rows go to `rejected.parquet` with a reason instead of being silently dropped.
- **The target is never forward-filled.** Prices ≤ 0 are kept, but the return is NaN. Example: WTI had a negative price on 20/4/2020.
- **Known gap:** Yahoo is missing the 5/12/2018 session for BZ=F and CL=F. NYSE was closed that day for a national day of mourning, but CME energy kept trading. The team does not fill this session, so windows containing that day are flagged `invalid_input_window`.

#### Layout on S3

```text
s3://cpaws-dev-lake/
├── raw/yfinance/dt=<date>/{GLD,SLV,CL_F,BZ_F,VND_X,...}.csv.gz
├── raw/eia/dt=<date>/ , raw/nbp/dt=<date>/
├── clean/prices/dt=<date>/{prices.parquet, rejected.parquet, quality.json}
├── clean/reference_prices/ , clean/display_fx/ , clean/display_factors/
├── models/        # experimental model artifacts
└── eval/          # validation, test and shadow results (append-only, never overwritten)
```

Each daily run creates a new `dt=` partition. Predict reads the latest partition. The bucket has versioning, so the exact snapshot used in an evaluation can be rebuilt.

#### Checks

```bash
pytest -q tests/test_data_ingest.py tests/test_lambda_ingest.py tests/test_data_clean_quality.py
```

When a main source fails (for example Yahoo returns 429 for one symbol), the other assets are still written. The failed asset keeps its old data with flag `stale_source`. The Lambda logs `INGEST_SOURCE_FAILED` and then raises an error so the alarm sends an email. The next successful run backfills the gap automatically.

![S3 clean/prices](/images/5-Workshop/5.3.1-s3-clean-prices.png)

*Folder `clean/prices/` in bucket `cpaws-dev-lake`: each ingest run creates one `dt=<date>` partition; the first one is `dt=2026-10-15`.*
