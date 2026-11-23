---
title: "Prerequisite"
date: 2025-09-09
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### AWS account and Region

+ An AWS account with MFA enabled on the root user. Use the root user only for account-level tasks.
+ Region **`ap-southeast-1` (Singapore)**. The CDK app pins this Region, and every resource in the workshop is created there.
+ AWS Budgets set up before you start. The team uses a budget of 50 USD/month with email alerts (see [5.8](../5.8-cost/)).

#### Amazon Bedrock model access

The chatbot calls **Claude Haiku 4.5** through a **global cross-Region inference profile** from `ap-southeast-1`:

```text
global.anthropic.claude-haiku-4-5-20251001-v1:0
```

Open the Amazon Bedrock console in `ap-southeast-1`, go to **Model access**, and make sure Claude Haiku 4.5 is available to your account before you deploy the API stack.

{{% notice info %}}
A global profile lets Bedrock route a request to another Region, so the team cannot promise that chat questions stay in Singapore.
{{% /notice %}}

<!-- IMAGE: Bedrock console, Model access page showing Claude Haiku 4.5 as available (mask the Account ID) -->
[ẢNH: Bedrock console – Model access, Claude Haiku 4.5 available]

#### IAM permissions

Deploy with a person's IAM identity that signs in with short-term credentials, not with long-term access keys. In the team, the members who deploy belong to a platform group that can deploy infrastructure but cannot administer IAM. To run this workshop your identity needs to:

+ create and update CloudFormation stacks, and use the CDK bootstrap resources;
+ create the IAM roles and policies that CDK generates for Lambda, Scheduler and API Gateway;
+ manage Lambda, ECR, S3, DynamoDB, EventBridge Scheduler, API Gateway, Cognito, CloudWatch (logs, metric filters, alarms), SNS and SSM Parameter Store;
+ call Amazon Bedrock (`bedrock:InvokeModel`) to test the model;
+ manage AWS Amplify Hosting apps.

Sign in with the AWS CLI using short-term credentials, for example:

```bash
aws login --profile cpaws-dev
aws sts get-caller-identity --profile cpaws-dev
```

`cpaws-dev` is the profile name the team uses; any profile name works if you use it consistently in the commands below.

#### Tools

| Tool | Used for |
|---|---|
| AWS CLI v2 | Sign-in, ECR login, checking logs and alarms |
| Python 3.12 + `venv` | Lambda code, tests and the CDK app (`infra/`) |
| Node.js 22.12 or later + npm | AWS CDK CLI and the web app (`web/`) |
| AWS CDK CLI | `cdk diff`, `cdk deploy`, `cdk destroy` |
| Docker | Building the ingest and predict container images |
| Git | Cloning the team repository |

Get the team's source code, [CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting](https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting), and install the dependencies:

```bash
git clone https://github.com/CPAWS-AWS-FCAJ/CPAWS-commodity-forecasting.git
cd CPAWS-commodity-forecasting
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements-dev.txt
pip install -r infra/requirements.txt   # aws-cdk-lib for the CDK app
python -m pytest -q                # unit tests, including CDK synth tests
npm install -g aws-cdk
```

Bootstrap CDK once per account and Region:

```bash
cd infra
cdk bootstrap --profile cpaws-dev
```

#### One secret: the EIA API key

The ingest Lambda reads an EIA API key (used only to cross-check oil prices) from **SSM Parameter Store** as a `SecureString`. The CDK app only receives the **name** of the parameter, `/cpaws/dev/eia-api-key` by default. Never put the key itself in the repository, in a `.env` file or in a CDK context value. Create the parameter once in the Systems Manager console, choosing the type **SecureString**.

<!-- IMAGE: Systems Manager Parameter Store, the /cpaws/dev/eia-api-key parameter of type SecureString (value hidden) -->
[ẢNH: SSM Parameter Store – parameter `/cpaws/dev/eia-api-key`, type SecureString, value hidden]
