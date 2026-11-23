---
title: "Data pipeline"
date: 2025-09-09
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

This part builds the back end that runs without users: the tables, the S3 data lake, the two container images, the daily ingest → predict chain and the rolling-accuracy job. It corresponds to flows 1 and 2 of the [architecture](../5.3-architecture/).

Work from the `infra/` folder with the virtual environment active. The team follows a few rules for every deployment, and they are worth copying:

+ Run `cdk diff <Stack>` first and read it before `cdk deploy <Stack>`.
+ Deploy one stack per command, one person at a time.
+ Do not deploy `CpawsData` between 19:05 and 19:35 Vietnam time, so you do not collide with the 19:15 run.
+ Tag images with the short SHA of the merge commit, never `latest`.

Order when a change touches several stacks: `CpawsServing` (tables) → `CpawsData` (Lambda, schedules) → `CpawsOps` (alarms) → `CpawsApi` (routes). Resources are created first, then the things that read them. `CpawsAuth` only has to exist before `CpawsApi`.

#### Content

1. [Storage and tables](5.4.1-storage-and-tables/)
2. [Ingest and predict](5.4.2-ingest-and-predict/)
3. [Rolling accuracy](5.4.3-rolling-accuracy/)
