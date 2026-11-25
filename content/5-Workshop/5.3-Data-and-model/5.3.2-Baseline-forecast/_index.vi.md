---
title: "Mốc dự báo phân vị"
date: 2026-11-20
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---
#### Target: log return h phiên

CPAWS không dự báo mức giá. Target là log return từ phiên t tới phiên t + h, tính theo lịch giao dịch của từng tài sản (h = 1–7):

```python
import numpy as np
import pandas as pd

QUANTILE_LEVELS = (0.05, 0.10, 0.25, 0.50, 0.75, 0.90, 0.95)

def log_return(close: pd.Series, h: int) -> pd.Series:
    # close đã căn theo lịch giao dịch; NaN khi thiếu giá hoặc giá <= 0
    logp = np.log(close.where(close > 0))
    return logp.shift(-h) - logp
```

#### Mốc phân vị 252 phiên (nhân quả)

Return h phiên bắt đầu tại phiên s chỉ biết được ở phiên s + h. Vì vậy tại thời điểm t, mốc chỉ dùng các return r_h(s) với s nằm trong cửa sổ [t − 251, t − h]. Rút gọn từ `src/forecast/baselines.py`:

```python
WINDOW, MIN_OBS = 252, 200

def quantile_baseline(r_h: pd.Series, h: int) -> pd.DataFrame:
    # dịch h phiên: giá trị tại t là r_h(t - h), tức return đã kết thúc
    shifted = r_h.astype(float).shift(h)
    roll = shifted.rolling(WINDOW - h, min_periods=MIN_OBS)
    # cửa sổ có ít hơn MIN_OBS return hợp lệ thì cả dòng là NaN
    return pd.DataFrame({q: roll.quantile(q, interpolation="linear") for q in QUANTILE_LEVELS})
```

Trong Lambda predict, hàm này được bọc thành `Quantile252Model` với `model_version = "baseline-q252"`. Đầu ra mỗi tài sản là một ma trận 7 kỳ hạn × 7 phân vị.

#### Chuẩn hóa phân vị và xác suất tăng

```python
def normalize_quantiles(q: np.ndarray) -> np.ndarray:
    # gọi đúng MỘT lần ngay sau khi mô hình xuất kết quả; sắp xếp để các phân vị không cắt nhau
    return np.sort(q, axis=-1)

def prob_up(q: np.ndarray) -> float:
    # P(r > 0) = 1 - F(0), F nội suy tuyến tính giữa hai phân vị kẹp mức 0, kẹp trong [0.05, 0.95]
    lv = np.array(QUANTILE_LEVELS)
    if 0.0 < q[0]:
        return 0.95
    if 0.0 > q[-1]:
        return 0.05
    f0 = np.interp(0.0, q, lv)
    return float(np.clip(1.0 - f0, 0.05, 0.95))
```

Bản đầy đủ trong `src/forecast/returns.py` xử lý thêm trường hợp 0 trùng đúng một phân vị và ghi log tỷ lệ phân vị bị cắt nhau trước khi sắp xếp.

#### Quy đổi ra khoảng giá

Giá tại phân vị q: `price_q = last_price × exp(q)`. Web hiển thị q10, q50, q90 theo USD và VND, giả định tỷ giá USD/VND không đổi trong h phiên. Với vàng và bạc, giá được quy đổi từ quỹ GLD/SLV, nên web ghi rõ đây là chuỗi đại diện, không phải giá giao ngay. Khoảng 80% là khoảng dự báo, không phải xác suất đã được kiểm chứng: mức bao phủ thật được đo riêng bằng `coverage_80` trên dữ liệu chạy thật.

#### Đánh giá bằng pinball loss

```python
def pinball_loss(y: np.ndarray, q_pred: np.ndarray) -> float:
    # y: (n,) return thật; q_pred: (n, 7) phân vị dự báo
    tau = np.array(QUANTILE_LEVELS)
    diff = y[:, None] - q_pred
    return float(np.mean(np.maximum(tau * diff, (tau - 1) * diff)))

# pinball_skill = 1 - loss_model / loss_baseline  (> 0 nghĩa là tốt hơn mốc 252 phiên)
```

Ngoài chỉ số chính, nhóm còn đo:
- **Hiệu chỉnh:** `coverage_80`, `width_80`, tỷ lệ trúng từng phân vị;
- **Xác suất tăng:** `brier_skill`;
- **Chỉ số phụ:** `sign_accuracy`, luôn đi kèm `up_rate`.

Nhóm không dùng MAPE hay RMSE trên mức giá. Để so hai mô hình, nhóm dùng kiểm định Diebold–Mariano với sai số chuẩn HAC.

**Quy tắc đổi mô hình (ADR-0008):** một mô hình học máy chỉ thay mốc khi **cận dưới khoảng tin cậy 95% của `pinball_skill` lớn hơn 0**.

#### Kết quả

- **Validation 2019:** DLinear thua mốc 252 phiên ở **28/28 nhóm** (4 tài sản × 7 kỳ hạn). Mốc 252 phiên được chọn làm champion cho cả 4 tài sản, còn DLinear giữ vai challenger.
- **Test 2020 → nay** (chạy đúng một lần ngày 16/11/2026): ứng viên S1a lấy mốc 252 phiên rồi co giãn khoảng theo tỷ lệ giữa biến động ngắn hạn (EWMA, λ = 0,94) và biến động dài hạn, kẹp trong [0,5; 2]. S1a vẫn chạy song song (shadow) trong Lambda predict từ 04/11 nhưng không hiển thị. So với mốc 252 phiên:

| Tài sản | `pinball_skill` của S1a | CI 95% | Đánh giá |
|---|---:|---|---|
| Vàng | +0,11% | [−1,29%; +1,49%] | Không khác biệt |
| Bạc | +2,12% | [+0,17%; +4,10%] | Dương nhưng không bền: CI chứa 0 khi dùng block 40 |
| WTI | +4,49% | [+2,02%; +6,73%] | Dương |
| Brent | +4,21% | [+1,88%; +6,35%] | Dương |

Skill của S1a âm ở cả 4 tài sản trong năm 2023, và 18/42 kiểm định DM không xác định được. Nhóm quyết định **giữ `baseline-q252` tới hết kỳ thực tập**. Muốn chuyển sang S1a cho dầu và bạc thì phải có ADR mới; nhóm ghi việc này là hướng phát triển.

![Trang Vàng](/images/5-Workshop/5.3.2-asset-page.png)

*Trang Vàng trên web, ngày cơ sở 20/11/2026: nhãn "Mốc thống kê 252 phiên" và 7 thẻ kỳ hạn, mỗi thẻ có xác suất tăng và khoảng 80% theo %. Các mức giá USD đã được che; giá vàng trên web là giá quy đổi từ quỹ GLD, không phải giá vàng giao ngay.*
