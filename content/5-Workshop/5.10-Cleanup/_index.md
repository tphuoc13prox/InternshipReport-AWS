---
title: "Clean up resources"
date: 2025-09-09
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

The team's `dev` environment keeps running until grading is finished. The steps below are for anyone who reproduced the workshop in their own account, and are the order the team will follow afterwards.

#### Step 1 – Close sign-up and the web

1. Self sign-up is open for grading. Close it within one day after grading, and no later than 10/12/2026: turn it off in `CpawsAuth` first, then build the web with `VITE_ALLOW_SIGNUP=false`, then disable or delete the self-created users.
2. In the Amplify console, delete the app `cpaws-dev-web`. It is not part of CDK.

#### Step 2 – Destroy the stacks in reverse order

Readers first, then the resources they read:

```bash
cdk destroy CpawsApi --profile cpaws-dev
cdk destroy CpawsOps --profile cpaws-dev
cdk destroy CpawsData --profile cpaws-dev
cdk destroy CpawsAuth --profile cpaws-dev
cdk destroy CpawsServing --profile cpaws-dev
```

What disappears and what stays:

| Resource | Removal policy | After `cdk destroy` |
|---|---|---|
| Lambda functions, schedules, API, alarms, SNS topic | default | Deleted |
| ECR `cpaws-dev-ingest`, `cpaws-dev-predict` | `DESTROY`, emptied on delete | Deleted with all images |
| `cpaws-dev-chat-usage` | `DESTROY` | Deleted |
| S3 `cpaws-dev-lake` | `RETAIN` | **Kept** |
| `cpaws-dev-forecasts`, `cpaws-dev-news-labels`, `cpaws-dev-accuracy` | `RETAIN` | **Kept** |
| User pool `cpaws-dev-users` | `RETAIN` | **Kept** |

#### Step 3 – Delete the retained resources by hand

Only do this once you are sure the data is no longer needed.

**S3 bucket.** Versioning is on, so the bucket must be emptied of every object version and delete marker before it can be deleted. The easiest way is the console: select `cpaws-dev-lake` → **Empty**, then **Delete**.

{{% notice warning %}}
In the team's account some prefixes are protected by explicit deny statements on `s3:DeleteObject` (evaluation artifacts). Emptying the bucket fails until an administrator removes those statements from the bucket policy and the IAM policies. In a fresh account without them, this does not apply.
{{% /notice %}}

**DynamoDB and Cognito:**

```bash
aws dynamodb delete-table --table-name cpaws-dev-forecasts --profile cpaws-dev
aws dynamodb delete-table --table-name cpaws-dev-news-labels --profile cpaws-dev
aws dynamodb delete-table --table-name cpaws-dev-accuracy --profile cpaws-dev
aws cognito-idp delete-user-pool --user-pool-id <USER_POOL_ID> --profile cpaws-dev
```

#### Step 4 – Resources outside CDK

| Resource | How to remove |
|---|---|
| SSM parameter `/cpaws/dev/eia-api-key` | `aws ssm delete-parameter --name /cpaws/dev/eia-api-key --profile cpaws-dev` |
| CloudWatch log groups `/aws/lambda/cpaws-dev-*` | Delete in the console if any are left (their events expire after 1 month, but the groups themselves stay) |
| AWS Budgets, Cost Anomaly Detection | Keep until the final bill has arrived, then delete in the Billing console |
| CDK bootstrap stack `CDKToolkit` | Delete only if no other CDK app uses this account and Region |

#### Step 5 – Check

```bash
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "StackSummaries[?starts_with(StackName, 'Cpaws')].StackName" --profile cpaws-dev
aws dynamodb list-tables --query "TableNames[?starts_with(@, 'cpaws-dev')]" --profile cpaws-dev
aws s3 ls --profile cpaws-dev | grep cpaws-dev
```

All three should return nothing. Check the Billing console a few days later to confirm no new charges.
