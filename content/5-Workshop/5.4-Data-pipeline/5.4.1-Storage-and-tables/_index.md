---
title: "Storage and tables"
date: 2025-09-09
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### The four DynamoDB tables (`CpawsServing`)

`CpawsServing` creates every table the system reads at request time. All of them use on-demand billing, because traffic is low and uneven.

| Table | Keys | Written by | Read by | Removal policy |
|---|---|---|---|---|
| `cpaws-dev-forecasts` | `asset` + `base_date` | Lambda predict (conditional `PutItem`) | forecasts-api, chat `get_forecast`, accuracy job | `RETAIN` |
| `cpaws-dev-news-labels` | `asset` + `sk` (`<published_at>#<article_id>`) | News collector (planned) | chat `get_news` | `RETAIN` |
| `cpaws-dev-chat-usage` | `pk` + `sk`, TTL on `expires_at` | chat-api (quota counter) | chat-api | `DESTROY` |
| `cpaws-dev-accuracy` | `PK` + `SK` | Accuracy job (conditional write) | accuracy-api | `RETAIN` |

`chat-usage` only stores counters, never questions or answers, which is why it is safe to destroy with the stack. The forecast, news and accuracy tables are kept when the stack is deleted, because they hold published history.

Deploy it:

```bash
cdk diff CpawsServing --profile cpaws-dev
cdk deploy CpawsServing --profile cpaws-dev
```

Check the result:

```bash
aws dynamodb list-tables --profile cpaws-dev --query "TableNames[?starts_with(@, 'cpaws-dev')]"
```

<!-- IMAGE: DynamoDB console, the four cpaws-dev-* tables with capacity mode On-demand -->
[ẢNH: DynamoDB console – four `cpaws-dev-*` tables, on-demand]

#### The S3 data lake (created by `CpawsData`)

The bucket `cpaws-dev-lake` is created in the next step together with the Lambda functions, but its settings matter for the whole pipeline:

+ public access blocked and SSL required;
+ versioning on, with old versions expiring after 30 days;
+ `RETAIN`, so deleting the stack never deletes the data.

Prefixes used by the system:

| Prefix | Content |
|---|---|
| `raw/` | Files as downloaded from the sources |
| `clean/prices/dt=<date>/` | One cleaned price snapshot per run |
| `eval/shadow/s1a-l094/dt=<date>/` | Shadow forecasts of the S1a challenger |
| `eval/shadow_eval/` | Champion vs S1a comparison written by the accuracy job |
| `serving/price-history` | Short price history read by forecasts-api for the 2-year chart (since v1.0.0) |
