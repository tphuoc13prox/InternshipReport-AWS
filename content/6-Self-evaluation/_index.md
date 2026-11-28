---
title: "Self-Assessment"
date: 2025-09-09
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

From **14/09/2026** to **14/12/2026** I was an intern in the First Cloud AI Journey program at **Amazon Web Services Viet Nam Company Limited**, where I could put what I had learned at university into a real project and a real team.  
I worked on **CPAWS – Commodity Price Forecasting on AWS**, a five-person team project, and was responsible for the news and chatbot part: the Bedrock chatbot behind `POST /chat`, its tools, prompts and benchmark, plus the repository CI. Through it I improved my skills in **serverless architecture on AWS, infrastructure as code with AWS CDK, building and evaluating an LLM application on Amazon Bedrock, reviewing code through pull requests, and writing technical documents**.  

I tried to finish my tasks on time, follow the team's working rules (pull requests with review, deploy windows, no secrets in a public repository), and keep in close contact with my teammates so problems were found early.

To look back on the internship as objectively as I can, I rate myself on the criteria below:

| No. | Criteria                            | Description                                                                                      | Good | Fair | Average |
| --- | ----------------------------------- | ------------------------------------------------------------------------------------------------ | ---- | ---- | ------- |
| 1   | **Professional knowledge & skills** | Understanding of the field, applying knowledge in practice, proficiency with tools, work quality | ✅    | ☐    | ☐       |
| 2   | **Ability to learn**                | Ability to absorb new knowledge and learn quickly                                                | ☐    | ✅    | ☐       |
| 3   | **Proactiveness**                   | Taking initiative, seeking out tasks without waiting for instructions                            | ✅    | ☐    | ☐       |
| 4   | **Sense of responsibility**         | Completing tasks on time and ensuring quality                                                    | ✅    | ☐    | ☐       |
| 5   | **Discipline**                      | Adhering to schedules, rules, and work processes                                                 | ✅    | ☐    | ☐       |
| 6   | **Progressive mindset**             | Willingness to receive feedback and improve oneself                                              | ✅    | ☐    | ☐       |
| 7   | **Communication**                   | Presenting ideas and reporting work clearly                                                      | ✅    | ☐    | ☐       |
| 8   | **Teamwork**                        | Working effectively with colleagues and participating in teams                                   | ✅    | ☐    | ☐       |
| 9   | **Professional conduct**            | Respecting colleagues, partners, and the work environment                                        | ✅    | ☐    | ☐       |
| 10  | **Problem-solving skills**          | Identifying problems, proposing solutions, and showing creativity                                | ✅    | ☐    | ☐       |
| 11  | **Contribution to project/team**    | Work effectiveness, innovative ideas, recognition from the team                                  | ✅    | ☐    | ☐       |
| 12  | **Overall**                         | General evaluation of the entire internship period                                               | ✅    | ☐    | ☐       |

### Strengths

- **Seeing my part through to the end:** I took the chatbot from proposing the `POST /chat` contract to running it in production: a Lambda that calls Claude Haiku on Amazon Bedrock with tool use, a daily question limit, a hand-labelled Fed/EIA news set and benchmark sets to measure answer quality.
- **Measuring before concluding:** Each time I changed the prompt or tried the router, I ran it on a question set locked in advance and compared it with a threshold set beforehand. When an idea missed its threshold, such as a router that sends simple questions to a smaller model or automatic news labelling with Bedrock, I recorded the result and chose a simpler approach instead of forcing it into the product.
- **Keeping the shared repository healthy:** I set up CI that runs tests and lint on every pull request in the first weeks of the project, and I regularly reviewed my teammates' PRs, so many bugs were caught before merging.
- **Writing documents for others:** I recorded the key decisions of the chatbot part in ADRs and repository docs, together with the demo script and the Q&A script, so my teammates could read them later and present on my behalf when needed.

### Needs Improvement

- **Machine learning and statistics foundation.** I need a stronger grasp of time-series forecasting and how to evaluate a forecasting model, so I can follow and question the modelling side of a project instead of relying only on teammates.
- **Testing with natural questions early.** Some chatbot bugs, such as missing a news item when the question names no asset, or news dates off by one day because of UTC, only showed up when the team tried the chatbot in production after the freeze. Next time I want to put the kinds of questions users actually ask into the test sets from the start.
- **Splitting time between the project and learning.** In the last weeks I put most of my effort into the project and the report, so I learned little new material. I need a more even plan so I can keep building the product while still studying my weaker areas.

### Reflection

- Before the internship, I thought the hardest part of building a chatbot was choosing the model and writing the prompt. In practice, the most work went into measuring: whether an answer is right or wrong, how right it is, and how much each question costs.
- I also understood better why a chatbot should get its numbers through tools instead of letting the model remember them. When the figures come from the system and the news comes with sources, users can check the answer, and the team can see where it went wrong and fix it.
- Building a product that really runs on AWS made me think about cost from the design stage: every question costs tokens, so question limits, `maxTokens` and watching Cost Explorer matter as much as answer quality. I would like to thank the mentors, the FCAJ organizers and my CPAWS teammates for supporting me during these three months.
