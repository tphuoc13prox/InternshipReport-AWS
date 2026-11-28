---
title: "Week 10 Worklog"
date: 2026-11-16
publishDate: 2026-11-13
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:
- Use the AWS Toolkit and Amazon Q in VS Code.
- Hold a team project meeting to finalize the evaluation results and how graders access the web app.
- Chatbot work: check the usage limits, review the docs and refresh the demo news set.
- Start my personal worklog following the team's report requirements.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|:---:|---|:---:|:---:|---|
| Mon | **- Team project meeting: finalize evaluation results and grader access**<br>+ Review the test2020 results, keep the `baseline-q252` model<br>+ Review the report PRs and the web self sign-up feature<br>+ Agree to open Cognito self sign-up during grading (ADR-0014)<br>**- Team project: check the chatbot usage limits**<br>+ Answer Tuấn about the chatbot limits of 20 requests per user per day and 100 requests per day, used for ADR-0014<br>+ Check them again against the quota code, the API stack settings and the unit tests | 16/11/2026 | 16/11/2026 | [DynamoDB items](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html) |
| Tue | **- Team project: review the chatbot docs and the README**<br>+ Review and approve Khải's PR that reworks the chatbot docs<br>+ Review and approve Khải's PR that rewrites the README: overview, architecture, running locally<br>+ Ask the chatbot on the web about the early-October FOMC news to check the answer (passed)<br>+ Get the report requirements from Khải: the Proposal and Workshop are written for the whole team, the worklog is personal, bilingual, with a COST section | 17/11/2026 | 17/11/2026 | [Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html) |
| Wed | **- Practice the lab: AWS Toolkit for VS Code with Amazon Q**<br>+ Install the AWS Toolkit for Visual Studio Code<br>+ Connect the AWS account and change the Region<br>+ Authenticate with IAM credentials and AWS Builder ID<br>+ Work with AWS services: AWS Explorer, Amazon Q, Amazon CodeWhisperer | 18/11/2026 | 18/11/2026 | [AWS Toolkit for VS Code: Amazon Q & CodeWhisperer](https://000087.awsfcaj.com/) |
| Thu | **- Report: write my personal worklog following the team's report requirements**<br>+ Go over the past weeks and record the daily tasks in English and Vietnamese | 19/11/2026 | 19/11/2026 | [FCAJ HCM Rules](https://hcm-rules.awsfcaj.com/) |
| Fri | **- Team project: refresh the chatbot demo news set**<br>+ Look up the new Fed and EIA releases since the 11/11 update, pick 2 and label them by hand following ADR-0013<br>+ Open PR #142: add the FOMC minutes of 27–28/10 (gold, silver) and the WPSR of 18/11 (Brent, WTI) to the chat Lambda's news files, update the demo news doc, add 1 test<br>+ Khải approved it and I merged it | 20/11/2026 | 20/11/2026 | [Bedrock tool use](https://docs.aws.amazon.com/bedrock/latest/userguide/tool-use.html)<br>[Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) |

### Week 10 Achievements:
1. Connected VS Code to AWS through the AWS Toolkit (IAM credentials, AWS Builder ID) and tried Amazon Q.
2. Agreed with the team on the evaluation results, kept `baseline-q252` and agreed to open Cognito self sign-up during grading (ADR-0014).
3. Confirmed the chatbot usage limits against the code and unit tests for ADR-0014.
4. Reviewed the team's chatbot docs and README; refreshed the chatbot demo news set with the FOMC minutes of 27–28/10 and the WPSR of 18/11 (PR #142).
5. Started my bilingual personal worklog following the team's report requirements.
