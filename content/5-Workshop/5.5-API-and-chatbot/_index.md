---
title: "API and chatbot"
date: 2025-09-09
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

This part builds everything a signed-in user talks to: the Cognito user pool, the REST API with its three Lambda handlers, and the chatbot behind `POST /chat`. It corresponds to flow 3 of the [architecture](../5.3-architecture/).

| Route | Auth | Lambda | Reads |
|---|---|---|---|
| `GET /health` | None | forecasts-api | — |
| `GET /forecasts`, `GET /forecasts/{asset}` | Cognito | `cpaws-dev-forecasts-api` (256 MB, 10 s) | `cpaws-dev-forecasts` |
| `GET /accuracy/{asset}` | Cognito | `cpaws-dev-accuracy-api` | `cpaws-dev-accuracy` |
| `POST /chat` | Cognito | `cpaws-dev-chat-api` (512 MB, 28 s) | `cpaws-dev-forecasts`, `cpaws-dev-chat-usage`, Bedrock |

Deploy `CpawsAuth` first, then `CpawsApi`. The tables from [5.4.1](../5.4-data-pipeline/5.4.1-storage-and-tables/) must already exist.

#### Content

1. [Cognito and REST API](5.5.1-cognito-and-api/)
2. [Bedrock chatbot](5.5.2-bedrock-chatbot/)
