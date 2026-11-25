---
title: "Prerequisites"
date: 2026-11-20
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---
#### Local tools

| Tool | Version | Used for |
|---|---|---|
| Python | 3.11 locally and in CI; 3.12 in the Lambda images | Data, model, Lambda and CDK code |
| Node.js + npm | 22.12 or later (required by Vite 8) | AWS CDK CLI and the web app (React + Vite) |
| AWS CLI | v2 (a version with `aws login`) | Signing in and working with AWS |
| AWS CDK CLI | v2 | `npm install -g aws-cdk` |
| Docker with `buildx` | latest | Building `linux/amd64` container images for Lambda |
| Git | latest | Getting the source code |

#### Get the code and run the tests

```bash
git clone https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting.git
cd CPAWS-commodity-forecasting
git switch production   # submitted version (tag v1.0.1); use branch developer for further work

python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements-dev.txt  # requirements.txt holds runtime libraries only
python -m pytest -q
ruff check .
```

Tests must pass before you touch AWS. The `tests/test_infra_*.py` tests synthesize CDK to check IAM permissions, environment variables, Scheduler schedules and alarms, so they need `aws-cdk-lib` (`pip install -r infra/requirements.txt`).

#### AWS account

- **Do not use root day to day.** Root has MFA and is used only for account setup and billing.
- **One IAM user per person**, placed in groups by role (admin, platform, ML, GenAI, read-only). Every member has MFA enabled.
- **Sign in to the CLI with short-lived sessions**; do not create long-lived access keys:

```bash
aws login --profile cpaws-dev
aws sts get-caller-identity --profile cpaws-dev
```

- **Region:** `ap-southeast-1` (Singapore) for all resources.
- **Amazon Bedrock:** before building the chatbot, check that the account can call **Claude Haiku 4.5** through the global inference profile (cross-Region inference). The team tried Claude Haiku 5.5 but was refused quota, so the MVP uses Haiku 4.5 (ADR-0004).

#### Put cost guards in place before creating resources

1. Create an **AWS Budgets** budget of 50 USD/month, with alerts on actual and forecasted cost.
2. Enable **Cost Anomaly Detection** with the default monitor.
3. Activate the cost allocation tags `Project`, `Owner`, `Env`. CDK applies these three tags to every resource.

#### Secrets

- `.env` is listed in `.gitignore`; the repository only commits `.env.example`, which holds variable names without values.
- On AWS, the EIA API key is stored in **SSM Parameter Store** as a `SecureString`. CDK only receives the parameter **name**, by default `/cpaws/dev/eia-api-key`.

```bash
# type the key at the prompt so it does not end up in shell history
read -rs EIA_KEY
aws ssm put-parameter --profile cpaws-dev \
  --name /cpaws/dev/eia-api-key --type SecureString --value "$EIA_KEY"
unset EIA_KEY
```

{{% notice warning %}}
The team repository is public. Never commit API keys, passwords, personal email addresses or a full Account ID. Pre-commit runs `gitleaks` and `detect-aws-credentials` to block them before a commit. In this workshop, `<ACCOUNT_ID>` is where you put your own account.
{{% /notice %}}

![AWS Budgets](/images/5-Workshop/5.2-budgets.png)

*AWS Budgets has two budgets: `cpaws-monthly-cost-budget` at 50 USD/month and a 1 USD budget that alerts as soon as any cost appears. Both are OK.*
