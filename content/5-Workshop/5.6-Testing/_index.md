---
title: "Testing and monitoring"
date: 2026-11-20
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---
#### CI on GitHub Actions

Every push and pull request runs:

- **Python:** `ruff check`, `ruff format --check`, `pytest`. This includes CDK synth tests (`tests/test_infra_*.py`) that check IAM permissions, environment variables, Scheduler schedules and alarms.
- **Web:** `npm ci`, `typecheck`, `test`, `build`.
- **Pre-commit:** `gitleaks`, `detect-aws-credentials`, to block keys and Account IDs before they reach the public repository.

CI **does not deploy**. Branch `developer` needs at least 1 approval to merge; branch `production` only accepts release PRs from `developer`.

In the submitted version (branch `production`, commit `ac1a31a`, same content as `developer`): **pytest 1,658 tests passed, 2 skipped** (including the CDK synth tests), **web 265 tests passed**, `ruff` reports no issues, and CI was green on every PR before merge.

#### Tests after each deployment

| Check | How | Expected |
|---|---|---|
| Ingest can reach its sources | `aws lambda invoke ... --payload '{"dry_run": true}'` (section [5.4.1](../5.4-infrastructure/5.4.1-data-stack/)) | About 10 rows per symbol, `error = null` |
| API without a token | `curl "$API/forecasts/gold"` | `401` |
| API with a token | `curl -H "Authorization: $ID_TOKEN" "$API/forecasts/gold"` | `200`, with `base_date`, `model_version` |
| Health | `curl "$API/health"` | `200` |
| CORS | Open the web app on Amplify and look at the browser's Network tab | Preflight returns the right origin, no CORS errors |
| Chatbot | Ask one forecast question and one news question in the web app | `200`; numbers in the answer match the page; news has links |
| Reliability | Call `/accuracy/gold` | `200`, `data_status = collecting` (fewer than 20 base dates) or `ready` |

#### Checking the 19:15 run

```bash
# predict logs: REPORT lines (duration, memory) and event codes
aws logs tail /aws/lambda/cpaws-dev-predict --since 90m --format short --profile cpaws-dev \
  --filter-pattern "?REPORT ?PREDICT_DONE ?PREDICT_FAILED ?PREDICT_ALREADY_PUBLISHED ?PREDICT_SHADOW"

# alarm states
aws cloudwatch describe-alarms --profile cpaws-dev \
  --query "MetricAlarms[].[AlarmName,StateValue]" --output text
```

The `PREDICT_DONE` line contains the latest session price, so never paste that log line into the public repository or a PR.

#### Failure cases checked

| Failure case | Behaviour | Evidence |
|---|---|---|
| One price source fails | Other assets still update; ingest raises `IngestSourceFailed` and the alarm sends email | Test with a source failing for 7 sessions in a row; the next run backfills |
| Predict runs twice on the same day | The second run logs `PREDICT_ALREADY_PUBLISHED` and overwrites nothing | Conditional `PutItem` test |
| More than 20 questions in a day | `/chat` returns `429 rate_limited` | Conditional DynamoDB transaction test |
| Out of time before calling the model | `/chat` returns `504` before counting the question | Deadline test |
| Overwriting test2020 results | Writing the same key again is refused (HTTP 412); deleting an object is blocked by an explicit deny | Checked with the AWS CLI on 04/11 and 11/11/2026 |

#### Monitoring

Stack `CpawsOps` creates 6 CloudWatch alarms. Each fires on **≥ 1 error in 5 minutes** and sends email through SNS topic `cpaws-dev-alarms`:

| Alarm | Watches |
|---|---|
| `cpaws-dev-ingest-errors` | Ingest Lambda errors: a main source failed or the Lambda crashed |
| `cpaws-dev-predict-errors` | Predict Lambda errors: an asset was skipped or the Lambda crashed |
| `cpaws-dev-predict-shadow-errors` | Metric filter on predict logs: the S1a shadow failed while the champion was still published |
| `cpaws-dev-accuracy-job-errors` | Reliability job errors |
| `cpaws-dev-chat-api-errors` | Chat Lambda timeout or crash |
| `cpaws-dev-chat-api-internal-errors` | Metric filter: the chat handler returned `500` because of an unexpected error |

All Lambda log groups keep logs for 1 month.

#### Reliability measured on live data

Job `cpaws-dev-accuracy-job` runs at 20:45 Vietnam time, Tuesday–Saturday. It compares published forecasts with realized prices now known and computes `coverage_80`, `width_80` and the other metrics over the last 8 weeks. Until there are 20 base dates, the web app shows "Đang thu thập (n/20)" ("Collecting") instead of numbers. The first scheduled run took place on 12/11/2026: 4/4 assets completed in 30.3 seconds, the alarm stayed OK, at level 9/20. By the run of 21/11/2026, the last one before the freeze, all 4 assets were at 16/20.

#### Remaining limitations

- The alarms cannot detect a schedule that **did not run**, because missing data is treated as normal. For now the team checks by hand after each 19:15 run. Planned fix: an alarm on the ingest Lambda's invocation count that fires when there is no run for 3 days in a row.
- There is only one `dev` environment, deployed by hand from one machine, with no staging.

![CloudWatch alarms](/images/5-Workshop/5.6-alarms.png)

*The 6 CPAWS alarms in CloudWatch are all OK with Actions enabled; each one fires on 1 or more errors within 5 minutes.*

![Accuracy page](/images/5-Workshop/5.6-reliability.png)

*Top of the Accuracy page: the assets are still "Đang thu thập (16/20)" (collecting), counted from 29/10/2026, so the web app does not show an official result yet.*
