---
title: "Lessons learned"
date: 2025-09-09
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### What changed between the proposal and the system

| In the proposal | In the running system | Why |
|---|---|---|
| CEEMDAN + DLinear/LSTM as the main model | `baseline-q252`, a 252-session statistical baseline | 13 configurations (DLinear, CEEMDAN + DLinear, volatility scaling, GJR-GARCH + FHS) did not beat it on the 2019 validation year |
| Weekly retraining with Step Functions + SageMaker | Daily rolling window and champion/challenger with a shadow model | Neither the champion nor the challenger has learned parameters, so there is nothing to retrain or host |
| News labelled automatically with Nova, served through a Knowledge Base | Fed/EIA news labelled by hand, packaged with the chat Lambda | Automatic labelling missed its quality thresholds |
| Router: simple questions to Nova, complex ones to Haiku | Every question goes to Haiku | The router scored 26/30 on the held-out set, below the locked 90% threshold |
| — | Rolling accuracy job, shadow S1a, six alarms, chat quota | Added along the way to measure live forecasts and keep cost bounded |

None of these are missing features. Each change followed an experiment whose plan and threshold were fixed before it ran.

#### Lessons from the whole project

+ **A strong, simple baseline is worth more than a complex model that does not beat it.** With price-only data at 1–7 sessions, the direction of short-term returns is close to unpredictable, and most of the value of a quantile forecast is the width of the range. The 252-session baseline gets that width right, so the team shipped it and labelled it honestly as a statistical baseline, not a forecasting model.
+ **Lock the test before looking at results.** Locked questions and code (SHA-256), thresholds written down in advance, and one official run per set made it possible to report negative results without arguing about them afterwards: the router, prompt v3 and prompt v4 were all rejected this way.
+ **Design for partial failure.** Ingest calls predict even when one source fails; the shadow cannot block the champion; conditional writes make every re-run safe.
+ **Make rollback a parameter, not a rebuild.** Image tags, the shadow switch, the accuracy schedule and CORS can all be reverted with one `cdk deploy -c …`.
+ **Measure live, not only in backtests.** The rolling accuracy job and the 20-date display gate show users how published forecasts actually turned out, and hide numbers that are not yet meaningful.

#### What I learned from the chatbot

+ **Shape the tool output, not only the prompt.** The biggest jump in quality came from giving the model fewer, pre-computed numbers (q10/q50/q90 in %, probability already in %, no gold price) instead of adding more rules to the prompt.
+ **Enforce important rules in code.** Citations are built by the server from the tool results, the disclaimer is always in the response, and error messages never leak internal names. The model can ignore a prompt; it cannot ignore the server.
+ **Put a hard cap on anything that grows with users.** The per-user and global quota turn an open-ended Bedrock bill into a known worst case.
+ **A dev set says little once you have tuned on it.** The router was 83/83 on the questions it was written with and 26/30 on questions it had never seen.

#### What is still weak

+ **Manual operations.** One environment, deployed from a laptop; `-c` values are not saved, so `allowed_origin` or `alarm_email` can be forgotten; the Amplify zip is uploaded by hand.
+ **Alarms cannot see a run that never happened.** Missing data counts as OK, so a schedule that does not fire is only caught by the manual check.
+ **One CORS origin**, so localhost and Amplify cannot call the API at the same time.
+ **Chatbot quality.** Set B is still below its threshold; the remaining errors are about reading numbers. Guardrails are not attached.
+ **Data.** Gold and silver come from GLD and SLV, not spot prices, and the main price source is an unofficial library with known terms-of-use risk.

#### Next steps

+ An alarm on ingest `Invocations` that fires after three days without a run.
+ Connect Amplify to GitHub so each deployment is tied to a commit, and support more than one CORS origin.
+ Experiments with signals beyond price (volatility indices, event calendars, news labels), each with its own pre-registered plan.
+ Deploy v1.0.1 so the chatbot knows the current date, and split the single `dev` environment into separate development and production environments.
+ Attach Bedrock Guardrails and add an alarm on `citation_link_mismatch`.
