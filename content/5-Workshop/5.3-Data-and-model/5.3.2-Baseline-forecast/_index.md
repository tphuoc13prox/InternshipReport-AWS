---
title: "Quantile forecasting baseline"
date: 2026-11-20
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---
#### Target: h-session log return

CPAWS does not forecast price levels. The target is the log return from session t to session t + h, on each asset's own trading calendar (h = 1–7):

```python
import numpy as np
import pandas as pd

QUANTILE_LEVELS = (0.05, 0.10, 0.25, 0.50, 0.75, 0.90, 0.95)

def log_return(close: pd.Series, h: int) -> pd.Series:
    # close is aligned to the trading calendar; NaN when a price is missing or <= 0
    logp = np.log(close.where(close > 0))
    return logp.shift(-h) - logp
```

#### Causal 252-session quantile baseline

An h-session return starting at session s is only known at session s + h. So at time t the baseline only uses returns r_h(s) with s in the window [t − 251, t − h]. Simplified from `src/forecast/baselines.py`:

```python
WINDOW, MIN_OBS = 252, 200

def quantile_baseline(r_h: pd.Series, h: int) -> pd.DataFrame:
    # shift by h: the value at t is r_h(t - h), i.e. a return that has already ended
    shifted = r_h.astype(float).shift(h)
    roll = shifted.rolling(WINDOW - h, min_periods=MIN_OBS)
    # windows with fewer than MIN_OBS valid returns give a NaN row
    return pd.DataFrame({q: roll.quantile(q, interpolation="linear") for q in QUANTILE_LEVELS})
```

Inside the predict Lambda this is wrapped as `Quantile252Model` with `model_version = "baseline-q252"`. The output for each asset is a matrix of 7 horizons × 7 quantiles.

#### Quantile normalization and probability of a rise

```python
def normalize_quantiles(q: np.ndarray) -> np.ndarray:
    # call exactly ONCE right after the model output; sort so quantiles never cross
    return np.sort(q, axis=-1)

def prob_up(q: np.ndarray) -> float:
    # P(r > 0) = 1 - F(0), F interpolated linearly between the two quantiles around 0, clipped to [0.05, 0.95]
    lv = np.array(QUANTILE_LEVELS)
    if 0.0 < q[0]:
        return 0.95
    if 0.0 > q[-1]:
        return 0.05
    f0 = np.interp(0.0, q, lv)
    return float(np.clip(1.0 - f0, 0.05, 0.95))
```

The full version in `src/forecast/returns.py` also handles 0 falling exactly on a quantile, and logs the share of crossed quantiles before sorting.

#### Converting to a price range

Price at quantile q: `price_q = last_price × exp(q)`. The web app shows q10, q50 and q90 in USD and VND, assuming the USD/VND rate stays constant over the h sessions. For gold and silver the price is converted from the GLD/SLV funds, so the web app states that this is a proxy series, not the spot price. The 80% range is a forecast interval, not a verified probability: real coverage is measured separately with `coverage_80` on live data.

#### Evaluating with pinball loss

```python
def pinball_loss(y: np.ndarray, q_pred: np.ndarray) -> float:
    # y: (n,) realized returns; q_pred: (n, 7) forecast quantiles
    tau = np.array(QUANTILE_LEVELS)
    diff = y[:, None] - q_pred
    return float(np.mean(np.maximum(tau * diff, (tau - 1) * diff)))

# pinball_skill = 1 - loss_model / loss_baseline  (> 0 means better than the 252-session baseline)
```

Besides the main metric, the team also measures:
- **Calibration:** `coverage_80`, `width_80`, the hit rate of each quantile;
- **Probability of a rise:** `brier_skill`;
- **Secondary metric:** `sign_accuracy`, always reported with `up_rate`.

The team does not use MAPE or RMSE on price levels. To compare two models, it uses the Diebold–Mariano test with HAC standard errors.

**Model switching rule (ADR-0008):** a machine learning model replaces the baseline only when **the lower bound of the 95% confidence interval of `pinball_skill` is above 0**.

#### Results

- **Validation 2019:** DLinear lost to the 252-session baseline in **28/28 groups** (4 assets × 7 horizons). The 252-session baseline became the champion for all 4 assets, and DLinear stayed as a challenger.
- **Test 2020 → today** (run exactly once, on 16/11/2026): candidate S1a takes the 252-session baseline and stretches the range by the ratio of short-term volatility (EWMA, λ = 0.94) to long-term volatility, clipped to [0.5; 2]. S1a still runs in shadow inside the predict Lambda since 04/11 but is not shown to users. Compared with the 252-session baseline:

| Asset | S1a `pinball_skill` | 95% CI | Reading |
|---|---:|---|---|
| Gold | +0.11% | [−1.29%; +1.49%] | No difference |
| Silver | +2.12% | [+0.17%; +4.10%] | Positive but not robust: the CI contains 0 with block size 40 |
| WTI | +4.49% | [+2.02%; +6.73%] | Positive |
| Brent | +4.21% | [+1.88%; +6.35%] | Positive |

S1a's skill was negative for all 4 assets in 2023, and 18/42 DM tests could not be computed. The team decided to **keep `baseline-q252` until the end of the internship**. Switching oil and silver to S1a would require a new ADR; the team lists it as future work.

<!-- ẢNH: trang web một tài sản: 7 thẻ kỳ hạn, nhãn "Mốc thống kê 252 phiên" (CHE mọi con số giá và khoảng giá) -->
