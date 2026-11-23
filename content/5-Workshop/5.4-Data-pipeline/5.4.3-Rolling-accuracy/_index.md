---
title: "Rolling accuracy"
date: 2025-09-09
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

A forecast page is only honest if users can see how past forecasts turned out. CPAWS does this with a small batch job that scores the forecasts it has actually published, so the web never computes anything when a page opens.

#### What the job does

+ **Schedule:** `cpaws-dev-accuracy-daily`, `cron(45 13 ? * TUE-SAT *)` UTC, which is 20:45 Vietnam time, 90 minutes after the 19:15 run.
+ **Input:** published forecasts from `cpaws-dev-forecasts` (`Query` only), the price snapshots in `clean/prices/`, and the S1a shadow files.
+ **Window:** `rolling_8w`, the 56 calendar days of base dates up to the run date. Past forecasts are never back-filled to fill the window.
+ **Metrics:** `coverage_80` (how often the real price landed inside the nominal 80% forecast range), `width_80`, `sign_accuracy` and `up_rate`, for each of the seven horizons (1 to 7 sessions ahead).
+ **Output:** results go to `cpaws-dev-accuracy` with conditional writes; the champion vs S1a comparison goes to `eval/shadow_eval/` and is never overwritten.

Pairs whose actual price has not arrived yet are simply not scored, so the newest base dates only count for the shortest horizons at first.

#### Display gate

Numbers from a handful of days would look precise but mean little. Until at least **20 distinct base dates** are scored, the API returns `data_status = collecting` and the web puts "Đang thu thập (n/20)" ("collecting") where the value would be. On 12/11 all four assets were at 9/20; one more base date is added per run.

#### Deploy and enable

The team rolled this out in small steps so each part could be checked on its own:

| Date | Change | Stack |
|---|---|---|
| 10/11 | Table `cpaws-dev-accuracy` | `CpawsServing` |
| 10/11 | Route `GET /accuracy/{asset}` and the read-only Lambda `cpaws-dev-accuracy-api` | `CpawsApi` |
| 11/11 | Lambda `cpaws-dev-accuracy-job` (image `e31c928`), schedule **off** | `CpawsData` |
| 11/11 | Alarm `cpaws-dev-accuracy-job-errors` | `CpawsOps` |
| 11/11 | Manual dry run, 4 assets | — |
| 12/11 | Schedule turned on; `cdk diff` showed only the schedule `State` | `CpawsData` |
| 13/11 | Web built with `VITE_ACCURACY_API=true` | Amplify |

To switch the schedule on or off, redeploy `CpawsData`; the default is on:

```bash
cdk deploy CpawsData -c accuracy_schedule=off --profile cpaws-dev
```

Only the schedule `State` changes. The Lambda, its IAM role and the data already written stay as they are.

#### Check a run

```bash
aws logs tail /aws/lambda/cpaws-dev-accuracy-job --since 1h --format short --profile cpaws-dev \
  --filter-pattern "?REPORT ?ACCURACY_DONE ?ACCURACY_FAILED ?ACCURACY_NOT_READY ?WARNING ?ERROR ?Traceback"
aws scheduler get-schedule --name cpaws-dev-accuracy-daily --profile cpaws-dev \
  --query "[State,ScheduleExpression]" --output text
```

<!-- IMAGE: CloudWatch Logs, accuracy job log with ACCURACY_DONE for the four assets -->
[ẢNH: CloudWatch Logs – accuracy job with `ACCURACY_DONE`]

| Run | Mode | Duration | Max memory | Result |
|---|---|---:|---:|---|
| 11/11 20:58 | Dry run, manual | 23.7 s | 290 / 1,024 MB | 4 assets, no error |
| 12/11 20:45 | Scheduled | 30.3 s | 297 / 1,024 MB | 4 assets `COMPLETED`, 9 base dates, alarm OK |

The job also logs a `prob_up_mismatch` warning when the stored probability of rising differs from the value recomputed from the quantiles. On 12/11 the largest gap was below 1e-5, caused by rounding to six digits when writing to DynamoDB, so the team kept the threshold and did not change the job.
