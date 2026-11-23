---
title: "Data and model"
date: 2026-11-20
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
#### Overview

This part builds two logic blocks that run the same way locally and inside Lambda:

1. [Price ingestion](5.3.1-ingest-prices/): download prices by trading calendar, clean, flag and write a snapshot to S3.
2. [Quantile forecasting baseline](5.3.2-baseline-forecast/): compute the log-return target, the 252-session baseline, quantile normalization and the probability of a rise, then evaluate with pinball loss.

#### Folder layout

```text
src/
├── data/        # ingest_prices.py (CLI + run()), lambda_ingest.py, calendars.py, clean.py
├── features/    # build.py: features shared by training and prediction
├── forecast/    # baselines.py, returns.py, predict.py, lambda_predict.py, forecast_record.py
├── eval/        # metrics.py, walk_forward.py, rolling_job.py, run_test2020.py
└── chatbot/     # chat.py, tools.py, citations.py, quota.py, router.py
infra/
├── app.py       # CDK entry point, -c parameters and their defaults
├── stacks/      # 5 stacks: data, serving, auth, api, ops
└── lambda/      # handlers of forecasts-api, accuracy-api, chat-api
docker/          # Dockerfiles of the ingest and predict images
web/             # React + TypeScript + Vite
tests/
```

#### Data split

| Period | Used for |
|---|---|
| 2012–2013 | Warm-up, so the baseline has 252 sessions of history |
| 2014–2018 | Training |
| 2019 | Validation: choosing models and parameters |
| 2020 → today | Test: **run only once**, after every parameter is locked |

#### Leakage rules

- At forecast time t, only use information **published before t**. Data for session D is treated as available from 12:00 UTC on day D+1 (`PUBLISHED_AT_RULES`). The Lambda runs at 12:15 UTC, so production uses the same information cut-off as the backtest.
- Training and prediction call **the same function** to build features (`src/features/build.py`).
- An h-session return starting at s is only known at session s + h, so the baseline at t only uses returns that have **already ended**.
- The 2020 → today test set was not looked at before parameters were locked. The team ran it exactly once, on 16/11/2026.
- CI runs `tests/test_no_leakage.py` on every PR: appending data after t must not change the features at t, and a CEEMDAN decomposition over the whole series must be detected as leakage.
