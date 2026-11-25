---
title: "Yêu cầu tiên quyết"
date: 2026-11-20
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---
#### Công cụ trên máy

| Công cụ | Phiên bản | Dùng để |
|---|---|---|
| Python | 3.11 trên máy và CI; 3.12 trong image Lambda | Code dữ liệu, mô hình, Lambda, CDK |
| Node.js + npm | 22.12 trở lên (Vite 8 yêu cầu) | AWS CDK CLI và web (React + Vite) |
| AWS CLI | v2 (bản có lệnh `aws login`) | Đăng nhập và thao tác với AWS |
| AWS CDK CLI | v2 | `npm install -g aws-cdk` |
| Docker có `buildx` | mới nhất | Build container image `linux/amd64` cho Lambda |
| Git | mới nhất | Lấy mã nguồn |

#### Lấy mã nguồn và chạy test

```bash
git clone https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting.git
cd CPAWS-commodity-forecasting
git switch production   # bản nộp (tag v1.0.1); muốn phát triển tiếp thì dùng nhánh developer

python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements-dev.txt  # requirements.txt chỉ chứa thư viện runtime
python -m pytest -q
ruff check .
```

Test phải xanh trước khi đụng tới AWS. Các test `tests/test_infra_*.py` synth CDK để kiểm tra quyền IAM, biến môi trường, lịch Scheduler và alarm, nên cần cài `aws-cdk-lib` (`pip install -r infra/requirements.txt`).

#### Tài khoản AWS

- **Không dùng root hằng ngày.** Root bật MFA và chỉ dùng cho việc cấp tài khoản và billing.
- **Mỗi người một IAM user**, được gán vào group theo vai trò (quản trị, platform, ML, GenAI, chỉ đọc). Mọi thành viên đều bật MFA.
- **Đăng nhập CLI bằng phiên ngắn hạn**, không tạo access key dài hạn:

```bash
aws login --profile cpaws-dev
aws sts get-caller-identity --profile cpaws-dev
```

- **Region:** `ap-southeast-1` (Singapore) cho toàn bộ tài nguyên.
- **Amazon Bedrock:** kiểm tra tài khoản gọi được **Claude Haiku 4.5** qua inference profile global (cross-Region inference) trước khi làm chatbot. Nhóm thử Claude Haiku 5.5 nhưng bị từ chối quota, nên MVP dùng Haiku 4.5 (ADR-0004).

#### Chặn chi phí trước khi tạo tài nguyên

1. Tạo **AWS Budgets** 50 USD/tháng, gửi cảnh báo theo chi phí thực tế và chi phí dự báo.
2. Bật **Cost Anomaly Detection** với monitor mặc định.
3. Bật cost allocation tag `Project`, `Owner`, `Env`. CDK gắn ba tag này cho mọi tài nguyên.

#### Bí mật

- `.env` nằm trong `.gitignore`; repo chỉ commit `.env.example`, chứa tên biến mà không có giá trị.
- API key của EIA lưu trên AWS trong **SSM Parameter Store** dạng `SecureString`. CDK chỉ nhận **tên** tham số, mặc định là `/cpaws/dev/eia-api-key`.

```bash
# nhập key từ bàn phím để key không nằm trong lịch sử shell
read -rs EIA_KEY
aws ssm put-parameter --profile cpaws-dev \
  --name /cpaws/dev/eia-api-key --type SecureString --value "$EIA_KEY"
unset EIA_KEY
```

{{% notice warning %}}
Repo của nhóm để public. Không commit API key, mật khẩu, email cá nhân hay Account ID đầy đủ. Pre-commit chạy `gitleaks` và `detect-aws-credentials` để chặn những thứ này trước khi commit. Trong workshop, `<ACCOUNT_ID>` là chỗ bạn thay bằng tài khoản của mình.
{{% /notice %}}

![AWS Budgets](/images/5-Workshop/5.2-budgets.png)

*AWS Budgets có hai budget: `cpaws-monthly-cost-budget` 50 USD/tháng và một budget 1 USD để báo ngay khi bắt đầu phát sinh chi phí. Cả hai đang ở trạng thái OK.*
